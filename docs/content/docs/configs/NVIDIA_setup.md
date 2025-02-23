---
type: "docs"
weight: 1200
title: "NVIDIA setup"
author: "Andrea Spinelli"
icon: "browser_updated"
description: "NVIDIA drivers setup"
date: "2025-02-19T12:00:00+00:00"
BuildDate: "2025-02-19T12:00:00+00:00"
draft: false
toc: true
tags: ["Arch Linux", "NVIDIA", "GPU", "Graphics"]
categories: ["NVIDIA"]
---
### BIOS MUX Mode

BIOS MUX mode enables switching between integrated and dedicated graphics. This mode is particularly useful for optimizing performance in systems that use dual graphics configurations.

Mak shure to select your preferred configuration: Reboot your device and open BIOS (press `del` while booting); navigate in advanced options and search for a `Display Mode` option.

You shold see two options:

- **dGPU only**: Indicates the setup restricts to discrete GPUs exclusively.
- **dynamic**: Aspects of the setup (like resource allocation or switching) are determined at runtime rather than being static.

{{% alert context="tip" %}}
Using the **dynamic** configuration is recommended as it typically reduces power consumption while optimizing resource allocation based on your system's workload.

Keep in mind that some applications may not fully leverage these dynamic adjustments.
{{% /alert %}}

### NVIDIA Driver setup

```shell
paru -S nvidia nvidia-utils nvidia-settings 
```

or (recommended)

```shell
paru -S linux-headers nvidia-dkms nvidia-utils nvidia-settings
```

verify with `nvidia-smi`. You should see something like this:

```shell
+------------------------------------------------------------------------------------+
| NVIDIA-SMI 570.86.16        Driver Version: 570.86.16       CUDA Version: 12.8     |
+-------------------------------------+-----------------------+----------------------+
| GPU  Name             Persistence-M | Bus-Id         Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf       Pwr:Usage/Cap |          Memory-Usage | GPU-Util  Compute M. |
|                                     |                       |               MIG M. |
|=====================================+=======================+======================|
|   0  NVIDIA GeForce RTX 4060   Off  |  00000000:01:00.0 Off |                 N/A  |
| N/A   39C    P0         590W /  80W |       1MiB /  8188MiB |    10%       Default |
|                                     |                       |                  N/A |
+-------------------------------------+-----------------------+----------------------+

+------------------------------------------------------------------------------------+
| Processes:                                                                         | 
| GPU   GI   CI               PID   Type   Process name                   GPU Memory |
|       ID   ID                                                           Usage      |
|====================================================================================|
|  No running processes found                                                        |
+------------------------------------------------------------------------------------+
```

### Prime Setup

NVIDIA PRIME is a technology used mainly on Linux systems with hybrid graphics. It lets you switch between an integrated graphics card (usually from Intel or AMD) and a dedicated NVIDIA GPU. With PRIME, the system can offload resource-heavy tasks (like gaming or GPU-accelerated applications) to the NVIDIA card while using the integrated GPU for everyday tasks, which helps save power.

Let's create a prime-run command. It should allow you to launch an application using the dedicated NVIDIA GPU without needing to change system-wide settings.

```shell
sudo nano /usr/bin/prime-run
```

If you want to check if it works, if you have not set any desktop environment yet it involves different steps:

We need to install some xorg packages and `mesa-demos` to test the command and create a `.xinitrc` file:

```shell
paru -S xorg-server xorg-xinit xorg-xrandr xorg-xset glxgears mesa-demos xterm
echo "exec xterm" > ~/.xinitrc
```

then run the following command to start a xterm session

```shell
startx
```
Should appear a little terminal, there run the following command:

```shell
prime-run glxinfo | grep "OpenGL renderer"
```

You should see something like:

```shell
OpenGL renderer string: NVIDIA GeForce RTX 4060 Laptop GPU/PCIe/SSE2
```


### Kernel and GRUB Configuration

Make shure to enable DRM mode for NVIDIA in `/etc/default/grub`:

```shell
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet splash nvidia_drm.modeset=1"
```

This parameter enables DRM mode setting for the Nvidia driver. In effect, it lets the driver configure the display mode during boot. This not only *improves performance* by ensuring proper initialization of the graphical interface but also *enhances compatibility* with features like PRIME Render Offload and Wayland-based systems such as Hyperland, especially in multi-GPU setups.

Then update GRUB with:

```shell
sudo grub-mkconfig -o /boot/grub/grub.cfg
```