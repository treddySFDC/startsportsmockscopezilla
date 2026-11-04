# Intent Statements — Phase 3 (StartSports Match Operations)

> Reference role: the **load-bearing build target** for Phase 3. Each intent below is one capability — one firing trigger or user action, one outcome, one walkthrough. Build one at a time. The phase brief (`10-phase-3.md`) is orchestration; this file is what to build.
>
> **For architects:** walk these with the customer to assign priority and answer open questions. Edit `data/intents.json` (canonical) or this file directly — the next quantum-leap run re-renders from JSON.

## INT-006 — Deploy all metadata and seed the demo dataset

epic `E04` · priority _(unassigned)_ · confidence _Confirmed_ · surface `devops`

### 1. Outcome

The full build is deployed to astroNewOrgOne and a realistic demo dataset exists — the Organiser can run the 4-step demo flow immediately, and the demo can be reset and repeated cleanly.

### 2. Build target

- SFDX project with all metadata: custom objects (5), fields, validation rules, duplicate rules, Apex classes (5: StaffAvailabilityChecker, StaffAssigner, MatchConfirmer, GetTeamsAction, GetUpcomingMatchesAction), Apex test classes (4), permission sets (2), Agentforce topic + 5 actions
- Deployment sequence (order matters): (1) custom objects + fields + rules, (2) Apex classes + test classes, (3) permission sets, (4) Agentforce topic + actions
- Anonymous Apex seed data script:
  - 5 Teams: Mumbai Indians, Chennai Super Kings, Delhi Capitals, Royal Challengers Bangalore, Kolkata Knight Riders
  - 3 Stadiums: Wankhede Stadium (Mumbai), MA Chidambaram Stadium (Chennai), Narendra Modi Stadium (Ahmedabad)
  - 18 Staff Members: 6 Ground Staff, 6 Commentators, 6 Camera Operators (all Is_Active__c = true)
  - 5 Matches: Match 1-4 spread across different stadiums and dates; Match 3 is the demo focus match at Wankhede Stadium on a specific future date
  - Pre-seed 3 assignments on Match 3: 2 Camera Operators already assigned, so Action 3 shows realistic availability during the demo
  - At least 1 staff member per role has an overlapping assignment on a different match (to show availability filtering)
- Assign Match_Organiser permission set to the demo user: sf org assign permset -n Match_Organiser --target-org astroNewOrgOne
- Anonymous Apex reset script (run before each demo session):
  delete [SELECT Id FROM Match_Assignment__c];
  List<Match__c> matches = [SELECT Id, Status__c FROM Match__c];
  for (Match__c m : matches) { m.Status__c = 'Draft'; }
  update matches;
  System.debug('Reset complete.');
- Post-deploy smoke test checklist: agent active in Agent Studio, all 5 actions linked, demo flow runs end-to-end once

### 3. Guardrails

- Must deploy objects before Apex (Apex references the objects) and Apex before the agent actions (actions reference the Apex classes).
- Must not seed real personal data — all seed data is fictional.
- Must not run the reset script during deployment — it's for pre-demo use only.

### 4. Out of scope

- Must not set up CI/CD pipeline — single deploy, Phase 1 only.
- Must not migrate any real data.

### 5. Acceptance

sf project deploy start completes with 0 errors and 0 warnings. All 4 Apex test classes pass with >75% coverage. The demo user can open the Salesforce app, find the Match Operations Agent, and complete the 4-step demo flow: (1) list matches, (2) check Camera Operator availability for Match 3 (sees 4 available, 2 already assigned), (3) assign one Camera Operator, (4) attempt to confirm — agent reports how many more crew are still needed. The reset script runs in under 5 seconds and leaves Teams, Stadiums, Staff Members, and Matches intact.

### 6. Dependencies

- **Internal (build first):** INT-001, INT-002, INT-003, INT-004, INT-005

### Open questions

- [ ] **Q-002** — What are the specific dates/times for the 5 demo matches? They need to be in the future at demo time. (Resolver: Demo owner)

