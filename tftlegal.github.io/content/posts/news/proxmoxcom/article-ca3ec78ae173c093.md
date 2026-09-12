---
title: "Proxmox VE 5.1 with production-ready Ceph Lumious"
date: 2017-10-19T09:29:46Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-5-1"
summary: "Proxmox Server Solutions анонсировала выпуск Proxmox VE 5.1 — новой версии открытой платформы виртуализации.   Главное обновление — поддержка production-ready Ceph v12.2 Luminous, которая теперь стабильна для продуктивных сред и входит в подписку на корпоративную поддержку.   Релиз построен на Debian 9.2 и Linux-ядре 4.13, а пакеты Ceph теперь готовятся самой командой Proxmox для более быстрого получения исправлений и новых функций.   В версии также улучшена работа ZFS 0.7.2: оптимизирована статистика, добавлено возобновляемое zfs send/receive, снижено потребление CPU и ускорена работа аппаратной проверкой контрольных сумм.   Пользовательский интерфейс и управление получили ряд улучшений, включая упрощённое создание ВМ, выбор ВМ в диалоге high availability, импорт OVF из командной строки и поддержку SLES-контейнеров.   Proxmox VE 5.1 доступна для скачивания под лицензией GNU AGPL v3, а корпоративная поддержка предлагается по подписке от 69,90 евро в год за CPU.   Компания отмечает, что Proxmox VE остаётся ведущей open-source платформой для виртуализации с большой мировой пользовательской базой и поддержкой кластеризации, хранения, бэкапов и миграций."
---

# Proxmox VE 5.1 with production-ready Ceph Lumious

## Краткое содержание

Proxmox Server Solutions анонсировала выпуск Proxmox VE 5.1 — новой версии открытой платформы виртуализации.  
Главное обновление — поддержка production-ready Ceph v12.2 Luminous, которая теперь стабильна для продуктивных сред и входит в подписку на корпоративную поддержку.  
Релиз построен на Debian 9.2 и Linux-ядре 4.13, а пакеты Ceph теперь готовятся самой командой Proxmox для более быстрого получения исправлений и новых функций.  
В версии также улучшена работа ZFS 0.7.2: оптимизирована статистика, добавлено возобновляемое zfs send/receive, снижено потребление CPU и ускорена работа аппаратной проверкой контрольных сумм.  
Пользовательский интерфейс и управление получили ряд улучшений, включая упрощённое создание ВМ, выбор ВМ в диалоге high availability, импорт OVF из командной строки и поддержку SLES-контейнеров.  
Proxmox VE 5.1 доступна для скачивания под лицензией GNU AGPL v3, а корпоративная поддержка предлагается по подписке от 69,90 евро в год за CPU.  
Компания отмечает, что Proxmox VE остаётся ведущей open-source платформой для виртуализации с большой мировой пользовательской базой и поддержкой кластеризации, хранения, бэкапов и миграций.

## Полная статья

**VIENNA, Austria – October 24, 2017 –**Proxmox Server Solutions GmbH, developer of the open-source virtualization platform Proxmox Virtual Environment (VE), today announced the release of its version 5.1. Most important enhancement is the software-defined storage solution Ceph v12.2 Luminous which is now stable for production and included in the enterprise support agreement. Proxmox VE 5.1 is based on Debian 9.2 and comes with a 4.13 Linux kernel.

### Proxmox VE 5.1 with production-ready CEPH Luminous

Proxmox VE 5.1 comes with production-ready Ceph cluster packages. The virtualization platform integrates Ceph v12.2 Luminous, the long term stable release of the software-defined storage solution. Users can now implement Ceph clusters as distributed storage solution in production. Help and support is provided by the Proxmox team via the Proxmox VE subscription service. Ceph is a distributed object store and file system designed to provide excellent performance, reliability and scalability. The Ceph OSD storage backend Bluestore FS is the new default in Proxmox VE. Bluestore delivers more performance (up to 200 percent in certain use cases), full data check-summing, and it has built-in compression.

With Proxmox VE 5.1 the Proxmox VE Ceph cluster packages are now prepared by the Proxmox developers. This means that bugfixes can be packaged and used faster with Proxmox VE. The new packaging procedure allows users to access the newest Ceph features with Proxmox VE immediately, and they don’t have to wait for the packaging done by Ceph which sometimes took weeks.

### ZFS 0.7.2 Storage

After updating the integrated ZFS 0.7.2 in Proxmox VE 5.1 statistics are now optimized, and analyzing storage usage is even better now. The resumable “zfs send/receive” allows an interrupted ZFS receive to be resumed if the stream was prematurely terminated due to remote system or network failure. Also the new release brings reduced CPU usage and improved performance through hardware accelerated check summing.

Many other notable changes in Proxmox VE 5.1 can be seen on the web-based management interface. The dialog for “Create VM” has been improved and contents are merged together for easier selection. On the high availability management dialog VMs can now be selected via a drop-down menu instead of entering the identifier of a VM manually. The system management level has seen optimizations with shutdown (services optimized, bug fixes). The packaging format Open Virtualization Format (OVF) can now be imported directly via command line. SLES container (instead of openSUSE) are supported in Proxmox VE 5.1.

### Availability

Proxmox VE 5.1 is available for download now at[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads)
The Proxmox VE solution is licensed under the free software license GNU Affero GPL, v3. Enterprise support is available from the company Proxmox Server Solutions GmbH on a subscription basis starting at EUR 69,90 per year and CPU.

##### Further information:

- Forum announcement and release notes:[https://forum.proxmox.com](https://forum.proxmox.com/threads/proxmox-ve-5-1-released.37649/)[](http://pve.proxmox.com/wiki/Roadmap)
- Release note:[https://pve.proxmox.com/wiki/Roadmap#Proxmox_VE_5.1](https://pve.proxmox.com/wiki/Roadmap#Proxmox_VE_5.1)

**Facts**
The open source project Proxmox VE has a huge worldwide user base with over 180,000 hosts. The web-based management interface is translated into 19 languages. More than 38,000 members are active in the community support forum. Over 9,500 customers from companies regardless sector, size or industry have a Proxmox VE support subscription, a services offered by Proxmox Server Solutions GmbH.

**About Proxmox Virtual Environment**
Proxmox VE is the leading open-source platform for all-inclusive enterprise virtualization. With the central built-in web interface you can easily run VMs and containers, manage software-defined storage and networking functionality, high-availability clustering, and multiple integrated out-of-the-box tools like backup/restore, live migration, replication, and the firewall. Enterprises use the powerful yet easy-to-manage all-in-one solution Proxmox VE to meet the core requirements—less complexity, more elasticity— of today’s modern data centers ensuring to stay adaptable for future growth thanks to the flexible, modular and open architecture.

**About Proxmox Server Solutions GmbH**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises regardless of size, sector or industry use the stable, secure, scalable, and open Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**Contact:**Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-5-1
