# Day 14 — Detailed Notes

## 1. Core Concept

Day 14 focuses on **Linux service troubleshooting, Apache configuration, port conflicts, and systemd service management**.

The overall troubleshooting workflow was:

```text
Monitoring Alert
      ↓
Identify Application Servers
      ↓
Check Apache Service
      ↓
Check Apache Port
      ↓
Read Service Error
      ↓
Identify Conflicting Process
      ↓
Resolve Port Conflict
      ↓
Start Apache
      ↓
Enable Apache
      ↓
Verify All App Servers
```

---

# 2. Why This Task Matters

In a production environment, monitoring systems may report that an application or service is unavailable.

The first response should not be to blindly restart the service.

A better troubleshooting approach is:

```text
Service Failure
      ↓
Understand the Error
      ↓
Identify Root Cause
      ↓
Apply Targeted Fix
      ↓
Verify Recovery
```

This task demonstrates exactly that approach.

---

# 3. Apache HTTP Server

Apache HTTP Server is a widely used web server.

On the application servers, the service is managed through systemd:

```text
httpd.service
```

The common service commands are:

```bash
sudo systemctl start httpd
sudo systemctl stop httpd
sudo systemctl restart httpd
sudo systemctl status httpd
```

---

# 4. `systemctl`

`systemctl` is used to manage services controlled by **systemd**.

### Check status

```bash
sudo systemctl status httpd
```

Shows whether Apache is:

```text
active
inactive
failed
```

### Start service

```bash
sudo systemctl start httpd
```

Starts Apache immediately.

### Stop service

```bash
sudo systemctl stop httpd
```

Stops Apache.

### Restart service

```bash
sudo systemctl restart httpd
```

Stops and starts Apache again.

### Enable service

```bash
sudo systemctl enable httpd
```

Configures Apache to start automatically during boot.

---

# 5. Active vs Enabled

These two concepts are different.

### Active

```bash
sudo systemctl is-active httpd
```

Answers:

> Is Apache running right now?

Expected:

```text
active
```

### Enabled

```bash
sudo systemctl is-enabled httpd
```

Answers:

> Will Apache automatically start during system boot?

Expected:

```text
enabled
```

Therefore, the desired state is:

```text
active + enabled
```

---

# 6. The Initial Apache Failure on `stapp01`

On `stapp01`, Apache initially showed:

```text
Active: failed (Result: exit-code)
```

The important error in the service logs was:

```text
AH00072: make_sock: could not bind to address 0.0.0.0:8083
(98)Address already in use
```

This means another process had already claimed port `8083`.

---

# 7. Understanding "Address Already in Use"

A network port can normally be bound by only one process for the same listening address/port combination.

For example:

```text
Apache
  ↓
0.0.0.0:8083
```

If another process already owns the port:

```text
Sendmail
  ↓
127.0.0.1:8083
```

Apache may fail to bind to the required address/port.

Conceptually:

```text
Port 8083
    |
    +---- Sendmail ❌
    |
    +---- Apache   ❌
```

After removing the conflict:

```text
Port 8083
    |
    +---- Apache   ✅
```

---

# 8. Checking Listening Ports

The command used was:

```bash
sudo ss -lntp | grep 8083
```

`ss` is a utility for inspecting sockets.

Important options:

```text
-l  = listening sockets
-n  = numeric addresses and ports
-t  = TCP sockets
-p  = show process information
```

Therefore:

```bash
sudo ss -lntp
```

shows TCP listening sockets together with their associated processes.

---

# 9. Identifying the Conflicting Process

The output on `stapp01` showed:

```text
127.0.0.1:8083
users:(("sendmail",pid=17560,...))
```

This immediately identified:

```text
Process = sendmail
PID     = 17560
Port    = 8083
```

The process was then inspected:

```bash
sudo ps -fp 17560
```

This confirmed:

```text
sendmail: accepting connections
```

---

# 10. Checking the Sendmail Service

To determine whether Sendmail was managed by systemd:

```bash
sudo systemctl status sendmail --no-pager
```

The service was found to be running.

This is better than simply killing the PID because stopping the service allows systemd to manage the process cleanly.

---

# 11. Stopping the Conflicting Service

The conflict was resolved with:

```bash
sudo systemctl stop sendmail
```

This released port `8083`.

The port was then checked again:

```bash
sudo ss -lntp | grep 8083
```

Once the port was free, Apache could bind to it.

---

# 12. Apache Port Configuration

Apache's configured port was checked with:

```bash
sudo grep -R "^[[:space:]]*Listen" /etc/httpd/conf /etc/httpd/conf.d 2>/dev/null
```

The relevant configuration was:

```text
Listen 8083
```

Therefore, Apache was already configured for the required port.

This was an important observation:

> The Apache configuration did not need to be changed; the actual problem was a port conflict.

---

# 13. Why We Check Configuration Before Editing

