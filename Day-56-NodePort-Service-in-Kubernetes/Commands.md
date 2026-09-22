# Day 56 — Commands Reference

This file contains the commands used to complete and verify the Kubernetes Nginx Deployment and NodePort Service challenge.

---

## 1. Create the Kubernetes Manifest

```bash
cat <<'EOF' > day56-nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
          ports:
            - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service

spec:
  type: NodePort

  selector:
    app: nginx

  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30011
EOF
```

---

## 2. Apply the Manifest

```bash
kubectl apply -f day56-nginx.yaml
```

Expected resources:

```text
deployment.apps/nginx-deployment created
service/nginx-service created
```

---

## 3. Check Deployment Rollout

```bash
kubectl rollout status deployment/nginx-deployment
```

Expected:

```text
deployment "nginx-deployment" successfully rolled out
```

---

## 4. Verify the Deployment

```bash
kubectl get deployment nginx-deployment
```

Expected replica status:

```text
READY   3/3
```

For more details:

```bash
kubectl describe deployment nginx-deployment
```

---

## 5. Verify Pods

```bash
kubectl get pods -l app=nginx
```

Detailed output:

```bash
kubectl get pods -l app=nginx -o wide
```

All three Pods should eventually show:

```text
READY   STATUS
1/1     Running
```

---

## 6. Verify Container Name and Image

```bash
kubectl get deployment nginx-deployment \
-o jsonpath='{.spec.template.spec.containers[0].name}{"\n"}{.spec.template.spec.containers[0].image}{"\n"}'
```

Expected:

```text
nginx-container
nginx:latest
```

---

## 7. Verify Replica Count

```bash
kubectl get deployment nginx-deployment \
-o jsonpath='{.spec.replicas}{"\n"}'
```

Expected:

```text
3
```

---

## 8. Verify the Service

```bash
kubectl get svc nginx-service
```

Expected service characteristics:

```text
TYPE       PORT(S)
NodePort   80:30011/TCP
```

Detailed information:

```bash
kubectl describe svc nginx-service
```

---

## 9. Verify Service Type and NodePort

```bash
kubectl get svc nginx-service \
-o jsonpath='{.spec.type}{"\n"}{.spec.ports[0].nodePort}{"\n"}'
```

Expected:

```text
NodePort
30011
```

---

## 10. Verify Service Endpoints

```bash
kubectl get endpoints nginx-service
```

The endpoints should contain the IP addresses of the Nginx Pods on port `80`.

Alternative:

```bash
kubectl describe svc nginx-service
```

Look for:

```text
Selector:    app=nginx
Endpoints:   <pod-ip>:80,...
```

---

## 11. View Cluster Nodes

```bash
kubectl get nodes -o wide
```

This displays node IP addresses that may be used to test the NodePort service.

---

## 12. Test the Application

```bash
curl http://<NODE-IP>:30011
```

A successful response should return the default Nginx HTML page.

---

## 13. Combined Verification

```bash
echo "=== Deployment ==="

kubectl get deployment nginx-deployment \
-o jsonpath='Replicas: {.spec.replicas}{"\n"}Container: {.spec.template.spec.containers[0].name}{"\n"}Image: {.spec.template.spec.containers[0].image}{"\n"}'

echo
echo "=== Service ==="

kubectl get svc nginx-service \
-o jsonpath='Type: {.spec.type}{"\n"}NodePort: {.spec.ports[0].nodePort}{"\n"}'

echo
echo "=== Pods ==="

kubectl get pods -l app=nginx
```

Expected key values:

```text
Replicas: 3
Container: nginx-container
Image: nginx:latest

Type: NodePort
NodePort: 30011
```

---

## 14. Useful Troubleshooting Commands

### Check all relevant resources

```bash
kubectl get deployment,pods,svc
```

### Inspect Pod events

```bash
kubectl describe pod <pod-name>
```

### Check Nginx container logs

```bash
kubectl logs <pod-name>
```

### Watch Pods in real time

```bash
kubectl get pods -w
```

### Check rollout history

```bash
kubectl rollout history deployment/nginx-deployment
```

### Restart the Deployment if needed

```bash
kubectl rollout restart deployment/nginx-deployment
```

### Verify labels

```bash
kubectl get pods --show-labels
```

---

## Cleanup Commands

Use these only if the resources need to be removed:

```bash
kubectl delete -f day56-nginx.yaml
```

Or:

```bash
kubectl delete deployment nginx-deployment
kubectl delete service nginx-service
```
