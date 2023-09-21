### Set Timezone 
```console
# timedatectl set-timezone "Pacific/Honolulu"
```
### Partition Disks 
check your disk drives
```console 
# lsblk 
```
![lsblk output](/assets/screenshot_1.jpg)

Partition the disk 
use gdisk (for efi systems) 
```console
# gdisk /dev/sda
```
sda can be is a substitue for what your disk drive is called. for this vbox install mine is called sda but can be vda or anything else 

![gdisk](/assets/screenshot_2.jpg)

gdisk commands for creating a new partitions 
- 300M Partition for the efi boot loader
- Rest of the partition for the btrfs file system 

```console
# lsblk
```
![lsblk_2](/assets/screenshot_3.jpg)

Now there are 2 partitions we will need to format them accordingly
- efi bootloader partition
- btrfs partition

```console
# mkfs.fat -F32 /dev/sda1
# mkfs.btrfs /dev/sda2
```

### Mount partitions
The reson why you are mounting the partitions to the /mnt is that you are currently on the .iso image file system using these commands. You need to mount your newly created file systems on your /dev/sda (which is your current computer drive) to be able to install arch on it. 

```console
# mount /dev/sda2 /mnt
# btrfs subvolume create /mnt/@
# btrfs subvolume create /mnt/@home
# btrfs subvolume create /mnt/@snapshots
# btrfs subvolume create /mnt/@var_log
```

then we have to unmount the mount directory and then remount each subvolume to it's own respective directory. 

```console
# umount /mnt
# mount -o noatime,compress=lzo,space_cache=v2,subvol=@ /dev/sda2 /mnt
# mkdir -p /mnt/{boot,home,.snapshots}
# mkdir -p /mnt/var/log
# mount -o noatime,compress=lzo,space_cache=v2,subvol=@home /dev/sda2 /mnt/home
# mount -o noatime,compress=lzo,space_cache=v2,subvol=@snapshots /dev/sda2 /mnt/.snapshots
# mount -o noatime,compress=lzo,space_cache=v2,subvol=@var_log /dev/sda2 /mnt/var/log
```


btrfs mount options: 
- noatime - no access time 
- compress - compression options (we picked lzo due to tutorial followed)
- space_cache - help btrfs know where are free blocks on the partition
- subvol - which subvolume to mount to which partition


Mount boot partition
```console
# mount /dev/sda1 /mnt/boot
```

### Install arch linux onto system

```console
# pacstrap /mnt base linux linux-firmware
```
optional additions: 
- amd-ucode (amd systems)
- intel-ucode (intel systems) 
- vim (vim editor)

Generate fstab to autoload mount point on boot
```console
# genfstab -U /mnt >> /mnt/etc/fstab
```


Take a look in fstab file 
```console
 # cat /mnt/etc/fstab
```

### Go to root of Arch 
```console
# arch-chroot /mnt
```

create symbolic link 
```console
# ln -sf /usr/share/zoneinfo/Pacific/Honolulu /etc/localtime
# hwclock --systohc
```

first find all the localization files

```console
# vim /etc/locale.gen
```

find and uncomment 'en_US, UTF-8 UTF-8'

![local_gen](/assets/screenshot_4.jpg)

Generate locale character set for your language
```console
# locale-gen
```

create a new locale.conf and set the language argument
```console
# echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

### Network Configuration
```console
# vim /etc/hostname
```

Name whatever hostname you would like

![hostname](/assets/screenshot_5.jpg)

### _optional_

Edit hosts file
```console
# vim /etc/hosts
```

![host_file](/assets/screenshot_6.jpg)

### Root Password
```console
# passwd
```

![rootpwd](/assets/screenshot_7.jpg)

### Boot loader 

Install grub 
```console
# pacman -S grub efibootmgr
```

configure mkinitcpio 
```console
# vim /etc/mkinitcpio.conf
```

![mkinitcpio](/assets/screenshot_8.jpg)

regenerate kernel image with btrfs module included
```console
# mkinitcpio -p linux
```

install grub bootloader 
```console
# grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```
note: the efi directory is not not /mnt/boot b/c the <code>[root@archiso /] #</code> is already in the /mnt directory. We moved in here we we defined the archroot cmd

generate config file for grub
```console
# grub-mkconfig -o /boot/grub/grub.cfg
```

exit arch root 
```console
# exit
```

unmount partitions 
```console
# umount -a
```

reboot system
```console
# reboot
```

Remove install disk


## Post Install 
### Set up snapshots

Install Snapper package
```console
# packman -S snapper
```

Unmount and delete all references to .snapshots directory (why did I even do this in the first place?)

I guess to get the fstab directory to be correctly created before configuring snapper

```
# sudo umount /.snapshots
# sudo rm -r /.snapshots
# sudo snapper -c root create-config /
# sudo btrfs subvolume delete /.snapshots
# sudo mkdir /.snapshots
# sudo mount -a 
# sudo chmod 750 /.snapshots
# sudo vim /etc/snapper/configs/root
```

Suggested snapshot config froma arch wiki

![snapshot_config](/assets/screenshot_9.jpg)



optinal installs
- networkmanager
- network-manager-applet
- dialog 
- wpa_supplicant 
- mtools 
- dosfstools
- git
- reflector
- snapper 
- bluez 
- bluez-utils 
- cups
- xdg-utils 
- xdg-user-dirs
- alsa-utils
- pulseaudio
- pulseaudio-bluetooth
- inetutils 
- base-devel 
- linux-headers
- bash-completion


















