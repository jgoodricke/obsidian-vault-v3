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
- Logging, monitoring.

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

## Benefits and Risks
### Advantages
- Reduces duplicated CRM integration code across Carebridge and Schedule Mee.
- Keeps CRM-specific behaviour out of the internal applications.
- Makes future CRM integrations easier to add.
- Centralises retries, logging, monitoring and record mappings.
- Improves reliability through asynchronous processing.
- Provides a single place to troubleshoot integration failures.
- Separates internal application changes from CRM API changes.
### Disadvantages
- Introduces another production application to deploy and maintain.
- Creates additional architectural complexity.
### Caveats
- The integration service should remain focused on integration responsibilities.
- Business rules should remain within Carebridge and Schedule Mee.
- CRM-specific behaviour should remain within CRM Adapters.
- Avoid attempting to create a universal business model covering all applications and CRMs.

## Suggested Stack
- Rust
	- TODO: Look into server frameworks and crates.
- RDS for configuration, record mappings and sync state 
	- Use existing Carebridge Instance.
- AWS Secrets Manager for CRM credentials.
- AWS Cloudwatch for logging/monitoring services? Or Build a bespoke UI?

## Suggested Delivery

1. Build the Integration / Sync Core and interfaces.
2. Implement one Application Connector and one CRM Adapter end-to-end.
3. Validate bidirectional synchronisation and record mappings.
4. Add additional CRM Adapters.
5. Add the second internal application.
6. Add operational/admin tooling as required.

## Questions for Lisa/Rob
- Can we get access to the ScheduleMee codebase to have a look?
- Where is Schedule Mee hosted?


### ToDo
- Data Matching
