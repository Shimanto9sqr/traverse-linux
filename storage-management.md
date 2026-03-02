# Storage related commands

**`lsblk` to list the block devices**
> Displays block devices, partition, size, RM(removable), RO(Read Only), Mountpoints
```
lsblk
```
>Shows UUID of each partition
```
lsblk -fs
```
**`du` to see File system space usage**
> Displays Filesystems, Type, Used space, Available Space, percentage, Mount points/ or from where the virtual devices are accessible

```du -hT
```
Output:
```
Filesystem                      Type      Size  Used Avail Use% Mounted on
/dev/mapper/ol_web-root         xfs       8.0G  5.9G  2.2G  74% /
devtmpfs                        devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs                           tmpfs     758M     0  758M   0% /dev/shm
tmpfs                           tmpfs     303M  6.3M  297M   3% /run
tmpfs                           tmpfs     1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/nvme0n1p2                  xfs       960M  434M  527M  46% /boot
/dev/mapper/vg_training-lv_data xfs        11G  248M   11G   3% /data
/dev/mapper/vg_training-lv_logs xfs        10G  228M  9.8G   3% /logs
/dev/sdd1                       xfs        10G  228M  9.8G   3% /projectdata
tmpfs                           tmpfs     152M   72K  152M   1% /run/user/42
tmpfs                           tmpfs     152M   56K  152M   1% /run/user/1000

```
