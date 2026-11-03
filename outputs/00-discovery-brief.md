# Discovery Brief — StartSports Match Operations

**Project:** StartSports Match Operations  
**Date:** 2026-11-02  
**Phase:** Phase 1 (Demo / PoC)  
**Engagement type:** Standard scoping  

---

## Executive Summary

StartSports is a fictional sports tournament organiser used as a Salesforce PS demo asset. The Phase 1 goal is to replace a manual, spreadsheet-driven crew-assignment process with an Agentforce agent that allows an organiser to view upcoming matches, check crew availability, assign staff, and confirm a match as "Set" — all in natural language inside the Salesforce app. This is a net-new scratch-org build with seeded demo data; there is no real data migration or external customer to validate against.

---

## Company and Industry Context

- **Client:** StartSports (demo/internal asset — no public company footprint, no Org62 opportunity)
- **Domain:** Amateur / semi-professional sports tournament management
- **Current state:** Spreadsheet-based tracking of teams, fixtures, stadiums, and match-day crew; crew assignment is manual; double-booking is common
- **No Salesforce Sports & Entertainment Industry Cloud** — Salesforce does not publish a vertical accelerator for amateur tournament operations; this build uses core platform only

**Market note:** Specialist SaaS tools (TeamSnap, SportsEngine, LeagueApps) dominate amateur sports crew/schedule management. This Agentforce demo is a differentiated, non-standard use case that positions AI-native internal operations as an alternative to vertical SaaS.

---

## Current vs. Target Salesforce Landscape

| Dimension | Current | Target (Phase 1) |
|---|---|---|
| Environment | None (spreadsheets) | Net-new scratch org |
| Data store | Excel/Google Sheets | 5 custom Salesforce objects |
| User interface | Manual spreadsheet entry | Internal Salesforce app (Lightning / Agent Studio) |
| Intelligence | None | Agentforce agent — natural language for organiser workflow |
| Crew assignment | Ad-hoc, error-prone | Agent-assisted with conflict-checking and confirmation gate |

---

## Project Scope and Objectives

### Business objectives
1. Eliminate manual, error-prone crew assignment from spreadsheets
2. Prevent double-booking (staff across overlapping matches; stadiums hosting simultaneous fixtures)
3. Enforce minimum crew thresholds before a match can be confirmed as Set
4. Demonstrate Agentforce as a viable internal operations surface

### Data model (Phase 1)

| Object | Key Fields |
|---|---|
| Team | Name, City |
| Stadium | Name, City, Capacity |
| Match | Team A, Team B, Date/Time, Stadium, Status (Draft → Staffing → Set) |
| Staff Member | Name, Role (Ground Staff / Commentator / Camera Operator), Status |
| Match Assignment | Match, Staff Member, Role |

### Agent capabilities

| # | Capability |
|---|---|
| 1 | List participating teams |
| 2 | List upcoming matches with date, time, and stadium |
| 3 | Check staff availability by role for a given match date/time |
| 4 | Assign staff to a match for a role (with organiser confirmation) |
| 5 | Confirm a match (returns crew summary; enforces minimum thresholds) |

### Business rules (all Phase 1, all fixed)
- A staff member cannot be assigned to two time-overlapping matches
- Minimum crew to mark a match Set: **4 ground staff, 2 commentators, 3 camera operators** (fixed; applies to all stadiums)
- One role per person per match — no dual-role assignments
- No backup/reserve concept — all assignments are active crew
- If no staff member is available for a required role: agent alerts and stops; no auto-reassignment
- Agent recommends and assigns only after organiser confirms; never removes assignments without being asked
- A stadium cannot host two matches simultaneously

### Out of scope (Phase 1)
- Ticketing, payments, fan-facing features
- Staff payroll and travel
- Live scores
- Automatic schedule generation
- Phase 2 capabilities (undefined — see Open Questions)

### User personas

| Persona | Capabilities |
|---|---|
| Organiser | Interacts with the agent; views and assigns crew |
| Admin | Manages master data (teams, stadiums, staff) |

No license count confirmed — demo context; actual seat sizing deferred to Phase 2 planning.

---

## Technical Architecture (Phase 1)

**Platform:** Salesforce (scratch org)  
**Agent surface:** Internal Salesforce app — Lightning / Agent Studio  
**Agentforce pattern:**
- **Topic:** Match Operations
- **Actions:** Get Teams · Get Upcoming Matches · Check Staff Availability · Assign Staff · Confirm Match
- **Backing logic:** Flows or Apex invocables (one per action); conflict-detection logic lives here

**Complexity driver:** The scheduling conflict logic — overlap detection across matches, minimum crew enforcement, stadium double-booking prevention — is entirely custom. There is no native Agentforce built-in for resource scheduling. This is the highest-risk element of the build.

**Relevant Trailhead build paths:**
- *Agent Customization with Flows* (`agent-customization-with-flows`) — 70 min
- *Agent Customization with Apex* (`agent-customization-with-apex`) — 30 min
- *Build Agentforce Solutions with Low-Code Tools* — 350 min (full trail)

---

## Data and Compliance Considerations

