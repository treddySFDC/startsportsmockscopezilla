# Solution Document — StartSports Match Operations

**Project:** StartSports Match Operations  
**Phase:** Phase 1 (Demo / PoC)  
**Date:** 2026-11-03  
**Status:** Architecture approved, estimates approved

---

## Architecture Foundations

### Platform

Salesforce core platform only — no Industry Cloud, no Data Cloud, no Marketing Cloud. All functionality is built on Platform + Agentforce. This keeps the build focused, the licensing simple, and the demo self-contained.

### Org Strategy

Single persistent Developer Edition org (confirmed — no scratch org, no DevHub required). All metadata lives in one org: custom objects, agent configuration, and backing Apex logic. `[assumption: confirmed by user — persistent DE org]`

### Automation Architecture — The Key Decision

The scheduling logic (E03) **must be Apex, not Flow**. Flow cannot perform reliable datetime overlap arithmetic across multiple records without Apex. The pattern throughout is:

> Agentforce Action → calls an `@InvocableMethod` Apex class → runs SOQL + business rule logic → returns results or structured error message back to the agent

Each of the five agent Actions calls its own dedicated Apex invocable class. This compound-per-action pattern — not a single god-class — is the defensible, testable approach: each class has a single responsibility, which keeps governor limits manageable and test coverage straightforward. `[extends: agent-customization-with-apex Trailhead trail]`

### Sharing / Security

Standard OWD + Profile-based access. Two permission sets:

- `Match_Organiser` — agent surface access + Read/Create/Edit on all five custom objects
- `Match_Admin` — full CRUD on master data objects (Team, Stadium, Staff Member)

No territory management or Apex-managed sharing required. `[assumption: simple internal two-persona model with no complex row-level sharing]`

### DevOps / Deployment

SFDX project with metadata API deployment into the persistent DE org. No CI/CD pipeline for Phase 1. Deploy once; reset between demo runs via an anonymous Apex reset script.

---

## Solution by Business Process

### E01 — Data Model & Custom Objects · Size: M

The entire application rests on five net-new custom objects. Nothing exists in Salesforce today.

#### Object Map

| Object | Relationship | Key Fields |
|---|---|---|
| `Team__c` | — | Name, City__c |
| `Stadium__c` | — | Name__c, City__c, Capacity__c |
| `Match__c` | ← Team (×2), ← Stadium | Home_Team__c, Away_Team__c, Stadium__c, Start_DateTime__c, End_DateTime__c, Status__c |
| `Staff_Member__c` | — | Name, Role__c (picklist), Is_Active__c |
| `Match_Assignment__c` | Master-Detail → Match, Lookup → Staff Member | Match__c (M-D), Staff_Member__c, Role__c |

#### Design Decisions

- **Master-Detail on Match_Assignment → Match** — assignments are owned by and cascade-delete with their Match. No orphaned assignments. `[assumption: standard cascade-delete pattern; no reason to retain assignments after Match deletion]`
- **`End_DateTime__c` is required** — E03 overlap detection is impossible without it. Both Start and End datetime fields are required fields on Match. `[assumption: confirmed in gap-fill session]`
- **Validation rule — Set is final** — blocks any edit to a Match_Assignment record when the parent Match `Status__c = 'Set'`. Enforces the "no un-confirm" decision at the database level, regardless of how the record is accessed.
- **Duplicate rule** — unique constraint on (`Match__c`, `Staff_Member__c`) — one person per match, database-enforced.
- **Match status lifecycle** — `Draft` → `Staffing` (first assignment created) → `Set` (Action 5 threshold check passes).

#### Admin UI

Standard Lightning Experience page layouts. Admin manages Teams, Stadiums, and Staff Members through standard list views and record pages. No custom UI required for this persona. `[assumption: standard Lightning page layouts are sufficient; no Admin-specific LWC needed]`

---

### E02 — Agentforce Agent Configuration · Size: L

The agent is the Organiser's entire interface. Everything here is about what the agent understands, what it refuses, when it confirms, and how clearly it communicates results.

#### Agent Structure

| Component | Value |
|---|---|
| Agent name | Match Operations Agent |
| Surface | Internal Salesforce app — Lightning / Agent Studio |
| Topic | `Match_Operations` |
| Topic instructions | "You help the Organiser assign crew to upcoming matches. Always ask for confirmation before assigning. Never remove assignments unless asked. If asked about anything outside match crew scheduling, politely decline." |

