# Day 13 -- Commands Reference

A practical command reference for securing Apache TCP port `6400` with
iptables.

------------------------------------------------------------------------

# 1. Identify the Load Balancer

Resolve the Load Balancer hostname:

``` bash
getent hosts stlb01
```

Example:

``` text
10.244.49.26    stlb01.3b62b32rlweptyna.svc.cluster.local
```

Trusted Load Balancer IP:

``` text
10.244.49.26
```

------------------------------------------------------------------------

# 2. Connect to Application Servers

### Application Server 1

``` bash
ssh tony@stapp01
```

### Application Server 2

``` bash
ssh steve@stapp02
```

### Application Server 3

``` bash
ssh banner@stapp03
```

Exit:

``` bash
exit
```

------------------------------------------------------------------------

# 3. Verify the Host

``` bash
hostname
```

Expected examples:

``` text
stapp01
stapp02
stapp03
```

------------------------------------------------------------------------

# 4. Install iptables

For the lab's RHEL/CentOS-style environment:

``` bash
sudo yum install -y iptables iptables-services
```

If `yum` is unavailable:

``` bash
sudo dnf install -y iptables iptables-services
```

Verify:

``` bash
iptables --version
```

------------------------------------------------------------------------

# 5. Inspect Existing Firewall Rules

``` bash
sudo iptables -L -n -v
```

Detailed INPUT-chain view:

``` bash
sudo iptables -L INPUT -n -v --line-numbers
```

------------------------------------------------------------------------

# 6. Check Apache Port 6400

``` bash
sudo ss -lntp | grep 6400
```

Alternative:

``` bash
sudo ss -lntp | grep ':6400'
```

Expected:

``` text
LISTEN ... *:6400 ... httpd
```

------------------------------------------------------------------------

# 7. Allow the Load Balancer

``` bash
sudo iptables -I INPUT 1 -p tcp -s 10.244.49.26 --dport 6400 -j ACCEPT
```

Meaning:

``` text
Allow TCP port 6400
from source 10.244.49.26
```

------------------------------------------------------------------------

# 8. Block Everyone Else

``` bash
sudo iptables -A INPUT -p tcp --dport 6400 -j DROP
```

Meaning:

``` text
Drop all other TCP traffic to port 6400
```

------------------------------------------------------------------------

# 9. Verify Rule Order

``` bash
sudo iptables -L INPUT -n -v --line-numbers
```

Expected structure:

``` text
1  ACCEPT  tcp  10.244.49.26  ... tcp dpt:6400
2  DROP    tcp  0.0.0.0/0     ... tcp dpt:6400
```

------------------------------------------------------------------------

# 10. Save Firewall Rules

``` bash
sudo iptables-save | sudo tee /etc/sysconfig/iptables
```

Inspect saved configuration:

``` bash
sudo cat /etc/sysconfig/iptables
```

Look for:

``` text
-A INPUT -s 10.244.49.26/32 -p tcp --dport 6400 -j ACCEPT
-A INPUT -p tcp --dport 6400 -j DROP
```

------------------------------------------------------------------------

# 11. Enable Persistence

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

# 12. Start iptables Service

``` bash
sudo systemctl start iptables
```

Check status:

``` bash
sudo systemctl status iptables --no-pager
```

On `iptables-services`, an expected healthy state can be:

``` text
Active: active (exited)
```

Check current service state:

``` bash
sudo systemctl is-active iptables
```

------------------------------------------------------------------------

# 13. Reload/Restore Saved Rules

If required during troubleshooting:

``` bash
sudo systemctl restart iptables
```

Then verify:

``` bash
sudo iptables -L INPUT -n -v --line-numbers
```

------------------------------------------------------------------------

# 14. Test Connectivity from Load Balancer

Connect to the Load Balancer:

``` bash
ssh loki@stlb01
```

Test Application Server 1:

``` bash
curl -I --connect-timeout 5 http://stapp01:6400
```

Test Application Server 2:

``` bash
curl -I --connect-timeout 5 http://stapp02:6400
```

Test Application Server 3:

``` bash
curl -I --connect-timeout 5 http://stapp03:6400
```

A successful HTTP response confirms that the authorized source can reach
the protected port.

------------------------------------------------------------------------

# 15. Useful Connectivity Commands

### TCP connectivity with nc

``` bash
nc -vz -w 5 stapp01 6400
```

### TCP connectivity with telnet

