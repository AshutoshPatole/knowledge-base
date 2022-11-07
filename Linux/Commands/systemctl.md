[[command]]
systemd:
Systems with systemd start up faster than those with earlier init methods. This is largely because it replaces a serialized set of steps with aggressive parallelization techniques, which permits multiple services to be initiated simultaneously.

Complicated startup shell scripts are replaced with simpler configuration files, which enumerate what has to be done before a service is started, how to execute service startup, and what conditions the service should indicate have been accomplished when startup is finished. One thing to note is that /sbin/init now just points to /lib/systemd/systemd; i.e. systemd takes over the init process.

One systemd command (systemctl) is used for most basic tasks. While we have not yet talked about working at the command line, here is a brief listing of its use:

Starting, stopping, restarting a service (using httpd, the Apache web server, as an example) on a currently running system:

```
$ sudo systemctl start|stop|restart httpd.service
```

Enabling or disabling a system service from starting up at system boot:

```
$ sudo systemctl enable|disable httpd.service
```

In most cases, the .service can be omitted