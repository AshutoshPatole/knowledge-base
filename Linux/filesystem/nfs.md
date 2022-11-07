Network File System (NFS) is a distributed file system protocol originally developed by Sun Microsystems (Sun), available in [[filesystem]], allowing a user on a client computer to access files over a computer network much like local storage is accessed. NFS, like many other protocols, builds on the Open Network Computing Remote Procedure Call (ONC RPC) system. NFS is an open IETF standard defined in a Request for Comments (RFC), allowing anyone to implement the protocol.

---
## Install NFS
Install NFS Client on Ubuntu
```
sudo apt -y update
sudo apt -y install nfs-common
```



A Network File System (NFS) allows remote hosts to mount file systems over a network and interact with those file systems as though they are mounted locally. 
## How It Works
Currently, there are three versions of NFS.
- NFS version 2 (NFSv2) is older and is widely supported.
- NFS version 3 (NFSv3) has more features, including 64bit file handles, Safe Async writes and more robust error handling. 
- NFS version 4 (NFSv4) works through firewalls and on the Internet, no longer requires portmapper, supports ACLs, and utilizes stateful operations. 

All versions of NFS can use Transmission Control Protocol (TCP) running over an IP network, with NFSv4 requiring it. NFSv2 and NFSv3 can use the User Datagram Protocol (UDP) running over an IP network to provide a stateless network connection between the client and server.

NFSv4 listens on the well-known TCP port **2049**.


After the client is granted access by TCP wrappers, the NFS server refers to its configuration file, /etc/exports, to determine whether the client is allowed to access any of the exported file systems. Once access is granted, all file and directory operations are available to the user.
> [!important]
In order for NFS to work with a default installation of Red Hat Enterprise Linux with a firewall enabled, IPTables with the default TCP port 2049 must be configured. Without proper IPTables configuration, NFS does not function properly.


## NFS Client Configuration
NFS shares are mounted on the client side using the mount command. The format of the command is as follows:
```bash
mount -t <nfs-type> -o <options> <host>:</remote/export> </local/directory>
```
- Replace `<nfs-type>` with either nfs for NFSv2 or NFSv3 servers, or nfs4 for NFSv4 servers.
- Replace `<host>` with the remote host, `</remote/export>` with the remote directory being mounted.
- `</local/directory>` with the local directory where the remote file system is to be mounted.

If accessing an NFS share by manually issuing the mount command, the file system must be remounted manually after the system is rebooted. RHEL offers two methods for mounting remote file systems automatically at boot time: 
- the **/etc/fstab** file.
- the **autofs** service.

## Mounting NFS File Systems using /etc/fstab
An alternate way to mount an NFS share from another machine is to add a line to the /etc/fstab file. The /etc/fstab file is referenced by the netfs service at boot time, so lines referencing NFS shares have the same effect as manually typing the mount command during the boot process. Each line in this file must state the hostname of the NFS server, the directory on the server being exported, and the directory on the local machine where the NFS share is to be mounted. You must be root to modify the /etc/fstab file.

The general syntax for a line in /etc/fstab is as follows:
```bash
<server>:</remote/export> </local/directory> <nfs-type> <options> 0 0
```
The following is a sample /etc/fstab line to mount an NFS export:
```bash
server:/usr/local/pub    /pub   nfs    defaults 0 0
```

## Autofs
One drawback to using /etc/fstab is that, regardless of how infrequently a user accesses the NFS mounted file system, the system must dedicate resources to keep the mounted file system in place. This is not a problem with one or two mounts, but when the system is maintaining mounts to many systems at one time, overall system performance can be affected. An alternative to /etc/fstab is to use the kernel-based automount utility. An automounter consists of two components. One is a kernel module that implements a file system, while the other is a user-space daemon that performs all of the other functions. The automount utility can mount and unmount NFS file systems automatically (on demand mounting) therefore saving system resources. The automount utility can be used to mount other file systems including AFS, SMBFS, CIFS and local file systems.

autofs uses **/etc/auto.master** (master map) as its default primary configuration file. This can be changed to use another supported network source and name using the autofs configuration (in **/etc/sysconfig/autofs**) in conjunction with the Name Service Switch mechanism. 

### Autofs configuration
The primary configuration file for the automounter is /etc/auto.master, also referred to as the master map which may be changed. The master map lists autofs-controlled mount points on the system, and their corresponding configuration files or network sources known as automount maps. The format of the master map is as follows:
```bash
<mount-point> <map-name> <options>
```

where:
- `mount-point` is the autofs mount point such as /home.
- `map-name` is the name of a map source which contains a list of mount points, and the file system location from which those mount points should be mounted. The syntax for a map entry is described below.
- `options` if supplied, will apply to all entries in the given map provided they don't themselves have options specified. This behavior is different from autofs version 4 where the options where cumulative. This has been changed to meet our primary goal of mixed environment compatibility.

