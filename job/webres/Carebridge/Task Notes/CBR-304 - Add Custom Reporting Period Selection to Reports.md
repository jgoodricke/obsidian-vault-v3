## Technical Details

### Date Handling and Validation

- Use `Australia/Sydney` as the authoritative timezone for Reporting Period boundaries.
- Custom periods include both complete boundary days, regardless of browser timezone or daylight-saving transitions.
- Validate both dates, including malformed dates, impossible dates, missing boundaries, reversed ranges and dates after the current Sydney date.
- Apply equivalent server-side validation so browser checks cannot be bypassed.
- Invalid direct page links must return no report data, and invalid export requests must produce no CSV.
- Invalid custom dates must never silently fall back to the default Reporting Period.
- Do not impose an arbitrary maximum duration on historical Reporting Periods.
- Date controls must be labelled and keyboard-accessible.

### Default Reporting Period

- When both date fields are blank, preserve the existing rolling 12-month behaviour.
- The default starts at the beginning of the Sydney day 12 months ago and has no explicit end date.
- Preserve the existing current-month limit for monthly counts, including eligible scheduled admissions later in the current month.
- Default URLs must remain rolling when reopened on another day.
- Custom Reporting Period URLs must retain their explicitly selected dates.
- Do not populate the date fields with calculated default dates, as an explicit custom period would produce different results.

### Application and Metric Filtering

- All date-filtered metrics, charts and CSVs must use the same Application population, selected by Application creation date.
- For example, an Application created in February must not appear in March reporting simply because an Enquiry was sent in March.
- Preserve current-status calculations and existing event eligibility rules.
- Reporting Period filters must not reconstruct historical Application statuses.
- Duration metrics may include eligible events occurring after the selected End date. For example, an Application created in March may contribute an April response, outcome or admission.
- Do not introduce a new event start-date cutoff.
- Monthly counts must continue following their existing event rules and exclude events after a custom End date.
- Duration metrics and Time Metrics by Month must continue including eligible later events, grouped by Application creation month.

### Charts and Empty Results

- Charts remain grouped by month, including for single-day Reporting Periods.
- Display the months touched by the selected period, without rounding Application selection boundaries to whole months.
- Missing counts should display as zero.
- Missing averages and durations should display no value rather than a misleading zero.
- Preserve existing metric calculations and definitions.

### Engaged Providers

- Engaged providers continue to represent the current global Facility inventory.
- Its count and CSV export must ignore both Reporting Period and Company filters.
- Clearly label the metric as independent of the Reporting Period.

### Filter State and Navigation

- Maintain separate states for edited filter values and successfully applied filters.
- Date and Company selections must be applied together.
- Date changes must preserve the Company selection, and Company changes must preserve the Reporting Period.
- The Clear button must reset both draft date fields and the draft Company selection, without modifying the applied filters until Apply is selected.
- Clearing the draft dates must restore the default rolling 12-month Reporting Period.
- Update report results, applied-period labels and CSV download selections together following successful application.
- Prevent duplicate submissions while loading.
- On failure, retain the last successful report results, applied filter labels and download selections.
- Use a fixed, safe error message without exposing internal diagnostics.
- Store applied date and Company filters in the URL so refresh, copied links and browser Back restore the correct report state rather than unsaved edits.

### Access and Compatibility

- Continue restricting Company selection to users who already have permission to use it.
- Reporting Period filters must not expand access to Applications or Companies.
- Preserve existing Company hierarchy filtering and Organisation Access behaviour.
- Ensure all report data requests and CSV exports apply the same authorised Company scope.
- Leave the Home dashboard unchanged.