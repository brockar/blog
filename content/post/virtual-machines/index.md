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
    "vm"
]
image = "qemuvirt.png"
+++


## Sharing Folders Between Host and Guest Systems


### For QEMU/KVM Virtual Machines

If you're using QEMU with SPICE protocol for better integration:

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
[YouTube: Working with Physical Disks in VMs](https://www.youtube.com/watch?v=vYQ9Bkv7VG4)

- Physical disk passthrough requires proper permissions and can be risky - always back up data first.

## VMware Virtual Machines

If you're using VMware and need to set up shared folders between your host and guest systems:

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

### Notes

- Remember to replace `$USER` with your actual username or use the environment variable as shown
- For VMware, you may need to enable folder sharing in the VM settings first
