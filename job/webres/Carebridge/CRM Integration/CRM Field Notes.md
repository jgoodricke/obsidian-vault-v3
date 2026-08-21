## Purpose

This document records how the proposed Schedule Mee and Carebridge integration data could be represented in HubSpot and SugarCRM. It identifies:

- Fields that have a direct stock CRM equivalent.
- Data that can use a standard CRM object or module but still requires configuration.
- Bespoke fields, relationships, statuses and value mappings required for each connected provider.
- Reporting values that should be derived from underlying records rather than stored as fields.
- Sensitive-data and product constraints that must be resolved before implementation.

This is a proposed integration model, not a final provider configuration. The actual objects, fields, permissions and values must be discovered and confirmed against each connected HubSpot account, SugarCRM instance or Resident Select organisation during onboarding.

## Key Conclusions

- Only a small number of resident fields have direct, stable equivalents across the connected CRMs. First name, last name and date of birth are the main examples.
- Most aged-care, admission, referral and clinical concepts are not native CRM fields.
- HubSpot custom properties are account-specific. SugarCRM custom fields are instance-specific and normally end in `_c`. Resident Select has no custom-field mechanism at all, so gaps there cannot be closed by configuration.
- Standard objects and modules do not eliminate mapping. Pipelines, stages, dropdown values, relationships and semantic use still need to be agreed per connected account or instance.
- Provider-specific mapping should be scoped per CRM account or instance, not automatically per facility. Facilities can reuse a mapping where they share the same schema and workflow.
- Service analytics are derived metrics. They should normally be calculated from appointment or meeting records rather than copied into repeatedly overwritten Contact fields.
- Clinical summaries, health-related workflow data and documents require separate privacy, contractual and product approval. Technical API support alone is insufficient. Resident Select is the exception in kind rather than degree: it is built to hold this data, so the constraint there is minimising what Webres reads rather than obtaining approval to write.

## Classification

| Classification              | Meaning                                                                                                                 |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **Stock field**             | A suitable CRM field exists with a platform-defined internal API name.                                                  |
| **Standard structure**      | A suitable stock object, module, activity or relationship exists, but configuration or value mapping is still required. |
| **Conditional stock field** | HubSpot defines the field, but availability or suitability must be checked in the connected account.                    |
| **Bespoke**                 | A custom property, field, relationship, module, pipeline value or interpretation is required.                           |
| **Derived**                 | The value should be calculated from underlying records rather than treated as an individual CRM field.                  |
| **No API equivalent**       | The platform has a fixed schema with no custom-field mechanism, so the concept cannot be represented at all.            |
| **Approval required**       | The technical mapping is possible, but privacy, contractual, licensing or security approval is unresolved.              |

## Mapping Conventions

The tables use proposed canonical integration keys such as `resident.first_name`. These keys belong to the integration contract and should remain independent of any connected CRM.

CRM mappings must use internal API identifiers, not display labels:

| CRM | Stable identifiers | Provider-specific identifiers |
|---|---|---|
| HubSpot | HubSpot-defined object and property names such as `contacts`, `firstname` and `hs_appointment_start` | Custom property names, pipeline IDs, stage IDs, custom association labels and enumeration values |
| SugarCRM | Stock module and field names such as `Contacts`, `first_name`, `Meetings` and `date_start` | Custom fields ending in `_c`, package-prefixed custom modules, relationship link names and dropdown item names |
| Resident Select | Vendor-defined, fixed object and attribute names such as `clients`, `first_name`, `person_sites[].room_number` and `sites` | No custom fields. Provider-specific values are the numeric IDs in the organisation's lookup tables, such as `site_id`, `person_status_id`, `archive_reason_id` and `gender_id` |

Example custom names in this document, such as `webres_room_number` or `room_number_c`, are illustrative only. They must not be assumed to exist in a provider's CRM. Resident Select is the exception: its attribute names below are taken from the vendor's published API documentation and are fixed for every organisation, but the ID values behind them are not.

### Resident Select

Resident Select (RS) publishes a documented REST API. Unlike HubSpot and SugarCRM it is a fixed-schema, aged-care-specific product: there is no Studio, no custom-property mechanism and no user-defined field capability anywhere in the documentation. Where a canonical concept has no RS attribute it cannot be created, so the **Bespoke** classification used for the other two CRMs does not apply. Those rows are classified **No API equivalent** instead.

