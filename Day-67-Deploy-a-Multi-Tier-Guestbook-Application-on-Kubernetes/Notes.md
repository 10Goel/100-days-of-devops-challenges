# Day 67 — Kubernetes Multi-Tier Guestbook Application Notes

## 1. What Was the Main Objective?

Day 67 focused on deploying a complete multi-tier application rather than a single isolated Pod or Deployment.

The application had three logical layers:

```text
Frontend
   ↓
Redis Services
   ↓
Redis Master + Redis Slaves
```

This introduced real application-to-application communication inside Kubernetes.

---

# 2. Multi-Tier Application Architecture

A multi-tier application separates responsibilities into independent components.

In this challenge:

```text
Frontend Tier
    ↓
Caching / Data Tier
    ↓
Redis Master + Redis Slaves
```

The frontend served user traffic while Redis handled application data.

This design improves:

- scalability
- separation of concerns
- independent deployment
- service discovery
- maintainability

---

# 3. Redis Master Deployment

The Redis master Deployment used one replica:

```yaml
replicas: 1
```

Its main role was to receive write operations.

Conceptually:

```text
Frontend
   ↓
redis-master Service
   ↓
Redis Master Pod
```

The Service provides a stable endpoint even though the Pod IP can change.

---

# 4. Redis Slave Deployment

The Redis slave Deployment used:

```yaml
replicas: 2
```

This created two Pods.

```text
redis-slave Deployment
      │
      ├── Redis Slave Pod 1
      └── Redis Slave Pod 2
```

Multiple replicas provide additional backend capacity and redundancy.

---

# 5. Replica Management

The challenge used:

```text
Redis Master = 1 replica
Redis Slave  = 2 replicas
Frontend     = 3 replicas
```

Total:

```text
1 + 2 + 3 = 6 Pods
```

Deployments maintain the requested replica count.

If one frontend Pod is deleted:

```text
Desired replicas = 3
Actual replicas  = 2
        ↓
Deployment detects mismatch
        ↓
New Pod created
```

This is Kubernetes self-healing in action.

---

# 6. Resource Requests

Each application container requested:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "100Mi"
```

A request tells the scheduler how much capacity the container needs.

### CPU

```text
100m = 0.1 CPU
1000m = 1 CPU
```

### Memory

```text
100Mi = 100 mebibytes
```

Requests affect scheduling.

They are not the same as resource limits.

---

# 7. Requests vs Limits

### Request

```yaml
requests:
  cpu: "100m"
  memory: "100Mi"
```

Used by the scheduler when choosing a node.

### Limit

Example:

```yaml
limits:
  cpu: "500m"
  memory: "256Mi"
```

Sets the maximum allowed resource usage.

The task specifically required requests, so limits were not necessary.

---

# 8. Kubernetes Services

Pods are temporary.

Their IP addresses may change when Pods are recreated.

A Service provides a stable network identity.

```text
Pod IP
may change

Service IP / DNS name
remains stable
```

This is why applications should normally communicate through Services rather than Pod IPs.

---

# 9. Redis Master Service

The service:

```text
redis-master
```

exposed port:

```text
6379
```

Flow:

```text
Application
   ↓
redis-master
   ↓
Redis Master Pod
```

The frontend can use the Service name as a DNS hostname.

---

# 10. Redis Slave Service

The service:

```text
redis-slave
```

selected Pods with:

```text
app=redis-slave
```

Therefore both Redis slave Pods became endpoints behind the Service.

```text
redis-slave
     │
     ├── Slave Pod 1
     └── Slave Pod 2
```

---

# 11. Redis Follower Service

The challenge also required:

```text
redis-follower
```

This Service selected the same Redis slave Pods.

```text
redis-follower
     │
     ├── Slave Pod 1
     └── Slave Pod 2
```

The important idea is that multiple Services can point to the same set of Pods if their selectors match.

---

# 12. Labels and Selectors

Labels identify Kubernetes resources.

Example:

```yaml
labels:
  app: redis-slave
