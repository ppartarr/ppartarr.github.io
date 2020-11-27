---
layout: post
title:  "Mounting WD MyPassport in Linux"
date:   2020-11-14 23:09:010 +0200
categories: blog
---

I recently bought a WD MyPassword 2TB external hard drive to backup my laptop. When I first plugged it in and attempted to mount it, the drive was read-only:

```bash
# get the external drive device name
fdisk -l

[...]
Disk /dev/sda: 1.82 TiB, 2000365289472 bytes, 3906963456 sectors
Disk model: My Passport 2626
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: $REMOVED_DISK_IDENTIFIER

Device     Start        End    Sectors  Size Type
/dev/sda1   2048 3906961407 3906959360  1.8T Microsoft basic data

# mount the drive
mkdir /mnt/wd-passport
mount -t /dev/sda1 /mnt/wd-passport
touch /mnt/wd-passport/file
touch: cannot touch 'file': Read-only file system
```

Frustratingly, the [product page](https://shop.westerndigital.com/products/portable-drives/wd-my-passport-usb-3-0-hdd#WDBYVG0020BBK-WESN) says the drive is compatible with:
> Windows® 10 or Windows 8.1 operating systems. Requires reformatting for other operating systems.

After a quick search, [1] I found out I was simply missing support for the external hard drive's filesystem type: ntfs. I fixed this by installing the missing package:
```bash
yay -Sy ntfs-3g

# I can do a backup of my disk (nvme0n1)
dd if=/dev/nvme0n1 bs=4096 | gzip > /mnt/wd-passport/xps.img.gz
```

Hope this helps!

### Links

[1] https://bbs.archlinux.org/viewtopic.php?id=169310