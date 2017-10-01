# Extended installation
This extended installation covert user management, Desktop environment, etc.

1. Delete the home folder 
```rm -r /home```

2. setup a btrfs subvolume for home
```btrfs subvolume create /home```

3. enable network again
```
ip link #get device name
ip link set <devicename> up
dhcpch <devicename>
```

4. install sudo
```pacman -S sudo```

5. grant yourself root privileges
```
visudo
<username> ALL=(ALL) ALL
```

6. Add the user you granted root privileges for
```
useradd -m -s /bin/bash <username>
```

7. setup your password
```
passwd <username>
```

8. Install Xorg
```
pacman -S xorg-server
```

9. Install xfce4-goodies
```
pacman -S xfce4-goodies
```

10. Install display manager
```
pacman -S lightdm lightdm-gtk-greeter
```

11. Enable display manager
```
pacman -S enable lightdm.service
```