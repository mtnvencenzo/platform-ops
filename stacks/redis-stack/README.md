# Redis Stack - Kubernetes-First Redis Environment

This stack deploys Redis with RedisInsight for local cache and data-structure workflows.

![Redis Architecture Diagram](./assets/redis-stack.drawio.svg)

## Cluster Setup

Set up the base k3d/k3s cluster first using [the main cluster setup guide](../../INSTALL.md).

## Deployment Priority

1. Kubernetes via Argo CD + Kustomize (primary)

## Stack Contents

- `k8s/`: Kubernetes manifests for Redis + RedisInsight
- `argocd/redis-stack-app.yaml`: Argo CD Application manifest
- `docker-compose.yml`: Legacy Docker Compose deployment (unmaintained)

## Kubernetes Deployment (Primary)

### Prerequisites

- Kubernetes cluster available
- Argo CD installed in namespace `argocd`
- Ingress controller available (Traefik assumed)

### Option A: Argo CD (recommended)

Using the public GitHub manifest URL:

```bash
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/redis-stack/argocd/redis-stack-app.yaml

# Remove Argo CD app + all stack resources
kubectl delete -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/redis-stack/argocd/redis-stack-app.yaml
kubectl delete namespace redis-platform
```

This deploys to namespace `redis-platform` and syncs from `stacks/redis-stack/k8s`.

### Option B: Direct Kustomize apply

From repo root:

```bash
kubectl apply -k stacks/redis-stack/k8s

# Remove resources applied from this kustomization
kubectl delete -k stacks/redis-stack/k8s
kubectl delete namespace redis-platform
```

### Verify

```bash
kubectl -n redis-platform get pods,svc,ingress,pvc
```

### Access Endpoints

- RedisInsight UI: http://redis-insight.127.0.0.1.sslip.io
- Redis service (cluster-internal): `redis.redis-platform.svc.cluster.local:6380`

## Stack-Specific Notes

- Redis service port is `6380` in Kubernetes and maps to container port `6379`.
- RedisInsight is preconfigured to connect to `redis:6380` in this stack.
- In-cluster clients should target `redis.redis-platform.svc.cluster.local:6380`.

## Configuration Notes

- Redis runs with AOF persistence enabled.
- Kubernetes service exposes Redis as service port `6380` mapped to container port `6379`.
- Redis and RedisInsight data are persisted to PVCs.

## Troubleshooting

```bash
kubectl -n redis-platform get pods
kubectl -n redis-platform logs deploy/redis
kubectl -n redis-platform logs deploy/redis-insight
```

## References

- https://redis.io/docs/
- https://redis.io/docs/connect/insight/
