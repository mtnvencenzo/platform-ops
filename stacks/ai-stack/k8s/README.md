# AI Stack – Kubernetes Manifests (k3d)

Kubernetes manifests for deploying the AI stack to a local k3d cluster via ArgoCD.

## Services

| Service | Description | Internal Port | K8s Service | Ingress |
|---|---|---|---|---|
| **Ollama (host)** | LLM runtime running on the host, accessed via EndpointSlice | 11434 | `ollama-host:11434` | — |
| **Qdrant** | Vector database for RAG | 6333/6334 | `qdrant:6333`, `qdrant:6334` | http://qdrant.127.0.0.1.sslip.io |
| **TEI** | Text embeddings (all-mpnet-base-v2) | 8989 | `tei:8989` | http://tei.127.0.0.1.sslip.io |
| **TEI Reranker** | Cross-encoder reranker | 8990 | `tei-reranker:8990` | http://tei-reranker.127.0.0.1.sslip.io |
| **TEI SPLADE** | Sparse encoder | 8991 | `tei-splade:8991` | http://tei-splade.127.0.0.1.sslip.io |

> **Note:** Ollama and Open WebUI manifests exist but are commented out in `kustomization.yaml`. Ollama currently runs on the host and is exposed to the cluster via `ollama-host-service.yml` using a Service + EndpointSlice pointing at the k3d network gateway (`172.18.0.1`).

## Prerequisites

- k3d cluster running (see [INSTALL_K3D.md](../../../INSTALL_K3D.md))
- NVIDIA GPU + drivers installed (for TEI GPU workloads)
- [NVIDIA k8s device plugin](https://github.com/NVIDIA/k8s-device-plugin) installed in the cluster (see [INSTALL_K8S_GPU.md](../../../INSTALL_K8S_GPU.md))
- `nvidia` RuntimeClass configured
- ArgoCD installed and configured to allow EndpointSlice resources (see [INSTALL_K8S_ARGOCD.md](../../../INSTALL_K8S_ARGOCD.md) Step 4)

## Deploy

This stack is deployed via ArgoCD. The ArgoCD Application points at the `stacks/ai-stack/k8s` directory and uses Kustomize automatically.

```bash
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/ai-stack/argocd/ai-stack-app.yaml
```

ArgoCD will sync all resources defined in `kustomization.yaml`:
- `namespace.yml` – creates the `ai-platform` namespace
- `configmap.yml` – configuration and secrets (HF token, model IDs, API keys)
- `pvcs.yml` – persistent volume claims for Qdrant and HF model caches
- `qdrant.yml` – Qdrant vector database deployment + service
- `tei.yml` – text embeddings inference deployment + service
- `tei-reranker.yml` – reranker deployment + service
- `tei-splade.yml` – SPLADE sparse encoder deployment + service
- `ollama-host-service.yml` – Service + EndpointSlice for host Ollama
- `ingress.yml` – Traefik ingress routes

## Configuration

### Secrets

Edit `configmap.yml` to set your Hugging Face token:

```yaml
stringData:
  HF_TOKEN: "hf_your_actual_token_here"
```

### Models

The default models can be changed in the ConfigMap:

| Setting | Default |
|---|---|
| `TEI_MODEL_ID` | `sentence-transformers/all-mpnet-base-v2` |
| `TEI_RERANKER_MODEL_ID` | `cross-encoder/ms-marco-MiniLM-L-6-v2` |
| `TEI_SPLADE_MODEL_ID` | `naver/splade-cocondenser-ensembledistil` |

### Ollama Host IP

The `ollama-host-service.yml` EndpointSlice uses `172.18.0.1` as the host IP. This is the gateway of the k3d Docker network (`k3d-prd-local-apps-001`), which is pinned via a pre-created network during cluster setup (see [INSTALL_K3D.md](../../../INSTALL_K3D.md)).

To verify the gateway matches:

```bash
docker network inspect k3d-prd-local-apps-001 --format '{{(index .IPAM.Config 0).Gateway}}'
```

### Running Without GPU

If you don't have an NVIDIA GPU, remove or comment out these lines from the TEI deployments:

1. `runtimeClassName: nvidia`
2. `nvidia.com/gpu: "1"` from resource limits
3. The `CUDA_VISIBLE_DEVICES` and `NVIDIA_VISIBLE_DEVICES` env vars

You may also need to switch to a CPU-compatible image tag (e.g., `ghcr.io/huggingface/text-embeddings-inference:cpu-1.8`).

## Verify

```bash
# Check all pods are running
kubectl get pods -n ai-platform

# Watch pod status
kubectl get pods -n ai-platform --watch

# Check logs for a specific service
kubectl logs -n ai-platform deployment/qdrant -f
kubectl logs -n ai-platform deployment/tei -f

# Verify ollama host connectivity from within the cluster (expects "Ollama is running")
kubectl run curl-test --rm -it --restart=Never --image=curlimages/curl -n ai-platform -- curl -s http://ollama-host:11434
```

## Cleanup

To remove the stack, delete the ArgoCD application:

```bash
kubectl delete application ai-stack -n argocd
```

Or to remove the namespace directly:

```bash
kubectl delete namespace ai-platform
```
