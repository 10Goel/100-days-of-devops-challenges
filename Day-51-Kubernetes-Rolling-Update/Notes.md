# DevOps Challenge – Day 51: Kubernetes Rolling Update Notes

## 1. What Is a Kubernetes Deployment?

A Kubernetes **Deployment** is a controller used to manage stateless application Pods.

Instead of managing Pods directly, a Deployment manages them through ReplicaSets.

The relationship is:

```text
Deployment
    |
    v
ReplicaSet
    |
    v
Pods
```

The Deployment defines the desired application state, including:

- Container image
- Number of replicas
- Pod labels
- Ports
- Environment variables
- Resource configuration
- Update strategy

Kubernetes continuously attempts to make the actual cluster state match this desired state.

---

## 2. What Is a Rolling Update?

A **rolling update** replaces application Pods gradually instead of stopping all existing Pods at once.

Example:

```text
Old Pods:

Pod-A -> nginx:old
Pod-B -> nginx:old
Pod-C -> nginx:old
```

After updating the image:

```text
nginx:old -> nginx:1.18
```

Kubernetes gradually performs a transition similar to:

```text
Start new Pod
      |
      v
Wait until it becomes available
      |
      v
Terminate an old Pod
      |
      v
Repeat
```

Eventually:

```text
Pod-A -> nginx:1.18
Pod-B -> nginx:1.18
Pod-C -> nginx:1.18
```

---

## 3. Why Does Changing the Image Trigger a Rollout?

The container image is part of the Deployment's Pod template:

```yaml
spec:
  template:
    spec:
      containers:
        - image: nginx:1.18
```

When the Pod template changes, Kubernetes treats it as a new Deployment revision.

Therefore:

```text
Pod template changes
        |
        v
Deployment detects a new revision
        |
        v
New ReplicaSet is created
        |
        v
New Pods are created
        |
        v
Old ReplicaSet is scaled down
```

This is why `kubectl set image` is enough to trigger the rolling update.

---

## 4. `kubectl set image`

Syntax:

```bash
kubectl set image deployment/<deployment-name>   <container-name>=<image>
```

For this challenge:

```bash
kubectl set image deployment/nginx-deployment   "$CONTAINER"=nginx:1.18
```

This modifies the image stored in the Deployment's Pod template.

It does **not** manually edit existing Pods.

Existing Pods are replaced by the Deployment controller.

---

## 5. Deployment vs ReplicaSet vs Pod

### Deployment

Manages application releases and desired state.

```text
Deployment
```

### ReplicaSet

Ensures a specified number of matching Pods exist.

```text
Deployment
    |
    v
ReplicaSet
```

### Pod

Runs the actual application containers.

```text
Deployment
    |
    v
ReplicaSet
    |
    v
Pods
```

During a rolling update, a Deployment usually manages both:

```text
Old ReplicaSet -> old Pods
New ReplicaSet -> new Pods
```

until the transition is complete.

---

## 6. What Happens During a Rolling Update?

Suppose a Deployment runs three replicas.

Before the update:

```text
Deployment
   |
   +-- Old ReplicaSet
          |
          +-- Pod 1
          +-- Pod 2
          +-- Pod 3
```

After the image changes:

```text
Deployment
   |
   +-- Old ReplicaSet
   |      |
   |      +-- Old Pods
   |
   +-- New ReplicaSet
          |
          +-- New Pods
```

Kubernetes gradually increases the new ReplicaSet and decreases the old ReplicaSet.

At the end:

```text
Deployment
   |
   +-- Old ReplicaSet -> 0 Pods
   |
   +-- New ReplicaSet
          |
          +-- All active Pods
```

The old ReplicaSet can remain in the cluster with zero replicas so that Kubernetes can use it for rollback history.

---

## 7. RollingUpdate Strategy

A Deployment normally uses:

```yaml
strategy:
  type: RollingUpdate
```

Two important settings control the rollout:

```yaml
rollingUpdate:
  maxSurge: ...
  maxUnavailable: ...
```

### `maxSurge`

Defines how many extra Pods Kubernetes may temporarily create above the desired replica count.

Example:

```text
replicas: 4
maxSurge: 1
```

Kubernetes may temporarily run up to:

```text
5 Pods
```

during the rollout.

---

### `maxUnavailable`

Defines how many desired Pods may temporarily be unavailable during the rollout.

Example:

```text
replicas: 4
maxUnavailable: 1
```

At least three Pods should remain available while the rollout progresses, subject to readiness and other rollout conditions.

---

## 8. Why Rolling Updates Reduce Downtime

A recreate-style deployment can behave like:

```text
Stop old version
      |
      v
Start new version
```

This can create downtime.

A rolling update behaves more like:

```text
Old version running
        |
        +--> Start new version
                  |
                  v
             New Pod ready
                  |
                  v
             Stop old Pod
```

This allows traffic to continue being served while the application version is updated.

---

## 9. `kubectl rollout status`

Command:

```bash
kubectl rollout status deployment/nginx-deployment
```

This waits for Kubernetes to determine whether the rollout has completed.

Successful output looks similar to:

