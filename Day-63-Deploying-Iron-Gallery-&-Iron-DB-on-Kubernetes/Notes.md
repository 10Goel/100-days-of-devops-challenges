# 📘 Day 63 — Kubernetes Notes

## 🗂️ 1. Kubernetes Namespace

A **Namespace** provides logical separation for Kubernetes resources inside the same cluster.

For this challenge:

```text
iron-namespace-nautilus
```

All Deployments and Services were created inside this namespace.

```yaml
metadata:
  namespace: iron-namespace-nautilus
```

Namespaces are useful for organizing resources by environment, team, project, or application.

---

## 📦 2. Kubernetes Deployment

A Deployment manages Pods and ensures the desired replica count remains available.

```yaml
spec:
  replicas: 1
```

If the managed Pod is deleted or crashes, the Deployment controller creates a replacement Pod.

---

## 🏷️ 3. Labels

Labels are key-value metadata attached to Kubernetes objects.

Gallery:

```yaml
run: iron-gallery
```

Database:

```yaml
db: mariadb
```

They are commonly used by selectors to connect Kubernetes resources.

---

## 🔗 4. Selectors

A selector identifies which Pods a Deployment manages or which Pods a Service sends traffic to.

Gallery Deployment:

```yaml
selector:
  matchLabels:
    run: iron-gallery
```

Gallery Pod template:

```yaml
labels:
  run: iron-gallery
```

The selector and Pod labels must match.

---

## 🧩 5. Deployment Label Relationship

```text
Deployment Selector
        ↓
Pod Template Label
        ↓
Created Pod Label
```

For Iron Gallery:

```text
run=iron-gallery
        ↓
run=iron-gallery
        ↓
run=iron-gallery
```

---

## ⚙️ 6. Resource Limits

The gallery container uses:

```yaml
resources:
  limits:
    memory: "100Mi"
    cpu: "50m"
```

### CPU

```text
50m = 50 millicores = 0.05 CPU core
```

### Memory

```text
100Mi = 100 mebibytes
```

Resource limits restrict the maximum CPU and memory a container can consume.

---

## 💾 7. `emptyDir` Volume

An `emptyDir` volume is created when a Pod is assigned to a node.

```yaml
volumes:
  - name: config
    emptyDir: {}
```

The data remains available while that Pod exists.

If the Pod is deleted and replaced, the `emptyDir` data is lost.

Typical use cases include:

- Temporary files
- Cache data
- Runtime-generated data
- Sharing files between containers in the same Pod

---

## 📂 8. Gallery Volume Mounts

The gallery container has two mounts:

```text
config → /usr/share/nginx/html/data
images → /usr/share/nginx/html/uploads
```

Manifest:

```yaml
volumeMounts:
  - name: config
    mountPath: /usr/share/nginx/html/data
  - name: images
    mountPath: /usr/share/nginx/html/uploads
```

Backing volumes:

```yaml
volumes:
  - name: config
    emptyDir: {}
  - name: images
    emptyDir: {}
```

---

## 🗃️ 9. Database Volume

MariaDB uses:

```yaml
volumeMounts:
  - name: db
    mountPath: /var/lib/mysql
```

with:

```yaml
volumes:
  - name: db
    emptyDir: {}
```

`/var/lib/mysql` is where MariaDB stores its database files.

For this lab, `emptyDir` was explicitly required. In production, persistent storage would normally be used for databases.

---

## 🌱 10. Environment Variables

The DB container receives configuration through environment variables.

```yaml
env:
  - name: MYSQL_DATABASE
    value: database_blog
```

Required variables:

```text
MYSQL_DATABASE
MYSQL_ROOT_PASSWORD
MYSQL_PASSWORD
MYSQL_USER
```

Environment variables allow configuration to be injected into a container at runtime.

---

## 🔐 11. Password Handling

This lab allowed passwords directly in the Deployment manifest.

In real environments, sensitive values should normally be stored using Kubernetes Secrets instead of plain text in YAML.

```text
Kubernetes Secret
        ↓
Environment Variable
        ↓
Container
```

---

## 🌐 12. Kubernetes Service

A Kubernetes Service provides a stable network endpoint for a set of Pods.

Pods can be recreated and receive new IP addresses, but the Service remains stable.

```text
Client
  ↓
Service
  ↓
Selector
  ↓
Matching Pod
```

---

## 🏠 13. ClusterIP

The database Service uses:

```yaml
type: ClusterIP
```

Configuration:

```text
iron-db-service-nautilus
Port: 3306
TargetPort: 3306
```

ClusterIP exposes the Service only inside the Kubernetes cluster.

---

## 🌍 14. NodePort

The gallery Service uses:

```yaml
type: NodePort
```

with:

```text
Port: 80
TargetPort: 80
NodePort: 32678
```

Traffic flow:

```text
User
  ↓
<Node-IP>:32678
  ↓
NodePort Service
  ↓
Service Port 80
  ↓
Gallery Pod Port 80
```

---

## 🔌 15. `port` vs `targetPort` vs `nodePort`

### `port`

The port exposed by the Kubernetes Service.

```yaml
port: 80
```

### `targetPort`

The destination port on the Pod.

```yaml
targetPort: 80
```

### `nodePort`

The port exposed on Kubernetes Nodes.

```yaml
nodePort: 32678
```

Therefore:

```text
<Node-IP>:32678
        ↓
Service:80
        ↓
Pod:80
```

---

## 🔍 16. Service Selectors

Gallery Service:

```yaml
selector:
  run: iron-gallery
```

matches Pods with:

```yaml
labels:
  run: iron-gallery
```

Database Service:

```yaml
selector:
  db: mariadb
```

matches Pods with:

```yaml
labels:
  db: mariadb
```

---

## 🎯 17. Endpoints

A Service tracks matching Pods through endpoints.

```bash
kubectl get endpoints -n iron-namespace-nautilus
```

If a Service shows:

```text
<none>
```

then a common cause is:

```text
Service selector ≠ Pod labels
```

This is an important Kubernetes troubleshooting pattern.

---

## 🧪 18. Application Verification

The Iron Gallery application was exposed through:

```text
NodePort: 32678
```

It can be tested using:

```bash
curl http://<NODE-IP>:32678
```

For this challenge, seeing the installation page was sufficient. The frontend and database did not need to be connected yet.

---

## ⚠️ 19. Common Mistakes

### ❌ Wrong Namespace

Resources must be created inside:

```text
iron-namespace-nautilus
```

### ❌ Selector/Label Mismatch

A Service with:

```yaml
selector:
  run: iron-gallery
```

will not select a Pod labelled differently.

### ❌ Wrong Image Tags

Use the exact images:

```text
kodekloud/irongallery:2.0
kodekloud/irondb:2.0
```

### ❌ Mismatched Volume Names

`volumeMounts.name` must match `volumes.name`.

```text
config → config
images → images
db     → db
```

### ❌ Wrong NodePort

The challenge explicitly required:

```text
32678
```

### ❌ Missing Resource Limits

Gallery limits must be:

```text
CPU: 50m
Memory: 100Mi
```

---

## 🧠 20. Core Takeaway

Day 63 combines several core Kubernetes building blocks:

```text
Namespace
   ↓
Deployments
   ↓
Pods
   ↓
Labels
   ↓
Service Selectors
   ↓
ClusterIP / NodePort
```

At the Pod level:

```text
Pod
├── Environment Variables
├── Resource Limits
└── Volumes
```

Understanding how these resources connect is essential for deploying and troubleshooting multi-component applications on Kubernetes.
