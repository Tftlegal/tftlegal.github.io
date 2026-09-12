---
title: "Proxmox Datacenter Manager 1.1 available"
date: 2026-05-28T08:17:43Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-datacenter-manager-1-1"
summary: "Proxmox Server Solutions представила обновление Proxmox Datacenter Manager 1.1 — централизованную платформу для управления распределенными инфраструктурами Proxmox.   Ключевым нововведением стал автоматизированный workflow установки хостов с централизованными answer files, токенами безопасности и отслеживанием прогресса в веб-интерфейсе.   Добавлен реестр подписок, позволяющий управлять пулом ключей, назначать их на удаленные системы и автоматически регистрировать новые хосты при установке.   Обновление обеспечивает единый мониторинг Ceph-кластеров, включая состояние OSD, мониторов, менеджеров, MDS, пулов, CephFS и флагов.   Расширены возможности визуализации и управления инфраструктурой: появились географические виджеты, графики загрузки CPU, памяти и хранилища, а также кросс-remote просмотр QEMU-ВМ и LXC-контейнеров с управлением снапшотами.   Платформа построена на Debian 13.5, Linux kernel 7.0 и ZFS 2.4, доступна как open-source под GNU AGPLv3, а пользователям Enterprise-подписок предоставляются обновления и поддержка."
---

# Proxmox Datacenter Manager 1.1 available

## Краткое содержание

Proxmox Server Solutions представила обновление Proxmox Datacenter Manager 1.1 — централизованную платформу для управления распределенными инфраструктурами Proxmox.  
Ключевым нововведением стал автоматизированный workflow установки хостов с централизованными answer files, токенами безопасности и отслеживанием прогресса в веб-интерфейсе.  
Добавлен реестр подписок, позволяющий управлять пулом ключей, назначать их на удаленные системы и автоматически регистрировать новые хосты при установке.  
Обновление обеспечивает единый мониторинг Ceph-кластеров, включая состояние OSD, мониторов, менеджеров, MDS, пулов, CephFS и флагов.  
Расширены возможности визуализации и управления инфраструктурой: появились географические виджеты, графики загрузки CPU, памяти и хранилища, а также кросс-remote просмотр QEMU-ВМ и LXC-контейнеров с управлением снапшотами.  
Платформа построена на Debian 13.5, Linux kernel 7.0 и ZFS 2.4, доступна как open-source под GNU AGPLv3, а пользователям Enterprise-подписок предоставляются обновления и поддержка.

## Полная статья

**VIENNA, Austria – May 28, 2026**– Enterprise software developer Proxmox Server Solutions GmbH today announced the availability of a new point release for Proxmox Datacenter Manager. The centralized management platform designed to overseedistributedProxmox infrastructures introduces new enhancements including an automated installation workflow, comprehensive subscription handling, unified Ceph cluster monitoring, and expanded central guest and snapshot management.

## Highlights in Proxmox Datacenter Manager 1.1

Integrated automated installation workflows

Proxmox Datacenter Manager 1.1 now acts as a central configuration server for provisioning. The integration of automated installation functionality standardizes the deployment of hosts across distributed infrastructures. Administrators can centrally manage answer file configurations containing predefined installation parameters and provide them for unattended installations of new hosts. A new ‘Automated Installations’ tab in the ‘Remotes’ section provides access to these workflows, while installation progress can be tracked directly from within the Proxmox Datacenter Manager web interface. A token-based security mechanism protects the installation process and helps ensure that prepared configurations are accessed only by authorized installations.

Centralized management of subscription keys

For large-scale deployments, managing subscriptions across multiple sites can be complex. A new subscription registry in Proxmox Datacenter Manager enables administrators to manage a central pool of subscription keys, assign them to specific remotes, and remove assignments when no longer needed. A prepared answer file can also include a specific subscription key, allowing a newly provisioned host to register its subscription automatically during installation.

Unified Ceph cluster monitoring

For organizations utilizing hyper-converged infrastructure (HCI) powered by Proxmox VE, tracking storage health across distributed sites is vital. Proxmox Datacenter Manager 1.1 delivers deep, unified visibility across these distributed storage environments by introducing native monitoring for all connected Ceph clusters. A single, consolidated panel allows administrators to verify the health, capacity, and real-time performance of multiple Ceph clusters at a glance. The dashboard provides comprehensive, granular insights into the status of Object Storage Daemons (OSDs), monitors, managers, Metadata Servers (MDS), storage pools, CephFS, and specific cluster flags.

