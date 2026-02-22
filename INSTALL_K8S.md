# K8s / K3d setup
Install and create the cluster using docker and k3d

<br />

## Install Docker CE on Ubuntu 
If not already installed, follow these steps to install Docker Engine: 

### Step 1: Update and install dependencies

``` bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release
```

### Step 2: Add Docker Official GPG Key & Repository

``` bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Step 3: Install Docker Engine

``` bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Step 4: Verify Installation

``` bash
sudo docker run hello-world
```

### Step 5: Configure Docker without Sudo (Optional but Recommended) 

``` bash
sudo usermod -aG docker $USER
# Log out and log back in for changes to take effect
```

<br />

## Install Utils (Kubectl, Helm)
[Utils install steps](INSTALL_K8S_UTILS.md)

<br />

## Performance Tune Host Machine
Use the following settings to allow for heavy workloads on you host machine

### Step 1: Ensure Inotify values are sufficent

First see what values are on the system. 
``` shell
sysctl fs.inotify
#fs.inotify.max_queued_events = 16384
#fs.inotify.max_user_instances = 65536
#fs.inotify.max_user_watches = 1048576
```

### Step 2: Append settings to /etc/sysctl.d/99-k3d-heavy-workloads.conf
``` shell
sudo tee /etc/sysctl.d/99-k3d-heavy-workloads.conf <<EOF
vm.max_map_count=262144
fs.inotify.max_user_instances=65536
EOF

# Apply the changes immediately without rebooting
sudo sysctl --system 

# verify
cat /proc/sys/vm/max_map_count
cat /proc/sys/fs/inotify/max_user_instances
cat /etc/sysctl.d/99-k3d-heavy-workloads.conf
```

### Step 3: Add file descriptor limits to /etc/security/limits.conf
``` shell
sudo tee -a /etc/security/limits.conf <<EOF
* soft nofile 65536
* hard nofile 65536
EOF

# verify
ulimit -n
ulimit -Sn  # View current Soft limit
ulimit -Hn  # View current Hard limit
```

<br/>

## Install K3D 
Use the official installation script to install the latest version of k3d: 

### Step 1: Install k3d
``` shell
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```


### Step 2: Create the Kubernetes Cluster 
Create a new cluster using k3d, which spins up K3s inside Docker: 

``` shell
k3d cluster delete prd-local-apps-001 || true

# Setup a d1rectory for k3d data for better performance
sudo mkdir -p /mnt/k3d-data
sudo chmod -R 777 /mnt/k3d-data

k3d cluster create prd-local-apps-001 \
  -p "8080:80@loadbalancer" \
  -p "8443:443@loadbalancer" \
  -p "30672:30672@loadbalancer" \
  -p "30092:30092@loadbalancer" \
  -p "9000:9000@loadbalancer" \
  --volume "/mnt/k3d-data:/var/lib/rancher/k3s/storage@all" \
  --k3s-arg "max-pods=200@server:*;agent:*" \
  --api-port 6443 \
  --servers 1 \
  --agents 2 \
  --agents-memory 12G \
  --runtime-label "com.k3d.io.ulimit.nofile=65536:65536@server:*;agent:*" \
  --gpus all \
  --k3s-arg "--disable=metrics-server@server:0" \
  --runtime-label "com.k3d.io.ulimit.nofile=65536:65536@server:*;agent:*"

# add more ports to the lb (adding node ports is not enough, need to tell the cluster lb to map the ports as well)
# The additional cluter lb port mappings are already added to the cluster create command above
# rabbitmq
# k3d cluster edit prd-local-apps-001 --port-add 30672:30672@loadbalancer
# kafka
# k3d cluster edit prd-local-apps-001 --port-add 30092:30092@loadbalancer
```

### Step 2: Verify the Cluster 
Use kubectl (installed automatically with k3d or installed separately) to verify: 

```  shell
k3d cluster list
```

<br/> 

## [Experimental / Optional] Enable GPU in Cluster
[Gpu setup steps](INSTALL_K8S_GPU.md)

## Install Rancher
[Rancher install steps](INSTALL_K8S_RANCHER.md)

## Install ArgoCD
[ArgoCD install steps](INSTALL_K8S_ARGOCD.md)

## Install Dapr
[Dapr install steps](INSTALL_K8S_DAPR.md)

## Install Various Stacks and Applications
[Stack install steps](INSTALL_STACKS.md)



## [Optional] Traefik Dashboard
``` shell
kubectl apply -f k8s-setup/traefik-dashboard.yml
```