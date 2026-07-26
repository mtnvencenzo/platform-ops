# Azure Stack - Kubernetes-First Azure Emulator Environment

This stack provides Azure service emulators for local development with Kubernetes-first deployment.

![Azure Stack Architecture](./assets/azure-stack.drawio.svg)

## Cluster Setup

Set up the base k3d/k3s cluster first using [the main cluster setup guide](../../INSTALL.md).

For the full repository overview, see [the root README](../../README.md).

## Deployment Priority

1. Kubernetes via Argo CD + Kustomize (primary)

## Stack Contents

- `k8s/`: Kubernetes manifests for Azure emulators
- `argocd/azure-stack-app.yaml`: Argo CD Application manifest
- `docker-compose.yml`: Legacy Docker Compose deployment (unmaintained)
- `assets/`: Service-specific setup notes

## Kubernetes Deployment (Primary)

### Prerequisites

- A running Kubernetes cluster (k3d/k3s supported)
- Argo CD installed in namespace `argocd` (for GitOps flow)
- Ingress controller available (Traefik is assumed)

### Option A: Argo CD (recommended)

Using the public GitHub manifest URL:

```bash
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/azure-stack/argocd/azure-stack-app.yaml

# Remove Argo CD app + all stack resources
kubectl delete -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/azure-stack/argocd/azure-stack-app.yaml

kubectl delete namespace azure-platform
```

This deploys to namespace `azure-platform` and syncs from `stacks/azure-stack/k8s`.

### Option B: Direct Kustomize apply

From repo root:

```bash
kubectl apply -k stacks/azure-stack/k8s

# Remove resources applied from this kustomization
kubectl delete -k stacks/azure-stack/k8s
kubectl delete namespace azure-platform
```

### Verify

```bash
kubectl -n azure-platform get pods,svc,ingress,pvc
```

### Access Endpoints (current default kustomization)

- Azurite Blob: http://azure-blob.127.0.0.1.sslip.io
- Azurite Queue: http://azure-queue.127.0.0.1.sslip.io
- Azurite Table: http://azure-table.127.0.0.1.sslip.io

### Important Notes

- Current `k8s/kustomization.yaml` enables Azurite only by default.
- CosmosDB, SQL Server, Event Hubs, Service Bus, and App Config manifests exist but are commented out.
- To enable Cosmos ingress routes, also include `k8s/ingress-cosmosdb.yml`.

## Stack-Specific Notes

- Optional service dependencies:
	- Event Hubs depends on Azurite.
	- Service Bus depends on SQL Server.
- If deploying resources individually instead of `kubectl apply -k`, apply in dependency order (Azurite/SQL first).
- Cross-namespace service access examples:
	- `azurite.azure-platform.svc.cluster.local:10000`
	- `cosmosdb.azure-platform.svc.cluster.local:8081`
	- `sqlserver.azure-platform.svc.cluster.local:1433`

### Optional Service Endpoints

- CosmosDB emulator: `cosmosdb.azure-platform.svc.cluster.local:8081`
- CosmosDB Data Explorer: service port `1234`
- Event Hubs emulator: `eventhubs.azure-platform.svc.cluster.local:5672` and `:9092`
- Service Bus emulator: `servicebus.azure-platform.svc.cluster.local:5671`
- SQL Server: `sqlserver.azure-platform.svc.cluster.local:1433`
- App Configuration: `appconfig.azure-platform.svc.cluster.local:8483`

## Configuration Notes

- Core settings and secrets are defined in `k8s/configmap.yml`.
- Default local credentials and keys are intended for development only.
- If enabling additional emulators in Kubernetes, uncomment resources in `k8s/kustomization.yaml` first.

## Troubleshooting

```bash
kubectl -n azure-platform get pods
kubectl -n azure-platform describe pod <pod-name>
kubectl -n azure-platform logs deploy/azurite
```

## Disclaimer

These manifests and instructions are intended for local development, testing, and homelab usage. They are not production-hardened and should be reviewed and adapted before use in shared or production environments.

## References

- https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azurite
- https://learn.microsoft.com/en-us/azure/cosmos-db/emulator-linux
- https://learn.microsoft.com/en-us/azure/event-hubs/overview-emulator
- https://github.com/Azure/azure-service-bus-emulator-installer
