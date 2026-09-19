# DevOps Challenge – Day 53  
## Kubernetes Nginx + PHP-FPM Troubleshooting Notes

## 1. Challenge Objective

The challenge involved troubleshooting a multi-container Kubernetes Pod containing:

```text
Nginx
+
PHP-FPM
```

The Pod was:

```text
nginx-phpfpm
```

and its Nginx configuration was provided through:

```text
nginx-config
```

Both containers were running, but the application was unavailable because their filesystem paths were inconsistent.

---

## 2. `Running` Does Not Mean the Application Works

A Pod may show:

```text
2/2 Running
```

while the application still fails.

Kubernetes can confirm that the container processes are alive, but it does not automatically guarantee that:

- Nginx uses the correct document root.
- PHP-FPM can access the required PHP script.
- Volume mounts match application expectations.
- Configuration files are logically correct.
- The website can successfully serve requests.

Therefore:

```text
Container health != Application correctness
```

---

## 3. Multi-Container Pod Basics

The Pod contained:

```text
nginx-container
php-fpm-container
```

Containers inside the same Pod share:

- The same Pod IP
- The same network namespace
- `localhost`
- Volumes that are explicitly mounted into them

They do **not** automatically share their root filesystems.

---

## 4. Nginx and PHP-FPM Architecture

Nginx handles HTTP traffic.

For static files:

```text
Client
  |
  v
Nginx
  |
  v
Static content
```

For PHP requests:

```text
Client
  |
  v
Nginx
  |
  v
PHP request
  |
  v
PHP-FPM
  |
  v
PHP execution
  |
  v
Response
```

Nginx does not execute PHP itself. It forwards PHP requests to PHP-FPM.

---

## 5. Why Both Containers Need the PHP File

Suppose the browser requests:

```text
/index.php
```

If Nginx is configured with:

```nginx
root /var/www/html;
```

then it resolves the file to:

```text
/var/www/html/index.php
```

PHP-FPM must also be able to access that same script path.

If one container sees the file somewhere else, the request flow breaks.

---

## 6. Root Cause of Day 53

Before the fix:

```text
PHP-FPM:
shared-files -> /var/www/html

Nginx:
shared-files -> /usr/share/nginx/html
```

But Nginx was configured with:

```nginx
root /var/www/html;
```

Therefore:

```text
Nginx expects:
    /var/www/html/index.php

Shared file is exposed to Nginx at:
    /usr/share/nginx/html/index.php
```

The paths did not match.

---

## 7. Kubernetes Volumes

A Pod-level volume is defined under:

```yaml
volumes:
```

Example:

```yaml
volumes:
  - name: shared-files
    emptyDir: {}
```

A container accesses that volume through:

```yaml
volumeMounts:
```

Example:

```yaml
volumeMounts:
  - name: shared-files
    mountPath: /var/www/html
```

A volume exists at the Pod level, but each container decides where that volume appears inside its own filesystem.

---

## 8. `emptyDir`

An `emptyDir` volume is created when the Pod starts.

Conceptually:

```text
Pod created
   |
   v
emptyDir created
   |
   v
Containers mount it
   |
   v
Containers share data
   |
   v
Pod deleted
   |
   v
emptyDir data removed
```

Typical uses:

- Temporary shared files
- Cache
- Scratch data
- Inter-container file exchange

It is not persistent storage.

---

## 9. Can the Same Volume Use Different Mount Paths?

Yes.

This is valid:

```text
Container A:
shared-volume -> /data

Container B:
shared-volume -> /shared
```

Both paths still refer to the same underlying volume.

The Day 53 problem was not simply that the paths were different. The real issue was that the Nginx configuration explicitly expected:

```text
/var/www/html
```

while Nginx mounted the shared data elsewhere.

---

## 10. Corrected Architecture

After the fix:

```text
                     shared-files
                      emptyDir
                         |
              +----------+----------+
              |                     |
              v                     v
       nginx-container       php-fpm-container
       /var/www/html         /var/www/html
              |                     |
              +----------+----------+
                         |
                     index.php
```

Both processes now agreed on the application path.

---

## 11. ConfigMaps

A ConfigMap stores non-sensitive application configuration.

In this challenge:

```text
nginx-config
```

contained the Nginx configuration.

It allowed the configuration to be separated from the container image.

Example concept:

```text
ConfigMap
   |
   v
nginx.conf
   |
   v
Nginx container
```

---

## 12. ConfigMap Mounted as a File

