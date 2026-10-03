# Day 67 — Deploy a Multi-Tier Guestbook Application on Kubernetes

## 📌 Task Overview

The Nautilus application development team completed a guestbook application and needed to deploy it on a Kubernetes cluster.

The application architecture consisted of:

- A **Redis master** backend
- Two **Redis slave/follower** replicas
- Three **frontend application** replicas
- Internal Kubernetes Services for Redis communication
- A NodePort Service to expose the frontend application
- Kubernetes DNS-based service discovery
- Resource requests for all application containers

This challenge brought together several Kubernetes concepts in a single multi-tier application deployment.

---

## 🎯 Requirements

### Back-End Tier

#### Redis Master Deployment
- Deployment name: `redis-master`
- Replicas: `1`
- Container name: `master-redis-datacenter`
- Image: `redis`
- CPU request: `100m`
- Memory request: `100Mi`
- Container port: `6379`

#### Redis Master Service
- Service name: `redis-master`
- Port: `6379`
- TargetPort: `6379`

#### Redis Slave Deployment
- Deployment name: `redis-slave`
- Replicas: `2`
- Container name: `slave-redis-datacenter`
- Image: `gcr.io/google_samples/gb-redisslave:v3`
- CPU request: `100m`
- Memory request: `100Mi`
- Environment variable:
  ```text
  GET_HOSTS_FROM=dns
  ```
- Container port: `6379`

#### Redis Slave Service
- Service name: `redis-slave`
- Port: `6379`

#### Redis Follower Service
- Service name: `redis-follower`
- Port: `6379`
- TargetPort: `6379`
- Selector:
  ```text
  app=redis-slave
  ```

### Front-End Tier

#### Frontend Deployment
- Deployment name: `frontend`
- Replicas: `3`
- Container name: `php-redis-datacenter`
- Image:
  ```text
  gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff
  ```
- CPU request: `100m`
- Memory request: `100Mi`
- Environment variable:
  ```text
  GET_HOSTS_FROM=dns
  ```
- Container port: `80`

#### Frontend Service
- Service name: `frontend`
- Type: `NodePort`
- Port: `80`
- TargetPort: `80`
- NodePort: `30009`

---

## 🏗️ Architecture

```text
                              User
                               │
                               │ NodePort 30009
                               ▼
                     ┌──────────────────┐
                     │ frontend Service │
                     │   Port 80        │
                     └────────┬─────────┘
                              │
               ┌──────────────┼──────────────┐
               ▼              ▼              ▼
         Frontend Pod 1  Frontend Pod 2  Frontend Pod 3
               │              │              │
               └──────────────┼──────────────┘
                              │
                       Kubernetes DNS
                              │
                 ┌────────────┴────────────┐
                 │                         │
                 ▼                         ▼
       redis-master Service        redis-follower Service
           Port 6379                    Port 6379
                 │                         │
                 ▼                 ┌───────┴───────┐
         Redis Master Pod          ▼               ▼
                              Redis Slave 1   Redis Slave 2
```

---

## 🧾 Kubernetes Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis-master
  template:
    metadata:
      labels:
        app: redis-master
    spec:
      containers:
        - name: master-redis-datacenter
          image: redis
          resources:
            requests:
              cpu: "100m"
              memory: "100Mi"
          ports:
            - containerPort: 6379

---
apiVersion: v1
kind: Service
metadata:
  name: redis-master
spec:
  selector:
    app: redis-master
  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis-slave
  template:
    metadata:
      labels:
        app: redis-slave
    spec:
      containers:
        - name: slave-redis-datacenter
          image: gcr.io/google_samples/gb-redisslave:v3
          resources:
            requests:
              cpu: "100m"
              memory: "100Mi"
          env:
            - name: GET_HOSTS_FROM
              value: "dns"
          ports:
            - containerPort: 6379

---
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
spec:
  selector:
    app: redis-slave
  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379

---
apiVersion: v1
kind: Service
metadata:
  name: redis-follower
spec:
  selector:
    app: redis-slave
  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: php-redis-datacenter
          image: gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff
          resources:
            requests:
              cpu: "100m"
              memory: "100Mi"
          env:
            - name: GET_HOSTS_FROM
              value: "dns"
          ports:
            - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30009
```

---

## ⚙️ Implementation

### 1. Create the Manifest

```bash
vi guestbook.yaml
```

### 2. Apply the Manifest

```bash
kubectl apply -f guestbook.yaml
```

### 3. Verify Deployments

```bash
kubectl get deployments
```

Expected:

```text
frontend       3/3
redis-master   1/1
redis-slave    2/2
```

### 4. Verify Pods

```bash
kubectl get pods
```

Expected total application Pods:

```text
6
```

All should show:

```text
1/1 Running
```

### 5. Verify Services

```bash
kubectl get services
```

Expected services:

```text
redis-master
redis-slave
redis-follower
frontend
```

The frontend service should show:

```text
80:30009/TCP
```

### 6. Verify Service Endpoints

```bash
kubectl get endpoints
```

Each application service should have one or more backend endpoints.

### 7. Verify Environment Variables

Frontend:

```bash
kubectl exec -it $(kubectl get pods -l app=frontend -o jsonpath='{.items[0].metadata.name}') -- env | grep GET_HOSTS_FROM
```

Redis slave:

```bash
kubectl exec -it $(kubectl get pods -l app=redis-slave -o jsonpath='{.items[0].metadata.name}') -- env | grep GET_HOSTS_FROM
```

Expected:

```text
GET_HOSTS_FROM=dns
```

### 8. Verify Redis Master

```bash
kubectl exec -it $(kubectl get pods -l app=redis-master -o jsonpath='{.items[0].metadata.name}') -- redis-cli ping
```

Expected:

```text
PONG
```

---

## ✅ Validation Checklist

| Requirement | Expected Value |
|---|---|
| Redis master deployment | `redis-master` |
| Redis master replicas | `1` |
| Redis master container | `master-redis-datacenter` |
| Redis master port | `6379` |
| Redis master service | `redis-master` |
| Redis slave deployment | `redis-slave` |
| Redis slave replicas | `2` |
| Redis slave container | `slave-redis-datacenter` |
| Redis slave service | `redis-slave` |
| Redis follower service | `redis-follower` |
| Frontend deployment | `frontend` |
| Frontend replicas | `3` |
| Frontend container | `php-redis-datacenter` |
| Frontend port | `80` |
| Frontend service type | `NodePort` |
| Frontend NodePort | `30009` |
| Resource requests | `100m CPU`, `100Mi memory` |
| Environment variable | `GET_HOSTS_FROM=dns` |
| Total application Pods | `6` |
| Final Pod state | All `Running` |

---

## 🧠 Key Concepts Practiced

- Multi-tier Kubernetes architecture
- Deployments and replica management
- Kubernetes Services
- ClusterIP communication
- NodePort exposure
- Service selectors
- Service endpoints
- Resource requests
- Kubernetes DNS service discovery
- Environment variables
- Redis master/slave architecture
- Pod-to-service communication
- Application-level verification

---

## 🏁 Result

The Guestbook application was deployed successfully as a multi-tier Kubernetes workload.

The Redis master ran with one replica, the Redis slave tier ran with two replicas, the frontend ran with three replicas, internal services provided stable backend connectivity, and the frontend application was exposed using NodePort `30009`.

All six application Pods reached the `Running` state successfully.

**Status: ✅ Day 67 Completed Successfully**
