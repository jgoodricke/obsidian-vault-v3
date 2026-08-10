## Purpose

This document records how the proposed Schedule Mee and Carebridge integration data could be represented in HubSpot and SugarCRM. It identifies:

- Fields that have a direct stock CRM equivalent.
- Data that can use a standard CRM object or module but still requires configuration.
- Bespoke fields, relationships, statuses and value mappings required for each connected provider.
- Reporting values that should be derived from underlying records rather than stored as fields.
- Sensitive-data and product constraints that must be resolved before implementation.

This is a proposed integration model, not a final provider configuration. The actual objects, fields, permissions and values must be discovered and confirmed against each connected HubSpot account or SugarCRM instance during onboarding.

## Key Conclusions

- Only a small number of resident fields have direct, stable equivalents in both CRMs. First name, last name and date of birth are the main examples.
- Most aged-care, admission, referral and clinical concepts are not native CRM fields.
- HubSpot custom properties are account-specific. SugarCRM custom fields are instance-specific and normally end in `_c`.
- Standard objects and modules do not eliminate mapping. Pipelines, stages, dropdown values, relationships and semantic use still need to be agreed per connected account or instance.
- Provider-specific mapping should be scoped per CRM account or instance, not automatically per facility. Facilities can reuse a mapping where they share the same schema and workflow.
- Service analytics are derived metrics. They should normally be calculated from appointment or meeting records rather than copied into repeatedly overwritten Contact fields.
- Clinical summaries, health-related workflow data and documents require separate privacy, contractual and product approval. Technical API support alone is insufficient.

## Classification

| Classification | Meaning |
|---|---|
| **Stock field** | A suitable CRM field exists with a platform-defined internal API name. |
| **Standard structure** | A suitable stock object, module, activity or relationship exists, but configuration or value mapping is still required. |
| **Conditional stock field** | HubSpot defines the field, but availability or suitability must be checked in the connected account. |
| **Bespoke** | A custom property, field, relationship, module, pipeline value or interpretation is required. |
| **Derived** | The value should be calculated from underlying records rather than treated as an individual CRM field. |
| **Approval required** | The technical mapping is possible, but privacy, contractual, licensing or security approval is unresolved. |

## Mapping Conventions

The tables use proposed canonical integration keys such as `resident.first_name`. These keys belong to the integration contract and should remain independent of either CRM.

CRM mappings must use internal API identifiers, not display labels:

| CRM | Stable identifiers | Provider-specific identifiers |
|---|---|---|
| HubSpot | HubSpot-defined object and property names such as `contacts`, `firstname` and `hs_appointment_start` | Custom property names, pipeline IDs, stage IDs, custom association labels and enumeration values |
| SugarCRM | Stock module and field names such as `Contacts`, `first_name`, `Meetings` and `date_start` | Custom fields ending in `_c`, package-prefixed custom modules, relationship link names and dropdown item names |

Example custom names in this document, such as `webres_room_number` or `room_number_c`, are illustrative only. They must not be assumed to exist in a provider's CRM.

## CRM to Schedule Mee

### Resident Details

| Integration field | Canonical key | HubSpot mapping | HubSpot classification | SugarCRM mapping | SugarCRM classification | Provider-specific work |
|---|---|---|---|---|---|---|
| First name | `resident.first_name` | Contact `firstname` | **Stock field** | Contacts `first_name` | **Stock field** | Confirm that the provider has not chosen a separate custom identity field. |
| Last name | `resident.last_name` | Contact `lastname` | **Stock field** | Contacts `last_name` | **Stock field** | Confirm matching and duplicate rules. |
| Room number | `resident.room_number` | Custom Contact property, for example `webres_room_number` | **Bespoke** | Custom Contacts field, for example `room_number_c` | **Bespoke** | Select or create the property and confirm whether room numbers are unique only within a facility. |
| Facility | `resident.facility_id` | Associate the Contact with a Company representing the facility, or use a custom property/association | **Standard structure or bespoke** | Relate the Contact to an Account representing the facility, or use a custom Facility module/relationship | **Standard structure or bespoke** | Confirm what Company/Account represents in the provider's existing CRM. Map by a stable facility identifier rather than facility name where possible. |
| Wing | `resident.wing_id` | Custom Contact property or relationship to a custom facility/wing object | **Bespoke** | Custom Contacts field or relationship to a custom Wing/Facility module | **Bespoke** | Confirm whether a free-text value, controlled list or related record is required. |
| Date of birth | `resident.date_of_birth` | Contact `date_of_birth` where available | **Conditional stock field** | Contacts `birthdate` | **Stock field** | Verify the HubSpot property through the connected account's schema. Confirm date-only handling and whether a custom sensitive property is required. |
| Gender | `resident.gender` | Contact `gender` where available | **Conditional stock field** | Custom Contacts dropdown, for example `gender_c` | **Bespoke** | Map internal enumeration values. Confirm terminology, allowed values, null handling and sensitive-data treatment. |
| Admission date | `resident.admission_date` | Custom date property | **Bespoke** | Custom date field, for example `admission_date_c` | **Bespoke** | Do not reuse generic CRM close-date fields because their meaning differs. Confirm the authoritative system. |
| Expected discharge date | `resident.expected_discharge_date` | Custom date property | **Bespoke** | Custom date field, for example `expected_discharge_date_c` | **Bespoke** | Confirm whether this is planned, estimated or confirmed and how changes are audited. |
| About Me profile | `resident.about_me` | Custom Sensitive Data multiline property, or an associated Note if timeline history is required | **Bespoke** | A dedicated custom text-area field is recommended; Contacts `description` is technically available but too general | **Bespoke recommended** | Confirm maximum length, formatting, edit ownership, history requirements and whether the content is approved for the CRM. |

