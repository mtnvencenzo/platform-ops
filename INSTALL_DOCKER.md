# Install Docker CE on Ubuntu 
[<< back](INSTALL.md)

If not already installed, follow these steps to install Docker Engine: 

<br />

## Base Docker Installation

### Step 1: Update and install dependencies

``` bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release
```

### Step 2: Add Docker Official GPG Key & Repository

``` bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Step 3: Install Docker Engine

``` bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Step 4: Verify Installation

``` bash
sudo docker run hello-world
```

### Step 5: Configure Docker without Sudo (Optional but Recommended) 

``` bash
sudo usermod -aG docker $USER
# Log out and log back in for changes to take effect
```
<br/>

## Enable GPU Support (Experimental)

### Enable NVIDIA GPU Support on the Host

#### Step 1: Install NVIDIA Drivers on the Host

Make sure your host system has the latest NVIDIA drivers installed.  
You can check with:

```bash
nvidia-smi
```

If not installed, follow the official NVIDIA instructions for your OS.

#### Step 2: Install NVIDIA Container Toolkit

This allows Docker containers to access the GPU. Install the `nvidia-container-toolkit` package (replaces the deprecated `nvidia-docker2`).

```bash
# Add the package repositories
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
```

#### Step 3: Configure Docker Default Runtime

K3d creates its K3s node containers through Docker. For GPU access to propagate into the K3s nodes, Docker **must** use the nvidia runtime as its default runtime.

```bash
# Configure the nvidia runtime for Docker
sudo nvidia-ctk runtime configure --runtime=docker --set-as-default
sudo systemctl restart docker
```

Verify the configuration was applied:

```bash
cat /etc/docker/daemon.json
```

You should see output similar to:

```json
{
  "default-runtime": "nvidia",
  "runtimes": {
    "nvidia": {
      "path": "nvidia-container-runtime",
      "runtimeArgs": []
    }
  }
}
```

#### Step 4: Generate CDI Specs (Optional)

Some environments require CDI (Container Device Interface) specs to be generated for proper device discovery.

```bash
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
nvidia-ctk cdi list
```

#### Step 5: Verify GPU Access at the Docker Level

Before proceeding to K3d/K8s setup, confirm Docker can see the GPU:

```bash
docker run --rm --gpus all nvidia/cuda:12.0.1-base-ubuntu22.04 nvidia-smi
```

If this command does not show your GPU, stop here and troubleshoot before continuing.

<br/>
