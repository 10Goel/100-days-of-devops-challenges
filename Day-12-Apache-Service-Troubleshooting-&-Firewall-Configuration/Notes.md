# Day 12 — Detailed Notes

## 1. Core Concept

Day 12 is a practical Linux and DevOps troubleshooting task involving:

```text
Network Connectivity
        +
TCP Ports
        +
Apache/httpd
        +
Linux Services
        +
Process Management
        +
iptables
        +
Remote Testing
        +
Root-Cause Analysis
```

The overall workflow was:

```text
Monitoring Alert
      ↓
Port Connectivity Test
      ↓
Inspect Target Server
      ↓
Identify Port Conflict
      ↓
Release Port
      ↓
Start Apache
      ↓
Inspect Firewall
      ↓
Allow Required Port
      ↓
Local Test
      ↓
Remote Test
      ↓
Challenge Success
```

---

# 2. Why This Task Matters

In production environments, an application can become unreachable for many different reasons.

For example:

```text
Application stopped
       OR
Port conflict
       OR
Incorrect listener
       OR
Firewall restriction
       OR
Network problem
```

A DevOps engineer should not immediately restart services or disable security controls.

Instead, the problem should be isolated step by step.

Day 12 demonstrates this troubleshooting mindset.

---

# 3. Understanding "No Route to Host"

The initial test returned:

```text
No route to host
```

This does not necessarily mean that the server itself has no network route.

A firewall can reject traffic in a way that causes the client to report:

```text
No route to host
```

Therefore, the correct response is to inspect:

```text
Network
+
Firewall
+
Listening Port
+
Service
```

rather than assuming that routing is broken.

---

# 4. TCP Port 6000

The challenge required Apache to be reachable on:

```text
TCP 6000
```

A TCP port identifies a service endpoint.

Conceptually:

```text
IP Address
    +
Port
    ↓
Network Service
```

For this challenge:

```text
10.244.13.10:6000
```

represented the target application endpoint.

---

# 5. Listening Sockets

The command:

```bash
sudo ss -lntp
```

is extremely useful during service troubleshooting.

Important options:

```text
-l
↓
listening sockets

-n
↓
numeric addresses and ports

-t
↓
TCP

-p
↓
process information
```

Therefore:

```bash
sudo ss -lntp | grep 6000
```

answers:

> "Is anything listening on TCP port 6000, and which process owns it?"

---

# 6. The Initial Port Conflict

The investigation showed:

```text
127.0.0.1:6000
```

was occupied by:

```text
sendmail
```

with PID:

```text
19053
```

This was the first major root cause.

The situation was:

```text
Port 6000
   |
   +--> Sendmail
          |
          +--> Already using the port

Apache
   |
   +--> Wants port 6000
          |
          +--> Cannot bind
```

Two processes cannot normally bind the same address/port combination.

---

# 7. Apache Error: "Address Already in Use"

Apache reported:

```text
Address already in use
AH00072: make_sock: could not bind to address [::]:6000
```

This is a classic port-conflict error.

The important phrase is:

```text
Address already in use
```

When a service cannot start because of this message, immediately investigate which process is using the requested port.

A useful command is:

```bash
sudo ss -lntp | grep <PORT>
```

---

# 8. Why Apache Could Not Start

The failure sequence was:

```text
Apache configured for port 6000
        ↓
Apache attempts to bind to 6000
        ↓
Port already occupied by Sendmail
        ↓
Apache cannot create listening socket
        ↓
Apache exits
```

Therefore, simply restarting Apache would not solve the problem while Sendmail continued to own the port.

This is an important troubleshooting lesson:

> **A failed service restart does not necessarily mean the service itself is broken.**

---

# 9. Process vs Service

A process is a running instance of a program.

A service is a managed application unit, often controlled by `systemd`.

In this challenge:

```text
Service:
sendmail.service

Process:
sendmail
PID 19053
```

The relationship can be inspected with:

```bash
sudo systemctl status sendmail
```

and:

```bash
sudo ps -fp 19053
```

---

# 10. Why `ps -fp` Was Useful

The command:

```bash
sudo ps -fp 19053
```

identified the process associated with the PID reported by `ss`.

This created a reliable chain of evidence:

```text
Port 6000
    ↓
PID 19053
    ↓
sendmail process
    ↓
sendmail.service
```

This is much better than guessing which service might be responsible.

---

# 11. Stopping the Conflicting Service

The conflicting service was stopped using:

```bash
sudo systemctl stop sendmail
```

This released port `6000`.

The port was then checked again:

```bash
sudo ss -lntp | grep 6000
```

After the conflict was removed, Apache could successfully bind to the port.

---

# 12. Starting Apache

Apache was started with:

```bash
sudo systemctl start httpd
```

Its status was verified using:

```bash
sudo systemctl status httpd --no-pager
```

The final service state was:

```text
Active: active (running)
```

---

# 13. `start` vs `enable`

These two commands have different purposes.

### Start

```bash
sudo systemctl start httpd
```

Means:

> Start Apache now.

### Enable

```bash
sudo systemctl enable httpd
```

Means:

