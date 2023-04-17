---
layout: post
title: "Working with Proxmox"
tags: [proxmox, homelab]
---

A comprehensive guide on working with Proxmox

As you venture into using Proxmox on your home server, you may come across various options and features that the platform offers. In this blog post, we will explore the different aspects of Proxmox and provide a comprehensive guide to help you get started.

## Containers vs VMs

Proxmox offers support for both containers and VMs, which provide different approaches to running multiple instances on a single host. VMs emulate complete systems, including the operating system (OS) and hardware, while containers share the host OS and are more lightweight.

### VMs

VMs run on a hypervisor, such as Proxmox's built-in KVM, which allows multiple VMs with different operating systems to coexist on a single physical host. Each VM has its own kernel, memory, and virtualized hardware resources. VMs provide strong isolation, but this comes at the cost of increased resource consumption and longer startup times compared to containers.

### Containers

Containers, like LXC and Docker, are more resource-efficient because they share the host OS kernel and run directly on the host system. Containers are isolated from each other and the host, providing a secure and efficient environment for deploying applications. Proxmox supports LXC containers out-of-the-box, offering a lightweight alternative to VMs for certain use cases.

### Launching a Windows VM in Proxmox

To launch a Windows VM in Proxmox, follow these steps:

1. Download the Windows Server ISO that also includes the drivers.
2. Add the hard drive ISO to the VM.
3. You will have two drives (one for the installer and the other one for the drivers).

## Templates

Templates in Proxmox VE are a powerful feature that allows you to create pre-configured base images for VMs and containers. By using templates, you can significantly reduce the time and effort required to deploy new instances, as well as ensure consistent configurations across multiple deployments.

Templates can include installed software, custom configurations, and any other settings that you want to apply to new instances created from the template. Once a template is created, you can quickly clone it to create new VMs or containers with the same base configuration.

### VM Templates

To create a VM template in Proxmox VE, follow these steps:

1. Prepare a VM with the desired configuration, software, and settings.
2. Update the VM to the latest version
```sh
    apt update && apt dist-upgrade
```
3. Remove any unique identifiers, such as SSH host keys
```sh
    rm /etc/ssh/ssh_host_*
```
4. Clean up unnecessary files and packages
```sh
    apt clean
    apt autoremove
    apt-get install qemu-guest-agent
```
5.  Remove the `machine_id` 
```sh
    truncate -s 0 /etc/machine-id
```
6. In the Proxmox web interface, select the VM and click "Convert to Template."

Keep in mind that converting a VM to a template is a destructive operation. Ensure that you have removed any sensitive data and installation media before proceeding.

To create a new VM from a template, simply select the template in the Proxmox web interface and click "Clone." You can customize the new VM's settings, such as hostname and network configuration, during the cloning process.

### Container Templates

Creating container templates in Proxmox VE is similar to creating VM templates. To create a container template, follow these steps:

1. Prepare a container with the desired configuration, software, and settings.
2. Update the container to the latest version.
3. Remove any unique identifiers, such as SSH host keys
```sh
    sudo rm /etc/ssh/ssh_host_* && sudo dpkg-reconfigure openssh-server`
