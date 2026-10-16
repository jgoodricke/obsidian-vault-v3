## Technical Details

### Date Filtering and Validation

- Add labelled, keyboard-accessible Start date and End date controls to the existing Reports filter form.
- Both dates blank must preserve the existing rolling 12-month default, including:
    - No explicit end-date constraint on the selected Application population.
    - The existing current-month limit for monthly event counts.
    - Empty custom date controls when using the default period.
- A custom Reporting Period requires both dates. A single-day period is valid, and no arbitrary maximum duration should be imposed.
- Validate invalid or malformed dates, missing boundaries, reversed ranges and dates after the current date in `Australia/Sydney`.
- Enforce validation on the server, including when browser-side validation is bypassed.
- Invalid page and CSV requests must not return report data or silently fall back to the broader default.
- Treat both dates as inclusive calendar days in `Australia/Sydney`, correctly handling browser time zones and daylight-saving transitions.

### Filter Behaviour

- Date and Company selections must be applied together through the existing **Apply** action.
- Unapplied changes must not affect the displayed reports or CSV export requests.
- **Clear** resets both date fields and Company selections but does not update results until **Apply** is used.
- Retain the application's existing request, navigation and error-handling behaviour.

### Report Calculations

- Date filtering determines the selected Application population using **Application creation date**.
- All date-filtered metrics, charts and CSV exports must use that same Application population.
- Preserve existing current-status calculations and metric definitions. Do not introduce historical status snapshots.
- Keep chart groupings monthly without expanding custom date boundaries to whole months.
- Preserve existing event-date rules:
    - Monthly counts retain their event end-date limit.
    - Duration metrics continue to include eligible events occurring after the selected period.
    - Existing event start-date rules remain unchanged.
- **Engaged Providers** must remain independent of both Reporting Period and Company filters, including its CSV export.
- Valid periods with no matching Applications must return zero counts and empty duration values where appropriate.

### Access and Compatibility

- Preserve existing Company filtering rules and Organisation Access restrictions.
- Apply existing permissions consistently to the reports and CSV exports.
- Reuse existing request and error handling without introducing custom loading states, duplicate-submission controls, retry flows or guarantees that previous results remain visible after failed requests.
- Retain standard browser navigation behaviour without adding custom Back handling, bookmark/share-link features or restoration of unapplied selections.