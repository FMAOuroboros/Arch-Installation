# My Arch linux installation
This is a step by step manual installation for a basic Arch linux.
The installation follows my preferences and is just meant as reminder
for me or as an inspiration for others interested in Arch Linux 

## Install
* Set german keyboard layout
```
loadkeys de-latin1
```

* Check if UEFI boot  
If the directory does not exist => no UEFI boot  
```
ls /sys/firmware/efi/efivars
```


* Test whether you're connected to the internet. Required to go any further
```
ping archlinux.org
```

* Set time
```
timedatectl set-ntp true
```

* Test time
```
timedatectl status
```

* Setup partitioning
```
cfdisk
```
1 Partition for /boot (~200MB)
1 Partition for everything else

* Load kernel modules for encryption
```
modprobe dm-crypt
```

* Encrypt the working partition

    * Setup encryption of sda2
         ```
         cryptsetup -y -v luksFormat /dev/sda2
         ```

    * Open the encrypted partition
        ```
        cryptsetup open /dev/sda2 cryptroot
        ```

    * Install btrfs on the partition
        ```
        mkfs.btrfs /dev/mapper/cryptroot
        ```

    * Mount the partition
        ```
        mount /dev/mapper/cryptroot /mnt
        ```

    * Check wheather the cryptsetup mapping works as intended
        ```
        umount /mnt
        cryptsetup close cryptroot
        cryptsetup open /dev/sdaX cryptroot
        mount /dev/mapper/cryptroot /mnt
        ```


* Preparing the boot partition
```
mkfs.ext2 /dev/sda1
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

* Install base packages, kernel, etc
```
pacstrap /mnt base
```

* Create and check fstab
```
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

* Change root into the new systems
```
arch-chroot /mnt
```

* Set timezone
```
ln -sf /usr/share/zoneinfo/CET /etc/localtime
```

* Set hardware clock
```
hwclock --systohc
```

* Generate locales
```
locale-gen
```

* Set os locale
```
vi /etc/locale.conf
LANG=en_US.UTF-8
```

* Set keyboard layout
```
vi /etc/vconsole.conf
KEYMAP=de-latin1
```

* Set hostname
```
vi /etc/hostname
myhostname
```

* Add hostname resolution
```
vi /etc/hosts
***1	myhostname.localdomain	myhostname
```

* Install btrfs utils.  
This is required, because the btrfs-progs are not installed during the base installation,
but required while recreating the initramfs.
```
pacman -S btrfs-progs
```

* Edit /etc/mkinitcpio.conf  
Add the missing entries around "block"
```
HOOKS="... keyboard keymap block encrypt ... filesystems ..."
```

* Recreate initramfs
```
mkinitcpio -p linux
```

* Set the root password
```
passwd
```

* Install vim - Just for me to make some editing easier
```
pacman -S vim
```

* Choose a bootloader - I took grub
```
pacman -S grub
```

* Install grub
```
grub-install --target=i386-pc /dev/sda
```

*  Add kernel parameter for encryption
```
vim /etc/default/grub
cryptdevice=/dev/sda2:cryptroot
```

* Create grub config
```
grub-mkconfig -o /boot/grub/grub.cfg
```

* Reboot
```
exit
reboot
```

# Sources
* [Installation guide](https://wiki.archlinux.org/index.php/installation_guide)
* [Encryption](https://wiki.archlinux.org/index.php/dm-crypt/Encrypting_an_entire_system#Simple_partition_layout_with_LUKS)
* [Bootloader](https://wiki.archlinux.org/index.php/Category:Boot_loaders)
