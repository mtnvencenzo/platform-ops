# AI Stack - Kubernetes-First AI Engineering Environment

This stack provides a local AI engineering platform for chat, vector search, and embeddings workflows.

![AI Stack](./assets/ai-stack.drawio.svg)

## Cluster Setup

Set up the base k3d/k3s cluster first using [the main cluster setup guide](../../INSTALL.md).

For the full repository overview, see [the root README](../../README.md).

## Deployment Priority

1. Kubernetes via Argo CD + Kustomize (primary)

## Stack Contents

- `k8s/`: Kubernetes manifests (Kustomize base for the stack)
- `argocd/ai-stack-app.yaml`: Argo CD Application manifest
- `docker-compose.yml`: Legacy Docker Compose deployment (unmaintained)
- `docker-compose-langfuse.yml`: Legacy optional Langfuse compose deployment (unmaintained)

## Kubernetes Deployment (Primary)

### Prerequisites

- A running Kubernetes cluster (k3d/k3s works well)
- Argo CD installed in namespace `argocd` (for GitOps flow)
- Ingress controller available (Traefik is assumed in manifests)

### Option A: Argo CD (recommended)

Using the public GitHub manifest URL:

```bash
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/ai-stack/argocd/ai-stack-app.yaml

# Remove Argo CD app + all stack resources
kubectl delete -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/ai-stack/argocd/ai-stack-app.yaml
kubectl delete namespace ai-platform
```

This deploys to namespace `ai-platform` and syncs from `stacks/ai-stack/k8s`.

### Option B: Direct Kustomize apply

From repo root:

```bash
kubectl apply -k stacks/ai-stack/k8s

# Remove resources applied from this kustomization
kubectl delete -k stacks/ai-stack/k8s
kubectl delete namespace ai-platform
```

### Verify

```bash
kubectl -n ai-platform get pods,svc,ingress,pvc
```

### Access Endpoints

- Open WebUI: http://open-webui.127.0.0.1.sslip.io
- Qdrant: http://qdrant.127.0.0.1.sslip.io

### Important Notes

- The default Kustomize resources enable Open WebUI + Qdrant with host Ollama service.
- TEI and in-cluster Ollama manifests are present but commented out in `k8s/kustomization.yaml`.
- `k8s/ollama-host-service.yml` points to `172.18.0.1:11434`; update this if your host network differs.

## Stack-Specific Notes

### Host Ollama Setup (required for current default manifests)

```bash
# Install Ollama on the host
curl -fsSL https://ollama.com/install.sh | sh

# Make Ollama listen on a non-loopback interface
sudo systemctl edit ollama.service

# Add this to the override file:
[Service]
Environment="OLLAMA_HOST=0.0.0.0"

# Reload and restart
sudo systemctl daemon-reload
sudo systemctl restart ollama.service

# Verify Ollama is listening on 0.0.0.0 or *:11434
sudo ss -antp | grep 11434
```

### Network + Firewall

```bash
# Verify the k3d Docker network gateway matches the EndpointSlice target
docker network inspect k3d-prd-local-apps-001 --format '{{(index .IPAM.Config 0).Gateway}}'

# If UFW is enabled, allow k3d pod traffic to host Ollama
sudo ufw allow from 172.18.0.0/16 to any port 11434 comment 'k3d pods -> host ollama'
```

### TEI GPU Compatibility

- TEI runs in CPU mode by default for Radeon-based local setups.
- ROCm TEI currently supports AMD Instinct accelerators (MI200/MI300), not typical RDNA consumer cards.
- If switching to NVIDIA, follow the commented instructions in the TEI manifests for `runtimeClassName`, GPU resources, and image tags.

## Configuration Notes

- Main runtime config and secrets are defined in `k8s/configmap.yml`.
- Replace `HF_TOKEN` before enabling TEI workloads that require gated model pulls.
- If running Ollama on the host, ensure it listens on a non-loopback address.

## Troubleshooting

```bash
kubectl -n ai-platform get pods
kubectl -n ai-platform describe pod <pod-name>
kubectl -n ai-platform logs deploy/open-webui
kubectl -n ai-platform logs deploy/qdrant
```

## Disclaimer

These manifests and instructions are intended for local development, testing, and homelab usage. They are not production-hardened and should be reviewed and adapted before use in shared or production environments.

## References

- https://docs.openwebui.com/
- https://ollama.com/
- https://qdrant.tech/documentation/
- https://huggingface.co/docs/text-embeddings-inference/
