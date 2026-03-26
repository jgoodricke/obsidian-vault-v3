# Investigate Input Handling and Error Exposure

## Input Inventory

### Form Input Entry Points

| Surface | Frontend entry point | Backend endpoint(s) | Key user-controlled fields | Type | Risk | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| Authentication | `resources/js/pages/Auth/Login.tsx` | `POST /login` | `email`, `password` | Structured credentials | High | Backend uses `LoginRequest`; frontend also enforces email format and non-whitespace password. |
| Authentication | `resources/js/pages/Auth/ForgotPassword.tsx` | `POST /forgot-password` | `email` | Structured identifier | Medium | Password-reset initiation; email existence is reflected back to user. |
| Authentication | `resources/js/pages/Auth/ResetPassword.tsx` | `POST /reset-password` | `email`, `password`, `passwordConfirmation`, hidden `token` | Structured identifier + secret | High | Frontend checks password confirmation; backend custom flow only validates `email`, `token`, `password`. |
| Authentication | `resources/js/pages/Auth/TwoFA.tsx` | `POST /2fa` | `two_factor_code` | Structured code | High | Frontend expects a 6-digit code; backend only requires presence. |
| User admin | `resources/js/pages/users/components/form/index.tsx` | `POST /users`, `PUT /users/{id}` | `first_name`, `last_name`, `email`, `phone`, `password`, `role_id`, `company_id`, derived `company_ids`, `facility_ids[]` | Semi-structured profile data + structured IDs | High | Sensitive administrative surface; mixed frontend and service validation. |
| User profile | `resources/js/pages/users/profile.tsx` | `PUT /user-profile` | `first_name`, `last_name`, `email`, `phone`, `password` | Semi-structured profile data + secret | Medium | Same backend service as user admin, but with profile-specific field stripping. |
| Company admin | `resources/js/pages/Companies/FormCompany.tsx` | `POST /companies`, `POST /companies/{id}` | `name`, `abn_number`, `email`, `phone`, `mobile`, `type_id`, `plan_id`, `portal`, `street`, `city`, `state`, `postcode`, `logo` upload | Semi-structured business/contact data + file upload | High | Includes image upload and multiple structured selectors. |
| Facility admin | `resources/js/pages/Facilities/FormFacility.tsx` | `POST /facilities`, `PUT /facilities/{id}` | `name`, `email`, `phone`, `fax_number`, `website`, `street_number`, `street`, `city`, `state`, `postcode`, `lat`, `lng`, `description`, `company_id`, `accommodation_type_ids[]`, `room_type_ids[]`, `specific_service_deliverie_ids[]`, feature enums | Semi-structured address/contact data + free text + structured IDs/enums | High | Address parts are populated from Google Places on the client, then posted back to Laravel. |
| Vacancy update | `resources/js/pages/Facilities/Vacancy/AddVacancy.tsx`, `resources/js/pages/Facilities/Vacancy/FormUpdateVacancy.tsx` | `PUT /facilities/{facility}/vacancy` | `accommodation_type_ids[]`, `room_type_ids[]`, `specific_service_deliverie_ids[]`, `dementia_care`, `gender`, `govt_subsidies`, `cares_gateways`, `cbc_engaged_provider` | Structured IDs/enums | Medium | Repeating form rows post an array of vacancy payloads. |
| Application create/edit | `resources/js/pages/applications/components/form/index.tsx` plus section components | `POST /applications/new`, `POST /applications/edit/{id}` | Patient identity, DOB, demographics, address, EPOA details, care requirements, multiple assessment/support fields, ADL fields, free-text summaries, `facility_id`, `submit_button`, `files[]` upload | Mixed structured, semi-structured, free text, and file upload | High | Largest input surface in the app; contains multiple free-text clinical/context fields and attachment uploads. |
| Map enquiry submission | `resources/js/pages/map/components/content/components/info-panel/components/enquiry-dialog/index.tsx` | `POST /applications-enquiry` | `application_id`, `facility_ids[]`, `additional_information`, nested `criteria` copied from URL/search state (`search_option`, `room_types[]`, `accommodation_types[]`, `specific_service_deliveries[]`, `dementia_care`, `govt_subsidies`, `cbc_engaged_provider`, `provider`, `cares_gateways`, `gender`, `lat`, `lng`, `zoom`) | Structured IDs/enums + free text + query-derived state | High | This is a form submission that mixes explicit form fields with query-parameter-derived filters from the map UI. |
| Application status transition | `resources/js/components/app/status-transition-modal/index.tsx` via `resources/js/pages/applications/components/view-controls/components/status-transition-controls/index.tsx` | `POST /application/{id}/transition/{statusId}` | `comments` | Free text | Medium | Comment requirement depends on target status. |
| Enquiry status transition | `resources/js/components/app/status-transition-modal/index.tsx` via `resources/js/pages/enquiries/components/view-controls/index.tsx` | `POST /enquiries/{id}/transition/{statusId}` | `comments` | Free text | Medium | Same modal pattern as application bulk transition. |
| Patient transfer | `resources/js/pages/applications/components/view-controls/components/patient-transfer-controls/index.tsx` | `POST /applications/{id}/patient-transfer` | `company_id` | Structured ID | Medium | Narrow structured input, but changes ownership and triggers notification emails. |
| Provider acknowledgement / decision | `resources/js/pages/Email/Email.tsx` | `GET /enquiry-acknowledge/{status}/{uuid}/{decision}`, `GET /applications-enquiry/{status}/{uuid}/{decision}` | `decision` from select button, route params `status`, `uuid` | Structured enum/token route input | Medium | Rendered as a form-like confirmation page, then submitted back through route params. |

