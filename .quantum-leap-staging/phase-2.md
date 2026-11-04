# Phase 2 — Scheduling Logic & Agent

**Build order within this phase: E03 first, then E02.** Agent Actions reference Apex @InvocableMethod classes — the classes must exist before the agent can wire Actions to them.

## Pre-decided

- **3 Apex classes**: StaffAvailabilityChecker (check), StaffAssigner (write), MatchConfirmer (confirm)
- **Compound-per-action pattern**: one class per agent action, not a god-class
- **Hardcoded thresholds**: Ground Staff ≥ 4, Commentators ≥ 2, Camera Operators ≥ 3 — Apex private static final constants
- **Touching-times = conflict**: `Start < requestedEnd AND End > requestedStart` — strict less-than; a match starting exactly when another ends is a conflict
- **StaffAssigner re-validates inline**: even though the agent already called StaffAvailabilityChecker — race conditions are real
- **Status transition**: Draft → Staffing on first assignment (StaffAssigner); Staffing → Set only when all 3 thresholds pass (MatchConfirmer)
- **5 agent Actions**: Get Teams, Get Upcoming Matches, Check Staff Availability, Assign Staff, Confirm Match
- **Confirmation gate on Assign Staff**: agent must state name + role and ask for explicit Organiser confirmation before calling the action
- **Agent declines out-of-scope**: ticketing, scores, payroll, un-assign requests are all politely declined
- **Internal Lightning app only**: no Slack, no Experience Cloud, no mobile

## Plan-mode questions

1. **Stadium conflict check**: Should StaffAvailabilityChecker return a stadium conflict error and stop (current design), or should it continue and return available staff anyway with a stadium conflict flag alongside? The current design stops on conflict — confirm.
2. **GetUpcomingMatchesAction date filter**: Return all non-Set matches regardless of date, or filter to matches in the next N days? The open question in INT-005 (Q-001) — the builder needs a decision before writing the query.
3. **Set matches in availability check**: StaffAvailabilityChecker excludes assignments on Set matches from blocking availability (a confirmed Set match's staff can be assigned elsewhere). Confirm this is the correct behavior, or should Set-match staff remain locked?

## Build-mode questions (for the build agent)

1. Apex test class naming convention — `StaffAvailabilityCheckerTest`, `StaffAssignerTest`, `MatchConfirmerTest`, `(GetTeams|GetUpcomingMatches)ActionTest`?
2. Agent topic API name — `Match_Operations` as set in INT-005, confirm.
3. Action API names — should they match the class names exactly (e.g. `StaffAvailabilityChecker`) or get distinct action-facing names (e.g. `CheckStaffAvailability`)?

## Out of scope for this phase

- Seed data (Phase 3)
- Deployment to persistent org (Phase 3)
- Un-assign action
- Configurable thresholds (Custom Metadata is the Phase 3+ upgrade path)
- Email/notification on confirmation

## Build targets (INT reference summary)

| Intent | Surface | What gets built |
|--------|---------|-----------------|
| INT-002 | automation | StaffAvailabilityChecker (read-only) + test class (5 scenarios) |
| INT-003 | automation | StaffAssigner (write + status transition) + test class (4 scenarios) |
| INT-004 | automation | MatchConfirmer (threshold check + lock) + test class (4 scenarios) |
| INT-005 | agentforce | Agentforce Topic + 5 Actions + GetTeamsAction + GetUpcomingMatchesAction + confirmation gate |

## Phase-level acceptance

All 4 Apex test classes pass at >75% coverage. A developer can call each @InvocableMethod from Anonymous Apex and see the expected outcome (availability list returned, assignment inserted, status updated, confirmation blocked on insufficient crew). The Match Operations Agent appears in Agent Studio with all 5 Actions linked. An Organiser types 'Who is available as a camera operator for Match 3?' — the agent returns a named list. The Organiser types 'Assign Carlos as camera operator for Match 3' — the agent asks for explicit confirmation before calling Assign Staff. The Organiser types 'Book me a hotel' — the agent declines politely and stays on topic.
