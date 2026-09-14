# DevOps Day 48 — Kubernetes Notes

These notes begin with the concepts used directly in Day 48 and then progress into intermediate and advanced Kubernetes topics.

---

# 1. What Is Kubernetes?

Kubernetes, commonly written as **K8s**, is a container orchestration platform.

A container runtime can run containers on a single machine.

Kubernetes manages containerized workloads across an entire cluster.

```text
Container Runtime
→ Runs containers

Kubernetes
→ Schedules, connects, scales, heals, and manages containers across nodes
```

Major capabilities include:

- scheduling,
- self-healing,
- horizontal scaling,
- service discovery,
- load balancing,
- rolling updates,
- rollbacks,
- storage orchestration,
- configuration management,
- secret management.

---

# 2. Cluster Architecture

A Kubernetes cluster consists of:

```text
Control Plane
+
Worker Nodes
```

## Control Plane Components

### API Server

The API server is the primary entry point to the cluster.

```text
kubectl
   ↓
API Server
   ↓
Kubernetes Resources
```

Every object creation, update, read, or deletion is ultimately processed through the API.

### etcd

`etcd` is the distributed key-value database that stores cluster state.

It stores information about:

- Pods
- Deployments
- Services
- ConfigMaps
- Secrets
- Nodes
- Desired state

### Scheduler

The scheduler chooses which worker node should run a Pod.

It considers:

- CPU
- memory
- node labels
- affinity
- anti-affinity
- taints
- tolerations
- topology rules

### Controller Manager

Controllers compare:

```text
Desired State
vs
Actual State
```

and work to reconcile the difference.

Example:

```text
Desired replicas = 3
Actual replicas  = 2

Controller → creates another Pod
```

---

# 3. Worker Node Components

## kubelet

`kubelet` is the node agent.

It ensures that the containers specified for the node are actually running.

## Container Runtime

The runtime launches and manages containers.

A common runtime is:

```text
containerd
```

## Networking Components

Kubernetes networking is implemented through CNI plugins such as:

- Calico
- Cilium
- Flannel

---

# 4. Kubernetes Object Structure

A typical object looks like:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example
spec:
  containers:
    - name: app
      image: nginx
```

Important fields:

| Field | Meaning |
|---|---|
| `apiVersion` | API group/version |
| `kind` | Resource type |
| `metadata` | Name, labels, annotations |
| `spec` | Desired configuration |
| `status` | Current state reported by Kubernetes |

You normally define:

```text
spec
```

Kubernetes maintains:

```text
status
```

---

# 5. Pods

A Pod is the smallest deployable unit in Kubernetes.

```text
Pod
├── Container A
└── Container B
```

Containers inside one Pod share:

- the Pod IP,
- network namespace,
- `localhost`,
- attached volumes.

They are scheduled together on the same node.

---

# 6. Pod Lifecycle and Common States

Pod phases include:

```text
Pending
Running
Succeeded
Failed
Unknown
```

Common states shown during troubleshooting include:

```text
ContainerCreating
CrashLoopBackOff
ImagePullBackOff
ErrImagePull
Terminating
```

---

# 7. Labels and Selectors

Labels are key-value metadata.

Example:

```yaml
labels:
  app: nginx_app
  tier: frontend
  environment: production
```

Selectors identify matching resources.

Example:

```bash
kubectl get pods -l app=nginx_app
```

Services, ReplicaSets, Deployments, and NetworkPolicies all rely heavily on selectors.

---

# 8. Annotations

Annotations also attach metadata, but are generally not used for selecting resources.

Example:

```yaml
metadata:
  annotations:
    owner: platform-team
```

Typical uses include:

- tooling metadata,
- build information,
- controller configuration,
- documentation.

---

# 9. Namespaces

Namespaces logically organize resources inside a cluster.

Typical namespaces:

```text
default
kube-system
kube-public
kube-node-lease
```

Organizations often add:

```text
dev
qa
staging
production
```

Namespaces help with:

- organization,
- RBAC,
- quotas,
- isolation boundaries.

---

# 10. ReplicaSets

A ReplicaSet ensures that a defined number of Pod replicas exist.

```text
Desired = 3
Actual  = 2

ReplicaSet → creates 1 more Pod
```

ReplicaSets are normally managed through Deployments.

---

# 11. Deployments

Deployments are the standard controller for stateless applications.

Architecture:

```text
Deployment
   ↓
ReplicaSet
   ↓
Pods
```

Deployments provide:

- replication,
- rolling updates,
- rollbacks,
- declarative updates,
- self-healing.

---

# 12. Services

Pod IP addresses are ephemeral.

A Service provides a stable network endpoint.

```text
Client
  ↓
Service
  ↓
