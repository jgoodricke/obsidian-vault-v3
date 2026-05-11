## Understanding the database

- Refer to the `.agents/docs/mysql-schema-summary.md` for a broad overview of the database structure.
- Use `database/schema/mysql-schema.sql` if exact SQL structure is needed.
- Do not refer to the migration files, unless they aren't listed in the migrations table in the mysql-schema.sql dump.
  - Any migrations that are in the schema file should always be ignored.

## Before completing a task
- Run the following commands before concluding work, unless the task only changes documentation.

```
./vendor/bin/sail exec web yarn fix
./vendor/bin/sail exec web ./vendor/bin/php-cs-fixer fix
./vendor/bin/sail exec web composer lint
./vendor/bin/sail exec web yarn ci
./vendor/bin/sail exec web yarn lint-styles
./vendor/bin/sail exec web yarn test
./vendor/bin/sail exec web composer test
```


## Testing
- When writing unit tests, use Factories instead of queries wherever possible.
- Do not run multiple PHP or Pest test commands in parallel. These test runs share the same database and can reset, migrate, seed, or mutate state underneath each other, which leads to flaky failures and misleading results.
- Always wait for one Pest command to finish completely before starting the next one.

## Authorisation
- Add any new authorisation logic to a policy rather than embedding it inline in controllers, actions, or views.
- Treat policies as the current home for this logic, with the long-term goal of moving authorisation concerns into a clearer DDD architecture over time.

## Blade templates
- When creating Blade templates for emails and notifications, do not pass models to the template.
- Always extract the required fields from the model first and pass those scalar values or simple arrays to the template instead.

## Browser checks
- When making UI or email changes, test them in a browser to confirm they look correct.
- Before doing browser checks, start Sail with `./vendor/bin/sail up`, start Yarn inside the web container with `./vendor/bin/sail exec web yarn dev`, and populate the database with `./vendor/bin/sail artisan migrate:fresh --seed`.
- When asked to check a page using the browser, log in before checking the page.
- Use the usernames and passwords in `.env` for the available users.
- Use the app URL in `.env` when opening the application.
- After logging in, expect a 2FA prompt. the 2FA code is printed on the page above the 2FA input in the format `Use this code in demo mode: XXXXXX`.
- You can use Mailpit to check emails that were sent.
- The Mailpit URL is `http://localhost:$FORWARD_MAILPIT_DASHBOARD_PORT`, using the `FORWARD_MAILPIT_DASHBOARD_PORT` value from `.env`.


## Blocked Commands
Do not run these commands under any circumstances:
- `git push` (all variants including `--force`)
- `git reset --hard`
- `git clean -f` / `git clean -fd`
- `git branch -D`
- `git checkout .` / `git restore .`


<!-- code-review-graph MCP tools -->
## MCP Tools: code-review-graph

**IMPORTANT: This project has a knowledge graph. ALWAYS use the
code-review-graph MCP tools BEFORE using Grep/Glob/Read to explore
the codebase.** The graph is faster, cheaper (fewer tokens), and gives
you structural context (callers, dependents, test coverage) that file
scanning cannot.

### When to use graph tools FIRST

- **Exploring code**: `semantic_search_nodes` or `query_graph` instead of Grep
- **Understanding impact**: `get_impact_radius` instead of manually tracing imports
- **Code review**: `detect_changes` + `get_review_context` instead of reading entire files
- **Finding relationships**: `query_graph` with callers_of/callees_of/imports_of/tests_for
- **Architecture questions**: `get_architecture_overview` + `list_communities`

Fall back to Grep/Glob/Read **only** when the graph doesn't cover what you need.

### Key Tools

| Tool | Use when |
| ------ | ---------- |
| `detect_changes` | Reviewing code changes — gives risk-scored analysis |
| `get_review_context` | Need source snippets for review — token-efficient |
| `get_impact_radius` | Understanding blast radius of a change |
| `get_affected_flows` | Finding which execution paths are impacted |
| `query_graph` | Tracing callers, callees, imports, tests, dependencies |
| `semantic_search_nodes` | Finding functions/classes by name or keyword |
| `get_architecture_overview` | Understanding high-level codebase structure |
| `refactor_tool` | Planning renames, finding dead code |

### Workflow

1. The graph auto-updates on file changes (via hooks).
2. Use `detect_changes` for code review.
3. Use `get_affected_flows` to understand impact.
4. Use `query_graph` pattern="tests_for" to check coverage.
