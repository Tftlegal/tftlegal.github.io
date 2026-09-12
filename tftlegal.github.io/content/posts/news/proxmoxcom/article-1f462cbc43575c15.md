---
title: "Proxmox VE 6.3 with Proxmox Backup Server Integration and Ceph Octopus released"
date: 2020-11-13T17:02:45Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-6-3"
summary: "26 ноября 2020 года Proxmox Server Solutions выпустила версию 6.3 своей open-source платформы виртуализации Proxmox VE. Новая версия построена на Debian Buster 10.6 и ядре Linux 5.4 и включает обновления ключевых технологий, таких как QEMU 5.1, LXC 4.0, Ceph и ZFS. Главной новинкой стала интеграция Proxmox Backup Server 1.0, который обеспечивает быстрые инкрементальные, дедуплицированные и шифрованные резервные копии виртуальных машин, контейнеров и физических хостов. В Proxmox VE 6.3 также улучшена поддержка Ceph Octopus и Ceph Nautilus, добавлены новые функции управления хранилищем и расширены возможности веб-интерфейса. Обновление расширяет поддержку контейнеров, управление пользователями и разрешениями, настройки резервного копирования и мониторинг, а также упрощает администрирование кластеров. Proxmox VE 6.3 доступна для загрузки, поддерживает обновление через apt и распространяется под лицензией GNU Affero GPL v3, с возможностью подключения платной корпоративной поддержки."
---

# Proxmox VE 6.3 with Proxmox Backup Server Integration and Ceph Octopus released

## Краткое содержание

26 ноября 2020 года Proxmox Server Solutions выпустила версию 6.3 своей open-source платформы виртуализации Proxmox VE. Новая версия построена на Debian Buster 10.6 и ядре Linux 5.4 и включает обновления ключевых технологий, таких как QEMU 5.1, LXC 4.0, Ceph и ZFS. Главной новинкой стала интеграция Proxmox Backup Server 1.0, который обеспечивает быстрые инкрементальные, дедуплицированные и шифрованные резервные копии виртуальных машин, контейнеров и физических хостов. В Proxmox VE 6.3 также улучшена поддержка Ceph Octopus и Ceph Nautilus, добавлены новые функции управления хранилищем и расширены возможности веб-интерфейса. Обновление расширяет поддержку контейнеров, управление пользователями и разрешениями, настройки резервного копирования и мониторинг, а также упрощает администрирование кластеров. Proxmox VE 6.3 доступна для загрузки, поддерживает обновление через apt и распространяется под лицензией GNU Affero GPL v3, с возможностью подключения платной корпоративной поддержки.

## Полная статья

**VIENNA, Austria – November 26, 2020 –**Proxmox Server Solutions GmbH, developer of open-source enterprise software, has today released version 6.3 of its server virtualization management platform, Proxmox VE. The new version is based on Debian Buster 10.6, but uses the latest long-term support Linux kernel (5.4), and includes updates to the latest versions of leading open-source technologies for virtual environments like QEMU 5.1, LXC 4.0, Ceph 15.2, and ZFS 0.85.

### Integration for Proxmox Backup Server

The most notable new feature is the integration of the stable version 1.0 of Proxmox Backup Server, the company’s new, open-source, enterprise backup solution, for backing up and restoring VMs, containers, and physical hosts. Proxmox Backup Server supports incremental, fully deduplicated backups and facilitates strong encryption. The virtualization platform Proxmox VE is fully supported, allowing users to easily backup virtual machines and containers – even between remote locations. For virtual machines, Proxmox VE uses QEMU dirty-bitmaps, meaning backups from the Proxmox VE client to the Proxmox Backup Server are very fast, as only the changed data is transmitted.

In Proxmox VE 6.3, users just need to add the Proxmox Backup Server datastore as a new storage target. All client-to-server traffic can be encrypted on the client-side, to safeguard data before backing up to Proxmox Backup Server. The backup client makes creating and managing encryption keys simple. The program offers multiple ways to store keys, so that they remain safe and can be quickly obtained when needed:: Users can save key files to a secure file server or USB drive, copy the key as text into a password manager, or print the key on paper and, for example, store it in a safe.

### Support for Ceph Octopus and Ceph Nautilus

Proxmox VE 6.3 supports Ceph Octopus 15.2.6 and Ceph Nautilus 14.2.15. Users can select their preferred Ceph version during the installation process. Many new Ceph-specific management features have been added to the Proxmox VE dashboard during the Ceph Octopus development cycle, including the displaying of recovery progress in the Ceph status panel.

