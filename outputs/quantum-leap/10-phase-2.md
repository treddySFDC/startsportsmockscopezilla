# Phase 2 — Scheduling Logic & Agent (StartSports Match Operations)

> 🛑 **DO NOT BUILD AGAINST THIS FILE — STUB.**
> Phase 2 has not been staged with a real build brief. The sections below contain placeholders only.
> Regenerate this phase via the `quantum-leap` skill before any planning or build action targets it.
> If the user asks you to plan or build Phase 2, **stop** and tell them the bundle needs to be regenerated.

## Phase metadata (planning only — not a build target)

- **Planning duration:** sequence only — no committed duration
- **Outcome (from roadmap):** Build the 3 Apex @InvocableMethod classes (StaffAvailabilityChecker, StaffAssigner, MatchConfirmer) and configure the Agentforce agent (Match_Operations topic, 5 custom actions, confirmation gate, out-of-scope guardrails). E03 Apex must exist before E02 agent Actions can reference it.
- **Measured by (from roadmap):** All 3 Apex classes pass at >75% test coverage including touching-times and stadium conflict scenarios; agent correctly routes all 5 action types; confirmation gate fires before every Assign Staff call; agent declines out-of-scope requests politely.

## Epics tentatively in scope for this phase

These are the epics roadmap.json assigned to Phase 2. They will become a real build target when this phase is staged via the `quantum-leap` skill — until then, **do not build against them**.

- **E03: Scheduling Logic & Business Rules** — Build the custom backing logic (Flow or Apex invocables) that enforces all Phase 1 business rules: no double-booking of staff across overlapping matches, minimum crew thresholds (4 ground staff / 2 commentators / 3 camera operators) before a match can be marked Set, one role per person per match, and stadium conflict check. This is the highest-complexity epic — none of these rules are native Agentforce capabilities. Includes the alert-and-stop behaviour when no staff is available for a required role.
- **E02: Agentforce Agent Configuration** — Define and configure the Agentforce agent for the Organiser persona. Covers one Topic (Match Operations), five agent Actions (Get Teams, Get Upcoming Matches, Check Staff Availability, Assign Staff, Confirm Match), agent instructions and persona guardrails, and the confirmation gate (agent never assigns without Organiser confirmation). Surface: internal Salesforce app via Agent Studio.

## Dependencies and risks (from roadmap)

**Dependencies:** Phase 1 complete — Apex queries the custom objects; agent actions call Apex invocables.

**Risks:** Agent instruction quality requires iteration — budget time for prompt tuning with realistic Organiser test prompts before declaring E02 complete. Overlap detection bug silently breaks the demo — comprehensive Apex test coverage is the mitigation.

---

To stage this phase as a real build target, re-run the `quantum-leap` skill and select Phase 2. The skill will produce intent, pre-decided constraints, plan-mode questions, build targets, and dual-shaped acceptance for the agent to use.
