# DevOps Challenge – Day 52: Kubernetes Deployment Rollback

## Overview

In this challenge, a new application release had already been deployed to a Kubernetes cluster, but a bug was reported in that release. The objective was to roll back the existing Deployment to its **previous working revision**.

The Deployment involved in this task was:

```text
nginx-deployment
```

The rollback was performed using Kubernetes' built-in Deployment revision history and rollout functionality.

---

## Task Requirements

- Identify the existing Deployment named `nginx-deployment`.
- Check its rollout history.
- Roll back the Deployment to the previous revision.
- Ensure the rollback completes successfully.
- Verify that all Pods are operational after the rollback.
- Confirm that the Deployment is healthy.

---

## Environment

The `kubectl` utility on the jump host was already configured to communicate with the Kubernetes cluster.

No additional Kubernetes configuration was required.

---

## Solution Approach

### 1. Verify the Existing Deployment

```bash
kubectl get deployment nginx-deployment
```

This confirms that the Deployment exists and displays its current readiness state.

---

### 2. Check Deployment Rollout History

```bash
kubectl rollout history deployment/nginx-deployment
```

This displays the Deployment revisions available for rollback.

Example:

```text
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

The current revision represents the recently deployed release, while the previous revision represents the earlier application configuration.

---

### 3. Roll Back to the Previous Revision

```bash
kubectl rollout undo deployment/nginx-deployment
```

This instructs Kubernetes to restore the previous Deployment revision.

Expected output:

```text
deployment.apps/nginx-deployment rolled back
```

---

### 4. Monitor the Rollback

```bash
kubectl rollout status deployment/nginx-deployment
```

A successful rollback reports:

```text
deployment "nginx-deployment" successfully rolled out
```

---

### 5. Verify Pod Health

```bash
kubectl get pods
```

All Pods belonging to the application should reach the `Running` state and show their containers as ready.

---

### 6. Verify Deployment Health

```bash
kubectl get deployment nginx-deployment
```

The number of Ready, Up-to-Date, and Available replicas should match the desired replica count once the rollback is complete.

---

### 7. Verify Rollout History Again

```bash
kubectl rollout history deployment/nginx-deployment
```

This helps confirm that Kubernetes recorded the rollback as part of the Deployment's revision history.

---

## Rollback Flow

```text
Current buggy release
        |
        | kubectl rollout undo
        v
Previous Pod template restored
        |
        v
Previous ReplicaSet scaled up
        |
        v
Buggy ReplicaSet scaled down
        |
        v
Previous working application version restored
```

---

## Important Kubernetes Behavior

A rollback does not simply move the revision counter backward.

For example:

```text
Revision 1 -> Working release
Revision 2 -> Buggy release
```

After a rollback, Kubernetes can create a new revision containing the configuration from Revision 1:

```text
Revision 2 -> Buggy release
Revision 3 -> Configuration restored from Revision 1
```

The Deployment history continues moving forward even though an older application configuration is restored.

---

## Verification Checklist

- [x] `nginx-deployment` verified.
- [x] Deployment rollout history inspected.
- [x] Rollback to previous revision initiated.
- [x] Rollback completed successfully.
- [x] Pods reached the `Running` state.
- [x] Deployment became fully available.
- [x] Rollout history verified after rollback.

---

## Key Concepts Practiced

- Kubernetes Deployments
- Deployment revisions
- Rollout history
- Deployment rollback
- ReplicaSets
- `kubectl rollout undo`
- `kubectl rollout status`
- Pod health verification
- Application recovery after a faulty release

---

## Useful Rollback Commands

View rollout history:

```bash
kubectl rollout history deployment/nginx-deployment
```

Roll back to the immediately previous revision:

```bash
kubectl rollout undo deployment/nginx-deployment
```

Roll back to a specific revision:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=<revision-number>
```

Inspect a specific revision:

```bash
kubectl rollout history deployment/nginx-deployment --revision=<revision-number>
```

---

## Result

The `nginx-deployment` was successfully rolled back to its previous application revision using Kubernetes Deployment rollout history. The rollback completed successfully, and all application Pods were verified to be operational.

---