### Non-Form Inputs With Message / Error Feedback Potential

This subsection narrows to user-controlled inputs that are not primarily ordinary form fields, but still drive success, failure, or error feedback that can reveal workflow state, resource existence, permission boundaries, or environment details.

| Surface | Backend endpoint(s) | User-controlled input | Feedback channel | Risk | Notes |
| --- | --- | --- | --- | --- | --- |
| Provider email decision links | `GET /enquiry-acknowledge/{status}/{uuid}/{decision}`, `GET /applications-enquiry/{status}/{uuid}/{decision}` | Route params `status`, `uuid`, `decision` | `403` with `Your URL is invalid.`, success flash `Your response has been recorded.`, or backend error surfaced via `withErrors(...)` | High | Different outcomes disclose whether the UUID resolves, whether the status/decision pair is allowed, and whether the underlying decision write succeeded. |
| Status-transition routes | `POST /application/{id}/transition/{statusId}`, `POST /enquiries/{id}/transition/{statusId}` | Route params `id`, `statusId` | Validation errors returned to Inertia and shown in shared toast UI | High | Invalid transitions can surface enum names via messages like `Transition from X to Y is not allowed.`, revealing internal workflow states rather than a generic rejection. |
| Application file access | `GET /application-files/{applicationFile}`, `DELETE /application-files/{applicationFile}` | Route param `applicationFile` | `403`, `404`, or success flash `File deleted successfully.` | Medium | Distinct authorization, existence, and success outcomes allow probing whether a file record exists, whether backing storage is present, and whether the current user can act on it. |
| Map query string | `GET /map` | Query params such as `search_option`, `application_uuid`, and filter arrays | Validation errors via `withErrors($validator)` and conditional UI state changes | Medium | `search_option` is the only validated query key; invalid values produce explicit errors, while valid `application_uuid` values change whether application context is hydrated into the page. |
| Login credentials | `POST /login` | Credential pair `email` + `password` | Validation errors `auth.failed` and `auth.throttle` shown in the global toast UI | Medium | Although submitted via a form, the sensitive feedback comes from auth logic: throttling reveals rate-limit state and failed logins distinguish generic auth rejection from local validation failures. |
| Password reset initiation | `POST /forgot-password` | `email` | Validation errors and explicit message `We can't find a user with that e-mail address.` | High | The response path reflects account existence directly, making the email value a message-sensitive input rather than just a form field. |
| Two-factor challenge | `POST /2fa` | `two_factor_code` | Inline error `You entered the wrong code or the code is expired.` and success redirect | Medium | Combined with the non-production `GET /2fa` code disclosure, this input lets the user distinguish missing, wrong, expired, and valid codes. |

### Frontend-Only / Not Wired To A Backend Endpoint

`resources/js/pages/Owner/LongApplication/LongApplication.tsx` contains a large form surface, but its `onSubmit` is currently a no-op. It should be treated as frontend-only until it is connected to a backend route.

## Validation Gaps

Only surfaces with substantive validation gaps are listed below. `Login credentials` and `Patient transfer` were reviewed and omitted because the current server-side enforcement did not show a concrete validation-bypass issue.

### Authentication: Forgot Password

- Validation implementation: inline controller validation in `PasswordResetLinkController@forgotPassword`; no `FormRequest`; frontend adds its own regex checks in `resources/js/pages/Auth/ForgotPassword.tsx`.
- Consistency / reuse: weak. The same `required|email` rule is duplicated in two controller methods, and the frontend regex is looser than Laravel's `email` validator.
- Uploaded files: not applicable.
- Gaps: low-severity validation drift only. The active backend rule is sound, but validation is not centralized or reusable, and client-side email validation can accept strings the backend rejects.

### Authentication: Reset Password

- Validation implementation: service-level validation in `PasswordResetTokenService::validatePasswordResetInput()`, called from `NewPasswordController::resetPassword()`; no `FormRequest`; frontend confirmation logic lives in `resources/js/pages/Auth/ResetPassword.tsx`.
- Consistency / reuse: inconsistent. The active `/reset-password` path uses weaker service rules than the alternate `/reset-password-store` flow, which uses Laravel password defaults and `confirmed`.
- Uploaded files: not applicable.
- Gaps: password confirmation is frontend-only and bypassable; the live backend only requires `email`, `token`, and `password`; the password rule is overly permissive (`string|min:8|max:32`) and does not enforce confirmation or stronger password semantics.

### Authentication: Two-Factor Challenge

- Validation implementation: inline controller validation in `TwoFAController@store`; no `FormRequest` or reusable service validator; frontend enforces a 6-digit pattern in `resources/js/pages/Auth/TwoFA.tsx`.
- Consistency / reuse: inconsistent. The UI treats the field as a 6-digit numeric code, while the backend accepts any non-empty string and relies on lookup logic afterward.
- Uploaded files: not applicable.
- Gaps: server-side validation is overly permissive; format validation is frontend-only; missing-format cases are not strongly covered by tests.

### User Admin

- Validation implementation: plain `Request` objects in `UserController`, with manual service validation in `UserService::validateInputs()`; frontend also adds separate `react-hook-form` rules in `resources/js/pages/users/components/form/index.tsx`.
- Consistency / reuse: partially reused across create/update, but not aligned with the actual payload. Backend validates `company_id`, while the form derives and posts `company_ids`; create-mode password requirements also differ between frontend and backend.
- Uploaded files: not applicable.
- Gaps: `company_ids` is not validated at all before being written to `user_groups`; `role_id` is only `numeric`, with no `exists` or allowlist check; `company_id` lacks `exists`; `facility_ids` validation can dereference a null company and 500; password is only frontend-required on create, so a crafted create request can fall through to a database error instead of a validation error; frontend and backend disagree on whitespace/name handling.

