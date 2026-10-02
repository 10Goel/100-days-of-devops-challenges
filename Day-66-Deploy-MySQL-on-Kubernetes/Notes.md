# Day 66 — Kubernetes MySQL, Persistent Storage, Secrets and NodePort Notes

## 1. Core Objective

This challenge deployed a stateful MySQL workload while separating three concerns:

```text
Application
Configuration / Credentials
Persistent Data
```

Kubernetes resources handled each responsibility independently.

---

## 2. PersistentVolume

A PersistentVolume represents storage made available to the cluster.

```yaml
kind: PersistentVolume
metadata:
  name: mysql-pv
```

The task required:

```text
Capacity: 250Mi
Access Mode: ReadWriteOnce
```

Mental model:

```text
PV = storage offered by the cluster
```

---

## 3. PersistentVolumeClaim

A PVC represents an application's request for storage.

```yaml
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
```

The claim requested:

```text
250Mi
```

Binding concept:

```text
PV:  "I have storage"
        +
PVC: "I need storage"
        ↓
Kubernetes matches them
        ↓
Bound
```

---

## 4. Why the Pod Uses a PVC

The Pod references:

```yaml
persistentVolumeClaim:
  claimName: mysql-pv-claim
```

not the PV directly.

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

This decouples the application from the underlying storage implementation.

---

## 5. Mounting Storage into MySQL

The claim-backed volume was mounted at:

```text
/var/lib/mysql
```

This is where MySQL stores its database files.

```text
MySQL
  ↓
/var/lib/mysql
  ↓
PVC
  ↓
PV
```

This is why storage can survive beyond the lifecycle of an individual container.

---

## 6. `ReadWriteOnce`

The PV used:

```yaml
accessModes:
  - ReadWriteOnce
```

`ReadWriteOnce` means the volume may be mounted read-write by a single node at a time.

Common modes:

| Access Mode | Meaning |
|---|---|
| `ReadWriteOnce` | Read/write from one node |
| `ReadOnlyMany` | Read-only from many nodes |
| `ReadWriteMany` | Read/write from many nodes |
| `ReadWriteOncePod` | Read/write by one Pod |

Storage backend capabilities still matter.

---

## 7. `hostPath`

The lab used a host-backed path:

```yaml
hostPath:
  path: /mnt/data/mysql
  type: DirectoryOrCreate
```

`hostPath` is useful for:

- labs
- development
- demos
- single-node environments

It is node-specific, so production environments usually prefer CSI-backed storage, cloud disks, or network storage.

---

## 8. PersistentVolume Reclaim Policy

The PV used:

```yaml
persistentVolumeReclaimPolicy: Retain
```

`Retain` means the underlying data is not automatically discarded simply because the PVC is removed.

For databases, this behavior can help protect persistent data.

---

## 9. Kubernetes Secrets

The task created three Secrets:

```text
mysql-root-pass
mysql-user-pass
mysql-db-url
```

Secrets keep credential values separate from the main Deployment definition.

---

## 10. `stringData` and `data`

A Secret may be declared using:

```yaml
stringData:
  password: example
```

Kubernetes stores the resulting Secret data in Base64 form.

Important:

```text
Base64 ≠ encryption
```

Secrets still require proper RBAC and cluster security.

---

## 11. `secretKeyRef`

Environment variables were populated from specific Secret keys.

Example:

```yaml
- name: MYSQL_ROOT_PASSWORD
  valueFrom:
    secretKeyRef:
      name: mysql-root-pass
      key: password
```

Flow:

```text
mysql-root-pass
      │
      └── password
             │
             ▼
       secretKeyRef
             │
             ▼
MYSQL_ROOT_PASSWORD
             │
             ▼
     MySQL Container
```

Both the Secret name and key must match exactly.

---

## 12. MySQL Initialization Variables

The container used:

```text
MYSQL_ROOT_PASSWORD
MYSQL_DATABASE
MYSQL_USER
MYSQL_PASSWORD
```

Purpose:

- `MYSQL_ROOT_PASSWORD` → sets root password
- `MYSQL_DATABASE` → creates the requested database
- `MYSQL_USER` → creates an application user
- `MYSQL_PASSWORD` → sets that user's password

---

## 13. Persistent Data and First Initialization

MySQL initialization variables are most important when `/var/lib/mysql` is empty.

```text
Empty data directory
       ↓
MySQL starts
       ↓
Reads environment variables
       ↓
Creates root credentials, database, and user
```

If a previously initialized data directory is reused, changing initialization environment variables does not automatically rebuild the existing database.

This is an important persistent-storage troubleshooting concept.

---

## 14. Deployment

The workload used:

```yaml
kind: Deployment
```

Control flow:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
    ↓
