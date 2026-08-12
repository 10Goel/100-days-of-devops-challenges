# Day 13 -- Secure Apache Port 6400 with iptables

## 📌 Challenge Overview

This challenge focuses on securing the Nautilus application servers by
restricting access to Apache's exposed **TCP port 6400** using
**iptables**.

The security team identified that port `6400` was open to all sources
because no host-level firewall was installed. The requirement was to
introduce a firewall layer while ensuring that the Load Balancer Server
could continue communicating with all application servers.

------------------------------------------------------------------------

## 🎯 Objectives

The task required the following:

1.  Install `iptables` and its required dependencies on every
    application server.
2.  Block incoming TCP traffic to port `6400` from all sources **except
    the Load Balancer Server**.
3.  Ensure the firewall rules persist after a system reboot.
4.  Validate that the Load Balancer can still reach the applications on
    port `6400`.

------------------------------------------------------------------------

## 🏗️ Infrastructure

  -----------------------------------------------------------------------
  Server            Hostname          User              Role
  ----------------- ----------------- ----------------- -----------------
  Application       `stapp01`         `tony`            Hosts Nautilus
  Server 1                                              Application 1

  Application       `stapp02`         `steve`           Hosts Nautilus
  Server 2                                              Application 2

  Application       `stapp03`         `banner`          Hosts Nautilus
  Server 3                                              Application 3

  Load Balancer     `stlb01`          `loki`            Distributes
  Server                                                traffic for
                                                        Nautilus HTTP
  -----------------------------------------------------------------------

### Load Balancer IP

The Load Balancer hostname resolved to:

``` text
10.244.49.26    stlb01
```

> **Security note:** Credentials from the lab infrastructure are
> intentionally not documented in this repository.

------------------------------------------------------------------------

## 🔍 Initial Assessment

Before making changes, the application server was inspected to confirm
the current state.

### Check existing firewall rules

``` bash
sudo iptables -L -n -v
```

The application server initially had no filtering rules:

``` text
Chain INPUT (policy ACCEPT)
Chain FORWARD (policy ACCEPT)
Chain OUTPUT (policy ACCEPT)
```

### Confirm Apache is listening on port 6400

``` bash
sudo ss -lntp | grep 6400
```

Apache was listening on:

``` text
*:6400
```

This confirmed that the service was reachable on all interfaces and
required host-level access control.

------------------------------------------------------------------------

# 🔐 Firewall Design

The required access policy was:

``` text
                    TCP :6400
                         |
              +----------+----------+
              |                     |
       Source = 10.244.49.26    Any other source
              |                     |
           ACCEPT                  DROP
```

The rules were deliberately ordered so that the Load Balancer's traffic
is accepted before the general DROP rule.

### Rule 1 --- Allow Load Balancer

``` bash
sudo iptables -I INPUT 1 -p tcp -s 10.244.49.26 --dport 6400 -j ACCEPT
```

### Rule 2 --- Drop everyone else

``` bash
sudo iptables -A INPUT -p tcp --dport 6400 -j DROP
```

This produces the desired behavior:

``` text
10.244.49.26 → TCP/6400 → ACCEPT
All other sources → TCP/6400 → DROP
```

Other ports are not affected by these rules.

------------------------------------------------------------------------

# 💾 Persistence Configuration

Firewall rules must survive a reboot.

The rules were saved using:

``` bash
sudo iptables-save | sudo tee /etc/sysconfig/iptables
```

The iptables service was enabled:

``` bash
sudo systemctl enable iptables
```

The saved configuration contains rules equivalent to:

``` text
-A INPUT -s 10.244.49.26/32 -p tcp --dport 6400 -j ACCEPT
-A INPUT -p tcp --dport 6400 -j DROP
```

### Service State

On systems using `iptables-services`, seeing:

``` text
Active: active (exited)
```

is acceptable. The service loads the rules and does not need to remain
as a continuously running daemon.

The important persistence check is:

``` bash
sudo systemctl is-enabled iptables
```

Expected result:

``` text
enabled
```

------------------------------------------------------------------------

# 🧪 Validation

## 1. Verify firewall rule order

``` bash
sudo iptables -L INPUT -n -v --line-numbers
```