- **Data migration:** None for Phase 1 — small demo dataset seeded manually into the scratch org
- **Compliance/regulatory:** No signals (GDPR, HIPAA, PCI) in source material — expected given demo context
- **Data residency:** Not applicable for Phase 1

---

## Demo Flow

| Step | Organiser says | Agent does |
|---|---|---|
| 1 | "Show upcoming matches." | Lists all upcoming matches with date, time, stadium |
| 2 | "Who's available for Match 3 as a camera operator?" | Checks availability; lists eligible staff |
| 3 | "Assign these three." | Asks for confirmation, then assigns |
| 4 | "Confirm the match." | Shows final summary: A vs B, date/time, stadium, crew counts |

---

## Risks and Constraints

| Risk | Severity | Notes |
|---|---|---|
| Conflict-detection logic complexity | Medium | Overlap-checking, minimum crew enforcement, stadium double-booking are all custom — no platform shortcut |
| Agent action reliability | Medium | Natural language → correct action mapping requires careful topic/instruction design and testing |
| Phase 1 → Phase 2 scope creep | Low (demo context) | Phase 2 undefined; Phase 1 boundary is well-drawn |
| No real user feedback loop | Low | Demo asset — no end-user validation until a real engagement follows |

---

## Design Decisions Closed (Gap Review — 2026-11-02)

The following were open gaps; all resolved in the gap-fill session:

| Gap | Decision |
|---|---|
| Match time window | Match has `Start_DateTime__c` + `End_DateTime__c`. Touching times (start B ≤ end A) count as a conflict. |
| Confirmation gate | Assign Staff (Action 4) writes immediately after per-assignment in-conversation confirm. Confirm Match (Action 5) checks thresholds + marks Set. |
| Mark Set trigger | Organiser explicitly calls "Confirm the match" (Action 5). No automatic trigger on Assignment creation. |
| Role field location | `Role__c` is a permanent picklist on Staff Member. Assignment inherits it; Organiser cannot cross-assign roles. |
| Match → Team relationship | Two lookup fields on Match: `Home_Team__c` + `Away_Team__c`. |
| Staff availability flag | `Is_Active__c` checkbox on Staff Member. Agent considers only active staff. |
| Set is final | Validation rule blocks Assignment changes on a Set match. No un-confirm in Phase 1. |
| DB-level uniqueness | Duplicate rule on (`Match__c`, `Staff_Member__c`) — one person per match, full stop. |
| No-staff halt | Agent halts entire flow, alerts Organiser with the specific role, stops. No partial-role continuation. |
| Unassign action | Out of scope for Phase 1. Agent declines politely. |
| Post-confirm behaviour | Session signals done after Confirm Match summary. No further commands accepted on that match. |
| Stadium conflict scope | Whole stadium blocked per match window. No sub-venue concept. |
| Crew minimum scope | Per match — each match independently needs 4 ground staff + 2 commentators + 3 camera operators. |
| One-person-per-match rule | Unique person per match (not per role). No Staff Member can appear twice on the same match. |
| Partial assignment on failure | Partial assignments are kept. Match stays in Staffing. Organiser can return to assign remaining crew. |
| Multi-session staffing | Allowed — Organiser can assign crew across multiple sessions until the match is Set. |
| Agent access | Organiser only. Admin uses standard Salesforce UI. Dedicated permission set required for Organiser. |
| Crew thresholds configurable? | Hardcoded constants (Apex / Custom Metadata) for Phase 1. No admin config. |
| Org type | Persistent Developer Edition org (no scratch org, no DevHub). |
| Demo reset | Apex anonymous reset script: deletes all Match Assignments, resets Match statuses to Draft/Staffing. Run before each demo. |
| Seed data size | 5 matches, 3 stadiums, 6 staff per role. Some staff pre-assigned to create realistic conflicts for demo step 2. |
| Demo script vs. seed data | Script is written after seed data is built; staff names in the demo script match the seeded records. |

---

## Open Questions

Items still unresolved — relevant for Phase 2 planning or a real customer engagement:

1. **Phase 2 scope** — What capabilities come after Phase 1? (Ticketing? Staff self-service? Schedule auto-generation?) The "Phase 1" framing implies a roadmap, but Phase 2 is entirely undefined.
2. **License model** — How many Organiser and Admin seats? What Salesforce license tier (full Salesforce license vs. External App / low-cost agent surface)?
3. **Agent channel (future)** — Phase 1 is internal Salesforce app only. Would Phase 2 include Slack, Experience Cloud, or mobile?
4. **Real customer analogue** — If this demo is intended to lead a real customer conversation, who is the target account profile? What vertical or size?
5. **Governance / adoption** — For a real engagement: change management plan, Admin training, and a CoE owner for the agent topic/actions.
6. **Demo environment owner** — Who on the build team owns the Developer Edition org and is responsible for running the reset script before each session?

---

## Knowledge Base Status

⚠️ `knowledge/` is empty — client-specific grounding is missing. General Salesforce architecture is grounded via Trailhead MCP (confirmed reachable). For product implementation depth, download Agentforce and Flow implementation guides from https://help.salesforce.com/s/products and place in `knowledge/`.

---

*Brief produced by Scopezilla discover · 2026-11-02*