A ConfigMap can be exposed inside a container using a volume.

Example:

```yaml
volumes:
  - name: nginx-config-volume
    configMap:
      name: nginx-config
```

and:

```yaml
volumeMounts:
  - name: nginx-config-volume
    mountPath: /etc/nginx/nginx.conf
    subPath: nginx.conf
```

---

## 13. What `subPath` Does

`subPath` allows a specific file from a volume to be mounted at a specific destination.

Conceptually:

```text
ConfigMap
   |
   +-- nginx.conf
          |
          | subPath
          v
/etc/nginx/nginx.conf
```

This is useful when the application expects a single configuration file at an exact path.

---

## 14. Why the Pod Had to Be Recreated

Many Pod specification fields are effectively immutable after a Pod is created.

Changing a volume mount path on an existing standalone Pod is not the normal workflow.

Therefore:

```text
Inspect Pod
   |
   v
Create corrected manifest
   |
   v
Delete old Pod
   |
   v
Create corrected Pod
```

In production, a higher-level controller such as a Deployment would usually manage Pod replacement.

---

## 15. `kubectl cp`

The challenge provided:

```text
/home/thor/index.php
```

The file was copied using:

```bash
kubectl cp /home/thor/index.php   nginx-phpfpm:/var/www/html/index.php   -c nginx-container
```

The `-c` option was important because the Pod contained multiple containers.

---

## 16. Why the File Appeared in Both Containers

`/var/www/html` was backed by the same shared volume.

Therefore:

```text
kubectl cp
    |
    v
nginx-container:/var/www/html/index.php
    |
    v
shared-files
    |
    +---------------------------+
    |                           |
    v                           v
Nginx sees file           PHP-FPM sees file
```

The file was written once into the shared volume.

---

## 17. `kubectl exec`

`kubectl exec` runs a command inside a container.

Example:

```bash
kubectl exec nginx-phpfpm -c nginx-container --   ls -l /var/www/html/
```

Breakdown:

```text
nginx-phpfpm       -> Pod name
-c nginx-container -> Container name
--                  -> End of kubectl options
ls -l ...           -> Command executed inside container
```

---

## 18. Why `-c` Matters

For multi-container Pods, always specify the target container when necessary.

Examples:

```bash
kubectl logs nginx-phpfpm -c nginx-container
```

```bash
kubectl exec nginx-phpfpm -c php-fpm-container -- <command>
```

```bash
kubectl cp <source> nginx-phpfpm:<destination> -c nginx-container
```

This removes ambiguity.

---

## 19. Shared Network vs Shared Filesystem

These concepts are different.

### Network

Containers inside the same Pod automatically share:

```text
Pod IP
localhost
network namespace
```

### Filesystem

Containers do not automatically share files.

Shared files require:

```text
Volume
+
volumeMount
```

---

## 20. Why `localhost` Works Between Nginx and PHP-FPM

Containers in the same Pod share one network namespace.

Therefore Nginx can forward PHP requests to something like:

```nginx
fastcgi_pass 127.0.0.1:9000;
```

even though PHP-FPM runs in a different container.

Conceptually:

```text
Pod network namespace
        |
        +-- Nginx
        |
        +-- PHP-FPM :9000
```

Both processes can communicate through `localhost`.

---

## 21. Nginx `root`

Example:

```nginx
root /var/www/html;
```

This determines the filesystem path Nginx uses for requested files.

For:

```text
/index.php
```

Nginx expects:

```text
/var/www/html/index.php
```

This path therefore had to align with the shared volume mount.

---

## 22. `SCRIPT_FILENAME`

A common Nginx PHP configuration contains:

```nginx
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
```

If:

```text
$document_root = /var/www/html
$fastcgi_script_name = /index.php
```

then:

```text
SCRIPT_FILENAME
=
/var/www/html/index.php
```

PHP-FPM tries to execute that exact path.

This is why the filesystem mapping mattered.

---

## 23. Troubleshooting Method

A strong troubleshooting flow for this type of issue is:

```text
Check Pod state
      |
      v
Inspect containers
      |
      v
Inspect volume mounts
      |
      v
Inspect application ConfigMap
      |
      v
Compare expected vs actual paths
      |
      v
Fix the manifest
      |
      v
Recreate the Pod
      |
      v
Verify files from both containers
      |
      v
Validate the application
```

---

## 24. Why `kubectl describe` Is Important

```bash
kubectl describe pod nginx-phpfpm
```

provides operational details including:

- Container state
- Container image
- Restart count
- Volume mounts
- Pod volumes
- Conditions
- Events

It is one of the first commands to use when troubleshooting Kubernetes workloads.

---

## 25. Why JSONPath Is Useful

Pod YAML can be large.

JSONPath allows you to extract only the fields you need.

Example:

```bash
kubectl get pod nginx-phpfpm   -o jsonpath='{range .spec.containers[*]}{.name}{"\n"}{range .volumeMounts[*]}{"  "}{.name}{" -> "}{.mountPath}{"\n"}{end}{end}'
```

This makes mount-path comparisons much easier.

---

## 26. Common Mistakes

### Assuming `2/2 Running` Means Success

Incorrect:

```text
Both containers are running
therefore
the website must work
```

Correct:

```text
Processes running
!=
Application functioning
```

### Looking Only at the Pod Manifest

The Pod showed that paths differed, but the ConfigMap revealed which path Nginx actually expected.

Always correlate:

```text
Kubernetes configuration
+
Application configuration
```

### Forgetting `-c`

For a multi-container Pod, specify the target container when using:

```text
kubectl logs
kubectl exec
kubectl cp
```

### Treating `emptyDir` as Persistent

Deleting the Pod destroys its `emptyDir` contents.

That is why the application file had to be copied again after Pod recreation.

---

## 27. Production Perspective

Manually copying application code into a Pod is generally not the preferred production model.

Normally, application code would be:

- Included in the container image, or
- Stored on persistent/shared storage when appropriate.

A Deployment would also usually manage the Pod.

This challenge intentionally uses manual Pod recreation and file copying to teach the underlying Kubernetes concepts.

---

## 28. Essential Commands to Remember

### Pod status

```bash
kubectl get pods
```

### Pod details

```bash
kubectl describe pod <pod-name>
```

### Pod YAML

```bash
kubectl get pod <pod-name> -o yaml
```

### ConfigMap YAML

```bash
kubectl get configmap <configmap-name> -o yaml
```

### Logs

```bash
kubectl logs <pod-name> -c <container-name>
```

### Execute inside a container

```bash
kubectl exec <pod-name> -c <container-name> -- <command>
```

### Copy a file

```bash
kubectl cp <source> <pod>:<destination> -c <container-name>
```

### Wait for Pod readiness

```bash
kubectl wait --for=condition=Ready pod/<pod-name> --timeout=120s
```

---

## 29. Final Mental Model

The entire issue can be summarized as:

```text
Nginx configuration expects:
    /var/www/html

PHP-FPM already uses:
    /var/www/html

Nginx shared volume was mounted at:
    /usr/share/nginx/html
```

Therefore:

```text
Nginx configuration
        !=
Nginx shared-volume path
```

Fix:

```text
Mount shared-files in Nginx at /var/www/html
```

Then:

```text
Nginx sees index.php
        |
        v
Nginx forwards PHP request
        |
        v
PHP-FPM sees the same index.php
        |
        v
PHP executes successfully
        |
        v
Website works
```

---

## 30. Key Takeaways

1. Multi-container Pods share networking but not filesystems automatically.
2. Shared files require Kubernetes volumes and `volumeMounts`.
3. `emptyDir` provides temporary Pod-scoped shared storage.
4. The same volume can technically be mounted at different paths.
5. Application configuration determines whether those paths are logically correct.
6. Nginx and PHP-FPM must agree on the path of the PHP script.
7. ConfigMaps allow configuration to be injected into containers.
8. `subPath` is useful for mounting one specific file.
9. A running Pod does not guarantee a working application.
10. `kubectl describe` is essential for troubleshooting.
11. `kubectl exec` helps validate the runtime filesystem.
12. `kubectl cp` can copy data into a selected container.
13. Files written into a shared volume are visible to every container mounting it.
14. `emptyDir` data is removed when the Pod is deleted.
15. Effective troubleshooting compares expected application configuration with actual runtime configuration.

---

## Challenge Summary

In Day 53, the `nginx-phpfpm` Pod contained both Nginx and PHP-FPM, but the application failed because the Nginx container mounted the shared application volume at a path that did not match the document root defined in `nginx-config`.

The problem was resolved by correcting the Nginx `shared-files` mount to `/var/www/html`, recreating the Pod, copying `index.php` into the shared directory, verifying the file from both containers, and validating the final Nginx configuration.

This challenge demonstrated how Kubernetes storage, multi-container networking, application configuration, and filesystem paths must work together for an application to function correctly.
