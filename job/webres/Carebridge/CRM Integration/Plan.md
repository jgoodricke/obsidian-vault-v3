# PROTOTYPE — Carebridge–Resident Select Initial Production Release estimate

## Approved decision

Use the following scope, labour envelope, milestone sequence, staffing scenarios, assumptions, risks, and acceptance gates for project approval.

**Proposed approval labour envelope: 240–360 person-days.**

This envelope includes a base estimate of 198–297 person-days and a 20% management reserve of 40–60 person-days. Keep vendor wait time and recurring service costs outside this labour envelope.

## Estimate basis

One person-day means one focused engineering workday. The estimate:

- Covers the first production Carebridge–Resident Select Connection and its agreed Sites.
- Covers engineering in the Integration Platform and Carebridge codebases.
- Treats this repository as a starter. The Core, persistence, authentication, administration UI, linking API, and deployment are new work.
- Includes implementation, normal code review and rework, automated tests, failure recovery, AWS automation, coordination, acceptance support, and first-Connection onboarding.
- Uses the conservative architecture in [the technical design and discovery record](../technical-design-and-discovery-record.md).
- Uses the workflow and acceptance boundary in [the Initial Production Release Flow](../integration-flows/carebridge-resident-select.md).
- Uses the release scope in [Deliver an Initial Production Release for one Carebridge–Resident Select Connection](../adr/0036-deliver-an-initial-production-release-for-one-carebridge-resident-select-connection.md).
- Uses one scoped reverse-call credential per Connection and environment as set by [Authenticate Carebridge calls to the Integration Core with scoped service credentials](../adr/0037-authenticate-carebridge-calls-to-the-integration-core.md).

The workstream ranges allow for normal implementation variation under the documented API. The management reserve covers normal vendor surprises and integration rework. It does not cover a missing core vendor capability or a decision to increase release scope.

## Person-day estimate

| Workstream                                | Included outcome                                                                                                                                                                              | Person-days |
| ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------: |
| Delivery design and contracts             | Confirm schema, API contracts, security boundaries, mappings, acceptance data, and release controls.                                                                                          |        8–12 |
| Integration Core foundation               | Tenant and Connection model, durable records, PostgreSQL work ledger, leases, checkpoints, scheduling, scoped service authentication, and minimum audit.                                      |       26–38 |
| Carebridge machine boundary               | Read projections, Integration Principal, policy scope, idempotent domain commands, optimistic concurrency, native side effects, and link display projection.                                  |       22–32 |
| Provider link-or-create workflow          | Carebridge modal, server-to-Core calls, search and exact-ID UX, stale-preview control, link-existing, create-new confirmation, and result display.                                            |       16–24 |
| Resident Select adapter and outbound Flow | Basic authentication, pagination, lookups, search, exact reads, Prospect creation, demographic comparison and update, retry classification, and reconciliation.                               |       20–30 |
| Inbound workflow Flow                     | Evidence snapshots, precedence, progression lanes, Proposed Actions, approval and automatic paths, terminal controls, revalidation, and Carebridge transition calls.                          |       22–34 |
| Administration and operations             | Local role access, credentials, mappings, health, approvals, conflicts, safe retry, exceptional linking, redacted history, and bounded notifications.                                         |       20–30 |
| AWS delivery                              | Version-controlled staging and production infrastructure, GitHub Actions, Fargate, RDS, secrets, private networking, production HTTPS, telemetry, backup controls, and deployment procedures. |       18–28 |
| Cross-cutting verification and hardening  | Integration and browser tests, concurrency tests, idempotency, recovery, credential rotation, privacy and log review, reconciliation, and release stabilization.                              |       24–36 |
| Test Organisation and first Connection    | Synthetic validation, pilot mappings, first live onboarding, controlled writes, Provider acceptance, and production support.                                                                  |       12–18 |
| Coordination and release evidence         | Cross-codebase sequencing, vendor and managed-service coordination, decision records, runbooks, acceptance evidence, and approval support.                                                    |       10–15 |
| **Base engineering labour**               |                                                                                                                                                                                               | **198–297** |
| **20% management reserve**                | Normal vendor uncertainty, integration rework, and estimate variance.                                                                                                                         |   **40–60** |
| **Approval labour envelope, rounded**     |                                                                                                                                                                                               | **240–360** |

The item estimates include unit and component tests. The cross-cutting item includes end-to-end, browser, concurrency, recovery, and release tests. This separation avoids counting the same test twice.

### Named reductions

