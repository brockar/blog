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
- **[VSCode Server](https://github.com/coder/code-server)**: Cloud-based VS Code instance for remote development
- **[Immich](https://github.com/immich-app/immich)**: Self-hosted photo and video backup solution
- **[Calibre](https://hub.docker.com/r/linuxserver/calibre)**: E-book management and server
- **[FreshRSS](https://github.com/FreshRSS/FreshRSS)**: RSS feed aggregator
- **[Vikunja](https://vikunja.io/)**: Personal wiki and note-taking app
- **[Nextcloud](https://nextcloud.com/)**: Self-hosted cloud storage and collaboration platform
- **[Paperless](https://github.com/paperless-ngx/paperless-ngx)**: Document management system
- **[Portainer](https://www.portainer.io/)**: Web UI for Docker management
- \ **[Uptime Kuma](https://github.com/louislam/uptime-kuma)**: Service monitoring and uptime checker
- \ **[Grafana](https://grafana.com/)**: Metrics visualization platform (works with Prometheus)
- **[Beszel](https://beszel.dev/)**: Simple, lightweight server monitoring

> **Note:**  
> For home server monitoring, I now use **Beszel** instead of Uptime Kuma or Grafana. In the past, I used those tools, but found Beszel provides simple, lightweight status checks and metrics that are sufficient for most personal setups.

#### Securing Docker with userns-remap (Optional)

For enhanced security, you can enable Docker's user namespace remapping feature (`userns-remap`). This isolates container processes from the host by mapping container users to non-root users on the host.

**How to enable userns-remap:**

1. **Edit or create the Docker daemon config:**
   ```bash
   sudo mkdir -p /etc/docker
   sudo nvim /etc/docker/daemon.json
   ```
   Add:
   ```json
   {
     "userns-remap": "default"
   }
   ```

2. **Restart Docker:**
   ```bash
   sudo systemctl restart docker
   ```

Docker will now run containers with remapped user IDs, reducing the risk of privilege escalation from containers to the host.

> **Important:**  
> Enabling userns-remap will make all your existing Docker images and containers inaccessible. They will not be deleted, but Docker will not see them under the new user namespace. You can revert the change to regain access, or migrate images/containers as needed.

> This is optional, but highly recommended for internet-exposed servers. Some images may require adjustments to work with userns-remap.

**User Namespace Known Limitations:**

- The following Docker features are incompatible with user namespaces:
  - Sharing PID or NET namespaces with the host (`--pid=host` or `--network=host`)
  - External volume/storage drivers that do not support user mappings
  - Using `--privileged` mode without also specifying `--userns=host`

**Disabling Namespace Remapping for a Container:**

If user namespaces are enabled on the daemon, all containers use them by default. To disable user namespaces for a specific container (e.g., for privileged containers), add the `--userns=host` flag to your `docker run` or `docker create` command:

```bash
docker run --userns=host ...
```

> Note: The container filesystem will still be owned by the remapped user (e.g., 231072), which may cause issues for programs expecting root ownership (like `sudo` or setuid binaries).


> You can also check:  
> [Docker Rootless Mode (docs.docker.com)](https://docs.docker.com/engine/security/rootless/)  
> Running Docker in rootless mode is another way to improve security, especially in multi-user environments.

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

## Power Optimization with powertop & AutoASPM

To reduce power consumption, I use `powertop` and the `AutoASPM` script to automatically tune power settings.

### Powertop

```bash
sudo apt update
sudo apt install powertop
```

This command applies recommended power-saving settings:

```bash
sudo powertop --auto-tune
```

To run this automatically at startup, add it to root's crontab:

```bash
sudo crontab -e
```
Add this line at the end:
```bash
@reboot /usr/sbin/powertop --auto-tune
```

### AutoASPM 

This is a script that enables PCIe ASPM (Active State Power Management) for better power savings.  
See: [AutoASPM GitHub](https://github.com/notthebee/AutoASPM)

**Install and run AutoASPM:**

```bash
git clone https://github.com/notthebee/AutoASPM.git
cd AutoASPM
sudo python3 autoasmp.py
```

> **Note:**
> Always test power-saving settings for stability, especially on servers. Some devices may not support all ASPM states.
