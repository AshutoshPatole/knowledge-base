

Samba is an open source implementation of the Server Message Block (SMB) protocol. It allows the networking of Windows, [[filesystem]], UNIX, and other operating systems together, enabling access to Windows-based file and printer shares. Samba's use of SMB allows it to appear as a Windows server to Windows clients.

## Samba Daemon and related services

Samba is comprised of three daemons (`smbd`, `nmbd`, and `winbindd`). Two services (smb and windbind) control how the daemons are started, stopped, and other service-related features.

### smbd
The smbd server daemon provides file sharing and printing services to Windows clients. In addition, it is responsible for user authentication, resource locking, and data sharing through the SMB protocol. The default ports on which the server listens for SMB traffic are TCP ports **139** and **445**.

The smbd daemon is controlled by the smb service.

### nmbd
nmbd
The nmbd server daemon understands and replies to NetBIOS name service requests such as those produced by SMB/CIFS in Windows-based systems. These systems include Windows 95/98/ME, Windows NT, Windows 2000, Windows XP, and LanManager clients. It also participates in the browsing protocols that make up the Windows Network Neighborhood view. The default port that the server listens to for NMB traffic is UDP port 137.

The nmbd daemon is controlled by the smb service.

### winbindd
The winbind service resolves user and group information on a server running Windows NT 2000 or Windows Server 2003. This makes Windows user / group information understandable by UNIX platforms. This is achieved by using Microsoft RPC calls, Pluggable Authentication Modules (PAM), and the Name Service Switch (NSS). This allows Windows NT domain users to appear and operate as UNIX users on a UNIX machine. Though bundled with the Samba distribution, the winbind service is controlled separately from the smb service.

>[!info]
 The Name Service Switch (NSS) framework was designed to let administrators specify which files or directory services to query to obtain information. For example, it's frequently used to specify whether a system should perform hostname lookups in /etc/hosts, NIS, or DNS.
>
 > CIFS - Common Internet File System

The winbindd daemon is controlled by the winbind service and does not require the smb service to be started in order to operate. Winbindd is also used when Samba is an Active Directory member, and may also be used on a Samba domain controller (to implement nested groups and/or interdomain trust). 

## Configuring a samba server

The default configuration file `/etc/samba/smb.conf` allows users to view their home directories as a Samba share. It also shares all printers configured for the system as Samba shared printers. In other words, you can attach a printer to the system and print to it from the Windows machines on your network.

> [!important]
If you change this configuration file, the changes do not take effect until you restart the Samba daemon with the command 
```bash
service smb restart
```
or 
```bash
systemctl start smb
```


To specify the Windows workgroup and a brief description of the Samba server, edit the following lines in your smb.conf file:
```bash
workgroup = WORKGROUPNAME
server string = BRIEF COMMENT ABOUT SERVER
```
Replace *WORKGROUPNAME* with the name of the Windows workgroup to which this machine should belong. The *BRIEF COMMENT ABOUT SERVER* is optional and is used as the Windows comment about the Samba system.

To create a Samba share directory on your Linux system, add the following section to your smb.conf file.
```
[sharename]
comment = Insert a comment here
path = /home/share/
valid users = thor captain
public = no
writable = yes
printable = no
create mask = 0765
```

The above example allows the users thor and captain to read and write to the directory /home/share, on the Samba server, from a Samba client.

> [!tip]
Encrypted passwords are enabled by default because it is more secure to do so. To create a user with an encrypted password, use the command

```bash
smbpasswd -a <username> 
```


#networking #filesystem #storage 