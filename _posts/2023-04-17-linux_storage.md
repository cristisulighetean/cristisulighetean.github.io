---
layout: post
title: "Storage volumes in Linux"
tags: [linux, storage]
---

Exploring various aspects of Linux storage systems and volumes, including block devices, partition tables, filesystems, and mounting volumes

## Overview of Linux Storage Systems

A Linux storage system typically consists of several components, including:

1. `Storage devices`: Physical hardware that stores data, such as hard drives, solid-state drives (SSDs), and USB flash drives.
2. `Partitions`: Logical divisions of a storage device that can hold different filesystems.
3. `Filesystems`: Organize and manage data on storage devices and enable access to files and directories.
4. `Mount points`: Locations in the directory structure where storage devices are attached and made accessible to the system.

## Identifying and Working with Block Devices

Block devices are storage devices that Linux can read and write in fixed-size blocks. Examples of block devices include hard drives, SSDs, and USB flash drives. To list all block devices, use the `lsblk` command:

When working with block devices, you will typically target a specific partition, such as `sda1` or `sda2`, rather than the entire device (e.g. `sda`).

To get more detailed information about your storage devices, use the `sudo fdisk -l` command.

### Mounting and Unmounting Storage Devices

To make a storage device accessible, you need to mount it to a specific directory or mount point. To mount a device, use the `mount` command:

```bash
    mount <device> <path>
```

To unmount a storage device, use the `umount` command:

```bash
    umount <path> OR <device>
```

## Partition Tables: GPT vs. MBR

A partition table is a data structure that resides on a storage device, such as a hard disk or SSD. It stores information about the organization and layout of the storage device, including the number, size, and locations of partitions.

A partition is a contiguous section of a storage device that can hold a filesystem. Partition tables allow you to divide a single storage device into multiple logical storage units, which can be managed and accessed independently.

Partition tables are essential for operating systems to read and write data on storage devices. When a system boots, the BIOS or UEFI firmware reads the partition table to determine which partition contains the boot loader, and then loads the operating system from that partition.

### Why Use GPT?

GPT has several advantages over MBR, including:

1. `Larger storage device support`: GPT can handle storage devices larger than 2 TiB, while MBR has a 2 TiB limit.
2. `More partitions`: GPT allows for up to 128 primary partitions, while MBR supports only four primary partitions or three primary partitions and an extended partition containing logical partitions.
3. `Data redundancy`: GPT stores multiple copies of partition data, reducing the risk of data loss due to corruption.
4. `Compatibility`: GPT is compatible with modern UEFI systems and can also work with legacy BIOS systems using a protective MBR.

## Filesystems: Choosing the Right One and Creating It with MKFS

A filesystem is responsible for organizing and managing data on storage devices. Choosing the right filesystem depends on your requirements and the specific use case.

### Which Filesystem to Use?

Some popular Linux filesystems include ext4, XFS, and Btrfs. Ext4 is the most widely used filesystem due to its maturity, stability, and compatibility with Linux distributions. In this guide, we will focus on ext4 and exFAT filesystems.

### Creating a Filesystem with MKFS

To create a filesystem, you can use the `mkfs` command followed by the desired filesystem type and the target partition. For example, to create an ext4 filesystem on `/dev/sda1`, use the following command:

```bash
    sudo mkfs.ext4 /dev/sda1
```

For compatibility with macOS and Windows systems, you can create an exFAT filesystem. First, ensure that you have the `exfat-utils` and `exfat-fuse` packages installed on Ubuntu and Debian systems. Then, use the following command to create an exFAT filesystem on `/dev/sda1`:

```bash
    sudo mkfs.exfat /dev/sda1
```

To add a label to the exFAT filesystem, use the `-n` option followed by the desired label:

```bash
    sudo mkfs.exfat /dev/sda1 -n "label"
```

### Viewing Available Space on Storage Devices

To view the available space on your storage devices, use the `df -h` command. This command displays the disk space usage in a human-readable format.

### Mounting Storage Volumes

Mount points are locations in the directory structure where storage devices are attached and made accessible to the system. The `/media` directory is typically used for mounting temporary storage volumes.

### Analyzing Disk Usage with NCDU

`ncdu` (NCurses Disk Usage) is a command-line utility that allows you to analyze disk usage on your system. To scan the whole filesystem, use the following command:

```bash
    sudo ncdu /
```

To show only the local filesystem, use the `-x` option:

```bash
    sudo ncdu / -x
```

### Permanently Mounting a Device by UUID

To ensure that a storage device is mounted at every system boot, you need to add an entry for the device in the `/etc/fstab` file, which stores the configuration for storage volumes.

First, use the `blkid` command to get the UUID (Universally Unique Identifier) of the device. Next, open the `/etc/fstab` file with a text editor, such as `nano`.

Add a new line to the file with the following format:

```bash
    UUID=<UUID> <mount_point> <filesystem_type> <mount_options> <dump_pass> <fsck_order>
```

Replace `<UUID>` with the UUID of the device, `<mount_point>` with the desired mount point, `<filesystem_type>` with the filesystem type (e.g., ext4 or exFAT), `<mount_options>` with any desired mount options (such as `defaults`), `<dump_pass>` with the dump pass value (usually `0`), and `<fsck_order>` with the filesystem check order (usually `0` or `1`).

For example, to mount an ext4 filesystem with UUID `12345678-9abc-def0-1234-56789abcdef0` at `/mnt/data`, add the following line to `/etc/fstab`:

```bash
    UUID=12345678-9abc-def0-1234-56789abcdef0 /mnt/data ext4 defaults 0 1
```

Save the changes and close the text editor. To test the new configuration, run the following command:

```bash
    sudo mount -a
```

If there are no errors, the device will be mounted at the specified mount point.

## Conclusion

In this guide, we have covered various aspects of Linux storage systems and volumes, including block devices, partition tables, filesystems, and mounting volumes. We also explored useful commands and tools for managing storage in Linux. With this knowledge, you should be able to confidently work with storage devices, partitions, and filesystems in your Linux environment.

In a future blog post, we will dive deeper into understanding the available filesystems and their features, benefits, and trade-offs. This will help you make more informed decisions when choosing a filesystem for your specific needs and use cases.
