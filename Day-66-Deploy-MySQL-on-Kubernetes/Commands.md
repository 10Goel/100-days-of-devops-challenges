# Day 66 — Commands Reference

## 1. Create the Kubernetes manifest

```bash
vi mysql.yaml
```

---

## 2. Apply all resources

```bash
kubectl apply -f mysql.yaml
```

---

## 3. Verify PersistentVolume

```bash
kubectl get pv
kubectl get pv mysql-pv
kubectl describe pv mysql-pv
```

---

## 4. Verify PersistentVolumeClaim

```bash
kubectl get pvc
kubectl get pvc mysql-pv-claim
kubectl describe pvc mysql-pv-claim
```

---

## 5. Verify PV/PVC binding

```bash
kubectl get pv,pvc
```

Expected:

```text
mysql-pv        Bound
mysql-pv-claim  Bound
```

---

## 6. List Secrets

```bash
kubectl get secrets
```

---

## 7. Inspect Secrets

```bash
kubectl describe secret mysql-root-pass
kubectl describe secret mysql-user-pass
kubectl describe secret mysql-db-url
```

---

## 8. Verify Deployment

```bash
kubectl get deployment
kubectl get deployment mysql-deployment
kubectl describe deployment mysql-deployment
```

---

## 9. Verify Pods

```bash
kubectl get pods
kubectl get pods -l app=mysql
```

Watch Pod startup:

```bash
kubectl get pods -w
```

---

## 10. Describe the MySQL Pod

```bash
kubectl describe pod $(kubectl get pods -l app=mysql -o jsonpath='{.items[0].metadata.name}')
```

---

## 11. Check MySQL Logs

```bash
kubectl logs $(kubectl get pods -l app=mysql -o jsonpath='{.items[0].metadata.name}')
```

---

## 12. Verify the Service

```bash
kubectl get service mysql
kubectl get svc mysql
kubectl describe service mysql
```

Expected mapping:

```text
3306:30007/TCP
```

---

## 13. Check Service Endpoints

```bash
kubectl get endpoints mysql
```

---

## 14. Verify Root Login and Databases

```bash
kubectl exec -it $(kubectl get pods -l app=mysql -o jsonpath='{.items[0].metadata.name}') -- mysql -u root -pYUIidhb667 -e "SHOW DATABASES;"
```

---

## 15. Verify Custom User Login

```bash
kubectl exec -it $(kubectl get pods -l app=mysql -o jsonpath='{.items[0].metadata.name}') -- mysql -u kodekloud_aim -pGyQkFRVNr3 -e "SHOW DATABASES;"
```

---

## 16. Check MySQL Environment Variables

```bash
kubectl exec -it $(kubectl get pods -l app=mysql -o jsonpath='{.items[0].metadata.name}') -- env | grep MYSQL
```

---

## 17. Verify Mounted Storage

```bash
kubectl exec -it $(kubectl get pods -l app=mysql -o jsonpath='{.items[0].metadata.name}') -- ls -lah /var/lib/mysql
```

---

## 18. Inspect Resource YAML

```bash
kubectl get deployment mysql-deployment -o yaml
kubectl get service mysql -o yaml
kubectl get pv mysql-pv -o yaml
kubectl get pvc mysql-pv-claim -o yaml
```

---

## 19. Troubleshooting Commands

Check all application resources:

```bash
kubectl get all
```

Check storage:

```bash
kubectl get pv,pvc
```

Check cluster events:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Check Pod details:

```bash
kubectl describe pod $(kubectl get pods -l app=mysql -o jsonpath='{.items[0].metadata.name}')
```

Check application logs:

```bash
kubectl logs $(kubectl get pods -l app=mysql -o jsonpath='{.items[0].metadata.name}')
```

---

## Final Validation Sequence

```bash
kubectl get pv
kubectl get pvc
kubectl get secrets
kubectl get deployment mysql-deployment
kubectl get pods
kubectl get service mysql
```

Expected final state:

```text
mysql-pv          Bound
mysql-pv-claim    Bound
mysql-deployment  1/1
mysql Pod         1/1 Running
mysql Service     NodePort 3306:30007/TCP
```
