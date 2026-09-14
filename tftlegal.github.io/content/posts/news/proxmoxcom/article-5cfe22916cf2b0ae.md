---
title: "Proxmox Backup Server 4.1 released"
date: 2025-11-26T09:51:37Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-4-1"
summary: "Proxmox объявила о выпуске Proxmox Backup Server 4.1. Обновление построено на базе Debian 13.2 «Trixie» и включает ядро Linux 6.17, ZFS 2.3, обновленные пакеты, улучшенную поддержку оборудования и повышенную безопасность. В версии добавлено ограничение трафика с учетом конкретных пользователей, что позволяет более гибко приоритизировать резервные и восстановительные операции. Администраторы могут настраивать параллелизм заданий проверки резервных копий, что ускоряет верификацию и помогает балансировать нагрузку. Также появилась настройка ограничения пропускной способности для S3-совместимых конечных точек хранения. Решение доступно бесплатно под лицензией GNU AGPLv3, а для предприятий Proxmox предлагает платные подписки с поддержкой и enterprise-репозиторием."
---

# Proxmox Backup Server 4.1 released

## Краткое содержание

Proxmox объявила о выпуске Proxmox Backup Server 4.1.
Обновление построено на базе Debian 13.2 «Trixie» и включает ядро Linux 6.17, ZFS 2.3, обновленные пакеты, улучшенную поддержку оборудования и повышенную безопасность.
В версии добавлено ограничение трафика с учетом конкретных пользователей, что позволяет более гибко приоритизировать резервные и восстановительные операции.
Администраторы могут настраивать параллелизм заданий проверки резервных копий, что ускоряет верификацию и помогает балансировать нагрузку.
Также появилась настройка ограничения пропускной способности для S3-совместимых конечных точек хранения.
Решение доступно бесплатно под лицензией GNU AGPLv3, а для предприятий Proxmox предлагает платные подписки с поддержкой и enterprise-репозиторием.

## Полная статья

**VIENNA, Austria – November 26, 2025**– Leading open-source server solutions provider Proxmox Server Solutions GmbH (henceforth "Proxmox"), today announced the release of Proxmox Backup Server 4.1. This update is based on Debian 13.2 “Trixie”, bringing the latest Debian release with refreshed packages, better hardware support, and enhanced security. Proxmox Backup Server 4.1 ships with Linux kernel 6.17 as the new stable default and includes ZFS 2.3 for reliable, enterprise-grade storage. The release also delivers major improvements in traffic control, verification performance, and S3-based backup operations.

## Highlights in Proxmox Backup Server 4.1

User-based traffic limiting for optimized bandwidth control

Proxmox Backup Server 4.1 extends the existing traffic control capabilities, which already allow administrators to limit backup and restore traffic for specific client networks. With this release, traffic control can additionally take the authenticated Proxmox Backup Server user into account. This enables more convenient and fine-grained prioritization of backup and restore workloads, for example by assigning higher backup bandwidth to business-critical services or separating production and test environments at the user level.

Configurable parallelism for verify jobs

Backup snapshot verification is both I/O- and CPU-intensive, as it reads backup data chunks from disk and validates their checksums. With version 4.1, administrators can configure the number of threads used for disk reads and checksum verification in verify jobs. Depending on the hardware and workload, this can significantly reduce verify runtimes and helps balance verification tasks against other resource-intensive operations.

Bandwidth rate limiting for S3 endpoints

Building on the native support for S3-compatible object storage introduced in Proxmox Backup Server 4.0, version 4.1 adds bandwidth rate limiting for S3 endpoints. Administrators can now cap the bandwidth used by backup and restore operations to and from S3 object stores. This helps prevent network congestion between Proxmox Backup Server instances and the object storage infrastructure, especially in shared or bandwidth-constrained environments.

### Availability

Proxmox Backup Server 4.1 is immediately available for download. Users can obtain a complete installation image via ISO download, which contains the full feature set of the solution and can be installed quickly on bare-metal systems using an intuitive installation wizard.

Seamless distribution upgrades from older versions of Proxmox Backup Server are possible using the standard APT package management system. Furthermore, it is also possible to install Proxmox Backup Server on top of an existing Debian installation. As Free/Libre and Open Source Software (FLOSS), the entire solution is published under the GNU AGPLv3.

For enterprise users, Proxmox Server Solutions GmbH offers professional support through subscription plans. Pricing for these subscriptions starts at EUR 540 per server per year, including unlimited backup storage and unlimited backup-clients. A subscription provides access to the stable Enterprise Repository with timely updates via the web interface, as well as to certified technical support. It is recommended for production use.

Resources:

- ISO Image Download:[https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/downloads)
- Forum Announcement:[https://forum.proxmox.com/](https://forum.proxmox.com/threads/proxmox-backup-server-4-1-released.176867/)
- Roadmap: For published and upcoming features, see the[Release Notes & Roadmap](https://pbs.proxmox.com/wiki/Roadmap#Proxmox_Backup_Server_4.1)

###

**About Proxmox Backup Server
**Proxmox Backup Server is an enterprise backup solution for backing up and restoring virtual machines, containers, and physical hosts. The open-source client-server software supports incremental backups, deduplication, Zstandard compression, and authenticated encryption. To increase productivity, the easy-to-use Proxmox Backup Server allows you to back up your data in a space-efficient manner, restore VMs, archives or single objects in a flash. With the web-based user interface, you can effectively reduce work hours thanks to simplified management. Proxmox Backup Server is licensed under the GNU AGPLv3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions
**Proxmox provides powerful and user-friendly open-source server software. Enterprises of all sizes and industries use the Proxmox solutions to deploy efficient and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support, training services, and an extensive partner ecosystem to ensure business continuity for its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH, marketing@proxmox.com

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-4-1
