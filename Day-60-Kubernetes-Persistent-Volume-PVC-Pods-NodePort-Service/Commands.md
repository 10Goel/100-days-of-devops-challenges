# DevOps Challenge — Day 60
## Commands Reference

### 1. Create the Kubernetes Manifest

```bash
vi /tmp/day60.yaml
```

Insert the required PV, PVC, Pod, and NodePort Service definitions and save the file.

---

### 2. Apply the Manifest

```bash
kubectl apply -f /tmp/day60.yaml
```

Expected resources:

```text
persistentvolume/pv-datacenter
persistentvolumeclaim/pvc-datacenter
pod/pod-datacenter
service/web-datacenter
```

---

### 3. Verify the PersistentVolume

```bash
kubectl get pv
```

Detailed inspection:

```bash
kubectl describe pv pv-datacenter
```

Confirm:
- Capacity: `4Gi`
- Access mode: `RWO`
- Storage class: `manual`
- Host path: `/mnt/security`
- Status: `Bound`

---

### 4. Verify the PersistentVolumeClaim

```bash
kubectl get pvc
```

Detailed inspection:

```bash
kubectl describe pvc pvc-datacenter
```

Confirm:
- Requested storage: `2Gi`
- Access mode: `RWO`
- Storage class: `manual`
- Bound volume: `pv-datacenter`

---

### 5. Verify the Pod

```bash
kubectl get pods
```

Detailed inspection:

```bash
kubectl describe pod pod-datacenter
```

Wide output:

```bash
kubectl get pod pod-datacenter -o wide
```

Confirm:
- Pod status: `Running`
- Container name: `container-datacenter`
- Image: `httpd:latest`

---

### 6. Verify the Mounted Persistent Storage

```bash
kubectl exec pod-datacenter -- df -h
```

Optional mount verification:

```bash
kubectl exec pod-datacenter -- mount | grep htdocs
```

List the Apache document-root contents:

```bash
kubectl exec pod-datacenter -- ls -la /usr/local/apache2/htdocs
```

---

### 7. Verify the NodePort Service

```bash
kubectl get svc web-datacenter
```

Detailed inspection:

```bash
kubectl describe svc web-datacenter
```

Confirm:
- Type: `NodePort`
- Service port: `80`
- Target port: `80`
- NodePort: `30008`

---

### 8. Verify the Service Endpoint

```bash
kubectl get endpoints web-datacenter
```

The endpoint should resolve to the IP address of `pod-datacenter` on port `80`.

For newer Kubernetes versions, EndpointSlice can also be checked:

```bash
kubectl get endpointslice
```

---

### 9. Test the Service Through ClusterIP

```bash
curl $(kubectl get svc web-datacenter -o jsonpath='{.spec.clusterIP}')
```

A valid Apache response confirms that traffic reaches the Pod.

---

### 10. Combined Verification

```bash
kubectl get pv,pvc,pod,svc
```

---

### Useful Troubleshooting Commands

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

```bash
kubectl logs pod-datacenter
```

```bash
kubectl get pod pod-datacenter --show-labels
```

```bash
kubectl get svc web-datacenter -o yaml
```

```bash
kubectl get pvc pvc-datacenter -o yaml
```

```bash
kubectl get pv pv-datacenter -o yaml
```

### Command History Used During the Challenge

```bash
vi /tmp/day60.yaml
kubectl apply -f /tmp/day60.yaml
kubectl get pv
kubectl describe pv pv-datacenter
kubectl get pvc
kubectl get pods
kubectl describe pod pod-datacenter
kubectl exec pod-datacenter -- df -h
kubectl get svc web-datacenter
kubectl describe svc web-datacenter
kubectl get endpoints web-datacenter
kubectl get pod pod-datacenter -o wide
curl $(kubectl get svc web-datacenter -o jsonpath='{.spec.clusterIP}')
```
