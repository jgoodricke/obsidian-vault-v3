# Cloud-Hosted ERP MCP Server Pilot

## Objective

Move the client's existing ERP MCP proof of concept from a local workstation into Azure so that it can be used by a small group of approximately 3–4 users through Claude Code.

The implementation will retain the existing read-only and whitelist-based security model while replacing the local DAB/ngrok setup with a centrally hosted HTTPS endpoint protected by Microsoft Entra ID authentication.

The pilot is intended to minimise initial implementation effort and cost while providing a secure and practical shared deployment.

## Existing Proof of Concept

The current proof of concept uses Microsoft's Data API Builder (DAB) SQL MCP Server to provide Claude with controlled, read-only access to ERP data.

The existing implementation already includes several useful controls:

- Only explicitly configured database entities are exposed to Claude.
    
- Create, update and delete MCP tools are disabled.
    
- Database access uses a dedicated read-only account.
    
- REST and GraphQL endpoints are disabled.
    
- Claude accesses the MCP server over HTTP via an ngrok HTTPS tunnel.
    
- The MCP server currently runs on a user's workstation rather than as a shared service.
    

These controls should be retained where applicable in the hosted version.

The client has specifically identified centralised hosting and multi-user access as areas they would like addressed before broader use of the proof of concept.

---

# Proposed Solution

## Architecture

Deploy the existing Data API Builder MCP server as a standalone application in **Azure Container Apps**.

The proposed request flow is:

```text
User workstation
      │
      │ Claude Code
      │
      │ HTTPS + Entra bearer token
      ▼
Azure Container Apps
      │
      │ Microsoft Entra authentication
      ▼
Data API Builder / MCP Server
      │
      │ Read-only database connection
      ▼
Existing cloud database
```

Azure Container Apps provides the public HTTPS endpoint and TLS termination. Microsoft specifically supports securing standalone MCP servers hosted in Container Apps using its built-in Microsoft Entra ID authentication.

The MCP server itself will remain responsible for controlling which database entities Claude can query, while the database credentials will remain restricted to read-only access.

---

# Azure Hosting

## Azure Container Apps

The existing DAB server will be packaged as a container and deployed to Azure Container Apps.

The deployment will include:

- Data API Builder / SQL MCP Server
    
- Existing MCP entity configuration
    
- HTTPS external ingress
    
- Microsoft Entra authentication
    
- Environment-specific database configuration
    
- Database credentials stored as Azure Container Apps secrets rather than embedded in the image or source-controlled configuration
    
- Application logging through Azure
    

Container Apps is suitable for the pilot because it has relatively little infrastructure to manage and can scale down when the service is not being used.

By default, Container Apps can scale to zero replicas. This reduces ongoing compute cost for an MCP server that may only receive occasional requests, although the first request after a period of inactivity can experience a cold start. Microsoft notes that this can typically add approximately 10–30 seconds to the first request.

For this pilot, scale-to-zero is acceptable in exchange for keeping the ongoing hosting cost low.

---

# Database Connection

The hosted MCP server will connect directly to the client's existing cloud-accessible database rather than relying on a database running or accessible only from an individual workstation.

The existing DAB configuration will be reviewed against the cloud database and adjusted as required so that the same intended ERP entities remain available.

The existing principles will be retained:

- Read-only database credentials
    
- Only approved entities exposed through DAB
    
- No create, update or delete MCP tools
    
- No database write functionality
    
- Sensitive or unnecessary entities excluded from the MCP configuration
    

Microsoft Data API Builder supports SQL Server, Azure SQL, PostgreSQL and other supported database platforms, including PostgreSQL-based databases.

Changes to the client's database synchronisation process, database schema or write APIs are outside the scope of this implementation.

---

# Authentication

## Microsoft Entra ID

The Azure Container App will be protected using its built-in Microsoft Entra ID authentication.

Unauthenticated requests will receive an HTTP `401` response and will not reach the MCP server. Microsoft documents this as the supported authentication model for standalone MCP servers hosted in Azure Container Apps.

An Entra application registration will be created for the MCP service.

Because the initial pilot has only a small number of users, access can be granted directly to the individual pilot users rather than introducing a more complex group or role structure.

This provides:

- Individual user authentication
    
- Existing Microsoft account security and MFA where configured
    
- Ability to revoke a single user's access
    
- No shared MCP password
    
- No long-lived API key distributed between users
    
- An Entra sign-in record for each user
    

The database connection will remain separate from user authentication. DAB will continue to connect using its restricted application-level database credential rather than each user authenticating individually against the database.

This avoids introducing unnecessary database permission management for the pilot.

---

# Claude Code Authentication

To minimise initial implementation effort, the pilot will not implement the complete native MCP OAuth browser-login flow.

Instead, each user's workstation will use the **Azure CLI** to obtain a short-lived Microsoft Entra access token.

Microsoft documents token retrieval for a secured Container Apps MCP server using:

```bash
az account get-access-token \
    --resource api://<MCP_APP_ID> \
    --query accessToken \
    -o tsv
```