MySQL Container
```

The Deployment maintains the desired replica count and recreates failed Pods.

---

## 15. Labels and Selectors

The Pod used:

```yaml
labels:
  app: mysql
```

The Deployment selector and Service selector also used:

```text
app=mysql
```

Relationship:

```text
Deployment selector ──┐
                      ├──→ Pod: app=mysql
Service selector ─────┘
```

If the Service selector does not match the Pod label, the Service will have no usable backend endpoints.

---

## 16. NodePort Service

The Service was:

```yaml
type: NodePort
```

with:

```text
nodePort: 30007
```

Traffic flow:

```text
Client
  ↓
NodeIP:30007
  ↓
Service Port 3306
  ↓
TargetPort 3306
  ↓
MySQL Pod
```

---

## 17. `port`, `targetPort`, and `nodePort`

```yaml
port: 3306
targetPort: 3306
nodePort: 30007
```

Meaning:

```text
nodePort   = exposed node port
port       = Service port
targetPort = Pod/container destination port
```

Flow:

```text
30007 → 3306 → 3306
Node     Service   Pod
```

---

## 18. Service Endpoints

Useful command:

```bash
kubectl get endpoints mysql
```

If the Service has no endpoints, investigate:

- Pod labels
- Service selector
- Pod readiness
- namespace mismatch

---

## 19. Strong Validation vs Basic Validation

This:

```bash
kubectl get pods
```

showing:

```text
1/1 Running
```

proves the container is running.

It does not fully prove that MySQL is correctly initialized.

A stronger test is:

```bash
kubectl exec -it <pod> -- mysql ... -e "SHOW DATABASES;"
```

That confirms:

- MySQL is responding
- credentials work
- database initialization succeeded

---

## 20. Troubleshooting: PVC Pending

If:

```text
mysql-pv-claim   Pending
```

check:

```bash
kubectl describe pvc mysql-pv-claim
kubectl get pv
```

Possible causes:

- storage capacity mismatch
- access mode mismatch
- storage class mismatch
- no compatible PV

---

## 21. Troubleshooting: Pod Pending

If the Pod stays `Pending`:

```bash
kubectl describe pod <pod-name>
```

Possible causes:

- unbound PVC
- scheduling issue
- unavailable node resources
- volume attachment problem

---

## 22. Troubleshooting: CrashLoopBackOff

If MySQL enters:

```text
CrashLoopBackOff
```

check:

```bash
kubectl logs <pod-name>
kubectl describe pod <pod-name>
```

Typical causes include:

- missing root password
- incorrect Secret name
- incorrect Secret key
- data directory permissions
- incompatible existing database data
- initialization failure

---

## 23. Storage Validation Chain

```text
mysql-pv
  250Mi
    ↓
mysql-pv-claim
  250Mi
    ↓
mysql-storage
    ↓
/var/lib/mysql
    ↓
MySQL database files
```

---

## 24. Secret Validation Chain

```text
mysql-root-pass
  └── password
       ↓
MYSQL_ROOT_PASSWORD

mysql-db-url
  └── database
       ↓
MYSQL_DATABASE

mysql-user-pass
  ├── username → MYSQL_USER
  └── password → MYSQL_PASSWORD
```

---

## 25. Full Mental Model

```text
                ┌────────────────────────────┐
                │ Kubernetes Secrets         │
                │                            │
                │ mysql-root-pass            │
                │ mysql-user-pass            │
                │ mysql-db-url               │
                └─────────────┬──────────────┘
                              │
                              ▼
                    Environment Variables
                              │
                              ▼
                     ┌────────────────┐
                     │ MySQL Pod      │
                     │ Port 3306      │
                     └───────┬────────┘
                             │
                 ┌───────────┴───────────┐
                 │                       │
                 ▼                       ▼
          /var/lib/mysql             Service mysql
                 │                   NodePort 30007
                 ▼
         mysql-pv-claim
                 │
                 ▼
             mysql-pv
```

---

## 26. Key Takeaways

```text
PV
→ provides storage

PVC
→ requests storage

volumeMount
→ exposes storage inside a container

Secret
→ stores sensitive values

secretKeyRef
→ injects a specific Secret key

Deployment
→ manages Pods

Service
→ provides stable network access

NodePort
→ exposes the Service through a node port

kubectl describe
→ shows configuration and events

kubectl logs
→ helps diagnose application startup issues

kubectl exec
→ verifies runtime behavior
```

---

## Final Result

Day 66 successfully demonstrated the deployment of a stateful MySQL application with:

- persistent storage
- secure credential references
- environment-based MySQL initialization
- NodePort exposure
- infrastructure-level validation
- application-level validation

**Status: ✅ Day 66 Completed Successfully**