HubSpot's default Contact catalogue includes first name and last name and includes date of birth and gender among its Ads properties. The connected account's property schema remains authoritative. Sugar's stock Contacts fields include `first_name`, `last_name`, `birthdate` and `description`, but not the proposed accommodation and admission fields.

### Service Information

The preferred model is a resident Contact associated with a service-enrolment record. HubSpot's Services object is a close semantic match and explicitly supports offerings such as personal care, but it must first be activated in the account. SugarCRM's Purchases and Purchased Line Items can hold service start and end dates, although they are commercially oriented and may be an awkward fit for resident-level care services.

| Integration field | Canonical key | HubSpot mapping | HubSpot classification | SugarCRM mapping | SugarCRM classification | Provider-specific work |
|---|---|---|---|---|---|---|
| Service name | `service_enrolment.service_name` | Services `hs_name` | **Stock field in standard structure** | Purchased Line Items Name/Product, or a related Product Catalog record | **Standard structure** | Activate/confirm the module, associate the service with the resident, and map the provider's service catalogue to Schedule Mee service identifiers. |
| Service frequency | `service_enrolment.frequency` | Custom Service property | **Bespoke** | Custom Purchased Line Items field, for example `service_frequency_c` | **Bespoke** | Define whether frequency is structured, such as weekly with quantity, or free text. Do not reuse commerce billing-frequency fields. |
| Commencement date | `service_enrolment.commencement_date` | Custom Service date property | **Bespoke** | Purchased Line Items Start Date | **Stock field in standard structure** | Retrieve the Sugar internal field identifier from instance metadata. Confirm actual versus planned commencement. |
| End date | `service_enrolment.end_date` | Services `hs_target_end_date` for a planned end; use a custom field for an actual end date | **Stock field with semantic check** | Purchased Line Items End Date | **Stock field in standard structure** | Confirm whether the value represents a target, expected or actual end date. Retrieve the Sugar internal identifier from metadata. |

If a provider does not already use Sugar Purchases and Purchased Line Items for individual care services, a custom Service Enrolment module may be cleaner than repurposing a commercial purchase model.

## Schedule Mee to CRM

### Service History

One record should represent one scheduled or delivered service occurrence. The proposed baseline mappings are:

- HubSpot: an Appointment associated with the resident Contact and, where useful, the related Service.
- SugarCRM: a Meeting associated with the resident Contact and, where useful, the related service record.

