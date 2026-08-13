# Day 16 --- Notes: Nginx Load Balancer

## 1. What Is a Load Balancer?

A load balancer is a component that receives requests from clients and
distributes those requests across multiple backend servers.

Instead of allowing users to directly access one application server:

``` text
Client → Application Server
```

we use:

``` text
Client → Load Balancer → Application Servers
```

For this challenge:

``` text
Client → stlb01:80 → stapp01/stapp02/stapp03:5003
```

This improves availability and allows traffic to be distributed across
multiple application servers.

------------------------------------------------------------------------

# 2. Why Was a Load Balancer Needed?

The Nautilus application was receiving increasing traffic.

If all traffic were sent to a single application server:

``` text
All users
   |
   v
stapp01
```

the server could become overloaded.

With multiple servers:

``` text
             Load Balancer
                   |
        +----------+----------+
        |          |          |
     stapp01    stapp02    stapp03
```

requests can be distributed between the available servers.

This provides better scalability and availability.

------------------------------------------------------------------------

# 3. What Is Nginx?

Nginx is a high-performance web server and reverse proxy that can also
act as a load balancer.

In this challenge, Nginx was not primarily being used to serve the
application itself.

Instead, its role was:

``` text
Client
   ↓
Nginx
   ↓
Apache application server
```

Nginx receives the HTTP request and forwards it to a backend Apache
server.

------------------------------------------------------------------------

# 4. Reverse Proxy

A reverse proxy sits in front of backend servers.

The client communicates with the reverse proxy rather than directly with
the backend.

In this challenge:

``` text
Client
   |
   | HTTP :80
   v
stlb01
Nginx
   |
   | HTTP :5003
   v
Apache backend
```

The client does not need to know which application server actually
handles the request.

------------------------------------------------------------------------

# 5. Frontend Port vs Backend Port

One of the most important concepts in this task is understanding that
the load balancer's listening port and the backend application's port
can be different.

Nginx listens on:

``` text
stlb01:80
```

Apache listens on:

``` text
stapp01:5003
stapp02:5003
stapp03:5003
```

Therefore:

``` text
Client
  |
  | :80
  v
Nginx
  |
  | :5003
  v
Apache
```

The task specifically instructed us **not to change the Apache port**.

Therefore, we did not change `5003`.

------------------------------------------------------------------------

# 6. Understanding `upstream`

The Nginx configuration contained:

``` nginx
upstream nautlius_backend {
    server stapp01:5003;
    server stapp02:5003;
    server stapp03:5003;
}
```

The `upstream` block defines a group of backend servers.

We can think of it as a backend pool:

``` text
nautlius_backend
       |
       +--- stapp01:5003
       +--- stapp02:5003
       +--- stapp03:5003
```

Nginx can then select a backend from this group.

------------------------------------------------------------------------

# 7. Round-Robin Load Balancing

When multiple servers are listed in an Nginx `upstream` block and no
other balancing method is specified, Nginx uses round robin by default.

Conceptually:

``` text
Request 1 → stapp01
Request 2 → stapp02
Request 3 → stapp03
Request 4 → stapp01
Request 5 → stapp02
...
```

This is useful when backend servers have similar capacity.

The actual request distribution can also depend on connection behavior
and backend availability.

------------------------------------------------------------------------

# 8. Understanding `proxy_pass`

The configuration contained:

``` nginx
location / {
    proxy_pass http://nautlius_backend;
}
```

`proxy_pass` tells Nginx where to forward the request.

Here:

``` text
http://nautlius_backend
```

refers to the upstream backend group.

So the request flow is:

``` text
GET /
   |
   v
Nginx :80
   |
   v
nautlius_backend
   |
   +--> stapp01:5003
   +--> stapp02:5003
   +--> stapp03:5003
```

------------------------------------------------------------------------

# 9. Why `server_name _;`?

The configuration used:

``` nginx
server_name _;
```

The underscore is commonly used as a catch-all/default-style server name
in simple Nginx configurations.

For this lab, the important requirement is that Nginx accepts the
request arriving on port `80`.

------------------------------------------------------------------------

# 10. Why `listen 80`?

HTTP normally uses TCP port `80`.

The task explicitly required the application to be accessible through:

``` text
stlb01:80
```

Therefore:

``` nginx
listen 80;
```

was configured.

The backend Apache servers continued using their existing port:

``` text
5003
```

------------------------------------------------------------------------

# 11. Checking Listening Ports with `ss`

The command:

``` bash
sudo ss -lntp
```

was used to investigate which ports were already in use.

Important options:

  Option   Meaning
  -------- ------------------------------
  `-l`     Show listening sockets
  `-n`     Show numeric addresses/ports
  `-t`     Show TCP sockets
  `-p`     Show associated processes

For example:

``` text
*:5003
```

with `httpd` in the process column indicated that Apache was listening
on port `5003`.

This was an important troubleshooting step because changing an existing
application port could have broken the application.

------------------------------------------------------------------------

# 12. Nginx Configuration Validation

Before restarting Nginx, we used:

``` bash
sudo nginx -t
```

This is a very important DevOps practice.

It checks whether the Nginx configuration has valid syntax.

Successful output:

``` text
syntax is ok
test is successful
```

The general workflow should be:

``` text
Edit configuration
       ↓
nginx -t
       ↓
If successful
       ↓
Restart/reload Nginx
```

Never blindly restart a production service after editing its
configuration.

------------------------------------------------------------------------

# 13. `systemctl` and Nginx

The Nginx service was managed using systemd.

Restart:

``` bash
sudo systemctl restart nginx
```

Check status:

``` bash
sudo systemctl status nginx
```

A successful status showed:

``` text
Active: active (running)
```

------------------------------------------------------------------------

# 14. `curl` Verification

The final application test was:

