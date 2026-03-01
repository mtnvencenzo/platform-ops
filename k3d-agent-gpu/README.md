
# k3d-agent-gpu

## Overview

This directory contains resources to build a custom K3s agent Docker image with NVIDIA GPU support, CUDA, and time-slicing configuration for use in GPU-enabled Kubernetes clusters. The image is designed for use with k3d and includes the NVIDIA container toolkit and device plugin for GPU workloads.

## Features

- Custom K3s agent image with CUDA and NVIDIA container toolkit
- Time-slicing configuration for GPU sharing
- Device plugin DaemonSet for GPU management
- Ready for use with k3d and Rancher

## Prerequisites

- Docker installed and running
- Access to Azure Container Registry (ACR) if you wish to push the image
- Azure CLI installed and authenticated (for pushing to ACR)

## Building the Docker Image

The Makefile provides convenient targets to build and push the Docker image. The main targets are:

- `build-k3d`: Builds the Docker image using the provided Dockerfile
- `push-k3d`: Pushes the built image to the configured Azure Container Registry (ACR)

### Build the Image

To build the Docker image locally, run:

```sh
make build-k3d
```

This will build the image using the parameters defined in the Makefile (repository, tag, etc.).

### Push the Image to ACR

To push the built image to your Azure Container Registry, run:

```sh
make push-k3d
```

This will log in to the ACR and push the image using the tag specified in the Makefile.

## Usage Example

To use the custom image with k3d:

```sh
k3d cluster create gputest --image=<your-repo>/<your-org>/k3s:<tag> --gpus=all --api-port 6443
```

Replace `<your-repo>`, `<your-org>`, and `<tag>` with the values from your Makefile.

## Testing GPU Support

To test GPU functionality in your cluster:

```sh
kubectl apply -f cuda-vector-add.yaml
kubectl logs cuda-vector-add
```

## GPU Time-Slicing

If more pods need to use the GPU, update the `time-slicing-config-all.yaml` file to include more replicas. Then, reapply the timeslicing configMap:

```sh
kubectl apply -f time-slicing-config-all.yaml
```

Restart the device-plugin DaemonSet:

```sh
kubectl rollout restart daemonset nvidia-device-plugin-daemonset -n kube-system
```

> **Note:** Make sure to create a new Docker image and push it to the registry so that the next time the cluster is built it will use the recent changes.

## File Structure

- `dockerfile`: Dockerfile for building the custom K3s agent image
- `Makefile`: Build and push automation
- `device-plugin-daemonset.yaml`: DaemonSet manifest for NVIDIA device plugin
- `time-slicing-config-all.yaml`: GPU time-slicing configuration
- `cuda-vector-add.yaml`: Test manifest for CUDA
- `README.md`: This documentation

## References

- [K3d CUDA GPU Support](https://k3d.io/v5.7.2/usage/advanced/cuda/#the-nvidia-device-plugin)
- [k3d Issue #1108](https://github.com/k3d-io/k3d/issues/1108)
- [K3s Documentation](https://rancher.com/docs/k3s/latest/en/)
- [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/overview.html)
- [k3d](https://k3d.io/)