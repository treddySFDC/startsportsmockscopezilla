# StartSports Match Operations Agent — Phase 1 Brief

## 1. Background

StartSports organizes a multi-team tournament. The organizing team tracks teams, fixtures, stadiums and match-day crew (ground staff, commentators, camera operators) in spreadsheets. Assigning crew is manual, and double-booking is common.

## 2. Goal

An Agentforce agent that lets an organizer view upcoming matches, check crew availability, assign crew, and confirm the match as set, all in natural language.

## 3. Scope (Phase 1, demo)

### Data model

| Entity | Fields |
|---|---|
| Team | name, city |
| Stadium | name, city, capacity |
| Match | Team A, Team B, date/time, stadium, status (Draft → Staffing → Set) |
| Staff Member | name, role (Ground Staff / Commentator / Camera Operator), status |
| Match Assignment | match, staff member, role |

### Agent capabilities

1. List teams that are participating.
2. List upcoming matches with date, time and stadium.
3. Check availability of staff by role for a given match date and time.
4. Assign staff to a match for a role.
5. Confirm the match. The agent returns a summary such as:
   > Match set: Team A vs Team B, 12 Nov, 6 PM, Stadium X. Crew: 6 ground staff, 2 commentators, 3 camera operators.

## 4. Business rules

- A staff member can't be assigned to two matches that overlap in time.
- Each match needs a minimum crew before it can be marked Set: **4 ground staff, 2 commentators, 3 camera operators**. These thresholds are fixed and apply to all stadiums.
- Each staff member may hold only one role per match. A commentator cannot simultaneously be logged as ground staff for the same fixture.
- All assignments are active crew. There is no backup or reserve concept in Phase 1.
- If no staff member is available for a required role, the agent alerts the organizer and stops. It does not auto-reassign from other matches.
- The agent only recommends and assigns after the organizer confirms. It never removes assignments without being asked.
- A stadium can't host two matches at the same time.

## 5. Out of scope

- Ticketing, payments and fan-facing features
- Staff payroll and travel
- Live scores
- Automatic schedule generation

## 6. Users

- **Organizer** — uses the agent; can view and assign crew.
- **Admin** — manages teams, stadiums and staff data.

## 7. Design decisions (closed)

These were identified as open questions and have been resolved for Phase 1:

| # | Question | Decision |
|---|---|---|
| 1 | Are crew minimums fixed or stadium-variable? | Fixed: 4 ground staff, 2 commentators, 3 camera operators for all stadiums. |
| 2 | Can one person hold two roles in the same match? | No. One role per person per match. |
| 3 | What happens when no one is available for a role? | Agent alerts the organizer and stops. No auto-reassignment. |
| 4 | Do minimums include backup/reserve staff? | No backup concept in Phase 1. All assignments are active crew. |

## 8. Demo flow

| Step | Organizer says | Agent does |
|---|---|---|
| 1 | "Show upcoming matches." | Lists all upcoming matches with date, time, stadium. |
| 2 | "Who's available for Match 3 as a camera operator?" | Checks availability; lists eligible staff. |
| 3 | "Assign these three." | Asks for confirmation, then assigns. |
| 4 | "Confirm the match." | Shows final summary: A vs B, date/time, stadium, crew counts. |

## 9. Agentforce build (rough)

- **Topic:** Match Operations
- **Actions:** Get Teams · Get Upcoming Matches · Check Staff Availability · Assign Staff · Confirm Match
- **Backing logic:** Flows or Apex invocables, one custom object per entity