``` bash
curl http://stlb01:80
```

The server returned:

``` text
Welcome to xFusionCorp Industries!
```

This is stronger than merely checking whether Nginx is running.

There are two different checks:

### Service check

``` bash
systemctl status nginx
```

This tells us Nginx is running.

### Application check

``` bash
curl http://stlb01:80
```

This tells us the complete HTTP path is functioning.

------------------------------------------------------------------------

# 15. Why We Didn't Change Apache

The task explicitly stated:

> Do not update the Apache port.

Apache was already listening on:

``` text
5003
```

Therefore the correct solution was to make Nginx communicate with Apache
on `5003`.

Incorrect approach:

``` text
Change Apache to port 80
```

Correct approach:

``` text
Nginx :80 → Apache :5003
```

This distinction is extremely important in real-world infrastructure.

------------------------------------------------------------------------

# 16. Why a Load Balancer Can Listen on 80 While Backend Servers Use 5003

Ports belong to individual network endpoints.

There is no requirement for every server in a request path to use the
same port.

For example:

``` text
Client
  |
  | TCP 80
  v
Load Balancer
  |
  | TCP 5003
  v
Application Server
```

The load balancer terminates the incoming client connection and
creates/forwards a connection to the backend.

Therefore:

``` text
Frontend port ≠ Backend port
```

This is normal in reverse-proxy architectures.

------------------------------------------------------------------------

# 17. High Availability Concept

Using three application servers provides redundancy.

If the infrastructure is healthy:

``` text
             Nginx
               |
       +-------+-------+
       |       |       |
      App1    App2    App3
```

If one backend becomes unavailable, Nginx can avoid sending normal
traffic to an unavailable backend according to its upstream behavior and
configuration.

However, a production HA design may additionally use:

-   Health checks
-   Timeouts
-   Retry policies
-   Monitoring
-   Alerting
-   Multiple load balancers
-   DNS failover
-   TLS termination
-   Session management

This lab demonstrates the foundational load-balancing layer.

------------------------------------------------------------------------

# 18. Important DevOps Lesson

A very important troubleshooting principle from this task is:

> **First inspect the existing environment, then configure around it.**

We did not assume that Apache was using port `80` or `8080`.

We checked:

``` bash
sudo ss -lntp
```

and discovered:

``` text
httpd → 5003
```

Only then did we configure Nginx to proxy to:

``` text
stapp01:5003
stapp02:5003
stapp03:5003
```

This approach prevents accidental service disruption.

------------------------------------------------------------------------

# 19. Request Flow --- Complete Explanation

Suppose a user opens:

``` text
http://stlb01:80
```

### Step 1 --- Client connects

The client creates a TCP connection to:

``` text
stlb01:80
```

### Step 2 --- Nginx receives the request

Nginx is listening on port `80`.

### Step 3 --- Nginx matches the server block

The request matches:

``` nginx
server {
    listen 80;
    server_name _;
}
```

### Step 4 --- Nginx matches `/`

The URI matches:

``` nginx
location / {
}
```

### Step 5 --- `proxy_pass` forwards the request

Nginx forwards the request to:

``` text
nautlius_backend
```

### Step 6 --- Backend selection

The upstream contains:

``` text
stapp01:5003
stapp02:5003
stapp03:5003
```

Nginx selects an available backend.

### Step 7 --- Apache processes the request

The selected Apache server returns the application response.

### Step 8 --- Nginx returns the response

Nginx sends the response back to the original client.

Final flow:

``` text
Client
  |
  | HTTP :80
  v
Nginx / stlb01
  |
  | Reverse Proxy
  v
Upstream Pool
  |
  +--> stapp01:5003
  |
  +--> stapp02:5003
  |
  +--> stapp03:5003
  |
  v
Apache
  |
  v
Application Response
```

------------------------------------------------------------------------

# 20. Key Commands to Remember

``` bash
nginx -v
```

Check Nginx version.

``` bash
sudo nginx -t
```

Validate Nginx configuration.

``` bash
sudo systemctl restart nginx
```

Restart Nginx.

``` bash
sudo systemctl status nginx
```

Check Nginx service status.

``` bash
sudo ss -lntp
```

Inspect listening TCP ports and processes.

``` bash
curl http://stlb01:80
```

Test the load-balanced application.

------------------------------------------------------------------------

## 🧠 Interview Questions From This Task

### Q1. What is a load balancer?

A load balancer distributes incoming client traffic across multiple
backend servers.

### Q2. What is a reverse proxy?

A reverse proxy receives client requests and forwards them to backend
servers while hiding the backend infrastructure from clients.

### Q3. What is an Nginx upstream?

An `upstream` block defines a group of backend servers that Nginx can
proxy requests to.

### Q4. What is the default Nginx load-balancing method?

Round robin.

### Q5. Why did Nginx listen on port 80 while Apache used 5003?

Port `80` was the client-facing HTTP port, while `5003` was the existing
Apache backend port.

### Q6. Why should `nginx -t` be run before restarting Nginx?

It validates the configuration syntax and helps prevent restarting the
service with a broken configuration.

### Q7. How did we verify Apache's existing port?

Using:

``` bash
sudo ss -lntp
```

### Q8. What is `proxy_pass`?

It tells Nginx where to forward requests received by a location/server
block.

------------------------------------------------------------------------

## 🏁 Day 16 Summary

Day 16 provided practical experience with:

-   Nginx
-   Reverse proxying
-   HTTP load balancing
-   Upstream backend pools
-   Apache backend services
-   Port mapping
-   Linux service management
-   Configuration validation
-   Application connectivity testing

The final architecture was:

``` text
stlb01:80
    |
    +--> stapp01:5003
    +--> stapp02:5003
    +--> stapp03:5003
```

The application was successfully accessed through the load balancer.
