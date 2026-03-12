# Kubernetes Demo: MongoDB + Mongo Express

This repository is a practical Kubernetes lab for deploying MongoDB and Mongo Express and validating service exposure patterns.

The documentation below is aligned with Kubernetes official concepts for Deployments, Services, ConfigMaps, Secrets, and Ingress.

## What This Project Demonstrates

- Internal stateful-style workload access using a Service (`mongodb-service`).
- External web access patterns for the same app family:
  - Service `type: LoadBalancer` (with a fixed `nodePort`) in `mongo-express.yaml`.
  - Ingress routing in `mongo-express-2.yaml`.
- Config separation using ConfigMap (`mongodb-config`).
- Sensitive value injection using Secret (`mongodb-secret`).

## Architecture

Two access paths are intentionally provided so you can study different exposure models.

```text
Path A (LoadBalancer / NodePort)
Browser --> mongo-express-service (LoadBalancer, nodePort 30000)
        --> mongo-express Pod --> mongodb-service:27017 --> mongodb Pod

Path B (Ingress)
Browser --> Ingress host mongo-express-2.com
        --> mongo-express-2-service:8081
        --> mongo-express-2 Pod --> mongodb-service:27017 --> mongodb Pod

Shared configuration
- Secret: mongodb-secret (admin username/password)
- ConfigMap: mongodb-config (database_url=mongodb-service)
```

## Repository Structure

| File                   | Purpose                                                 | Main Kubernetes Objects            |
| ---------------------- | ------------------------------------------------------- | ---------------------------------- |
| `mongo-secret.yaml`    | Stores MongoDB credentials (base64-encoded)             | `Secret`                           |
| `mongo-configmap.yaml` | Stores MongoDB service hostname                         | `ConfigMap`                        |
| `mongo.yaml`           | Deploys MongoDB and internal access                     | `Deployment`, `Service`            |
| `mongo-express.yaml`   | Deploys Mongo Express exposed via LoadBalancer/NodePort | `Deployment`, `Service`            |
| `mongo-express-2.yaml` | Deploys a second Mongo Express exposed via Ingress      | `Deployment`, `Service`, `Ingress` |

## Prerequisites

- Kubernetes cluster (Minikube recommended for local study).
- `kubectl` configured to your cluster context.
- `minikube` installed (for local setup examples).
- Ingress controller enabled when using `mongo-express-2.yaml`.

Example local setup:

```bash
minikube start
kubectl cluster-info
kubectl get nodes
```

## Deployment

Apply resources in dependency order so consumers start after configuration is available.

```bash
kubectl apply -f mongo-secret.yaml
kubectl apply -f mongo-configmap.yaml
kubectl apply -f mongo.yaml

# Choose one or both exposure options:
kubectl apply -f mongo-express.yaml
kubectl apply -f mongo-express-2.yaml
```

Wait for rollouts:

```bash
kubectl rollout status deployment/mongodb-deployment
kubectl rollout status deployment/mongo-express-deployment
kubectl rollout status deployment/mongo-express-2-deployment
```

## Access Examples

### Option A: LoadBalancer / NodePort (`mongo-express.yaml`)

For Minikube, use one of these approaches:

1. Direct service URL helper:

```bash
minikube service mongo-express-service --url
```

2. Tunnel for LoadBalancer IP allocation:

```bash
minikube tunnel
kubectl get svc mongo-express-service
```

Then open the returned URL in your browser.

### Option B: Ingress (`mongo-express-2.yaml`)

Enable ingress and map the host locally.

```bash
minikube addons enable ingress
kubectl apply -f mongo-express-2.yaml
kubectl get ingress
```

Get the Minikube IP:

```bash
minikube ip
```

Add host mapping (example):

```text
<MINIKUBE_IP> mongo-express-2.com
```

Then open:

```text
http://mongo-express-2.com
```

## Verification and Day-2 Operations

Cluster inventory:

```bash
kubectl get all
kubectl get configmap mongodb-config
kubectl get secret mongodb-secret
kubectl get ingress
```

Inspect runtime behavior:

```bash
kubectl describe deployment mongodb-deployment
kubectl describe deployment mongo-express-deployment
kubectl logs deployment/mongodb-deployment
kubectl logs deployment/mongo-express-deployment
```

Validate service endpoints:

```bash
kubectl get svc mongodb-service mongo-express-service mongo-express-2-service
kubectl get endpoints mongodb-service mongo-express-service mongo-express-2-service
```

Check Secret values safely (for troubleshooting only):

```bash
kubectl get secret mongodb-secret -o jsonpath='{.data.mongodb-root-username}' | base64 --decode; echo
kubectl get secret mongodb-secret -o jsonpath='{.data.mongodb-root-password}' | base64 --decode; echo
```

## Troubleshooting Runbook

### 1) Mongo Express cannot connect to MongoDB

- Verify MongoDB Pod is running:

```bash
kubectl get pods -l app=mongodb
kubectl logs deployment/mongodb-deployment
```

- Verify service discovery name in ConfigMap:

```bash
kubectl get configmap mongodb-config -o yaml
```

- Verify Secret key names match deployment env references:
  - `mongodb-root-username`
  - `mongodb-root-password`

### 2) UI is not reachable from browser

- For LoadBalancer path, run `minikube service ... --url` or `minikube tunnel`.
- For Ingress path, ensure ingress addon is enabled and `/etc/hosts` contains `mongo-express-2.com`.

### 3) Pod restart loops

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name> --previous
kubectl get events --sort-by=.metadata.creationTimestamp
```

## Security and Production Notes

This repository is intentionally simple for study purposes. For production-grade deployments:

- Do not commit real credentials in Git.
- Remember: Kubernetes Secrets are base64-encoded by default, not encrypted by themselves.
- Use encrypted secret management (for example, External Secrets + cloud secret manager, or Sealed Secrets).
- Pin image tags (`mongo:<version>`, `mongo-express:<version>`) instead of floating tags.
- Add CPU/memory requests and limits.
- Add readiness and liveness probes.
- Use TLS for ingress traffic.
- Apply NetworkPolicies to restrict east-west traffic.
- Prefer StatefulSet + PersistentVolumeClaim for MongoDB durability in real environments.

Example hardening direction for MongoDB persistence:

```text
Current: Deployment (ephemeral container filesystem)
Recommended for production: StatefulSet + PVC + StorageClass
```

## Cleanup

```bash
kubectl delete -f mongo-express-2.yaml
kubectl delete -f mongo-express.yaml
kubectl delete -f mongo.yaml
kubectl delete -f mongo-configmap.yaml
kubectl delete -f mongo-secret.yaml
```

If running locally:

```bash
minikube stop
```

## Official References

- Kubernetes Components Overview: https://kubernetes.io/docs/concepts/overview/components/
- Deployments: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- Services: https://kubernetes.io/docs/concepts/services-networking/service/
- Ingress: https://kubernetes.io/docs/concepts/services-networking/ingress/
- ConfigMaps: https://kubernetes.io/docs/concepts/configuration/configmap/
- Secrets: https://kubernetes.io/docs/concepts/configuration/secret/
- StatefulSets: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
- Persistent Volumes: https://kubernetes.io/docs/concepts/storage/persistent-volumes/
- Minikube Handbook: https://minikube.sigs.k8s.io/docs/
