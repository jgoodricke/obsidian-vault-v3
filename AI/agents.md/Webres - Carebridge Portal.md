## Understanding the database
- Refer to the `.agents/docs/mysql-schema-summary.md` for a broad overview of the database structure.
- Use `database/schema/mysql-schema.sql` if exact SQL structure is needed. 
- Do not refer to the migration files, unless they aren't listed in the migrations table in the mysql-schema.sql dump. 
  - Any migrations that are in the schema file should always be ignored.

## Before completing a task
- `./vendor/bin/sail` commands cannot be run from inside the sandbox. Seek approval before running them against the local container, and ensure the containers are running first. Start them with `./vendor/bin/sail up -d` if needed.
- Run `just run-ci` before concluding work, unless the task only changes documentation.

## Testing
- When writing unit tests, use Factories instead of queries wherever possible.
