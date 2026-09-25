# DevOps Day 59 — Kubernetes Pod Troubleshooting Notes

## Introduction

Kubernetes troubleshooting should be evidence-driven.

A good troubleshooting path for a Deployment-managed workload is:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
    ↓
Container
```

At each layer, Kubernetes provides status, events, logs, and configuration that can reveal the root cause.

---

# 1. Start with Pod Status

```bash
kubectl get pods
```

Common states include:

```text
Pending
ContainerCreating
Running
CrashLoopBackOff
ImagePullBackOff
ErrImagePull
CreateContainerConfigError
Error
```

The status gives the first clue about the failure category.

---

# 2. `kubectl get` vs `kubectl describe`

Use:

```bash
kubectl get pods
```

to answer:

```text
What is happening?
```

Use:

```bash
kubectl describe pod <pod-name>
```

to answer:

```text
Why is it happening?
```

`kubectl describe` shows:

- container state
- restart count
- node placement
- volumes
- mounts
- conditions
- events

---

# 3. Events Are Critical

The `Events` section often provides the fastest path to the root cause.

Typical event reasons include:

```text
FailedMount
FailedScheduling
FailedCreate
FailedPull
BackOff
Unhealthy
```

Example:

```text
configmap "redis-conig" not found
```

This immediately indicates that Kubernetes cannot find a referenced ConfigMap.

---

# 4. ConfigMap Troubleshooting

Check existing ConfigMaps:

```bash
kubectl get configmap
```

Inspect one:

```bash
kubectl get configmap redis-config -o yaml
```

In this challenge, the Deployment referenced:

```text
redis-conig
```

while the real ConfigMap was:

```text
redis-config
```

Resource names must match exactly.

---

# 5. ImagePullBackOff

`ImagePullBackOff` means Kubernetes tried to pull the image but failed and is backing off before retrying.

Common causes:

- incorrect image name
- invalid tag
- private registry authentication failure
- registry/network issue

Useful commands:

```bash
kubectl describe pod <pod-name>
kubectl get deployment <deployment-name> -o yaml
```

In Day 59, the image was incorrectly configured as:

```text
redis:alpin
```

instead of:

```text
redis:alpine
```

---

# 6. CrashLoopBackOff

`CrashLoopBackOff` is different from `ImagePullBackOff`.

Here, the container usually starts but the process exits repeatedly.

Common causes:

- application configuration errors
- missing environment variables
- invalid startup command
- dependency failures
- permission errors
- application crashes

Check:

```bash
kubectl logs <pod-name>
```

For the previous crashed container instance:

```bash
kubectl logs <pod-name> --previous
```

---

# 7. CreateContainerConfigError

This usually means Kubernetes cannot construct the container configuration.

Common causes:

- missing ConfigMap
- missing Secret
- invalid environment source
- invalid volume reference

Start with:

```bash
kubectl describe pod <pod-name>
```

---

# 8. Pending Pods

A `Pending` pod may indicate:

- insufficient CPU
- insufficient memory
- node selector mismatch
- taints without tolerations
- unavailable PersistentVolume
- scheduling constraints

Useful commands:

```bash
kubectl describe pod <pod-name>
kubectl get nodes
kubectl describe node <node-name>
```

---

# 9. ContainerCreating

`ContainerCreating` is not automatically an error.

Kubernetes may still be:

- pulling the image
- creating networking
- mounting volumes
- preparing Secrets
- preparing ConfigMaps

If it remains in this state too long:

```bash
kubectl describe pod <pod-name>
```

and inspect Events.

---

# 10. Running but Not Ready

A pod can be:

```text
STATUS: Running
READY: 0/1
```

This means the process is running, but Kubernetes does not consider the pod ready to receive traffic.

Possible causes:

- failing readiness probe
- application not listening yet
- dependency unavailable
- wrong readiness endpoint
- wrong port

Inspect:

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

---

# 11. Events vs Logs

Use **Events** for Kubernetes/platform problems such as:

```text
Image pull failures
Scheduling problems
Volume mount failures
Missing ConfigMaps
Missing Secrets
Probe failures
Node-related issues
```

Use **Logs** for application/runtime problems such as:

```text
Application crashes
Runtime exceptions
Configuration parsing errors
Database connection errors
Application startup failures
```

This distinction prevents wasted troubleshooting effort.

---

# 12. Inspect the Live Configuration

Always verify the configuration currently stored in Kubernetes:

```bash
kubectl get deployment redis-deployment -o yaml
```

Do not assume a local YAML file matches the live cluster state.

Focused JSONPath checks can be even faster.

Image:

```bash
kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.containers[*].image}{"\n"}'
```

ConfigMap reference:

```bash
kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.volumes[?(@.name=="config")].configMap.name}{"\n"}'
```

---

# 13. Fix the Controller, Not the Managed Pod

Deployment-managed pods are disposable.

Relationship:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
```

