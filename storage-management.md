## Storage related commands

### **`lsblk` to list the block devices**
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
### **`du` to see File system space usage**
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

### **`parted` to create partition**
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

### **`mkfs` to format the partition**
```
mkfs -t type /dev/sdXp
```
### **`mount` to mount filesystem with devices**
>need root priv
```
mount mount_point
```

## Logical Volume Manager 

### **`pvcreate` `vgcreate` `lvcreate` `vgextend` `lvextend` `xfs_growfs` `resize2fs` `lvremove` `vgdisplay` `pvdisplay` `lvdisplay` `vgs`**

## NFS Server
### **`nfs-utils` `exportfs`**

## Task1 - Create standart partition
> Need root privilege
```
Identify unused partition -> Create partition -> Register with Kernel -> Format with xfs fs -> mount
```
```
parted /dev/sdd mklabel gpt mkpart primary 2048s 10G
```
```
udevadm settle
```
```
mkfs -t xfs /dev/sdd1
```
```
mkdir /projectdata
```
> Add `UUID` `mountpoint` `fs` `mode` `dump` `filecheck priority` to /etc/fstab for persistence
> find `UUID` using `blkid /dev/sdXn`

```
vim /etc/fstab
```
```
systemctl daemon-reload
```
```
mount /projectdata
```
```
reboot
```
```
lsblk/df -hT
```

## Task2 - Create two LVM using two unused disk
> Need root privilege
>Create partition -> set lvm on -> create pv -> create vg -> create lv -> format each lv -> edit /etc/fstab -> mount fs_mountpoint

```
parted /dev/sda mklabel gpt mkpart lvm_part1 2048s 100%
```
```
parted /dev/sda1 set lvm on
```
```
pvcreate /dev/sda1
```
>Repeat for `/dev/sdb`
```
vgcreate vg_training /dev/sda1 /dev/sdb1
```
```
lvcreate -L 6G -n lv_data vg_training
```
```
lvcreate -L 4G -n lv_logs vg_training
```
```
mkfs -t xfs /dev/vg_training/lv_data
```
```
mkfs -t xfs /dev/vg_training/lv_logs
```
> Get UUID's of lvm using `blkid` add them to `/etc/fstab`
>Reload daemon to make /etc/fstab update effective
```
systemctl daemon-reload
```
```
mkdir /data /logs
```
```
mount /data
```
```
mount /logs
```

## Task3 - Extend a LVM

>Create new PV if necessary -> Extend VG -> Extend LV -> Expand FS
```
vgextend vg_training /dev/sdXn
```
```
lvextend -L +6G /dev/vg_training/lv_logs
```
> To expand xfs FS
```
xfs_growfs /logs
```

## Task4 - NFS Configuration

> Install nfs-utils on host server and client
> Enable nfs-server and rpcbind service
> Edit /etc/exports to configure sharing
> Change ownership of the directory to allow anyone on the client system to read and write

>Need root privilege
### Setup NFS server on host
```
dnf install nfs-utils
```
```
systemctl enable --now nfs-server
```
```
systemctl enable --now rpcbind
```
> On /etc/exports
```
/path/to/shared_dir client_ip(rw,sync,no_subtree_check)
```
> Set Ownership
```
chown nobody:nobody /path/to/shared_dir \
chmod 755 /path/to/shared_dir
```
```
exportfs -r
```
>Disable Firewall and SELinux
```
systemctl stop firewalld
```
```
systemctl disable firewalld
```
> disable SElinux on /etc/selinux/config
```
SELINUX=disable
```
```
reboot
```
> Configure nfs-utils on client
```
dnf install nfs-utils
```
```
mount -t nfs 192.168.122.17:/data /nfs/data
```
> Edit /etc/fstab for persistence
```
192.168.98.131:/data                      /nfs/data               nfs     rw        0 0
```
> Check mount status
```
mount | grep /nfs/data
```



