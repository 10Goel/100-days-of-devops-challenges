# 🛠️ Day 63 — Commands

This file contains the commands used to deploy and verify the Iron Gallery and Iron DB workloads.

---

## 1️⃣ Create the Namespace

```bash
kubectl create namespace iron-namespace-nautilus
```

Verify:

```bash
kubectl get namespace iron-namespace-nautilus
```

---

## 2️⃣ Create the Manifest File

```bash
vi /tmp/iron-gallery.yaml
```

Add the required Deployments and Services.

---

## 3️⃣ Apply the Manifest

```bash
kubectl apply -f /tmp/iron-gallery.yaml
```

---

## 4️⃣ Verify Deployments

```bash
kubectl get deployments -n iron-namespace-nautilus
```

```bash
kubectl describe deployment iron-gallery-deployment-nautilus \
  -n iron-namespace-nautilus
```

```bash
kubectl describe deployment iron-db-deployment-nautilus \
  -n iron-namespace-nautilus
```

---

## 5️⃣ Verify Pods

```bash
kubectl get pods -n iron-namespace-nautilus
```

```bash
kubectl get pods -n iron-namespace-nautilus -o wide
```

---

## 6️⃣ Verify Services

```bash
kubectl get svc -n iron-namespace-nautilus
```

```bash
kubectl get svc iron-gallery-service-nautilus \
  -n iron-namespace-nautilus -o yaml
```

```bash
kubectl get svc iron-db-service-nautilus \
  -n iron-namespace-nautilus -o yaml
```

---

## 7️⃣ Verify Endpoints

```bash
kubectl get endpoints -n iron-namespace-nautilus
```

Both Services should have backend endpoints.

---

## 8️⃣ Verify Gallery Labels

```bash
kubectl get pods -n iron-namespace-nautilus \
  -l run=iron-gallery --show-labels
```

---

## 9️⃣ Verify DB Labels

```bash
kubectl get pods -n iron-namespace-nautilus \
  -l db=mariadb --show-labels
```

---

## 🔟 Verify Resource Limits

```bash
kubectl describe deployment iron-gallery-deployment-nautilus \
  -n iron-namespace-nautilus
```

Expected:

```text
Limits:
  cpu:     50m
  memory:  100Mi
```

---

## 1️⃣1️⃣ Verify Gallery Volume Mounts

```bash
kubectl describe pod \
  -n iron-namespace-nautilus \
  -l run=iron-gallery
```

Expected mount paths:

```text
/usr/share/nginx/html/data
/usr/share/nginx/html/uploads
```

---

## 1️⃣2️⃣ Verify DB Volume Mount

```bash
kubectl describe pod \
  -n iron-namespace-nautilus \
  -l db=mariadb
```

Expected:

```text
/var/lib/mysql
```

---

## 1️⃣3️⃣ Verify Database Environment Variables

```bash
kubectl exec -n iron-namespace-nautilus \
  deployment/iron-db-deployment-nautilus -- env | grep MYSQL
```

---

## 1️⃣4️⃣ Verify Gallery NodePort

```bash
kubectl get svc iron-gallery-service-nautilus \
  -n iron-namespace-nautilus
```

Expected service mapping:

```text
80:32678/TCP
```

---

## 1️⃣5️⃣ Get Node IP

```bash
kubectl get nodes -o wide
```

---

## 1️⃣6️⃣ Test the Application

```bash
curl http://<NODE-IP>:32678
```

The Iron Gallery installation page should be returned.

---

# 🔍 Troubleshooting Commands

## Check Pod Events

```bash
kubectl describe pod <pod-name> \
  -n iron-namespace-nautilus
```

## Check Logs

```bash
kubectl logs <pod-name> \
  -n iron-namespace-nautilus
```

For a specific container:

```bash
kubectl logs <pod-name> \
  -c <container-name> \
  -n iron-namespace-nautilus
```

## Check Deployment Rollout

```bash
kubectl rollout status deployment/iron-gallery-deployment-nautilus \
  -n iron-namespace-nautilus
```

```bash
kubectl rollout status deployment/iron-db-deployment-nautilus \
  -n iron-namespace-nautilus
```

## List All Main Resources

```bash
kubectl get all -n iron-namespace-nautilus
```

## Delete and Recreate if Required

```bash
kubectl delete -f /tmp/iron-gallery.yaml
```

```bash
kubectl apply -f /tmp/iron-gallery.yaml
```
