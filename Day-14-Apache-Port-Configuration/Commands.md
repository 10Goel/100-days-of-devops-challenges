# Day 14 — Commands Reference

This file documents the important commands used during the Day 14 Apache troubleshooting challenge and explains **what each command does, why it is used, and where it fits into the troubleshooting workflow**.

---

# 1. SSH Into an Application Server

```bash
ssh <app-server-user>@stapp01
```

## Purpose

Connects to Application Server 1.

General syntax:

```bash
ssh USER@HOST
```

For this challenge, the three application servers were:

```text
stapp01
stapp02
stapp03
```

---

# 2. Check Apache Service Status

```bash
sudo systemctl status httpd
```

## Purpose

Displays the current state of the Apache HTTP Server.

Important states include:

```text
active (running)
inactive (dead)
failed
```

On `stapp01`, this command initially showed:

```text
Active: failed (Result: exit-code)
```

This indicated that Apache was not running successfully.

---

# 3. Check Apache Status Without Paging

```bash
sudo systemctl status httpd --no-pager
```

## Purpose

Displays the service status directly in the terminal without opening the pager.

Useful for quick troubleshooting and screenshots.

---

# 4. Check Whether Apache Is Active

```bash
sudo systemctl is-active httpd
```

## Purpose

Returns the current active state of Apache.

Expected result:

```text
active
```

This is a concise health check.

---

# 5. Check Whether Apache Is Enabled

```bash
sudo systemctl is-enabled httpd
```

## Purpose

Checks whether Apache is configured to start automatically during boot.

Expected:

```text
enabled
```

---

# 6. Start Apache

```bash
sudo systemctl start httpd
```

## Purpose

Starts the Apache HTTP Server immediately.

Used after the port conflict on `stapp01` was resolved.

---

# 7. Stop Apache

```bash
sudo systemctl stop httpd
```

## Purpose

Stops the Apache service.

This was not required as part of the final fix, but is useful when managing or troubleshooting the service.

---

# 8. Restart Apache

```bash
sudo systemctl restart httpd
```

## Purpose

Stops and starts Apache again.

Useful after making a valid Apache configuration change.

---

# 9. Enable Apache

```bash
sudo systemctl enable httpd
```

## Purpose

Configures Apache to start automatically when the server boots.

This was performed on the application servers to ensure persistent service availability.

---

# 10. Check Listening Port 8083

```bash
sudo ss -lntp | grep 8083
```

## Purpose

Checks whether port `8083` is currently being used and identifies the process listening on it.

Important `ss` options:

```text
-l = listening
-n = numeric
-t = TCP
-p = process information
```

---

# 11. Identify the Process Using Port 8083

On `stapp01`, the command returned a process similar to:

```text
127.0.0.1:8083
users:(("sendmail",pid=17560,...))
```

This identified:

```text
Process = sendmail
PID     = 17560
Port    = 8083
```

This was the root cause of the Apache startup failure.

---

# 12. Inspect a Process by PID

```bash
sudo ps -fp 17560
```

## Purpose

Displays detailed information about the process with PID `17560`.

Example:

```text
UID    PID    PPID    CMD
root   17560  ...     sendmail: accepting connections
```

General syntax:

```bash
ps -fp <PID>
```

---

# 13. Check Sendmail Service

```bash
sudo systemctl status sendmail --no-pager
```

## Purpose

Determines whether the process belongs to a systemd-managed Sendmail service.

In this challenge, Sendmail was confirmed to be running.

---

# 14. Stop the Conflicting Sendmail Service

```bash
sudo systemctl stop sendmail
```

## Purpose

Stops Sendmail cleanly through systemd and releases the port it was using.

This was the key corrective action on `stapp01`.

---

# 15. Verify Port 8083 After Stopping Sendmail

```bash
sudo ss -lntp | grep 8083
```

## Purpose

Confirms that the port has been released and is available for Apache.

Before the fix:

```text
8083 → sendmail
```

After the fix:

```text
8083 → available
```

---