The API is also read-rich and write-poor. Every documented endpoint except three is `GET` only. Only these three accept writes:

| Operation | Endpoint | Effect |
|---|---|---|
| Create Client | `POST /clients` | Creates a person with `person_type_id` 1 as a **Prospect** against exactly one Site. |
| Update Client | `PATCH /clients/[id]` | Updates demographic, eligibility and Home Care attributes only. |
| Create Contact | `POST /contacts` | Creates a person with `person_type_id` 2 as a Prospect against exactly one Site. No update endpoint is documented. |

This has two structural consequences for the integration model. There is no writable target for Schedule Mee service occurrences, so the Schedule Mee to CRM direction cannot be implemented against RS as specified. And the Carebridge to CRM direction can only create a Prospect: status, room, referral progression, documents and notes are all read-only.

#### API conventions

| Convention | Detail |
|---|---|
| Base URL | `https://app.residentselect.com.au/api/v1` |
| Authentication | HTTP Basic. `Authorization: Basic ` followed by `base64(api_key:secret_key)`. |
| Credential scope | API keys are issued per **Organisation** and grant access only to that Organisation's data. |
| Key permissions | Separate Read (`GET`), Write (`POST`/`PATCH`) and Delete permissions, plus optional IP allowlisting, set per key in Organisation Settings. The secret key is displayed once and cannot be retrieved afterwards. |
| Request encoding | Form-encoded requests, JSON-encoded responses. |
| Datetimes | Always UTC, `YYYY-MM-DD HH:MM:SS`. |
| Dates | `YYYY-MM-DD`. |
| Pagination | `limit` (default and maximum 500) and `offset`. Responses are wrapped in `from`, `to`, `total`, `offset`, `max_offset`, `limit` and `data`. |
| Sorting | `sort=attribute`, prefixed with `-` for descending, comma-separated for multiple keys. |
| Filtering | `filter[attribute]=value`. Comma-separated values are OR; separate `filter[]` parameters are AND. Name, postcode and phone filters are `%LIKE%`; identifier filters are `=` or `IN`. |
| Errors | Conventional HTTP status codes with a `{"code": …, "error": …}` body. |
| Change notification | **None.** No webhooks, callbacks or event subscriptions are documented. Change detection must poll `filter[updated_start_date]` and `filter[updated_end_date]`, which are available on Activities, Aged Care Quotes, Clients, Client Contacts, Clinical Reviews, Contacts, Contracts, Home Care Quotes and Relations. |
| Rate limits | Not documented. Must be confirmed with the vendor before designing polling frequency. |

#### Object model

RS uses a single person model split across three endpoints by `person_type_id`: Client (1), Contact (2) and Client Contact (3). A resident is a **Client**. Next of kin and other related parties are **Client Contacts**, joined through the `/relations` endpoint using `relation_type_id`.

The attributes that matter most to this integration are not on the Client itself. They sit in `person_sites`, an array on the Client (and Contact) representing that person's state at each Site:

| `person_sites[]` attribute | Meaning |
|---|---|
| `site_id` | The Site (facility). Resolve through the Site object for `name` and `service_id`. |
| `person_status_id` | Lifecycle status at that Site. Refer Client Status object. |
| `archive_reason_id` | Set when the status becomes Archived. Refer Archive Reason object. |
| `referrer_id` | The referral source record. |
| `room_number` | Room at that Site. **Per site, not per person.** |
| `prospect_date`, `permanent_waitlist_date`, `permanent_resident_date`, `respite_waitlist_date`, `respite_resident_date`, `archived_date` | The date each lifecycle stage was reached. |
| `current_status_date` | The date of the current status. |
| `is_prospect_status` | Whether the current status is a prospect status. |

A Site is the RS equivalent of a facility: `id`, `name` and `service_id`, where the Service is Aged Care (1), Home Care (2) or RV (3). Because `room_number` and status are per `person_site`, a Client present at more than one Site has more than one room and more than one status, and the integration must select the correct `person_site` rather than assuming a single value per resident.

#### Resident details (Resident Select)

