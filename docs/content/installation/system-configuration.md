---
type: "docs"
weight: 500
title: "System Configuration"
icon: "settings"
description: "Step-by-step guide to system configuration during Arch Linux installation."
date: "2025-02-19T12:00:00+00:00"
BuildDate: "2025-02-19T12:00:00+00:00"
draft: false
toc: true
tags: ["Arch Linux", "Installation", "Guide"]
categories: [""]
---

### System Configuration

#### Generate the Filesystem Table (fstab)
   After installing the base system, generate the filesystem table by running:
   ```bash
   genfstab -U /mnt >> /mnt/etc/fstab
   ```
   It's important to review the generated `/mnt/etc/fstab` file to ensure that all partitions are correctly listed and configured.

#### Chroot into the New System

   Finally, change root into your new system environment to continue with further configuration:
   ```bash
   arch-chroot /mnt
   ```


#### Time Zone and Localization
   
   If you need to **set your timezone manually** you can do so by creating a symlink:

   ```bash
   ln -sf /usr/share/zoneinfo/Your/Timezone /etc/localtime 
   hwclock --systohc
   ```

   Edit then `/etc/locale.gen` (e.g. using `nano`) and uncomment your locale settings (e.g. `en_US.UTF-8 UTF-8`).

   Generate the locales by running: 
   ```bash
   locale-gen
   ```
   Now create `/etc/locale.conf` and set the `LANG` variable.
   You can do this in a single command as the following:
     
   ```bash
   echo LANG=en_US.UTF-8 > /etc/locale.conf
   ```
   *(Replace `en_US.UTF-8` with your preferred locale settings)*

   If you set a keyboard layout, make the changes persistent in `vconsole.conf`. The following example sets the `de-latin1` keymap:

   ```bash
   echo KEYMAP=de-latin1 > /etc/vconsole.conf
   ```

#### Network Configuration
   
   Create an hostname:
   ```bash
   echo <your_hostname> > /etc/hostname
   ```

   Edit the file `/etc/hosts` appending the following lines:
     ```
     127.0.0.1   localhost
     ::1         localhost
     127.0.1.1   <your_hostname>.localdomain <your_hostname>
     ```
   Install and configure a network manager such as [NetworkManager](https://wiki.archlinux.org/title/NetworkManager):
   ```bash
   pacman -S networkmanager
   ```
   Enable with:
   ```bash
   systemctl enable NetworkManager
   ```
   And connect with:
   ```bash
   nmcli device wifi connect <your_SSID_or_BSSID> password <your_password>
   ```

#### Root Password and User Creation

   Set now the **root password**
   ```bash
   passwd
   ```
   Create a **regular user**:

   - First, install `sudo` (if not already installed)
      ```bash
      pacman -S sudo
      ```
   - Then create the user with:
      ```bash
      useradd -m -G wheel -s /bin/bash <your_username>
      ```
   - Set password
      ```bash
      passwd <your_username>
      ```
   - Edit `/etc/sudoers` (using `visudo`) and enable wheel group privileges. Find the commented line `# %wheel ALL=(ALL) ALL` and remove `#` at the beginning of the line:
      ```bash
      EDITOR=nano visudo
      ```

#### Bootloader Installation

   Install a bootloader like `systemd-boot` or `GRUB`:
   - For **systemd-boot**:
      ```bash
      bootctl install
      ```
      Create an entry in `/boot/loader/entries/arch.conf` with the necessary options.
   - For **GRUB**:
      ```bash
      pacman -S grub efibootmgr
      ```
      Install:
      ```bash
      grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
      ```
      Generate config:
      ```bash
      grub-mkconfig -o /boot/grub/grub.cfg
      ```