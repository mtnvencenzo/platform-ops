# Dapr Stack - Kubernetes-First Self-Hosted Runtime

This stack deploys core Dapr self-hosted runtime services for local development on Kubernetes.

![Dapr Architecture Diagram](./assets/dapr-stack.drawio.svg)

## Cluster Setup

Set up the base k3d/k3s cluster first using [the main cluster setup guide](../../INSTALL.md).

## Deployment Priority

1. Kubernetes via Argo CD + Kustomize (primary)

## Stack Contents

- `k8s/`: Kubernetes manifests for placement and scheduler
- `argocd/dapr-stack-app.yaml`: Argo CD Application manifest
- `docker-compose.yml`: Legacy Docker Compose deployment (unmaintained)

## Kubernetes Deployment (Primary)

### Prerequisites

- Kubernetes cluster available
- Argo CD installed in namespace `argocd`

### Option A: Argo CD (recommended)

Using the public GitHub manifest URL:

```bash
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/dapr-stack/argocd/dapr-stack-app.yaml

# Remove Argo CD app + all stack resources
kubectl delete -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/dapr-stack/argocd/dapr-stack-app.yaml
kubectl delete namespace dapr-platform
```

This deploys to namespace `dapr-platform` and syncs from `stacks/dapr-stack/k8s`.

### Option B: Direct Kustomize apply

From repo root:

```bash
kubectl apply -k stacks/dapr-stack/k8s

# Remove resources applied from this kustomization
kubectl delete -k stacks/dapr-stack/k8s
kubectl delete namespace dapr-platform
```

### Verify

```bash
kubectl -n dapr-platform get pods,svc,pvc
```

### Runtime Endpoints

- Dapr placement service (ClusterIP): `dapr-placement.dapr-platform.svc.cluster.local:50005`
- Dapr scheduler service (ClusterIP): `dapr-scheduler.dapr-platform.svc.cluster.local:50006`
- Dapr scheduler NodePort: `localhost:30006` (maps to scheduler `50006`)

## Stack-Specific Notes

- These manifests are intended for local self-hosted parity.
- For production Kubernetes deployments, prefer the official Dapr Helm chart.
- If workloads run in-cluster, use ClusterIP DNS endpoints.
- If workloads run outside the cluster, use scheduler NodePort `30006`.

## Configuration Notes

- Scheduler persistence uses PVC `dapr-scheduler-data`.
- If your applications run outside the cluster, use the exposed scheduler NodePort.

## Troubleshooting

```bash
kubectl -n dapr-platform get pods
kubectl -n dapr-platform logs deploy/dapr-placement
kubectl -n dapr-platform logs deploy/dapr-scheduler
```

## References

- https://docs.dapr.io/
- https://docs.dapr.io/operations/hosting/self-hosted/
