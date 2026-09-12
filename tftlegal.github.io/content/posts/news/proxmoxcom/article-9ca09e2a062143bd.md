---
title: "Proxmox VE 3.4 released with ZFS filesystem, ZFS storage plugin, hotplug"
date: 2015-02-18T19:16:32Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-3-4-released"
summary: "19 февраля 2015 года Proxmox Server Solutions выпустила версию 3.4 своей открытой платформы управления серверной виртуализацией Proxmox VE. Ключевыми нововведениями стали встроенная файловая система ZFS, плагин хранения ZFS, поддержка hotplug и NUMA, а также обновление до Debian Wheezy 7.8. ZFS теперь можно выбрать в качестве корневого раздела при установке и использовать для локального хранения, RAID-массивов, снапшотов, шаблонов и клонов. Платформа получила возможность горячего подключения и замены виртуальных дисков, сетевых карт и USB-устройств, а также улучшенный веб-интерфейс. Proxmox VE 3.4 распространяется под лицензией GNU AGPL v3, доступна для скачивания и предлагает платные подписки для бизнес-клиентов. Решение используется на десятках тысяч хостов в более чем 140 странах и поддерживает KVM, OpenVZ-контейнеры и высокодоступность."
---

# Proxmox VE 3.4 released with ZFS filesystem, ZFS storage plugin, hotplug

## Краткое содержание

19 февраля 2015 года Proxmox Server Solutions выпустила версию 3.4 своей открытой платформы управления серверной виртуализацией Proxmox VE.
Ключевыми нововведениями стали встроенная файловая система ZFS, плагин хранения ZFS, поддержка hotplug и NUMA, а также обновление до Debian Wheezy 7.8.
ZFS теперь можно выбрать в качестве корневого раздела при установке и использовать для локального хранения, RAID-массивов, снапшотов, шаблонов и клонов.
Платформа получила возможность горячего подключения и замены виртуальных дисков, сетевых карт и USB-устройств, а также улучшенный веб-интерфейс.
Proxmox VE 3.4 распространяется под лицензией GNU AGPL v3, доступна для скачивания и предлагает платные подписки для бизнес-клиентов.
Решение используется на десятках тысяч хостов в более чем 140 странах и поддерживает KVM, OpenVZ-контейнеры и высокодоступность.

## Полная статья

**Vienna (Austria) – February 19, 2015 –**Proxmox Server Solutions GmbH today released version 3.4 of its open source server virtualization management platform Proxmox Virtual Environment (VE). Highlights are the integrated ZFS file system, a ZFS storage plug-in, hotplug and NUMA support (non-uniform memory access), all based on latest Debian Wheezy 7.8. The Proxmox developers considered many user feature requests and added many GUI improvements like start/stop all VMs, migrate all VMs or disconnect virtual network cards.

The integrated[ZFS (OpenZFS)](https://pve.proxmox.com/wiki/ZFS)is an open source file system and logical volume manager in one, allowing huge storage capacities. Starting with the new ISO installer for Proxmox VE 3.4, users can now select their preferred root file system during installation (ext3, ext4 or ZFS). All ZFS raid levels can be selected, including raid-0, 1, or 10 as well as all raidz levels (z-1, z-2, z3). ZFS on Proxmox VE can be used either as a local directory, supporting all storage content types (instead of ext3 or ext4) or as zvol block-storage, currently supporting KVM images in raw format (with the new ZFS storage plugin).

Using ZFS allows advanced setups for local storage like live snapshots and rollbacks but also space and performance efficient linked templates and clones. The ZFS storage plugin in Proxmox VE 3.4 complements already existing storage plugins like Ceph or the ZFS for iSCSI, GlusterFS, NFS, iSCSI and others.

The new hot plugging feature for virtual machines allows installing or replacing virtual hard disks, network cards or USB devices while the server is running. If hot plug is not possible, the new “pending changes” (marked now in red) show that the changes need a power off to be applied - the admin always overviews the actual status of his changes.

“ZFS in Proxmox VE 3.4 is absolutely powerful,” says Martin Maurer, CEO of Proxmox Server Solutions. “Users can replace cost intense hardware raid cards by moderate CPU and memory load, and it’s combined with easy management.” Maurer summarizes “By using ZFS, our users can achieve maximum enterprise features with low budget hardware – but they should not forget to add a SSD for a fast cache (L2ARC and ZIL).“

**Download**
Proxmox VE 3.4 is released under the free open-source license GNU AGPL, v3 and is available as ISO-image for download at[http://www.proxmox.com/downloads](https://www.proxmox.com/downloads). For enterprise customers, Proxmox offers subscriptions starting at 59.90 euros per year and CPU socket.

**Facts and Milestones**
Proxmox VE is used by more than 67.000 hosts in over 140 countries. The active community counts almost 26.000 forum members. The GUI is translated in 19 languages including Farsi, Basque and the two official forms of Norwegian Bokmål and Nynorsk.

- Proxmox VE 3.4 (February 2015): ZFS file system, ZFS storage plug-in, hotplug, NUMA support, Debian Wheezy 7.8
- Proxmox VE 3.3 (September 2014): HTML5 console, Proxmox VE Firewall, Two-factor authentication, ZFS storage plugin, and a touch interface Proxmox VE Mobile, qemu 2.1
- Proxmox VE 3.2 (March 2014): SPICE and spiceterm, Ceph storage system, Open vSwitch, support for VMware™ pvscsi and vmxnet3, new ZFS storage plugin, qemu 1.7
- Proxmox VE 3.1 (August 2013): Enterprise-Repository updates via GUI, SPICE, GlusterFS storage plugin
- Proxmox VE 3.0 (May 2013): VM templates and cloning, new event driven API server, Debian 7.0 (Wheezy), bootlogd
- Proxmox VE 2.0 (April 2012): High-Availability (HA) based on Redhat Cluster and Corosync; RESTful web API
- Proxmox VE 1.0 (October 2008): First stable release with KVM and container live migration and vzdump backups
- Proxmox VE 0.9 (April 2008): First public release with Web-GUI for managing KVM and containers

**About Proxmox Virtual Environment**
Proxmox Virtual Environment is an open source virtualization management solution for servers. It supports KVM-based guests as well as container-virtualization with OpenVZ and includes strong high-availability (HA) support based on Redhat Cluster and Corosync. Proxmox VE allows to virtualize even the most demanding Linux and Windows application workloads. Installation is fast and easy with a bare-metal installer and configuration is done via the integrated web-based management interface. Based on Debian GNU/Linux and fully licensed under the GNU Affero General Public License, Version 3 (AGPL-3.0), Proxmox VE is a solution without restrictions for home and business use.

**About Proxmox Server Solutions GmbH**
Proxmox Server Solutions GmbH is an open source software provider dedicated to develop powerful and efficient server solutions. With its two core products, Proxmox Virtual Environment (Proxmox VE) and Proxmox Mail Gateway, the company offers flexible, affordable and easy-to-use software for businesses implementing secure and open-source IT infrastructures. Proxmox solutions are widely used in businesses regardless of size, sector or industry as well as in NGOs and in the educational sector. The company also offers services like commercial support subscriptions as well as trainings. A worldwide partner network and a huge active community guarantee business continuity for Proxmox users. The company is an active member of the Linux Foundation and the Open Virtualization Alliance. Proxmox Server Solutions GmbH is an independent and profitable company based in Vienna, Austria.

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-3-4-released
