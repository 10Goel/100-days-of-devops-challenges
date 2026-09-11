# Docker Networking --- Complete DevOps Notes

## 1. What Is Docker Networking?

Docker networking provides the communication layer that allows
containers to communicate with:

-   Other containers
-   The Docker host
-   External networks
-   Services outside the Docker environment
-   Other hosts, depending on the selected network driver

Networking determines how containers receive IP addresses, how they
communicate, and how isolated they are.

------------------------------------------------------------------------

# 2. Docker Network Architecture

A simplified model:

``` text
Application
    │
Container
    │
Docker Network
    │
Network Driver
    │
Host Network Interface
    │
Physical / Virtual Network
    │
External Network
```

Docker uses network namespaces and virtual networking components to
provide isolated networking environments to containers.

------------------------------------------------------------------------

# 3. Network Namespace

A Linux network namespace provides an isolated networking environment.

A container normally gets its own:

-   Network interfaces
-   IP addresses
-   Routing table
-   Network namespace

This allows multiple containers to use networking independently.

------------------------------------------------------------------------

# 4. Docker Network Drivers

Docker supports multiple network drivers.

## Bridge

The most common single-host networking model.

``` text
Container A ─┐
Container B ─┼── Docker Bridge ── Host
Container C ─┘
```

Useful for:

-   Local development
-   Application stacks
-   Container-to-container communication on one host

Create:

``` bash
docker network create mybridge
```

------------------------------------------------------------------------

## Host

The container uses the host's network namespace.

``` text
Container
    │
    └── Host Network
```

Create/run example:

``` bash
docker run --network host nginx
```

Advantages:

-   Very low networking overhead
-   Direct access to host networking

Trade-off:

-   Less network isolation
-   Port behavior differs from bridge networking

------------------------------------------------------------------------

## None

The container has no normal network connectivity.

``` bash
docker run --network none alpine
```

Useful when networking should be deliberately disabled.

------------------------------------------------------------------------

## Overlay

Overlay networks provide networking across multiple Docker hosts.

Conceptually:

``` text
Host A                    Host B

Container A               Container B
     │                         │
     └──── Overlay Network ────┘
```

Commonly associated with multi-host Docker environments and
orchestration.

------------------------------------------------------------------------

## Macvlan

`macvlan` allows containers to appear as separate network endpoints on
the physical network.

Conceptually:

``` text
              Physical Network
                     │
                   eth0
                     │
          ┌──────────┴──────────┐
       Container A          Container B
       MAC Address           MAC Address
       IP Address            IP Address
```

This can be useful for workloads that need to appear directly on the
external Layer-2 network.

Day 42 used:

``` text
Driver: macvlan
Parent: eth0
```

------------------------------------------------------------------------

## IPvlan

`ipvlan` is another driver designed for integrating containers with
external networks.

Compared with macvlan, ipvlan can use different Layer-2/Layer-3 behavior
and can reduce the number of MAC addresses exposed to the physical
network.

------------------------------------------------------------------------

# 5. Default Docker Networks

A normal Docker installation commonly provides:

``` bash
docker network ls
```

Typical networks:

``` text
bridge
host
none
```

## bridge

Docker's default bridge network.

## host

Uses the host networking namespace.

## none

Disables normal container networking.

------------------------------------------------------------------------

# 6. User-Defined Networks

A user-defined network is preferable for many application deployments
because it provides explicit network configuration and better
service-to-service organization.

Create:

``` bash
docker network create app-network
```

Connect containers:

``` bash
docker run -d --name web --network app-network nginx
```

Another container can join the same network:

``` bash
docker run -d --name api --network app-network my-api
```

------------------------------------------------------------------------

# 7. Container-to-Container Communication

Containers attached to the same user-defined network can communicate
over that network.

Example:

``` text
web ───── app-network ───── api
```

Modern Docker user-defined networks also provide Docker's built-in
DNS-based service discovery.

For example, an application can often reach another container using its
container/service name rather than hard-coding an IP address.

------------------------------------------------------------------------

# 8. Network Isolation

Different Docker networks can isolate application stacks.

``` text
Network A
├── frontend
└── backend

Network B
├── database
└── monitoring
```

A container only communicates with networks to which it is attached,
subject to routing and application configuration.

This is an important security and architecture concept.

------------------------------------------------------------------------

# 9. Connecting and Disconnecting Containers

Connect:

``` bash
docker network connect app-network container
```

Disconnect:

``` bash
docker network disconnect app-network container
```

A container can be attached to multiple networks.

Example:

