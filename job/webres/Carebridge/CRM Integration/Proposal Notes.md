# CRM Integration Service Proposal

## Summary

Introduce a shared integration service between Schedule Mee, Carebridge and external CRM platforms such as HubSpot, Resident Select and SugarCRM.

Rather than building separate CRM integrations directly into each internal application, the service would centralise common integration responsibilities and provide a reusable foundation for additional applications and CRM platforms.

The recommended approach is to validate one low-risk, end-to-end integration before committing to the complete data scope. CRM schemas, licences and rules for storing health information vary between providers, so field mapping and data-governance checks must be treated as onboarding requirements rather than implementation details.

## Proposed Architecture

![[Drawing 2026-08-07 11.17.12.excalidraw]]

### Application Connectors

Provide a standard interface between each internal application and the integration service.

Responsibilities:

- Communicating with the application API.
- Authenticating with the application.
- Receiving application events and changes.
- Retrieving the application data required for synchronisation.
- Translating application-specific data into standard integration contracts.
- Translating inbound integration requests into application-specific operations.
- Exposing application capabilities and limitations.

Each internal application would have its own connector. Application-specific business rules should remain within Carebridge and Schedule Mee rather than being moved into the integration service.

### Integration and Synchronisation Core

Acts as the central orchestration layer.

Responsibilities:

- Routing information between systems.
- Queueing and asynchronous processing.
- Storing cross-system record identifiers and associations.
- Storing account-specific object, field and value mappings.
- Tracking synchronisation state and checkpoints.
- Retry and failure handling.
- Idempotency and duplicate prevention.
- Applying agreed source-of-truth and conflict-resolution rules.
- Scheduled synchronisation and reconciliation.
- Audit logging, operational logging and monitoring.
- Detecting invalid or incompatible mappings.

The Core should manage integration behaviour without containing application-specific or CRM-specific business logic. Its standard contracts should cover the information that must be exchanged, but should not attempt to create a universal business model for every application and CRM.

### Web UI and Administration

Provide the configuration and operational interface required to onboard and support a connected provider.

Initial responsibilities:

- OAuth authorisation and connection management where supported.
- Credential setup status for integrations that do not use OAuth.
- Object, field, pipeline and value mapping configuration.
- Mapping validation and connection testing.
- Visibility of connection and synchronisation status.

Possible later responsibilities:

- Failed-record inspection and controlled retry.
- Synchronisation history and audit views.
- Provider-facing monitoring and support tools.

### CRM Adapters

Provide a standard interface between the integration service and each CRM platform.

Responsibilities:

- Communicating with the CRM API.
- CRM authentication, OAuth and token handling.
- Discovering the connected account's objects, fields, relationships, pipelines and supported capabilities.
- Translating standard integration contracts into CRM-specific objects and fields.
- Translating CRM webhooks and events into standard integration events.
- Handling CRM-specific pagination, rate limits, retries and errors.
- Validating configured mappings against the current CRM schema.
- Exposing platform, edition and subscription limitations.

Each supported CRM would have its own adapter.

## Data Mapping Approach

The integration cannot assume that every provider uses the same CRM objects, fields, pipelines or values. Mappings should be scoped to each connected CRM account or instance. They only need to differ by facility when facilities genuinely use different schemas or workflows within the same account.

The mapping model should support:

- **Object or module mappings**, such as a Carebridge enquiry to a HubSpot Deal or SugarCRM Case.
- **Field mappings**, such as `resident.first_name` to HubSpot `firstname` or SugarCRM `first_name`.
- **Value mappings**, such as Carebridge `accepted` to a provider-specific pipeline stage identifier.
- **Relationship mappings**, such as linking a resident to a facility, service or enquiry.
- **Direction and ownership rules**, including one-way fields and the authoritative source for bidirectional fields.
- **Transformations**, including date formats, identifiers, enumeration values and null handling.

Mappings should use stable internal API names rather than user-facing labels. The adapter should retrieve the connected CRM schema during onboarding, expose available fields and values for selection, and periodically validate that the configuration remains compatible.

