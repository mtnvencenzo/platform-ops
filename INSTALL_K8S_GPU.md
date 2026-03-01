# Enable GPU Support in K3D Cluster
[<< back](INSTALL.md)

> **Important:** The K3d cluster must be created with the `--gpus all` flag so that GPU devices are forwarded into the K3s node containers. See the cluster creation command in the [K3d setup guide](INSTALL_K8S.md) and ensure `--gpus all` is included.

<br/>

## Install the NVIDIA GPU Operator in the Cluster

### Step 1: Add the helm repository

``` shell
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia
helm repo update
```

### Step 2: Install the Operator
Set the toolkit.env variables to match K3s's internal paths.

``` shell
# helm install --wait --generate-name -n gpu-operator --create-namespace nvidia/gpu-operator

helm install gpu-operator nvidia/gpu-operator \
  -n gpu-operator --create-namespace \
  --set toolkit.env[0].name=CONTAINERD_CONFIG \
  --set toolkit.env[0].value=/var/lib/rancher/k3s/agent/etc/containerd/config.toml \
  --set toolkit.env[1].name=CONTAINERD_SOCKET \
  --set toolkit.env[1].value=/run/k3s/containerd/containerd.sock \
  --set driver.enabled=false
```
_Note: `driver.enabled=false` because k3d runs inside Docker, which already has access to the host's GPU drivers. The toolkit remains **enabled** (default) so that the operator configures containerd inside the K3s nodes to use the nvidia runtime — without this, scheduled pods won't be able to access GPU devices._

### Step 3: Wait for the GPU Operator to be Ready

The GPU Operator deploys several daemonsets and pods that take a few minutes to roll out. Wait for them before proceeding:

``` shell
kubectl -n gpu-operator wait --for=condition=ready pod --all --timeout=300s
```

You can also monitor the rollout:

``` shell
kubectl -n gpu-operator get pods -w
```

### Step 4: Enable GPU sharing

``` shell
kubectl apply -f k8s-setup/time-slicing-config-all.yml
kubectl patch clusterpolicy/cluster-policy -n gpu-operator --type merge -p '{"spec": {"devicePlugin": {"config": {"name": "time-slicing-config-all", "default": "any"}}}}'
```

### Step 5: Verify GPU Availability
Once the operator pods are running, verify the nodes have allocatable GPUs:

``` shell
kubectl get nodes "-o=custom-columns=NAME:.metadata.name,GPU:.status.allocatable.nvidia\.com/gpu"
```

<br/>

## Troubleshooting: GPU Access After Sleep/Wake

If your machine goes to sleep and resumes, GPU access may be lost in the K3d cluster due to stale device handles. Here are steps to check and recover:

### 1. Check GPU access at each layer

```bash
# Host GPU
nvidia-smi

# Docker GPU
docker run --rm --gpus all nvidia/cuda:12.0.1-base-ubuntu22.04 nvidia-smi

# K3d cluster and K8s nodes
k3d cluster list
kubectl get nodes

# GPU Operator pods
kubectl -n gpu-operator get pods

# GPUs still allocatable?
kubectl get nodes "-o=custom-columns=NAME:.metadata.name,GPU:.status.allocatable.nvidia\.com/gpu"
```

### 2. If GPU access is lost

Restart the K3d cluster to refresh device handles:

```bash
k3d cluster stop <your-cluster-name>
k3d cluster start <your-cluster-name>
```

Wait for GPU Operator pods to recover:

```bash
kubectl -n gpu-operator wait --for=condition=ready pod --all --timeout=300s
```

If that doesn't work, restart Docker and then restart the cluster:

```bash
sudo systemctl restart docker
k3d cluster start <your-cluster-name>
```

No reinstall is needed; your configuration persists.

<br/>

## Verify

### Step 1: Run a test Workload
Deploy a pod that requests a GPU to ensure everything is connected:

``` shell
kubectl apply -f k8s-setup/cuda-test.yml
```

### Step 2: Check the test pod output

``` shell
kubectl wait --for=condition=ready pod/gpu-test --timeout=120s || true
kubectl logs gpu-test
```

You should see `nvidia-smi` output showing your GPU inside the pod. Clean up when done:

``` shell
kubectl delete pod gpu-test
```