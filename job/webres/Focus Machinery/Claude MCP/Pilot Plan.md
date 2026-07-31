# Azure-Hosted ERP MCP Server — Limited Rollout

## Proposal Summary

Webres will move the client's existing Data API Builder (DAB) SQL MCP Server from a local workstation to Azure.

The hosted service will support an initial group of approximately 3–4 users through Claude Desktop. It may remain limited to that group or be expanded later under a separate scope.

The solution will preserve the existing read-only database access and DAB entity allowlist while replacing the workstation and ngrok dependency with a centrally managed Azure service.

## Recommended Solution

The supplied DAB configuration will be deployed to Azure App Service and connected to the existing reporting database.

Users will connect through a Claude Desktop custom connector. Microsoft Entra ID will provide individual browser-based authentication, and only named users will be assigned access.

```text
Rollout user
    │
    │ Claude Desktop
    ▼
Anthropic connector infrastructure
    │
    │ HTTPS + Microsoft Entra OAuth
    ▼
Azure App Service
    │
    │ Authenticated MCP requests
    ▼
Data API Builder / SQL MCP Server
    │
    │ Existing read-only database account
    ▼
Existing test/reporting database
```

Azure App Service is recommended because its authentication integration supports the OAuth discovery flow required by remote MCP clients such as Claude Desktop.

The service will use a small, single-instance deployment.

## Security Controls

The deployment will retain or introduce the following controls:

- A single-tenant Microsoft Entra application restricted to named rollout users.
- Browser-based OAuth authentication for each user, using their existing Microsoft identity.
- HTTPS enforced by Azure App Service.
- App Service ingress restricted to Anthropic's published outbound IP ranges.
- DAB entity access changed from `anonymous:read` to `authenticated:read`.
- Only the entities already present in the supplied DAB configuration exposed.
- Create, update, delete, and execute MCP tools disabled.
- REST and GraphQL endpoints disabled.
- The existing database account retained with read-only access.
- The connection limited to the existing test/reporting database.
- Database and App Service authentication secrets stored in protected App Service configuration.
- The Claude OAuth client secret distributed only through an approved secure channel.
- Azure operational and error logging enabled for support purposes.

Per-user query auditing is not included. Entra and Azure logs will support authentication and operational troubleshooting, but will not provide a formal user-to-query audit trail.

## Scope and Deliverables

### 1. Pre-deployment Validation

- Confirm that the nominated Claude account supports remote custom connectors.
- Confirm the database engine, endpoint, database name, and read-only credential.
- Verify the available network path from Azure App Service to the reporting database.
- Validate the supplied DAB configuration against a pinned DAB 2.0 release.

Database connectivity is currently unverified. Simple firewall or access-list changes are included; private networking or material network redesign is out of scope.

### 2. Azure App Service Deployment

- Create or configure the required App Service resources in the client's Azure environment.
- Deploy a pinned, current stable DAB 2.0 runtime with the supplied entity configuration.
- Configure HTTPS, runtime settings, secrets, and basic operational logging.
- Restrict inbound MCP traffic to Anthropic's published outbound IP ranges.
- Connect DAB to the existing test/reporting database using the existing read-only account.

### 3. Microsoft Entra and MCP OAuth

- Create a single-tenant Entra resource registration for the MCP service.
- Create a separate OAuth client registration for the Claude connector.
- Configure the MCP authorization scope and OAuth discovery metadata.
- Require authentication and reject unauthenticated requests.
- Require user assignment and grant access only to the named rollout users.
- Configure a time-limited OAuth client secret for the individual Claude accounts.
- Update DAB to use App Service identity and `authenticated:read` entity permissions.

The OAuth client ID and secret will be supplied securely to the approved users. The secret authenticates the connector client, while Entra sign-in and server-side user assignment still control access.

### 4. Claude Desktop Setup

- Add and authenticate the custom connector for one nominated Claude Desktop user.
- Verify the complete connection flow from Claude Desktop to DAB.
- Provide concise setup instructions for the remaining approved users.

Hands-on setup for additional users can be handled through the existing support arrangement.

### 5. Technical Verification and Handover

Verify that:

- The MCP endpoint is reachable through Claude Desktop.
- An assigned user can complete Entra authentication.
- An unassigned or unauthenticated user is rejected.
- The supplied entities can be discovered.
- A representative read operation reaches the reporting database.
- Create, update, delete, and execute tools are unavailable.
- REST and GraphQL remain disabled.
- The service recovers after an App Service restart.
- Anthropic IP restrictions are active.

Provide a short handover covering the Azure resources, secrets, OAuth registrations, user assignment, connector setup, and basic troubleshooting.

Business-query accuracy, reporting logic, and ERP entity design are the client's responsibility and are not acceptance criteria for this work.

## Responsibilities and Assumptions

Webres already administers the client's Azure and Entra environments and will perform the required cloud and identity configuration.

The estimate assumes:

- The supplied DAB entity configuration can be deployed without redesign.
- Only compatibility and authentication changes are required.
- The existing test/reporting database remains the target.
- The existing database credential remains valid and read-only.
- The database is reachable through a straightforward Azure network path.
- Each rollout user has an eligible Claude account and an identity in the client's Entra tenant.
- The users can receive the OAuth client details through an approved secure channel.

If a material assumption is incorrect, Webres will confirm the effect on scope and effort before proceeding with additional work.

## Estimated Effort and Costs

| Area | Indicative effort |
| --- | ---: |
| DAB 2.0 preparation and compatibility | 2–3 hours |
| App Service deployment and database connectivity | 3–4 hours |
| Entra ID and native MCP OAuth | 4–6 hours |
| Verification, one-user setup, and documentation | 3–5 hours |
| **Total** | **Approximately 2–3 engineering days** |

The estimate includes a modest allowance for ordinary deployment, OAuth, and configuration issues.

It excludes delays waiting for access and material work involving private networking, tenant restrictions, database changes, or custom OAuth middleware.

Recurring Azure consumption charges are not included. The App Service cost will be confirmed from the client's subscription, region, network requirements, and any suitable existing App Service Plan before deployment.

## Out of Scope

- Changes to DAB entities, relationships, descriptions, or reporting logic.
- New reporting views or database schema changes.
- Business-query design, validation, or acceptance testing.
- Changes to the database synchronization process.
- Database write access or write APIs.
- Replacing the existing database-wide read account with view-level grants.
- Private endpoints, VPNs, or material Azure network redesign.
- Per-user query auditing or formal audit-log retention.
- Hands-on onboarding for more than one nominated Claude Desktop user.
- High availability, disaster recovery, load testing, or an uptime contingencies.
- Azure API Management or a custom authentication gateway.
- A custom domain or certificate.
- Recurring Azure charges.
- Ongoing support, which remains covered by the existing support arrangement.

## Constraints and Risks

- Claude custom connectors are currently a beta feature and may change over time.
- Database connectivity from App Service must be confirmed before deployment.
- OAuth compatibility will be validated early because it is essential to the Claude Desktop design.
- Individual personal Claude accounts require the shared OAuth client details to be entered separately.
- The existing database account can read more objects than DAB exposes. The DAB entity allowlist remains the primary surface restriction for this rollout.
- Upgrading DAB may require minor configuration changes, but entity redesign is excluded.

## Next Steps

1. Approve this proposal and the indicative 2–3-day effort range.
2. Nominate the first Claude Desktop user and confirm their connector eligibility.
3. Confirm the reporting database endpoint and read-only credential.
4. Select or approve the Azure App Service Plan and recurring cost.
5. Schedule the deployment and technical verification.

## Appendix A — Technical Design

### A.1 Request and Data Flow

Claude Desktop remote connectors are brokered through Anthropic's cloud infrastructure. Requests do not originate directly from the user's workstation.

The App Service endpoint must therefore be publicly addressable to Anthropic while remaining protected by Entra OAuth and IP restrictions.

For each MCP operation:

1. The user asks Claude Desktop a question.
2. Claude invokes the configured remote MCP tool through Anthropic's connector infrastructure.
3. Anthropic presents the user's Entra access token to Azure App Service.
4. App Service validates the token and forwards the authenticated request to DAB.
5. DAB checks its tool and entity permissions.
6. DAB queries the reporting database using the existing read-only account.
7. Results return through DAB and Anthropic to Claude for summarisation.

### A.2 Azure App Service

DAB will be deployed using Microsoft's App Service deployment pattern and pinned to a tested DAB 2.0 release.

The deployment package will contain the DAB runtime and configuration, but no database or OAuth secrets.

