---
title: "Proxmox VE 6.2 released"
date: 2020-04-21T10:19:02Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-6-2"
summary: "Proxmox Server Solutions объявила о выпуске Proxmox VE 6.2 — новой версии открытой платформы управления виртуализацией.   Обновление построено на Debian 10.4 и Linux 5.4 и включает актуальные версии QEMU 5.0, LXC 4.0, Ceph Nautilus и ZFS 0.8.3.   Среди ключевых улучшений — расширенные возможности веб-интерфейса, поддержка DNS-валидации Let's Encrypt, до восьми corosync-соединений и удобный фильтр хранимых данных по дате создания.   Для контейнеров добавлены шаблоны LXC 4.0, а для резервного копирования — поддержка быстрого алгоритма сжатия Zstandard.   Также улучшено управление пользователями и правами доступа, добавлена синхронизация LDAP и поддержка API-токенов с индивидуальными правами и сроком действия.   Proxmox VE 6.2 уже доступна для загрузки, а обновление с версий 4.x и 5.x можно выполнить через apt.   Решение позиционируется как гибкая и экономичная основа для построения современного software-defined data center."
---

# Proxmox VE 6.2 released

## Краткое содержание

Proxmox Server Solutions объявила о выпуске Proxmox VE 6.2 — новой версии открытой платформы управления виртуализацией.  
Обновление построено на Debian 10.4 и Linux 5.4 и включает актуальные версии QEMU 5.0, LXC 4.0, Ceph Nautilus и ZFS 0.8.3.  
Среди ключевых улучшений — расширенные возможности веб-интерфейса, поддержка DNS-валидации Let's Encrypt, до восьми corosync-соединений и удобный фильтр хранимых данных по дате создания.  
Для контейнеров добавлены шаблоны LXC 4.0, а для резервного копирования — поддержка быстрого алгоритма сжатия Zstandard.  
Также улучшено управление пользователями и правами доступа, добавлена синхронизация LDAP и поддержка API-токенов с индивидуальными правами и сроком действия.  
Proxmox VE 6.2 уже доступна для загрузки, а обновление с версий 4.x и 5.x можно выполнить через apt.  
Решение позиционируется как гибкая и экономичная основа для построения современного software-defined data center.

## Полная статья

**VIENNA, Austria – May 12, 2020 –**Proxmox Server Solutions GmbH today announced the general availability of Proxmox VE 6.2, the latest version of the open-source virtualization management platform. Proxmox VE 6.2 includes new features aimed at addressing issues facing modern datacenter administrators and IT teams. The new version of the virtualization management solution comes with a lot of new features, notable improvements, and many advanced options for the web-based user interface. It's based on Debian Buster 10.4 and a 5.4 longterm Linux kernel and includes updates to the latest versions of the leading open-source virtualization technologies QEMU 5.0, LXC 4.0, Ceph Nautilus (14.2.9), and ZFS 0.8.3.

### Highlights of Proxmox VE 6.2

**Advanced options for the web-based management interface**

- Proxmox VE implements built-in validation of domains for Let's Encrypt TLS certificates via the DNS-based challenge mechanism, in addition to the already existing HTTP-based validation mode.
- Full support for up to eight corosync network links is available. The more links are used, the higher the cluster availability.
- In the storage content view, administrators can now filter the stored data with the new column ‘Creation Date’ which, for example, simplifies to search for a backup from a certain date.
- The language of the web interface can be seamlessly changed without the need to restart the session. An Arabic translation has been added and thus Proxmox VE supports 20 languages in total.

**Linux Container**

- The integrated container technology has been updated to LXC 4.0.2 and lxcfs 4.0.3. Proxmox VE 6.2 now allows to create templates for containers on directory-based storage.
- New LXC templates for Ubuntu 20.04, Fedora 32, CentOS 8.1, Alpine Linux and Arch Linux.

**Zstandard for Backup/Restore**

- The integrated backup manager supports the fast and highly efficient lossless data compression algorithm Zstandard (zstd).

**User and permission management**

- Proxmox VE uses a role-based user and permission management for all objects such as VMs, storage, nodes, etc. The new LDAP sync enables synchronization of LDAP users and groups into the Proxmox user and group permission framework.
- Full support and the integration for API tokens has been added allowing stateless access to most parts of the REST API by another system, software or API client. API Tokens can be generated for individual users and can optionally be configured with separate permissions and expiration dates to limit the scope and duration of the access. Should the API token get compromised it can be revoked without having to disable the user itself.

### Further notable enhancements

- QEMU/KVM: Support for Live Migration with replicated disks (storage replication with zfs) is enabled.
- Testing the Ceph storage has become easier as the uninstall process has been simplified.

### Availability

Proxmox VE 6.2 is now available for download at[https://www.proxmox.com/downloads](https://www.proxmox.com/en/component/zoo/?view=frontpage&layout=frontpage&Itemid=120). Distribution upgrades from Proxmox VE 4.x or 5.x versions to 6.x are possible with apt.

Proxmox VE is licensed under the free software license GNU Affero GPL, v3. Enterprise support is available from Proxmox Server Solutions on a subscription basis starting at EUR 85 per year and CPU, see[https://www.proxmox.com](https://www.proxmox.com/en/products/proxmox-virtual-environment/pricing).

**Facts**
The open-source project Proxmox VE has a huge worldwide user base with more than 350,000 hosts. The virtualization platform is translated into over 20 languages. More than 60,000 active community members in the support forum engage with each other. By using Proxmox VE as an alternative to proprietary virtualization management solutions, enterprises are able to centralize and modernize their IT infrastructure and turn it into a cost-effective and flexible software-defined data center based on latest open-source technologies. Tens of thousands of customers from companies regardless of sector, size or industry rely on a Proxmox VE support subscription, a service offered by Proxmox Server Solutions GmbH.

**About Proxmox VE**
Proxmox VE is the leading open-source platform for all-inclusive enterprise virtualization. With the central web interface you can easily run VMs and containers, manage software-defined storage and networking functionality, high-availability clustering, and multiple integrated out-of-the-box tools like backup/restore, live migration, replication, and the firewall. Enterprises use the powerful yet easy-to-manage all-in-one solution Proxmox VE to meet the core requirements—less complexity, more elasticity— of today’s modern data centers ensuring to stay adaptable for future growth thanks to the flexible, modular and open architecture.

**About Proxmox Server Solutions**
Founded in 2005, Proxmox is a global provider of powerful yet easy-to-use open-source server software. Enterprises regardless of size, sector or industry use the stable, secure, scalable, and open Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox offers commercial support and training to ensure business continuity to its customers. Proxmox Server Solutions GmbH is headquartered in Vienna, Austria and has a global partner network with several hundreds of partners all around the world.

**Contact:**Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-6-2
