# Enable GPU Support in K3D Cluster
[<< back](INSTALL.md)

Follow the section that matches your GPU hardware.

<br/>

## Radeon GPU (ROCm)

AMD GPUs use standard Linux device nodes and a lightweight Kubernetes device plugin — no GPU operator or custom container runtime required.

### Step 1: Verify ROCm Host Prerequisites

Before installing the device plugin, confirm ROCm is working on the host and that the K3d cluster was created with the GPU device volume mounts (see [INSTALL_K3D.md](INSTALL_K3D.md)):

```bash
# Host GPU detection
rocm-smi
ls /dev/kfd
ls /dev/dri/renderD*

# Verify devices are visible inside the K3d nodes
docker exec -it k3d-prd-local-apps-001-agent-0 ls /dev/kfd
docker exec -it k3d-prd-local-apps-001-agent-0 ls /dev/dri/
```

Verify that the K3d nodes can actually **access** the GPU devices (not just see the files). Ubuntu 24.04+ uses cgroup v2 by default, and bind-mounted devices may lack proper cgroup device-allow rules:

```bash
# Check cgroup version (cgroup2fs = v2)
stat -fc %T /sys/fs/cgroup/

# Test actual device access inside a K3d node — this reads GPU topology info
docker exec -it k3d-prd-local-apps-001-agent-0 cat /sys/class/kfd/kfd/topology/nodes/1/properties | head -5
```

If the topology read fails with a permission error, the cgroup v2 device filter is blocking access. Apply device cgroup rules to each K3d node container:

```bash
# Allow access to renderD* devices (major 226) and kfd (major 236)
for cid in $(docker ps -q --filter "label=app=k3d"); do
  docker update --device-cgroup-rule='c 226:* rwm' --device-cgroup-rule='c 236:* rwm' "$cid"
done
# Restart the K3d node containers to apply
docker restart $(docker ps -q --filter "label=app=k3d")
```

Also confirm the K3d nodes run as root (required for device access without additional group membership):

```bash
docker exec -it k3d-prd-local-apps-001-agent-0 id
```

You should see `uid=0(root)`. If workloads run as non-root users, pods may need `securityContext.runAsGroup` set to the `render` group GID for `/dev/dri` access.

> **RDNA 4 Note:** ROCm support for RDNA 4 (Radeon RX 9000 series) is relatively new. If `rocm-smi` does not detect your card, ensure you are running the latest ROCm version and a kernel ≥ 6.12.

### Step 2: Verify GPU Ordering Inside K3d

If your system has multiple AMD GPUs (e.g. discrete + integrated), verify that the device ordering inside the K3d nodes matches the host. Run `rocm-smi` from within a K3d node using the ROCm container image:

```bash
docker exec -it k3d-prd-local-apps-001-agent-0 \
  /bin/sh -c "ls -la /dev/kfd /dev/dri/"
```

The device ordering should match what you see on the host with `rocm-smi`. The test workload in Step 5 uses `ROCR_VISIBLE_DEVICES=0` to target the discrete GPU. For multiple discrete GPUs, use comma-separated indices (e.g. `0,1`). If your device ordering differs, adjust the value in [k8s-setup/rocm-test.yml](k8s-setup/rocm-test.yml) and the configmap in [stacks/ai-stack/k8s/configmap.yml](stacks/ai-stack/k8s/configmap.yml).

### Step 3: Install the AMD GPU Device Plugin

