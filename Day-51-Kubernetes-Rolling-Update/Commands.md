# DevOps Challenge – Day 51: Commands

This file contains the commands used to complete the Kubernetes rolling update challenge.

---

## 1. Check the Existing Deployment

```bash
kubectl get deployment nginx-deployment
```

Displays the current status of the Deployment.

---

## 2. Inspect Deployment Details

```bash
kubectl get deployment nginx-deployment -o wide
```

Displays additional information about the Deployment.

---

## 3. Check Container Name, Current Image, and Strategy

```bash
kubectl get deployment nginx-deployment   -o jsonpath='{.spec.template.spec.containers[*].name}{"\n"}{.spec.template.spec.containers[*].image}{"\n"}{.spec.strategy.type}{"\n"}'
```

This helps verify:

- Container name
- Current image
- Deployment update strategy

---

## 4. Store the Container Name

```bash
CONTAINER=$(kubectl get deployment nginx-deployment   -o jsonpath='{.spec.template.spec.containers[0].name}')
```

Verify the value:

```bash
echo "$CONTAINER"
```

---

## 5. Update the Deployment Image

```bash
kubectl set image deployment/nginx-deployment   "$CONTAINER"=nginx:1.18
```

This updates the Deployment Pod template and triggers a rolling update.

---

## 6. Monitor the Rollout

```bash
kubectl rollout status deployment/nginx-deployment
```

Wait until Kubernetes reports that the Deployment has successfully rolled out.

---

## 7. Check Pods

```bash
kubectl get pods
```

Verify that the Pods are running and ready.

To continuously watch the Pods during the rollout:

```bash
kubectl get pods -w
```

Press `Ctrl+C` to stop watching.

---

## 8. Verify the Final Deployment Image

```bash
kubectl get deployment nginx-deployment   -o jsonpath='{.spec.template.spec.containers[*].image}{"\n"}'
```

Expected output:

```text
nginx:1.18
```

---

## 9. Verify Deployment Readiness

```bash
kubectl get deployment nginx-deployment
```

Check that the Deployment reports all required replicas as ready and available.

---

## 10. View Rollout History

```bash
kubectl rollout history deployment/nginx-deployment
```

Displays the revision history of the Deployment.

---

## Additional Useful Commands

### Describe the Deployment

```bash
kubectl describe deployment nginx-deployment
```

Useful for checking:

- Events
- ReplicaSets
- Update strategy
- Pod template
- Container image

---

### List ReplicaSets

```bash
kubectl get replicasets
```

During a rolling update, Kubernetes creates a new ReplicaSet for the updated Pod template.

---

### Roll Back the Deployment

```bash
kubectl rollout undo deployment/nginx-deployment
```

Reverts the Deployment to the previous revision if the new rollout fails.

---

### Check a Specific Revision

```bash
kubectl rollout history deployment/nginx-deployment --revision=<revision-number>
```

Example:

```bash
kubectl rollout history deployment/nginx-deployment --revision=2
```

---

## Minimal Command Sequence

```bash
kubectl get deployment nginx-deployment

CONTAINER=$(kubectl get deployment nginx-deployment   -o jsonpath='{.spec.template.spec.containers[0].name}')

kubectl set image deployment/nginx-deployment   "$CONTAINER"=nginx:1.18

kubectl rollout status deployment/nginx-deployment

kubectl get pods

kubectl get deployment nginx-deployment   -o jsonpath='{.spec.template.spec.containers[*].image}{"\n"}'

kubectl get deployment nginx-deployment
```
