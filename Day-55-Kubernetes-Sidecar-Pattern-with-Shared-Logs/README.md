# DevOps Challenge Day 55 – Kubernetes Sidecar Pattern with Shared Logs

## Overview

This challenge demonstrates the **Kubernetes Sidecar Pattern** using a shared `emptyDir` volume.

The Pod contains:

- An **Nginx application container** that serves web traffic and writes access/error logs.
- A **sidecar container** that continuously reads those logs.
- A shared `emptyDir` volume mounted at `/var/log/nginx` in both containers.

The sidecar is implemented as a **restartable init container** using `restartPolicy: Always`, which allows it to remain running alongside the main Nginx container.

---

## Task Requirements

Create a Pod with the following configuration:

### Pod

- **Name:** `webserver`

### Shared Volume

- **Name:** `shared-logs`
- **Type:** `emptyDir`

### Main Container

- **Name:** `nginx-container`
- **Image:** `nginx:latest`
- **Mount path:** `/var/log/nginx`

### Sidecar Container

- **Name:** `sidecar-container`
- **Image:** `ubuntu:latest`
- **Defined under:** `initContainers`
- **Restart policy:** `Always`
- **Mount path:** `/var/log/nginx`

The sidecar runs:

```bash
sh -c 'while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done'
```

---

## Architecture

```text
Pod: webserver
│
├── nginx-container
│   ├── Image: nginx:latest
│   ├── Serves web traffic
│   └── Writes logs to:
│       /var/log/nginx
│            │
│            ▼
│       shared-logs
│        (emptyDir)
│            ▲
│            │
└── sidecar-container
    ├── Image: ubuntu:latest
    ├── restartPolicy: Always
    └── Reads logs from:
        /var/log/nginx
```

Both containers access the same underlying log files through the shared volume.

---

## Kubernetes Manifest

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

## Implementation

Create the manifest:

```bash
vi /tmp/webserver.yaml
```

Apply it:

```bash
kubectl apply -f /tmp/webserver.yaml
```

Verify the Pod:

```bash
kubectl get pod webserver
```

Inspect the Pod:

```bash
kubectl describe pod webserver
```

Verify the main container:

```bash
kubectl get pod webserver -o jsonpath='{.spec.containers[*].name}'
```

Expected:

```text
nginx-container
```

Verify the sidecar:

```bash
kubectl get pod webserver -o jsonpath='{.spec.initContainers[*].name}'
```

Expected:

```text
sidecar-container
```

---

## Verify the Shared Volume

Check the Nginx log directory:

```bash
kubectl exec webserver -c nginx-container -- ls -la /var/log/nginx
```

Check the same directory from the sidecar:

```bash
kubectl exec webserver -c sidecar-container -- ls -la /var/log/nginx
```

Because both containers mount the same `emptyDir`, they access the same underlying files.

---

## Verify Sidecar Log Reading

View sidecar output:

```bash
kubectl logs webserver -c sidecar-container
```

Follow it continuously:

```bash
kubectl logs -f webserver -c sidecar-container
```

The sidecar repeatedly reads:

```text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

every 30 seconds.

---

## Why `restartPolicy: Always` Matters

A traditional init container must finish before the main application container starts.

However, the sidecar command contains:

```bash
while true
```

which never exits.

Without:

```yaml
restartPolicy: Always
```

the Pod would remain stuck during initialization.

Using `restartPolicy: Always` makes the init container a **restartable sidecar**, allowing it to keep running while Kubernetes continues starting the main application container.

---

## Separation of Concerns

This challenge demonstrates a clean separation of responsibilities:

```text
nginx-container
    ├── Serves web traffic
    └── Writes logs

sidecar-container
    └── Reads/processes log data
```

Each container specializes in one task.

---

## Key Concepts Practiced

- Kubernetes Pods
- Multi-container Pods
- Sidecar pattern
- Restartable init containers
- `restartPolicy: Always`
- `emptyDir` volumes
- Shared storage between containers
- Nginx access/error logs
- `kubectl exec`
- `kubectl logs`
- Separation of concerns

---

## Real-World Use Cases

The sidecar pattern is commonly used for:

- Log collection and forwarding
- Metrics exporters
- Reverse proxies
- Configuration refreshers
- File synchronization
- Security agents
- Supporting application processes

Example:

```text
Application Container
        │
        ▼
   Shared Log Volume
        │
        ▼
Logging Sidecar
        │
        ▼
Log Aggregation Platform
```

---

## Result

The challenge was completed successfully.

The `webserver` Pod runs Nginx as the main application container and a restartable init container as the sidecar. Both share the `shared-logs` `emptyDir` volume at `/var/log/nginx`, allowing the sidecar to continuously read the Nginx access and error logs.
