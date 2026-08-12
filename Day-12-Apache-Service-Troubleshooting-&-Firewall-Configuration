# Day 12 — Apache Service Troubleshooting & Firewall Configuration

## 📌 Challenge Overview

The monitoring system reported an issue in the **Stratos Datacenter** where the Apache service on **Application Server 1** was not reachable on TCP port **6000**.

The task was to:

- Diagnose why Apache was not reachable on port `6000`
- Determine whether the issue was caused by the service, port usage, firewall, or another configuration problem
- Restore Apache availability on the required port
- Ensure the application is reachable from the **Jump Host**
- Preserve the existing security configuration rather than disabling the firewall
- Validate the solution using:

```bash
curl http://stapp01:6000
```

> ⚠️ **Important:** The existing `index.html` must not be modified.

---

## 🎯 Objective

The intended architecture was:

```text
Jump Host
    |
    | HTTP :6000
    v
Application Server 1 (stapp01)
    |
    +--> Apache/httpd
    |      |
    |      +--> Listen on TCP 6000
    |
    +--> iptables
           |
           +--> Allow TCP/6000
```

The final result should allow the Jump Host to successfully access the application without disabling the server firewall.

---

## 🖥️ Server Details

| Component | Details |
|---|---|
| Datacenter | Stratos Datacenter |
| Application Server | Application Server 1 |
| Hostname | `stapp01` |
| Application User | `tony` |
| Application Port | `6000/tcp` |
| Web Server | Apache HTTP Server (`httpd`) |
| Firewall | `iptables` |
| Test Host | `jump-host` |
| Test User | `thor` |

---

# 🔍 Initial Diagnosis

The first connectivity test was performed from the Jump Host.

### Telnet

```bash
telnet stapp01 6000
```

Result:

```text
Trying 10.244.13.10...
telnet: connect to address 10.244.13.10: No route to host
```

### Ncat

```bash
nc -zv stapp01 6000
```

Result:

```text
Ncat: No route to host.
```

The hostname resolved successfully, so the next step was to inspect the target server rather than modifying DNS or application files.

---

# 🧪 Troubleshooting Process

## 1. Check the Firewall

The system did not have `firewall-cmd` available, so the firewall implementation was investigated.

```bash
sudo iptables -L -n -v
```

The INPUT chain contained a final rejection rule:

```text
REJECT ... reject-with icmp-host-prohibited
```

There was no rule allowing TCP port `6000`.

This explained why external traffic from the Jump Host was being rejected.

---

## 2. Check for an Existing Listener on Port 6000

The following command was used:

```bash
sudo ss -lntp | grep 6000
```

The output showed that **sendmail** was already using port `6000`:

```text
LISTEN ... 127.0.0.1:6000 ... users:(("sendmail",pid=19053,...))
```

This was a critical finding.

Apache was configured to use port `6000`, but another process was already occupying the port.

---

## 3. Check Apache Status

```bash
sudo systemctl status httpd --no-pager
```

Apache was in a failed state.

The service log showed:

```text
Address already in use
AH00072: make_sock: could not bind to address [::]:6000
no listening sockets available, shutting down
```

This confirmed a **port conflict** rather than an Apache configuration syntax problem.

---

# 🔧 Resolution

## 1. Verify the Process Using Port 6000

The PID reported by `ss` was inspected:

```bash
sudo ps -fp 19053
```

The process was:

```text
sendmail: accepting connections
```

The service was also confirmed with:

```bash
sudo systemctl status sendmail --no-pager
```

---

## 2. Stop the Conflicting Service

Sendmail was stopped:

```bash
sudo systemctl stop sendmail
```

The port was then checked again:

```bash
sudo ss -lntp | grep 6000
```

The conflicting listener was no longer present.

---

## 3. Start Apache

Apache was started:

```bash
sudo systemctl start httpd
```

Its status was verified:

```bash
sudo systemctl status httpd --no-pager
```

Apache became:

```text
Active: active (running)
```

The service log confirmed:

```text
Server configured, listening on port 6000
```

The listening socket was then verified:

```bash
sudo ss -lntp | grep 6000
```

Apache was now listening on:

