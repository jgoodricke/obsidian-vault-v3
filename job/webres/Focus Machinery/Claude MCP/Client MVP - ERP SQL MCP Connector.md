_Audit record and implementation guide for connecting Claude Desktop to a read-only copy of the ERP database via Microsoft SQL MCP Server._

# 1\. Purpose and scope

This document records the working architecture used to enable natural-language queries of ERP data through Claude Desktop, and provides reproducible steps for other staff to implement the same setup. It serves two purposes: (a) an audit record of what was deployed, what permissions were granted, and what controls are in place; and (b) a practical handover guide so the same configuration can be rebuilt or extended without re-discovering it.

The system enables read-only, controlled-surface querying of the ERP database via an AI assistant. Users can ask questions in plain English (e.g. "how many orders did we close last quarter?") and receive answers backed by live database results, without writing SQL and without exposing the entire database to the AI.

# 2\. Database environment in scope

**Important.** This proof-of-concept used a TEST database, not production. The connection string, the SQL login, and every entity exposed to Claude all point at the test database only. Production was never touched. This is a deliberate design decision, not an incidental detail.

The test database contains the same schema as production (and similar data, restored from a recent backup), which means the proof-of-concept exercises realistic queries without putting production at risk. If this approach is approved for wider use, the same test-database-only posture should be retained, or revisited explicitly as a separate decision.

Why a test database, not production:

- Query load is unpredictable. An AI assistant will explore tables, run joins, and issue SELECT TOP 1000 queries while it figures out what's useful. Production ERP databases have triggers, locking patterns, and indexes that are not friendly to ad-hoc analytical traffic.
- Data exposure is one-way. Query results pass through Anthropic's API for summarisation. Anything Claude reads has, in effect, left our network. Using a test database means the data leaving is a copy, not the live system of record.
- Mistakes are containable. Even with read-only enforcement, exposing the wrong column or view is a real possibility while iterating on entity definitions. On a test database, the consequence is at most a stale exposure that disappears at the next refresh.
- Compliance surface is smaller. ERP databases hold financial, payroll, and customer PII. Limiting AI access to a test copy reduces the regulatory exposure of the proof-of-concept.

The test database is a restore of a recent production backup. It lives on the same SQL Server instance as production (or a separate instance, if available). Either way, the connection string used by DAB only references the test database. The read-only SQL login is mapped only to the test database, so even if the connection string were altered to point elsewhere, the login would fail to authenticate against any other database. Creating the test database, restoring backups into it, and refreshing it on a regular cadence are standard DBA tasks and are out of scope for this document.

# 3\. Architecture overview

The deployed architecture has four moving parts running on a single user workstation:

- SQL Server hosting the ERP database, accessed by a dedicated read-only login.
- Data API builder (DAB) running locally in HTTP mode, exposing only the database entities defined in dab-config.json.
- ngrok creating a temporary public HTTPS tunnel to the local DAB instance.
- Claude Desktop, configured with a custom MCP connector pointing at the ngrok HTTPS URL.

Request flow when a user asks a question in Claude:

- Claude Desktop sends an MCP tool call to the ngrok HTTPS URL.
- ngrok forwards the request to DAB on localhost:5000.
- DAB validates the request against the configured entities and roles, then constructs deterministic T-SQL via the Data API builder Query Builder.
- T-SQL executes against SQL Server under the read-only login.
- Result rows return to DAB, through ngrok, back to Claude Desktop, where Claude summarises them in natural language.

## 3.1 Why this approach

- Microsoft-published, MIT-licensed, free. No third-party supply-chain risk and no licence cost.
- Whitelist-by-default: Claude can only access database objects explicitly configured as entities. The rest of the database is invisible.
- Deterministic query construction: DAB builds parameterised T-SQL via its own Query Builder. The AI does not write free-form SQL.
- Read-only at multiple layers: DAB's DML tools restricted to read-records only, plus a SQL login that only has db_datareader.
- HTTP transport via ngrok proved stable. The alternative (MCP stdio) had multiple bugs in DAB 1.7.93 and was abandoned.

# 4\. Security posture

**Audit summary.** The system is read-only against a TEST database (never production), and uses a least-privileged SQL login. Data leaves the local network through an HTTPS tunnel to Claude when, and only when, a user actively queries it. No service runs unattended; the connector is brought up and down per-session.

## 4.1 Controls in place

