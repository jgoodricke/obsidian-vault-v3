## Technical Details

The 100-row limit must apply when retrieving Notifications from the database, not after loading the complete backlog. Performance acceptance concerns the full Home request, not a notification-only probe. The existing notification-only 100-item probe took 2.31 seconds; that is not a full Home measurement or an agreed response-time target.

### Additional Implementation Notes

- The existing 30-second execution timeout was reproduced locally using a dataset containing 2,361 unread Notifications.
- Impersonation itself succeeds. The failure occurs during the subsequent Home request, which uses the same path as normal sign-in.
- Retrieve only the current User's 100 newest unread Notifications, then apply existing validation and access checks within that fixed selection.
- Display the first 50 valid Notifications in descending order of recency.
- Preserve existing recipient and target access checks, including Organisation Access restrictions.
- Delete selected orphaned Notifications before any title, patient details, content, or URL is returned to the browser.
- Do not fetch additional batches to replace deleted or inaccessible Notifications.
- Only display "50+ unread" when at least 51 valid Notifications are confirmed within the selected 100. Do not infer the count from unverified Notifications.
- Notifications outside the selected 100 must not be inspected, deleted, or marked as read during the Home request.
- Valid Notifications that are checked but not displayed must remain unread.
- Individual mark-as-read actions must retain their existing recipient-only behaviour.
- "Mark all as read" must continue to operate across the current User's entire unread backlog, including Notifications outside the selected or displayed set, without affecting other Users.
- Do not introduce caching, query batching, or an impersonation-specific workaround as part of this change.