Where commercially and operationally acceptable, the integration could create and manage a known set of Webres-owned CRM fields. This would reduce manual mapping but may duplicate fields that a provider already uses. The onboarding process should therefore support both approaches:

1. Map existing provider fields.
2. Create integration-managed fields where no appropriate field exists.

Derived analytics should normally be calculated from synchronised service or appointment records rather than repeatedly written to resident or contact fields. This includes service completion rates, outstanding services, companion hours and refusal trends.

## Initial CRM Findings

### HubSpot

HubSpot provides fixed internal names for standard properties and account-specific internal names for custom properties. The generic `properties` object in API responses is only a container; its keys are the stable internal property names used by integrations.

Relevant findings:

- First name and last name are standard Contact properties.
- Date of birth and gender are HubSpot-defined properties, but their availability and accepted values should be confirmed against the connected account.
- Room, wing, admission dates, discharge dates and most aged-care concepts will normally require custom properties or relationships.
- HubSpot provides standard Services, Appointments, Notes, Files, Deals and Tickets structures, but their availability and suitability depend on the account and agreed workflow.
- Pipeline stages, custom fields and enumeration values require account-specific mapping.
- Reporting values such as completion rate and refusal trends are derived metrics rather than standard fields.
- Sensitive or highly sensitive data requires suitable HubSpot licensing, configuration and OAuth scopes.
- Clinical documents require separate validation because general HubSpot file storage does not receive all Sensitive Data protections.

HubSpot is technically suitable for configurable integration, but it should not be assumed that providers share a common schema.

### SugarCRM

SugarCRM provides fixed internal names for stock fields. Custom fields created through Studio are instance-specific and conventionally end in `_c`. The adapter can retrieve module and field metadata from each connected instance.

Relevant findings:

- First name, last name and date of birth are stock Contact fields.
- Gender, room, wing, admission dates, discharge dates and most aged-care concepts require custom fields or relationships.
- Accounts, Meetings, Cases, Notes, Documents, Purchases and Purchased Line Items may provide useful standard structures, but several are only semantic approximations of the proposed aged-care workflows.
- Statuses, dropdown values, relationships and custom modules require instance-specific mapping.
- Reporting values should normally be derived from individual service records.
- Product, edition and licence differences may affect module availability.
- SugarCRM's current standard customer terms appear to restrict certain sensitive-information categories. Because Australian health information is sensitive information, contractual approval is required before sending clinical or patient data to SugarCRM.

The SugarCRM adapter is technically feasible, but health-data synchronisation should be treated as blocked until the provider's deployment, contract and permitted data classifications are confirmed in writing.

### Resident Select

Public API documentation has not yet been confirmed as readily available. Before estimating or committing to this adapter, obtain the vendor's current API documentation and confirm:

- Available APIs and supported objects.
- Authentication method and credential provisioning.
- Webhook or change-notification support.
- Rate limits and integration licensing.
- Test or sandbox access.
- Field and schema discovery capabilities.
- Permitted handling of personal, health and clinical information.

Resident Select should remain a discovery dependency rather than being assumed to offer capabilities equivalent to HubSpot or SugarCRM.

## Security and Data Governance

The proposed data includes personal information, health information and potentially clinical documents. CRM field availability alone does not establish that the data can be stored there.

Before enabling each data flow, confirm:

- The provider's CRM product, edition, deployment region and contract.
- Whether the CRM is approved to store the relevant data classification.
- Required consent, retention, deletion and audit requirements.
- Encryption and access-control requirements.
- Appropriate OAuth scopes or API-user permissions.
- Whether documents may be copied into the CRM or should remain in Carebridge.

The initial implementation should follow data minimisation. Until sensitive-data handling is approved, keep clinical summaries and documents in Carebridge and synchronise only the minimum workflow data, external identifiers and secure links required by the provider.

Credentials and tokens should be held in AWS Secrets Manager. Logs must exclude access tokens, clinical content and unnecessary personal information.

