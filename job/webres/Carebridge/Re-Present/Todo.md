
# Add the Missing Email


# Refactoring
## Urgent
- [x] Update the email text.
- [x] Add the missing email that Lisa requested.
- [x] Update the controls so they look nicer.


## General
- [ ] Refactor the request-review transition into a dedicated action/service with a DB transaction.
- [ ] Move `can_request_review` calculation out of the resource or at least avoid per-item DB queries.
- [ ] Extract frontend action config generation from `ViewControls`.
## Clean
- Introduce a `RequestEnquiryReview` action and move controller logic into it.
- Move “can request review” rule out of the resource into domain or policy code.
- Split listener responsibilities and reduce event payload coupling.
- Remove seeded IDs from Playwright and make test setup explicit.

## Later
- [ ] Do something about that ugly PHP block in the \wsl.localhost\archlinux\home\james\Projects\portal\e2e\enquiry-review-flow.spec.ts file.
- [ ] Move Playwright env/login helpers into shared test utilities.