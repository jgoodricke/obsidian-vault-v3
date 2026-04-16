# Pre-merge Authorization Regression Plan

Summary

- Goal: prove this branch did not break organisation scoping, policy enforcement, plan-derived permissions, or the highest-risk workflow paths before merge.
- Pass criteria: targeted authorization suites pass, the full repo-required quality gates pass, the carebridge manual matrix passes, and there are no unexpected visibility leaks, false 403s, or misrouted review emails.
- Overlap rule: prove each distinct scope rule once. Do not repeat the same happy path for both free and paid unless the paid plan changes permissions or fields.

Coverage Axes / Interfaces

- Organize coverage around the scope engine in `app/Services/OrganisationAccessService.php`, policy wiring in `app/Providers/AuthServiceProvider.php`, permission/role definitions in `config/reference-data.php`, and seeded personas in `database/seeders/LocalTestDataSeeder.php`.
- Distinct rules to prove: hospital company scope, provider assigned-facility scope, provider admin company-wide facility scope, facility-manager assigned-facility scope, account-manager scope bypass without treating it as global permission, system-admin global bypass, and paid-plan premium-field permissions.
- No external API contract changes need separate validation; this is an access-control and workflow-regression plan.

Execution Order

- Prep the primary environment in carebridge at http://localhost:8082. Use seeded local users as the base dataset, complete 2FA with the demo code shown on the page, and create disposable records only for flows that mutate state.
- Run fast, branch-focused automation first: organization authorization, access service, application pages/write, user/company write, user pages, and enquiry review suites.
- Run the manual browser matrix below on carebridge.
- Run the full repo-required gates before merge:

```sh
./vendor/bin/sail exec web yarn fix
./vendor/bin/sail exec web ./vendor/bin/php-cs-fixer fix
./vendor/bin/sail exec web composer lint
./vendor/bin/sail exec web yarn ci
./vendor/bin/sail exec web yarn lint-styles
./vendor/bin/sail exec web yarn test
./vendor/bin/sail exec web composer test
```

- If a separate bedsearch environment is already practical, run one conditional cross-portal smoke only: one hospital-user-free happy path and one provider-user-free happy path. Do not repeat the full matrix there.

Manual Matrix

- [x] Hospital User Free:
	- [x] own-company application list/view/create
	- [x] map access with scoped application context
	- [x] request review on an unsuccessful enquiry
	- [x] one explicit denial when opening another company's application
- [x] Provider User Free: 
	- [x] assigned-facility application access, 
	- [x] application file download, 
	- [x] enquiry view/transition on the assigned facility, 
	- [x] and one explicit denial through an unassigned or foreign-company facility.
- [x] Hospital Administrator: 
	- [x] same-company visibility without needing facility assignment, 
	- [x] one admin-managed user or company update inside company scope, 
	- [x] and one denial for updating a target outside company scope.
- [ ] Provider Administrator: 
	- [ ] company-wide facility and enquiry access without explicit per-facility assignment, - THIS ISN'T WORKING!!! 
	- [ ] one valid enquiry transition, 
	- [ ] one valid vacancy2 delete inside company scope, 
	- [ ] and one denial outside company scope.
- [ ] Hospital User Paid: 
	- [ ] paid-only premium fields are visible/editable on an accessible application and persist correctly.
- [ ] Provider User Paid: 
	- [ ] paid-only premium fields are visible on an accessible application; 
		- [ ] do not duplicate full provider authorization coverage here.
- [ ] Facility Manager: smoke only. 
	- [ ] Verify assigned facility detail and vacancy delete succeed, 
	- [ ] and an out-of-assignment facility/vacancy is forbidden.
- [ ] Account Manager: smoke only. 
	- [ ] Verify company, user, and facility pages remain accessible outside the manager's own company. 
		- [ ] Do not use this role to prove application permissions.
- [ ] System Administrator: smoke only. 
	- [ ] Verify out-of-organisation application/map access and vacancy refresh/global admin behavior.
- [ ] Admin mutation: 
	- [ ] run one focused mutation scenario that changes a user/company relationship or company plan, 
	- [ ] then confirm direct `company_id` behavior, facility assignments still make sense, and the resulting role/scope behavior refreshes as expected.
- [ ] Review email side effect: 
	- [ ] cover one enquiry review path end-to-end 
	- [ ] and confirm the expected recipient/routing once in Mailpit at http://localhost:8027.

Test Scenarios and Evidence

- For each actor above, record one happy path, one boundary/denial where applicable, and the disposable record used so stateful scenarios stay isolated.
- For each denial, prefer a direct URL or submit action once per rule instead of multiplying 403 checks across equivalent roles.
- For paid-plan checks, record only premium-field behavior differences; do not repeat baseline list/view/scope checks already proven by the free-plan role.

Assumptions

- Primary manual validation is carebridge; bedsearch is conditional, not required, unless a second flavor env is already easy to run.
- Seeded personas exist from local seed data; if `LOCAL_TEST_DATA_PASSWORD` is not set, use the default password `password`.
- This plan is intentionally workflow-heavy on applications, enquiries, files, and map access, with company/user/facility/vacancy pages kept to targeted access smokes unless a failure suggests deeper follow-up.
