---
layout: post
title:  "Encrypted Arch installation"
date:   2022-01-18 21:09:010 +0200
categories: blog
---


I recently setup a new desktop with an encrypted Arch disk. This is a slightly modified version of my [Arch/Windows dual-boot guide](https://www.partarrieu.me/blog/2019/09/24/arch-dual-boot.html) with additional steps to encrypt the root partition. I removed the fluff and kept the key parts of the installation. If you run into any problems please be sure to read through the [Arch Installation guide](https://wiki.archlinux.org/index.php/Installation_guide) and the [dm-crypt guide](https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system)

<!--more-->

This guide is for HDD disks and a motherboard supporting UEFI.

## Let's get started!

### Connect to the internet
``` bash
# if you have a wireless connection
$ iw dev
$ wifi-menu whatever-your-interface-is

# if you have a wired connection
$ ip link set dev enp3s0 up

# check your connection
$ ping cloudflare.com
```

### Create partitions for your new system
Here you will need to create 3 partitions: UEFI (100MB), boot (500MB), and a root partition (whatever space you have left)

``` bash
# find out the name of your drive (sometimes /dev/sda - I'll be using /dev/nvme0n1)
$ fdisk -l

# make sure to check the storage of the drive to verify that you're not formatting your USB stick
$ fdisk /dev/sda
n                    # create a new partition
<Enter>              # use default partition number
<Enter>              # use default starting sector
+100MB               # or whatever size you want your UEFI sector to be
t                    # change the partition type
<Enter>              # use default partition
1                    # use EFI System partition type

n                    # create a new partition
<Enter>              # use default partition number
<Enter>              # use default starting sector
+500MB               # or whatever size you want your boot to be
t                    # change the partition type
<Enter>              # use default partition
82                   # use Linux partition type

n                    # create a new partition
<Enter>              # use default partition number
<Enter>              # use default starting sector
<Enter>              # fill the rest of the disk
t                    # change the partition type
<Enter>              # use default partition
82                   # use Linux partition type
w                    # write the changes to disk & exit
```

You should end up with something similar to this:
``` bash
$ fdisk -l
Device       Start        End    Sectors  Size Type
/dev/sda1     2048     206847     204800  100M EFI System
/dev/sda2   206848    1230847    1024000  500M Linux filesystem
/dev/sda3  1230848 7814037134 7812806287  3.6T Linux filesystem
```

### Encrypt your root partition

```
cryptsetup -y -v luksFormat /dev/sda3
cryptsetup open /dev/sda3 cryptroot
```

### Prepare your swap & boot partitions
``` bash
# format the partitions
$ mkfs.ext4 /dev/mapper/cryptroot
$ mkfs.ext4 /dev/sda2
$ mkfs.fat -F32 /dev/sda1

# create the necessary directory hierarchy
$ mkdir -p /mnt/boot/efi

# mount the partitions
$ mount /dev/mapper/cryptroot /mnt
$ mount /dev/sda2 /mnt/boot
$ mount /dev/sda1 /mnt/boot/efi
```

### Install the base Arch packages
``` bash
# optionally add base-devel at the end of the following command for development tools
# intel-ucode is for cpu microcode updates
$ pacstrap /mnt base base-devel linux linux-firmware intel-ucode
```

### Generate your fstab file so that partitions are mounted when your reboot
The fstab (short for filesystems table) is used to list disk partitions or various block devices and specify how they should be mounted into the filesystem.
``` bash
# generate the fstab file
$ genfstab -U /mnt >> /mnt/etc/fstab

# verify that /, /boot, /boot/efi are present
$ cat /mnt/etc/fstab
# /dev/mapper/cryptroot
UUID=94efc2aa-302f-4243-b456-940f772aa0d9       /               ext4            rw,relatime     0 1

# /dev/sda2
UUID=89184e86-af87-4493-b7d0-8ff7e73b6391       /boot           ext4            rw,relatime,stripe=4    0 2

# /dev/sda1
UUID=F841-6718          /boot/efi       vfat            rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,utf8,errors=remount-ro     0 2

```

### Change root & install GRUB
Now that your base packages have been installed and that your partitions are mounted, let's [change the root](https://wiki.archlinux.org/index.php/chroot) directory for the current running process. These steps are the exact same as arch's installation guide [configure the system](https://wiki.archlinux.org/index.php/Installation_guide#Configure_the_system) section, so feel free to switch, follow those instructions and come back before you start the boot loader setup.
``` bash
# change root
$ arch-chroot /mnt

# set the timezone
$ ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
$ hwclock --systohc

# set your locale by uncommenting en_US.UTF-8 UTF-8 in /etc/locale.gen (and any other locales you may use)
$ locale-gen
$ echo "LANG=en_US.UTF-8" > /etc/locale.conf

# set your hostname
$ echo "hostname" > /etc/hostname

# set the root password
$ passwd
```

### Setup the crypttab
To mount your encrypted drive at boot time, enter the device name in /etc/crypttab as shown below
```bash
$ cat /etc/crypttab
# <name>       <device>                                     <password>              <options>
cryptroot      /dev/mapper/cryptroot                        none                    luks,timeout=180
```

### Initramfs & mkinitcpio
Add the encrypt hook to /etc/mkinitcpio.conf as shown below
```bash
$ grep HOOKS /etc/mkinitcpio.conf | tail -1
HOOKS=(base udev autodetect modconf block filesystems keyboard fsck encrypt)
```

### Install & configure grub
And now for the main event: the boot loader installation! For this step to work, the partitions must be properly mounted:
``` bash
$ lsblk -f
NAME          FSTYPE      FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda                                                                                       
├─sda1        vfat        FAT32       F841-6718                              98.3M     0% /boot/efi
├─sda2        ext4        1.0         89184e86-af87-4493-b7d0-8ff7e73b6391  368.6M    17% /boot
└─sda3        crypto_LUKS 2           9f7be7b1-7ce7-4579-adbf-f2c8d8e843f0                
  └─cryptroot ext4        1.0         94efc2aa-302f-4243-b456-940f772aa0d9    3.4T     0% /
```

``` bash
$ pacman -S grub efibootmgr os-prober
$ grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub
$ os-prober
```

In order to unlock the encrypted root partition at boot time, you will need to edit the kernel parameters in the boot loader as shown below:
```bash
$ grep GRUB_CMDLINE_LINUX_DEFAULT /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="rootdelay=10 loglevel=3 quiet cryptdevice=UUID=9f7be7b1-7ce7-4579-adbf-f2c8d8e843f0:cryptroot"

# generate the grub config file
$ grub-mkconfig -o /boot/grub/grub.cfg
```

### Finally we got here
If you got to this stage without any errors, crashes or additional reboots - [please let me know](mailto:philippe@partarrieu.me) because you're probably [the one](https://matrix.fandom.com/wiki/The_One) and I could use your help debuggin some C.

``` bash
$ exit
$ umount -R /mnt
$ reboot
```

## Post-install steps
These are very personal configuration steps, feel free to stop following the guide here.

### SSH
It's much easier to finish the configuration by SSHing into this computer. This way you can use tmux, copy/paste from your mouse and use your browser to look things up rather than faff about trying to install a display manager then a browser.
```bash
# set the two properties below in your sshd_config
# PermitRootLogin yes
# PasswordAuthentication yes
$ vim /etc/ssh/sshd_config

$ systemctl start sshd

# find your ip address
$ ip a
```

From another computer, connect using
```bash
$ ssh root@<ip>
```

### Install an Arch User Repository (AUR) package manager
```bash
mkdir src && cd src
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

### Install packages
```bash
yay -Sy xorg-server xorg-apps xorg-xinit xorg-twm lightdm lightdm-webkit-theme-aether i3-gaps vim unzip tmux htop git zsh firefox chromium vlc libsecret gnome-keyring libgnome-keyring xclip zsh feh urxvt curl compton mailspring rofi i3lock-color polybar code noto-fonts noto-fonts-emoji alsa-utils alsa-plugins alsa-lib pavucontrol udisks2 xbindkeys protonvpn-cli wireguard flameshot neofetch
```
### Configure git
I've configured a lot of machines with git in the past so I've written a little helper script to do the work. Get the script from [here](https://gist.github.com/ppartarr/72964efb5e60a3cb11774a32ddd0e779) and run
```bash
chmod +x git.sh
# I like signing my git commits but you can set -gpg to false if you don't want to bother
./git.sh -e $EMAIL -n $FULL_NAME -g true
```

### Dotfiles
```bash
git clone git@github.com:ppartarr/dotfiles.git ~/.dotfiles
cd ~/.dotfiles
./setup.sh
```

### Oh-my-zsh
```bash
# install oh-my-zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

# install plugins
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# clone my zsh theme from gist
git clone git@gist.github.com:6eeb9585dcbc3c6a8db79982ec8c21ad.git ~/.oh-my-zsh/custom/themes/haskell.zsh-theme
```
