# Guide: How to Expand Disk Space on Cloud Platforms

## Introduction
When you increase disk space on cloud platforms (like DigitalOcean, AWS, or VMs), the OS won't automatically show the increased space. This is because both the partition and filesystem need to be manually expanded.

## Steps to Expand Disk Space

1. Check the new disk size:
```bash
fdisk -l /dev/sda
```
Example output:
```
Disk /dev/sda: 100 GiB, 107374182400 bytes, 209715200 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
```
Here you can see the total disk size is 100 GiB

2. View the current partition table:
```bash
gdisk -l /dev/sda
```
Example output:
```
GPT fdisk (gdisk) version 1.0.5
Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.
Disk /dev/sda: 209715200 sectors, 100.0 GiB
Model: Virtual disk    
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 1234ABCD-1234-5678-ABCD-1234567890AB
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 167772126
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048       167772126   80.0 GiB    8300  Linux filesystem
```
In this example, you can see:
- Total disk size: 100 GiB (from fdisk)
- Current partition size: 80 GiB (from gdisk)
- This means we have 20 GiB of unallocated space that needs to be expanded

3. Resize the partition to use the new space:
```bash
growpart /dev/sda 1
```

4. Expand the filesystem to use the newly available partition space:
```bash
resize2fs /dev/sda1
```

## Note
- These commands use `/dev/sda1` as an example
- If you're working with a different partition (like `/dev/sda2`), replace the number accordingly in both the `growpart` and `resize2fs` commands
- You can verify the new size after completion by running `df -h` to check the filesystem size
- Official documentation reference: https://docs.digitalocean.com/products/droplets/how-to/resize/

This process works across various cloud platforms, including DigitalOcean, AWS, and virtual machines.
