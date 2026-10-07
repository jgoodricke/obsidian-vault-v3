https://linear.app/webres-solutions/issue/CBR-300
## Developer Notes

- Apply one consistent browser-only submission rule to Application, User, Company and Facility creation and updates, and to Enquiry sending.
- Disable all submit controls while validation or submission is pending and prevent rapid repeated clicks, switching between submit buttons or keyboard submission from starting another request.
- Protection must remain active for the full pending request.
- Both Application submit actions must share the same pending state while retaining their existing behaviour and destination.
- After request failure or cancellation, submission must become available again.
- After success, further submissions must remain blocked until the form closes or navigation completes. Reopening the form later must not leave it locked.
- Preserve existing submitted data, uploads, Completion Timing, optional initial Enquiry creation and navigation.
- Enquiry sending must retain selected Facilities and additional information in the submitted data. Successful sending must continue to clear the selection and close the dialog, while validation errors leave the dialog available for correction.
- No patient-data uniqueness rule is introduced.
- No server-side duplicate protection, submission keys or database changes are included.
- Frontend regression tests cover all five flows using real form integration, including pending, validation, failure, cancellation and success behaviour. Add targeted Pest browser regressions only for Applications and Enquiry sending.
- Browser regressions must use the isolated testing database and synthetic supported users. Preserve the imported production-derived database.
- The form's submit handler returning is not evidence that its Inertia request has finished; submission protection must follow actual request completion.