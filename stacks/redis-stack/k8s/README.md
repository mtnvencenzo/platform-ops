# Redis Platform – Kubernetes Manifests (k3d)

Kubernetes manifests for deploying Redis and Redis Insight to a local k3d cluster.

## Services

| Service | Description | Internal Port | K8s Service |
|---|---|---|---|
| **Redis** | In-memory data store (Alpine) | 6380 | `redis:6380` |
| **Redis Insight** | Web UI for Redis management | 5540 | `redis-insight:5540` |

## Deploy

```bash
kubectl apply -k k8s/
```

Or in dependency order:

```bash
kubectl apply -f k8s/namespace.yml
kubectl apply -f k8s/pvcs.yml
kubectl apply -f k8s/redis.yml
kubectl apply -f k8s/redis-insight.yml  # depends on redis
kubectl apply -f k8s/ingress.yml
```

## Cross-namespace Access

```
redis.redis-platform.svc.cluster.local:6380
```

## Access

### Web UI (via Ingress)

- **Redis Insight**: http://redis-insight.127.0.0.1.sslip.io:8080

### Port Forwarding (alternative)

```bash
kubectl port-forward -n redis-platform svc/redis 6380:6379
```

## Cleanup

```bash
kubectl delete namespace redis-platform
```
