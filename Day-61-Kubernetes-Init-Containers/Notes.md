# DevOps Day 61 — Notes

## Kubernetes Init Containers

An **Init Container** is a special container that runs during Pod initialization and completes its work before the regular application containers begin running.

Init Containers are defined using:

```yaml
spec:
  initContainers:
```

Example:

```yaml
initContainers:
  - name: initialization-container
    image: fedora:latest
    command:
      - /bin/bash
      - -c
      - echo "Initialization completed"
```

---

## Init Containers vs Normal Containers

| Feature | Init Container | Application Container |
|---|---|---|
| Purpose | Initialization/setup | Run the application |
| Startup time | Before app containers | After Init Containers finish |
| Typical lifetime | Short-lived | Usually long-running |
| Must finish successfully | Yes | Normally expected to remain running |
| Multiple containers | Run sequentially | Usually run concurrently |
| Shared volumes | Supported | Supported |
| Different image | Supported | Supported |

---

## Execution Order

Suppose a Pod contains:

```text
Init Container A
Init Container B
Main Container
```

Kubernetes executes them as:

```text
Init A
  |
  | success
  v
Init B
  |
  | success
  v
Main Container
```

The next Init Container does not start until the previous one exits successfully.

Application containers do not start until **all Init Containers complete successfully**.

---

## Why Init Containers Exist

Applications frequently need some work performed before startup.

Examples:

- wait for a database;
- download configuration;
- clone a Git repository;
- generate files;
- populate a shared volume;
- check dependent services;
- change file permissions;
- perform setup scripts.

Rather than putting all of this logic into the main application image, Kubernetes allows initialization logic to be placed in separate containers.

This creates cleaner separation between:

```text
Initialization responsibility
```

and

```text
Application responsibility
```

---

## Day 61 Scenario

The Pod contains two containers.

### Init Container

```text
ic-msg-nautilus
```

It executes:

```bash
echo "Init Done - Welcome to xFusionCorp Industries" > /ic/ecommerce
```

This creates:

```text
/ic/ecommerce
```

inside the mounted shared volume.

### Main Container

```text
ic-main-nautilus
```

It executes:

```bash
while true; do cat /ic/ecommerce; sleep 5; done
```

Therefore the main container continuously reads the file created by the Init Container.

---

## Why the File Is Available to Both Containers

Containers normally have independent writable filesystems.

Therefore, writing a file inside one container does **not automatically** make that file available inside another container.

The reason this task works is the shared volume:

```yaml
volumes:
  - name: ic-volume-nautilus
    emptyDir: {}
```

Both containers mount that same volume:

```yaml
volumeMounts:
  - name: ic-volume-nautilus
    mountPath: /ic
```

The actual flow is:

```text
Init Container filesystem
        |
        | mounted /ic
        v
+-----------------------+
|     emptyDir          |
|                       |
| /ecommerce            |
+-----------------------+
        ^
        | mounted /ic
        |
Main Container filesystem
```

---

## `emptyDir` Volume

`emptyDir` is temporary Pod-level storage.

It is created when the Pod is assigned to a node.

Example:

```yaml
volumes:
  - name: application-data
    emptyDir: {}
```

Important properties:

- initially empty;
- shared between containers in the same Pod;
- survives individual container restarts;
- removed when the Pod is permanently removed;
- suitable for temporary files, caches, and inter-container sharing.

It is **not persistent storage**.

---

## Container Restart vs Pod Deletion

This distinction is important.

### Container restarts

If the application container crashes and Kubernetes restarts it inside the same Pod:

```text
emptyDir data remains available
```

### Pod deletion/replacement

If the Pod is deleted and a new Pod is created:

```text
old emptyDir data is lost
```

A new `emptyDir` is created for the new Pod.

Therefore it should never be treated as permanent application storage.

---

## Init Container Failure Behavior

If the Init Container fails:

```text
Init Container fails
        |
        v
Main container does NOT start
```

Kubernetes retries initialization according to the Pod restart behavior.

You may observe states such as:

```text
Init:0/1
```

or:

```text
Init:Error
```

or:

```text
Init:CrashLoopBackOff
```

depending on the problem.

---

## Understanding `Init:0/1`

While the Init Container is still running, `kubectl get pods` can show:

```text
READY   STATUS
0/1     Init:0/1
```

This does not mean the application container is broken.

It means:

```text
0 out of 1 Init Containers have completed
```

When initialization completes, Kubernetes proceeds to the main container.

---

## Why a Running Pod Shows `1/1`

This Pod defines:

```text
1 Init Container
1 Application Container
```

But after startup:

```bash
kubectl get pods
```

shows:

```text
1/1 Running
```

The READY column counts application containers, not completed Init Containers.

The Init Container has already terminated successfully.

---

## Checking Init Container Status

Run:

```bash
kubectl describe pod <pod-name>
```

A successful Init Container should show something similar to:

```text
Init Containers:
  ic-msg-nautilus:

    State:
      Terminated:

        Reason: Completed
        Exit Code: 0
```

The two most important fields are:

```text
Reason: Completed
Exit Code: 0
```

---

## Logs for Specific Containers

When a Pod contains multiple containers, specify the container name.

