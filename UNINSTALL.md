# Complete K3D Cluster Teardown & Cleanup
[<< back](INSTALL.md)

Commands to fully delete the k3d cluster, clean up all associated resources, and optionally restart Docker.

<br />

## Step 1: Remove ArgoCD Applications

Delete all ArgoCD-managed applications first so finalizers and child resources are cleaned up gracefully:

``` shell
# List all argocd applications
kubectl get applications -n argocd

# Delete all argocd applications
kubectl delete applications --all -n argocd

# If any applications are stuck deleting due to finalizers, patch them:
kubectl get applications -n argocd -o name | xargs -r -I{} kubectl patch {} --type json --patch='[ { "op": "remove", "path": "/metadata/finalizers" } ]' -n argocd
```

<br />

## Step 2: Delete the K3D Cluster

This removes all k3d node containers, the load balancer, and the associated Docker network:

``` shell
k3d cluster delete prd-local-apps-001
```

Verify the cluster is gone:

``` shell
k3d cluster list
```

<br />

## Step 3: Clean Up K3D Data Directories

Remove the persistent data directories that were mounted into the cluster:

``` shell
sudo rm -rf /mnt/data/k3d-node-data
sudo rm -rf /mnt/data/k3d-app-data
```

<br />

## Step 4: Clean Up Leftover Docker Resources

After the cluster is deleted, orphaned Docker resources (volumes, networks, images) may remain:

``` shell
# Remove any stopped containers related to k3d
docker ps -a --filter "label=app=k3d" -q | xargs -r docker rm -f

# Remove k3d-related volumes
docker volume ls --filter "label=app=k3d" -q | xargs -r docker volume rm

# Remove k3d-related networks
docker network ls --filter "label=app=k3d" -q | xargs -r docker network rm

# Remove dangling/unused images (frees disk space)
docker image prune -a -f

# Remove unused volumes (WARNING: removes ALL unused Docker volumes, not just k3d)
# docker volume prune -f

# Full system prune — removes all unused containers, networks, images, and optionally volumes
# docker system prune -a -f --volumes
```

> **WARNING:** `docker system prune -a -f --volumes` and `docker volume prune -f` will remove **all** unused Docker resources, not just k3d-related ones. Only use these if you have no other Docker workloads you want to keep.

<br />

## Step 5: Restart Docker

Restarting Docker ensures all GPU device handles, runtime state, and cgroup resources are fully released:

``` shell
sudo systemctl restart docker
```

Verify Docker is running:

``` shell
sudo systemctl status docker
docker info
```

If you have GPU support, verify GPU access is still available after the restart:

**Radeon GPU (ROCm):**

``` shell
rocm-smi
docker run --rm --device /dev/kfd --device /dev/dri --group-add video --group-add render rocm/pytorch:latest rocm-smi
```

**NVIDIA GPU:**

``` shell
docker run --rm --gpus all nvidia/cuda:12.0.1-base-ubuntu22.04 nvidia-smi
```

<br />

## Step 6: Clean Up Kubectl Context

k3d automatically adds a context to your kubeconfig. Remove it after deleting the cluster:

``` shell
# k3d usually cleans this up on cluster delete, but verify:
kubectl config get-contexts

# If the context still exists, remove it manually:
kubectl config delete-context k3d-prd-local-apps-001
kubectl config delete-cluster k3d-prd-local-apps-001
kubectl config delete-user admin@k3d-prd-local-apps-001
```

<br />

## Quick Reference: Full Teardown (All Steps Combined)

Run all cleanup steps in sequence:

``` shell
# 1. Delete argocd applications gracefully
kubectl delete applications --all -n argocd 2>/dev/null || true
kubectl get applications -n argocd -o name 2>/dev/null | xargs -r -I{} kubectl patch {} --type json --patch='[ { "op": "remove", "path": "/metadata/finalizers" } ]' -n argocd 2>/dev/null || true

# 2. Delete the k3d cluster
k3d cluster delete prd-local-apps-001

# 3. Clean up data directories
sudo rm -rf /mnt/k3d-data
sudo rm -rf /mnt/k3d-data-apps

# 4. Clean up leftover docker resources
docker ps -a --filter "label=app=k3d" -q | xargs -r docker rm -f
docker volume ls --filter "label=app=k3d" -q | xargs -r docker volume rm
docker network ls --filter "label=app=k3d" -q | xargs -r docker network rm
docker image prune -a -f

# 5. Restart docker
sudo systemctl restart docker

# 6. Clean up kubectl context (usually handled by k3d)
kubectl config delete-context k3d-prd-local-apps-001 2>/dev/null || true
kubectl config delete-cluster k3d-prd-local-apps-001 2>/dev/null || true
kubectl config delete-user admin@k3d-prd-local-apps-001 2>/dev/null || true
```
