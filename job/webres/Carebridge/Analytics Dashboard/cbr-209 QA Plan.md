2QA Plan — Facility Notification Emails (cbr-52d460f8 / cbr-209)

▎ Feature branch: feature/cbr-209-allow-facilities-to-have-multiple-emails
▎ Check out before testing: git checkout feature/cbr-209-allow-facilities-to-have-multiple-emails

0. Setup

1. ./vendor/bin/sail up
2. ./vendor/bin/sail exec web yarn dev
3. ./vendor/bin/sail artisan migrate:fresh --seed
4. Open the app URL from .env; log in as an admin user from .env; complete the 2FA prompt (code is
printed on the page).
5. Open Mailpit at http://localhost:$FORWARD_MAILPIT_DASHBOARD_PORT and clear the inbox before each
   notification scenario.

---
1. Database / migration integrity

| # | Step | Expected |
|---|---|---|
| 1.1 | ./vendor/bin/sail artisan migrate:fresh --seed then inspect the facilities table | notification_emails column exists (JSON, default json_array()) sitting after postcode; email column has been renamed to DEPRECATED_email and now sits after updated_at |
| 1.2 | SELECT id, notification_emails, DEPRECATED_email FROM facilities LIMIT 10; | Every seeded facility has notification_emails populated as a JSON array with at least one address. DEPRECATED_email is populated for legacy rows but is not read anywhere. |
| 1.3 | Re-run the migration down then up | Reversal restores email column from notification_emails[0]; re-running up re-creates notification_emails, backfills with trimmed values, and renames the column again. No data lost. |
| 1.4 | Create a facility row directly with email = ' spaced@example.com ' in a fresh DB, then run the migration | After migration, notification_emails JSON contains "spaced@example.com" (whitespace trimmed, per 8b95739b). |

---
2. Facility form — Create

Navigate to Facilities → "Add Facility".

| # | Step | Expected |
|---|---|---|
| 2.1 | Open the create form | "Notification Emails" label visible; one empty email input row; "Add Email" button below; trash icon next to the only row is disabled |
| 2.2 | Click "Add Email" twice; fill three valid addresses (e.g. a@x.com, b@x.com, c@x.com); fill required form fields; Save | Facility saves; success redirect; notification_emails in DB is ["a@x.com","b@x.com","c@x.com"] |
| 2.3 | Repeat 2.2 but enter the same address twice (e.g. a@x.com, a@x.com) | Form accepts and saves both entries (per acceptance criteria: backend does not enforce dedup beyond what the frontend provides) |
| 2.4 | Click "Add Email" then leave the new row blank and Save | Inline "Provide a valid email address" error on the blank row; submission blocked |
| 2.5 | Enter not-an-email in one row, valid in another, Save | Row-level error on the invalid row; submission blocked |
| 2.6 | Enter valid@x.com  (trailing space) and Save | Saves successfully; stored value is trimmed to valid@x.com (verify in DB) |
| 2.7 | Try to remove the only row | Trash icon disabled; the row cannot be removed (form-level minimum of 1 enforced visually) |

---
3. Facility form — Edit

| # | Step | Expected |
|---|---|---|
| 3.1 | Open the edit form for a seeded facility that has a single notification email | Form pre-populates the existing address in row 1; trash icon disabled on that row |
| 3.2 | Add two more addresses, save | Detail view shows all three; DB array reflects all three |
| 3.3 | Re-open edit form, remove the middle row, save | Detail view and DB reflect the remaining two rows in original order |
| 3.4 | Re-open edit form, edit the first row to a different valid address, save | Detail view and DB reflect the new address; remaining row unchanged |
| 3.5 | Re-open edit form, clear row 1 (leave the input empty), Save | Row-level error; submission blocked |
| 3.6 | Re-open edit form, add a new row, type an invalid value into an existing row | Adding a new row does not wipe the existing row's error (regression covered by e8639fab) |
| 3.7 | Open DevTools Network tab and submit a valid edit | Payload notification_emails is a flat array of strings (e.g. ["a@x.com","b@x.com"]), not the internal {value: …} shape |

---
4. Facility detail view

| # | Step | Expected |
|---|---|---|
| 4.1 | View a facility with three notification emails | Detail row labelled "Notification Emails"; value is a comma-separated list of all three addresses in order |
| 4.2 | View a facility with one notification email | Single address rendered with no trailing comma |
| 4.3 | View a (hypothetical) facility with an empty notification_emails array | Field renders as empty string without throwing (defensive null-coalescing path) |

---
5. Notification dispatch — Enquiry Submitted

6. Clear Mailpit.
7. From the hospital side, submit an Enquiry to a facility that has three notification emails.

