# Day 62 — Kubernetes Secrets

## Overview

This challenge focuses on securely storing sensitive information in Kubernetes using a **Secret** and making that data available inside a Pod through a mounted volume.

Instead of hardcoding confidential values such as passwords, API keys, or license numbers directly inside container images or Pod manifests, Kubernetes Secrets allow sensitive data to be stored separately and consumed by workloads when required.

---

## Task Requirements

The challenge required the following:

- Use the existing file `/opt/blog.txt` as the source of the secret data.
- Create a generic Kubernetes Secret named `blog`.
- Create a Pod named `secret-datacenter`.
- Configure the container with:
  - **Container name:** `secret-container-datacenter`
  - **Image:** `fedora:latest`
- Keep the container running using a `sleep` command.
- Mount the Secret inside the container at `/opt/games`.
- Verify that the secret file is available inside the running container.

---

## Solution Summary

The task was completed by creating a generic Secret directly from the existing `/opt/blog.txt` file.

The Secret was then exposed to the Pod as a Kubernetes Secret volume. That volume was mounted inside the Fedora container at `/opt/games`.

Because the Secret key was created with the filename `blog.txt`, Kubernetes automatically exposed the value as:

```text
/opt/games/blog.txt
```

The Pod was kept alive with a long-running `sleep` command so that the mounted Secret could be verified using `kubectl exec`.

---

## Architecture

```text
/opt/blog.txt
      |
      | kubectl create secret generic
      v
+----------------------------+
| Kubernetes Secret          |
| Name: blog                 |
| Key: blog.txt              |
+-------------+--------------+
              |
              | Secret Volume
              v
+--------------------------------------+
| Pod: secret-datacenter               |
|                                      |
| Container:                           |
| secret-container-datacenter          |
| Image: fedora:latest                 |
|                                      |
| Secret mounted at:                   |
| /opt/games/blog.txt                  |
+--------------------------------------+
```

---

## Kubernetes Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-datacenter
spec:
  containers:
    - name: secret-container-datacenter
      image: fedora:latest
      command:
        - /bin/sh
        - -c
        - "sleep infinity"
      volumeMounts:
        - name: blog-secret-volume
          mountPath: /opt/games
          readOnly: true

  volumes:
    - name: blog-secret-volume
      secret:
        secretName: blog
```

---

## Verification

The Secret was verified with:

```bash
kubectl get secret blog
kubectl describe secret blog
```

The Pod status was checked with:

```bash
kubectl get pod secret-datacenter
```

The mounted secret was verified directly inside the container:

```bash
kubectl exec secret-datacenter   -c secret-container-datacenter --   cat /opt/games/blog.txt
```

The Pod remained in the `Running` state and the secret data was successfully available at the required mount location.

---

## Key Concepts Practiced

- Kubernetes Secrets
- Generic Secrets
- Creating Secrets from files
- Secret keys and values
- Secret-backed volumes
- Pod volume configuration
- `volumeMounts`
- Read-only Secret mounts
- Secure configuration handling
- `kubectl exec`
- Kubernetes workload verification

---

## Why Kubernetes Secrets Matter

Sensitive configuration should not normally be embedded directly into:

- Container images
- Application source code
- Git repositories
- Plain-text Pod specifications

Kubernetes Secrets separate sensitive information from workload definitions and allow applications to consume that information only when required.

In this challenge, the sensitive license/password information remained separate from the Fedora image and was injected into the running Pod as a mounted file.

---

## Result

Day 62 was completed successfully.

The `blog` Kubernetes Secret was created from `/opt/blog.txt`, mounted into the `secret-datacenter` Pod at `/opt/games`, and successfully verified from inside the `secret-container-datacenter` container.
