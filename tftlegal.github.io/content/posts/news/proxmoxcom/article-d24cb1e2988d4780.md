---
title: "Proxmox Virtual Environment 8.3 released"
date: 2024-11-21T13:10:47Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-8-3"
summary: "Proxmox Server Solutions GmbH представила версию 8.3 своей виртуализационной платформы Proxmox Virtual Environment. Обновление построено на Debian 12.8, по умолчанию использует ядро Linux 6.8.12-4 и позволяет включить ядро 6.11, а также включает новые версии QEMU 9.0.2, LXC 6.0.0 и ZFS 2.2.6. Среди главных улучшений — более тесная интеграция SDN с файрволом, новые вебхуки для уведомлений, режим просмотра ресурсов по меткам, поддержка Ceph Squid, ускоренное резервное копирование контейнеров и упрощенный импорт OVF/OVA для миграции виртуальных машин. Решение остается свободным и открытым под лицензией AGPLv3, его можно скачать как ISO, установить на Debian или обновить через apt. Компания также предлагает платные подписки на поддержку с доступом к Enterprise Repository, начиная с 110 евро в год за CPU-socket. Proxmox сообщила, что платформа уже установлена примерно на 1,3 млн активных хостов и продолжает привлекать предприятия как открытый и масштабируемый вариант для построения виртуальной инфраструктуры."
---

# Proxmox Virtual Environment 8.3 released

## Краткое содержание

Proxmox Server Solutions GmbH представила версию 8.3 своей виртуализационной платформы Proxmox Virtual Environment. Обновление построено на Debian 12.8, по умолчанию использует ядро Linux 6.8.12-4 и позволяет включить ядро 6.11, а также включает новые версии QEMU 9.0.2, LXC 6.0.0 и ZFS 2.2.6. Среди главных улучшений — более тесная интеграция SDN с файрволом, новые вебхуки для уведомлений, режим просмотра ресурсов по меткам, поддержка Ceph Squid, ускоренное резервное копирование контейнеров и упрощенный импорт OVF/OVA для миграции виртуальных машин. Решение остается свободным и открытым под лицензией AGPLv3, его можно скачать как ISO, установить на Debian или обновить через apt. Компания также предлагает платные подписки на поддержку с доступом к Enterprise Repository, начиная с 110 евро в год за CPU-socket. Proxmox сообщила, что платформа уже установлена примерно на 1,3 млн активных хостов и продолжает привлекать предприятия как открытый и масштабируемый вариант для построения виртуальной инфраструктуры.

## Полная статья

**VIENNA, Austria – November 21, 2024 –**Enterprise software developer Proxmox Server Solutions GmbH (henceforth "Proxmox") has today released version 8.3 of its virtualization platform, Proxmox Virtual Environment. The enterprise virtualization solution features essential management tools and a user-friendly web interface, allowing organizations of all sizes, sectors and industries, to deploy open-source solutions in clustered, highly available setups. The world-wide customer base uses it to deploy extensive infrastructures across vast geographical locations, and integrate multiple applications seamlessly.

This version is based on Debian 12.8 (Bookworm), but uses the Linux kernel 6.8.12-4 as stable default, and allows for opt-in use of kernel 6.11. The software includes updates to the latest versions of leading open-source technologies for virtual environments like QEMU 9.0.2, LXC 6.0.0 , and ZFS 2.2.6 (with compatibility patches for Kernel 6.11).

## Enhancements in Proxmox Virtual Environment 8.3

