# DevOps Day 48 — Kubernetes Pod Deployment

## Overview

Day 48 marks the beginning of the Kubernetes section of this DevOps challenge.

The task was to create a Kubernetes Pod with an exact name, label, container name, and image specification using `kubectl` from the jump host.

This challenge introduces the core Kubernetes workflow:

```text
Write Desired State
        ↓
Submit to Kubernetes API
        ↓
Kubernetes Creates Resource
        ↓
Verify Actual State
```

---

## Task Requirements

Create a Pod with the following configuration:

| Property | Required Value |
|---|---|
| Pod Name | `pod-nginx` |
| Image | `nginx:latest` |
| Container Name | `nginx-container` |
| Label Key | `app` |
| Label Value | `nginx_app` |
| Management Host | `jump-host` |

---

## Kubernetes Manifest Used

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx
  labels:
    app: nginx_app
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
```

---

## Deployment Steps

### 1. Verify Cluster Access

```bash
kubectl get nodes
```

### 2. Create the Pod Manifest

```bash
vi pod-nginx.yaml
```

### 3. Validate the Manifest

```bash
kubectl apply --dry-run=client -f pod-nginx.yaml
```

### 4. Deploy the Pod

```bash
kubectl apply -f pod-nginx.yaml
```

### 5. Verify the Pod

```bash
kubectl get pods
```

Expected state:

```text
NAME        READY   STATUS    RESTARTS   AGE
pod-nginx   1/1     Running   0          ...
```

### 6. Verify the Label

```bash
kubectl get pod pod-nginx --show-labels
```

Expected:

```text
app=nginx_app
```

### 7. Verify the Container Name

```bash
kubectl get pod pod-nginx   -o jsonpath='{.spec.containers[0].name}{"\n"}'
```

Expected:

```text
nginx-container
```

### 8. Verify the Image

```bash
kubectl get pod pod-nginx   -o jsonpath='{.spec.containers[0].image}{"\n"}'
```

Expected:

```text
nginx:latest
```

---

# Kubernetes Fundamentals Introduced in Day 48

## 1. Kubernetes Object

Kubernetes manages resources as objects.

Examples include:

```text
Pod
Deployment
ReplicaSet
Service
ConfigMap
Secret
PersistentVolumeClaim
StatefulSet
DaemonSet
Job
CronJob
```

A standard Kubernetes YAML manifest commonly contains:

```yaml
apiVersion:
kind:
metadata:
spec:
```

---

## 2. Pod

A Pod is the smallest deployable unit in Kubernetes.

```text
Pod
└── One or More Containers
```

Containers inside the same Pod share the same network namespace and can communicate through `localhost`.

In this task:

```text
pod-nginx
└── nginx-container
    └── nginx:latest
```

---

## 3. Labels

Labels are key-value metadata attached to Kubernetes objects.

This challenge used:

```yaml
labels:
  app: nginx_app
```

Labels are heavily used by:

- Services
- Deployments
- ReplicaSets
- NetworkPolicies
- Monitoring systems

---

## 4. `kubectl`

`kubectl` is the Kubernetes command-line client.

Conceptually:

```text
kubectl
   ↓
Kubernetes API Server
   ↓
Cluster Resources
```

Examples:

```bash
kubectl get pods
kubectl describe pod pod-nginx
kubectl logs pod-nginx
kubectl delete pod pod-nginx
```

---

# Why Declarative Kubernetes Matters

Kubernetes supports both imperative and declarative workflows.

### Imperative

```bash
kubectl run nginx --image=nginx
```

### Declarative

```bash
kubectl apply -f pod-nginx.yaml
```

Declarative management is preferred for real DevOps environments because manifests can be:

- stored in Git,
- reviewed through pull requests,
- version controlled,
- reused across environments,
- integrated with CI/CD,
- managed by GitOps tools such as Argo CD and Flux.

---

# Kubernetes Learning Path — Fundamentals to Advanced

```text
Level 1 — Foundations
Pods
kubectl
YAML
Labels
Selectors
Namespaces

        ↓

Level 2 — Workload Management
ReplicaSets
Deployments
Rolling Updates
Rollbacks
Jobs
CronJobs

        ↓

Level 3 — Networking & Configuration
Services
DNS
ConfigMaps
Secrets
Ingress

        ↓

Level 4 — Reliability
Readiness Probes
Liveness Probes
Startup Probes
Resource Requests
Resource Limits

        ↓

Level 5 — Storage
Volumes
PersistentVolumes
PersistentVolumeClaims
StorageClasses
CSI

        ↓

Level 6 — Scheduling & Scaling
nodeSelector
Affinity
Anti-Affinity
Taints
Tolerations
HPA
VPA
Cluster Autoscaler

        ↓

Level 7 — Security & Operations
RBAC
ServiceAccounts
SecurityContexts
Pod Security
NetworkPolicies
Observability

        ↓

Level 8 — Advanced Kubernetes
Helm
Kustomize
CRDs
Operators
Admission Controllers
GitOps
Service Mesh
Multi-cluster Operations
```

---

# Why Pods Are Usually Not Created Directly in Production

A standalone Pod is useful for learning, but production applications are generally managed by controllers.

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

A Deployment continuously maintains the desired number of Pods.

If a managed Pod fails:

```text
Pod Failure
    ↓
Controller Detects Difference
    ↓
Replacement Pod Created
```

This self-healing model is one of Kubernetes' most important capabilities.

---

# Key Takeaway

The most important Kubernetes idea introduced by this challenge is **desired state**.

You define what you want:

```text
A Pod named pod-nginx
using nginx:latest
with label app=nginx_app
```

Kubernetes then works to make the actual cluster state match that desired state.

That reconciliation model is the foundation for everything from Pods and Deployments to autoscaling, Operators, and GitOps.

---

## Final Result

The task was completed successfully with:

```text
Pod Name:       pod-nginx
Container Name: nginx-container
Image:          nginx:latest
Label:          app=nginx_app
Status:         Running
```

✅ **DevOps Day 48 completed successfully.**

---
