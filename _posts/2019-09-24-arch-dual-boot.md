---
layout: post
title:  "Dual boooting arch linux & windows 10"
date:   2019-09-24 23:09:010 +0200
categories: blog
---

There are already dozens of tutorials to setup an arch and Windows dual boot - welcome to a dozen + 1. Like most others this is a step by step guide. Unlike most others the steps are ordered in a way that makes sense (starting with the download first - duh!) so you won't have to restart your computer a gazillion times.

I did this on a single SSD Dell XPS 15 with windows 10 preinstalled. It's obviously possible to follow this guide if you're installing arch onto a different drive or if you're running older hardware. If you run into any problems please be sure to read through the [Arch Installation Guide](https://wiki.archlinux.org/index.php/Installation_guide) and the [Arch Dual boot with Windows wiki](https://wiki.archlinux.org/index.php/Dual_boot_with_Windows). If those didn't help, try out [headspace](https://www.headspace.com/) before attempting to copy/paste commands from forums that looked like they were designed by a developer who thinks HTML is a pRoGrAmMiNg LaNgUaGe.

To get started you will need:
  * a stable internet connection
  * a USB key with 1GB storage
  * a cup of coffee ☕ & and a can do attitude

## Warmup

### Download the arch image
Let's begin by [downloading the .iso](https://www.archlinux.org/download/) preferably using BitTorrent and verifying the checksum [1].

### Check the partition table type in Windows 

If you recently purchased a laptop it will most likely be using UEFI. In case you're performing this dual-boot installation on an old laptop you should check this by:
  1. starting the run dialog with `Win key + R`
  2. type `msinfo32` and press enter
  3. in the **System information** section open **System Summary** and take note of the value of BIOS mode

### Make some space for arch

Let's begin by doing some wiping. In Windows 10 launch the **Disk Management** tool and create a partition of the desired size by reducing the size of the existing windows installation: 
  1. start the run dialog with `Win key + R`
  2. type `diskmgmt.msc` and press enter
  3. shrink the partition to the desired size

### Create a bootable USB with Arch

I recommend you use [Etcher](https://www.balena.io/etcher/) but you can also use [Rufus](https://rufus.ie/). Simply download the tool, plug in your USB and flash it with the .iso.

### Disable Fast Boot & Secure Boot & opionally switch from RAID to AHCI

This may sound dumb but trust me on this one: since you'll be restarting your computer, read all the steps in this section before following through:
  1. do an online search using you're favourite search engine to lookup what key to press during boot to enter the UEFI settings
  2. restart your computer and press it
  3. the UEFI interface varies depending on your motherboard but they shouldn't be too hard to find:
     1. disable Fast Boot
     2. diable Secure Boot
     3. change the boot order to prioritise your USB over the Windows Boot Loader
     4. (optional) if you have problems partioning your disk switch from RAID to AHCI
  4. save & exit & restart your computer & drop into the arch root prompt (yay!)

## Fun stuff

### Keyboard setup
If you're using a particular keyboard layout, search for it with:
``` bash
# list available layouts
$ ls /usr/share/kbd/keymaps/**/*.map.gz

# modify the layout by running 
$ loadkeys whatever-your-layout-is1
```

### Connect to the interwebs
``` bash
# Find your interface name
$ iw dev

# use an network manager to do the work for you
$ wifi-menu whatever-your-interface-is

# check your connection
$ ping cloudflare.com
```

### Create partitions for your new system
Here you will need to create 3 partitions: boot (>200MB), swap (match your RAM [3]), and a root partition (whatever space you have left)

``` bash
# find out the name of your drive (sometimes /dev/sda - I'll be using /dev/nvme0n1)
$ fdisk -l

# make sure to check the storage of the drive to verify that you're not formatting your USB stick
$ fdisk /dev/nvme0n1
n                    # create a new partition
<Enter>              # use default partition number
<Enter>              # use default starting sector
+256MB               # or whatever size you want your boot sector to be

n                    # create a new partition
<Enter>              # use default partition number
<Enter>              # use default starting sector
+256MB               # or whatever size you want your swap to be
t                    # change the partition type
<Enter>              # use default partition
19                   # change partition type to Linux swap

n                    # create a new partition
<Enter>              # use default partition number
<Enter>              # use default starting sector
<Enter>              # fill the rest of the disk
t                    # change the partition type
<Enter>              # use default partition
w                    # write the changes to disk & exit
```

You should end up with something similar to this:
``` bash
$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
nvme0n1     259:0    0   477G  0 disk 
├─nvme0n1p1 259:1    0   680M  0 part 
├─nvme0n1p2 259:2    0   128M  0 part 
├─nvme0n1p3 259:3    0 204.8G  0 part 
├─nvme0n1p4 259:4    0   990M  0 part 
├─nvme0n1p5 259:5    0  13.2G  0 part 
├─nvme0n1p6 259:6    0   1.3G  0 part 
├─nvme0n1p7 259:7    0   244M  0 part
├─nvme0n1p8 259:8    0   954M  0 part
└─nvme0n1p9 259:9    0 254.8G  0 part
```

### Prepare your swap & boot partitions
``` bash
# enable your swap
$ swapon /dev/nvme0n1p8

# mount & format your boot & root partitions
$ mount /dev/nvme0n1p9 /mnt
$ mount /dev/nvme0n1p7 /mnt/boot
$ mkfs.ext4 /dev/nvme0n1p9
$ mkfs.ext4 /dev/nvme0n1p7

# mount your UEFI boot partition
$ mkdir /mnt/boot/efi
$ mount /dev/nvme0n1p1 /mnt/boot/efi
```

### Install the base arch packages
``` bash
# optionally add base-devel at the end of the following command for development tools
$ pacstrap /mnt base
```

### Generate your fstab file so that partitions are mounted when your reboot
``` bash
# verify that /, /boot, /boot/efi, and swap are here
$ cat /mnt/etc/fstab
# /dev/nvme0n1p9
UUID=UUID-should-be-here       /               ext4            rw,relatime     0 1

# /dev/nvme0n1p7
UUID=UUID-should-be-here       /boot           ext4            rw,relatime     0 2

# /dev/nvme0n1p1 LABEL=ESP
UUID=UUID-should-be-here       /boot/efi       vfat            rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,utf8,errors=remou
nt-ro   0 2

# /dev/nvme0n1p8
UUID=UUID-should-be-here       none            swap            defaults        0 0
```

### Change root & install GRUB
Now that your base packages have been installed and that your partitions are mounted, let's [change the root](https://wiki.archlinux.org/index.php/chroot) directory for the current running process. These steps are the exact same as arch's installation guide [configure the system](https://wiki.archlinux.org/index.php/Installation_guide#Configure_the_system) section, so feel free to switch, follow those instructions and come back before you start the boot loader setup. If you prefer to stay here, the commands are the exact same except you won't get the usefull documentation links in case of a boo-boo.
``` bash
# change root
$ arch-chroot /mnt

# set the timezone
$ ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
$ hwclock --systohc

# set your locale by uncommenting en_US.UTF-8 UTF-8 in /etc/locale.gen (and any other locales you may use)
$ locale-gen
$ echo "LANG=en_US.UTF-8" > /etc/locale.conf

# set the keyboard layout
$ echo "KEYMAP=whatever-your-keymap-from-the-keyboard-setup-section-is1" > /etc/vconsole.conf

# set your hostname
$ echo "whatever-you-want-your-hostname-to-be" > /etc/hostname

# set the root password
$ passwd
```

And now for the main event: the boot loader installation! For this step to work, the partitions must be properly mounted:
``` bash
$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
nvme0n1     259:0    0   477G  0 disk 
├─nvme0n1p1 259:1    0   680M  0 part /boot/efi
├─nvme0n1p2 259:2    0   128M  0 part 
├─nvme0n1p3 259:3    0 204.8G  0 part 
├─nvme0n1p4 259:4    0   990M  0 part 
├─nvme0n1p5 259:5    0  13.2G  0 part 
├─nvme0n1p6 259:6    0   1.3G  0 part 
├─nvme0n1p7 259:7    0   244M  0 part /boot
├─nvme0n1p8 259:8    0   954M  0 part
└─nvme0n1p9 259:9    0 254.8G  0 part /
```

``` bash
$ pacman -S grub efibootmgr os-prober
$ grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub
$ grub-mkconfig -o /boot/grub/grub.cfg
$ os-prober
```

To check that grub has properly detected the windows boot loader have a look in /boot/efi/EFI/grub/grub.cfg you should see a menuentry that looks a little like this:
``` bash
menuentry 'Windows Boot Manager (on /dev/nvme0n1p1)' --class windows --class os $menuentry_id_option 'osprober-efi-UUID-XXXX' {
	insmod part_gpt
	insmod fat
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root  UUID-XXXX
	else
	  search --no-floppy --fs-uuid --set=root BUUID-XXXX
	fi
	chainloader /EFI/Microsoft/Boot/bootmgfw.efi
}

```

### One last thing
You've partitoned your system, installed your base packages and your bootloader - what could you possibly be missing? Oh yeah the **wpa_supplicant** package used for key negociation when you want to connect via WiFi... I made these mistakes so that you don't have too:
``` bash
# update & install wpa_supplicant along with the easy to use wifi-menu
$ pacman -Syyu wpa_supplicant wifi-menu dialog
```

### Finally we got here
If you got to this stage without any errors, crashes or additional reboots - [please let me know](mailto:philippe@partarrieu.me) because you're probably [the one](https://matrix.fandom.com/wiki/The_One) and I could use your help with some C debugging.

``` bash
$ exit
$ umount -R /mnt
$ reboot
```

## Next steps
There are so many different ways forward from here but I recommend following the [arch general recommendations](https://wiki.archlinux.org/index.php/General_recommendations) to secure your system by doing some system administration before you move on to more exciting stuff. If you want to dive into the rabbit hole you should have a look at [i3](https://i3wm.org/). A good guide to get started is this [i3 installation, configuration & operation post](https://medium.com/@mudrii/arch-linux-installation-on-hw-with-i3-windows-manager-part-2-x-window-system-and-i3-installation-86735e55a0a0). I plan on releasing my own i3 setup guide sometime in the near future.

### Links & Credits

[1] This is recommended to guarantee that the file you downloaded hasn't been tampered with. You shouldn't compare the checksum of the downloaded .iso with the checksum provided to you in the install since if the torrent or ftp server has been compromised it is likely both image and the checksum will have been modified. To verify the checksum, simply follow these steps:
  1. open a command line by running `Win key + R` and typing `cmd` and pressing enter
  2. run `certutil -hashfile C:\path\to\arch-linux.iso MD5` and compare the output to the MD5 checksum on the arch download page

[2] As you can see in the output of `lsblk` I only allocated 1GB for my swap. Since then, I have upgraded this to 8GB. It is also possible to use a [swapfile](https://wiki.archlinux.org/index.php/Swap#Swap_file) instead of creating a swap partition.

[3] Although [this guide](http://tech.memoryimprintstudio.com/dual-boot-installation-of-arch-linux-with-preinstalled-windows-10-with-encryption/) is for setting up an encrypted arch linux system it is clear & easy to follow