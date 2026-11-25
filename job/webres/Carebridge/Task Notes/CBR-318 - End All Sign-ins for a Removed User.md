## Technical Details

- Retain the existing User record and soft deletion behaviour.
- Check the User’s current removal status when an authenticated request reaches the server.
- Terminate the session of a Removed User before the protected request is processed, preventing both unauthorised data access and modifications.
- Apply this check to all authenticated sessions, including those established before the User was removed and those belonging to Users removed before deployment.
- Ensure Remember Me functionality cannot restore access for a Removed User.
- Prevent Removed Users with pending two-factor authentication from completing verification or requesting another verification code.
- Redirect unauthorised browser requests to the usual sign-in page.
- Return HTTP 401 for unauthorised JSON requests.
- Neither response should disclose the reason the session was terminated.
- Prevent Super Admins from initiating impersonation of a Removed User.
- If a User is removed during impersonation:
  - Restore the Super Admin’s original authenticated session and redirect them to their usual landing page.
  - Do not execute the interrupted request under either identity.
  - Require reauthentication if the Super Admin has also been removed or their original session cannot be safely restored.
- Bulk deletion of stored sessions is not required.
- Requests already authorised by the server before the User was removed do not need to be cancelled.
- Existing Deployment Admission behaviour, including the HTTP 403 response, must remain unchanged.