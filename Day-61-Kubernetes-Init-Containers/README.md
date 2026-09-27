# DevOps Day 61 — Kubernetes Init Containers

## Overview

This challenge focused on deploying a Kubernetes application that requires some initialization work to be completed **before the main application container starts**.

To achieve this, a Kubernetes **Init Container** was configured inside a Deployment. The init container wrote a startup message into a shared `emptyDir` volume, and the main container continuously read that message from the same mounted volume.

This task demonstrates one of the most important Kubernetes workload patterns: using **Init Containers for application prerequisites and setup tasks**.

---

## Task Requirements

The Deployment had to meet the following requirements:

- Create a Deployment named `ic-deploy-nautilus`.
- Configure `1` replica.
- Configure the pod label:

```yaml
app: ic-nautilus
```

- Configure an Init Container:
  - Name: `ic-msg-nautilus`
  - Image: `fedora:latest`
  - Command:

```bash
/bin/bash -c 'echo "Init Done - Welcome to xFusionCorp Industries" > /ic/ecommerce'
```

- Configure the main container:
  - Name: `ic-main-nautilus`
  - Image: `fedora:latest`
  - Command:

```bash
/bin/bash -c 'while true; do cat /ic/ecommerce; sleep 5; done'
```

- Create a volume named `ic-volume-nautilus`.
- Use an `emptyDir` volume.
- Mount the shared volume at `/ic` in both containers.

---

## Architecture

```text
Kubernetes Deployment
        |
        v
+--------------------------------------+
| Pod                                  |
|                                      |
|  Init Container                      |
|  ic-msg-nautilus                     |
|                                      |
|  echo message > /ic/ecommerce        |
|            |                         |
|            v                         |
|      +-------------+                 |
|      |  emptyDir   |                 |
|      |   Volume    |                 |
|      +-------------+                 |
|            ^                         |
|            |                         |
|  Main Container                      |
|  ic-main-nautilus                    |
|                                      |
|  cat /ic/ecommerce every 5 seconds   |
+--------------------------------------+
```

The Init Container runs first and must complete successfully before Kubernetes starts the main container.

---

## Kubernetes Manifest

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

## Deployment Procedure

Create the manifest:

```bash
vi /tmp/ic-deploy-nautilus.yaml
```

Validate the YAML:

```bash
kubectl apply -f /tmp/ic-deploy-nautilus.yaml --dry-run=client
```

Create the Deployment:

```bash
kubectl apply -f /tmp/ic-deploy-nautilus.yaml
```

Verify the Deployment:

```bash
kubectl get deployments
```

Verify the pod:

```bash
kubectl get pods
```

Get the pod name:

```bash
POD=$(kubectl get pods -l app=ic-nautilus -o jsonpath='{.items[0].metadata.name}')
```

Verify Init Container status:

```bash
kubectl describe pod "$POD"
```

Verify main-container logs:

```bash
kubectl logs "$POD" -c ic-main-nautilus
```

Expected output:

```text
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
...
```

---

## What Is an Init Container?

An **Init Container** is a special container that runs before the regular application containers in a Kubernetes Pod.

Unlike normal containers, Init Containers:

- run before application containers;
- normally perform short-lived setup work;
- must complete successfully before application containers start;
- execute sequentially when multiple Init Containers are defined;
- can use volumes shared with the main containers;
- can use different images and tools from the main application.

They are useful when application startup depends on some prerequisite.

---

## Why Init Containers Are Useful

Typical use cases include:

- creating configuration files;
- preparing directories;
- downloading required files;
- waiting for another service to become available;
- performing database initialization;
- fixing file permissions;
- generating certificates or configuration;
- cloning application content;
- populating shared volumes before application startup.

This helps keep the main application image focused on the application itself rather than embedding initialization logic inside it.

---

## Shared `emptyDir` Volume

The task used:

```yaml
volumes:
  - name: ic-volume-nautilus
    emptyDir: {}
```

An `emptyDir` volume is created when the Pod is assigned to a node.

Both containers mount it:

```yaml
volumeMounts:
  - name: ic-volume-nautilus
    mountPath: /ic
```

Therefore:

```text
Init Container
     |
     | writes
     v
/ic/ecommerce
     ^
     | reads
     |
Main Container
```

The data survives container restarts within the same Pod but is removed when the Pod itself is deleted.

---

## Init Container Execution Flow

The lifecycle is:

```text
Pod scheduled
     |
     v
Init Container starts
     |
     v
Initialization command runs
     |
     v
Init Container exits successfully
     |
     v
Main container starts
     |
     v
Application reaches Running state
```

If the Init Container fails, the main application container does not start until initialization eventually succeeds or the Pod is otherwise terminated.

---

## Verification

The following command verifies the Init Container:

```bash
kubectl describe pod "$POD"
```

A successful Init Container typically shows:

```text
State:
  Terminated:
    Reason: Completed
    Exit Code: 0
```

The main container can then be checked with:

```bash
kubectl logs "$POD" -c ic-main-nautilus
```

---

## Key Learning Outcomes

This challenge demonstrated:

- how Kubernetes Init Containers work;
- how initialization can be separated from application runtime;
- how container startup ordering works inside a Pod;
- how Init Containers and application containers share volumes;
- how `emptyDir` provides Pod-scoped temporary storage;
- how to verify Init Container completion;
- how to inspect logs from specific containers in multi-container Pods.

---

## Final Result

The `ic-deploy-nautilus` Deployment was successfully created with:

- one replica;
- one Init Container;
- one main application container;
- one shared `emptyDir` volume;
- correct startup ordering;
- successful data sharing between the containers.

The Kubernetes cluster successfully started the Init Container, created `/ic/ecommerce`, completed initialization, and then started the main container, which continuously read the generated content.

**Status: Completed Successfully**
