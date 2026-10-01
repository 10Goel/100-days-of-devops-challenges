# Day 65 — Kubernetes Redis, ConfigMap and Volumes Notes

## 1. What was the main objective?

The objective was to deploy Redis inside Kubernetes while keeping its configuration outside the container image.

Instead of modifying the Redis image, Kubernetes provides the configuration to the container using a **ConfigMap volume**.

This creates a clean separation between:

```text
Application Image
       +
Configuration
```

The Redis image remains reusable while configuration can be changed independently.

---

# 2. Kubernetes ConfigMap

A `ConfigMap` stores non-sensitive configuration data as key-value pairs.

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-redis-config
data:
  redis-config: |
    maxmemory 2mb
```

Here:

```text
ConfigMap Name = my-redis-config
Key            = redis-config
Value          = maxmemory 2mb
```

Conceptually:

```text
my-redis-config
       │
       └── redis-config
               │
               └── maxmemory 2mb
```

A ConfigMap is suitable for normal configuration data.

For credentials, passwords, API keys, or certificates, Kubernetes `Secret` is normally more appropriate.

---

# 3. ConfigMap as a Volume

The ConfigMap was connected to the Pod using:

```yaml
volumes:
  - name: redis-config
    configMap:
      name: my-redis-config
```

This tells Kubernetes:

> Create a Pod volume named `redis-config` whose content comes from the ConfigMap `my-redis-config`.

However, defining a volume alone does not make it visible inside the container.

The volume must also be mounted.

---

# 4. volumeMounts

The ConfigMap volume was mounted into the Redis container:

```yaml
volumeMounts:
  - name: redis-config
    mountPath: /redis-master
```

The ConfigMap key becomes a file.

Therefore:

```text
ConfigMap:
my-redis-config
      │
      └── key: redis-config
              │
              ▼
Container:
 /redis-master/redis-config
```

The file contains:

```text
maxmemory 2mb
```

This is one of the most important Kubernetes ConfigMap concepts:

```text
ConfigMap key → file name
ConfigMap value → file content
```

when a ConfigMap is mounted as a volume.

---

# 5. Why was the Redis command explicitly defined?

The deployment used:

```yaml
command:
  - redis-server
  - /redis-master/redis-config
```

Mounting a configuration file inside the container does not necessarily mean the application will automatically read it.

Redis needs to be started with the configuration file:

```bash
redis-server /redis-master/redis-config
```

Therefore the complete flow is:

```text
ConfigMap
   │
   ▼
ConfigMap-backed Volume
   │
   ▼
/redis-master/redis-config
   │
   ▼
redis-server reads the file
   │
   ▼
maxmemory = 2mb
```

---

# 6. Kubernetes emptyDir Volume

The second volume was:

```yaml
volumes:
  - name: data
    emptyDir: {}
```

and it was mounted at:

```yaml
volumeMounts:
  - name: data
    mountPath: /redis-master-data
```

`emptyDir` is temporary storage created when a Pod is assigned to a node.

Its lifetime is tied to the Pod.

```text
Pod created
    │
    ▼
emptyDir created
    │
    ▼
Container uses volume
    │
    ▼
Pod deleted
    │
    ▼
emptyDir data removed
```

Important:

- Container restart inside the same Pod does not necessarily remove `emptyDir` data.
- Pod deletion removes the `emptyDir` volume.
- It is not persistent storage.

For long-term database persistence, Kubernetes PersistentVolumes and PersistentVolumeClaims are generally used instead.

---

# 7. Difference Between ConfigMap Volume and emptyDir

| Feature | ConfigMap Volume | emptyDir |
|---|---|---|
| Purpose | Provide configuration | Temporary writable storage |
| Data source | ConfigMap object | Created empty by Kubernetes |
| Initial data | ConfigMap keys/values | Empty |
| Writable use | Primarily configuration | Yes |
| Lifetime | ConfigMap exists independently | Tied to Pod |
| Example in this task | Redis config | Redis runtime data |

---

# 8. CPU Resource Request

The Redis container requested:

```yaml
resources:
  requests:
    cpu: "1"
```

A resource **request** tells Kubernetes how much CPU should be considered necessary when scheduling the Pod.

```text
Pod
 │
 │ requests 1 CPU
 ▼
Kubernetes Scheduler
 │
 │ finds a node with enough allocatable CPU
 ▼
Node selected
```

`cpu: "1"` means approximately one full CPU core worth of Kubernetes CPU units.

Another representation would be:

```yaml
cpu: "1000m"
```

where:

```text
1000m = 1 CPU
500m  = 0.5 CPU
250m  = 0.25 CPU
```

---

# 9. Requests vs Limits

A request is not the same as a limit.

### Request

```yaml
requests:
  cpu: "1"
```

Used primarily by the scheduler to decide where the Pod can run.

### Limit

Example:

```yaml
limits:
  cpu: "1"
