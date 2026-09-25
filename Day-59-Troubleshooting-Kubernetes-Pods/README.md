# DevOps Day 59 — Troubleshooting Kubernetes Pods

## Task Overview

The Nautilus DevOps team had an existing Redis application running on Kubernetes. After configuration changes were made, the application went down and the pods managed by the Deployment were no longer reaching the `Running` state.

The task was to troubleshoot and repair the existing Deployment:

```text
redis-deployment
```

without replacing it with a new deployment.

This challenge focused on practical **Kubernetes pod troubleshooting** using pod status, events, Deployment inspection, ConfigMap validation, and rollout verification.

---

## Problem Identified

Two configuration mistakes were found in the existing Deployment:

1. Incorrect Redis image:

```text
redis:alpin
```

Correct value:

```text
redis:alpine
```

2. Incorrect ConfigMap reference:

```text
redis-conig
```

Correct value:

```text
redis-config
```

Because of these mistakes, the Redis pod could not start successfully.

---

## Troubleshooting Workflow

```text
Deployment unhealthy
        |
        v
Check Pod status
        |
        v
Describe failing Pod
        |
        v
Inspect Events
        |
        v
Inspect Deployment YAML
        |
        v
Validate ConfigMap
        |
        v
Fix configuration
        |
        v
Verify rollout
        |
        v
Confirm Pod is Running and Ready
```

---

## Step 1 — Inspect Kubernetes Resources

```bash
kubectl get deployment redis-deployment
kubectl get pods
kubectl get configmap
```

The ConfigMap list confirmed that the valid ConfigMap was:

```text
redis-config
```

---

## Step 2 — Inspect the Failing Pod

```bash
POD=$(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}')
kubectl describe pod "$POD"
```

The `Events` section revealed that Kubernetes was trying to mount a non-existent ConfigMap:

```text
redis-conig
```

---

## Step 3 — Inspect the Deployment

```bash
kubectl get deployment redis-deployment -o yaml
```

The live configuration revealed:

```yaml
image: redis:alpin
```

and:

```yaml
configMap:
  name: redis-conig
```

Both values were incorrect.

---

## Step 4 — Fix the Existing Deployment

```bash
kubectl edit deployment redis-deployment
```

The image was corrected to:

```yaml
image: redis:alpine
```

The ConfigMap reference was corrected to:

```yaml
configMap:
  name: redis-config
```

No unnecessary changes were made.

---

## Step 5 — Verify the Rollout

```bash
kubectl rollout status deployment/redis-deployment
```

After the Deployment template was corrected, Kubernetes created a new ReplicaSet and replacement pod automatically.

---

## Step 6 — Verify Final State

```bash
kubectl get deployment redis-deployment
kubectl get pods -l app=redis
```

Expected healthy state:

```text
Deployment: 1/1 READY
Pod:        1/1 Running
```

The corrected values were also verified directly:

```bash
kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.containers[*].image}{"\n"}'
```

```bash
kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.volumes[?(@.name=="config")].configMap.name}{"\n"}'
```

Expected output:

```text
redis:alpine
redis-config
```

---

## Final Architecture

```text
redis-deployment
       |
       v
ReplicaSet
       |
       v
Redis Pod
       |
       +--> Image: redis:alpine
       |
       +--> ConfigMap: redis-config
       |
       v
     Running
```

---

## Key Kubernetes Troubleshooting Concepts

This challenge reinforced the following practices:

- Start with `kubectl get` to identify the symptom.
- Use `kubectl describe pod` to investigate why the pod is failing.
- Read the `Events` section carefully.
- Use `kubectl logs` for application/runtime failures.
- Inspect the live controller configuration with `kubectl get ... -o yaml`.
- Validate ConfigMaps, Secrets, volumes, images, ports, and selectors.
- Fix the controller instead of manually modifying Deployment-managed pods.
- Use `kubectl rollout status` after changing a Deployment.
- Confirm both `Running` and `Ready` states.

---

## Result

The Day 59 challenge was completed successfully.

The existing Redis Deployment was repaired by correcting the Redis image and ConfigMap reference. Kubernetes recreated the pod successfully, and the Redis workload returned to a healthy running state.

---
## Conclusion

The most important lesson from this task is to troubleshoot Kubernetes systematically rather than making random changes.

A reliable debugging path is:

```text
Deployment
   ↓
ReplicaSet
   ↓
Pod
   ↓
Events / Logs / Live Configuration
   ↓
Root Cause
```

Following this sequence makes Kubernetes troubleshooting faster, safer, and easier to reproduce.