``` text
             frontend-network
                    │
                  Web
                    │
             backend-network
                    │
                   API
                    │
             database-network
                    │
                    DB
```

This can be used to create controlled communication paths.

------------------------------------------------------------------------

# 10. Port Publishing vs Docker Networking

These are different concepts.

## Port Publishing

Example:

``` bash
docker run -p 8080:80 nginx
```

Meaning:

``` text
Host port 8080 → Container port 80
```

It exposes a container service through the host.

## Docker Network Communication

Containers on the same Docker network can communicate internally without
necessarily publishing their ports to the host.

This distinction is important:

``` text
-p 8080:80
```

is about exposing a container port through the host.

Whereas:

``` text
--network app-network
```

is about connecting the container to a Docker network.

------------------------------------------------------------------------

# 11. IP Address Management --- IPAM

Docker uses IPAM to manage addresses assigned to containers and
networks.

A network can specify:

-   Subnet
-   Gateway
-   IP range
-   IP allocation behavior

Example:

``` bash
docker network create \
  --subnet=172.30.0.0/16 \
  mynetwork
```

Inspect:

``` bash
docker network inspect mynetwork
```

------------------------------------------------------------------------

# 12. CIDR and Subnets

Day 42 used:

``` text
172.28.0.0/24
```

The `/24` represents the network prefix length.

A typical IPv4 `/24` contains:

``` text
256 total addresses
```

from:

``` text
172.28.0.0
```

through:

``` text
172.28.0.255
```

The exact usable allocation behavior depends on the networking
implementation and reserved addresses.

------------------------------------------------------------------------

# 13. IP Range

Docker can define a specific allocation range within a network.

Example:

``` bash
--ip-range=172.28.0.0/24
```

This tells Docker which IP address range should be used for container
allocation.

Day 42 deliberately configured the IP range to match the subnet:

``` text
Subnet:  172.28.0.0/24
IPRange: 172.28.0.0/24
```

------------------------------------------------------------------------

# 14. Gateway

A gateway is the routing point through which traffic can leave the local
subnet.

Example:

``` bash
docker network create \
  --subnet=172.30.0.0/24 \
  --gateway=172.30.0.1 \
  app-network
```

Inspect it:

``` bash
docker network inspect app-network
```

------------------------------------------------------------------------

# 15. Macvlan Networking

## What Is Macvlan?

Macvlan is a Linux networking technology that allows multiple virtual
network interfaces to be created on top of a physical parent interface.

Docker can use it to give containers their own MAC addresses and IP
addresses on the attached Layer-2 network.

Example:

``` text
Physical / Virtual Network
          │
        eth0
          │
    ┌─────┼─────┐
    │     │     │
   C1    C2    C3
 MAC1   MAC2   MAC3
 IP1    IP2    IP3
```

------------------------------------------------------------------------

# 16. Macvlan Parent Interface

The parent interface is the host interface used by the macvlan network.

Day 42:

``` text
Parent: eth0
```

Configured with:

``` bash
--opt parent=eth0
```

The parent interface must exist on the host.

------------------------------------------------------------------------

# 17. Creating a Macvlan Network

General pattern:

``` bash
docker network create \
  --driver macvlan \
  --subnet=<SUBNET> \
  --gateway=<GATEWAY> \
  --ip-range=<IP_RANGE> \
  --opt parent=<PARENT_INTERFACE> \
  <NETWORK_NAME>
```

Day 42:

``` bash
docker network create \
  --driver macvlan \
  --subnet=172.28.0.0/24 \
  --ip-range=172.28.0.0/24 \
  --opt parent=eth0 \
  news
```

------------------------------------------------------------------------

# 18. Macvlan Advantages

Potential benefits:

-   Containers can appear directly on the physical network
-   Containers can have unique MAC addresses
-   Useful for legacy applications expecting direct network presence
-   Can integrate workloads into existing Layer-2 networks

------------------------------------------------------------------------

# 19. Macvlan Considerations

Macvlan is not automatically the best choice for every application.

Important considerations include:

-   Physical network configuration
-   Switch/security policies
-   MAC address limits
-   Host-to-macvlan-container communication behavior
-   IP address planning
-   Network isolation requirements

A common Linux macvlan behavior is that the host cannot communicate
directly with its macvlan child interfaces through the parent interface
in the same straightforward way as normal bridge networking. A host-side
macvlan interface or another architecture may be required when
host-to-container communication is needed.

------------------------------------------------------------------------

# 20. Docker Network Inspection

The most important troubleshooting command:

``` bash
docker network inspect <network>
```

Example:

``` bash
docker network inspect news
```

It can reveal:

-   Network name
-   Network ID
-   Driver
-   Scope
-   IPAM configuration
-   Subnet
-   IP range
-   Gateway
-   Connected containers
-   Driver-specific options

------------------------------------------------------------------------

# 21. Useful Inspection Patterns

Driver:

``` bash
docker network inspect news -f '{{.Driver}}'
```

Parent interface:

``` bash
docker network inspect news -f '{{index .Options "parent"}}'
```

Subnet:

``` bash
docker network inspect news -f '{{(index .IPAM.Config 0).Subnet}}'
```

IP range:

``` bash
docker network inspect news -f '{{(index .IPAM.Config 0).IPRange}}'
```

------------------------------------------------------------------------

# 22. Network Lifecycle

A typical Docker networking workflow:

``` text
Create
  ↓
Inspect
  ↓
Connect containers
  ↓
Run workloads
  ↓
Disconnect containers
  ↓
Remove network
```

Commands:

``` bash
docker network create
docker network inspect
docker network connect
docker network disconnect
docker network rm
```

------------------------------------------------------------------------

# 23. Network Troubleshooting Checklist

When a container cannot communicate:

### Step 1 --- Check the container

``` bash
docker ps
```

### Step 2 --- Inspect its network configuration

``` bash
docker inspect <container>
```

### Step 3 --- List networks

``` bash
docker network ls
```

### Step 4 --- Inspect the relevant network

``` bash
docker network inspect <network>
```

### Step 5 --- Confirm the containers share a network

``` text
Container A ── same network ── Container B
```

### Step 6 --- Check IP/subnet configuration

Verify:

-   IP address
-   Subnet
-   Gateway
-   IP range

### Step 7 --- Check host interfaces

On systems with the `ip` utility:

``` bash
ip addr
```

``` bash
ip route
```

On Day 42's server, `/proc/net/dev` was used because `ip` was
unavailable:

``` bash
cat /proc/net/dev
```

------------------------------------------------------------------------

# 24. Bridge vs Host vs Macvlan

  ---------------------------------------------------------------------------
  Feature           Bridge            Host                  Macvlan
  ----------------- ----------------- --------------------- -----------------
  Container         High              Low                   Depends on design
  isolation                                                 

  Own container IP  Yes               No separate namespace Yes
                                      IP                    

  Own MAC address   Virtual Docker    Host MAC              Typically unique
                    networking                              MAC

  Multi-container   Excellent         Possible              Specialized
  app use                                                   

  Direct physical   No                Host's presence       Yes
  LAN presence                                              

  Common default    Yes               No                    No
  choice                                                    

  Typical use       Application       Performance/special   LAN-integrated
                    stacks            cases                 workloads
  ---------------------------------------------------------------------------

------------------------------------------------------------------------

# 25. Important DevOps Mental Model

Think about Docker networking in layers:

``` text
Application
    ↓
Container Port
    ↓
Container Network Namespace
    ↓
Docker Network
    ↓
Network Driver
    ↓
Host Interface
    ↓
Physical / Virtual Network
```

Understanding this model makes Docker networking troubleshooting much
easier.

------------------------------------------------------------------------

# 26. Day 42 Practical Summary

The completed task created:

``` text
Network Name:  news
Driver:        macvlan
Parent:        eth0
Subnet:        172.28.0.0/24
IP Range:      172.28.0.0/24
```

Creation command:

``` bash
docker network create \
  --driver macvlan \
  --subnet=172.28.0.0/24 \
  --ip-range=172.28.0.0/24 \
  --opt parent=eth0 \
  news
```

Verification:

``` bash
docker network inspect news
```

------------------------------------------------------------------------

## 🎯 Key Takeaways

1.  Docker networking controls how containers communicate.
2.  Network drivers define the underlying networking behavior.
3.  User-defined networks are important for organized application
    networking.
4.  Bridge networking is the common single-host application model.
5.  Host networking removes much of the network isolation.
6.  Overlay networking enables multi-host networking scenarios.
7.  Macvlan allows containers to appear directly on a Layer-2 network.
8.  IPAM controls subnet and address allocation.
9.  CIDR notation is fundamental to Docker network configuration.
10. `docker network inspect` is one of the most useful networking
    troubleshooting commands.
11. Port publishing and Docker network connectivity are different
    concepts.
12. A container can be attached to multiple Docker networks.
13. Network design is also a security and isolation decision.

------------------------------------------------------------------------

**Day 42 Status: ✅ Completed**

**Primary Skill:** Docker Networking\
**Hands-on Focus:** Custom macvlan network + IPAM configuration\
**Environment:** App Server 3 (`stapp03`)
