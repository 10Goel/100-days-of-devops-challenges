# Day 42 --- Docker Networking: macvlan Network

## 📌 Challenge Overview

This challenge focused on creating and validating a custom Docker
network for the Nautilus DevOps environment.

### Objective

On **App Server 3 (`stapp03`)**, create a Docker network with:

  Configuration      Value
  ------------------ -----------------
  Network Name       `news`
  Driver             `macvlan`
  Parent Interface   `eth0`
  Subnet             `172.28.0.0/24`
  IP Range           `172.28.0.0/24`

## 🏗️ Environment

-   Host: `stapp03`
-   Linux user: `banner`
-   Docker network driver: `macvlan`

## 🚀 Implementation

The network was created using:

``` bash
docker network create \
  --driver macvlan \
  --subnet=172.28.0.0/24 \
  --ip-range=172.28.0.0/24 \
  --opt parent=eth0 \
  news
```

## 🔍 Verification

The configuration was verified with:

``` bash
docker network ls
```

and:

``` bash
docker network inspect news
```

Target configuration:

``` text
Name:       news
Driver:     macvlan
Parent:     eth0
Subnet:     172.28.0.0/24
IP Range:   172.28.0.0/24
```

Additional focused checks:

``` bash
docker network inspect news -f '{{index .Options "parent"}}'
docker network inspect news -f '{{.Driver}}'
docker network inspect news -f '{{(index .IPAM.Config 0).Subnet}}'
docker network inspect news -f '{{(index .IPAM.Config 0).IPRange}}'
```

## 🧠 Key Concepts Demonstrated

-   Docker network architecture
-   Docker bridge networking
-   User-defined networks
-   Network drivers
-   `macvlan` networking
-   Parent network interfaces
-   IPAM
-   Subnets and CIDR notation
-   IP ranges
-   Network inspection
-   Docker network lifecycle
-   Container-to-container communication
-   Network isolation
-   Host networking vs container networking

## 🌐 Docker Networking at a Glance

Docker networking allows containers to communicate with:

1.  Other containers
2.  The Docker host
3.  External networks
4.  Other Docker hosts, depending on the network driver

Common Docker network drivers include:

  -----------------------------------------------------------------------
  Driver                              Typical Use
  ----------------------------------- -----------------------------------
  `bridge`                            Default single-host container
                                      networking

  `host`                              Container shares the host network
                                      namespace

  `none`                              No network connectivity

  `overlay`                           Multi-host/container orchestration
                                      networking

  `macvlan`                           Containers appear directly on the
                                      physical network

  `ipvlan`                            Similar physical-network
                                      integration with different L2/L3
                                      behavior
  -----------------------------------------------------------------------

## 📚 Learning Outcome

This challenge strengthened practical understanding of Docker networking
and demonstrated how a custom `macvlan` network can be configured with
explicit IPAM settings.

The most important lesson is that a Docker network is not simply a name
assigned to containers---it defines the networking model, address space,
connectivity, isolation, and behavior available to workloads.

------------------------------------------------------------------------

**Status:** ✅ Completed\
**Challenge:** KodeKloud 100 Days of DevOps --- Day 42\
**Primary Topic:** Docker Networking
