# Day 57 — Kubernetes Environment Variables and Pod Lifecycle Notes

## 1. Core Objective

The purpose of this challenge was to understand how Kubernetes passes environment variables into a container and how those values can be consumed by a process running inside that container.

The Pod was intentionally designed as a short-lived workload.

Its job was simply to:

```text
Start
  ↓
Receive environment variables
  ↓
Print a greeting
  ↓
Exit successfully
```

---

# 2. Kubernetes Pod

A **Pod** is the smallest deployable workload unit in Kubernetes.

A Pod contains one or more containers that share the same Pod-level environment, such as:

- Network namespace
- Pod IP
- Storage volumes
- Lifecycle

In this challenge, the Pod contains only one container.

```text
Pod: print-envars-greeting
└── Container: print-env-container
```

---

# 3. Pod Definition Used

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
```

Important values:

```text
Pod name       = print-envars-greeting
Container name = print-env-container
Image          = bash
Restart policy = Never
```

---

# 4. Environment Variables in Kubernetes

Environment variables are values made available to processes running inside a container.

In Kubernetes they can be defined directly using the `env` field.

Example:

```yaml
env:
  - name: COMPANY
    value: "DevOps"
```

Inside the container, the process receives:

```text
COMPANY=DevOps
```

The shell can access the variable using:

```bash
$COMPANY
```

or:

```bash
${COMPANY}
```

---

# 5. Environment Variables Used in the Challenge

Three variables were configured:

```yaml
env:
  - name: GREETING
    value: "Welcome to"

  - name: COMPANY
    value: "DevOps"

  - name: GROUP
    value: "Group"
```

Inside the container they become:

```text
GREETING=Welcome to
COMPANY=DevOps
GROUP=Group
```

---

# 6. Why `"Welcome to"` Was Quoted

The value contains a space:

```text
Welcome to
```

Using quotes makes the YAML value clear and unambiguous:

```yaml
value: "Welcome to"
```

The actual stored environment-variable value does not contain the quote characters.

The container receives:

```text
Welcome to
```

not:

```text
"Welcome to"
```

---

# 7. Kubernetes `command`

The challenge used:

```yaml
command:
  - /bin/sh
  - -c
  - 'echo "${GREETING} ${COMPANY} ${GROUP}"'
```

This becomes conceptually:

```bash
/bin/sh -c 'echo "${GREETING} ${COMPANY} ${GROUP}"'
```

The shell receives the command string and executes it.

---

# 8. Why `/bin/sh -c` Is Used

Environment variables are expanded by a shell.

For example:

```bash
echo "${COMPANY}"
```

requires shell interpretation so that:

```text
${COMPANY}
```

is replaced with:

```text
DevOps
```

Using:

```bash
/bin/sh -c
```

starts a shell and tells it to execute the following command string.

---

# 9. Variable Expansion

Given:

```text
GREETING=Welcome to
COMPANY=DevOps
GROUP=Group
```

the command:

```bash
echo "${GREETING} ${COMPANY} ${GROUP}"
```

is expanded by the shell into:

```bash
echo "Welcome to DevOps Group"
```

The resulting output is:

```text
Welcome to DevOps Group
```

---

# 10. Why `${VARIABLE}` Syntax Is Useful

Both forms can work:

```bash
$COMPANY
```

and:

```bash
${COMPANY}
```

The brace form is often clearer because it explicitly marks the variable name boundary.

Example:

```bash
echo "${COMPANY}_team"
```

clearly means:

```text
DevOps_team
```

Without braces:

```bash
echo "$COMPANY_team"
```

the shell may interpret the variable name as:

```text
COMPANY_team
```

instead.

---

# 11. Pod Restart Policy

Kubernetes Pods support three restart policies:

```text
Always
OnFailure
Never
```

---

## `Always`

The container is restarted whenever it exits.

This is the default restart policy for Pods.

Typical long-running applications often use behavior equivalent to:

```yaml
restartPolicy: Always
```

---

## `OnFailure`

The container is restarted only when it exits unsuccessfully.

For example:

```text
Exit code 1 → restart
Exit code 0 → do not restart
```

---

## `Never`

The container is not restarted after it exits.

This challenge required:

```yaml
restartPolicy: Never
```

because the command is intentionally short-lived.

---

# 12. Why `restartPolicy: Never` Was Required

The container executes:

```bash
echo "${GREETING} ${COMPANY} ${GROUP}"
```

`echo` finishes almost immediately.

If Kubernetes continuously restarted this container, it would repeatedly print the same message and terminate.

With:

```yaml
restartPolicy: Never
```

the workflow is:

```text
Container starts
      ↓