The [AMD k8s device plugin](https://github.com/ROCm/k8s-device-plugin) registers `amd.com/gpu` resources on nodes where AMD GPUs are detected:

```bash
kubectl create -f https://raw.githubusercontent.com/ROCm/k8s-device-plugin/master/k8s-ds-amdgpu-dp.yaml
```

Wait for the DaemonSet pods to be ready:

```bash
kubectl -n kube-system wait --for=condition=ready pod -l name=amdgpu-dp-ds --timeout=120s
```

### Step 4: Verify GPU Availability

Check that nodes report allocatable AMD GPUs:

```bash
kubectl get nodes "-o=custom-columns=NAME:.metadata.name,GPU:.status.allocatable.amd\.com/gpu"
```

You should see `1` (or more) in the GPU column for your agent nodes. If the GPU column shows `<none>`, the device plugin did not detect any GPUs — revisit Steps 1 and 2.

### Step 5: Run a Test Workload

Deploy a pod that requests an AMD GPU:

```bash
kubectl apply -f k8s-setup/rocm-test.yml
```

Check the output:

```bash
kubectl wait --for=condition=ready pod/gpu-test-rocm --timeout=120s || true
kubectl logs gpu-test-rocm
```

You should see `rocm-smi` output showing **only your discrete GPU(s)** (not the iGPU). The test pod uses `ROCR_VISIBLE_DEVICES=0` to target the discrete GPU. For multiple discrete GPUs, use comma-separated indices (e.g. `0,1`). If you see the wrong GPU or no GPU, check the device ordering in Step 2 and adjust `ROCR_VISIBLE_DEVICES` accordingly.

Clean up when done:

```bash
kubectl delete pod gpu-test-rocm
```

> **`HSA_OVERRIDE_GFX_VERSION`:** If `rocm-smi` detects the GPU but application workloads (PyTorch, Ollama, etc.) fail with "no GPU agent" or similar errors, your GPU architecture may not have pre-built kernels in the container image. Set `HSA_OVERRIDE_GFX_VERSION` in the pod environment to the major.minor.0 of your architecture (e.g. `12.0.0` for `gfx1201`, `11.0.0` for `gfx1100`). See [INSTALL_DOCKER.md](INSTALL_DOCKER.md) Step 5 for details.

### [Optional] Sleep/Wake GPU Recovery Hook

If your machine sleeps and resumes, GPU device handles inside the K3d Docker containers may become stale. You can install a systemd hook to automatically restart the K3d node containers after resume:

```bash
sudo tee /usr/lib/systemd/system-sleep/k3d-gpu-resume.sh << 'EOF'
#!/bin/bash
if [ "$1" = "post" ]; then
    # Give the amdgpu driver a moment to re-initialize
    sleep 2
    # Restart k3d server and agent node containers to pick up fresh device handles
    docker restart $(docker ps -q --filter "label=k3d.cluster=prd-local-apps-001" --filter "status=running") 2>/dev/null || true
fi
EOF
sudo chmod +x /usr/lib/systemd/system-sleep/k3d-gpu-resume.sh
```

This is optional — AMD GPUs generally recover from sleep/wake better than NVIDIA since the `amdgpu` driver is in-kernel. If you don't experience issues, you can skip this.

To manually recover without the hook:

```bash
k3d cluster stop prd-local-apps-001
k3d cluster start prd-local-apps-001
```

<br/>

## NVIDIA GPU

> **Important:** The K3d cluster must be created with the `--gpus all` flag so that GPU devices are forwarded into the K3s node containers. See the cluster creation command in the [K3d setup guide](INSTALL_K3D.md) and ensure `--gpus all` is included.

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

### Troubleshooting: GPU Access After Sleep/Wake

If your machine goes to sleep and resumes, GPU access may be lost in the K3d cluster due to stale device handles. Here are steps to check and recover:

#### 1. Check GPU access at each layer

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

#### 2. If GPU access is lost

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

### Verify

#### Step 1: Run a test Workload
Deploy a pod that requests a GPU to ensure everything is connected:

``` shell
kubectl apply -f k8s-setup/cuda-test.yml
```

#### Step 2: Check the test pod output

``` shell
kubectl wait --for=condition=ready pod/gpu-test --timeout=120s || true
kubectl logs gpu-test
```

You should see `nvidia-smi` output showing your GPU inside the pod. Clean up when done:

``` shell
kubectl delete pod gpu-test
```