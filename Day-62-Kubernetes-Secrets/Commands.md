# Day 62 — Commands

This file contains the commands used to complete and verify the Kubernetes Secrets challenge.

---

## 1. Verify the Source Secret File

```bash
ls -l /opt/blog.txt
```

Optional content check:

```bash
cat /opt/blog.txt
```

---

## 2. Create the Kubernetes Secret

```bash
kubectl create secret generic blog   --from-file=blog.txt=/opt/blog.txt
```

This creates:

```text
Secret name: blog
Secret key:  blog.txt
Secret value: contents of /opt/blog.txt
```

---

## 3. Verify the Secret

```bash
kubectl get secret blog
```

Inspect its metadata:

```bash
kubectl describe secret blog
```

---

## 4. Create the Pod Manifest

```bash
vi /tmp/secret-datacenter.yaml
```

Add:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-datacenter
spec:
  containers:
    - name: secret-container-datacenter
      image: fedora:latest
      command:
        - /bin/sh
        - -c
        - "sleep infinity"
      volumeMounts:
        - name: blog-secret-volume
          mountPath: /opt/games
          readOnly: true

  volumes:
    - name: blog-secret-volume
      secret:
        secretName: blog
```

---

## 5. Create the Pod

```bash
kubectl apply -f /tmp/secret-datacenter.yaml
```

---

## 6. Check Pod Status

```bash
kubectl get pods
```

Or check the specific Pod:

```bash
kubectl get pod secret-datacenter
```

Expected state:

```text
Running
```

---

## 7. Inspect the Pod

```bash
kubectl describe pod secret-datacenter
```

Confirm:

- Container name is `secret-container-datacenter`
- Image is `fedora:latest`
- `/opt/games` is mounted from the Secret-backed volume
- Secret `blog` is referenced by the volume

---

## 8. Enter the Container

```bash
kubectl exec -it secret-datacenter   -c secret-container-datacenter -- /bin/bash
```

If Bash is unavailable:

```bash
kubectl exec -it secret-datacenter   -c secret-container-datacenter -- /bin/sh
```

---

## 9. Verify the Secret Mount

Inside the container:

```bash
ls -l /opt/games
```

Display the secret:

```bash
cat /opt/games/blog.txt
```

Exit:

```bash
exit
```

---

## 10. Verify Without Opening an Interactive Shell

```bash
kubectl exec secret-datacenter   -c secret-container-datacenter --   cat /opt/games/blog.txt
```

---

## Useful Troubleshooting Commands

### Check all Pods

```bash
kubectl get pods -o wide
```

### View Pod events and configuration

```bash
kubectl describe pod secret-datacenter
```

### Check the Secret

```bash
kubectl get secret blog
```

### Inspect Secret YAML

```bash
kubectl get secret blog -o yaml
```

> Secret values displayed in Kubernetes YAML are Base64-encoded representations, not encrypted plaintext.

### Decode a specific Secret value

```bash
kubectl get secret blog   -o jsonpath='{.data.blog\.txt}' | base64 --decode
```

### Delete the Pod if recreation is required

```bash
kubectl delete pod secret-datacenter
```

### Re-apply the manifest

```bash
kubectl apply -f /tmp/secret-datacenter.yaml
```
