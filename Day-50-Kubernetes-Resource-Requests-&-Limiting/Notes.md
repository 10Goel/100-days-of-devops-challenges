# Day 50 — Kubernetes Resource Requests and Limits Notes

## 1. Why Resource Management Is Needed

A Kubernetes worker node may run many Pods at the same time.

Without resource controls, one workload could consume excessive CPU or memory and negatively affect other workloads running on the same node.

Kubernetes provides container-level resource configuration using:

```yaml
resources:
  requests:
    cpu: ...
    memory: ...
  limits:
    cpu: ...
    memory: ...
```

---

## 2. Resource Requests

A **resource request** represents the amount of CPU or memory Kubernetes should account for when scheduling a container.

Example:

```yaml
requests:
  cpu: "100m"
  memory: "15Mi"
```

The scheduler uses these values when evaluating whether a node has enough allocatable capacity for the Pod.

```text
Pod
 |
 v
Resource Requests
 |
 v
Kubernetes Scheduler
 |
 v
Checks candidate nodes
 |
 v
Selects a suitable node
```

---

## 3. Resource Limits

A **resource limit** defines the maximum amount of a resource that a container is allowed to consume.

Example:

```yaml
limits:
  cpu: "100m"
  memory: "20Mi"
```

Limits are enforced at runtime through Linux resource-control mechanisms used by the container runtime and Kubernetes.

---

## 4. Requests vs Limits

This distinction is fundamental:

```text
REQUEST
   |
   +--> Used for scheduling and capacity accounting

LIMIT
   |
   +--> Upper bound on container resource consumption
```

For Day 50:

```text
              httpd-container
                     |
          +----------+----------+
          |                     |
       Requests                Limits
          |                     |
      CPU: 100m             CPU: 100m
      RAM: 15Mi             RAM: 20Mi
```

---

## 5. CPU Units

Kubernetes represents fractional CPU using millicores.

```text
1000m = 1 CPU
500m  = 0.5 CPU
250m  = 0.25 CPU
100m  = 0.1 CPU
```

Therefore:

```text
100m = 10% of one CPU core
```

---

## 6. CPU Is a Compressible Resource

CPU can be throttled.

If a container attempts to consume more CPU than its configured CPU limit:

```text
Container requests extra CPU
          |
          v
CPU limit reached
          |
          v
CPU throttling occurs
          |
          v
Application continues running
but receives less CPU time
```

The container is normally not killed merely because it attempts to exceed its CPU limit.

---

## 7. Memory Units

Common Kubernetes memory units include:

```text
Ki = Kibibytes
Mi = Mebibytes
Gi = Gibibytes
```

For Day 50:

```text
Memory Request = 15Mi
Memory Limit   = 20Mi
```

---

## 8. Memory Is a Non-Compressible Resource

Memory cannot simply be slowed down the way CPU can.

If a process exceeds the enforced memory limit and sufficient memory cannot be reclaimed, the process may be terminated.

This often appears as:

```text
OOMKilled
```

OOM means:

```text
Out Of Memory
```

Conceptually:

```text
Memory limit reached
        |
        v
Container requires more memory
        |
        v
Memory cannot be reclaimed
        |
        v
Process may be OOM-killed
```

---

## 9. CPU Throttling vs Memory OOM

| Resource | Typical Effect When Limit Is Reached |
|---|---|
| CPU | Throttling |
| Memory | Possible OOM termination |

Remember:

```text
CPU over limit
    ↓
Throttle

Memory over limit
    ↓
Possible kill
```

---

## 10. How the Scheduler Uses Requests

The Kubernetes scheduler primarily considers **resource requests** when deciding whether a Pod fits on a node.

Example:

```text
Node allocatable CPU     = 500m
Existing CPU requests    = 350m
New Pod CPU request      = 100m
```

Calculation:

```text
350m + 100m = 450m
```

Since:

```text
450m <= 500m
```

the Pod may fit on the node.

If the new Pod requested `200m`:

```text
350m + 200m = 550m
```

That exceeds the available allocatable CPU, so the scheduler would need another suitable node. If none exists, the Pod can remain `Pending`.

---

