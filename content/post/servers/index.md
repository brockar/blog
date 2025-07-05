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

[[links]]
title = "fail2ban"
website = "https://github.com/fail2ban/fail2ban"
image = "https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fesencialsistemas.com%2Fwp-content%2Fimages%2Ffail2ban-logo.jpg&f=1&nofb=1&ipt=4940baacca29f8d0c8e812ad81b2015a1840eeb403262f513fb52e8c4e8b9431"

+++

## Server Essentials

### SSH with Key Authentication

To secure your server, configure SSH to allow only key-based authentication and disable password logins. Also, create a non-root user for SSH access and disable root login:

1. **Create a new user and add to sudo group:**
   ```bash
   sudo adduser user
   sudo usermod -aG sudo user
   ```
   Replace `user` with your preferred username.

2. **Add your public key to the new user:**
   - Copy your public key from your PC to the server for the new user:
     ```bash
     ssh-copy-id user@your_server_ip
     ```
   - Or manually append your public key to `/home/user/.ssh/authorized_keys`.

3. **Edit the SSH daemon configuration:**
   - Open the SSH config file:
     ```bash
     sudo nvim /etc/ssh/sshd_config
     ```
   - Set or update these lines:
     ```config
     PubkeyAuthentication yes
     PasswordAuthentication no
     PermitRootLogin no
     ChallengeResponseAuthentication no
     UsePAM no
     ```

4. **Restart SSH service:**
   ```bash
   sudo systemctl restart ssh
   ```

**Note:**
- **Before closing your root session, open a new terminal and test logging in as the new user**.
- Disabling password authentication and root login increases security by preventing brute-force and privilege escalation attacks.
> **Important:**  
> Keep your SSH private key secure. If you lose your key, you will lose access to the server. Always keep a backup in a safe place, such as another device you own or a secure storage location.


### File Sharing with SMB

For network file sharing between Linux and other systems, I use Samba. Check my detailed **[Samba configuration guide]({{< relref "post/samba/index.md" >}})** for implementation details.

### Security Monitoring

- **fail2ban**: Essential for protecting against brute force attacks by monitoring log files and banning suspicious IP addresses.

1. **Install fail2ban:**
   ```bash
   sudo apt update
   sudo apt install fail2ban
   ```

2. **Basic configuration:**
   - Copy the default config to create a local override:
     ```bash
     sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
     ```
   - Edit `/etc/fail2ban/jail.local` to adjust settings (e.g., ban time, findtime, maxretry).

3. **Enable and start the service:**
   ```bash
   sudo systemctl enable --now fail2ban
   ```

4. **Check status:**

   ```bash
   sudo fail2ban-client status
   sudo fail2ban-client status sshd
   ```

fail2ban will now monitor for suspicious activity and ban offending IPs automatically. For more advanced configuration, refer to the official [documentation](https://github.com/fail2ban/fail2ban) or add custom jails as needed.

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
I usually run this with [screen](https://www.gnu.org/software/screen/). To keep running NeverIdle in the background.

```bash
# Example installation
wget https://github.com/layou233/NeverIdle/releases/latest/download/NeverIdle-linux-arm64 -O NeverIdle
chmod +x NeverIdle
./NeverIdle -c 2h -m 2 -n 4h
```

This configuration:

- Uses 2 CPU cores every 2 hours (`-c 2h -m 2`)
- Performs network activity every 4 hours (`-n 4h`)