Expected structure:

``` text
1  ACCEPT  tcp  10.244.49.26  ...  tcp dpt:6400
2  DROP    tcp  0.0.0.0/0     ...  tcp dpt:6400
```

The ACCEPT rule must appear before the DROP rule.

------------------------------------------------------------------------

## 2. Verify persistence

``` bash
sudo cat /etc/sysconfig/iptables
```

And:

``` bash
sudo systemctl is-enabled iptables
```

Expected:

``` text
enabled
```

------------------------------------------------------------------------

## 3. Validate through the Load Balancer

Connectivity was tested from the Load Balancer Server against the
application servers:

``` bash
curl -I --connect-timeout 5 http://stapp01:6400
curl -I --connect-timeout 5 http://stapp02:6400
curl -I --connect-timeout 5 http://stapp03:6400
```

The applications returned the expected HTTP responses.

This confirmed:

-   Load Balancer → Application Server 1: ✅
-   Load Balancer → Application Server 2: ✅
-   Load Balancer → Application Server 3: ✅
-   TCP port `6400` remains available to the authorized source.
-   Unauthorized traffic is filtered by the application-server firewall.

------------------------------------------------------------------------

# 🧠 Key Concepts Learned

### iptables

`iptables` is a Linux firewall framework used to inspect, filter, and
control network packets.

### INPUT chain

The `INPUT` chain controls packets destined for the local server.

### Rule ordering

iptables evaluates rules in order. Therefore:

``` text
ACCEPT trusted source
DROP all remaining traffic
```

is fundamentally different from:

``` text
DROP all traffic
ACCEPT trusted source
```

The second arrangement would block the trusted source before it could
reach the ACCEPT rule.

### `-I` vs `-A`

``` bash
-I INPUT 1
```

inserts a rule at the beginning of the chain.

``` bash
-A INPUT
```

appends a rule to the end.

This challenge used `-I` for the trusted-source rule and `-A` for the
final DROP rule to guarantee the correct evaluation order.

### `--dport 6400`

Limits the rule to traffic destined for TCP port `6400`.

### `-s 10.244.49.26`

Restricts the ACCEPT rule to packets originating from the Load Balancer.

------------------------------------------------------------------------

# 🏛️ Final Architecture

``` text
                         ┌──────────────────┐
                         │      stlb01      │
                         │   10.244.49.26   │
                         │   Load Balancer   │
                         └────────┬─────────┘
                                  │
                         TCP / HTTP :6400
                                  │
             ┌────────────────────┼────────────────────┐
             │                    │                    │
             ▼                    ▼                    ▼
       ┌───────────┐        ┌───────────┐        ┌───────────┐
       │  stapp01  │        │  stapp02  │        │  stapp03  │
       │   :6400   │        │   :6400   │        │   :6400   │
       └───────────┘        └───────────┘        └───────────┘
             ▲                    ▲                    ▲
             │                    │                    │
             └────────────── ❌ OTHER SOURCES ─────────┘
                               BLOCKED
```

------------------------------------------------------------------------

# 🛡️ Security Outcome

Before the change:

``` text
Any source ───────────────→ TCP/6400 ❌ Unrestricted
```

After the change:

``` text
Load Balancer ────────────→ TCP/6400 ✅ Allowed
Any other source ─────────→ TCP/6400 ❌ Blocked
```

This implements a simple **least-privilege network access model**: only
the system that requires access to the application port is explicitly
trusted.

------------------------------------------------------------------------

# 📋 Completion Checklist

-   [x] Installed `iptables` on application servers
-   [x] Identified Load Balancer IP
-   [x] Allowed Load Balancer access to TCP `6400`
-   [x] Blocked all other incoming TCP `6400` traffic
-   [x] Verified rule ordering
-   [x] Saved iptables rules
-   [x] Enabled iptables persistence
-   [x] Verified HTTP connectivity through Load Balancer
-   [x] Confirmed expected application responses

------------------------------------------------------------------------

## 🏁 Result

**Day 13 DevOps Challenge --- Successfully Completed ✅**

The Nautilus application servers are now protected on Apache port `6400`
while maintaining the required communication path from the Load
Balancer.
