# Day 14 — Apache Service Troubleshooting and Port Configuration

## 📌 Challenge Overview

The production support team of **xFusionCorp Industries** reported an Apache service availability issue on one of the application servers in the **Stratos Datacenter**.

The objective was to:

- Identify the faulty application server.
- Ensure Apache HTTP Server is running on **all three application servers**.
- Ensure Apache is listening on **port `8083`** on all application servers.
- Investigate and resolve any service or port conflict preventing Apache from starting.
- Ensure Apache is enabled so that it starts automatically after a reboot.
- Verify the final service and port state.

---

## 🎯 Objective

The required final state was:

```text
                 Stratos Datacenter
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
       stapp01        stapp02        stapp03
          |              |              |
       Apache          Apache          Apache
       RUNNING         RUNNING         RUNNING
          |              |              |
       :8083          :8083          :8083
          |              |              |
       ENABLED         ENABLED        ENABLED
```

No application content was required. The important requirements were service availability and correct port configuration.

---

## 🖥️ Server Details

| Component | Hostname | User | Requirement |
|---|---|---|---|
| Application Server 1 | `stapp01` | `tony` | Apache on `8083` |
| Application Server 2 | `stapp02` | `steve` | Apache on `8083` |
| Application Server 3 | `stapp03` | `banner` | Apache on `8083` |

---

# 🔍 Initial Investigation

## 1. Check Apache on App Server 1

Connect to `stapp01` and inspect the service:

```bash
sudo systemctl status httpd
```

The service was found in a failed state:

```text
Active: failed (Result: exit-code)
```

The Apache journal showed:

```text
AH00072: make_sock: could not bind to address 0.0.0.0:8083
(98)Address already in use
```

This indicated a **port conflict** rather than an Apache configuration error.

---

## 2. Verify Apache Port Configuration

The configured Apache port was checked with:

```bash
sudo grep -R "^[[:space:]]*Listen" /etc/httpd/conf /etc/httpd/conf.d 2>/dev/null
```

Result:

```text
/etc/httpd/conf/httpd.conf:Listen 8083
```

Therefore, Apache was correctly configured to use port `8083`.

---

## 3. Identify the Process Using Port 8083

The listening process was identified using:

```bash
sudo ss -lntp | grep 8083
```

The output showed:

```text
127.0.0.1:8083
users:(("sendmail",pid=17560,...))
```

The process was confirmed with:

```bash
sudo ps -fp 17560
```

and:

```bash
sudo systemctl status sendmail --no-pager
```

This confirmed that **Sendmail was occupying port `8083`** on `stapp01`.

---

# 🛠️ Resolution

## 1. Stop the Conflicting Sendmail Service

Sendmail was stopped to release port `8083`:

```bash
sudo systemctl stop sendmail
```

The port was then checked again:

```bash
sudo ss -lntp | grep 8083
```

The port was available for Apache.

---

## 2. Start Apache

Apache was started:

```bash
sudo systemctl start httpd
```

The service was verified:

```bash
sudo systemctl status httpd --no-pager
```

Result:

```text
Active: active (running)
```

---

## 3. Enable Apache

Apache was enabled to ensure automatic startup after reboot:

```bash
sudo systemctl enable httpd
```

Verification:

```bash
sudo systemctl is-enabled httpd
```

Expected:

```text
enabled
```

---

## 4. Verify Apache Port

The listening port was verified:

```bash
sudo ss -lntp | grep 8083
```

The result showed `httpd` listening on:

```text
*:8083
```

Therefore, Apache successfully obtained the required port.

---

# 🔎 Verification of App Server 2

On `stapp02`, Apache was already running and listening on port `8083`.

The service was also enabled:

```bash
sudo systemctl enable httpd
```

Final state:

```text
Apache:  active (running)
Port:    8083
Enabled: yes
```

---

# 🔎 Verification of App Server 3

On `stapp03`, Apache was already running and listening on port `8083`.

The service was enabled:

```bash
sudo systemctl enable httpd
```

Final state:

```text
Apache:  active (running)
Port:    8083
Enabled: yes
```

---

# ✅ Final Verification

The following checks can be performed on every application server:

```bash
sudo systemctl is-active httpd
sudo systemctl is-enabled httpd
sudo ss -lntp | grep 8083
```

Expected result:

```text
active
enabled
LISTEN ... :8083 ... httpd
```

### Final Status

| Server | Apache Service | Port `8083` | Enabled | Status |
|---|---|---|---|---|
| `stapp01` | ✅ Running | ✅ Listening | ✅ Enabled | Fixed |
| `stapp02` | ✅ Running | ✅ Listening | ✅ Enabled | Healthy |
| `stapp03` | ✅ Running | ✅ Listening | ✅ Enabled | Healthy |

---

# ⚠️ Apache FQDN Warning

During startup, Apache displayed a warning similar to:

```text
AH00558: httpd: Could not reliably determine the server's fully qualified domain name
```

This is a **warning and not a service failure**.

Apache successfully started and was listening on port `8083`, so no FQDN change was required for this challenge.

---

# 📚 Concepts Practiced

- Apache HTTP Server
- Linux systemd services
- `systemctl`
- Service troubleshooting
- Port conflict troubleshooting
- `ss` socket inspection
- Process identification with `ps`
- Apache port configuration
- Service enablement
- Log-based troubleshooting
- Application server health checks
- Basic production support workflow

---

# 🧠 Key Troubleshooting Lesson

A service showing `failed` does not always mean the service configuration itself is broken.

In this task:

```text
Apache failed
      ↓
Check service status
      ↓
Read failure message
      ↓
"Address already in use"
      ↓
Check port 8083
      ↓
Sendmail found using 8083
      ↓
Stop conflicting service
      ↓
Start Apache
      ↓
Verify :8083
```

The important DevOps habit is:

> **Read the error first, identify the actual root cause, then make the smallest required change.**

---

# 🏆 Result

**Day 14 completed successfully.**

The faulty host was identified as **`stapp01`**. Sendmail was occupying the required Apache port `8083`, preventing Apache from starting.

After stopping the conflicting Sendmail service, Apache was started and enabled successfully.

All three application servers now have:

```text
Apache → Running
Port   → 8083
Boot   → Enabled
```

The Day 14 Apache availability and port configuration requirements were successfully satisfied.
