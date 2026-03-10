## AWS Infrastructure
- Using Elastic Beanstalk
- Need to SSH into an instance to access the DB. If you want to dump the DB, copy the dump to S3 Bucket.
- We need to run a script to copy the environment variables - this is a bug with Elastic Beanstalk and PHP.hh

## Codebase
- Use command to copy permissions, enums etc to the JavaScript app.
- Role json is stored in AWS folder
- Notification data is stored in the `resources/views/notifications/` directory, under `title` and `content`
- Custom JS router uses the routes from Laravel.f

## Visibility
- Need to set up log retention.

