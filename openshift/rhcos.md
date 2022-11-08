
## Why RHCOS ?

- **Based on RHEL**: The underlying operating system consists primarily of RHEL components. The same quality, security, and control measures that support RHEL also support RHCOS. 

- **Controlled immutability**: Although it contains RHEL components, RHCOS is designed to be managed more tightly than a default RHEL installation. When you set up your RHCOS machines, you can modify only a few system settings. This controlled immutability allows [[OpenShift]] Container Platform to store the latest state of RHCOS systems in the cluster so it is always able to create additional machines and perform updates based on the latest RHCOS configurations.

- **CRI-O container runtime**: Although RHCOS contains features for running the OCI- and libcontainer-formatted containers that Docker requires, it incorporates the CRI-O container engine instead of the Docker container engine. By focusing on features needed by Kubernetes platforms, such as OpenShift Container Platform, CRI-O can offer specific compatibility with different Kubernetes versions. CRI-O also offers a smaller footprint and reduced attack surface than is possible with container engines that offer a larger feature set. At the moment, CRI-O is the only engine available within OpenShift Container Platform clusters.

- **rpm-ostree upgrades**: RHCOS features transactional upgrades using the rpm-ostree system. Updates are delivered by means of container images and are part of the OpenShift Container Platform update process. When deployed, the container image is pulled, extracted, and written to disk, then the bootloader is modified to boot into the new version. The machine will reboot into the update in a rolling manner to ensure cluster capacity is minimally impacted.

- **Updated through the Machine Config Operator**: In OpenShift Container Platform, the Machine Config Operator handles operating system upgrades. Instead of upgrading individual packages, as is done with yum upgrades, rpm-ostree delivers upgrades of the OS as an atomic unit. The new OS deployment is staged during upgrades and goes into effect on the next reboot. If something goes wrong with the upgrade, a single rollback and reboot returns the system to the previous state. RHCOS upgrades in OpenShift Container Platform are performed during cluster updates.

## About Ignitions
Ignition is the utility that is used by RHCOS to manipulate disks during initial configuration. It completes common disk tasks, including partitioning disks, formatting partitions, writing files, and configuring users. On first boot, Ignition reads its configuration from the installation media or the location that you specify and applies the configuration to the machines.

Whether you are installing your cluster or adding machines to it, Ignition always performs the initial configuration of the OpenShift Container Platform cluster machines. Most of the actual system setup happens on each machine itself. For each machine, Ignition takes the RHCOS image and boots the RHCOS kernel. Options on the kernel command line, identify the type of deployment and the location of the Ignition-enabled initial Ram disk (initramfs).

### How Ignition works
To create machines by using Ignition, you need Ignition config files. The OpenShift Container Platform installation program creates the Ignition config files that you need to deploy your cluster. These files are based on the information that you provide to the installation program directly or through an install-config.yaml file.

The way that Ignition configures machines is similar to how tools like `cloud-init` or Linux Anaconda `kickstart` configure systems, but with some important differences:

- Ignition runs from an initial RAM disk that is separate from the system you are installing to. Because of that, Ignition can repartition disks, set up file systems, and perform other changes to the machine's permanent file system. In contrast, cloud-init runs as part of a machine's init system when the system boots, so making foundational changes to things like disk partitions cannot be done as easily. With cloud-init, it is also difficult to reconfigure the boot process while you are in the middle of the node's boot process.

- Ignition is meant to initialize systems, not change existing systems. After a machine initializes and the kernel is running from the installed system, the Machine Config Operator from the OpenShift Container Platform cluster completes all future machine configuration.

- After Ignition finishes configuring a machine, the kernel keeps running but discards the initial RAM disk and pivots to the installed system on disk. All of the new system services and other features start without requiring a system reboot.

- Because Ignition confirms that all new machines meet the declared configuration, you cannot have a partially-configured machine. If a machine''s setup fails, the initialization process does not finish, and Ignition does not start the new machine. Your cluster will never contain partially-configured machines. If Ignition cannot complete, the machine is not added to the cluster. You must add a new machine instead. This behavior prevents the difficult case of debugging a machine when the results of a failed configuration task are not known until something that depended on it fails at a later date.

- Because Ignition can start with a completely empty hard disk, it can do something cloud-init cannot do: set up systems on bare metal from scratch (using features such as `PXE boot`). In the bare metal case, the Ignition config is injected into the boot partition so Ignition can find it and configure the system correctly.

### The Ignition sequence
The Ignition process for an RHCOS machine in an OpenShift Container Platform cluster involves the following steps:

- The machine gets its Ignition config file. Control plane machines (also known as the master machines) get their Ignition config files from the bootstrap machine, and worker machines get Ignition config files from a control plane.

- Ignition creates disk partitions, file systems, directories, and links on the machine. It supports RAID arrays but **does not support LVM volumes**.

- Ignition mounts the root of the permanent file system to the `/sysroot` directory in the initramfs and starts working in that `/sysroot` directory.

- Ignition configures all defined file systems and sets them up to mount appropriately at runtime.

- Ignition runs systemd temporary files to populate required files in the `/var` directory.

- Ignition runs the Ignition config files to set up users, `systemd unit` files, and other configuration files.

- Ignition unmounts all components in the permanent system that were mounted in the initramfs.

- Ignition starts up new machine's init process which, in turn, starts up all other services on the machine that run during system boot.

The machine is then ready to join the cluster and does not require a reboot.