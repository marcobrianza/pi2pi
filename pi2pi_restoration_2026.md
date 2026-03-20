# pi2pi
A self-contained network between two Raspberry Pis  
**Restoration: March 2026**

---

## Overview

Two Raspberry Pis exchange files bidirectionally via USB drives using rsync over SSH.

- rpa ↔ rpb
- static IP
- no DHCP
- autonomous system

---

## Operating System

Download latest Raspberry Pi OS (32-bit):

https://downloads.raspberrypi.com/raspios_armhf/images/raspios_armhf-2025-12-04/2025-12-04-raspios-trixie-armhf.img.xz

Legacy reference:

https://downloads.raspberrypi.com/raspbian/images/raspbian-2018-11-15/2018-11-13-raspbian-stretch.zip

---

## Flashing SD Cards

Flash image using:

Balena Etcher 2.1.4

(minimum 8GB SD card, tested with 16GB)

---

## First Boot and Initial Setup

1. Insert SD card and boot Raspberry Pi
2. Connect to internet (Ethernet recommended)

Configure:

- Country: USA  
- Language: English  
- Keyboard: US  
- Location: New York  

Create user:

- username: pi  
- password: pi  

Then update system:

```
sudo apt update && sudo apt upgrade -y
sudo reboot
```

---

## Hostname Configuration

Run:

```
sudo raspi-config
```

Set:

- rpa → hostname: rpa  
- rpb → hostname: rpb  

Enable SSH:

- Interface Options → SSH → Enable

Reboot if required.

---

## Network Configuration (Static IP)

Check connection name:

```
nmcli connection show
```

### rpa

```
sudo nmcli con mod "Wired connection 1" ipv4.addresses 10.0.0.1/24 ipv4.method manual
sudo nmcli con down "Wired connection 1"
sudo nmcli con up "Wired connection 1"
sudo reboot
```

### rpb

```
sudo nmcli con mod "Wired connection 1" ipv4.addresses 10.0.0.49/24 ipv4.method manual
sudo nmcli con down "Wired connection 1"
sudo nmcli con up "Wired connection 1"
sudo reboot
```

---

## SSH Keys (Passwordless)

### rpa

```
ssh-keygen -t rsa
ssh-copy-id pi@10.0.0.49
```

### rpb

```
ssh-keygen -t rsa
ssh-copy-id pi@10.0.0.1
```

---

## USB Drives Preparation

- Format: FAT32  
- SAME label (example: USB or PHOTOS)

Create:

```
/sending
/receiving
```

Behavior:

- A → B  
- B → A  

---

## Script Creation

```
nano /home/pi/imagesync.sh
```

```bash
#!/bin/sh
echo "Host: ""$(hostname)"
echo "Beginning photo sync..."

if [ "$(hostname)" = "rpa" ]; then
  recipient_host="10.0.0.49"
else
  recipient_host="10.0.0.1"
fi

usb_drive_name="$(ls /media/pi/ | head -n 1)"

sender_path=/media/pi/$usb_drive_name/sending
recipient_path=/media/pi/$usb_drive_name/receiving

rsync --progress -ab --recursive --ignore-times $sender_path/. pi@$recipient_host:$recipient_path

echo "Fin."
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

```
* * * * * /home/pi/imagesync.sh
```

---

## Disk Imaging / Backup

See:


[sd-card-copy-macos_en.md](sd-card-copy-macos_en.md)


⚠️ USB drives must have identical labels.

---

## Summary

- reproducible system  
- no DHCP  
- hardware independent  
- bidirectional sync  
