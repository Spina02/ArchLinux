---
type: "docs"
weight: 600
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
   ```shell
   exit           # exits chroot
   umount -R /mnt # not mandatory but 
   ```

##### Reboot the System
   
   Run: `reboot` and remove the installation media when prompted.
   
   Then, Log in as your regular user.

##### Post Installation Tasks

   After rebooting and logging in, it's time to fine-tune your system configurations by enabling and starting essential services.

   First, ensure that the network connection remains stable by enabling the NetworkManager service you configured earlier:

   ```shell
   systemctl enable NetworkManager
   ```

   Once your connection is confirmed, update your package mirrors and upgrade the system by executing:

   ```shell
   sudo pacman -Syyu
   ```

   Next, install some useful packages for further customization:

   ```shell
   sudo pacman -S git base-devel python bat htop neofetch bluez
   ```

   - `git`: A distributed version control system to track changes in your projects.  
   - `base-devel`: A group of essential build tools (like gcc, make, etc.) for compiling software.  
   - `python`: A versatile high-level programming language for scripting and development.  
   - `bat`: A modern replacement for cat with syntax highlighting and Git integration.  
   - `htop`: An interactive process viewer to monitor system performance in real time.  
   - `neofetch`: A tool that displays system information in a visually appealing way.  
   - `bluez`: The official Bluetooth protocol stack for Linux, enabling Bluetooth functionality.
