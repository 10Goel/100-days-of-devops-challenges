# Day 49 — Kubernetes Deployment Commands

This file contains the commands used to complete and verify the Day 49 Kubernetes challenge.

---

## 1. Verify Kubernetes Cluster Access

```bash
kubectl get nodes
```

Purpose:

- Confirms that `kubectl` can communicate with the Kubernetes cluster
- Displays the available Kubernetes nodes
- Allows verification that the node status is `Ready`

---

## 2. Create the Nginx Deployment

```bash
kubectl create deployment nginx --image=nginx:latest
```

Purpose:

- Creates a Deployment named `nginx`
- Configures the Pod template to use `nginx:latest`
- Creates one replica by default

Expected response:

```text
deployment.apps/nginx created
```

---

## 3. Verify the Deployment

```bash
kubectl get deployments
```

or:

```bash
kubectl get deployment nginx
```

Expected result:

```text
NAME    READY   UP-TO-DATE   AVAILABLE
nginx   1/1     1            1
```

---

## 4. Verify the Pod

```bash
kubectl get pods
```

Expected state:

```text
READY   STATUS
1/1     Running
```

If the Pod initially shows `ContainerCreating`, wait a few seconds and run the command again.

---

## 5. Display Additional Pod Information

```bash
kubectl get pods -o wide
```

Useful for viewing:

- Pod name
- Pod IP
- Node
- Readiness
- Status
- Restart count

---

## 6. Describe the Deployment

```bash
kubectl describe deployment nginx
```

Useful for inspecting:

- Deployment labels
- Replica status
- Selector
- Pod template
- Container image
- Deployment strategy
- Events

---

## 7. Verify the Exact Container Image

```bash
kubectl get deployment nginx -o jsonpath='{.spec.template.spec.containers[0].image}'; echo
```

Expected output:

```text
nginx:latest
```

---

## 8. View the Deployment YAML

```bash
kubectl get deployment nginx -o yaml
```

This displays the live Kubernetes manifest generated for the Deployment.

---

## 9. View the ReplicaSet Created by the Deployment

```bash
kubectl get replicasets
```

Short form:

```bash
kubectl get rs
```

The ReplicaSet name will typically look similar to:

```text
nginx-xxxxxxxxxx
```

---

## 10. View All Main Workload Resources

```bash
kubectl get deployments,replicasets,pods
```

This is useful for seeing the complete relationship:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
```

---

## Final Verification Commands

```bash
kubectl get deployment nginx
kubectl get pods
kubectl get deployment nginx -o jsonpath='{.spec.template.spec.containers[0].image}'; echo
```

The challenge is complete when:

```text
Deployment = nginx
Ready      = 1/1
Pod        = Running
Image      = nginx:latest
```

---

## Optional Cleanup Command

> Do not run this before the challenge has been validated.

```bash
kubectl delete deployment nginx
```

This removes the Deployment and the ReplicaSet and Pods managed by it.