Matching Pods
```

Main types:

- `ClusterIP`
- `NodePort`
- `LoadBalancer`
- `ExternalName`

---

# 13. Kubernetes DNS

Kubernetes provides internal DNS.

A Service can often be reached as:

```text
service-name
```

or:

```text
service-name.namespace
```

Fully qualified:

```text
service-name.namespace.svc.cluster.local
```

This avoids hardcoding Pod IP addresses.

---

# 14. ConfigMaps

ConfigMaps store non-sensitive configuration.

Examples:

- environment names,
- URLs,
- feature flags,
- config files.

They can be injected as:

- environment variables,
- mounted files.

---

# 15. Secrets

Secrets are intended for sensitive data:

- passwords,
- tokens,
- keys,
- credentials.

Important:

```text
Base64 encoding ≠ encryption
```

Production environments should also use:

- encryption at rest,
- external secret managers,
- least-privilege RBAC.

---

# 16. Resource Requests and Limits

Example:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

## Requests

Used by the scheduler when placing Pods.

## Limits

Maximum resource consumption.

Memory limit violations can result in:

```text
OOMKilled
```

CPU usage above the limit is generally throttled.

---

# 17. Health Probes

## Liveness Probe

Answers:

```text
Is the container still healthy enough to keep running?
```

Repeated failure can trigger a restart.

## Readiness Probe

Answers:

```text
Should this Pod receive traffic?
```

If readiness fails, Services stop routing traffic to the Pod.

## Startup Probe

Useful for slow-starting applications.

It protects startup from premature liveness/readiness failures.

---

# 18. Jobs and CronJobs

## Job

Runs work to completion.

Typical examples:

- database migration,
- batch processing,
- report generation.

## CronJob

Runs Jobs on a schedule.

Examples:

- nightly backup,
- hourly cleanup,
- scheduled synchronization.

---

# 19. Storage

Pods are ephemeral, so persistent data requires storage abstractions.

## PersistentVolume

Represents storage available to the cluster.

## PersistentVolumeClaim

Represents a workload request for storage.

Flow:

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Storage Backend
```

---

# 20. StorageClasses

StorageClasses enable dynamic provisioning.

A PVC can request storage, and Kubernetes provisions it automatically through a CSI driver.

Typical backends:

- AWS EBS
- Azure Disk
- Google Persistent Disk
- Ceph
- NFS

---

# 21. StatefulSets

StatefulSets are designed for applications requiring stable identity or storage.

Common use cases:

- databases,
- Kafka,
- ZooKeeper,
- distributed systems.

They provide:

- stable Pod names,
- stable storage association,
- ordered deployment and termination.

---

# 22. DaemonSets

A DaemonSet ensures a Pod runs on every eligible node.

Common uses:

- logging agents,
- monitoring agents,
- security agents,
- networking components.

```text
Node 1 → Agent Pod
Node 2 → Agent Pod
Node 3 → Agent Pod
```

---

# 23. Ingress

Ingress manages HTTP/HTTPS traffic into the cluster.

```text
Internet
   ↓
Ingress Controller
   ↓
Ingress Rules
   ↓
Services
   ↓
Pods
```

Example routing:

```text
example.com/api → api-service
example.com/web → web-service
```

The Kubernetes Gateway API is a newer, more expressive alternative for many use cases.

---

# 24. NetworkPolicies

NetworkPolicies restrict communication between Pods and networks.

They can control:

- ingress traffic,
- egress traffic,
- source/destination Pods,
- allowed ports.

Policy enforcement depends on the CNI plugin.

---

# 25. Scheduling

The scheduler decides where Pods run.

Placement can be influenced by:

## nodeSelector

Simple node label matching.

## Node Affinity

More expressive node placement.

## Pod Affinity

Place workloads near other workloads.

## Pod Anti-Affinity

Spread workloads apart for availability.

---

# 26. Taints and Tolerations

Taints belong to nodes.

Tolerations belong to Pods.

Conceptually:

```text
Taint:
"Normal Pods should not run here."

Toleration:
"This Pod is allowed here."
```

Useful for:

- dedicated nodes,
- GPU nodes,
- infrastructure nodes,
- specialized workloads.

---

# 27. Horizontal Pod Autoscaler

HPA changes the number of Pod replicas.

```text
Load increases
   ↓
Metrics increase
   ↓
HPA increases replicas
```

It may scale on:

- CPU,
- memory,
- custom metrics,
- external metrics.

---

# 28. Vertical Pod Autoscaler

VPA adjusts or recommends Pod CPU/memory requests.

Conceptually:

```text
HPA → number of Pods
VPA → size of Pods
```

---

# 29. Cluster Autoscaler

Cluster Autoscaler changes worker-node capacity.

```text
Pods cannot be scheduled
        ↓
More node capacity needed
        ↓
New nodes added
```

---

# 30. RBAC

