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

# restart
kubectl rollout restart deploy -n dapr-system
```

## Step 3: Scope the Injector Webhook

In this cluster, the injector webhook must be namespace-scoped before enabling fail-closed admission. If `failurePolicy` is set to `Fail` while the webhook still matches all pod creations, a reboot or control-plane restart can deadlock Kubernetes admission: Dapr control-plane pods and unrelated workloads such as Argo CD will both be blocked waiting on the injector.

Use the namespace label `dapr-injection=enabled` on application namespaces that should be eligible for Dapr sidecar injection. Do not add this label to infrastructure namespaces such as `argocd`, `kube-system`, or `dapr-system`.

Patch the webhook so it only matches labeled namespaces and fails closed inside those namespaces:

``` shell
kubectl patch mutatingwebhookconfiguration dapr-sidecar-injector \
    --type='json' \
    -p='[
        {
            "op": "add",
            "path": "/webhooks/0/namespaceSelector",
            "value": {
                "matchLabels": {
                    "dapr-injection": "enabled"
                }
            }
        },
        {
            "op": "replace",
            "path": "/webhooks/0/failurePolicy",
            "value": "Fail"
        }
    ]'
```

Then label only the application namespaces that should participate in Dapr sidecar injection:


``` yaml
# Example namespace manifest:

apiVersion: v1
kind: Namespace
metadata:
    name: cezzis
    labels:
        app.kubernetes.io/part-of: cezzis
        app.kubernetes.io/managed-by: argocd
        dapr-injection: enabled
    annotations:
        argocd.argoproj.io/sync-wave: "0"
```

Verify the webhook and namespace labels:

``` shell
kubectl get mutatingwebhookconfiguration dapr-sidecar-injector -o yaml
kubectl get ns --show-labels
```

If the cluster is ever left in the unsafe all-pods configuration with `failurePolicy=Fail`, recover by restoring the default behavior first:

``` shell
kubectl patch mutatingwebhookconfiguration dapr-sidecar-injector \
    --type='json' \
    -p='[{"op": "replace", "path": "/webhooks/0/failurePolicy", "value": "Ignore"}]'
```

## Optional: Allow external access to the scheduler
Optionally add a node port to the scheduler so it's available outside the cluster.  This is helpful for local development. 

__NOTE:__ For local app development I am using a docker compose setup on the host machine
since the k3d dapr-system requires tls and mtls.  Apps within the cluster connect to the k3d clusters dapr system though.

``` shell
# This only works with the dapr platform stack, not the dapr-system because the dapr system
# requires tls and is a pain from local dev
kubectl apply -f k8s-setup/dapr-scheduler-nodeport.yml

```

## Optional: Install the dapr dashboard
Optionally install the dapr dashboard

``` shell
helm install dapr-dashboard dapr/dapr-dashboard --namespace dapr-system
```