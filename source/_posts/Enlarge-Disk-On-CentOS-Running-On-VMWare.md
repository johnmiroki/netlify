title: Enlarge Disk On CentOS Running On VMWare
author: John Miroki
date: 2018-09-07 12:00:02
tags:
---
TL;DR: Hard drive has physical partitions (physical volumes), which can be assigned to logical volume groups, which can be devided to logical volumes, which can be somehow mapped as disk, which can be mounted to linux directories. using an example: 

`/dev/sda(hard drive) -> /dev/sda2(partition) -> centos(volume group) -> /dev/centos/root -> /dev/mapper/centos-root -> /`

and the commands used are:
`fdisk -l` -> `vgdisplay` -> `lvdisplay` -> `df -h`



I'd like to increase the general space of my vmware virtual host. After allocating more space (1GB) to the host by modifying the configuratiton on vmware, I feel confused.

When installing CentOS, I used its default setting. This is the current situation:
```shell
[root@cluster01 ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   17G   14G  3.4G  81% /
devtmpfs                 1.9G     0  1.9G   0% /dev
tmpfs                    1.9G     0  1.9G   0% /dev/shm
tmpfs                    1.9G   12M  1.9G   1% /run
tmpfs                    1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/sda1               1014M  142M  873M  14% /boot
tmpfs                    378M     0  378M   0% /run/user/0
tmpfs                    378M     0  378M   0% /run/user/998
cm_processes             1.9G     0  1.9G   0% /run/cloudera-scm-agent/process
```

As can be seen, whatever mapping to "/" has limited space left.
```
[root@cluster01 ~]# fdisk -l

Disk /dev/sda: 32.2 GB, 32212254720 bytes, 62914560 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000a2f79

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    41943039    19921920   8e  Linux LVM

Disk /dev/mapper/centos-root: 18.2 GB, 18249416704 bytes, 35643392 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/centos-swap: 2147 MB, 2147483648 bytes, 4194304 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

I am honestly dazzled by this info, what I can gather is:

1. `Disk /dev/sda: 32.2 GB` this looks like the size of the physical disk
2. `/dev/sda2         2099200    41943039    19921920   8e  Linux LVM` this looks like a device using the physical disk
3. `Disk /dev/mapper/centos-root: 18.2 GB` whatever it is, this is what's mapped to "/" folder

At this point, I naturally want to move space from 1 to 3. After some googling, the following is what I do:

1. Create a new primary partition, making it a "linux lvm" type:

```
[root@cluster01 ~]# fdisk /dev/sda
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): m
Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   g   create a new empty GPT partition table
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)

Command (m for help): n
Partition type:
   p   primary (2 primary, 0 extended, 2 free)
   e   extended
Select (default p): p
Partition number (3,4, default 3):
First sector (41943040-62914559, default 41943040):
Using default value 41943040
Last sector, +sectors or +size{K,M,G} (41943040-62914559, default 62914559):
Using default value 62914559
Partition 3 of type Linux and of size 10 GiB is set

Command (m for help): t
Partition number (1-3, default 3):
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
```
What I do here is to create a new partition which takes all the free space, and then change its type to "8e"(code for "linux lv")

```
[root@cluster01 ~]# fdisk -l

Disk /dev/sda: 32.2 GB, 32212254720 bytes, 62914560 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000a2f79

/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    41943039    19921920   8e  Linux LVM
/dev/sda3        41943040    62914559    10485760   8e  Linux LVM
```

2. After rebooting, allocating space to volume group

```
[root@cluster01 ~]# vgdisplay
  --- Volume group ---
  VG Name               centos
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <19.00 GiB
  PE Size               4.00 MiB
  Total PE              4863
  Alloc PE / Size       4863 / <19.00 GiB
  Free  PE / Size       0 / 0
  VG UUID               QrSMSB-dJyP-7JYM-GyAy-UKPW-0XBE-PqCYQb
  
[root@cluster01 ~]# vgextend centos /dev/sda3
  Physical volume "/dev/sda3" successfully created.
  Volume group "centos" successfully extended
  
[root@cluster01 ~]# vgdisplay
  --- Volume group ---
  VG Name               centos
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               28.99 GiB
  PE Size               4.00 MiB
  Total PE              7422
  Alloc PE / Size       4863 / <19.00 GiB
  Free  PE / Size       2559 / <10.00 GiB
  VG UUID               QrSMSB-dJyP-7JYM-GyAy-UKPW-0XBE-PqCYQb

```
3. Allocating free space from volume group to logical volume

```
[root@cluster01 ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/centos/swap
  LV Name                swap
  VG Name                centos
  LV UUID                dbl59z-eFfF-DJ8G-QHKS-akGb-VBc2-ZQ93sD
  LV Write Access        read/write
  LV Creation host, time cluster01.john42.com, 2018-08-21 13:58:38 +0800
  LV Status              available
  # open                 2
  LV Size                2.00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1

  --- Logical volume ---
  LV Path                /dev/centos/root
  LV Name                root
  VG Name                centos
  LV UUID                eiV78Y-pizG-9frt-y1ak-LTII-baUQ-NZhuJF
  LV Write Access        read/write
  LV Creation host, time cluster01.john42.com, 2018-08-21 13:58:38 +0800
  LV Status              available
  # open                 1
  LV Size                <17.00 GiB
  Current LE             4351
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
  
[root@cluster01 ~]# lvextend -l +100%FREE /dev/centos/root
  Size of logical volume centos/root changed from <17.00 GiB (4351 extents) to 26.99 GiB (6910 extents).
  Logical volume centos/root successfully resized.
  
[root@cluster01 ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/centos/swap
  LV Name                swap
  VG Name                centos
  LV UUID                dbl59z-eFfF-DJ8G-QHKS-akGb-VBc2-ZQ93sD
  LV Write Access        read/write
  LV Creation host, time cluster01.john42.com, 2018-08-21 13:58:38 +0800
  LV Status              available
  # open                 2
  LV Size                2.00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1

  --- Logical volume ---
  LV Path                /dev/centos/root
  LV Name                root
  VG Name                centos
  LV UUID                eiV78Y-pizG-9frt-y1ak-LTII-baUQ-NZhuJF
  LV Write Access        read/write
  LV Creation host, time cluster01.john42.com, 2018-08-21 13:58:38 +0800
  LV Status              available
  # open                 1
  LV Size                26.99 GiB
  Current LE             6910
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
```

4. resize the volume:

```
[root@cluster01 ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   17G   14G  3.4G  81% /
devtmpfs                 1.9G     0  1.9G   0% /dev
tmpfs                    1.9G     0  1.9G   0% /dev/shm
tmpfs                    1.9G   12M  1.9G   1% /run
tmpfs                    1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/sda1               1014M  142M  873M  14% /boot
tmpfs                    378M     0  378M   0% /run/user/0
tmpfs                    378M     0  378M   0% /run/user/998
cm_processes             1.9G     0  1.9G   0% /run/cloudera-scm-agent/process

[root@cluster01 ~]# xfs_growfs /dev/centos/root
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=1113856 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=4455424, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 4455424 to 7075840

[root@cluster01 ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   27G   14G   14G  51% /
devtmpfs                 1.9G     0  1.9G   0% /dev
tmpfs                    1.9G     0  1.9G   0% /dev/shm
tmpfs                    1.9G   12M  1.9G   1% /run
tmpfs                    1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/sda1               1014M  142M  873M  14% /boot
tmpfs                    378M     0  378M   0% /run/user/0
tmpfs                    378M     0  378M   0% /run/user/998
cm_processes             1.9G     0  1.9G   0% /run/cloudera-scm-agent/process

```