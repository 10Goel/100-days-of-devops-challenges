# Day 42 --- Docker Networking Commands

This file contains the commands used during the challenge, along with
commonly useful Docker networking commands for future DevOps work.

------------------------------------------------------------------------

## 1. Connect to App Server 3

``` bash
ssh banner@stapp03
```

Verify the host:

``` bash
hostname
```

Expected:

``` text
stapp03
```

------------------------------------------------------------------------

## 2. Identify Network Interfaces

The `ip` utility was unavailable on the server, so `/proc/net/dev` was
used:

``` bash
cat /proc/net/dev
```

The active Docker parent interface identified for this task was:

``` text
eth0
```

If available on another Linux system, useful alternatives include:

``` bash
ip addr
```

``` bash
ip -br addr
```

------------------------------------------------------------------------

## 3. List Docker Networks

``` bash
docker network ls
```

This displays existing Docker networks and their drivers.

------------------------------------------------------------------------

## 4. Create the macvlan Network

``` bash
docker network create \
  --driver macvlan \
  --subnet=172.28.0.0/24 \
  --ip-range=172.28.0.0/24 \
  --opt parent=eth0 \
  news
```

### Command breakdown

``` text
--driver macvlan
```

Selects the `macvlan` network driver.

``` text
--subnet=172.28.0.0/24
```

Defines the Docker network's subnet.

``` text
--ip-range=172.28.0.0/24
```

Defines the IP allocation range.

``` text
--opt parent=eth0
```

Associates the macvlan network with the host's `eth0` interface.

``` text
news
```

Names the network.

------------------------------------------------------------------------

## 5. Inspect a Network

``` bash
docker network inspect news
```

This is the primary troubleshooting and verification command.

------------------------------------------------------------------------

## 6. Check the Parent Interface

``` bash
docker network inspect news -f '{{index .Options "parent"}}'
```

Expected:

``` text
eth0
```

------------------------------------------------------------------------

## 7. Check the Network Driver

``` bash
docker network inspect news -f '{{.Driver}}'
```

Expected:

``` text
macvlan
```

------------------------------------------------------------------------

## 8. Check the Subnet

``` bash
docker network inspect news -f '{{(index .IPAM.Config 0).Subnet}}'
```

Expected:

``` text
172.28.0.0/24
```

------------------------------------------------------------------------

## 9. Check the IP Range

``` bash
docker network inspect news -f '{{(index .IPAM.Config 0).IPRange}}'
```

Expected:

``` text
172.28.0.0/24
```

------------------------------------------------------------------------

# Useful Docker Network Commands

## Create a Default User-Defined Bridge Network

``` bash
docker network create mynetwork
```

------------------------------------------------------------------------

## Create a Bridge Network Explicitly

``` bash
docker network create \
  --driver bridge \
  mybridge
```

------------------------------------------------------------------------

## Create a Network with a Custom Subnet

``` bash
docker network create \
  --driver bridge \
  --subnet=172.30.0.0/16 \
  mynetwork
```

------------------------------------------------------------------------

## Create a Network with a Gateway

``` bash
docker network create \
  --driver bridge \
  --subnet=172.30.0.0/16 \
  --gateway=172.30.0.1 \
  mynetwork
```

------------------------------------------------------------------------

## Attach a Running Container to a Network

``` bash
docker network connect mynetwork container_name
```

------------------------------------------------------------------------

## Disconnect a Container

``` bash
docker network disconnect mynetwork container_name
```

------------------------------------------------------------------------

## Inspect a Container's Network Configuration

``` bash
docker inspect container_name
```

Useful filtered form:

``` bash
docker inspect -f '{{json .NetworkSettings.Networks}}' container_name
```

------------------------------------------------------------------------

## Remove a Network

``` bash
docker network rm mynetwork
```

The network must generally have no connected containers.

------------------------------------------------------------------------

## Remove Unused Networks

``` bash
docker network prune
```

Be careful: this removes unused Docker networks.

------------------------------------------------------------------------

## List Network Details

``` bash
docker network ls
```

------------------------------------------------------------------------

## Inspect the Default Bridge

``` bash
docker network inspect bridge
```

------------------------------------------------------------------------

# Common Troubleshooting Commands

Check Docker status:

``` bash
systemctl status docker
```

Check Docker information:

``` bash
docker info
```

Check containers:

``` bash
docker ps
```

Check all containers:

``` bash
docker ps -a
```

Inspect a container:

``` bash
docker inspect <container>
```

Inspect a network:

``` bash
docker network inspect <network>
```

------------------------------------------------------------------------

## Day 42 Final Verification

``` bash
docker network inspect news -f '{{index .Options "parent"}}'
docker network inspect news -f '{{.Driver}}'
docker network inspect news -f '{{(index .IPAM.Config 0).Subnet}}'
docker network inspect news -f '{{(index .IPAM.Config 0).IPRange}}'
```

Expected:

``` text
eth0
macvlan
172.28.0.0/24
172.28.0.0/24
```

------------------------------------------------------------------------

**Status:** ✅ Day 42 completed successfully