| **Control**                   | **How enforced**                                                                                                                                                                                               |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test database only**        | Connection string points to &lt;<TestDatabase&gt;>, a restored copy. The mcp_readonly login is mapped only to &lt;<TestDatabase&gt;>; it has no rights on the production database. Production is not in scope. |
| **Read-only data access**     | SQL login has only db_datareader role; DAB MCP runtime has only read-records enabled; create-record, update-record, delete-record explicitly set to false.                                                     |
| **Surface restriction**       | Only entities defined in dab-config.json are visible to Claude. Tables and views not added as entities cannot be queried, even though the login could technically read them.                                   |
| **REST and GraphQL disabled** | DAB exposes only the /mcp endpoint. REST and GraphQL endpoints are disabled in runtime configuration so no alternative attack surface exists.                                                                  |
| **Transient public exposure** | ngrok tunnel runs only while the user has the PowerShell window open. When closed, the public HTTPS URL stops resolving. There is no always-on service.                                                        |
| **Credential isolation**      | SQL credentials live only in C:\\Tools\\sql-mcp\\.env on the user's workstation. Not committed to source control, not shared between machines.                                                                 |
| **Audit trail**               | All SQL executed by the read-only login can be captured via SQL Server Audit (recommended, see section 11).                                                                                                    |

## 4.2 Residual risks

- Query results pass through Anthropic's API for natural-language summarisation. Sensitive columns should be excluded at the entity-definition layer (use database views with PII stripped, not raw tables).
- While ngrok is running, the public URL is reachable from the internet. Authentication is by URL obscurity plus the read-only SQL login as the real boundary. Acceptable for development and ad-hoc analytics; not acceptable for unattended production use.
- Local workstation compromise would expose the SQL credentials in .env and the ngrok URL. Mitigation: short rotation cadence on the SQL password, and the principle of least privilege on the login itself.

# 5\. Prerequisites

## 5.1 Database side

- A SQL Server instance reachable from the user's workstation.
- A test database (restored from a recent production backup) created per section 2.2. The connection will reference this database only.
- Mixed Mode authentication enabled on SQL Server (SQL logins permitted, not just Windows auth).
- TCP/IP enabled in SQL Server Configuration Manager.

## 5.2 Workstation side

- Windows 10/11 with PowerShell.
- .NET 9 SDK or later (not just the runtime). Verify with: dotnet --version
- Claude Desktop installed and signed in.
- Internet access for ngrok and Claude.
- An ngrok account (free tier is sufficient).

# 6\. Database setup (handled by the DBA)

The following are arranged with the DBA before the connector is configured. They are standard SQL Server administration tasks and are not detailed here.

- A dedicated SQL login (e.g. mcp_readonly) exists, with db_datareader on the test database only and no rights on production. The login should have explicit DENY SELECT on any tables containing data that should never be queryable by the AI, even though entities not configured in DAB are not visible to Claude. This is defence-in-depth.
- Recommended (not mandatory): expose database views rather than base tables. Views allow PII columns to be excluded, deleted rows filtered, columns renamed to natural-language-friendly names, and row count capped. Grant SELECT on those views to the mcp_readonly login.
- After every test-database refresh, the mcp_readonly user mapping and any view grants are re-applied.
- SQL Server Audit is enabled for the mcp_readonly login so that every query the connector runs is logged. Audit retention and review cadence follow existing organisational policy.

Once these are in place, the workstation-side setup in section 7 onwards can proceed.

# 7\. Workstation setup (one-time)

## 7.1 Install .NET 9 SDK

In a new PowerShell window:

winget install -e --id Microsoft.DotNet.SDK.9

Close and reopen PowerShell, then verify:

dotnet --version

If the version is 9.0.x or higher, proceed.

## 7.2 Install Data API builder

dotnet tool install --global Microsoft.DataApiBuilder

Close and reopen PowerShell, then verify:

dab --version

Expected: Microsoft.DataApiBuilder 1.7.93 or higher.

**PATH note.** If 'dab' is not recognised after install, close and reopen PowerShell. The .NET SDK installer adds %USERPROFILE%\\.dotnet\\tools to PATH, but only new shells see the update.

## 7.3 Install ngrok

winget install -e --id Ngrok.Ngrok

Verify in a new PowerShell window:

ngrok version

## 7.4 Configure ngrok authtoken

- Sign up at <https://dashboard.ngrok.com/signup> (free).
- Copy the authtoken from <https://dashboard.ngrok.com/get-started/your-authtoken>.
- In PowerShell, register the token (one-time):

ngrok config add-authtoken &lt;<YOUR_TOKEN&gt;>

## 7.5 (Recommended) Reserve a static ngrok domain

Free ngrok URLs change every restart, requiring a connector update in Claude each time. Reserving a free static domain avoids this.

