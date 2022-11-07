# System Security Services Daemon

Linux [[command]]

The System Security Services Daemon (SSSD) provides access to different identity and authentication providers. SSSD is an intermediary between local clients and any configured data store. The local clients connect to SSSD and then SSSD contacts the external providers. 

- *Reducing the load on identification/authentication servers*. Rather than having every client service attempt to contact the identification server directly, all of the local clients can contact SSSD which can connect to the identification server or check its cache.

- *Permitting offline authentication*. SSSD can optionally keep a cache of user identities and credentials that it retrieves from remote services. This allows users to authenticate to resources successfully, even if the remote identification server is offline or the local machine is offline.
- *Using a single user account*. Remote users frequently have two (or even more) user accounts, such as one for their local system and one for the organizational system. This is necessary to connect to a virtual private network (VPN). Because SSSD supports caching and offline authentication, remote users can connect to network resources simply by authenticating to their local machine and then SSSD maintains their network credentials.

## SSSD Configuration
SSSD services and domains are configured in a .conf file. The default file is `/etc/sssd/sssd.conf`, although alternative files can be passed to SSSD by using the -c option with the sssd command:

```bash
$ sssd -c /etc/sssd/customfile.conf
```
Both services and domains are configured individually, in separate sections on the configuration identified by [type/name] divisions, such as [domain/LDAP]. The configuration file uses simple key = value lines to set the configuration. Comment lines are set by either a hash sign (#) or a semicolon (;)

```
[section]
# Comment line
key1 = val1
key10 = val1,val2
```

SSSD currently provides several services:
- _A Name Service Switch (NSS)_ provider service that answers name service requests from the `sssd_nss` module. This is configured in the [nss] section of the SSSD configuration.
- _A PAM provider_ service that manages a PAM conversation through the sssd_pam module. This is configured in the [pam] section of the configuration.
- *monitor*, a special service that monitors and starts or restarts all other SSSD services. Its options are specified in the [sssd] section of the `/etc/sssd/sssd.conf` configuration file.

### Configuring NSS Services
SSSD provides an NSS module, `sssd_nss`, which instructs the system to use SSSD to retrieve user information. The NSS configuration must include a reference to the SSSD module, and then the SSSD configuration sets how SSSD interacts with NSS.

The Name Service Switch (NSS) provides a central configuration for services to look up a number of configuration and name resolution services. NSS provides one method of mapping system identities and services with configuration sources.
SSSD works with NSS as a provider services for several types of NSS maps:
- Passwords (`passwd`)
- User groups (`shadow`)
- Groups (`groups`)
- Netgroups (`netgroups`)

NSS can use multiple identity and configuration providers for any and all of its service maps. The default is to use system files for services; for SSSD to be included, the nss_sss module has to be included for the desired service type.

```bash
[captain@marvel-studios ~]$ sudo yum install authconfig -y
Last metadata expiration check: 2:44:17 ago on Sun 20 Mar 2022 10:18:48 AM GMT.
Dependencies resolved.
==========================================================================================
 Package                  Architecture   Version               Repository            Size
==========================================================================================
Installing:
 authselect-compat        aarch64        1.2.2-3.el8           ol8_appstream         38 k

Transaction Summary
==========================================================================================
Install  1 Package

Total download size: 38 k
Installed size: 82 k
Is this ok [y/N]: y
Downloading Packages:
authselect-compat-1.2.2-3.el8.aarch64.rpm                 792 kB/s |  38 kB     00:00
------------------------------------------------------------------------------------------
Total                                                     758 kB/s |  38 kB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                  1/1
  Installing       : authselect-compat-1.2.2-3.el8.aarch64                            1/1
  Running scriptlet: authselect-compat-1.2.2-3.el8.aarch64                            1/1
  Verifying        : authselect-compat-1.2.2-3.el8.aarch64                            1/1

Installed:
  authselect-compat-1.2.2-3.el8.aarch64

Complete!
```

```bash
[captain@marvel-studios ~]$ authconfig --enablesssd --update
Running authconfig compatibility tool.
The purpose of this tool is to enable authentication against chosen services with authselect and minimum configuration. It does not provide all capabilities of authconfig.

IMPORTANT: authconfig is replaced by authselect, please update your scripts.
See man authselect-migration(7) to help you with migration to authselect

Executing: /usr/bin/authselect check
Executing: /usr/bin/authselect select sssd --force
Executing: /usr/bin/systemctl enable sssd.service
Executing: /usr/bin/systemctl stop sssd.service
Executing: /usr/bin/systemctl start sssd.service
```
This automatically configures the password, shadow, group, and netgroups services maps to use the SSSD module:

```
passwd:     files sss
shadow:     files sss
group:      files sss

netgroup:   files sss
```
#### Configuring SSSD to Work with NSS
The options and configuration that SSSD uses to service NSS requests are configured in the SSSD configuration file, in the [nss] services section. Open the sssd.conf file and add these lines
```bash title="sssd.conf"
[sssd]
config_file_version = 2
reconnection_retries = 3
sbus_timeout = 30
services = nss  # add nss as a service

###

[nss]
filter_groups = root
filter_users = root
reconnection_retries = 3
entry_cache_timeout = 300
entry_cache_nowait_percentage = 75
```

and then restart the sssd service
```bash
[captain@marvel-studios ~]$ sudo systemctl restart sssd
```
### Configuring PAM service
SSSD provides a PAM module, `sssd_pam`, which instructs the system to use SSSD to retrieve user information. The PAM configuration must include a reference to the SSSD module, and then the SSSD configuration sets how SSSD interacts with PAM.

> [!attention]
 A mistake in the PAM configuration file can lock users out of the system completely. Always back up the configuration files before performing any changes, and keep a session open so that any changes can be reverted.


To configure the PAM service:
remove the `/etc/pam.d/system-auth` symlink
```bash
[root@server ~]$ rm /etc/pam.d/system-auth
rm: remove symbolic link `/etc/pam.d/system-auth'? y
```

Create a new `/etc/pam.d/system-auth-local` file. One easy way to do this is simply to copy the `/etc/pam.d/system-auth-ac` file.
```bash
[root@server ~]$ cp /etc/pam.d/system-auth-ac /etc/pam.d/system-auth-local
```
Create a new symlink between the `/etc/pam.d/system-auth-local` file and `/etc/pam.d/system-auth`.
```bash
[root@server ~]$ ln -s /etc/pam.d/system-auth-local /etc/pam.d/system-auth
```
Edit the `/etc/pam.d/system-auth-local` file, and add all of the SSSD modules to the PAM configuration.
```bash
#%PAM-1.0
...
auth        sufficient    pam_sss.so use_first_pass
auth        required      pam_deny.so

...
account [default=bad success=ok user_unknown=ignore] pam_sss.so
account     required      pam_permit.so

...
password    sufficient    pam_sss.so use_authtok
password    required      pam_deny.so

...
session     sufficient    pam_sss.so
session     required      pam_unix.so
```
4. Open sssd.conf and add pam as a service.
```bash
[sssd]
config_file_version = 2
reconnection_retries = 3
sbus_timeout = 30
services = nss, pam  # add pam here
```
Change or add any parameters in [pam] section
```bash
[pam]
reconnection_retries = 3
offline_credentials_expiration = 2
offline_failed_login_attempts = 3
offline_failed_login_delay = 5
```
then restart the sssd service.

## Creating domains
SSSD recognizes domains, which are associated with the different identity servers. Domains are a combination of an identity provider and an authentication method. SSSD works with LDAP identity providers (including OpenLDAP, Red Hat Directory Server, and Microsoft Active Directory) and can use native LDAP authentication or Kerberos authentication.

As long as they belong to different domains, SSSD can recognize different users with the same username. For example, SSSD can successfully authenticate both captain in the ldap.example.com domain and captain in the ldap.otherexample.com domain. SSSD allows requests using fully-qualified domain names, so requesting information for captain@ldap.example.com returns the proper user account. Specifying only the username returns the user for whichever domain comes first in the lookup order.

### General Rules and Options for Configuring a Domain

A domain configuration defines the identity provider, the authentication provider, and any specific configuration to access the information in those providers. There are two types of identity providers — LDAP and proxy —three types of authentication providers — LDAP, Kerberos, and proxy. The identity and authentication providers can be configured in any combination in a domain entry.

```
domains = LOCAL,Name

[domain/Name]
id_provider = type
auth_provider = type
provider_specific = value
global = value
```
_global_ attributes are available to any type of domain, such as cache and time out settings. Each identity and authentication provider has its own set of required and optional configuration parameters.

#linux  #security 