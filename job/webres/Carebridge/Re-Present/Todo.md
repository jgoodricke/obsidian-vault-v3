
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
- [ ] Refactor the Event Listeners?
## Clean
- Introduce a `RequestEnquiryReview` action and move controller logic into it.
- Move “can request review” rule out of the resource into domain or policy code.
- Split listener responsibilities and reduce event payload coupling.
- Remove seeded IDs from Playwright and make test setup explicit.

## Later
- [ ] Do something about that ugly PHP block in the \wsl.localhost\archlinux\home\james\Projects\portal\e2e\enquiry-review-flow.spec.ts file.
- [ ] Move Playwright env/login helpers into shared test utilities.



  ## Explicit Event Listener Registration Plan

  ### Summary

  - App-defined listeners currently present in the codebase:
      - App\Events\EnquirySubmitted -> App\Listeners\SendEnquirySubmittedNotification
      - App\Events\EnquiryStatusChanged -> App\Listeners\SendEnquiryStatusChangedNotification
      - App\Events\EnquiryStatusChanged -> App\Listeners\SendEnquiryReviewRespondedNotification
      - App\Events\EnquiryStatusStale -> App\Listeners\SendEnquiryStatusStaleNotification
      - App\Events\EnquiryReviewRequested -> App\Listeners\SendEnquiryReviewRequestedNotification
      - Illuminate\Auth\Events\Logout -> App\Listeners\LogAuthenticationEvents@handleLogout
      - Logout -> LogAuthenticationEvents@handleLogout is not registered there and would have
      - No other app-level Event::listen, subscribers, observers, or additional discovery-based
        listeners were found.
  - Framework-managed listener to leave alone:
      - Illuminate\Auth\Events\Registered ->
        Illuminate\Auth\Listeners\SendEmailVerificationNotification
      - Laravel wires this automatically; it does not need to be copied into the app provider.

  ### Implementation Changes

  - Update app/Providers/EventServiceProvider.php so the provider is the single explicit source of
    truth for all app-owned listeners.
  - Add imports for:
      - Illuminate\Auth\Events\Logout
      - App\Listeners\LogAuthenticationEvents
  - Extend $listen with:
      - Logout::class => [LogAuthenticationEvents::class.'@handleLogout']
  - Keep the existing enquiry event mappings unchanged.
  - Do not refactor LogAuthenticationEvents to handle() or __invoke; register the existing
    handleLogout method explicitly.
  - Clear or rebuild cached events after the change, because this repo has a checked-in/generated
    bootstrap/cache/events.php and it is already stale relative to current listener code.

  ### Test Plan

  - Add a focused feature test for logout listener wiring:
      - Fake only OwenIt\Auditing\Events\AuditCustom
      - Dispatch Illuminate\Auth\Events\Logout
      - Assert AuditCustom is dispatched with the user carrying auditEvent = 'logged-out'
  - Keep existing enquiry event tests as regression coverage for the five already-explicit
    mappings.
  - After the code change, verify cached-event behavior by clearing caches before the suite and
    confirming tests pass with the explicit provider registration in place.

  ### Assumptions And Defaults

  - Scope is limited to app-owned listeners; vendor/framework listeners stay framework-managed
    unless there is a specific reason to take ownership of them.
  - The minimal safe fix is to make EventServiceProvider complete for app listeners, not to re-
    enable discovery.
  - The only currently affected behavior from discover: false is logout auditing; no other
    discovery-only listeners were found in the repository.