# TODO
- Add Cards:
	- Group 1
		- Applications submitted
		- Active applications
		- Patients placed
		- Total enquiries
		- Engaged providers
	- Group 2
		- Avg time to first response
		- Avg time to outcome
		- Avg time to place
- Add Graphs:
	- Applications, enquiries and placements over time
	- Application Status Mix
	- Applications by region
	- Unsuccessful enquiry insights
	- Application profile
	- Enquiries by status
	- Time metrics by month
- Add Lists
	- Grouped suburb regions
	- Complex needs breakdown
- Add Tables
	- Team comparison table


---


Note: 
- All cards, charts, tables and lists should contain data exporting to csv.

Other:
- Add Page
	- Use existing menu item
- basic Date range filtering  (30 days, 90 days, 1 year)
- advanced date range filtering (filter using two date pickers)
- Add filtering by Sites/Facilities
	- Requires setting up parent companies in order to filter facilities correctly.
- Add Filtering by status?
- Add different dashboard types
	- Restrict access to dashboard types depending on User level
	- These users have distinct dashboards:
		- Super-Admin
		- Hospital/Referrer Organisation
		- Provider
		- Executive (new user type)
			- Depends on having sites/facilities set up.
		- Supervisor (New User Type)

---

- Applications submitted
    - Type: KPI card
    - Data: Total count of applications (distinct patient-level records)
    - Notes: Represents overall volume; baseline metric for most other ratios
- Active applications
    - Type: KPI card
    - Data: Applications excluding “Placed” and “No Longer Required”
    - Notes: Derived metric, requires status filtering logic
- Patients placed
    - Type: KPI card
    - Data: Count of applications with “Placed” outcome
    - Notes: Outcome-focused metric
- Total enquiries
    - Type: KPI card
    - Data: Sum of all enquiry records (provider-level)
    - Notes: Different grain to applications (1 application → many enquiries)
- Engaged providers
    - Type: KPI card
    - Data: Count of providers active in the system
    - Notes: Likely sourced from organisations/providers table

---

- Applications, enquiries and placements over time
    - Type: Composite chart (area + line)
    - Data: Monthly counts of:
        - Applications (distinct)
        - Enquiries (total)
        - Placements
    - Notes: Mixed aggregation levels, needs careful query design to avoid duplication
- Application status mix
    - Type: Pie chart + legend
    - Data: Distribution of applications by status
    - Notes: One record per application; patient-level aggregation

---

- Applications by region
    - Type: Map-style visual + ranking list
    - Data: Application counts grouped by region/suburb clusters
    - Notes:
        - Uses deduplicated application IDs
        - Includes derived grouping logic for suburbs
        - Map is conceptual (lat/lng not yet implemented)
- Region ranking
    - Type: Ranked list with progress bars
    - Data: Same as above, sorted by volume
    - Notes: Adds percentage of total

---

- Unsuccessful enquiry insights
    - Type: Pie chart + legend
    - Data: Breakdown of unsuccessful enquiries by reason
    - Notes:
        - Based on enquiry-level data
        - Includes percentage of total enquiries
        - Total unsuccessful count displayed centrally

---

- Application profile – Support profile
    - Type: Progress bar list
    - Data: Counts and percentages of:
        - MSU required
        - Complex care needs
        - Behavioural needs
        - Other supports
    - Notes: Uses total application count as denominator
- Application profile – Financial status
    - Type: Progress bar list
    - Data: Distribution of:
        - Fully supported
        - Partially supported
        - Self-funded
        - Unsure
- Application profile – Type of care
    - Type: Pie chart + legend
    - Data: Counts of:
        - Permanent
        - Respite
        - Home care

---

- Avg time to first response
    - Type: KPI card
    - Data: Average time (hours converted to days) from enquiry creation to first response
    - Notes: Requires timestamp tracking on enquiries
- Avg time to outcome
    - Type: KPI card
    - Data: Average time from enquiry creation to outcome
- Avg time to placed
    - Type: KPI card
    - Data: Average time from enquiry to placement

---

- Enquiries by status
    - Type: Ranked bar-style list
    - Data: Count of enquiries grouped by workflow status
    - Notes: Provider workflow visibility

---

- Time metrics by month
    - Type: Multi-line chart
    - Data: Monthly averages for:
        - First response time
        - Outcome time
        - Placement time
    - Notes: Currently placeholder data; requires historical aggregation

---

- Grouped suburb regions
    - Type: List
    - Data: Region name, included suburbs, application counts
    - Notes: Defines reporting grouping logic

---

- Complex needs breakdown
    - Type: List
    - Data: Counts of specific needs (e.g. wandering, aggression, bariatric)
    - Notes: More granular than the earlier “support profile”

---

- Provider status matrix
    - Type: Table
    - Data (per provider):
        - Facilities count
        - Active applications
        - Clinical review
        - Waitlisted
        - Placed
        - Unsuccessful
    - Notes: Cross-dimensional dataset (provider × status)

---

- Facility snapshot
    - Type: Card list
    - Data:
        - Engaged providers
        - Reported vacancies
        - Facilities with vacancies
    - Notes: Some marked as placeholders/future state

---

- Team comparison table
    - Type: Table
    - Data (per team/site):
        - Applications submitted
        - Active
        - Avg enquiries per application
        - Avg form completion time
        - Placed
    - Notes: Combines operational and efficiency metrics

---

- User engagement metrics
    - Type: Card grid
    - Data:
        - Logins this month
        - Average session time
        - Map searches
        - Active users
    - Notes: Product usage analytics rather than care workflow




