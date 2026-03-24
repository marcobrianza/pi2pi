# Copying and Restoring a Raspberry Pi SD Card on macOS

These instructions show how to copy a Raspberry Pi SD card, verify its integrity, and write it back to another SD card. In the examples, `disk4` is used for reading and `disk6` for writing, to make it clear that the disk number may change and must always be checked.

## 1. Copying the original SD card (read)

Insert the original SD card into the Mac and open the Terminal app.

```bash
diskutil list
```

Example output:

```text
/dev/disk4 (external or internal, physical):
#:                       TYPE NAME                    SIZE       IDENTIFIER
0:     FDisk_partition_scheme                        *4.0 GB    disk4
1:             Windows_FAT_32 boot                   46.0 MB    disk4s1
2:                      Linux                        3.9 GB     disk4s2
```

- Identify the SD card to copy:

```bash
diskutil list
```

- Unmount the SD card:

```bash
diskutil unmountDisk /dev/disk4
```

- Create the compressed image:

```bash
sudo dd if=/dev/rdisk4 bs=4m status=progress | gzip > SD_RaspberryPi.img.gz
```

- Create the checksum:

```bash
shasum -a 256 SD_RaspberryPi.img.gz > SD_RaspberryPi.img.gz.sha256
```

- Eject the SD card:

```bash
diskutil eject /dev/disk4
```

## 2. Verifying the checksum

After transferring the files, open Terminal and verify the image integrity:

```bash
shasum -a 256 -c SD_RaspberryPi.img.gz.sha256
```

## 3. Writing to a new SD card

**WARNING – DESTRUCTIVE OPERATION**

Writing the image completely overwrites the selected disk without asking for confirmation.

If the correct disk is not checked with extreme care, you may accidentally erase a different disk than intended, for example the computer’s internal disk, with irreversible data loss.

Before proceeding:

- always run `diskutil list` immediately before writing;
- verify the size and partition structure of the SD card;
- double-check that the disk number (`diskX` in this example is intentionlay to be changed intentiolany);
- if in doubt, stop.

Insert the new SD card into the Mac and open Terminal.

- Identify the destination SD card:

```bash
diskutil list
```

- Unmount the destination SD card:

```bash
diskutil unmountDisk /dev/diskX
```

- Write the image to the new SD card:

```bash
gunzip -c SD_RaspberryPi.img.gz | sudo dd of=/dev/rdiskX bs=4m status=progress
```

- Eject the SD card:

```bash
diskutil eject /dev/diskX
```
