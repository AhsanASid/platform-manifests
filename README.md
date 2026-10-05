# platform-manifests

Kubernetes manifests (desired state) for the platform project. Application code and image builds live in [hello-api](https://github.com/AhsanASid/hello-api).

## Layout
- `hello-api/`: Deployment (2 replicas, readiness/liveness probes, resource limits) and ClusterIP Service
- `argocd/`: ArgoCD Application manifests
- `monitoring/`: Helm values for lean Prometheus, Grafana, and Loki

## How deployment works
1. Pushing to `hello-api` triggers GitHub Actions, which builds and publishes `ghcr.io/ahsanasid/hello-api:sha-<commit>`
2. The image tag in `hello-api/deployment.yaml` is updated to that commit
3. The cluster is reconciled to match this repo

## GitOps & ArgoCD Status (Phase 3 Note)
Declarative GitOps repository structures and ArgoCD Application manifests are configured in `argocd/`. In this local single-node `kind` environment, automated in-cluster ArgoCD sync was parked due to an outbound pod-network TLS handshake limitation (host and node image pulls function normally). To avoid false claims, maintain engineering honesty, and save memory, ArgoCD workloads remain scaled down, with manifests and Helm values applied directly from the host.

## Observability Stack
A lean, local-first observability stack is deployed in the `monitoring` namespace:
- **Prometheus** (`prometheus-community/prometheus`): Single server instance, 2h retention, ephemeral storage; Alertmanager, Node Exporter, and Pushgateway disabled to minimize RAM footprint.
- **Grafana** (`grafana/grafana`): Declarative datasource provisioning for Prometheus and Loki, ephemeral storage.
- **Loki & Promtail** (`grafana/loki-stack`): Monolithic single-binary log engine with local filesystem storage and Promtail container log scraper.

## Local usage
```bash
kind create cluster --name platform-dev
kubectl apply -f hello-api/
kubectl rollout status deployment/hello-api
```