| Integration field | Canonical key | HubSpot mapping | HubSpot classification | SugarCRM mapping | SugarCRM classification | Provider-specific work |
|---|---|---|---|---|---|---|
| Resident name | `service_visit.resident_id` | Associate Appointment to Contact; display name comes from `firstname` and `lastname` | **Standard structure** | Relate Meeting to Contact; display name comes from `first_name` and `last_name` | **Standard structure** | Store cross-system record IDs. Do not duplicate the resident's name on every service occurrence unless required for exports. |
| Service name | `service_visit.service_name` | Appointment `hs_appointment_name`, optionally associated with a Service | **Stock field in standard structure** | Meetings `name` (Subject), optionally related to a service record | **Stock field in standard structure** | Decide whether the name is a snapshot or derived from a related service catalogue record. |
| Service date | `service_visit.starts_at` | Appointment `hs_appointment_start` | **Stock field** | Meetings `date_start` | **Stock field** | Store a full timestamp and agree timezone conversion. |
| Service time | `service_visit.starts_at` and `service_visit.ends_at` | Appointment `hs_appointment_start` and `hs_appointment_end` | **Stock fields** | Meetings `date_start` and `date_end` | **Stock fields** | Normalise timestamps to UTC in the integration while preserving provider-local display behaviour. |
| Service status | `service_visit.status` | Appointment `hs_appointment_status` | **Stock field plus value mapping** | Meetings `status` | **Stock field plus value mapping** | Map canonical values to the account/instance's internal enumeration values. Missed, Refused and Incomplete need a custom outcome or reason. |
| Service notes | `service_visit.notes` | Associated Note `hs_note_body`, or a custom Appointment property if direct filtering is essential | **Standard activity or bespoke** | Meetings `description` (Summary), Internal Notes where exposed, or an associated Note | **Stock structure** | Confirm author, source, edit ownership, maximum length and whether notes contain health information. |

HubSpot Appointment records use the verified stock identifiers `hs_appointment_name`, `hs_appointment_start`, `hs_appointment_end` and `hs_appointment_status`. Sugar's Meetings REST API uses `name`, `description`, `date_start` and `date_end`; the instance's metadata must be used to confirm all available fields and internal dropdown values.

### Status and Outcome Mapping

The integration should use a canonical status model and map it to the connected CRM. Do not embed one provider's stage IDs or dropdown item names in application code.

| Canonical Schedule Mee status | HubSpot Appointment | SugarCRM Meeting | Mapping requirement |
|---|---|---|---|
| Scheduled | Native appointment status | Native Scheduled status | Map to the discovered internal value. |
| In progress | Native where enabled; otherwise custom | Custom status or outcome | Confirm whether the provider needs this intermediate state. |
| Completed | Native Completed status | Native Held status | Map Sugar Held to the canonical Completed meaning. |
| Cancelled | Native Cancelled status | Native Canceled status | Normalise Australian spelling only in the canonical model; retain CRM internal values. |
| Rescheduled | Native where enabled | Deferred may be usable, subject to provider meaning | Confirm whether a new service occurrence is also created and linked. |
| Missed | Custom outcome/reason | Custom outcome/reason | Do not overload Cancelled because reporting meaning differs. |
| Refused | Custom outcome/reason | Custom outcome/reason | Include an optional refusal reason if approved. |
| Incomplete | Custom outcome/reason | Custom outcome/reason | Define whether the service remains open or closes with an incomplete outcome. |

SugarCRM 26.1 documents the stock Meeting status labels Scheduled, Held, Canceled and Deferred. Their internal dropdown item names must still be retrieved and mapped per instance.

### Analytics and Reporting

These values do not have direct stock-field equivalents and should generally not be synchronised as Contact fields.

| Integration metric | Canonical key | HubSpot source | SugarCRM source | Classification | Mapping or calculation requirement |
|---|---|---|---|---|---|
| Services delivered during the current month | `analytics.services_delivered_current_month` | Completed Appointments in the reporting month | Held Meetings in the reporting month | **Derived** | Count by canonical completed outcome, using the provider's reporting timezone. |
| Missed, refused, incomplete or cancelled visits | `analytics.service_outcomes` | Appointments grouped by mapped status/outcome | Meetings grouped by mapped status/outcome | **Derived plus bespoke outcomes** | Requires the custom outcome mapping described above. |
| Companion hours | `analytics.companion_hours` | Sum Appointment duration for mapped companion services | Sum Meeting duration for mapped companion services | **Derived** | Requires a reliable service-category mapping and agreed duration rules. |
| Service completion rate | `analytics.service_completion_rate` | Completed Appointments divided by expected Appointments | Held Meetings divided by expected Meetings | **Derived** | Agree exclusions, date range and treatment of cancellations and reschedules. |
| Refusal trends | `analytics.refusal_trends` | Refused outcome records grouped over time | Refused outcome records grouped over time | **Derived plus bespoke outcome** | Confirm report dimensions such as facility, service, resident and reason. |
| Outstanding services | `analytics.outstanding_services` | Past-due Scheduled/In Progress Appointments | Past-due Scheduled/Deferred Meetings, subject to mapping | **Derived** | Define overdue thresholds and whether rescheduled records are excluded. |

If the CRM requires snapshot metrics for dashboards, the integration may write calculated values to dedicated custom fields. Those values must be treated as cached reporting outputs with an explicit calculation timestamp, not as source records.

