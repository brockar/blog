+++
author = "brockar"
title = "Nvidia on Linux"
description = "Nvidia Graphics Setup on Debian SID with Secure Boot"
date = '2025-05-06'
categories = [
    "Linux"
]
tags = [
    "linux",
    "nvidia",
    "gpu"
]
image = "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSVCKqDEVDa7m8R_BGTKftq-nJ_HedtjkJPnw&s"

[[links]]
title = "Debian docs"
website = "https://wiki.debian.org/NvidiaGraphicsDrivers#Debian_Unstable_.22Sid.22"
image = "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRqMV5jTVWkdHRNkn3Iz0Bn_DuTwsWmp0gE1w&s"

[[links]]
title = "Debian Secure Boot"
website = "https://wiki.debian.org/SecureBoot#MOK_-_Machine_Owner_Key"
image = "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRqMV5jTVWkdHRNkn3Iz0Bn_DuTwsWmp0gE1w&s"

[[links]]
title = "Kinetica Nvidia Docs"
website = "https://docs.kinetica.com/7.1/install/nvidia_deb/"
image = "https://docs.kinetica.com/7.1/images/kinetica_university_blue_web.svg"
+++
## Nvidia Drivers Installation

First check the specific steps for your Debian version in the [official documentation](https://wiki.debian.org/NvidiaGraphicsDrivers#Debian_Unstable_.22Sid.22).

### 1. Add Required Repositories

Edit your `/etc/apt/sources.list` to include `contrib non-free non-free-firmware` components:

```bash
# Example for Debian Sid
deb http://deb.debian.org/debian/ sid main contrib non-free non-free-firmware
```

### 2. Install Nvidia Drivers

Update packages and install the necessary components:

```bash
sudo apt update
sudo apt install linux-headers-amd64
sudo apt install nvidia-driver firmware-misc-nonfree
```

## Secure Boot Configuration

If you have Secure Boot enabled, you'll need to enroll the MOK (Machine Owner Key):

```bash
sudo mokutil --import /var/lib/dkms/mok.pub  # Prompts for one-time password
```

After rebooting:
1. The MOK manager will appear
2. Select "Enroll MOK"
3. Confirm the key enrollment using your one-time password

Verify the MOK was loaded correctly:

```bash
sudo mokutil --test-key /var/lib/shim-signed/mok/MOK.der
```

Expected output:
```
/var/lib/shim-signed/mok/MOK.der is already enrolled
```

## Troubleshooting

### Failed to Start Nvidia Persistence Daemon

If you encounter issues with the persistence daemon:

```bash
sudo apt purge nvidia-*
sudo apt install nvidia-kernels-dkms
```

Then repeat the driver installation steps from the beginning.

## Power Management

### Undervolting Nvidia GPU

Create a systemd service to manage power limits:

```bash
sudo vim /etc/systemd/system/nvidia-powerlimit.service
```

Add this content:

```ini
[Unit]
Description=Set Nvidia power limit

[Service]
Type=oneshot
ExecStartPre=/usr/bin/nvidia-smi --persistence-mode=1
ExecStart=/usr/bin/nvidia-smi --power-limit=125
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Enable the service:

```bash
sudo chmod 644 /etc/systemd/system/nvidia-powerlimit.service
sudo systemctl daemon-reload
sudo systemctl enable nvidia-powerlimit.service
```

## Thermal Management

### Fan Control

Add this to root's crontab (`sudo crontab -e`):

```bash
@reboot nvidia-settings -V -c :0 -a '[gpu:0]/GPUFanControlState=1' -a '[fan:0]/GPUTargetFanSpeed=50' -a '[fan:1]/GPUTargetFanSpeed=50'
```

## Wayland Support

For Wayland compatibility, consider using [TuxClocker](https://github.com/Lurkki14/tuxclocker), a Qt-based overclocking tool that works with Nvidia cards.