---
title: "Proxmox Virtual Environment 6.4 with Single File Restore and Live Restore released"
date: 2021-03-22T18:06:32Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-6-4-available"
summary: "Proxmox Server Solutions выпустила версию 6.4 платформы Proxmox Virtual Environment. Обновление добавляет live restore и восстановление отдельных файлов из резервных копий, что ускоряет восстановление виртуальных машин. Поддерживаются две версии Ceph — Octopus 15.2.11 и Nautilus 14.2.20, а также улучшено управление Ceph через веб-интерфейс. В состав вошли актуальные компоненты виртуализации, включая QEMU 5.2, LXC 4.0 и OpenZFS 2.0.4, а по умолчанию используется ядро Linux 5.4. Также улучшены возможности KVM/QEMU, веб-интерфейс, установка ZFS,"
---

# Proxmox Virtual Environment 6.4 with Single File Restore and Live Restore released

## Краткое содержание

Proxmox Server Solutions выпустила версию 6.4 платформы Proxmox Virtual Environment.
Обновление добавляет live restore и восстановление отдельных файлов из резервных копий, что ускоряет восстановление виртуальных машин.
Поддерживаются две версии Ceph — Octopus 15.2.11 и Nautilus 14.2.20, а также улучшено управление Ceph через веб-интерфейс.
В состав вошли актуальные компоненты виртуализации, включая QEMU 5.2, LXC 4.0 и OpenZFS 2.0.4, а по умолчанию используется ядро Linux 5.4.
Также улучшены возможности KVM/QEMU, веб-интерфейс, установка ZFS,

## Полная статья

**VIENNA, Austria – April 28, 2021 –**Enterprise software developer Proxmox Server Solutions GmbH (or "Proxmox") has today released version 6.4 of its server virtualization management platform Proxmox Virtual Environment. This latest version comes with important new features such as live-restore and single file restore, support for Ceph Octopus 15.2.11 and Ceph Nautilus 14.2.20, many enhancements to KVM/QEMU, and notable bug fixes. The usability of Proxmox VE has improved significantly with the addition of many features and management options to the web interface.

The new version is based on Debian Buster 10.9, but using a newer, long-term supported Linux kernel 5.4. Optionally, the 5.11 kernel can be installed, providing support for the latest hardware. The 5.4 kernel remains the default on Proxmox VE 6.x series. The latest versions of leading open-source technologies for virtualization like QEMU 5.2, LXC 4.0, and OpenZFS 2.0.4 have been included. The Proxmox maintainers support two versions of Ceph, the massively scalable, distributed storage system, in their virtualization platform. During the installation process, users can select their preferred version, either Ceph Octopus 15.2.11 or Ceph Nautilus 14.2.20.

### Single-File Restore and Live Restore for KVM

Proxmox Virtual Environment 6.4 brings Live Restore and Single File Restore enabling users to simplify restore tasks and further improve recovery time objectives (RTO).

- Single-File Restore: Quite often, users only need to recover a single file. This feature is now available for virtual machine and container backup archives stored on a Proxmox Backup Server, meaning an individual file or directory can be selected for restore, without having to download the entire archive. To restore a file via the Proxmox VE web interface, users can open a file browser directly via the 'File Restore' button. A 'Download' button then allows the user to download files and directories, the latter being compressed into a zip archive on the fly. In case users want to download a VM image, which might contain untrusted data, Proxmox VE starts a temporary VM to download the data from it. This avoids exposing the hypervisor system to danger.
- Live Restore: The new live restore feature can be enabled via the GUI or through the command ‘qmrestore’. The restore of a selected VM starts immediately after activation. This feature currently works for all VMs saved on a Proxmox Backup Server storage. It is especially useful for large VMs, for example, a web server, where only a small amount of data is required for the initial operation. The VM becomes operational as soon as the operating system and all necessary services have been started, while – in the background – the lesser used data is continuously restored.

### Support for Ceph Octopus 15.2.11 and Ceph Nautilus 14.2.20

Proxmox Virtual Environment supports two versions of the massively scalable, distributed storage system Ceph. Users can select their preferred Ceph version-- Ceph Octopus 15.2.11 or Ceph Nautilus 14.2.20--during the installation process. The integration of the placement group (PG) auto-scaler has improved in version 6.4. This allows administrators to configure Target Size or Target Ratio settings and see the optimal numbers of PGs in the GUI. For easier usage, the Ceph pool view has been optimized, making it possible to show the columns related to the auto-scaler, as well as to configure the major pool properties from the web interface.

