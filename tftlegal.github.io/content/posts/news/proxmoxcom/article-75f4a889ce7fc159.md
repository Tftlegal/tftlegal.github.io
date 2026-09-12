---
title: "Proxmox VE 6.0 with Ceph Nautilus and Corosync 3"
date: 2019-07-16T08:25:33Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-6-0"
summary: "16 июля 2019 года Proxmox Server Solutions выпустила крупную версию Proxmox VE 6.0 — открытую платформу для управления виртуализацией и построения программно-определяемого центра обработки данных на базе Debian 10. Обновление включает актуальные компоненты: Linux 5.0, QEMU 4.0, LXC 3.1, Ceph 14.2, ZFS 0.8.1 и Corosync 3. Среди главных нововведений — улучшенное управление Ceph Nautilus через веб-интерфейс, новый кластерный стек Corosync 3 с приоритизацией сетей и поддержка ZFS 0.8.1 с нативным шифрованием и TRIM для SSD. В Proxmox VE 6 также добавлены загрузка ZFS через UEFI/NVMe в установщике, новые возможности QEMU, включая live migration локальных дисков и улучшенная поддержка Windows, а также пользовательские конфигурации Cloud-init. К другим изменениям относятся автоматическая очистка старых ядер, отображение статусов гостей,"
---

# Proxmox VE 6.0 with Ceph Nautilus and Corosync 3

## Краткое содержание

16 июля 2019 года Proxmox Server Solutions выпустила крупную версию Proxmox VE 6.0 — открытую платформу для управления виртуализацией и построения программно-определяемого центра обработки данных на базе Debian 10.
Обновление включает актуальные компоненты: Linux 5.0, QEMU 4.0, LXC 3.1, Ceph 14.2, ZFS 0.8.1 и Corosync 3.
Среди главных нововведений — улучшенное управление Ceph Nautilus через веб-интерфейс, новый кластерный стек Corosync 3 с приоритизацией сетей и поддержка ZFS 0.8.1 с нативным шифрованием и TRIM для SSD.
В Proxmox VE 6 также добавлены загрузка ZFS через UEFI/NVMe в установщике, новые возможности QEMU, включая live migration локальных дисков и улучшенная поддержка Windows, а также пользовательские конфигурации Cloud-init.
К другим изменениям относятся автоматическая очистка старых ядер, отображение статусов гостей,

## Полная статья

**VIENNA, Austria – July 16, 2019 –**Proxmox Server Solutions GmbH, developer of the open-source virtualization management platform Proxmox VE, today released its major version Proxmox VE 6.0. The comprehensive solution, designed to deploy an open-source software-defined data center (SDDC), is based on Debian 10.0 Buster. It includes updates to the latest versions of the leading open-source technologies for virtual environments like a 5.0 Linux kernel (based on Ubuntu 19.04 "Disco Dingo"), QEMU 4.0.0, LXC 3.1.0, Ceph 14.2 (Nautilus), ZFS 0.8.1, and Corosync 3.0.2. Proxmox VE 6.0 delivers several new major features, enhancements, and bug fixes.

### What’s new in Proxmox VE 6

- **Ceph Nautilus (14.2) and improved Ceph dashboard management:**Proxmox VE allows to setup and manage a hyperconverged infrastructure with a Proxmox VE/Ceph-cluster. Version 6 integrates the features of the latest Ceph 14.2 release, and also brings many new management functionality to the web-based user interface. This includes: a cluster-wide overview for Ceph being displayed in the ‘Datacenter View’; a new donut chart visualizing the activity and state of the placement groups (PGs); the version of all Ceph services is displayed, making detection of outdated services easier; the configuration settings from config file and databases can be displayed; users can select the public and cluster networks in the web interface with a new network selector; encryption for OSDs can be activated easily during creation with a checkbox.
- **Cluster communication stack with Corosync 3 using Kronosnet:**with Proxmox VE 6.0 the cluster communication stack has been updated to Corosync 3 by which the on-the-wire format has changed. Corosync now uses unicast as default transport method. This provides a better control of failovers as different networks can now be prioritized. A new selection widget for the network is available in the user interface helping to choose the correct link address and preventing users from making typos.
- **ZFS 0.8.1 with native encryption and SSD TRIM support:**the new features for ZFS include enhanced security and data protection thanks to the added support for native encryption with comfortable key-handling by integrating the encryption directly into the `zfs` utilities. Encryption is as flexible as volume creation. TRIM support is included. The subcommand `zpool trim` notifies devices about unused sectors, thus TRIM can improve the usage of the resources and contribute to longer SSD life. Also checkpoints on pool level are available.
- **Support for ZFS on UEFI and on NVMe devices in the ISO installer:**the installer now supports ZFS root via UEFI, for example you can boot a ZFS mirror on NVMe SSDs. By using `systemd-boot` as bootloader instead of grub all pool-level features can be enabled on the root pool.
- **QEMU 4.0.0:**new QEMU functionalities are included in Promxox VE 6.0. Users can now use the web interface to live migrate guests with disks backed by local storage, and to set more VM CPU-flags. Support for more Hyper-V enlightenment has been added thus improving Windows performance in a virtual machine under QEMU/KVM.
- **Custom Cloudinit configurations:**Proxmox VE 6 brings support for custom Cloudinit configurations and lets users store it as Snippet. The command `qm cloudinit dump` can be used to get the current Cloudinit configuration as a starting point for extensions.

