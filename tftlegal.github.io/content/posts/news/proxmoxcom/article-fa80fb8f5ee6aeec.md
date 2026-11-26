---
title: "Proxmox Virtual Environment 9.3 with focus on improved Enterprise storage experience"
date: 2026-11-25T12:57:58Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-9-3"
summary: "25 ноября 2026 года Proxmox Server Solutions GmbH (Вена, Австрия) выпустила Proxmox Virtual Environment 9.3 — новую версию своей open-source платформы виртуализации. Ключевые нововведения включают централизованное управление общим LVM-хранилищем на SAN с кластерным обзором и пошаговой настройкой multipath, а также полную производственную поддержку снапшотов в виде цепочек томов для LVM. Каждому гостевому серверу теперь присваивается уникальный идентификатор, сохраняющийся при миграциях, бэкапах и восстановлении, что упрощает отслеживание в экосистеме внешних инструментов. В разделе SDN расширена изоляция зон с использованием VRF и добавлена визуализация состояния туннелей WireGuard. Платформа построена на Debian 13.8 «Trixie» с ядром Linux 7.0/7.3, QEMU 11.1, LXC 7.0, ZFS 2.4, а для гиперконвергентного хранения доступны Ceph Tentacle 20.2 (stable) и Ceph Umbrella 21.2 (preview). Proxmox VE 9.3 доступна для бесплатного скачивания, а для production-средств компания предлагает платные планы технической поддержки."
---

# Proxmox Virtual Environment 9.3 with focus on improved Enterprise storage experience

## Краткое содержание

25 ноября 2026 года Proxmox Server Solutions GmbH (Вена, Австрия) выпустила Proxmox Virtual Environment 9.3 — новую версию своей open-source платформы виртуализации. Ключевые нововведения включают централизованное управление общим LVM-хранилищем на SAN с кластерным обзором и пошаговой настройкой multipath, а также полную производственную поддержку снапшотов в виде цепочек томов для LVM. Каждому гостевому серверу теперь присваивается уникальный идентификатор, сохраняющийся при миграциях, бэкапах и восстановлении, что упрощает отслеживание в экосистеме внешних инструментов. В разделе SDN расширена изоляция зон с использованием VRF и добавлена визуализация состояния туннелей WireGuard. Платформа построена на Debian 13.8 «Trixie» с ядром Linux 7.0/7.3, QEMU 11.1, LXC 7.0, ZFS 2.4, а для гиперконвергентного хранения доступны Ceph Tentacle 20.2 (stable) и Ceph Umbrella 21.2 (preview). Proxmox VE 9.3 доступна для бесплатного скачивания, а для production-средств компания предлагает платные планы технической поддержки.

## Полная статья

**VIENNA, Austria – November 25, 2026 –**Enterprise software developer Proxmox Server Solutions GmbH today announced the release of Proxmox Virtual Environment 9.3, the latest version of its open-source virtualization platform. This version simplifies shared LVM storage configuration on SANs and brings full production support for volume-chain snapshots on LVM. It also assigns unique guest identities for reliable backup tracking, and expands Software-Defined Networking (SDN) isolation and visibility.

### Centralized SAN visibility, volume-chain snapshots, and custom storage plugin support

To eliminate manual, node-by-node configuration across multi-node clusters, Proxmox VE 9.3 introduces centralized multipath management alongside a new cluster-wide SAN view under Datacenter > SAN that displays LUN path state across all nodes in real time, complemented by a node-level SAN panel listing local LUNs, paths, usage, and active iSCSI sessions. A guided setup workflow configures multipath across cluster nodes and provisions shared LVM storage on iSCSI, Fibre Channel, and NVMe over Fabrics LUNs directly from the web interface.

In addition, Proxmox VE 9.3 now offers full production support for snapshots as volume chains on thick-provisioned LVM, including shared LVM storage on iSCSI and Fibre Channel SANs. Administrators can enable this capability directly in the advanced storage settings via the “Allow Snapshots as Volume-Chain” option.

Third-party storage plugins can also now be added and edited directly in the web interface and are listed in the "Add" menu under Datacenter > Storage. Proxmox VE dynamically generates configuration forms from each plugin's property schema – supporting existing plugins regardless of their storage API version – while masking sensitive credentials such as passwords and keys.

### Unique guest identities & optimized cluster maintenance

Proxmox VE 9.3 assigns each guest a unique identity that stays the same across migrations, backups, and restores – even if a guest receives a new VMID or is restored to another cluster. This enables backup software and external management tools to reliably recognize the same guest across its lifecycle, which is useful for the partner and tooling ecosystem around Proxmox VE.
Ongoing cluster administration, routine maintenance, and incident troubleshooting are further streamlined in the web interface and installer:

- **HA maintenance mode of nodes in the GUI:**Administrators can enable or disable High Availability (HA) maintenance mode for individual nodes directly from the “Node Maintenance” menu in the HA status panel or via a new API endpoint, simplifying planned host maintenance without using the “ha-manager” command line tool.
- **Structured log and update views:**Proxmox VE 9.3 introduces a color-coded, filterable System Log, a new API Access Log, and an APT History view listing past package transactions and changelogs.
- **Nested match rules for notification matchers:**Notification routing logic that previously had to be split across several matchers now fits into one. A new tree-based rule editor with drag-and-drop support allows administrators to combine, invert, and nest alert routing conditions – including weekday and time-range selectors.
- **Management bond & VLAN setup in the installer:**The installer (GUI, TUI, and automated) can set up the management network on a bond, a VLAN, or both.

### Software-Defined Networking enhancements

Proxmox VE 9.3 expands its integrated Software-Defined Networking (SDN) stack with improved network isolation for Simple zones, which can now use a dedicated Virtual Routing and Forwarding (VRF) instance to isolate their routing table with custom static routes. In addition, WireGuard fabrics now report the status of their interfaces and peers directly in the management interface, giving administrators clear visibility into tunnel health.

### Updated technology stack, Ceph Umbrella preview, and Proxmox Offline Mirror 0.8

Proxmox Virtual Environment 9.3 is based on Debian 13.8 “Trixie” shipping with Linux Kernel 7.0 as the stable default for broad hardware compatibility and a new, opt-in Linux Kernel 7.3 based directly on the upstream stable branch from kernel.org. Version 9.3 also includes the latest versions of QEMU 11.1, LXC 7.0, and ZFS 2.4. For hyperconverged storage, it provides Ceph Tentacle 20.2 as stable version alongside Ceph Umbrella 21.2 as a technology preview, which brings an optimized erasure-coding I/O path, hardened CephX authentication with a new default cipher and online key rotation for daemons, and an optional backup mechanism for the monitor database. For air-gapped environments, the updated Proxmox Offline Mirror 0.8 introduces automated snapshot pruning, “last” snapshot symlinks, and parallelized, lower-memory package synchronization.

### Availability

Proxmox VE 9.3 is open-source software and immediately available for download at the official website. Users can obtain a complete installation image via ISO download, which contains the full feature-set of the solution and can be installed quickly on bare-metal systems using an intuitive installation wizard.

Seamless distribution upgrades from older versions of Proxmox VE are possible using the standard APT package management system. Furthermore, it is also possible to install Proxmox VE on top of an existing Debian installation.

For production environments, Proxmox Server Solutions offers transparent per-socket support plans – starting with the Proxmox Basic plan – that provide ticket-based access to Proxmox’s engineering support and stable updates. Optional 24/7 support and coverage for Arm64 deployments are also available.

**Resources:**
• ISO Image Download:[https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/downloads)
• Forum Announcement:[https://forum.proxmox.com/](https://forum.proxmox.com/threads/proxmox-virtual-environment-9-3-released.187800/)
• Video tutorial:[What’s new in Proxmox VE 9.3](https://www.proxmox.com/en/services/training-courses/videos/proxmox-virtual-environment/whats-new-in-proxmox-ve-9-3)
• Roadmap: For published and upcoming features, see the[Release Notes & Roadmap](https://pve.proxmox.com/wiki/Roadmap#Proxmox_VE_9.3)

###

**About Proxmox Virtual Environment**
Powering over 2.6 million servers globally, Proxmox Virtual Environment is a complete open-source platform for enterprise virtualization and hyper-converged infrastructure. It natively unifies KVM virtualization, LXC containers, software-defined storage, and networking on a single platform. Alongside its dedicated Backup Server and Datacenter Manager, the Proxmox ecosystem eliminates multi-site complexity as well as dependency on proprietary stacks. Backed by a global community of over 250,000 members, the platform serves as a scalable, cost-effective foundation for modern data centers.

**About Proxmox Server Solutions**
Proxmox Server Solutions provides powerful, intuitive open-source server software that guarantees vendor independence and minimizes total cost of ownership. Enterprises of all sizes rely on the company’s reliable vendor support, certified training services, and a rapidly growing global network of 3,600 integration partners to ensure business continuity. More than 60,000 enterprises worldwide hold active Proxmox support plans to secure their mission-critical IT. Established in 2005, Proxmox Server Solutions GmbH is headquartered in Vienna, Austria, with its subsidiary Proxmox North America Inc. based in Kingston, Ontario.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH,[press@proxmox.com](mailto:press@proxmox.com)

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-9-3
