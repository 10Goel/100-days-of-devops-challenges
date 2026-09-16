# Day 50 — Kubernetes Resource Requests and Limits Commands

This file contains the commands used to create and verify the Day 50 Kubernetes Pod.

---

## 1. Verify Kubernetes Access

```bash
kubectl get nodes
```

---

## 2. Create the Pod Manifest

```bash
cat > /tmp/httpd-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
spec:
  containers:
    - name: httpd-container
      image: httpd:latest
      resources:
        requests:
          memory: "15Mi"
          cpu: "100m"
        limits:
          memory: "20Mi"
          cpu: "100m"
EOF
```

---

## 3. Review the Manifest

```bash
cat /tmp/httpd-pod.yaml
```

---

## 4. Create the Pod

```bash
kubectl apply -f /tmp/httpd-pod.yaml
```

Expected:

```text
pod/httpd-pod created
```

---

## 5. Verify Pod Status

```bash
kubectl get pod httpd-pod
```

Expected:

```text
NAME        READY   STATUS    RESTARTS
httpd-pod   1/1     Running   0
```

---

## 6. Get Detailed Pod Information

```bash
kubectl get pod httpd-pod -o wide
```

---

## 7. Describe the Pod

```bash
kubectl describe pod httpd-pod
```

Look for:

```text
Limits:
  cpu:     100m
  memory:  20Mi

Requests:
  cpu:     100m
  memory:  15Mi
```

---

## 8. Verify Container Name and Image

```bash
kubectl get pod httpd-pod   -o jsonpath='{.spec.containers[0].name}{"\n"}{.spec.containers[0].image}{"\n"}'
```

Expected:

```text
httpd-container
httpd:latest
```

---

## 9. Verify CPU Request

```bash
kubectl get pod httpd-pod   -o jsonpath='{.spec.containers[0].resources.requests.cpu}'; echo
```

Expected:

```text
100m
```

---

## 10. Verify Memory Request

```bash
kubectl get pod httpd-pod   -o jsonpath='{.spec.containers[0].resources.requests.memory}'; echo
```

Expected:

```text
15Mi
```

---

## 11. Verify CPU Limit

```bash
kubectl get pod httpd-pod   -o jsonpath='{.spec.containers[0].resources.limits.cpu}'; echo
```

Expected:

```text
100m
```

---

## 12. Verify Memory Limit

```bash
kubectl get pod httpd-pod   -o jsonpath='{.spec.containers[0].resources.limits.memory}'; echo
```

Expected:

```text
20Mi
```

---

## 13. Verify All Resource Values Together

```bash
kubectl get pod httpd-pod   -o jsonpath='CPU Request: {.spec.containers[0].resources.requests.cpu}{"\n"}Memory Request: {.spec.containers[0].resources.requests.memory}{"\n"}CPU Limit: {.spec.containers[0].resources.limits.cpu}{"\n"}Memory Limit: {.spec.containers[0].resources.limits.memory}{"\n"}'
```

Expected:

```text
CPU Request: 100m
Memory Request: 15Mi
CPU Limit: 100m
Memory Limit: 20Mi
```

---

## 14. View the Live Pod YAML

```bash
kubectl get pod httpd-pod -o yaml
```

---

## Final Verification

```bash
kubectl get pod httpd-pod
kubectl describe pod httpd-pod
```

Confirm:

```text
Pod             = httpd-pod
Container       = httpd-container
Image           = httpd:latest
CPU Request     = 100m
Memory Request  = 15Mi
CPU Limit       = 100m
Memory Limit    = 20Mi
```

---

## Optional Cleanup

Do not run this before validation.

```bash
kubectl delete pod httpd-pod
```
