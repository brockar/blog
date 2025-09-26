+++
author = "brockar"
title = "Using Your Phone as a Webcam on Linux"
date = '2025-09-26'
lastmod = '2025-09-26'
description = "Turn your phone into a webcam on Linux using IP Webcam and v4l2loopback - perfect for video calls and streaming."
categories = [
    "Linux",
    "Android"
]
tags = [
    "linux",
    "webcam",
    "android",
]
image = "own.jpg"

[[links]]
title = "IP Webcam - Google Play"
website = "https://play.google.com/store/apps/details?id=com.pas.webcam&pcampaignid=web_share"
image = "https://play-lh.googleusercontent.com/qn6LAGjRbz_8T92SCJJ28vrRUmh6FsvTV6-ZHFenWcxx86Mtgq74r6iKPOig8syTSDA=w240-h480"

[[links]]
title = "v4l2loopback - Arch Wiki"
website = "https://wiki.archlinux.org/title/V4l2loopback"
image = "https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Ftse1.mm.bing.net%2Fth%2Fid%2FOIP.yh48v_vbOPrS-TK5uDgnqAHaHa%3Fpid%3DApi&f=1&ipt=b9cbafbe331d4cf852602097572a1a4ae4824e8694599214ca962b026f49c4d4"

+++

Need a webcam for your Linux system? This guide shows you how to use your phone as a webcam using IP Webcam and v4l2loopback.

## Prerequisites

- Android phone
- Linux system (examples use Arch Linux)
- Both devices connected to the same network

## Download and Set Up IP Webcam

1. Install [**IP Webcam**](https://play.google.com/store/apps/details?id=com.pas.webcam&pcampaignid=web_share) from the Google Play Store.  
2. Open the app and scroll down to find **Start server**  
3. Tap **Start server** - the app will display a URL like:  

   ```
   http://192.168.100.6:8080
   ```

4. Note this IP address for later use

## Install Required Packages

Install the necessary dependencies on your Linux system:

```bash
sudo pacman -S --needed v4l2loopback-dkms \
    gstreamer gst-plugins-base gst-plugins-good gst-plugins-bad \
    gst-plugins-ugly gst-libav
```

## Create Virtual Camera Device

Load the v4l2loopback module to create a virtual webcam:

```bash
sudo modprobe v4l2loopback exclusive_caps=1 video_nr=10 card_label="IPCam"
```

Verify the device was created:

```bash
ls -l /dev/video10
```

Add your user to the video group for proper permissions:

```bash
sudo usermod -aG video $USER
```

**Important:** Log out and log back in for the group changes to take effect.

## Test the Connection

Verify your phone's video stream is accessible:

```bash
xdg-open http://192.168.100.6:8080/videofeed
```

Replace `192.168.100.6` with your phone's IP address. You should see live video in your browser.

## Stream to Virtual Camera

Choose one of the following methods based on your preference:

### Method A: MJPEG Stream (More Compatible)

```bash
gst-launch-1.0 souphttpsrc is-live=true do-timestamp=true \
  location=http://192.168.100.6:8080/videofeed ! \
  multipartdemux ! jpegdec ! videoconvert ! v4l2sink device=/dev/video10
```

### Method B: H.264 Stream (Better Quality)

```bash
gst-launch-1.0 souphttpsrc is-live=true do-timestamp=true \
  location=http://192.168.100.6:8080/video ! \
  decodebin ! videoconvert ! v4l2sink device=/dev/video10
```

Keep this command running while you want to use your phone as a webcam.

## Use in Applications

Your phone camera will now appear as **IPCam** in video applications like:

- Zoom
- Google Meet
- OBS Studio
- Discord
- Any application that supports V4L2 devices

## Bonus: Automation Script

Create a convenient script to automate the process. Save this as `~/.local/bin/ipcam`:

```bash
#!/bin/bash
# IP Webcam to Virtual Camera Script

PHONE_IP="192.168.100.6"  # Change to your phone's IP
PORT="8080"
DEVICE="/dev/video10"

# Load v4l2loopback module if device doesn't exist
if ! ls $DEVICE &>/dev/null; then
    echo "Loading v4l2loopback module..."
    sudo modprobe v4l2loopback exclusive_caps=1 video_nr=10 card_label="IPCam"
fi

# Check if device exists
if ! ls $DEVICE &>/dev/null; then
    echo "Error: Could not create virtual camera device"
    exit 1
fi

echo "Starting IP Webcam stream..."
echo "Phone IP: $PHONE_IP:$PORT"
echo "Virtual device: $DEVICE"
echo "Press Ctrl+C to stop"

# Stream configuration
STREAM="/videofeed"                   # Use "/video" for H.264
PIPE="multipartdemux ! jpegdec"       # Use "decodebin" for H.264

gst-launch-1.0 souphttpsrc is-live=true do-timestamp=true \
  location="http://$PHONE_IP:$PORT$STREAM" ! \
  $PIPE ! videoconvert ! v4l2sink device=$DEVICE
```

Make it executable and run:

```bash
chmod +x ~/.local/bin/ipcam
ipcam
```

## Troubleshooting

**Stream not working?**

- Ensure both devices are on the same network
- Verify the IP address in the URL
- Try another decoding method.

**Permission denied on `/dev/video10`?**

- Make sure you're in the video group: `groups $USER`
- Log out and back in after adding to the group

**Poor video quality?**

- Try the H.264 stream (`/video` endpoint)
- Adjust video settings in the IP Webcam app
- Check network bandwidth

## Conclusion

You now have a high-quality webcam using your Android phone. This setup provides better video quality than some built-in laptop cameras and gives you the flexibility to position your camera anywhere within your network range.

Also, if you don't have a webcam on your laptop/PC, this is a great alternative. Enjoy your new webcam setup!

For more details, please refer to the links below.