### User Profile

- Validation implementation: plain `Request` in `UserController@updateProfile`, delegating to `UserService::validateInputs()`; the dedicated `ProfileUpdateRequest` is used by a different controller flow; frontend rules live in `resources/js/pages/users/profile.tsx`.
- Consistency / reuse: reused from the admin flow, but only partially appropriate for profile updates and still inconsistent with the frontend.
- Uploaded files: not applicable.
- Gaps: profile updates can change the password without verifying the current password or requiring confirmation; the shared validator still has the same `company_ids` / `facility_ids` weaknesses as the admin path; frontend rules differ from the admin form and are not the server source of truth.

### Company Admin

- Validation implementation: plain `Request` in `CompanyController`, with manual service validation in `CompanyService::validateInputs()`; frontend required checks live in `resources/js/pages/Companies/FormCompany.tsx`.
- Consistency / reuse: create and update reuse the same backend validator, and `logo` uses reusable upload rules, but frontend and backend rules have drifted badly.
- Uploaded files: yes. `logo` is validated server-side as an image upload with mime, extension, size, and dimension checks.
- Gaps: the backend validates `company_name`, but the form submits `name`, so the submitted company name is effectively not validated; `type_id`, `plan_id`, and `portal` are frontend-required but not properly validated server-side; dropdown-backed fields are not checked against allowed values; frontend upload limits for `logo` are far looser than backend limits.

### Facility Admin

- Validation implementation: plain `Request` in `FacilityController`, with service-level validation in `FacilityService::validateInputs()`; frontend only marks a small subset of fields required in `resources/js/pages/Facilities/FormFacility.tsx`.
- Consistency / reuse: create and update share the same backend validator, but the validation path is non-standard because the controller validates decoded JSON while persistence uses `$request->all()`.
- Uploaded files: not applicable.
- Gaps: server-side rules do not require `name` or `email` even though the UI does; `company_id` and related ID arrays are only validated as numeric/distinct, with no `exists` enforcement; `state` is free text server-side despite a constrained dropdown in the UI; `lat` / `lng` have no range checks; `website`, `phone`, and `fax_number` only have shallow string validation; the composite `address` field is collected client-side but not validated or stored consistently.

### Vacancy Update

- Validation implementation: plain `Request` in `VacancyController`, with per-row service validation in `VacancyService::validateInputs()`; the React forms mostly rely on dropdown option lists without substantive validation rules.
- Consistency / reuse: repeated row payloads reuse the same backend validator per row, but the outer array shape is never validated and the route/controller semantics are inconsistent with a true single-vacancy update.
- Uploaded files: not applicable.
- Gaps: empty vacancy rows can pass validation and be saved because all backend fields are optional; the repeated array container is assumed rather than validated; relationship ID arrays only use `numeric|distinct`, with no `exists` checks and no database FKs to catch bad IDs.

### Application Create / Edit

- Validation implementation: plain `Request` in `ApplicationController`, with service-level validation in `ApplicationService::validateInputs()`; frontend sections in `resources/js/pages/applications/components/form/` carry the majority of `required` logic.
- Consistency / reuse: create and edit share the same backend validator, but that validator covers only a small subset of the fields the form actually submits.
- Uploaded files: yes. `files.*` is validated server-side with `file`, mime, extension, and size rules.
- Gaps: most required patient, address, aged-care, care-needs, ADL, and free-text fields are frontend-only; conditional requirements such as EPOA details, referral/update comments, homecare details, and behavioural summaries are not enforced server-side; create-only consent is explicitly dropped before submission and never validated; `facility_id` is used later in the flow without validation; many select/boolean fields are persisted without backend `in`, `array`, or `boolean` rules; the model is effectively filled from `$request->post()` without a tight validated whitelist.

### Map Enquiry Submission

- Validation implementation: plain `Request` in `ApplicationController@mapSubmitEnquiry`, with service-level validation in `ApplicationService::validateEnquiryInputs()`; frontend validation in the enquiry dialog is effectively absent beyond UI wiring.
- Consistency / reuse: weak. GET `/map` and POST `/applications-enquiry` validate different subsets of the same criteria state, and the typed frontend criteria shape does not match the server validator.
- Uploaded files: not applicable.
- Gaps: query-derived criteria such as `lat`, `lng`, `zoom`, `postcode`, and `application_uuid` are copied into the POST body but not validated server-side; arbitrary `criteria[*]` keys are later persisted to `search_details`; `facility_ids` only uses `numeric|distinct` and has no FK protection; `application_id` is only validated as numeric, with no request-time existence or access check.

### Application Status Transition

- Validation implementation: inline controller validation in `ApplicationController::transition`; no `FormRequest`; comment rules are built from `EnquiryStatus.comments_required`; the modal in `resources/js/components/app/status-transition-modal/index.tsx` has no meaningful client-side requirement enforcement.
- Consistency / reuse: duplicated inline with the enquiry transition flow instead of being centralized. Comment enforcement is server-side, but the actual transition-state enforcement is inconsistent.
- Uploaded files: not applicable.
- Gaps: the bulk transition path updates enquiries with `updateQuietly()`, which bypasses the model event hooks that normally enforce allowed transitions; invalid or no-op transitions can still create history rows; comment rules are enforced server-side, but not surfaced as strong frontend constraints.

### Enquiry Status Transition

- Validation implementation: inline controller validation in `EnquiryController::transition`; no `FormRequest`; frontend modal only reflects server-returned errors.
- Consistency / reuse: the allowed-transition map is enforced server-side, but comment validation is duplicated inline and there are multiple sources of truth for whether comments are required.
- Uploaded files: not applicable.
- Gaps: when comments are optional, the controller still unconditionally reads `$validated['comments']`, so crafted requests that omit the key can trigger an undefined-array-key / 500 path; frontend validation is weaker than backend validation for comment requirements and length.