```

A Service finds matching Pods using:

```yaml
selector:
  app: redis-slave
```

Flow:

```text
Service selector
 app=redis-slave
        ↓
Matches Pods
        ↓
Traffic forwarded
```

If the selector is wrong, the Service will have no valid endpoints.

---

# 13. Service Endpoints

A Service does not automatically guarantee connectivity.

The Service must have matching backend endpoints.

Useful command:

```bash
kubectl get endpoints
```

Example:

```text
redis-master   <pod-ip>:6379
redis-slave    <pod-ip>:6379,<pod-ip>:6379
frontend       <pod-ip>:80,...
```

If a Service shows no endpoints, investigate:

- Pod labels
- Service selectors
- Pod readiness
- namespace mismatch

---

# 14. Kubernetes DNS

The task required:

```text
GET_HOSTS_FROM=dns
```

This tells the application to resolve backend services using Kubernetes DNS.

Kubernetes Services receive DNS names.

For example:

```text
redis-master
redis-slave
redis-follower
```

A Pod in the same namespace can resolve these names directly.

---

# 15. Why DNS Is Better Than Pod IPs

Pod IPs are ephemeral.

Example:

```text
Old Pod IP: 10.244.1.5
Pod recreated
New Pod IP: 10.244.2.7
```

If applications hardcode Pod IPs, connectivity breaks.

With a Service:

```text
redis-master
```

the DNS name remains stable.

```text
Application
    ↓
redis-master
    ↓
Current backend Pod
```

---

# 16. GET_HOSTS_FROM Environment Variable

The Redis slave and frontend containers used:

```yaml
env:
  - name: GET_HOSTS_FROM
    value: "dns"
```

Inside the container:

```text
GET_HOSTS_FROM=dns
```

The application then uses DNS-based service discovery rather than legacy host-file lookup.

---

# 17. ClusterIP Services

The Redis services defaulted to:

```text
ClusterIP
```

A ClusterIP Service is reachable inside the Kubernetes cluster.

Example:

```text
Frontend Pod
    ↓
redis-master ClusterIP Service
    ↓
Redis Master Pod
```

It is not intended for direct external user access.

---

# 18. NodePort Service

The frontend needed external access.

The Service used:

```yaml
type: NodePort
```

with:

```text
nodePort: 30009
```

Traffic flow:

```text
User
 ↓
NodeIP:30009
 ↓
Frontend Service
 ↓
Frontend Pod
```

---

# 19. `port`, `targetPort`, and `nodePort`

Frontend:

```yaml
port: 80
targetPort: 80
nodePort: 30009
```

Meaning:

```text
NodePort 30009
      ↓
Service Port 80
      ↓
Pod Port 80
```

These fields represent different networking layers.

---

# 20. Frontend Load Distribution

The frontend Deployment had three replicas.

The frontend Service selected all three.

```text
frontend Service
      │
      ├── Frontend Pod 1
      ├── Frontend Pod 2
      └── Frontend Pod 3
```

Kubernetes distributes traffic across healthy endpoints.

This provides basic load distribution and availability.

---

# 21. Pod-to-Service Communication

A major learning point from Day 67 is the network path between application tiers.

Example:

```text
Frontend Pod
      ↓
DNS lookup: redis-master
      ↓
Redis Master Service
      ↓
Redis Master Pod
```

For follower traffic:

```text
Frontend Pod
      ↓
redis-follower
      ↓
Redis Slave Pods
```

The frontend does not need to know individual Redis Pod IP addresses.

---

# 22. Why Services Are Essential in Distributed Applications

Without Services:

```text
Application
   ↓
Hardcoded Pod IP
   ↓
Pod recreated
   ↓
IP changes
   ↓
Application breaks
```

With Services:

```text
Application
   ↓
Stable Service name
   ↓
Kubernetes finds current Pod
```

This is a foundational Kubernetes design pattern.

---

# 23. Deployment Self-Healing

Suppose one frontend Pod fails.

```text
frontend replicas desired = 3
frontend replicas actual  = 2
```

The Deployment controller creates another Pod.

This gives:

```text
Desired state
      ↓
