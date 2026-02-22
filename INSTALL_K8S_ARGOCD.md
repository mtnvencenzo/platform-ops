# Install Argo CD
To install Argo CD into your k3d cluster alongside Rancher, the most efficient method is using its official Helm chart.

## Step 1: Add the Argo helm repo

``` shell
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

## Step 2: Install Argo CD
Create a dedicated namespace and install the chart. Since this is a local k3d environment, you can use the non-HA (High Availability) version to save resources

``` shell
helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace

# verify
kubectl get pods -n argocd
```

## Step 3: Create an ingress in Rancher
Since you have Rancher and likely a k3d load balancer running, you can create an Ingress in the Rancher UI to avoid port-forwarding

When you create the Ingress in the Rancher UI, use the values confirmed from the command above: 

__Namespace:__ argocd  
__Name:__ argocd-ingress  
__Hostname:__ argocd.127.0.0.1.sslip.io (This resolves to your local machine)  
__Path (prefix):__ /  
__Target Service:__ argocd-server  
__Port:__ 80 (Choosing 80 avoids SSL mismatch issues during the initial setup) 

_Note: Before looking for the Target Service, ensure the Namespace dropdown at the top of the "Create Ingress" screen is set specifically to argocd. If it is set to "All Namespaces" or "Default," the argocd-server won't appear in the list.  Also, argocd should be added to the default project in Rancher_


## Step 4: Allow insecure connections to Argo CD
``` shell
kubectl patch cm argocd-cmd-params-cm -n argocd -p '{"data": {"server.insecure": "true"}}'
kubectl rollout restart deployment argocd-server -n argocd
```

## Step 5: Retreive the auto-generated admin password

``` shell
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

## Step 6: Browse to Argo CD and Login

http://argocd.127.0.0.1.sslip.io:8080
Username: admin

_Note: if using an non standard (80) port number like 8080 for the cluster then that port would need to be used on the ingress urls when access them_

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
  --docker-password=<ACR-Admin-Password> \
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
      credentials: secret:argocd/acr-pull-secret
```

### Restart the image updater
The Image Updater doesn't always hot-reload registries.conf changes. Restart the pod to be safe:
```
kubectl rollout restart deployment argocd-image-updater-controller -n argocd
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

#### Step 3: Update each repositories bypass list to include the app __mtnvencenzo-argocd-writeback-app__
Make sure to select 'Exempt from rules'

#### Step 4:  Ensure you exclude the directory that the argo cd version file gets written to from triggering a build or you'll end up in an infinite look of creating new images