# 16. Check Apache `Listen` Configuration

```bash
sudo grep -R "^[[:space:]]*Listen" /etc/httpd/conf /etc/httpd/conf.d 2>/dev/null
```

## Purpose

Finds Apache `Listen` directives in the main configuration and configuration directory.

The required configuration was:

```text
Listen 8083
```

This confirmed that Apache was already configured for the required port.

---

# 17. Apache Configuration Test

```bash
sudo apachectl configtest
```

## Purpose

Checks whether the Apache configuration syntax is valid.

Expected result:

```text
Syntax OK
```

This is useful when Apache fails to start because of a configuration problem.

---

# 18. Alternative Apache Configuration Test

```bash
sudo httpd -t
```

## Purpose

Performs an Apache configuration syntax test.

It can be used as an alternative to:

```bash
sudo apachectl configtest
```

---

# 19. View Apache Service Logs

```bash
sudo journalctl -u httpd --no-pager -n 50
```

## Purpose

Displays the most recent Apache service logs from the systemd journal.

Useful when:

```text
systemctl status httpd
```

shows a failure.

---

# 20. Read the Port Conflict Error

The important Apache error was:

```text
AH00072: make_sock: could not bind to address 0.0.0.0:8083
(98)Address already in use
```

## Meaning

Apache attempted to listen on port `8083`, but another process already owned the port.

The troubleshooting path was:

```text
Apache
  ↓
bind 8083
  ↓
Address already in use
  ↓
ss -lntp
  ↓
sendmail identified
```

---

# 21. Verify Apache Listening After the Fix

```bash
sudo ss -lntp | grep 8083
```

Expected result should show `httpd`, for example:

```text
LISTEN ... *:8083 ... users:(("httpd",...))
```

This confirms that Apache successfully acquired the required port.

---

# 22. Verify Apache Is Running

```bash
sudo systemctl is-active httpd
```

Expected:

```text
active
```

---

# 23. Verify Apache Is Enabled

```bash
sudo systemctl is-enabled httpd
```

Expected:

```text
enabled
```

---

# 24. Complete Health Check for One App Server

```bash
sudo systemctl is-active httpd
sudo systemctl is-enabled httpd
sudo ss -lntp | grep 8083
```

Expected:

```text
active
enabled
LISTEN ... :8083 ... httpd
```

---

# 25. Check All Application Servers

## `stapp01`

```bash
ssh tony@stapp01

sudo systemctl is-active httpd
sudo systemctl is-enabled httpd
sudo ss -lntp | grep 8083
```

## `stapp02`

```bash
ssh steve@stapp02

sudo systemctl is-active httpd
sudo systemctl is-enabled httpd
sudo ss -lntp | grep 8083
```

## `stapp03`

```bash
ssh banner@stapp03

sudo systemctl is-active httpd
sudo systemctl is-enabled httpd
sudo ss -lntp | grep 8083
```

---

# 26. Check Apache Port Configuration on All Servers

```bash
sudo grep -R "^[[:space:]]*Listen" /etc/httpd/conf /etc/httpd/conf.d 2>/dev/null
```

The desired configuration is:

```text
Listen 8083
```

---

# 27. Check Which Process Owns a Port

General command:

```bash
sudo ss -lntp | grep <PORT>
```

Example:

```bash
sudo ss -lntp | grep 8083
```

This is a very useful command when troubleshooting:

```text
Address already in use
```

---

# 28. Inspect a Service After a Failure

```bash
sudo systemctl status <service> --no-pager
```

Examples:

```bash
sudo systemctl status httpd --no-pager
sudo systemctl status sendmail --no-pager
```

---

# 29. Stop a Conflicting Service

General syntax:

```bash
sudo systemctl stop <service>
```

For this task:

```bash
sudo systemctl stop sendmail
```

Only stop a service after confirming that it is the actual source of the conflict and that stopping it is appropriate.

---

# 30. Start a Service

General syntax:

```bash
sudo systemctl start <service>
```

For Apache:

```bash
sudo systemctl start httpd
```