Claude Code supports a `headersHelper` configuration that can execute a local command when establishing an MCP connection and use its output to populate HTTP headers. Anthropic specifically documents this mechanism for short-lived credentials and internal authentication systems.

A small PowerShell script will therefore obtain the current Entra token and return it to Claude Code as:

```http
Authorization: Bearer <token>
```

The Claude MCP configuration will be similar to:

```json
{
  "mcpServers": {
    "erp": {
      "type": "http",
      "url": "https://<container-app>.azurecontainerapps.io/mcp",
      "headersHelper": "powershell.exe -NoProfile -File C:\\Tools\\erp-mcp\\auth.ps1"
    }
  }
}
```

Claude Code runs the helper when establishing or re-establishing the MCP connection. Current Claude Code versions will also rerun the helper following a `401` or `403` response before retrying the connection, allowing expired access tokens to be refreshed automatically.

### User Workstation Setup

Each pilot user's initial setup will consist of:

1. Install Azure CLI if it is not already installed.
    
2. Sign in using:
    

```bash
az login
```

3. Add the supplied authentication helper script.
    
4. Add the supplied Claude Code MCP configuration.
    
5. Verify the MCP server connects successfully.
    

After the initial Azure login, normal use should not require users to manually obtain or copy access tokens.

Expected workstation setup is approximately **10–20 minutes per user**, assuming their Microsoft account and Azure CLI installation work normally.

---

# Security Model

The resulting solution uses several independent security layers.

```text
Microsoft Entra ID
        │
        │ Controls who may connect
        ▼
Azure Container Apps
        │
        │ HTTPS + authenticated ingress
        ▼
Data API Builder
        │
        │ Controls which entities and MCP tools exist
        ▼
Read-only database account
        │
        │ Prevents modification of source data
        ▼
Database
```

This retains the defence-in-depth principles already present in the client's proof of concept while removing the existing reliance on a temporary public ngrok URL.

### Authentication Boundary

Azure Container Apps will be responsible for authenticating users before requests reach DAB.

For the pilot, DAB does not need to perform a second independent authentication flow because all externally accessible traffic passes through the authenticated Container Apps ingress.

Microsoft's DAB documentation explicitly supports deployments where an upstream trusted service handles authentication before requests reach DAB.

This keeps the configuration simple and allows the existing DAB entity permissions to be reused with minimal change.

---

# Implementation Tasks

## 1. Prepare MCP Server for Hosting

- Review the existing DAB configuration.
    
- Update DAB to an appropriate current supported version.
    
- Validate configured MCP entities against the target cloud database.
    
- Confirm create, update and delete MCP functionality remains disabled.
    
- Package the MCP server and configuration for container deployment.
    
- Move database credentials into environment/secrets configuration.
    

**Estimated effort: 2–3 hours**

## 2. Azure Deployment

- Create the required Azure Container Apps resources.
    
- Deploy the MCP container.
    
- Configure external HTTPS ingress.
    
- Configure database connectivity.
    
- Configure application secrets.
    
- Configure scale-to-zero behaviour.
    
- Confirm `/mcp` is reachable through the Azure endpoint.
    

**Estimated effort: 2–3 hours**

## 3. Authentication

- Create the Microsoft Entra application registration.
    
- Expose the MCP application scope.
    
- Enable Microsoft Entra authentication on the Container App.
    
- Configure unauthenticated requests to return `401`.
    
- Restrict access to the pilot users.
    
- Verify valid and invalid authentication behaviour.
    

**Estimated effort: 1–2 hours**

## 4. Claude Code Authentication Helper

- Create the PowerShell token helper.
    
- Configure Azure CLI token acquisition.
    
- Configure Claude Code `headersHelper`.
    
- Test token renewal and reconnection behaviour.
    
- Create reusable configuration for the pilot users.
    

**Estimated effort: 1–2 hours**

## 5. Testing and Handover Documentation

Test:

- MCP connectivity from outside Azure
    
- Authorised user access
    
- Unauthorised user rejection
    
- Database read operations
    
- Entity restrictions
    
- Write operations remain unavailable
    
- Token renewal
    
- Container restart
    
- Scale-from-zero behaviour
    

Provide short instructions covering:

- Azure CLI installation
    
- Microsoft login
    
- Claude Code MCP configuration
    
- Basic connection troubleshooting
    

**Estimated effort: 1–2 hours**

---

# Estimated Implementation Effort

Assuming the existing MCP configuration can largely be reused and the required Azure, Entra and database access is available:

|Area|Estimated effort|
|---|--:|
|MCP/container preparation|2–3 hours|
|Azure deployment|2–3 hours|
|Entra authentication|1–2 hours|
|Claude Code authentication setup|1–2 hours|
|Testing and documentation|1–2 hours|
|**Total**|**Approximately 1–1.5 engineering days**|

A small amount of contingency may be required if there are restrictions around the client's Microsoft Entra tenant, Azure permissions or network access to the database.