#### Five Actions

| # | Action | Backing class | What it returns |
|---|---|---|---|
| 1 | Get Teams | `GetTeamsAction` | List of team names |
| 2 | Get Upcoming Matches | `GetUpcomingMatchesAction` | Match name, teams, date/time, stadium, status |
| 3 | Check Staff Availability | `StaffAvailabilityChecker` | Available staff members for the requested role and match window |
| 4 | Assign Staff | `StaffAssigner` | Confirmation message or conflict error |
| 5 | Confirm Match | `MatchConfirmer` | Crew summary (counts + names per role) or threshold error |

#### Confirmation Gate

Action 4's agent instruction: *"Before calling Assign Staff, state the name and role of the staff member you are about to assign and ask the Organiser: 'Shall I assign [Name] as [Role]?' Only call Assign Staff after the Organiser confirms."*

The database write happens inside Action 4's Apex — not an intermediary Flow step. The confirmation is a conversational gate enforced by instruction, not a technical lock.

#### Permission Set

`Match_Organiser` — assigned to the demo user. Grants:
- Agentforce agent surface access
- Read/Create/Edit on Match__c, Match_Assignment__c, Team__c, Stadium__c, Staff_Member__c

`[assumption: standard Salesforce license covers internal agent access for Phase 1 demo; no separate Agent User license required at this scale]`

---

### E03 — Scheduling Logic & Business Rules · Size: L

This is the core of the build — and the highest-risk epic. All of the agent's intelligent answers come from here.

#### Three Apex Invocable Classes

**`StaffAvailabilityChecker` (backs Action 3)**

Checks two things:
1. **Stadium conflict** — is the stadium already booked for another match in the same window?
2. **Staff availability** — which staff members are active, have the right role, and have no time-overlapping assignment?

Overlap detection SOQL:
```apex
// Staff with conflicting assignments
SELECT Staff_Member__c
FROM Match_Assignment__c
WHERE Match__r.Start_DateTime__c < :requestedEndDT
  AND Match__r.End_DateTime__c > :requestedStartDT
  AND Match__r.Id != :matchId
  AND Match__r.Status__c != 'Set'

// Stadium conflict check
SELECT Id FROM Match__c
WHERE Stadium__c = :stadiumId
  AND Id != :matchId
  AND Start_DateTime__c < :requestedEndDT
  AND End_DateTime__c > :requestedStartDT
  AND Status__c != 'Set'
```

The touching-times boundary condition (`start B < end A`, not `<=`) is intentional — two matches back-to-back at the same stadium are a conflict. This matches the gap-fill decision.

Returns: list of available `Staff_Member__c` records (Name + Role) for the requested role, or a stadium-conflict error if the window is blocked.

**`StaffAssigner` (backs Action 4)**

1. Re-runs availability check inline — defensive re-validation catches race conditions or state changes between the Organiser's decision and the write.
2. Inserts `Match_Assignment__c` record.
3. If Match `Status__c = 'Draft'`, updates to `'Staffing'` (first assignment triggers the status move).
4. Returns success message with staff name/role, or structured conflict error.

**`MatchConfirmer` (backs Action 5)**

1. Queries Match_Assignment__c grouped by Role__c for the match.
2. Checks against thresholds (hardcoded Phase 1 constants):
   - Ground Staff ≥ 4
   - Commentator ≥ 2
   - Camera Operator ≥ 3
3. If all pass: updates Match `Status__c = 'Set'`, returns crew summary (counts + names per role).
4. If any fail: returns "Need X more [Role] before this match can be confirmed."

Hardcoded thresholds as `private static final Integer` constants. Custom Metadata is the Phase 2 upgrade path if configurable thresholds are required. `[assumption: hardcoded constants are sufficient for Phase 1 demo]`

#### Test Coverage Plan

Non-trivial test scenarios required:
- **Touching times** — B starts exactly when A ends → conflict
- **Fully overlapping** — B wholly inside A's window → conflict
- **No conflict** — separate windows → staff is available
- **Partial crew + return** — assign 2 staff, come back and assign 2 more → match stays Staffing
- **Stadium conflict** — two matches at same stadium in overlapping window → error returned
- **Minimum threshold failure** — attempt Confirm with 3 ground staff (below the 4 minimum) → correct error message
- **Confirm success** — all thresholds met → status flips to Set, crew summary returned