### Further enhancements

- Proxmox VE API Proxy Daemon: pveproxy listens to both IPv4 and IPv6 addresses by default. The Listening IP addresses are configurable in /etc/default/pveproxy. This can help to limit the exposure to the outside, e.g., by only binding to an internal IP.
- Container: Appliance templates or support for Alpine Linux 3.13, Devuan 3, Fedora 34, and Ubuntu 21.04. Improved handling of cgroup v2 (control group).
- External metric server: In Proxmox VE, you can define external metric servers, providing you with various statistics about your hosts, virtual guests, and storages. The new version supports InfluxDB HTTPs API and instances of InfluxDB behind a reverse proxy.
- Improved ISO installer: The boot setup for ZFS installations is now better equipped for legacy hardware. Installations on ZFS now install the boot-loader to all selected disks, instead of only to the first mirror vdev, improving the experience with hardware where the boot-device is not easily selectable. Before installation, an NTP synchronization is attempted.
- Storage: Proxmox VE 6.4 now allows for adding backup notes on any CephFS, CIFS, or NFS storage. Users can also configure a namespace for accessing a Ceph pool.
- VMs (KVM/QEMU):

- Support pinning a VM to a specific QEMU machine version.
- Automatically pin VMs with Windows as OS type to the current QEMU machine on VM creation. This improves stability and guarantees that the hardware layout stays the same, even with newer QEMU versions.
- cloud-init: re-add Stateless Address Autoconfiguration (SLAAC) option to IPv6 configuration.

- Enhancements to the GUI

- Show current usage of host memory and CPU resources by each guest in the node search-view.
- Use binary (1 KiB equals 1024 B instead of 1 KB equals 1000 B) as base in the node and guest memory usage graphs, ensuring it is consistent with the current usage gauge.
- Firewall rules: Columns are more responsive and flexible by default.

### Notable bugfixes

- Container restores now default to the privilege setting from the backup archive.
- ZFS: Checking if a pool is mounted (in addition to imported) and trying to mount it, improves robustness for ZFS on slower disks.
- Address issues with hanging qmp commands, causing VMs to freeze

### Availability

Proxmox Virtual Environment is free and open-source software, published under the GNU Affero General Public License, v3. The downloadable ISO image can be installed on bare-metal. Proxmox VE 6.4 is available for download at[https://www.proxmox.com/downloads.](https://www.proxmox.com/downloads.)

The virtualization platform from Proxmox comes stocked with all the essential management tools, as well as an easy-to-use, web-based user interface. This allows for simple, out-of-the-box management of the host, either through the command line or a standard web browser. Distribution upgrades from older versions of Proxmox VE are possible with apt. It’s also possible to install Proxmox VE 6.4 on top of Debian Buster. For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 90 per year and CPU.

**Facts**
The open-source project Proxmox VE has a huge worldwide user base with more than 450,000 hosts. The virtualization platform has been translated into over 26 languages. More than 65,000 active community members in the support forum engage with each other. By using Proxmox VE as an alternative to proprietary virtualization management solutions, enterprises are able to centralize and modernize their IT infrastructure, and turn it into a cost-effective and flexible software-defined data center, based on the latest open-source technologies. Tens of thousands of customers from companies regardless of sector, size or industry rely on a Proxmox VE support subscription, a service offered by Proxmox Server Solutions GmbH.

**About Proxmox VE**
Proxmox Virtual Environment (Proxmox VE) is the leading open-source platform for all-inclusive enterprise virtualization. With the central web interface, you can easily run VMs and containers, manage software-defined storage and networking functionality, high-availability clustering, and multiple integrated out-of-the-box tools like backup/restore, live migration, replication, and the firewall. Enterprises use the powerful yet easy-to-manage, all-in-one solution, Proxmox VE, to meet the core requirements of today’s modern data centers. Proxmox VE allows them to remain adaptable for future growth, thanks to its flexible, modular and open architecture.

**About Proxmox Server Solutions**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**Contact:**Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-6-4-available
