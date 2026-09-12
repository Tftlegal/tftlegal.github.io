---
title: "Proxmox Virtual Environment 7.2 available"
date: 2022-05-03T15:54:28Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-7-2-available"
summary: "Proxmox выпустил версию Proxmox Virtual Environment 7.2 — платформу управления виртуализацией на базе Debian 11.3 с обновлённым ядром Linux 5.15 и актуальными компонентами QEMU, LXC и ZFS. Обновление улучшает функции резервного копирования и восстановления, включая шаблоны заметок, гибкое планирование задач и новые настройки восстановления. Поддержка Ceph расширена: добавлена совместимость с Ceph Pacific 16.2.7, а поддержка Ceph Octopus сохранена до середины 2022 года. Также улучшены управление кластером, высокая доступность, веб-интерфейс, мобильное приложение и управление загрузкой ядра. Proxmox VE остаётся бесплатным открытым решением, доступным для установки на bare-metal и Debian, с возможностью обновления через apt. Для предприятий компания предлагает подписку на поддержку, включая enterprise-репозиторий и техническую помощь от разработчиков."
---

# Proxmox Virtual Environment 7.2 available

## Краткое содержание

Proxmox выпустил версию Proxmox Virtual Environment 7.2 — платформу управления виртуализацией на базе Debian 11.3 с обновлённым ядром Linux 5.15 и актуальными компонентами QEMU, LXC и ZFS.
Обновление улучшает функции резервного копирования и восстановления, включая шаблоны заметок, гибкое планирование задач и новые настройки восстановления.
Поддержка Ceph расширена: добавлена совместимость с Ceph Pacific 16.2.7, а поддержка Ceph Octopus сохранена до середины 2022 года.
Также улучшены управление кластером, высокая доступность, веб-интерфейс, мобильное приложение и управление загрузкой ядра.
Proxmox VE остаётся бесплатным открытым решением, доступным для установки на bare-metal и Debian, с возможностью обновления через apt.
Для предприятий компания предлагает подписку на поддержку, включая enterprise-репозиторий и техническую помощь от разработчиков.

## Полная статья

**VIENNA, Austria – May 04, 2022 –**Enterprise software developer Proxmox Server Solutions GmbH (henceforth "Proxmox") has today released Proxmox Virtual Environment 7.2. The new version of the server virtualization management platform is based on Debian 11.3 (Bullseye), but using a newer Linux kernel 5.15.30, and includes updates to the latest versions of leading open-source technologies for virtual environments, like QEMU 6.2.0, LXC 4.0.12, and ZFS 2.1.4. Proxmox VE supports Ceph Pacific 16.2.7, in addition to offering continued support for Ceph Octopus 15.2.16 (until mid 2022).

### What’s new in Proxmox Virtual Environment 7.2

- Backup/Restore:

- Notes templates: Meta-information can be added via a notes-template for backup jobs, to better distinguish and search for backups. This template is evaluated as soon as the job is executed, and added to any resulting backup. Notes templates can contain template variables like {{guestname}} or {{cluster}}.
- To benefit from the Rust code of Proxmox Backup Server, the Proxmox developers make use of perlmod, a Rust crate which allows exporting Rust modules as Perl packages. perlmod is used by Proxmox to transfer data between Rust and Perl, thus implementing parts of Proxmox VE and Proxmox Mail Gateway in Rust.
- The next-event scheduling code was updated via this Perl-to-Rust-binding (perlmod) and now uses the same code as Proxmox Backup Server. Users can not only specify the existing weekday, time, and time range, but now also a specific date and time (e.g., *-12-31 23:50; New Year's Eve, 10 minutes before midnight every year), date ranges (e.g., Sat *-1..7 15:00; first Saturday every month at 15:00), or repeating ranges (e.g., Sat *-1..7 */30; first Saturday every month, every half hour).
- Some basic restore settings, for example guest name or memory, can now be overwritten in the enhanced backup-restore dialog in the web interface.
- A new ‘job-init’ hook step was added to the backup process. Among other things, it can be used to prepare the backup storage, for example, by starting the storage server.

- High Availability Manager:

- By improving the local resource manager (pve-ha-lrm) scheduler which launches workers, the amount of configurable services that can be handled per single node has increased. This helps in large deployments, as the services at the end of the queue are also checked to ensure that they are still in the target state.
- By introducing a skip-round command to the integrated HA simulator in Proxmox VE, it has become easier to test races in scheduling (on the different nodes).

- Cluster: Regarding the creation of new VMs or containers, version 7.2 allows you to configure a desired range from which the new VMIDs are proposed via the web interface. The lower and upper boundaries can be set in the Datacenter -> Options panel. Setting lower equal to upper disables auto-suggestion completely, meaning the administrator has to manually enter an ID.
- Ceph: Proxmox VE supports Ceph Pacific 16.2.7 and Ceph Octopus 15.2.16 (with continued support until mid 2022). This version now also supports creating and destroying erasure-coded pools, which can be added as Proxmox VE storage entries, and help to reduce the amount of disk space required. A new option in the GUI allows for passing the keyring secrets of external Ceph clusters when adding an RBD or CephFS storage to Proxmox VE.
- Web interface: Further enhancements in the web interface allow for example for safe reassignment of a VM disk or CT volume to another guest on the same node; the reassigned disk/volume can be attached at a different bus/mountpoint on the destination guest. This can help in cases of upgrades, restructuring, or after disaster recovery.
- Management: Many improvements in Proxmox VE 7.2 enable even more convenient management of the system. For example, a particular kernel version can be selected to boot persistently from a running system, through ‘proxmox-boot-tool kernel pin’. The selection can be used either indefinitely or just for the next boot. This eliminates the need to watch the boot process to select the desired kernel version in the bootloader screen.

### Further enhancements and bug fixes

- In the installation ISO, ZFS installs can be configured to use various compression algorithms (e.g., zstd, gzip, etc.).Additionally, the memtest86+ package, a tool aimed at memory failure detection, has been updated to the completely rewritten 6.0b.
- Further improvements have been added to virtual machines (KVM/QEMU); one to highlight is support for the accelerated virtio-gl (VirGL) display driver. For VirtIO and VirGL display types, SPICE is enabled by default. In modern Linux distributions, changing the graphics card to VirGL can significantly increase frames per second (FPS). For Proxmox containers (LXC), many templates have also been refreshed or newly added, such as the NixOS container template.
- The Proxmox VE Android app now provides a simple dark theme and enables it if the system settings are configured to use dark designs. The mobile app also provides an inline console by relaying noVNC for VMs, and xterm.js for containers and the Proxmox VE node shell in the GUI.
- To prevent a network outage during the transition from ifupdown to ifupdown2, the ifupdown package was modified to not stop networking upon its removal.

### Availability

Proxmox Virtual Environment is free and open-source software, published under the GNU Affero General Public License, v3. The ISO contains the complete feature-set and can be installed on bare-metal. Proxmox VE 7.2 is available for[download](https://www.proxmox.com/en/component/zoo/?view=frontpage&layout=frontpage&Itemid=120).

The virtualization platform from Proxmox comes stocked with all the essential management tools, as well as an easy-to-use, web-based user interface. This allows for simple, out-of-the-box management of the host, either through the command line or a standard web browser.

Distribution upgrades from older versions of Proxmox VE are possible with apt. It’s also possible to install Proxmox VE 7.2 on top of Debian.

For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to an Enterprise Repository, with regular updates via the web interface, as well as[technical support directly from the developers](https://www.proxmox.com/en/products/proxmox-virtual-environment/pricing). Prices start at EUR 95 per year and CPU.

**Forum Announcement**
[https://forum.proxmox.com/threads/proxmox-ve-7-2-released.108970](https://forum.proxmox.com/threads/proxmox-ve-7-2-released.108970/)

**Video tutorial**
[What's new in Proxmox VE 7.2](https://www.proxmox.com/en/training/video-tutorials/item/what-s-new-in-proxmox-ve-7-2)

**Facts**
The open-source project Proxmox VE has a huge worldwide user base with more than 600,000 hosts. The virtualization platform has been translated into over 26 languages. More than 88,000 active community members in the support forum engage with and help each other. By using Proxmox VE as an alternative to proprietary virtualization management solutions, enterprises are able to centralize and modernize their IT infrastructure, and turn it into a cost-effective and flexible software-defined data center, based on latest open-source technologies. Tens of thousands of customers rely on an enterprise support subscription from Proxmox Server Solutions GmbH.

**About Proxmox Virtual Environment**
Proxmox Virtual Environment (Proxmox VE) is the leading open-source platform for all-inclusive enterprise virtualization. With the central web interface, you can easily run VMs and containers, manage software-defined storage and networking functionality, high-availability clustering, and multiple integrated out-of-the-box tools like backup/restore, live migration, replication, and the firewall. Enterprises use the powerful yet easy-to-manage, all-in-one solution to meet the core requirements of today’s modern data centers. Proxmox VE allows them to remain adaptable for future growth, thanks to its flexible, modular and open architecture.

**About Proxmox Server Solutions**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training services to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-7-2-available
