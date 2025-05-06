+++
author = "brockar"
title = "Creating Private Tracker Torrents with mktorrent: A Complete Guide"
date = '2025-05-06'
categories = [
    "Linux"
]
tags = [
    "linux",
    "server",
    "networking",
    "trackers",
    "torrents"
]
image = "torrent-network.webp"
+++
# Guide to Creating Torrents with *mktorrent* for Private Trackers

In this guide, I'll show you how to use [*mktorrent*](https://github.com/pobrn/mktorrent) to create torrents compatible with private trackers.  
*mktorrent* is a lightweight and efficient command-line tool, ideal for those who prefer creating torrents without a graphical interface.

## 1. Installing mktorrent

To install *mktorrent*, open your terminal and use your preferred package manager:  
**On Debian/Ubuntu:**
```bash
sudo apt install mktorrent
```

## 2. Prerequisites

Before starting, make sure you have:
- The file or folder you want to share.
- The announce URL.
- Know what name to give the torrent.

## 3. Creating and Configuring the Torrent

The basic *mktorrent* command is:
```bash
mktorrent -a [announce] -d -l 22 -o [output_file].torrent [folder/file]
```

**Parameter explanation:**
- **-a**: Defines the private tracker's announce URL (required)
- **-d**: Disables DHT, mandatory for private trackers
- **-l**: Adjusts the piece size (file fragments) in powers of 2
- **-o**: Specifies the name of the .torrent file to be created
- **[folder/file]**: Path to the folder or file you want to share

**Important additional options:**

- **Piece size**: You can adjust the piece size (file fragments) with the **-l** parameter, using values in powers of 2.  
   Example: 
   ```bash
   mktorrent -a http://privatetracker.com:8080/announce -l 24 -o myfile.torrent /path/to/file/
   ```  
   The value *24*  represents 2^**24**, which is 16MB. Here are some common values:
   - 20 = 1 MB (0GiB - 10GiB)
   - 21 = 2 MB (10GiB - 30GiB)
   - 22 = 4 MB (30GiB - 75GiB)
   - 23 = 8 MB (75GiB - 200GiB)
   - 24 = 16 MB (+200GiB)

- **Adding comments**: You can add a comment to the torrent with the **-c** parameter.  
   Example: 
   ```bash
   mktorrent -a http://privatetracker.com:8080/announce -c "Uploaded by user on privatetracker.com" -o myfile.torrent /path/to/file/
   ```

## 4. Final Example

Now that you have all the options, you can create a complete torrent with this syntax:
```bash
mktorrent -a announce -d -l 22 -c "comment" -o myfile.torrent /path/to/file/
```

## 5. Uploading the Torrent to the Tracker

Finally, once you have the torrent file, upload it to the tracker following the instructions.  
Make sure to re-download the new .torrent from the tracker to seed the file properly.  
Once your upload is accepted, you'll be ready to start sharing the torrent.

**Happy uploading!**
