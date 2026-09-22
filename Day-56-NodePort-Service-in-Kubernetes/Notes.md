# Day 56 — Kubernetes Deployment and NodePort Notes

## 1. Core Objective

The goal of this challenge was to deploy Nginx on Kubernetes in a way that is:

- Highly available
- Scalable
- Managed declaratively
- Accessible through the cluster nodes

The final solution used two Kubernetes objects:

```text
Deployment
   +
Service
```

The Deployment manages the application Pods, while the Service provides network access to them.

---

# 2. Kubernetes Deployment

A **Deployment** is a higher-level Kubernetes workload controller used to manage stateless application Pods.

Instead of manually creating individual Pods, a Deployment defines the desired state of the application.

Example:

```yaml
spec:
  replicas: 3
```

This tells Kubernetes:

> Keep three replicas of this application running.

Kubernetes continuously works to ensure the actual state matches this desired state.

---

## Deployment Hierarchy

```text
Deployment
    │
    ▼
ReplicaSet
    │
    ├── Pod 1
    ├── Pod 2
    └── Pod 3
```

A Deployment normally manages a ReplicaSet, and the ReplicaSet maintains the required number of Pods.

---

# 3. Why Multiple Replicas Matter

The challenge required:

```yaml
replicas: 3
```

This creates three Nginx Pods.

Advantages include:

- Higher application availability
- Better ability to distribute traffic
- Reduced dependency on one Pod
- Automatic replacement of failed Pods
- Easier horizontal scaling

For example:

```text
Desired replicas = 3
Actual replicas  = 3
```

If one Pod fails:

```text
Actual replicas = 2
```

The Deployment controller detects the mismatch and creates another Pod.

Eventually:

```text
Actual replicas = 3
```

again.

This is part of Kubernetes' self-healing behavior.

---

# 4. Pod Template

A Deployment contains a Pod template:

```yaml
template:
  metadata:
    labels:
      app: nginx

  spec:
    containers:
      - name: nginx-container
        image: nginx:latest
```

This template defines how every Pod created by the Deployment should look.

Each replica uses the same template.

---

# 5. Container Image

The required image was:

```text
nginx:latest
```

Container image syntax:

```text
repository:tag
```

Therefore:

```text
nginx:latest
│      │
│      └── image tag
└───────── image repository
```

The explicit image tag was important because the challenge specifically required `nginx:latest`.

---

# 6. Container Name

The required container name was:

```text
nginx-container
```

The container name is defined inside the Pod specification:

```yaml
containers:
  - name: nginx-container
```

This is different from:

- Deployment name
- Pod name
- Service name
- Docker image name

In this challenge:

```text
Deployment: nginx-deployment
Container:  nginx-container
Image:      nginx:latest
Service:    nginx-service
```

These are separate Kubernetes concepts.

---

# 7. Labels

Labels are key-value metadata attached to Kubernetes resources.

The Nginx Pods use:

```yaml
labels:
  app: nginx
```

Conceptually:

```text
Pod
└── Label
    └── app=nginx
```

Labels allow Kubernetes resources to identify groups of related objects.

---

# 8. Selectors

A selector tells Kubernetes which resources should be matched.

The Deployment uses:

```yaml
selector:
  matchLabels:
    app: nginx
```

The Service uses:

```yaml
selector:
  app: nginx
```

The Pod template also contains:

```yaml
labels:
  app: nginx
```

Therefore:

```text
Service selector
     │
     │ app=nginx
     ▼
Pods whose label is app=nginx
```

This is how the Service discovers the application Pods.

---

# 9. Why Service Discovery Matters

Pod IP addresses are not considered permanent.

Suppose the original Pods have:

```text
Pod 1 → 10.244.1.5
Pod 2 → 10.244.2.7
Pod 3 → 10.244.3.4
```

If Pod 2 is deleted and recreated, its new address may become:

```text
10.244.2.11
```

Clients should therefore not depend directly on Pod IPs.

A Kubernetes Service provides a stable logical endpoint and automatically tracks matching Pods.

---

# 10. Kubernetes Service

A **Service** provides a stable network abstraction for a group of Pods.

In this challenge:

```yaml
kind: Service
```

with:

```yaml
type: NodePort
```

was used.

The Service forwards incoming traffic to matching Pods.

---

# 11. NodePort Service

A NodePort Service exposes the application using a port on the Kubernetes nodes.

The important values were:

```yaml
port: 80
targetPort: 80
nodePort: 30011
```

The traffic flow is:

```text
Client
  │
  │ <Node-IP>:30011
  ▼
Kubernetes Node
  │
  ▼
nginx-service
  │
  │ targetPort 80
  ▼
Nginx Pod
  │
  ▼
Container port 80
```

---

# 12. `port`, `targetPort`, and `nodePort`

These three fields are commonly confused.

## `port`

```yaml
port: 80
```

This is the port exposed by the Service inside the cluster.

For example:

```text
nginx-service:80
```

---

## `targetPort`

```yaml
targetPort: 80
```

This is the destination port on the backend Pod/container.

Nginx listens on port `80`, so the Service forwards traffic to:

```text
Pod-IP:80
```

---

## `nodePort`

```yaml
nodePort: 30011
```

This exposes the Service through the Kubernetes nodes.

Clients may access it using:

```text
<Node-IP>:30011
```

---

## Combined Flow

