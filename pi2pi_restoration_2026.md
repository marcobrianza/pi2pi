# pi2pi
A self-contained network between two Raspberry Pis  
**Restoration: March 2026**

---

## Overview

This installation consists of two Raspberry Pis:

- **rpa** → sender / receiver
- **rpb** → sender / receiver

Each device:
- runs a script (`imagesync.sh`)
- copies files from a USB drive (`/sending`)
- syncs them to the other device (`/receiving`) using `rsync` over SSH

The system is fully autonomous:
- no internet required after setup
- no DHCP server
- no DNS / `.local` resolution

Communication happens via **static IP addresses over Ethernet**.

This system is typically used in the artwork  
**"Personal Photographs" by Eva and Franco Mattes**.

---

## Network Configuration

- rpa → 10.0.0.1  
- rpb → 10.0.0.49  

---

## Preparing USB Drives

- Format: FAT32  
- Label: **must be the same on both drives (for example: USB or PHOTOS)**  
- Folders:

```
/sending
/receiving
```

### Behavior

- sending (A) → receiving (B)  
- sending (B) → receiving (A)  

---

## Script Creation

Create the script:

```
nano /home/pi/imagesync.sh
```

Paste:

```bash
#!/bin/sh
echo "Host: ""$(hostname)"
echo "Beginning photo sync..."

if [ "$(hostname)" = "rpa" ]; then
  recipient_host="10.0.0.49"
  echo "Sending photos to "$recipient_host"..."
else
  recipient_host="10.0.0.1"
  echo "Sending photos to "$recipient_host"..."
fi

usb_drive_name="$(ls /media/pi/ | head -n 1)"

echo "Using USB drive:" $usb_drive_name

sender_path=/media/pi/$usb_drive_name/sending
recipient_path=/media/pi/$usb_drive_name/receiving

rsync --progress -ab --recursive --ignore-times $sender_path/. pi@$recipient_host:$recipient_path

echo "Fin."
```

Save:

```
CTRL + O
ENTER
CTRL + X
```

Make executable:

```
chmod +x /home/pi/imagesync.sh
```

---

## Test

```
/home/pi/imagesync.sh
```

---

## Cron

```
crontab -e
```

Add:

```
* * * * * /home/pi/imagesync.sh
```

---

## Disk Imaging / Backup

See:

[sd-card-copy-macos_en.md](sd-card-copy-macos_en.md)

⚠️ Both USB drives MUST have the same label (e.g. "USB", "PHOTOS", etc.).

---

## Summary

- No DHCP  
- Static IP  
- Simple rsync system  
- Bidirectional sync  
