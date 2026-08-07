## Proposal

Introduce a shared integration service between Schedule Mee/Carebridge and external CRM platforms such as HubSpot, Resident Select and SugarCRM.

Rather than building separate CRM integrations directly into each application, the integration service would centralise common integration responsibilities and provide a reusable foundation for future CRM platforms.

## Proposed Architecture
![[Drawing 2026-08-07 11.17.12.excalidraw]]

### Application Connectors
Provide a standard interface between internal applications and the integration service.

Responsibilities:
- Application API communication.
- Application authentication?
- Receiving application events/changes.
- Retrieving required application data.
- Translating application-specific data into standard integration formats.
- Translating inbound integration requests into application-specific operations.

### Integration / Sync Core

Acts as the central orchestration layer.

Responsibilities:
- Routing information between systems.
- Queueing and asynchronous processing.
- Record mapping between internal and external systems.
- Tracking synchronisation state.
- Retry and failure handling.
- Idempotency and duplicate prevention.
- Source-of-truth and conflict rules.
- Scheduled synchronisation and reconciliation.
- Integration configuration.
- Logging, monitoring and audit history.

The Core should manage integration behaviour without containing application-specific or CRM-specific business logic.

### CRM Adapters

Provide a standard interface between the integration service and individual CRM platforms.

Responsibilities:

- CRM API communication.
- CRM authentication and token handling.
- Translating integration data into CRM-specific objects and fields.
- Translating CRM webhooks/events into standard integration events.
- Handling CRM-specific API behaviour, rate limits and errors.
- Exposing platform capabilities and limitations.

Each supported CRM would have its own adapter.
## Key Benefits
- Reduces duplicated CRM integration code across Carebridge and Schedule Mee.
- Keeps CRM-specific behaviour out of the internal applications.
- Makes future CRM integrations easier to add.
- Centralises retries, logging, monitoring and record mappings.
- Improves reliability through asynchronous processing.
- Provides a single place to troubleshoot integration failures.
- Separates internal application changes from CRM API changes.
## Disadvantages / Risks
- Introduces another production application to deploy and maintain.
- Creates additional architectural complexity.
- Requires good monitoring and correlation between systems.
## Important Boundary
The integration service should remain focused on integration responsibilities.

Business rules should remain within Carebridge and Schedule Mee.

CRM-specific behaviour should remain within CRM Adapters.

Avoid attempting to create a universal business model covering all applications and CRMs.

## Suggested Implementation

Start with a single modular application rather than multiple microservices.

Potential stack:
- Rust.
- RDS for configuration, record mappings and sync state.
- AWS Secrets Manager for CRM credentials.
- AWS logging/monitoring services.

## Suggested Delivery

1. Build the Integration / Sync Core and interfaces.
2. Implement one Application Connector and one CRM Adapter end-to-end.
3. Validate bidirectional synchronisation and record mappings.
4. Add the second internal application.
5. Add additional CRM Adapters.
6. Add operational/admin tooling as required.

## Recommendation

Use the shared integration service approach.

There are already two internal applications and three initial CRM platforms, with additional CRMs expected in future.

Without the integration layer, each application/CRM combination could require its own integration implementation.

The proposed Connector → Core → Adapter model provides a more maintainable and scalable approach while keeping application and CRM concerns separated.


## Questions for Lisa
- Can we get access to the ScheduleMee codebase to have a look?
- Where is Schedule Mee hosted?

## Questions for Jordan
- How should the DB work in this case?
