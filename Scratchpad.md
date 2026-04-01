# TODO
- Make backup of DB before deploying these changes.

# Things to Update
- [x] ApplicationRepresentative.php
	- [x] Convert those arrays to Enums
- [x] 2026_03_31_000001_create_application_representatives_table.php
	- [x] Change the name of the other_epoa_contact_info column to 'notes', rather than making a new column.
- [x] Form formatting
	- [x] Delete button is too large, move into the top corner.
	- [x] Add a prompt before deleting
	- [x] Move title to the next row
	- [x] Remove Representative X title
- [ ] resources/js/pages/applications/components/form/components/epoa-details-section/index.tsx
	- [ ] Rather than using the append method directly, make a seperate helper method to wrap that function.
- [x] In /home/james/Projects/portal/app/Http/Resources/ApplicationResource.php, update the gate to use edit instead of view permission.
- [x] Backfill required EPOA fields during migration — /home/james/Projects/portal/database/migrations/2026_03_31_000001_create_application_representatives_table.php:60-72  
	  This backfill creates every legacy EPOA representative with authority and primary_decision_maker set to null, but the new validation now requires both for any EPOA row. After deploying this migration, any existing application that had legacy EPOA data will fail validation on the next edit unless a user manually repairs those new fields first, even if they were only changing an unrelated field.
- [ ] Tests
	- [ ] Also write tests for edge-cases.

Fix these:
  - [x] Gate premium payload with the view permission — /home/james/Projects/
    portal/app/Http/Resources/ApplicationResource.php:18-18
    This switched the resource gate from application-view-premium-fields to
    application-edit-premium-fields, but the React view still decides whether to
    render the section from the view permission. In any role setup where a user
    may view-but-not-edit premium data, the page will now show -/empty
    representative data because patient_has_capacity_to_make_decisions, notes,
    and representatives are omitted from the Inertia payload. - REVERT IN GIT 
  - [ ] Clear stale other_type when a representative stops being Other — /home/
    james/Projects/portal/resources/js/pages/applications/components/form/
    index.tsx:107-115
    This mapper forwards every representative field unchanged except
    primary_decision_maker. Because other_type is rendered conditionally and the
    form is not using shouldUnregister, a user can pick Other, enter Advocate,
    then switch the same row to EPOA or POA; React Hook Form will keep the hidden
    other_type value and it will be saved. The read-only card then renders
    incorrect labels such as EPOA: Advocate. 
  - [ ] Throw on transactional save failures instead of returning strings — /
    home/james/Projects/portal/app/Services/ApplicationService.php:269-269
    The new transaction only rolls back on exceptions, but the failure paths
    inside this closure all return error strings (store() and
    syncRepresentatives() swallow exceptions). If a representative insert fails
    after syncRepresentatives() has deleted the old rows, the request reports an
    error while still committing the delete and any earlier inserts, leaving the
    application partially updated. 



- [ ] Code review feedback:
	- [ ] Serialize representative fields for premium editors — /home/james/Projects/portal/app/Http/Resources/ApplicationResource.php:41-49  
	  The edit form is hydrated from application.data, but these new premium fields are only sent when the user has application-view-premium-fields. A role can have application-edit-premium-fields without that view permission, and in that case the sidebar opens with patient_has_capacity_to_make_decisions = null, notes = undefined, and representatives = []; submitting any unrelated edit then clears the saved representative data. Please expose these fields whenever the current user is allowed to edit them, or load the edit form from a source that is not gated by the view permission.
	- [ ] Backfill required EPOA fields during migration — /home/james/Projects/portal/database/migrations/2026_03_31_000001_create_application_representatives_table.php:60-72  
	  This backfill creates every legacy EPOA representative with authority and primary_decision_maker set to null, but the new validation now requires both for any EPOA row. After deploying this migration, any existing application that had legacy EPOA data will fail validation on the next edit unless a user manually repairs those new fields first, even if they were only changing an unrelated field.
	- [ ] Throw on save failures inside the new transaction — /home/james/Projects/portal/app/Services/ApplicationService.php:265-280  
	  DB::transaction() only rolls back when the closure throws, but the failure paths here just return error strings. Because store()/syncRepresentatives() catch exceptions and return strings, a representative insert failure after representatives()->delete() or an application_files save failure will still commit the earlier writes. On update that can permanently delete the old representatives even though the request is reported as failed.
# Things to Change
- [ ] Move ApplicationEnquiryReviewController to the Aplication Controller?
- [ ] 


# LLM Skills to Install
grill-me
improve-codebase-architecture

write-a-prd
prd-to-issues


# GitKraken on CLI
- Better log
- rename commits
- squash commits
	- 
- undo commits
- AI commit messages
- Better merge conflict resolution
- move commits up and down
- Create PRs



Car Service:
Wednesday 25th







---

Hi Lisa,

 Thanks for sending through the pen test report. I've updated the cards we discussed in the meeting today. If you could please review them and let me know if anything needs to be added or modified that would be great, thanks. 

ADD_CARDS_HERE

Cheers,
James

- https://linear.app/webres-solutions/issue/CBR-48/allow-re-presentation-of-previously-unsuccessful-patients
- https://linear.app/webres-solutions/issue/CBR-49/allow-hospital-users-to-respond-to-provider-comments
- https://linear.app/webres-solutions/issue/CBR-135/notification-subscriptions

---





Job to Apply for:
https://www.linkedin.com/jobs/view/4377110300/?trackingId=pUHbpsDyXQWpPSdEqlmo2g%3D%3D&refId=hw%2Bv1%2B55F4kyOiRIuhhUVA%3D%3D&midToken=AQEzzsZtjvla8Q&midSig=1h1zF_9OLbNs81&trk=eml-email_job_alert_digest_01-primary_job_list-0-job_posting_0_jobid_4377110300_ssid_15402721724_fmid_35t8t3~mma2qvgq~2g&trkEmail=eml-email_job_alert_digest_01-primary_job_list-0-job_posting_0_jobid_4377110300_ssid_15402721724_fmid_35t8t3~mma2qvgq~2g-null-35t8t3~mma2qvgq~2g-null-null&eid=35t8t3-mma2qvgq-2g&otpToken=MTMwYzFmZTExNDJmYzljMGIzMjQwNGVkNDExY2UyYjU4ZmNmZDE0Mjk4YWE4ODYxNzdjNzA4NmQ0ZTVhNThmMmY2ZGY4NmJkNTVjN2M0ODY2NzlmN2NmODlhMTc1NDMyZGViMWJmOTk1OGNjZGEyNTg2ZmIsMSwx


# Home Loan Application

| Item               | 2024     | 2025      |
| ------------------ | -------- | --------- |
| Taxable Income     | 6,534.00 | 10,216.00 |
| Depreciation       | 1,157.65 | 0         |
| Interest Add Backs | 0        | 0         |
| Operating Expenses | 5,269.09 | 1721<br>  |








Keyboards I like:
- [Charybdis MK2](https://bastardkb.com/product/charybdis-mk2-prebuilt-preorder/)
- [Ergokeyboards Crosses/Bridges Keyboard](https://ergokeyboards.com/products/crosses-modular-keyboard?variant=50272542228762)
- ZSA Moonlander



Good switch upgrades:
- Gateron Baby Kangaroo Switch
- K Pro Banana Switch