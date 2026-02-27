https://k3d.io/v5.7.2/usage/advanced/cuda/#the-nvidia-device-plugin

https://github.com/k3d-io/k3d/issues/1108

```shell
# use the custom image
k3d cluster create gputest --image=gputest/k3d-gpu-support:1.0.1 --gpus=all --api-port 6443
```

### test
```shell
# test with apod
kubectl apply -f cuda-vector-add.yaml
kubectl logs cuda-vector-add
```