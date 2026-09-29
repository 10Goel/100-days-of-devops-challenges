# 🚀 Day 63 — Deploying Iron Gallery & Iron DB on Kubernetes

## 📌 Challenge Overview

The Nautilus DevOps team needed to deploy the **Iron Gallery frontend** and its **MariaDB backend** inside a dedicated Kubernetes namespace.

This challenge combines several important Kubernetes concepts:

- ✅ Namespace creation
- ✅ Deployments
- ✅ Labels and selectors
- ✅ Resource limits
- ✅ Environment variables
- ✅ `emptyDir` volumes
- ✅ ClusterIP Service
- ✅ NodePort Service
- ✅ Application and endpoint verification

---

## 🏗️ Architecture

```text
iron-namespace-nautilus
│
├── Deployment: iron-gallery-deployment-nautilus
│   └── Pod
│       └── Container: iron-gallery-container-nautilus
│           ├── Image: kodekloud/irongallery:2.0
│           ├── /usr/share/nginx/html/data
│           │   └── emptyDir volume: config
│           └── /usr/share/nginx/html/uploads
│               └── emptyDir volume: images
│
├── Service: iron-gallery-service-nautilus
│   └── Type: NodePort
│       ├── Port: 80
│       ├── TargetPort: 80
│       └── NodePort: 32678
│
├── Deployment: iron-db-deployment-nautilus
│   └── Pod
│       └── Container: iron-db-container-nautilus
│           ├── Image: kodekloud/irondb:2.0
│           ├── Environment variables
│           └── /var/lib/mysql
│               └── emptyDir volume: db
│
└── Service: iron-db-service-nautilus
    └── Type: ClusterIP
        ├── Port: 3306
        └── TargetPort: 3306
```

---

## 🎯 Task Requirements

### 1️⃣ Namespace

```text
iron-namespace-nautilus
```

### 2️⃣ Iron Gallery Deployment

| Requirement | Value |
|---|---|
| Deployment | `iron-gallery-deployment-nautilus` |
| Label | `run: iron-gallery` |
| Replicas | `1` |
| Container | `iron-gallery-container-nautilus` |
| Image | `kodekloud/irongallery:2.0` |
| CPU Limit | `50m` |
| Memory Limit | `100Mi` |
| Volume 1 | `config` |
| Mount 1 | `/usr/share/nginx/html/data` |
| Volume 2 | `images` |
| Mount 2 | `/usr/share/nginx/html/uploads` |

Both volumes use:

```yaml
emptyDir: {}
```

### 3️⃣ Iron DB Deployment

| Requirement | Value |
|---|---|
| Deployment | `iron-db-deployment-nautilus` |
| Label | `db: mariadb` |
| Replicas | `1` |
| Container | `iron-db-container-nautilus` |
| Image | `kodekloud/irondb:2.0` |
| Volume | `db` |
| Mount | `/var/lib/mysql` |
| Volume Type | `emptyDir` |

Required environment variables:

```text
MYSQL_DATABASE
MYSQL_ROOT_PASSWORD
MYSQL_PASSWORD
MYSQL_USER
```

### 4️⃣ Database Service

```text
Name: iron-db-service-nautilus
Type: ClusterIP
Selector: db=mariadb
Port: 3306
TargetPort: 3306
Protocol: TCP
```

### 5️⃣ Gallery Service

```text
Name: iron-gallery-service-nautilus
Type: NodePort
Selector: run=iron-gallery
Port: 80
TargetPort: 80
NodePort: 32678
Protocol: TCP
```

---

## 🧾 Kubernetes Manifest

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: iron-namespace-nautilus

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery-deployment-nautilus
  namespace: iron-namespace-nautilus
  labels:
    run: iron-gallery
spec:
  replicas: 1
  selector:
    matchLabels:
      run: iron-gallery
  template:
    metadata:
      labels:
        run: iron-gallery
    spec:
      containers:
        - name: iron-gallery-container-nautilus
          image: kodekloud/irongallery:2.0
          resources:
            limits:
              memory: "100Mi"
              cpu: "50m"
          volumeMounts:
            - name: config
              mountPath: /usr/share/nginx/html/data
            - name: images
              mountPath: /usr/share/nginx/html/uploads
      volumes:
        - name: config
          emptyDir: {}
        - name: images
          emptyDir: {}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db-deployment-nautilus
  namespace: iron-namespace-nautilus
  labels:
    db: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      db: mariadb
  template:
    metadata:
      labels:
        db: mariadb
    spec:
      containers:
        - name: iron-db-container-nautilus
          image: kodekloud/irondb:2.0
          env:
            - name: MYSQL_DATABASE
              value: database_blog
            - name: MYSQL_ROOT_PASSWORD
              value: "NautilusRoot@2026#DB"
            - name: MYSQL_PASSWORD
              value: "BlogUser@2026#Pass"
            - name: MYSQL_USER
              value: bloguser
          volumeMounts:
            - name: db
              mountPath: /var/lib/mysql
      volumes:
        - name: db
          emptyDir: {}

---
apiVersion: v1
kind: Service
metadata:
  name: iron-db-service-nautilus
  namespace: iron-namespace-nautilus
spec:
  selector:
    db: mariadb
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: ClusterIP

---
apiVersion: v1
kind: Service
metadata:
  name: iron-gallery-service-nautilus
  namespace: iron-namespace-nautilus
spec:
  selector:
    run: iron-gallery
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 32678
  type: NodePort
```

---

## ✅ Verification

```bash
kubectl get all -n iron-namespace-nautilus
```

```bash
kubectl get deployments -n iron-namespace-nautilus
```

```bash
kubectl get pods -n iron-namespace-nautilus
```

```bash
kubectl get svc -n iron-namespace-nautilus
```

```bash
kubectl get endpoints -n iron-namespace-nautilus
```

The gallery application is exposed through:

```text
<Node-IP>:32678
```

---

## 🧠 Key Concepts Practiced

- 🗂️ Kubernetes Namespaces
- 📦 Deployments
- 🏷️ Labels and Selectors
- ⚙️ Resource Limits
- 🌱 Environment Variables
- 💾 `emptyDir` Volumes
- 🔗 ClusterIP Services
- 🌐 NodePort Services
- 🧩 Service-to-Pod Mapping
- 🔍 Kubernetes Resource Verification

---

## 🏁 Result

✅ **Day 63 completed successfully.**

The Iron Gallery frontend and Iron DB backend were deployed inside the dedicated namespace with the required labels, selectors, images, environment variables, volume mounts, resource limits, and Kubernetes Services.
