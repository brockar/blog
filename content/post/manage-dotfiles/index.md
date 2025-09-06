+++
author = "brockar"
title = "Dotfiles: Stow"
description = "My way to manage my dotfiles."
date = '2025-05-07'
lastmod = '2025-05-07'
categories = [
    "Linux"
]
tags = [
    "linux", "git", "github", "debian", "arch", "stow", "configuration"
]
image = "dotfiles.jpg"

[[links]]
title = "Stow GNU"
website = "https://www.gnu.org/software/stow/"
image = "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ4_q9sDAguJCohzBd3c09ZwyJtf5HGFOCNqQ&s"
+++

# Managing Dotfiles with GNU Stow: Setup and Migration

Dotfiles – those crucial hidden configuration files that define your Linux environment – find their perfect management companion in GNU Stow, offering elegant symlink organization and effortless system migration through a simple setup process.

**Why Stow?**

- Automatic symlink creation
- Conflict resolution capabilities
- Package-based organization
- Cross-distribution compatibility

## Getting Started with Stow

### Installation

First, install Stow on your system:

```bash
# Debian/Ubuntu
sudo apt install stow -y

# Arch Linux (yay AUR helper)
yay -S stow --noconfirm
```

### Initial Setup

1. Create a dedicated directory for your dotfiles (typically in your home directory) and initialize a Git repository:

```bash
mkdir ~/dotfiles
cd ~/dotfiles
git init
```

2. Organize your configuration files by application. For each program, create a folder that mimics the directory structure where the config files belong.

For example, for Neovim:

```bash
mkdir -p nvim/.config/nvim
mv ~/.config/nvim/init.lua nvim/.config/nvim/
```

3. Create a basic .gitignore file to exclude unnecessary files:

```bash
echo "*~" >> .gitignore
echo "*.swp" >> .gitignore
```

4. Commit your initial configuration:

```bash
git add .
git commit -m "Initial dotfiles commit"
```

5. To symlink your files to their proper locations:

```bash
stow . --adopt
git restore .
```

The `--adopt` flag tells Stow to adopt existing files, and `git restore .` helps clean up any conflicts if you're using Git.

6. (Optional) Push to a remote repository:

```bash
git remote add origin https://github.com/yourusername/dotfiles.git
git push -u origin main
```

### Handling Potential Conflicts

If files already exist in the target locations, you have several options:

1. **Overwrite carefully** (after backing up):

```bash
stow --override=* .
```

2. **Adopt and compare changes**:

```bash
stow --adopt .
# Then review differences before committing
```

## Advanced Configuration

### The .stowignore File

GNU Stow's `.stowignore` file works similarly to `.gitignore` - it specifies files or patterns that Stow should ignore when creating symlinks.

#### Key Features

- **Pattern matching**: Supports wildcards (`*`) and basic regex
- **Line-based**: Each pattern goes on a separate line
- **Location-specific**: Place in your package directory (e.g., `~/dotfiles/nvim/.stowignore`)

#### Common Use Cases

```bash
# Ignore temporary files
*.tmp
*.swp

# Ignore version control directories
.git/
.vscode/

# Ignore specific config files
local_settings.*
```

## Restoring Dotfiles on a New System

One of the biggest advantages of this system is how easy it makes migrating to a new machine:

1. Clone or copy your dotfiles repository to the new machine:

```bash
git clone https://github.com/yourusername/dotfiles.git ~/dotfiles
```

2. Install Stow on the new system (as shown in the installation section above).

3. Navigate to your dotfiles directory and deploy all configurations:

```bash
cd ~/dotfiles
stow .
```

With this setup, you'll never have to manually reconfigure your environment again.

**Just clone and stow!**
