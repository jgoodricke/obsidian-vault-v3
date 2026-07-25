I've been exploring a way to let me ask natural-language questions about our ERP data using an AI assistant (Claude Desktop), without having to write SQL or wait on report requests. Over the past couple of days I've built a working proof-of-concept and I'd like your input before I take it any further.

  

What I built

  

The setup uses Microsoft SQL MCP Server (a free, MIT-licensed component of Microsoft's Data API builder) as a controlled bridge between Claude and SQL Server. Key design choices:

  

- Connected only to a TEST database (a restore of production). Production was never touched.

- A dedicated read-only SQL login with db_datareader on the test database only.

- Whitelist-by-default: Claude can only see database objects explicitly configured as DAB entities. The rest of the database is invisible.

- Read-only at multiple layers: DAB's create/update/delete tools are disabled in config, and the login itself has no write rights.

- Local-only runtime: DAB runs on my workstation; ngrok provides a temporary HTTPS tunnel only while I'm using it.

  

The attached document is the full audit and implementation record — architecture, security posture, prerequisites, ongoing maintenance (adding entities), troubleshooting, and known limitations.

  

What I'd like from you

  

This is working, but before I use this regularly or suggest others adopt it, I'd value your view on:

  

1. Security posture — Are the controls I've described sufficient for the data class involved? Anything you'd add, remove, or tighten?

2. The ngrok dependency — The current setup relies on a public HTTPS tunnel while in use. Is there a better pattern (e.g. internal hosting, VPN-only access) you'd recommend for ongoing use?

3. Production data flow — Query results pass through Anthropic's API for AI summarisation. Are there organisational or compliance constraints I should be aware of?

4. Multi-user path — If this is useful to more than just me, what would a proper shared deployment look like? Centralised DAB instance?

5. Anything I've missed or should be thinking about that isn't in the document.