# Day 57 — Kubernetes Environment Variables in a Pod

## Overview

In this challenge, a Kubernetes Pod was created to test how environment variables can be passed into a container and consumed by a shell command.

The Pod uses the `bash` image, defines three environment variables, prints them using `/bin/sh -c`, and exits successfully. Since this is a one-time task rather than a continuously running application, the Pod uses `restartPolicy: Never`.

---

## Task Requirements

| Requirement | Configuration |
|---|---|
| Pod name | `print-envars-greeting` |
| Container name | `print-env-container` |
| Image | `bash` |
| Environment variable 1 | `GREETING=Welcome to` |
| Environment variable 2 | `COMPANY=DevOps` |
| Environment variable 3 | `GROUP=Group` |
| Command | `/bin/sh -c 'echo "${GREETING} ${COMPANY} ${GROUP}"'` |
| Restart policy | `Never` |

Expected output:

```text
Welcome to DevOps Group
```

---

## Architecture

```text
Kubernetes Pod
└── print-envars-greeting
    │
    └── Container: print-env-container
        │
        ├── Image: bash
        │
        ├── GREETING="Welcome to"
        ├── COMPANY="DevOps"
        ├── GROUP="Group"
        │
        └── /bin/sh -c
              │
              ▼
        echo "${GREETING} ${COMPANY} ${GROUP}"
              │
              ▼
        Welcome to DevOps Group
              │
              ▼
          Process exits
              │
              ▼
        Pod status: Completed
```

---

## Kubernetes Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: print-envars-greeting

spec:
  restartPolicy: Never

  containers:
    - name: print-env-container
      image: bash

      env:
        - name: GREETING
          value: "Welcome to"

        - name: COMPANY
          value: "DevOps"

        - name: GROUP
          value: "Group"

      command:
        - /bin/sh
        - -c
        - 'echo "${GREETING} ${COMPANY} ${GROUP}"'
```

---

## Implementation

The manifest was created and applied using:

```bash
kubectl apply -f day57-pod.yaml
```

Kubernetes then:

1. Created the Pod.
2. Pulled the `bash` image if required.
3. Injected the configured environment variables into the container.
4. Started `/bin/sh`.
5. Executed the `echo` command.
6. Printed the expected greeting.
7. Allowed the container to exit successfully.
8. Left the Pod in the `Completed` state because the restart policy was `Never`.

---

## Verification

### Check the Pod

```bash
kubectl get pod print-envars-greeting
```

A successful one-time Pod eventually shows:

```text
STATUS
Completed
```

---

### Check the Logs

```bash
kubectl logs print-envars-greeting
```

Expected output:

```text
Welcome to DevOps Group
```

---

### Verify Environment Variables

```bash
kubectl get pod print-envars-greeting \
-o jsonpath='{range .spec.containers[0].env[*]}{.name}={.value}{"\n"}{end}'
```

Expected:

```text
GREETING=Welcome to
COMPANY=DevOps
GROUP=Group
```

---

### Verify Container Configuration

```bash
kubectl get pod print-envars-greeting \
-o jsonpath='{.spec.containers[0].name}{"\n"}{.spec.containers[0].image}{"\n"}'
```

Expected:

```text
print-env-container
bash
```

---

### Verify Restart Policy

```bash
kubectl get pod print-envars-greeting \
-o jsonpath='{.spec.restartPolicy}{"\n"}'
```

Expected:

```text
Never
```

---

## Key Concepts Practiced

- Kubernetes Pods
- Pod specifications
- Container environment variables
- Shell environment-variable expansion
- Kubernetes `command`
- `/bin/sh -c`
- One-time container execution
- Pod lifecycle
- `restartPolicy`
- Pod logs
- JSONPath-based verification
- Declarative Kubernetes configuration

---

## Why `restartPolicy: Never` Was Important

The container in this challenge performs only one action:

```bash
echo "${GREETING} ${COMPANY} ${GROUP}"
```

After printing the message, the process exits successfully.

With:

```yaml
restartPolicy: Never
```

Kubernetes does not restart the container after it exits.

The expected lifecycle is therefore:

```text
Pending
  ↓
Running
  ↓
Command executes
  ↓
Process exits with code 0
  ↓
Completed
```

This behavior is correct for a short-lived one-time task.

---

## Environment Variable Flow

The environment variables are defined in the Pod specification:

```yaml
env:
  - name: GREETING
    value: "Welcome to"
```

Kubernetes injects them into the container process environment.

Inside the shell, they can then be referenced as:

```bash
${GREETING}
${COMPANY}
${GROUP}
```

The shell expands them before executing `echo`.

---

## Result

The Day 57 challenge was completed successfully.

The Pod:

- Used the required `bash` image
- Used the required container name
- Received all three environment variables
- Printed the expected greeting
- Exited successfully
- Did not enter a restart loop because `restartPolicy` was set to `Never`

---
## Conclusion

This challenge demonstrated how Kubernetes can inject environment variables into containers and how those variables can be consumed by commands running inside the container.

It also highlighted an important Pod lifecycle concept: not every Pod is expected to run forever. For short-lived workloads, a successful `Completed` state can be the correct final state.
