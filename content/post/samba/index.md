+++
author = "brockar"
title = "Samba"
date = '2025-05-06'
description = "My way to install and configure samba"
categories = [
    "Linux"
]
tags = [
    "linux",
    "server",
    "debian",
    "samba",
    "networking"
]
image = "https://i.redd.it/hqqvc4qq3w1e1.png"

[[links]]
title = "Ubuntu setup"
website = "https://ubuntu.com/tutorials/install-and-configure-samba#4-setting-up-user-accounts-and-connecting-to-share"
image = "https://repository-images.githubusercontent.com/335476519/61ef634e-0b5f-4d27-9fb6-c64d526c595c"

[[links]]
title = "Odisea Geek"
website = "https://odiseageek.es/posts/montar-unidades-smb-o-cifs-en-linux-con-mount-o-fstab/"
image = "https://avatars.githubusercontent.com/u/60941836?v=4"

[[links]]
title = "Cuaderno de bitácora"
website = "https://jlsmorilloblog.wordpress.com/2017/06/25/montar-carpeta-de-red-en-linux/"
image = "https://cdn-icons-png.flaticon.com/512/123/123793.png"
+++

# Samba

Depends on which linux you are, use your package manager, in this post I'll use apt for Debian or derivates.
For arch you can use pacman/yay/paru or whatever.

## Install samba
``` bash
sudo apt update
sudo apt install samba 
```
Check if it was installed
``` bash
whereis samba
```
the output must to be something like `samba: /usr/lib/samba`

## Config 

### Make a dir
Make the folder to share, u can put it where you want and with the name that u want, if you want to have a folder on your home
```bash
mkdir ~/smbshare
```
### Samba config
Edit samba config on `/etc/samba/smb.conf`

```bash
sudo nvim /etc/samba/smb.conf
```
At the end of file add your share
```conf
[sambashare]
    comment = Samba Share
    path = /home/username/sambashare
    read only = no
    browsable = yes
```

Save and exit with `:wq`

Restart samba service
```bash
sudo systemctl restart smbd
```

### Setup an user for samba
```bash
sudo smbpasswd -a username
```

## Accessing Network Shares

### Prerequisites

First, install the necessary utilities:

```bash
sudo apt-get install cifs-utils smbclient
```

### Temporary Mount (Manual Method)

1. Create a local directory where you'll mount the network share:

```bash
sudo mkdir /media/share_name
```

2. Mount the network share using this command:

```bash
sudo mount -t cifs //server/share_name /media/share_name -o username=samba_user,password=your_password
```

Where:
- `server` is the hostname or IP of the computer sharing the folder
- `share_name` is the name of the shared resource
- `samba_user` is the user (local or domain) with access permissions

Example:

```bash
sudo mount -t cifs //192.168.0.20/smbshare /home/user/Documents/share -o username=user123,password=pass123
```

### Permanent Mount (via fstab)

To make the mount persistent across reboots, edit `/etc/fstab`:

```bash
sudo nvim /etc/fstab
```

Add this line:

```
//server/share_name  /media/share_name  cifs  username=samba_user,password=your_password,noexec,user,rw,nounix,uid=1000,iocharset=utf8  0  0
```

For a more robust configuration that handles network delays and systemd integration:

```
//server/share_name  /local/mount_point  cifs  username=samba_user,password=your_password,iocharset=utf8,uid=local_user,gid=local_group,forceuid,forcegid,cifsacl,noauto,x-systemd.automount,_netdev  0  0
```

Key additional options:
- `noauto,x-systemd.automount`: Delays mounting until first access
- `_netdev`: Ensures network is available before mounting

## Extras

For more detailed information, consult these Arch Linux wiki pages (which are excellent resources even for non-Arch users):
- [Samba](https://wiki.archlinux.org/index.php/samba)
- [fstab](https://wiki.archlinux.org/index.php/Fstab)
