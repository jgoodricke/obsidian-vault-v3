# Carebridge–Resident Select Integration Proposal

## Approval requested

Approve the design and delivery of an Initial Production Release of the Webres Integration Platform.

The release will be a production application that clients can use. It will connect one Webres Internal Application, Carebridge, to one Connected Platform, Resident Select. Initial delivery and acceptance will cover the first Carebridge Provider Company, one Resident Select Organisation and its agreed Sites.

The service will be designed so that later projects can add other Internal Applications and Connected Platforms. ScheduleMee, HubSpot, SugarCRM and additional Resident Select Connections are not part of this release.

## Business outcome

The release will reduce duplicate referral administration between Carebridge and Resident Select.

For a mapped Provider and Site, it will:

- Read existing active and newly submitted Carebridge Applications and Enquiries.
- Find and manually link an existing Resident Select Client where one already represents the person.
- Create a Resident Select Prospect from a Carebridge Enquiry after an authorised review.
- Keep approved, non-empty Carebridge demographic values current in Resident Select.
- Read Resident Select workflow progress.
- Propose and apply legal Carebridge Enquiry transitions with approval and audit controls.
- Give authorised users the tools to configure, operate and support the Connection.

## Proposed solution

Build a standalone, multi-tenant Integration Platform rather than embedding Resident Select-specific orchestration in Carebridge.

The platform will contain:

- **Carebridge Application Connector** — service-authenticated resource APIs and domain transition commands that preserve Carebridge rules and side effects.
- **Resident Select Platform Adapter** — API authentication, polling, pagination, lookups, Prospect creation and supported Client updates.
- **Integration Core** — Connections, mappings, Record Links, workflow evidence, Proposed Actions, checkpoints, retries, reconciliation and audit records.
- **Administration application** — invitation-only user access for configuration, approval and operational support.
- **Carebridge–Resident Select Integration Flow** — versioned rules for the records, fields and workflow states in this release.

The Core will communicate with Carebridge only through its API. It will not access the Carebridge MySQL database.

## Production scope

### Connection boundary

Initial delivery includes:

- One Carebridge Provider Company.
- One Resident Select Organisation.
- An agreed set of Carebridge Facility to Resident Select Site mappings.
- Existing active Carebridge Applications and Enquiries from an agreed cutoff date.
- New and changed in-scope records after the Connection is enabled.
- Live-data onboarding, reconciliation and validation for the first Connection.

The release does not include all historical records, bulk data cleansing or onboarding additional Providers.

### Carebridge to Resident Select

A Carebridge Enquiry for a mapped Facility can create a Resident Select Client as a Prospect at the mapped Site.

Production acceptance requires these core fields:

- First name.
- Last name.
- Mapped Site.
- Prospect date.

The estimate will conservatively allow for these optional deterministic fields:

- Date of birth.
- Gender.
- Street address, suburb and postcode.
- State.
- Financial status.
- Care timeframe.

An optional field can be deferred if Resident Select does not support it reliably or if it would add disproportionate cost without blocking the main client workflow. Clinical summaries, contacts, documents and free-text notes are excluded.

Before creating a Prospect, the platform will search for possible existing Clients and require an authorised user to confirm a link or approve creation. It will not match people automatically. Resident Select `external_id` will not be read, written or reserved.

### Resident Select to Carebridge

The platform will read Site lifecycle, Clinical Review and Contract evidence and use it to propose supported Carebridge Enquiry states, including:

- Clinical Review.
- Clinically Approved.
- Waitlisted.
- Bed Accepted.
- Contract Complete.
- Placed.
- Unsuccessful or No Longer Required where an approved mapping provides the required evidence.

Carebridge remains responsible for deciding whether a transition is legal. The platform will not write raw status identifiers or manufacture intermediate history.

Placed, Unsuccessful and No Longer Required transitions always require approval. Other safe mappings can progress from observation to approval and then to automatic processing after validation.

## Administration scope

The production application includes:

- Connection setup, lifecycle and health.
- Write-only credential entry, replacement and testing.
- Facility-to-Site and bounded value mappings.
- Invitation-only local accounts and tenant-restricted roles.
- Existing Resident Select Client review and manual linking.
- Proposed Action review, approval, rejection and supersession.
- Conflict and failure inspection.
- Explicit safe retry.
- Resolution of creates whose outcome is unknown.
- Redacted synchronisation history and audit views.
- Non-personal operational notifications that link to the authenticated application.

