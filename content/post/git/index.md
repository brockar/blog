+++
author = "brockar"
title = "Git"
description = "My way to use git and github."
date = '2025-05-07'
categories = [
    "Linux"
]
tags = [
    "linux", "git", "github", "ssh"
]
image = "lg.jpg"

[[links]]
title = "Lazygit"
website = "https://github.com/jesseduffield/lazygit"
image = "https://user-images.githubusercontent.com/8456633/174470852-339b5011-5800-4bb9-a628-ff230aa8cd4e.png"
+++

## Basic Git Configuration

Set up your global Git configuration:

```bash
git config --global user.name "Your name here"
git config --global user.email "your_email@example.com"
git config --global color.ui true
```

## SSH Key Setup for GitHub

1. Generate a new SSH key (recommended to use Ed25519 algorithm):
   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

2. Add your public key to GitHub:
   - Copy the public key to clipboard:
     ```bash
     xclip < ~/.ssh/id_ed25519.pub
     ```
   - Or display it in terminal:
     ```bash
     cat ~/.ssh/id_ed25519.pub
     ```
   - Paste it in GitHub Settings → SSH and GPG keys → New SSH key

3. Test your SSH connection:
   ```bash
   ssh -T git@github.com
   ```

## SSH Agent Setup

To avoid repeatedly entering your SSH passphrase:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

## Lazygit - Terminal UI for Git

[Lazygit](https://github.com/jesseduffield/lazygit) is a simple terminal UI for Git commands that makes version control more visual and interactive.

### Installation

#### Linux (Debian/Ubuntu):
For Debian 13 (Trixie) or Ubuntu 25.10:
```bash
sudo apt install lazygit
```

For Debian 12 or Ubuntu 25.04:
```bash
LAZYGIT_VERSION=$(curl -s "https://api.github.com/repos/jesseduffield/lazygit/releases/latest" | \grep -Po '"tag_name": *"v\K[^"]*')
curl -Lo lazygit.tar.gz "https://github.com/jesseduffield/lazygit/releases/download/v${LAZYGIT_VERSION}/lazygit_${LAZYGIT_VERSION}_Linux_x86_64.tar.gz"
tar xf lazygit.tar.gz lazygit
sudo install lazygit -D -t /usr/local/bin/
```

![Lazygit Screenshot](/lg_tp.png)

### Basic Lazygit Commands

| Key | Action                |
| --- | --------------------- |
| `x` | Show menu of actions  |
| `P` | Push changes          |
| `p` | Pull changes          |
| `c` | Commit changes        |
| `s` | Stage/unstage files   |
| `b` | View branches         |
| `m` | Merge branches        |
| `d` | View diff             |
| `?` | Show keybindings help |

### Why Use Lazygit?

1. Visual branch management
2. Easy staging/unstaging of changes
3. Built-in diff viewer
4. Interactive rebase capabilities
5. Reduced need to remember Git commands
6. Works entirely in terminal (great for remote servers)

### Custom Configuration

Create a config file at `~/.config/lazygit/config.yml` to customize keybindings and appearance.

## Common Git Workflow with Lazygit

1. Open lazygit in your repo:
   ```bash
   lazygit
   ```
2. Stage changes with `s`
3. Commit with `c`
4. Push with `P`
5. Create/switch branches with `b`
6. Merge with `m`