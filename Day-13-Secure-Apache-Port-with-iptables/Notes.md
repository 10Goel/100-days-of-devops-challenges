# Day 13 -- Notes

## 1. Core Problem

Apache was running on TCP port `6400`, but the application servers did
not have a host-level firewall.

The requirement was:

> Allow the Load Balancer to access port `6400`, but block all other
> incoming sources.

This is a classic **source-based firewall restriction**.

------------------------------------------------------------------------

# 2. Important Infrastructure Detail

The Load Balancer hostname was resolved using:

``` bash
getent hosts stlb01
```

Result:

``` text
10.244.49.26    stlb01.3b62b32rlweptyna.svc.cluster.local
```

Therefore:

``` text
stlb01 = 10.244.49.26
```

This IP was used as the trusted source in the firewall rules.

------------------------------------------------------------------------

# 3. Why Check the Listening Port First?

Before changing the firewall, verify that the application is actually
listening:

``` bash
sudo ss -lntp | grep 6400
```

Example:

``` text
LISTEN ... *:6400 ... httpd
```

This tells us:

-   Apache is running.
-   Port `6400` is open locally.
-   The service is listening on all interfaces.
-   The firewall is the correct layer for restricting network access.

------------------------------------------------------------------------

# 4. Understanding iptables

iptables provides packet filtering through chains and rules.

The main chains are:

``` text
INPUT    → traffic entering the local server
OUTPUT   → traffic leaving the local server
FORWARD  → traffic routed through the server
```

For this task, the important chain is:

``` text
INPUT
```

because the application servers are receiving connections on port
`6400`.

------------------------------------------------------------------------

# 5. The Firewall Policy

The desired policy is:

``` text
Trusted source:
10.244.49.26 → TCP 6400 → ACCEPT

Everyone else:
any source → TCP 6400 → DROP
```

The exact commands were:

``` bash
sudo iptables -I INPUT 1 -p tcp -s 10.244.49.26 --dport 6400 -j ACCEPT
```

``` bash
sudo iptables -A INPUT -p tcp --dport 6400 -j DROP
```

------------------------------------------------------------------------

# 6. Why Rule Order Matters

iptables processes rules from top to bottom.

Correct:

``` text
1. ACCEPT  10.244.49.26 → TCP/6400
2. DROP    anyone       → TCP/6400
```

Incorrect:

``` text
1. DROP    anyone       → TCP/6400
2. ACCEPT  10.244.49.26 → TCP/6400
```

With the incorrect order, the Load Balancer's traffic would be dropped
before reaching the ACCEPT rule.

### Mental model

Think of the rules like a security guard:

``` text
Is the visitor the Load Balancer?
        │
      YES ──→ Allow
        │
       NO
        │
        └────→ Block port 6400
```

------------------------------------------------------------------------

# 7. `-I` vs `-A`

### Insert

``` bash
-I INPUT 1
```

Places a rule at position 1.

### Append

``` bash
-A INPUT
```

Places a rule at the end of the chain.

For this challenge:

``` bash
-I INPUT 1
```

was used for the trusted Load Balancer because it must be evaluated
before the DROP rule.

------------------------------------------------------------------------

# 8. Understanding the Rule Components

Consider:

``` bash
sudo iptables -I INPUT 1 -p tcp -s 10.244.49.26 --dport 6400 -j ACCEPT
```

### `-I INPUT 1`

Insert into INPUT chain at position 1.

### `-p tcp`

Match TCP traffic.

### `-s 10.244.49.26`

Match packets originating from the Load Balancer.

### `--dport 6400`

Match destination port `6400`.

### `-j ACCEPT`

Allow matching packets.

------------------------------------------------------------------------

The second rule:

``` bash
sudo iptables -A INPUT -p tcp --dport 6400 -j DROP
```

means:

``` text
Any TCP packet
        +
Destination port 6400
        +
No previous ACCEPT match
        ↓
DROP
```

------------------------------------------------------------------------

# 9. Verifying Rules

Useful command:

``` bash
sudo iptables -L INPUT -n -v --line-numbers
```

Expected:

``` text
1  ACCEPT  tcp  10.244.49.26  ... tcp dpt:6400
2  DROP    tcp  0.0.0.0/0     ... tcp dpt:6400
```

The `--line-numbers` option makes rule ordering obvious.

The `-n` option prevents reverse DNS lookups and makes output easier to
read.

The `-v` option provides packet and byte counters.

------------------------------------------------------------------------