---

# 31. Enable a Service at Boot

General syntax:

```bash
sudo systemctl enable <service>
```

For Apache:

```bash
sudo systemctl enable httpd
```

---

# 32. Restart a Service

General syntax:

```bash
sudo systemctl restart <service>
```

For Apache:

```bash
sudo systemctl restart httpd
```

Use this when a service needs to reload its configuration or restart after a valid change.

---

# 33. Check Service Logs

General syntax:

```bash
sudo journalctl -u <service> --no-pager -n 50
```

For Apache:

```bash
sudo journalctl -u httpd --no-pager -n 50
```

This is useful when `systemctl status` does not provide enough information.

---

# 34. Check a Specific Process

```bash
sudo ps -fp <PID>
```

Example:

```bash
sudo ps -fp 17560
```

This is useful after `ss` identifies a PID using a required port.

---

# 35. Check Apache Configuration

```bash
sudo grep -R "^[[:space:]]*Listen" /etc/httpd/conf /etc/httpd/conf.d 2>/dev/null
```

Purpose:

```text
Find configured Apache listening ports
```

For this challenge:

```text
Listen 8083
```

was the required configuration.

---

# 36. Verify the Local HTTP Service

Once Apache is running on `8083`, a local test can be performed:

```bash
curl http://localhost:8083
```

The task does not require actual application content.

Therefore, an HTTP response such as:

```text
403 Forbidden
```

or:

```text
404 Not Found
```

does not necessarily indicate failure, provided Apache is running and listening on the correct port.

---

# 37. Final Verification Sequence

Run on each application server:

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

# 38. Complete Troubleshooting Sequence Used in Day 14

```bash
sudo systemctl status httpd --no-pager

sudo ss -lntp | grep 8083

sudo grep -R "^[[:space:]]*Listen" /etc/httpd/conf /etc/httpd/conf.d 2>/dev/null

sudo ps -fp <PID>

sudo systemctl status sendmail --no-pager

sudo systemctl stop sendmail

sudo ss -lntp | grep 8083

sudo systemctl start httpd

sudo systemctl enable httpd

sudo systemctl status httpd --no-pager

sudo ss -lntp | grep 8083
```

---

# 39. Quick Command Cheat Sheet

| Command | Purpose |
|---|---|
| `systemctl status httpd` | Check Apache status |
| `systemctl start httpd` | Start Apache |
| `systemctl stop httpd` | Stop Apache |
| `systemctl restart httpd` | Restart Apache |
| `systemctl enable httpd` | Enable Apache at boot |
| `systemctl is-active httpd` | Check whether Apache is active |
| `systemctl is-enabled httpd` | Check whether Apache is enabled |
| `ss -lntp` | Show listening TCP sockets and processes |
| `ss -lntp \| grep 8083` | Check who is using port 8083 |
| `ps -fp <PID>` | Inspect a process |
| `systemctl status sendmail` | Check Sendmail service |
| `systemctl stop sendmail` | Stop Sendmail |
| `grep -R "Listen"` | Find Apache listening configuration |
| `apachectl configtest` | Test Apache configuration |
| `httpd -t` | Test Apache configuration |
| `journalctl -u httpd` | View Apache service logs |
| `curl http://localhost:8083` | Test local HTTP access |

---

# 40. Final Day 14 Command Flow

```text
Check Apache
     ↓
systemctl status httpd
     ↓
Read error
     ↓
Check port
     ↓
ss -lntp | grep 8083
     ↓
Identify process
     ↓
ps -fp <PID>
     ↓
Check service
     ↓
systemctl status <service>
     ↓
Stop conflicting service
     ↓
systemctl start httpd
     ↓
systemctl enable httpd
     ↓
Verify service
     ↓
Verify port 8083
```

---

# 41. Final Result

All application servers were verified with the required state:

```text
stapp01 → Apache active → 8083 → enabled
stapp02 → Apache active → 8083 → enabled
stapp03 → Apache active → 8083 → enabled
```

**Day 14 completed successfully.**
