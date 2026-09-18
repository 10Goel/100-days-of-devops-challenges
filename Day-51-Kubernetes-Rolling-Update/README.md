# DevOps Challenge – Day 51: Kubernetes Rolling Update

## Overview

In this challenge, an application running on a Kubernetes cluster was already deployed using an Nginx container image. The development team released an updated image, `nginx:1.18`, and the task was to deploy this version using a **rolling update** without recreating the Deployment manually.

The existing Kubernetes Deployment was:

```text
nginx-deployment
```

The goal was to update its container image to:

```text
nginx:1.18
```

and verify that all Pods were healthy after the rollout.

---

## Task Requirements

- Use the existing Deployment named `nginx-deployment`.
- Update the application image to `nginx:1.18`.
- Perform the change through a Kubernetes rolling update.
- Ensure the rollout completes successfully.
- Verify that all Pods are operational after the update.
- Confirm that the Deployment is using the required image.

---

## Environment

The `kubectl` utility on the jump host was already configured to communicate with the Kubernetes cluster.

No additional cluster configuration was required.

---

## Solution Approach

### 1. Inspect the existing Deployment

First, verify that the Deployment exists and review its current state:

```bash
kubectl get deployment nginx-deployment
```

To inspect the container name, image, and update strategy:

```bash
kubectl get deployment nginx-deployment   -o jsonpath='{.spec.template.spec.containers[*].name}{"\n"}{.spec.template.spec.containers[*].image}{"\n"}{.spec.strategy.type}{"\n"}'
```

---

### 2. Get the Deployment container name

Instead of assuming the container name, retrieve it directly:

```bash
CONTAINER=$(kubectl get deployment nginx-deployment   -o jsonpath='{.spec.template.spec.containers[0].name}')
```

Verify it:

```bash
echo "$CONTAINER"
```

---

### 3. Perform the rolling update

Update the Deployment image:

```bash
kubectl set image deployment/nginx-deployment   "$CONTAINER"=nginx:1.18
```

Updating the Pod template causes Kubernetes to create a new ReplicaSet and gradually replace the old Pods with Pods running the new image.

---

### 4. Monitor the rollout

Wait for the Deployment rollout to complete:

```bash
kubectl rollout status deployment/nginx-deployment
```

A successful rollout reports:

```text
deployment "nginx-deployment" successfully rolled out
```

---

### 5. Verify Pod health

Check that the newly created Pods are running:

```bash
kubectl get pods
```

The Pods should be in the `Running` state and their containers should be ready.

---

### 6. Verify the deployed image

Confirm that the Deployment template now references the required Nginx version:

```bash
kubectl get deployment nginx-deployment   -o jsonpath='{.spec.template.spec.containers[*].image}{"\n"}'
```

Expected result:

```text
nginx:1.18
```

---

### 7. Verify Deployment readiness

Check the final Deployment status:

```bash
kubectl get deployment nginx-deployment
```

The desired replicas should match the ready, updated, and available replicas.

---

## Rolling Update Flow

```text
Existing Deployment
        |
        | Update container image
        v
Pod Template changes
        |
        v
New ReplicaSet created
        |
        v
New Pods start gradually
        |
        v
New Pods become Ready
        |
        v
Old Pods are terminated gradually
        |
        v
Rolling update completed
```

This process allows Kubernetes to deploy a new application version while minimizing service interruption.

---

## Verification Checklist

- [x] `nginx-deployment` existed.
- [x] Container image updated to `nginx:1.18`.
- [x] Rolling update completed successfully.
- [x] New Pods reached the `Running` state.
- [x] Deployment became fully available.
- [x] Final Deployment image verified as `nginx:1.18`.

---

## Key Concepts Practiced

- Kubernetes Deployments
- Rolling updates
- ReplicaSets
- Pod template changes
- `kubectl set image`
- `kubectl rollout status`
- Deployment validation
- Image version verification
- Zero/minimal-downtime application updates

---

## Useful Rollout Commands

View rollout history:

```bash
kubectl rollout history deployment/nginx-deployment
```

Pause a rollout:

```bash
kubectl rollout pause deployment/nginx-deployment
```

Resume a paused rollout:

```bash
kubectl rollout resume deployment/nginx-deployment
```

Rollback to the previous revision:

```bash
kubectl rollout undo deployment/nginx-deployment
```

---

## Result

The Kubernetes Deployment was successfully updated to use `nginx:1.18` through the Deployment rolling-update mechanism, and all application Pods were verified to be operational after the rollout.

---
