# Day 12 — Commands Reference

This document contains the important commands used during the Day 12 Apache troubleshooting challenge, including the purpose of each command, expected behavior, and where it fits into the troubleshooting workflow.

---

# 1. Connect to Application Server 1

```bash
ssh tony@stapp01
```

## Purpose

Connects to Application Server 1, where the Apache service and firewall configuration were investigated.

General syntax:

```bash
ssh USER@HOST
```

---

# 2. Test TCP Connectivity with Telnet

From the Jump Host:

```bash
telnet stapp01 6000
```

## Purpose

Tests whether TCP port `6000` on `stapp01` is reachable.

Initial result:

```text
Trying 10.244.13.10...
telnet: connect to address 10.244.13.10: No route to host
```

This indicates that traffic is not successfully reaching the application port.

---

# 3. Test TCP Connectivity with Ncat

If Telnet is unavailable or another TCP test is preferred:

```bash
nc -zv stapp01 6000
```

or:

```bash
ncat -zv stapp01 6000
```

## Purpose

Performs a TCP connectivity test without sending application data.

Initial result:

```text
Ncat: No route to host.
```

---

# 4. Check iptables Rules

On `stapp01`:

```bash
sudo iptables -L -n -v
```

## Purpose

Displays the active IPv4 firewall rules.

Important finding:

```text
REJECT ... reject-with icmp-host-prohibited
```

The INPUT chain did not contain an allow rule for TCP port `6000`.

---

# 5. Check Whether nftables Is Available

```bash
sudo nft list ruleset
```

## Purpose

Checks whether nftables is being used.

In this challenge, the command returned:

```text
sudo: nft: command not found
```

Therefore, nftables was not the firewall interface being used.

---

# 6. Check Whether firewalld Is Available

```bash
sudo firewall-cmd --state
```

## Purpose

Checks whether `firewalld` is available and active.

In this challenge, `firewall-cmd` was not available.

The active firewall was therefore investigated through `iptables`.

---

# 7. Check Listening Ports

```bash
sudo ss -lntp | grep 6000
```

## Purpose

Determines whether anything is listening on TCP port `6000` and identifies the associated process.

Initial finding:

```text
127.0.0.1:6000
users:(("sendmail",pid=19053,...))
```

This showed that Sendmail was occupying the required port.

After the fix, the result showed Apache:

```text
*:6000
users:(("httpd",...))
```

---

# 8. Check Apache Status

```bash
sudo systemctl status httpd --no-pager
```

## Purpose

Checks whether Apache is running, stopped, or failed.

Initial state:

```text
Active: failed
```

Apache's log showed:

```text
Address already in use
AH00072: make_sock: could not bind to address [::]:6000
```

This confirmed the port conflict.

---

# 9. Check the Process Using Port 6000

The PID reported by `ss` was:

```text
19053
```

Inspect it with:

```bash
sudo ps -fp 19053
```

## Result

The process was:

```text
sendmail: accepting connections
```

This confirmed that Sendmail was the process occupying port `6000`.

---

# 10. Check Sendmail Service

```bash
sudo systemctl status sendmail --no-pager
```

## Purpose

Confirms whether Sendmail is running as a systemd service.

The service was active and its main PID was the same process identified by `ss`.

---

# 11. Stop Sendmail

```bash
sudo systemctl stop sendmail
```

## Purpose

Stops the conflicting Sendmail process and releases port `6000`.

After stopping it, verify the port:

```bash
sudo ss -lntp | grep 6000
```

The Sendmail listener should no longer be present.

---

# 12. Start Apache

```bash
sudo systemctl start httpd
```

## Purpose

Starts the Apache HTTP Server after port `6000` has been released.

---

# 13. Verify Apache Is Running

```bash
sudo systemctl status httpd --no-pager
```

Expected:

```text
Active: active (running)
```

The service log should indicate:

```text
Server configured, listening on port 6000
```

---

# 14. Verify Apache Is Listening on Port 6000

```bash
sudo ss -lntp | grep 6000
```

Expected:

```text
*:6000
```

with `httpd` as the listening process.

## Why `*` Matters

A listener such as:

```text
127.0.0.1:6000
```

accepts connections only from the local machine.

A listener such as:

```text
*:6000
```

can accept connections through the server's network interfaces, subject to firewall rules.

---

# 15. Enable Apache at Boot

```bash
sudo systemctl enable httpd
```

## Purpose

Configures Apache to start automatically during system boot.

This is different from:

```bash
sudo systemctl start httpd
```

`start` affects the current session.

`enable` configures automatic startup after reboot.

---

# 16. Verify Apache Is Enabled

```bash
sudo systemctl is-enabled httpd
```

Expected:

```text
enabled
```

---

# 17. Add the Firewall Allow Rule

```bash
sudo iptables -I INPUT 4 -p tcp --dport 6000 -j ACCEPT
```

## Purpose

Allows incoming TCP traffic destined for port `6000`.

### Command Breakdown

