# Phase 3 – Linux Filesystem & Storage Basics

## Purpose

In this phase, I examined and managed the filesystem and storage structure of the Ubuntu Server.

The goal was not only to learn Linux storage commands, but also to prepare the Ubuntu Server with a clean and maintainable storage design for future use as a domain member, Docker host, and Linux service server.

## Objectives

- Understand the existing Linux filesystem structure.
- Understand the relationship between disks, partitions, and filesystems.
- Learn how Linux mount points work.
- Configure persistent mounts using `/etc/fstab` and UUIDs.
- Check disk space and directory usage using `df` and `du`.
- Understand basic inode usage.
- Understand and implement the LVM (Logical Volume Manager) structure.
- Design a suitable storage layout that separates the operating system from application data.
- Make storage decisions with future Docker and container workloads in mind.

---

## Environment

| Component | Configuration |
| :--- | :--- |
| **Server** | Ubuntu Server 26.04 LTS |
| **Hostname** | `ubuntu01` |
| **Virtualization** | Proxmox VE |
| **System Disk** | 32 GiB |
| **Data Disk** | 24 GiB (newly added) |
| **Filesystem** | ext4 |
| **Storage Management** | LVM |
| **Swap** | 2.6 GiB swap file |

---

## Initial Storage Assessment

Before making changes, I reviewed the existing storage setup.

The Ubuntu Server uses a 32 GiB system disk. The root filesystem (`/`) is an ext4 filesystem on an LVM logical volume with about 23 GiB of free space. The `/boot` filesystem is located on a separate 2 GiB ext4 partition.

The server also uses a 2.6 GiB swap file. No additional swap configuration was required.

### Initial Storage Structure

```text
System Disk – 32 GiB
│
├── /boot → 2 GiB ext4
│
└── / → 30 GiB ext4 on LVM
    └── ubuntu-vg
        └── ubuntu-lv

Swap
└── /swap.img → 2.6 GiB
```

---

## Storage Design & Architecture

The existing 32 GiB system disk remains dedicated to the Ubuntu operating system.

I added a separate 24 GiB virtual disk for Linux service and application data, including future Docker container workloads.

The existing `/srv/projects` directory was kept because it was created and used during the previous phase for Linux users, groups, ownership, and permissions testing.

The Windows Server remains the central file server for company department shares. The Ubuntu data disk is intended for Linux services and application workloads, not as a replacement for the Windows file server.

**Final Data Storage Structure:**

```text
Data Disk – 24 GiB
│
└── /dev/sdb1 (Partition)
    │
    └── LVM Physical Volume
        │
        └── data-vg (Volume Group)
            │
            └── data-lv (Logical Volume)
                │
                └── ext4 (Filesystem)
                    │
                    └── /srv/data (Mount Point)
```
This design separates the operating system from application and service data and provides a dedicated location for future Linux workloads.

---

## Storage Implementation

### 1. Review the Existing Storage
I checked the existing disks, filesystems, mount points, and LVM structure using:
```bash
lsblk
df -hT
findmnt
cat /etc/fstab
sudo blkid
sudo pvs
sudo vgs
sudo lvs
```
This showed that the existing root filesystem uses LVM and that the system volume group has no free space.

### 2. Add a Separate Data Disk
I added a new 24 GiB virtual disk to the Ubuntu VM through Proxmox. Ubuntu detected the new disk as `/dev/sdb`, while the system disk remained `/dev/sda`.

### 3. Create a GPT Partition Table
I created a GPT (GUID Partition Table) on the new disk to define how partitions are organized. The partition utilizes the full capacity of the new disk:

```text
/dev/sdb
└── /dev/sdb1 → 24 GiB
```

### 4. Create the LVM Structure
I initialized the new partition as an LVM Physical Volume:
```bash
sudo pvcreate /dev/sdb1
```
Next, I created a new Volume Group named `data-vg`:
```bash
sudo vgcreate data-vg /dev/sdb1
```
Finally, I created a Logical Volume using all available space:
```bash
sudo lvcreate -l 100%FREE -n data-lv data-vg
```
The resulting LVM structure is:
```text
/dev/sdb1
    ↓
Physical Volume
    ↓
data-vg
    ↓
data-lv
```

### 5. Create the ext4 Filesystem
I created an ext4 filesystem on the new Logical Volume:
```bash
sudo mkfs.ext4 /dev/data-vg/data-lv
```
I then verified the filesystem and its UUID using `blkid`:
`UUID="708af4c4-c10d-4fe9-96ff-0a1a8c0f9caf"`

### 6. Create the Mount Point
The existing `/srv` directory already contained the `projects` directory from the previous phase.

For this reason, I created a separate mount point for the new data filesystem:
```bash
sudo mkdir /srv/data
```
The resulting structure is:
```bash
/srv
├── projects
└── data
```

