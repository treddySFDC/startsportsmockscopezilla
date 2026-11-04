# Intent Statements — Phase 2 (StartSports Match Operations)

> Reference role: the **load-bearing build target** for Phase 2. Each intent below is one capability — one firing trigger or user action, one outcome, one walkthrough. Build one at a time. The phase brief (`10-phase-2.md`) is orchestration; this file is what to build.
>
> **For architects:** walk these with the customer to assign priority and answer open questions. Edit `data/intents.json` (canonical) or this file directly — the next quantum-leap run re-renders from JSON.

## INT-002 — Detect staff availability and stadium conflicts for a match window

epic `E03` · priority _(unassigned)_ · confidence _Confirmed_ · surface `automation`

### 1. Outcome

Given a match ID and a requested role, the system returns the staff members who are available (active, correct role, no time-overlapping assignment) and flags if the stadium is already booked in that window.

### 2. Build target

- Apex class StaffAvailabilityChecker with @InvocableMethod annotation
- Input: matchId (Id), requestedRole (String)
- Logic step 1 — stadium conflict check: query Match__c WHERE Stadium__c = [match's stadium] AND Id != matchId AND Start_DateTime__c < [match's End_DateTime__c] AND End_DateTime__c > [match's Start_DateTime__c] AND Status__c != 'Set'. If any rows returned, return a stadium-conflict error message; do not proceed to staff check.
- Logic step 2 — staff availability: query all Staff_Member__c WHERE Role__c = requestedRole AND Is_Active__c = true. Then query Match_Assignment__c WHERE Match__r.Start_DateTime__c < [match's End_DateTime__c] AND Match__r.End_DateTime__c > [match's Start_DateTime__c] AND Match__r.Id != matchId AND Match__r.Status__c != 'Set' to get conflicted staff IDs. Return the difference (available = all active staff of role minus conflicted IDs).
- Touching-times boundary: use strict less-than (<) not less-than-or-equal — two matches where B starts exactly when A ends are a conflict.
- Output: List<StaffAvailabilityChecker.Result> where Result has: availableStaff (List<String> of names), stadiumConflict (Boolean), errorMessage (String)
- Apex test class StaffAvailabilityCheckerTest with scenarios: (a) no conflict — returns available staff, (b) touching times — B starts exactly when A ends — returns conflict, (c) full overlap — returns conflict, (d) stadium booked by another match in same window — returns stadium-conflict error, (e) no active staff of requested role — returns empty list

### 3. Guardrails

- Must not modify any records — this method is read-only.
- Must not exclude Set matches from the stadium conflict check — a Set match still occupies the stadium.
- Must not use Status__c != 'Set' in the staff conflict query for a different reason — the intent is to exclude assignments on already-confirmed matches from blocking availability.

### 4. Out of scope

- Must not insert or update any records (handled by INT-003).
- Must not check crew minimums or threshold counts (handled by INT-004).

### 5. Acceptance

A developer runs StaffAvailabilityCheckerTest with all 5 test scenarios. All pass at >75% coverage. In the org: for Match 3 (pre-seeded with 2 Camera Operators already assigned), calling StaffAvailabilityChecker with matchId=Match3 and requestedRole='Camera Operator' returns only the Camera Operators who have no other overlapping assignment — the 2 already assigned on Match 3 are not in the available list (they're already assigned, not conflicted). A test call with a role that has a stadium conflict returns stadiumConflict=true and an appropriate errorMessage.

### 6. Dependencies

- **Internal (build first):** INT-001

### Open questions

_(no open questions captured)_

---

## INT-003 — Assign a staff member to a match and advance match status

epic `E03` · priority _(unassigned)_ · confidence _Confirmed_ · surface `automation`

### 1. Outcome

A staff member is assigned to a match for a specific role. The match status moves from Draft to Staffing on the first assignment. The assignment is blocked if the staff member is unavailable.

### 2. Build target

- Apex class StaffAssigner with @InvocableMethod annotation
- Input: matchId (Id), staffMemberId (Id), role (String)
- Logic step 1 — re-validate availability inline: call StaffAvailabilityChecker for matchId + role. If the requested staff member is not in the returned available list (or a stadium conflict exists), return an error message and abort without writing.
- Logic step 2 — insert Match_Assignment__c: Match__c = matchId, Staff_Member__c = staffMemberId, Role__c = role
- Logic step 3 — status transition: if Match__c.Status__c = 'Draft', update it to 'Staffing' (first assignment triggers this; subsequent assignments leave status as-is)
- Output: StaffAssigner.Result with: success (Boolean), message (String — confirmation or error)
- Apex test class StaffAssignerTest with scenarios: (a) successful assignment on a Draft match — inserts record, status moves to Staffing, (b) successful assignment on a Staffing match — inserts record, status unchanged, (c) conflict detected inline — no record inserted, error returned, (d) duplicate assignment (same match + staff) — duplicate rule fires, error returned gracefully

### 3. Guardrails

- Must re-validate availability inline even though the agent already called StaffAvailabilityChecker — race conditions between the check and the write are real.
- Must not set Status__c to 'Set' — that is MatchConfirmer's responsibility (INT-004).
- Must not bypass the duplicate rule — let the database enforce it and catch the exception gracefully.

### 4. Out of scope

- Must not check crew minimums or confirm the match (handled by INT-004).
- Must not remove or update existing assignments.

### 5. Acceptance

Developer runs StaffAssignerTest — all 4 scenarios pass. In the org: assign Staff Member 'Bob (Ground Staff)' to Match 3 (currently Draft). Match 3 status moves to Staffing and a Match_Assignment__c record exists for Bob on Match 3. Assign Bob to Match 3 again — duplicate rule error returned, no duplicate record created. Call with a staff member who has a conflicting assignment on another match in the same window — conflict detected, no record inserted.

### 6. Dependencies

- **Internal (build first):** INT-001, INT-002

### Open questions

_(no open questions captured)_

---

## INT-004 — Confirm a match when minimum crew thresholds are met

epic `E03` · priority _(unassigned)_ · confidence _Confirmed_ · surface `automation`

### 1. Outcome

When an Organiser requests match confirmation, the system checks crew minimums per role. If all thresholds are met, the match is locked as Set and a crew summary is returned. If not, the missing crew count per role is returned.

### 2. Build target

- Apex class MatchConfirmer with @InvocableMethod annotation
- Input: matchId (Id)
- Hardcoded thresholds as private static final Integer constants: GROUND_STAFF_MIN = 4, COMMENTATOR_MIN = 2, CAMERA_OPERATOR_MIN = 3
- Logic: query Match_Assignment__c GROUP BY Role__c for the match, count per role
- Check each role against its threshold. Collect failures as a list of 'Need X more [Role]' strings.
- If all pass: update Match__c.Status__c = 'Set', return crew summary (count + names per role)
- If any fail: do NOT update status, return the list of threshold failures
- Output: MatchConfirmer.Result with: confirmed (Boolean), message (String), crewSummary (String — formatted list when confirmed)
- Apex test class MatchConfirmerTest with scenarios: (a) all thresholds met — status flips to Set, crew summary returned, (b) Ground Staff below threshold — correct error message, status unchanged, (c) multiple roles below threshold — all failures listed, (d) call on already-Set match — returns already confirmed message, no re-update

### 3. Guardrails

- Must not use Custom Metadata for thresholds in Phase 1 — hardcoded constants only.
- Must not flip status to Set unless ALL three role thresholds pass in the same invocation.
- Must not allow status to move back from Set — the validation rule on Match_Assignment__c handles this, but this class must not attempt to set Status__c to anything other than 'Set'.

### 4. Out of scope

- Must not remove assignments or modify crew.
- Must not send notifications or emails — Phase 1 is agent-conversation only.

### 5. Acceptance

Developer runs MatchConfirmerTest — all 4 scenarios pass. In the org: assign exactly 4 Ground Staff, 2 Commentators, and 3 Camera Operators to Match 3. Call MatchConfirmer for Match 3. Match 3 Status__c updates to 'Set'. The returned crew summary lists all 9 assigned staff by role. Attempt to edit a Match_Assignment__c on Match 3 — validation rule blocks it. Call MatchConfirmer with only 3 Ground Staff assigned — status stays Staffing, error message says 'Need 1 more Ground Staff'.

### 6. Dependencies

- **Internal (build first):** INT-001, INT-002, INT-003

### Open questions

_(no open questions captured)_

---

## INT-005 — Configure the Agentforce agent for match crew scheduling

epic `E02` · priority _(unassigned)_ · confidence _Confirmed_ · surface `agentforce`

### 1. Outcome

An Organiser can ask the agent in plain English to list matches, check who's available, assign crew, and confirm a match. The agent always asks for confirmation before assigning, and politely declines out-of-scope requests.

### 2. Build target

- Create Agentforce Topic: Match_Operations
- Topic description: 'You help the Organiser assign crew to upcoming matches. Always ask for confirmation before assigning. Never remove assignments unless asked. If asked about anything outside match crew scheduling, politely decline.'
- Action 1 — Get Teams: backed by Apex @InvocableMethod GetTeamsAction. Returns list of team names.
- Action 2 — Get Upcoming Matches: backed by Apex @InvocableMethod GetUpcomingMatchesAction. Returns match name, teams, date/time, stadium, status for matches with Status__c IN ('Draft','Staffing').
- Action 3 — Check Staff Availability: backed by StaffAvailabilityChecker (INT-002). Inputs: matchId, requestedRole.
- Action 4 — Assign Staff: backed by StaffAssigner (INT-003). Inputs: matchId, staffMemberId, role. Agent instruction: 'Before calling Assign Staff, state the name and role of the staff member you are about to assign and ask the Organiser: Shall I assign [Name] as [Role]? Only call Assign Staff after the Organiser confirms.'
- Action 5 — Confirm Match: backed by MatchConfirmer (INT-004). Input: matchId.
- Create Apex classes GetTeamsAction and GetUpcomingMatchesAction (simple SOQL — no overlap logic needed)
- Assign Match_Organiser permission set to the demo user in the org

### 3. Guardrails

- Must not call Assign Staff without an explicit Organiser confirmation in the same conversation turn.
- Must not attempt to remove or edit existing assignments — if asked, decline politely.
- Must not respond to questions outside match crew scheduling (ticketing, scores, payroll, schedule generation).
- GetUpcomingMatchesAction must not return Set matches — the Organiser has no actions to take on a confirmed match.

### 4. Out of scope

- Must not build a Slack or Experience Cloud channel — internal Lightning app only.
- Must not build an un-assign action in Phase 1.
- Must not make crew thresholds configurable via the agent — Phase 1 only.

### 5. Acceptance

The demo user opens the internal Salesforce app and the Match Operations Agent is available. They type: 'Show me upcoming matches.' The agent returns the list of Draft/Staffing matches. They type: 'Who is available as a camera operator for Match 3?' The agent calls Check Staff Availability and returns a list of available Camera Operators by name. They type: 'Assign Carlos as camera operator for Match 3.' The agent responds: 'Shall I assign Carlos as Camera Operator for Match 3?' The Organiser says yes. The agent calls Assign Staff and confirms the assignment. The Organiser types: 'Book me a hotel for the stadium.' The agent declines politely and stays on topic.

### 6. Dependencies

- **Internal (build first):** INT-001, INT-002, INT-003, INT-004

### Open questions

- [ ] **Q-001** — Should GetUpcomingMatchesAction filter by a date range (e.g. next 30 days), or return all non-Set matches regardless of date? (Resolver: Demo owner / Organiser persona)

