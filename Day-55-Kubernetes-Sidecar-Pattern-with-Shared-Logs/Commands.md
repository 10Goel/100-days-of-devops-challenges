# DevOps Challenge Day 55 – Commands

## Create the Kubernetes Manifest

```bash
vi /tmp/webserver.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  initContainers:
    - name: sidecar-container
      image: ubuntu:latest
      restartPolicy: Always
      command:
        - sh
        - -c
        - while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx

  containers:
    - name: nginx-container
      image: nginx:latest
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx

  volumes:
    - name: shared-logs
      emptyDir: {}
```

---

## Apply the Manifest

```bash
kubectl apply -f /tmp/webserver.yaml
```

---

## Verify Pod Status

```bash
kubectl get pod webserver
```

Detailed view:

```bash
kubectl get pod webserver -o wide
```

---

## Verify Main Container

```bash
kubectl get pod webserver -o jsonpath='{.spec.containers[*].name}'
```

Expected:

```text
nginx-container
```

---

## Verify Sidecar Container

```bash
kubectl get pod webserver -o jsonpath='{.spec.initContainers[*].name}'
```

Expected:

```text
sidecar-container
```

---

## Verify Sidecar Restart Policy

```bash
kubectl get pod webserver -o jsonpath='{.spec.initContainers[0].restartPolicy}'
```

Expected:

```text
Always
```

---

## Inspect Pod Configuration

```bash
kubectl describe pod webserver
```

Or:

```bash
kubectl get pod webserver -o yaml
```

---

## Verify Shared Volume

```bash
kubectl get pod webserver -o jsonpath='{.spec.volumes[*].name}'
```

Expected:

```text
shared-logs
```

---

## Verify Nginx Log Directory

```bash
kubectl exec webserver -c nginx-container -- ls -la /var/log/nginx
```

---

## Verify Shared Directory from Sidecar

```bash
kubectl exec webserver -c sidecar-container -- ls -la /var/log/nginx
```

---

## Read Nginx Access Log

```bash
kubectl exec webserver -c nginx-container -- cat /var/log/nginx/access.log
```

---

## Read Nginx Error Log

```bash
kubectl exec webserver -c nginx-container -- cat /var/log/nginx/error.log
```

---

## View Sidecar Logs

```bash
kubectl logs webserver -c sidecar-container
```

Follow continuously:

```bash
kubectl logs -f webserver -c sidecar-container
```

Stop with:

```text
Ctrl+C
```

---

## Verify Container States

```bash
kubectl describe pod webserver
```

Look for:

```text
Init Containers:
  sidecar-container:
    State: Running
```

and:

```text
Containers:
  nginx-container:
    State: Running
```

---

## Check Events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

## Troubleshooting Commands

### Check Pod status

```bash
kubectl get pod webserver
```

### Inspect the Pod

```bash
kubectl describe pod webserver
```

### View sidecar logs

```bash
kubectl logs webserver -c sidecar-container
```

### View previous sidecar logs after a restart

```bash
kubectl logs webserver -c sidecar-container --previous
```

### Enter Nginx container

```bash
kubectl exec -it webserver -c nginx-container -- sh
```

### Enter sidecar container

```bash
kubectl exec -it webserver -c sidecar-container -- sh
```

---

## Optional Cleanup

```bash
kubectl delete pod webserver
```

Deleting the Pod also removes the `shared-logs` `emptyDir` volume.
