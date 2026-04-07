## Understanding the database
- Refer to the `.agents/docs/mysql-schema-summary.md` for a broad overview of the database structure.
- Use `database/schema/mysql-schema.sql` if exact SQL structure is needed. 
- Do not refer to the migration files, unless they aren't listed in the migrations table in the mysql-schema.sql dump. 
  - Any migrations that are in the schema file should always be ignored.

## Before completing a task
- `./vendor/bin/sail` commands cannot be run from inside the sandbox. Seek approval before running them against the local container, and ensure the containers are running first. Start them with `./vendor/bin/sail up -d` if needed.
- Run `just run-ci` before concluding work, unless the task only changes documentation. Again, seek approval before running it.

## Testing
- When writing unit tests, use Factories instead of queries wherever possible.

## Browser checks
- When making UI or email changes, test them in a browser to confirm they look correct.
- When asked to check a page using the browser, log in before checking the page.
- Use the usernames and passwords in `.env` for the available users.
- Use the app URL in `.env` when opening the application.
- After logging in, expect a 2FA prompt. the 2FA code is printed on the page above the 2FA input in the format `Use this code in demo mode: XXXXXX`.
- You can use Mailpit to check emails that were sent.
- The Mailpit URL is `http://localhost:$FORWARD_MAILPIT_DASHBOARD_PORT`, using the `FORWARD_MAILPIT_DASHBOARD_PORT` value from `.env`.

<!-- BEGIN BEADS INTEGRATION v:1 profile:minimal hash:ca08a54f -->
## Beads Issue Tracker

This project uses **bd (beads)** for issue tracking. Run `bd prime` to see full workflow context and commands.

### Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work
bd close <id>         # Complete work
```

### Rules

- Use `bd` for ALL task tracking — do NOT use TodoWrite, TaskCreate, or markdown TODO lists
- Run `bd prime` for detailed command reference and session close protocol
- Use `bd remember` for persistent knowledge — do NOT use MEMORY.md files

## Session Completion

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd dolt push
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds
<!-- END BEADS INTEGRATION -->
