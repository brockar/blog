
+++
author = "brockar"
title = "Hibernation on Linux"
date = '2025-09-05'
lastmod = '2025-09-05'
description = "My way to setup hibernation on Linux with zram and swapfile."
categories = [
    "linux",
    "arch"
]
tags = [
    "linux",
    "configuration",
    "arch"
]
image = "image.png"

[[links]]
title = "Arch Wiki - Hibernation"
website = "<https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate>"
image = "<https://archlinux.org/static/logos/archlinux-logo-light-90dpi.png>"
+++

---

In this post, I'll show how to setup hibernation on Linux (using Arch as example), with zram and swapfile together.  
Why combine them?  

- **zram**: great for everyday use → faster, avoids writing to disk. (comes with the default Arch install)
- **swapfile**: required for hibernation, since you need a persistent swap that survives power-off.

## What is Hibernation

Hibernation saves the entire contents of *RAM* to the *swap area* on disk and **then powers the system off completely**. When you turn the laptop back on, the kernel reloads that memory image, restoring your session exactly where you left it. **It uses no power while off**, but takes a bit longer to resume compared to sleep.

### Diff between Sleep and Hibernation

- Sleep (Suspend to RAM):
  - Keeps data in RAM with low **power consumption**. The system wakes up quickly, **but will lose its state if power is lost**.

- Hibernation (Suspend to Disk):
  - Stores RAM contents to disk and fully powers down. Consumes no power while off, and the session can survive battery drain or shutdown, but resuming is slower.

### Why Hibernation doesn't destroy your SSD/NVME

Hibernate does write a lot at once (it dumps all RAM to swap), but it’s not continuous.  
If you hibernate a few times per day, your drive’s wear is negligible compared to its rated endurance. A consumer NVMe often has 300–600 TBW.  
Even if you had 16 GiB RAM and hibernated every single hour, you’d write ~384 GiB per day → ~140 TB per year. That’s still within warranty for years — and in practice, you won’t hibernate that often.

## Setup Hibernation on Linux

> **Note:** I use `rg` (ripgrep) as a faster alternative to `grep` in my commands. If you don't have `rg` installed, you can substitute `grep` for similar results.

### Check kernel support

Run the following command:

```bash
cat /sys/power/state
```

and check if `disk` is listed.

### Check Swap

Hibernation needs enough swap space to store the content of your RAM.
Check your swap size with:

```bash
swapon --show
free -h
```

#### Swapfile vs Swap Partition

There is two ways to create swap: a swapfile or a dedicated swap partition.  
Swapfile:

- Easy to set up, flexible size.
- Perfectly fine on NVMe (use fallocate or dd).  

Swap partition:

- Slightly lower overhead.
- Required only if you want maximum performance or use encrypted setups with special constraints.
- Harder to resize.

For hibernation, both work equally well (as long as you configure resume and resume_offset properly).  

If you hibernate rarely (say, once or twice a day, or just occasionally): Swapfile on NVMe is totally fine.  
If you plan to hibernate many times per hour and leave the laptop running for years → maybe consider a partition. But honestly, that’s overkill.  

## Setup swapfile

Decide swapfile size  
You have 15 GiB RAM, so make the swapfile at least 16 GiB.  
More is okay (RAM size + 10–20% is safe).  

1. **Create the swapfile**:

```bash
sudo fallocate -l 16G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
```

2. **Check**:

```bash
ls -lh /swapfile
```

Should show ~16G or your chosen size  

3. **Enable the swapfile**:

```bash
sudo swapon /swapfile
```

4. **Make it permanent**:

```bash
echo '/swapfile none swap defaults 0 0' | sudo tee -a /etc/fstab
```

5. **Verify**:

```bash
swapon --show
free -h
```

## Find resume offset

For swapfile, you need to find the resume offset (the block where the swapfile starts on disk).
Run:

```bash
sudo filefrag -v /swapfile | rg " 0:"
```

You should see output like:

```text
0: 0.. 0: 16601088.. 16601088: 1:
```

E.g., `16601088` is your offset.

## Update kernel parameters

I'll do with GRUB, but if you use another bootloader, adapt accordingly.

First check your root device:

```bash
findmnt -no SOURCE /
```

It might show something like `/dev/nvme0n1p2` or `/dev/sda2`.

Edit your bootloader `/etc/default/grub` and add the following line to the `GRUB_CMDLINE_LINUX_DEFAULT`:  
**With your root device and the found resume offset.**

```bash
resume=/dev/nvme0n1p2 resume_offset=16601088
```

Example (append to your existing line):

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash resume=/dev/nvme0n1p2 resume_offset=16601088"
```

Rebuild the bootloader config:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### Add resume hook

Edit `/etc/mkinitcpio.conf` → in `HOOKS=()`, add `resume` before `filesystems`.

Example:

```md
HOOKS=(base udev autodetect modconf block resume filesystems keyboard fsck)
```

Rebuild initramfs:

```bash
sudo mkinitcpio -P
```

## Test hibernation

Make sure to save all work before testing.
Run:

```bash
sudo systemctl hibernate
```

On reboot look for messages about resuming from swap.

```bash
sudo dmesg | rg -i hibernation
```

Example output:

```text
Sep 02 17:24:07 archlaptop kernel: PM: hibernation: Registered nosave memory: [mem 0x00000000-0x00000fff] 
Sep 02 17:24:07 archlaptop kernel: PM: hibernation: Registered nosave memory: [mem 0x0009f000-0x000fffff] 
```

## Troubleshooting

If hibernation doesn't work:

- Ensure swap is at least as large as your RAM.
- Double-check the resume device and offset in GRUB.
- Run `journalctl -b` after a failed boot to see errors.
- For more help, check the [Arch Wiki on Hibernation](https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate).

That's it! Now you can hibernate your Linux system confidently.
