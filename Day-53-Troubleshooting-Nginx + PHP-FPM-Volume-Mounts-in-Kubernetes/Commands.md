# DevOps Challenge – Day 53  
## Commands Reference

This file contains the commands used to troubleshoot and resolve the Nginx + PHP-FPM Kubernetes volume-mount issue.

---

## 1. Check the Pod

```bash
kubectl get pod nginx-phpfpm
```

---

## 2. Inspect Pod Details

```bash
kubectl describe pod nginx-phpfpm
```

Useful for checking:

- Container names
- Container images
- Volume mounts
- Pod volumes
- Container status
- Events

---

## 3. Inspect the Nginx ConfigMap

```bash
kubectl get configmap nginx-config -o yaml
```

This was used to identify the Nginx document root.

---

## 4. Display Container Volume Mounts

```bash
kubectl get pod nginx-phpfpm   -o jsonpath='{range .spec.containers[*]}{.name}{"\n"}{range .volumeMounts[*]}{"  "}{.name}{" -> "}{.mountPath}{"\n"}{end}{end}'
```

---

## 5. Back Up the Existing Pod Specification

```bash
kubectl get pod nginx-phpfpm -o yaml > /tmp/nginx-phpfpm-original.yaml
```

---

## 6. Check Pod Labels

```bash
kubectl get pod nginx-phpfpm --show-labels
```

---

## 7. Create/Edit the Corrected Manifest

```bash
vi /tmp/nginx-phpfpm-fixed.yaml
```

The essential fix was:

```yaml
- name: shared-files
  mountPath: /var/www/html
```

for the Nginx container.

---

## 8. Delete the Existing Pod

```bash
kubectl delete pod nginx-phpfpm
```

---

## 9. Create the Corrected Pod

```bash
kubectl apply -f /tmp/nginx-phpfpm-fixed.yaml
```

---

## 10. Wait for Pod Readiness

```bash
kubectl wait --for=condition=Ready pod/nginx-phpfpm --timeout=120s
```

Alternative:

```bash
kubectl get pods -w
```

---

## 11. Verify Corrected Volume Mounts

```bash
kubectl get pod nginx-phpfpm   -o jsonpath='{range .spec.containers[*]}{.name}{"\n"}{range .volumeMounts[*]}{"  "}{.name}{" -> "}{.mountPath}{"\n"}{end}{end}'
```

The shared volume should now appear at:

```text
/var/www/html
```

inside both containers.

---

## 12. Copy `index.php`

```bash
kubectl cp /home/thor/index.php   nginx-phpfpm:/var/www/html/index.php   -c nginx-container
```

---

## 13. Verify the File from Nginx

```bash
kubectl exec nginx-phpfpm -c nginx-container --   ls -l /var/www/html/
```

---

## 14. Verify the Same File from PHP-FPM

```bash
kubectl exec nginx-phpfpm -c php-fpm-container --   ls -l /var/www/html/
```

---

## 15. Validate Nginx Configuration

```bash
kubectl exec nginx-phpfpm -c nginx-container -- nginx -t
```

---

## 16. Verify Final Pod Status

```bash
kubectl get pod nginx-phpfpm
```

Expected:

```text
READY   STATUS
2/2     Running
```

---

# Additional Troubleshooting Commands

## View Full Pod YAML

```bash
kubectl get pod nginx-phpfpm -o yaml
```

## Describe the ConfigMap

```bash
kubectl describe configmap nginx-config
```

## Nginx Logs

```bash
kubectl logs nginx-phpfpm -c nginx-container
```

## PHP-FPM Logs

```bash
kubectl logs nginx-phpfpm -c php-fpm-container
```

## Open a Shell in Nginx

```bash
kubectl exec -it nginx-phpfpm -c nginx-container -- /bin/sh
```

## Open a Shell in PHP-FPM

```bash
kubectl exec -it nginx-phpfpm -c php-fpm-container -- /bin/sh
```

## List Nginx Web Root

```bash
kubectl exec nginx-phpfpm -c nginx-container --   ls -la /var/www/html/
```

## List PHP-FPM Web Root

```bash
kubectl exec nginx-phpfpm -c php-fpm-container --   ls -la /var/www/html/
```

---

# Minimal Command Sequence

```bash
kubectl get pod nginx-phpfpm

kubectl describe pod nginx-phpfpm

kubectl get configmap nginx-config -o yaml

kubectl get pod nginx-phpfpm   -o jsonpath='{range .spec.containers[*]}{.name}{"\n"}{range .volumeMounts[*]}{"  "}{.name}{" -> "}{.mountPath}{"\n"}{end}{end}'

kubectl get pod nginx-phpfpm -o yaml > /tmp/nginx-phpfpm-original.yaml

vi /tmp/nginx-phpfpm-fixed.yaml

kubectl delete pod nginx-phpfpm

kubectl apply -f /tmp/nginx-phpfpm-fixed.yaml

kubectl wait --for=condition=Ready pod/nginx-phpfpm --timeout=120s

kubectl cp /home/thor/index.php   nginx-phpfpm:/var/www/html/index.php   -c nginx-container

kubectl exec nginx-phpfpm -c nginx-container --   ls -l /var/www/html/

kubectl exec nginx-phpfpm -c php-fpm-container --   ls -l /var/www/html/

kubectl exec nginx-phpfpm -c nginx-container -- nginx -t

kubectl get pod nginx-phpfpm
```
