# AI Stack – Kubernetes Manifests (k3d)

Kubernetes manifests for deploying the AI stack to a local k3d cluster.

## Services

| Service | Description | Internal Port | K8s Service |
|---|---|---|---|
| **Ollama** | Local LLM runtime (GPU) | 11434 | `ollama:11434` |
| **Open WebUI** | Chat UI for Ollama/OpenAI endpoints | 8080 | `open-webui:3000` |
| **Qdrant** | Vector database for RAG | 6333/6334 | `qdrant:6333`, `qdrant:6334` |
| **TEI** | Text embeddings (all-mpnet-base-v2) | 8989 | `tei:8989` |
| **TEI Reranker** | Cross-encoder reranker | 8990 | `tei-reranker:8990` |
| **TEI SPLADE** | Sparse encoder | 8991 | `tei-splade:8991` |

## Prerequisites

- k3d cluster running (see [INSTALL_K8S.md](../../INSTALL_K8S.md))
- NVIDIA GPU + drivers installed on (if using GPU workloads)
- [NVIDIA k8s device plugin](https://github.com/NVIDIA/k8s-device-plugin) installed in the cluster
- `nvidia` RuntimeClass configured (for Ollama and TEI services)

Install the NVIDIA device plugin:

```bash
kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.17.0/deployments/static/nvidia-device-plugin.yml
```

Create the `nvidia` RuntimeClass:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: nvidia
handler: nvidia
EOF
```

## Deploy

Apply all manifests in order:

```bash
# Create namespace
kubectl apply -f k8s/namespace.yml

# Create config and secrets
kubectl apply -f k8s/configmap.yml

# Create persistent volume claims
kubectl apply -f k8s/pvcs.yml

# Deploy services
kubectl apply -f k8s/ollama.yml
kubectl apply -f k8s/open-webui.yml
kubectl apply -f k8s/qdrant.yml
kubectl apply -f k8s/tei.yml
kubectl apply -f k8s/tei-reranker.yml
kubectl apply -f k8s/tei-splade.yml

# Create ingress
kubectl apply -f k8s/ingress.yml
```

Or apply everything at once:

```bash
kubectl apply -f k8s/
```

Or use Kustomize (recommended):

```bash
kubectl apply -k k8s/overlays/main/
```

This will apply all core AI stack resources in the correct order.

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

### Running Without GPU

If you don't have an NVIDIA GPU, remove or comment out these lines from the deployments:

1. `runtimeClassName: nvidia` from ollama, tei, tei-reranker, tei-splade
2. `nvidia.com/gpu: "1"` from resource limits
3. The `CUDA_VISIBLE_DEVICES` and `NVIDIA_VISIBLE_DEVICES` env vars

For the TEI services, you may also need to switch to a CPU-compatible image tag
(e.g., `ghcr.io/huggingface/text-embeddings-inference:cpu-1.8`).

## Access

Once deployed, Open WebUI is accessible via the Ingress:

- **Open WebUI**: http://ai.127.0.0.1.sslip.io:8080

Or use port-forwarding for individual services:

```bash
# Open WebUI
kubectl port-forward -n ai-platform svc/open-webui 3000:3000

# Ollama
kubectl port-forward -n ai-platform svc/ollama 11434:11434

# Qdrant
kubectl port-forward -n ai-platform svc/qdrant 6333:6333
```

## Verify

```bash
# Check all pods are running
kubectl get pods -n ai-platform

# Watch pod status
kubectl get pods -n ai-platform --watch

# Check logs for a specific service
kubectl logs -n ai-platform deployment/ollama -f
kubectl logs -n ai-platform deployment/open-webui -f
```

## Cleanup

```bash
kubectl delete namespace ai-platform
```
