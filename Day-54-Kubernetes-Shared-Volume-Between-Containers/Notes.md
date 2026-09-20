# DevOps Challenge Day 54 – Notes

## 1. Kubernetes Volumes

Containers normally have their own isolated filesystems.

If two containers run inside the same Pod:

```text
Container A filesystem != Container B filesystem
```

A file created in one container is not automatically visible in another container.

Kubernetes volumes solve this problem by providing storage that containers can mount.

---

## 2. What Is `emptyDir`?

`emptyDir` is a Kubernetes volume type designed for temporary data.

Example:

```yaml
volumes:
  - name: volume-share
    emptyDir: {}
```

Kubernetes creates the volume when the Pod is assigned to a node.

The volume initially contains no files, which is why it is called `emptyDir`.

---

## 3. Lifetime of `emptyDir`

The lifetime of an `emptyDir` volume is tied to the **Pod**, not to an individual container.

```text
Pod created
   ↓
emptyDir created
   ↓
Containers use the volume
   ↓
Container restart
   ↓
emptyDir usually still exists
   ↓
Pod deleted
   ↓
emptyDir deleted
```

Therefore, it is useful for temporary shared data, but not for long-term persistence.

---

## 4. Shared Volume Between Containers

The same volume can be mounted by multiple containers inside one Pod.

Example:

```yaml
containers:
  - name: container-1
    volumeMounts:
      - name: shared-volume
        mountPath: /data/a

  - name: container-2
    volumeMounts:
      - name: shared-volume
        mountPath: /data/b

volumes:
  - name: shared-volume
    emptyDir: {}
```

The mount paths can be different.

```text
Container 1           Container 2

/data/a               /data/b
   │                      │
   └──────────┬───────────┘
              │
              ▼
        shared-volume
```

Both paths reference the same underlying storage.

---

## 5. Day 54 Volume Mapping

The challenge uses:

```text
Volume name:
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

Therefore:

```text
/tmp/ecommerce/ecommerce.txt
```

inside the first container and:

```text
/tmp/demo/ecommerce.txt
```

inside the second container represent the same underlying file.

---

## 6. `volumes` vs `volumeMounts`

These two Kubernetes fields perform different jobs.

### `volumes`

Defines the storage at Pod level.

```yaml
volumes:
  - name: volume-share
    emptyDir: {}
```

This means:

> Create a Pod-level volume named `volume-share` using `emptyDir`.

### `volumeMounts`

Attaches that volume to an individual container.

```yaml
volumeMounts:
  - name: volume-share
    mountPath: /tmp/ecommerce
```

This means:

> Mount the volume named `volume-share` inside this container at `/tmp/ecommerce`.

The names must match:

```text
volumeMounts.name
        │
        ▼
   volume-share
        │
        ▼
volumes.name
```

If the names do not match, the Pod configuration is invalid.

---

## 7. Why Different Mount Paths Still Share Data

A common misconception is that containers must mount a volume at the same path.

They do not.

For example:

```text
Container 1:
    /tmp/ecommerce

Container 2:
    /tmp/demo
```

Both can still point to:

```text
volume-share
```

The mount path is only where the volume appears inside each container.

The actual storage object is the same.

---

## 8. Why the `sleep` Command Is Used

The Fedora containers need to remain running.

A container exits when its main process exits.

Using:

```yaml
command: ["sleep", "3600"]
```

starts a long-running process so that the container remains in the `Running` state.

Without a long-running foreground process, the container could terminate before `kubectl exec` is used.

---

## 9. Multi-Container Pod Concept

A Pod can contain one or more containers.

Containers within the same Pod:

- Run on the same Kubernetes node.
- Share the same Pod network namespace.
- Can communicate through `localhost`.
- Can mount shared Pod-level volumes.
- Have separate container filesystems unless shared storage is explicitly mounted.

Example:

```text
Pod
├── Application container
├── Sidecar container
└── Shared volume
```

This pattern is commonly used for:

- Sidecar logging
- File processing
- Proxy containers
- Configuration generation
- Temporary data exchange

---

## 10. `kubectl exec`

`kubectl exec` executes a command inside a running container.

General format:

```bash
kubectl exec <pod-name> -c <container-name> -- <command>
```

Example:

```bash
kubectl exec volume-share-devops   -c volume-container-devops-1   -- ls /tmp/ecommerce
```

When a Pod contains more than one container, specifying the container using `-c` is important.

---

## 11. Writing a File Through the Shared Volume

The command used was:

```bash
kubectl exec volume-share-devops   -c volume-container-devops-1   -- sh -c 'echo "Welcome to xFusionCorp Industries" > /tmp/ecommerce/ecommerce.txt'
```

Why `sh -c`?

Because shell operators such as:

```text
>
```

are interpreted by a shell.

The command:

```text
echo "..." > file
```

therefore needs to execute through:

```text
sh -c
```

inside the container.

---

## 12. Why `emptyDir` Is Not Persistent Storage

`emptyDir` is temporary Pod storage.

It should not be used when data must survive Pod deletion or recreation.

For persistent application data, Kubernetes commonly uses:

```text
PersistentVolume (PV)
        +