App Service will provide:

- Managed HTTPS and TLS termination.
- Microsoft Entra authentication through App Service Authentication.
- OAuth protected-resource metadata for MCP client discovery.
- A stable Azure hostname for the custom connector.
- Protected runtime configuration for secrets.
- Basic application and authentication logs.

The rollout will use one small instance. App Service does not provide the scale-to-zero behavior described in the earlier Container Apps design.

### A.3 Microsoft Entra OAuth

The OAuth design uses two single-tenant registrations.

The resource registration represents the MCP API. It exposes the scope requested by Claude and is used by App Service to validate access tokens.

The client registration represents Claude as the OAuth client. Its redirect URI will use Claude's documented MCP callback URL.

The client registration will use a time-limited client secret because Microsoft Entra does not provide MCP dynamic client registration.

The Enterprise Application will require assignment. Only the named rollout users will be assigned, preventing other tenant users from obtaining access merely because they can authenticate.

The OAuth client will be pre-authorised or consented to the MCP scope as required by the client's Entra policies.

App Service will return `401 Unauthorized` for unauthenticated requests and publish the protected-resource metadata required for Claude's OAuth discovery.

### A.4 DAB Runtime Configuration

The supplied entity list will remain unchanged unless a compatibility issue prevents deployment.

Required hosting and security changes are:

- Upgrade and pin a current stable DAB 2.0 version.
- Set the DAB authentication provider to `AppService`.
- Change configured entity permissions from `anonymous:read` to `authenticated:read`.
- Keep the MCP `read-records` tool enabled.
- Disable MCP create, update, delete, and execute tools.
- Keep REST and GraphQL disabled.
- Load the database connection string from an App Service setting.

DAB will continue to generate deterministic database queries through its entity abstraction. It will not accept arbitrary SQL from Claude.

### A.5 Network Controls

App Service access restrictions will allow Anthropic's current published outbound IP ranges. Temporary Webres access may be enabled during deployment and removed after verification.

The outbound connection from App Service to the reporting database will use the database's approved Azure endpoint and firewall rules.

If the database requires a private endpoint, VPN, VNet redesign, or cross-tenant network work, that work will be estimated separately.

### A.6 Secrets

The database connection string and App Service authentication secret will be held in protected App Service configuration.

The Claude OAuth client secret will be held in Entra and supplied to approved users through a secure channel for entry into their connector settings.

No secret values will be included in source-controlled files or deployment packages.

The OAuth client secret will have a defined expiry. Its storage and future rotation will be recorded in the handover for management under the existing support arrangement.

### A.7 Logging

App Service authentication and application logs will support deployment checks and fault diagnosis. DAB logging will be set to an operational level suitable for the limited rollout.

The deployment will not intentionally log returned database rows. Formal per-user query attribution, audit retention, and audit review are outside scope.

### A.8 Verification Checklist

- DAB starts successfully on the pinned version.
- The supplied configuration validates.
- The App Service endpoint enforces HTTPS.
- MCP OAuth discovery metadata is available.
- An assigned user completes the Claude-to-Entra OAuth flow.
- Unassigned and unauthenticated requests are rejected.
- App Service forwards an authenticated identity to DAB.
- DAB evaluates the request as `authenticated`.
- The configured entity list is visible through MCP.
- A read request succeeds against the reporting database.
- Create, update, delete, and execute tools are absent.
- REST and GraphQL endpoints are disabled.
- Database and OAuth secrets are absent from deployment files.
- Anthropic IP restrictions are enabled.
- The service reconnects after an App Service restart.

### A.9 References

- [Deploy Data API Builder to Azure App Service](https://learn.microsoft.com/azure/data-api-builder/deployment/azure-app-service)
- [Configure App Service authentication for Data API Builder](https://learn.microsoft.com/azure/data-api-builder/concept/security/authenticate-easy-auth)
- [Configure MCP server authorization in Azure App Service](https://learn.microsoft.com/azure/app-service/configure-authentication-mcp)
- [Model Context Protocol authorization specification](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization)
- [Claude custom connectors using remote MCP](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp)
- [Anthropic IP addresses](https://platform.claude.com/docs/en/api/ip-addresses)
- [SQL MCP Server overview](https://learn.microsoft.com/azure/data-api-builder/mcp/overview)
