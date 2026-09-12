---
title: "Proxmox enables Namespaces in Proxmox Backup Server 2.2"
date: 2022-05-07T10:45:58Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-2-2"
summary: "Proxmox Server Solutions представила обновление Proxmox Backup Server 2.2. Новая версия работает на Debian 11.3 с ядром Linux 5.15 и ZFS 2.1.4. Ключевые нововведения включают namespace для организации бэкапов из разных источников в одном хранилище, режимы обслуживания с отслеживанием операций и улучшенное восстановление файлов. Обновление также повышает производительность, снижает потребление памяти и добавляет dry-run бэкапа, поддержку Zstandard-архивов и GUI-фильтры. Proxmox Backup Server предназначен для резервного копирования и восстановления виртуальных машин, контейнеров и физических хостов. Продукт остается бесплатным и открытым с опциональной платной поддержкой для предприятий."
---

# Proxmox enables Namespaces in Proxmox Backup Server 2.2

## Краткое содержание

Proxmox Server Solutions представила обновление Proxmox Backup Server 2.2. Новая версия работает на Debian 11.3 с ядром Linux 5.15 и ZFS 2.1.4. Ключевые нововведения включают namespace для организации бэкапов из разных источников в одном хранилище, режимы обслуживания с отслеживанием операций и улучшенное восстановление файлов. Обновление также повышает производительность, снижает потребление памяти и добавляет dry-run бэкапа, поддержку Zstandard-архивов и GUI-фильтры. Proxmox Backup Server предназначен для резервного копирования и восстановления виртуальных машин, контейнеров и физических хостов. Продукт остается бесплатным и открытым с опциональной платной поддержкой для предприятий.

## Полная статья

Updates include a new namespace feature for efficient backup management, maintenance modes with operations tracking, optimized file restore, and general performance improvements.

**VIENNA, Austria – May 18, 2022 –**Enterprise software developer Proxmox Server Solutions GmbH (henceforth "Proxmox") today announced the latest update to its open-source backup solution Proxmox Backup Server, which makes it easy to back up and restore virtual machines, containers, and physical hosts. The new version 2.2 is based on Debian 11.3 “Bullseye”, but using the newer Linux kernel 5.15, and including ZFS 2.1.4.

The enterprise backup solution from Proxmox supports incremental, fully deduplicated backups, significantly reducing network load, required storage space and, as a result, total cost of infrastructure. Featuring strong client-side encryption, Proxmox Backup Server allows businesses to back up data to targets which are not fully trusted, in a space-efficient manner and restore VMs, archives or single objects in a flash. New features and capabilities in Proxmox Backup Server 2.2 further ensure that data is reliably backed up and restored, efficiently organized, and easily managed.

### What’s new in Proxmox Backup Server 2.2

**“Namespace” feature to significantly improve backup management of multiple sources on-premises, remotely, and in the cloud:**Proxmox Backup Server stores backup data (that is: backup snapshots and their referenced chunks) in datastores, and multiple datastores can be configured. The deduplication of data, which helps improve storage utilization, is based on reusing chunks, which are referenced by the indexes in a backup snapshot. This means that multiple indexes can reference the same chunks (even across backup snapshots), reducing the amount of space needed to contain the data.

To help neatly organize backups and minimize the required storage space, Proxmox Backup Server 2.2 introduces namespaces. Backups from multiple sites (local and remote) or Proxmox VE setups can now be organized into namespaces within a single datastore. This allows for optimal use of deduplication, as this technique is only applied at the datastore level, and reduces the maintenance burden of managing multiple datastores.

With namespaces, businesses can safely back up from different sources to one (physical) datastore, with more fine-grained access control and without having naming conflicts. The "namespace" feature allows sharing a single datastore, including the chunk store, across multiple backup ID namespaces (e.g., one per Proxmox VE cluster).

**Maintenance Mode with Active Operations Tracking:**To enable administrators to safely execute maintenance tasks on a datastore, the newly implemented “read-only” and “offline” maintenance modes were added. Additionally, by tracking active operations, Proxmox Backup Server can check for currently conflicting access operations, for example a previously started backup restore task, and await their completion before entering maintenance mode, while simultaneously blocking new incoming operations.

**Performance and Back-End Improvements:**General improvements in the back-end help to drastically reduce memory usage during backup. By extensively analyzing the behavior of the system allocator, and then optimizing the interaction between the allocator and Proxmox Backup Server, for example, the memory footprint has been reduced significantly.
Additionally, with version 2.2, it is now possible to do a dry-run of a desired backup via the command line, to view what will happen.

**Improved File Restore:**To confidently restore data and get back to business quickly, this version includes enhanced file restore. Proxmox Backup Server adds support for downloading Zstandard-compressed tar archives. Compared to the already included zip format, tar archives support more file types (for example, hard links and device nodes), and zstd allows for fast and efficient compression.
To improve the handling of non-ASCII code point extraction under Windows, the language encoding flag is added to files when creating a zip archive, if the entry is valid UTF-8. Additionally, ZFS pools get mounted only on demand, and the automatic pre-mounting of ZFS pools is avoided to prevent the upfront time-cost.

**New GUI enhancements greatly improve productivity:**With the included web interface, backup administrators can effectively reduce work hours. To simplify management, this version now brings the ‘group-filter’ to the GUI. The "Add" and "Edit" windows of the sync jobs and tape-backup jobs panels have an added tab for the filter option. Users can specify in the GUI if they want to process only a specific backup type (ct, vm, host), a specific group, or a regex that matches the group-ID for such a job. The node configuration file now supports multi-line comments and has a Markdown-aware panel for recording structured notes. Some translations have been updated including Arabic, French, German, Japanese, Polish, and Turkish.

> "Our customers and partners are looking for efficient ways to protect their business data, while keeping the maintenance burden low. With Proxmox Backup Server 2.2, we make this even easier by adding namespaces and lowering overhead."Thomas Lamprecht - Lead developer at Proxmox

Proxmox Backup Server is designed as a standalone solution, thus avoiding vendor lock-in. The client-server backup solution enables secure backup and restore of critical business data and is currently optimized for the open-source virtualization management platform Proxmox Virtual Environment, the company’s flagship product. Users simply need to add the Proxmox Backup Server datastore as a new backup storage target to Proxmox VE.

### Availability

New capabilities are now available in the Proxmox Backup Server 2.2 release for all users. New users can[download the ISO image](https://www.proxmox.com/en/component/zoo/?view=frontpage&layout=frontpage&Itemid=120), which can be easily installed on bare-metal, using the installation wizard. Proxmox Backup Server is free and open-source software, published under the GNU AGPL, v3 license.

For enterprise users, Proxmox Server Solutions GmbH offers a[subscription-based support model](https://www.proxmox.com/en/products/proxmox-backup-server/pricing), which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 449 per server, including unlimited backup storage and unlimited backup-clients.

**Forum Announcement**
[https://forum.proxmox.com/threads/proxmox-backup-server-2-2-available.109724](https://forum.proxmox.com/threads/proxmox-backup-server-2-2-available.109724)

**Video tutorial**
[What's new in Proxmox Backup Server 2.2](https://www.proxmox.com/en/training/video-tutorials/item/what-s-new-in-proxmox-backup-server-2-2)

**About Proxmox Server Solutions**
Proxmox® is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructure, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training services to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.
Follow us on LinkedIn[https://www.linkedin.com/company/proxmox](https://www.linkedin.com/company/proxmox)or on YouTube[https://www.youtube.com/user/ProxmoxVE](https://www.youtube.com/user/ProxmoxVE)

Contact: Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-2-2
