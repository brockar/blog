+++
author = "brockar"
title = "Virtual Machines: Notes"
description = "My notes to run VMs"
date = '2025-05-06'
categories = [
    "Linux"
]
tags = [
    "linux",
    "networking",
    "vitual-machines",
    "vm",
    "windows"
]
image = "qemuvirt.png"
+++


This post documents my setup and configuration notes for running virtual machines
with QEMU and VirtManager on Linux. It covers:

- Shared folder configuration.
- Physical disk passthrough.
- Bridge networking setup.
- Windows guest optimization.

## Sharing Folders Between Host and Guest Systems

If you're using QEMU with SPICE protocol for better integration with linux guest:

```bash
sudo apt install open-vm-tools spice-vdagent
```

This provides:

- Clipboard sharing
- Better display resizing
- Mouse pointer integration

## Working With Physical Disks in Virtual Machines

For advanced users who need to work with physical disks in their VMs:

1. First identify your disks:

   ```bash
   sudo fdisk -l
   ```

2. Create physical volume (LVM):

   ```bash
   sudo pvcreate /dev/sdXX
   ```

   (Replace XX with your actual disk identifier)

For a complete tutorial on this process, see:
[Working with Physical Disks in VMs](https://www.youtube.com/watch?v=vYQ9Bkv7VG4)

- Physical disk passthrough requires proper permissions and can be risky - always
back up data first.

## Bridge network for QEMU/VirtManager

For create a Bridge network to share with the VM I do this on host:

```bash
sudo pacman -S bridge-utils dnsmasq
nmcli connection add type bridge ifname br0 con-name br0
nmcli connection add type bridge-slave ifname eno1 master br0
sudo nmcli connection modify br0 bridge.stp no
nmcli connection up br0
sudo iptables -I FORWARD -m physdev --physdev-is-bridged -j ACCEPT
```

(Replace `eno1` with your actual physical interface name - find it with `ip link`)

Add `allow br0` on `/etc/qemu/bridge.conf`

## VMware Virtual Machines

## Windows guest

When I install Windows VM on my Linux Host, I do this:

### Prerequisites

Before install Windows, you need to download VirtIO drivers, for use VirtIO disks
and network.  

- Go to [VirtIO](https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers#Using_the_ISO) drivers page and download the ISO.  
- Create a new VM on VirtManager and mount the ISO as **CD-ROM** with SATA protocol.  
- Create a new disk for the VM as VirtIO disk.  
- Mount Win ISO to install.  
- For network use Bridge `br0` as source and `virtio` as device model.  
- On `Display Spice` select **Type=Spice Server** and **Listen type=None**.  

### Installation

- Run the VM with Win ISO as first boot.  
- Next, next, next.  
- When u have to select the disk to install, nothing will appear, select `Load Drivers` and select the drivers on `disk/viostor/w1x/amd64` and Next.  
  - W1x will be W10 or W11.  
- Also can load the driver for the network on `disk/NetKVM/W1x/amd64` and Next.  

Install Windows as usual.  

After Windows install, go to the CD Drive and install
`virtio-win-gt-x64` and `virtio-win-guest-tools` drivers.

### Share folder

#### On Host

Create the Shared Folder

- On your Linux host, create a folder you wish to share (e.g., ~/share).

Open Virt-Manager and Edit the VM

- Select your Windows VM and click "Open".
- Click the "Show virtual hardware details" button.
- Select the "Memory" section.
- Check the "Enable shared memory" option and apply the changes

Add the Shared Folder:

- Click "Add Hardware".
- Select "Filesystem".
- Set the driver to `virtiofs`.
- Set the "Source path" to the folder you want to share (e.g., `/home/youruser/share`).
- Set the "Target path" to a descriptive name (e.g., `HostShare`). This will
appear as the share name in Windows

#### On Guest

Install [WINFSP](https://winfsp.dev/rel/) driver.  
Restart.  
Open `Services` app.
Find `WINFSP` and start it. Also set it to "Automatic".  

The folder will appear as `HostShare` on "This PC".

## Extra: VMware (Windows host)

If you're using VMware and need to set up shared folders between your
host and Linux guest systems:

1. First install VMware tools:

   ```bash
   sudo apt install open-vm-tools
   ```

2. Create a directory for your shared folders:

   ```bash
   mkdir /home/$USER/shares
   ```

3. Mount the shared folder using:

   ```bash
   vmhgfs-fuse .host:/ /home/$USER/shares -o subtype=vmhgfs-fuse,allow_other
   ```

The `allow_other` option allows all users to access the shared folder.
