# DevOps Challenge — Day 60
## Kubernetes Persistent Storage Notes

### 1. Why Persistent Storage Is Needed
Container filesystems are normally ephemeral. If a container is recreated, files written only inside its writable container layer can be lost.

Kubernetes Volumes allow data to exist independently from the container filesystem. PersistentVolumes and PersistentVolumeClaims add a storage abstraction that separates application workloads from the underlying storage implementation.

---

## 2. PersistentVolume

A **PersistentVolume (PV)** is a storage resource available to the Kubernetes cluster.

In this challenge:

```yaml
metadata:
  name: pv-datacenter
```

The PV provided:

```yaml
capacity:
  storage: 4Gi
```

with:

```yaml
accessModes:
  - ReadWriteOnce
```

and:

```yaml
storageClassName: manual
```

The physical backing used:

```yaml
hostPath:
  path: /mnt/security
```

### Important
`hostPath` maps a path from the Kubernetes node directly into the Pod through the volume.

It is useful for lab environments and node-local workloads, but it is not generally appropriate for highly available production storage because the data is tied to a particular node.

---

## 3. PersistentVolumeClaim

A **PersistentVolumeClaim (PVC)** is a request for storage made by a workload.

The PVC in this challenge requested:

```yaml
resources:
  requests:
    storage: 2Gi
```

with:

```yaml
storageClassName: manual
```

and:

```yaml
accessModes:
  - ReadWriteOnce
```

The PVC did not need to request exactly the same storage size as the PV.

The requirement is effectively:

```text
PV capacity >= PVC requested capacity
```

Here:

```text
PV  = 4Gi
PVC = 2Gi
```

Therefore the PV could satisfy the claim.

---

## 4. PV and PVC Binding

For the PVC to bind to the PV, Kubernetes checks compatibility such as:

- Storage class
- Access mode
- Requested capacity
- Volume availability

In this challenge:

```text
pv-datacenter
     |
     | storageClass: manual
     | accessMode: RWO
     | capacity: 4Gi
     v
pvc-datacenter
       storageClass: manual
       accessMode: RWO
       request: 2Gi
```

The compatible resources were bound successfully.

---

## 5. ReadWriteOnce

`ReadWriteOnce` is abbreviated as:

```text
RWO
```

It means the volume can be mounted read-write by Pods on a single node.

It does **not** simply mean that only one Pod can ever use the volume. Multiple Pods on the same node can sometimes use an RWO volume depending on the storage implementation.

---

## 6. Pod Volume and volumeMount

The Pod references the PVC:

```yaml
volumes:
  - name: datacenter-storage
    persistentVolumeClaim:
      claimName: pvc-datacenter
```

The container then mounts that Pod volume:

```yaml
volumeMounts:
  - name: datacenter-storage
    mountPath: /usr/local/apache2/htdocs
```

The relationship is:

```text
PV
 |
 v
PVC
 |
 v
Pod Volume
 |
 v
Container volumeMount
 |
 v
/usr/local/apache2/htdocs
```

---

## 7. Apache Document Root

The official Apache HTTP Server container uses:

```text
/usr/local/apache2/htdocs
```

as its default document root.

Mounting the PVC there means the web server serves files stored on the persistent volume instead of relying only on files inside the container image.

---

## 8. Why the Default Apache Page Can Disappear

The `httpd:latest` image normally contains files in:

```text
/usr/local/apache2/htdocs
```

Once another filesystem is mounted on that same path, the original files from the container image are hidden while the mount is active.

Conceptually:

```text
Container image:
  /usr/local/apache2/htdocs/index.html

After volume mount:
  /usr/local/apache2/htdocs
          ^
          |
        PVC mount
```

This is standard Linux mount behavior and is important when mounting storage over an application directory.

---

## 9. NodePort Service

A `NodePort` Service exposes an application through a port opened on Kubernetes nodes.

The service used:

```yaml
type: NodePort
```

