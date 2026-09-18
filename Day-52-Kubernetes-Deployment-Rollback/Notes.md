# DevOps Challenge – Day 52: Kubernetes Deployment Rollback Notes

## 1. What Is a Kubernetes Deployment Rollback?

A Kubernetes Deployment rollback restores an earlier Deployment configuration when a newer release causes problems.

Typical scenario:

```text
Working version
      |
      v
New release deployed
      |
      v
Bug discovered
      |
      v
Rollback
      |
      v
Previous working configuration restored
```

Kubernetes supports this using Deployment revisions and ReplicaSets.

---

## 2. Why Rollbacks Are Important

Application releases can fail because of:

- Application bugs
- Incorrect container images
- Invalid configuration
- Dependency problems
- Environment variable mistakes
- Failed readiness checks
- Runtime errors

Instead of manually rebuilding the old configuration, Kubernetes allows the previous Deployment revision to be restored quickly.

This reduces recovery time and makes application releases safer.

---

## 3. Deployment Revision History

A Deployment tracks revisions whenever its Pod template changes.

Examples of changes that can create a new revision include:

- Changing the container image
- Changing environment variables
- Changing container commands or arguments
- Modifying labels inside the Pod template
- Updating resource settings
- Updating probes

View Deployment revisions with:

```bash
kubectl rollout history deployment/nginx-deployment
```

Example:

```text
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

---

## 4. Deployment, ReplicaSet, and Revision Relationship

The relationship is:

```text
Deployment
    |
    +-- ReplicaSet A -> old revision
    |
    +-- ReplicaSet B -> current revision
```

Each ReplicaSet corresponds to a particular Pod-template configuration.

A Deployment uses these ReplicaSets to manage rollouts and rollbacks.

---

## 5. What `kubectl rollout undo` Does

Command:

```bash
kubectl rollout undo deployment/nginx-deployment
```

This tells Kubernetes:

```text
Restore the previous Deployment Pod-template configuration
```

Kubernetes then updates the Deployment and performs a controlled rollout back to that configuration.

It does not require manually deleting Pods.

---

## 6. Rollback Process Internally

Suppose the Deployment currently has:

```text
Revision 1 -> nginx:1.17
Revision 2 -> nginx:1.18
```

If Revision 2 is faulty:

```bash
kubectl rollout undo deployment/nginx-deployment
```

Kubernetes restores the Pod template associated with the previous version.

Conceptually:

```text
Current Deployment
      |
      v
Revision 2 configuration
      |
      | rollout undo
      v
Revision 1 configuration restored
      |
      v
ReplicaSet for old version scaled up
      |
      v
ReplicaSet for buggy version scaled down
```

---

## 7. Important: Revision Numbers Continue Forward

A common misconception is that rollback changes the current Deployment revision number back to an older number.

That is not how Kubernetes generally behaves.

Example:

```text
Revision 1 -> nginx:1.17
Revision 2 -> nginx:1.18
```

After rollback:

```text
Revision 3 -> nginx:1.17 configuration
```

So Kubernetes restores the **configuration**, not the old revision number itself.

This means:

```text
Application state can move backward
Revision history still moves forward
```

---

## 8. Why Old ReplicaSets Are Kept

Kubernetes may keep old ReplicaSets with zero replicas.

Example:

```text
NAME                         DESIRED   CURRENT   READY
nginx-deployment-abc123      0         0         0
nginx-deployment-def456      3         3         3
```

The zero-replica ReplicaSet is useful because it represents a previous Deployment revision.

It can be used for rollback.

---

## 9. Rollout History vs Rollback

### Rollout History

```bash
kubectl rollout history deployment/nginx-deployment
```

Used to view available revisions.

### Rollback

```bash
kubectl rollout undo deployment/nginx-deployment
```

Used to restore the immediately previous revision.

### Specific Revision Rollback

```bash
kubectl rollout undo deployment/nginx-deployment   --to-revision=<revision-number>
```

Used when you want a specific earlier revision rather than simply the previous one.

---

## 10. Why the Rollback Still Looks Like a Rollout

A rollback is not an instant replacement.

Kubernetes still needs to transition Pods safely.

Conceptually:

```text
Buggy Pods running
      |
      v
Old working template restored
      |
      v
Working-version Pods created
      |
      v
New Pods become Ready
      |
      v
Buggy Pods removed
```

Therefore, a rollback still uses Deployment rollout behavior.

---

## 11. `kubectl rollout status`

Command:

```bash
kubectl rollout status deployment/nginx-deployment
```

This monitors the Deployment until Kubernetes completes the transition.

Successful output:

```text
deployment "nginx-deployment" successfully rolled out
```

This confirms that Kubernetes reached the desired Deployment state.

---

## 12. Why Manual Pod Deletion Is Not a Rollback

Consider:

```bash
kubectl delete pod <pod-name>
```

The Deployment controller will simply recreate the Pod from the current Deployment template.

If the current Deployment template contains the buggy image:

```text
Delete buggy Pod
      |
      v
