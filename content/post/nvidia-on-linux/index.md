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

[[links]]
title = "NVML-GPU-Control"
website = "https://github.com/HackTestes/NVML-GPU-Control"
image = "https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fpngimg.com%2Fuploads%2Fgithub%2Fgithub_PNG80.png&f=1&nofb=1&ipt=b6c906ba477cf0eaf58b87eba210263c12f7074f3c39761c1eb60c0096c06c8b"
+++

> Note: I don't use NVIDIA anymore so this update will probably be the last one I make to this post. 

## Nvidia Drivers Installation

First check the specific steps for your Debian version in the [official documentation](https://wiki.debian.org/NvidiaGraphicsDrivers#Debian_Unstable_.22Sid.22).

### Add Required Repositories

Edit your `/etc/apt/sources.list` to include `contrib non-free non-free-firmware` components:

```bash
# Example for Debian Sid
deb http://deb.debian.org/debian/ sid main contrib non-free non-free-firmware
```

### Install Nvidia Drivers

Update packages and install the necessary components:

```bash
sudo apt update
sudo apt install linux-headers-amd64
sudo apt install nvidia-driver firmware-misc-nonfree
```

### Secure Boot Configuration 

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

### Verify Installation

After completing the driver installation (and rebooting if you enrolled MOK keys), verify that the Nvidia drivers are working correctly with:

```bash
nvidia-smi
```

## NVIDIA Management

### Cooler management
For managing Nvidia GPU coolers, I use [NVML-GPU-Control](https://github.com/HackTestes/NVML-GPU-Control) which is a simple Python script that allows you to control the GPU fans with a curve. 

You can install it with:

```bash
sudo pipx install --global caioh-nvml-gpu-control
```
and then run it with:

```bash
sudo chnvml fan-control -n 'NVIDIA GeForce RTX 3070' -sp '10:35,20:50,30:50,35:100'
```

Also I put it on my crontab to run at boot:

```bash
@reboot /usr/local/bin/chnvml fan-control -n 'NVIDIA GeForce RTX 3070' -sp '10:35,20:50,30:50,35:100'
```

#### Official cooler management
As an alternative, you can use the official Nvidia settings tool for fan control.
It just sets the fan speed to 50% on boot, which is not ideal for cooling but works if you don't want to use a custom script.
Add this to root's crontab (`sudo crontab -e`):

```bash
@reboot nvidia-settings -V -c :0 -a '[gpu:0]/GPUFanControlState=1' -a '[fan:0]/GPUTargetFanSpeed=50' -a '[fan:1]/GPUTargetFanSpeed=50'
```

### Undervolting Nvidia GPU

Undervolting (reducing power consumption) your GPU can provide several benefits reducing the maximum voltage while maintaining stable performance. This can lead to:

- **Lower temperatures** - Reduced power consumption means less heat generation
- **Quieter operation** - Lower temps allow fans to run slower and quieter
- **Extended GPU lifespan** - Cooler operation reduces thermal stress on components
- **Energy savings** - Lower power consumption reduces electricity costs
- **Better performance stability** - Prevents thermal throttling during heavy loads

Power limiting is a simple form of undervolting that caps the maximum power draw without complex voltage adjustments.

#### Using crontab (recommended)

The simplest approach is using crontab. Add this to root's crontab (`sudo crontab -e`):

```bash
@reboot /usr/bin/nvidia-smi --persistence-mode=1 && /usr/bin/nvidia-smi --power-limit=125
```

> **Note:** Replace `125` with your desired power limit in watts. Check your GPU's maximum power limit with `nvidia-smi -q -d POWER` before setting a value.

#### Using systemd service (alternative)

For users who prefer systemd services, create a service to manage power limits:

```bash
sudo nvim /etc/systemd/system/nvidia-powerlimit.service
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

## GUI Controllers

I can recommend these two GUI tools for managing Nvidia GPUs:
 - [LACT](https://github.com/ilya-zlobintsev/LACT) Originally designed for AMD GPUs, but it also supports Nvidia cards now. It has GPU info, overclocking, and fan control features. You can see the hardware compatibility list [here](https://github.com/ilya-zlobintsev/LACT/wiki/Hardware-Support#nvidia).  

 - Also can try [TuxClocker](https://github.com/Lurkki14/tuxclocker), for almost the same functionality. I prefer LACT because it has more features and is actively developed but TuxClocker is also a good option.

## Troubleshooting

### Failed to Start Nvidia Persistence Daemon

If you encounter issues with the persistence daemon:

```bash
sudo apt purge nvidia-*
sudo apt install nvidia-kernels-dkms
```

Then repeat the [driver installation steps](#nvidia-drivers-installation) from the beginning.