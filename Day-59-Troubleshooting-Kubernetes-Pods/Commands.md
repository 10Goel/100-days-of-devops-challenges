# DevOps Day 59 — Commands

## 1. Check Deployment Status

```bash
kubectl get deployment redis-deployment
```

## 2. Check Pod Status

```bash
kubectl get pods
kubectl get pods -l app=redis
kubectl get pods -l app=redis -o wide
```

## 3. Check ConfigMaps

```bash
kubectl get configmap
kubectl get configmap redis-config
kubectl get configmap redis-config -o yaml
```

## 4. Identify the Redis Pod

```bash
POD=$(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}')
echo "$POD"
```

## 5. Describe the Failing Pod

```bash
kubectl describe pod "$POD"
```

Pay special attention to:

```text
State
Reason
Volumes
Mounts
Conditions
Events
```

## 6. Inspect Kubernetes Events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Optional Redis filter:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp | grep -i redis
```

## 7. Inspect the Deployment YAML

```bash
kubectl get deployment redis-deployment -o yaml
```

Check the image:

```bash
kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.containers[*].image}{"\n"}'
```

Check the ConfigMap reference:

```bash
kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.volumes[?(@.name=="config")].configMap.name}{"\n"}'
```

## 8. Fix the Deployment

```bash
kubectl edit deployment redis-deployment
```

Correct:

```yaml
image: redis:alpin
```

to:

```yaml
image: redis:alpine
```

Correct:

```yaml
configMap:
  name: redis-conig
```

to:

```yaml
configMap:
  name: redis-config
```

## 9. Alternative Non-Interactive Fix

Correct the image:

```bash
kubectl set image deployment/redis-deployment redis-container=redis:alpine
```

Correct the ConfigMap:

```bash
kubectl patch deployment redis-deployment --type='strategic' -p='{"spec":{"template":{"spec":{"volumes":[{"name":"config","configMap":{"name":"redis-config"}}]}}}}'
```

Use either the edit method or the patch method.

## 10. Monitor Pod Recreation

```bash
kubectl get pods -w
```

Stop watching with `Ctrl+C`.

## 11. Verify Rollout

```bash
kubectl rollout status deployment/redis-deployment
kubectl get deployment redis-deployment
```

## 12. Verify Final Pod State

```bash
kubectl get pods -l app=redis
```

Expected:

```text
1/1 Running
```

## 13. Verify Corrected Image

```bash
kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.containers[*].image}{"\n"}'
```

Expected:

```text
redis:alpine
```

## 14. Verify Corrected ConfigMap Reference

```bash
kubectl get deployment redis-deployment -o jsonpath='{.spec.template.spec.volumes[?(@.name=="config")].configMap.name}{"\n"}'
```

Expected:

```text
redis-config
```

## 15. Check Container Logs

```bash
POD=$(kubectl get pods -l app=redis --field-selector=status.phase=Running -o jsonpath='{.items[0].metadata.name}')

kubectl logs "$POD"
```

If a container has restarted:

```bash
kubectl logs "$POD" --previous
```

## 16. Inspect ReplicaSets

```bash
kubectl get replicasets
kubectl get replicasets | grep redis
```

## 17. Inspect All Redis Resources

```bash
kubectl get all -l app=redis
```

## 18. Useful Troubleshooting Commands

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
kubectl describe deployment redis-deployment
kubectl get deployment redis-deployment -o yaml
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get nodes
kubectl get all
```

## 19. Final Verification

```bash
kubectl get deployment redis-deployment
kubectl get pods -l app=redis
kubectl get configmap redis-config
kubectl rollout status deployment/redis-deployment
```

The challenge is complete when the Deployment is healthy and the Redis pod is:

```text
1/1 Running
```
