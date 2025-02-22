---
type: "docs"
weight: 1100
title: "GRUB Configuration"
author: "Andrea Spinelli"
icon: "login"
description: "Configuring GRUB bootloader on Arch Linux"
date: "2025-02-19T12:00:00+00:00"
BuildDate: "2025-02-19T12:00:00+00:00"
draft: false
toc: true
tags: ["Arch Linux", "GRUB", "Config", "Configuration"]
categories: ["Bootloader"]
---

{{% alert context="danger" %}}

This page is still under development. Check back later for more details!

{{% /alert %}}

### GRUB Installation

Firstly, let's check if we have already installed GRUB:

```shell
ls /boot/EFI/GRUB
# or ls /boot/efi/EFI/GRUB
```

you should see files like `grubx64.efi` or similar. This means GRUB have been installed successfully.

This means we should also have the `grub.cfg` config file; you can check using:

```shell
cat /boot/grub/grub.cfg
```

#### Add Windows Entry (only if you created a new EFI partition)

If you created a new EFI partition in [pre-Installation]({{% ref "docs/installation/pre-installation.md" %}}) phase, you will probably need to add windows manually. Before doing this we can try to make GRUB detect other operating systems if the tool os-prober is installed and enabled:

```shell
paru -S os-prober # or use `sudo pacman`
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

Then check if grub managed to add it:

```shell
cat /boot/grub/grub.cfg | grep windows
```

If this does not work, then we have to add Windows manually.

1. **Identify the Windows EFI Partition**

    Firstly we have to identify which one is the Windows EFI Partition and note its UUID. To do that you can simply use:

    ```shell
    sudo blkid | grep -i efi
    ```

    you should see something like `/dev/nvme1n1p1` with partition label '***EFI system partition***'. If not, 

    Note its UUID (a string like `XXXX-XXXX`).

2. **Edit the GRUB Custom file**

    Let's open now the GRUB Custom file with root privileges:

    ```shell
    sudo nano /etc/grub.d/40_custom
    ```

    At the end of the file, add a new menu entry for Windows (make sure to replace `XXXX-XXXX` with the right UID):

    ```shell
    menuentry "Windows 10" {
        insmod part_gpt
        insmod fat
        search --fs-uuid --no-floppy --set=root XXXX-XXXX
        chainloader /EFI/Microsoft/Boot/bootmgfw.efi
    }
    ```

3. **Reorder the Entries (Optional)**
    
    The order of the GRUB entries depends on the name of the files in `/etc/grub.d/` folder. 
    
    Let's check the current situation:

    ```shell
    $ ls /etc/grub.d/
    # You should see something like this:
    00_header 10_linux 20_linux_xen 25_bli 30_0s_prober 30_wefi firmware 40_custom 41_custom README
    ```

    To reorder the entries is sufficient to rename those files. Let's rename now the file we have modified in the previous step:

    ```shell
    rename /etc/grub.d/40_custom /etc/grub.d/25_windows /etc/grub.d/40_custom
    ```

4. **Timeout and Default boot option**

    We can define a timeout before GRUB automatically boots from the default option. To do that we shell modify the `/etc/default/grub` file:

    ```shell
    sudo nano /etc/default/grub 
    ```

    and modify the following variables:

    ```shell
    GRUB_DEFAULT=saved      # default entry is the last selection
    GRUB_TIMEOUT=5          # 5 second timeout
    # ...
    GRUB_SAVEDEFAULT=true   # needed to make GRUB remember the last option
    ```

5. **Regenerate the GRUB Configuration file and Reboot**

    Now, let's generate the GRUB configuration file and then reboot to see if everything works as expected:

    ```
    sudo grub-mkconfig -o /boot/grub/grub.cfg
    reboot
    ```

{{% alert context = "danger" %}}
**Be careful** when working with grub: If you mispell something you could incurr to boot issues
{{% /alert %}}

#### Customize GRUB - [WIP]

To customize GRUB we can create a theme from zero or use an existing one

{{% alert context="success" %}}
There are many repositories where you can find a lot of themes of all types. Here I found a well-structured repo:
[GitHub - Gorgeous-GRUB](https://github.com/jacksaur/Gorgeous-GRUB)
{{% /alert %}}