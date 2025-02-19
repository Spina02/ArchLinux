---
type: "docs"
weight: 500
title: "Post-Installation"
author: "Andrea Spinelli"
icon: "restart_alt"
description: "Configure your Arch Linux system after installation: enable services, update mirrors, upgrade packages, and install essential tools."
date: "2025-02-19T12:00:00+00:00"
BuildDate: "2025-02-19T12:00:00+00:00"
draft: false
toc: true
tags: ["Arch Linux", "Installation", "Guide"]
categories: [""]
---

### Reboot and Post Installation

 #### Exit Chroot and Unmount Filesystems
   ```bash
   exit           # exits chroot
   umount -R /mnt # not mandatory but 
   ```

##### Reboot the System
   
   Run: `reboot` and remove the installation media when prompted.
   
   Then, Log in as your regular user.

##### Post Installation Tasks

   After rebooting and logging in, it's time to fine-tune your system configurations by enabling and starting essential services.
   
   First, ensure that the network connection remains stable by enabling the network management tool you configured earlier.
   
   For example, if you opted for NetworkManager, run:

   ```bash
   systemctl enable NetworkManager
   ```

   Once your connection is confirmed, update your package mirrors and upgrade the system by executing:

   ```bash
   sudo pacman -Syyu
   ```

   Next, install some useful packages that you may need for further customization, such as Git, development tools, Python:

   ```bash
   sudo pacman -S git base-devel python bat htop neofetch bluez
   ```