Use the full envelope until the responsible provider confirms these conditions:

- Reuse of suitable VPC, subnet, routing, and operator-access patterns: reduce about 4–7 person-days.
- No Resident Select source-IP allowlisting: reduce about 2–4 person-days.
- If both conditions are confirmed, cap the combined reduction at about 6–10 person-days because the network work overlaps.

Do not spend these reductions before confirmation.

## Outcome-based milestones

These are planning groups, not a detailed implementation backlog.

| Milestone                                | Outcome and acceptance gate                                                                                                                                                                                                      | Dependencies                                                                                                      | Base person-days |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ---------------: |
| 0. Delivery boundary ready               | Named owners accept the API, data, security, mapping, test-data, and acceptance boundaries. Vendor and AWS access requests have owners and target dates.                                                                         | Project approval and named decision owners.                                                                       |             8–12 |
| 1. Secure staging platform operates      | The Core deploys to isolated non-public staging. Authentication, tenancy, PostgreSQL durability, jobs, secrets, health, audit, and safe deployment work without production data.                                                 | AWS access, approved private Carebridge path, and infrastructure ownership.                                       |            44–66 |
| 2. A Provider can link or create safely  | An authorized Provider user can search or fetch a test Client, compare current values, link an existing Client, or create and link one Prospect. The Core validates identity and projects link state to Carebridge.              | Milestone 1, Carebridge machine boundary, Resident Select test Organisation, credentials, Sites, and lookup data. |            42–60 |
| 3. Both directional Flows are controlled | Outbound demographics and inbound Workflow Evidence operate through Observe, Approval, and allowed Automatic paths. Revalidation, terminal controls, idempotency, conflicts, and ambiguous-create handling pass synthetic tests. | Milestone 2 and confirmed vendor pagination, change detection, search, create, update, and lookup behaviour.      |            48–74 |
| 4. Production service is supportable     | Production automation, Multi-AZ data service, two tasks, managed HTTPS, telemetry, backups, credential rotation, reconciliation, recovery, runbooks, and security checks pass release review.                                    | Milestones 1–3, provider network and DNS changes, and SES production access if email is enabled.                  |            44–66 |
| 5. First Connection is accepted          | Agreed Sites are configured. Provider-authored live link or creation and the agreed directional outcomes pass. Acceptance evidence is stored and support ownership is active.                                                    | Milestone 4, client availability, live credentials, approved mappings, and a controlled live validation window.   |            12–19 |
| **Base engineering labour**              |                                                                                                                                                                                                                                  |                                                                                                                   |      **198–297** |

### Milestone rules

- Do not use production as a test environment.
- Stop and reconsider the plan if no Resident Select test Organisation is available.
- Do not pass Milestone 2 if Client search, exact lookup, or safe Prospect creation is not possible.
- Do not pass Milestone 3 if the required Workflow Evidence cannot be read or if safe Carebridge transitions cannot be applied.
- Defer an optional demographic field if unsupported vendor behaviour or disproportionate cost makes it unsuitable. Core fields and core workflow paths cannot be deferred.
- Do not pass Milestone 4 without a private Carebridge-to-Core staging path.

## Staffing scenarios

The person-day envelope does not change when more engineers join. More people can reduce elapsed time, but they add coordination and cannot remove serial acceptance gates.

### One engineer

- One full-stack integration engineer works across both codebases and AWS.
- Product, Carebridge domain, managed-service, and Resident Select contacts remain available for decisions and approvals.
- Engineering duration: about 48–72 productive weeks.
- Planning window: about 12–18 calendar months, plus any critical external wait that cannot overlap engineering.
- Main risk: one person owns two codebases, infrastructure, vendor integration, and release support. Context switching and absence directly affect the critical path.

### Two engineers

- Engineer A leads the Integration Core, Resident Select adapter, data model, and AWS delivery.
- Engineer B leads the Carebridge machine boundary and Provider workflow.
- Both engineers share contracts, integration tests, failure recovery, security review, and acceptance.
- Ideal capacity gives 24–36 weeks. Serial gates and integration reduce this gain.
- Planning window: about 28–42 productive weeks, or 7–10 calendar months, plus any critical external wait that cannot overlap engineering.
- Add a stable integration cadence from the first milestone. Do not delay cross-codebase testing until the end.

The coordination and acceptance work in the labour table is engineering labour. Client, vendor, Product Owner, and managed-service staff time is not estimated and must be booked separately.

## External wait time

Do not convert these waits into person-days. Obtain an owner and lead-time commitment for each item at approval. Add only the part that cannot overlap engineering to the calendar scenario.

