## Technical Details

The existing load balancer health check can report a healthy status even when the Laravel application is unable to serve requests. The fixed Nginx response at /_infrastructure/health does not check Laravel.

An independent, scheduled application health check will use Laravel's /up endpoint at the following addresses:

- https://staging.carebridgeconnect.com.au/up
- https://staging.bedsearch.com.au/up
- https://portal.carebridgeconnect.com.au/up
- https://portal.bedsearch.com.au/up

Monitoring requirements:

- Checks must run once per minute.
- A successful check requires an HTTP 200 response and a valid TLS certificate.
- Redirects, other HTTP status codes, timeouts, connection errors and certificate errors must be treated as failures.
- Redirects must not be followed.
- Two failures within a three-minute evaluation window must trigger an alert for the affected site.
- Missing check results must be treated as failures under the alarm policy.
- Existing load balancer health checks must remain unchanged and operate independently.
- Alerts must not restart, replace or remove servers.

Application health monitoring does not validate all business functionality.