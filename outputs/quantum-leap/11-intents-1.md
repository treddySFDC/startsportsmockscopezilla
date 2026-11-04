# Intent Statements — Phase 1 (StartSports Match Operations)

> Reference role: the **load-bearing build target** for Phase 1. Each intent below is one capability — one firing trigger or user action, one outcome, one walkthrough. Build one at a time. The phase brief (`10-phase-1.md`) is orchestration; this file is what to build.
>
> **For architects:** walk these with the customer to assign priority and answer open questions. Edit `data/intents.json` (canonical) or this file directly — the next quantum-leap run re-renders from JSON.

## INT-001 — Deploy the 5-object data model for match crew scheduling

epic `E01` · priority _(unassigned)_ · confidence _Confirmed_ · surface `data-model`

### 1. Outcome

All 5 custom objects exist in the org with their relationships, picklists, and rules — ready for the scheduling logic and agent to build on top of.

### 2. Build target

- Create Team__c: Name (standard), City__c (Text 100)
- Create Stadium__c: Name__c (Text 100), City__c (Text 100), Capacity__c (Number)
- Create Staff_Member__c: Name (standard), Role__c (Picklist: Ground Staff / Commentator / Camera Operator), Is_Active__c (Checkbox, default true)
- Create Match__c: Home_Team__c (Lookup → Team__c), Away_Team__c (Lookup → Team__c), Stadium__c (Lookup → Stadium__c), Start_DateTime__c (DateTime, required), End_DateTime__c (DateTime, required), Status__c (Picklist: Draft / Staffing / Set, default Draft)
- Create Match_Assignment__c: Match__c (Master-Detail → Match__c), Staff_Member__c (Lookup → Staff_Member__c), Role__c (Picklist: same values as Staff_Member__c.Role__c)
- Validation rule on Match_Assignment__c: block any edit (insert/update/delete) when parent Match__c.Status__c = 'Set'. Rule name: No_Edit_On_Set_Match
- Duplicate rule on Match_Assignment__c: unique constraint on (Match__c, Staff_Member__c) — one person per match
- Standard Lightning page layouts for all 5 objects (no custom LWC required)
- Permission set Match_Organiser: Read/Create/Edit on all 5 objects + Agentforce surface access
- Permission set Match_Admin: full CRUD on Team__c, Stadium__c, Staff_Member__c

### 3. Guardrails

- Must not use a Lookup on Match_Assignment → Match — it must be Master-Detail so cascade-delete works.
- Must not make Start_DateTime__c or End_DateTime__c optional — both are required for overlap detection.
- Must not add a Capacity or staffing-threshold field to Match__c — thresholds are hardcoded in Apex for Phase 1.
- Must not create any Apex triggers on these objects — all automation is invocable-method-based.

### 4. Out of scope

- Must not create any Agentforce Topics or Actions in this intent (handled by INT-003).
- Must not write any Apex classes in this intent (handled by INT-002).
- Must not create page layout customisations beyond standard layouts.

### 5. Acceptance

An Admin user opens Setup, navigates to Object Manager, and sees all 5 custom objects listed. They open Match__c and confirm Start_DateTime__c and End_DateTime__c are both required DateTime fields. They create a Match_Assignment__c record for Match 'Test Match 1' with Staff Member 'Alice'. They attempt to create a second Match_Assignment__c for the same match and same staff member — the duplicate rule blocks it with a clear error. They set the parent Match__c.Status__c to 'Set' and attempt to edit the assignment — the validation rule blocks it.

### Open questions

_(no open questions captured)_