Role-Based Access Control restricts what identities can do.

Important resources:

```text
Role
ClusterRole
RoleBinding
ClusterRoleBinding
```

Example:

```text
Developer
  ↓
May read Pods
  ↓
May not delete cluster nodes
```

Always aim for least privilege.

---

# 31. ServiceAccounts

ServiceAccounts provide workload identities inside Kubernetes.

They can be used by applications to authenticate to:

- the Kubernetes API,
- cloud services,
- platform integrations.

Avoid granting unnecessary privileges.

---

# 32. Security Contexts

Security contexts control runtime privileges.

Common settings:

```text
runAsNonRoot
runAsUser
readOnlyRootFilesystem
allowPrivilegeEscalation
capabilities
seccompProfile
```

Production containers should avoid privileged execution unless absolutely required.

---

# 33. Pod Security

Pod Security Admission can enforce workload-security standards.

Common profiles:

```text
Privileged
Baseline
Restricted
```

`Restricted` applies stronger hardening.

---

# 34. Helm

Helm is a package manager for Kubernetes.

A Helm chart commonly contains:

```text
Chart.yaml
values.yaml
templates/
```

Useful commands:

```bash
helm install
helm upgrade
helm rollback
helm uninstall
```

Helm helps package, templatize, and version Kubernetes deployments.

---

# 35. Kustomize

Kustomize modifies Kubernetes YAML without traditional templates.

Typical structure:

```text
base/
overlays/
  dev/
  staging/
  production/
```

Apply with:

```bash
kubectl apply -k .
```

---

# 36. CRDs

CustomResourceDefinitions extend the Kubernetes API.

They allow custom resource types, for example:

```yaml
kind: Database
```

CRDs form the basis of many Kubernetes platform extensions.

---

# 37. Operators

Operators combine:

```text
Custom Resources
+
Controllers
+
Domain Knowledge
```

They can automate:

- installation,
- backup,
- upgrade,
- recovery,
- scaling.

---

# 38. GitOps

GitOps stores desired cluster state in Git.

Flow:

```text
Developer changes Git
        ↓
Pull Request
        ↓
Merge
        ↓
GitOps Controller Detects Change
        ↓
Cluster Reconciles
```

Common tools:

- Argo CD
- Flux

Benefits include:

- auditability,
- repeatability,
- fewer manual cluster changes.

---

# 39. Observability

Production Kubernetes requires strong observability.

## Metrics

Common technologies:

- Prometheus
- Grafana
- Metrics Server

## Logs

Common technologies:

- Fluent Bit
- Loki
- Elasticsearch
- OpenSearch

## Tracing

Common technologies:

- OpenTelemetry
- Jaeger
- Tempo

---

# 40. Common Kubernetes Failures

## CrashLoopBackOff

The container repeatedly starts and crashes.

Check:

```bash
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl describe pod <pod>
```

## ImagePullBackOff

Kubernetes cannot pull the image.

Possible causes:

- wrong image name,
- wrong tag,
- private registry credentials missing,
- registry outage.

## Pending

The scheduler cannot place the Pod.

Possible causes:

- insufficient CPU,
- insufficient memory,
- node selector mismatch,
- taints,
- affinity constraints,
- unavailable PVC.

## OOMKilled

The container exceeded its memory limit.

Investigate:

- memory usage,
- application leaks,
- configured resource limits.

---

# 41. Production Request Flow

A common production architecture looks like:

```text
Users
  ↓
DNS
  ↓
Load Balancer
  ↓
Ingress / Gateway
  ↓
Service
  ↓
Deployment
  ↓
ReplicaSet
  ↓
Pods
```

Supporting resources may include:

```text
ConfigMaps
Secrets
PVCs
ServiceAccounts
NetworkPolicies
HPAs
PodDisruptionBudgets
Monitoring
Logging
```

---

# 42. Reconciliation — The Most Important Kubernetes Concept

Kubernetes is a desired-state system.

You declare:

```text
"I want 3 replicas."
```

Kubernetes observes:

```text
"Only 2 exist."
```

The controller acts:

```text
"Create 1 more."
```

The system continuously tries to reach:

```text
Actual State = Desired State
```

This reconciliation loop is the central idea behind Kubernetes.

---

# 43. Day 48 Learning Progression

The Day 48 manifest:

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

introduces:

```text
Resource Definition
     ↓
Metadata
     ↓
Desired Specification
     ↓
Container Workload
```

From this single Pod, the natural learning path is:

```text
Pod
 ↓
Deployment
 ↓
Service
 ↓
Configuration
 ↓
Storage
 ↓
Networking
 ↓
Security
 ↓
Scaling
 ↓
Observability
 ↓
GitOps / Platform Engineering
```

That progression forms the foundation of practical Kubernetes engineering.
