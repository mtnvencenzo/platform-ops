# Elastic Stack - Kubernetes-First Observability Stack

This stack deploys Elasticsearch, Kibana, APM Server, and OpenTelemetry Collector for local observability workflows.

![Elastic Stack Diagram](./assets/elastic-stack.png)

## Cluster Setup

Set up the base k3d/k3s cluster first using [the main cluster setup guide](../../INSTALL.md).

## Deployment Priority

1. Kubernetes via Argo CD + Kustomize (primary)

## Stack Contents

- `k8s/`: Kubernetes manifests for Elastic + OTEL collector
- `argocd/elastic-stack-app.yaml`: Argo CD Application manifest
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
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/elastic-stack/argocd/elastic-stack-app.yaml

# Remove Argo CD app + all stack resources
kubectl delete -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/elastic-stack/argocd/elastic-stack-app.yaml
kubectl delete namespace elastic-platform
```

This deploys to namespace `elastic-platform` and syncs from `stacks/elastic-stack/k8s`.

### Option B: Direct Kustomize apply

From repo root:

```bash
kubectl apply -k stacks/elastic-stack/k8s

# Remove resources applied from this kustomization
kubectl delete -k stacks/elastic-stack/k8s
kubectl delete namespace elastic-platform
```

### Verify

```bash
kubectl -n elastic-platform get pods,svc,ingress,pvc
```

### Access Endpoints

- Kibana: http://kibana.127.0.0.1.sslip.io
- OTEL HTTP ingest: http://otel-collector.127.0.0.1.sslip.io
- OTEL gRPC ingest: `otel-collector.elastic-platform.svc.cluster.local:4317`

## Stack-Specific Notes

- On k3d, ensure ingress port mapping is configured for your cluster (for example `8080:80@loadbalancer` if you expose ingress on host port 8080).
- If you change ingress hostnames or port mappings, update the endpoint URLs accordingly.
- For cross-namespace telemetry, use:
	- `otel-collector.elastic-platform.svc.cluster.local:4317` (gRPC)
	- `otel-collector.elastic-platform.svc.cluster.local:4318` (HTTP)

## Configuration Notes

- Current manifests run Elasticsearch and Kibana with security disabled for local usage.
- OTEL receiver tokens and pipeline settings are in `k8s/configmap.yml`.
- Elasticsearch needs enough host memory; monitor pod scheduling if startup is slow.

## Troubleshooting

```bash
kubectl -n elastic-platform get pods
kubectl -n elastic-platform logs deploy/elasticsearch
kubectl -n elastic-platform logs deploy/kibana
kubectl -n elastic-platform logs deploy/otel-collector
```

## References

- https://www.elastic.co/guide/en/observability/current/index.html
- https://opentelemetry.io/docs/collector/
