# Day 66 — Deploy MySQL on Kubernetes with Persistent Storage, Secrets, and NodePort

## 📌 Task Overview

The Nautilus DevOps team needed to deploy a new **MySQL server** on a Kubernetes cluster with persistent storage, secure credential handling, and external access through a NodePort service.

This challenge combined several important Kubernetes concepts:

- PersistentVolume (PV)
- PersistentVolumeClaim (PVC)
- Kubernetes Secrets
- Secret-backed environment variables
- Deployment
- Volume mounting
- NodePort Service
- MySQL application validation

---

## 🎯 Requirements

The deployment had to meet the following requirements:

- Create a PersistentVolume named `mysql-pv`
- PV capacity: `250Mi`
- Create a PersistentVolumeClaim named `mysql-pv-claim`
- PVC storage request: `250Mi`
- Create a Deployment named `mysql-deployment`
- Use a MySQL container image
- Mount persistent storage at `/var/lib/mysql`
- Create a NodePort Service named `mysql`
- Set NodePort to `30007`
- Create Secrets:
  - `mysql-root-pass`
  - `mysql-user-pass`
  - `mysql-db-url`
- Inject secret values into:
  - `MYSQL_ROOT_PASSWORD`
  - `MYSQL_DATABASE`
  - `MYSQL_USER`
  - `MYSQL_PASSWORD`

---

## 🏗️ Architecture

```text
                         Kubernetes Cluster
                                │
                                ▼
                      ┌────────────────────┐
                      │ mysql-deployment   │
                      │    Replicas: 1     │
                      └─────────┬──────────┘
                                │
                                ▼
                      ┌────────────────────┐
                      │ mysql-container    │
                      │    mysql:8.0       │
                      │   Port: 3306       │
                      └──────┬──────┬──────┘
                             │      │
                  Secrets ───┘      └── Persistent Storage
                     │                     │
                     ▼                     ▼
        ┌────────────────────────┐   mysql-pv-claim
        │ MYSQL_ROOT_PASSWORD    │          │
        │ MYSQL_DATABASE         │          ▼
        │ MYSQL_USER             │       mysql-pv
        │ MYSQL_PASSWORD         │          │
        └────────────────────────┘          ▼
                                     /var/lib/mysql

                                │
                                ▼
                      ┌────────────────────┐
                      │ Service: mysql     │
                      │ Type: NodePort     │
                      │ 3306 → 30007       │
                      └────────────────────┘
```

---

## 🧾 Kubernetes Manifest

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data/mysql
    type: DirectoryOrCreate

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 250Mi

---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-root-pass
type: Opaque
stringData:
  password: "YUIidhb667"

---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-user-pass
type: Opaque
stringData:
  username: "kodekloud_aim"
  password: "GyQkFRVNr3"

---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-db-url
type: Opaque
stringData:
  database: "kodekloud_db4"

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql-container
          image: mysql:8.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-root-pass
                  key: password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-db-url
                  key: database
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: username
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: password
          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-storage
          persistentVolumeClaim:
            claimName: mysql-pv-claim

---
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
      nodePort: 30007
```

---

## ⚙️ Implementation

### 1. Create the manifest

```bash
vi mysql.yaml
```

### 2. Apply the configuration

```bash
kubectl apply -f mysql.yaml
```

### 3. Verify the PersistentVolume

```bash
kubectl get pv
```

Expected:

```text
mysql-pv   250Mi   RWO   Retain   Bound
```

### 4. Verify the PersistentVolumeClaim

```bash
kubectl get pvc
```

Expected:

```text
mysql-pv-claim   Bound   mysql-pv   250Mi
```

### 5. Verify Secrets

```bash
kubectl get secrets
```

### 6. Verify Deployment and Pod

```bash
kubectl get deployment mysql-deployment
kubectl get pods
```

Expected Pod state:

```text
1/1 Running
```

### 7. Verify the Service

```bash
kubectl get service mysql
```

Expected mapping:

```text
3306:30007/TCP
```

### 8. Validate MySQL

```bash
kubectl exec -it $(kubectl get pods -l app=mysql -o jsonpath='{.items[0].metadata.name}') -- mysql -u root -pYUIidhb667 -e "SHOW DATABASES;"
```

The output should include:

```text
kodekloud_db4
```

---

## ✅ Validation Checklist

| Requirement | Expected Value |
|---|---|
| PersistentVolume | `mysql-pv` |
| PV Capacity | `250Mi` |
| PVC | `mysql-pv-claim` |
| PVC Request | `250Mi` |
| Deployment | `mysql-deployment` |
| Mount Path | `/var/lib/mysql` |
| Service | `mysql` |
| Service Type | `NodePort` |
| NodePort | `30007` |
| Root Secret | `mysql-root-pass` |
| User Secret | `mysql-user-pass` |
| DB Secret | `mysql-db-url` |
| Pod Status | `1/1 Running` |

---

## 🧠 Key Concepts Practiced

- PersistentVolume and PersistentVolumeClaim
- PV/PVC binding
- `ReadWriteOnce`
- `hostPath`
- Kubernetes Secrets
- `secretKeyRef`
- Environment variable injection
- Persistent volume mounts
- NodePort networking
- MySQL initialization
- Application-level validation

---

## 🏁 Result

The MySQL workload was deployed successfully with persistent storage, Kubernetes Secrets, environment-based configuration, and NodePort access.

The PV and PVC successfully reached the `Bound` state, the MySQL Pod reached `1/1 Running`, the Service exposed MySQL on NodePort `30007`, and the configured database was verified successfully.

**Status: ✅ Day 66 Completed Successfully**