# 10. Rule Counters

Example:

``` text
pkts bytes target
10   800   ACCEPT
25   2000  DROP
```

These counters indicate how many packets and bytes matched each rule.

For troubleshooting, counters can help determine whether traffic is
actually reaching a rule.

------------------------------------------------------------------------

# 11. Persistence

iptables rules are kernel state. Saving them is necessary if they need
to be restored after reboot.

Save the current rules:

``` bash
sudo iptables-save | sudo tee /etc/sysconfig/iptables
```

Enable the service:

``` bash
sudo systemctl enable iptables
```

Verify:

``` bash
sudo systemctl is-enabled iptables
```

Expected:

``` text
enabled
```

------------------------------------------------------------------------

# 12. `active (exited)` vs `inactive (dead)`

On systems using `iptables-services`, the service may appear as:

``` text
Active: active (exited)
```

This is normal because the service loads/restores the rules and does not
need to remain as a continuously running daemon.

The important persistence check is:

``` bash
sudo systemctl is-enabled iptables
```

which should return:

``` text
enabled
```

After starting the service, this can also be checked with:

``` bash
sudo systemctl is-active iptables
```

------------------------------------------------------------------------

# 13. Testing Through the Load Balancer

The most meaningful functional test is to test from the authorized
source.

From `stlb01`:

``` bash
curl -I --connect-timeout 5 http://stapp01:6400
curl -I --connect-timeout 5 http://stapp02:6400
curl -I --connect-timeout 5 http://stapp03:6400
```

Expected result:

``` text
HTTP response
```

The exact status code may vary depending on the application, but the
connection should succeed.

------------------------------------------------------------------------

# 14. Security Principle

This challenge demonstrates **least privilege**.

Instead of allowing:

``` text
0.0.0.0/0 → TCP/6400
```

we explicitly trust only:

``` text
10.244.49.26 → TCP/6400
```

Everything else is denied.

This reduces the exposed attack surface of the application servers.

------------------------------------------------------------------------

# 15. Troubleshooting Checklist

If the Load Balancer cannot reach the application:

### Check the service

``` bash
sudo ss -lntp | grep 6400
```

### Check firewall rules

``` bash
sudo iptables -L INPUT -n -v --line-numbers
```

### Check the Load Balancer IP

``` bash
getent hosts stlb01
```

### Check persistence

``` bash
sudo cat /etc/sysconfig/iptables
sudo systemctl is-enabled iptables
```

### Test HTTP

``` bash
curl -I --connect-timeout 5 http://stapp01:6400
```

------------------------------------------------------------------------

# 16. Common Mistakes

## Mistake 1 --- DROP before ACCEPT

Bad:

``` text
DROP 6400
ACCEPT 10.244.49.26:6400
```

Result: Load Balancer is blocked.

Correct:

``` text
ACCEPT 10.244.49.26:6400
DROP 6400
```

------------------------------------------------------------------------

## Mistake 2 --- Forgetting persistence

Running only:

``` bash
iptables ...
```

changes the live firewall but does not by itself guarantee restoration
after reboot.

Use:

``` bash
sudo iptables-save | sudo tee /etc/sysconfig/iptables
sudo systemctl enable iptables
```

------------------------------------------------------------------------

## Mistake 3 --- Blocking the wrong port

The challenge concerns:

``` text
TCP 6400
```

Do not create a blanket firewall rule that unnecessarily blocks
unrelated application traffic.

------------------------------------------------------------------------

## Mistake 4 --- Trusting a hostname without checking its current IP

Always verify:

``` bash
getent hosts stlb01
```

before creating a source-IP-based rule.

------------------------------------------------------------------------

# 17. Practical Mental Model

Remember this sequence:

``` text
1. Identify the application port
2. Identify the trusted source
3. Allow the trusted source
4. Drop everyone else
5. Check rule order
6. Save the rules
7. Enable persistence
8. Test from the trusted source
```

This workflow is useful for many real-world Linux firewall tasks.

------------------------------------------------------------------------

# 18. Day 13 Takeaways

-   `iptables` can provide host-level network access control.
-   The `INPUT` chain controls incoming traffic.
-   Rule order is critical.
-   `-I` inserts a rule; `-A` appends a rule.
-   Source-based filtering can restrict access to trusted infrastructure
    components.
-   Firewall rules should be persisted when reboot survival is required.
-   `active (exited)` can be normal for `iptables-services`.
-   Always validate both the firewall configuration and the actual
    application connectivity.
