---
title: "Proxmox Virtual Environment 8.1 with Software-Defined Network and Secure Boot"
date: 2023-11-23T09:19:23Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-8-1"
summary: "Компания Proxmox объявила о выпуске версии 8.1 своей открытой платформы виртуализации Proxmox Virtual Environment.   Новая версия построена на Debian 12.2 и использует ядро Linux 6.5, а также обновленные QEMU, LXC и ZFS.   Среди ключевых нововведений — поддержка Secure Boot, встроенный software-defined network (SDN) и гибкая система уведомлений.   Proxmox VE 8.1 также добавляет поддержку Ceph Reef 18.2.0 и сохраняет поддержку Ceph Quincy 17.2.7.   Платформа доступна для бесплатного скачивания, устанавливается на bare-metal или поверх Debian и управляется через веб-интерфейс или командную строку.   Для предприятий Proxmox предлагает подписку на коммерческую поддержку с доступом к Enterprise Repository и регулярным обновлениям."
---

# Proxmox Virtual Environment 8.1 with Software-Defined Network and Secure Boot

## Краткое содержание

Компания Proxmox объявила о выпуске версии 8.1 своей открытой платформы виртуализации Proxmox Virtual Environment.  
Новая версия построена на Debian 12.2 и использует ядро Linux 6.5, а также обновленные QEMU, LXC и ZFS.  
Среди ключевых нововведений — поддержка Secure Boot, встроенный software-defined network (SDN) и гибкая система уведомлений.  
Proxmox VE 8.1 также добавляет поддержку Ceph Reef 18.2.0 и сохраняет поддержку Ceph Quincy 17.2.7.  
Платформа доступна для бесплатного скачивания, устанавливается на bare-metal или поверх Debian и управляется через веб-интерфейс или командную строку.  
Для предприятий Proxmox предлагает подписку на коммерческую поддержку с доступом к Enterprise Repository и регулярным обновлениям.

## Полная статья

**VIENNA, Austria – November 23, 2023**– Enterprise software developer Proxmox Server Solutions GmbH (henceforth "Proxmox") today announced the release of version 8.1 of Proxmox Virtual Environment, its open-source server virtualization management platform. This version comes with several new features, support for Secure Boot, a Software-defined Network stack, a new flexible notification system, and many further enhancements and bug fixes.

Proxmox VE 8.1 is based on Debian 12.2 (“Bookworm”), but uses a newer Linux kernel 6.5 as stable default, and includes updates to the latest versions of leading open-source technologies for virtual environments like QEMU 8.1.2 and LXC 5.0.2. It comes with ZFS 2.2.0 including the most important bugfixes from 2.2.1 already. The virtualization platform adds support for Ceph Reef 18.2.0 and continues to support Ceph Quincy 17.2.7.

## Highlights in Proxmox Virtual Environment 8.1

- **Support for Secure Boot:**This version is now compatible with Secure Boot. This security feature is designed to protect the boot process of a computer by ensuring that only software with a valid digital signature launches on a machine. Proxmox VE now includes a signed shim bootloader trusted by most hardware's UEFI implementations. This allows installing Proxmox VE in environments with Secure Boot active.
- **Software-defined Network (SDN):**With this version the core Software-defined Network (SDN) packages are installed by default. The SDN technology in Proxmox VE enables to create virtual zones and networks (VNets), which enables users to effectively manage and control complex networking configurations and multitenancy setups directly from the web interface at the datacenter level. Use cases for SDN range from an isolated private network on each individual node to complex overlay networks across multiple Proxmox VE clusters on different locations. The benefits result in a more responsive and adaptable network infrastructure that can scale according to business needs.
- **New Flexible Notification System:**This release introduces a new framework that uses a matcher-based approach to route notifications. It lets users designate different target types as recipients of notifications. Alongside the current local Postfix MTA, supported targets include Gotify servers or SMTP servers that require SMTP authentication. Notification matchers determine which targets will get notifications for particular events based on predetermined rules. The new notification system now enables greater flexibility, allowing for more granular definitions of when, where, and how notifications are sent.
- **Support for Ceph Reef and Ceph Quincy:**Proxmox Virtual Environment 8.1 adds support for Ceph Reef 18.2.0 and continues to support Ceph Quincy 17.2.7. The preferred Ceph version can be selected during the installation process. Ceph Reef brings better defaults improving performance and increased reading speed.

### Availability

Proxmox VE 8.1 is available for download at[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads). The ISO contains the complete feature-set and can be installed on bare-metal.

The virtualization platform from Proxmox comes stocked with all the essential management tools, as well as an easy-to-use, web-based user interface. This allows for simple, out-of-the-box management of the host, either through the command line or a standard web browser. Distribution upgrades from older versions of Proxmox VE are possible with apt. It’s also possible to install Proxmox VE 8.1 on top of Debian.

License: Proxmox Virtual Environment is free and open-source software, published under the GNU Affero General Public License, v3.

Support Subscriptions: For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 105 per year and CPU.

##### Further information:

- Release Notes -[https://pve.proxmox.com/wiki/Roadmap#Proxmox_VE_8.1](https://pve.proxmox.com/wiki/Roadmap#Proxmox_VE_8.1)
- Video["What's new in Proxmox VE 8.1"](https://www.proxmox.com/en/services/training-courses/videos/proxmox-virtual-environment/what-s-new-in-proxmox-ve-8-1)
- Forum Announcement:[https://forum.proxmox.com](https://forum.proxmox.com/threads/proxmox-ve-8-1-released.136960/#post-608423)

###

**About Proxmox Server Solutions**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training services to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**Facts Proxmox VE**
The open-source project Proxmox VE has a huge worldwide user base with more than 900,000 hosts. The virtualization platform has been translated into over 28 languages. More than 130,000 active community members in the support forum engage with and help each other. By using Proxmox VE as an alternative to proprietary virtualization management solutions, enterprises are able to centralize and modernize their IT infrastructure, and turn it into a cost-effective and flexible software-defined data center, based on the latest open-source technologies. Tens of thousands of customers rely on a enterprise support subscription from Proxmox Server Solutions GmbH.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-8-1
