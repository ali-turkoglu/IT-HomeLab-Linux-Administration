# Linux Administration

> Part 3 of the IT-HomeLab Series
>
> **Status:** ✅ Completed
>
> This repository documents my practical Linux administration work with Ubuntu Server, remote access, services, networking, security, and troubleshooting.

**Note:** This repository covers **Part 3 – Linux Administration**. Part 4 covers Docker and containers. Part 5 will focus on IT support and troubleshooting, followed by Linux Operations in Part 6.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Project Objectives](#project-objectives)
- [Technology Stack](#technology-stack)
- [Hardware & Environment](#hardware--environment)
- [Architecture](#architecture)
- [Project Roadmap](#project-roadmap)
- [Project Timeline](#project-timeline)
- [Next Projects](#next-projects)
- [IT-HomeLab Series](#it-homelab-series)
- [License](#license)

---

## Project Overview

This repository is the third part of my IT-HomeLab Series.

I installed and configured Ubuntu Server on the existing Proxmox environment to develop practical Linux administration skills.

The work included users and permissions, storage, SSH, packages, services, processes, logs, networking, and basic security. I also configured WireGuard for remote access and joined Ubuntu Server to the existing Active Directory domain.

The documentation records what I configured, tested, and troubleshot, together with selected screenshots and lessons learned.

---

## Project Objectives

The main goals were to:

- Install and configure Ubuntu Server.
- Understand the Linux filesystem and basic storage management.
- Manage users, groups, permissions, and ownership.
- Use SSH keys for remote administration.
- Manage packages with APT and dpkg.
- Manage services with systemd.
- Review processes, resource usage, and logs.
- Configure and troubleshoot basic networking.
- Set up secure remote access with WireGuard.
- Integrate Ubuntu Server with Active Directory.
- Review firewall rules, SSH settings, sudo permissions, and updates.
- Document configuration, testing, problems, and solutions.

---

## Technology Stack

| Area | Technologies |
|:---|:---|
| Virtualization | Proxmox VE |
| Operating System | Ubuntu Server |
| Remote Administration | OpenSSH, ED25519 keys |
| Package Management | APT, dpkg |
| Services & Logs | systemd, journalctl |
| Networking | Netplan, systemd-resolved |
| VPN | WireGuard |
| Firewall | nftables |
| Centralized Identity | Active Directory, realmd, SSSD |
| Automatic Updates | unattended-upgrades |

---

## Hardware & Environment

The Linux environment runs on the existing HomeLab infrastructure prepared in the previous projects.

| Component | Detail |
|:---|:---|
| Physical Host | Mini PC with AMD Ryzen 5 7430U, 16 GB RAM, and 512 GB SSD |
| Hypervisor | Proxmox VE |
| Linux VM | Ubuntu Server |
| VM Resources | 2 vCPU, 4 GB RAM, and 32 GB disk |
| Administration Device | MacBook |
| Existing Infrastructure | Windows Server with Active Directory and DNS |

The available resources are limited, so I kept the environment focused on practical administration tasks.

---

## Architecture

The Ubuntu Server is part of the same HomeLab as the Windows and Microsoft cloud projects.

| Basic Structure |
|:---:|
| ![HomeLab basic structure](docs/1-Ubuntu-Server-Installation-Base-Configuration/images/Basic-Structure.png) |

Ubuntu Server runs as a VM on Proxmox VE. It uses the existing internal DNS service and Active Directory identities. WireGuard provides remote access to the HomeLab.

This server provides the foundation for the Docker environment in Part 4 and the planned Linux Operations work in Part 6.

---

## Project Roadmap

The project was divided into phases. Each phase focused on one main topic and built on the previous work.

- ✅ [Phase 1 – Ubuntu Server Installation & Base Configuration](docs/1-Ubuntu-Server-Installation-Base-Configuration/README.md)
- ✅ [Phase 2 – Linux Users, Groups & Permissions](docs/2-Linux-Users-Groups-Permissions/README.md)
- ✅ [Phase 3 – Linux Filesystem & Storage Basics](docs/3-Linux-Filesystem-Storage-Basics/README.md)
- ✅ [Phase 4 – SSH & Remote Administration](docs/4-SSH-Remote-Administration/README.md)
- ✅ [Phase 5 – Package & Service Management](docs/5-Package-Service-Management/README.md)
- ✅ [Phase 6 – Processes, Logs & Troubleshooting](docs/6-Processes-Logs-Troubleshooting/README.md)
- ✅ [Phase 7 – Linux Networking](docs/7-Linux-Networking/README.md)
- ✅ [Phase 8 – WireGuard VPN & Secure Remote Access](docs/8-WireGuard-VPN-Secure-Remote-Access/README.md)
- ✅ [Phase 9 – Active Directory Integration & Centralized Authentication](docs/9-Active-Directory-Integration-Centralized-Authentication/README.md)
- ✅ [Phase 10 – Linux Security & Hardening](docs/10-Linux-Security-Hardening/README.md)

---

## Project Timeline

| Date | Milestone |
|:---|:---|
| 17-08-2026 | GitHub repository created |
| 17-08-2026 | Phase 1 – Ubuntu Server Installation & Base Configuration completed |
| 22-08-2026 | Phase 2 – Linux Users, Groups & Permissions completed |
| 26-08-2026 | Phase 3 – Linux Filesystem & Storage Basics completed |
| 31-08-2026 | Phase 4 – SSH & Remote Administration completed |
| 04-09-2026 | Phase 5 – Package & Service Management completed |
| 08-09-2026 | Phase 6 – Processes, Logs & Troubleshooting completed |
| 09-09-2026 | Phase 7 – Linux Networking completed |
| 21-09-2026 | Phase 8 – WireGuard VPN & Secure Remote Access completed |
| 29-09-2026 | Phase 9 – Active Directory Integration & Centralized Authentication completed |
| 03-10-2026 | Phase 10 – Linux Security & Hardening completed |

---

## Next Projects

### Part 4 – Docker & Containers

> **Status:** ✅ Completed

I used the Linux foundation from this repository to install Docker Engine, work with persistent storage and container networking, and manage services with Docker Compose and Portainer.

I deployed GLPI with MySQL and tested the ticket workflow, data persistence, backups, and LAN/VPN access.

See [Part 4 – Docker & Containers](https://github.com/ali-turkoglu/IT-HomeLab-Docker-Containers) for the completed phases.

### Part 5 – IT Support & Troubleshooting Lab

> **Status:** ⏳ Planned

I will use GLPI and the existing HomeLab to investigate simulated incidents and service requests, verify solutions, and document the ticket workflow.

All cases will be maintained in one dedicated support repository, with a separate page for each case.

The overview and case summaries will be in English. Detailed case documentation and GLPI ticket notes will be in German.

### Part 6 – Linux Operations

> **Status:** ⏳ Planned

Part 6 will focus on operating and maintaining the Linux and container environment.

The planned phases are:

1. Network Operations & Troubleshooting
2. Security Operations & Maintenance
3. Storage Operations & Capacity Management
4. Backup, Restore & Recovery Validation
5. Monitoring & Service Health
6. Logging, Alerting & Incident Basics
7. Final Small Company Linux Operations Scenario

These topics will build on the work in Parts 3 and 4.

---

## IT-HomeLab Series

This repository is Part 3 of my IT-HomeLab Series.

Each repository focuses on a different area of IT administration while remaining part of the same connected HomeLab environment.

| Part | Focus | Status | Repository |
|:---|:---|:---:|:---|
| **Part 1** | Windows Infrastructure | ✅ Completed | [IT-HomeLab-Windows-Infrastructure](https://github.com/ali-turkoglu/IT-HomeLab-Windows-Infrastructure) |
| **Part 2** | Cloud Identity & Microsoft 365 | ✅ Completed | [IT-HomeLab-Cloud-Identity](https://github.com/ali-turkoglu/IT-HomeLab-Cloud-Identity) |
| **Part 3** | Linux Administration | ✅ Completed | **IT-HomeLab-Linux-Administration** (Current Repository) |
| **Part 4** | Docker & Containers | ✅ Completed | [IT-HomeLab-Docker-Containers](https://github.com/ali-turkoglu/IT-HomeLab-Docker-Containers) |
| **Part 5** | IT Support & Troubleshooting Lab | ⏳ Planned | Repository coming soon |
| **Part 6** | Linux Operations | ⏳ Planned | Repository not created yet |

[← Back to IT HomeLab Journey](https://github.com/ali-turkoglu/IT-HomeLab-Journey)

---

## License

This project is licensed under the MIT License.

See the [LICENSE](LICENSE) file for more information.
