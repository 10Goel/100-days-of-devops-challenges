# DevOps Challenge – Day 53  
## Troubleshooting Nginx + PHP-FPM Volume Mounts in Kubernetes

## Overview

In this challenge, an Nginx and PHP-FPM application running inside a Kubernetes Pod was not functioning correctly even though both containers were operational.

The resources involved were:

```text
Pod:       nginx-phpfpm
ConfigMap: nginx-config
```

The issue was caused by an inconsistent shared-volume mount path between the Nginx and PHP-FPM containers.

The Nginx configuration expected application files under:

```text
/var/www/html
```

but the shared volume inside the Nginx container was mounted at a different path.

The task required identifying the mismatch, correcting the Pod configuration, recreating the Pod, copying the provided `index.php` file into the shared application directory, and verifying that the application worked correctly.

---

## Task Requirements

- Inspect the existing `nginx-phpfpm` Pod.
- Inspect the `nginx-config` ConfigMap.
- Identify the incorrect volume mount.
- Correct the Nginx container's shared-volume path.
- Recreate the Pod with the corrected configuration.
- Copy `/home/thor/index.php` into the Nginx container.
- Verify that both Nginx and PHP-FPM can access the same file.
- Confirm that the website is operational.

---

## Root Cause

The shared volume was mounted differently in the two containers.

### Before

```text
PHP-FPM container
shared-files -> /var/www/html

Nginx container
shared-files -> /usr/share/nginx/html
```

However, the Nginx configuration defined:

```nginx
root /var/www/html;
```

So Nginx expected the PHP file at:

```text
/var/www/html/index.php
```

while its shared volume was mounted somewhere else.

---

## Correct Architecture

Both containers needed access to the same shared application directory:

```text
                    shared-files
                     (emptyDir)
                         |
              +----------+----------+
              |                     |
              v                     v
       nginx-container       php-fpm-container
       /var/www/html         /var/www/html
              |                     |
              +----------+----------+
                         |
                     index.php
```

---

## Solution

### 1. Inspect the Pod

```bash
kubectl get pod nginx-phpfpm
```

```bash
kubectl describe pod nginx-phpfpm
```

This revealed the container configuration and volume mounts.

---

### 2. Inspect the Nginx ConfigMap

```bash
kubectl get configmap nginx-config -o yaml
```

The Nginx configuration showed that the document root was:

```text
/var/www/html
```

---

### 3. Inspect Volume Mounts Directly

```bash
kubectl get pod nginx-phpfpm   -o jsonpath='{range .spec.containers[*]}{.name}{"\n"}{range .volumeMounts[*]}{"  "}{.name}{" -> "}{.mountPath}{"\n"}{end}{end}'
```

This confirmed that the Nginx and PHP-FPM containers did not use the same application path.

---

### 4. Back Up the Existing Pod Manifest

```bash
kubectl get pod nginx-phpfpm -o yaml > /tmp/nginx-phpfpm-original.yaml
```

---

### 5. Create the Corrected Pod Manifest

The important correction was to mount `shared-files` at `/var/www/html` inside the Nginx container:

```yaml
volumeMounts:
  - name: shared-files
    mountPath: /var/www/html
```

The PHP-FPM container already used the required path.

---

### 6. Recreate the Pod

```bash
kubectl delete pod nginx-phpfpm
```

```bash
kubectl apply -f /tmp/nginx-phpfpm-fixed.yaml
```

---

### 7. Wait for the Pod to Become Ready

```bash
kubectl wait --for=condition=Ready pod/nginx-phpfpm --timeout=120s
```

Expected final state:

```text
READY   STATUS
2/2     Running
```

---

### 8. Copy the Application File

```bash
kubectl cp /home/thor/index.php   nginx-phpfpm:/var/www/html/index.php   -c nginx-container
```

Because `/var/www/html` was backed by the shared volume, PHP-FPM could access the same file.

---

### 9. Verify Shared Storage

From Nginx:

```bash
kubectl exec nginx-phpfpm -c nginx-container --   ls -l /var/www/html/
```

From PHP-FPM:

```bash
kubectl exec nginx-phpfpm -c php-fpm-container --   ls -l /var/www/html/
```

Both containers successfully showed:

```text
index.php
```

---

### 10. Validate Nginx Configuration

```bash
kubectl exec nginx-phpfpm -c nginx-container -- nginx -t
```

This confirmed that the Nginx configuration syntax was valid.

---

## Verification Checklist

- [x] `nginx-phpfpm` Pod inspected.
- [x] `nginx-config` ConfigMap inspected.
- [x] Incorrect Nginx volume mount identified.
- [x] Shared volume corrected to `/var/www/html`.
- [x] Pod recreated successfully.
- [x] Both containers reached `Running` state.
- [x] `index.php` copied successfully.
- [x] Both containers could access the same file.
- [x] Nginx configuration validated.
- [x] Website functionality restored.

---

## Key Concepts Practiced

- Kubernetes multi-container Pods
- Shared volumes
- `emptyDir`
- `volumeMounts`
- ConfigMaps
- `subPath`
- Nginx + PHP-FPM architecture
- `kubectl describe`
- `kubectl exec`
- `kubectl cp`
- JSONPath
- Pod recreation
- Kubernetes application troubleshooting

---

## Key Learning

A Pod showing:

```text
2/2 Running
```

does **not** automatically mean the application is working.

Application-level correctness depends on configuration such as:

```text
Application configuration
        +
Volume mounts
        +
Filesystem paths
        +
Container communication
```

In this challenge, Kubernetes successfully ran both containers, but the application failed because Nginx and PHP-FPM did not agree on the location of the application files.

---

## Result

The issue was successfully resolved by mounting the shared application volume at `/var/www/html` in both containers, recreating the Pod, copying the provided `index.php` file into the shared directory, and verifying access from both Nginx and PHP-FPM.

The application became operational after the filesystem-path mismatch was corrected.

---