### Provider Acknowledgement / Decision

- Validation implementation: inline controller and service checks in `ApplicationController` and `ApplicationService`; no `FormRequest`; frontend only constrains the button choice in `resources/js/pages/Email/Email.tsx`.
- Consistency / reuse: weak. Validation is split across two endpoints and a service method, with different allowlists and different response paths for the same input classes.
- Uploaded files: not applicable.
- Gaps: the public routes have no route-level constraints for `status`, `uuid`, or `decision`; `uuid` is existence-checked but not format-validated; `/enquiry-acknowledge/...` and `/applications-enquiry/...` handle invalid `uuid` values differently; the service uses a broader status allowlist than the public-link controller path, so the contract is not enforced from one reusable source of truth.

### Non-Form Inputs With Message / Error Feedback Potential

All rows in the non-form table were reviewed with GPT-5.4 sub-agents. `Login credentials` is intentionally omitted here because the `LoginRequest` remains the authoritative server-side validator for `POST /login`, and the review did not find a concrete validation gap in that message-sensitive path.

#### Provider Email Decision Links

- Validation implementation: inline controller checks in `ApplicationController::providerStatusDecision()` and `ApplicationController::providerEnquiryAcknowledgement()`, plus a second service-layer check in `ApplicationService::storeProviderStatusDecision()`; no `FormRequest`; frontend only constrains the button selection in `resources/js/pages/Email/Email.tsx`.
- Consistency / reuse: weak. The controller allowlist is `EnquiryHistory::PROVIDER_EMAIL_DECISION_STATUSES`, but the service re-validates against the broader `EnquiryHistory::ENQUIRY_STATUSES`, and the acknowledgement page does not reuse the service validator at all.
- Uploaded files: not applicable.
- Gaps: the public GET routes have no route-level constraints or signed-link protection for `status`, `uuid`, or `decision`; `uuid` is checked by lookup rather than format validation; valid link inputs render sensitive enquiry context before authentication; invalid inputs are handled differently across the two controller paths; the same input class is enforced by duplicated, drifting controller/service logic instead of one reusable validator.

#### Status-Transition Routes

- Validation implementation: route params are only number-constrained in `routes/web.php`; `comments` is validated inline in `ApplicationController::transition()` and `EnquiryController::transition()`; transition legality is enforced separately in model logic via `HasTransitions`; frontend transition lists are duplicated in `resources/js/transitions/enquiry-status-transitions.ts`.
- Consistency / reuse: inconsistent. Comment validation is duplicated inline, while transition legality is split between controller flow, model events, and frontend filtering rather than centralized in a shared validator or request object.
- Uploaded files: not applicable.
- Gaps: the bulk application route `POST /application/{id}/transition/{statusId}` updates enquiries with `updateQuietly()`, bypassing the `saving` hook that normally blocks illegal transitions; the single-enquiry route does use `$enquiry->save()` and therefore enforces different server behavior for the same status-input class; `statusId` resolution also leaks different outcomes for nonexistent versus existing-but-forbidden statuses because `findOrFail()` and permission errors take separate paths.

#### Application File Access

- Validation implementation: route-level `whereNumber('applicationFile')` plus implicit model binding; download authorization is in `ApplicationFileController::canAccessApplicationFile()` and delete authorization is in `ApplicationFile::canBeDeletedBy()`; there is no `FormRequest`, service validator, policy, or scoped binding for this surface.
- Consistency / reuse: weak. The same `applicationFile` route param is validated by routing, then handled through two different authorization paths, and the file-storage path is trusted directly from the database in both actions.
- Uploaded files: not applicable to this access path, although the records originate from validated upload creation.
- Gaps: callers can distinguish non-numeric IDs, nonexistent records, and existing-but-forbidden records through different outcomes; the read/delete actions do not validate that the persisted storage path still matches the expected `applications/{id}` namespace before calling filesystem operations; the download path assumes the bound file has a valid owning application even though the schema does not enforce that relation, so an orphaned record can fall into an inconsistent 500-style path instead of a controlled validation failure.

#### Map Query String

- Validation implementation: one-off inline controller validation in `MapController::index()` for `search_option` only; no `FormRequest`; reusable criteria validation exists later in `ApplicationService::validateEnquiryInputs()` for the POST enquiry flow, not for `GET /map`; frontend constraints are only TypeScript and controlled-widget assumptions.
- Consistency / reuse: weak. The GET surface validates only one field, while the POST surface validates a larger overlapping criteria set, so the same query-derived inputs are treated differently across the map flow.
- Uploaded files: not applicable.
- Gaps: array filters such as `room_types`, `accommodation_types`, and `specific_service_deliveries` are consumed as arrays without GET-time validation; `lat`, `lng`, and `zoom` have no server-side validation and are only frontend-coerced; `application_uuid` is looked up directly without UUID-format validation and silently drops context when invalid or inaccessible; enum-style filters are silently ignored on GET but validated on POST, creating inconsistent feedback for the same user-controlled criteria.

#### Password Reset Initiation

- Validation implementation: inline controller validation in `PasswordResetLinkController`; no `FormRequest`; frontend regex checks exist in `resources/js/pages/Auth/ForgotPassword.tsx`; there is no reusable service validator for the initiation path.
- Consistency / reuse: inconsistent. `routes/web.php` and `routes/auth.php` both register `POST /forgot-password` to different controller methods, and those methods use different validation and response behavior.
- Uploaded files: not applicable.
- Gaps: the custom initiation flow performs a direct `User::where('email', ...)` lookup and returns an explicit account-existence message; the two controller methods duplicate `email` validation instead of sharing one validator; frontend and backend checks are not the same source of truth; the message-sensitive path therefore depends on fragmented controller logic rather than a reusable request or service rule set.

