
Partitioning [[filesystem]] :

The first issue with partitioning is the question of why you should do it. The answer is that partitioning provides a variety of advantages, including the following:

- Multiple-OS Support Partitioning enables you to keep the data for different OSs separate. In fact, many OSs can’t easily coexist on the same partition because they don’t support each other’s primary fi esystems. This feature is obviously important mainly if you want the computer to boot multiple OSs. It can also be handy to help maintain an emergency
system—you can install a single OS twice, using the second installation as an emergency maintenance tool for the fi st in case problems develop.

- Filesystem Choice By partitioning your disk, you can use different fi lesystems—data structures designed to hold all of the files on a partition—on each partition. Perhaps one filesystem is faster than another and so it is important for time-critical or frequently accessed files, but another may provide accounting or backup features you want to use for
users’ data fies.

- Disk Space Management By partitioning your disk, you can lock certain sets of files into a fixed space. For instance, if you restrict users to storing files on one or two partitions, they can fill those partitions without causing problems on other partitions, such as system partitions. This feature can help keep your system from crashing if space runs out. On the other hand, if you get the partition sizes wrong, you can run out of disk space on just one partition much sooner than would be the case if you’d used fewer partitions.

- Disk Error Protection Disks sometimes develop problems. These problems can be the result of bad hardware or errors that creep into the filesystems. In either case, splitting a disk into partitions provides some protection against such problems. If data structures on one partition become corrupted, the errors affect only the fi les on that partition. This separation can therefore protect data on other partitions and simplify data recovery. Security You can use different security-related mount options on different partitions. For instance, you might mount a partition that holds critical system files in read-only mode, preventing users from writing to that partition. Linux’s file security options should provide similar protection, but taking advantage of Linux filesystem mount options provides redundancy that can be helpful in case of an error in setting up file or directory permissions.

- Backup Some backup tools work best on whole partitions. By keeping partitions small, you may be able to back up more easily than you could if your partitions were large.


**Understanding Partitioning Systems**

Partitions are defined by data structures that are written to specified parts of the hard disk. Several competing systems for defining these partitions exist. On x86 and x86-64 hardware, the most common method up until 2010 had been the Master Boot Record (MBR) partitioning system, so called because it stores its data in the first sector of the disk, which is also known as the MBR. The MBR system, however, is limited to partitions and partition placement of 2 tebibytes (TiB; 1TiB is 240 bytes), at least when using the nearly universal sector size of 512 bytes. The successor to MBR is the GUID Partition Table (GPT) partitioning system, which has much higher limits and certain other advantages.