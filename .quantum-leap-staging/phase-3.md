# Phase 3 — Demo Ready

**Prerequisite**: Phases 1 and 2 complete and validated. Deploy everything before seeding.

## Pre-decided

- **Target org**: astroNewOrgOne (persistent Developer Edition, greenfield)
- **Deployment sequence**: objects → Apex → permission sets → agent topic/actions (this order is mandatory — Apex references objects; agent actions reference Apex)
- **Seed data**: 5 teams, 3 stadiums, 18 staff members (6 per role), 5 matches
- **Demo focus match**: Match 3 at Wankhede Stadium — 2 Camera Operators pre-assigned so the availability check shows realistic filtering during the demo
- **At least 1 staff member per role** has an overlapping assignment on a different match (to demonstrate availability filtering works)
- **Reset script**: wipes all Match_Assignment__c records and resets all Match__c statuses to 'Draft'. Runs before each demo session. Does not touch Teams, Stadiums, Staff Members, or Matches.
- **Demo user**: assigned Match_Organiser permission set

## Plan-mode questions

1. **Demo match dates** (Q-002 from INT-006): The 5 seed matches need specific future datetimes. What dates should be used? Recommendation: spread them across the 2–4 weeks following the build date so they're clearly in the future during the demo.
2. **Demo user alias**: What is the username/alias of the user who will run the demo in astroNewOrgOne? (Needed for `sf org assign permset`.)
3. **Deployment method**: SFDX project with `sf project deploy start`, or metadata API retrieve/deploy? Recommendation: SFDX project.

## Build-mode questions (for the build agent)

1. Should the seed data script be a standalone Anonymous Apex file (`seed-data.cls`) in the project root, or stored under `scripts/apex/`?
2. Should the reset script be stored alongside the seed script, or as a separate named file (e.g. `reset-demo.cls`)?
3. Post-deploy smoke test — manual checklist in the README, or an Apex test that validates seed data counts?

## Out of scope for this phase

- CI/CD pipeline
- Real data migration
- Additional orgs or sandbox promotion
- Any agent channels beyond internal Lightning app

## Build targets (INT reference summary)

| Intent | Surface | What gets built |
|--------|---------|-----------------|
| INT-006 | devops | SFDX deploy, seed data script, permission set assignment, reset script, post-deploy smoke test checklist |

## Deployment sequence (mandatory order)

```
1. sf project deploy start --target-org astroNewOrgOne
   (deploys: custom objects, fields, rules → Apex classes + test classes → permission sets → agent topic + actions)

2. Run seed data Anonymous Apex script
   (creates: 5 teams, 3 stadiums, 18 staff, 5 matches, 3 pre-seeded assignments on Match 3)

3. sf org assign permset -n Match_Organiser --target-org astroNewOrgOne -o <demo-user>

4. Manual: open Agent Studio → confirm agent active, all 5 actions present and linked

5. Run end-to-end demo flow once as smoke test
```

## Reset script (run before every demo session)

```apex
delete [SELECT Id FROM Match_Assignment__c];
List<Match__c> matches = [SELECT Id, Status__c FROM Match__c];
for (Match__c m : matches) { m.Status__c = 'Draft'; }
update matches;
System.debug('Reset complete. Assignments deleted, Match statuses reset to Draft.');
```

## Phase-level acceptance

`sf project deploy start` completes with 0 errors and 0 warnings. All 4 Apex test classes pass at >75% coverage. The demo user opens the Salesforce app and the Match Operations Agent is available. The full 4-step demo flow completes without errors: (1) list matches, (2) check Camera Operator availability for Match 3 and see 4 available with 2 already assigned, (3) assign one Camera Operator with explicit confirmation, (4) request match confirmation and receive a 'need more crew' response listing the shortfall. The reset script runs in under 5 seconds and all Match statuses return to 'Draft' with all assignments cleared.
