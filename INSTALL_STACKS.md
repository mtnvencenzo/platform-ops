## Integrate namespaces with azure container registry if they pull images from there
Run this command for every namespace where you plan to deploy images from ACR (including the cezzis namespace from your example):
```
kubectl create namespace cezzis

kubectl create secret docker-registry acr-pull-secret \
  --docker-server=acrveceusgloshared001.azurecr.io \
  --docker-username=acrveceusgloshared001 \
  --docker-password=<ACR-Admin-Password> \
  -n cezzis
```
Then, update your Job/App manifest to include imagePullSecrets so the cluster knows which key to use:

``` yaml
spec:
  template:
    spec:
      imagePullSecrets:
        - name: acr-pull-secret
      containers:
        - name: cezzis-com-local-bootstrapper
          image: acrveceusgloshared001.azurecr.io/cezziscombootstrapper:latest
```


## Commands to install the various stacks in the repo via argocd

``` shell
# postgres stack
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/postgres-stack/argocd/postgres-stack-app.yaml

# ai stack
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/ai-stack/argocd/ai-stack-app.yaml

# azure stack
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/azure-stack/argocd/azure-stack-app.yaml

# rabbitmq stack
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/rabbitmq-stack/argocd/rabbitmq-stack-app.yaml

# redis stack
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/redis-stack/argocd/redis-stack-app.yaml

# kafka stack (kraft)
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/kafka-stack/argocd/kafka-stack-kraft-app.yaml

# dapr stack
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/dapr-stack/argocd/dapr-stack-app.yaml

# elastic stack
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/elastic-stack/argocd/elastic-stack-app.yaml

# openobserve stack
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/openobserve-stack/argocd/openobserve-stack-app.yaml

```

## Commands to install cezzis.com stacks via argo cd with the image updater

``` shell
# postgres stack
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/cezzis-com-local-bootstrapper/refs/heads/main/.iac/argocd/cezzis-com-local-boostrapper.yaml
```