# DevOps Day 58 — Notes

## Grafana

Grafana is an observability and visualization platform commonly used to build dashboards from metrics, logs, and other data sources.

In this task, Grafana itself did not require additional configuration. The objective was only to deploy it successfully and expose its login page.

---

# 1. Kubernetes Deployment

A Kubernetes Deployment is a higher-level controller that manages pods through ReplicaSets.

Instead of creating an individual Pod directly, the task used:

```yaml
kind: Deployment
```

with:

```yaml
metadata:
  name: grafana-deployment-nautilus
```

The Deployment ensures that the requested number of Grafana replicas remains available.

For this task:

```yaml
replicas: 1
```

means Kubernetes should maintain one Grafana pod.

---

# 2. Deployment Selector and Pod Labels

The Deployment selector was:

```yaml
selector:
  matchLabels:
    app: grafana-nautilus
```

The Pod template contained the same label:

```yaml
metadata:
  labels:
    app: grafana-nautilus
```

These values must match.

The Deployment uses this selector to identify the pods that it manages.

---

# 3. Grafana Container Port

Grafana normally listens on TCP port:

```text
3000
```

The container configuration therefore included:

```yaml
ports:
  - containerPort: 3000
```

`containerPort` documents the port exposed by the application inside the container.

It does not, by itself, make the application externally reachable.

---

# 4. Kubernetes Service

Pods are ephemeral and their IP addresses can change.

A Kubernetes Service provides a stable network endpoint for accessing a group of pods.

The Service locates the Grafana pod using this selector:

```yaml
selector:
  app: grafana-nautilus
```

This value must match the Grafana pod label.

---

# 5. NodePort Service

The challenge specifically required a Service of type:

```yaml
type: NodePort
```

A NodePort Service exposes an application through a port on Kubernetes nodes.

The service configuration was:

```yaml
ports:
  - port: 3000
    targetPort: 3000
    nodePort: 32000
```

The traffic flow is:

```text
NodeIP:32000
      |
      v
Kubernetes Service
port 3000
      |
      v
targetPort 3000
      |
      v
Grafana Pod
port 3000
```

---

# 6. Difference Between port, targetPort, and nodePort

## `port`

```yaml
port: 3000
```

This is the port exposed by the Kubernetes Service inside the cluster.

Clients inside the cluster can access the service through this port.

---

## `targetPort`

```yaml
targetPort: 3000
```

This tells the Service which port on the selected pod should receive the traffic.

Since Grafana listens on port `3000`, the target port is also `3000`.

---

## `nodePort`

```yaml
nodePort: 32000
```

This exposes the Service through port `32000` on Kubernetes nodes.

External traffic can therefore reach the service using:

```text
<Node-IP>:32000
```

---

# 7. Why Service Selectors Matter

The Service contained:

```yaml
selector:
  app: grafana-nautilus
```

The Grafana pod had:

```yaml
labels:
  app: grafana-nautilus
```

Kubernetes uses this label match to associate the Service with the Grafana pod.

If these values do not match, the Service may exist but will have no usable endpoints.

A common symptom is:

```text
Endpoints: <none>
```

This means the Service selector is not matching any ready pod.

---

# 8. Kubernetes Endpoints

The command:

```bash
kubectl get endpoints grafana-service-nautilus
```

shows the actual pod IP and port behind the Service.

Example:

```text
10.244.x.x:3000
```

This confirms that the Service successfully discovered the Grafana pod.

---

# 9. Why a Deployment Was Preferred Over a Standalone Pod

A standalone Pod provides no built-in mechanism to maintain the desired application state.

A Deployment provides:

- Replica management
- Automatic pod replacement
- Rolling updates
- Rollback capabilities
- Declarative application management

Therefore, a Deployment is normally preferred for applications that should remain running.

---

# 10. Rollout Status

The command:

```bash
kubectl rollout status deployment/grafana-deployment-nautilus
```

waits for the Deployment rollout to complete.

A successful rollout confirms that the Deployment created the required pod and that the new replica became available.

---

# 11. Useful Pod States

### `Pending`

The pod has been accepted by Kubernetes but is not yet running.

Possible causes include:

- Scheduling delays
- Storage requirements
- Resource constraints

### `ContainerCreating`

The node is preparing the container.

This can include:

- Pulling the image
- Creating networking
- Mounting volumes

### `Running`

The pod has been scheduled and its container is running.

### `ImagePullBackOff`

Kubernetes could not pull the container image.

Possible causes:

- Invalid image name
- Registry authentication issue
- Network issue
- Image tag does not exist

### `CrashLoopBackOff`

The container repeatedly starts and crashes.

Logs should be inspected using:

```bash
kubectl logs <pod-name>
```

---

# 12. Useful Verification Commands

Check Deployment:

```bash
kubectl get deployment grafana-deployment-nautilus
```

Check pod:

```bash
kubectl get pods -l app=grafana-nautilus
```

Check Service:

```bash
kubectl get svc grafana-service-nautilus
```

Check endpoints:

```bash
kubectl get endpoints grafana-service-nautilus
```

Inspect Service:

```bash
kubectl describe svc grafana-service-nautilus
```

View all resources:

```bash
kubectl get all
```

---

# 13. Important Troubleshooting Logic

If the Grafana page is not reachable, troubleshoot in this order:

```text
Is the Pod running?
       |
       v
Does the Service exist?
       |
       v
Does the Service selector match the Pod label?
       |
       v
Does the Service have endpoints?
       |
       v
Is targetPort correct?
       |
       v
Is nodePort 32000 configured?
       |
       v
Is the application actually listening on port 3000?
```

This approach helps isolate whether the issue is with the container, Kubernetes networking, Service configuration, or external exposure.

---

# 14. Core Learning from Day 58

This challenge connected several important Kubernetes networking concepts:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
    ↓
Pod Labels
    ↓
Service Selector
    ↓
NodePort Service
    ↓
External Access
```

The key idea is that the Deployment manages the application lifecycle, while the Service provides stable network access to the pods.

For this task:

```text
Grafana Deployment
        ↓
Grafana Pod :3000
        ↓
NodePort Service
        ↓
Node Port 32000
        ↓
Grafana Login Page
```

This is a foundational pattern used when exposing applications running inside Kubernetes.
