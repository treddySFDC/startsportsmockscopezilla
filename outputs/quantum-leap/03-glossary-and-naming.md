# Glossary & Naming — StartSports Match Operations

> Reference role: **authoritative naming source**. The build agent consults this file every time it's about to invent an object API name, field API name, picklist value, label, or custom permission name.

If a name appears here, **use it verbatim**. If a name doesn't appear here, **ask the user before inventing one**.

## Object naming conventions
Use the following naming patterns. Anything else, **ask the user**.

- Custom objects: `Pascal_Snake_Case__c` (e.g. `Partner_Referral__c`)
- Custom fields: `Pascal_Snake_Case__c` (e.g. `Lead_Score__c`)
- Boolean fields: prefix with `Is_` or `Has_`
- Picklist fields: singular noun (e.g. `Source_Detail__c`, not `Source_Details__c`)

## Field naming conventions
Match the conventions above. When extending a standard object (e.g., Lead, Account, Opportunity), prefix custom fields with the team or feature name (e.g. `Sales_Lead_Score__c`, not just `Score__c`).

## Picklist values
_(No picklist values captured. The build agent must ask the user for picklist values before creating any picklist field. When confirmed, append the values here so subsequent agents can read them.)_

## Custom permissions and permission sets
_(No custom permissions or permission sets named in scope yet. Default convention: one permission set per persona — e.g. `Northwind_Sales_User`, `Northwind_Sales_Manager`. Confirm with user before creating.)_

## Client-specific terms
- **Organiser** — The user persona who interacts with the Agentforce agent to view matches and assign crew
- **Admin** — The user persona who manages master data (teams, stadiums, staff)
- **Match Assignment** — The junction object linking a Staff Member to a Match for a specific role
- **Set** — The final confirmed status of a Match once minimum crew thresholds are met
- **Staffing** — The in-progress status of a Match while crew is being assigned
- Agent Studio
- Agentforce Agent Configuration
- Assign Staff
- Business Rules
- Check Staff Availability
- Confirm Match
- Custom Objects
- Data Model
- Demo Environment
- Get Teams
- Get Upcoming Matches
- Match Operations
- Scheduling Logic
- Seed Data
- Staff Member

---

## What's missing here

This file was generated from the Scopezilla scope. It is **not exhaustive** — it captures the names that appeared explicitly in epics, stories, and discovery. When the build agent needs a name not listed here, it must ask the user to confirm before proceeding.
