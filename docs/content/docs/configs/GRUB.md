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

If you created a new EFI partition in [pre-Installation]({{< ref "docs/installation/pre-installation.md" >}}) phase, you will probably need to add windows manually. Before doing this we can try to make GRUB detect other operating systems if the tool os-prober is installed and enabled:

```shell
paru -S os-prober # or use `sudo pacman`
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

Then check if grub managed to add it:

```shell
cat /boot/grub/grub.cfg | grep windows
```

If this does not work, then we have to add Windows manually.

1. Identify the Windows EFI Partition: 
    ```shell
    sudo blkid | grep -Ei "vfat|efi"
    ```
    you should see something like `/dev/nvme1n1p1` with partition label '***EFI system partition***'.

    Note its UUID

2. 