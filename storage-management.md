## Storage related commands

###**`lsblk` to list the block devices**
```
lsblk
```
Output:
> Displays block devices, partition, size, RM(removable), RO(Read Only), Mountpoints
```
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                       8:0    0   10G  0 disk 
└─sda1                    8:1    0   10G  0 part 
  └─vg_training-lv_data 252:2    0   11G  0 lvm  /data
sr0                      11:0    1  9.5G  0 rom  
nvme0n1                 259:0    0   10G  0 disk 
├─nvme0n1p1             259:1    0    1M  0 part 
├─nvme0n1p2             259:2    0    1G  0 part /boot
└─nvme0n1p3             259:3    0    9G  0 part 
  ├─ol_web-root         252:0    0    8G  0 lvm  /
  └─ol_web-swap         252:1    0    1G  0 lvm  [SWAP]

```
>Shows UUID of each partition
```
lsblk -fs
```
###**`du` to see File system space usage**
```
du -hT
```
Output:
> Displays Filesystems, Type, Used space, Available Space, percentage, Mount points/ or from where the virtual devices are accessible
```
Filesystem                      Type      Size  Used Avail Use% Mounted on
/dev/mapper/ol_web-root         xfs       8.0G  5.9G  2.2G  74% /
tmpfs                           tmpfs     1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/nvme0n1p2                  xfs       960M  434M  527M  46% /boot
/dev/mapper/vg_training-lv_data xfs        11G  248M   11G   3% /data
/dev/mapper/vg_training-lv_logs xfs        10G  228M  9.8G   3% /logs
/dev/sdd1                       xfs        10G  228M  9.8G   3% /projectdata
```

###**`parted` to create partition**
> Used in standard partitioning
> Create partition table
> both GPT and MBR
*need root privilege*
```
parted /dev/sdX mklabel type mkpart partition_type name fs_type start end
```
> type: GPT, msdos, loop
> partition_type: "primary",  "logical", or "extended"
> fs_type: xfs, ext4, ntfs -optional
> use `rm` to remove a partition

###**`mkfs` formatting the partition**
```
mkfs -t type /dev/sdXp
```
###**`mount` to mount filesystem with devices**
>need root priv
```
mount mount_point
```

## Logical Volume Manager 

###`pvcreate` `vgcreate` `lvcreate` `vgextend` `lvextend` `xfs_growfs` `resize2fs` `lvremove` `vgdisplay` `pvdisplay` `lvdisplay` `vgs`
