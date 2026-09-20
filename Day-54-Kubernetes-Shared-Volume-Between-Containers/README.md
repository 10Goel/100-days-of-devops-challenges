# DevOps Challenge Day 54 – Kubernetes Shared Volume Between Containers

## Overview

This challenge demonstrates how multiple containers running inside the same Kubernetes Pod can share temporary data using an `emptyDir` volume.

The Pod contains two Fedora containers. Both containers mount the same Kubernetes volume, but at different filesystem paths. A file created from the first container becomes immediately accessible from the second container because both mount points reference the same underlying `emptyDir` volume.

---

## Task Requirements

Create a Kubernetes Pod with the following configuration:

- **Pod name:** `volume-share-devops`
- **Shared volume name:** `volume-share`
- **Volume type:** `emptyDir`

### Container 1

- **Name:** `volume-container-devops-1`
- **Image:** `fedora:latest`
- **Command:** `sleep`
- **Mount path:** `/tmp/ecommerce`

### Container 2

- **Name:** `volume-container-devops-2`
- **Image:** `fedora:latest`
- **Command:** `sleep`
- **Mount path:** `/tmp/demo`

After the Pod is running:

1. Create `/tmp/ecommerce/ecommerce.txt` inside the first container.
2. Add the following content:

```text
Welcome to xFusionCorp Industries
```

3. Verify that the same file is available in the second container at:

```text
/tmp/demo/ecommerce.txt
```

---

## Architecture

```text
Pod: volume-share-devops
│
├── Container: volume-container-devops-1
│   ├── Image: fedora:latest
│   └── /tmp/ecommerce
│          │
│          └────┐
│               │
├── Container: volume-container-devops-2
│   ├── Image: fedora:latest
│   └── /tmp/demo
│          │
│          └────┤
│               │
└── Volume: volume-share
    └── Type: emptyDir
```

Both containers use different mount paths, but the mounted storage is the same.

---

## Kubernetes Manifest

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

## Implementation

Create the manifest:

```bash
vi /tmp/volume-share-devops.yaml
```

Apply it:

```bash
kubectl apply -f /tmp/volume-share-devops.yaml
```

Verify that both containers are running:

```bash
kubectl get pods
```

Expected state:

```text
NAME                  READY   STATUS    RESTARTS
volume-share-devops   2/2     Running   0
```

Create the required file inside the first container:

```bash
kubectl exec volume-share-devops   -c volume-container-devops-1   -- sh -c 'echo "Welcome to xFusionCorp Industries" > /tmp/ecommerce/ecommerce.txt'
```

Verify it from the first container:

```bash
kubectl exec volume-share-devops   -c volume-container-devops-1   -- cat /tmp/ecommerce/ecommerce.txt
```

Verify the same file from the second container:

```bash
kubectl exec volume-share-devops   -c volume-container-devops-2   -- cat /tmp/demo/ecommerce.txt
```

Expected output:

```text
Welcome to xFusionCorp Industries
```

---

## Why the File Is Shared

Both containers mount the Kubernetes volume named:

```text
volume-share
```

The first container mounts it at:

```text
/tmp/ecommerce
```

The second container mounts it at:

```text
/tmp/demo
```

These paths look different inside the containers, but both represent the same underlying storage.

Therefore:

```text
Container 1:
    /tmp/ecommerce/ecommerce.txt

             │
             ▼

       volume-share

             ▲
             │

Container 2:
    /tmp/demo/ecommerce.txt
```

A file written through one mount is visible through the other.

---

## Key Concepts Practiced

- Kubernetes Pods
- Multi-container Pods
- Kubernetes volumes
- `emptyDir`
- `volumeMounts`
- Shared storage between containers
- Container filesystem isolation
- `kubectl exec`
- Pod verification and troubleshooting

---

## Important Learning

An `emptyDir` volume:

- Is created when the Pod is assigned to a node.
- Exists for the lifetime of that Pod.
- Can be mounted by multiple containers within the same Pod.
- Allows containers to exchange temporary data.
- Is deleted when the Pod itself is permanently removed.
- Is not intended for persistent application data.

Container restarts do not normally remove the `emptyDir` data while the Pod continues to exist, but deleting or recreating the Pod causes the temporary volume to be lost.

---

## Verification Commands

```bash
kubectl get pod volume-share-devops

kubectl get pod volume-share-devops   -o jsonpath='{.spec.containers[*].name}'

kubectl exec volume-share-devops   -c volume-container-devops-1   -- cat /tmp/ecommerce/ecommerce.txt

kubectl exec volume-share-devops   -c volume-container-devops-2   -- cat /tmp/demo/ecommerce.txt

kubectl describe pod volume-share-devops
```

---

## Result

The challenge was completed successfully.

The Pod `volume-share-devops` was created with two Fedora containers sharing the same `emptyDir` volume. The file created under `/tmp/ecommerce` in the first container was successfully accessed under `/tmp/demo` in the second container, confirming that both containers were using the same shared Kubernetes volume.
