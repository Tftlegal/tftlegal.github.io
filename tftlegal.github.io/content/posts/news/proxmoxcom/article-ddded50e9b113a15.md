---
title: "Proxmox VE 5.0 with new open-source storage replication stack"
date: 2017-07-03T18:30:22Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-5-0"
summary: "Proxmox Server Solutions выпустила Proxmox VE 5.0, основанную на Debian 9 и Linux Kernel 4.10.   Главная новинка — встроенный open-source-стек асинхронной репликации хранилищ, который упрощает создание резервных копий данных между узлами через веб-интерфейс.   Обновление также улучшает интеграцию Ceph RBD, которую Proxmox теперь сама упаковывает и поддерживает как стандартное распределенное хранилище.   Пользователи получили упрощенный импорт дисков из VMware, Hyper-V и других гипервизоров, а также live-миграцию с локальным хранилищем через QEMU.   Дополнительно улучшен веб-интерфейс, добавлены фильтры, массовые действия и отображение USB/PCI-адресов.   Proxmox VE 5.0 доступна для скачивания, поддерживает обновление через apt и распространяется под лицензией AGPL v3, с платной корпоративной поддержкой."
---

# Proxmox VE 5.0 with new open-source storage replication stack

## Краткое содержание

Proxmox Server Solutions выпустила Proxmox VE 5.0, основанную на Debian 9 и Linux Kernel 4.10.  
Главная новинка — встроенный open-source-стек асинхронной репликации хранилищ, который упрощает создание резервных копий данных между узлами через веб-интерфейс.  
Обновление также улучшает интеграцию Ceph RBD, которую Proxmox теперь сама упаковывает и поддерживает как стандартное распределенное хранилище.  
Пользователи получили упрощенный импорт дисков из VMware, Hyper-V и других гипервизоров, а также live-миграцию с локальным хранилищем через QEMU.  
Дополнительно улучшен веб-интерфейс, добавлены фильтры, массовые действия и отображение USB/PCI-адресов.  
Proxmox VE 5.0 доступна для скачивания, поддерживает обновление через apt и распространяется под лицензией AGPL v3, с платной корпоративной поддержкой.

## Полная статья

**VIENNA, Austria – July 4, 2017 –**Proxmox Server Solutions GmbH, developer of the open-source platform for enterprise virtualization Proxmox VE, today announced the general availability of its major release Proxmox VE 5.0. New main features are the open-source storage replication stack for asynchronous replication, and updates to the fully integrated distributed storage Ceph RBD, now packaged by the Proxmox team. With Proxmox VE 5.0 the import of disk images from other hypervisors such as VMware or Hyper-V to Proxmox VE has become far easier. The new version 5.0 is based on Debian 9 codename “Stretch” and a specially modified Linux Kernel 4.10.

### New Proxmox VE storage replication stack

Proxmox VE 5.0 introduces a new open-source storage replication stack for enterprise virtualization workloads, fully integrated into the easy-to-manage web management interface. Replicas provide asynchronous data replication between two or multiple nodes in a cluster, thus minimizing data loss in case of failure. The storage replication feature is especially designed to help smaller organizations to improve reliability, fault-tolerance, and accessibility of their data. For all organizations using local storage the Proxmox replication feature is a great option to increase data redundancy for high I/Os avoiding the need of complex shared or distributed storage configurations.

Proxmox users can now easily create, modify and remove replication jobs on the web GUI. The flexible Proxmox replication scheduler is based on the systemd time calendar event format allowing to configure scheduling, frequency and times. The minimum replication time can be even set to one minute. Replication also speeds up migration of data between nodes. This new tightly integrated Proxmox VE storage replication technology sets the base for many new features in future Proxmox VE releases.

### Ceph – massively scalable, software-defined storage

With Proxmox VE version 5.0 Ceph Rados Block Device (RBD) becomes the de-facto standard for distributed storage in Proxmox VE. Ceph is a highly scalable software-defined storage solution integrated with VMs and containers into Proxmox VE since 2013. It enables organizations to deploy and manage compute (VMs and containers) and storage centrally from Proxmox VE web interface removing the need for additional costly storage infrastructure and leveraging hyper-converged infrastructures.
The new version Proxmox VE 5.0 now integrates the Ceph packages directly into the Proxmox VE repository. Packaging is done by the Proxmox developers instead of getting the deb packages from ceph.com. This will allow Proxmox to roll-out future bug-fixes as soon as they are available to its users.

Supporting Quotes: Martin Maurer, founder and CEO, Proxmox:
“Our users love the Ceph integration shipped with Proxmox VE. Since we first implemented the powerful distributed storage in 2013 with Proxmox VE 2.3, we see numerous community members running Ceph implementations in production, also with blazing fast NVMe SSDs. Thanks to their input we could extremely improve the integration over the last four years, and the result is now a perfectly integrated Ceph storage support into the Proxmox storage stack enabling enterprises to manage their Ceph storage directly from the Proxmox VE interface. For the future, we are extremely excited about what’s coming next from the great Ceph community, just thinking about CephFS. We have already tested the new ‘bluestore’-format in the Ceph Luminous 12 release candidate and we have seen speed increase by up to 50% in some of our setups! We plan to provide the new ‘bluestore’-OSD format as default in Proxmox VE as soon as Ceph Luminous is production ready, later this summer.”

### Other notable changes in Proxmox VE 5.0

Another new feature in Proxmox VE 5.0 is the simplified import procedure for disk images from different hypervisors. Users can easily import disks from VMware, Hyper-V, or other hypervisors via a new command line tool called ‘qm importdisk’.
Other important improvement is the new live migration with local storage via QEMU. Also, the GUI team has made many updates to the web interface such as added USB und Host PCI address visibility, and bulk actions now visible in the GUI. The screen size of the console has been adjusted, and filtering options have been integrated to the GUI.

### Availability

Proxmox VE 5.0 is available now and ready for download at[https://www.proxmox.com/downloads.](https://www.proxmox.com/en/component/zoo/?view=frontpage&layout=frontpage&Itemid=120)Distribution upgrades from Proxmox VE 4.4 or 5.0 beta versions are possible with apt. The Proxmox VE solution is licensed under the free software license GNU Affero GPL, v3. Enterprise support is available from Proxmox Server Solutions on a subscription basis starting at EUR 69,90 per year and CPU.

##### Further information:

- Forum announcement and release notes:[https://forum.proxmox.com](https://forum.proxmox.com/threads/proxmox-ve-5-0-released.35450/)[](http://pve.proxmox.com/wiki/Roadmap)
- Watch the video["What's new in Proxmox VE 5.0"](https://www.proxmox.com/en/component/zoo/?view=frontpage&layout=frontpage&Itemid=301)

**About Proxmox Virtual Environment**
Proxmox VE is the leading open-source platform for all-inclusive enterprise virtualization. With the central built-in web interface you can easily run VMs and containers, manage software-defined storage and networking functionality, high-availability clustering, and multiple integrated out-of-the-box tools like backup/restore, live migration, replication, and the firewall. Enterprises use the powerful yet easy-to-manage all-in-one solution Proxmox VE to meet the core requirements—less complexity, more elasticity— of today’s modern data centers ensuring to stay adaptable for future growth thanks to the flexible, modular and open architecture.

**About Proxmox Server Solutions GmbH**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises regardless of size, sector or industry use the stable, secure, scalable, and open Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**Contact:**Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-5-0