If the Deployment specification is wrong, manually fixing or replacing a pod does not repair the root cause.

Correct approach:

```text
Fix Deployment
      ↓
New ReplicaSet
      ↓
Corrected Pod
```

---

# 14. Verify the Rollout

After updating a Deployment:

```bash
kubectl rollout status deployment/redis-deployment
```

Successful output indicates that the new Deployment revision became available.

You can also inspect ReplicaSets:

```bash
kubectl get replicasets
```

---

# 15. Strong Kubernetes Pod Troubleshooting Workflow

Use this sequence:

```text
1. kubectl get pods
        ↓
2. Identify abnormal status
        ↓
3. kubectl describe pod
        ↓
4. Read Events
        ↓
5. kubectl logs
        ↓
6. Inspect Deployment/StatefulSet/DaemonSet YAML
        ↓
7. Check ConfigMaps, Secrets, PVCs, Services
        ↓
8. Fix root cause
        ↓
9. Monitor rollout
        ↓
10. Verify Running + Ready
```

---

# 16. Troubleshooting by Pod Status

| Pod Status | Primary Investigation |
|---|---|
| `Pending` | scheduler events, resources, node constraints |
| `ContainerCreating` | mounts, images, ConfigMaps, Secrets, CNI |
| `ErrImagePull` | image name, tag, registry access |
| `ImagePullBackOff` | image/registry/authentication issue |
| `CrashLoopBackOff` | application logs |
| `CreateContainerConfigError` | ConfigMaps, Secrets, env references |
| `Running 0/1` | readiness probe and application health |
| `Terminating` | finalizers, volume detach, node problems |

---

# 17. Day 59 Root Cause

Incorrect image:

```text
redis:alpin
```

Correct image:

```text
redis:alpine
```

Incorrect ConfigMap:

```text
redis-conig
```

Correct ConfigMap:

```text
redis-config
```

Once both values were fixed in the Deployment, Kubernetes successfully created a healthy replacement pod.

---

# 18. Reusable Troubleshooting Decision Tree

```text
Pod unhealthy
     |
     v
kubectl get pods
     |
     +----------------------------+
     |                            |
     v                            v
Pending                     ContainerCreating
     |                            |
     v                            v
describe pod                describe pod
     |                            |
     v                            v
scheduler events        image / mounts / config


CrashLoopBackOff
     |
     v
kubectl logs
     |
     v
application/runtime problem


ImagePullBackOff
     |
     v
describe pod
     |
     v
image / registry / auth


Running but 0/1
     |
     v
describe pod + logs
     |
     v
readiness / application health
```

---

# 19. Final Troubleshooting Checklist

```text
[ ] Is the Deployment healthy?
[ ] Is the ReplicaSet created?
[ ] Is the Pod scheduled?
[ ] Is the image name correct?
[ ] Is the image tag valid?
[ ] Can Kubernetes pull the image?
[ ] Do referenced ConfigMaps exist?
[ ] Do referenced Secrets exist?
[ ] Are volume references correct?
[ ] Are environment references valid?
[ ] Are container ports correct?
[ ] Are health probes passing?
[ ] What do Events say?
[ ] What do Logs say?
[ ] Did the rollout complete?
[ ] Is the Pod Running?
[ ] Is the Pod Ready?
```

---

# 20. Key Learning

The core lesson from Day 59 is:

```text
Pod status shows the symptom.
Events and logs reveal the cause.
The controller configuration is where the permanent fix belongs.
```

This troubleshooting method is reusable across Deployments, StatefulSets, DaemonSets, Jobs, and many real Kubernetes incidents.
