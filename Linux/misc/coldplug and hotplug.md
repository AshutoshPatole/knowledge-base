
Coldplug and Hotplug devices :

Whenever you deal with hardware, you should keep in mind a distinction between two device types: coldplug and hotplug. These device types differ depending on whether they can be physically attached and detached when the computer is turned on (that is, “hot”), versus only when it’s turned off (“cold”).

Traditionally, components that are internal to the computer, such as the CPU, memory, PCI cards, and hard disks, have been coldplug devices. A hotplug variant of PCI, however, has been developed and is used on some computers—mainly on servers and other systems that can’t afford the downtime required to install or remove a device. 

Modern external devices, such as Ethernet, USB, and IEEE-1394 devices, are hotplug; you can attach and detach such devices as you see fit. These devices rely on specialized Linux software to detect the changes to the system as they’re attached and detached. Several utilities help in managing hotplug devices:

**Sysfs The sysfs virtual filesystem**, mounted at /sys, exports information about devices so that user-space utilities can access the information.

>[!note]
> A user space program is one that runs as an ordinary program, whether it runs as an ordinary user or as root. This contrasts with kernel space code, which runs as part of the kernel. Typically, only the kernel (and hence kernel-space code) can communicate directly with hardware. User-space programs are the ultimate users of hardware, though. Traditionally, the /dev filesystem has provided the main means of interface between user-space programs and hardware; however, the tools described here help expand on this access, particularly in ways that are useful for hotplug devices.

**HAL Daemon The Hardware Abstraction Layer (HAL) Daemon**, or hald, is a user-space program that runs at all times (that is, as a daemon) and provides other user-space programs with information about available hardware.

**D-Bus The Desktop Bus (D-Bus)** provides a further abstraction of hardware information access. Like hald, D-Bus runs as a daemon. D-Bus enables processes to communicate with each other as well as to register to be notified of events, both by other processes and by hardware (such as the availability of a new USB device).

**udev** Traditionally, Linux has created device nodes as conventional files in the /dev directory tree. The existence of hotplug devices and various other issues, however, have motivated the creation of udev: a virtual filesystem, mounted at /dev, which creates dynamic device fi les as drivers are loaded and unloaded. You can configure udev through files in /etc/udev, but the standard configuration is usually sufficient for common hardware.