## Existing Branch
cbr-316-add-enquiries-csv-download-with-status

## Technical Details

### Access and availability

- Reuse the existing Exports tab permissions. Only authorised Carebridge Super Admins can access the download.
- Enforce the same permissions on the direct download endpoint.
- Reports access alone must not grant access.
- The download must not be available in Bedsearch.

### CSV structure

Use the existing **Enquiries by Status CSV** as the basis for the export.

Columns must appear in this order:

1. `enquiry_id`
2. `current_status`
3. `application_id`
4. `enquiry_created_at`
5. `facility_name`
6. `no_longer_required_comments`
7. `unsuccessful_comments`

The first five columns must retain the same values and formatting as the existing Enquiries by Status CSV.

Do not include comment author fields.

### Comment selection

- Only populate the comment column matching the Enquiry's **current status**. Both comment columns must be blank for all other statuses.
- Use the **first comment recorded against the current status entry**, including:
    - A comment entered when the status changed.
    - An automatically generated "placed elsewhere" comment.
    - The earliest subsequent reply if no comment was initially recorded.
- Do not retrieve comments from previous occurrences of the same status.
    - Example: An Enquiry is marked Unsuccessful, reopened, then marked Unsuccessful again without a comment. The export must leave the Unsuccessful comment blank rather than reuse the earlier comment.
- If the current status entry has no comment, leave the cell blank.
- For Enquiries owned by a Removed Referrer, always leave both comment columns blank, including when downloaded by a Super Admin.
- Preserve the original recorded comment text without automatic redaction.
- Prevent spreadsheet formula execution by ensuring comments and other text values are treated as text.

### Export behaviour

- Include Enquiries across **all Companies and all dates**, regardless of filters applied on the Reports page.
- Read current records as the download runs. Do not use a point-in-time snapshot.
- Stream the CSV in bounded batches to handle large datasets efficiently.
- Apply current-status and Removed Referrer rules as records are read to prevent historical comments from being exposed.
- Do not change the existing Reports page, Enquiries by Status CSV, or Facilities and Vacancies exports.