## Branch Name
feature/cbr-315-retire-company-logo-uploads-and-display

## Technical Details

### Data Retention and Deployment

- Required data-retention contract: rename companies.logo to DEPRECATED_logo, preserve its nullable varchar(255) definition and all values, and move it to the end of companies. It must no longer be an active application attribute.
- The owner will schedule deployment downtime for the shared schema and both application flavours.
- Agreed decision record: docs/adr/0054-retire-company-logos.md. Removing application use does not itself end public access to existing objects.

### Application Changes

- Remove Company Logo upload controls, previews and Company table images from both application flavours.
- Reject or ignore legacy Company Logo submissions without uploading objects or modifying retained filenames.
- Company responses must no longer expose uploaded logo filenames or public logo URLs.
- Remove application references to the deprecated logo attribute, including reads and writes.
- Preserve existing permissions, organisation scoping and unrelated Company validation.

### Dependencies and Storage

- Remove the unused public-storage helper, public-disk settings and logo-only image-processing package and configuration.
- Company administration must work without public-disk configuration or public-bucket access.
- Preserve shared upload functionality and AWS configuration required for private Application File storage.
- Keep existing Application File uploads, downloads and malware-clearance rules unchanged.

### Cloud Resources and Future Cleanup

- Do not delete or modify the deployed public bucket, existing objects or cloud permissions.
- Do not inventory bucket contents or investigate external consumers.
- The owner will handle removal of the deprecated database column and public bucket later.
- No temporary compatibility release or zero-downtime deployment design is required.