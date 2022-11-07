[[partitioning]]

Creating Swap Space: 

Some partitions don’t hold fi les. Most notably, Linux can use a swap partition, which is a partition that Linux treats as an extension of memory. (Linux can also use a swap file, which is a file that works in the same way. Both are examples of swap space.) Linux uses the MBR partition type code of 0x82 to identify swap space, but as with other partitions,
this code is mostly a convenience to keep other OSs from trying to access Linux swap partitions; Linux uses /etc/fstab to define which partitions to use as swap space,

Although swap space doesn’t hold a filesystem per se, and it isn’t mounted in the way that filesystem partitions are mounted, swap space does require preparation similar to that for creation of a filesystem. This task is accomplished with the mkswap command, which you can generally use by passing it nothing but the device identifier:

```
$ mkswap /dev/sda7
```

This example turns /dev/sda7 into swap space. To use the swap space, you must activate it with the swapon command:

```
$ swapon /dev/sda7
```

To activate swap space permanently, you must create an entry for it in /etc/fstab,