## Benefits and Risks

### Advantages

- Reduces duplicated CRM integration code across Carebridge and Schedule Mee.
- Keeps CRM-specific behaviour out of the internal applications.
- Makes additional CRM integrations easier to add.
- Centralises retries, logging, monitoring and record mappings.
- Improves reliability through asynchronous processing.
- Provides a single place to troubleshoot integration failures.
- Separates internal application changes from CRM API changes.
- Provides a consistent onboarding and mapping process for providers.
### Disadvantages
- Introduces another production application to deploy, secure and maintain.
- Creates additional architectural and operational complexity.
- Requires configuration and support processes for provider-specific mappings.
- Becomes shared infrastructure whose failures may affect multiple applications or providers.
### Caveats
- The integration service should remain focused on integration responsibilities.
- Business rules should remain within Carebridge and Schedule Mee.
- CRM-specific behaviour should remain within CRM Adapters.
- Avoid creating a universal business model covering every application and CRM.
- Do not assume that matching display labels have equivalent meaning.
- Do not enable sensitive-data flows based only on technical API capability.
- A new CRM adapter will still require discovery, mapping, testing and operational support.

## Suggested Stack

- **Rust**
	- Dioxus
	- Axum
		- Axum::routing
		- Axum OpenAPI 3?
	 - SQLX
	 - Tokyo 
- AWS
	- **ECS**
	- **RDS**
	- **Secrets Manager**
	- **CloudWatch**

## Suggested Delivery

1. **Complete feasibility and data-governance discovery**
   - Obtain Resident Select API documentation and test access.
   - Confirm HubSpot and SugarCRM licences, deployment details and permitted data classifications for the pilot provider.
   - Agree which information may be copied into each CRM.
2. **Define the initial integration contracts and mapping model**
   - Select the first application, CRM and workflow.
   - Define record ownership, identifiers, conflict rules and failure behaviour.
   - Define object, field, relationship and value mappings.
3. **Build the Integration and Synchronisation Core interfaces**
   - Implement configuration, queueing, state tracking, idempotency, retry and audit foundations.
4. **Implement one Application Connector and one CRM Adapter end-to-end**
   - Start with a low-risk data subset and one-way synchronisation where practical.
5. **Validate the pilot**
   - Test record matching, duplicate prevention, mapping changes, failures, reconciliation and provider onboarding.
6. **Add bidirectional synchronisation**
   - Introduce inbound events, source-of-truth rules and conflict handling after the one-way flow is stable.
7. **Expand the approved data scope**
   - Add sensitive or document flows only after contractual, security and privacy approval.
8. **Add additional CRM Adapters and the second internal application**
   - Reuse the proven contracts and onboarding process without assuming identical provider schemas.
9. **Add operational and provider-facing tooling as required**

## Decisions and Information Required from Lisa and Rob

- Which provider and workflow should be used for the first end-to-end pilot?
- Where is Schedule Mee hosted, and what APIs, events or database interfaces are available?
- Can API documentation, sandbox access and vendor contacts be obtained for Resident Select?
- Which CRM products, editions and deployment models are used by the initial providers?
- Should onboarding map existing provider fields, create Webres-managed fields, or support both approaches?
- Who will approve provider-specific field, pipeline and value mappings?
- Which system is authoritative for each bidirectional field?
- Are clinical summaries and documents required in the CRM, or would identifiers and secure Carebridge links meet the business need?
- Has each CRM been contractually and technically approved to store the proposed personal and health information?
- What provider onboarding process will be used to authorise OAuth connections or securely provision API credentials?

## Recommendation

Proceed with the shared integration-service architecture, but treat CRM onboarding, schema mapping and sensitive-data approval as first-class parts of the product.

The first milestone should be a deliberately narrow pilot using one internal application, one CRM, one provider account and a non-clinical data flow. This will validate the architecture and operational model before Webres commits to bidirectional synchronisation, clinical documents or multiple CRM adapters.
