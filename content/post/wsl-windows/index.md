+++
author = "brockar"
title = "WSL: My config"
description = "How I configure my WSL when running on Windows."
date = '2025-05-07'
categories = [
    "Linux"
]
tags = [
    "linux",
    "docker",
    "wls",
]
image = "wsl.jpg"
+++

## Install WSL

Run **Powershell** as admin and run this command:

```bash
wsl --install
wsl --install -d Debian
```

Or, you can search for "Debian" in the Microsoft Store and click "Get" to install.

## Install Docker

```bash
sudo apt update && sudo apt upgrade
sudo apt remove docker docker-engine docker.io containerd runc
sudo apt install --no-instport-https ca-certificates curl gnupg2

update-alternatives --config iptables # And select iptables-legacy

. /etc/os-release
curl -fsSL https://download.docker.com/linux/${ID}/gpg | sudo tee /etc/apt/trusted.gpg.d/docker.asc

echo "deb [arch=amd64] https://download.docker.com/linux/${ID} ${VERSION_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io

sudo usermod -aG docker $USER
```

Reference: [Install Docker on Windows WSL without Docker Desktop](https://dev.to/bowmanjd/install-docker-on-windows-wsl-without-docker-desktop-34m9)

## AutoStart Docker

Run this to start Docker daemon automatically when logging in if not running

```bash
echo '# Start Docker daemon automatically when logging in if not running.' >> ~/.bashrc
echo 'RUNNING=`ps aux | grep dockerd | grep -v grep`' >> ~/.bashrc
echo 'if [ -z "$RUNNING" ]; then' >> ~/.bashrc
echo ' sudo dockerd > /dev/null 2>&1 &' >> ~/.bashrc
echo ' disown' >> ~/.bashrc
echo 'fi' >> ~/.bashrc
```

Also create or edit `/etc/wsl.conf` with:

```ini
[boot]
command= service docker start
```

## WSL Port Forwarding

First install net-tools:

```bash
sudo apt install net-tools
```

### Mode 1

In Windows:

```powershell
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=8080 connectaddress=(wsl hostname -I).split(" ")[0]
```

Create `rtorrent.ps1`:

```powershell
$wsl_ip = (wsl hostname -I).split(" ")[0]
Write-Host "WSL Machine IP: ""$wsl_ip"""
netsh interface portproxy add v4tov4 listenport=8080 connectport=8080 connectaddress=$wsl_ip
```

Then in Admin PowerShell:

```powershell
$trigger = New-JobTrigger -AtStartup -RandomDelay 00:00:15
Register-ScheduledJob -Trigger $trigger -FilePath C:\Users\marti\rtorrent.ps1 -Name RouteRTorrent
```

### Mode 2

Create `network.ps1`:

```powershell
If (-NOT ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")) {
  $arguments = "& '" + $myinvocation.mycommand.definition + "'"
  Start-Process powershell -Verb runAs -ArgumentList $arguments
  Break
}

$remoteport = bash.exe -c "ifconfig eth0 | grep 'inet '"
$found = $remoteport -match '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}';

if ($found) {
  $remoteport = $matches[0];
}
else {
  Write-Output "IP address could not be found";
  exit;
}

$ports = @(80,9696,7878,5055,8989,8081,8191);

for ($i = 0; $i -lt $ports.length; $i++) {
  $port = $ports[$i];
  Invoke-Expression "netsh interface portproxy delete v4tov4 listenport=$port";
  Invoke-Expression "netsh advfirewall firewall delete rule name=$port";

  Invoke-Expression "netsh interface portproxy add v4tov4 listenport=$port connectport=$port connectaddress=$remoteport";
  Invoke-Expression "netsh advfirewall firewall add rule name=$port dir=in action=allow protocol=TCP localport=$port";
}

Invoke-Expression "netsh interface portproxy show v4tov4";
```

Run the script with Admin PowerShell:

```powershell
./network.ps1
```

## Autostart WSL

1. Press Win + R and type `shell:common startup`
2. Create a file.vbs with:

```vbs
set ws=wscript.CreateObject("wscript.shell")
ws.run "wsl -d Debian", 0
```

(Replace "Debian" with your distro name)

Alternatively, create a `deb.bat` file with:

```bat
wsl.exe -d Debian
```

## SSH Configuration

In Admin PowerShell:

```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
Get-Service ssh-agent
```