#### Governor Limits

No concern at demo scale (5 matches × ~20 assignments). SOQL runs synchronously in the Agentforce action context; the 100-SOQL limit is trivially met. No async Apex required. `[assumption: Phase 2 scaling to hundreds of matches/staff would require SOQL optimisation — not in scope for Phase 1]`

---

### E04 — Demo Environment & Seed Data · Size: S

Everything above is code and configuration. This epic makes it runnable and repeatable.

#### Deployment Sequence

1. `sf project deploy start` — deploys custom objects, field definitions, validation rules, duplicate rules, Apex classes, permission sets, agent topic/actions
2. Anonymous Apex seed data script — creates the demo dataset
3. `sf org assign permset -n Match_Organiser` — assigns permission set to demo user
4. Manual verify in Agent Studio — confirm agent is active and all 5 actions are linked
5. Run the four-step demo flow end-to-end as a smoke test

#### Seed Data Design

| Entity | Count | Notes |
|---|---|---|
| Teams | 5 | Named for the demo script |
| Stadiums | 3 | One shared across two matches to enable stadium conflict demo |
| Staff Members | 18 | 6 per role (Ground Staff, Commentator, Camera Operator) |
| Matches | 5 | Match 3 is the demo focus match |
| Pre-seeded assignments | 3 | On Match 3: 2 Camera Operators already assigned — creates visible availability conflict for demo step 2 |

At least 1 staff member per role has a time-overlapping assignment on a different match, so the availability check in demo step 2 shows realistic results. `[assumption: confirmed in gap-fill — seed data designed to support realistic demo; builder writes demo script after seed data is finalized to ensure name alignment]`

#### Reset Script (Anonymous Apex)

```apex
delete [SELECT Id FROM Match_Assignment__c];
List<Match__c> matches = [SELECT Id, Status__c FROM Match__c];
for (Match__c m : matches) { m.Status__c = 'Draft'; }
update matches;
System.debug('Reset complete. Assignments deleted, Match statuses reset to Draft.');
```

Run before each demo session. Leaves Teams, Stadiums, Staff Members, and Match records intact — only wipes assignments and resets statuses. Takes under 5 seconds at demo data volume.

---

## Estimate Summary

*Sizes express relative complexity, not effort. Not hour-convertible. For a timeline range: `roadmap`. For indicative pricing: `commercials` (requires a validated rate).*

| Epic | Size | Primary Driver |
|---|---|---|
| E01 — Data Model & Custom Objects | **M** | 5-object relational model with M-D and lifecycle rules |
| E02 — Agentforce Agent Configuration | **L** | 5 custom actions, instruction engineering, confirmation gate |
| E03 — Scheduling Logic & Business Rules | **L** | Custom Apex overlap detection, 3 invocable classes, multi-condition rules |
| E04 — Demo Environment & Seed Data | **S** | SFDX deployment + seed data + reset script |

**Benchmark orientation (Agentforce internal use case, 4 epics, 1×S + 1×M + 2×L):** The Agentforce parametric table suggests an 8–12 week benchmark range for this profile. This is a benchmark-derived orientation, not a commitment — based on model training data, not this specific engagement. For a committed timeline, run `roadmap`.

---

## Risks

| Risk | Severity | Mitigation |
|---|---|---|
| Overlap detection bug silently breaks demo | High | Comprehensive Apex test coverage across all conflict scenarios before demo day |
| Agent instruction quality (natural-language mapping) | Medium | Iterative instruction testing with realistic Organiser prompts; test out-of-scope requests |
| Deployment sequence failure (first deploy) | Low | SFDX deploy with `--dry-run` first; post-deploy smoke test checklist |
| Demo reset forgotten before a session | Low | Documented reset procedure; runner verifies after each run |

---

## What Is Not in Scope (Phase 1)

- Ticketing, payments, fan-facing features
- Staff payroll and travel
- Live scores or match results
- Automatic schedule generation
- Agent channels other than internal Salesforce app (Slack, Experience Cloud, mobile)
- Configurable crew thresholds (hardcoded Phase 1; Custom Metadata upgrade is Phase 2)
- Un-assign action (agent declines politely if asked)

---

*Solution document produced by Scopezilla design · 2026-11-03*
*Sizes are relative complexity, not effort. Benchmark orientation carries the Benchmark Disclaimer (source: model-training-data).*