```
4. In the Proxmox web interface, select the container and click `Convert to Template`

Note that cloud-init is not supported for container templates, so you'll need to manually configure settings like hostname and network configuration when cloning a container template.

### QEMU-guest-agent

The QEMU-guest-agent is a helper daemon installed in the guest. It is used to exchange information between the host and guest and execute commands in the guest. In Proxmox VE, the QEMU-guest-agent is used mainly for two purposes:

1. To properly shut down the guest, instead of relying on ACPI commands or Windows policies.
2. To freeze the guest filesystem when making a backup (on Windows, use the Volume Shadow Copy Service VSS).

### Using Cloud-init

Cloud-init is a tool that provides various features for customizing instances, including setting the hostname, configuring network interfaces, setting up SSH keys, and running scripts or commands on the first boot. Cloud-init can also be used to install packages, configure software, and perform other system administration tasks.

One of the main benefits of using cloud-init is that it enables the automation of instance configuration, which can save time and reduce errors compared to manual configuration. Additionally, cloud-init is highly customizable, making it suitable for a wide range of use cases.

Cloud-init might already be installed on some servers. To install it, use `apt install cloud-init`. The configuration file for cloud-init is located at `/etc/cloud/cloud.cfg`.

## User Management

Proxmox supports two types of user accounts: PAM (Pluggable Authentication Modules) users and Proxmox VE (PVE) users. PAM users are regular Linux users with the ability to log in via SSH, while PVE users are specific to the Proxmox web interface and are used for managing the Proxmox environment.

To effectively manage users in Proxmox, create groups with specific permissions and assign users to these groups. This approach simplifies the process of granting and revoking permissions for users in your environment. For example, you can create a group called "Admins" with full administrative permissions and another group called "Developers" with limited permissions to manage specific VMs or containers.

### Roles and Permissions

Roles are predefined sets of permissions that can be assigned to users or groups. Proxmox includes several built-in roles, such as Administrator, PVEAuditor, and PVEVMUser, each with a specific set of permissions. You can also create custom roles to tailor permissions to your organization's requirements.

### Groups

Groups simplify user management by allowing you to assign permissions to multiple users at once. You can create groups, add users to them, and assign roles to the groups. Users inherit the permissions of the roles assigned to their groups.

### Realms

A realm is a logical partition of a computer network or system that defines a boundary for security policies, authentication, and authorization. Realms are often used in authentication systems, such as the Kerberos protocol, to identify a set of objects that share the same authentication policy. These objects can include users, services, and hosts. In Proxmox, you can have the same user in both PAM and PVE realms.

## CLI

Proxmox provides a powerful CLI for managing your virtualization environment. The CLI allows you to perform various tasks, such as creating, starting, stopping, and managing VMs and containers, configuring networking and storage, and managing users and permissions. Common Proxmox CLI utilities include:

- `pvesh`: A shell for interacting with the Proxmox API, allowing you to perform most tasks available in the web interface.
- `qm`: A utility for managing QEMU-based VMs, providing commands for creating, configuring, and controlling VMs.
- `pct`: A utility for managing LXC containers, offering similar functionality to `qm` but for containers instead of VMs.

### Most used CLI commands

Basic VM commands:

```bash
    qm list # list VMs
    qm start <VMID> # start VM by ID
    qm shutdown <VMID>
    qm reboot <VMID>
    qm reset <VMID> # hard reset VM
    qm stop <VMID>
```

Config VM commands:

```bash
    qm set --onboot 0 <VMID> # set onboot to 0
    qm config <VMID> # list settings
```

Basic PCT (container) commands:

```bash
    pct list
    pct start <VMID>
    pct shutdown <VMID>
    pct reboot <VMID>
    pct reset <VMID>
    pct stop <VMID>

    pct enter <VMID> # enter the container
```

Config PCT commands:

```bash
    pct set --onboot 1 <VMID>
    pct config <VMID> # list configurable settings