## Carebridge to CRM

Carebridge enquiries and referrals do not have a universal direct equivalent in either CRM. The provider must first choose the remote object or module that represents the workflow.

| Integration information | Canonical key or contract | HubSpot mapping | SugarCRM mapping | Classification | Provider-specific work |
|---|---|---|---|---|---|
| New referrals | `referral.created` and referral record | Deal, Ticket or agreed custom object associated with the patient Contact and provider/facility Company | Case, Lead, Opportunity or custom Referral module associated with the patient Contact and provider/facility Account | **Standard structure with object/module mapping** | Agree the semantic model before mapping fields. Store Carebridge's immutable external identifier. |
| Referral updates | `referral.updated` | Deal stage, Ticket status or custom status property | Case Status, Lead Status, Opportunity Sales Stage or custom status field | **Standard field plus value mapping** | Map pipeline/stage/dropdown internal values and define source-of-truth rules. |
| Documents | `referral.document` | File associated through a Note using `hs_attachment_ids` | Document record or Note attachment associated with the referral record | **Standard capability; approval required** | Confirm allowed document types, size limits, retention, permissions and whether copying clinical documents is permitted. Prefer a secure Carebridge link if not approved. |
| Patient demographics | `resident.*` approved subset | Contact properties using the resident mappings above | Contacts fields using the resident mappings above | **Mixed** | Apply field-by-field mapping and data-minimisation rules. |
| Clinical summary | `referral.clinical_summary` | Custom Sensitive Data multiline property or another approved sensitive structure | Custom text field or Note only if contractually and legally approved | **Bespoke; approval required** | HubSpot requires suitable Sensitive Data configuration and scopes. SugarCRM is blocked pending written contractual confirmation. |
| Hospital messages or comments | `enquiry_message.created`, direction `hospital_to_provider` | Note `hs_note_body` associated with the enquiry/referral record | Associated Note or Case Comment Log | **Standard activity plus bespoke metadata** | Add or map immutable external message ID, source, direction, author and timestamp. Define update and deletion behaviour. |

### Suggested Referral Field Baseline

Once the provider selects a HubSpot object and Sugar module for referrals, the following logical fields will still be required:

| Canonical field | HubSpot target | SugarCRM target | Notes |
|---|---|---|---|
| `referral.external_id` | Dedicated unique custom property | Integration Sync ID where suitable, otherwise a dedicated custom field | Required for idempotent upsert and reconciliation. Do not match on name. |
| `referral.status` | Pipeline stage or custom enumeration | Module status/stage or custom dropdown | Requires bidirectional value mapping. |
| `referral.patient_id` | Association to Contact | Relationship to Contact | Store cross-system record IDs in the Integration Core. |
| `referral.provider_id` | Association to Company | Relationship to Account | Confirm whether the provider or facility is the organisational record. |
| `referral.facility_id` | Company association, custom association or property | Account relationship, custom relationship or field | May differ from provider organisation. |
| `referral.created_at` | Stock create timestamp for the CRM record plus optional source timestamp | Stock Date Created plus optional source timestamp | Preserve the Carebridge event timestamp where it can differ from CRM creation. |
| `referral.updated_at` | Stock modified timestamp plus source update timestamp if required | Stock Date Modified plus source update timestamp if required | CRM modification timestamps are not a substitute for application event time. |

## CRM to Carebridge

| Integration information | Canonical key or contract | HubSpot mapping | SugarCRM mapping | Classification | Provider-specific work |
|---|---|---|---|---|---|
| Enquiry status changes | `enquiry.status` | Deal stage, Ticket status or custom status property | Case Status or status/stage field of the selected referral module | **Standard field plus value mapping** | Map values, determine the authoritative source and prevent update loops. |
| Admission outcome | `enquiry.admission_outcome` | Custom enumeration or deliberately modelled pipeline stage | Custom dropdown or deliberately modelled Case/Lead/Opportunity status | **Bespoke or status mapping** | Agree outcome vocabulary and distinguish outcome from workflow status. |
| Date admitted | `enquiry.date_admitted` | Custom date property | Custom date field, for example `date_admitted_c` | **Bespoke** | Do not reuse HubSpot Contact Close date or a Sugar sales close date. |
| Information requests | `information_request.created` | Ticket, Task or Note depending on provider workflow | Case, Task or Note depending on provider workflow | **Standard structure with object mapping** | Define whether this is a separate actionable record or a message on the enquiry. Map ownership, due date and completion state if actionable. |
| Provider messages or comments | `enquiry_message.created`, direction `provider_to_hospital` | Associated Note `hs_note_body` | Associated Note or Case Comment Log | **Standard activity plus bespoke metadata** | Map source, direction, author, timestamp and external ID. Avoid treating arbitrary CRM notes as Carebridge messages without an integration marker. |

