# Day 56 — Deploy a Highly Available Nginx Application on Kubernetes

## Overview

In this challenge, a static website needed to be deployed on a Kubernetes cluster in a highly available and scalable manner. The solution was implemented using a Kubernetes **Deployment** with multiple replicas and a **NodePort Service** to expose the application outside the cluster.

The Deployment runs three Nginx Pods using the `nginx:latest` image, while the NodePort Service forwards external traffic from port `30011` to port `80` of the Nginx containers.

---

## Task Requirements

The challenge required the following Kubernetes resources:

| Requirement | Configuration |
|---|---|
| Deployment name | `nginx-deployment` |
| Container name | `nginx-container` |
| Container image | `nginx:latest` |
| Replica count | `3` |
| Service name | `nginx-service` |
| Service type | `NodePort` |
| Application port | `80` |
| NodePort | `30011` |

---

## Architecture

```text
                         Kubernetes Cluster
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                    nginx-service                            │
│                  Type: NodePort                             │
│                  NodePort: 30011                            │
│                         │                                   │
│                         │ Selector: app=nginx               │
│                         ▼                                   │
│             nginx-deployment (3 replicas)                   │
│                                                             │
│        ┌──────────────┬──────────────┬──────────────┐        │
│        │              │              │              │        │
│        ▼              ▼              ▼              │        │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐         │        │
│   │ Nginx   │    │ Nginx   │    │ Nginx   │         │        │
│   │ Pod 1   │    │ Pod 2   │    │ Pod 3   │         │        │
│   │ Port 80 │    │ Port 80 │    │ Port 80 │         │        │
│   └─────────┘    └─────────┘    └─────────┘         │        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                              ▲
                              │
                     <Node-IP>:30011
```

---

## Implementation

A single Kubernetes manifest was created containing both the Deployment and Service definitions.

### Kubernetes Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
          ports:
            - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service

spec:
  type: NodePort

  selector:
    app: nginx

  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30011
```

The manifest was applied using:

```bash
kubectl apply -f day56-nginx.yaml
```

---

## Verification

### Verify Deployment

```bash
kubectl get deployment nginx-deployment
```

The Deployment should show:

```text
READY   3/3
```

This confirms that all three replicas are available.

### Verify Pods

```bash
kubectl get pods -l app=nginx
```

Three Nginx Pods should be in the `Running` state.

### Verify Service

```bash
kubectl get svc nginx-service
```

The service should show:

```text
TYPE       PORT(S)
NodePort   80:30011/TCP
```

### Verify Service Endpoints

```bash
kubectl get endpoints nginx-service
```

The output should contain the IP addresses of the Nginx Pods, confirming that the Service selector successfully discovered the backend Pods.

---

## Key Concepts Practiced

- Kubernetes Deployments
- Replica management
- Pod templates
- Kubernetes labels and selectors
- Nginx container deployment
- Kubernetes Services
- NodePort networking
- Service-to-Pod routing
- Declarative Kubernetes configuration
- Deployment rollout verification

---

## Why a Deployment Was Used

A Deployment manages a desired number of identical Pod replicas.

For this challenge:

```yaml
replicas: 3
```

instructs Kubernetes to maintain three Nginx Pods.

If a Pod terminates unexpectedly, the Deployment controller automatically creates a replacement Pod so that the desired replica count remains three.

This provides better availability than running a single standalone Pod.

---

## Why a NodePort Service Was Used

Pods receive internal cluster IP addresses that may change when Pods are recreated. A Service provides a stable endpoint for reaching the Pods.

With:

```yaml
type: NodePort
```

Kubernetes exposes the Service on a port of every cluster node.

For this challenge:

```text
NodePort = 30011
```

Traffic sent to:

```text
<Node-IP>:30011
```

is forwarded by the Service to port `80` on one of the matching Nginx Pods.

---

## Labels and Selectors

The Pods created by the Deployment were assigned:

```yaml
labels:
  app: nginx
```

The Service uses:

```yaml
selector:
  app: nginx
```

This matching label allows the Service to discover the Nginx Pods dynamically.

```text
Service selector: app=nginx
          │
          ▼
Pods with label: app=nginx
```

The Service does not need to know individual Pod IP addresses.

---

## Result

The Day 56 challenge was completed successfully.

The final configuration provided:

- Three highly available Nginx replicas
- Automatic Pod management through a Deployment
- Stable Service discovery through labels and selectors
- External access through NodePort `30011`
- Traffic forwarding to Nginx on container port `80`

---

## Conclusion

This challenge demonstrated the standard Kubernetes pattern of deploying an application through a **Deployment** and exposing it through a **Service**.

The Deployment maintains the required application replicas, while the Service provides a stable network endpoint and distributes traffic across the matching Pods. This architecture is a fundamental building block for running highly available applications on Kubernetes.
