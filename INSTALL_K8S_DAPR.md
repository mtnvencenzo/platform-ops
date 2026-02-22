# Installing DAPR into a K8s Cluster
https://docs.dapr.io/operations/hosting/kubernetes/kubernetes-deploy/

## Installing with Helm

## Step 1: Add the dapr repo
``` shell
# Add the official Dapr Helm chart.
helm repo add dapr https://dapr.github.io/helm-charts/
helm repo update

# See which chart versions are available
helm search repo dapr --devel --versions
```

## Step 2: Install the helm chart

``` shell
helm upgrade --install dapr dapr/dapr \
    --version=1.16.9 \
    --namespace dapr-system \
    --create-namespace \
    --set global.ha.enabled=false \
    --wait
    
# verify the pods are running
kubectl get pods --namespace dapr-system
```

## Step 3: [Optional] Allow external access to the scheduler
Optionally add a node port to the scheduler so it's available outside the cluster.
This is helpful for local development.

``` shell
kubectl apply -f k8s-setup/dapr-scheduler-nodeport.yml
```

## Step 4: [Optional] Install the dapr dashboard
Optionally install the dapr dashboard

``` shell
helm install dapr-dashboard dapr/dapr-dashboard --namespace dapr-system
```