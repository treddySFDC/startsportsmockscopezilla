# Phase 3 — Demo Ready (StartSports Match Operations)

> 🛑 **DO NOT BUILD AGAINST THIS FILE — STUB.**
> Phase 3 has not been staged with a real build brief. The sections below contain placeholders only.
> Regenerate this phase via the `quantum-leap` skill before any planning or build action targets it.
> If the user asks you to plan or build Phase 3, **stop** and tell them the bundle needs to be regenerated.

## Phase metadata (planning only — not a build target)

- **Planning duration:** sequence only — no committed duration
- **Outcome (from roadmap):** SFDX deploy to persistent Developer Edition org, seed demo dataset (5 teams, 3 stadiums, 18 staff, 5 matches with pre-seeded assignments), assign permission sets to demo user, run end-to-end smoke test, verify reset script.
- **Measured by (from roadmap):** Full 4-step demo flow completes without errors; reset script clears all assignments and resets Match statuses to Draft in under 5 seconds; demo owner has rehearsed the flow at least once; post-deploy smoke test checklist signed off.

## Epics tentatively in scope for this phase

These are the epics roadmap.json assigned to Phase 3. They will become a real build target when this phase is staged via the `quantum-leap` skill — until then, **do not build against them**.

- **E04: Demo Environment & Seed Data** — Provision the scratch org, deploy all metadata (custom objects, agent configuration, backing logic), and seed a realistic demo dataset: teams, stadiums, staff members across all three roles, and a set of upcoming matches. Supports the four-step demo flow documented in the brief. No real data migration — all data is hand-crafted for the demo.

## Dependencies and risks (from roadmap)

**Dependencies:** Phases 1 and 2 complete — all custom objects, Apex classes, and agent configuration must be deployed before seeding.

**Risks:** Deployment sequence matters — objects must deploy before agent config, agent config before seed data. First-deploy dry run (`sf project deploy start --dry-run`) recommended. Demo owner must know the reset procedure before each session.

---

To stage this phase as a real build target, re-run the `quantum-leap` skill and select Phase 3. The skill will produce intent, pre-decided constraints, plan-mode questions, build targets, and dual-shaped acceptance for the agent to use.