> Configure Apache to start automatically during boot.

Therefore, a robust service configuration commonly requires:

```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

when the service should be running now and after future reboots.

---

# 14. Apache Listening on `*:6000`

After the fix:

```bash
sudo ss -lntp | grep 6000
```

showed Apache listening on:

```text
*:6000
```

This is important.

Compare:

```text
127.0.0.1:6000
```

with:

```text
*:6000
```

### `127.0.0.1:6000`

The service is bound to localhost.

Remote hosts cannot normally connect directly to it.

### `*:6000`

The service is listening on the server's available network interfaces, subject to firewall rules.

This was necessary for the Jump Host test.

---

# 15. The Second Root Cause — Firewall

Even after Apache was fixed, the Jump Host still needed permission to reach the port.

The existing `iptables` INPUT chain contained a broad rejection rule:

```text
REJECT ... reject-with icmp-host-prohibited
```

There was no specific rule allowing TCP port `6000`.

Therefore:

```text
Apache
   ↓
Listening correctly
   ↓
Firewall
   ↓
Traffic rejected
   ↓
Remote connection fails
```

---

# 16. Why the Firewall Was Not Disabled

A poor solution would be:

```bash
sudo systemctl stop iptables
```

or:

```bash
sudo iptables -F
```

These approaches remove or weaken existing security controls.

The challenge specifically required Apache to be reachable **without compromising security settings**.

The correct solution was to permit only the required application port.

---

# 17. Targeted iptables Rule

The rule used was:

```bash
sudo iptables -I INPUT 4 -p tcp --dport 6000 -j ACCEPT
```

Conceptually:

```text
Traffic
   ↓
TCP?
   ↓
Destination port 6000?
   ↓
YES
   ↓
ACCEPT
```

Other firewall protections remain in place.

This is an example of a **least-change / targeted firewall modification**.

---

# 18. Understanding `iptables -I`

The `-I` option means:

```text
Insert
```

For example:

```bash
iptables -I INPUT 4 ...
```

means:

```text
Insert this rule into the INPUT chain at position 4.
```

This matters because firewall rules are processed in order.

---

# 19. Why Rule Ordering Matters

Suppose the firewall contains:

```text
Rule 1 → ACCEPT established connections
Rule 2 → ACCEPT ICMP
Rule 3 → ACCEPT loopback
Rule 4 → REJECT everything else
```

If you append an allow rule after Rule 4:

```text
Rule 5 → ACCEPT TCP/6000
```

the earlier rejection may already have matched the traffic.

Therefore, the specific allow rule must be placed before the broad rejection.

This is why:

```bash
sudo iptables -I INPUT 4 -p tcp --dport 6000 -j ACCEPT
```

was appropriate for the observed rule ordering.

---

# 20. Checking Firewall Rule Numbers

Use:

```bash
sudo iptables -L INPUT -n -v --line-numbers
```

The `--line-numbers` option makes it easier to understand rule ordering.

A healthy configuration should show the TCP/6000 `ACCEPT` rule before the broad rejection.

---

# 21. Local vs Remote Testing

A very important troubleshooting concept is separating local and remote tests.

### Local test

```bash
curl http://localhost:6000
```

Tests:

```text
Apache
+
Local socket
+
Application
```

It does not prove that remote clients can reach the service.

### Remote test

From the Jump Host:

```bash
curl http://stapp01:6000
```

Tests:

```text
Network
+
Firewall
+
TCP/6000
+
Apache
+
Application
```

This is why both tests are valuable.

---

# 22. Why `curl` Was the Final Test

The challenge explicitly provided:

```bash
curl http://stapp01:6000
```

This is stronger than only testing the TCP connection.

A TCP test proves:

```text
Port reachable
```

`curl` proves:

```text
Port reachable
+
HTTP service responding
```

Therefore, the final `curl` test is an application-level validation.

---

# 23. Troubleshooting Method Used

The challenge can be remembered as:

```text
CONNECT
   ↓
LISTEN
   ↓
PROCESS
   ↓
SERVICE
   ↓
FIREWALL
   ↓
LOCAL TEST
   ↓
REMOTE TEST
```

More specifically:

```text
1. Test port from Jump Host
2. SSH into target
3. Inspect listening port
4. Identify owning process
5. Check service status
6. Remove port conflict
7. Start service
8. Verify listener
9. Inspect firewall
10. Add targeted firewall rule
11. Test locally
12. Test remotely
```

---

# 24. Common Mistake — Restarting Apache Repeatedly

A common mistake is:

```bash
sudo systemctl restart httpd
```

again and again.

If the error says:

```text
Address already in use
```

the correct next question is:

> "Who is using the port?"

Use:

```bash
sudo ss -lntp | grep 6000
```

Troubleshooting should be evidence-driven.

---

# 25. Common Mistake — Disabling the Firewall

Another common mistake is:

```bash
sudo systemctl stop iptables
```

This may make connectivity work, but it is not a proper production fix.

Instead:

```text
Identify required port
        ↓
Allow only required traffic
        ↓