In the new version, it’s now possible to view and set the placement groups (PGs) auto-scaling mode for each Ceph pool in the storage cluster. This brings a lot of flexibility to the Ceph storage cluster and reduces the maintenance effort. Ceph has been integrated in Proxmox VE since 2014 with version 3.2, and thanks to the Proxmox VE user interface, installing and managing Ceph clusters is very easy. Ceph Octopus now adds significant multi-site replication capabilities, that are important for large-scale redundancy and disaster recovery.

### Enhanced Web Interface

With the integrated, web-based user interface, Proxmox VE makes configuring, managing, and monitoring the virtualized datacenter very straightforward, and with version 6.3 even more functionality and usability enhancements have been added. This includes:

- Editing external metric servers: Proxmox VE nodes can be easily connected to InfluxDB or Graphite via the GUI to facilitate monitoring.
- Improved editor for VM boot order: It is now possible to select multiple devices per type (disk, network) for booting. The user experience has been improved with drag-and-drop functionality.
- Optional TLS certificate verification for LDAP and AD authentication realms.
- Backup/Restore: Users can get an overview of all guests, which aren’t included in any backup at any backup state. Also, a detailed view per backup job is available, showing all covered guests and the disks which are backed up.
- Optional comments for all storage types can be displayed in the web interface. Additionally, the Proxmox Backup Server displays the verification state of all backed up snapshots.

### Further enhancements

- Storage:

- The new backup retention settings allow users to adapt and control retention periods based on their business needs. For each storage or backup job, users can implement enhanced retention policies, and decide on how many backups to keep per time frame.
- To monitor SSD wear leveling, querying has been improved.

- Container: Proxmox VE 6.3 now supports systems up to 8192 cores and officially supports Kali Linux and Devuan distribution containers, as well as the latest versions of Ubuntu, Fedora and CentOS. In addition, features such as per-container timezone support and improved startup monitoring have been added.
- Many improvements to user and permission management have been included, such as support for using client certificates/keys when connecting to AD/LDAP realms and support for optional case-insensitive logins with AD/LDAP realms.
- General improvements for virtual guests include improved handling of replicated guests when migrating.
- Firewall: Improved API and GUI for matching ICMP-types.
- Installer: Reboot automatically upon successful installation.

### Availability

Proxmox VE 6.3 is available for download at[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads)

Distribution upgrades from some older versions of Proxmox VE are possible with apt. It’s also possible to install Proxmox VE 6.3 on top of Debian Buster.
Proxmox VE is published under the free software license GNU Affero GPL, v3. Enterprise support is available from Proxmox Server Solutions on a subscription basis, starting at EUR 85 per year, per CPU. For more information, please visit[https://www.proxmox.com/pricing](https://www.proxmox.com/en/products/proxmox-virtual-environment/pricing)

Facts
The open-source project Proxmox VE has a huge worldwide user base with more than 390,000 hosts. The virtualization platform has been translated into over 26 languages. More than 55,000 active community members in the support forum engage with each other. By using Proxmox VE as an alternative to proprietary virtualization management solutions, enterprises are able to centralize and modernize their IT infrastructure, and turn it into a cost-effective and flexible software-defined data center, based on the latest open-source technologies. Tens of thousands of customers from companies regardless of sector, size or industry rely on a Proxmox VE support subscription, a service offered by Proxmox Server Solutions GmbH.

About Proxmox VE
Proxmox VE is the leading open-source platform for all-inclusive enterprise virtualization. With the central web interface, you can easily run VMs and containers, manage software-defined storage and networking functionality, high-availability clustering, and multiple integrated out-of-the-box tools like backup/restore, live migration, replication, and the firewall. Enterprises use the powerful yet easy-to-manage, all-in-one solution, Proxmox VE, to meet the core requirements of today’s modern data centers. Proxmox VE allows them to remain adaptable for future growth, thanks to its flexible, modular and open architecture.

About Proxmox Server Solutions
Proxmox is a provider of powerful yet easy-to-use, open-source server software. Enterprises regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

Press contact
Daniela Häsler, Proxmox Server Solutions GmbH

Follow us:

- LinkedIn:[https://www.linkedin.com/company/proxmox](https://www.linkedin.com/company/proxmox)
- YouTube:[https://www.youtube.com/user/ProxmoxVE](https://www.youtube.com/user/ProxmoxVE)

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-6-3