The following is a sample /etc/auto.master file:
```bash
thor@marvel-studios ~]$ cat /etc/auto.master
/home /etc/auto.misc
```
The general format of maps is similar to the master map, however the *"options"* appear between the mount point and the location instead of at the end of the entry as in the master map:
```bash
<mount-point>   [<options>]   <location>
```

## The `/etc/exports` configuration file
The /etc/exports file controls which file systems are exported to remote hosts and specifies options. Blank lines are ignored, comments can be made by starting a line with the hash mark `(#)`, and long lines can be wrapped with a backslash `(\)`. Each exported file system should be on its own individual line, and any lists of authorized hosts placed after an exported file system must be separated by space characters. Options for each of the hosts must be placed in parentheses directly after the host identifier, without any spaces separating the host and the first parenthesis.

A line for an exported file system has the following structure:
```bash
<export> <host1>(<options>) <hostN>(<options>)...
```
In this structure, replace `<export>` with the directory being exported, replace `<host1>` with the host or network to which the export is being shared, and replace `<options>` with the options for that host or network. Additional hosts can be specified in a space separated list.
The following methods can be used to specify host names:

- single host — Where one particular host is specified with a fully qualified domain name, hostname, or IP address.

- wildcards — Where a * or ? character is used to take into account a grouping of fully qualified domain names that match a particular string of letters. Wildcards should not be used with IP addresses; however, it is possible for them to work accidentally if reverse DNS lookups fail.
Be careful when using wildcards with fully qualified domain names, as they tend to be more exact than expected. For example, the use of *.ashutoshpatole.me as a wildcard allows live.ashutoshpatole to access an exported file system, but not wetty.live.ashutoshpatole.me. To match both possibilities both `*.ashutoshpatole.me` and `*.*.ashutoshpatole.me` must be specified.

- IP networks — Allows the matching of hosts based on their IP addresses within a larger network. For example, 192.168.0.0/28 allows the first 16 IP addresses, from 192.168.0.0 to 192.168.0.15, to access the exported file system, but not 192.168.0.16 and higher.

- netgroups — Permits an NIS netgroup name, written as @`<group-name>`, to be used. This effectively puts the NIS server in charge of access control for this exported file system, where users can be added and removed from an NIS group without affecting /etc/exports.

### Options in /etc/exports
- **ro** — Mounts of the exported file system are read-only. Remote hosts are not able to make changes to the data shared on the file system. To allow hosts to make changes to the file system, the read/write (rw) option must be specified.

- **wdelay** — Causes the NFS server to delay writing to the disk if it suspects another write request is imminent. This can improve performance by reducing the number of times the disk must be accessed by separate write commands, reducing write overhead. The no_wdelay option turns off this feature, but is only available when using the sync option.

- **root_squash** — Prevents root users connected remotely from having root privileges and assigns them the user ID for the user nfsnobody. This effectively "squashes" the power of the remote root user to the lowest local user, preventing unauthorized alteration of files on the remote server. Alternatively, the no_root_squash option turns off root squashing. To squash every remote user, including root, use the all_squash option. To specify the user and group IDs to use with remote users from a particular host, use the anonuid and anongid options, respectively. In this case, a special user account can be created for remote NFS users to share and specify (anonuid=`<uid-value>`,anongid=`<gid-value>`), where `<uid-value>` is the user ID number and `<gid-value>` is the group ID number.

>[!important]
By default, access control lists (ACLs) are supported by NFS under RHEL. To disable this, specify the **no_acl** option when exporting the file system.


>[!warning]
The format of the /etc/exports file is very precise, particularly in regards to use of the space character. Remember to always separate exported file systems from hosts and hosts from one another with a space character. However, there should be no other space characters in the file except on comment lines.
For example, the following two lines do not mean the same thing:
```bash
/home live.ashutoshpatole.me(rw)
/home live.ashutoshpatole.me (rw)
```
The first line allows only users from live.ashutoshpatole.me read/write access to the /home directory. The second line allows users from live.ashutoshpatole.me to mount the directory as read-only (the default), while the rest of the world can mount it read/write.


## The exportfs Command

Every file system being exported to remote users via NFS, as well as the access level for those file systems, are listed in the /etc/exports file. When the nfs service starts, the /usr/sbin/exportfs command launches and reads this file, passes control to rpc.mountd (if NFSv2 or NFSv3) for the actual mounting process, then to rpc.nfsd where the file systems are then available to remote users.
When issued manually, the /usr/sbin/exportfs command allows the root user to selectively export or unexport directories without restarting the NFS service. When given the proper options, the /usr/sbin/exportfs command writes the exported file systems to /var/lib/nfs/xtab. Since rpc.mountd refers to the xtab file when deciding access privileges to a file system, changes to the list of exported file systems take effect immediately.



## Server Configuration
### Configuration
*TEMP EXAMPLE*:
`/srv/nfs 192.168.1.2(rw,sync,no_root_squash,subtree_check)`

### root rw permissions
Note the **root_squash** mount option. This option is set by default and must be disabled if not wanted.
*Fix:* enable `no_root_squash`in the `/etc/exports` file and reload the permissions with `sudo exportfs -ra`

#storage #linux #filesystem 