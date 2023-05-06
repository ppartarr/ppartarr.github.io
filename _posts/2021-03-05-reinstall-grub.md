---
layout: post
title:  "Reinstall grub after a Windows updates"
date:   2021-03-05 12:00:010 +0200
categories: blog
---

When running a dual boot Arch + Windows setup, Windows updates can sometimes mess with your boot partition. Rather than booting directly into grub, Windows will override your /boot partition and you will be left without a grub.cfg. Due to this, you will boot directly into windows. Luckily there is a simple fix for this.

<!--more-->

You will need a USB key with the Arch .iso on hand. If you don't have one simply follow the steps from **Download the Arch image** & **Create a bootable USB with Arch** in [my previous guide](https://www.partarrieu.me/blog/2019/09/24/arch-dual-boot.html). Once you have it simply follow these steps:

1. Plug-in the USB key with the Arch .iso
1. Boot into BIOS settings and switch from RAID to AHCI
2. Boot into Arch
3. Reinstall grub by running the commands below (partitions may vary)

```bash
#!/usr/bin/env bash

# mount the boot partition
mount /dev/nvme0n1p9 /mnt
mount /dev/nvme0n1p7 /mnt/boot

# change root to /mnt partition
arch-chroot /mnt

# reinstall grub
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub
grub-mkconfig -o /boot/grub/grub.cfg
os-prober

# exit partition, unmount and reboot
exit
umount -R /mnt
reboot
```

Voilà!