Raw Resident Select payloads will not be stored or displayed.

## Carebridge work

Carebridge needs a new generic machine API. This is a substantial workstream in the delivery estimate and includes:

- Scoped Application and Enquiry read projections.
- Stable incremental polling contracts and reconciliation support.
- A non-interactive Integration Principal authenticated with Laravel Sanctum.
- Policy enforcement for the Provider Company and mapped Facilities.
- Enquiry transition commands that use existing Carebridge business rules.
- Idempotency and optimistic concurrency.
- Native histories, notifications, required data and the Placed-Elsewhere Cascade.
- Automated API and domain-side-effect tests.

## Reliability and data handling

The platform will:

- Use scheduled polling because Resident Select has no documented webhooks.
- Store durable work, checkpoints and retries in PostgreSQL.
- Reconcile periodically to detect missed changes.
- Prevent concurrent workers from processing the same Connection.
- Stop automatic calls when credentials are invalid.
- Require manual resolution when Resident Select may have accepted a create request but its response was lost.
- Keep stable identifiers, mappings, workflow projections, audit events and redacted failures.
- Never persist raw Connected Platform payloads or transient demographics for unlinked Clients.
- Never propagate deletion between Carebridge and Resident Select.

## Deployment

Deploy one Rust and Dioxus application image to AWS ECS Fargate with:

- A dedicated PostgreSQL RDS instance.
- AWS Secrets Manager for Connection and Carebridge credentials.
- Redacted CloudWatch logs, metrics and alerts.
- An Application Load Balancer with TLS for production.
- Static outbound network access where Resident Select requires IP allowlisting.
- Separate staging and production storage, secrets, credentials and administration accounts.

Staging will remain non-public and will not use the production load balancer. Deployment automation and both environments are part of the delivery. Recurring AWS and Resident Select costs will be shown separately from development labour.

## Delivery assumptions

The estimate will assume:

- One initial Provider Connection and its agreed Sites.
- Existing active Carebridge records from an agreed cutoff date.
- No bulk migration of all historical records.
- No large-scale manual data-cleansing service.
- A production release and first live Connection, not a prototype.
- Conservative contingency for Resident Select API and test-environment unknowns.
- Optional fields can be deferred without blocking acceptance when vendor behaviour or cost is disproportionate.

Resident Select test access, polling behaviour, lookup values and rate limits remain vendor dependencies. They will be resolved during delivery and included as estimate uncertainty rather than placed in a separate quoted discovery release.

## Exclusions

The Initial Production Release excludes:

- ScheduleMee, HubSpot, SugarCRM and other Connected Platforms.
- Additional live Provider onboarding.
- Creation of Carebridge Applications or Enquiries from Resident Select-only Clients.
- Resident Select lifecycle, room, Clinical Review, Contract, Activity or document writes.
- Client Contact creation or linking.
- Clinical content, documents, representative details and free-text notes.
- Bulk historical migration and data cleansing.
- Arbitrary field expressions or a general-purpose mapping language.
- SQS, EventBridge and a Carebridge transactional outbox.

## Main delivery risks

- Resident Select test Organisation and credential availability.
- Detection of nested Site Association changes.
- Pagination consistency while Resident Select records change.
- Resident Select lookup values and undocumented validation behaviour.
- Resident Select Client creation has no documented idempotency mechanism.
- Carebridge requires a new generic machine API before the complete Flow can operate.

The estimate will carry conservative contingency for these risks. Unsupported optional fields can be deferred, but the core Prospect and workflow path must work before production acceptance.

## Acceptance outcome

The release is complete when the first Provider can use the production application to configure its Connection, reconcile existing active Carebridge records, link or create Resident Select Clients, observe Resident Select workflow progress, apply controlled Carebridge transitions, inspect failures and audit the resulting activity.

## Recommendation

Proceed with the Initial Production Release for Carebridge and Resident Select. Keep the first release narrow in participating systems and data fields, but deliver the administration, reliability and deployment controls required for real client use.
