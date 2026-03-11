# 1. Event-driven scanning using AWS services

**Overview**

- User uploads file to **S3**
- S3 event triggers:
    - Lambda function, or
    - ECS worker
- Worker downloads file
- Runs **ClamAV**
- Updates scan result

**Typical setup**

- S3
- Lambda or container worker
- Event notifications
- DB status update

**Pros**
 - Very sec
 - Highly scalable
- Offloads work from application servers
- Works well with serverless systems

**Cons**

- More AWS complexity
- Harder to debug
- Lambda cold starts and file size limits
- Infrastructure setup required

**Best suited for**

- High-scale systems
- Large file volumes
- Event-driven architectures

---

# 2. Asynchronous scanning with queue worker (recommended pattern)

**Overview**

- User uploads file
- Application stores file (usually **S3 or local storage**)
- File marked as `scan_status = pending`
- Queue job triggered
- Worker runs **ClamAV scan**
- File marked `clean` or `infected`
- Only clean files become accessible

**Typical setup**

- Laravel Queue worker
- ClamAV container
- Storage in S3
- File status stored in DB

**Pros**

- Uploads remain fast
- Scales well with queue workers
- Handles large files safely
- Simple to integrate with Laravel

**Cons**

- Slightly more complex architecture
- Requires queue system
- File temporarily exists before scan

**Best suited for**

- Most SaaS applications
- Medium to high upload volumes
- Cloud architectures

---

# 3. Inline scanning during upload (synchronous)

**Overview**
- User uploads file to application
- Application sends file to **ClamAV daemon**
- File is scanned immediately
- If clean → stored
- If infected → rejected

**Typical setup**
- ClamAV running in a container
- Laravel sends file to `clamd`
- Scan occurs during request lifecycle

**Pros**
- Simple architecture
- Immediate scan result
- Easy to implement
- No background infrastructure required

**Cons**
- Slows down upload requests
- Poor scalability for large files
- Can cause request timeouts
- Increased load on web servers

**Best suited for**
- Small apps
- Low upload volumes
- Small file sizes

---

# 4. Managed malware scanning services

**Overview**

- Files scanned by third-party security services

Examples:

- Trend Micro Cloud One
- Sophos
- Cloudflare malware scanning

**Pros**

- Minimal engineering effort
- Enterprise-grade detection
- Vendor maintained

**Cons**

- Expensive
- Vendor lock-in
- Often unnecessary for typical SaaS apps

**Best suited for**

- Large enterprise environments
- Compliance-heavy industries