- **Alignment of the Software-Defined Networking (SDN) stack with the firewall:**The SDN technology allows users to create virtual zones and networks (VNets), enabling them to effectively manage and control complex networking configurations and multitenancy setups via the web interface of Proxmox VE. The SDN stack is now more closely integrated with the firewall, with the ability to automatically generate IP sets for VNets and virtual guests managed by the IP address management plugin. Referencing IP sets in the firewall rules simplifies their creation and maintenance. In addition, the opt-in firewall based on nftables now has the capability to filter forwarded network traffic at the host and at the virtual network (VNet) level. For example, this can be used to restrict SNAT traffic or traffic flowing from one Simple Zone to another.
- **Webhook target for the notification system:**The flexible notification system in Proxmox solutions uses a matcher-based approach to route notifications to various target types, allowing for granular control over when, where, and how notifications are sent. The new webhook notification target enables users to trigger HTTP requests for events like system updates, cluster node issues, or backup job. It supports customizable request headers and body content, allowing for seamless integration with any webhook-compatible service.
- **A new ‘Tag View’ functionality for the resource tree**offers users a rapid and customizable overview of the virtual guests: The new ‘Tag View’ allows users to view their virtual guests grouped according to the assigned tags.
- **Support for Ceph Squid**(technology preview): Proxmox Virtual Environment 8.3 adds support for Ceph Squid 19.2.0 and continues to support Ceph Reef 18.2.4 and Ceph Quincy 17.2.7. The preferred Ceph version can be selected during the installation process.
- **Faster container backups:**When backing up containers to Proxmox Backup Server, it is now possible to efficiently detect files that have not changed since the last backup snapshot. When possible, unchanged files are not processed, which can make container backups complete faster.
- **Migration from other hypervisors:**The Proxmox developers have streamlined the guest import from files in Open Virtualization Format (OVF) and Open Virtualization Appliances (OVA). Proxmox VE now allows users to import OVF and OVA files directly from file-based storage via the web interface. This simplifies the guest import process and allows users to either upload OVA files from a local machine or download them from a URL. Additionally, Proxmox VE also offers an import wizard facilitating the migration of VMs from other hypervisors, such as VMware guests.

Citation CTO Thomas Lamprecht:

*"Our virtualization platform is designed to empower enterprises with unparalleled efficiency and control," said Thomas Lamprecht, CTO of Proxmox. "By integrating advanced Software-Defined Networking capabilities, we’re enabling businesses to simplify their IT infrastructure, improve agility, and drive scalable growth. We are currently developing the inaugural version of the Proxmox Datacenter Manager, which will provide a unified management interface for multi-datacenter operations and facilitate the effective management of multiple clusters across diverse data centers.”*

Citation COO Tim Marx:

*"The open-source project Proxmox VE has experienced significant growth over the past few months, adding 300,000 active hosts, bringing the total to around 1.3 million. In the current dynamic business environment, we are well-positioned for growth, particularly as recent pricing shifts in the market create new opportunities for us to serve our customers more effectively," said Tim Marx, COO of Proxmox. "Our commitment to quality and value has resonated strongly, and we’re excited to continue building on this momentum."*

### Availability

Proxmox Virtual Environment 8.3 is available for download at[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads). The ISO contains the complete feature-set and can be installed on bare-metal. Distribution upgrades from older versions of Proxmox VE are possible with apt. It is also possible to install Proxmox VE 8.3 on top of Debian.

License: Proxmox Virtual Environment is free and open-source software, published under the GNU Affero General Public License, v3.

Support Subscriptions: For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 110 per year and CPU socket.

##### Further information:

Release Notes -[https://pve.proxmox.com/wiki/Roadmap#Proxmox_VE_8.3](https://pve.proxmox.com/wiki/Roadmap#Proxmox_VE_8.3)
Video:[What's new in Proxmox VE 8.3](https://www.proxmox.com/services/videos/proxmox-virtual-environment/whats-new-in-proxmox-ve-8-3)
Forum Announcement:[https://forum.proxmox.com](https://forum.proxmox.com/threads/proxmox-ve-8-3-released.157793/)

###

**Facts**
The open-source project Proxmox VE has a huge worldwide user base with more than 1.3 million installed hosts. The virtualization platform is available in 30 languages. More than 190.000 active community members in the support forum engage with and help each other. By using Proxmox VE as an alternative to proprietary virtualization management solutions, enterprises are able to centralize and modernize their IT infrastructure, and turn it into a cost-effective and flexible software-defined data center, based on the latest open-source technologies. Tens of thousands of customers rely on a enterprise support subscription from Proxmox Server Solutions GmbH.

**About Proxmox Server Solutions**
Proxmox provides powerful and user-friendly open-source server software. Enterprises of all sizes and industries use Proxmox solutions to deploy efficient and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support, training services, and an extensive partner ecosystem to ensure business continuity for its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-8-3
