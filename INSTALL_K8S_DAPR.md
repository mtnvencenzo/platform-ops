# Installing DAPR into a K8s Cluster
[<< back](INSTALL.md)

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
    --set global.mtls.enabled=false

# verify the pods are running
kubectl get pods --namespace dapr-system

# Patch the injector webhook failure policy so it fails if it cant inject dapr
# into the pod.  This will force a retry to get dapr in there
kubectl patch mutatingwebhookconfiguration dapr-sidecar-injector \
    --type='json' \
    -p='[{"op": "replace", "path": "/webhooks/0/failurePolicy", "value": "Fail"}]'
```

## ~~Step 3: [Optional] Allow external access to the scheduler~~
Optionally add a node port to the scheduler so it's available outside the cluster.  This is helpful for local development. 

__NOTE:__ For local app development I am using a docker compose setup on the host machine
since the k3d dapr-system requires tls and mtls.  Apps within the cluster connect to the k3d clusters dapr system though.

``` shell
# This only works with the dapr platform stack, not the dapr-system because the dapr system
# requires tls and is a pain from local dev
kubectl apply -f k8s-setup/dapr-scheduler-nodeport.yml
```

## Step 4: [Optional] Install the dapr dashboard
Optionally install the dapr dashboard

``` shell
helm install dapr-dashboard dapr/dapr-dashboard --namespace dapr-system
```