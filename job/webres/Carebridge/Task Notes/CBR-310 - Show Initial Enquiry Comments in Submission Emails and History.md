### Technical Details

- Map submission text is currently stored in enquiries.comments; history comments use the separate enquiry_comments table. Preserve the existing stored submission text.
- The submission email uses Markdown rendering. HTML escaping alone does not ensure that Markdown syntax stays literal.
- The initial comment must be associated with the first Facilities Enquired history entry for each newly created Enquiry. It must remain attached to that entry after subsequent status changes.
- For submissions to multiple Facilities, create one initial comment per Enquiry using the same submission text. Separate submissions must retain their own comments.
- Attribute the comment to the actual submitting User, including their Company and submission timestamp, rather than assuming the Application creator submitted it.
- Initial comments must be read-only. They should not be editable or deletable through existing discussion controls.
- Treat missing, empty, or whitespace-only input as no comment. Nonblank values such as 0 must be preserved.
- Preserve line breaks and render HTML and Markdown syntax as literal text in the history and both HTML and plain-text email versions. Ensure the complete comment is available through the existing Show more control and included in full in the email.
- Only the initial comment should appear in the Enquiry Submitted email. Subsequent discussion comments must not be included. The Additional Comments section should be omitted entirely when no initial comment exists.
- Preserve existing organisation-based access rules. Hospital Users may access comments on their authorised Enquiries, while Providers must only access comments associated with their own Enquiries.
- Creating an initial comment must not trigger a separate comment email or additional in-app Notification. Existing Enquiry Submitted recipients and notification counts must remain unchanged.
- The submitting User must be recognised as a previous commenter when determining recipients for subsequent discussion emails, even if they are not the Application creator. Existing recipient rules must still apply, including current Enquiry access and excluding the User posting the new comment.
- Ensure Enquiry creation and initial comment creation are atomic across the entire map submission. If any Enquiry or comment fails to save, roll back all Enquiries and comments created by that submission, and do not send submission emails or in-app Notifications.
- Existing duplicate-Enquiry checks and creation flows without an initial comment must remain unchanged.
- Do not backfill historical comments or generate emails for existing Enquiries.