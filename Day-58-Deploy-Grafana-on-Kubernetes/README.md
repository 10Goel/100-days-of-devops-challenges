# DevOps Day 58 — Deploy Grafana on Kubernetes

## Task Overview

The Nautilus DevOps team planned to deploy **Grafana** on a Kubernetes cluster so that application analytics could be collected and analyzed.

The objective of this challenge was to:

- Create a Kubernetes Deployment named `grafana-deployment-nautilus`.
- Use a Grafana container image.
- Expose Grafana through a Kubernetes `NodePort` Service.
- Configure the NodePort as `32000`.
- Verify that the Grafana login page is accessible.
- No additional Grafana application configuration was required.

---

## Architecture

```text
Client / Browser
       |
       | NodeIP:32000
       v
+-----------------------------+
| Kubernetes NodePort Service |
| Port: 3000                  |
| TargetPort: 3000            |
| NodePort: 32000             |
+-------------+---------------+
              |
              v
+-----------------------------+
| Grafana Pod                 |
| Container Port: 3000        |
| Image: grafana/grafana      |
+-------------+---------------+
              ^
              |
+-----------------------------+
| Deployment                  |
| grafana-deployment-nautilus |
+-----------------------------+
```

---

## Kubernetes Resources Created

### Deployment

**Name**

```text
grafana-deployment-nautilus
```

The Deployment manages the Grafana pod and ensures that the desired replica remains available.

### Service

A Kubernetes `NodePort` Service was created to expose Grafana outside the cluster.

```text
Service Type : NodePort
Service Port : 3000
Target Port  : 3000
NodePort     : 32000
```

---

## Manifest Used

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-deployment-nautilus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana-nautilus
  template:
    metadata:
      labels:
        app: grafana-nautilus
    spec:
      containers:
        - name: grafana
          image: grafana/grafana:latest
          ports:
            - containerPort: 3000

---
apiVersion: v1
kind: Service
metadata:
  name: grafana-service-nautilus
spec:
  type: NodePort
  selector:
    app: grafana-nautilus
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      nodePort: 32000
```

---

## Implementation

The manifest was saved as:

```text
grafana.yaml
```

It was applied using:

```bash
kubectl apply -f grafana.yaml
```

The deployment rollout was verified with:

```bash
kubectl rollout status deployment/grafana-deployment-nautilus
```

The pod, service, and service endpoints were then validated.

---

## Verification

### Verify Deployment

```bash
kubectl get deployment grafana-deployment-nautilus
```

The expected state is:

```text
READY   1/1
```

### Verify Pod

```bash
kubectl get pods -l app=grafana-nautilus
```

The Grafana pod should show:

```text
STATUS    Running
READY     1/1
```

### Verify Service

```bash
kubectl get svc grafana-service-nautilus
```

The important service mapping is:

```text
3000:32000/TCP
```

This confirms that Kubernetes exposes Grafana's port `3000` using NodePort `32000`.

### Verify Endpoints

```bash
kubectl get endpoints grafana-service-nautilus
```

The service should resolve to the Grafana pod IP on port `3000`.

### Verify Grafana

The Grafana login page was successfully accessed through the lab's Grafana endpoint, confirming that the Deployment and NodePort Service were functioning correctly.

---

## Result

The Day 58 challenge was completed successfully.

The final Kubernetes configuration provided:

- A running Grafana Deployment.
- A healthy Grafana pod.
- A NodePort Service exposing Grafana on port `32000`.
- Successful access to the Grafana login page.

---

## Key Concepts Practiced

- Kubernetes Deployments
- Kubernetes Services
- NodePort Services
- Pod labels and Service selectors
- Container ports
- Service ports and target ports
- Kubernetes resource verification
- Application exposure from a Kubernetes cluster
- Grafana deployment on Kubernetes
---

## Conclusion

This challenge demonstrated how an application such as Grafana can be deployed and exposed in Kubernetes. The Deployment handled the Grafana pod lifecycle, while the NodePort Service provided external access through port `32000`.
