---
title: "Proxmox Virtual Environment 7.0 with Debian 11 \"Bullseye\" and Ceph Pacific 16.2 released"
date: 2021-06-17T13:17:22Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-7-0"
summary: "Proxmox объявил о выпуске стабильной версии 7.0 своей платформы виртуализации Proxmox Virtual Environment. Новая версия построена на базе Debian 11 «Bullseye» с ядром Linux 5.11 и включает QEMU 6.0, LXC 4.0 и OpenZFS 2.0.4. Ключевые обновления — полноценная интеграция Ceph Pacific 16.2, поддержка файловой системы Btrfs и новый интерфейс для управления APT-репозиториями. Платформа также получила Single Sign-On через OpenID Connect, улучшенный веб-интерфейс, поддержку cgroups v2 в LXC 4.0 и переработанный установщик. Proxmox VE 7.0 распространяется как бесплатное open-source ПО под лицензией AGPLv3, а для предприятий доступен платный подписочный support. Компания отмечает, что Proxmox VE — это зрелое решение для построения гибких и экономичных программных дата-центров без привязки к вендору."
---

# Proxmox Virtual Environment 7.0 with Debian 11 "Bullseye" and Ceph Pacific 16.2 released

## Краткое содержание

Proxmox объявил о выпуске стабильной версии 7.0 своей платформы виртуализации Proxmox Virtual Environment. Новая версия построена на базе Debian 11 «Bullseye» с ядром Linux 5.11 и включает QEMU 6.0, LXC 4.0 и OpenZFS 2.0.4. Ключевые обновления — полноценная интеграция Ceph Pacific 16.2, поддержка файловой системы Btrfs и новый интерфейс для управления APT-репозиториями. Платформа также получила Single Sign-On через OpenID Connect, улучшенный веб-интерфейс, поддержку cgroups v2 в LXC 4.0 и переработанный установщик. Proxmox VE 7.0 распространяется как бесплатное open-source ПО под лицензией AGPLv3, а для предприятий доступен платный подписочный support. Компания отмечает, что Proxmox VE — это зрелое решение для построения гибких и экономичных программных дата-центров без привязки к вендору.

## Полная статья

**VIENNA, Austria – July 6, 2021 –**Enterprise software developer Proxmox Server Solutions GmbH (or "Proxmox") today announced the stable version 7.0 of its server virtualization management platform Proxmox Virtual Environment. This major release is based on Debian 11 “Bullseye” but using a Linux kernel 5.11, and includes QEMU 6.0, LXC 4.0, and OpenZFS 2.0.4.

> “The release of Proxmox VE 7.0 this time comes ahead of the stable release of Debian Bullseye. Although almost all packages were ready, the Debian project team postponed the release originally planned for May, due to an unresolved issue in the Debian installer. Since we maintain our own Proxmox installer, and are not affected by this particular issue, we decided to release despite the delay of Debian. The core packages of Proxmox VE are either already subject to the very strict Debian freeze policy for essential packages (for example, libc or compiler) or are maintained by our Proxmox developers (for example, QEMU, Kernel, Ceph, LXC, Rust compiler).”
Thomas Lamprecht - Lead developer at Proxmox

### What’s new in Proxmox Virtual Environment 7.0

This major release brings a large set of new enhancements:

- Ceph Pacific 16.2: Proxmox Virtual Environment fully integrates Ceph, giving you the ability to run and manage Ceph storage directly from any of your cluster nodes. This enables users to setup and manage a hyper-converged infrastructure. Ceph Pacific 16.2 is now the default in Proxmox VE, while Ceph Octopus 15.2 remains available with continued support.

- Beginning with Ceph Pacific 16.2, the balancer-module is enabled by default for new clusters. This will lead to better distribution of placement groups among the OSDs, and help to balance the data more evenly across OSDs reducing the chances that a single OSD is disproportionately full, resulting in less available space than expected in the cluster.
- Ceph monitors with multiple public networks can be created using the CLI, should users have multiple configured links.

- Btrfs Storage Technology: The copy-on-write (COW) file system, natively supported by the Linux kernel, implements features such as snapshots, built-in RAID, and self-healing via checksumming for data and metadata. It allows taking subvolume snapshots and supports offline storage migration while keeping snapshots. For users of enterprise storage systems, Btrfs provides file system integrity after unexpected power loss, helps prevent bitrot, and is designed for high-capacity and high-performance storage servers.
- New Panel for easy management of APT repositories via GUI: The Proxmox developers have added a new 'Repositories' panel to the web interface which allows to inspect a node's configured APT repositories. The new panel provides a single place to see all package repository configuration, which usually is scattered across multiple files, and warns about potential misconfiguration. Users can enable and disable repositories as needed, and add the standard repositories provided by Proxmox. For instance, it’s possible to test a new Ceph release which is not yet available in the main repository. The Ceph test repository, provided by Proxmox can simply be enabled (or added), the new version tested, and then disabled again when it’s not longer needed.
- Access Control: The new open protocol standard OpenID Connect provides Single Sign-On (SSO) resulting in a seamless user experience. Administrators can integrate an external authorization server, by either using existing public services or their own identity and access management solution. Also, a newly added permission ‘Pool.Audit’ allows users to see pools, but not to change them.
- Enhancements to the web-based user interface (GUI):