| Integration field | Canonical key | Resident Select mapping | Classification | Provider-specific work |
|---|---|---|---|---|
| First name | `resident.first_name` | Client `first_name` | **Stock field** | Writable. Required on create. |
| Last name | `resident.last_name` | Client `last_name` | **Stock field** | Writable. Required on create. |
| Room number | `resident.room_number` | Client `person_sites[].room_number` | **Stock field, read-only** | Select the correct `person_site` by `site_id`. Cannot be written back. Do not confuse with Clinical Review `bed_number`, which is the offered bed during assessment rather than the occupied room. |
| Facility | `resident.facility_id` | Client `person_sites[].site_id`, resolved through Site `id`, `name` and `service_id` | **Standard structure** | Map by `site_id`, never by Site name. Confirm whether the provider's Schedule Mee facilities are one-to-one with RS Sites. |
| Wing | `resident.wing_id` | None | **No API equivalent** | RS has no wing, unit, house or neighbourhood concept and no custom field to hold one. Either drop the field for RS providers or derive it from `room_number` using a provider-supplied convention, which must be agreed and documented. |
| Date of birth | `resident.date_of_birth` | Client `date_of_birth` | **Stock field** | Writable. Date only, `YYYY-MM-DD`. |
| Gender | `resident.gender` | Client `gender_id`, resolved through the Gender object | **Stock field plus value mapping** | Writable. Documented values are Female (1), Male (2), Not Specified (3) and Indeterminate/Intersex/Unspecified (4). Confirm IDs at runtime rather than hard-coding them. |
| Admission date (expected) | `resident.admission_date` | Client `expected_admission_date`, or Contract `admission_date` | **Stock field, semantic check required** | `expected_admission_date` is writable on the Client. Contract `admission_date` is documented as the *expected* admission date and is read-only. Agree which is authoritative. |
| Admission date (actual) | `resident.admission_date` | Client `person_sites[].permanent_resident_date` or `respite_resident_date` | **Stock field, read-only** | The actual admission is the date the person reached Permanent Resident or Respite Resident status at the Site. Permanent and respite are distinct dates and must not be collapsed into one field without agreement. |
| Expected discharge date | `resident.expected_discharge_date` | None | **No API equivalent** | `person_sites[].archived_date` is not a discharge date. Archiving covers outcomes such as admission elsewhere, no vacancy and death, and is recorded when the record is closed rather than when a departure is planned. Retain this field in Schedule Mee or Carebridge. |
| About Me profile | `resident.about_me` | Client `notes`, `other_information`, or the Home Care `hc_notes_*` category fields | **Stock field, semantic check required** | `notes` is documented as generic aged care notes and `hc_notes` as generic home care notes. Both are shared operational fields already used by RS staff, so writing an About Me profile into them risks overwriting provider content. Confirm ownership, maximum length and whether a shared field is acceptable before use. |

RS also exposes resident attributes with no counterpart in the current canonical model. They are listed here so the integration contract can decide explicitly whether to consume them, not because they are in scope:

| Group                      | Client attributes                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Contact details            | `email`, `phone`, `mobile`, `address1`, `address2`, `suburb`, `state_id`, `zip`                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Identity                   | `nationality_id`, `is_aboriginal_origin`, `aged_care_recipient_id`, `external_id`                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Assessment and eligibility | `received_aged_care_assessment`, `aged_care_assessment_date`, `approved_for_permanent_care`, `approved_for_respite`, `completed_centrelink_assessment`, `centrelink_assessment_date`, `permanent_referral_code`, `respite_referral_code`, `timeline_required_for_care_id`                                                                                                                                                                                                                                                               |
| Financial                  | `aged_care_financial_status_id`, `pension_number`, `pension_status_id`, `an_acc`, `private_health_provider`, `private_health_member_number`, `expected_room_price`, `likelihood_of_admission`                                                                                                                                                                                                                                                                                                                                           |
| Health identifiers         | `medicare_number`, `medicare_card_position`, `medicare_expiry_year`, `medicare_expiry_month`                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| General practitioner       | `gp_name`, `gp_clinic_name`, `gp_phone`, `gp_email`, `gp_address`, `gp_suburb`, `gp_zip`, `gp_state_id`                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| Home Care                  | `user_id`, `mac_assessment_status_id`, `hc_funding_type_id`, `hc_package_id`, `hc_interim_package_id`, `hc_package_wait_time_id`, the `hc_package_*` and `hc_chsp_*` approval and expiry dates, and nineteen `hc_notes_*` service-category fields covering allergies, cleaning, communication, continence, delivered meals, domestic assistance, exercise physiology, gardening, home modifications, meal preparation, medication, mobility, nursing, occupational therapy, personal care, physiotherapy, respite, social and transport |
| External integration       | `epicor_id`, `epicor_customer_number`, `epicor_status`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |

The `epicor_*` attributes indicate that RS already supports at least one ERP integration. Confirm whether the provider uses it before assuming Webres owns the integration surface.

#### Service information (Resident Select)

RS has no service-enrolment object for individual care services. There is no equivalent of a HubSpot Service or a Sugar Purchased Line Item, and no per-service commencement or end date.

| Integration field | Canonical key | Resident Select mapping | Classification | Provider-specific work |
|---|---|---|---|---|
| Service name | `service_enrolment.service_name` | Site `service_id`, resolved through the Service object as Aged Care, Home Care or RV | **Programme only, not a service** | The Service object identifies which programme a Site belongs to. It is not a catalogue of deliverable services and must not be mapped as one. |
| Service category | `service_enrolment.service_name` | The nineteen `hc_notes_*` Home Care fields | **Approximation, Home Care only** | These are free-text note fields per category, not enrolment records. They indicate which categories a Home Care client has notes against; they do not establish that a service is enrolled or active. |
| Service frequency | `service_enrolment.frequency` | None | **No API equivalent** | Must remain in Schedule Mee. |
| Commencement date | `service_enrolment.commencement_date` | For Home Care packages only: `hc_package_assigned_date`, `hc_chsp_date_approved` | **Package-level, not service-level** | These are funding milestones for the package, not the commencement of an individual service. |
| End date | `service_enrolment.end_date` | For Home Care packages only: `hc_package_expiry_date`, `hc_package_extension_expiry_date`, `hc_chsp_expiry_date` | **Package-level, not service-level** | As above. Do not present a package expiry as a service end date. |

Service enrolment should therefore remain authoritative in Schedule Mee for RS providers. RS can supply funding context, but it cannot hold the enrolment itself.

#### Service history (Resident Select)

**This direction is blocked for Resident Select.** There is no writable endpoint for a service occurrence.

The nearest object is the Activity, which records `activity_type_id` or `system_activity_type_id`, `site_id`, `user_id`, `person_id`, `referrer_id`, `activity_date`, `description`, `date_completed`, `created_by`, `created_at` and `updated_at`. It is `GET` only, and its Activity Types are sales and administration events such as Application pack given, Charter signed and Contract created rather than delivered care. System Activity Types are generated by RS itself and are explicitly documented as not manually creatable.

Activities should not be mapped to `service_visit`. Doing so would misrepresent both the write feasibility and the meaning of the record. Service history for RS providers must stay in Schedule Mee, with reporting derived there.

The consequence for the Analytics and Reporting table is that every derived metric must be calculated in Schedule Mee for RS providers. There is no RS-side record set to count and no field to write a cached metric into.

#### Referral, enquiry and admission outcome

