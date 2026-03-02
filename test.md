# Disk Management and Partitioning

## Current Disk Layout
```
http1@web:~$ lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda               8:0    0   10G  0 disk 
sr0              11:0    1  9.5G  0 rom  
nvme0n1         259:0    0   10G  0 disk 
├─nvme0n1p1     259:1    0    1M  0 part 
├─nvme0n1p2     259:2    0    1G  0 part /boot
└─nvme0n1p3     259:3    0    9G  0 part 
 ├─ol_web-root   ...
 └─ol_web-swap   ...
```

**Note:** The disk `sda` is currently unused.

## Attempted Operations on `/dev/sda`
- Checked the disk label:
```bash
sudo parted /dev/sda print
```
> Error: `/dev/sda`: unrecognised disk label
> Model: VMware, VMware Virtual S (scsi)
> Disk /dev/sda: **10.7GB**
> Sector size (logical/physical): **512B/512B**
> Partition Table: **unknown**
> Disk Flags:

- Created a new GPT partition table:
```bash
sudo parted /dev/sda mklabel gpt
```
> *Information:* You may need to update `/etc/fstab`.

- Created a primary partition with XFS filesystem:
```bash
sudo parted /dev/sda mkpart primary xfs **2048s** **100%**
```
to be continued...