- Go to <https://dashboard.ngrok.com/cloud-edge/domains>.
- Click "Create Domain". A free static subdomain is generated, e.g. yourname-erp.ngrok-free.app.
- Note the domain; you will pass it to ngrok at startup.

# 8\. DAB project setup (one-time per user)

## 8.1 Create the project folder

mkdir C:\\Tools\\sql-mcp

cd C:\\Tools\\sql-mcp

## 8.2 Create the .env file

DAB reads the SQL connection string from a file named exactly .env in the working directory. The Database= field must point to the test database, never production. Create the file via PowerShell (Notepad and Explorer struggle with dot-prefixed filenames on Windows):

@"

MSSQL_CONNECTION_STRING=Server=&lt;<SERVER&gt;>,1433;Database=&lt;<TestDatabase&gt;>;User Id=mcp_readonly;Password=&lt;<PASSWORD&gt;>;Encrypt=True;TrustServerCertificate=True

"@ | Out-File -Encoding ASCII -FilePath .env -NoNewline

Verify:

Get-ChildItem -Force | Where-Object Name -like ".env\*"

**Important.** The file must be literally named .env (just the dot-extension). connectsql.env, env.txt, etc. will not be found. Use -Encoding ASCII to avoid BOM markers that confuse DAB's parser. Never commit this file to source control.

## 8.3 Initialise dab-config.json

dab init ^

\--database-type mssql ^

\--connection-string "@env('MSSQL_CONNECTION_STRING')" ^

\--host-mode Development ^

\--config dab-config.json

## 8.4 Lock down dab-config.json

Open dab-config.json. The runtime section should look like this. Modify the rest and graphql blocks to disable them, and ensure the mcp block has only read tools enabled:

"runtime": {

"rest": {

"enabled": false,

"path": "/api",

"request-body-strict": true

},

"graphql": {

"enabled": false,

"path": "/graphql",

"allow-introspection": true

},

"mcp": {

"enabled": true,

"path": "/mcp",

"dml-tools": {

"create-record": false,

"read-records": true,

"update-record": false,

"delete-record": false

}

},

"host": {

"cors": { "origins": \[\], "allow-credentials": false },

"authentication": { "provider": "AppService" },

"mode": "development"

}

}

## 8.5 Validate

dab validate --config dab-config.json

Expected output: "Config is valid." If any error appears, fix before proceeding.

# 9\. Adding entities (initial and ongoing)

Each entity defines a table or view in the test database that Claude can query. Adding entities is the main ongoing maintenance task: as new questions emerge, new entities are added. Removing entities works the same way. Entities can only reference objects in the test database, because the SQL login has access to no other database.

## 9.1 Add a view (recommended)

cd C:\\Tools\\sql-mcp

dab add Orders ^

\--source dbo.vw_OrdersForAi ^

\--source.type view ^

\--source.key-fields OrderId ^

\--permissions "anonymous:read" ^

\--description "Customer orders (last 3 years, deleted excluded). Use for sales analysis and revenue questions."

## 9.2 Add a table

dab add Currencies ^

\--source dbo.Currencies ^

\--source.type table ^

\--permissions "anonymous:read" ^

\--description "Currency codes, names, and exchange rates. Use for FX questions."

## 9.3 Key conventions

- Entity name (the first argument after "dab add") is what Claude sees. Use clear singular or plural English names (Orders, Customer, Products).
- \--source is the SQL object name, fully qualified with schema.
- \--source.type is either table or view. For views, --source.key-fields is mandatory.
- \--description is critical. Claude uses it to decide which entity fits a user's question. Write it like a hint for a new analyst, mentioning what's included, what's excluded, and example questions it answers.
- \--permissions "anonymous:read" gives the default anonymous role read access. This is the correct setting for local stdio/HTTP development mode. Roles and richer permissions become relevant only in multi-user deployment.

## 9.4 Removing or modifying entities

To remove an entity:

dab remove Orders --config dab-config.json

To modify an entity, edit dab-config.json directly (the entities section) or remove and re-add.

## 9.5 After any change

- Run: dab validate --config dab-config.json
- Stop the running DAB process (Ctrl+C in its PowerShell window).
- Restart it (see section 9). ngrok and Claude do not need restarting; only DAB reloads the config at startup.

# 10\. Daily use - starting and using the connector

Three small windows need to be brought up in order. Once running, leave them open for the duration of your Claude session.

## 10.1 Window 1 - start DAB

cd C:\\Tools\\sql-mcp

dab start

Wait for "Now listening on: <http://localhost:5000>" before continuing. Leave this window open.

## 10.2 Window 2 - start ngrok

With reserved static domain (recommended):

