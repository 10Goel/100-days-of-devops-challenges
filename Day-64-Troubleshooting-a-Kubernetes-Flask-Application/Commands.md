# DevOps Day 64 - Commands

This file contains the commands used while troubleshooting the Kubernetes Flask application.

---

## 1. Inspect the Deployment

```bash
kubectl get deployment python-deployment-xfusion
```

```bash
kubectl describe deployment python-deployment-xfusion
```

```bash
kubectl get deployment python-deployment-xfusion -o yaml
```

---

## 2. Check the Configured Container Image

```bash
kubectl get deployment python-deployment-xfusion -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

This command reads the image directly from the Deployment's Pod template.

---

## 3. Inspect Pods

```bash
kubectl get pods
```

For continuous observation:

```bash
kubectl get pods -w
```

Exit watch mode with:

```text
Ctrl+C
```

---

## 4. Inspect Pod Failure Details

```bash
kubectl describe pod <pod-name>
```

Example:

```bash
kubectl describe pod python-deployment-xfusion-9759485b9-bg88v
```

The Events section exposed the image pull error.

---

## 5. Inspect Kubernetes Events

```bash
kubectl get events --sort-by=.lastTimestamp | tail -20
```

This helped identify:

```text
ErrImagePull
ImagePullBackOff
pull access denied
```

---

## 6. Fix the Deployment Image

Correct image:

```text
poroko/flask-demo-app
```

Command:

```bash
kubectl set image deployment/python-deployment-xfusion python-container-xfusion=poroko/flask-demo-app
```

---

## 7. Verify the Updated Image

```bash
kubectl get deployment python-deployment-xfusion -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

Expected output:

```text
poroko/flask-demo-app
```

---

## 8. Monitor the New Pod

```bash
kubectl get pods -w
```

Expected final state:

```text
1/1   Running
```

---

## 9. Verify Deployment Rollout

```bash
kubectl rollout status deployment/python-deployment-xfusion
```

Expected:

```text
deployment "python-deployment-xfusion" successfully rolled out
```

---

## 10. Inspect the Service

```bash
kubectl get svc python-service-xfusion
```

The service showed:

```text
8080:32345/TCP
```

which means:

```text
Service port = 8080
NodePort     = 32345
```

---

## 11. Describe the Service

```bash
kubectl describe svc python-service-xfusion
```

The incorrect configuration showed:

```text
TargetPort: 8080/TCP
NodePort:   32345/TCP
```

The required target port was `5000`.

---

## 12. Patch the Service

```bash
kubectl patch svc python-service-xfusion -p '{"spec":{"ports":[{"port":8080,"targetPort":5000,"nodePort":32345,"protocol":"TCP"}]}}'
```

This preserved:

```text
port     = 8080
nodePort = 32345
```

and corrected:

```text
targetPort = 5000
```

---

## 13. Verify Service Configuration

```bash
kubectl describe svc python-service-xfusion
```

Expected:

```text
Port:        8080/TCP
TargetPort:  5000/TCP
NodePort:    32345/TCP
```

---

## 14. Check Service Endpoints

```bash
kubectl get endpoints python-service-xfusion
```

Expected pattern:

```text
<pod-ip>:5000
```

Example:

```text
10.22.0.11:5000
```

---

## 15. Check Pod Labels

```bash
kubectl get pods --show-labels
```

Useful when debugging a Service with no endpoints.

---

## 16. Inspect the Service Selector

```bash
kubectl get svc python-service-xfusion -o jsonpath='{.spec.selector}{"\n"}'
```

The selector must match labels on the application Pod.

---

## 17. Check Application Logs

```bash
kubectl logs <pod-name>
```

Or:

```bash
kubectl logs $(kubectl get pods -o name | grep python-deployment | head -1)
```

Useful for confirming whether the application started successfully and which port it is listening on.

---

## 18. Test Through the ClusterIP

First obtain the Service information:

```bash
kubectl get svc python-service-xfusion
```

Then test:

```bash
curl http://<CLUSTER-IP>:8080
```

---

## 19. Get Node IP

```bash
kubectl get nodes -o wide
```

---

## 20. Test the Required NodePort

```bash
curl http://<NODE-IP>:32345
```

A successful application response confirms the full path:

```text
Client -> NodePort -> Service -> Pod -> Flask application
```

---

## Final Verification Set

```bash
kubectl get deployment python-deployment-xfusion
kubectl get pods
kubectl get svc python-service-xfusion
kubectl describe svc python-service-xfusion
kubectl get endpoints python-service-xfusion
kubectl rollout status deployment/python-deployment-xfusion
```