| Integration information | Canonical key or contract | Resident Select mapping | Classification | Provider-specific work |
|---|---|---|---|---|
| New referrals into RS | `referral.created` | `POST /clients` with `first_name`, `last_name` and `site_id` required | **Standard structure, constrained** | Creates the person as a Prospect at exactly one Site. Multiple sites require multiple calls or manual work in RS. `prospect_date` defaults to today if omitted. Set `referrer_id` on create; the documentation recommends it and it cannot be corrected later through the API. |
| Referral status updates | `referral.updated` | Client `person_sites[].person_status_id` | **Read-only** | Status cannot be written. RS is authoritative for lifecycle status and the integration can only observe it. |
| Enquiry status changes | `enquiry.status` | Client `person_sites[].person_status_id` and `current_status_date` | **Read-only, value mapping required** | Poll with `filter[person_status_id]` and the `updated_*` date filters. |
| Admission outcome | `enquiry.admission_outcome` | Client `person_sites[].person_status_id`, plus `archive_reason_id` for negative outcomes | **Read-only, value mapping required** | Archive Reasons carry the outcome detail, for example "Admission to one of our other homes", "Another facility had availability" and "Discharge - gone back home". The published list is truncated, so retrieve it in full per Organisation. |
| Date admitted | `enquiry.date_admitted` | Client `person_sites[].permanent_resident_date` or `respite_resident_date` | **Read-only** | Distinguish permanent from respite. |
| Referral source | `referral.provider_id`, `referral.facility_id` | Client `person_sites[].referrer_id`, resolved against Referrer Organisations, Referrer People, Referrer Events, Referrer Campaigns or Miscellaneous Referrers | **Standard structure, polymorphic** | `referrer_id` is a single identifier resolved across five separate endpoints with independent ID spaces. The referrer type must be known to resolve it correctly. All five are read-only. |
| Documents | `referral.document` | Contract `unsigned_pdf_url` and `signed_pdf_url`; Aged Care Quote `signed_pdf_url`; Home Care Quote `pdf_url` | **Read-only, not storable as links** | These are AWS S3 presigned URLs valid for 60 minutes (Contracts and Aged Care Quotes) or 10 minutes (Home Care Quotes), and only returned when retrieving an individual record. They cannot serve as the durable secure link the Documents row recommends. No document upload endpoint exists. |
| Agreements | `enquiry.admission_outcome` supporting detail | Contract `contract_type_id`, `contract_status_id`, `admission_date`, `accepted_at`, `is_using_docusign`, `docusign_envelope_id` | **Read-only** | Contract Types are Residential (1), Additional Services Agreement (2), Respite (3) and Home Care (4). |
| Clinical assessment | `referral.clinical_summary` | Clinical Review `status`, `status_date`, `bed_type`, `bed_number`, `bed_offer_status`, `offer_date`, `client_response` | **Read-only, health information** | Values are documented string literals rather than ID lookups. Read only what the approved data-minimisation scope permits. |
| Messages or comments | `enquiry_message.created` | None | **No API equivalent** | Activities are read-only and Client `notes` is a shared operational field. There is no message or comment object. |

#### Status and outcome values

Client Statuses are per Service, so the same label appears with different IDs for Aged Care and Home Care. The published Aged Care example shows Prospect (1), Permanent Waitlist (2), Permanent Resident (3) and Respite Waitlist (4), then truncates; a worked example elsewhere in the documentation shows Archived as 6, and Home Care begins at Prospect (7). Respite Resident is not shown with an ID. Treat the full list as unknown until retrieved from the endpoint. Contact Statuses are a separate list, with Prospect (1) and Archived (6) for Aged Care and Prospect (7) and Archived (9) for Home Care.

| Canonical concept | Resident Select representation | Mapping requirement |
|---|---|---|
| Enquiry received | Client Status Prospect, with `prospect_date` | The only status the API can create. |
| Waitlisted | Permanent Waitlist or Respite Waitlist, with the matching date attribute | Permanent and respite are separate statuses and must not be merged. |
| Admitted | Permanent Resident or Respite Resident, with the matching date attribute | Read-only. Use `current_status_date` to detect the transition. |
| Not proceeding | Archived, qualified by `archive_reason_id` | The reason carries the outcome. Map reasons per organisation. |
| In progress or intermediate | No general equivalent | Clinical Review `bed_offer_status` and `client_response` are the closest signals during assessment, and are health information. |

Because both status and archive reason are read-only, RS is the authoritative source for the enquiry lifecycle. Any provider workflow that expects the CRM to accept a status update from Carebridge or Schedule Mee is not supported by this API and must be resolved as a process question rather than a mapping question.

#### Schema and value discovery

RS has a fixed schema, so property discovery of the HubSpot or SugarCRM kind is neither available nor necessary. What must be discovered per Organisation is the contents of the lookup tables. The published examples are truncated with `{...}`, and the documentation does not state that the ID space is stable across organisations, so the values must be retrieved rather than assumed.

The adapter should retrieve and cache, at minimum:

```http
GET /sites
GET /services
GET /client-statuses
GET /contact-statuses
GET /person-types
GET /genders
GET /nationalities
GET /archive-reasons
GET /relation-types
GET /activity-types
GET /system-activity-types
GET /aged-care-timelines
GET /aged-care-financial-statuses
GET /aged-care-quote-statuses
GET /contract-statuses
GET /contract-types
GET /home-care/funding-types
GET /home-care/packages
GET /home-care/package-wait-times
GET /home-care/mac-assessment-statuses
GET /users
```

Two gaps must be raised with the vendor. The Client object references a **State object** and a **Pension Status object**, but no `/states` or `/pension-statuses` endpoint is documented, so `state_id`, `gp_state_id` and `pension_status_id` cannot be resolved to labels through the API. Runtime discovery is therefore mandatory but incomplete.

