# Archlinux Installation Roadmap

This document outlines the steps I followed to install and configure Arch Linux with Hyprland on my Asus A15 FA507NV. It is intended both as a personal reference and as a guide for anyone wishing to reproduce the setup.

### Computer Specs

| Component     | Specification                            |
|---------------|------------------------------------------|
| Processor     | Ryzen 7 7735HS with iGPU                 |
| Graphics Card | RTX 4060 8GB DDR6                        |
| Memory        | 16GB DDR5 4800MHz                        |
| Storage (1)   | native SSD NVMe 500GB (for Windows)      |
| Storage (2)   | SSD NVMe Samsung 990 Pro 1TB (for Linux) |

---

## Index

  - [Installation Roadmap](#installation-roadmap)
    - [Pre-Installation](#pre-installation)
    - [Installation](#installation)
    - [System Configuration](#system-configuration)
    - [Reboot and Post Installation](#reboot-and-post-installation)

---

## Installation Roadmap

> [!WARNING]
Always consult the [Arch Wiki Installation Guide](https://wiki.archlinux.org/title/Installation_guide) for the most current procedures and troubleshooting tips as the installation process may evolve over time. 

### Pre-Installation

1. #### Preparation & Research
   - Review the [Arch Linux Installation Guide](https://wiki.archlinux.org/title/Installation_guide) for a detailed walkthrough.
   - Backup any important data (recommended if working on a non empty storage).
   - Download the latest Arch Linux ISO from the official [Arch Linux website](https://archlinux.org/download/#download-mirrors).

2. #### Create Bootable Media

   Use tools such as [Balena Etcher](https://etcher.balena.io/), [Ventoy](https://www.ventoy.net/en/index.html), or, if you prefer the command line, the `dd` utility to write the ISO to a USB drive.
  
   > [!NOTE] Verify Download Integrity (Optional)
   > Download checksum file from [this page](https://archlinux.org/download/#checksums) and use [gpg](https://wiki.archlinux.org/title/GnuPG#Verify_a_signature) to verify the signature.
   > ```bash
   > gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig
   > ```
   >Alternatively, **from an existing Arch Linux installation** run:
   > ```bash
   > pacman-key -v archlinux-version-x86_64.iso.sig
   > ```

3. #### BIOS/UEFI Configuration
   
   Enter the BIOS/UEFI settings:
   - Enable **UEFI mode** (it could also be the only one available)
   - **Disable Secure Boot** if necessary.
   - Set USB as the primary boot device.

### Installation

1. #### **Boot into the Arch Linux Live Environment
   - Boot from the USB drive and confirm you are in UEFI mode by checking for the presence of `/sys/firmware/efi`.

2. #### Set Up the Keyboard Layout (if needed)
   
   You can list all available keymaps using:
   ```
   localectl list-keymaps
   ```
   Then set your preferred keymap:
   ```bash
   loadkeys us
   ```
   *(replace `us` with your chosen layout)*

   Additionally, you can change the console font with:
   ```bash
   setfont <font_name>
   ```
   fonts are located in `/usr/share/kbd/consolefonts/`.
   

3. #### Connect to the internet

   make sure the network interface is unblocked using [rfkill](https://wiki.archlinux.org/title/Rfkill):
   ```bash
   rfkill
   ```

   Then connect to the network:
   
   - **Ethernet**: plug in the cable
   - **WiFi**: authenticate to the wireless network using [iwctl](https://wiki.archlinux.org/title/Iwctl)

   The connection may be verified with [ping](https://wiki.archlinux.org/title/Ping)

   ```bash
   ping archlinux.org
   ```

   > [!NOTE] Note: This connection setup is temporary
   > This is only a ***temporary connection setup***, needed for the installation process from live usb.
   > 
   > Once installed the base system and chroot it into it, we'll need to install and configure a network management tool such as `NetworkManager`

4. #### Update the System Clock

   Check the system clock status with
   ```
   timedatectl
   ```

   To **enable Network Time Protocol** (NTP) synchronization, type:

   ```bash
   timedatectl set-ntp true
   ```

5. #### **Partition the Disk**

   It's time to set up disk partitions. We can do this using tools like [fdisk](https://wiki.archlinux.org/title/Fdisk).

   Firstly, identify your target disk (e.g., `/dev/nvme0n1`). 
   
   ```bash
   lsblk
   ```
   
   Then use `fdisk` (or any other partition manager) to create partitions (EFI, root, swap, etc.).

   > [!TIP] Tip: Planning Your Partition Scheme
   > Take time to plan a long-term partitioning scheme to avoid risky and complicated conversion or re-partitioning procedures in the future.

   When partitioning your disk, pay attention to the **boot partition**. If dual booting with Windows, and if there is an existing EFI partition of sufficient size (~512 MB), it is recommended to reuse it rather than creating a new one

   Regarding the **swap partition**, if you plan to use hibernation, consider allocating swap space that is at least equal to your system’s RAM. 
   Alternatively, you can opt for **swapfiles**. This method offers greater flexibility in managing swap space without requiring a separate partition, though it may deliver slightly lower performance.

   **Typical partition setup:**
   | Mount Point | Partition                 | Partition Type                  | Suggested Size                                     |
   |-------------|---------------------------|---------------------------------|----------------------------------------------------|
   | `/boot`       | `/dev/efi_system_partition` | EFI system partition            | 1 GiB                                              |
   | `[SWAP]`      | `/dev/swap_partition`       | Linux swap                      | At least 4 GiB                                     |
   | `/`           | `/dev/root_partition`       | Linux x86-64 root (/)           | Remainder of the device. At least 32 GiB           |

   > In my personal configuration I decided to create a new boot partition as the one existing in windows was quite small (256MB). Moreover I created a 24GB swap partition to enable the hibernation later.
   >
   > ```bash
   >$ lsblk
   >NAME          MAJ:MIN  RM    SIZE  RO TYPE MOUNTPOINTS
   >nvme0n1       259:0    0   931.5G  0  disk 
   >├─nvme0n1p1   259:1    0       1G  0  part /boot
   >├─nvme0n1p2   259:2    0      24G  0  part [swap]
   >└─nvme0n1p3   259:3    0   906.5G  0  part /
   >nvme1n1       ...
   >...
   >```

6. #### Format the Partitions

   > [!WARNING] Warning: Formatting EFI partition
   > Only format the EFI system partition if you created it during the partitioning step.
   >
   > Format the EFI partition by executing:
   > ```bash
   > mkfs.fat -F32 /dev/efi_system_partition
   > ```

   Format your root partition with:
   ```bash
   mkfs.ext4 /dev/root_partition
   ```
   If you wish to enable swap, prepare the designated swap partition by running:
   ```bash
   mkswap /dev/swap_partition
   swapon /dev/swap_partition
   ``` 

7. #### Mount the Filesystems

   Begin by mounting the root partition to `/mnt` using:
   ```bash
   mount /dev/root_partition /mnt
   ```
   Then, create the EFI directory and mount the EFI partition to it with the following commands:
   ```bash
   mount --mkdir /dev/efi_system_partition /mnt/boot
   ```
   *Another typical directory locations is `/mnt/boot/efi`*

8. #### Install the Base System

   Before installing the system it's recommended to update mirrors using tools as [reflector](https://wiki.archlinux.org/title/Reflector). For example, the following command finds and updates the top 10 best-rated mirrors based on speed and reliability:

   ```bash
   sudo reflector --latest 10 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
   ```

   Now you can now install the base system, the Linux kernel, and the necessary firmware.

   ```bash
   pacstrap -K /mnt base linux linux-firmware
   ```

   > [!TIP] 
   > You may add other essential packages such as network tools or your preferred text editor, based on your needs.
   > 
   > A more complete command is the following one:
   > ```bash
   > pacstrap -K /mnt base linux linux-firmware nano networkmanager man-db man-pages texinfo
   > ```
   > Consider also adding cpu [microcode](https://wiki.archlinux.org/title/Microcode) updates using `amd-ucode` or `intel-ucode`

### System Configuration

1. #### Generate the Filesystem Table (fstab)
   After installing the base system, generate the filesystem table by running:
   ```bash
   genfstab -U /mnt >> /mnt/etc/fstab
   ```
   It's important to review the generated `/mnt/etc/fstab` file to ensure that all partitions are correctly listed and configured.

2. #### Chroot into the New System

   Finally, change root into your new system environment to continue with further configuration:
   ```bash
   arch-chroot /mnt
   ```


3. #### Time Zone and Localization
   
   If you need to **set your timezone manually** you can do so by creating a symlink:

   ```bash
   ln -sf /usr/share/zoneinfo/Your/Timezone /etc/localtime 
   hwclock --systohc
   ```


   Edit then `/etc/locale.gen` (e.g. unisng `nano`) and uncomment your locale settings (e.g. `en_US.UTF-8 UTF-8`).

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

4. **Network Configuration**
   
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

5. #### Root Password and User Creation

   Set now the **root password**
   ```bash
   passwd
   ```
   Create a **regular user**:

   - First, install `sudo` (if not already installed)
      ```bash
      pacman -S sudo
      ```
   - Then create the used with:
      ```bash
      useradd -m -G wheel -s /bin/bash <your_username>
      ```
   - Set password
      ```bash
      passwd <your_username>
      ```
   - Edit `/etc/sudoers` (using `visudo`) and enable wheel group privileges. Find the commented line `# %wheel ALL=(ALL) ALL` and remove `#` at the begionning of the line:
      ```bash
      EDITOR=nano visudo
      ```

6. #### Bootloader Installation

   Install a bootloader like `systemd-boot` or `GRUB`:
   - For **systemd-boot**:
      ```bash
      bootctl install
      ```
      Create an entry in `/boot/loader/entries/arch.conf` with necessary options.
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

### Reboot and Post Installation

1. **Exit Chroot and Unmount Filesystems**
   ```bash
   exit           # exits chroot
   umount -R /mnt # not mandatory but 
   ```

2. **Reboot the System**
   
   Run: `reboot`and remove the installation media when prompted.
   
   Then, Log in as your regular user.

3. **Post Installation Tasks**

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
   
      >[!TIP]
      > For managing AUR packages, consider installing an AUR helper like `yay` or `paru`. I personally prefer `paru` because of its useful features. 
      >
      >If you'd like to install `paru` use the following commands:
      > ```bash
      > sudo pacman -S --needed base-devel git # if not already installed
      > git clone https://aur.archlinux.org/paru.git
      > cd paru
      > makepkg -si
      > ```

### Set up GPU drivers

 **Documentation and Backup**
   - Save this markdown file in your repository.
   - Consider versioning your configuration files (using Git, for example) for future reference and reproducibility.


### Additional Software and Hyprland Setup
   - Install Xorg or Wayland dependencies as required by Hyprland.
   - Follow the [Hyprland installation guide](https://wiki.archlinux.org/title/Hyprland) for window manager-specific configurations.
   - Install and configure any additional applications (e.g., terminal emulator, compositor settings, etc.).

### TODO

[] configure kitty
[] 
[] configure htop