```text
*:6000
```

This is important because Apache was no longer restricted to localhost.

---

## 4. Enable Apache at Boot

The service was enabled so that Apache starts automatically after a reboot:

```bash
sudo systemctl enable httpd
```

Verify:

```bash
sudo systemctl is-enabled httpd
```

Expected:

```text
enabled
```

---

## 5. Allow TCP Port 6000 Through iptables

The firewall was not disabled.

Instead, a specific allow rule was inserted before the existing rejection rule:

```bash
sudo iptables -I INPUT 4 -p tcp --dport 6000 -j ACCEPT
```

Verify the INPUT chain:

```bash
sudo iptables -L INPUT -n -v --line-numbers
```

The resulting configuration should contain an `ACCEPT` rule for:

```text
tcp dpt:6000
```

before the final `REJECT` rule.

---

# ✅ Validation

## Local Test

On `stapp01`:

```bash
curl http://localhost:6000
```

The application responded successfully.

---

## Remote Test

From the Jump Host:

```bash
curl http://stapp01:6000
```

The application responded successfully.

This confirmed that:

- Apache was running
- Apache was listening on port `6000`
- Apache was reachable beyond localhost
- The firewall was permitting TCP/6000
- The application content was accessible without modifying `index.html`

---

# 🏆 Final Result

**Day 12 completed successfully.**

The issue was caused by **two independent problems**:

```text
Problem 1
sendmail was occupying TCP/6000
        ↓
Apache could not bind to the port
        ↓
Stopped sendmail
        ↓
Apache started successfully


Problem 2
iptables rejected incoming traffic
        ↓
Jump Host received "No route to host"
        ↓
Allowed TCP/6000
        ↓
Remote access restored
```

Final state:

```text
Jump Host
   |
   | curl http://stapp01:6000
   v
stapp01
   |
   +--> iptables
   |      └── ACCEPT tcp/6000
   |
   +--> Apache/httpd
          └── LISTEN *:6000
```

The challenge verification returned a **success mark**.

---

# 📚 Concepts Practiced

- Linux service troubleshooting
- Apache HTTP Server
- `systemctl`
- TCP port troubleshooting
- `ss`
- `telnet`
- `nc` / Ncat
- `curl`
- Process identification with `ps`
- Port conflicts
- `iptables`
- Firewall rule ordering
- Service enablement
- Local vs remote connectivity testing
- Root-cause analysis
- Security-conscious troubleshooting

---

# 🔐 Security Considerations

The solution intentionally avoided disabling the firewall.

Instead of:

```bash
sudo systemctl stop iptables
```

or removing firewall protection, only the required application port was permitted:

```bash
sudo iptables -I INPUT 4 -p tcp --dport 6000 -j ACCEPT
```

This follows the principle of making the **minimum necessary security change**.

The existing `index.html` was also left untouched as required by the challenge.

---

# 🧠 Key Takeaways

### `ss`

Use `ss` to determine which process is listening on a port:

```bash
sudo ss -lntp | grep 6000
```

### `ps`

Use `ps` to inspect the process associated with a PID:

```bash
sudo ps -fp <PID>
```

### `systemctl`

Check service status:

```bash
sudo systemctl status httpd
```

Start a service:

```bash
sudo systemctl start httpd
```

Enable a service at boot:

```bash
sudo systemctl enable httpd
```

### `iptables`

Inspect firewall rules:

```bash
sudo iptables -L INPUT -n -v --line-numbers
```

Allow a specific TCP port:

```bash
sudo iptables -I INPUT 4 -p tcp --dport 6000 -j ACCEPT
```

### `curl`

Test the application:

```bash
curl http://stapp01:6000
```

---

## 🚀 Final DevOps Lesson

> **When a service is unreachable, do not assume the service itself is the only problem.**

A reliable troubleshooting workflow is:

```text
Connectivity
    ↓
Port
    ↓
Listening Process
    ↓
Service Status
    ↓
Firewall
    ↓
Local Test
    ↓
Remote Test
```

Day 12 demonstrates an important real-world DevOps skill: **identify the actual root cause, make the smallest safe configuration change, and verify the result end-to-end.**