```

## Networking

Proxmox offers versatile networking capabilities, allowing you to create and manage various network configurations for your VMs and containers. Some of the key networking features include:

- Virtual network switches (Linux bridges or Open vSwitch): These switches enable communication between VMs, containers, and the host system, as well as external networks.
- Network interface cards (NICs): Proxmox supports various virtual NIC types, such as VirtIO, E1000, and VMXNET3, to provide optimal performance and compatibility with different guest operating systems.
- VLANs and VXLANs: Proxmox allows you to create and manage VLANs and VXLANs, offering network segmentation and isolation for enhanced security and organization.
- Network Address Translation (NAT): Proxmox supports NAT, enabling you to share a single public IP address among multiple VMs and containers.
- IP aliasing: Proxmox permits IP aliasing, allowing you to assign multiple IP addresses to a single network interface.

## Integrated Firewall

Do not turn on the firewall before adding rules. If you add firewall rules at the data center level, they will propagate to all PVEs and other objects. Otherwise, you have to add rules individually for each object. The `/etc/pve/firewall/cluster.fw` file allows you to enable or disable the firewall for the entire cluster.

To allow only one IP address to ping (ICMP traffic), refer to the Proxmox documentation and follow the steps provided.

## Managing Storage in Proxmox

To manage storage in Proxmox, navigate to the "Datacenter" view and select "Storage." In this section, you can add, edit, or remove storage configurations for your Proxmox environment.

Proxmox provides extensive storage options, including:

- Local storage: Proxmox supports various local storage options like LVM, ZFS, ext4, and xfs, offering flexibility in how you manage and allocate storage resources on your host system.
- Network storage: Proxmox can integrate with network storage solutions like NFS, iSCSI, Ceph, and GlusterFS, allowing you to leverage shared and distributed storage architectures for improved performance and redundancy.
- Backups and snapshots: Proxmox enables you to create and manage backups and snapshots of your VMs and containers. This feature provides a robust method for data protection and disaster recovery, ensuring that your critical data remains secure and accessible even in the event of hardware failures or other issues.

By understanding and leveraging the various storage options available in Proxmox, you can create a more resilient, scalable, and efficient infrastructure for your VMs and containers.

### Best Practices for Storage Configuration

When configuring storage in Proxmox, consider the following best practices:

1. Separate storage for VMs and containers from the Proxmox host system to ensure isolation and better resource management.
2. Utilize shared storage for VMs and containers that require high availability, allowing for seamless failover and load balancing.
3. Opt for storage options that match your performance, scalability, and redundancy requirements, considering factors such as throughput, latency, and data protection.

## High Availability

Proxmox provides high availability (HA) features, ensuring that VMs and containers remain operational in the event of hardware failures or other issues. HA is achieved using a combination of Proxmox VE HA Manager and Corosync, which manages cluster communication and consensus.

To enable high availability for your VMs and containers, follow these steps:

1. Set up a Proxmox cluster with a minimum of three nodes.
2. Configure shared storage for VMs and containers that require high availability.
3. Enable HA for individual VMs or containers by configuring the HA settings in their respective configuration dialogs.

Keep in mind that HA relies on a majority quorum, which means that at least 50% of the cluster nodes must be operational for the cluster to function properly. This is why a minimum of three nodes is recommended.

## Backup and Restore

Proxmox VE offers a comprehensive backup and restore solution for VMs and containers. Regular backups are essential for safeguarding your data and ensuring that you can quickly recover from hardware failures, data corruption, or other unforeseen events.

### To create a backup, follow these steps:

1. Navigate to the `Datacenter` view and select `Backup`.
2. Choose the storage location for your backups.
3. Schedule a backup by setting the time, frequency, and days of the week.
4. Select the VMs or containers to be backed up and choose the backup mode (snapshot, suspend, or stop).
5. Configure additional backup options, such as compression and encryption.

### To restore a backup, follow these steps:

1. Navigate to the `Datacenter` view and select `Backup`.
2. Choose the storage location where your backups are stored.
3. Select the backup file you want to restore.
4. Choose the target node and storage for the restored VM or container.
5. Configure additional restore options, such as VM ID and network settings.

Keep in mind that restoring a backup will overwrite any existing data on the target VM or container.

## Monitoring and Logging

Proxmox VE provides built-in monitoring and logging features to help you keep track of the performance and health of your virtual infrastructure. Monitoring tools such as graphs and charts are available for various metrics, including CPU usage, memory usage, disk activity, and network traffic.

### To access monitoring and logging features, follow these steps:

1. Navigate to the `Datacenter` view and select `Monitoring`.
2. Choose the node, VM, or container you want to monitor.
3. Select the metric you want to view (e.g., CPU usage, memory usage, etc.).
4. Adjust the time range and other display options as needed.

In addition to monitoring, Proxmox VE also provides a centralized logging system that collects logs from all nodes, VMs, and containers in the cluster. This can be useful for troubleshooting and identifying issues across your virtual infrastructure.

### To access logs, follow these steps:

1. Navigate to the `Datacenter` view and select `Logs`.
2. Choose the node, VM, or container you want to view logs for.
3. Filter the logs by type, priority, or other criteria as needed.

By utilizing Proxmox VE's monitoring and logging features, you can stay informed about the health and performance of your virtual infrastructure and quickly address any issues that may arise.

With the knowledge of Proxmox's capabilities in backup and restore, monitoring, and logging, you can ensure a robust and well-maintained virtual infrastructure. These features, combined with the previously discussed topics, make Proxmox an indispensable tool for managing virtual environments.
