# GPU BIOS and GRUB Settings

## Hardware & Software
- **Gigabyte Radeon AI TOP Pro 32GB GPU**
- **ASUS Pro Art 870E Creator WIFI Motherboard**
- **Ubuntu 26.04**

## Settings
The following firmware and operating-system settings are intended for a headless compute system using this GPU and motherboard.


### ASUS ProArt BIOS Firmware Settings

- Above 4G Decoding: Enabled (Crucial for mapping the large 32GB VRAM allocation pool).
- Re-Size BAR Support: Enabled (Allows the CPU to negotiate full access to the GPU frame buffer simultaneously).
- Integrated Graphics: Forced explicitly to IGFX (Ensures your monitors run cleanly off the motherboard's onboard ports, leaving the discrete 32GB card completely headless for compute).
- PCIe ASPM Mode Control: Disabled (Disables native PCIe ASPM to help prevent low-power link states that may cause driver timeouts).
- IOMMU / ACS Controls: Enabled (ASUS firmware integrates Access Control Services directly into IOMMU; this isolates the PCIe groups cleanly for deep learning frameworks).


### Ubuntu GRUB Kernel Parameters

To configure how the Linux kernel initializes and manages the AMD GPU, add these parameters to `GRUB_CMDLINE_LINUX_DEFAULT` in `/etc/default/grub`:

``` bash
sudo nano /etc/default/grub

# Add or edit this line
# GRUB_CMDLINE_LINUX_DEFAULT="quiet splash amdgpu.aspm=0 amdgpu.gpu_recovery=1 amdgpu.runpm=0"

```

- **`amdgpu.aspm=0`:** Disables PCIe link power management in the AMD GPU driver, which can help prevent bus-related device errors.
- **`amdgpu.gpu_recovery=1`:** Enables the driver's GPU recovery mechanism if a ring buffer stalls.
- **`amdgpu.runpm=0`:** Disables runtime power management for the GPU, preventing it from being powered down or suspended while idle. The default behavior may reduce power use and heat.


#### Apply GRUB and Initramfs Changes

After changing the GRUB parameters, update GRUB and regenerate the initramfs so the updated AMD GPU driver settings are available during boot.

``` bash
sudo update-grub
sudo update-initramfs -u
```

## Troubleshooting

### Power usage

Display GPU power usage:
``` bash
rocm-smi --showpower
```

### System Logs & Kernel Errors

Search kernel messages for AMD GPU initialization errors, missing-device warnings, or driver crashes:

``` bash
dmesg | grep -i amdgpu
```

### GPU Telemetry & Status

Check whether the GPU is visible on the PCIe bus and which kernel driver is bound to it:

``` bash
lspci -nnk | grep -A 3 VGA
```
