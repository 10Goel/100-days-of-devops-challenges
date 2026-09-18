# DevOps Challenge – Day 52: Commands

This file contains the commands used to complete the Kubernetes Deployment rollback challenge.

---

## 1. Check the Deployment

```bash
kubectl get deployment nginx-deployment
```

Displays the Deployment and its current readiness status.

---

## 2. Check the Pods

```bash
kubectl get pods
```

Used to verify the current application Pod state.

---

## 3. View Rollout History

```bash
kubectl rollout history deployment/nginx-deployment
```

Displays available Deployment revisions.

---

## 4. Inspect a Specific Revision

```bash
kubectl rollout history deployment/nginx-deployment --revision=<revision-number>
```

Example:

```bash
kubectl rollout history deployment/nginx-deployment --revision=1
```

This helps inspect the Pod template associated with a particular revision.

---

## 5. Roll Back to the Previous Revision

```bash
kubectl rollout undo deployment/nginx-deployment
```

This restores the previous Deployment revision.

Expected response:

```text
deployment.apps/nginx-deployment rolled back
```

---

## 6. Monitor Rollback Status

```bash
kubectl rollout status deployment/nginx-deployment
```

Wait until Kubernetes reports:

```text
deployment "nginx-deployment" successfully rolled out
```

---

## 7. Verify the Pods

```bash
kubectl get pods
```

All application Pods should eventually be in the `Running` state.

To watch Pods continuously:

```bash
kubectl get pods -w
```

Press `Ctrl+C` when the rollout has stabilized.

---

## 8. Verify Deployment Readiness

```bash
kubectl get deployment nginx-deployment
```

Check the:

- READY
- UP-TO-DATE
- AVAILABLE

columns.

These should align with the desired replica count after the rollback.

---

## 9. Check Rollout History After Rollback

```bash
kubectl rollout history deployment/nginx-deployment
```

This confirms the revision history after restoring the previous application configuration.

---

## 10. Verify the Current Image

```bash
kubectl get deployment nginx-deployment   -o jsonpath='{.spec.template.spec.containers[*].image}{"\n"}'
```

This shows the image currently defined in the Deployment Pod template.

---

## Additional Useful Commands

### Describe the Deployment

```bash
kubectl describe deployment nginx-deployment
```

Useful for reviewing:

- ReplicaSets
- Deployment strategy
- Events
- Pod template
- Current image

---

### List ReplicaSets

```bash
kubectl get replicasets
```

This helps identify old and current ReplicaSets associated with Deployment revisions.

---

### Roll Back to a Specific Revision

```bash
kubectl rollout undo deployment/nginx-deployment   --to-revision=<revision-number>
```

Example:

```bash
kubectl rollout undo deployment/nginx-deployment   --to-revision=1
```

Use this only when a specific revision is required.

---

### Inspect a Particular Revision

```bash
kubectl rollout history deployment/nginx-deployment   --revision=<revision-number>
```

---

## Minimal Command Sequence

```bash
kubectl get deployment nginx-deployment

kubectl rollout history deployment/nginx-deployment

kubectl rollout undo deployment/nginx-deployment

kubectl rollout status deployment/nginx-deployment

kubectl get pods

kubectl get deployment nginx-deployment

kubectl rollout history deployment/nginx-deployment
```