Keep firewall protection active
```

---

# 26. Common Mistake — Changing the Application Port

The challenge required:

```text
TCP 6000
```

Changing Apache to another port would not satisfy the requirement.

The correct approach was to make Apache work on the existing required port.

---

# 27. Common Mistake — Modifying `index.html`

The task explicitly warned against modifying the existing application page.

Troubleshooting the service should not involve changing application content.

The correct approach was:

```text
Service configuration
+
Process management
+
Firewall
```

rather than modifying:

```text
index.html
```

---

# 28. Why the Solution Had Two Root Causes

It is important to recognize that fixing only one problem would not have been enough.

### If Sendmail remained active

```text
Port 6000
   ↓
Sendmail owns port
   ↓
Apache cannot start
```

### If only Sendmail was stopped

```text
Apache starts
   ↓
Firewall rejects remote traffic
   ↓
Jump Host still cannot connect
```

### Complete solution

```text
Stop Sendmail
       +
Start Apache
       +
Allow TCP/6000
       ↓
Remote application access
```

This is a good example of **multiple contributing failures**.

---

# 29. DevOps Principle — Root Cause Before Change

A useful production troubleshooting rule is:

> **Measure first, change second.**

The investigation followed:

```text
Observed symptom
      ↓
Connectivity test
      ↓
Evidence
      ↓
Process identification
      ↓
Service verification
      ↓
Firewall inspection
      ↓
Targeted fix
```

This minimizes unnecessary changes.

---

# 30. DevOps Principle — Least Privilege / Least Change

The firewall was not disabled.

Only the required traffic was allowed:

```text
TCP
Port 6000
```

This follows the broader principle:

> Make the smallest change necessary to restore the required functionality.

---

# 31. Useful Commands to Remember

### Check listening ports

```bash
sudo ss -lntp
```

### Find a specific port

```bash
sudo ss -lntp | grep 6000
```

### Inspect a process

```bash
sudo ps -fp <PID>
```

### Check a service

```bash
sudo systemctl status <service>
```

### Start a service

```bash
sudo systemctl start <service>
```

### Stop a service

```bash
sudo systemctl stop <service>
```

### Enable at boot

```bash
sudo systemctl enable <service>
```

### Check firewall

```bash
sudo iptables -L -n -v
```

### Show rule order

```bash
sudo iptables -L INPUT -n -v --line-numbers
```

### Test HTTP

```bash
curl http://host:port
```

---

# 32. Real-World Applications

The same troubleshooting approach can be used for:

### Nginx

```text
nginx
  ↓
TCP/80 or TCP/443
  ↓
Firewall
```

### SSH

```text
sshd
  ↓
TCP/22
  ↓
Firewall
```

### Database

```text
MySQL/PostgreSQL
  ↓
Database Port
  ↓
Firewall
```

### Application Servers

```text
Application
  ↓
Application Port
  ↓
Load Balancer
  ↓
Firewall
```

---

# 33. Relationship With Previous DevOps Skills

Day 12 builds on foundational Linux knowledge:

```text
Linux Commands
      +
Processes
      +
Services
      +
Networking
      +
Firewall
      +
Remote Access
```

The key progression is from simply knowing commands to using them as part of a structured troubleshooting process.

---

# 34. Final Mental Model

Remember the Day 12 incident as:

```text
MONITORING ALERT
      |
      v
"Port 6000 unreachable"
      |
      v
JUMP HOST TEST
      |
      v
"No route to host"
      |
      v
CHECK TARGET
      |
      +----------------------+
      |                      |
      v                      v
PORT CHECK              FIREWALL CHECK
      |                      |
      v                      v
sendmail:6000            REJECT rule
      |                      |
      v                      v
STOP SENDMAIL           ALLOW TCP/6000
      |                      |
      +----------+-----------+
                 |
                 v
          START APACHE
                 |
                 v
           LISTEN *:6000
                 |
                 v
          LOCAL CURL TEST
                 |
                 v
          REMOTE CURL TEST
                 |
                 v
              SUCCESS
```

---

# 35. Most Important Takeaways

### 1. `ss` is essential for port troubleshooting

```bash
sudo ss -lntp | grep 6000
```

It tells you both the listener and the owning process.

### 2. "Address already in use" usually means a port conflict

Find the process before changing the application configuration.

### 3. Service status and process state are different pieces of evidence

Use:

```bash
systemctl status
```

and:

```bash
ps
```

together when necessary.

### 4. Firewall rules are order-sensitive

A broad `REJECT` can prevent a later `ACCEPT` from ever being reached.

### 5. Local success does not prove remote success

Always perform an end-to-end remote test when the problem involves network accessibility.

### 6. Do not disable security controls to solve a connectivity problem

Prefer a targeted firewall rule.

---

# 36. Final DevOps Lesson

The most important lesson from Day 12 is:

> **A good DevOps engineer does not simply make the service work — they determine why it failed, fix the smallest necessary component, preserve security controls, and verify the complete path.**

The final troubleshooting pattern is:

```text
Observe
  ↓
Test
  ↓
Identify
  ↓
Fix
  ↓
Verify
```

That pattern is applicable far beyond this challenge and is one of the most useful habits in production operations.
