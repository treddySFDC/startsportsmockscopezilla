# Epics — Context Only — StartSports Match Operations

> Reference role: **background**, not load-bearing. The phase briefs are authoritative. This file dereferences epic IDs cited in phase briefs (e.g. `(E01)`) and provides scoping-stage context for trade-off reasoning.
>
> Do not plan against epics. Plan against `10-phase-N.md`.

## E01: Data Model & Custom Objects
Set up the five custom Salesforce objects that back the entire application: Team, Stadium, Match, Staff Member, and Match Assignment. Includes field definitions, object relationships (Match → Stadium, Match Assignment → Match + Staff Member), status picklist on Match (Draft / Staffing / Set), and role picklist on Staff Member. Admin persona manages data through standard Salesforce UI.

_Confidence: Confirmed_
**KB sources:** [assumption: field-level validation rules (e.g. capacity not negative) not specified — assumed standard; what would validate: Admin UX requirements]

## E02: Agentforce Agent Configuration
Define and configure the Agentforce agent for the Organiser persona. Covers one Topic (Match Operations), five agent Actions (Get Teams, Get Upcoming Matches, Check Staff Availability, Assign Staff, Confirm Match), agent instructions and persona guardrails, and the confirmation gate (agent never assigns without Organiser confirmation). Surface: internal Salesforce app via Agent Studio.

_Confidence: Confirmed_
**KB sources:** [extends: agent-customization-with-flows — agent-ready Flow pattern (https://trailhead.salesforce.com/en/content/learn/modules/agent-customization-with-flows)], [extends: agent-customization-with-apex — custom agent actions via Apex (https://trailhead.salesforce.com/en/content/learn/modules/agent-customization-with-apex)]

## E03: Scheduling Logic & Business Rules
Build the custom backing logic (Flow or Apex invocables) that enforces all Phase 1 business rules: no double-booking of staff across overlapping matches, minimum crew thresholds (4 ground staff / 2 commentators / 3 camera operators) before a match can be marked Set, one role per person per match, and stadium conflict check. This is the highest-complexity epic — none of these rules are native Agentforce capabilities. Includes the alert-and-stop behaviour when no staff is available for a required role.

_Confidence: Confirmed_
**KB sources:** [assumption: overlap detection is time-based (match date/time); no duration or end-time field specified on Match — what would validate: confirm Match object has a duration or end-time field, or overlap is inferred from a fixed window]

## E04: Demo Environment & Seed Data
Provision the scratch org, deploy all metadata (custom objects, agent configuration, backing logic), and seed a realistic demo dataset: teams, stadiums, staff members across all three roles, and a set of upcoming matches. Supports the four-step demo flow documented in the brief. No real data migration — all data is hand-crafted for the demo.

_Confidence: Confirmed_
**KB sources:** [assumption: demo dataset is small (handful of teams, ~10 staff per role, ~5 matches); what would validate: confirm with demo owner]

---

## Estimates and complexity drivers

These T-shirt sizes are scoping-stage planning estimates. They are **not** build instructions — the build agent should plan against the per-phase briefs, not against epic size. Sizes are included here for context only.

| Epic | T-shirt | Complexity drivers | Risks |
| --- | --- | --- | --- |
| E01: Data Model & Custom Objects | M | Five net-new custom objects with relational dependencies (M-D, dual lookups). Validation rule and duplicate rule are the only automation. Admin works entirely in standard UI. | Field-level validation rules not fully specified — assumed standard; confirm Admin UX requirements before field schema is finalised. |
| E02: Agentforce Agent Configuration | L | Agentforce agent configuration is the UX layer for this build. Getting the natural-language → action mapping reliable across 5 actions, with a confirmation gate and scope-limiting instructions, takes careful instruction design and iterative testing. Not a simple plug-and-play configuration. | Agent instruction quality determines demo reliability. Poor guardrails will cause the agent to attempt out-of-scope actions or skip the confirmation gate. Instruction testing with realistic Organiser prompts is required before demo day. |
| E03: Scheduling Logic & Business Rules | L | The overlap detection SOQL is the hardest piece: must correctly handle the touching-times boundary (start B <= end A). Three separate Apex classes each with distinct logic: (1) availability checker + stadium conflict, (2) assigner with re-validation + status update, (3) confirmer with threshold check + Set transition. Non-trivial Apex test data setup required to cover all conflict scenarios. | Overlap detection bug silently breaks the demo. Test coverage must include: (a) touching times, (b) fully overlapping matches, (c) no-conflict case, (d) partial crew assignment across sessions. Governor limits are not a concern at demo scale (5 matches, 18 staff) but note for Phase 2 scaling. |
| E04: Demo Environment & Seed Data | S | Straightforward SFDX deploy + data seeding. Reset script is 5 lines of anonymous Apex. Seed data must be designed deliberately: Match 3 pre-seeded with 2 Camera Operators to create visible conflicts; at least 1 staff per role with a time-overlapping assignment on another match. | Deployment sequence matters — objects must deploy before agent config, agent config before seed data. Post-deploy smoke test required. Demo owner must know the reset procedure and run it before each session. |
