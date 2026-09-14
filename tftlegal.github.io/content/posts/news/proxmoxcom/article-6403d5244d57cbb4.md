---
title: "Proxmox Virtual Environment 8.2 with Import Wizard released"
date: 2024-04-22T16:49:01Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-8-2"
summary: "Proxmox выпустила версию 8.2 открытой платформы управления виртуализацией Proxmox Virtual Environment. Обновление основано на Debian 12.5, использует ядро Linux 6.8 и обновляет ключевые технологии виртуализации, включая QEMU 8.1, LXC 6.0, Ceph 18.2 и ZFS 2.2. Главные новинки — мастер импорта гостевых систем из VMware ESXi, автоматическая установка Proxmox VE на физические серверы и функция backup fleecing, снижающая влияние резервного копирования на производительность виртуальных машин. В версии также добавлен новый сетевой экран на базе nftables в режиме технологического превью, а также улучшены настройки резервного копирования, GUI и поддержка ACME-сертификатов. Proxmox VE 8.2 доступна для загрузки, поддерживает обновление с предыдущих версий через apt и может устанавливаться поверх Debian. Платформа является свободным программным обеспечением под GNU AGPLv3, а Proxmox предлагает корпоративным клиентам платную поддержку и enterprise-репозиторий."
---

# Proxmox Virtual Environment 8.2 with Import Wizard released

## Краткое содержание

Proxmox выпустила версию 8.2 открытой платформы управления виртуализацией Proxmox Virtual Environment. Обновление основано на Debian 12.5, использует ядро Linux 6.8 и обновляет ключевые технологии виртуализации, включая QEMU 8.1, LXC 6.0, Ceph 18.2 и ZFS 2.2. Главные новинки — мастер импорта гостевых систем из VMware ESXi, автоматическая установка Proxmox VE на физические серверы и функция backup fleecing, снижающая влияние резервного копирования на производительность виртуальных машин. В версии также добавлен новый сетевой экран на базе nftables в режиме технологического превью, а также улучшены настройки резервного копирования, GUI и поддержка ACME-сертификатов. Proxmox VE 8.2 доступна для загрузки, поддерживает обновление с предыдущих версий через apt и может устанавливаться поверх Debian. Платформа является свободным программным обеспечением под GNU AGPLv3, а Proxmox предлагает корпоративным клиентам платную поддержку и enterprise-репозиторий.

## Полная статья

**VIENNA, Austria – April 24, 2024 –**Enterprise software developer Proxmox Server Solutions GmbH (henceforth "Proxmox") has today released version 8.2 of its server virtualization management platform, Proxmox Virtual Environment. The new version is based on Debian 12.5 (Bookworm), but using a newer Linux kernel 6.8, and includes updates to the latest versions of leading open-source technologies for virtual environments like QEMU 8.1, LXC 6.0, Ceph 18.2, and ZFS 2.2.

Proxmox Virtual Environment 8.2 comes with an import wizard to migrate VMware ESXi guests to Proxmox VE, with a new tool for automated installation from the ISO to bare-metal servers, a backup fleecing feature implementation to decouple slower backup storage from the VM performance, and several UI improvements.

## Highlights in Proxmox Virtual Environment 8.2

- **Import Wizard for VMware ESXi VMs:**Proxmox VE provides an integrated VM importer presented as storage plugin for native integration into the API and web-based user interface. It offers users the ability to import guests directly from other hypervisors. Currently, it allows to import VMware-based VMs (ESXi and vCenter). You can use this to import the VM as a whole, with most of the original configuration settings mapped to Proxmox VE's configuration model.
- **Automated and Unattended Installation:**Proxmox offers a new*‘proxmox-auto-inst**all**-**assistant**’*tool that fully automates the setup process on bare-metal. Automated installation allows for the rapid deployment of Proxmox VE hosts without the need for manual access to the systems, saving time and reducing the risk of errors. To use this method, an answer file must be prepared with the necessary configuration settings for the installation process. This file can be provided directly in the ISO, on an additional disk such as a USB flash drive, or over the network. Automated installation is useful in various scenarios, such as deploying large-scale infrastructure, automating the setup process, and ensuring consistent configurations across multiple systems.
- **Backup Fleecing:**When creating a backup of a running VM, a slow backup target can negatively impact guest IO performance during the backup process. Fleecing can reduce this impact by caching data blocks in a fleecing image rather than sending it directly to the backup target, which can help guest IO performance and even prevent hangs at the cost of requiring more storage space. Backup fleecing is especially beneficial when backing up IO-heavy guests to a remote Proxmox Backup Server or other backup storage with a slow network connection.
- **Firewall modernization with nftables (technology preview):**Proxmox VE comes with a new firewall implementation that uses nftables instead of iptables. The opt-in feature in tech preview is written in the Rust programming language. Although the new implementation is close to feature parity with the existing one, the nftables firewall must be enabled manually and remains a preview to first gather feedback from the community.

### Further enhancements

- Device passthrough for containers via GUI: While the API and CLI have supported LXC device passthrough since version 8.1, GUI configuration is now possible as well.
- Advanced backup settings: Backup jobs come with new advanced settings, including performance settings and bandwidth limits.
- Custom ACME directories/providers: This release now supports custom ACME-enabled Certificate Authorities (CA) with optional External Account Binding (EAB) authentication.
- Multiple enhancements in the GUI: Disabling the double-click editing option in the notes field makes it easier to select and copy text without accidentally opening the editor. On edit screens, the reset button is now in a new location reducing the risk of accidental clicks.

### Availability

Proxmox VE 8.2 is available for download at[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads). The ISO contains the complete feature-set and can be installed on bare-metal.

The virtualization platform from Proxmox comes stocked with all the essential management tools, as well as an easy-to-use, web-based user interface. This allows for simple, out-of-the-box management of the host, either through the command line or a standard web browser. Distribution upgrades from older versions of Proxmox VE are possible with apt. It’s also possible to install Proxmox VE 8.2 on top of Debian.

License: Proxmox Virtual Environment is free and open-source software, published under the GNU Affero General Public License, v3.

Support Subscriptions: For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 110 per year and CPU.

##### Further information:

- Release Notes -[https://pve.proxmox.com/wiki/Roadmap#Proxmox_VE_8.2](https://pve.proxmox.com/wiki/Roadmap#Proxmox_VE_8.2)
- Videos:[What's new in Proxmox VE 8.2](https://www.proxmox.com/en/services/training-courses/videos/proxmox-virtual-environment/whats-new-in-proxmox-ve-8-2)&[Proxmox VE Import Wizard: How to import VMs from VMware ESXi](https://www.proxmox.com/en/services/training-courses/videos/proxmox-virtual-environment/proxmox-ve-import-wizard-for-vmware)
- Forum Announcement:[https://forum.proxmox.com](https://forum.proxmox.com/threads/proxmox-ve-8-2-released.145723/)

###

**About Proxmox Server Solutions**
Proxmox provides powerful and user-friendly open-source server software. Enterprises of all sizes and industries use Proxmox solutions to deploy efficient and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support, training services, and an extensive partner ecosystem to ensure business continuity for its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**Facts Proxmox VE**
The open-source project Proxmox VE has a huge worldwide user base with more than one million hosts. The virtualization platform is available in 29 languages. More than 150 000 active community members in the support forum engage with and help each other. By using Proxmox VE as an alternative to proprietary virtualization management solutions, enterprises are able to centralize and modernize their IT infrastructure, and turn it into a cost-effective and flexible software-defined data center, based on the latest open-source technologies. Tens of thousands of customers rely on a enterprise support subscription from Proxmox Server Solutions GmbH.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-8-2