```text
<Node-IP>:30011
        │
        ▼
 Service port 80
        │
        ▼
 targetPort 80
        │
        ▼
    Nginx Pod
```

---

# 13. Service Endpoints

After the Service finds matching Pods, Kubernetes creates/maintains endpoint information for those backends.

Check with:

```bash
kubectl get endpoints nginx-service
```

Conceptually:

```text
nginx-service
     │
     ├── 10.244.x.x:80
     ├── 10.244.x.x:80
     └── 10.244.x.x:80
```

These correspond to the backend Pods.

---

# 14. How Traffic Reaches the Pods

The Service does not point to only one fixed Pod.

Instead:

```text
                 nginx-service
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
        Pod 1        Pod 2        Pod 3
```

Because the Service dynamically tracks Pods through labels, replacement Pods can automatically become eligible backends as long as they have the expected label.

---

# 15. Deployment Self-Healing

Suppose there are initially three healthy Pods:

```text
Pod 1 → Running
Pod 2 → Running
Pod 3 → Running
```

If Pod 2 crashes or disappears:

```text
Pod 1 → Running
Pod 2 → Failed
Pod 3 → Running
```

The ReplicaSet managed by the Deployment creates a replacement:

```text
Pod 4 → Running
```

The desired state returns to:

```text
3 replicas
```

This is why a Deployment is much more resilient than manually running a single Pod.

---

# 16. Scaling

Because the application is managed by a Deployment, scaling can be performed easily.

Example:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

The controller would then work toward:

```text
Desired replicas = 5
```

and create additional Pods.

For the challenge, however, the required value remained:

```text
3
```

---

# 17. Declarative Kubernetes Management

The challenge was completed using a YAML manifest.

Declarative management means defining the desired state in a file:

```yaml
replicas: 3
image: nginx:latest
```

and applying it with:

```bash
kubectl apply -f day56-nginx.yaml
```

Advantages include:

- Repeatability
- Version control
- Easy auditing
- Clear configuration history
- Easier collaboration
- Simple recreation of resources

This is generally preferable to manually creating every component through several imperative commands.

---

# 18. Important Verification Commands

Check Deployment:

```bash
kubectl get deployment nginx-deployment
```

Check Pods:

```bash
kubectl get pods -l app=nginx
```

Check Service:

```bash
kubectl get svc nginx-service
```

Check endpoints:

```bash
kubectl get endpoints nginx-service
```

Check Deployment details:

```bash
kubectl describe deployment nginx-deployment
```

Check Service details:

```bash
kubectl describe svc nginx-service
```

---

# 19. Troubleshooting Logic

If the Service exists but the application is unreachable, troubleshoot layer by layer.

## Step 1 — Are the Pods running?

```bash
kubectl get pods
```

Expected:

```text
STATUS
Running
```

---

## Step 2 — Is the Deployment healthy?

```bash
kubectl get deployment nginx-deployment
```

Expected:

```text
READY
3/3
```

---

## Step 3 — Does the Service have the correct selector?

```bash
kubectl describe svc nginx-service
```

Verify:

```text
Selector: app=nginx
```

---

## Step 4 — Do the Pods have the matching label?

```bash
kubectl get pods --show-labels
```

Verify:

```text
app=nginx
```

If the Service selector and Pod labels do not match, the Service will not find the Pods.

---

## Step 5 — Are endpoints present?

```bash
kubectl get endpoints nginx-service
```

If the output shows no endpoints, investigate the selector/label relationship and Pod readiness.

---

## Step 6 — Is NodePort correct?

```bash
kubectl get svc nginx-service
```

Verify:

```text
80:30011/TCP
```

---

# 20. Deployment vs Pod

A standalone Pod:

```text
Pod
```

does not itself provide Deployment-level replica management or rollout control.

A Deployment adds a controller hierarchy:

```text
Deployment
    ▼
ReplicaSet
    ▼
Pods
```

For production-style stateless applications, Deployments are commonly used because they provide controlled replica management and updates.

---

# 21. Deployment vs Service

These objects solve different problems.

| Deployment | Service |
|---|---|
| Manages Pods | Provides networking |
| Maintains replicas | Provides a stable endpoint |
| Supports rollouts | Finds Pods through selectors |
| Replaces failed Pods | Routes traffic to matching Pods |
| Controls application lifecycle | Controls application access |

A typical application often uses both.

---

# 22. Final Architecture Summary

```text
User / Client
      │
      │ TCP 30011
      ▼
Kubernetes Node
      │
      ▼
nginx-service
Type: NodePort
      │
      │ Selector: app=nginx
      ▼
nginx-deployment
      │
      ├── nginx-container (Pod 1)
      ├── nginx-container (Pod 2)
      └── nginx-container (Pod 3)
              │
              ▼
           Port 80
```

---

# 23. Key Takeaways

1. A **Deployment** manages the lifecycle and desired replica count of application Pods.
2. A **ReplicaSet** is normally managed by the Deployment and maintains the required number of Pods.
3. **Labels and selectors** connect Services with the correct Pods.
4. A **Service** provides stable access even when backend Pod IPs change.
5. A **NodePort** exposes a Service through a port on Kubernetes nodes.
6. `port` is the Service port.
7. `targetPort` is the backend Pod/container port.
8. `nodePort` is the externally reachable port on the nodes.
9. Multiple replicas improve application availability.
10. Kubernetes continuously reconciles the actual state with the desired state defined in the manifest.
