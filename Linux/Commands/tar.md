[[command]]
Using the tar Utility :

The tar program’s name stands for “tape archiver.” Despite this fact, you can use tar to archive data to other media. In fact, tarballs are often used for transferring multiple files between computers in one step, such as when distributing source code. A tarball is an archive file created by tar and typically compressed with gzip, xz, or bzip2. The tar program is a complex package with many options. However, most of what you’ll do with this utility can be covered with a few common commands.

If you fail to specify a filename with the --file qualifier, tar will attempt to use a default device, which is often (but not always) a tape device file. Thus, it is always best to specify a filename. Three compression tools—gzip, bzip2, and xz—are often used with the tar command. They apply compression to the archive file as a whole rather than to the individual files. This compression method reduces the tarball’s size compared to compressing individual files and then adding them to the archive. However, this compression method makes the archive more susceptible to damage. Of the three compression tools, gzip is the oldest and provides the least compression, bzip2 provides improved compression, and xz is the newest and provides the best compression. Typically, files compressed with these utilities have .gz, .bz2, or .xz filename extensions, respectively. Compressed archives sometimes use their own special extensions, such as .tgz for a gzip-compressed tarball or .tbz for one compressed with bzip2.

> The xz compression tool has gained so much popularity, the Linux kernel is now compressed with it. 


The tar utility is easy to use but very powerful. The basic syntax for the tar command is as follows:
```tar [options] destination/tar_file_name files-to-archive```

For example, a compressed archive can be made up of a subdirectory within your home directory. In the same tar command, the archive can be sent to a USB fl ash drive mounted at /media/pen. Also, the tar command’s abbreviated commands and qualifi ers (options) can be grouped together for simplicity:
`
```sh
$ tar cvfz /media/pen/my-work.tgz ~/my-work
tar: Removing leading '/' from member names
/home/Christine/my-work/
/home/Christine/my-work/punch_list.txt
/home/Christine/my-work/project_a354
/home/Christine/my-work/project_m1321
/home/Christine/my-work/project_c923
$
$ ls -l /media/pen/my-work.tgz
-rw-rw-r--. 1 Christine Users 780 Sep 25 10:44 my-work.tgz
```