### 7. Mount and Test the Filesystem
I temporarily mounted the filesystem:
```bash
sudo mount /dev/data-vg/data-lv /srv/data
df -hT /srv/data
```
The filesystem was successfully mounted with approximately 24 GiB of capacity. I also created a small test file to verify read/write access.

---

## Persistent Mount (`/etc/fstab`)

To ensure that the filesystem is mounted automatically after a reboot, I configured a persistent mount.

First, I created a backup of the configuration file:
```bash
sudo cp /etc/fstab /etc/fstab.bak
```

I added the following line to `/etc/fstab` using the UUID:
```text
UUID=708af4c4-c10d-4fe9-96ff-0a1a8c0f9caf /srv/data ext4 defaults 0 2
```
Using the UUID provides a stable way to identify the filesystem instead of depending on a device name such as `/dev/sdb1`.

I tested the configuration without rebooting:
```bash
sudo mount -a
```
After verifying there were no errors, I rebooted the server. The disk automatically mounted successfully.

---

## Disk Usage vs. Inodes

During this phase, I explored two different ways to measure storage:

###Disk Space

df shows filesystem capacity and available space:
```bash
df -h /srv/data
```
`du` shows the space used by files and directories:
```bash
sudo du -sh /srv/data
```

The important difference is:

`df` shows filesystem usage, while `du` shows the space used by files and directories.

###Inodes

Inodes store metadata about files and directories, including ownership, permissions, timestamps, and information about where file data is stored.

A filesystem can run out of inodes even when it still has free disk space, for example when it contains a very large number of small files.

I checked inode usage with:
```bash
df -i /srv/data
```
The data filesystem has 1,572,864 inodes, with only 12 currently in use. This means that inode usage is very low.

---

## Important Commands Reference

| Command     | Meaning                     | Purpose                                 |
| :---------- | :-------------------------- | :-------------------------------------- |
| `lsblk`     | list block devices          | View disks, partitions, and LVM volumes |
| `blkid`     | block device identification | View UUIDs and filesystem types         |
| `df`        | disk free                   | Check filesystem capacity and usage     |
| `du`        | disk usage                  | Check directory and file usage          |
| `mount`     | mount filesystem            | Attach a filesystem to a directory      |
| `findmnt`   | find mounts                 | View mounted filesystems                |
| `fdisk`     | fixed disk utility          | Manage disk partition tables            |
| `pvs`       | physical volumes            | View LVM Physical Volumes               |
| `vgs`       | volume groups               | View LVM Volume Groups                  |
| `lvs`       | logical volumes             | View LVM Logical Volumes                |
| `pvcreate`  | physical volume create      | Initialize a partition for LVM          |
| `vgcreate`  | volume group create         | Create an LVM Volume Group              |
| `lvcreate`  | logical volume create       | Create an LVM Logical Volume            |
| `mkfs.ext4` | make filesystem             | Create an ext4 filesystem               |
| `mkdir`     | make directory              | Create a directory                      |


---

## Verification

After rebooting, I verified the final storage configuration:

```text
Ubuntu01
│
├── System Disk – 32 GiB
│   ├── /boot → ext4
│   └── / → ext4 on LVM
│
├── Swap
│   └── /swap.img → 2.6 GiB
│
└── Data Disk – 24 GiB
    └── /dev/sdb1
        └── data-vg
            └── data-lv
                └── ext4
                    └── /srv/data
```
The data filesystem was automatically mounted at `/srv/data` after the reboot.

The filesystem has approximately 23 GiB of available space, confirming that the persistent mount configuration is working correctly.

---

## Lessons Learned

- **Storage Layers:** Linux storage is built in layers: Physical Disk → Partition (GPT) → LVM (PV, VG, LV) → Filesystem (ext4) → Mount Point.
- **Flexibility of LVM:** LVM provides a flexible way to manage storage, allowing volume expansion in the future without reinstalling the OS.
- **Persistence:** `/etc/fstab` is used for persistent mounts, and using UUIDs instead of `/dev/sdx` provides a more stable way to identify filesystems if disk device names change.
- **Inodes vs. Space:** Disk management requires monitoring both physical space (`df -h`) and inode availability (`df -i`).
- **Separation of Concerns:** Separating the OS disk from application data provides a cleaner and more manageable storage design.

## Result

The Ubuntu Server now features a dedicated 24 GiB LVM-based data filesystem persistently mounted at `/srv/data`. The system disk remains isolated for the OS, while the new data disk is fully prepared for hosting future Linux services, applications, and Docker container workloads.

    ---

## Navigation

| Previous | Home | Next |
|:--------:|:----:|:----:|
| ⬅️ [2- Linux Users Groups & Permissions](../2-Linux-Users-Groups-Permissions/README.md) | 🏠 [Home](../../README.md) | ➡️ [4- SSH-Remote-Administration](../4-SSH-Remote-Administration/README,md) |
