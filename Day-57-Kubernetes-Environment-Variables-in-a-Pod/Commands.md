# Day 57 — Commands Reference

This file contains the commands used to complete and verify the Kubernetes environment-variable Pod challenge.

---

## 1. Create the Pod Manifest

```bash
cat <<'EOF' > day57-pod.yaml
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
EOF
```

---

## 2. Apply the Manifest

```bash
kubectl apply -f day57-pod.yaml
```

Expected:

```text
pod/print-envars-greeting created
```

---

## 3. Check Pod Status

```bash
kubectl get pod print-envars-greeting
```

Expected final state:

```text
Completed
```

Detailed view:

```bash
kubectl get pod print-envars-greeting -o wide
```

---

## 4. Check Container Logs

```bash
kubectl logs print-envars-greeting
```

Expected:

```text
Welcome to DevOps Group
```

Follow logs:

```bash
kubectl logs -f print-envars-greeting
```

---

## 5. Verify Environment Variables

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

## 6. Verify Container Name and Image

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

## 7. Verify Restart Policy

```bash
kubectl get pod print-envars-greeting \
-o jsonpath='{.spec.restartPolicy}{"\n"}'
```

Expected:

```text
Never
```

---

## 8. View the Complete Pod Definition

```bash
kubectl get pod print-envars-greeting -o yaml
```

This is useful for checking:

- Environment variables
- Image
- Container name
- Command
- Restart policy
- Pod status

---

## 9. Describe the Pod

```bash
kubectl describe pod print-envars-greeting
```

Useful sections include:

```text
Containers
Environment
Command
State
Reason
Exit Code
Events
```

---

## 10. Verify Exit Status

```bash
kubectl get pod print-envars-greeting \
-o jsonpath='{.status.containerStatuses[0].state.terminated.exitCode}{"\n"}'
```

Expected:

```text
0
```

Exit code `0` means the command completed successfully.

---

## 11. Verify Completion Reason

```bash
kubectl get pod print-envars-greeting \
-o jsonpath='{.status.containerStatuses[0].state.terminated.reason}{"\n"}'
```

Expected:

```text
Completed
```

---

## 12. Combined Verification

```bash
echo "=== Pod ==="
kubectl get pod print-envars-greeting

echo
echo "=== Container ==="
kubectl get pod print-envars-greeting \
-o jsonpath='Name: {.spec.containers[0].name}{"\n"}Image: {.spec.containers[0].image}{"\n"}RestartPolicy: {.spec.restartPolicy}{"\n"}'

echo
echo "=== Environment Variables ==="
kubectl get pod print-envars-greeting \
-o jsonpath='{range .spec.containers[0].env[*]}{.name}={.value}{"\n"}{end}'

echo
echo "=== Output ==="
kubectl logs print-envars-greeting
```

Expected key output:

```text
Name: print-env-container
Image: bash
RestartPolicy: Never

GREETING=Welcome to
COMPANY=DevOps
GROUP=Group

Welcome to DevOps Group
```

---

## 13. Useful Troubleshooting Commands

### Check all Pods

```bash
kubectl get pods
```

### Inspect Pod events

```bash
kubectl describe pod print-envars-greeting
```

### Check logs

```bash
kubectl logs print-envars-greeting
```

### Check previous logs if the container restarted

```bash
kubectl logs --previous print-envars-greeting
```

### Check configured command

```bash
kubectl get pod print-envars-greeting \
-o jsonpath='{.spec.containers[0].command}{"\n"}'
```

### Check environment block

```bash
kubectl get pod print-envars-greeting \
-o jsonpath='{.spec.containers[0].env}{"\n"}'
```

---

## Cleanup

If the Pod needs to be removed:

```bash
kubectl delete pod print-envars-greeting
```

Or:

```bash
kubectl delete -f day57-pod.yaml
```
