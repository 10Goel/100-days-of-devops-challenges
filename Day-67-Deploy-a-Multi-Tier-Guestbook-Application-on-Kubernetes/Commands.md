# Day 67 — Commands Reference

## 1. Create the Manifest

```bash
vi guestbook.yaml
```

---

## 2. Apply the Manifest

```bash
kubectl apply -f guestbook.yaml
```

---

## 3. Verify Deployments

```bash
kubectl get deployments
```

Check individually:

```bash
kubectl get deployment redis-master
kubectl get deployment redis-slave
kubectl get deployment frontend
```

---

## 4. Describe Deployments

```bash
kubectl describe deployment redis-master
kubectl describe deployment redis-slave
kubectl describe deployment frontend
```

Use these to verify:

- replica counts
- container names
- images
- ports
- CPU requests
- memory requests
- environment variables

---

## 5. Verify Pods

```bash
kubectl get pods
```

Detailed view:

```bash
kubectl get pods -o wide
```

Expected application Pod count:

```text
6
```

Expected status:

```text
1/1 Running
```

---

## 6. Verify Services

```bash
kubectl get services
```

Or:

```bash
kubectl get svc
```

Check individually:

```bash
kubectl get svc redis-master
kubectl get svc redis-slave
kubectl get svc redis-follower
kubectl get svc frontend
```

---

## 7. Describe Services

```bash
kubectl describe service redis-master
kubectl describe service redis-slave
kubectl describe service redis-follower
kubectl describe service frontend
```

---

## 8. Verify Endpoints

```bash
kubectl get endpoints
```

Check a specific service:

```bash
kubectl get endpoints redis-master
kubectl get endpoints redis-slave
kubectl get endpoints redis-follower
kubectl get endpoints frontend
```

---

## 9. Verify Frontend Environment Variable

```bash
kubectl exec -it $(kubectl get pods -l app=frontend -o jsonpath='{.items[0].metadata.name}') -- env | grep GET_HOSTS_FROM
```

Expected:

```text
GET_HOSTS_FROM=dns
```

---

## 10. Verify Redis Slave Environment Variable

```bash
kubectl exec -it $(kubectl get pods -l app=redis-slave -o jsonpath='{.items[0].metadata.name}') -- env | grep GET_HOSTS_FROM
```

Expected:

```text
GET_HOSTS_FROM=dns
```

---

## 11. Test Kubernetes DNS from Frontend

```bash
kubectl exec -it $(kubectl get pods -l app=frontend -o jsonpath='{.items[0].metadata.name}') -- getent hosts redis-master
```

Test Redis follower:

```bash
kubectl exec -it $(kubectl get pods -l app=frontend -o jsonpath='{.items[0].metadata.name}') -- getent hosts redis-follower
```

---

## 12. Verify Redis Master Logs

```bash
kubectl logs $(kubectl get pods -l app=redis-master -o jsonpath='{.items[0].metadata.name}')
```

---

## 13. Test Redis Master

```bash
kubectl exec -it $(kubectl get pods -l app=redis-master -o jsonpath='{.items[0].metadata.name}') -- redis-cli ping
```

Expected:

```text
PONG
```

---

## 14. Verify Redis Slave Logs

```bash
kubectl logs $(kubectl get pods -l app=redis-slave -o jsonpath='{.items[0].metadata.name}')
```

---

## 15. Describe a Redis Slave Pod

```bash
kubectl describe pod $(kubectl get pods -l app=redis-slave -o jsonpath='{.items[0].metadata.name}')
```

---

## 16. Verify Frontend NodePort

```bash
kubectl get svc frontend
```

Expected:

```text
TYPE       PORT(S)
NodePort   80:30009/TCP
```

Detailed view:

```bash
kubectl describe svc frontend
```

---

## 17. Inspect Deployment YAML

```bash
kubectl get deployment redis-master -o yaml
kubectl get deployment redis-slave -o yaml
kubectl get deployment frontend -o yaml
```

---

## 18. Inspect Service YAML

```bash
kubectl get svc redis-master -o yaml
kubectl get svc redis-slave -o yaml
kubectl get svc redis-follower -o yaml
kubectl get svc frontend -o yaml
```

---

## 19. Troubleshooting Commands

Check all workload resources:

```bash
kubectl get all
```

Check cluster events:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Check Pod details:

```bash
kubectl describe pod <pod-name>
```

Check logs:

```bash
kubectl logs <pod-name>
```

Watch Pod state:

```bash
kubectl get pods -w
```

---

## 20. Final Validation Sequence

```bash
kubectl get deployments
kubectl get pods
kubectl get services
kubectl get endpoints
```

Expected:

```text
Deployments:
frontend       3/3
redis-master   1/1
redis-slave    2/2

Pods:
6 application Pods, all Running

Services:
redis-master
redis-slave
redis-follower
frontend

Frontend:
NodePort 80:30009/TCP
```