Main container:

```bash
kubectl logs <pod-name> -c ic-main-nautilus
```

Init Container:

```bash
kubectl logs <pod-name> -c ic-msg-nautilus
```

General pattern:

```bash
kubectl logs <pod-name> -c <container-name>
```

---

## `command` in Kubernetes

A Kubernetes container definition can override the image command.

For example:

```yaml
command:
  - /bin/bash
  - -c
  - echo "Hello"
```

Conceptually this executes:

```bash
/bin/bash -c 'echo "Hello"'
```

The `-c` option tells Bash to execute the following string as a shell command.

It is especially useful for shell constructs such as:

```bash
echo
```

```bash
while
```

```bash
&&
```

```bash
>
```

and pipelines.

---

## Shell Redirection in the Init Container

The command:

```bash
echo "Init Done - Welcome to xFusionCorp Industries" > /ic/ecommerce
```

uses:

```text
>
```

to redirect stdout into a file.

Therefore:

```text
echo output
    |
    v
/ic/ecommerce
```

The message is stored in the shared volume rather than being printed as normal container output.

---

## Main Container Loop

The main container uses:

```bash
while true; do cat /ic/ecommerce; sleep 5; done
```

Breakdown:

```bash
while true
```

creates an infinite loop.

```bash
cat /ic/ecommerce
```

prints the file.

```bash
sleep 5
```

waits five seconds.

```bash
done
```

ends the loop body and starts the next iteration.

The container therefore remains alive while continuously printing the initialized content.

---

## Common Init Container Use Cases

### 1. Wait for a Dependency

```bash
until nc -z database 3306; do sleep 2; done
```

The application starts only when the database becomes reachable.

---

### 2. Download Configuration

```bash
curl -o /config/app.conf https://example.com/app.conf
```

The main container then reads `/config/app.conf`.

---

### 3. Clone Application Code

```bash
git clone https://example.com/project.git /app
```

The application container can use the shared `/app` volume.

---

### 4. Generate Configuration

An Init Container can dynamically generate:

```text
nginx.conf
application.properties
JSON configuration
environment-specific files
```

before application startup.

---

### 5. Set Permissions

For volumes that require specific ownership:

```bash
chown -R 1000:1000 /data
```

The main application can then access the volume using its non-root user.

---

## Important Kubernetes Design Principle

Init Containers make it possible to separate concerns.

Instead of creating one large container image containing:

```text
application
curl
git
database tools
setup scripts
debug utilities
```

you can use:

```text
Init Container
    |
    +--> initialization tools

Main Container
    |
    +--> application only
```

This can keep application images smaller and easier to maintain.

---

## Init Containers vs Sidecar Containers

These concepts should not be confused.

### Init Container

```text
starts
  |
performs setup
  |
exits
  |
main application starts
```

### Sidecar

```text
main application + sidecar
         |
         |
         +--> normally run together
```

Common sidecars include:

- log collectors;
- proxies;
- monitoring agents;
- configuration reloaders.

The key distinction is that an Init Container normally **finishes before the application starts**, while a sidecar usually continues running with the application.

---

## Troubleshooting Init Containers

If the Pod does not start correctly, begin with:

```bash
kubectl get pods
```

Then:

```bash
kubectl describe pod <pod-name>
```

Check:

- Init Container state;
- exit code;
- image pull errors;
- command failures;
- volume mount errors;
- cluster events.

Then inspect logs:

```bash
kubectl logs <pod-name> -c <init-container-name>
```

For this task:

```bash
kubectl logs "$POD" -c ic-msg-nautilus
```

---

## Common Problems

### Wrong volume name

This fails if the volume mount references a volume that does not exist:

```yaml
volumeMounts:
  - name: wrong-volume
```

The `name` must match the volume declaration exactly.

---

### Different mount paths

If the Init Container writes:

```text
/ic/ecommerce
```

but the main container does not mount the shared volume at `/ic`, the application will not see that file.

---

### Init Container never exits

If an Init Container runs an infinite process, Kubernetes never proceeds to the application container.

For example, this would be inappropriate for most Init Containers:

```bash
while true; do sleep 10; done
```

because it never finishes.

---

### Incorrect shell syntax

Commands involving redirection or loops should generally be executed through a shell:

```yaml
command:
  - /bin/bash
  - -c
  - <shell-command>
```

Without a shell, operators such as `>`, `|`, `&&`, and loops are not interpreted in the same way.

---

## Key Commands to Remember

```bash
kubectl get pods
```

```bash
kubectl describe pod <pod-name>
```

```bash
kubectl logs <pod-name> -c <container-name>
```

```bash
kubectl exec <pod-name> -c <container-name> -- <command>
```

```bash
kubectl get pod <pod-name> -o yaml
```

---

## Key Takeaway

The central concept from Day 61 is:

> **Init Containers prepare the environment of a Kubernetes Pod before the main application containers are allowed to start.**

In this challenge:

```text
Init Container
      |
      | writes initialization data
      v
shared emptyDir
      |
      | provides initialized data
      v
Main Container
```

This pattern is extremely useful for dependency checks, configuration generation, shared-volume preparation, downloads, migrations, and other startup prerequisites.