The authentication component itself is expected to represent approximately **3–5 hours** of the implementation.

---

# Out of Scope

The following are not required for the initial pilot:

- Changes to the existing database synchronisation process
    
- Database schema changes
    
- Write access from Claude
    
- API implementation for updating the primary ERP database
    
- Per-user database accounts
    
- Per-user or row-level database permissions
    
- Complex DAB role management
    
- Azure API Management
    
- Private VPN connectivity
    
- Native MCP OAuth discovery/browser authentication
    
- Changes to the client's broader Claude or AI governance arrangements
    

---

# Alternatives Considered

## Native MCP OAuth with Microsoft Entra ID

Claude Code supports OAuth 2.0 authentication for remote HTTP MCP servers, including browser-based authentication, stored credentials, token refresh and preconfigured OAuth clients.

This would provide the cleanest user experience:

```text
Claude Code
    │
    ▼
Microsoft login in browser
    │
    ▼
MCP Server
```

However, implementing the complete MCP OAuth discovery and client-registration flow introduces additional configuration and testing that provides limited benefit for a pilot involving only a few users.

The Azure CLI token helper gives the pilot effectively the same individual Entra authentication boundary with substantially less implementation work.

**Decision:** Not selected for the pilot because of the additional initial setup cost.

**Potential later use:** Good option if the service is eventually distributed to a significantly larger user base and completely seamless onboarding becomes worthwhile.

---

## Azure App Service

Azure App Service can also host DAB and provides mature Microsoft Entra EasyAuth integration.

It would be technically suitable, but Container Apps is a better fit for a small standalone containerised service and provides flexible consumption-based scaling, including scale-to-zero.

**Decision:** Container Apps is preferred because it better matches the low-usage pilot and minimises unnecessary hosting infrastructure.

---

## Azure API Management

Azure API Management could sit in front of the MCP server and provide centralised authentication, rate limiting, logging and API governance.

Architecture:

```text
Claude Code
    │
    ▼
Azure API Management
    │
    ▼
MCP Server
```

This would be useful where an organisation operates several MCP services or needs centralised governance policies.

For one internal read-only MCP endpoint serving only a few users, it introduces additional Azure resources, configuration and implementation effort without solving a problem the pilot currently has.

**Decision:** Not selected because it would over-engineer the initial deployment.

**Potential later use:** Appropriate if the client eventually operates multiple MCP services or requires centralised API/MCP governance.

---

## Shared API Key

A shared API key would appear simpler from the user's perspective because Claude Code can send static authentication headers.

However, Azure's API-key-based MCP authentication is associated with its platform-managed Dynamic Sessions MCP offering rather than arbitrary standalone Container Apps. A standalone DAB deployment would therefore require a custom proxy or authentication layer to implement an opaque shared API key. Microsoft recommends Entra bearer authentication for standalone Container Apps MCP servers.

A shared key would also:

- Provide no individual user identity.
    
- Need to be distributed securely.
    
- Need to be rotated for all users if compromised.
    
- Need to be rotated when a user should lose access.
    

**Decision:** Not selected because Entra authentication is both more secure and simpler to operate using Azure's built-in functionality.

---

## Continue Using ngrok and a Local Workstation

The existing proof of concept runs DAB locally and exposes it through a temporary ngrok tunnel.

This has been useful for validating the concept, but it is not appropriate as the shared pilot architecture because:

- The service depends on a particular workstation being available.
    
- DAB and ngrok must be started manually.
    
- The endpoint can change unless a static ngrok domain is configured.
    
- Authentication currently relies primarily on the obscurity of the endpoint and downstream database restrictions.
    
- Availability depends on the workstation and local internet connection.
    

**Decision:** Not selected because the main purpose of this project is to remove the local-workstation dependency and provide a centrally accessible service.

---

## VPN-Only / Private Network Access

The MCP endpoint could instead be exposed only through a private Azure network or corporate VPN.

This would provide a strong additional network boundary but would require every pilot user's machine to have suitable VPN or private network connectivity.

Given the endpoint will already be HTTPS-only and protected by individual Microsoft Entra authentication, the extra workstation configuration and network infrastructure is not justified for the initial pilot.

**Decision:** Not selected because it increases initial deployment and user setup effort without being necessary for the current scale.

---

# Summary

The recommended pilot deployment is:

```text
Claude Code
      │
      │ Azure CLI-generated Entra token
      │ HTTPS
      ▼
Azure Container Apps
      │
      │ Microsoft Entra authentication
      ▼
Data API Builder MCP Server
      │
      │ Read-only connection
      ▼
Existing cloud database
```

This approach provides a centrally hosted MCP service without introducing substantial new authentication infrastructure.

It retains the strongest parts of the existing proof of concept, including read-only database access and an explicit DAB entity whitelist, while replacing the local workstation and ngrok dependency with a managed Azure HTTPS endpoint.

For a pilot involving only a small number of users, it provides a practical balance between security, implementation effort and ongoing operating cost.

**Indicative implementation effort: approximately 1–1.5 engineering days.**