## CRM Schema Discovery

### HubSpot

For each connected account, the HubSpot adapter should retrieve available properties using the date-versioned Properties API:

```http
GET /crm/properties/2026-03/{objectType}
```

For example:

```http
GET /crm/properties/2026-03/contacts
GET /crm/properties/2026-03/appointments
GET /crm/properties/2026-03/services
```

The adapter should capture:

- Internal property `name` and user-facing `label`.
- Data type and field type.
- Whether the property is HubSpot-defined.
- Enumeration option internal values and labels.
- Uniqueness, read-only and archive state.
- Data-sensitivity classification.
- Available objects, associations, pipelines and stages.

For Sensitive Data properties, schema discovery and value access use different permissions. Sensitive or Highly Sensitive values require the relevant object scopes and an Enterprise account with Sensitive Data enabled.

### SugarCRM

For each connected instance, the Sugar adapter should retrieve the authenticated user's metadata using the instance's supported REST API version:

```http
GET /rest/v11_x/metadata
```

The adapter should capture:

- Available modules and module keys.
- Stock and custom field names, labels and data types.
- Read, write and field-level access for the integration user.
- Dropdown definitions, internal item names and labels.
- Relationship and link-field names.
- Custom module package prefixes.
- Product, edition and licence-dependent module availability.

Sugar custom fields created through Studio are automatically suffixed with `_c`, and their field name cannot be changed after creation. Display labels can still change and must not be used as integration keys.

## Required Provider Mapping Configuration

At minimum, each connected CRM configuration should store:

| Configuration item | Example |
|---|---|
| Connected account or instance | HubSpot portal/account ID or Sugar base URL and instance identifier |
| Canonical record type | `resident`, `service_enrolment`, `service_visit`, `referral`, `enquiry_message` |
| Remote object or module | HubSpot `contacts`; Sugar `Contacts` |
| Canonical field | `resident.room_number` |
| Remote internal field | HubSpot `webres_room_number`; Sugar `room_number_c` |
| Direction | CRM to Schedule Mee, Schedule Mee to CRM, Carebridge to CRM or CRM to Carebridge |
| Authoritative source | Schedule Mee, Carebridge, CRM or explicitly manual |
| Transformation | Date-only conversion, timestamp timezone, identifier formatting or null handling |
| Value map | Canonical `completed` to HubSpot/Sugar internal value |
| Sensitivity classification | Personal, sensitive health, highly sensitive or approved non-clinical workflow data |
| Validation state | Valid, missing, archived, permission denied, incompatible type or approval blocked |

Mappings should be versioned and audited. A change to a field, relationship or value map can alter synchronisation behaviour and should be treated as configuration deployment rather than an untracked edit.

## Matching and Identity Rules

- Store the CRM record ID against the corresponding Schedule Mee or Carebridge record in the Integration Core.
- Use immutable external IDs and CRM unique properties where available for idempotent upsert.
- Do not match residents using names alone.
- Email and phone may assist with candidate matching but should not be assumed unique or current.
- Facility relationships must use stable facility identifiers rather than display names where possible.
- Referral, service-enrolment and service-visit records each need their own cross-system identifier mapping.
- Any manual candidate-matching workflow must be auditable and must prevent automatic merging where confidence is insufficient.

## Sensitive Data and Documents

### HubSpot

HubSpot supports Sensitive Data and Highly Sensitive Data properties for supported objects, but the customer must have an Enterprise subscription, enable the feature and authorise the necessary OAuth scopes. Default properties cannot simply be assumed to have the required sensitivity configuration.

General Files-tool uploads do not receive the additional Sensitive Data protection. Clinical-document synchronisation must therefore remain disabled until the provider's supported upload path, permissions and data-handling approval have been confirmed.

### SugarCRM

SugarCRM's standard Customer Terms dated 1 November 2025 state that the service is not configured to receive or store GDPR special-category data or similar categories of sensitive information under applicable data-protection laws. Australian health information is sensitive information.

It is therefore a reasonable technical and contractual risk conclusion that the proposed clinical summaries, documents, admission information and health-related service history must not be enabled for SugarCRM until the provider and SugarCRM confirm the permitted data classifications in writing. The provider's negotiated agreement may differ from the standard terms, so this is a blocker requiring confirmation rather than a final legal determination.

