---
title: "Proxmox VE 5.3 with CephFS released"
date: 2018-11-16T10:18:31Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-5-3"
summary: "4 декабря 2018 года Proxmox Server Solutions представила Proxmox VE 5.3 — новое обновление открытой платформы управления серверной виртуализацией.   Система построена на Debian Stretch 9.6 и Linux-ядре 4.15, а Ceph Storage обновлен до стабильной версии 12.2.8.   Главной новинкой стала поддержка CephFS в веб-интерфейсе: распределенная файловая система теперь может использоваться для резервных копий, ISO-образов и шаблонов контейнеров.   CephFS позволяет отказаться от внешних файловых хранилищ, таких как NFS или Samba, упрощая управление и снижая стоимость инфраструктуры.   Для работы CephFS требуется действующий Ceph-кластер и узлы Ceph Metadata Server, а несколько MDS-узлов помогают обеспечить высокую доступность.   Кроме того, версия 5.3 улучшает управление дисками, добавляет вложенность LXC-контейнеров, доступ к NFS/CIFS внутри контейнеров и упрощенную настройку PCI passthrough и vGPU через веб-интерфейс.   Proxmox VE 5.3 доступна для бесплатного скачивания под лицензией GNU AGPL v3, а платная корпоративная поддержка предоставляется по подписке."
---

# Proxmox VE 5.3 with CephFS released

## Краткое содержание

4 декабря 2018 года Proxmox Server Solutions представила Proxmox VE 5.3 — новое обновление открытой платформы управления серверной виртуализацией.  
Система построена на Debian Stretch 9.6 и Linux-ядре 4.15, а Ceph Storage обновлен до стабильной версии 12.2.8.  
Главной новинкой стала поддержка CephFS в веб-интерфейсе: распределенная файловая система теперь может использоваться для резервных копий, ISO-образов и шаблонов контейнеров.  
CephFS позволяет отказаться от внешних файловых хранилищ, таких как NFS или Samba, упрощая управление и снижая стоимость инфраструктуры.  
Для работы CephFS требуется действующий Ceph-кластер и узлы Ceph Metadata Server, а несколько MDS-узлов помогают обеспечить высокую доступность.  
Кроме того, версия 5.3 улучшает управление дисками, добавляет вложенность LXC-контейнеров, доступ к NFS/CIFS внутри контейнеров и упрощенную настройку PCI passthrough и vGPU через веб-интерфейс.  
Proxmox VE 5.3 доступна для бесплатного скачивания под лицензией GNU AGPL v3, а платная корпоративная поддержка предоставляется по подписке.

## Полная статья

**VIENNA, Austria – December 04, 2018 –**Proxmox Server Solutions GmbH today unveiled Proxmox VE 5.3, its latest open-source server virtualization management platform. Proxmox VE is based on Debian Stretch 9.6 with a modified Linux Kernel 4.15. Ceph Storage has been updated to version 12.2.8 (Luminous LTS, stable), and is packaged by Proxmox.

### Proxmox VE and CephFS

Proxmox VE 5.3 now includes CephFS in its web-based management interface thus expanding its comprehensive list of already supported file and block storage types. CephFS is a distributed, POSIX-compliant file system and builds on the Ceph cluster. Like Ceph RBD (Rados Block Device), which is already integrated into Proxmox VE, CephFS now serves as an alternative interface to the Ceph storage. For CephFS Proxmox allows storing VZDump backup files, ISO images, and container templates. The distributed file system CephFS eliminates the need for external file storage such as NFS or Samba and thus helps reducing hardware cost and simplifies management.

The CephFS file system can be created and configured with just a few clicks in the Proxmox VE management interface. To deploy CephFS users need a working Ceph storage cluster and a Ceph Metadata Server (MDS) node, which can also be created in the Proxmox VE interface. The MDS daemon separates metadata and data from each other and stores them in the Ceph file system. At least one MDS is needed, but its recommended to deploy multiple MDS nodes to improve high availability and avoid SPOF. If several MDS nodes are created only one will be marked as ‘active’ while the others stay ‘passive’ until they are needed in case of failure of the active one.

### Further improvements in Proxmox VE 5.3

Proxmox VE 5.3 brings many improvements in storage management. Via the Disk management it is possible to easily add ZFS raid volumes, LVM, and LVMthin pools as well as additional simple disks with a traditional file system. The existing ZFS over iSCSI storage plug-in can now access LIO target in the Linux kernel. Nesting is enabled for LXC containers making it possible to use LXC or LXD inside a container. Also, access to NFS or CIFS/Samba server can be configured inside containers. For the keen and adventurous user, Proxmox VE brings a simplified configuration of PCI passthrough and virtual GPUs (vGPUs such as Intel KVMGT)–now even possible via the web GUI.

Countless bugfixes and smaller improvements are listed in the release notes and can be found in detail in the Proxmox bugtracker or in the Git repository.

### Availability

Proxmox VE 5.3 is now available for download at https://www.proxmox.com/downloads. Proxmox VE comes with the free software license GNU AGPL, v3. Enterprise support is available from Proxmox Server Solutions on a subscription basis starting at EUR 74,90 per year and CPU.

##### Further information:

- Forum announcement:[https://forum.proxmox.com](https://forum.proxmox.com/forums/announcements.7/)[](http://pve.proxmox.com/wiki/Roadmap)
- Release note:[https://pve.proxmox.com/wiki/Roadmap#Proxmox_VE_5.3](https://pve.proxmox.com/wiki/Roadmap#Proxmox_VE_5.3)
- Video tutorial:["What's new in Proxmox VE 5.3"](https://www.proxmox.com/en/training/video-tutorials/item/what-s-new-in-proxmox-ve-5-3)

**Facts**
The open-source project Proxmox VE has a huge worldwide user base with over 230,000 installations. The web-based management interface is translated into 20 languages. More than 40,000 members are active in the community support forum. Over 13,000 customers from companies regardless of size, sector or industry rely on the Proxmox VE support services offered by Proxmox Server Solutions GmbH.

**About Proxmox Virtual Environment**
Proxmox VE is the leading open-source platform for all-inclusive enterprise virtualization. With the central built-in web interface you can easily run VMs and containers, manage software-defined storage and networking functionality, high-availability clustering, and multiple integrated out-of-the-box tools like backup/restore, live migration, replication, and the firewall. Enterprises use the powerful yet easy-to-manage all-in-one solution Proxmox VE to meet the core requirements—less complexity, more elasticity— of today’s modern data centers ensuring to stay adaptable for future growth thanks to the flexible, modular and open architecture.

**About Proxmox Server Solutions GmbH**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises regardless of size, sector or industry use the stable, secure, scalable, and open Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**Contact:**Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-5-3