#### Two-Factor Challenge

- Validation implementation: inline controller validation in `TwoFAController::store()`; no `FormRequest` or service-level validator; frontend-only format checks are implemented in `resources/js/pages/Auth/TwoFA.tsx`.
- Consistency / reuse: weak. The login step uses a `FormRequest`, but the follow-up 2FA challenge falls back to ad hoc controller validation, and the frontend enforces a stricter six-digit rule than the backend.
- Uploaded files: not applicable.
- Gaps: server-side validation only requires presence, so non-digit, wrong-length, whitespace-padded, or oversized codes are accepted into the lookup path; the six-digit numeric constraint is frontend-only; empty input and malformed-but-non-empty input produce different response paths and message handling, which makes this message-sensitive auth challenge rely on inconsistent validation layers instead of one authoritative backend rule.

## Rendering Risks

Only forms with substantive rendering or output-encoding risks are listed below. Login, forgot password, reset password, 2FA, vacancy update, and map enquiry submission were reviewed but not included because their current React or Blade output paths render submitted values as ordinary escaped text rather than through a known raw-HTML, markdown-reinterpretation, or export-injection sink.

### Non-Form Inputs With Message / Error Feedback Potential

All rows in the non-form table were reviewed with GPT-5.4 sub-agents. None of them adds a new standalone rendering-risk row beyond what is already captured elsewhere in this section:

- `Provider email decision links`: omitted as a separate row. The concrete sink is still the existing `Application create / edit` issue, where controller-built `confirmationMsg` HTML is rendered via `dangerouslySetInnerHTML`; the route params themselves are allowlisted and do not independently create a new encoding problem.
- `Status-transition routes`: omitted as a separate row. The route params and resulting status-name errors are rendered as ordinary escaped toast text; the only substantive sink remains transition `comments`, which are already covered under `Application status transition` and `Enquiry status transition`.
- `Application file access`: omitted. The `applicationFile` route param is not rendered, file names are rendered through normal React text/attribute sinks, and success/error messages are fixed strings. This remains a feedback/authorization surface, not a concrete output-encoding issue.
- `Map query string`: omitted. Query values are used as React state, input values, and selection state, not raw HTML/markdown; the canonical downstream map/application views do not replay arbitrary query strings into unsafe sinks.
- `Login credentials`: omitted. Authentication errors are fixed translated strings shown through the shared escaped toast UI, and submitted credentials are not reflected back to the user.
- `Password reset initiation`: omitted. The submitted email is only shown via masked plain-text React output, and the visible success/error strings are static. The issue here is account enumeration, not rendering.
- `Two-factor challenge`: omitted. The submitted code is never reflected, the visible error is static, and the non-production demo code is rendered as escaped React text rather than HTML.

### User Admin

