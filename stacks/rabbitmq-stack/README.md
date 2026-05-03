# RabbitMQ Stack - Kubernetes-First Messaging Environment

This stack deploys RabbitMQ with the management UI for local messaging workflows.

![RabbitMq Stack](./assets/rabbitmq-stack.drawio.svg)

## Cluster Setup

Set up the base k3d/k3s cluster first using [the main cluster setup guide](../../INSTALL.md).

## Deployment Priority

1. Kubernetes via Argo CD + Kustomize (primary)

## Stack Contents

- `k8s/`: Kubernetes manifests for RabbitMQ
- `argocd/rabbitmq-stack-app.yaml`: Argo CD Application manifest
- `docker-compose.yml`: Legacy Docker Compose deployment (unmaintained)
- `rabbitmq.conf`: Compose RabbitMQ configuration

## Kubernetes Deployment (Primary)

### Prerequisites

- Kubernetes cluster available
- Argo CD installed in namespace `argocd`
- Ingress controller available (Traefik assumed)

### Option A: Argo CD (recommended)

Using the public GitHub manifest URL:

```bash
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/rabbitmq-stack/argocd/rabbitmq-stack-app.yaml

# Remove Argo CD app + all stack resources
kubectl delete -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/rabbitmq-stack/argocd/rabbitmq-stack-app.yaml
kubectl delete namespace rabbitmq-platform
```

This deploys to namespace `rabbitmq-platform` and syncs from `stacks/rabbitmq-stack/k8s`.

### Option B: Direct Kustomize apply

From repo root:

```bash
kubectl apply -k stacks/rabbitmq-stack/k8s

# Remove resources applied from this kustomization
kubectl delete -k stacks/rabbitmq-stack/k8s
kubectl delete namespace rabbitmq-platform
```

### Verify

```bash
kubectl -n rabbitmq-platform get pods,svc,ingress,pvc
```

### Access Endpoints

- RabbitMQ Management UI: http://rabbitmq.127.0.0.1.sslip.io
- AMQP NodePort: `localhost:30672` (maps to RabbitMQ `5672`)

### Default Credentials

- Username: `admin`
- Password: `password`

## Stack-Specific Notes

- Current Kubernetes manifests expose non-TLS AMQP (`5672`) for local usage.
- TLS configuration templates are present but commented out in `k8s/configmap.yml`.
- In-cluster clients can use service DNS `rabbitmq.rabbitmq-platform.svc.cluster.local:5672`.

## Configuration Notes

- Current Kubernetes config exposes non-TLS AMQP (`5672`) via NodePort.
- TLS settings in `k8s/configmap.yml` are currently commented out.
- RabbitMQ data and logs are persisted via PVCs.

## Troubleshooting

```bash
kubectl -n rabbitmq-platform get pods
kubectl -n rabbitmq-platform logs deploy/rabbitmq
```

## References

- https://www.rabbitmq.com/documentation.html
- https://www.rabbitmq.com/docs/management
