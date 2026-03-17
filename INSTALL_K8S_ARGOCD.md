# Install Argo CD
[<< back](INSTALL.md)

To install Argo CD into your k3d cluster alongside Rancher, the most efficient method is using its official Helm chart.

## [Optional] Install the ArgoCD Cli

``` shell
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

## Install ArgoCd into the Cluster

### Step 1: Add the Argo helm repo

``` shell
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

### Step 2: Install Argo CD
Create a dedicated namespace and install the chart. Since this is a local k3d environment, you can use the non-HA (High Availability) version to save resources.
The `argocd-helm-values.yml` file contains cluster-wide configuration such as RBAC policies.

``` shell
helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace \
  --version 9.4.7 \
  -f ./k8s-setup/argocd-helm-values.yml

# verify
kubectl get pods -n argocd
```

### Upgrading ArgoCD or Applying Values Changes
When you need to apply changes to `argocd-helm-values.yml` or upgrade the chart version, use `helm upgrade`. The `--reuse-values` flag merges your values file on top of existing settings and `--version` pins the chart to avoid an unintentional upgrade.

``` shell
helm upgrade argocd argo/argo-cd \
  --namespace argocd \
  -f ./k8s-setup/argocd-helm-values.yml \
  --reuse-values \
  --version 9.4.7

# verify
kubectl get pods -n argocd
```

_Note: Run `helm list -n argocd` to check the currently installed chart version before upgrading._

### Step 3: Allow insecure connections to Argo CD
``` shell
kubectl patch cm argocd-cmd-params-cm -n argocd -p '{"data": {"server.insecure": "true"}}'
kubectl rollout restart deployment argocd-server -n argocd
```

### Step 4: Allow ArgoCD to manage EndpointSlice resources
By default ArgoCD excludes `Endpoints` and `EndpointSlice` resources. Since we use manually-defined EndpointSlice resources (e.g. `ollama-host-service` for routing to host services), we need to remove `EndpointSlice` from the exclusion list.

``` shell
KUBE_EDITOR=nano kubectl edit configmap argocd-cm -n argocd
```

In the editor, find the first exclusion block under `resource.exclusions` and remove the `- discovery.k8s.io` and `- EndpointSlice` lines:

``` yaml
# Before:
- apiGroups:
  - ''
  - discovery.k8s.io    # <-- remove this line
  kinds:
  - Endpoints
  - EndpointSlice       # <-- remove this line

# After:
- apiGroups:
  - ''
  kinds:
  - Endpoints
```

Then restart the server:

``` shell
kubectl rollout restart deployment argocd-server -n argocd
```

### Step 5: Create the ingress for the UI
kubectl apply -f ./k8s-setup/argocd-ingress.yml

### Step 6: Retreive the auto-generated admin password

``` shell
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

### Step 7: Browse to Argo CD and Login

http://argocd.127.0.0.1.sslip.io
Username: admin

_Note: if using an non standard (80) port number like 8080 for the cluster then that port would need to be used on the ingress urls when access them_

<br />

## Add Helm External Secret Support
In order to use external secret stores (like azure keyvault) you need to install the helm chart.

``` shell
# Add the helm repo
helm repo add external-secrets https://charts.external-secrets.io
helm repo update

# Install the chart
helm install external-secrets external-secrets/external-secrets \
  -n external-secrets \
  --create-namespace

```

<br />

## Argo CD Image Updater
Add the argo cd image updater to the cluster and within the argocd namespace

``` shell
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/config/install.yaml
```

### Configure Argocd Image Updater for Azure Container Registry 
The Image Updater needs its own access to the registry's API to check for new tags

``` shell
kubectl create secret docker-registry acr-pull-secret \
  --docker-server=acrveceusgloshared001.azurecr.io \
  --docker-username=acrveceusgloshared001 \
  --docker-password=$ACR_ACCESS_KEY \
  -n argocd
```

### Edit the config map
Because the install was from github we have to manually edit the config map to add the registry config.  Inside the editor, add the registries.conf block under data:.

Navigate to Rancher and select More Resources > Core > ConfigMaps and find the argocd-image-updater-config.  It should look something like this after adding he repository to the data section:

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-image-updater-config
  namespace: argocd
data:
  registries.conf: |
    registries:
    - name: Azure Container Registry
      prefix: acrveceusgloshared001.azurecr.io
      api_url: https://acrveceusgloshared001.azurecr.io
      credentials: pullsecret:argocd/acr-pull-secret
```

### Use the git writeback method

#### Step 1: Create a github app
Name: mtnvencenzo-argocd-writeback-app
Home page is reuqired  
Make sure it has repository permissions read/write.  

After creating it generate and download the private key
After creating it click 'Install App' and select for all repositories.
-- You will be redirected to the installation/config page.  The installation Id is in the browser url, you will need this.  Ex https://github.com/settings/installations/000000000

installationid=111603325

#### Step 2: Create the secret in the cluster (argocd namespace)

``` shell
kubectl -n argocd create secret generic git-creds \
  --from-literal=githubAppID=2918282 \
  --from-literal=githubAppInstallationID=111603325 \
  --from-file=githubAppPrivateKey=mtnvencenzo-argocd-writeback-app.private-key.pem
```

#### Restart the image updater
The Image Updater doesn't always hot-reload registries.conf changes. Restart the pod to be safe:

``` shell
kubectl rollout restart deployment argocd-image-updater-controller -n argocd
```

#### Step 3: Update each repositories bypass list to include the app __mtnvencenzo-argocd-writeback-app__
Make sure to select 'Exempt from rules'

#### Step 4:  Ensure you exclude the directory that the argo cd version file gets written to from triggering a build or you'll end up in an infinite look of creating new images