```

Defines the maximum CPU allocation available to the container.

The task specifically required a CPU **request**, so adding an unnecessary limit was not required.

---

# 10. Why Redis Uses Port 6379

Redis uses TCP port:

```text
6379
```

The manifest declared:

```yaml
ports:
  - containerPort: 6379
```

This documents the port used by the application inside the container.

Important distinction:

```text
containerPort ≠ Service
```

Declaring `containerPort: 6379` does not automatically expose Redis outside the Pod.

A Kubernetes `Service` would normally be required for stable network access from other Pods or external clients.

The challenge only required the container port.

---

# 11. Deployment and Replica Management

The Redis container was managed by a Kubernetes Deployment.

```yaml
kind: Deployment
```

with:

```yaml
replicas: 1
```

Conceptually:

```text
Deployment
    │
    ▼
ReplicaSet
    │
    ▼
Pod
    │
    ▼
Redis Container
```

The Deployment controller ensures the requested number of replicas remain running.

If the Redis Pod disappears, the ReplicaSet managed by the Deployment can create a replacement Pod.

---

# 12. Labels and Selectors

The Deployment used:

```yaml
selector:
  matchLabels:
    app: redis
```

and the Pod template used:

```yaml
labels:
  app: redis
```

These must match.

```text
Deployment selector
     app=redis
         │
         ▼
Pod label
     app=redis
```

A selector mismatch can make a Deployment invalid or prevent it from managing the intended Pods.

---

# 13. Verifying the Configuration Inside the Pod

Checking only:

```bash
kubectl get pods
```

proves that a Pod is running, but it does not prove that Redis received the intended configuration.

A stronger verification is:

```bash
kubectl exec -it <pod-name> -- cat /redis-master/redis-config
```

Expected:

```text
maxmemory 2mb
```

This verifies the ConfigMap volume mount.

---

# 14. Verifying Redis Loaded the Configuration

Even seeing the file is not enough to prove Redis is actively using it.

The Redis runtime can be queried:

```bash
redis-cli CONFIG GET maxmemory
```

The value should be:

```text
2097152
```

because:

```text
2 × 1024 × 1024 = 2097152 bytes
```

This creates a stronger verification chain:

```text
ConfigMap exists
        ↓
Config file mounted
        ↓
Redis process started with file
        ↓
Redis reports maxmemory = 2097152
```

---

# 15. Important Troubleshooting Concepts

## Pod stuck in Pending

Check:

```bash
kubectl describe pod <pod-name>
```

Because the Pod requests an entire CPU, a cluster without enough allocatable CPU may fail to schedule it.

Look for messages such as:

```text
Insufficient cpu
```

---

## Pod enters CrashLoopBackOff

Check:

```bash
kubectl logs <pod-name>
```

Common causes include:

- incorrect Redis command
- missing configuration file
- invalid mount path
- invalid Redis configuration
- malformed YAML

---

## ConfigMap file is missing

Check:

```bash
kubectl describe pod <pod-name>
```

Then confirm:

```bash
kubectl get configmap my-redis-config
```

Typical mistakes:

- wrong ConfigMap name
- incorrect volume name
- incorrect `volumeMounts.name`
- incorrect mount path

---

## Volume name mismatch

These names must correspond:

```yaml
volumeMounts:
  - name: redis-config
```

and:

```yaml
volumes:
  - name: redis-config
```

If they do not match, Kubernetes rejects the Pod specification.

---

# 16. Important Command Pattern

A convenient way to get the Redis Pod dynamically is:

```bash
kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}'
```

This avoids manually copying the generated Pod name.

It can be embedded inside another command:

```bash
kubectl exec -it $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}') -- redis-cli ping
```

This is especially useful because Deployment-managed Pods have dynamically generated names.

---

# 17. Key Takeaways

```text
ConfigMap
   → stores non-sensitive configuration

ConfigMap Volume
   → exposes ConfigMap keys as files

volumeMount
   → makes a volume visible inside a container

emptyDir
   → provides temporary Pod-level storage

Resource Request
   → tells the scheduler how much capacity is required

Deployment
   → maintains the desired number of Pods

containerPort
   → documents the application port inside the container

kubectl exec
   → verifies behavior from inside the running Pod
```

---

## Final Mental Model

```text
                      redis-deployment
                              │
                              ▼
                         Redis Pod
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
             redis-container       Volumes
                    │              /       \
                    │             /         \
                    ▼            ▼           ▼
              redis:alpine    emptyDir     ConfigMap
                    │           data       redis-config
                    │            │             │
                    │            ▼             ▼
                    │    /redis-master-data   /redis-master
                    │                          │
                    │                          ▼
                    │                 redis-config file
                    │                          │
                    └──────────────reads───────┘
                                               │
                                               ▼
                                         maxmemory 2mb
```

**Day 65 successfully reinforced how Kubernetes can combine application deployment, configuration injection, temporary storage, container resources, and runtime validation in one workload.**
