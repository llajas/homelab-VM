# metal6 Virtual Machine Configuration

This repository contains the configuration files for the metal6 virtual machine, which is part of and heavily reliant on my [homelab](https://github.com/llajas/homelab) setup. The `metal6` VM is designed to function as a virtual Kubernetes node with GPU pass-through capabilities.

## Dependencies

The homelab repository provides the necessary scripts and configurations to:

- PXE boot the VM
- Install the operating system and required packages
- Join the VM to the Kubernetes cluster

More information on how the full installation process works can be found in the [docs](https://homelab.lajas.tech/reference/architecture/overview/)

Optionally, you'll want to install the qemu guest agent on the VM to allow for better communication between the host and the VM. This can be done by running the following command:

```bash
sudo dnf install qemu-guest-agent
sudo systemctl enable --now qemu-guest-agent
sudo systemctl status qemu-guest-agent
```

## Passed-through Hardware

1. Intel NIC
Allows the VM to PXE boot and dedicate network resources to the VM.

2. NVIDIA GPU
This is the only node in the cluster that is GPU equipped, allowing GPU-based workloads to run.

## Host Device Configuration

The host device must have the passed-through devices bound to VFIO. Additionally, the boot configuration must include the following options:

```bash
initcall_blacklist=sysfb_init
append vfio-pci.ids=<ID_HERE>
```
Where <ID_HERE> consists of the hardware IDs for your particular setup that you want to pass to the VM.

# Usage

## Starting the VM

To start the VM, run the following command:

```bash
virsh start metal6
```

## Required Drivers and Repositories

### Repositories

To install the NVIDIA drivers, you'll need to add the RPM Fusion repository:

```bash
sudo dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

### NVIDIA Drivers

Install the NVIDIA drivers and related packages, using the following command:

```bash
sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda nvidia-settings nvidia-persistenced
```

After installing the drivers, build and install the kernel modules:

```bash
sudo akmods --force
sudo dracut --force
```

## Checking the GPU

To check if the GPU is being recognized and used by the VM, use the `nvidia-smi` command:

```bash
nvidia-smi
```

Successful output should look like this:

```bash
[root@metal6 ~]# nvidia-smi
Mon May 27 10:00:38 2024
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.78                 Driver Version: 550.78         CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 2060        Off |   00000000:00:05.0 Off |                  N/A |
| 31%   38C    P8              9W /  160W |       1MiB /   6144MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
[root@metal6 ~]#
```

## Compatibility

This configuration has been tested on the following hardware:

- Intel Core i7-5820K @ 3.30GHz
- NVIDIA GeForce RTX 2060
- MSI X99A SLI Plus

This VM configuration was created using Unraid, but it should be compatible with any hypervisor that uses libvirt and virsh for VM management, including but not limited to:

- Proxmox VE
- Red Hat Virtualization (RHV)
- CentOS with KVM
- Fedora with KVM
- Ubuntu with KVM

Ensure that the necessary dependencies and configurations are in place for your specific hypervisor environment.
