# DevOps Challenge — Day 50: Kubernetes Resource Requests and Limits

## Overview

Day 50 focused on **Kubernetes container resource management**.

The objective was to create an Apache HTTP Server Pod with explicitly defined **CPU and memory requests and limits** so that Kubernetes can schedule the workload correctly and prevent it from consuming more resources than intended.

This challenge is important because resource control is one of the core practices for running stable and predictable workloads in Kubernetes.

---

## Task Requirements

Create a Pod with the following configuration:

| Property | Required Value |
|---|---|
| Pod Name | `httpd-pod` |
| Container Name | `httpd-container` |
| Image | `httpd:latest` |
| CPU Request | `100m` |
| Memory Request | `15Mi` |
| CPU Limit | `100m` |
| Memory Limit | `20Mi` |

The `kubectl` utility was already configured on the jump host.

---

## Solution

A Kubernetes YAML manifest was created to define the Pod and its container-level resource requirements.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
spec:
  containers:
    - name: httpd-container
      image: httpd:latest
      resources:
        requests:
          memory: "15Mi"
          cpu: "100m"
        limits:
          memory: "20Mi"
          cpu: "100m"
```

The Pod was created using:

```bash
kubectl apply -f /tmp/httpd-pod.yaml
```

---

## Core Concept: Resource Requests and Limits

Kubernetes resource configuration has two main parts:

```text
resources
├── requests
│   ├── cpu
│   └── memory
└── limits
    ├── cpu
    └── memory
```

### Requests

A resource **request** is the amount of CPU or memory Kubernetes considers necessary for the container when deciding where to schedule the Pod.

For this challenge:

```text
CPU Request    = 100m
Memory Request = 15Mi
```

### Limits

A resource **limit** defines the maximum amount of CPU or memory that the container is allowed to consume.

For this challenge:

```text
CPU Limit    = 100m
Memory Limit = 20Mi
```

---

## Requests vs Limits

| Setting | Main Purpose |
|---|---|
| CPU Request | Used by the scheduler for placement decisions |
| Memory Request | Used by the scheduler for placement decisions |
| CPU Limit | Caps CPU usage through throttling |
| Memory Limit | Caps memory usage; exceeding it can result in OOM termination |

Conceptually:

```text
                    httpd-container
                           |
              +------------+------------+
              |                         |
           Requests                    Limits
              |                         |
       Scheduling need             Usage ceiling
              |                         |
       CPU: 100m                  CPU: 100m
       RAM: 15Mi                  RAM: 20Mi
```

---

## CPU Units in Kubernetes

CPU is commonly expressed in **millicores**:

```text
1000m = 1 CPU core
500m  = 0.5 CPU core
250m  = 0.25 CPU core
100m  = 0.1 CPU core
```

Therefore:

```text
100m = 10% of one CPU core
```

---

## Memory Units in Kubernetes

Memory is commonly expressed using binary units:

```text
Ki = Kibibytes
Mi = Mebibytes
Gi = Gibibytes
```

For this challenge:

```text
Memory Request = 15Mi
Memory Limit   = 20Mi
```

---

## What Happens When a Limit Is Reached?

### CPU

CPU is a compressible resource. If the container tries to use more than its CPU limit, Linux cgroup controls can throttle the container.

```text
CPU usage exceeds limit
        |
        v
CPU throttling
        |
        v
Application continues running, but more slowly
```

### Memory

Memory is treated differently. If a container exceeds its memory limit and enough memory cannot be reclaimed, its process may be terminated with an Out Of Memory event.

```text
Memory usage exceeds limit
        |
        v
Memory pressure
        |
        v
Process may be OOM-killed
```

---

## Verification

Check Pod status:

```bash
kubectl get pod httpd-pod
```

Verify the container and image:

```bash
kubectl get pod httpd-pod   -o jsonpath='{.spec.containers[0].name}{"\n"}{.spec.containers[0].image}{"\n"}'
```

Verify resource configuration:

```bash
kubectl describe pod httpd-pod
```

Expected resource section:

```text
Limits:
  cpu:     100m
  memory:  20Mi

Requests:
  cpu:     100m
  memory:  15Mi
```

---

## Why Resource Limitation Matters

Without appropriate resource configuration, one container can consume excessive node resources and affect neighboring workloads.

Requests and limits help with:

- Predictable Pod scheduling
- Workload isolation
- Cluster stability
- Capacity planning
- Performance control
- Fair resource sharing
- Reducing noisy-neighbor problems
- Production readiness

---

## Completed Configuration

```text
httpd-pod
└── httpd-container
    ├── Image: httpd:latest
    │
    └── Resources
        ├── Requests
        │   ├── CPU: 100m
        │   └── Memory: 15Mi
        │
        └── Limits
            ├── CPU: 100m
            └── Memory: 20Mi
```

---

## Result

The Day 50 challenge was completed successfully with the required resource configuration:

```text
Pod             = httpd-pod
Container       = httpd-container
Image           = httpd:latest

CPU Request     = 100m
Memory Request  = 15Mi

CPU Limit       = 100m
Memory Limit    = 20Mi
```

---
## Key Takeaway

> **Requests tell Kubernetes what a workload needs for scheduling, while limits define an upper boundary on what the container may consume.**

Understanding this distinction is fundamental to building stable and efficient Kubernetes workloads.