```text
iptables
    ↓
Firewall management command

-I INPUT 4
    ↓
Insert a rule at position 4 in INPUT

-p tcp
    ↓
Match TCP traffic

--dport 6000
    ↓
Match destination port 6000

-j ACCEPT
    ↓
Allow matching traffic
```

---

# 18. Why the Rule Was Inserted at Position 4

The INPUT chain already contained a final rejection rule:

```text
REJECT ... reject-with icmp-host-prohibited
```

Firewall rules are evaluated in order.

Therefore, the allow rule must appear before the broad rejection rule.

Using:

```bash
sudo iptables -I INPUT 4 ...
```

inserts the rule at the required position rather than appending it after the rejection rule.

---

# 19. Verify iptables Rule Ordering

```bash
sudo iptables -L INPUT -n -v --line-numbers
```

Look for a rule similar to:

```text
ACCEPT  tcp  --  0.0.0.0/0  0.0.0.0/0  tcp dpt:6000
```

It should appear before the final broad `REJECT` rule.

---

# 20. Test Apache Locally

On `stapp01`:

```bash
curl http://localhost:6000
```

## Purpose

Tests the web server locally without involving the external firewall path.

A successful response confirms that Apache is serving content on port `6000`.

---

# 21. Test HTTP Headers

```bash
curl -I http://localhost:6000
```

## Purpose

Displays HTTP response headers without retrieving the complete page body.

A successful application should return an HTTP success response.

---

# 22. Exit Application Server 1

```bash
exit
```

## Purpose

Closes the SSH session and returns to the Jump Host.

---

# 23. Test from Jump Host

From:

```text
thor@jump-host
```

run:

```bash
curl http://stapp01:6000
```

## Purpose

Performs the final end-to-end connectivity test required by the challenge.

This verifies:

```text
Jump Host
   ↓
Network connectivity
   ↓
iptables
   ↓
TCP/6000
   ↓
Apache
   ↓
Application
```

---

# 24. Optional Remote Port Test

From the Jump Host:

```bash
nc -zv stapp01 6000
```

A successful connection confirms that TCP port `6000` is reachable.

---

# 25. Complete Troubleshooting Command Sequence

```bash
# From Jump Host
telnet stapp01 6000
nc -zv stapp01 6000

# Connect to Application Server 1
ssh tony@stapp01

# Inspect firewall
sudo iptables -L -n -v

# Check alternative firewall interfaces
sudo nft list ruleset
sudo firewall-cmd --state

# Inspect port 6000
sudo ss -lntp | grep 6000

# Check Apache
sudo systemctl status httpd --no-pager

# Inspect the process occupying the port
sudo ps -fp 19053

# Check Sendmail
sudo systemctl status sendmail --no-pager

# Release port 6000
sudo systemctl stop sendmail

# Start Apache
sudo systemctl start httpd

# Verify Apache
sudo systemctl status httpd --no-pager
sudo ss -lntp | grep 6000

# Enable Apache at boot
sudo systemctl enable httpd
sudo systemctl is-enabled httpd

# Allow application traffic
sudo iptables -I INPUT 4 -p tcp --dport 6000 -j ACCEPT

# Verify firewall ordering
sudo iptables -L INPUT -n -v --line-numbers

# Local application test
curl http://localhost:6000

# Return to Jump Host
exit

# Remote application test
curl http://stapp01:6000
```

---

# 26. Quick Command Cheat Sheet

| Command | Purpose |
|---|---|
| `ssh user@server` | Connect to a remote server |
| `telnet host port` | Test TCP connectivity |
| `nc -zv host port` | Test TCP connectivity |
| `ss -lntp` | Show listening TCP sockets and processes |
| `ps -fp PID` | Inspect a process |
| `systemctl status` | Check service status |
| `systemctl start` | Start a service |
| `systemctl stop` | Stop a service |
| `systemctl enable` | Enable service at boot |
| `systemctl is-enabled` | Verify boot enablement |
| `iptables -L` | List firewall rules |
| `iptables -I` | Insert a firewall rule |
| `curl` | Test HTTP/application connectivity |
| `exit` | Close an SSH session |

---

# 27. Commands That Should NOT Be Used for This Fix

Do not disable the firewall:

```bash
sudo systemctl stop iptables
```

Do not flush all firewall rules:

```bash
sudo iptables -F
```

Do not modify the application HTML.

Do not replace the required port with another port.

The correct approach is to make the **smallest targeted configuration change** required for TCP/6000.

---

# 28. Troubleshooting Mental Model

When an application port is unreachable:

```text
1. Can the hostname resolve?
        ↓
2. Can the port be reached?
        ↓
3. Is anything listening on the port?
        ↓
4. Which process owns the port?
        ↓
5. Is the application service healthy?
        ↓
6. Is the service listening on the correct interface?
        ↓
7. Is the firewall allowing the port?
        ↓
8. Does the application respond locally?
        ↓
9. Does it respond remotely?
```

This sequence prevents random configuration changes and makes troubleshooting systematic.
