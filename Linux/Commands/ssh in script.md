[[command]]
### Setting up demo

To copy this ssh from local to remote machine use `scp`(Secure Copy Protocol) or `ssh-copy-id`.
It is always better to use the later one but I am going to put an example for both.

```bash title="scp"
[thor@marvel-studios ~]$ scp ~/.ssh/id_rsa.pub username@ip-address:~/.ssh/authorized_keys
```
> [!note]
You can also use domain name instead of ip-address as long as the ip address is bound to the domain name.


```bash title="ssh-copy-id"
[thor@marvel-studios ~]$ ssh-copy-id -i ~/.ssh/id_rsa.pub username@domain-name
```
Once it is done you can login into your remote server as
```bash
[thor@marvel-studios ~]$ ssh -i ~/.ssh/id_rsa username@domain-name
# For the first time it will ask you to type YES to save the credentials
# in known hosts file inside .ssh folder
[username@domain-name ~] $  
```
### Using SSH in script
```bash
[thor@marvel-studios ~]$ ssh -i ~/.ssh/id_rsa username@domain-name 'ls; pwd; whoami; ls -la | grep .*txt;'
sample.sh hello.sh sample.txt  # ls
/home/username                 # pwd
username                       # whoami
sample.txt                     # ls -la | grep .*txt
```

This can become pretty unmaintainable if you have 10's of commands to run after ssh-ing into the machine.
So we will create a script here which will store all the commands that needs to be run in remote machine

```bash title="mycmd.sh"

#!/bin/bash
ls
pwd
whoami
ls -la | grep .*txt
# ...
# ...
```
and then
```bash
[thor@marvel-studios ~]$ cat mycmd.sh | ssh -i ~/.ssh/id_rsa username@domain-name

Pseudo-terminal will not be allocated because stdin is not a terminal.
Activate the web console with: systemctl enable --now cockpit.socket

sample.sh hello.sh sample.txt  # ls
/home/username                 # pwd
username                       # whoami
sample.txt                     # ls -la | grep .*txt
```
You can also use this inside a script.
