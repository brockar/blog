+++
author = "brockar"
title = "Free Obsidian Sync"
date = '2025-05-07'
lastmod = '2025-05-07'
description = "Sync your Obsidian vault across devices for free using Google Drive and rclone."
tags = [
    "linux",
    "obsidian",
    "debian",
    "arch",
    "windows",
    "android"
]
image = "obsidian-sync.webp"

[[links]]
title = "Obsidian"
website = "https://obsidian.md/"
image = "https://raw.githubusercontent.com/linuxserver/docker-templates/master/linuxserver.io/img/obsidian-logo.png"

[[links]]
title = "rclone doc"
website = "https://rclone.org/drive/"
image = "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcScG0gwCU3A_F8EXy8mUngwx6xaS_dzC4C_PA&s"

[[links]]
title = "Crontab easy way"
website = "https://crontab.guru/"
image = "https://dl.flathub.org/media/org/gnome/clocks/7f09a8f97b07c848cde84737f243be37/icons/128x128@2/org.gnome.clocks.png"

[[links]]
title = "DriveSync (Android)"
website = "https://play.google.com/store/apps/details?id=com.ttxapps.drivesync"
image = "https://play-lh.googleusercontent.com/o8JKQ48Oqkj6EUE7fr1d5jJcXvtJMhkYUa_A8ekBcIJQtRlH1FEmKSc8vr8ndXBtNQ=w480-h960"

[[links]]
title= "GDrive for Windows"
website= "https://workspace.google.com/products/drive/#download"
image= "https://storage.googleapis.com/gweb-workspace-assets/uploads/7uffzv9dk4sn-2bULAXbDcx21uhZ4nKipU6-e19ac531533e4a400edb1ca031724f1c-Drive_Product_Logo_2x.svg"
+++

{{< quote >}}
**Crucial:** For this synchronization method to work correctly, you **MUST** create a dedicated folder inside your Google Drive for your Obsidian vault. **DO NOT** attempt to sync your vault directly in the root folder of your Google Drive.
{{< /quote >}}

*Keep your Obsidian notes synchronized across all your devices without paying.*

If you're an Obsidian user looking for a free and reliable way to keep your vaults synchronized across multiple devices, this guide is for you. We'll walk through setting up automatic syncing using your Google Drive storage..

### Why This Method?

- 🆓 **Cost-Effective:** Completely free to use (within your Google Drive storage limits).
- 🔄 **Automatic Syncing:** Set it and forget it with scheduled syncs (e.g., every 15 minutes on Linux).
- 💻 **Cross-Platform:** Works on Linux, Windows, and Android.

---

## Linux Setup

This section details how to configure rclone and cron jobs for automatic synchronization on Linux systems.

### Prerequisites

- Obsidian installed.
- A Google account.
- rclone installed. You can install it via your package manager:
  - Debian/Ubuntu: `sudo apt install rclone`
  - Arch Linux: `sudo pacman -S rclone`

### Configure rclone with Google Drive

First, you need to authorize rclone to access your Google Drive. Open a terminal and run:

```bash
rclone config
```

Follow the interactive prompts:

1. Choose `n` for a new remote.
2. Enter a name for your remote (e.g., `gcloud`). Remember this name.
3. Select `20` (Google Drive) from the list of storage types.
4. Leave `client_id` and `client_secret` blank (press Enter for defaults).
5. Choose `1` for Full access (`drive`).
6. Leave `root_folder_id` blank unless you want to restrict rclone to a specific folder.
7. Leave `service_account_file` blank.
8. Say `N` to "Edit advanced config?".
9. Say `Y` to "Use auto config?". This will open a browser window for you to authorize rclone with your Google account.
10. Confirm the configuration and quit. `q`

### First sync

Create the folder to save our Obsidian Vault, I use `~/Documents/gcloud`.

```bash
mkdir ~/Documents/gcloud
```

If you already have your vault on GDrive, do the first clone.

```bash
rclone sync gcloud: /home/user/Documents/gcloud --transfer=40 --checkers=40
```

### Automatic Syncing with a Cron Job

We'll use cron to schedule regular syncs.

1. Open your user's crontab for editing:

    ```bash
    crontab -e
    ```

    (If prompted, choose an editor like `nvim`.)

2. Add the following lines to your crontab.  
**Replace `gcloud:` with your rclone remote name and the path to your Obsidian vault folder in Google Drive.**  
**Replace `/path/to/obsidian/vault` with the actual path to your Obsidian vault on your Linux machine.**

    ```cron
    # Sync from Google Drive to local vault on reboot (important for initial state)
    @reboot /usr/bin/rclone sync gcloud: /path/to/obsidian/vault --transfers=40 --checkers=40

    # Sync from local vault to Google Drive every 15 minutes
    */15 * * * * /usr/bin/rclone sync /path/to/obsidian/vault gcloud: --transfers=40 --checkers=40
    ```

**Explanation of Cron Job Lines:**

- `@reboot ...`: This line ensures that when your computer starts, your local vault is updated with the latest version from Google Drive.
- `*/15 * * * * ...`: This line syncs changes from your local vault up to Google Drive every 15 minutes.
- `/usr/bin/rclone`: Using the full path to rclone is good practice in cron jobs. Verify your rclone path with `which rclone`.
- `sync`: This command makes the destination match the source, deleting any files in the destination that are not in the source.
- `--transfers=40`: Number of file transfers to run in parallel.
- `--checkers=40`: Number of checkers to run in parallel (checks if files need transferring).

Now just open the vault from Obsidian and write!

---
**Important Notes for Linux:**