| External item | Effect if late |
| --- | --- |
| Resident Select test Organisation and read/write credentials | Blocks supported-path validation and Milestones 2 and 3. Production is not a fallback. |
| Resident Select answers about search, pagination, nested change detection, rate limits, errors, create, update, and lookups | Can consume reserve. A missing core capability triggers scope review. |
| AWS account access and managed-service network approval | Blocks staging infrastructure and the private Carebridge path. |
| DNS, certificates, routing, peering, security-group, or endpoint changes | Can block staging integration or production release. |
| SES production access | Can block production email. Keep it parallel to core engineering. |
| First client validation window and live credentials | Blocks Milestone 5 but should not block synthetic staging work. |

## Recurring costs outside labour

The labour envelope excludes all recurring charges. Approval must acknowledge this service basket:

- Three baseline Fargate tasks: one staging task and two production tasks.
- One staging Single-AZ RDS instance and one production Multi-AZ RDS deployment, with storage and automated backups.
- Secrets Manager secrets for each environment and Connection.
- CloudWatch logs, metrics, alarms, and data retention.
- One production Application Load Balancer and ACM certificate use.
- Independent staging and production NAT or static-egress resources, EIPs, and data processing.
- The selected private Carebridge-to-Core staging construct, if it has a recurring charge.
- Low-volume SES transactional email.
- Resident Select licence, API, test-Organisation, support, or environment charges if the vendor applies them.

A numeric monthly total is not defensible before the team selects task and database sizes, confirms the private-network construct, measures log volume, and receives the Resident Select commercial terms. Before production infrastructure purchase, run an AWS cost forecast with the selected sizes and obtain the vendor charge in writing. Keep both values separate from development labour.

## Main risks and treatment

| Risk | Estimate treatment | Stop or escalation condition |
| --- | --- | --- |
| Resident Select test access arrives late | Track as external wait. Use stubs only for early engineering. | Stop before live use if there is no test Organisation. |
| Search, pagination, nested change detection, or update behaviour differs from documentation | Use the management reserve and configurable adapter controls. | Re-scope if Client search, Prospect creation, or core Workflow Evidence reads are unavailable. |
| Resident Select create has no idempotency key | Include a unique local operation and Creation Outcome Unknown support. | Never retry an ambiguous create automatically. |
| Carebridge transition rules or side effects are more complex than expected | Include domain-command, concurrency, and integration-test work. | Do not bypass the Carebridge state machine or Placed-Elsewhere Cascade. |
| Private staging connectivity is not available | Include conservative dedicated-network work and provider coordination. | Block delivery. Do not expose staging publicly. |
| New Dioxus full-stack, session, and administration paths need more hardening | Include authentication evaluation, browser tests, SSR and hydration tests, and reserve. | Do not release if role isolation or session behaviour is not proven. |
| Privacy or telemetry can expose patient data | Include minimum durable data, redaction review, and test evidence. | Block release if raw payloads or transient unlinked demographics are retained. |
| One engineer becomes the only system expert | Show the long one-engineer window and require review and runbook evidence. | Move to two engineers if the approval date needs the shorter planning window. |

## Scope controls

The estimate includes:

- Integration Platform engineering.
- Carebridge machine APIs, domain commands, display projection, and Provider link-or-create modal.
- The narrow synchronous Carebridge-to-Core linking API.
- Both directional Flows.
- Exceptional Webres support linking.
- Testing, recovery, AWS automation, coordination, acceptance support, and first-Connection onboarding.

The estimate excludes:

- Automatic record discovery or demographic matching.
- Automatic existing-active reconciliation.
- Bulk historical migration or large manual data-cleansing work.
- Carebridge Provider accounts in the Integration Platform.
- Additional Internal Applications, Connected Platforms, Provider Companies, or live Connections.
- ScheduleMee and other unrelated integrations.
- OAuth, mTLS, a new identity provider, SQS, EventBridge, WAF, a VPN product, extra AWS accounts, or a second AWS region.
- A detailed implementation backlog before approval.

## Human review record

Approved without changes on 2026-09-03.

The 240–360 person-day labour envelope, milestone plan, one-engineer and two-engineer scenarios, external-wait treatment, recurring-cost treatment, scope controls, risks, and acceptance gates form the pre-approval baseline. The staffing scenarios remain named alternatives until project approval selects the funded team. Project approval must also name owners for external dependencies. Recurring service and vendor costs remain outside engineering labour.
