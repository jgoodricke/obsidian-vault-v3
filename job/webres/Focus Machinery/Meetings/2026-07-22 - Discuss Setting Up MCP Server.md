## Notes
### MCP Server
- Has an MCP server connected to Claude Code desktop app.
- Wants to move that MCP server into the cloud so it can be accessed from multiple devices.
- Long-term plan is to roll it out to the whole company using Lovable.
### Database
- Currently in Supabase, being synced with the main database using some sort of cron job (he referred to them as "actions").
- One-way sync, so no concern about data integrity.
- Any database updates will be done on the main DB through API calls, currently not implemented.

