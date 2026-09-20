# DevOps Challenge Day 54 – Commands

## Create the Kubernetes Manifest

```bash
vi /tmp/volume-share-devops.yaml
```

Use the following configuration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-devops
spec:
  containers:
    - name: volume-container-devops-1
      image: fedora:latest
      command: ["sleep", "3600"]
      volumeMounts:
        - name: volume-share
          mountPath: /tmp/ecommerce

    - name: volume-container-devops-2
      image: fedora:latest
      command: ["sleep", "3600"]
      volumeMounts:
        - name: volume-share
          mountPath: /tmp/demo

  volumes:
    - name: volume-share
      emptyDir: {}
```

---

## Create the Pod

```bash
kubectl apply -f /tmp/volume-share-devops.yaml
```

---

## Check Pod Status

```bash
kubectl get pods
```

Or:

```bash
kubectl get pod volume-share-devops
```

Expected:

```text
READY   STATUS
2/2     Running
```

---

## Verify Container Names

```bash
kubectl get pod volume-share-devops   -o jsonpath='{.spec.containers[*].name}'
```

Expected:

```text
volume-container-devops-1 volume-container-devops-2
```

---

## Create the Required File in Container 1

```bash
kubectl exec volume-share-devops   -c volume-container-devops-1   -- sh -c 'echo "Welcome to xFusionCorp Industries" > /tmp/ecommerce/ecommerce.txt'
```

---

## Verify the File in Container 1

```bash
kubectl exec volume-share-devops   -c volume-container-devops-1   -- cat /tmp/ecommerce/ecommerce.txt
```

Expected:

```text
Welcome to xFusionCorp Industries
```

---

## List the Shared Directory in Container 2

```bash
kubectl exec volume-share-devops   -c volume-container-devops-2   -- ls -l /tmp/demo
```

Expected to contain:

```text
ecommerce.txt
```

---

## Verify the Shared File from Container 2

```bash
kubectl exec volume-share-devops   -c volume-container-devops-2   -- cat /tmp/demo/ecommerce.txt
```

Expected:

```text
Welcome to xFusionCorp Industries
```

---

## Inspect the Pod Configuration

```bash
kubectl describe pod volume-share-devops
```

Look for:

```text
Volumes:
  volume-share:
    Type: EmptyDir
```

Also verify that both containers mount `volume-share`.

---

## Display the Pod YAML from the Cluster

```bash
kubectl get pod volume-share-devops -o yaml
```

---

## Useful Troubleshooting Commands

### Show detailed Pod status

```bash
kubectl describe pod volume-share-devops
```

### Check Pod events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Inspect the first container

```bash
kubectl exec -it volume-share-devops   -c volume-container-devops-1 -- bash
```

If `bash` is unavailable:

```bash
kubectl exec -it volume-share-devops   -c volume-container-devops-1 -- sh
```

### Inspect the second container

```bash
kubectl exec -it volume-share-devops   -c volume-container-devops-2 -- bash
```

If `bash` is unavailable:

```bash
kubectl exec -it volume-share-devops   -c volume-container-devops-2 -- sh
```

### Check shared directory contents

```bash
kubectl exec volume-share-devops   -c volume-container-devops-1   -- ls -la /tmp/ecommerce

kubectl exec volume-share-devops   -c volume-container-devops-2   -- ls -la /tmp/demo
```

---

## Optional Cleanup

```bash
kubectl delete pod volume-share-devops
```

Deleting the Pod also removes the `emptyDir` volume and all data stored inside it.