#### Identity and idempotency

RS provides `external_id` on Clients, Contacts and Client Contacts, described as a field for storing a value corresponding to a record in a separate system. It is writable on `POST /clients` and `PATCH /clients/[id]` and filterable with `filter[external_id]=`, which makes it the correct anchor for idempotent upsert.

It is a **single slot per person**. If the provider already uses it, for an Epicor or other integration, Webres cannot also own it. The rules are therefore:

- Store the RS `id` against the Schedule Mee or Carebridge record in the Integration Core. This is the durable link and does not depend on `external_id`.
- Confirm during onboarding whether `external_id` is already in use, and agree a single owner per Organisation.
- Where `external_id` is unavailable, fall back to Integration Core mapping only. Do not match residents on name, and note that name, postcode, phone and mobile filters are `%LIKE%` rather than exact, so they can only ever produce match candidates.
- `person_sites[].id` identifies a person's record at a specific Site and is the correct key for room, status and status-date synchronisation.

#### Write constraints to confirm

| Item | Documented position | Action |
|---|---|---|
| Contact updates | `POST /contacts` exists but no `PATCH` is documented | Confirm whether Contacts can be updated at all. If not, a Contact created in error can only be corrected inside RS. |
| Delete endpoints | The API key configuration offers an "API Delete Permission" granting access to "all DELETE API endpoints", but no `DELETE` endpoint is documented for any resource | Confirm whether undocumented delete endpoints exist. Until confirmed, issue keys without delete permission. |
| Client Contacts | Read-only | Confirm whether next of kin can be created through any supported route. |
| Multi-site residents | Create accepts only one `site_id` | Confirm the process for a resident who moves between Sites. |
| Rate limits | Not documented | Obtain limits before setting polling frequency, given that polling is the only change-detection mechanism. |

#### Sensitive data

The sensitivity position is the reverse of the SugarCRM one. RS is purpose-built for Australian aged care and already holds Medicare numbers, pension details, ACAT and Centrelink assessment outcomes, GP details, `is_aboriginal_origin` and clinical review outcomes by design, so the question is not whether the platform may receive health information but how little of it Webres should read: data minimisation applies on read, and the adapter should request only the attributes the approved scope requires rather than storing whole Client objects.

## CRM to Schedule Mee

### Resident Details

| Integration field       | Canonical key                      | HubSpot mapping                                                                                      | HubSpot classification            | SugarCRM mapping                                                                                                   | SugarCRM classification           | Provider-specific work                                                                                                                                |
| ----------------------- | ---------------------------------- | ---------------------------------------------------------------------------------------------------- | --------------------------------- | ------------------------------------------------------------------------------------------------------------------ | --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| First name              | `resident.first_name`              | Contact `firstname`                                                                                  | **Stock field**                   | Contacts `first_name`                                                                                              | **Stock field**                   | Confirm that the provider has not chosen a separate custom identity field.                                                                            |
| Last name               | `resident.last_name`               | Contact `lastname`                                                                                   | **Stock field**                   | Contacts `last_name`                                                                                               | **Stock field**                   | Confirm matching and duplicate rules.                                                                                                                 |
| Room number             | `resident.room_number`             | Custom Contact property, for example `webres_room_number`                                            | **Bespoke**                       | Custom Contacts field, for example `room_number_c`                                                                 | **Bespoke**                       | Select or create the property and confirm whether room numbers are unique only within a facility.                                                     |
| Facility                | `resident.facility_id`             | Associate the Contact with a Company representing the facility, or use a custom property/association | **Standard structure or bespoke** | Relate the Contact to an Account representing the facility, or use a custom Facility module/relationship           | **Standard structure or bespoke** | Confirm what Company/Account represents in the provider's existing CRM. Map by a stable facility identifier rather than facility name where possible. |
| Wing                    | `resident.wing_id`                 | Custom Contact property or relationship to a custom facility/wing object                             | **Bespoke**                       | Custom Contacts field or relationship to a custom Wing/Facility module                                             | **Bespoke**                       | Confirm whether a free-text value, controlled list or related record is required.                                                                     |
| Date of birth           | `resident.date_of_birth`           | Contact `date_of_birth` where available                                                              | **Conditional stock field**       | Contacts `birthdate`                                                                                               | **Stock field**                   | Verify the HubSpot property through the connected account's schema. Confirm date-only handling and whether a custom sensitive property is required.   |
| Gender                  | `resident.gender`                  | Contact `gender` where available                                                                     | **Conditional stock field**       | Custom Contacts dropdown, for example `gender_c`                                                                   | **Bespoke**                       | Map internal enumeration values. Confirm terminology, allowed values, null handling and sensitive-data treatment.                                     |
| Admission date          | `resident.admission_date`          | Custom date property                                                                                 | **Bespoke**                       | Custom date field, for example `admission_date_c`                                                                  | **Bespoke**                       | Do not reuse generic CRM close-date fields because their meaning differs. Confirm the authoritative system.                                           |
| Expected discharge date | `resident.expected_discharge_date` | Custom date property                                                                                 | **Bespoke**                       | Custom date field, for example `expected_discharge_date_c`                                                         | **Bespoke**                       | Confirm whether this is planned, estimated or confirmed and how changes are audited.                                                                  |
| About Me profile        | `resident.about_me`                | Custom Sensitive Data multiline property, or an associated Note if timeline history is required      | **Bespoke**                       | A dedicated custom text-area field is recommended; Contacts `description` is technically available but too general | **Bespoke recommended**           | Confirm maximum length, formatting, edit ownership, history requirements and whether the content is approved for the CRM.                             |

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

