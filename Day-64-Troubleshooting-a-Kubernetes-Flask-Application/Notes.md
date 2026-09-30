# DevOps Day 64 - Troubleshooting Notes

## 1. Kubernetes Troubleshooting Should Be Layered

When an application is unavailable, do not immediately delete and recreate everything.

Troubleshoot the request path layer by layer:

```text
Deployment
   ↓
ReplicaSet
   ↓
Pod
   ↓
Container
   ↓
Application
   ↓
Service
   ↓
NodePort
```

Find the first broken layer and fix that layer.

---

# 2. Deployment vs ReplicaSet vs Pod

A Kubernetes Deployment manages ReplicaSets.

A ReplicaSet ensures the requested number of Pods exist.

The hierarchy is:

```text
Deployment
   |
   +--> ReplicaSet
           |
           +--> Pod
```

When the Deployment's Pod template changes, for example when the container image changes, Kubernetes normally creates a new ReplicaSet and gradually replaces Pods from the old ReplicaSet.

This is why running:

```bash
kubectl set image
```

can cause a new Pod with a different generated name to appear.

---

# 3. Understanding `ErrImagePull`

`ErrImagePull` means Kubernetes tried to download the container image but the pull operation failed.

Common causes include:

- Incorrect repository name
- Incorrect image name
- Incorrect image tag
- Private registry without authentication
- Registry unavailable
- Image no longer exists

Example:

```text
ErrImagePull
```

is usually accompanied by useful detail in:

```bash
kubectl describe pod <pod-name>
```

---

# 4. Understanding `ImagePullBackOff`

`ImagePullBackOff` does not mean Kubernetes permanently stopped trying.

It means:

```text
Image pull failed
      ↓
Kubernetes waits
      ↓
Retries
      ↓
Failure
      ↓
Waits longer
```

Kubernetes uses a backoff mechanism to avoid continuously hammering the registry.

Therefore:

```text
ErrImagePull
```

and:

```text
ImagePullBackOff
```

are closely related states.

The real root cause should be found in Pod Events.

---

# 5. Why `kubectl describe` Is So Important

`kubectl get` gives a summary.

Example:

```bash
kubectl get pods
```

may only show:

```text
ImagePullBackOff
```

But:

```bash
kubectl describe pod <pod-name>
```

shows the Events section where the actual reason may be visible:

```text
pull access denied
repository does not exist
authorization failed
```

A useful troubleshooting rule is:

```text
get      -> What is happening?
describe -> Why is it happening?
logs     -> What is the application saying?
```

---

# 6. Kubernetes Events

Events often reveal failures that are not obvious from the resource state.

Useful command:

```bash
kubectl get events --sort-by=.lastTimestamp
```

Recent events can be viewed with:

```bash
kubectl get events --sort-by=.lastTimestamp | tail -20
```

Events are especially useful for:

- Scheduling failures
- Image pull errors
- Volume mount errors
- Probe failures
- Failed container starts
- Service/endpoint issues

---

# 7. Fixing a Deployment Image

Instead of editing the full Deployment YAML, a container image can be updated directly:

```bash
kubectl set image deployment/<deployment-name> <container-name>=<image>
```

Example:

```bash
kubectl set image deployment/python-deployment-xfusion python-container-xfusion=poroko/flask-demo-app
```

This modifies the Pod template and triggers a rolling update.

---

# 8. Deployment Rollout

After changing the Deployment, verify the rollout:

```bash
kubectl rollout status deployment/python-deployment-xfusion
```

Successful output:

```text
deployment "python-deployment-xfusion" successfully rolled out
```

If the rollout hangs or exceeds its progress deadline, inspect the Pods rather than repeatedly running the rollout command.

For example:

```bash
kubectl get pods
```

followed by:

```bash
kubectl describe pod <pod-name>
```

---

# 9. Why a Running Pod Does Not Guarantee a Working Application

A Pod showing:

```text
1/1 Running
```

only confirms that the container is running.

It does not automatically prove:

- The Service selector is correct
- The Service points to the correct port
- The application responds successfully
- NodePort traffic reaches the Pod

Therefore Service-level verification is still required.

---

# 10. Kubernetes Service Port Concepts

The most important networking concept from this challenge is the difference between:

```text
port
targetPort
nodePort
```

For the final application:

```text
NodePort   = 32345
ServicePort= 8080
TargetPort = 5000
```

---

## `port`

`port` is the port exposed by the Service inside the Kubernetes cluster.

Example:

```yaml
port: 8080
```

Other Pods can reach the Service through:

```text
<Service-IP>:8080
```

---

## `targetPort`

`targetPort` is the port on the destination Pod/container where the application is actually listening.

Example:

```yaml
targetPort: 5000
```

For this challenge, Flask was listening on:

```text
5000
```

Therefore the Service had to forward traffic to port `5000`.

---

## `nodePort`

`nodePort` exposes the Service on a port of each Kubernetes node.

Example:

```yaml
nodePort: 32345
```

External request:

```text
<Node-IP>:32345
```

---

# 11. Complete NodePort Traffic Flow

For this challenge:

```text
User
 |
 | NodeIP:32345
 v
Kubernetes Node
 |
 | NodePort 32345
 v
NodePort Service
 |
 | Service port 8080
 v
targetPort 5000
 |
 v
Pod
 |
 v
Flask application
```

The Service can expose one port while forwarding to a completely different application port.

That is why:

```text
port: 8080
targetPort: 5000
```

is valid.

---

# 12. Why the Original Service Failed

The Service originally had:

```text
Port:       8080
TargetPort: 8080
NodePort:   32345
```

But the application was listening on:

```text
5000
```

So Kubernetes forwarded packets to:

```text
PodIP:8080
```

where the Flask application was not listening.

Correct mapping:

```text
NodePort 32345
   ↓
Service port 8080
   ↓
targetPort 5000
   ↓
Flask application
```

---

# 13. Service Selectors

A Service does not automatically know which Pod to send traffic to.

It uses labels.

Example Pod:

```yaml
metadata:
  labels:
    app: python_app
```

Service:

```yaml
spec:
  selector:
    app: python_app
```

These must match.

If the Service selector does not match any Pods, the Service can exist normally but will have no backend endpoints.

---

# 14. Endpoints Are One of the Best Service Troubleshooting Checks

Command:

```bash
kubectl get endpoints <service-name>
```

Healthy example:

```text
10.22.0.11:5000
```

This tells us:

- The Service found a matching Pod
- Kubernetes resolved the Pod IP
- Traffic will be forwarded to port `5000`

Problem example:

```text
<none>
```

Possible causes:

- Service selector does not match Pod labels
- Pod is not ready
- No matching Pods exist

---

# 15. Endpoint Port Comes From `targetPort`

If the Service has:

```yaml
targetPort: 5000
```

the endpoint will typically appear as:

```text
PodIP:5000
```

For example:

```text
10.22.0.11:5000
```

This is a very useful way to confirm whether the Service is targeting the intended application port.

---

# 16. `containerPort` vs `targetPort`

A Pod specification may contain:

```yaml
ports:
- containerPort: 5000
```

This mainly documents which port the container is expected to use.

It does not itself expose the application outside the Pod.

The Service actually routes traffic using:

```yaml
targetPort: 5000
```

Important distinction:

```text
containerPort -> container metadata/documentation
targetPort    -> Service forwarding destination
```

The application itself must genuinely be listening on that port.

---

# 17. `kubectl get pods -w`

Watch mode:

```bash
kubectl get pods -w
```

is useful during:

- Deployment rollouts
- Pod recreation
- Image updates
- Crash troubleshooting
- Scheduling

Typical state progression:

```text
Pending
ContainerCreating
Running
```

For image failures:

```text
ErrImagePull
ImagePullBackOff
```

Exit with:

```text
Ctrl+C
```

---

# 18. Why Old Pods May Still Appear Temporarily

During a rolling update, both old and new ReplicaSet Pods can appear for a short period.

Example:

```text
old-pod    ImagePullBackOff
new-pod    ContainerCreating
```

Kubernetes gradually transitions from the old ReplicaSet to the new one.

Do not assume every visible old Pod must be manually deleted.

Verify the Deployment rollout instead.

---

# 19. Logs vs Events

These solve different problems.

## Logs

```bash
kubectl logs <pod-name>
```

Used when the container successfully starts and the application produces output.

Useful for:

- Application exceptions
- Flask startup messages
- Runtime failures
- Database connection errors

## Events

```bash
kubectl describe pod <pod-name>
```

Used for Kubernetes-level problems.

Useful for:

- Image pull failures
- Scheduling failures
- Volume problems
- Probe failures

If the container never starts, logs may provide little or nothing, while Events are often much more useful.

---

# 20. A Strong Kubernetes Debugging Sequence

A reusable troubleshooting sequence is:

```bash
kubectl get deployments
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get events --sort-by=.lastTimestamp
kubectl get svc
kubectl describe svc <service>
kubectl get endpoints <service>
```

Conceptually:

```text
1. Is the desired workload defined correctly?
2. Did Kubernetes create a Pod?
3. Is the Pod running?
4. If not, why?
5. Is the application running inside it?
6. Does the Service select the Pod?
7. Is the Service forwarding to the correct targetPort?
8. Is the NodePort exposed correctly?
9. Does an actual request succeed?
```

---

# 21. Troubleshoot From the Inside Out

A useful strategy for Kubernetes networking issues is:

```text
Application
   ↓
Pod
   ↓
Service
   ↓
NodePort
   ↓
External Client
```

Verify the innermost layer first.

For example:

```text
Is Flask running?
      ↓
Can the Service reach the Pod?
      ↓
Does the Service have endpoints?
      ↓
Is NodePort correct?
      ↓
Can the client connect?
```

This is much easier than debugging all networking layers simultaneously.

---

# 22. Final Key Lessons

### Lesson 1

Do not treat:

```text
ImagePullBackOff
```

as the root cause.

It is a symptom.

Inspect Events to find the actual pull failure.

---

### Lesson 2

A Pod being `Running` does not prove the Service configuration is correct.

---

### Lesson 3

Always understand:

```text
port
targetPort
nodePort
```

before troubleshooting a Kubernetes Service.

---

### Lesson 4

A Service with no endpoints usually indicates a Pod-selection/readiness problem.

---

### Lesson 5

A Service with an endpoint on the wrong port usually indicates an incorrect `targetPort`.

---

### Lesson 6

Change only what is wrong.

The Deployment and Service already existed, so patching the broken configuration was safer and cleaner than recreating all resources.

---

# Final Mental Model

For this challenge, the complete troubleshooting chain was:

```text
Wrong image
   ↓
ErrImagePull
   ↓
ImagePullBackOff
   ↓
Inspect Events
   ↓
Correct image
   ↓
Deployment rolls out
   ↓
Pod becomes Running
   ↓
Inspect Service
   ↓
NodePort correct
   ↓
targetPort incorrect
   ↓
Change targetPort to 5000
   ↓
Endpoint becomes PodIP:5000
   ↓
Application accessible through NodePort 32345
```

This layered approach is one of the most important habits for real Kubernetes troubleshooting.