```text
deployment "nginx-deployment" successfully rolled out
```

If the rollout is stuck, this command helps reveal that the Deployment has not yet become fully available.

---

## 10. Rollout History

Each change to the Pod template can create a new Deployment revision.

View revisions:

```bash
kubectl rollout history deployment/nginx-deployment
```

Example conceptually:

```text
REVISION
1
2
```

The newer revision represents the updated Pod template.

---

## 11. Rollback

If a new application version causes a problem, Kubernetes can roll back to a previous Deployment revision.

```bash
kubectl rollout undo deployment/nginx-deployment
```

Conceptually:

```text
Revision 1 -> old working version
Revision 2 -> problematic version

kubectl rollout undo
        |
        v
Return to previous working revision
```

A specific revision can also be targeted:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=<revision-number>
```

---

## 12. Why Not Delete the Existing Pods Manually?

Deleting Pods is not the correct way to deploy a new application version.

For example:

```bash
kubectl delete pod <pod-name>
```

only removes a Pod.

The Deployment then recreates it using the **same Pod template**.

If the Deployment still specifies the old image, the recreated Pod will also use the old image.

Correct approach:

```text
Modify Deployment template
        |
        v
Deployment controller performs rollout
```

---

## 13. Why Not Delete and Recreate the Deployment?

Deleting and recreating the Deployment is unnecessary for a normal application version update.

Doing so can:

- Remove existing Deployment state
- Increase the chance of downtime
- Break rollout history
- Make rollback more difficult
- Introduce avoidable configuration errors

The intended Kubernetes workflow is to modify the existing Deployment.

---

## 14. Why Verify the Actual Container Name?

A Deployment may use any valid container name.

For example:

```yaml
containers:
  - name: nginx-container
```

The name does not have to be `nginx`.

Therefore, this is safer:

```bash
CONTAINER=$(kubectl get deployment nginx-deployment   -o jsonpath='{.spec.template.spec.containers[0].name}')
```

Then:

```bash
kubectl set image deployment/nginx-deployment   "$CONTAINER"=nginx:1.18
```

This avoids failures caused by assuming an incorrect container name.

---

## 15. Important Deployment Status Fields

When running:

```bash
kubectl get deployment nginx-deployment
```

you may see columns such as:

```text
READY
UP-TO-DATE
AVAILABLE
```

### READY

How many replicas are ready compared with the desired count.

Example:

```text
3/3
```

means all three desired replicas are ready.

### UP-TO-DATE

Number of replicas running the latest Pod template.

### AVAILABLE

Number of replicas considered available to serve workloads.

After a successful rollout, these values should align with the desired replica count.

---

## 16. Common Rollout Problems

### `ImagePullBackOff`

Possible causes:

- Wrong image name
- Wrong tag
- Registry authentication problem
- Registry/network issue

Check:

```bash
kubectl describe pod <pod-name>
```

---

### `CrashLoopBackOff`

The image may start successfully but the application process repeatedly crashes.

Check:

```bash
kubectl logs <pod-name>
```

and:

```bash
kubectl describe pod <pod-name>
```

---

### Pods Stuck in `Pending`

Possible causes include:

- Insufficient CPU or memory
- Scheduling restrictions
- Node issues
- Persistent volume problems

Inspect:

```bash
kubectl describe pod <pod-name>
```

---

### Rollout Does Not Finish

Check:

```bash
kubectl rollout status deployment/nginx-deployment
```

```bash
kubectl get pods
```

```bash
kubectl describe deployment nginx-deployment
```

```bash
kubectl get replicasets
```

---

## 17. Essential Troubleshooting Sequence

A useful troubleshooting flow is:

```text
kubectl get deployment
        |
        v
kubectl get pods
        |
        v
kubectl get replicasets
        |
        v
kubectl describe deployment
        |
        v
kubectl describe pod
        |
        v
kubectl logs
```

This helps determine whether the problem is at the Deployment, ReplicaSet, scheduling, container startup, or application level.

---

## 18. Key Takeaways

1. A Deployment manages application Pods through ReplicaSets.
2. Changing the Deployment Pod template creates a new rollout revision.
3. Updating the image is enough to trigger a rolling update.
4. Rolling updates gradually replace old Pods with new Pods.
5. `kubectl set image` is a convenient command for updating container images.
6. `kubectl rollout status` verifies rollout progress.
7. Old ReplicaSets can remain with zero replicas to support rollback.
8. `maxSurge` controls temporary extra Pods during the rollout.
9. `maxUnavailable` controls how many desired replicas may be unavailable.
10. Manual Pod deletion is not a replacement for updating the Deployment.
11. Rollout history makes it possible to inspect revisions and perform rollbacks.
12. Final validation should confirm both Pod health and the actual deployed image.

---

## Challenge Summary

In Day 51, the existing `nginx-deployment` was updated to use the `nginx:1.18` image. Kubernetes detected the Pod-template change, created a new ReplicaSet, gradually replaced the old Pods, and completed the deployment through its rolling-update mechanism. The rollout was then verified using Deployment status, Pod status, and the final configured container image.
