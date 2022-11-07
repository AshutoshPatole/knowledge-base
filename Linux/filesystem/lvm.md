
## What is LVM?
Managing disk space in [[filesystem]] has always been a significant task for sysadmins. Running out of disk space used to be the start of a long and complex series of tasks to increase the space available to a disk partition. It also required taking the system off-line. This usually involved installing a new hard drive, booting to recovery or single-user mode, creating a partition and a filesystem on the new hard drive, using temporary mount points to move the data from the too-small filesystem to the new, larger one, changing the content of the /etc/fstab file to reflect the correct device name for the new partition, and rebooting to remount the new filesystem on the correct mount point.

LVM allows for very flexible disk space management. It provides features like the ability to add disk space to a logical volume and its filesystem while that filesystem is **mounted and active** and it allows for the collection of multiple physical hard drives and partitions into a single volume group which can then be divided into logical volumes.

>[!important]
The EXT2, 3, and 4 filesystems all allow both offline (unmounted) and online (mounted) resizing when increasing the size of a filesystem, and **ONLY** offline resizing when reducing the size. You should check the details of the filesystems you intend to use in order to verify whether they can be resized at all and especially whether they can be resized while online.

## LVM Structure

![](https://media.springernature.com/original/springer-static/image/chp%3A10.1007%2F978-1-4842-5455-4_1/MediaObjects/489914_1_En_1_Fig1_HTML.jpg)

> Note: Layer numbering starts from bottom

**Layer 1** Physical hard drives : Self explanatory term.

**Layer 2** Partitioning: This is an optional step and should be performed when you are required to deal with some portion of the physical hard drive.

**Layer 3** Physical Volumes : This is the first step for LVM which acts as a individual hard drive / partition.

>[!warning]
The **`/boot`** partition cannot reside on an LVM volume because the GRUB boot loader cannot read it.


**Layer 4** All Physcial Volumes (PV) are combined into one Volume Group (VG)

**Layer 5** The VG is then divided into multiple Logic Volumes (LV) containing their own filesystem types.

## Use case of LVM

- Creating single logical volumes of multiple physical volumes or entire hard disks (somewhat similar to RAID 0, but more similar to JBOD), allowing for dynamic volume resizing.
- Managing large hard disk farms by allowing disks to be added and replaced without downtime or service disruption, in combination with hot swapping.
- On small systems (like a desktop), instead of having to estimate at installation time how big a partition might need to be, LVM allows filesystems to be easily resized as needed.
- Performing consistent backups by taking snapshots of the logical volumes.
Encrypting multiple physical partitions with one password.

## Adding a new logical volume
There are times when it is necessary to add a new logical volume to a host. The basic steps for adding a new logical volume are as follows.

- If necessary, install a new hard drive.
- Optional: Create a partition on the hard drive.
- Create a physical volume (PV) of the complete hard drive or a partition on the hard drive.
- Assign the new physical volume to an existing volume group (VG) or create a new volume group.
- Create a new logical volumes (LV) from the space in the volume group.
- Create a filesystem on the new logical volume.
- Add appropriate entries to `/etc/fstab` for mounting the filesystem.
- Mount the filesystem.

>[!caution]
Only the EXT3 and EXT4 filesystems can be resized on the fly on a running, mounted filesystem. Many other filesystems including BTRFS and ZFS cannot be resized.


### Install the hard drive
If there is not enough space on the existing hard drive(s) in the system to add the desired amount of space it may be necessary to add a new hard drive and create the space to add to the Logical Volume. First, install the physical hard drive and then perform the following steps.

```bash
[root@centos ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0   12G  0 disk
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   11G  0 part
  ├─cs-root 253:0    0  9.8G  0 lvm  /
  └─cs-swap 253:1    0  1.2G  0 lvm  [SWAP]
sdb           8:16   0    4G  0 disk
sr0          11:0    1 1024M  0 rom
```

As you can see here we have sda disk with two partitions. One for **boot** and other is for **LVM**. The naming convention for LVM is mostly similar on all major distributions.

    ├─cs-root 
    |-VG_NAME-LV_NAME
    └─cs-swap 
We have around 9.8GB disk space for our root partition which is filling up quickly hence we have added a new disk **sdb** of 4GB. Now here comes 2 scenarios we can either assign the entire disk for PV or create a parition first and then assign it. We are going with the later one for now.
So let's create a partition of 2GB on the /dev/sdb.

>[!info]
> This is an optional step and if you want to allocate entire disk space then it is not needed.


```bash
[root@centos ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x93d8aa54.

Command (m for help): n                     # n - Create a new partition
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p                       # p - primary partition
Partition number (1-4, default 1): 1        # 1 for naming disk as sdb1
First sector (2048-8388607, default 2048):  # just hit enter
Last sector, +sectors or +size{K,M,G,T,P} (2048-8388607, default 8388607): +2G      # + is Required

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help):

```
Until now we have created a partition of 2GB on the disk but do not exit the fdisk utility and continue for changing the filesystem type.

```bash
Command (m for help): t                      # t - change type
Selected partition 1
Hex code (type L to list all codes): 8e      # 8e is hex code for Linux LVM
Changed type of partition 'Linux' to 'Linux LVM'.

Command (m for help): w                      # w - write changes to disk 
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

```
We have got the parition of 2GB
```bash
[root@centos ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0   12G  0 disk
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   11G  0 part
  ├─cs-root 253:0    0  9.8G  0 lvm  /
  └─cs-swap 253:1    0  1.2G  0 lvm  [SWAP]
sdb           8:16   0    4G  0 disk
└─sdb1        8:17   0    2G  0 part        # <== sdb1
sr0          11:0    1 1024M  0 rom
```
> **Note** All commands related to LV Management is [[lvm cheatsheet]]

### Create a Physical Volume (PV)
As we can see we have only one PV on the machine located at /dev/sda2 and has a size of ~11GB.
```bash
[root@centos ~]# pvscan
  PV /dev/sda2   VG cs              lvm2 [<11.00 GiB / 0    free]
  Total: 1 [<11.00 GiB] / in use: 1 [<11.00 GiB] / in no VG: 0 [0   ]
```
To create a PV use pvcreate command on the disk sdb1
```bash

[root@centos ~]# pvcreate /dev/sdb1
Physical volume "/dev/sdb1" successfully created.

[root@centos ~]# lsblk -f
NAME        FSTYPE      LABEL UUID                                   MOUNTPOINT
sda
├─sda1      xfs               d153ab14-0930-42cf-bea7-41da3b75f166   /boot
└─sda2      LVM2_member       rnPx56-D2G4-vL2i-xBMT-b06u-sldD-ffm9Th
  ├─cs-root xfs               20f510f9-31bf-43e8-9b57-b96c80ca82d7   /
  └─cs-swap swap              b09431d9-6c74-4715-87db-1841802ba5de   [SWAP]
sdb
└─sdb1      LVM2_member       hMd1nq-oAmf-MBVI-xPsH-bqFm-wXu3-YJ91Lf
sr0

[root@centos ~]# pvscan
  PV /dev/sda2   VG cs              lvm2 [<11.00 GiB / 0    free]
  PV /dev/sdb1                      lvm2 [2.00 GiB]
  Total: 2 [<13.00 GiB] / in use: 1 [<11.00 GiB] / in no VG: 1 [2.00 GiB]
```
### Extending Volume Group size
Now you can see that PV collectible size is ~13GB but our PV is not connected to the VG (cs). So we have to extend our VG using vgextend command
```bash
[root@centos ~]# vgscan
  Found volume group "cs" using metadata type lvm2
[root@centos ~]# vgextend cs /dev/sdb1
  Volume group "cs" successfully extended
```

Now with this 2GB of newly added space we are going to do two things.
- Create a new Logical volume of space 1GB.
- Add 1GB of space to an existing root logical volume.
### Create a new LV
You can verify that we have only 2 LV present on the system.

```bash
[root@centos ~]# lvs
  LV   VG Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root cs -wi-ao---- 9.79g
  swap cs -wi-ao---- 1.20g
```
To create a new LV use lvcreate command.
```bash

[root@centos ~]# lvcreate --name new-lv -l 50%FREE cs
  Logical volume "new-lv" created.

[root@centos ~]# lvs
  LV     VG Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  new-lv cs -wi-a----- 1020.00m
  root   cs -wi-ao----    9.79g
  swap   cs -wi-ao----    1.20g
```


>[!tip]
> You can also specify capital -L 1G on the command.
```bash
$ lvcreate --name new-lv -L 1G cs
```
use `lvcreate --help` to learn about it.


### Extending existing LV
As you can see the root LV size is around 9.79 GB and we are going to extend it with another 1GB.
```bash
[root@centos ~]# lvs
  LV     VG Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  new-lv cs -wi-a----- 1020.00m
  root   cs -wi-ao----    9.79g
  swap   cs -wi-ao----    1.20g
```
use lvextend to extend the existing LV.

```bash
[root@centos ~]# lvextend -L +1G /dev/cs/root
  Size of logical volume cs/root changed from 9.79 GiB (2507 extents) to 10.79 GiB (2763 extents).
  Logical volume cs/root successfully resized.
[root@centos ~]# lvs
  LV     VG Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  new-lv cs -wi-a----- 1020.00m
  root   cs -wi-ao----   10.79g
  swap   cs -wi-ao----    1.20g
```

You can cross verify the disk UUID of cs-root if you want.

```bash
[root@centos ~]# lsblk -f
NAME           FSTYPE      LABEL UUID                                   MOUNTPOINT
sda
├─sda1         xfs               d153ab14-0930-42cf-bea7-41da3b75f166   /boot
└─sda2         LVM2_member       rnPx56-D2G4-vL2i-xBMT-b06u-sldD-ffm9Th
  ├─cs-root    xfs               20f510f9-31bf-43e8-9b57-b96c80ca82d7   /
  └─cs-swap    swap              b09431d9-6c74-4715-87db-1841802ba5de   [SWAP]
sdb
└─sdb1         LVM2_member       hMd1nq-oAmf-MBVI-xPsH-bqFm-wXu3-YJ91Lf
  ├─cs-root    xfs               20f510f9-31bf-43e8-9b57-b96c80ca82d7   /
  └─cs-new--lv
sr0
```
but this hasn't actually increased the disk space. If you check `df -h` output you will see nothing changed.

```bash
[root@centos ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             887M     0  887M   0% /dev
tmpfs                907M     0  907M   0% /dev/shm
tmpfs                907M  8.5M  898M   1% /run
tmpfs                907M     0  907M   0% /sys/fs/cgroup
/dev/mapper/cs-root  9.8G  2.4G  7.5G  25% /                        # Same 9.8G
/dev/sda1           1014M  212M  803M  21% /boot
tmpfs                182M     0  182M   0% /run/user/0
```
We have added the space but we haven't resized it actually. To resize it use `xfs_growfs` command

```bash
[root@centos ~]# xfs_growfs /dev/cs/new-lv
meta-data=/dev/cs/new-lv    isize=512    agcount=4, agsize=641792 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=2567168, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 2567168 to 2829312

[root@centos ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             887M     0  887M   0% /dev
tmpfs                907M     0  907M   0% /dev/shm
tmpfs                907M  8.5M  898M   1% /run
tmpfs                907M     0  907M   0% /sys/fs/cgroup
/dev/mapper/cs-root   11G  2.4G  8.5G  23% /                      # increased to 11G from 9.8G
/dev/sda1           1014M  212M  803M  21% /boot
tmpfs                182M     0  182M   0% /run/user/0
```
>[!info]
This part is specific to the filesystem type, although the ext2/3/4 filesystem types are similar enough that they can all be resized with a single **resize2fs** tool. For XFS, filesystems, you would use a **xfs_growfs** tool instead. Other filesystems may have their own extension tools. And if the logical volume did not contain a filesystem but instead something like a "raw" database or an Oracle ASM volume, a yet another procedure would need to be applied.


>[!tip]
You can also use `-r` flag while extending the lvm. This will automatically increase the filesystem space.
> ```bash
[root@centos ~]# lvextend -L +1G -r /dev/cs/root


### Format the newly created LV new-lv
The LV in the above examples are formated as xfs but the newly created LV is not formatted yet hence it shows nothing in the above output. To format a disk use mkfs utility.

```bash
[root@centos ~]# mkfs.xfs /dev/cs/new-lv
meta-data=/dev/cs/new-lv isize=512    agcount=4, agsize=65280 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=261120, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1566, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

[root@centos ~]# lsblk -f
NAME           FSTYPE      LABEL UUID                                   MOUNTPOINT
sda
├─sda1         xfs               d153ab14-0930-42cf-bea7-41da3b75f166   /boot
└─sda2         LVM2_member       rnPx56-D2G4-vL2i-xBMT-b06u-sldD-ffm9Th
  ├─cs-root    xfs               20f510f9-31bf-43e8-9b57-b96c80ca82d7   /
  └─cs-swap    swap              b09431d9-6c74-4715-87db-1841802ba5de   [SWAP]
sdb
└─sdb1         LVM2_member       hMd1nq-oAmf-MBVI-xPsH-bqFm-wXu3-YJ91Lf
  ├─cs-root    xfs               20f510f9-31bf-43e8-9b57-b96c80ca82d7   /
  └─cs-new--lv xfs               fa5880d7-2624-4bc1-9d74-0bd49193fbad
sr0
```

## Reducing LV Size

>[!caution]
Please go through full section before attempting anything.


To decrease the size of an LVM partition you must first decrease the file system within in order to avoid possible data corruption. As there is the potential for this to happen if you enter the command incorrectly, it is strongly recommended that you have a full backup of your data before proceeding.

> _The first step will depend on if you're looking to shrink a **LVM root volume**, or **non-root volume.**_


### Shrinking a root volume
The root volume would typically be the logical volume that is mounted to `/`. You cannot unmount this to shrink it as it's in use by the running operating system meaning that you will have to first boot from a **Live CD** to complete this. Once booted into the Live CD, you may first need to run the below command to pick up LVM volumes, however this usually happens during boot so may not be required, if in doubt just run it.
```bash
[root@centos ~]# vgchange -a y
3 logical volume(s) in volume group "cs" now active
```
### Shrinking a non-root volume
Alternatively if the volume you are shrinking is a non-root volume, that is any other volume not mounted to the root of the file system, you can unmount the volume as shown below to proceed. Please note that when you unmount the volume the data will not be available, so you may need to schedule down time and stop running applications that use data from it prior to unmounting. Unmount by specifying either the logical volume or the location it's currently mounted to, in the below example we specify the logical volume which can be found in `/dev/(vg-name)/(lv-name)`.

```bash
[root@centos ~]# lsblk -f
NAME           FSTYPE      LABEL UUID                                   MOUNTPOINT
sda
├─sda1         xfs               d153ab14-0930-42cf-bea7-41da3b75f166   /boot
└─sda2         LVM2_member       rnPx56-D2G4-vL2i-xBMT-b06u-sldD-ffm9Th
  ├─cs-root    xfs               20f510f9-31bf-43e8-9b57-b96c80ca82d7   /
  └─cs-swap    swap              b09431d9-6c74-4715-87db-1841802ba5de   [SWAP]
sdb
└─sdb1         LVM2_member       hMd1nq-oAmf-MBVI-xPsH-bqFm-wXu3-YJ91Lf
  ├─cs-root    xfs               20f510f9-31bf-43e8-9b57-b96c80ca82d7   /
  └─cs-new--lv ext4              6cdb2bc5-9e63-4cd0-8048-50e06c48cacf
sr0

[root@centos ~]# umount /dev/cs/new-lv
[root@centos ~]#

```

**All following steps now apply to both a root or non-root volume.**

Before being able to attempt to shrink the size of an LVM volume, you must first run a file system check on it. If you don't do this, you will get an error message and will not be able to proceed. This is a required step as resizing a file system in a bad state could cause data corruption. The `-f` flag makes the check run even if the file system appears clean, while -y assumes yes to all questions and will respond if asked to fix a problem.

```bash
[root@centos ~]# e2fsck /dev/cs/new-lv
e2fsck 1.45.6 (20-Mar-2020)
/dev/cs/new-lv: clean, 11/65280 files, 8843/261120 blocks
```

I have used `e2fsck` since my filesystem is ext4 for `cs-new--lv`. If it is xfs then `xfs_repair` should be used.

Next you need to shrink the file system, to be safe we're going to shrink the file system lower than what the logical volume will shrink to. This is because we don't want to accidentally shrink the logical volume to a size lower than the file system in the next step, as this can result in corruption and data loss. Don't worry, we'll reclaim the space at the end.

The command below will shrink the file system so that it is only 4G in size total, note that what ever size you specify to shrink to you must have in free space within the file system otherwise you must first delete data.

```bash
[root@centos ~]# resize2fs /dev/cs/new-lv 500M
resize2fs 1.45.6 (20-Mar-2020)
Resizing the filesystem on /dev/cs/new-lv to 128000 (4k) blocks.
The filesystem on /dev/cs/new-lv is now 128000 (4k) blocks long.

```
Once the file system has been reduced, we can shrink the size of the logical volume with the lvreduce command. Reduce this to the size that you want the volume to be, as specified by the -L flag. Instead if you want to reduce by a specified size, simply put a **`-`** in front of the size. 

```bash
[root@centos ~]# lvreduce -L -500M /dev/cs/new-lv
  WARNING: Reducing active logical volume to 520.00 MiB.
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce cs/new-lv? [y/n]: y
  Size of logical volume cs/new-lv changed from 1020.00 MiB (255 extents) to 520.00 MiB (130 extents).
  Logical volume cs/new-lv successfully resized.
```

Once you execute the lvreduce command you will get a warning advising the size you have chosen to reduce to so use this as a chance to confirm you're shrinking the logical volume to a size that is NOT smaller than the size you previously shrunk the file system to. Once you have confirmed it's fine to proceed enter 'y' and press enter.

After the logical volume has been lowered to the required size, run resize2fs on the volume as this will extend the file system to use all available space within the logical volume. This makes use of all remaining free space so that none is wasted from when we previously shrunk the file system to a lower size than the logical volume.
```bash
[root@centos ~]# resize2fs /dev/cs/new-lv
resize2fs 1.45.6 (20-Mar-2020)
Resizing the filesystem on /dev/cs/new-lv to 133120 (4k) blocks.
The filesystem on /dev/cs/new-lv is now 133120 (4k) blocks long.
```
Thus our new lvm is reduced by 500M. Verified.
```bash
[root@centos ~]# lsblk
NAME           MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda              8:0    0   12G  0 disk
├─sda1           8:1    0    1G  0 part /boot
└─sda2           8:2    0   11G  0 part
  ├─cs-root    253:0    0 10.8G  0 lvm  /
  └─cs-swap    253:1    0  1.2G  0 lvm  [SWAP]
sdb              8:16   0    4G  0 disk
└─sdb1           8:17   0    2G  0 part
  ├─cs-root    253:0    0 10.8G  0 lvm  /
  └─cs-new--lv 253:2    0  520M  0 lvm
sr0             11:0    1 1024M  0 rom
```

>[!important]
I have used ext4 example while demonstrating `lvreduce`. The reason is after an XFS file system is created, its size **cannot** be reduced. However, it can still be enlarged using the **xfs_growfs** command. Shrinking down an xfs filesystem would be a _rare_ case on most of the production systems but just in case you get this scenario, there is a tricky workaround.

Use `xfsdump` utility and take a snapshot of the LV. Create a new LV and format it as xfs and then use `xfsrestore` command to restore it.


#storage  #filesystem #linux 