# Day 16 --- Nginx Load Balancer Configuration

## 📌 Challenge Overview

Day 16 of the KodeKloud 100 Days of DevOps challenge focused on
configuring an **Nginx Load Balancer** for the Nautilus production
application.

Traffic to the Nautilus website was increasing, so the application team
decided to deploy the application on a high-availability infrastructure
using multiple application servers behind a dedicated load balancer.

The objective was to configure the Load Balancer Server (`stlb01`) with
Nginx so that HTTP requests received on port `80` are distributed across
all three application servers.

------------------------------------------------------------------------

## 🎯 Objectives

The task required the following:

1.  Install Nginx on the Load Balancer Server if it was not already
    installed.
2.  Configure Nginx as an HTTP load balancer.
3.  Use all three Nautilus application servers as backend servers.
4.  Listen for client traffic on port `80` on the load balancer.
5.  Preserve the existing Apache configuration and ports on the
    application servers.
6.  Modify only the main Nginx configuration file:
    -   `/etc/nginx/nginx.conf`
7.  Verify that the website is accessible through the load balancer.

------------------------------------------------------------------------

## 🏗️ Infrastructure

  Server                 Hostname    Purpose                      Application Port
  ---------------------- ----------- -------------------------- ------------------
  Application Server 1   `stapp01`   Nautilus Application 1                 `5003`
  Application Server 2   `stapp02`   Nautilus Application 2                 `5003`
  Application Server 3   `stapp03`   Nautilus Application 3                 `5003`
  Load Balancer          `stlb01`    Nginx HTTP Load Balancer                 `80`

### Credentials supplied by the lab

The KodeKloud lab provides server-specific users and passwords through
its infrastructure details page. Credentials are intentionally not
reproduced in this repository document.

------------------------------------------------------------------------

## 🔎 Initial Investigation

Before configuring Nginx, the existing listening ports on the
application servers were checked using:

``` bash
sudo ss -lntp
```

On the application servers, Apache/httpd was found listening on:

``` text
*:5003
```

Port `5003` was therefore treated as the existing Apache backend port.

The task specifically stated that the Apache port must not be changed.

The `8080` listener observed during troubleshooting belonged to `ttyd`,
not Apache, so it was not used as the backend port.

------------------------------------------------------------------------

## ⚙️ Nginx Installation

Nginx was already installed on `stlb01`.

The installed version was verified with:

``` bash
nginx -v
```

Result:

``` text
nginx version: nginx/1.20.1
```

Therefore, no additional Nginx installation was required.

------------------------------------------------------------------------

## 🛠️ Configuration

A backup of the original Nginx configuration was created before making
changes:

``` bash
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.backup
```

The main configuration file was then edited:

``` bash
sudo vi /etc/nginx/nginx.conf
```

The configuration used was:

``` nginx
events {
}

http {

    upstream nautlius_backend {
        server stapp01:5003;
        server stapp02:5003;
        server stapp03:5003;
    }

    server {
        listen 80;
        server_name _;

        location / {
            proxy_pass http://nautlius_backend;
        }
    }
}
```

------------------------------------------------------------------------

## 🔄 How the Configuration Works

### 1. `upstream`

``` nginx
upstream nautlius_backend {
    server stapp01:5003;
    server stapp02:5003;
    server stapp03:5003;
}
```

The `upstream` block defines the backend server pool.

Nginx sends incoming requests to the servers listed in this block.

The backend servers are:

``` text
stapp01:5003
stapp02:5003
stapp03:5003
```

By default, Nginx uses **round-robin load balancing** when multiple
servers are specified without another balancing method.

------------------------------------------------------------------------

### 2. `listen 80`

``` nginx
listen 80;
```

This makes Nginx accept normal HTTP requests on TCP port `80`.

Therefore, clients access the application through:

``` text
http://stlb01:80
```

------------------------------------------------------------------------

### 3. `location /`

``` nginx
location / {
    proxy_pass http://nautlius_backend;
}
```

This tells Nginx to act as a reverse proxy.

When a client sends an HTTP request to `stlb01:80`, Nginx forwards that
request to one of the backend application servers.

------------------------------------------------------------------------

## 🧪 Configuration Validation

Before restarting Nginx, the configuration syntax was tested:

``` bash
sudo nginx -t
```

The configuration test succeeded:

``` text
syntax is ok
test is successful
```

This confirmed that Nginx could parse the configuration successfully.

------------------------------------------------------------------------

## 🔄 Nginx Service Restart

Nginx was restarted after the configuration was validated:

``` bash
sudo systemctl restart nginx
```

The service status was then checked:

``` bash
sudo systemctl status nginx
```

The service was confirmed to be:

``` text
Active: active (running)
```

------------------------------------------------------------------------

## 🔍 Final Verification

The application was tested through the Load Balancer:

``` bash
curl http://stlb01:80
```

The response was:

``` text
Welcome to xFusionCorp Industries!
```

This confirmed that:

-   Nginx was listening on port `80`.
-   The load balancer could reach the backend application.
-   The reverse proxy configuration was functioning.
-   The Nautilus application was accessible through `stlb01`.

------------------------------------------------------------------------

## 🧠 Architecture

``` text
                         HTTP Request
                              |
                              v
                     +----------------+
                     |    stlb01      |
                     | Nginx :80      |
                     +----------------+
                              |
                    Load Balancing
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
      +-------------+  +-------------+  +-------------+
      |   stapp01   |  |   stapp02   |  |   stapp03   |
      | Apache :5003|  | Apache :5003|  | Apache :5003|
      +-------------+  +-------------+  +-------------+
```

------------------------------------------------------------------------

## ✅ Final Result

The Day 16 task was successfully completed.

Nginx was configured on `stlb01` as an HTTP load balancer, with all
three application servers configured as backend targets while preserving
their existing Apache port `5003`.

The final HTTP request through:

``` bash
curl http://stlb01:80
```

successfully returned:

``` text
Welcome to xFusionCorp Industries!
```

------------------------------------------------------------------------

## 📚 Key DevOps Concepts Learned

-   Nginx installation and configuration
-   Reverse proxy
-   Load balancing
-   Nginx `upstream` blocks
-   Backend server pools
-   Round-robin load balancing
-   HTTP port `80`
-   Apache backend port preservation
-   Nginx configuration validation
-   Linux systemd service management
-   Application connectivity testing with `curl`

------------------------------------------------------------------------

## 🏁 Challenge Status

**Day 16 --- Completed Successfully ✅**
