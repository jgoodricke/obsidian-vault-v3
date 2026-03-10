## AWS Infrastructure
- Using Elastic Beanstalk
- Need to SSH into an instance to access the DB. If you want to dump the DB, copy the dump to S3 Bucket.

## Codebase
- Use command to copy permissions, enums etc to the JavaScript app.
- Role json is stored in AWS folder
- Notification data is stored in the `resources/views/notifications/` directory, under `title` and `content`
- Custom JS router uses the routes from Laravel.

## Visibility
- Need to set up log retention.

