# DevOps Day 58 — Commands

This document contains the commands used to deploy and verify Grafana on Kubernetes.

---

## 1. Verify Kubernetes Connectivity

```bash
kubectl get nodes
```

Check existing resources:

```bash
kubectl get deployments
kubectl get pods
kubectl get svc
```

---

## 2. Create the Grafana Manifest

Create the manifest file:

```bash
vi grafana.yaml
```

Manifest:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-deployment-nautilus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana-nautilus
  template:
    metadata:
      labels:
        app: grafana-nautilus
    spec:
      containers:
        - name: grafana
          image: grafana/grafana:latest
          ports:
            - containerPort: 3000

---
apiVersion: v1
kind: Service
metadata:
  name: grafana-service-nautilus
spec:
  type: NodePort
  selector:
    app: grafana-nautilus
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      nodePort: 32000
```

---

## 3. Apply the Manifest

```bash
kubectl apply -f grafana.yaml
```

---

## 4. Verify Deployment Rollout

```bash
kubectl rollout status deployment/grafana-deployment-nautilus
```

Check Deployment details:

```bash
kubectl get deployment grafana-deployment-nautilus
```

Detailed Deployment information:

```bash
kubectl describe deployment grafana-deployment-nautilus
```

---

## 5. Verify Grafana Pod

```bash
kubectl get pods -l app=grafana-nautilus
```

Show additional pod information:

```bash
kubectl get pods -l app=grafana-nautilus -o wide
```

If troubleshooting is required:

```bash
kubectl describe pod -l app=grafana-nautilus
```

Check container logs:

```bash
kubectl logs -l app=grafana-nautilus
```

---

## 6. Verify NodePort Service

```bash
kubectl get svc grafana-service-nautilus
```

Detailed service information:

```bash
kubectl describe svc grafana-service-nautilus
```

The important mapping should be:

```text
3000:32000/TCP
```

---

## 7. Verify Service Endpoints

```bash
kubectl get endpoints grafana-service-nautilus
```

The endpoint should point to the Grafana pod IP on port `3000`.

---

## 8. Verify All Resources

```bash
kubectl get all
```

Filter Grafana-related resources if required:

```bash
kubectl get all -l app=grafana-nautilus
```

---

## 9. Test Service Internally

Get the Service ClusterIP:

```bash
kubectl get svc grafana-service-nautilus
```

Test Grafana through the Service:

```bash
curl -I http://$(kubectl get svc grafana-service-nautilus -o jsonpath='{.spec.clusterIP}'):3000/login
```

---

## 10. Test NodePort

Get node information:

```bash
kubectl get nodes -o wide
```

Store the node IP:

```bash
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
```

Display it:

```bash
echo "$NODE_IP"
```

Test Grafana on NodePort `32000`:

```bash
curl -I http://$NODE_IP:32000/login
```

---

## Useful Troubleshooting Commands

Check pod status:

```bash
kubectl get pods
```

Check recent cluster events:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Inspect the Service selector:

```bash
kubectl get svc grafana-service-nautilus -o yaml
```

Inspect pod labels:

```bash
kubectl get pods --show-labels
```

Check the manifest after deployment:

```bash
kubectl get deployment grafana-deployment-nautilus -o yaml
kubectl get svc grafana-service-nautilus -o yaml
```

---

## Cleanup Commands

These commands are not required for the challenge, but can be used to remove the created resources:

```bash
kubectl delete -f grafana.yaml
```

Or individually:

```bash
kubectl delete deployment grafana-deployment-nautilus
kubectl delete service grafana-service-nautilus
```
