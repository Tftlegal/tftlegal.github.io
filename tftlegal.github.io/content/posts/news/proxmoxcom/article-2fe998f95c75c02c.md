---
title: "Proxmox Backup Server 3.4 released"
date: 2025-04-10T09:17:50Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-3-4"
summary: "Компания Proxmox представила версию Proxmox Backup Server 3.4 — обновленное открытое решение для корпоративного резервного копирования. Обновление направлено на повышение производительности и удобства управления данными. В новой версии оптимизирована сборка мусора с кэшированием, что ускоряет освобождение дискового пространства за счет дедупликации. Появилась более гибкая выборка снимков для синхронизации, в том числе возможность передавать только зашифрованные или проверенные бэкапы. Также добавлены статический клиент для создания файловых резервных копий на Linux, повышенная скорость ленточного резервного копирования и поддержка более новых версий ZFS и ядра Linux. Proxmox Backup Server 3.4 уже доступен для загрузки и обновления, а для предприятий Proxmox предлагает платную подписку на корпоративную поддержку."
---

# Proxmox Backup Server 3.4 released

## Краткое содержание

Компания Proxmox представила версию Proxmox Backup Server 3.4 — обновленное открытое решение для корпоративного резервного копирования.
Обновление направлено на повышение производительности и удобства управления данными.
В новой версии оптимизирована сборка мусора с кэшированием, что ускоряет освобождение дискового пространства за счет дедупликации.
Появилась более гибкая выборка снимков для синхронизации, в том числе возможность передавать только зашифрованные или проверенные бэкапы.
Также добавлены статический клиент для создания файловых резервных копий на Linux, повышенная скорость ленточного резервного копирования и поддержка более новых версий ZFS и ядра Linux.
Proxmox Backup Server 3.4 уже доступен для загрузки и обновления, а для предприятий Proxmox предлагает платную подписку на корпоративную поддержку.

## Полная статья

**VIENNA, Austria – April 10, 2025 –**Enterprise software developer Proxmox Server Solutions GmbH (henceforth "Proxmox") today announced the release of Proxmox Backup Server version 3.4. Designed to address enterprise data protection needs, this latest version of the open-source backup solution introduces new capabilities that further improve performance and usability.

### Highlights in Proxmox Backup Server 3.4

- Optimized Performance for Garbage Collection (GC): To increase storage efficiency through deduplication, backup data in Proxmox Backup Server is saved as chunks. In order to free up storage space after backup snapshots have been deleted, a garbage collection (GC) process identifies and cleans up unreferenced backup data chunks. This version 3.4 optimizes the GC mechanism and integrates a cache to reduce the number of expensive file metadata updates. While this enhancement increases memory usage, it substantially reduces execution time, resulting in faster and more efficient garbage collection. The caching mechanism can be fine-tuned on a per-datastore basis.
- Granular backup snapshot selection for sync jobs: Offsite backups are of great interest for data security and therefore Proxmox Backup Server provides sync jobs to push or pull backup snapshots to and from remote Proxmox Backup Server instances. Group filters already allow the selection of which backup groups should be synchronized. In this latest version, the selection functionality has been extended to allow exclusive synchronization of encrypted or verified backup snapshots.
- Static build of Proxmox Backup client: While Proxmox Backup Server is tightly integrated with Proxmox VE, its command-line client can also be used outside of Proxmox VE. A new statically-linked binary of the command-line client makes it easier to create file-level backups of arbitrary Linux hosts.
- Increased throughput for tape backup: Since version 3.4, Proxmox Backup Server now allows to increase the number of worker threads when reading chunks during tape backup, which can significantly increase the throughput in certain setups.
- Latest versions of open-source technologies: This version is based on Debian 12.10 (“Bookworm”), but uses a newer Linux kernel 6.8.12-9 and includes ZFS 2.2.7 (with compatibility patches for Kernel 6.14). While the kernel 6.8.12-9 is the stable default, Linux kernel 6.14 can optionally be installed for better support of the latest hardware.

Proxmox Backup Server seamlessly integrates into Proxmox Virtual Environment—users just need to add a datastore of Proxmox Backup Server as a new storage backup target to Proxmox VE.

### Availability

Proxmox Backup Server 3.4 is now available for download. Users can either update their version of the solution or install the ISO image on bare-metal using the Installation Wizard. Distribution upgrades from older versions of Proxmox Backup Server are possible via APT. Optionally, Proxmox Backup Server can be installed on top of Debian.

License: Proxmox Backup Server is free and open-source software, published under the GNU AGPLv3.

Enterprise Support: For enterprise users, Proxmox Server Solutions offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 540 per server, including unlimited backup storage and unlimited backup-clients.

### Resources

- ISO Image Download:[https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/downloads)
- Forum Announcement:[https://forum.proxmox.com/threads/proxmox-backup-server-3-4-released.164869/](https://forum.proxmox.com/threads/proxmox-backup-server-3-4-released.164869/)
- Roadmap: For published and upcoming features, see the[Release Notes & Roadmap](https://pbs.proxmox.com/wiki/index.php/Roadmap)

###

**About Proxmox Backup Server**
Proxmox Backup Server is an enterprise backup solution for backing up and restoring virtual machines, containers, and physical hosts. The open-source client-server software supports incremental backups, deduplication, Zstandard compression, and authenticated encryption. To increase productivity, the easy-to-use Proxmox Backup Server allows you to back up your data in a space-efficient manner, restore VMs, archives or single objects in a flash. With the web-based user interface, you can effectively reduce work hours thanks to simplified management. Proxmox Backup Server is licensed under the GNU AGPLv3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions**
Proxmox provides powerful and user-friendly open-source server software. Enterprises of all sizes and industries use the Proxmox solutions to deploy efficient and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support, training services, and an extensive partner ecosystem to ensure business continuity for its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria. To learn more visit[www.proxmox.com](https://www.proxmox.com/)or follow us on[LinkedIn](https://www.linkedin.com/company/proxmox)and[YouTube](https://www.youtube.com/user/ProxmoxVE).

Contact: Daniela Häsler, Proxmox Server Solutions GmbH, marketing@proxmox.com

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-3-4
