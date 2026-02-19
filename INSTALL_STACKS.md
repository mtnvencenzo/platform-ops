# Commands to install the various stacks in the repo via argocd

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

```