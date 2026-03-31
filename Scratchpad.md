  1. What is the canonical ownership path for each model?
     Define it explicitly per model
  2. Is “organisation-owned” actually one concept here, or are there two scopes: company scope and
     facility-assignment scope? 
     they are two scopes.
  3. What is the rule precedence? 
     global admin bypass -> company scope -> facility assignment -> ability-specific rule.
  4. What does User ownership mean? 
     User is owned by company_id. user_groups is depricated and should be removed.
  5. What does create authorize against when no record exists yet? For company-owned models, authorize against the parent model
  6. Do you want query-time filtering, action-time authorization, or both? 
     Both
     ApplicationService.php:48 and app/Services/EnquiryService.php:14.
  7. How will you prevent route model binding leaks?
     Recommended: Either avoid implicit route-model binding for org-owned resources until scoped
     bindings are in place, or authorize immediately after binding and never vary the response by
     existence. Your acceptance criterion says 403, so be consistent: cross-org existing and
     nonexistent should not produce distinguishable behavior in these flows.
  8. Which abilities are truly generic, and which are domain actions?
     Recommended: Keep view/create/update/delete generic. Put transition, undo, transfer, refresh,
     download behind ability methods only where the action has distinct business rules. Don’t force
     every policy to implement meaningless methods just for symmetry.
  9. What is “global admin access still works where intended” supposed to exclude?
     Recommended: Write the exceptions down now. If there are records even super/system admins should
     not mutate, encode that explicitly. “Where intended” becomes a bug farm otherwise.
  10. What is the unauthorized contract for background jobs, exports, and file streaming?
     Recommended: Standardize one rule for web responses and another for internal callers. File
     downloads in app/Http/Controllers/Portal/ApplicationFileController.php:14 are a good example
     where you need the same policy logic but not necessarily the same UX handling.
  11. Are transition rules about object ownership, current state, or both?
     Recommended: Both, but separate them. Policy decides actor/resource access. Domain service
     decides whether the state transition itself is valid. Don’t bury state-machine logic in
     policies.
  12. How much migration are you actually willing to do in one pass?
     Recommended: Don’t try to flip every controller and query path at once. Start with the highest-
     risk record actions and the most duplicated rules: Application, Enquiry, ApplicationFile,
     Facility, Vacancy.






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