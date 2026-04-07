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
