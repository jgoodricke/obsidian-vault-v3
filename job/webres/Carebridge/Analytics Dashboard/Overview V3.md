[ChatGPT - Carebridge Reports Dashboard Mockup](https://chatgpt.com/canvas/shared/69e1ab1c05688191aa463fde427199a5)

TODO: 
- Check if any other cards need to be removed.
- Add estimates to cards.
- Add cards to sprints.




**Title**  
Export current dashboard view as CSV

**Summary**  
Allow users to export the current dashboard data as a CSV file, respecting all active filters.

**Acceptance Criteria**

- Export includes only data visible under current filters (date range, site, facility, etc.)
- Export is accessible from the dashboard UI
- Data structure matches the underlying datasets (not just visual summaries)
- File downloads as a valid CSV

**Notes**

- Should integrate with existing per-component export functionality
- Ensure consistency between displayed data and exported data

---

**Title**  
Generate summary report export for leadership

**Summary**  
Provide a simplified, high-level export suitable for executive reporting.

**Acceptance Criteria**

- Export includes key summary metrics (applications, placements, enquiries, etc.)
- Data is aggregated and easy to interpret
- Export format is CSV (or extendable to PDF later)
- Respects selected filters

**Notes**

- Prioritise clarity over raw detail
- Likely aligned with Executive dashboard view

---

**Title**  
Add scheduled email snapshot of dashboard

**Summary**  
Enable automated delivery of dashboard summaries via email on a scheduled basis.

**Acceptance Criteria**

- Users can configure a schedule (e.g. weekly, monthly)
- Email includes summary metrics and/or attached report
- Respects user permissions and data scope
- Can be enabled/disabled per user

**Notes**

- Future phase item
- Consider using queued jobs for generation and delivery
- Align with summary report export format

---

**Title**  
Add drill-through to raw application and enquiry data

**Summary**  
Allow users to navigate from dashboard metrics into underlying application or enquiry records.

**Acceptance Criteria**

- Users can click from charts/cards into a filtered list view
- Drill-through respects current filters and context
- Supports both application-level and enquiry-level views
- Resulting list is paginated and usable

**Notes**

- Critical for trust and usability of analytics
- Reuse existing list pages where possible to avoid duplication