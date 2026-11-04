# Phase 1 — Foundation (StartSports Match Operations)

> 🛑 **DO NOT BUILD AGAINST THIS FILE — STUB.**
> Phase 1 has not been staged with a real build brief. The sections below contain placeholders only.
> Regenerate this phase via the `quantum-leap` skill before any planning or build action targets it.
> If the user asks you to plan or build Phase 1, **stop** and tell them the bundle needs to be regenerated.

## Phase metadata (planning only — not a build target)

- **Planning duration:** sequence only — no committed duration
- **Outcome (from roadmap):** Deploy all 5 custom objects, field definitions, object relationships, status and role picklists, validation rule (Set is final), duplicate rule (one person per match), and both permission sets. This phase establishes the complete data model every subsequent phase depends on.
- **Measured by (from roadmap):** All 5 custom objects deployed to the target org; Match lifecycle enforced (Draft → Staffing → Set); validation rule blocks edits on Set matches; duplicate rule blocks double-assignment on (Match, Staff Member); Match_Organiser and Match_Admin permission sets created.

## Epics tentatively in scope for this phase

These are the epics roadmap.json assigned to Phase 1. They will become a real build target when this phase is staged via the `quantum-leap` skill — until then, **do not build against them**.

- **E01: Data Model & Custom Objects** — Set up the five custom Salesforce objects that back the entire application: Team, Stadium, Match, Staff Member, and Match Assignment. Includes field definitions, object relationships (Match → Stadium, Match Assignment → Match + Staff Member), status picklist on Match (Draft / Staffing / Set), and role picklist on Staff Member. Admin persona manages data through standard Salesforce UI.

## Dependencies and risks (from roadmap)

**Dependencies:** None — this is the build starting point.

**Risks:** Field schema changes in this phase cascade to E03 (SOQL) and E02 (agent action inputs). Lock the schema before moving to Phase 2.

---

To stage this phase as a real build target, re-run the `quantum-leap` skill and select Phase 1. The skill will produce intent, pre-decided constraints, plan-mode questions, build targets, and dual-shaped acceptance for the agent to use.