| Integration field | Canonical key                                         | HubSpot mapping                                                                                   | HubSpot classification                | SugarCRM mapping                                                                      | SugarCRM classification               | Provider-specific work                                                                                                                      |
| ----------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| Resident name     | `service_visit.resident_id`                           | Associate Appointment to Contact; display name comes from `firstname` and `lastname`              | **Standard structure**                | Relate Meeting to Contact; display name comes from `first_name` and `last_name`       | **Standard structure**                | Store cross-system record IDs. Do not duplicate the resident's name on every service occurrence unless required for exports.                |
| Service name      | `service_visit.service_name`                          | Appointment `hs_appointment_name`, optionally associated with a Service                           | **Stock field in standard structure** | Meetings `name` (Subject), optionally related to a service record                     | **Stock field in standard structure** | Decide whether the name is a snapshot or derived from a related service catalogue record.                                                   |
| Service date      | `service_visit.starts_at`                             | Appointment `hs_appointment_start`                                                                | **Stock field**                       | Meetings `date_start`                                                                 | **Stock field**                       | Store a full timestamp and agree timezone conversion.                                                                                       |
| Service time      | `service_visit.starts_at` and `service_visit.ends_at` | Appointment `hs_appointment_start` and `hs_appointment_end`                                       | **Stock fields**                      | Meetings `date_start` and `date_end`                                                  | **Stock fields**                      | Normalise timestamps to UTC in the integration while preserving provider-local display behaviour.                                           |
| Service status    | `service_visit.status`                                | Appointment `hs_appointment_status`                                                               | **Stock field plus value mapping**    | Meetings `status`                                                                     | **Stock field plus value mapping**    | Map canonical values to the account/instance's internal enumeration values. Missed, Refused and Incomplete need a custom outcome or reason. |
| Service notes     | `service_visit.notes`                                 | Associated Note `hs_note_body`, or a custom Appointment property if direct filtering is essential | **Standard activity or bespoke**      | Meetings `description` (Summary), Internal Notes where exposed, or an associated Note | **Stock structure**                   | Confirm author, source, edit ownership, maximum length and whether notes contain health information.                                        |

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

