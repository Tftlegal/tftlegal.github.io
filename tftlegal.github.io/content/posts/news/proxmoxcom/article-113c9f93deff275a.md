---
title: "Proxmox Virtual Environment 8.4 available"
date: 2025-04-09T09:18:02Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-8-4"
summary: "Proxmox Server Solutions представила версию 8.4 платформы Proxmox Virtual Environment для управления серверной виртуализацией.   Обновление добавляет живую миграцию работающих виртуальных машин с медированными устройствами, включая поддержку NVIDIA vGPU, и упрощает настройку соответствующих драйверов.   Теперь Proxmox VE предоставляет API для сторонних резервных решений, позволяя им полностью интегрироваться в систему резервного копирования и веб-интерфейс.   В версии также появилась прямая передача каталогов между хостом и виртуальными машинами через virtiofs, а также обновленный стек технологий на базе Debian 12.10, новых ядер Linux, QEMU, LXC, ZFS и Ceph.   Дополнительно улучшены надежность резервного копирования, программно-определяемая сеть и возможности установщика ISO.   Proxmox VE 8.4 доступна для бесплатного скачивания как open-source ПО, а для предприятий предлагается платная поддержка с ценой от 115 евро в год за CPU."
---

# Proxmox Virtual Environment 8.4 available

## Краткое содержание

Proxmox Server Solutions представила версию 8.4 платформы Proxmox Virtual Environment для управления серверной виртуализацией.  
Обновление добавляет живую миграцию работающих виртуальных машин с медированными устройствами, включая поддержку NVIDIA vGPU, и упрощает настройку соответствующих драйверов.  
Теперь Proxmox VE предоставляет API для сторонних резервных решений, позволяя им полностью интегрироваться в систему резервного копирования и веб-интерфейс.  
В версии также появилась прямая передача каталогов между хостом и виртуальными машинами через virtiofs, а также обновленный стек технологий на базе Debian 12.10, новых ядер Linux, QEMU, LXC, ZFS и Ceph.  
Дополнительно улучшены надежность резервного копирования, программно-определяемая сеть и возможности установщика ISO.  
Proxmox VE 8.4 доступна для бесплатного скачивания как open-source ПО, а для предприятий предлагается платная поддержка с ценой от 115 евро в год за CPU.

## Полная статья

**VIENNA, Austria – April 09, 2025**– Enterprise software developer Proxmox Server Solutions GmbH (henceforth "Proxmox") today released version 8.4 of its server virtualization management platform, Proxmox Virtual Environment. This version comes with a range of improvements and new features.

### Highlights in Proxmox VE 8.4

- Live migration with mediated devices: Mediated devices allow physical hardware resources to be partitioned into multiple virtual devices. It is now possible to migrate running virtual machines (VMs) which have mediated devices such as NVIDIA vGPU in use. Within a cluster, live migration to another node is possible if hardware and driver support for live migration is equally available on the target node. An available platform with support for live migration is NVIDIA vGPU. Additionally, the new ‘pve-nvidia-vgpu-helper’ tool simplifies the setup of NVIDIA vGPU drivers.
- API for third-party backup solutions: Proxmox VE now provides an API simplifying the development of plugins by external backup solution providers. Third party backup solutions can now directly implement backup and restore functionalities in Proxmox VE, and make use of advanced features. Third-party backup providers can implement a plugin that is fully integrated in the backup stack and in the web interface of Proxmox VE. This gives third-party backup solutions the opportunity to enhance the efficiency, reliability, and manageability of their backup and restore features in Proxmox VE.
- Virtiofs directory passthrough: Version 8.4 offers the functionality to share files and directories directly between a host and the VMs running on that host. This is achieved through the use of virtiofs, which allows VM guests to access host files and directories without the overhead of a network file system. Modern Linux guests support virtiofs out of the box while Windows guests need additional software to use this feature.
- Latest versions of open-source technologies: Proxmox VE 8.4 is based on Debian 12.10 (“Bookworm”), but uses the Linux kernel 6.8.12 as stable default and the newer kernel 6.14 as opt-in. This version of Proxmox VE includes updates to the latest versions of leading open-source technologies for virtual environments like QEMU 9.2.0, LXC 6.0.0, ZFS 2.2.7 with compatibility patches for kernel 6.14, and Ceph Squid 19.2.1 as stable option.

Further enhancements include more robust backup fleecing, improvements in the software-defined networking (SDN) stack, and additional options in the ISO installer.

### Availability

Proxmox VE 8.4 is now available for download. The ISO image contains the complete feature-set and can be installed on bare-metal. Distribution upgrades from older versions of Proxmox VE are possible with apt. It’s also possible to install Proxmox VE 8.4 on top of Debian. Proxmox Virtual Environment is free and open-source software, published under the GNU Affero General Public License, v3.

Support Subscriptions: For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 115 per year and CPU.

### Resources

- ISO Image Download:[https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/downloads)
- Forum Announcement:[https://forum.proxmox.com/threads/proxmox-ve-8-4-released.164821/](https://forum.proxmox.com/threads/proxmox-ve-8-4-released.164821/)
- Roadmap: For published and upcoming features, see the[Release Notes & Roadmap](https://pve.proxmox.com/wiki/Roadmap)

###

**Facts**
The open-source project Proxmox VE has a huge worldwide user base with more than 1.5 million hosts. The virtualization platform has been translated into over 30 languages. More than 200,000 active community members in the support forum engage with and help each other. By using Proxmox VE as an alternative to proprietary virtualization management solutions, enterprises are able to centralize and modernize their IT infrastructure, and turn it into a cost-effective and flexible software-defined data center, based on the latest open-source technologies. Tens of thousands of customers rely on enterprise support subscriptions from Proxmox Server Solutions GmbH.

**About Proxmox VE**
Proxmox Virtual Environment (Proxmox VE) is the leading open-source platform for all-inclusive enterprise virtualization. With the central web interface, you can easily run VMs and containers, manage software-defined storage and networking functionality, high-availability clustering, and multiple integrated out-of-the-box tools like backup/restore, live migration, replication, and the firewall. Enterprises use the powerful yet easy-to-manage, all-in-one solution to meet the core requirements of today’s modern data centers. Proxmox VE allows them to remain adaptable for future growth, thanks to its flexible, modular and open architecture.

**About Proxmox Server Solutions**
Proxmox provides powerful and user-friendly open-source server software. Enterprises of all sizes and industries use Proxmox solutions to deploy efficient and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support, training services, and an extensive partner ecosystem to ensure business continuity for its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria. To learn more visit[https://www.proxmox.com](https://www.proxmox.com)or follow us on[LinkedIn](https://www.linkedin.com/company/proxmox)or on[YouTube](https://www.youtube.com/user/ProxmoxVE).

Contact: Daniela Häsler, Proxmox Server Solutions GmbH, marketing@proxmox.com

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-8-4