PersistentVolumeClaim (PVC)
```

Comparison:

| Feature | emptyDir | Persistent Volume |
|---|---|---|
| Scope | Pod | Independent storage resource |
| Temporary | Yes | Usually No |
| Shared within Pod | Yes | Yes |
| Survives Pod deletion | No | Usually Yes |
| Good for temporary files | Yes | Possible, but unnecessary |
| Good for databases | No | Yes |

---

## 13. Container Restart vs Pod Deletion

This distinction is important.

### Container restart

If a container crashes and Kubernetes restarts it while the same Pod still exists:

```text
emptyDir remains available
```

### Pod deletion

If the Pod itself is deleted:

```text
emptyDir is destroyed
```

A newly created replacement Pod receives a new empty volume.

---

## 14. Common Mistakes

### Mistake 1: Creating two different volumes

Wrong:

```yaml
volumes:
  - name: volume-one
    emptyDir: {}

  - name: volume-two
    emptyDir: {}
```

If each container uses a separate volume, data will not be shared.

---

### Mistake 2: Mismatched volume names

Wrong:

```yaml
volumeMounts:
  - name: volume-share
```

but:

```yaml
volumes:
  - name: shared-data
```

The names must match exactly.

---

### Mistake 3: Using different file contents

The task requires:

```text
Welcome to xFusionCorp Industries
```

Exact values matter in automated DevOps challenge validation.

---

### Mistake 4: Forgetting the image tag

The required image is:

```text
fedora:latest
```

Even though Kubernetes may default to `latest` in some cases, explicitly specifying the requested tag is safer for automated validation.

---

### Mistake 5: Not keeping the containers alive

If the main process exits, the container terminates.

A long-running command such as:

```text
sleep 3600
```

keeps the container running long enough to perform the task.

---

## 15. Verification Strategy

A good verification sequence is:

```text
1. Verify Pod exists
        ↓
2. Verify READY = 2/2
        ↓
3. Verify both container names
        ↓
4. Create the file from container 1
        ↓
5. Read the file from container 1
        ↓
6. Read the same file from container 2
        ↓
7. Inspect volume configuration
```

Useful commands:

```bash
kubectl get pods
kubectl describe pod volume-share-devops
kubectl get pod volume-share-devops -o yaml
```

---

## 16. Real-World Example

Suppose a Pod contains:

```text
Application container
        +
Log-processing sidecar
```

The application writes logs to:

```text
/app/logs
```

The sidecar reads the same shared volume from:

```text
/logs
```

Architecture:

```text
Application
    │
    ▼
/app/logs
    │
    └────────┐
             ▼
        shared emptyDir
             ▲
    ┌────────┘
    │
  /logs
    ▲
    │
Sidecar
```

The application and sidecar can therefore exchange files without networking or external storage.

---

## 17. Core Takeaway

The main concept from Day 54 is:

> A Kubernetes `emptyDir` volume can be mounted into multiple containers within the same Pod, allowing them to share temporary filesystem data even when each container uses a different mount path.

For this challenge:

```text
/tmp/ecommerce
       │
       ▼
 volume-share
       ▲
       │
  /tmp/demo
```

The successful appearance of `ecommerce.txt` in the second container proves that the shared volume was configured correctly.
