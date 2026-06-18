# Update Salesforce Lead Assignment Rules — Territory Realignment

## User Story

**As a** Customer Success Manager,
**I want** to update the Salesforce Lead Assignment Rules to reflect the new territory map,
**So that** inbound leads are automatically routed to the correct Sales Representative based on their state or ZIP code.

---

## Background

I need to update the `uLab Lead Assignments` rule in Salesforce Assignment Rules to align with the new territory map. The changes include rep reassignments, onboarding a new rep, a territory split, data cleanup, and formatting normalization across all rule entries.

**File to modify:** `force-app/main/default/assignmentRules/Lead.assignmentRules-meta.xml`

---

## Acceptance Criteria

### AC1 — Reassign Nevada to Kayla Hughes
**Given** a Lead with `State = NV` or `State = Nevada`
**When** the assignment rule runs
**Then** the Lead is assigned to `kayla.hughes@ulabsystems.com`

> I need to remove `NV, Nevada` from `barbara@ulabsystems.com` and add a new rule entry for `kayla.hughes@ulabsystems.com` with `Lead.State equals NV, Nevada`.

---

### AC2 — Add Wyoming to Max Ward
**Given** a Lead with `State = WY` or `State = Wyoming`
**When** the assignment rule runs
**Then** the Lead is assigned to `max.ward@ulabsystems.com`

> I need to add `WY, Wyoming` to Max Ward's State criteria. His rule should cover: AZ, CO, UT, WY.

---

### AC3 — Add Michigan to James Van Hoose
**Given** a Lead with `State = MI` or `State = Michigan`
**When** the assignment rule runs
**Then** the Lead is assigned to `james.vanhoose@ulabsystems.com`

> I need to add `MI, Michigan` to James Van Hoose's State criteria. His rule should cover: IN, KY, MI, OH, WV (plus PA Western postal codes).

---

### AC4 — Add Georgia and South Carolina to Paul Morrison
**Given** a Lead with `State = GA`, `Georgia`, `SC`, or `South Carolina`
**When** the assignment rule runs
**Then** the Lead is assigned to `paul@ulabsystems.com`

> I need to add `GA, Georgia, SC, South Carolina` to Paul Morrison's State criteria. His rule should cover: GA, NC, SC, VA.

---

### AC5 — Split New York: City → Sean Bickford / Upstate → Tommy MacDonald
**Given** a Lead from New York State
**When** the assignment rule runs
**Then:**
- If the ZIP code starts with `100–104` or `110–119` (NYC metro) → assign to `sean.bickford@ulabsystems.com`
- If the ZIP code starts with `120–149` (Upstate NY) → assign to `tommy.macdonald@ulabsystems.com`

> For Sean: I need to replace `Lead.State equals NJ, NY, New York, New Jersey` with two criteria: `Lead.State equals NJ, New Jersey` OR `Lead.PostalCode startsWith 100,101,102,103,104,110,111,112,113,114,115,116,117,118,119`.
>
> For Tommy: I need to add `Lead.PostalCode startsWith 120–149` to his rule alongside the existing New England states.

---

### AC6 — Fix Nebraska abbreviation to NE
**Given** a Lead with `State = NE` or `State = Nebraska`
**When** the assignment rule runs
**Then** the Lead is assigned to `jeff.nolte@ulabsystems.com`

> I need to correct the incorrect abbreviation `NB` (New Brunswick, Canada) to `NE` (Nebraska) in Jeff Nolte's State criteria.

---

### AC7 — Remove Utah ZIP codes from Jeff Nolte's rule
**Given** a Lead with a Utah ZIP code (prefix `84xxx`)
**When** the assignment rule runs
**Then** the Lead is assigned to `max.ward@ulabsystems.com` via State criteria, not Jeff Nolte

> I need to remove `840, 841, 843, 844, 846` from Jeff's `PostalCode startsWith` and 21 specific Utah ZIP codes (`84501–84754`) from his `PostalCode equals` criteria. Utah is fully covered by Max Ward's `State = UT, Utah` rule.

---

### AC8 — Remove West Virginia ZIP codes from Paul Morrison's rule
**Given** a Lead with a West Virginia ZIP code or State
**When** the assignment rule runs
**Then** the Lead is assigned to `james.vanhoose@ulabsystems.com`, not Paul Morrison

> I need to remove WV ZIP prefixes `250–266` from Paul's `PostalCode startsWith` and all `26xxx` ZIP codes from his `PostalCode equals`. WV is fully covered by James Van Hoose's `State = WV, West Virginia` rule.

---

### AC9 — Remove redundant Virginia ZIP codes from Paul Morrison's rule
**Given** a Lead with `State = VA` or `State = Virginia`
**When** the assignment rule runs
**Then** the Lead is caught by Paul's State criteria alone — no postal code fallback needed

> I need to remove `PostalCode startsWith 224–249` and `PostalCode equals 20106–22664` (all Virginia ZIPs) since they are redundant — `State = VA, Virginia` already covers them. This simplifies Paul's rule to a single State criteria with no `booleanFilter`.

---

### AC10 — Normalize State criteria format across all rules
**Given** any Lead from a US state
**When** the assignment rule runs
**Then** the Lead matches correctly regardless of whether the State field contains the 2-letter code or the full name

> I need to reformat all `Lead.State` values to consecutive `CODE,Full Name` pairs (e.g., `AK,Alaska,ID,Idaho`). Rules that split codes and names across two separate criteria items should be merged into a single criteria item, removing the redundant `booleanFilter` where applicable.

---

### AC11 — Consolidate Canada province criteria into single criteria items
**Given** a Lead from a Canadian province
**When** the assignment rule runs
**Then** the Lead matches correctly on a single State criteria item

> For Darren Rowland (Canada West): I need to merge two State criteria into one: `AB,Alberta,BC,British Columbia,MB,Manitoba,SK,Saskatchewan`, and update `booleanFilter` from `1 AND (2 OR 3)` to `1 AND 2`.
>
> For Tyler (Canada East): I need to merge two State criteria into one: `NB,New Brunswick,NL,Newfoundland,NS,Nova Scotia,ON,Ontario,PE,Prince Edward Island,QC,Quebec`, and update `booleanFilter` from `1 AND (2 OR 3)` to `1 AND 2`.

---

## Rules I Will NOT Modify

| Rule | Assignee | Reason |
|---|---|---|
| Australia / New Zealand | jacqueline.doon@ulabsystems.com | No territory change |
| CA Northern (postal codes) | barbara@ulabsystems.com | Territory unchanged |
| CA Southern (postal codes) | glenda@ulabsystems.com | Territory unchanged |
| TX Northern (postal codes) | jeff.nolte@ulabsystems.com | Territory unchanged |
| TX Southern (postal codes) | michelle@ulabsystems.com | Territory unchanged |
| PA Western (postal codes) | james.vanhoose@ulabsystems.com | Territory unchanged |
| PA Eastern (postal codes) | sean.bickford@ulabsystems.com | Territory unchanged |
| Default queue | Client_Services | Catch-all unchanged |

---

## Testing

- Deploy metadata to scratch org or sandbox
- Create test Leads for each AC (one per state/ZIP affected)
- Verify assignment via Lead record → Assignment Rule log
- Confirm no leads fall to the Client_Services queue unexpectedly
