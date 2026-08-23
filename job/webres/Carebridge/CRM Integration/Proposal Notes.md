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
- Discovering the connected account's objects, fields, relationships, pipelines and supported capabilities where the platform allows it.
- Retrieving organisation-specific lookup values and capability limits for fixed-schema platforms.
- Translating standard integration contracts into CRM-specific objects and fields.
- Translating CRM webhooks, events or incrementally polled changes into standard integration events.
- Handling CRM-specific pagination, polling, rate limits, retries and errors.
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

Mappings should use stable internal API names rather than user-facing labels. For configurable CRMs such as HubSpot and SugarCRM, the adapter should retrieve the connected schema during onboarding, expose available fields and values for selection, and periodically validate that the configuration remains compatible. Resident Select is different: its schema and attribute names are fixed, but each Organisation's lookup IDs and supported operations must still be discovered and cached.

Where commercially and operationally acceptable, the HubSpot and SugarCRM integrations could create and manage a known set of Webres-owned CRM fields. This would reduce manual mapping but may duplicate fields that a provider already uses. Their onboarding process should therefore support both approaches:

1. Map existing provider fields.
2. Create integration-managed fields where no appropriate field exists.

Resident Select has no documented custom-field mechanism. If a canonical concept has no Resident Select attribute, the mapping must omit it, retain it in Schedule Mee or Carebridge, or use an explicitly agreed derivation; onboarding cannot create a field to close the gap.

Derived analytics should normally be calculated from synchronised service or appointment records rather than repeatedly written to resident or contact fields. This includes service completion rates, outstanding services, companion hours and refusal trends. For Resident Select, these calculations must remain in Schedule Mee because Resident Select has no service-occurrence record set to synchronise or count.

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

Resident Select publishes a documented REST API at `https://app.residentselect.com.au/api/v1`. It is a fixed-schema, aged-care-specific platform rather than a configurable general CRM. There is no documented custom-field mechanism, so unsupported canonical fields cannot be added during onboarding.

Relevant findings:

- Authentication uses an API key and secret over HTTP Basic authentication. Credentials are scoped to one Organisation, can have separate read, write and delete permissions, and may be IP allowlisted.
- The API documents no webhooks, callbacks or event subscriptions. Changes must be detected by polling supported `updated_start_date` and `updated_end_date` filters and maintaining per-resource checkpoints. Rate limits remain undocumented.
- Almost the entire API is read-only. The only documented writes are `POST /clients`, `PATCH /clients/[id]` and `POST /contacts`.
- Creating a Client always creates a **Prospect** against exactly one Site. Lifecycle status, archive reason, room, referral progression, documents, activities and clinical reviews are read-only.
- A resident is a Client. Facility, room and lifecycle state are held in `person_sites[]` per Site, so the adapter must select the correct `person_site` by `site_id`; it cannot assume one room or status per person.
- Resident Select has no service-enrolment object and no writable record for a scheduled or delivered service. Schedule Mee service history, per-service enrolment and derived service analytics cannot be synchronised into Resident Select and must remain authoritative in Schedule Mee.
- Carebridge can create a new referral only as a Prospect. Resident Select then becomes authoritative for enquiry status and admission outcome, which the integration can poll and return to Carebridge. Status updates, durable document links, document uploads and messages cannot be written through the documented API.
- `external_id` is writable and filterable on Clients and is the best available idempotency key, but it is a single shared slot that may already belong to another integration. The Integration Core must always store Resident Select record IDs and must not match people by name.
- Site, status, archive-reason, gender and other lookup IDs must be retrieved per Organisation rather than hard-coded. The published examples are incomplete, and no endpoints are documented for the referenced State and Pension Status lookups.
- Resident Select is designed to hold aged-care and health information. The primary data-governance constraint is therefore data minimisation: read and retain only the attributes approved for the integration rather than ingesting complete Client records.

The adapter is feasible for resident and referral reads and for constrained Prospect creation. It is not capability-equivalent to HubSpot or SugarCRM, and its unsupported write directions must be visible in configuration rather than treated as mapping errors.

## Security and Data Governance

The proposed data includes personal information, health information and potentially clinical documents. CRM field availability alone does not establish that the data can be stored there.

Before enabling each data flow, confirm:

- The provider's CRM product, edition, deployment region and contract.
- Whether the CRM is approved to store the relevant data classification.
- Required consent, retention, deletion and audit requirements.
- Encryption and access-control requirements.
- Appropriate OAuth scopes or API-user permissions.
- Whether documents may be copied into the CRM or should remain in Carebridge.
- For Resident Select, the minimum attributes Webres is permitted to read and retain, whether `external_id` is already owned by another integration, and which Organisation and Sites the credential may access.

The initial implementation should follow data minimisation. Until sensitive-data handling is approved, keep clinical summaries and documents in Carebridge and synchronise only the minimum workflow data, external identifiers and secure links required by the provider. Resident Select document URLs are short-lived presigned links, not durable links, and its API has no documented document-upload operation.

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
    - `axum::routing`
  - SQLx
  - Rayon / Tokio
- AWS
	- **ECS**
	- **RDS**
	- **Secrets Manager**
	- **CloudWatch**

### Suggested Architecture

- Use a layered architecture:
  - Adapters for connecting to Carebridge and Schedule Mee.
  - An Integration and Synchronisation Core for routing, mappings, checkpoints, identifiers, retries and reconciliation.
  - An adapter for each CRM, containing its API-specific logic and loading the provider configuration from the integration database.
- Include a basic frontend where:
  - Carebridge administrators can see synchronisation status.
  - CRM administrators can configure a connection using OAuth or API credentials, depending on the platform.

The initial system should use scheduled incremental synchronisation rather than depend on webhooks. Each connector should request only records changed since its last successful checkpoint, with periodic full reconciliation to detect missed changes. For Resident Select this is mandatory because no webhook mechanism is documented; polling must use the resources that expose `updated_start_date` and `updated_end_date`, respect the maximum page size of 500, and use overlap or another safe boundary strategy so records updated at a checkpoint are not skipped. Polling frequency cannot be finalised until the vendor confirms rate limits.

## Suggested Delivery

1. **Complete feasibility and data-governance discovery**
   - Obtain Resident Select test credentials and confirm credential provisioning, expiry or rotation, rate limits, test-environment availability and the undocumented State and Pension Status lookups.
   - Confirm whether the pilot provider already uses Resident Select `external_id` and identify the Organisation, Sites and lookup values in scope.
   - Confirm HubSpot and SugarCRM licences, deployment details and permitted data classifications for later providers.
   - Agree which information may be copied into each CRM and, for Resident Select, the minimum data Webres may read.
2. **Define the initial integration contracts and mapping model**
   - Select the first application, CRM and workflow.
   - Define record ownership, identifiers, conflict rules and failure behaviour.
   - Define object, field, relationship and value mappings.
3. **Build the Integration and Synchronisation Core interfaces**
   - Implement configuration, queueing, state tracking, idempotency, retry and audit foundations.
4. **Implement the Schedule Mee Connector and Resident Select Adapter end-to-end**
   - Start with a low-risk, one-way Resident Select to Schedule Mee resident flow, such as identity, Site and room data.
   - Store Client and `person_site` identifiers in the Core and implement checkpointed polling and reconciliation.
5. **Validate the pilot**
   - Test record identity, multi-Site selection, duplicate prevention, lookup changes, polling boundaries, failures, reconciliation and provider onboarding.
6. **Add supported reverse or bidirectional flows**
   - Add Prospect creation or Carebridge status ingestion only where Resident Select exposes the required operation. Do not plan Schedule Mee service-history writes or Resident Select lifecycle-status writes against the current API.
7. **Expand the approved data scope**
   - Add sensitive reads only after security and privacy approval. Resident Select documents and clinical records remain read-only and should be excluded unless specifically required.
8. **Add additional CRM Adapters and the second internal application**
   - Reuse the proven contracts and onboarding process without assuming identical provider schemas or capabilities.
9. **Add operational and provider-facing tooling as required**

## Recommendation

Proceed with the shared integration-service architecture, but treat CRM onboarding, schema mapping and sensitive-data approval as first-class parts of the product.

The first milestone should be a deliberately narrow pilot using Schedule Mee, Resident Select, one provider Organisation and a non-clinical, one-way Resident Select to Schedule Mee resident flow. This validates fixed-schema mapping, per-Site identity, incremental polling and reconciliation without depending on API operations Resident Select does not provide.
