[[command]]
GPT is part of Intel’s EFI specification, but GPT can be used on computers that don’t use EFI, and GPT is the preferred partitioning system for disks bigger than 2TiB. Most EFI based computers use GPT even on disks smaller than 2TiB.
GPT employs a protective MBR, which is a legal MBR definition that makes GPTunaware utilities think that the disk holds a single MBR partition that spans the entire disk. Additional data structures define the true GPT partitions. These data structures are duplicated, with one copy at the start of the disk and another at its end. This provides redundancy that can help in data recovery should an accident damage one of the two sets of data structures.

GPT does away with the primary/extended/logical distinction of MBR. You can define up to 128 partitions by default (and that limit may be raised, if necessary). Gaps can occur in partition numbering, so you can have a disk with three partitions numbered 3, 7, and 104, to name just one possibility. In practice, though, GPT partitions are usually numbered consecutively starting with 1.

GPT’s main drawback is that support for it is relatively immature. The fdisk utility doesn’t work with GPT disks, although alternatives to fdisk are available. Some versions of the GRUB boot loader also don’t support it.

The situation is worse in some OSs—particularly older ones. Nonetheless, you should be at least somewhat familiar with GPT because of MBR’s inability to handle disks larger than 2TiB.

Like MBR, GPT supports partition type codes; however, GPT type codes are 16-byte GUID values. Disk partitioning tools typically translate these codes into short descriptions, such as “Linux swap.” Confusingly, most Linux installations use the same type code for their filesystems that Windows uses for its filesystems, although a Linux-only code is available and is gaining in popularity among Linux distributions
