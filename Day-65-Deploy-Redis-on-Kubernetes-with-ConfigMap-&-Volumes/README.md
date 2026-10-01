# Day 65 — Deploy Redis on Kubernetes with ConfigMap and Volumes

## 📌 Task Overview

The Nautilus application deployment team identified performance issues in one of its Kubernetes-hosted applications and decided to introduce **Redis** as an in-memory caching layer for testing.

The objective of this challenge was to deploy Redis on Kubernetes using a **ConfigMap**, configure persistent runtime storage with an **emptyDir volume**, mount the Redis configuration into the container, expose the default Redis port, and define the required CPU resource request.

---

## 🎯 Requirements

The Kubernetes deployment had to satisfy the following requirements:

- Create a ConfigMap named `my-redis-config`.
- Add the Redis configuration:
  ```text
  maxmemory 2mb
  ```
  under the key `redis-config`.
- Create a Deployment named `redis-deployment`.
- Use the image:
  ```text
  redis:alpine
  ```
- Name the container:
  ```text
  redis-container
  ```
- Run exactly `1` replica.
- Request `1` CPU for the Redis container.
- Expose container port `6379`.
- Mount an `emptyDir` volume named `data` at:
  ```text
  /redis-master-data
  ```
- Mount a ConfigMap-backed volume named `redis-config` at:
  ```text
  /redis-master
  ```
- Ensure Redis starts successfully and the deployment remains healthy.

---

## 🏗️ Solution Architecture

```text
                    Kubernetes Cluster
                           │
                           ▼
                 ┌───────────────────┐
                 │ redis-deployment  │
                 │    Replicas: 1    │
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │  redis-container  │
                 │   redis:alpine    │
                 │   Port: 6379      │
                 │ CPU Request: 1    │
                 └──────┬─────┬──────┘
                        │     │
            ┌───────────┘     └─────────────┐
            ▼                               ▼
   ┌─────────────────┐             ┌──────────────────┐
   │ emptyDir volume │             │ ConfigMap volume │
   │ name: data      │             │ redis-config     │
   └────────┬────────┘             └────────┬─────────┘
            │                               │
            ▼                               ▼
 /redis-master-data              /redis-master/redis-config
                                         │
                                         ▼
                               maxmemory 2mb
```

---

## 🧾 Kubernetes Manifest

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-redis-config
data:
  redis-config: |
    maxmemory 2mb
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis-container
          image: redis:alpine
          command:
            - redis-server
            - /redis-master/redis-config
          resources:
            requests:
              cpu: "1"
          ports:
            - containerPort: 6379
          volumeMounts:
            - name: data
              mountPath: /redis-master-data
            - name: redis-config
              mountPath: /redis-master
      volumes:
        - name: data
          emptyDir: {}
        - name: redis-config
          configMap:
            name: my-redis-config
```

---

## ⚙️ Implementation Steps

### 1. Create the manifest

```bash
vi redis.yaml
```

Add the ConfigMap and Deployment definitions.

### 2. Apply the resources

```bash
kubectl apply -f redis.yaml
```

### 3. Verify the ConfigMap

```bash
kubectl get configmap my-redis-config
kubectl describe configmap my-redis-config
```

### 4. Verify the Deployment and Pod

```bash
kubectl get deployments
kubectl get pods
```

Expected state:

```text
redis-deployment   1/1   1   1
```

and the Redis pod should show:

```text
READY   STATUS
1/1     Running
```

### 5. Verify the mounted configuration

```bash
kubectl exec -it $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}') -- cat /redis-master/redis-config
```

Expected:

```text
maxmemory 2mb
```

### 6. Verify Redis is using the configured memory value

```bash
kubectl exec -it $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}') -- redis-cli CONFIG GET maxmemory
```

Expected Redis value:

```text
2097152
```

This corresponds to `2 MiB` in bytes.

---

## ✅ Validation Checklist

| Check | Expected Value |
|---|---|
| ConfigMap name | `my-redis-config` |
| ConfigMap key | `redis-config` |
| Redis setting | `maxmemory 2mb` |
| Deployment | `redis-deployment` |
| Image | `redis:alpine` |
| Container | `redis-container` |
| Replicas | `1` |
| CPU request | `1` |
| EmptyDir volume | `data` |
| Data mount path | `/redis-master-data` |
| ConfigMap volume | `redis-config` |
| Config mount path | `/redis-master` |
| Container port | `6379` |
| Pod state | `1/1 Running` |

---

## 🧠 Key Concepts Practiced

- Kubernetes `ConfigMap`
- Deployment configuration
- Pod template labels and selectors
- Container resource requests
- Kubernetes `emptyDir` volumes
- ConfigMap-backed volumes
- `volumeMounts`
- Container command override
- Redis configuration through a mounted file
- Pod and deployment validation using `kubectl`

---

## 🏁 Result

The Redis deployment was successfully created and validated. The pod started in the `Running` state, Redis loaded the configuration from the mounted ConfigMap, the `emptyDir` data volume was attached correctly, the CPU request was configured, and port `6379` was exposed as required.

**Status: ✅ Day 65 Completed Successfully**