Until approval is obtained, the safer model for either CRM is to retain clinical content and documents in Carebridge and synchronise only approved workflow data, external identifiers and secure Carebridge links.

## Recommended Initial Mapping Scope

For the first implementation, use one provider account and a deliberately narrow, non-clinical flow:

1. Resident identity using first name, last name and a stable external identifier.
2. One selected referral/enquiry object or module.
3. One-way creation or status synchronisation.
4. Explicit provider-approved status value mapping.
5. No clinical summary, service notes or document copying.
6. Reconciliation and duplicate-prevention testing before enabling bidirectional updates.

This pilot will validate schema discovery, configuration, record identity and operational support without making clinical-data approval a prerequisite for testing the integration architecture.

## Decisions Required Before Implementation

- Which HubSpot object and SugarCRM module will represent a Carebridge referral or enquiry for the pilot provider?
- Does the provider already use HubSpot Services/Appointments or Sugar Purchases/Purchased Line Items/Meetings for the proposed workflows?
- Should Webres map existing provider fields, create integration-managed fields, or support both approaches?
- Who approves each provider's object, field, relationship and value mappings?
- Which system is authoritative for each bidirectional field?
- What immutable external identifiers are available in Schedule Mee and Carebridge?
- Are facilities represented as Companies/Accounts, and how are provider organisations distinguished from individual facilities?
- What definitions should be used for Completed, Cancelled, Missed, Refused, Incomplete and Outstanding?
- Are clinical summaries, service notes and documents required in the CRM, or will secure Carebridge links meet the business need?
- Has the provider's CRM product, contract and security configuration been approved for the relevant personal and health information?

## Sources

### HubSpot

- [HubSpot Contacts API](https://developers.hubspot.com/docs/api-reference/latest/crm/objects/contacts/guide)
- [HubSpot default Contact properties](https://knowledge.hubspot.com/properties/hubspots-default-contact-properties)
- [HubSpot lead-ad field mappings](https://knowledge.hubspot.com/ads/lead-ads-field-mappings-to-hubspot)
- [HubSpot Properties API](https://developers.hubspot.com/docs/api-reference/latest/crm/properties/get-properties)
- [HubSpot Services API](https://developers.hubspot.com/docs/api-reference/latest/crm/objects/services/guide)
- [HubSpot Appointments API](https://developers.hubspot.com/docs/api-reference/latest/crm/objects/appointments/guide)
- [HubSpot Notes API](https://developers.hubspot.com/docs/api-reference/latest/crm/activities/notes/guide)
- [HubSpot Sensitive Data API](https://developers.hubspot.com/docs/api-reference/latest/crm/properties/sensitive-data)
- [Sensitive Data in HubSpot tools](https://knowledge.hubspot.com/account-security/sensitive-data-in-hubspot-tools)

### SugarCRM

- [SugarCRM Contacts](https://support.sugarai.com/documentation/sugar_versions/26.1/sell/application_guide/contacts/)
- [SugarCRM Meetings](https://support.sugarai.com/documentation/sugar_versions/26.1/serve/application_guide/meetings/)
- [SugarCRM Purchases and Purchased Line Items](https://support.sugarai.com/documentation/sugar_versions/26.1/serve/application_guide/purchases_and_purchased_line_items/)
- [SugarCRM Cases](https://support.sugarai.com/documentation/sugar_versions/26.1/serve/application_guide/cases/)
- [SugarCRM Studio fields](https://support.sugarai.com/documentation/sugar_versions/26.1/sell/administration_guide/developer_tools/studio/fields/)
- [SugarCRM Meetings REST endpoint example](https://support.sugarcrm.com/documentation/sugar_developer/sugar_developer_guide_14.0/integration/web_services/rest_api/endpoints/meetings_post/)
- [SugarCRM REST API collection](https://rest.apidocs.sugarcrm.com/)
- [SugarCRM Customer Terms](https://www.sugarcrm.com/wp-content/uploads/legal/Customer-Terms-of-Service-English.pdf)

### Australian privacy context

- [OAIC: What is personal information?](https://www.oaic.gov.au/privacy/your-privacy-rights/your-personal-information/what-is-personal-information)
- [OAIC Guide to Health Privacy](https://www.oaic.gov.au/privacy/privacy-guidance-for-organisations-and-government-agencies/health-service-providers/guide-to-health-privacy)
