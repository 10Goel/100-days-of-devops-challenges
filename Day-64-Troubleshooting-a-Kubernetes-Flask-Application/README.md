# DevOps Day 64 - Troubleshooting a Kubernetes Flask Application

## Overview

In this challenge, a Python Flask application was already deployed on a Kubernetes cluster, but it was not accessible because of multiple configuration issues in the Deployment and Service.

The objective was to troubleshoot the existing Kubernetes resources, identify the actual failures, correct them without unnecessarily recreating the application, and make the application reachable through the required NodePort.

---

## Task Requirements

The required final configuration was:

- **Deployment:** `python-deployment-xfusion`
- **Container image:** `poroko/flask-demo-app`
- **Application port:** Flask default port `5000`
- **Service type:** `NodePort`
- **Required NodePort:** `32345`
- **Required targetPort:** `5000`

The Deployment and Service already existed, so the main focus was troubleshooting and correcting the misconfiguration.

---

## Problems Identified

### 1. Incorrect Container Image

The existing Deployment referenced an invalid image, which caused the Pod to enter:

```text
ErrImagePull
ImagePullBackOff
```

The Kubernetes events showed an image pull failure similar to:

```text
pull access denied, repository does not exist or may require authorization
```

The correct image required by the task was:

```text
poroko/flask-demo-app
```

The Deployment image was corrected with `kubectl set image`.

---

### 2. Pod Could Not Start

Because Kubernetes could not pull the incorrect image, the Deployment never reached the required availability.

The Pod repeatedly moved between:

```text
ErrImagePull
ImagePullBackOff
```

After fixing the image, Kubernetes created a new ReplicaSet and a new Pod, which successfully reached:

```text
1/1 Running
```

---

### 3. Service targetPort Was Incorrect

The existing NodePort Service already had the correct external NodePort:

```text
32345
```

However, its `targetPort` was configured as:

```text
8080
```

while the Flask application was listening on:

```text
5000
```

This caused traffic to be forwarded to the wrong container port.

The Service was patched so that:

```text
NodePort   : 32345
ServicePort: 8080
TargetPort : 5000
```

The service port itself did not need to be changed because the task specifically required the NodePort and targetPort.

---

## Final Request Flow

```text
Client
  |
  | NodeIP:32345
  v
+--------------------------+
| Kubernetes NodePort      |
| Service                  |
|                          |
| port:       8080         |
| nodePort:   32345        |
| targetPort: 5000         |
+------------+-------------+
             |
             | forwards traffic
             v
+--------------------------+
| Flask Pod                |
|                          |
| Image:                   |
| poroko/flask-demo-app    |
|                          |
| Application listens on   |
| TCP/5000                 |
+--------------------------+
```

---

## Troubleshooting Workflow Used

The troubleshooting process followed an important Kubernetes principle:

> Diagnose the resource state first, then fix only the failing layer.

The sequence was:

1. Inspect the Deployment.
2. Inspect Pod status.
3. Read Pod events.
4. Identify the image pull failure.
5. Correct the Deployment image.
6. Verify the rollout.
7. Inspect the Service.
8. Compare Service `targetPort` with the application's actual listening port.
9. Patch the Service.
10. Verify endpoints and final connectivity.

This approach prevented unnecessary deletion and recreation of healthy resources.

---

## Key Verification Commands

```bash
kubectl get deployment python-deployment-xfusion
kubectl get pods
kubectl get svc python-service-xfusion
kubectl describe svc python-service-xfusion
kubectl get endpoints python-service-xfusion
kubectl rollout status deployment/python-deployment-xfusion
```

---

## Final Working State

```text
Deployment : python-deployment-xfusion
Image      : poroko/flask-demo-app
Pod        : 1/1 Running

Service    : python-service-xfusion
Type       : NodePort
Port       : 8080
TargetPort : 5000
NodePort   : 32345
```

The application became accessible on the required NodePort and the challenge was completed successfully.

---

## Concepts Practiced

- Kubernetes Deployment troubleshooting
- Pod lifecycle diagnosis
- `ErrImagePull` and `ImagePullBackOff`
- Kubernetes Events
- Container image troubleshooting
- Deployment rolling updates
- ReplicaSet replacement
- Kubernetes Services
- NodePort Services
- `port` vs `targetPort` vs `nodePort`
- Service selectors and endpoints
- Application-to-Service port mapping
- `kubectl describe`
- `kubectl get events`
- `kubectl set image`
- `kubectl patch`
- Rollout verification
- Layer-by-layer Kubernetes debugging

---

## Learning Outcome

This challenge demonstrated that Kubernetes troubleshooting should be performed systematically rather than by immediately recreating resources.

A failing application can involve different layers:

```text
Deployment
   ↓
ReplicaSet
   ↓
Pod
   ↓
Container
   ↓
Application Port
   ↓
Service
   ↓
NodePort
```

The most effective approach is to identify exactly where this chain breaks and fix only that layer.
