# Day 65 — Commands Reference

## 1. Create the Kubernetes manifest

```bash
vi redis.yaml
```

Paste the required ConfigMap and Deployment YAML, then save and exit.

---

## 2. Apply the manifest

```bash
kubectl apply -f redis.yaml
```

Expected output:

```text
configmap/my-redis-config created
deployment.apps/redis-deployment created
```

---

## 3. List ConfigMaps

```bash
kubectl get configmaps
```

Or inspect the required ConfigMap directly:

```bash
kubectl get configmap my-redis-config
```

---

## 4. Describe the ConfigMap

```bash
kubectl describe configmap my-redis-config
```

Useful for checking that the key and value are correct:

```text
redis-config
maxmemory 2mb
```

---

## 5. View the ConfigMap as YAML

```bash
kubectl get configmap my-redis-config -o yaml
```

---

## 6. Check the Deployment

```bash
kubectl get deployments
```

Or:

```bash
kubectl get deployment redis-deployment
```

---

## 7. Describe the Deployment

```bash
kubectl describe deployment redis-deployment
```

Check:

- Image: `redis:alpine`
- Container: `redis-container`
- Port: `6379`
- CPU request: `1`
- Volume mounts
- Replica status

---

## 8. Check Pod status

```bash
kubectl get pods
```

Filter only the Redis pod:

```bash
kubectl get pods -l app=redis
```

---

## 9. Describe the Redis Pod

```bash
kubectl describe pod $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}')
```

Use this to verify:

- Container image
- Port
- CPU request
- Volume mounts
- `emptyDir` volume
- ConfigMap volume
- Pod events

---

## 10. Check the mounted Redis configuration file

```bash
kubectl exec -it $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}') -- cat /redis-master/redis-config
```

Expected:

```text
maxmemory 2mb
```

---

## 11. Confirm Redis is responding

```bash
kubectl exec -it $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}') -- redis-cli ping
```

Expected:

```text
PONG
```

---

## 12. Verify Redis maxmemory value

```bash
kubectl exec -it $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}') -- redis-cli CONFIG GET maxmemory
```

Expected:

```text
maxmemory
2097152
```

---

## 13. Inspect the full Deployment YAML

```bash
kubectl get deployment redis-deployment -o yaml
```

---

## 14. Inspect the Pod YAML

```bash
kubectl get pod $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}') -o yaml
```

---

## 15. Check Pod logs if troubleshooting is required

```bash
kubectl logs $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}')
```

---

## 16. Useful troubleshooting commands

Check all Kubernetes resources:

```bash
kubectl get all
```

Watch Pod state changes:

```bash
kubectl get pods -w
```

Check recent events:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Check the Redis process:

```bash
kubectl exec -it $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}') -- ps
```

---

## Quick Final Validation

```bash
kubectl get configmap my-redis-config
kubectl get deployment redis-deployment
kubectl get pods -l app=redis
```

The final Pod state should be:

```text
1/1 Running
```