Command executes
      ↓
Output printed
      ↓
Process exits with code 0
      ↓
No restart
      ↓
Pod becomes Completed
```

---

# 13. `Completed` Is Not an Error

A common beginner mistake is to assume:

```text
STATUS = Completed
```

means something went wrong.

For this challenge, `Completed` is the expected state.

It means the container process:

```text
Started successfully
        +
Finished successfully
        +
Exited with code 0
```

The Pod was designed to finish.

---

# 14. Exit Codes

Linux processes return an exit code when they finish.

The most important rule is:

```text
0     = success
non-0 = failure/error
```

The Pod's terminated container can be checked with:

```bash
kubectl get pod print-envars-greeting \
-o jsonpath='{.status.containerStatuses[0].state.terminated.exitCode}'
```

Expected:

```text
0
```

This confirms successful execution.

---

# 15. Pod Lifecycle in This Challenge

The Pod may briefly move through states similar to:

```text
Pending
   ↓
ContainerCreating
   ↓
Running
   ↓
Completed
```

Because the command is extremely short, the `Running` state may be so brief that it is not observed manually.

---

# 16. Logs

Standard output produced by the container can be viewed through:

```bash
kubectl logs print-envars-greeting
```

Since the application executes:

```bash
echo ...
```

that output is captured by Kubernetes/container runtime logging.

Expected:

```text
Welcome to DevOps Group
```

---

# 17. Why Logs Still Exist After Completion

The container process has stopped, but its log output can still be retrieved while the Pod/container record remains available.

Therefore:

```text
Pod status = Completed
```

does not prevent:

```bash
kubectl logs print-envars-greeting
```

from retrieving the output.

---

# 18. Kubernetes `command` vs Docker ENTRYPOINT

A useful Kubernetes concept is that the Kubernetes `command` field corresponds conceptually to overriding the container image's default entrypoint.

For example:

```yaml
command:
  - /bin/sh
  - -c
  - 'echo ...'
```

tells Kubernetes to start the container using this command rather than relying solely on the image's default startup command.

---

# 19. `command` vs `args`

At a high level:

```text
Kubernetes command → overrides container ENTRYPOINT
Kubernetes args    → overrides/provides arguments similar to CMD
```

Example:

```yaml
command:
  - /bin/sh
args:
  - -c
  - echo hello
```

can be used to represent:

```bash
/bin/sh -c "echo hello"
```

In this challenge, the complete command was supplied in `command`.

---

# 20. Environment Variable Sources in Kubernetes

This challenge used literal values:

```yaml
env:
  - name: COMPANY
    value: "DevOps"
```

However, Kubernetes can also populate environment variables from other sources such as:

```text
ConfigMaps
Secrets
Pod fields
Resource fields
```

For example, an application configuration value could come from a ConfigMap rather than being hardcoded into the Pod manifest.

That becomes especially useful in real projects.

---

# 21. Literal Environment Variables vs ConfigMaps

Literal definition:

```yaml
env:
  - name: COMPANY
    value: "DevOps"
```

is simple and appropriate for small static values.

For larger configuration sets, a ConfigMap is often more manageable:

```text
ConfigMap
    ↓
Environment variables
    ↓
Pod
```

This separates application configuration from the Pod definition.

---

# 22. Environment Variables and Secrets

Sensitive values such as passwords should generally not be hardcoded directly into plain Pod YAML.

Instead, Kubernetes Secrets are commonly used.

Conceptually:

```text
Kubernetes Secret
       ↓