- Rendering implementation: submitted user names and emails are rendered back out in admin CSV exports via `SendCompanyFacilityUserCsvExport`, which concatenates allocated-user strings and writes them with `fputcsv()` in [SendCompanyFacilityUserCsvExport.php](/home/james/Projects/portal/app/Console/Commands/SendCompanyFacilityUserCsvExport.php#L71) and [SendCompanyFacilityUserCsvExport.php](/home/james/Projects/portal/app/Console/Commands/SendCompanyFacilityUserCsvExport.php#L140).
- Output encoding review: there is no neutralization step for spreadsheet-special leading characters before writing user-controlled `first_name`, `last_name`, or `email` values into CSV cells.
- Risk: a crafted user field starting with `=`, `+`, `-`, or `@` will be emitted as an active spreadsheet formula when admins open the CSV export, which is an export-layer injection issue rather than an HTML/XSS issue.

### User Profile

- Rendering implementation: profile edits feed the same exported allocated-user strings as the admin user form, again through [SendCompanyFacilityUserCsvExport.php](/home/james/Projects/portal/app/Console/Commands/SendCompanyFacilityUserCsvExport.php#L71) and [SendCompanyFacilityUserCsvExport.php](/home/james/Projects/portal/app/Console/Commands/SendCompanyFacilityUserCsvExport.php#L140).
- Output encoding review: the export path trusts user-maintained profile fields and writes them straight to CSV.
- Risk: profile-controlled names and emails can become spreadsheet formulas in exported CSV attachments because there is no prefixing or sanitization for formula-trigger characters.

### Company Admin

- Rendering implementation: company names and `portal` values are rendered into the company/facility export attachment in [SendCompanyFacilityUserCsvExport.php](/home/james/Projects/portal/app/Console/Commands/SendCompanyFacilityUserCsvExport.php#L77) and [SendCompanyFacilityUserCsvExport.php](/home/james/Projects/portal/app/Console/Commands/SendCompanyFacilityUserCsvExport.php#L140); company names also flow into notification and mail markdown templates such as [enquiry-submitted.blade.php](/home/james/Projects/portal/resources/views/notifications/content/enquiry-submitted.blade.php#L1) and [enquiry-submitted-notification.blade.php](/home/james/Projects/portal/resources/views/mail/enquiry-submitted-notification.blade.php#L4).
- Output encoding review: the CSV path does not escape or neutralize spreadsheet meta characters in `name` or `portal`, and the markdown templates do not neutralize markdown control characters before rendering.
- Risk: a crafted company name or portal value can execute as a formula in spreadsheet software when the export is opened, and a crafted company name can also alter formatting or introduce links in markdown-rendered notifications or emails.

### Facility Admin

- Rendering implementation: facility-controlled fields are written into two CSV export surfaces: facility names in [SendCompanyFacilityUserCsvExport.php](/home/james/Projects/portal/app/Console/Commands/SendCompanyFacilityUserCsvExport.php#L115) and full address/contact details in [SendHomesByStateCsvExport.php](/home/james/Projects/portal/app/Console/Commands/SendHomesByStateCsvExport.php#L75) plus [SendHomesByStateCsvExport.php](/home/james/Projects/portal/app/Console/Commands/SendHomesByStateCsvExport.php#L102). The `website` field is also rendered directly into a clickable anchor on the map info panel in [location-info/index.tsx](/home/james/Projects/portal/resources/js/pages/map/components/content/components/info-panel/components/body/components/location-info/index.tsx#L123).
- Output encoding review: exported `name`, `street`, `city`, `email`, `phone`, and `website` values are passed directly to `fputcsv()` without spreadsheet-safe encoding. The map link applies no scheme allowlist before assigning `website` to `href`; it only prepends `//` when no `//` is present.
- Risk: facility-managed text fields can trigger CSV formula injection when recipients open the exported file in Excel or Sheets, and a crafted `website` value such as `javascript://...` can be rendered as a clickable frontend link.

### Application Create / Edit

- Rendering implementation: application fields are rendered safely in most React and Blade views, but the provider acknowledgement flow lifts submitted patient/care details into an HTML string in [ApplicationController.php](/home/james/Projects/portal/app/Http/Controllers/Portal/ApplicationController.php#L399) through [ApplicationController.php](/home/james/Projects/portal/app/Http/Controllers/Portal/ApplicationController.php#L438), which is then rendered by [Email.tsx](/home/james/Projects/portal/resources/js/pages/Email/Email.tsx#L42).
- Output encoding review: `applicantName`, `careType`, and `careLength` are interpolated directly into HTML with `sprintf()` rather than escaped as text before React receives them.
- Risk: an application value such as the patient first name can inject markup into the provider acknowledgement page because the downstream renderer treats the whole string as trusted HTML.

### Application Status Transition

- Rendering implementation: transition comments are rendered safely in the timeline view through [comments-display/index.tsx](/home/james/Projects/portal/resources/js/components/app/application-view/components/comments-display/index.tsx#L4), but the same comments also flow into notification templates in [enquiry-status-changed.blade.php](/home/james/Projects/portal/resources/views/notifications/content/enquiry-status-changed.blade.php#L1) and [enquiry-status-changed-to-information-required.blade.php](/home/james/Projects/portal/resources/views/notifications/content/enquiry-status-changed-to-information-required.blade.php#L1), then into [NotificationPresenter.php](/home/james/Projects/portal/app/Services/NotificationPresenter.php#L14) and [notification-item.tsx](/home/james/Projects/portal/resources/js/pages/home/components/notification-item.tsx#L58).
- Output encoding review: the notification path treats user comments as markdown content rather than plain text. HTML is still escaped, but markdown control characters are not neutralized before `react-markdown` reinterprets the string.
- Risk: a crafted transition comment can alter formatting or introduce rendered links in the notification feed, so output handling is inconsistent between the safe timeline view and the markdown notification sink.

### Enquiry Status Transition

- Rendering implementation: enquiry-level transitions dispatch [EnquiryStatusChanged.php](/home/james/Projects/portal/app/Events/EnquiryStatusChanged.php#L8), which is turned into notifications by [SendEnquiryStatusChangedNotification.php](/home/james/Projects/portal/app/Listeners/SendEnquiryStatusChangedNotification.php#L16) and ultimately rendered through the same Blade-markdown-plus-`react-markdown` path as the bulk application transition flow.
- Output encoding review: as with application transitions, comments are inserted into markdown templates without markdown escaping.
- Risk: enquiry transition comments can be rendered back out with attacker-controlled formatting or links in notifications even though the direct enquiry/application history view keeps them as plain text.

### Patient Transfer

- Rendering implementation: patient-transfer inputs are rendered back out in the markdown email body sent by [PatientTransferNotification.php](/home/james/Projects/portal/app/Mail/PatientTransferNotification.php#L24) and [patient-transfer-notification.blade.php](/home/james/Projects/portal/resources/views/mail/patient-transfer-notification.blade.php#L2).
- Output encoding review: patient and hospital names are inserted into a Laravel markdown mailable with ordinary Blade escaping, but without markdown-specific escaping before the message body is parsed into email HTML.
- Risk: crafted patient or hospital names can inject markdown formatting or rendered links into the transfer email body. This is weaker than raw HTML injection, but it is still inconsistent output encoding for user-controlled administrative data.

### Provider Acknowledgement / Decision

- Rendering implementation: the confirmation page renders `confirmationMsg` with `dangerouslySetInnerHTML` in [Email.tsx](/home/james/Projects/portal/resources/js/pages/Email/Email.tsx#L42).
- Output encoding review: the raw-HTML render is fed by a controller-generated string that mixes HTML fragments with route-derived status text and application-derived content from [ApplicationController.php](/home/james/Projects/portal/app/Http/Controllers/Portal/ApplicationController.php#L411) through [ApplicationController.php](/home/james/Projects/portal/app/Http/Controllers/Portal/ApplicationController.php#L438).
- Risk: this page bypasses React’s normal escaping entirely; current allowlists constrain `status`/`decision`, but any untrusted data inserted into `confirmationMsg` is rendered as live HTML, making this the clearest XSS sink in the application.
 
## Error Handling Findings

Only surfaces with substantive error-handling, message-exposure, or inconsistent-response findings are listed below. `Authentication / login` and the non-form `Login credentials` row were reviewed and omitted because the active `LoginRequest` path keeps auth failures generic apart from standard throttle feedback. `Status-transition routes` is also omitted as a separate non-form row because its meaningful findings are already captured under the application and enquiry transition form surfaces.

### Cross-Cutting: Exception Handling And Environment

- `app/Exceptions/Handler.php` is effectively the framework default and does not normalize these flows into a consistent API / Inertia error contract. Several controllers and services therefore fall back to a mix of `abort(...)`, `withErrors(...)`, raw exception strings, and uncaught 500s depending on which layer fails.
- `InteractsWithDatabaseTrait::store()` and `update()` catch exceptions, log them, and return the raw exception message. Controllers then commonly pass that string into `withErrors(...)`, and the shared toast UI renders it directly. This is the main recurring mechanism by which database / storage / integration details can leak to end users.
- Checked-in local configuration currently has `APP_DEBUG=true` and `DEBUGBAR_ENABLED=true` in `.env`. Any uncaught exception path below therefore has a materially higher risk of exposing framework, stack, driver, or query details in local-style deployments.

### Authentication: Forgot Password

- Error handling implementation: the live `POST /forgot-password` route resolves to the custom `forgotPassword()` controller method, not the broker-backed `store()` method. It returns `withErrors(...)` for bad format and unknown email, then redirects known users to the reset-request page.
- Findings: account existence is directly exposed through both the message and navigation outcome. Unknown addresses get `We can't find a user with that e-mail address.`, while known addresses are redirected to `/reset-password-request/{email}`. The duplicate controller method / duplicate route registration also means the same surface has two different error-handling contracts in the codebase, even though only one currently matches requests.

### Authentication: Reset Password

- Error handling implementation: `POST /reset-password` uses the custom `NewPasswordController::resetPassword()` path, manually decodes JSON, validates in `PasswordResetTokenService`, and turns validation and token failures into `abort(403, ...)` responses instead of field-scoped validation errors.
- Findings: malformed reset payloads return concatenated validator text via a 403 page rather than the normal Inertia error bag. Success also flashes `success`, but Inertia only shares `flash.message`, so the success state is handled inconsistently with the rest of the app.

### Authentication: Two-Factor Challenge

- Error handling implementation: `POST /2fa` validates only presence, then distinguishes empty input, non-empty invalid / expired input, and valid input by taking different response paths. `GET /2fa` renders the destination email and, outside production, the active OTP value.
- Findings: the main exposure is not wrong-versus-expired differentiation, because those are intentionally collapsed to one message. The real issue is that the challenge page itself discloses the live code in non-production and that failure-mode page state differs between the normal GET render and the failed POST re-render.

### User Admin

- Error handling implementation: `POST /users` and `PUT /users/{id}` use service validation plus `withErrors(...)` on failure, while persistence runs through `InteractsWithDatabaseTrait::store()`.
- Findings: raw persistence errors can reach the UI through the shared toast when invalid `company_id`, missing create-time password, or other bad inputs fall through validation and fail in the DB layer. Response handling is also inconsistent after the base save: some bad relation inputs can silently succeed, some become user-visible DB errors, and some can bubble out as uncaught exceptions.

### User Profile

- Error handling implementation: `PUT /user-profile` reuses the same service validation / persistence pattern as admin user writes, but submits from a separate profile page.
- Findings: raw exception text can be exposed via `withErrors($saveResult)`, and the save-failure branch redirects to `users.index` instead of back to the profile page. Server-side errors are not rendered inline on the form, so profile-specific failures appear only in the global toast.

### Company Admin

- Error handling implementation: company create / update validates a narrow subset of fields, then persists through the shared DB helper. After the DB write, logo processing runs separately through Intervention Image and the configured filesystem.
- Findings: bad or missing `name`, `type_id`, and `plan_id` can bypass validation and surface raw DB exception text in the toast. Logo-processing and filesystem failures are uncaught and non-atomic: the company row can already be saved when image scaling or disk writes fail, producing a 500 instead of a normal form error. Update also uses hard `403` aborts such as `ID not found.` instead of the normal form-error path.

### Facility Admin

- Error handling implementation: facility create / update manually decode JSON, validate through `FacilityService`, and redirect back with errors on validation failure. The base model save is wrapped, but follow-up relationship writes are not.
- Findings: raw DB errors from the base save can reach the user through the shared toast, while relation attach / detach failures bypass that path and can become uncaught 500s. Validation failures redirect back, but save failures redirect to the facilities index, so the user loses form context for only one class of error. On the frontend, the Google Places path has almost no defensive error handling and assumes address / geometry data exists whenever the loader succeeds.

### Vacancy Update

- Error handling implementation: vacancy updates iterate over an array of row payloads, validate and save each row inside the same loop, and stop on the first validation failure.
- Findings: there is no outer payload validation, no row-indexed error reporting, and no transaction, so earlier rows can already be committed when a later row fails. More importantly, the controller combines boolean success with `&&`, so non-empty error strings from model saves remain truthy and can still produce `Vacancy updated successfully.` even when a row failed internally.

### Application Create / Edit

- Error handling implementation: application create / edit validate a small subset of fields plus attachments, then save the application row first and upload files second. The same shared DB helper is used for both the application row and `application_files` rows.
- Findings: raw DB / storage exception text can be returned via `withErrors(...)`, and multi-step failures are not atomic. A later file upload failure can return an error even though the application record and some files were already persisted. Create has an extra inconsistency: the optional first-enquiry side effect is not checked correctly, so an enquiry / history write can fail and the request can still return success.

### Map Enquiry Submission

- Error handling implementation: `POST /applications-enquiry` validates through `ApplicationService`, writes enquiries first, then writes map-search details, and reports failures through `withErrors(...)`.
- Findings: search-detail persistence failures can expose raw exception text in the shared toast. Enquiry persistence is handled even less safely: the save helper’s string return values are treated as truthy, so some enquiry / history failures are swallowed and the request can still continue or succeed. The result is a partially committed flow where the user can see an error after enquiries already exist, or see success even though part of the write path failed.

### Application Status Transition

- Error handling implementation: the bulk application transition route resolves the target status, checks a derived permission, validates `comments`, and then uses `updateQuietly()` on each enquiry before manually inserting history rows.
- Findings: failure modes are inconsistent by layer. Bad route params 404, inaccessible applications 403, forbidden statuses 403 with the target status name, and comment failures come back as Inertia validation errors. Because `updateQuietly()` bypasses model transition hooks, this route does not surface the normal `Transition from X to Y is not allowed.` validation path at all. There is also a likely crafted-request 500 when `comments` is optional but omitted and the controller still reads it unconditionally.

### Enquiry Status Transition

- Error handling implementation: the single-enquiry transition route resolves the enquiry and target status, checks a target-specific permission, validates only `comments`, then relies on `Enquiry::save()` to trigger the `HasTransitions` model guard.
- Findings: this surface exposes internal workflow metadata through messages like `Transition from X to Y is not allowed.` and `You don't have permission to transition to {status}`. Response types are inconsistent for similar bad inputs: route middleware can 403, missing statuses 404, inaccessible enquiries 403, illegal transitions redirect with validation errors, and only comment errors are wired inline on the modal while `status_id` errors fall through to the global toast.

### Patient Transfer

- Error handling implementation: patient transfer validates `company_id`, scopes the application lookup, persists the ownership change, then sends notification emails synchronously and dispatches the audit event.
- Findings: the transfer is committed before mail / audit side effects run, and those side effects are uncaught. A mail or audit failure can therefore produce a 500 after the ownership change has already been saved. The route also reveals internal state through differentiated messages for nonexistent / invalid company IDs, wrong company type, same-hospital transfers, and inaccessible applications.

### Provider Acknowledgement / Decision

- Error handling implementation: the acknowledgement page and decision-submit route are both public GET routes. The acknowledgement page collapses invalid UUIDs and invalid params into a fixed `403 Your URL is invalid.`, while the submit route distinguishes bad allowlist input, unknown UUIDs, write failures, and success.
- Findings: there is no raw exception leakage here, but the flow is still a message-sensitive oracle. A valid link reveals enquiry / application context before authentication, invalid UUIDs on the state-changing route become `Enquiry not found`, failed writes become `Your response cannot be recorded. Please try again.`, and successful submissions show `Your response has been recorded.`.

### Non-Form Inputs With Message / Error Feedback Potential

All rows in the non-form table were reviewed with GPT-5.4 sub-agents. `Login credentials` is intentionally omitted here because the active login flow keeps account-state failures behind the generic `auth.failed` path, and `Status-transition routes` is omitted as a separate row because its concrete findings are already captured under `Application status transition` and `Enquiry status transition`.

#### Provider Email Decision Links

- Error handling implementation: `/enquiry-acknowledge/{status}/{uuid}/{decision}` and `/applications-enquiry/{status}/{uuid}/{decision}` do not share one normalized response contract. The first collapses bad UUID and bad allowlist input into one `403`, while the second distinguishes bad allowlist input, unknown UUID, write failure, and success.
- Findings: this is a limited existence / update oracle for guessed UUIDs. It does not expose raw backend details, but it does let callers distinguish allowlisted links from invalid ones, resolvable UUIDs from missing ones on the submit route, and successful writes from failed ones.

#### Application File Access

- Error handling implementation: `GET /application-files/{applicationFile}` and `DELETE /application-files/{applicationFile}` rely on numeric route constraints, implicit model binding, controller-local authorization, and private-disk existence checks.
- Findings: the meaningful signal here is behavioral rather than textual. Nonexistent file rows 404, existing-but-forbidden rows 403, accessible-but-missing blobs 404 on download, and successful deletes return a flash success. One extra robustness gap is that `application_files.application_id` has no FK, so orphaned file rows can fall outside the intended 403 / 404 contract.

#### Map Query String

- Error handling implementation: `GET /map` validates only `search_option`, redirects with errors for invalid values, forces a redirect when `search_option` is missing, and silently renders `application = null` when `application_uuid` is unresolvable.
- Findings: this is a probeable state surface because a resolvable `application_uuid` hydrates enquiry mode and reveals application context in the map UI, while an invalid or missing one silently removes that context. Query handling is also inconsistent: `search_option` gets explicit error feedback, while the other overlapping criteria keys are accepted or ignored silently on GET even though the downstream POST flow validates a broader set.

#### Password Reset Initiation

- Error handling implementation: the active `POST /forgot-password` path validates email format, returns a direct existence message for unknown accounts, and redirects known accounts to the reset-request page after issuing the token.
- Findings: account existence is exposed through both the returned message and the navigation outcome. The success path also places the raw email address into the redirect URL even though the next page masks it visually. The presence of the alternate, unused `store()` method means the same surface still has a second dormant error contract in the codebase.

#### Two-Factor Challenge

- Error handling implementation: `POST /2fa` distinguishes empty input, non-empty invalid / expired input, and valid input, while `GET /2fa` always renders the masked destination and, outside production, the current OTP.
- Findings: the most important exposure is the non-production OTP disclosure on `GET /2fa`, not wrong-versus-expired differentiation. The challenge also has inconsistent page-state handling because the failed POST render does not include the demo code prop that the normal GET render supplies in non-production.