A common troubleshooting mistake is to change configuration immediately.

A better workflow is:

```text
Check configuration
      ↓
Check service status
      ↓
Check logs
      ↓
Check ports/processes
      ↓
Only then modify configuration if necessary
```

In this task, Apache already had:

```text
Listen 8083
```

so modifying `httpd.conf` would have been unnecessary.

---

# 14. Starting Apache After Fixing the Conflict

Once Sendmail was stopped:

```bash
sudo systemctl start httpd
```

Then:

```bash
sudo systemctl status httpd --no-pager
```

Apache successfully reported:

```text
Active: active (running)
```

---

# 15. Verifying Apache Is Listening

The final port check was:

```bash
sudo ss -lntp | grep 8083
```

The result showed `httpd` listening on:

```text
*:8083
```

This confirmed that Apache successfully acquired the required port.

---

# 16. Why `*:8083` Matters

An entry such as:

```text
*:8083
```

means the process is listening on port `8083` across the available local interfaces represented by the wildcard address.

This is different from a process listening only on:

```text
127.0.0.1:8083
```

which is restricted to localhost.

For this challenge, the important requirement was that Apache was listening on `8083`.

---

# 17. Enabling Apache

Apache was enabled with:

```bash
sudo systemctl enable httpd
```

This created the systemd relationship required for Apache to start automatically during boot.

Verification:

```bash
sudo systemctl is-enabled httpd
```

Expected:

```text
enabled
```

---

# 18. Why Enabling the Service Matters

Suppose Apache is running now:

```text
active
```

but the server reboots.

If Apache is not enabled, it may not automatically start again.

Therefore:

```text
active
```

is not enough for persistent service availability.

The desired production-style state is:

```text
active
+
enabled
```

---

# 19. Application Server 2

`stapp02` was checked and Apache was already:

```text
active (running)
```

It was listening on:

```text
8083
```

Apache was also enabled to ensure persistence after reboot.

---

# 20. Application Server 3

`stapp03` was checked and Apache was already:

```text
active (running)
```

It was listening on:

```text
8083
```

Apache was also enabled.

---

# 21. The FQDN Warning

Apache displayed a warning similar to:

```text
AH00558: httpd: Could not reliably determine the server's fully qualified domain name
```

This does **not** mean Apache failed.

The service still successfully started:

```text
Started The Apache HTTP Server.
```

and reported:

```text
Server configured, listening on: port 8083
```

Therefore, the warning was not relevant to the task's success criteria.

---

# 22. Service Troubleshooting Method

A useful Linux troubleshooting sequence is:

```text
1. systemctl status
2. Read the error
3. Check service logs
4. Check configuration
5. Check ports
6. Identify conflicting process
7. Apply targeted fix
8. Restart/start service
9. Verify service
10. Verify functionality
```

For this challenge:

```text
systemctl status httpd
        ↓
Address already in use
        ↓
ss -lntp | grep 8083
        ↓
sendmail identified
        ↓
systemctl stop sendmail
        ↓
systemctl start httpd
        ↓
ss -lntp | grep 8083
        ↓
Apache healthy
```

---

# 23. Root Cause vs Symptom

This distinction is important.

### Symptom

```text
Apache service failed
```

### Error

```text
Address already in use
```

### Root Cause

```text
Sendmail was occupying port 8083
```

### Fix

```text
Stop the conflicting Sendmail service
```

### Result

```text
Apache running on port 8083
```

This is a good example of why reading logs is important.

---

# 24. `ps` vs `systemctl`

### `ps`

Used to inspect processes:

```bash
sudo ps -fp 17560
```

Useful when you know a PID and want to understand what process is running.

### `systemctl`

Used to manage services:

```bash
sudo systemctl status sendmail
```

Useful when the process belongs to a systemd-managed service.

Together:

```text
ss → identifies PID
ps → identifies process
systemctl → identifies/manages service
```

---

# 25. Why We Did Not Kill the PID Directly

We could have used a command to terminate PID `17560`, but that would not be the preferred service-management approach.

Instead:

```bash
sudo systemctl stop sendmail
```

was used.

This is cleaner because:

- systemd knows the service has stopped.
- The service is managed through its normal lifecycle.
- It avoids treating the process as an unmanaged process.
- It makes troubleshooting more predictable.

---

# 26. Port Conflict Troubleshooting Pattern

Whenever a service reports:

```text
Address already in use
```

think:

```text
Which port?
       ↓
Which process owns it?
       ↓
Why is that process using it?
       ↓
Can the conflicting service be stopped or reconfigured?
       ↓
Restart the intended service
       ↓
Verify the port
```

Useful command:

```bash
sudo ss -lntp | grep <PORT>
```

---

# 27. Final Verification Commands

Run these on every application server:

```bash
sudo systemctl is-active httpd
sudo systemctl is-enabled httpd
sudo ss -lntp | grep 8083
```

