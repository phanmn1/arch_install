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

Other dependencies to install: 
- vim

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
```console
# locale-gen
```





















