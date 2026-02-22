# Install Utilities (Kubectl, Helm)

## Kubectl

### Step 1: Update your local package index and install required dependencies:

``` shell
sudo apt update && sudo apt install -y ca-certificates curl apt-transport-https gpg
```

### Step 2: Download the public signing key for the Kubernetes package repositories and add it to the system's keyring:

``` shell
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
_Note: The version in the URL (v1.29 in this example) should be checked against the latest stable release on the Kubernetes documentation for the most up-to-date instructions._

### Step 3: Add the Kubernetes apt repository to your system's package sources

``` shell
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### Step 4: Update the apt package index again to include the new repository

``` shell
sudo apt update
```

### Step 5: Install kubectl

``` shell
sudo apt install -y kubectl
```

<br />

## Install Helm


### Step 1: Install prerequisites

``` bash
sudo apt-get install curl gpg apt-transport-https --yes
```

### Step 2: Add the GPG key for the Helm repository

``` bash
curl -fsSL https://packages.buildkite.com/helm-linux/helm-debian/gpgkey | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
```
_Note: The Helm apt repository recently moved to Buildkite; these instructions reflect the current, official source._

### Step 3: Add the Helm apt repository to your system's software sources

``` shell
echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://packages.buildkite.com/helm-linux/helm-debian/any/ any main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list > /dev/null
```

### Step 4: Install helm and verify

``` shell
sudo apt-get update
sudo apt-get install helm
helm version
```