ngrok http --domain=&lt;<yourname-erp.ngrok-free.app&gt;> 5000

Without reserved domain (URL will change every restart):

ngrok http 5000

Note the https://...ngrok-free.app URL shown. Leave this window open.

## 10.3 Configure the Claude Desktop connector (one-time, or whenever URL changes)

- Open Claude Desktop.
- Settings → Connectors → Add custom connector.
- Name: ERP MCP (or any name).
- Remote MCP server URL: https://&lt;<your-ngrok-url&gt;>/mcp (note the /mcp at the end).
- Click Add.

If you reserved a static ngrok domain, this connector configuration is one-time. If you use a random ngrok URL, update it each time you restart ngrok.

## 10.4 Use it in a chat

- Open a new chat in Claude Desktop.
- Click the tools icon near the message input.
- Enable the ERP MCP connector for this chat.
- Ask a question, for example: "List the entities available in the ERP MCP connector" or "How many orders did we close last quarter?"

## 10.5 Shutting down

- Close the Claude chat or disable the connector when done.
- Stop ngrok (Ctrl+C in its window) so the public URL stops resolving.
- Stop DAB (Ctrl+C in its window).

# 11\. Audit logging

SQL Server Audit captures every SELECT executed by the mcp_readonly login. The DBA enables a server audit and a database audit specification scoped to that login on the test database, with retention and review cadence following organisational policy.

What to look for when reviewing the audit log: queries against tables that were not exposed as DAB entities (which would suggest an attempt to bypass the configuration), and any unusual access volume.

# 12\. Troubleshooting

| **Symptom**                                                              | **Likely cause and fix**                                                                                                                                                         |
| ------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| dab validate: 'Environmental Variable MSSQL_CONNECTION_STRING not found' | The .env file is missing, misnamed (e.g. connectsql.env), or you ran dab from the wrong directory. Confirm the file is named exactly .env in the same folder as dab-config.json. |
| Login failed for user mcp_readonly                                       | Password or username wrong in .env. Watch for special characters (a literal % is fine; ; " ' or spaces need wrapping in {braces} inside the connection string).                  |
| A network-related or instance-specific error                             | Server name or port wrong, SQL Server unreachable, or TCP/IP not enabled. Verify with SSMS using the same credentials.                                                           |
| Add button greyed out in Claude when entering localhost URL              | Expected. Claude Desktop custom connectors require https://. Use the ngrok URL instead, with /mcp appended.                                                                      |
| HTTP 406 when testing the ngrok URL in a browser                         | Expected. DAB rejects browser-style Accept headers but responds correctly to MCP clients. Not an error.                                                                          |
| Connector shows but Claude says 'I don't have a connector'               | Toggle the connector on for the specific chat via the tools icon next to the message input. Connectors must be enabled per-chat.                                                 |
| Entity added but Claude can't see it                                     | DAB only loads dab-config.json at startup. Restart DAB after every dab add or dab remove.                                                                                        |
| ngrok URL changed and Claude can't connect                               | Free ngrok URLs change at every restart. Either update the connector URL in Claude, or reserve a static domain (section 7.5) and use --domain=...                                |

# 13\. Known limitations of this version

- DAB 1.7.93 MCP stdio mode has multiple bugs (port binding in stdio, log leakage to stdout, post-handshake disconnect). HTTP mode is the only viable transport at present. As DAB matures, stdio may become viable and would remove the need for ngrok.
- Free ngrok tier has a session connection limit and bandwidth caps. Sufficient for ad-hoc analytics; not sufficient for high-volume automated use.
- DAB intentionally does not support NL2SQL (free-form natural-language-to-SQL). Claude can only use the entities you have defined. If a question needs data from an unconfigured entity, the answer is to add the entity, not to lower restrictions.
- Query results pass through Anthropic's API. Treat any column that Claude can see as data that has left your network. Exclude PII at the view-definition layer accordingly.

# 14\. Change log

| **Date** | **Changed by** | **Change**          |
| -------- | -------------- | ------------------- |
|          |                | Initial deployment. |
|          |                |                     |
|          |                |                     |

_Record each entity added, each SQL permission change, and each DAB or ngrok version upgrade in the table above._

# 15\. References

- Microsoft SQL MCP Server overview: learn.microsoft.com/azure/data-api-builder/mcp
- Data API builder source (MIT licensed): github.com/Azure/data-api-builder
- DAB CLI command reference: learn.microsoft.com/azure/data-api-builder/command-line
- ngrok documentation: ngrok.com/docs
- Claude Desktop custom connectors: support.claude.com (search "custom connector")

_Document version 1.0. Update on any architectural change, version upgrade, or entity addition._