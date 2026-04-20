# Install Platform Stacks
[<< back](INSTALL.md)

## Integrate namespaces with azure container registry if they pull images from there
Run this command for every namespace where you plan to deploy images from ACR:
```
kubectl create secret docker-registry acr-pull-secret \
  --docker-server=acrveceusgloshared001.azurecr.io \
  --docker-username=acrveceusgloshared001 \
  --docker-password=$ACR_ACCESS_KEY \
  -n <namespace>
```
Then, update your Job/App manifest to include imagePullSecrets so the cluster knows which key to use:

``` yaml
spec:
  template:
    spec:
      imagePullSecrets:
        - name: acr-pull-secret
      containers:
        - name: <app-name>
          image: acrveceusgloshared001.azurecr.io/<image>:latest
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
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/kafka-stack/argocd/kafka-stack-kraft-3broker-app.yaml
OR
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/kafka-stack/argocd/kafka-stack-kraft-app.yaml
OR
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/kafka-stack/argocd/kafka-stack-zookeeper-app.yaml

# dapr stack
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/dapr-stack/argocd/dapr-stack-app.yaml

# elastic stack
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/elastic-stack/argocd/elastic-stack-app.yaml

# openobserve stack
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/openobserve-stack/argocd/openobserve-stack-app.yaml

```

## Troubleshooting

### Clearing a stuck argocd application from deleting
This is typically due to stuck finalizers.

``` shell
kubectl patch application/<appname> --type json --patch='[ { "op": "remove", "path": "/metadata/finalizers" } ]' -n argocd
```


### Deleting pods in certain statuses

``` shell
kubectl get pods -n ai-platform --no-headers | awk '$3=="UnexpectedAdmissionError" {print $1}' | xargs -r kubectl delete pod -n ai-platform
```