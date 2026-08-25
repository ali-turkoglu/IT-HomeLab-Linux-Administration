# Phase 3 – Linux Filesystem & Storage Basics

## Purpose

In this phase, I will examine and manage the filesystem and storage structure of the Ubuntu Server.

The goal is not only to learn Linux storage commands, but also to prepare the Ubuntu Server with a clean storage design for future use as a domain member, Docker host and Linux service server.

## Objectives

- Understand the existing Linux filesystem structure
- Understand the relationship between disks, partitions and filesystems
- Learn how Linux mount points work
- Understand persistent mounts
- Check disk and filesystem usage
- Understand basic inode usage
- Understand the basic LVM structure
- Design a suitable storage layout for the Ubuntu Server
- Use a separate data disk for Linux service and application data
- Configure persistent mounts using UUID and `/etc/fstab`
- Test and verify storage changes
- Make storage decisions with future Docker and container workloads in mind

---

## Environment

| Component | Configuration |
|:--|:--|
| Server | Ubuntu Server 26.04 |
| Hostname | `ubuntu01` |
| Virtualization | Proxmox VE |
| System Disk | 32 GiB |
| Data Disk | 24 GiB |
| Filesystem | ext4 |
| Storage Management | LVM |
| Swap | 2.6 GiB swap file |

---
