# Delivery Plan — StartSports Match Operations

**Project:** StartSports Match Operations  
**Phase:** Phase 1 (Demo / PoC)  
**Date:** 2026-11-04  
**Timeline:** Sequence only — no week commitment. Phases show build order and dependencies.

---

**Critical path:** E01 → E03 → E02 → E04 — every epic gates the next; slippage on any one cascades.

---

## Phase 1 — Foundation

**Epics:** E01 — Data Model & Custom Objects (Size: M)  
**Dependencies:** None — build starts here.

**Objectives**  
Deploy all 5 custom objects, their relationships, picklists, validation rules, and permission sets. This phase locks the schema the rest of the build runs against.

**Success criteria**
- All 5 objects deployed: Team, Stadium, Match, Staff Member, Match Assignment
- Match lifecycle enforced: Draft → Staffing → Set (validation rule blocks edits on Set matches)
- Duplicate rule blocks double-assignment on (Match, Staff Member)
- Permission sets created: Match_Organiser (agent surface + CRUD on all 5 objects), Match_Admin (full CRUD on master-data objects)

**Risks**  
Field schema changes here cascade to E03 (SOQL uses Start_DateTime__c, End_DateTime__c) and E02 (agent action inputs reference field API names). Freeze the schema before Phase 2.

---

## Phase 2 — Scheduling Logic & Agent

**Epics:** E03 — Scheduling Logic & Business Rules (Size: L) → E02 — Agentforce Agent Configuration (Size: L)  
**Dependencies:** Phase 1 complete. Build E03 before E02 — Apex invocable classes must exist before agent Actions can reference them.

**Objectives**  
Build the Apex scheduling engine and wire it to the Agentforce agent.

**E03 — Apex invocable classes (build first)**
- `StaffAvailabilityChecker` — datetime overlap SOQL + stadium conflict check
- `StaffAssigner` — re-validate inline → insert Match_Assignment → update Match status to Staffing
- `MatchConfirmer` — threshold check (4/2/3 per role) → update Match status to Set → return crew summary

**E02 — Agent configuration (after E03)**
- Topic: Match_Operations with 5 Actions, each backed by an Apex @InvocableMethod
- Confirmation gate: agent must state name + role and get explicit Organiser approval before calling Assign Staff
- Guardrails: agent declines out-of-scope requests (ticketing, scores, payroll, un-assign)

**Success criteria**
- All 3 Apex classes pass at >75% test coverage, including touching-times boundary, stadium conflict, and threshold-failure scenarios
- Agent routes all 5 action types correctly from natural-language Organiser input
- Confirmation gate fires on every assign request without exception
- Agent declines out-of-scope requests politely

**Risks**  
Agent instruction quality is iterative — allow time for prompt tuning with realistic test conversations before declaring E02 done. Overlap detection bug silently breaks the demo — test coverage is the only mitigation.

---

## Phase 3 — Demo Ready

**Epics:** E04 — Demo Environment & Seed Data (Size: S)  
**Dependencies:** Phases 1 and 2 complete. All objects, Apex, and agent configuration deployed before seeding.

**Objectives**  
Deploy to the persistent Developer Edition org, seed the demo dataset, verify the reset script, and rehearse the demo flow.

**Deployment sequence (order matters)**
1. `sf project deploy start` — objects, fields, Apex, permission sets, agent topic/actions
2. Anonymous Apex seed data script — 5 teams, 3 stadiums, 18 staff, 5 matches, 3 pre-seeded assignments on Match 3
3. `sf org assign permset -n Match_Organiser` — assigns to demo user
4. Manual verify in Agent Studio — confirm agent active, all 5 actions linked
5. End-to-end smoke test — run the full 4-step demo flow

**Reset script** (run before each demo session)
```apex
delete [SELECT Id FROM Match_Assignment__c];
List<Match__c> matches = [SELECT Id, Status__c FROM Match__c];
for (Match__c m : matches) { m.Status__c = 'Draft'; }
update matches;
```

**Success criteria**
- Full 4-step demo flow completes without errors
- Reset script clears assignments and resets statuses in under 5 seconds
- Demo owner has rehearsed the flow end-to-end at least once

**Risks**  
Deployment sequence matters — a metadata-order error fails silently on agent config. Use `--dry-run` on first deploy. Demo owner must run the reset before every session.

---

## Cross-Cutting Activities

**Testing (throughout Phase 2)**  
Apex test classes with realistic conflict scenarios are required before Phase 3. Non-trivial scenarios: touching times, fully overlapping matches, stadium conflict, partial crew across sessions, threshold failure. Aim for >75% test coverage.

**Deployment (Phase 3)**  
SFDX project with metadata API. No CI/CD for Phase 1. One deploy to the persistent DE org; reset script handles repeatability.

**Demo rehearsal (end of Phase 3)**  
Demo owner must run the full flow at least once and know the reset procedure before any stakeholder session.

---

## Consolidated Risks

| Risk | Phase | Severity | Mitigation |
|---|---|---|---|
| Schema change after Phase 1 freeze | 1→2 | High | Lock all field API names at end of Phase 1; communicate to builder |
| Overlap detection bug silently passes tests | 2 | High | Comprehensive Apex test scenarios including boundary conditions |
| Agent instruction quality (confirmation gate breaks) | 2 | Medium | Iterative prompt testing with realistic Organiser inputs |
| Deployment sequence failure on first deploy | 3 | Low | `--dry-run` flag; post-deploy smoke test checklist |
| Demo reset forgotten before a session | 3 | Low | Document reset procedure; runner verifies after each session |

---

## What Is Not in Scope (Phase 1)

- Ticketing, payments, fan-facing features
- Staff payroll and travel
- Live scores or match results
- Automatic schedule generation
- Agent channels other than internal Salesforce app (Slack, Experience Cloud, mobile)
- Configurable crew thresholds — hardcoded Phase 1; Custom Metadata is the Phase 2 upgrade path
- Un-assign action

---

## Next Steps

The disciplines and named roster to deliver this — with defensible counts, per lane — come from `estimate`.

For the build handoff bundle (intents, per-phase briefs, org rules): run `quantum-leap`.

---

*Delivery plan produced by Scopezilla roadmap · 2026-11-04*  
*Timeline: sequence only — no commitment. No durations derived or stated.*