``` bash
telnet stapp01 6400
```

### HTTP headers

``` bash
curl -I http://stapp01:6400
```

### HTTP with timeout

``` bash
curl -I --connect-timeout 5 http://stapp01:6400
```

------------------------------------------------------------------------

# 16. Troubleshooting Commands

### Check listening ports

``` bash
sudo ss -lntp
```

### Check only port 6400

``` bash
sudo ss -lntp | grep 6400
```

### Check firewall rules

``` bash
sudo iptables -L -n -v --line-numbers
```

### Check INPUT chain

``` bash
sudo iptables -L INPUT -n -v --line-numbers
```

### Check saved rules

``` bash
sudo cat /etc/sysconfig/iptables
```

### Check service enablement

``` bash
sudo systemctl is-enabled iptables
```

### Check service state

``` bash
sudo systemctl is-active iptables
```

### Check complete service status

``` bash
sudo systemctl status iptables --no-pager
```

### Check Load Balancer resolution

``` bash
getent hosts stlb01
```

------------------------------------------------------------------------

# 17. Rule Management

### List rules

``` bash
sudo iptables -L INPUT -n --line-numbers
```

### Show rules in command format

``` bash
sudo iptables -S INPUT
```

### Delete a rule by number

First identify the rule number:

``` bash
sudo iptables -L INPUT -n --line-numbers
```

Then:

``` bash
sudo iptables -D INPUT <RULE_NUMBER>
```

> Use rule deletion carefully in a live firewall.

------------------------------------------------------------------------

# 18. Quick Deployment Sequence

Run the following on **each application server**:

``` bash
sudo yum install -y iptables iptables-services

sudo iptables -I INPUT 1 -p tcp -s 10.244.49.26 --dport 6400 -j ACCEPT

sudo iptables -A INPUT -p tcp --dport 6400 -j DROP

sudo iptables -L INPUT -n -v --line-numbers

sudo iptables-save | sudo tee /etc/sysconfig/iptables

sudo systemctl enable iptables

sudo systemctl start iptables

sudo systemctl status iptables --no-pager
```

------------------------------------------------------------------------

# 19. Final Verification Sequence

``` bash
sudo iptables -L INPUT -n -v --line-numbers
```

``` bash
sudo cat /etc/sysconfig/iptables
```

``` bash
sudo systemctl is-enabled iptables
```

``` bash
sudo ss -lntp | grep 6400
```

Then from `stlb01`:

``` bash
curl -I --connect-timeout 5 http://stapp01:6400
curl -I --connect-timeout 5 http://stapp02:6400
curl -I --connect-timeout 5 http://stapp03:6400
```

------------------------------------------------------------------------

# 20. Command Cheat Sheet

  --------------------------------------------------------------------------------------------------------------
  Purpose                             Command
  ----------------------------------- --------------------------------------------------------------------------
  Resolve LBR                         `getent hosts stlb01`

  Install iptables                    `sudo yum install -y iptables iptables-services`

  Check version                       `iptables --version`

  List firewall                       `sudo iptables -L -n -v`

  List INPUT with numbers             `sudo iptables -L INPUT -n -v --line-numbers`

  Check port 6400                     `sudo ss -lntp \| grep 6400`

  Allow LBR                           `sudo iptables -I INPUT 1 -p tcp -s 10.244.49.26 --dport 6400 -j ACCEPT`

  Drop other traffic                  `sudo iptables -A INPUT -p tcp --dport 6400 -j DROP`

  Save rules                          `sudo iptables-save \| sudo tee /etc/sysconfig/iptables`

  Enable persistence                  `sudo systemctl enable iptables`

  Start service                       `sudo systemctl start iptables`

  Check enabled                       `sudo systemctl is-enabled iptables`

  Check active                        `sudo systemctl is-active iptables`

  Check status                        `sudo systemctl status iptables --no-pager`

  View saved rules                    `sudo cat /etc/sysconfig/iptables`

  Test HTTP                           `curl -I --connect-timeout 5 http://stapp01:6400`

  Test TCP                            `nc -vz -w 5 stapp01 6400`
  --------------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

# 21. Golden Rule

For this type of firewall task, remember:

``` text
ALLOW trusted source
        ↓
BLOCK everyone else
        ↓
SAVE rules
        ↓
ENABLE persistence
        ↓
VERIFY connectivity
```

Always verify **rule order** before testing connectivity.
