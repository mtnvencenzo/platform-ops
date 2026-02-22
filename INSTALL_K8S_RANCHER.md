# Install Rancher

Rancher requires cert-manager to issue its own TLS certificate

## Step 1: Add the Jetstack Helm repository
``` shell
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

## Step 2: Install cert-manager in the cert-manager namespace

``` shell
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.19.3 \
  --set installCRDs=true

# verify
kubectl get pods -n cert-manager

# If reinstalling rancher and already had CRDs from cert manager run these commands
kubectl get crds | grep cattle
kubectl delete crd $(kubectl get crds | grep cattle | awk '{print $1}')
kubectl get crds | grep rancher
kubectl delete crd $(kubectl get crds | grep rancher | awk '{print $1}')
```
_Note: Check the cert-manager documentation for the latest version and CRD installation instructions._

## Step 3: Setup the cluster issuer

``` shell
kubectl apply -f k8s-setup/cluster-cert-issuer.yml

# verify
kubectl get clusterissuer selfsigned-cluster-issuer
```


## Step 4: Add the rancher helm repo
https://rancher.com/docs/

``` shell
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
```

## Step 5: Install Rancher, setting the hostname to localhost and specifying the exposed ports:

``` shell
# The hostname=localhost is important for local testing.
# ingress.tls.source=certmanager tells Rancher to use the cert-manager we installed.
# replicas=1 saves resources for a local, non-HA setup. 
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --create-namespace \
  --set hostname=localhost \
  --set replicas=1 \
  --set bootstrapPassword=admin \
  --set global.cattle.ingress.class=traefik \
  --set ingress.tls.source=secret \
  --set 'ingress.extraAnnotations.cert-manager\.io/cluster-issuer=selfsigned-cluster-issuer' \
  --set startupProbe.failureThreshold=100 \
  --set livenessProbe.initialDelaySeconds=60 \
  --set readinessProbe.initialDelaySeconds=60 \
  --set resources.requests.cpu=500m \
  --set resources.requests.memory=1024Mi
```

_The installation may take a few minutes. You can check the status by watching the pods in the cattle-system namespace_

``` shell
kubectl get pods -n cattle-system --watch
```

## Step 6: Patch rancher to use the cluster wide certificate issuer

``` shell
# shouldn't be needed anymore due to the helm install command and extra annotations
# kubectl patch certificate tls-rancher-ingress -n cattle-system \
# --type='json' -p='[{"op": "replace", "path": "/spec/issuerRef", "value": {"group": "cert-manager.io", "kind": "ClusterIssuer", "name": "selfsigned-cluster-issuer"}}]'
```

## Step 7: Login to Rancher

https://localhost:8443
__User:__ admin
__Password:__ admin

```
# To get the bootstrap password if you forgot that it was 'admin' :)
kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{ "\n" }}'
```