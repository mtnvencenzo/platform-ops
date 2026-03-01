# Install K3D 
[<< back](INSTALL.md)

Use the official installation script to install the latest version of k3d: 

## Step 1: Install k3d
``` shell
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```


## Step 2: Create the Kubernetes Cluster 
Create a new cluster using k3d, which spins up K3s inside Docker: 

> NOTE: The volume mounts assume your NVIDIA userland tools (like nvidia-smi) are installed at /usr/bin/nvidia-smi and libraries at /usr/lib/x86_64-linux-gnu on the host.  If your system uses different paths, adjust accordingly.

``` shell
k3d cluster delete prd-local-apps-001 || true

# Setup a d1rectory for k3d data for better performance
sudo rm -rf /mnt/k3d-data
sudo mkdir -p /mnt/k3d-data
sudo chmod -R 777 /mnt/k3d-data

k3d cluster create prd-local-apps-001 \
  -p "80:80@loadbalancer" \
  -p "443:443@loadbalancer" \
  -p "9000:9000@loadbalancer" \
  -p "30672:30672@loadbalancer" \
  -p "30092:30092@loadbalancer" \
  -p "30432:30432@loadbalancer" \
  --volume "/mnt/k3d-data:/var/lib/rancher/k3s/storage@all" \
  --k3s-arg "max-pods=200@server:*;agent:*" \
  --api-port 6443 \
  --servers 1 \
  --agents 2 \
  --agents-memory 12G \
  --runtime-label "com.k3d.io.ulimit.nofile=65536:65536@server:*;agent:*" \
  --gpus all \
  --image acrveceusgloshared001.azurecr.io/cezzis/k3s:v1.31.5-k3s1.12.2.2-cuda12.2.2-base-ubuntu22.04-v1 \
  --k3s-arg "--disable=metrics-server@server:0" \
  --k3s-arg "--kubelet-arg=eviction-hard=memory.available<256Mi,nodefs.available<5%@agent:*" \
  # --volume /usr/bin/nvidia-smi:/usr/bin/nvidia-smi@all \
  # --volume /usr/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu@all \



# add more ports to the lb (adding node ports is not enough, need to tell the cluster lb to map the ports as well)
# The additional cluter lb port mappings are already added to the cluster create command above
#
# rabbitmq
# k3d cluster edit prd-local-apps-001 --port-add 30672:30672@loadbalancer
#
# kafka
# k3d cluster edit prd-local-apps-001 --port-add 30092:30092@loadbalancer
#
# postgres
# k3d cluster edit prd-local-apps-001 --port-add 30432:30432@loadbalancer
#
# portainer
# k3d cluster edit prd-local-apps-001 --port-add 9000:9000@loadbalancer
```

## Step 3: Verify the Cluster 
Use kubectl (installed automatically with k3d or installed separately) to verify: 

```  shell
k3d cluster list
```