Controller compares
      ↓
Actual state corrected
```

This reconciliation loop is central to Kubernetes.

---

# 24. Why the Exact Frontend Image Digest Matters

The task provided a digest-based image:

```text
gcr.io/google-samples/gb-frontend@sha256:...
```

A digest identifies an exact immutable image.

Compared with a tag:

```text
image: app:latest
```

a digest is deterministic:

```text
same digest
   ↓
same image content
```

This improves reproducibility.

---

# 25. Image Tags vs Digests

### Tag

```text
redis:latest
```

A tag may point to different image content over time.

### Digest

```text
image@sha256:...
```

A digest identifies a specific image manifest/content.

For production deployments, immutable image references are often safer.

---

# 26. Useful Validation Strategy

Do not validate only with:

```bash
kubectl get pods
```

A stronger sequence is:

```text
1. Deployments ready?
2. Pods running?
3. Services created?
4. Endpoints populated?
5. Environment variables correct?
6. DNS resolving?
7. Redis responding?
8. Frontend exposed?
```

This validates both infrastructure and application connectivity.

---

# 27. Troubleshooting: Service Has No Endpoints

If:

```bash
kubectl get endpoints redis-master
```

shows no backend address, check:

```bash
kubectl get pods --show-labels
kubectl describe service redis-master
```

Typical cause:

```text
Service selector != Pod label
```

---

# 28. Troubleshooting: ImagePullBackOff

If a Pod shows:

```text
ImagePullBackOff
```

run:

```bash
kubectl describe pod <pod-name>
```

Common causes:

- wrong image name
- incorrect digest
- registry issue
- network issue
- authentication requirement

---

# 29. Troubleshooting: CrashLoopBackOff

Check:

```bash
kubectl logs <pod-name>
kubectl describe pod <pod-name>
```

Possible causes:

- application startup failure
- incorrect environment variable
- backend dependency unavailable
- invalid configuration

---

# 30. Troubleshooting: DNS

If application-to-service communication fails:

```bash
kubectl exec -it <pod-name> -- getent hosts redis-master
```

or use another DNS utility available in the image.

Check:

```bash
kubectl get svc
kubectl get endpoints
```

DNS resolution alone is not enough; the Service must also have healthy endpoints.

---

# 31. Full Application Flow

```text
                         External User
                              │
                              ▼
                     NodePort :30009
                              │
                              ▼
                     frontend Service
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
         Frontend-1      Frontend-2      Frontend-3
              │               │               │
              └───────────────┼───────────────┘
                              │
                        Kubernetes DNS
                              │
                ┌─────────────┴─────────────┐
                ▼                           ▼
         redis-master                 redis-follower
           Service                       Service
              │                           │
              ▼                    ┌───────┴───────┐
         Redis Master              ▼               ▼
                                Slave 1         Slave 2
```

---

# 32. Key Takeaways

```text
Deployment
→ manages replicated Pods

Replica
→ provides scalability and availability

Service
→ gives Pods a stable network endpoint

ClusterIP
→ provides internal cluster access

NodePort
→ exposes an application externally through node ports

Labels
→ identify Pods

Selectors
→ connect Services and Deployments to Pods

Endpoints
→ actual Pod destinations behind a Service

DNS
→ enables stable service discovery

Resource Requests
→ guide scheduling decisions

kubectl logs
→ helps diagnose application failures

kubectl describe
→ shows configuration and events

kubectl exec
→ validates runtime behavior
```

---

## Final Result

Day 67 successfully demonstrated how multiple Kubernetes workloads can work together as a complete application.

The final environment contained:

```text
1 Redis Master Pod
2 Redis Slave Pods
3 Frontend Pods
4 Services
6 total application Pods
```

All application Pods reached the `Running` state, service discovery worked through Kubernetes DNS, Redis backend services were available internally, and the frontend application was exposed externally through NodePort `30009`.

**Status: ✅ Day 67 Completed Successfully**