- Markdown in "Notes" -The “Notes” panels for Guest and Node can now interpret Markdown and render it as HTML. This gives administrators a better visualization of their notes.
- Pruning on manually triggered backups: Users can prune the target storage with its backup-retention parameters when starting a manual backup.
- Support for security keys (like YubiKey) as SSH keys, when creating containers or preparing cloud-init images.

- QEMU 6.0: The latest QEMU version with new functionalities is included in Proxmox VE 7. This includes support for the Linux IO interface ‘io_uring’. The asynchronous I/O engine for virtual drives will be applied to all newly launched or migrated guest systems by default. Also a clean-up option for unreferenced VM disks is available. Disks, which are not present in the configuration, don't get automatically destroyed anymore. It is now opt-in in the API and with CLI tools (in the GUI it is present since Proxmox VE 6.4). If this clean-up option is enabled, only storage with content-types of VM or CT disk images, or rootdir will be scanned for unused disk-volumes, helping to prevent accidental data loss.
- Container: LXC 4.0 has full support for cgroups v2, a mechanism for hierarchical organization of processes and allocation of system resources. A pure cgroup v2 layout is the default for Promox VE 7.0.
- Proxmox VE Installer: The installer environment has been reworked and now uses switch_root instead of chroot, when transitioning from initrd to the actual installer. This improves module and firmware loading, and slightly reduces memory usage during installation. The installer now automatically detects HiDPI screens, and increases the console font and GUI scaling accordingly. This improves the UX for workstations with Proxmox VE (for example, for pass-through). The ISO detection has been improved as well to work more reliably with slower storages. The new installer uses zstd compression for the initrd image and the squashfs images.

### Other notable enhancements

- Certificate management: The ACME standalone plugin now has improved support for dual-stacked (IPv4 and IPv6) environments and no longer relies on the configured addresses to determine its listening interface.
- Network: The modern ifupdown2 is the default network management tool for new installations using the Proxmox VE official ISO. The legacy ifupdown is still supported in Proxmox VE 7.
- Time Synchronization: New installations will install chrony as the default NTP daemon, because the design limitations of systemd-timesync make it problematic for server use. Users upgrading from a system using systemd-timesyncd, should manually install either chrony, ntp or openntpd.

### Availability

Proxmox Virtual Environment is free and open-source software, published under the GNU Affero General Public License, v3. The downloadable ISO image for Proxmox VE version 7.0 can be installed on bare-metal and is available at[https://www.proxmox.com/downloads](https://www.proxmox.com/en/component/zoo/?view=frontpage&layout=frontpage)

The virtualization platform from Proxmox comes stocked with all the essential management tools, as well as an easy-to-use, web-based user interface. This allows for simple, out-of-the-box management of the host, either through the command line or a standard web browser.

**Checklist tool ‘pve6to7’:**Users can check their installation before, during, and after the upgrade process with the checklist tool ‘pve6to7’. It is included in the latest Proxmox VE 6.4 packages and provides hints and warnings about potential issues to clear up before upgrading. See[https://pve.proxmox.com/wiki/Upgrade_from_6.x_to_7.0](https://pve.proxmox.com/wiki/Upgrade_from_6.x_to_7.0)

Distribution upgrades from older versions of Proxmox VE or from a beta version of Proxmox VE 7.0 are possible with apt. It is also possible to install Proxmox VE 7.0 on top of Debian 11 “Bullseye”.

**Ceph cluster upgrade:**Upgrading Proxmox VE from version 6.4 to 7.0 first is necessary. Afterwards upgrading Ceph from Octopus to Pacific. For the detailed upgrade guides please see[https://pve.proxmox.com/wiki/Ceph_Octopus_to_Pacific](https://pve.proxmox.com/wiki/Ceph_Octopus_to_Pacific)

For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to an Enterprise Repository, with regular updates via the web interface, as well as[technical support directly from the developers](https://www.proxmox.com/en/products/proxmox-virtual-environment/pricing). Prices start at EUR 90 per year and CPU.

**Facts**
The open-source project Proxmox VE has a huge worldwide user base with more than 450,000 hosts. The virtualization platform has been translated into over 26 languages. More than 65,000 active community members in the support forum engage with and help each other. By using Proxmox VE as an alternative to proprietary virtualization management solutions, enterprises are able to centralize and modernize their IT infrastructure, and turn it into a cost-effective and flexible software-defined data center, based on latest open-source technologies. Tens of thousands of customers rely on a enterprise support subscription from Proxmox Server Solutions GmbH.

**About Proxmox Virtual Environment**
Proxmox Virtual Environment (Proxmox VE) is the leading open-source platform for all-inclusive enterprise virtualization. With the central web interface, you can easily run VMs and containers, manage software-defined storage and networking functionality, high-availability clustering, and multiple integrated out-of-the-box tools like backup/restore, live migration, replication, and the firewall. Enterprises use the powerful yet easy-to-manage, all-in-one solution to meet the core requirements of today’s modern data centers. Proxmox VE allows them to remain adaptable for future growth, thanks to its flexible, modular and open architecture.

**About Proxmox Server Solutions**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training services to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria. To learn more visit[https://www.proxmox.com](https://www.proxmox.com)

Contact: Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-7-0
