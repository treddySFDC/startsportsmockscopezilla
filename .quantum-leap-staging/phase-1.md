# Phase 1 — Foundation

## Pre-decided

These are confirmed scope decisions the build agent inherits — no re-confirmation needed:

- **5 custom objects**: Team__c, Stadium__c, Match__c, Staff_Member__c, Match_Assignment__c
- **Match_Assignment → Match relationship**: Master-Detail (not Lookup) — cascade-delete semantics
- **Match lifecycle**: Draft → Staffing → Set (one-way, enforced by validation rule)
- **Duplicate rule**: unique constraint on (Match__c, Staff_Member__c) — one person per match
- **No Apex triggers**: all automation is @InvocableMethod-based; triggers are out of scope for this phase
- **No LWC**: standard Lightning page layouts only
- **Two permission sets**: Match_Organiser (agent surface + full object access), Match_Admin (master-data CRUD)
- **Thresholds NOT stored on the data model**: hardcoded in Apex (Phase 2); no threshold fields on Match__c

## Plan-mode questions

Surface these to the user for explicit yes/no before building:

1. **End_DateTime__c required?** — the overlap detection SOQL depends on both Start and End being non-null. Should both be required fields, or should End be optional and defaulted?
2. **Role__c as a global value set or per-object?** — Staff_Member__c.Role__c and Match_Assignment__c.Role__c share the same picklist values (Ground Staff / Commentator / Camera Operator). Should these share a global value set, or be independent per-object picklists?
3. **Status__c default** — Match__c.Status__c defaults to 'Draft'. Confirm this is the correct default for new match records created by an Admin.

## Build-mode questions (for the build agent)

Ask these only if not resolved by the plan-mode answers:

1. If Role__c is per-object: are the exact picklist labels Ground Staff, Commentator, Camera Operator (confirm casing — Apex uses these for threshold checks)?
2. Validation rule formula — should `Status__c = 'Set'` block insert/update/delete on Match_Assignment__c, or only update/delete?

## Out of scope for this phase

- Agentforce Topics or Actions (Phase 2)
- Apex classes (Phase 2)
- Any data seeding (Phase 3)
- Custom page layouts beyond standard

## Build targets (INT reference summary)

| Intent | Surface | What gets built |
|--------|---------|-----------------|
| INT-001 | data-model | 5 objects, relationships, picklists, validation rule, duplicate rule, 2 permission sets |

## Phase-level acceptance

An Admin user can navigate to Object Manager, find all 5 custom objects, create a Match record with Home/Away teams, a Stadium, Start and End datetimes in Status 'Draft'. They assign one Staff Member to the Match — a Match_Assignment__c record is created and the Match status moves to 'Staffing'. They try to assign the same staff member again — the duplicate rule blocks it. They manually set Match status to 'Set' and try to edit the assignment — the validation rule blocks it with a clear error.
