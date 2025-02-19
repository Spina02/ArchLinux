---
type: "docs"
weight: 200
title: "Pre-Installation"
icon: "download"
description: "Essential pre-installation steps: verify system, create bootable media, and configure BIOS/UEFI for a smooth installation."
date: "2025-02-19T12:00:00+00:00"
BuildDate: "2025-02-19T12:00:00+00:00"
draft: false
toc: true
tags: ["Arch Linux", "Installation", "Guide"]
categories: [""]
---

### Pre-Installation

#### Preparation & Research

- Review the [Arch Linux Installation Guide](https://wiki.archlinux.org/title/Installation_guide) for a detailed walkthrough.
- Backup any important data (recommended if working on a non empty storage).
- Download the latest Arch Linux ISO from the official [Arch Linux website](https://archlinux.org/download/#download-mirrors).

#### Create Bootable Media

Use tools such as [Balena Etcher](https://etcher.balena.io/), [Ventoy](https://www.ventoy.net/en/index.html), or, if you prefer the command line, the `dd` utility to write the ISO to a USB drive.
    
{{% alert context="info" %}}
**Verify Download Integrity (Optional)**  

Download checksum file from [this page](https://archlinux.org/download/#checksums) and use [gpg](https://wiki.archlinux.org/title/GnuPG#Verify_a_signature) to verify the signature.

```bash
gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig
```

Alternatively, **from an existing Arch Linux installation** run:

```bash
pacman-key -v archlinux-version-x86_64.iso.sig
```
{{% /alert %}}

#### BIOS/UEFI Configuration
   
Enter the BIOS/UEFI settings:
- Enable **UEFI mode** (it could also be the only one available)
- **Disable Secure Boot** if necessary.
- Set USB as the primary boot device.