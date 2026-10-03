# platform-manifests

Kubernetes manifests (desired state) for the platform project. Application code and image builds live in [hello-api](https://github.com/AhsanASid/hello-api).

## Layout
- `hello-api/`: Deployment (2 replicas, readiness/liveness probes, resource limits) and ClusterIP Service

## How deployment works
1. Pushing to `hello-api` triggers GitHub Actions, which builds and publishes `ghcr.io/ahsanasid/hello-api:sha-<commit>`
2. The image tag in `hello-api/deployment.yaml` is updated to that commit
3. The cluster is reconciled to match this repo (manually with `kubectl apply` for now, via ArgoCD in Phase 3)

## Local usage
```bash
kind create cluster --name platform-dev
kubectl apply -f hello-api/
kubectl rollout status deployment/hello-api
```