Enhanced infrastructure visualization

New dashboard widgets provide administrators with an overview of their distributed Proxmox infrastructures:

- Geographic widgets: A new world map widget visualizes the physical locations of connected remotes. Locations can be defined via the node or datacenter options on Proxmox VE remotes, or under the configuration settings for Proxmox Backup Server remotes.
- New gauge-based widgets display visual context for CPU, memory, and storage utilization at a glance.
- Local host metrics are now also collected for the Proxmox Datacenter Manager host itself, visualizing resource consumption through integrated Round-Robin Database (RRD) graphs on the node status panel.

Central guest and snapshot management

Proxmox Datacenter Manager 1.1 marks the initial milestone toward comprehensive, central guest management. A new cross-remote view expands guest management by displaying all QEMU virtual machines and LXC containers across connected remotes. Administrators can display these guests in a sortable table or in a tree grouped by remote, use text filtering to quickly locate individual guests, and access frequently used actions from a unified overview.

The same interface now also provides snapshot management for these guest environments. Administrators can view snapshots in a parent-child tree and create, roll back, delete, or edit snapshot descriptions. In addition, a new “Resume” action for paused or suspended QEMU virtual machines complements the existing start, stop, and shutdown operations. As this represents the initial phase of centralized guest orchestration, users can expect additional day-to-day management tasks to be integrated in upcoming point releases.

Updated technology stack

Proxmox Datacenter Manager 1.1 is based on Debian 13.5 “Trixie” and features Linux kernel 7.0 as the new stable default. Along with ZFS 2.4, this release provides an up-to-date open-source software stack for modern centralized infrastructure management and day-to-day lifecycle operations.

### Availability

Proxmox Datacenter Manager 1.1 is open-source software and immediately available for download at the official website. Users can obtain a complete installation image via ISO download, which contains the full feature set of the solution and can be installed quickly on bare-metal systems using an intuitive installation wizard.

Seamless distribution upgrades from older versions of Proxmox Datacenter Manager are possible using the standard APT package management system. Furthermore, it is also possible to install the platform on top of an existing Debian installation. As Free/Libre and Open Source Software (FLOSS), the entire solution is published under the GNU AGPLv3.

For enterprise environments, customers with active Enterprise support plans for their managed Proxmox Virtual Environment and Proxmox Backup Server remotes also gain access to Proxmox Datacenter Manager updates and support. No separate subscription key is required.

Resources:

- ISO Image Download:[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads)
- Forum Announcement:[https://forum.proxmox.com/](https://forum.proxmox.com/threads/proxmox-datacenter-manager-1-1-released.183903/)
- Video:[What’s new in Proxmox Datacenter Manager 1.1](https://www.proxmox.com/en/services/training-courses/videos/proxmox-datacenter-manager/whats-new-in-proxmox-datacenter-manager-1-1)
- Roadmap: For published and upcoming features, see the[Release Notes & Documentation](https://pdm.proxmox.com/docs/roadmap.html)

###

**About Proxmox Datacenter Manager
**Proxmox Datacenter Manager is a centralized open-source management layer for distributed, large-scale Proxmox infrastructures. As a core building block of the expanding Proxmox ecosystem, it unifies independent Proxmox Virtual Environment clusters and Proxmox Backup Server instances across multiple sites and data centers into a single control plane. The web interface provides consolidated dashboards for real-time health, performance, and capacity tracking of nodes, virtual machines, containers, and storage. IT teams can centrally manage guest lifecycles, perform migrations, and execute global updates across connected remotes. Developed by Proxmox Server Solutions GmbH, the software is written in Rust, based on Debian, and released under the GNU AGPLv3.

**About Proxmox Server Solutions
**Proxmox Server Solutions provides powerful, intuitive open-source server software that guarantees vendor independence and minimizes total cost of ownership. Enterprises of all sizes rely on the company’s reliable vendor support, certified training services, and a global network of 3,000 integration partners to ensure business continuity. Established in 2005 and headquartered in Vienna, Austria, tens of thousands of corporate customers worldwide trust Proxmox solutions to secure mission-critical IT environments. To learn more visit[https://www.proxmox.com](https://www.proxmox.com)or follow us on[LinkedIn](https://at.linkedin.com/company/proxmox)and[YouTube](https://www.youtube.com/@ProxmoxVE/).

**Media contact**
Daniela Häsler, Proxmox Server Solutions GmbH, marketing@proxmox.com

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-datacenter-manager-1-1
