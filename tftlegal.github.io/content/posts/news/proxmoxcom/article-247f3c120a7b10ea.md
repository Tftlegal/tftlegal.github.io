---
title: "Proxmox Backup Server 4.2 released"
date: 2026-04-28T10:44:29Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-4-2"
summary: "Proxmox выпустил Proxmox Backup Server 4.2 на базе Debian 13.4 «Trixie» с обновленными пакетами, улучшенной поддержкой оборудования и повышенной безопасностью.   Новое обновление включает Linux kernel 7.0, ZFS 2.4 и значительные улучшения в организации данных, синхронизации, S3-хранилищах и программной базе.   Теперь группы и пространства имен резервных копий можно перемещать внутри одного хранилища, а push-синхронизация поддерживает серверное шифрование снимков.   Синхронизация стала быстрее за счет параллельной обработки нескольких групп, а pull-задачи умеют расшифровывать данные с удаленных хранилищ.   Официально поддержаны S3-совместимые объектные хранилища как backend с отслеживанием запросов и трафика.   Proxmox Backup Server 4.2 уже доступен для загрузки как ISO, поддерживает обновление через APT и распространяется под GNU AGPLv3.   Для предприятий Proxmox предлагает платную подписку на поддержку, начиная с 560 евро за сервер в год, с доступом к Enterprise Repository и технической поддержке."
---

# Proxmox Backup Server 4.2 released

## Краткое содержание

Proxmox выпустил Proxmox Backup Server 4.2 на базе Debian 13.4 «Trixie» с обновленными пакетами, улучшенной поддержкой оборудования и повышенной безопасностью.  
Новое обновление включает Linux kernel 7.0, ZFS 2.4 и значительные улучшения в организации данных, синхронизации, S3-хранилищах и программной базе.  
Теперь группы и пространства имен резервных копий можно перемещать внутри одного хранилища, а push-синхронизация поддерживает серверное шифрование снимков.  
Синхронизация стала быстрее за счет параллельной обработки нескольких групп, а pull-задачи умеют расшифровывать данные с удаленных хранилищ.  
Официально поддержаны S3-совместимые объектные хранилища как backend с отслеживанием запросов и трафика.  
Proxmox Backup Server 4.2 уже доступен для загрузки как ISO, поддерживает обновление через APT и распространяется под GNU AGPLv3.  
Для предприятий Proxmox предлагает платную подписку на поддержку, начиная с 560 евро за сервер в год, с доступом к Enterprise Repository и технической поддержке.

## Полная статья

**VIENNA, Austria – April 29, 2026**– Leading open-source server solutions provider Proxmox Server Solutions GmbH (henceforth "Proxmox") today announced the release of Proxmox Backup Server 4.2. This update is based on Debian 13.4 “Trixie”, bringing updated packages, improved hardware support, and enhanced security. Proxmox Backup Server 4.2 ships with Linux kernel 7.0 as the new stable default and includes ZFS 2.4 for reliable, enterprise-grade storage. The new release also delivers major improvements in backup data organization, sync security, sync performance, S3-backed storage, and the underlying software stack.

## Highlights in Proxmox Backup Server 4.2

Support for moving groups and namespaces

Backup groups and namespaces can now be moved to different locations within the same datastore. This gives administrators more flexibility when reorganizing existing backups, while per-group locking helps ensure data consistency throughout the process.

Server-side en/decryption support for sync jobs

Push sync jobs can now be configured to encrypt snapshots on the fly before sending them to remote datastores. This is particularly useful when synchronizing backup data to less trusted remote Proxmox Backup Server instances. In addition, pull sync jobs can be configured to decrypt snapshots that were encrypted on remote datastores. To make key management easy, tape and sync encryption keys can now all be managed from the same centralized panel.

Concurrent group pull/push support for sync jobs

Sync jobs can now process multiple groups in parallel through the new worker-threads property. This significantly improves throughput on high-latency networks and helps overcome HTTP/2 connection limitations. Logging has also been improved, with contextual prefixes for log messages and better visibility for push sync jobs.

S3-compatible object stores as backup storage backend

S3-compatible object stores are now officially supported as a backup storage backend. S3-backed datastores can now also track request counts and traffic statistics for deeper operational insight and monitoring. This is especially useful for identifying unexpected traffic volume early. The request counters are visualized in the datastore summary.

### Availability

Proxmox Backup Server 4.2 is immediately available for download. Users can obtain a complete installation image via ISO download, which contains the full feature set of the solution and can be installed quickly on bare-metal systems using an intuitive installation wizard.

Seamless distribution upgrades from older versions of Proxmox Backup Server are possible using the standard APT package management system. Furthermore, it is also possible to install Proxmox Backup Server on top of an existing Debian installation. As Free/Libre and Open Source Software (FLOSS), the entire solution is published under the GNU AGPLv3.

For enterprise users, Proxmox Server Solutions GmbH offers professional support through subscription plans. Pricing for these subscriptions starts at EUR 560 per server per year, including unlimited backup storage and unlimited backup clients. A subscription provides access to the stable Enterprise Repository with timely updates via the web interface, as well as certified technical support. It is recommended for production use.

Resources:

- ISO Image Download:[https://www.proxmox.com/en/downloads/](https://www.proxmox.com/en/downloads/)
- Forum Announcement:[https://forum.proxmox.com/](https://forum.proxmox.com/threads/proxmox-backup-server-4-2-released.183130/)
- Roadmap: For published and upcoming features, see the[Release Notes & Roadmap](https://pbs.proxmox.com/wiki/Roadmap)

###

**About Proxmox Backup Server
**Proxmox Backup Server is an enterprise backup solution for backing up and restoring virtual machines, containers, and physical hosts. The open-source client-server software supports incremental backups, deduplication, Zstandard compression, and authenticated encryption. Designed for efficiency and ease of use, Proxmox Backup Server enables users to back up data in a space-efficient manner and restore virtual machines, archives, or single objects quickly. With its web-based user interface, the solution helps reduce administrative effort through simplified management. Proxmox Backup Server is licensed under the GNU AGPLv3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions
**Proxmox provides powerful and user-friendly open-source server software. Enterprises of all sizes and industries use the Proxmox solutions to deploy efficient and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support, training services, and an extensive partner ecosystem to ensure business continuity for its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria. To learn more visit[https://www.proxmox.com](https://www.proxmox.com). Further information is available on[LinkedIn](https://www.linkedin.com/company/proxmox)and on[YouTube](https://www.youtube.com/user/ProxmoxVE).

Contact: Michael Hiess, Proxmox Server Solutions GmbH,[marketing@proxmox.com](mailto:marketing@proxmox.com)

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-4-2
