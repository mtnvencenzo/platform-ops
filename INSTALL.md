# K8s / K3d / Docker Setup
Install and create the cluster using docker and k3d

<br />

## Install and Setup Docker CE
[Docker Setup](INSTALL_DOCKER.md)


## Install Utils (Kubectl, Helm)
[Utils install steps](INSTALL_K8S_UTILS.md)


## Performance Tune Host Machine
Use the following settings to allow for heavy workloads on you host machine

[Performance Tune](INSTALL_PERFORMANCE.md)


## Install K3D 
Use the official installation script to install the latest version of k3d: 

[K3D Install](INSTALL_K3D.md)


## Enable GPU in Cluster

> **Radeon GPU (ROCm):** GPU devices are passed through to K3d as volume mounts (`/dev/kfd`, `/dev/dri`) during cluster creation — no custom K3s image required. Install the AMD k8s device plugin to make GPUs allocatable.

> **NVIDIA GPU:** GPU support with NVIDIA drivers is part of the custom k3s image used when creating the k3d cluster. See the [GPU readme](./k3d-agent-gpu/README.md) for information on running the custom k3s image and building new custom images.

[GPU setup steps](INSTALL_K8S_GPU.md)


## Install a Cluster Manager (Portainer Recommended)
Rancher is a resource hog and crashes a lot in a home lab setup.  I suggest portainer as a simpler and more efficent cluster manager.  
[Portainer install steps](INSTALL_K8S_PORTAINER.md)  _OR_  [Rancher install steps](INSTALL_K8S_RANCHER.md)



## Install ArgoCD
[ArgoCD install steps](INSTALL_K8S_ARGOCD.md)


## Install Dapr
[Dapr install steps](INSTALL_K8S_DAPR.md)


## Install Various Stacks and Applications
[Stack install steps](INSTALL_STACKS.md)


## Complete Cluster Teardown & Cleanup
[Uninstall / cleanup steps](UNINSTALL.md)

