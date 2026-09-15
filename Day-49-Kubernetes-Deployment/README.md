# DevOps Challenge — Day 49: Kubernetes Deployment

## Overview

In Day 49 of the DevOps Challenge, the objective was to create a Kubernetes **Deployment** for an Nginx application using the `nginx:latest` container image.

The Kubernetes cluster was already configured, and `kubectl` access was available from the **jump host**.

---

## Task Requirements

The task required the following:

- Create a Kubernetes Deployment named `nginx`
- Use the container image `nginx:latest`
- Explicitly specify the `latest` image tag
- Verify that the Deployment is created successfully
- Confirm that the Pod managed by the Deployment reaches the `Running` state

---

## Environment

| Component | Details |
|---|---|
| Platform | Kubernetes |
| CLI Tool | `kubectl` |
| Execution Host | Jump Host |
| Deployment Name | `nginx` |
| Container Image | `nginx:latest` |
| Default Replicas | 1 |

---

## Solution

The Deployment was created using the following command:

```bash
kubectl create deployment nginx --image=nginx:latest
```

This command created a Kubernetes Deployment named `nginx` with one replica running the `nginx:latest` image.

---

## Kubernetes Object Relationship

The Deployment automatically created and managed the required lower-level Kubernetes objects:

```text
Deployment: nginx
        |
        v
ReplicaSet
        |
        v
Pod
        |
        v
Container: nginx:latest
```

### Deployment

A Deployment defines the desired state of an application and manages application updates and scaling.

### ReplicaSet

The ReplicaSet created by the Deployment ensures that the requested number of Pods remains available.

### Pod

The Pod is the smallest deployable Kubernetes unit and runs the Nginx container.

---

## Verification

### Check the Deployment

```bash
kubectl get deployments
```

Expected state:

```text
NAME    READY   UP-TO-DATE   AVAILABLE
nginx   1/1     1            1
```

### Check the Pod

```bash
kubectl get pods
```

The Pod should eventually show:

```text
READY   STATUS
1/1     Running
```

### Verify the Configured Image

```bash
kubectl get deployment nginx -o jsonpath='{.spec.template.spec.containers[0].image}'; echo
```

Expected output:

```text
nginx:latest
```

### Inspect the Deployment

```bash
kubectl describe deployment nginx
```

This provides detailed information about the Deployment, including:

- Replica status
- Pod template
- Container image
- Labels and selectors
- Deployment strategy
- Events

---

## Result

The Day 49 challenge was completed successfully.

The final state satisfied all task requirements:

- Deployment name: `nginx`
- Image: `nginx:latest`
- Deployment available: `1/1`
- Pod status: `Running`

---

## Key Learnings

This challenge demonstrated several Kubernetes fundamentals:

- Creating a Deployment using an imperative `kubectl` command
- Understanding the relationship between Deployments, ReplicaSets, and Pods
- Verifying application state with `kubectl get`
- Inspecting Kubernetes resources with `kubectl describe`
- Reading resource configuration using JSONPath
- Understanding how Kubernetes maintains the desired number of application replicas

---

## Useful Commands

```bash
kubectl get nodes
kubectl create deployment nginx --image=nginx:latest
kubectl get deployments
kubectl get pods
kubectl describe deployment nginx
kubectl get deployment nginx -o jsonpath='{.spec.template.spec.containers[0].image}'; echo
```

---
## Conclusion

Day 49 introduced the practical use of Kubernetes Deployments. Instead of creating an individual Pod directly, a Deployment was used to define and manage the desired state of the Nginx application.

The Deployment controller created a ReplicaSet, which in turn maintained the required Pod running the `nginx:latest` container image. This is the standard approach for running and managing stateless applications in Kubernetes.
