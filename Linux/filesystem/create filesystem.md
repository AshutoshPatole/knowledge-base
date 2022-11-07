
Creating a [[Filesystem]]
Linux tools exist that can create most filesystems on a partition, including all Linux-native filesystems. Typically, these tools have filenames in the form of mkfs.fstype, where fstype is the filesystem type code. These tools can also be called from a front-end tool called mkfs; you pass the filesystem type code to mkfs using its -t option:

```
$ mkfs -t ext3 /dev/sda6
```

> NOTE : For ext2 and ext3 filesystems, the mke2fs program is often used instead of mkfs. The mke2fs program is just another name for mkfs.ext2.

This command creates an ext3 filesystem on /dev/sda6. Depending on the filesystem, the speed of the disk, and the size of the partition, this process can take anywhere from a fraction of a second to a few seconds. Most filesystem-build tools support additional options, some of which can greatly increase the time required to build a filesystem. In
particular, the -c option is supported by several filesystems. This option causes the tool to perform a bad-block check—every sector in the partition is checked to be sure it can reliably hold data. If it can’t, the sector is marked as bad and isn’t used.