Environment variable
       ↓
Container
```

This challenge does not involve sensitive data, so direct values were appropriate.

---

# 23. Verification with JSONPath

`kubectl` supports JSONPath expressions that make it possible to extract specific fields.

Example:

```bash
kubectl get pod print-envars-greeting \
-o jsonpath='{.spec.restartPolicy}'
```

Output:

```text
Never
```

This is useful for precise validation when a challenge requires exact configuration values.

---

# 24. Checking All Environment Variables

The command:

```bash
kubectl get pod print-envars-greeting \
-o jsonpath='{range .spec.containers[0].env[*]}{.name}={.value}{"\n"}{end}'
```

uses a loop over:

```text
.spec.containers[0].env[*]
```

and prints:

```text
name=value
```

for every configured variable.

Expected:

```text
GREETING=Welcome to
COMPANY=DevOps
GROUP=Group
```

---

# 25. Troubleshooting Environment Variables

If the output is wrong, inspect the configured variables:

```bash
kubectl get pod print-envars-greeting -o yaml
```

or:

```bash
kubectl describe pod print-envars-greeting
```

Check for:

```text
Variable name spelling
Variable values
Shell syntax
Command quoting
Container image
Restart policy
```

---

# 26. Common Mistake: Wrong Variable Name

Suppose the manifest defines:

```yaml
- name: COMPANY
  value: "DevOps"
```

but the command references:

```bash
${COMPANYY}
```

Then that variable is undefined and usually expands to an empty string.

The output could become:

```text
Welcome to Group
```

Therefore variable names must match exactly.

---

# 27. Common Mistake: Restart Policy in the Wrong Place

Correct:

```yaml
spec:
  restartPolicy: Never

  containers:
    - name: ...
```

Incorrect conceptually:

```yaml
containers:
  - name: ...
    restartPolicy: Never
```

`restartPolicy` is a **Pod-level** setting, not a container-level field for this configuration.

---

# 28. Common Mistake: Expecting the Pod to Stay Running

This Pod is not a web server or daemon.

It does not have a long-running process.

Its main process is:

```bash
echo ...
```

Once `echo` finishes, there is nothing left to keep the container alive.

Therefore:

```text
Completed
```

is correct.

---

# 29. One-Time Pods vs Long-Running Pods

## One-Time Workload

Example:

```bash
echo "hello"
```

Lifecycle:

```text
Start → Execute → Exit
```

A completed state is expected.

---

## Long-Running Workload

Example:

```text
nginx
```

Lifecycle:

```text
Start → Continue serving requests
```

A persistent `Running` state is expected.

This distinction is fundamental when interpreting Kubernetes Pod status.

---

# 30. Important Mental Model

For this challenge, think of the Pod as a small job:

```text
Kubernetes creates container
        ↓
Kubernetes injects environment
        ↓
Shell starts
        ↓
Shell expands variables
        ↓
echo prints greeting
        ↓
Process exits successfully
        ↓
restartPolicy prevents restart
        ↓
Pod = Completed
```

---

# 31. Final Architecture Summary

```text
Pod: print-envars-greeting
│
└── Container: print-env-container
    │
    ├── Image: bash
    │
    ├── Environment
    │   ├── GREETING = Welcome to
    │   ├── COMPANY  = DevOps
    │   └── GROUP    = Group
    │
    └── Command
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
         Exit code 0
             │
             ▼
          Completed
```

---

# 32. Key Takeaways

1. Kubernetes can inject environment variables directly into container processes.
2. Environment variables are configured under the container's `env` section.
3. `/bin/sh -c` allows shell features such as variable expansion.
4. `${VARIABLE}` is a clear way to reference shell environment variables.
5. `restartPolicy` is configured at the Pod specification level.
6. `restartPolicy: Never` prevents a completed container from restarting.
7. `Completed` can be the correct state for a successful short-lived Pod.
8. Exit code `0` indicates successful command execution.
9. `kubectl logs` can retrieve output even after the container has completed.
10. JSONPath is useful for exact Kubernetes configuration verification.
