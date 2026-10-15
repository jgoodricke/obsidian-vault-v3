https://linear.app/webres-solutions/issue/CBR-303

## Technical Details

### Permissions and Access

- The action is available to Provider Users, System Administrators, and Super Admins with existing detail access.
- Hospital Users without administrative privileges are excluded.
- Existing record-level and field-level permissions must be respected.
- Printing must not expose competing Providers' Enquiry details or any otherwise restricted information.
- Enquiry status does not introduce additional printing restrictions while detail access remains permitted.

### Application Content

The printout must include all authorised fields and labels from the following sections:

- **Patient Basic Details**
- **Patient Representative Details**, including decision-making capacity, representative information, and Notes where authorised.
- **Patient Aged Care Requirements**, including financial information and Preferred Locations.
- **Care Needs**
- **Toileting**
- **ADLs**
- **Supporting Information**, including answers.

Unanswered fields should display blank values while retaining their labels. Explicit responses such as No, Unsure, and Not Applicable must remain distinguishable.

The printout must exclude:

- Enquiry Discussions and Discussion Comments.
- Enquiry status history.
- Hospital Transfer history.
- Application Files and download links.
- Editing and workflow controls.

### Printing Behaviour

- Use the authorised Application data already loaded on the detail page.
- Do not make additional data-refresh or access-check requests when printing.
- If the Application or the user's permissions change after the page loads, the previously loaded information remains printable. Any subsequent protected requests must continue to enforce current permissions.
- Open a print-friendly page or window and trigger the browser's print dialog.
- Allow the browser to handle paper size, pagination, printing, and Save as PDF.
- Do not introduce server-generated PDFs, stored printouts, or custom pagination.
- Display the print timestamp with its time zone. This represents when the copy was produced, not when the Application was last updated.

### Logging

- No new print-request audit logging is required.
- Existing required logging behaviour must remain unchanged.
- Additional print tracking will be considered as part of the broader logging project.