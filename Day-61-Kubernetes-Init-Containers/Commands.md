# DevOps Day 61 — Commands

## Create the Deployment Manifest

```bash
vi /tmp/ic-deploy-nautilus.yaml
```

Paste:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy-nautilus

spec:
  replicas: 1

  selector:
    matchLabels:
      app: ic-nautilus

  template:
    metadata:
      labels:
        app: ic-nautilus

    spec:
      initContainers:
        - name: ic-msg-nautilus
          image: fedora:latest
          command:
            - /bin/bash
            - -c
            - echo "Init Done - Welcome to xFusionCorp Industries" > /ic/ecommerce
          volumeMounts:
            - name: ic-volume-nautilus
              mountPath: /ic

      containers:
        - name: ic-main-nautilus
          image: fedora:latest
          command:
            - /bin/bash
            - -c
            - while true; do cat /ic/ecommerce; sleep 5; done
          volumeMounts:
            - name: ic-volume-nautilus
              mountPath: /ic

      volumes:
        - name: ic-volume-nautilus
          emptyDir: {}
```

---

## Validate the Manifest

```bash
kubectl apply -f /tmp/ic-deploy-nautilus.yaml --dry-run=client
```

---

## Create the Deployment

```bash
kubectl apply -f /tmp/ic-deploy-nautilus.yaml
```

---

## Check Deployments

```bash
kubectl get deployments
```

Specific Deployment:

```bash
kubectl get deployment ic-deploy-nautilus
```

Detailed information:

```bash
kubectl describe deployment ic-deploy-nautilus
```

---

## Check Pods

```bash
kubectl get pods
```

Using the application label:

```bash
kubectl get pods -l app=ic-nautilus
```

Watch pod startup:

```bash
kubectl get pods -l app=ic-nautilus -w
```

---

## Store the Pod Name

```bash
POD=$(kubectl get pods -l app=ic-nautilus -o jsonpath='{.items[0].metadata.name}')
```

Verify:

```bash
echo "$POD"
```

---

## Inspect the Pod

```bash
kubectl describe pod "$POD"
```

Look under:

```text
Init Containers:
  ic-msg-nautilus:
```

Successful initialization should show:

```text
Reason: Completed
Exit Code: 0
```

---

## Check Main Container Logs

```bash
kubectl logs "$POD" -c ic-main-nautilus
```

Expected output:

```text
Init Done - Welcome to xFusionCorp Industries
```

Because the main process runs continuously, the message should appear repeatedly.

---

## Check Init Container Logs

The command itself does not print the message to stdout because output is redirected into a file, but its logs can still be inspected:

```bash
kubectl logs "$POD" -c ic-msg-nautilus
```

---

## Check Init Container Status Using JSONPath

```bash
kubectl get pod "$POD" \
  -o jsonpath='{.status.initContainerStatuses[0].state.terminated.reason}'
```

Expected:

```text
Completed
```

Check the exit code:

```bash
kubectl get pod "$POD" \
  -o jsonpath='{.status.initContainerStatuses[0].state.terminated.exitCode}'
```

Expected:

```text
0
```

---

## Verify the Shared File From the Main Container

```bash
kubectl exec "$POD" -c ic-main-nautilus -- cat /ic/ecommerce
```

Expected:

```text
Init Done - Welcome to xFusionCorp Industries
```

---

## Inspect the Volume Configuration

```bash
kubectl get pod "$POD" -o yaml
```

Search for:

```yaml
volumes:
  - name: ic-volume-nautilus
    emptyDir: {}
```

And:

```yaml
volumeMounts:
  - name: ic-volume-nautilus
    mountPath: /ic
```

---

## Useful Troubleshooting Commands

Check pod status:

```bash
kubectl get pod "$POD" -o wide
```

Describe failures and events:

```bash
kubectl describe pod "$POD"
```

Check cluster events:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Check the Init Container logs:

```bash
kubectl logs "$POD" -c ic-msg-nautilus
```

Check the main container logs:

```bash
kubectl logs "$POD" -c ic-main-nautilus
```

Inspect the Deployment configuration:

```bash
kubectl get deployment ic-deploy-nautilus -o yaml
```

---

## Delete the Deployment

Use only when cleanup is required:

```bash
kubectl delete deployment ic-deploy-nautilus
```
