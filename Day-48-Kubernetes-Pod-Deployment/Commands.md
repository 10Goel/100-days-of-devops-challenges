# DevOps Day 48 — Kubernetes Commands

This file contains the commands used for Day 48 along with a practical Kubernetes command reference.

---

# 1. Cluster Verification

Check cluster nodes:

```bash
kubectl get nodes
```

Display cluster information:

```bash
kubectl cluster-info
```

---

# 2. Create the Manifest

```bash
vi pod-nginx.yaml
```

Manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx
  labels:
    app: nginx_app
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
```

---

# 3. Validate the Manifest

```bash
kubectl apply --dry-run=client -f pod-nginx.yaml
```

---

# 4. Create the Pod

```bash
kubectl apply -f pod-nginx.yaml
```

---

# 5. Verify Pod Status

```bash
kubectl get pods
```

Detailed view:

```bash
kubectl get pods -o wide
```

Watch changes continuously:

```bash
kubectl get pods -w
```

---

# 6. Verify Labels

```bash
kubectl get pod pod-nginx --show-labels
```

Filter using a label:

```bash
kubectl get pods -l app=nginx_app
```

---

# 7. Verify Container Name

```bash
kubectl get pod pod-nginx   -o jsonpath='{.spec.containers[0].name}{"\n"}'
```

---

# 8. Verify Image

```bash
kubectl get pod pod-nginx   -o jsonpath='{.spec.containers[0].image}{"\n"}'
```

---

# 9. Inspect the Pod

```bash
kubectl describe pod pod-nginx
```

Display stored YAML:

```bash
kubectl get pod pod-nginx -o yaml
```

Display JSON:

```bash
kubectl get pod pod-nginx -o json
```

---

# Pod Management Commands

List Pods:

```bash
kubectl get pods
```

List Pods in every namespace:

```bash
kubectl get pods -A
```

Describe:

```bash
kubectl describe pod <pod-name>
```

Delete:

```bash
kubectl delete pod <pod-name>
```

Delete using manifest:

```bash
kubectl delete -f pod-nginx.yaml
```

---

# Logs

View logs:

```bash
kubectl logs <pod-name>
```

Specific container:

```bash
kubectl logs <pod-name> -c <container-name>
```

Follow logs:

```bash
kubectl logs -f <pod-name>
```

Logs from the previous crashed container:

```bash
kubectl logs <pod-name> --previous
```

---

# Execute Commands in Containers

Run a command:

```bash
kubectl exec <pod-name> -- <command>
```

Example:

```bash
kubectl exec pod-nginx -- nginx -v
```

Open an interactive shell:

```bash
kubectl exec -it pod-nginx -- /bin/sh
```

Specific container in a multi-container Pod:

```bash
kubectl exec -it <pod-name> -c <container-name> -- /bin/sh
```

---

# Namespaces

List namespaces:

```bash
kubectl get namespaces
```

Short form:

```bash
kubectl get ns
```

Create namespace:

```bash
kubectl create namespace dev
```

List Pods in namespace:

```bash
kubectl get pods -n dev
```

---

# Deployments

List:

```bash
kubectl get deployments
```

Create:

```bash
kubectl create deployment nginx --image=nginx:latest
```

Scale:

```bash
kubectl scale deployment nginx --replicas=3
```

Rollout status:

```bash
kubectl rollout status deployment/nginx
```

History:

```bash
kubectl rollout history deployment/nginx
```

Rollback:

```bash
kubectl rollout undo deployment/nginx
```

---

# Services

List:

```bash
kubectl get services
```

Short form:

```bash
kubectl get svc
```

Expose a Deployment:

```bash
kubectl expose deployment nginx   --port=80   --target-port=80   --type=ClusterIP
```

---

# ConfigMaps

Create from literal values:

```bash
kubectl create configmap app-config   --from-literal=ENV=production
```

List:

```bash
kubectl get configmaps
```

---

# Secrets

Create:

```bash
kubectl create secret generic app-secret   --from-literal=username=admin   --from-literal=password='example'
```

List:

```bash
kubectl get secrets
```

> Avoid storing real production credentials directly in shell history or Git.

---

# Nodes

List:

```bash
kubectl get nodes
```

Describe:

```bash
kubectl describe node <node-name>
```

Metrics, when Metrics Server is available:

```bash
kubectl top nodes
```

```bash
kubectl top pods
```

---

# Events

Display events:

```bash
kubectl get events
```

Sort chronologically:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Events are especially useful for diagnosing:

```text
ImagePullBackOff
CrashLoopBackOff
Pending
FailedScheduling
Mount failures
Probe failures
```

---

# Output Formats

Wide:

```bash
kubectl get pods -o wide
```

YAML:

```bash
kubectl get pod pod-nginx -o yaml
```

JSON:

```bash
kubectl get pod pod-nginx -o json
```

JSONPath:

```bash
kubectl get pod pod-nginx   -o jsonpath='{.metadata.name}{"\n"}'
```

Custom columns:

```bash
kubectl get pods   -o custom-columns='NAME:.metadata.name,IMAGE:.spec.containers[*].image'
```

---

# Common Short Resource Names

| Resource | Short Name |
|---|---|
| Pods | `po` |
| Services | `svc` |
| Deployments | `deploy` |
| ReplicaSets | `rs` |
| Namespaces | `ns` |
| ConfigMaps | `cm` |
| PersistentVolumeClaims | `pvc` |
| PersistentVolumes | `pv` |
| ServiceAccounts | `sa` |

Examples:

```bash
kubectl get po
kubectl get svc
kubectl get deploy
kubectl get ns
```

---

# Troubleshooting Workflow

When a Pod is failing:

```bash
kubectl get pod <pod-name>
```

```bash
kubectl describe pod <pod-name>
```

```bash
kubectl logs <pod-name>
```

```bash
kubectl logs <pod-name> --previous
```

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

```bash
kubectl get pod <pod-name> -o yaml
```

If needed:

```bash
kubectl exec -it <pod-name> -- /bin/sh
```

Useful troubleshooting order:

```text
STATUS
  ↓
EVENTS
  ↓
LOGS
  ↓
CONFIGURATION
  ↓
NETWORK
  ↓
STORAGE
  ↓
SCHEDULING / NODE
```

---

# Day 48 Core Commands

```bash
kubectl get nodes

vi pod-nginx.yaml

kubectl apply --dry-run=client -f pod-nginx.yaml

kubectl apply -f pod-nginx.yaml

kubectl get pods

kubectl get pod pod-nginx --show-labels

kubectl get pod pod-nginx   -o jsonpath='{.spec.containers[0].name}{"\n"}'

kubectl get pod pod-nginx   -o jsonpath='{.spec.containers[0].image}{"\n"}'

kubectl describe pod pod-nginx
```
