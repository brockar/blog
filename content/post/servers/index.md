+++
author = "brockar"
title = "My Linux Server Config"
date = '2025-05-06'
description = "Essential tools and configurations I use on my Debian servers"
categories = [
    "Linux"
]
tags = [
    "linux",
    "server",
    "debian",
    "configuration"
]
image = "https://wiki.installgentoo.com/images/thumb/4/4b/Clark_griswold_builds_a_server.png/500px-Clark_griswold_builds_a_server.png"
+++

## Server Essentials

### File Sharing with SMB

For network file sharing between Linux and other systems, I use Samba. Check my detailed **[Samba configuration guide]({{< relref "post/samba/index.md" >}})** for implementation details.

### Security Monitoring

- **fail2ban**: Essential for protecting against brute force attacks by monitoring log files and banning suspicious IP addresses.

### Docker Containers

I run several services in Docker containers for easy management:

- **[Homepage](https://github.com/gethomepage/homepage)**: A simple dashboard for my services
- **VSCode Server**: Cloud-based VS Code instance for remote development
- **Portainer**: Web UI for Docker management
- **Uptime Kuma**: Service monitoring and uptime checker
- **Grafana**: Metrics visualization platform (often paired with Prometheus)

## Laptop Server Specifics

### Disabling Display on Boot

For headless laptop servers, I disable the display during boot:

1. Install required package:
   ```bash
   sudo apt install vbetool
   ```

2. Turn display off:
   ```bash
   sudo vbetool dpms off
   ```

3. To turn back on:
   ```bash
   sudo vbetool dpms on
   ```

To auto turn off screen I use crontab.
`sudo crontab -e`
```bash
@reboot sleep 60 && /usr/sbin/vbetool dpms off
```

### Preventing Sleep When Lid Closed

To keep the server running when lid is closed:

1. Edit the login manager configuration:
   ```bash
   sudo nano /etc/systemd/logind.conf
   ```

2. Change:
   ```ini
   #HandleLidSwitch=suspend
   ```
   To:
   ```ini
   HandleLidSwitch=ignore
   ```

3. Restart the service:
   ```bash
   sudo systemctl restart systemd-logind
   ```

## Boot Optimization

### Skipping GRUB Menu

To speed up boot time by skipping the GRUB menu:

1. Edit GRUB configuration:
   ```bash
   sudo vim /etc/default/grub
   ```

2. Modify these lines:
   ```cfg
   GRUB_TIMEOUT=0
   GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
   ```

3. Update GRUB:
   ```bash
   sudo update-grub
   ```

## Oracle Cloud Free Tier Tips

To prevent Oracle from reclaiming your ARM instances due to inactivity:

Use **[NeverIdle](https://github.com/layou233/NeverIdle/blob/master/README_en.md)** - a simple tool that keeps your instance active by periodically utilizing resources.

```bash
# Example installation
wget https://github.com/layou233/NeverIdle/releases/latest/download/NeverIdle-linux-arm64 -O NeverIdle
chmod +x NeverIdle
./NeverIdle -c 2h -m 2 -n 4h
```

This configuration:
- Uses 2 CPU cores every 2 hours (`-c 2h -m 2`)
- Performs network activity every 4 hours (`-n 4h`)
```
