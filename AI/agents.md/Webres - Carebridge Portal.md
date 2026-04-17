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

## Authorisation
- Add any new authorisation logic to a policy rather than embedding it inline in controllers, actions, or views.
- Treat policies as the current home for this logic, with the long-term goal of moving authorisation concerns into a clearer DDD architecture over time.

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
