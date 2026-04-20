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

## Enable GPU Support

Choose the section that matches your GPU hardware.

<br/>

### Radeon GPU (ROCm)

#### Step 1: Verify the amdgpu Kernel Driver is Loaded

The `amdgpu` driver is built into modern Linux kernels (6.x+). Verify it is loaded and your GPU devices are present:

```bash
lsmod | grep amdgpu
ls /dev/kfd
ls /dev/dri/renderD*
```

You should see `/dev/kfd` (Kernel Fusion Driver for compute) and at least one `/dev/dri/renderD128` device. If not, check that your kernel supports your GPU and that the `amdgpu` module is loaded.

#### Step 2: Install ROCm on the Host

Install the ROCm userspace packages. These provide `rocm-smi`, HIP runtime, and related tools:

```bash
# Add the AMD ROCm repository
sudo mkdir -p --mode=0755 /etc/apt/keyrings
wget https://repo.radeon.com/rocm/rocm.gpg.key -O - | \
  gpg --dearmor | sudo tee /etc/apt/keyrings/rocm.gpg > /dev/null

# Add the AMDGPU and ROCm repositories (adjust for your Ubuntu version)
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/amdgpu/latest/ubuntu $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/amdgpu.list
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/latest $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/rocm.list

# Pin ROCm packages to prefer the AMD repository
echo -e 'Package: *\nPin: release o=repo.radeon.com\nPin-Priority: 600' | \
  sudo tee /etc/apt/preferences.d/rocm-pin-600

sudo apt-get update
sudo apt-get install -y rocm-smi-lib rocm-hip-runtime
```

ROCm binaries install to `/opt/rocm/bin` which is not on the default PATH. Add it so that `rocm-smi`, `rocminfo`, and other tools are available:

```bash
echo 'export PATH=$PATH:/opt/rocm/bin' >> ~/.bashrc
source ~/.bashrc
```

_Note: Check the [ROCm installation docs](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/) for the latest instructions and supported Ubuntu versions._

#### Step 3: Add Your User to the video and render Groups

GPU device access requires membership in the `video` and `render` groups:

```bash
sudo usermod -aG video,render $USER
# Log out and log back in for changes to take effect
```

#### Step 4: Verify ROCm on the Host

```bash
rocm-smi
rocminfo | head -10000
```

You should see your Radeon GPU listed with its device name under the HSA Agents section (the first ~40 lines show only the CPU agent — the GPU agents appear after). If `rocm-smi` shows no devices, troubleshoot before continuing.

> **RDNA 4 Note:** ROCm support for RDNA 4 (Radeon RX 9000 series) is relatively new. If `rocm-smi` does not detect your card, ensure you are running the latest ROCm version and a kernel ≥ 6.12. Check the [ROCm compatibility matrix](https://rocm.docs.amd.com/en/latest/compatibility/compatibility-matrix.html) for your specific GPU.

#### Step 5: Identify Your GPU and Configure Selection

If your system has multiple AMD GPUs (e.g. a discrete Radeon plus an integrated Ryzen iGPU), you need to identify which device index corresponds to your discrete GPU so workloads target the correct one.

List all detected GPUs and note the device index, DID, power draw, and clock speeds:

```bash
rocm-smi
```

The discrete GPU will typically show higher power draw (e.g. 10W+ idle) and a meaningful power cap (e.g. 300W), while an integrated GPU shows near-zero power (e.g. 0.01W) and no power cap. Note the **Device** index (usually `0`) of your discrete GPU.

Next, identify the GPU architecture:

```bash
rocminfo | grep -i gfx
```

This shows the `gfx` target for each GPU (e.g. `gfx1201` for RDNA 4, `gfx1100` for RDNA 3). Note the architecture of your discrete GPU — you may need it for troubleshooting.

**GPU Selection with `ROCR_VISIBLE_DEVICES`:** To restrict ROCm workloads to specific GPUs, set the `ROCR_VISIBLE_DEVICES` environment variable to a comma-separated list of device indices. Exclude any GPUs you don't want used for compute (e.g. an integrated GPU).

```bash
# Single discrete GPU (Device 0 only)
ROCR_VISIBLE_DEVICES=0 rocm-smi

# Multiple discrete GPUs (e.g. Devices 0 and 1, excluding iGPU at Device 2)
ROCR_VISIBLE_DEVICES=0,1 rocm-smi
```

You should see only the targeted GPUs listed. This variable is used in Docker Compose and Kubernetes configurations to ensure workloads target the correct GPU(s). Adjust the indices based on your `rocm-smi` output.

> **`HSA_OVERRIDE_GFX_VERSION`:** Some ROCm applications (PyTorch, Ollama, etc.) may not include pre-built binaries for newer GPU architectures. If workloads fail with "no GPU agent" errors despite `rocm-smi` detecting the GPU, set `HSA_OVERRIDE_GFX_VERSION` to the major.minor.0 of your architecture (e.g. `12.0.0` for `gfx1201`, `11.0.0` for `gfx1100`). This tells the HIP runtime to use compatible code for your GPU.

#### Step 6: Verify GPU Access at the Docker Level

No special Docker runtime is required for AMD GPUs — devices are passed through directly. Verify Docker can access the GPU with the correct device targeted:

```bash
# Single discrete GPU
docker run --rm --device /dev/kfd --device /dev/dri \
  -e ROCR_VISIBLE_DEVICES=0 \
  rocm/pytorch:latest rocm-smi

# Multiple discrete GPUs (adjust indices to match your hardware)
# docker run --rm --device /dev/kfd --device /dev/dri \
#   -e ROCR_VISIBLE_DEVICES=0,1 \
#   rocm/pytorch:latest rocm-smi
```

You should see only the targeted discrete GPU(s) listed — not the iGPU. If this command does not show your GPU, stop here and troubleshoot before continuing.

> **Image Compatibility Note:** The `rocm/pytorch:latest` image may not include pre-built kernels for newer GPU architectures (e.g. RDNA 4 `gfx1201`). If `rocm-smi` works but PyTorch operations fail, add `-e HSA_OVERRIDE_GFX_VERSION=12.0.0` (adjust the version for your architecture) to the Docker run command.

<br/>

### NVIDIA GPU

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