## 11. Requests Do Not Mean Constant Usage

A request does not mean the container constantly consumes that amount.

For example:

```text
CPU Request = 100m
```

Actual CPU usage could vary:

```text
20m
40m
70m
100m
```

The request is mainly used for scheduling and resource-accounting purposes.

---

## 12. Why Limits Matter in Shared Nodes

Consider:

```text
Worker Node
├── Pod A
├── Pod B
├── Pod C
└── Pod D
```

If Pod A has no appropriate resource boundaries and consumes excessive resources, Pods B, C, and D may be affected.

Proper resource controls reduce this **noisy-neighbor** problem.

---

## 13. Day 50 Pod Manifest

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

---

## 14. Manifest Breakdown

### `apiVersion`

```yaml
apiVersion: v1
```

The Pod resource belongs to the Kubernetes core `v1` API.

### `kind`

```yaml
kind: Pod
```

Creates a Pod directly.

### Pod Name

```yaml
metadata:
  name: httpd-pod
```

### Container Name

```yaml
name: httpd-container
```

### Container Image

```yaml
image: httpd:latest
```

### Requests

```yaml
requests:
  memory: "15Mi"
  cpu: "100m"
```

### Limits

```yaml
limits:
  memory: "20Mi"
  cpu: "100m"
```

---

## 15. Inspecting Resource Configuration

Human-readable inspection:

```bash
kubectl describe pod httpd-pod
```

Exact values using JSONPath:

```bash
kubectl get pod httpd-pod   -o jsonpath='{.spec.containers[0].resources.requests.cpu}'
```

The same technique can be used for memory requests and both limits.

---

## 16. Resource Configuration in Production

Appropriate requests and limits help with:

- Capacity planning
- Stable scheduling
- Resource isolation
- Predictable application behavior
- Cluster utilization
- Cost optimization
- Troubleshooting
- Reducing noisy-neighbor problems
- Autoscaling design

Incorrect settings can also cause problems. Requests that are unnecessarily high can waste schedulable capacity, while limits that are too restrictive can throttle applications or cause OOM failures.

---

## 17. Day 50 Resource Hierarchy

```text
Kubernetes Cluster
      |
      v
Worker Node
      |
      v
httpd-pod
      |
      v
httpd-container
      |
      +--------------------------+
      |                          |
      v                          v
Requests                     Limits
CPU: 100m                    CPU: 100m
Memory: 15Mi                 Memory: 20Mi
```

---

## 18. Important Commands

```bash
kubectl get pod httpd-pod
kubectl get pod httpd-pod -o wide
kubectl describe pod httpd-pod
kubectl get pod httpd-pod -o yaml
```

Verify CPU request:

```bash
kubectl get pod httpd-pod   -o jsonpath='{.spec.containers[0].resources.requests.cpu}'; echo
```

Verify memory request:

```bash
kubectl get pod httpd-pod   -o jsonpath='{.spec.containers[0].resources.requests.memory}'; echo
```

Verify CPU limit:

```bash
kubectl get pod httpd-pod   -o jsonpath='{.spec.containers[0].resources.limits.cpu}'; echo
```

Verify memory limit:

```bash
kubectl get pod httpd-pod   -o jsonpath='{.spec.containers[0].resources.limits.memory}'; echo
```

---

## 19. Interview-Level Definitions

```text
Resource Request
= Amount Kubernetes uses for scheduling and resource accounting.
```

```text
Resource Limit
= Maximum configured CPU or memory available to the container.
```

```text
CPU limit exceeded
= Container can be throttled.
```

```text
Memory limit exceeded
= Container process may be OOM-killed.
```

```text
100m CPU
= 0.1 CPU core.
```

---

## 20. Key Takeaway

> **Requests are primarily about scheduling; limits are primarily about runtime resource control.**

For Day 50:

```text
httpd-container
│
├── CPU
│   ├── Request: 100m
│   └── Limit:   100m
│
└── Memory
    ├── Request: 15Mi
    └── Limit:   20Mi
```

This concept is essential for Kubernetes administration, workload reliability, performance troubleshooting, cluster capacity planning, and DevOps interviews.
