## Agenda

### MCP Server
- [ ] One adjustment we would make is to use a Claude business plan and use that to manage the MCP connection.
- [ ] That greatly reduces the attack surface, as the MCP server connects to Claudes servers rather than to the client.
- [ ] Also makes setup easier, as they only need to set up the MPC connection once, rather than on each machine.
- [ ] Will require them to get a Claude business subscription, which I don't believe they currently have.
### Grok Bot
- [ ] 

## Action Items
- [ ] Ideally, they would not use a dedicated machine. GrokBots run in the cloud, so if we can connect them directly to the service they are using that eliminates the need for an environment completely. 
- [ ] I would also question using GrokBot rather than OpenAI Dots or Gemini Spark.