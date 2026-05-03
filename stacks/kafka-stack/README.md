# Kafka Stack - Kubernetes-First Messaging Platform

This stack deploys Kafka with Schema Registry and Kafka UI in multiple modes using Kustomize overlays and Argo CD Applications. The preferred mode is KRaft; a classic Zookeeper-based mode is also available for compatibility and comparison.

## Cluster Setup

Set up the base k3d/k3s cluster first using [the main cluster setup guide](../../INSTALL.md).

## Supported Modes

### KRaft (preferred)

- Modern Kafka architecture without Zookeeper.
- Recommended for new deployments.
- Available as:
  - Single-broker local setup via `k8s/overlays/kraft/`
  - Three-broker local cluster via `k8s/overlays/kraft-3broker/`

![Kafka KRaft](./assets/kafka-stack-kraft.drawio.svg)

### Zookeeper

- Classic Kafka architecture using Zookeeper for cluster metadata and coordination.
- Kept for compatibility testing and comparison with older setups.
- Available via `k8s/overlays/zookeeper/`.

![Kafka Zookeeper](./assets/kafka-stack-zoo.drawio.svg)

## Deployment Priority

1. Kubernetes via Argo CD + Kustomize overlays (primary)

## Stack Contents

- `k8s/base/`: Shared resources (namespace, schema-registry, kafka-ui, ingress)
- `k8s/overlays/kraft/`: Single-broker KRaft mode
- `k8s/overlays/kraft-3broker/`: Three-broker KRaft mode
- `k8s/overlays/zookeeper/`: Classic Zookeeper-based mode
- `argocd/`: One Argo CD app manifest per mode
- `docker-compose*.yml`: Legacy compose alternatives for each mode (unmaintained)


## Kubernetes Deployment (Primary)

### Prerequisites

- Kubernetes cluster available
- Argo CD installed in namespace `argocd`
- Ingress controller available (Traefik assumed)

### Choose one deployment mode

- KRaft single broker (preferred default): `stacks/kafka-stack/argocd/kafka-stack-kraft-app.yaml`
- KRaft three brokers: `stacks/kafka-stack/argocd/kafka-stack-kraft-3broker-app.yaml`
- Zookeeper mode (classic compatibility option): `stacks/kafka-stack/argocd/kafka-stack-zookeeper-app.yaml`

### Option A: Argo CD (recommended)

Using the public GitHub manifest URL, apply one mode only:

```bash
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/kafka-stack/argocd/kafka-stack-kraft-app.yaml
# Remove Argo CD app + all stack resources
kubectl delete -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/kafka-stack/argocd/kafka-stack-kraft-app.yaml
kubectl delete namespace kafka-platform

# or
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/kafka-stack/argocd/kafka-stack-kraft-3broker-app.yaml
# Remove Argo CD app + all stack resources
kubectl delete -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/kafka-stack/argocd/kafka-stack-kraft-3broker-app.yaml
kubectl delete namespace kafka-platform

# or
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/kafka-stack/argocd/kafka-stack-zookeeper-app.yaml
# Remove Argo CD app + all stack resources
kubectl delete -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/kafka-stack/argocd/kafka-stack-zookeeper-app.yaml
kubectl delete namespace kafka-platform
```

All modes deploy to namespace `kafka-platform`.

### Option B: Direct Kustomize apply

From repo root, apply one overlay only:

```bash
kubectl apply -k stacks/kafka-stack/k8s/overlays/kraft
# Remove resources applied from this overlay
kubectl delete -k stacks/kafka-stack/k8s/overlays/kraft
kubectl delete namespace kafka-platform

# or
kubectl apply -k stacks/kafka-stack/k8s/overlays/kraft-3broker
# Remove resources applied from this overlay
kubectl delete -k stacks/kafka-stack/k8s/overlays/kraft-3broker
kubectl delete namespace kafka-platform

# or
kubectl apply -k stacks/kafka-stack/k8s/overlays/zookeeper
# Remove resources applied from this overlay
kubectl delete -k stacks/kafka-stack/k8s/overlays/zookeeper
kubectl delete namespace kafka-platform
```

### Verify

```bash
kubectl -n kafka-platform get pods,svc,ingress,pvc
```

### Access Endpoints

- Kafka UI: http://kafka-ui.127.0.0.1.sslip.io
- Schema Registry (cluster-internal): `schema-registry.kafka-platform.svc.cluster.local:8081`

### Kafka External Access (NodePort)

- KRaft single-broker and Zookeeper overlays: `localhost:30092`
- KRaft three-broker overlay:
  - Broker 1: `localhost:30092`
  - Broker 2: `localhost:30093`
  - Broker 3: `localhost:30095`

## Stack-Specific Notes

- Deploy only one overlay mode at a time; overlays share namespace and core resource names.
- Internal in-cluster clients should use broker/service DNS (for example `kafka-broker-1.kafka-platform.svc.cluster.local:19092`) instead of NodePorts.
- External clients should use the overlay NodePorts listed above.

## Configuration Notes

- Deploy only one mode at a time in `kafka-platform` to avoid conflicts.
- KRaft three-broker overlay includes tuned quorum timeout values for local cluster stability.
- Kafka UI and Schema Registry use shared config from each overlay `configmap.yml`.

## Troubleshooting

```bash
kubectl -n kafka-platform get pods
kubectl -n kafka-platform logs deploy/kafka-ui
kubectl -n kafka-platform logs deploy/schema-registry
kubectl -n kafka-platform logs deploy/kafka-broker-1
```

## References

- https://kafka.apache.org/documentation/
- https://docs.confluent.io/platform/current/schema-registry/index.html
- https://docs.kafka-ui.provectus.io/
