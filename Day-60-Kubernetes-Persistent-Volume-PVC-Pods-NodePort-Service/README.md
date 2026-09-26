# DevOps Challenge — Day 60
## Kubernetes Persistent Volume, PVC, Pod, and NodePort Service

### Overview
In this challenge, a Kubernetes template was created to deploy an Apache web application with persistent storage. The solution required provisioning a PersistentVolume (PV), binding it through a PersistentVolumeClaim (PVC), mounting the claim into the Apache container's document root, and exposing the application using a NodePort Service.

### Task Requirements
The following Kubernetes resources were required:

- **PersistentVolume:** `pv-datacenter`
  - Storage class: `manual`
  - Capacity: `4Gi`
  - Access mode: `ReadWriteOnce`
  - Volume type: `hostPath`
  - Host path: `/mnt/security`

- **PersistentVolumeClaim:** `pvc-datacenter`
  - Storage class: `manual`
  - Requested storage: `2Gi`
  - Access mode: `ReadWriteOnce`

- **Pod:** `pod-datacenter`
  - Container name: `container-datacenter`
  - Image: `httpd:latest`
  - PVC mounted at Apache document root:
    `/usr/local/apache2/htdocs`

- **Service:** `web-datacenter`
  - Type: `NodePort`
  - NodePort: `30008`
  - Exposes Apache running on container port `80`

### Architecture

```text
/mnt/security
      |
      v
PersistentVolume
pv-datacenter
      |
      v
PersistentVolumeClaim
pvc-datacenter
      |
      v
Pod: pod-datacenter
      |
      v
/usr/local/apache2/htdocs
      |
      v
Apache HTTP Server :80
      |
      v
NodePort Service
web-datacenter
      |
      v
NodePort 30008
```

### Kubernetes Manifest

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-datacenter
spec:
  storageClassName: manual
  capacity:
    storage: 4Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/security

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-datacenter
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi

---
apiVersion: v1
kind: Pod
metadata:
  name: pod-datacenter
  labels:
    app: datacenter
spec:
  containers:
    - name: container-datacenter
      image: httpd:latest
      ports:
        - containerPort: 80
      volumeMounts:
        - name: datacenter-storage
          mountPath: /usr/local/apache2/htdocs
  volumes:
    - name: datacenter-storage
      persistentVolumeClaim:
        claimName: pvc-datacenter

---
apiVersion: v1
kind: Service
metadata:
  name: web-datacenter
spec:
  type: NodePort
  selector:
    app: datacenter
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30008
```

### Implementation Summary
1. Created the Kubernetes YAML manifest containing the PV, PVC, Pod, and Service definitions.
2. Applied the manifest using `kubectl apply`.
3. Verified that `pv-datacenter` and `pvc-datacenter` were successfully bound.
4. Confirmed that `pod-datacenter` reached the `Running` state.
5. Verified that the PVC was mounted at `/usr/local/apache2/htdocs`.
6. Confirmed that `web-datacenter` exposed Apache through NodePort `30008`.
7. Verified that the Service endpoint correctly pointed to the Apache Pod.
8. Tested the Service through its ClusterIP and received an Apache response.

### Verification
The completed deployment was verified using:

```bash
kubectl get pv
kubectl get pvc
kubectl get pods
kubectl get svc web-datacenter
kubectl get endpoints web-datacenter
kubectl get pod pod-datacenter -o wide
kubectl exec pod-datacenter -- df -h
```

The Service endpoint resolved successfully to the Pod on port `80`, and an HTTP request to the Service returned an Apache directory index response, confirming end-to-end connectivity.

### Key Learning Outcomes
- Difference between PersistentVolumes and PersistentVolumeClaims
- Static Kubernetes storage provisioning
- PV/PVC binding requirements
- `ReadWriteOnce` access mode
- `hostPath` volume behavior
- Mounting persistent storage into a Pod
- Kubernetes labels and Service selectors
- NodePort Service configuration
- Service-to-Pod endpoint verification
- Testing Kubernetes application connectivity

### Status
**Day 60 completed successfully.**
