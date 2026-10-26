# GPU Bios and Grub Settings 

## Hardware & Software
- **Gigabyte Radeon AI TOP Pro 32GB GPU** 
- **ASUS Pro Art 870E Creator WIFI Motherboard**
- **Ubuntu 26.04**

Based on our recent troubleshooting for your headless AI workload setup, we have focused heavily on stabilizing your Gigabyte Radeon AI TOP Pro 32GB GPU on the ASUS ProArt X870E-Creator WiFi motherboard running Ubuntu 26.04.

## Settings 
Here is the exact breakdown of the system and firmware configurations we put in place to stop the GPU from crashing, throwing bus errors, or dropping off the topology:


### ASUS ProArt BIOS Firmware Settings

- Above 4G Decoding: Enabled (Crucial for mapping the large 32GB VRAM allocation pool).
- Re-Size BAR Support: Enabled (Allows the CPU to negotiate full access to the GPU frame buffer simultaneously).
- Integrated Graphics: Forced explicitly to IGFX (Ensures your monitors run cleanly off the motherboard's onboard ports, leaving the discrete 32GB card completely headless for compute).
- PCIe ASPM Mode Control: Disabled (Turned off Native ASPM to prevent the PCIe slots from entering low-power sleep states, which was causing the driver to time out).
- IOMMU / ACS Controls: Enabled (ASUS firmware integrates Access Control Services directly into IOMMU; this isolates the PCIe groups cleanly for deep learning frameworks).


### Ubuntu GRUB Kernel Parameters

To change how the Linux kernel initializes and interacts with the AMD hardware, we updated your /etc/default/grub configuration line to reflect the following parameters:

``` bash
sudo nano /etc/default/grub

# Add or edit this line
# GRUB_CMDLINE_LINUX_DEFAULT="quiet splash amdgpu.aspm=0 amdgpu.gpu_recovery=1 amdgpu.runpm=0"

sudo update-grub

```

- **amdgpu.aspm=0:** Explicitly blocks the kernel graphics driver from trying to power-manage the PCIe bus link, resolving the Device lost from bus faults.
- **amdgpu.gpu_recovery=1:** Instructs the driver to attempt a soft reset mechanism if a ring buffer stalls, preventing a hard system lockup.
- **amdgpu.runpm=0:** Keeps the GPU powered on: It stops the Linux kernel from dynamically turning off or putting the AMD graphics card to sleep during runtime.  Normally, the Linux amdgpu driver automatically powers down discrete GPUs (dGPUs) or puts them into a low-power/suspended state when they are idle to save energy and reduce heat.


#### OS & Initramfs Deployment Commands

After altering the GRUB underlying parameters, to bake the updated amdgpu kernal module instructions into the initial boot RAM disk environment.

``` bash
sudo update-initramfs -u
```

## Troubleshooting

### Power usage

How to check power usage
``` bash
rocm-smi --showpower
```

### System Logs & Kernal Errors

Used to search the raw kernel ring buffer for initialization errors, missing hardware warnings, or crashes specifically related to the AMD driver.

``` bash
dmesg | grep -i amdgpu
```

### GPU Telemetry & Status

To verify if the discrete PCIe device was physically visible on the bus and confirm which kernel driver (amdgpu) was bound to it.

``` bash
lspci -nnk | grep -A 3 VGA
```
