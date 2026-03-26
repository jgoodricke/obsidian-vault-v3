Add more tests:

```markdown
That looks good. Also implement the following tests (if they don't already exist):

BACKEND / FEATURE TESTS

Add tests for:
1. Authorisation
   - User without `enquiry-transition-to-re-present-status` permission cannot call the request-review endpoint
   - Response is forbidden and no enquiry status or history changes occur

2. Provider reject success path
   - Provider can reject a review request with valid comments
   - Enquiry moves from RE_PRESENT to UNSUCCESSFUL
   - Expected history entry is created

3. UI/resource exposure logic (API/resource level)
   - `can_request_review` (or equivalent) is false when user lacks permission
   - `can_request_review` is false when enquiry is not eligible

4. Notification/mail edge case
   - No mail is queued when facility email is null

5. Event dispatch
   - Successful request-review dispatches `EnquiryReviewRequested`
   - Use Event::fake() and Event::assertDispatched()

6. Validation boundary
   - Comments exceeding max length (eg 1001 chars for max:1000) are rejected
   - No invalid state/history mutation occurs

FRONTEND / PLAYWRIGHT TESTS

Add Playwright tests for the review workflow UI:
1. "Request Review" button visibility
   - Visible when enquiry is latest unsuccessful and user has permission
   - Not visible when user lacks permission
   - Not visible when enquiry is not eligible

2. Provider review modal validation
   - Reject action requires comments
   - Submitting without comments shows validation error
   - Submitting with comments succeeds

3. Approve / Reject controls visibility
   - Only visible when enquiry status is RE_PRESENT
   - Not visible in other statuses

GENERAL CONSTRAINTS
- Prefer extending existing test files rather than creating many new ones
- Follow existing factories and test patterns
- Keep assertions precise (response + database state)
- Do not modify production code unless necessary for testability
- Skip any scenario already covered in the test suite

At the end, return:
- list of tests added
- anything skipped because already covered
- whether any production code changed
```


# Add the Missing Email


# Refactoring
## Urgent
- Do something about that ugly PHP block in the \\wsl.localhost\archlinux\home\james\Projects\portal\e2e\enquiry-review-flow.spec.ts file.

## General
- Refactor the request-review transition into a dedicated action/service with a DB transaction.
- Move `can_request_review` calculation out of the resource or at least avoid per-item DB queries.
- Extract frontend action config generation from `ViewControls`.
- Move Playwright env/login helpers into shared test utilities.

## Clean
- Introduce a `RequestEnquiryReview` action and move controller logic into it.
- Move “can request review” rule out of the resource into domain or policy code.
- Split listener responsibilities and reduce event payload coupling.
- Remove seeded IDs from Playwright and make test setup explicit.