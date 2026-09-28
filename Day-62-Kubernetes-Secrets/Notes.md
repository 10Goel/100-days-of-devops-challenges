# Day 62 — Kubernetes Secrets Notes

## 1. What Is a Kubernetes Secret?

A Kubernetes Secret is an API object used to store and distribute sensitive configuration such as:

- Passwords
- API keys
- Tokens
- Certificates
- License keys
- Credentials

Secrets allow sensitive values to be separated from application code and container images.

---

## 2. Why Not Put Sensitive Data Directly in a Pod Manifest?

A Pod manifest may be:

- Stored in Git
- Shared with multiple team members
- Included in CI/CD pipelines
- Viewed through Kubernetes API access

Hardcoding sensitive values inside YAML therefore increases the risk of accidental exposure.

A Secret allows the workload definition and the confidential data to remain separate.

---

## 3. Generic Kubernetes Secrets

A generic Secret can be created using:

```bash
kubectl create secret generic <secret-name>
```

For this challenge:

```bash
kubectl create secret generic blog   --from-file=blog.txt=/opt/blog.txt
```

The resulting Secret contains:

```text
Secret name: blog
Key:         blog.txt
Value:       content of /opt/blog.txt
```

---

## 4. Understanding `--from-file`

The syntax:

```bash
--from-file=blog.txt=/opt/blog.txt
```

means:

```text
blog.txt        -> key stored in the Secret
/opt/blog.txt   -> source file on the machine running kubectl
```

Therefore, the Secret contains a key called:

```text
blog.txt
```

When mounted as a volume, that key becomes a filename.

---

## 5. Secret Mounted as a Volume

The Pod references the Secret using:

```yaml
volumes:
  - name: blog-secret-volume
    secret:
      secretName: blog
```

This tells Kubernetes:

> Create a volume whose contents come from the Secret named `blog`.

The container then mounts that volume:

```yaml
volumeMounts:
  - name: blog-secret-volume
    mountPath: /opt/games
    readOnly: true
```

The result is:

```text
Secret key: blog.txt

        becomes

/opt/games/blog.txt
```

---

## 6. Secret-to-File Mapping

Conceptually:

```text
Secret
└── blog.txt
      |
      v
Secret Volume
      |
      v
Container filesystem
└── /opt/games/blog.txt
```

Each Secret key becomes a file under the mounted directory.

The contents of that file are the decoded Secret value.

---

## 7. Why `readOnly: true`?

Secrets are configuration data and should normally not be modified by the application.

Using:

```yaml
readOnly: true
```

makes the intention explicit:

```text
Application consumes the secret
Application should not modify the secret
```

Kubernetes Secret volume mounts are effectively managed by Kubernetes rather than being ordinary writable application storage.

---

## 8. Why Was `sleep` Required?

A container exits when its main process exits.

The Fedora image does not automatically run a long-lived application for this task, so a command such as:

```bash
sleep infinity
```

keeps the container alive.

This allows the Pod to stay in:

```text
Running
```

and makes it possible to inspect the mounted Secret using:

```bash
kubectl exec
```

---

## 9. Kubernetes Secret Data and Base64

When inspecting a Secret using:

```bash
kubectl get secret blog -o yaml
```

the data may appear similar to:

```yaml
data:
  blog.txt: <base64-value>
```

Kubernetes represents Secret data using Base64 encoding.

Important:

```text
Base64 encoding != encryption
```

Base64 only converts binary data into a textual representation. Anyone with sufficient Kubernetes permissions to read the Secret can generally retrieve and decode the value.

---

## 10. Secret Security in Real Environments

Kubernetes Secrets improve separation of sensitive configuration, but production security requires additional controls such as:

- RBAC
- Least-privilege access
- Encryption at rest for Kubernetes Secret data
- Secure etcd configuration
- External secret-management systems when appropriate
- Avoiding secret values in logs
- Avoiding accidental commits of secret material to Git

A Secret object alone should not be treated as complete secret-management security.

---

## 11. Secrets vs ConfigMaps

| Feature | Secret | ConfigMap |
|---|---|---|
| Intended for sensitive data | Yes | No |
| Common use | Passwords, tokens, certificates | Application configuration |
| Can be mounted as files | Yes | Yes |
| Can be exposed as environment variables | Yes | Yes |
| Values shown in API representation | Base64-encoded | Plain configuration values |

Use a **Secret** for confidential information and a **ConfigMap** for non-sensitive configuration.

---

## 12. Secrets as Environment Variables

Secrets can also be consumed through environment variables.

Example:

```yaml
env:
  - name: LICENSE_KEY
    valueFrom:
      secretKeyRef:
        name: blog
        key: blog.txt
```

This challenge specifically required a **volume mount**, not an environment variable.

---

## 13. Pod Configuration Used in This Challenge

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

## 14. Important Relationships to Remember

```text
Secret name
    ↓
secretName: blog
    ↓
Volume
    ↓
volumeMounts.name
    ↓
Container mountPath
```

For this task:

```text
Secret: blog
    ↓
Volume: blog-secret-volume
    ↓
Container: secret-container-datacenter
    ↓
Mount: /opt/games
    ↓
File: /opt/games/blog.txt
```

The value of:

```yaml
volumeMounts:
  - name: blog-secret-volume
```

must match:

```yaml
volumes:
  - name: blog-secret-volume
```

Otherwise the Pod specification is invalid.

---

## 15. Verification Strategy

A reliable Kubernetes verification flow is:

```text
1. Check that the Secret exists
2. Check that the Pod exists
3. Confirm that the Pod is Running
4. Inspect the Pod specification/events
5. Exec into the container
6. Verify the mounted file
7. Confirm the file contains the expected secret value
```

Commands:

```bash
kubectl get secret blog
kubectl get pod secret-datacenter
kubectl describe pod secret-datacenter
kubectl exec secret-datacenter   -c secret-container-datacenter --   cat /opt/games/blog.txt
```

---

## 16. Common Mistakes

### Wrong Secret Name

```yaml
secretName: blogs
```

when the actual Secret is:

```text
blog
```

The Pod will not be able to mount the intended Secret.

### Mismatched Volume Names

Incorrect:

```yaml
volumeMounts:
  - name: secret-volume
```

```yaml
volumes:
  - name: blog-secret-volume
```

Both names must match.

### Wrong Mount Path

The task specifically required:

```text
/opt/games
```

Mounting somewhere else would not satisfy the requirement.

### Container Exits Immediately

Without a long-running process, the Fedora container may terminate.

Using:

```bash
sleep infinity
```

keeps the container alive.

### Confusing Base64 With Encryption

Kubernetes Secret data is commonly represented as Base64 in YAML. Base64 is encoding, not encryption.

---

## 17. Core Takeaway

The most important concept from Day 62 is:

```text
Sensitive information should be separated from the application image
and injected into the workload securely at runtime.
```

For this challenge:

```text
/opt/blog.txt
      ↓
Kubernetes Secret: blog
      ↓
Secret-backed Volume
      ↓
Pod: secret-datacenter
      ↓
/opt/games/blog.txt
```

This is a fundamental Kubernetes pattern for supplying sensitive configuration to containerized applications.
