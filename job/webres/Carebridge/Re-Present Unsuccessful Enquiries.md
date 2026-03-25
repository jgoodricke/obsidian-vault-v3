## Summary

- Add a new persisted enquiry status, Re-Present, to reopen an existing unsuccessful enquiry without creating a new enquiry or application.
- Hospital users request review from the application-side history view; providers respond from the provider enquiry controls.
- Approval returns the enquiry to Clinical Review; rejection returns it to Unsuccessful.
- All review comments are stored as normal `enquiry_histories` comments, so repeated review cycles remain on the same enquiry timeline.

## Public Interfaces

- Add backend and frontend enum value `RePresent = 45` with display label `Re-Present`.
- Add permission `enquiry-transition-to-re-present-status` for hospital roles plus system/super admin.
- Add a hospital-side `POST` action, e.g. `application.enquiry-request-review`, at `/applications/{application}/enquiries/{enquiry}/request-review`.
  - Request body: `comments` required, string, max `1000`.
- Reuse existing provider route `POST /enquiries/{id}/transition/{statusId}` for:
  - Approval: `Re-Present -> Clinical Review`
  - Rejection: `Re-Present -> Unsuccessful`
- Add notification type `ENQUIRY_REVIEW_REQUESTED = 6` with provider-facing notification templates and mail template.

## Implementation Changes

### Status Model / State Machine

- Seed `Re-Present` into `enquiry_statuses` with `comments_required = true`.
- Add transitions `Unsuccessful -> Re-Present` and `Re-Present -> Clinical Review|Unsuccessful` in both PHP and TS transition maps.
- Add `Re-Present` color mapping using the same active/in-progress styling as `Clinical Review`.

### Hospital Request Flow

- Implement a dedicated application-scoped controller action instead of reusing `EnquiryController@transition`, because hospital users cannot pass the provider-side transition checks cleanly.
- Update the enquiry to `Re-Present`, create the history row with the hospital comment, and dispatch a new provider-targeted review-request event/notification.
- In provider `ViewControls`, when current status is `Re-Present`, replace generic transition buttons with `Approve Request` and `Reject Request`.
- Approval posts to the existing transition endpoint with target `Clinical Review`; comment is optional.

### UI

- In the shared application history timeline, show `Request Review` only on the latest history row when the enquiry’s current status is `Unsuccessful` and the user has the new permission.
- Reuse or extend the existing modal so it supports custom title/body/submit label and required vs optional comments.
- Continue rendering comments in history exactly as other status comments are rendered.

### Notifications

- Create a dedicated review-request notification/email path that mirrors `EnquirySubmitted` recipient selection: facility-assigned provider users plus facility email.
- Include the hospital comment in the provider notification content.
- Route notification clicks to the provider enquiry view, likely by targeting `EnquiryHistory` and extending notification-presenter routing for the new type.

## Test Plan

- Feature test: hospital user can request review on an unsuccessful enquiry, which creates a `Re-Present` history row with the comment and updates the enquiry status.
- Feature test: request review fails for missing comment, wrong application/enquiry pairing, inaccessible application, or any current status other than `Unsuccessful`.
- Provider approval comment is optional; provider rejection comment is required.
- `Request Review` appears only on the latest/current unsuccessful history row, not on older unsuccessful rows from prior cycles.
- Unlimited re-presentation cycles are allowed on the same enquiry.
- Existing undo behaviour stays unchanged, even for review-flow history entries.
- No extra history attribution fields are added; status + comment + timestamp remain the visible audit trail.