Deployment notices missing Pod
      |
      v
Deployment creates another buggy Pod
```

Therefore, deleting Pods does not revert the application version.

The Deployment template itself must be changed or rolled back.

---

## 13. Why Editing the Image Manually Is Different

You could manually set the old image again with:

```bash
kubectl set image ...
```

but that requires knowing exactly which image or configuration should be restored.

A rollback is safer when the correct previous revision already exists in Deployment history.

Kubernetes can restore the entire previous Pod-template configuration rather than just one field.

---

## 14. Rollback to a Specific Revision

Sometimes the immediately previous revision is not the one you want.

Example:

```text
Revision 1 -> stable
Revision 2 -> stable
Revision 3 -> buggy
Revision 4 -> still buggy
```

You may choose:

```bash
kubectl rollout undo deployment/nginx-deployment   --to-revision=2
```

This restores the configuration stored in Revision 2.

---

## 15. Inspecting a Revision Before Rollback

Use:

```bash
kubectl rollout history deployment/nginx-deployment   --revision=<revision-number>
```

This helps inspect a particular Deployment revision before restoring it.

It is useful when:

- Multiple revisions exist
- You are unsure which revision is stable
- You want to confirm the previous image
- You want to compare configuration changes

---

## 16. Deployment Status Fields

When running:

```bash
kubectl get deployment nginx-deployment
```

important fields include:

### READY

Example:

```text
3/3
```

All three desired replicas are ready.

### UP-TO-DATE

Number of replicas using the current Deployment Pod template.

### AVAILABLE

Number of replicas available to serve application traffic.

After a successful rollback, these values should match the Deployment's desired replica count.

---

## 17. Troubleshooting a Failed Rollback

If the rollback does not complete, check:

```bash
kubectl rollout status deployment/nginx-deployment
```

Then:

```bash
kubectl get pods
```

Next:

```bash
kubectl get replicasets
```

Then:

```bash
kubectl describe deployment nginx-deployment
```

For a problematic Pod:

```bash
kubectl describe pod <pod-name>
```

Check logs:

```bash
kubectl logs <pod-name>
```

---

## 18. Common Problems

### ImagePullBackOff

Possible reasons:

- Old image no longer exists
- Wrong image tag
- Registry access failure
- Authentication failure

---

### CrashLoopBackOff

The restored image is running but the application repeatedly crashes.

Check:

```bash
kubectl logs <pod-name>
```

---

### Pending Pods

Possible causes:

- Insufficient cluster resources
- Node scheduling restrictions
- Volume problems
- Node availability issues

---

### Rollout Timeout

The restored Pods may not be becoming Ready.

Inspect:

```bash
kubectl describe deployment nginx-deployment
```

and:

```bash
kubectl describe pod <pod-name>
```

---

## 19. Useful Recovery Workflow

```text
Check Deployment
      |
      v
Check rollout history
      |
      v
Initiate rollback
      |
      v
Monitor rollout status
      |
      v
Check Pods
      |
      v
Check ReplicaSets
      |
      v
Verify Deployment readiness
```

If something fails:

```text
kubectl describe deployment
      |
      v
kubectl describe pod
      |
      v
kubectl logs
```

---

## 20. Relationship Between Day 51 and Day 52

Day 51 focused on deploying a new application version:

```text
Old version
    |
    v
kubectl set image
    |
    v
Rolling update
    |
    v
New version
```

Day 52 focuses on recovery when that new release is faulty:

```text
New buggy version
      |
      v
kubectl rollout undo
      |
      v
Rollback
      |
      v
Previous working version
```

Together, these two tasks demonstrate the normal application release lifecycle:

```text
Deploy
  |
  v
Monitor
  |
  +--> Success -> Continue
  |
  +--> Failure -> Roll Back
```

---

## 21. Key Takeaways

1. Kubernetes Deployments maintain revision history for Pod-template changes.
2. `kubectl rollout history` displays available Deployment revisions.
3. `kubectl rollout undo` restores the previous Deployment configuration.
4. Rollback restores configuration, not the old revision number.
5. Revision history generally continues moving forward.
6. ReplicaSets make Deployment rollback possible.
7. Old ReplicaSets may remain with zero replicas for revision history.
8. Rollbacks still perform a controlled Pod transition.
9. Manual Pod deletion does not revert an application release.
10. A specific revision can be restored using `--to-revision`.
11. `kubectl rollout status` verifies that the rollback completed.
12. Pod and Deployment health should always be checked after a rollback.

---

## Challenge Summary

In Day 52, the `nginx-deployment` had a recently deployed release that needed to be reverted because of a reported bug. The Deployment revision history was inspected, the previous revision was restored using `kubectl rollout undo`, the rollback was monitored until completion, and the final Deployment and Pod health were verified successfully.
