# OpenObserve Stack - Kubernetes-First Observability Stack

This stack deploys OpenObserve with OpenTelemetry Collector for local logs, metrics, and traces ingestion.

## Cluster Setup

Set up the base k3d/k3s cluster first using [the main cluster setup guide](../../INSTALL.md).

## Deployment Priority

1. Kubernetes via Argo CD + Kustomize (primary)

## Stack Contents

- `k8s/`: Kubernetes manifests for OpenObserve + OTEL collector
- `argocd/openobserve-stack-app.yaml`: Argo CD Application manifest
- `docker-compose.yml`: Legacy Docker Compose deployment (unmaintained)
- `otel-collector-config.yml`: Compose OTEL configuration

## Kubernetes Deployment (Primary)

### Prerequisites

- Kubernetes cluster available
- Argo CD installed in namespace `argocd`
- Ingress controller available (Traefik assumed)

### Option A: Argo CD (recommended)

Using the public GitHub manifest URL:

```bash
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/openobserve-stack/argocd/openobserve-stack-app.yaml

# Remove Argo CD app + all stack resources
kubectl delete -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/openobserve-stack/argocd/openobserve-stack-app.yaml
kubectl delete namespace openobserve-platform
```

This deploys to namespace `openobserve-platform` and syncs from `stacks/openobserve-stack/k8s`.

### Option B: Direct Kustomize apply

From repo root:

```bash
kubectl apply -k stacks/openobserve-stack/k8s

# Remove resources applied from this kustomization
kubectl delete -k stacks/openobserve-stack/k8s
kubectl delete namespace openobserve-platform
```

### Verify

```bash
kubectl -n openobserve-platform get pods,svc,ingress,pvc
```

### Access Endpoints

- OpenObserve UI: http://openobserve.127.0.0.1.sslip.io
- OTEL HTTP ingest: http://otel-collector.127.0.0.1.sslip.io
- OTEL gRPC ingest: `otel-collector.openobserve-platform.svc.cluster.local:4317`

### Default Credentials

- Email: `admin@example.com`
- Password: `ComplexPassword#123`

## Stack-Specific Notes

- OTEL receiver auth is token-based; clients must send a valid bearer token configured in `k8s/configmap.yml`.
- Collector forwards telemetry to OpenObserve internal ingest endpoint (`openobserve:5081`).
- If telemetry is rejected, validate both the bearer token and OpenObserve credentials in `k8s/configmap.yml`.

## Configuration Notes

- OTEL token auth and exporter settings are defined in `k8s/configmap.yml`.
- OpenObserve data persists to PVC `openobserve-data` in Kubernetes.

## Troubleshooting

```bash
kubectl -n openobserve-platform get pods
kubectl -n openobserve-platform logs deploy/openobserve
kubectl -n openobserve-platform logs deploy/otel-collector
```

## References

- https://openobserve.ai/docs/
- https://opentelemetry.io/docs/collector/
