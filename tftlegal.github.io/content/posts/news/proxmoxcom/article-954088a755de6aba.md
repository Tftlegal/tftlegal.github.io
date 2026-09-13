---
title: "Proxmox Backup Server 1.0 available"
date: 2020-11-09T17:11:38Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-1-0-available"
summary: "11 ноября 2020 года компания Proxmox Server Solutions объявила о первом стабильном выпуске Proxmox Backup Server 1.0. Это открытое корпоративное решение для резервного копирования и восстановления виртуальных машин, контейнеров и физических хостов. Система построена по клиент-серверной модели и поддерживает инкрементальные, дедуплицированные, сжатые и зашифрованные резервные копии. Proxmox Backup Server интегрируется с Proxmox VE, предлагает веб-интерфейс для управления хранилищами, задачами и восстановлением, а также средства контроля доступа и удаленной синхронизации. Для обеспечения целостности данных используются шифрование AES-256-GCM на стороне клиента, контрольные суммы SHA-256 и возможность быстрого восстановления отдельных объектов. Программа распространяется бесплатно под лицензией GNU AGPL v3, а платная корпоративная поддержка доступна по подписке."
---

# Proxmox Backup Server 1.0 available

## Краткое содержание

11 ноября 2020 года компания Proxmox Server Solutions объявила о первом стабильном выпуске Proxmox Backup Server 1.0.
Это открытое корпоративное решение для резервного копирования и восстановления виртуальных машин, контейнеров и физических хостов.
Система построена по клиент-серверной модели и поддерживает инкрементальные, дедуплицированные, сжатые и зашифрованные резервные копии.
Proxmox Backup Server интегрируется с Proxmox VE, предлагает веб-интерфейс для управления хранилищами, задачами и восстановлением, а также средства контроля доступа и удаленной синхронизации.
Для обеспечения целостности данных используются шифрование AES-256-GCM на стороне клиента, контрольные суммы SHA-256 и возможность быстрого восстановления отдельных объектов.
Программа распространяется бесплатно под лицензией GNU AGPL v3, а платная корпоративная поддержка доступна по подписке.

## Полная статья

**VIENNA, Austria – November 11, 2020 –**Proxmox Server Solutions GmbH has announced today the first stable release of its new, open-source server backup solution. Proxmox Backup Server 1.0 is an enterprise backup software solution to back up and restore virtual machines, containers, and physical hosts. It supports incremental, fully deduplicated backups, compression and authenticated encryption. The whole software stack is written in Rust, a modern, fast, and memory-efficient language. Proxmox Backup Server is based on Debian Buster 10.6, but using the latest long-term support Linux kernel (5.4), and including ZFS 0.8.4.

Designed as a client-server system, Proxmox Backup Server allows for storing data on-premises and remotely. This separation allows multiple, unrelated hosts to use the backup server, and, while the server stores the backup data and provides an API to create and manage datastores, the client tool allows the user to create and manage backups from all the hosts.

### Key Features

- **Incremental, Deduplicated Backups:**Proxmox backups are sent incrementally from the client to the Proxmox Backup Server, where data is deduplicated. Typically, changes between periodic backups are low. Reading and sending only the changes reduces the storage space used and the network impact. While periodic backups usually produce large amounts of duplicate data, the deduplication layer in the Proxmox Backup solution reduces that amount, reducing the required space for data storage too.
- **Proxmox VE Integration:**The virtualization platform Proxmox VE is fully supported allowing you can easily backup virtual machines (supporting QEMU dirty bitmaps) and containers – even between remote locations. In Proxmox VE, users just need to add the Proxmox Backup Server datastore as a new storage backup target.
- **Administration:**Proxmox Backup Server comes stocked with an integrated, web-based user interface to manage all backup and restore tasks on the server. Administration tasks are carried out through a web browser. In the web interface, users can create and manage datastores, browse and restore backups, manage network configuration and interfaces, and get an overview of performance, backup tasks, and usage statistics.
- **Data Integrity:**All client-to-server traffic in Proxmox Backup Server will be encrypted to safeguard data integrity and ensure that the data is not compromised. For high performance on modern hardware, the authenticated encryption is done on the client-side with AES-256 in Galois/Counter mode (GCM). By using a master key (RSA public/private key pair), an encrypted version of the encryption key can be stored alongside each backup and recovered later, should the original key be lost. Additionally, this master key can be printed on paper so that it's safe from any system disaster.
As data is encrypted before it reaches the server, it is useless to unauthorized users accessing the server. This is important in case you want to back up to targets which are not fully trusted, like for example, rented servers at a colocation facility.
- **Checksum Algorithm:**To ensure the accuracy and consistency of the data, Proxmox Backup Server uses a built-in SHA-256 checksum algorithm. This algorithm is used in backup verification to detect bit rot and confirm backups are safe. It’s also employed to find common data between backups of different machines (for example, multiple VMs with identical operating systems) - and store the data efficiently, only once.
- **Quick Restore with Granular Recovery:**The need to minimize downtime and to quickly recover from data loss or ransomware attacks is a central aspect of IT administration. Proxmox Backup Server provides fast and simple restore via the web interface. With granular recovery options and via an interactive recovery shell, you can have a VM, archive, or even single file back in seconds.
- **Remote Synchronization:**For redundancy, Proxmox Backup Server enables you to pull or synchronize datastores from other locations. This is an efficient method to synchronize data to or from remote hosts. Only changes since the previous sync get transferred.
- **Compression:**The ultra-fast Zstandard (ZSTD) compression is able to compress several gigabytes of data per second. ZSTD is characterized by high compression ratio and very fast compression speed.
- **E-mail Notification:**for scheduled background tasks (verification, pruning, garbage collection, sync jobs).
- **Access Control:**A range of access control options are available to ensure users are limited to the rules the administrator provides to them. With one realm for system users and another realm for permissions and data ownership, the admin can specify exactly what each user is allowed to do on the server.

### Availability

Proxmox Backup Server 1.0 is available for download at[https://www.proxmox.com/downloads](https://www.proxmox.com/en/component/zoo/?view=frontpage&layout=frontpage&Itemid=120). The downloadable ISO image can be installed on bare-metal. For a production setup, dedicated hardware is recommended.

The backup server comes stocked with all the essential management tools, as well as an easy-to-use, web-based interface. This allows for simple, out-of-the-box management of the server, either through the command line or a standard web browser.

Proxmox Backup Server is free and open-source software, published under the GNU Affero General Public License, v3. Enterprise support is available from the Proxmox team on an subscription-based model. A subscription provides access to the stable enterprise repository package and to several technical support levels, starting at EUR 449.

**Forum Announcement**
[https://forum.proxmox.com/threads/proxmox-backup-server-1-0-stable.78851/](https://forum.proxmox.com/threads/proxmox-backup-server-1-0-stable.78851/)

**About Proxmox Backup Server**
Proxmox Backup Server is an enterprise backup solution for backing up and restoring virtual machines, containers, and physical hosts. The open-source client-server software supports incremental backups, deduplication, Zstandard compression, and authenticated encryption. To increase productivity, the easy-to-use Proxmox Backup Server allows you to back up your data in a space-efficient manner, restore VMs, archives or single objects in a flash. With the web-based user interface you can effectively reduce work hours thanks to simplified management. Proxmox Backup Server is licensed under the GNU AGPL, v3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions
**Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, scalable, and open Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-1-0-available