| Integration information       | Canonical key or contract                                   | HubSpot mapping                                                                                        | SugarCRM mapping                                                                                                    | Classification                                    | Provider-specific work                                                                                                                                                    |
| ----------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| New referrals                 | `referral.created` and referral record                      | Deal, Ticket or agreed custom object associated with the patient Contact and provider/facility Company | Case, Lead, Opportunity or custom Referral module associated with the patient Contact and provider/facility Account | **Standard structure with object/module mapping** | Agree the semantic model before mapping fields. Store Carebridge's immutable external identifier.                                                                         |
| Referral updates              | `referral.updated`                                          | Deal stage, Ticket status or custom status property                                                    | Case Status, Lead Status, Opportunity Sales Stage or custom status field                                            | **Standard field plus value mapping**             | Map pipeline/stage/dropdown internal values and define source-of-truth rules.                                                                                             |
| Documents                     | `referral.document`                                         | File associated through a Note using `hs_attachment_ids`                                               | Document record or Note attachment associated with the referral record                                              | **Standard capability; approval required**        | Confirm allowed document types, size limits, retention, permissions and whether copying clinical documents is permitted. Prefer a secure Carebridge link if not approved. |
| Patient demographics          | `resident.*` approved subset                                | Contact properties using the resident mappings above                                                   | Contacts fields using the resident mappings above                                                                   | **Mixed**                                         | Apply field-by-field mapping and data-minimisation rules.                                                                                                                 |
| Clinical summary              | `referral.clinical_summary`                                 | Custom Sensitive Data multiline property or another approved sensitive structure                       | Custom text field or Note only if contractually and legally approved                                                | **Bespoke; approval required**                    | HubSpot requires suitable Sensitive Data configuration and scopes. SugarCRM is blocked pending written contractual confirmation.                                          |
| Hospital messages or comments | `enquiry_message.created`, direction `hospital_to_provider` | Note `hs_note_body` associated with the enquiry/referral record                                        | Associated Note or Case Comment Log                                                                                 | **Standard activity plus bespoke metadata**       | Add or map immutable external message ID, source, direction, author and timestamp. Define update and deletion behaviour.                                                  |

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

| Integration information       | Canonical key or contract                                   | HubSpot mapping                                            | SugarCRM mapping                                                      | Classification                              | Provider-specific work                                                                                                                             |
| ----------------------------- | ----------------------------------------------------------- | ---------------------------------------------------------- | --------------------------------------------------------------------- | ------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Enquiry status changes        | `enquiry.status`                                            | Deal stage, Ticket status or custom status property        | Case Status or status/stage field of the selected referral module     | **Standard field plus value mapping**       | Map values, determine the authoritative source and prevent update loops.                                                                           |
| Admission outcome             | `enquiry.admission_outcome`                                 | Custom enumeration or deliberately modelled pipeline stage | Custom dropdown or deliberately modelled Case/Lead/Opportunity status | **Bespoke or status mapping**               | Agree outcome vocabulary and distinguish outcome from workflow status.                                                                             |
| Date admitted                 | `enquiry.date_admitted`                                     | Custom date property                                       | Custom date field, for example `date_admitted_c`                      | **Bespoke**                                 | Do not reuse HubSpot Contact Close date or a Sugar sales close date.                                                                               |
| Information requests          | `information_request.created`                               | Ticket, Task or Note depending on provider workflow        | Case, Task or Note depending on provider workflow                     | **Standard structure with object mapping**  | Define whether this is a separate actionable record or a message on the enquiry. Map ownership, due date and completion state if actionable.       |
| Provider messages or comments | `enquiry_message.created`, direction `provider_to_hospital` | Associated Note `hs_note_body`                             | Associated Note or Case Comment Log                                   | **Standard activity plus bespoke metadata** | Map source, direction, author, timestamp and external ID. Avoid treating arbitrary CRM notes as Carebridge messages without an integration marker. |

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

### Resident Select

- [Resident Select API documentation](https://app.residentselect.com.au/api/v1/documentation/)
- [Resident Select Clients endpoint](https://app.residentselect.com.au/api/v1/documentation/clients)
- [Resident Select Contacts endpoint](https://app.residentselect.com.au/api/v1/documentation/contacts)
- [Resident Select Activities endpoint](https://app.residentselect.com.au/api/v1/documentation/activities)
- [Resident Select Clinical Reviews endpoint](https://app.residentselect.com.au/api/v1/documentation/clinical-reviews)
- [Resident Select Contracts endpoint](https://app.residentselect.com.au/api/v1/documentation/contracts)
- [Resident Select Sites endpoint](https://app.residentselect.com.au/api/v1/documentation/sites)
- [Resident Select Client Statuses endpoint](https://app.residentselect.com.au/api/v1/documentation/client-statuses)
- [Resident Select Archive Reasons endpoint](https://app.residentselect.com.au/api/v1/documentation/archive-reasons)

### Australian privacy context

- [OAIC: What is personal information?](https://www.oaic.gov.au/privacy/your-privacy-rights/your-personal-information/what-is-personal-information)
- [OAIC Guide to Health Privacy](https://www.oaic.gov.au/privacy/privacy-guidance-for-organisations-and-government-agencies/health-service-providers/guide-to-health-privacy)
