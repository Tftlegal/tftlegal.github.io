---
title: "Proxmox Backup Server 3.1 released"
date: 2023-11-24T15:18:36Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-3-1"
summary: "30 ноября 2023 года Proxmox представила версию 3.1 своего открытого решения Proxmox Backup Server для резервного копирования серверов. Программа работает с виртуальными машинами, контейнерами и физическими хостами, поддерживая инкрементальные бэкапы, дедупликацию, сжатие Zstandard и аутентифицированное шифрование. Новая версия построена на Debian 12.2 «Bookworm» с ядром Linux 6.5 и ZFS 2.2.0, а также получила улучшенный Secure Boot и поддержку локальных синхронизационных задач. Дополнительно улучшена совместимость системы ленточного резервного копирования с LTO 9 и некоторыми ленточными библиотеками, а веб-интерфейс дополнен переводами на хорватский и грузинский. Proxmox Backup Server 3.1 доступна для скачивания, устанавливается на bare-metal или поверх Debian, обновляется через APT и распространяется под лицензией GNU AGPLv3. Для корпоративных пользователей компания предлагает подписку на поддержку с доступом к Enterprise Repository, начиная с 495 евро за сервер."
---

# Proxmox Backup Server 3.1 released

## Краткое содержание

30 ноября 2023 года Proxmox представила версию 3.1 своего открытого решения Proxmox Backup Server для резервного копирования серверов. Программа работает с виртуальными машинами, контейнерами и физическими хостами, поддерживая инкрементальные бэкапы, дедупликацию, сжатие Zstandard и аутентифицированное шифрование. Новая версия построена на Debian 12.2 «Bookworm» с ядром Linux 6.5 и ZFS 2.2.0, а также получила улучшенный Secure Boot и поддержку локальных синхронизационных задач. Дополнительно улучшена совместимость системы ленточного резервного копирования с LTO 9 и некоторыми ленточными библиотеками, а веб-интерфейс дополнен переводами на хорватский и грузинский. Proxmox Backup Server 3.1 доступна для скачивания, устанавливается на bare-metal или поверх Debian, обновляется через APT и распространяется под лицензией GNU AGPLv3. Для корпоративных пользователей компания предлагает подписку на поддержку с доступом к Enterprise Repository, начиная с 495 евро за сервер.

## Полная статья

**VIENNA, Austria – November 30, 2023 –**Enterprise software developer Proxmox Server Solutions GmbH (henceforth "Proxmox") today announced version 3.1 of its open-source server backup solution. The enterprise solution for backing up and restoring VMs, containers, and physical hosts supports incremental backups, deduplication, Zstandard compression, and authenticated encryption. This version comes with secure boot compatibility, support for local sync jobs, and brings countless enhancements improving the general client and backend usability.

## What’s new in Proxmox Backup Server 3.1

- Proxmox Backup Server is based on**Debian 12.2 “Bookworm”**, but uses the newer**Linux kernel 6.5**as stable default. The backup platform comes with**ZFS 2.2.0**but already includes important fixes from the upcoming ZFS version 2.2.2.
- **Support for Secure Boot:**Like Proxmox Virtual Environment, this version is now compatible with Secure Boot. This security feature is designed to protect the boot process of a computer by ensuring that only software with a valid digital signature launches on a machine. Proxmox Backup Server now includes a signed shim bootloader trusted by most hardware's UEFI implementations. This allows installing it in environments with Secure Boot active. Existing Backup Server installations can be switched over to Secure Boot without reinstallation.
- **Support for local sync jobs:**A local sync job enables synchronization of backups between local datastores. Sync jobs can now pull contents not only from remote Proxmox Backup Server instances, but also from local datastores.
- The**Proxmox Tape Backup system**provides an easy way to copy datastore content to tapes. This version has improved compatibility with LTO 9 tapes and with certain tape libraries.
- The web interface is now available in Croatian and Georgian; other translations have been improved thanks to community contributions.

Proxmox Backup Server seamlessly integrates into Proxmox Virtual Environment – users just need to add the storage for the Proxmox Backup Server as a new storage backup target to Proxmox VE. The software is published under the GNU AGPLv3 license.

### Availability

The centrally managed software stack Proxmox Backup Server 3.1 is now available for download at[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads). The downloadable ISO image can be quickly installed on bare-metal using the installation wizard. Distribution upgrades from older versions of Proxmox Backup Server are possible via APT. It is also possible to install Proxmox Backup Server on top of Debian.

License: Proxmox Backup Server is free and open-source software, published under the GNU AGPLv3.

Support Subscriptions: For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 495 per server, including unlimited backup storage and unlimited backup-clients.

##### Further information:

- Find all details in the[Release Notes](https://pbs.proxmox.com/wiki/index.php/Roadmap#Proxmox_Backup_Server_3.1)
- Read the[Forum Announcement](https://forum.proxmox.com/threads/proxmox-backup-server-3-1-available.137371/)

###

**About Proxmox Server Solutions**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training services to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**About Proxmox Backup Server**
Proxmox Backup Server is an enterprise backup solution for backing up and restoring virtual machines, containers, and physical hosts. The open-source client-server software supports incremental backups, deduplication, Zstandard compression, and authenticated encryption. To increase productivity, the easy-to-use Proxmox Backup Server allows you to back up your data in a space-efficient manner, restore VMs, archives or single objects in a flash. With the web-based user interface, you can effectively reduce work hours thanks to simplified management. Proxmox Backup Server is licensed under the GNU AGPL, v3. Enterprise support subscriptions are available from Proxmox.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-3-1