Desired state:

```text
active
enabled
LISTEN ... :8083 ... httpd
```

---

# 28. Common Error — Apache Still Fails

If Apache still fails after stopping a conflicting service:

```bash
sudo systemctl status httpd --no-pager
```

Then check the configuration:

```bash
sudo apachectl configtest
```

or:

```bash
sudo httpd -t
```

Also inspect recent service logs:

```bash
sudo journalctl -u httpd --no-pager -n 50
```

These commands help distinguish configuration errors from runtime problems.

---

# 29. Common Error — Port Still Occupied

If Apache cannot bind to `8083`:

```bash
sudo ss -lntp | grep 8083
```

Identify the process.

Then inspect its PID:

```bash
sudo ps -fp <PID>
```

If it belongs to a systemd service:

```bash
sudo systemctl status <service>
```

Only after understanding the process should you decide whether it should be stopped or reconfigured.

---

# 30. Common Error — Apache Not Enabled

If:

```bash
sudo systemctl is-enabled httpd
```

returns:

```text
disabled
```

enable it:

```bash
sudo systemctl enable httpd
```

Then verify:

```bash
sudo systemctl is-enabled httpd
```

---

# 31. Common Error — Apache Not Listening on 8083

Check the Apache configuration:

```bash
sudo grep -R "^[[:space:]]*Listen" /etc/httpd/conf /etc/httpd/conf.d 2>/dev/null
```

If the required configuration is present:

```text
Listen 8083
```

restart Apache:

```bash
sudo systemctl restart httpd
```

Then:

```bash
sudo ss -lntp | grep 8083
```

---

# 32. Real-World DevOps Applications

The same troubleshooting skills apply to:

### Web Servers

```text
Apache / Nginx
```

### Application Servers

```text
Java / Node.js / Python applications
```

### Databases

```text
MySQL / PostgreSQL
```

### Monitoring Agents

```text
Monitoring service unavailable
```

### Containerized Applications

```text
Port mapping conflicts
```

The general principle remains:

> **Identify the service, inspect the error, identify the resource conflict, fix the root cause, and verify recovery.**

---

# 33. Relationship With Production Support

This task closely resembles a real production support incident:

```text
Monitoring Alert
      ↓
Service unavailable
      ↓
Engineer investigates
      ↓
Logs reveal root cause
      ↓
Port/process inspection
      ↓
Corrective action
      ↓
Service restored
      ↓
Health verification
```

This is a foundational production-support and DevOps workflow.

---

# 34. Security and Operational Lessons

### Avoid unnecessary changes

If Apache already has:

```text
Listen 8083
```

do not modify the configuration without a reason.

### Prefer service management

Use:

```bash
systemctl stop <service>
```

instead of manually killing processes when the process is managed by systemd.

### Verify after every change

After fixing a service:

```text
Check status
Check port
Check process
```

### Distinguish warnings from failures

An `AH00558` FQDN warning did not prevent Apache from starting.

---

# 35. Final Mental Model

Remember Day 14 as:

```text
Apache unavailable
       |
       v
systemctl status httpd
       |
       v
"Address already in use"
       |
       v
ss -lntp | grep 8083
       |
       v
Sendmail owns 8083
       |
       v
systemctl stop sendmail
       |
       v
systemctl start httpd
       |
       v
systemctl enable httpd
       |
       v
Verify :8083
       |
       v
All App Servers Healthy
```

---

# 36. Most Important Takeaways

### `systemctl status`

```bash
sudo systemctl status httpd
```

Checks the current service state and recent service messages.

### `systemctl start`

```bash
sudo systemctl start httpd
```

Starts Apache.

### `systemctl enable`

```bash
sudo systemctl enable httpd
```

Enables Apache at boot.

### `ss`

```bash
sudo ss -lntp | grep 8083
```

Identifies listening sockets and their processes.

### `ps`

```bash
sudo ps -fp <PID>
```

Inspects a process using its PID.

### `grep`

```bash
sudo grep -R "^[[:space:]]*Listen" /etc/httpd/conf /etc/httpd/conf.d 2>/dev/null
```

Finds Apache `Listen` directives.

### `journalctl`

```bash
sudo journalctl -u httpd --no-pager -n 50
```

Displays recent systemd journal entries for Apache.

---

# 37. Final DevOps Lesson

The most important lesson from Day 14 is:

> **Do not troubleshoot from assumptions. Follow the evidence.**

Apache was reported as unavailable, but the correct fix was not to reinstall Apache or rewrite its configuration.

The evidence showed:

```text
Apache configuration → correct
Apache port → 8083
Port 8083 → already occupied
Process → Sendmail
```

Therefore, the targeted fix was:

```bash
sudo systemctl stop sendmail
```

followed by starting and enabling Apache.

This is the mindset required for effective Linux and DevOps troubleshooting.