with:

```yaml
port: 80
targetPort: 80
nodePort: 30008
```

Meaning:

```text
Client
  |
  v
NodeIP:30008
  |
  v
Service:80
  |
  v
Pod:80
  |
  v
Apache
```

### Port Meanings

**`port`**
The port exposed by the Kubernetes Service.

**`targetPort`**
The destination port on the selected Pod.

**`nodePort`**
The port exposed on the Kubernetes node.

---

## 10. Labels and Selectors

The Pod was assigned:

```yaml
labels:
  app: datacenter
```

The Service selected:

```yaml
selector:
  app: datacenter
```

Therefore Kubernetes associated the Service with the Pod.

If the selector and labels do not match, the Service can exist but have no backend endpoints.

---

## 11. Endpoint Verification

The command:

```bash
kubectl get endpoints web-datacenter
```

confirmed that the Service had a backend endpoint:

```text
10.22.0.9:80
```

This verified that:

1. The Service selector matched the Pod.
2. Kubernetes discovered the Pod.
3. Traffic sent through the Service could be forwarded to port `80` of the Pod.

### Deprecation Note
Recent Kubernetes versions warn that the legacy `Endpoints` API is deprecated in favor of `EndpointSlice`.

A modern check is:

```bash
kubectl get endpointslice
```

The warning does not mean the Service is broken.

---

## 12. ClusterIP Testing

The service was tested using:

```bash
curl $(kubectl get svc web-datacenter -o jsonpath='{.spec.clusterIP}')
```

An Apache directory index response was returned.

This confirmed:

```text
Service ClusterIP
      |
      v
Service selector
      |
      v
Pod endpoint
      |
      v
Apache container
```

So the application path was functioning correctly.

---

## 13. Static Provisioning

This challenge is an example of **static provisioning**.

The administrator manually creates a PV first:

```text
Administrator -> PersistentVolume
```

and the workload later requests storage through:

```text
Application -> PersistentVolumeClaim
```

Kubernetes then binds a compatible PV.

With dynamic provisioning, a StorageClass and provisioner can automatically create storage when a PVC is submitted.

---

## 14. Troubleshooting Checklist

### PVC remains Pending
Check:

```bash
kubectl describe pvc pvc-datacenter
kubectl get pv
```

Possible causes:
- Storage classes do not match
- PV capacity is smaller than requested storage
- Access modes do not match
- PV is already bound

### Pod remains Pending
Check:

```bash
kubectl describe pod pod-datacenter
```

Possible storage-related causes:
- PVC is still Pending
- Volume cannot be mounted
- Node-specific `hostPath` problems

### Service has no endpoints
Check:

```bash
kubectl get pod pod-datacenter --show-labels
kubectl get svc web-datacenter -o yaml
```

Compare the Service selector with Pod labels.

### HTTP request fails
Check:

```bash
kubectl get endpoints web-datacenter
kubectl logs pod-datacenter
kubectl get pod pod-datacenter -o wide
```

Verify that Apache is running and the Service points to the correct Pod.

---

## 15. Final Conceptual Model

```text
Storage Layer
==============================

Host Node
/mnt/security
      |
      v
PV: pv-datacenter
4Gi / RWO / manual
      |
      v
PVC: pvc-datacenter
2Gi request / RWO / manual


Application Layer
==============================

Pod: pod-datacenter
      |
      v
container-datacenter
httpd:latest
      |
      v
/usr/local/apache2/htdocs


Networking Layer
==============================

web-datacenter
NodePort Service
      |
      +-- port: 80
      +-- targetPort: 80
      +-- nodePort: 30008
```

## Key Takeaway
A Kubernetes workload normally does not consume a PersistentVolume directly. The Pod references a PersistentVolumeClaim, Kubernetes binds that claim to a compatible PersistentVolume, and the Pod mounts the resulting volume into the container.

This separation allows storage requirements to be defined independently from the application's container configuration.
