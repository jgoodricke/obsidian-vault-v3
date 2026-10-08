## Technical Details

- Inventory Exports represent current inventory data and should remain separate from the existing Reports implementation.
- Use Eloquent queries and lazy streaming so exports do not require the entire dataset to be loaded into memory.
- Load related data efficiently in batches to avoid N+1 queries when building the embedded relationship columns.
- Generated export files should not be retained after download.
- A failure after streaming begins may result in a partial client-side file. The failure should be written to the application log.

### Access

- All Inventory Export routes must require Super Admin access.
- Authorisation must also apply directly to the download endpoints, not just navigation visibility.
- Impersonation must use the permissions of the impersonated User.

### Filenames and timestamps

- Filenames should contain the dataset name and generation date/time.
- Export timestamps and filename timestamps should use `Australia/Sydney`.
- Timestamp values should use `YYYY-MM-DD HH:mm:ss`.

### Facilities export

Include active Facilities only (`facilities.removed = 0`).

Each Facility should appear as a single row, with its related Accommodation Types, Room Types and Specific Service Deliveries embedded in additional columns.

Columns, in order:

`facility_id`, `company_id`, `company_name`, `facility_name`, `street_number`, `street`, `city`, `state`, `country`, `postcode`, `notification_emails`, `phone_number`, `website`, `fax_number`, `geohash`, `latitude`, `longitude`, `govt_subsidised`, `cbc_engaged_provider`, `benevolent_provider`, `carers_gateway`, `dementia_care`, `refreshed_at`, `created_at`, `updated_at`, `accommodation_type_names`, `room_type_names`, `specific_service_delivery_names`

The additional relationship columns should contain:

- `accommodation_type_names`: Names of related Accommodation Types.
- `room_type_names`: Names of related Room Types.
- `specific_service_delivery_names`: Names of related Specific Service Deliveries.

Retrieve these relationships from `facilities_relationships`, using `type_name` and `type_id` to resolve the corresponding catalogue records.

If an active Facility does not have a valid owning Provider:

- Include the Facility.
- Leave `company_name` blank.
- Log the data integrity issue.

### Vacancies export

Include only Vacancies that:

- Have not been removed.
- Were updated within the last 10 days, inclusive.
- Belong to an active Facility.

Each Vacancy should appear as a single row, with its related Accommodation Types, Room Types and Specific Service Deliveries embedded in additional columns.

Columns, in order:

`vacancy_id`, `facility_id`, `facility_name`, `benevolent_provider`, `carers_gateway`, `cbc_engaged_provider`, `dementia_care`, `gender`, `geohash`, `latitude`, `longitude`, `govt_subsidised`, `created_at`, `updated_at`, `accommodation_type_names`, `room_type_names`, `specific_service_delivery_names`

The additional relationship columns should contain:

- `accommodation_type_names`: Names of related Accommodation Types.
- `room_type_names`: Names of related Room Types.
- `specific_service_delivery_names`: Names of related Specific Service Deliveries.

Retrieve these relationships from `vacancies_relationships`, using `type_name` and `type_id` to resolve the corresponding catalogue records.

### Relationship handling

- Aggregate related names into the corresponding columns of the parent Facility or Vacancy row.
- Separate multiple names with semicolons (`;` ).
- Sort related names alphabetically to ensure consistent output.
- Leave relationship columns blank when no matching relationships exist.
- Only load relationships belonging to Facilities or Vacancies eligible for the corresponding export.
- Existing relationships to deprecated Accommodation Types should remain included.
- If a relationship references a catalogue entry that no longer exists, exclude that relationship from the aggregated values and log the data integrity issue.
- Relationship-specific `created_at` and `updated_at` values are not required in the new export format.
- Preserve all valid relationship names, including where multiple relationships reference entries with the same name.

### CSV formatting

- UTF-8 with a byte order mark.
- Snake-case column headings.
- Null values represented as blank cells.
- Facility Notification Emails separated using semicolons.
- Embedded relationship values separated using semicolons.
- Apply standard CSV quoting and escaping to fields containing special characters.
- Protect text values that spreadsheet applications could interpret as formulas, including aggregated relationship fields.
- Do not alter numeric latitude or longitude values as part of formula protection.
- Rows should have a stable order using the Facility or Vacancy identifier.
- Related names should have a stable alphabetical order within each cell.
- An export with no matching records should contain the header row only.
- Each download is generated independently from the data available when requested. The two downloads do not need to form a synchronised snapshot.