1. **First Sync:** The initial synchronization might take some time, depending on the size of your vault.
2. **Path Specificity:** Be very precise with your rclone remote name and paths.
3. **One-Way Sync Nature:** Each `rclone sync` command is one-way. The setup above creates a *pseudo* two-way sync. If you make changes on two devices simultaneously and both sync *up* before one syncs *down*, the last one to sync *up* will overwrite. For best results, ensure one device syncs *down* changes before you start editing on it.
4. **Conflict Management:** rclone's `sync` command will generally make the destination identical to the source. If a file was changed on both ends independently *between sync cycles*, the version from the *source* of the *last run sync command* will prevail for that file in the *destination*.

---

## Windows Setup

For Windows, the official Google Drive for desktop client provides an easy way to achieve synchronization by mirroring your files.

### Install Google Drive for Desktop

1. Download the official client from Google:
    [https://www.google.com/drive/download/](https://www.google.com/drive/download/)
2. Run the installer and follow the on-screen prompts (typically "next, next, finish").
3. Sign in with your Google account when prompted.

### Configure Google Drive for Mirroring

1. Once installed and signed in, click the Google Drive icon in your system tray.
2. Click the ⚙️ **Settings** icon, then select **Preferences**.
3. In the Preferences window, on the left sidebar, click **Google Drive**.
4. Choose the **"Mirror files"** option. This will store all your Google Drive files on your computer and keep them synced with the cloud.
    - Note the **"Folder location"** displayed (e.g., `C:\Users\[YourUsername]\My Drive`). This is where your mirrored Google Drive files will be stored locally. You can change this location if desired.

### Configure Obsidian

1. Open Obsidian.
2. Choose "Open folder as vault."
3. Navigate to the local Google Drive mirrored folder (e.g., `C:\Users\[YourUsername]\My Drive`) and then into the specific folder where your Obsidian notes are stored (e.g., `My Drive\ObsidianVault`). Select this folder.

Now, any changes you make in your Obsidian vault will be saved locally within the mirrored Google Drive folder. The Google Drive for Desktop client will automatically detect these changes and sync them to the cloud, and vice-versa.

---

## Android Setup

On Android, an app like DriveSync is excellent for syncing specific folders with Google Drive with full mirror sync..

### Install DriveSync

1. Download **DriveSync** (free version) from the [Google Play Store](https://play.google.com/store/apps/details?id=com.ttxapps.drivesync).
2. Launch DriveSync and grant the necessary permissions (e.g., for storage access).

### Connect Your Google Account

1. In DriveSync, tap "Connect to Google Drive" or find the option in the menu (often under "Accounts").
2. Select your Google account and follow the on-screen prompts to authorize DriveSync.

### Create a Synced Folder Pair

This tells DriveSync which folder in your Google Drive to sync with which folder on your Android device.

1. In DriveSync, go to the "Synced folders" tab.
2. Tap the `+` (Add) button to create a new folder pair.
3. Configure the folder pair:
    - **Name (Optional):** Give this sync pair a descriptive name (e.g., "Obsidian Vault").
    - **Remote folder in Google Drive:** Tap to select the folder within your Google Drive where your Obsidian vault is stored (the one you created as per the initial warning).
    - **Local folder in this device:** Tap to create or select a folder on your Android device where you want the vault to be stored locally (e.g., `/storage/emulated/0/Documents/gcloud`).
    - **Sync method:** Choose **"Two-way"**.
    - **Exclude hidden files:** **UNCHECK** or **DISABLE** this option. Obsidian uses a hidden `.obsidian` folder for its configuration, which you'll likely want to sync.
    - **Deletion:** Consider enabling "Delete permanently, skip Google Drive trash can".

### Configure Sync Settings

Adjust these settings according to your preference, found in DriveSync's main menu ➔ Settings:

| Setting                         | Recommendation                           | Notes                                                             |
| :------------------------------ | :--------------------------------------- | :---------------------------------------------------------------- |
| **Autosync interval**           | 1 hour (Free) / 15 minutes or less (Pro) | More frequent syncs reduce chances of conflicts.                  |
| **Autosync only on WiFi**       | ✅ Recommended                            | Saves mobile data.                                                |
| **Autosync only when charging** | Optional                                 | Saves battery; useful if your vault is very large or syncs often. |
| **Conflict resolution**         | "Newer file wins" (default) or "Ask me"  | "Newer file wins" is usually fine. "Ask me" gives more control.   |

Once configured, DriveSync will keep your specified Android folder and Google Drive folder in sync. You can then open the local folder as a vault in the Obsidian Android app.

Best Practice for Syncing Changes:  

- Before Making Changes on Android:
  - Always sync your files first to ensure you're working with the latest version.

- After Making Changes on Linux/Windows:
  - If you modify files on one platform (e.g., Linux/Windows) and plan to open them on another within an hour, make sure to sync again to avoid conflicts.

---

## Alternatives

While the rclone and Google Drive client methods are robust, here are a few other ways to sync Obsidian:

- [Syncthing](https://syncthing.net/): A free, open-source, peer-to-peer file synchronization application. Excellent for direct device-to-device sync without a central cloud server (though it requires devices to be online simultaneously or a relay).
- **Git (with a service like GitHub/GitLab):** More technical, but offers version control. Can be combined with tools like `obsidian-git` plugin.
- **Obsidian Sync (Paid):** The official, end-to-end encrypted, paid service from the Obsidian developers. It's the most seamless and specifically designed for Obsidian.

I haven't personally dived deep into these alternatives for my daily Obsidian use, as the Google Drive methods described above have worked perfectly for my needs.

Good luck keeping your notes in sync!
