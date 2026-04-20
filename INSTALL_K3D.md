# Install K3D 
[<< back](INSTALL.md)

Use the official installation script to install the latest version of k3d: 

## Step 1: Install k3d
``` shell
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```


## Step 2: Create the Kubernetes Cluster 
Create a new cluster using k3d, which spins up K3s inside Docker. The common setup steps below apply to both GPU types. Then follow the section for your GPU hardware.

``` shell
# Cleanup existing cluster
k3d cluster delete prd-local-apps-001 || true
sudo systemctl restart docker.service

# Setup a d1rectory for k3d data for better performance
sudo rm -rf /mnt/data/k3d-node-data
sudo mkdir -p /mnt/data/k3d-node-data
sudo chmod -R 777 /mnt/data/k3d-node-data

# Setup a d1rectory for k3d application data.  this will be mounted for any pod to use
sudo rm -rf /mnt/data/k3d-app-data
sudo mkdir -p /mnt/data/k3d-app-data
sudo chmod -R 777 /mnt/data/k3d-app-data

# Pre-create the k3d network with a fixed subnet so the gateway IP (172.18.0.1) is deterministic.
# This ensures k8s Endpoints resources that reference the host (e.g. ollama-host-service) remain stable across cluster rebuilds.
docker network create k3d-prd-local-apps-001 --subnet=172.18.0.0/16 --gateway=172.18.0.1 || true
```

### Radeon GPU (ROCm)

AMD GPUs use standard Linux device nodes (`/dev/kfd`, `/dev/dri`) and do not require a custom K3s image or special container runtime. The devices are passed through as volume mounts.

> **Note:** The `/dev/dri` volume mount exposes **all** render devices (including integrated GPUs) to the K3d nodes. GPU selection for workloads is handled at the application level via the `ROCR_VISIBLE_DEVICES` environment variable — see [INSTALL_K8S_GPU.md](INSTALL_K8S_GPU.md) for details. On Ubuntu 24.04+ (cgroup v2), you may also need to apply device cgroup rules after cluster creation — see Step 1 in [INSTALL_K8S_GPU.md](INSTALL_K8S_GPU.md).

``` shell
# Create the cluster with Radeon GPU device passthrough
k3d cluster create prd-local-apps-001 \
  -p "80:80@loadbalancer" \
  -p "443:443@loadbalancer" \
  -p "9000:9000@loadbalancer" \
  -p "30672:30672@loadbalancer" \
  -p "30092:30092@loadbalancer" \
  -p "30432:30432@loadbalancer" \
  --volume "/mnt/data/k3d-node-data:/var/lib/rancher/k3s/storage@all" \
  --volume "/mnt/data/k3d-app-data:/pods@all" \
  --volume "/dev/kfd:/dev/kfd@all" \
  --volume "/dev/dri:/dev/dri@all" \
  --k3s-arg "max-pods=200@server:*;agent:*" \
  --api-port 6443 \
  --servers 1 \
  --agents 2 \
  --agents-memory 12G \
  --runtime-label "com.k3d.io.ulimit.nofile=65536:65536@server:*;agent:*" \
  --k3s-arg "--disable=metrics-server@server:0" \
  --k3s-arg "--kubelet-arg=eviction-hard=memory.available<256Mi,nodefs.available<5%@agent:*"

# To add more ports to the lb (adding node ports is not enough, need to tell the cluster lb to map the ports as well)
# k3d cluster edit prd-local-apps-001 --port-add 30672:30672@loadbalancer
```

### NVIDIA GPU

> NOTE: The volume mounts assume your NVIDIA userland tools (like nvidia-smi) are installed at /usr/bin/nvidia-smi and libraries at /usr/lib/x86_64-linux-gnu on the host.  If your system uses different paths, adjust accordingly.

``` shell
# Pull the custom k3s image so we have it local
image_name=acrveceusgloshared001.azurecr.io/cezzis/k3s:v1.31.5-k3s1.12.2.2-cuda12.2.2-base-ubuntu22.04-v1
az acr login -n acrveceusgloshared001
docker pull "$image_name"

# Create the cluster with NVIDIA GPU support
k3d cluster create prd-local-apps-001 \
  -p "80:80@loadbalancer" \
  -p "443:443@loadbalancer" \
  -p "9000:9000@loadbalancer" \
  -p "30672:30672@loadbalancer" \
  -p "30092:30092@loadbalancer" \
  -p "30432:30432@loadbalancer" \
  --volume "/mnt/data/k3d-node-data:/var/lib/rancher/k3s/storage@all" \
  --volume "/mnt/data/k3d-app-data:/pods@all" \
  --k3s-arg "max-pods=200@server:*;agent:*" \
  --api-port 6443 \
  --servers 1 \
  --agents 2 \
  --agents-memory 12G \
  --runtime-label "com.k3d.io.ulimit.nofile=65536:65536@server:*;agent:*" \
  --gpus all \
  --image "$image_name" \
  --k3s-arg "--disable=metrics-server@server:0" \
  --k3s-arg "--kubelet-arg=eviction-hard=memory.available<256Mi,nodefs.available<5%@agent:*"

# To add more ports to the lb (adding node ports is not enough, need to tell the cluster lb to map the ports as well)
# k3d cluster edit prd-local-apps-001 --port-add 30672:30672@loadbalancer
```

## Step 3: Verify the Cluster 
Use kubectl (installed automatically with k3d or installed separately) to verify: 

```  shell
k3d cluster list
```