# My Arch linux installation
## Install
1. Set german keyboard layout
```loadkeys de-latin1```

2. Check if UEFI boot
```ls /sys/firmware/efi/efivars```
If the directory does not exist => no UEFI boot

3. test whether you're connected to the internet. Required to go any further
```ping archlinux.org```


4. set time
```timedatectl set-ntp true```

5. test time
```timedatectl status```

6. setup partitioning
```cfdisk```
1 Partition for /boot (~200MB)
1 Partition for everything else

7. load kernel modules for encryption
```modprobe dm-crypt```

8. Encryption
8.1. Encrypt the working partition
8.1.1. Setup encryption of sda2
```cryptsetup -y -v luksFormat /dev/sda2```

8.1.2. Open the encrypted partition
```cryptsetup open /dev/sda2 cryptroot```

8.1.3. Install btrfs on the partition
```mkfs.btrfs /dev/mapper/cryptroot```

8.1.4. Mount the partition
```mount /dev/mapper/cryptroot /mnt```

8.1.5. Check the mapping works as intended
```
umount /mnt
cryptsetup close cryptroot
cryptsetup open /dev/sdaX cryptroot
mount /dev/mapper/cryptroot /mnt
```


8.2. Preparing the boot partition
```
mkfs.ext2 /dev/sda1
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

9. Install base packages
```pacstrap /mnt base```

10. Create and check fstab
```
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

11. Change root into the new systems
```arch-chroot /mnt```

12. Set timezone
```ln -sf /usr/share/zoneinfo/CET /etc/localtime```

13. set hardware clock
```hwclock --systohc```

14. generate locales
```locale-gen```

15. set os locale
```
vi /etc/locale.conf
LANG=en_US.UTF-8
```

16. set keyboard layout
```
vi /etc/vconsole.conf
KEYMAP=de-latin1
```

17. Set hostname
```
vi /etc/hostname
myhostname
```

18. Add hostname resolution
```
vi /etc/hosts
127.0.1.1	myhostname.localdomain	myhostname
```

19. Instapacll btrfs utils
This is required, because the btrfs-progs are not installed during the base installation,
but required whilre recreating the initramfs.
```pacman -S btrfs-progs```

20. Edit /etc/mkinitcpio.conf
```HOOKS="... keyboard keymap block encrypt ... filesystems ..."```

21. recreate initramfs
```mkinitcpio -p linux```

22. Set the root password
```passwd```

23. Install vim - Just for me to make some editing easier
```pacman -S vim```

24. Choose a bootloader - I took grub
```pacman -S grub```

25. Install grub
```grub-install --target=i386-pc /dev/sda```

26.  Add kernel parameter for encryption
```
vim /etc/default/grub
cryptdevice=/dev/sda2:cryptroot
```

27. Create grub config
```grub-mkconfig -o /boot/grub/grub.cfg```

28. Reboot
```
exit
reboot
```

# Sources
* [Installation guide](https://wiki.archlinux.org/index.php/installation_guide)
* [Encryption](https://wiki.archlinux.org/index.php/dm-crypt/Encrypting_an_entire_system#Simple_partition_layout_with_LUKS)
* [Bootloader](https://wiki.archlinux.org/index.php/Category:Boot_loaders)
