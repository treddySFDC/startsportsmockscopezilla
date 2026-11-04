# Engagement Intent — StartSports Match Operations

> Reference role: the *why* of the build. The phase briefs are how. When weighing a Plan-mode trade-off, weigh against this.
>
> If you need scoping context (current-state challenges, business impacts, confidence summary), see `93-scoping-context.md` (emitted only when scoping data is present).

## Intent

- **Measured by:** _Foundation:_ All 5 custom objects deployed to the target org; Match lifecycle enforced (Draft → Staffing → Set); validation rule blocks edits on Set matches; duplicate rule blocks double-assignment on (Match, Staff Member); Match_Organiser and Match_Admin permission sets created. · _Scheduling Logic & Agent:_ All 3 Apex classes pass at >75% test coverage including touching-times and stadium conflict scenarios; agent correctly routes all 5 action types; confirmation gate fires before every Assign Staff call; agent declines out-of-scope requests politely. · _Demo Ready:_ Full 4-step demo flow completes without errors; reset script clears all assignments and resets Match statuses to Draft in under 5 seconds; demo owner has rehearsed the flow at least once; post-deploy smoke test checklist signed off.

## Engagement at a glance

StartSports Match Operations

- **Clouds in scope:** Agentforce
- **Phases planned:** 3
- **Target org:** `(target org name TBD)` (sandbox)
- **Build allowed:** yes

