---
type: "docs"
weight: 400
title: "Installation"
author: "Andrea Spinelli"
icon: "downloading"
description: "After pre-installation steps, we can install Arch Linux now"
date: "2025-02-19T12:00:00+00:00"
BuildDate: "2025-02-19T12:00:00+00:00"
draft: false
toc: true
tags: ["Arch Linux", "Installation", "Guide"]
categories: [""]
---

### Installation

#### Boot into the Arch Linux Live Environment

Boot from the USB drive and confirm you are in UEFI mode by checking for the presence of `/sys/firmware/efi`.

#### Set Up the Keyboard Layout (if needed)
   
You can list all available keymaps using:

```shell
localectl list-keymaps
```
Then set your preferred keymap:

```shell
loadkeys us
```
*(replace `us` with your chosen layout)*

Additionally, you can change the console font with:

```shell
setfont <font_name>
```
fonts are located in `/usr/share/kbd/consolefonts/`.
   

#### Connect to the internet

make sure the network interface is unblocked using [rfkill](https://wiki.archlinux.org/title/Rfkill):

```shell
rfkill
```

Then connect to the network:

- **Ethernet**: plug in the cable
- **WiFi**: authenticate to the wireless network using [iwctl](https://wiki.archlinux.org/title/Iwctl)

The connection may be verified with [ping](https://wiki.archlinux.org/title/Ping)

```shell
ping archlinux.org
```

{{% alert context="info" %}}
**Note: This connection setup is temporary**

This is only a ***temporary connection setup***, needed for the installation process from live usb.

Once installed the base system and chroot it into it, we'll need to install and configure a network management tool such as `NetworkManager`.
{{% /alert %}}

#### Update the System Clock

Check the system clock status with
```
timedatectl
```

To **enable Network Time Protocol** (NTP) synchronization, type:

```shell
timedatectl set-ntp true
```

#### Partition the Disk

It's time to set up disk partitions. We can do this using tools like [fdisk](https://wiki.archlinux.org/title/Fdisk).

Firstly, identify your target disk (e.g., `/dev/nvme0n1`). 

```shell
lsblk
```

Then use `fdisk` (or any other partition manager) to create partitions (EFI, root, swap, etc.).

{{% alert context="tip" %}}
**Planning Your Partition Scheme**

Take time to plan a long-term partitioning scheme to avoid risky and complicated conversion or re-partitioning procedures in the future.
{{% /alert %}}

When partitioning your disk, pay attention to the **boot partition**. If dual booting with Windows, and if there is an existing EFI partition of sufficient size (~512 MB), it is recommended to reuse it rather than creating a new one

Regarding the **swap partition**, if you plan to use hibernation, consider allocating swap space that is at least equal to your system’s RAM. 
Alternatively, you can opt for **swapfiles**. This method offers greater flexibility in managing swap space without requiring a separate partition, though it may deliver slightly lower performance.

**Typical partition setup:**

{{< table >}}
| Mount Point | Partition                 | Partition Type                  | Suggested Size                                     |
|-------------|---------------------------|---------------------------------|----------------------------------------------------|
| `/boot`     | `/dev/efi_system_partition` | EFI system partition          | 1 GiB                                              |
| `[SWAP]`    | `/dev/swap_partition`       | Linux swap                    | At least 4 GiB                                     |
| `/`         | `/dev/root_partition`       | Linux x86-64 root (/)         | Remainder of the device. At least 32 GiB           |
{{< /table >}}
    
<br>

{{% alert context="mycase" %}}
**Personal Configuration**

In my personal configuration I decided to create a new boot partition as the one existing in windows was quite small (~256MB). Moreover I created a 24GB swap partition to enable the hibernation later.
    
```shell
$ lsblk
NAME          MAJ:MIN  RM    SIZE  RO TYPE MOUNTPOINTS
nvme0n1       259:0    0   931.5G  0  disk 
├─nvme0n1p1   259:1    0       1G  0  part /boot
├─nvme0n1p2   259:2    0      24G  0  part [swap]
└─nvme0n1p3   259:3    0   906.5G  0  part /
nvme1n1       ...
...
```
{{% /alert %}}

#### Format the Partitions

{{% alert context="danger" %}}
**Warning: Formatting EFI partition**

Only format the EFI system partition if you created it during the partitioning step.
{{% /alert %}}

Format the EFI partition by executing:
```shell
mkfs.fat -F32 /dev/efi_system_partition
```

Format your root partition with:
```shell
mkfs.ext4 /dev/root_partition
```
If you wish to enable swap, prepare the designated swap partition by running:
```shell
mkswap /dev/swap_partition
swapon /dev/swap_partition
``` 

#### Mount the Filesystems

Begin by mounting the root partition to `/mnt` using:
```shell
mount /dev/root_partition /mnt
```
Then, create the EFI directory and mount the EFI partition to it with the following commands:
```shell
mount --mkdir /dev/efi_system_partition /mnt/boot
```
*Another typical directory locations is `/mnt/boot/efi`*

#### Install the Base System

Before installing the system it's recommended to update mirrors using tools as [reflector](https://wiki.archlinux.org/title/Reflector). For example, the following command finds and updates the top 10 best-rated mirrors based on speed and reliability:

```shell
sudo reflector --latest 10 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

Now you can install the base system, the Linux kernel, and the necessary firmware.

```shell
pacstrap -K /mnt base linux linux-firmware
```

{{% alert context="tip" %}}
You may add other essential packages such as network tools or your preferred text editor, based on your needs.
{{% /alert %}}

   A more complete command is the following one:
   ```shell
   pacstrap -K /mnt base linux linux-firmware nano networkmanager man-db man-pages texinfo
   ```
   Consider also adding cpu [microcode](https://wiki.archlinux.org/title/Microcode) updates using `amd-ucode` or `intel-ucode`

{{% alert context="tip" %}}
For managing AUR packages, consider installing an AUR helper like `yay` or `paru`. I personally prefer `paru` because of its useful features.

If you'd like to install `paru` use the following commands:
```shell
sudo pacman -S --needed base-devel git # if not already installed
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
```
{{% /alert %}}