| # | Expected |
|---|---|
| 5.1 | Three separate emails appear in Mailpit, each addressed to one of the three notification emails (one mail per recipient, not a single multi-To mail) |
| 5.2 | Each email's subject/body matches the existing EnquirySubmittedNotification template; "to" display name is the facility name |
| 5.3 | Repeat against a facility with one notification email — exactly one email sent (the migrated-data case) |
| 5.4 | Repeat against a facility with an empty notification_emails array (set manually in DB) — zero emails sent and no exception thrown |

---
8. Notification dispatch — Enquiry Review Requested

9. Clear Mailpit.
10. Trigger a "Review Requested" event on an Enquiry whose facility has multiple notification emails
   (via the provider flow or by firing the event in php artisan tinker).

| # | Expected |
|---|---|
| 6.1 | One EnquiryReviewRequestedNotification email queued per address (same dispatch behaviour as Enquiry Submitted) |
| 6.2 | Repeat for a single-email facility — exactly one email sent |
| 6.3 | Repeat for an empty list — zero emails sent, no exception |

---
11. Downstream consumers (regression sweep — covered by audit task cbr-52d460f8.09)

| # | Surface | Expected |
|---|---|---|
| 7.1 | Vacancy map list (/vacancy-search map view) — open a facility marker | Email link uses mailto:first@example.com,second@example.com (no spaces, comma-separated, no empty entries) |
| 7.2 | Same as 7.1 with a facility that has whitespace-padded addresses in the joined string | buildLocationEmailMailtoHref strips whitespace and empty entries (per 9bbc3f84) |
| 7.3 | php artisan Homes-by-state CSV export (SendHomesByStateCsvExport) | Email column contains comma-joined notification emails for each facility |
| 7.4 | Facility list page / table | Renders without errors — schema changes did not break any column lookups |
| 7.5 | Any feature that previously read facility.email in code | `git grep -n "facility->email\b" app/ resources/` and `git grep -n -- "->email\>" app/ resources/` should return no live references outside of the migration itself (covered by audit cbr-a24dc769) |

---
12. API contract (FacilityResource)

| # | Step | Expected |
|---|---|---|
| 8.1 | GET /facilities/{id} (full view) | Response JSON contains notification_emails: string[]; no email key |
| 8.2 | GET /facilities?mapList=1 | Same — notification_emails is in the map payload too |
| 8.3 | POST /facilities with notification_emails: null | 302 redirect back with validation error; no record created (regression covered by a0a7eeca) |
| 8.4 | POST /facilities omitting notification_emails | Validation error: notification_emails is required |
| 8.5 | POST /facilities with notification_emails: [] | Validation error: array min 1 |
| 8.6 | POST /facilities with notification_emails: ["not-an-email"] | Validation error on notification_emails.0 |

---
13. Legacy / backward compatibility

| # | Step | Expected |
|---|---|---|
| 9.1 | Migrated facility (one address that came from DEPRECATED_email) | Continues to receive Enquiry Submitted and Review Requested emails as before; no behaviour difference visible to the provider |
| 9.2 | DEPRECATED_email value is still present in the DB row | But is not used by any application code path — confirm by spot-changing DEPRECATED_email directly and verifying it has no effect on dispatch |

---
14. Explicit out-of-scope (do not treat as bugs)

- No per-email metadata (label, role, notification-type filter).
- No self-service management by Provider users.
- No multi-recipient support extended beyond Enquiry Submitted and Review Requested.
- No backend dedup of identical addresses.
- DEPRECATED_email column is intentionally still present.

---
11. Automated test gate

Before sign-off, on the feature branch:

./vendor/bin/sail exec web yarn fix
./vendor/bin/sail exec web ./vendor/bin/php-cs-fixer fix
./vendor/bin/sail exec web composer lint
./vendor/bin/sail exec web yarn ci
./vendor/bin/sail exec web yarn lint-styles
./vendor/bin/sail exec web yarn test
./vendor/bin/sail exec web composer test

Key tests to confirm green: FacilityNotificationDispatchTest,
FacilityNotificationEmailsFoundationTest, EnquiryReviewFlowTest, OrganizationAuthorizationTest,
VacancyMapNotificationEmailsTest, HomesByStateCsvExportTest,
resources/tests/Pages/Facilities/FormFacility.test.tsx,
resources/tests/Pages/Map/BuildLocationEmailMailtoHref.test.ts.

---
Want me to drop this into a file (e.g. docs/qa/cbr-209-facility-notification-emails.md) or attach
it as a comment on the bd epic via bd note cbr-52d460f8?
