# Enable GPU Support (Experimental)

## Enable NVIDIA GPU Support on the Host

### Step 1: Install NVIDIA Drivers on the Host

Make sure your host system has the latest NVIDIA drivers installed.  
You can check with:

```bash
nvidia-smi
```

If not installed, follow the official NVIDIA instructions for your OS.

### Step 2: Install NVIDIA Container Toolkit

This allows Docker to use the GPU.

```bash
# Add the package repositories
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update
sudo apt-get install -y nvidia-docker2
sudo systemctl restart docker
```

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
  --set driver.enabled=false \
  --set toolkit.enabled=false
```
_Note: driver.enabled=false is used because k3d runs inside Docker, which already has access to the host's drivers._


### Step 3: Enable GPU sharing

``` shell
kubectl apply -f k8s-setup/time-slicing-config-all.yaml
kubectl patch clusterpolicy/cluster-policy -n gpu-operator --type merge -p '{"spec": {"devicePlugin": {"config": {"name": "time-slicing-config-all", "default": "any"}}}}'
```

### Step 4: Verify GPU Availability
Once the operator pods are running, verify the nodes have allocatable GPUs:

``` shell
kubectl get nodes "-o=custom-columns=NAME:.metadata.name,GPU:.status.allocatable.nvidia\.com/gpu"
```

<br/>

## Verify

### Step 1: Run a test Workload
Deploy a pod that requests a GPU to ensure everything is connected:

``` shell
kubectl apply -f k8s-setup/cuda-test.yml
```