### Other Notable Changes in Proxmox VE 6.0

- Automatic clean-up of old kernel images: the old kernel images are no longer marked as ‘NeverAutoRemove’ which helps to prevent problems when /boot is mounted on a small partition.
- Guest status display in the tree view: Additional states for guests (migration, backup, snapshot, locked) are shown directly in the tree overview.
- Improved ISO detection in the installer: the way how the installer detects the ISO has been reworked to include more devices, alleviating problems of detection on certain hardware.
- Pool level backup: it’s now possible to create a backup task for a whole pool. By selecting a pool as backup target instead of an explicit list of guests, new members of the pool are automatically included and removed guests automatically excluded from the backup task.
- Automatic rotation of the authentication key every 24h: by limiting the key lifetime to 24h the impact of key leakage or a malicious administrator are reduced.
- The Node view in the user interface provides a faster syslog view.

By using Proxmox VE 6 as an open-source alternative to proprietary virtualization management solutions, enterprises are able to centralize and modernize their IT infrastructure and turn it into a cost-effective and flexible software-defined data center.

### Availability

Proxmox VE 6.0 is available for download at[https://www.proxmox.com/downloads.](https://www.proxmox.com/downloads)

**Checklist tool ‘pve5to6’:**
Users can check their installation before, during and after the upgrade process with the checklist tool ‘pve5to6’. It’s included in the latest Proxmox VE 5.4 packages and will provide hints and warnings about potential issues.

**Upgrade from Proxmox VE 5.4 to 6.0:**
Distribution upgrades from Proxmox VE 5.4 to 6.0 should follow the detailed instructions as a major version of Corosync is present (2.x to 3.x). There is a three-step upgrade path for clusters where users first need to upgrade to Corosync 3, then upgrade to Proxmox VE v6.0, and finally upgrade the Ceph cluster from Ceph Luminous to Nautilus.

For detailed upgrade guides please see[https://pve.proxmox.com/wiki/Upgrade_from_5.x_to_6.0](https://pve.proxmox.com/wiki/Upgrade_from_5.x_to_6.0)and[https://pve.proxmox.com/wiki/Ceph_Luminous_to_Nautilus](https://pve.proxmox.com/wiki/Ceph_Luminous_to_Nautilus)

Distribution upgrades from a beta version of Proxmox VE 6.0 is possible with apt. It’s also possible to install Proxmox VE 6.0 on top of Debian Buster.

Proxmox VE is published under the free software license GNU Affero GPL, v3. Enterprise support is available from Proxmox Server Solutions on a subscription basis starting at EUR 79,90 per year and CPU.

**Facts**
The open source project Proxmox VE has a huge worldwide user base with over 270,000 hosts. The web-based management interface is translated into 19 languages. More than 40,000 members are active in the community support forum. Tens of thousands of customers from companies regardless of sector, size or industry rely on a Proxmox VE support subscription, a service offered by Proxmox Server Solutions GmbH.

**About Proxmox VE**
Proxmox VE is the leading open-source platform for all-inclusive enterprise virtualization. With the central web interface you can easily run VMs and containers, manage software-defined storage and networking functionality, high-availability clustering, and multiple integrated out-of-the-box tools like backup/restore, live migration, replication, and the firewall. Enterprises use the powerful yet easy-to-manage all-in-one solution Proxmox VE to meet the core requirements—less complexity, more elasticity— of today’s modern data centers ensuring to stay adaptable for future growth thanks to the flexible, modular and open architecture.

**About Proxmox Server Solutions**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises regardless of size, sector or industry use the stable, secure, scalable, and open Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-6-0
