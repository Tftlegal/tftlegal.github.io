---
title: "Proxmox VE 3.2 with SPICE, Ceph and Open vSwitch released"
date: 2014-03-10T14:00:56Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-3-2-released"
summary: "Компания Proxmox Server Solutions выпустила версию Proxmox VE 3.2 — открытой платформы серверной виртуализации.   Главные улучшения — поддержка распределённой системы хранения Ceph, удалённого просмотра SPICE с утилитой spiceterm и сетевого коммутатора Open vSwitch.   Ceph RBD можно использовать для дисков виртуальных машин, а кластер Ceph настраивается через веб-интерфейс Proxmox.   SPICE теперь доступен как альтернатива VNC для консолей виртуальных машин, контейнеров OpenVZ и хоста.   Также обновлены qemu до 1.7, улучшен live backup, добавлены поддержка VMware pvscsi и vmxnet3, новый ZFS-плагин и исправления ошибок.   Proxmox VE 3.2 распространяется бесплатно под AGPLv3, а для бизнеса доступны платные подписки.   Компания подчёркивает, что новые функции повышают отказоустойчивость, масштабируемость и конкурентоспособность платформы."
---

# Proxmox VE 3.2 with SPICE, Ceph and Open vSwitch released

## Краткое содержание

Компания Proxmox Server Solutions выпустила версию Proxmox VE 3.2 — открытой платформы серверной виртуализации.  
Главные улучшения — поддержка распределённой системы хранения Ceph, удалённого просмотра SPICE с утилитой spiceterm и сетевого коммутатора Open vSwitch.  
Ceph RBD можно использовать для дисков виртуальных машин, а кластер Ceph настраивается через веб-интерфейс Proxmox.  
SPICE теперь доступен как альтернатива VNC для консолей виртуальных машин, контейнеров OpenVZ и хоста.  
Также обновлены qemu до 1.7, улучшен live backup, добавлены поддержка VMware pvscsi и vmxnet3, новый ZFS-плагин и исправления ошибок.  
Proxmox VE 3.2 распространяется бесплатно под AGPLv3, а для бизнеса доступны платные подписки.  
Компания подчёркивает, что новые функции повышают отказоустойчивость, масштабируемость и конкурентоспособность платформы.

## Полная статья

**Vienna - March 10, 2014 -**Proxmox Server Solutions GmbH, developer of the open source server virtualization platform Proxmox Virtual Environment (VE), today released version 3.2. Big enhancements in this release are the SPICE multi-monitor remote viewer for virtual servers and containers (with spiceterm), the distributed Ceph storage system and Open vSwitch. Countless updates are added like qemu 1.7, improved live backup, support for VMware™ pvscsi and vmxnet3, a new ZFS storage plugin, latest NIC drivers and bug fixes.

**Ceph Storage Server Integration**
Proxmox VE 3.2 includes the ability to build the Ceph storage cluster directly on Proxmox VE hosts. Ceph is a massively scalable, open source distributed object store and file system that is very popular in many cloud computing deployments. Proxmox VE 3.2 supports Ceph's RADOS Block Device (Ceph RBD) to be used for VM disks. Ceph storage cluster can be administered via the Proxmox web GUI and configuration is stored in Proxmox' shared file system (pmxcfs) which is replicated throughout the cluster. Ceph is a redundant system and has no single point of failure, which makes it perfect for critical environments.

**SPICE and spiceterm**
Proxmox VE integrates the Simple Protocol for Independent Computing Environments (SPICE) since version 3.1 as technology preview. With the new version 3.2, the user can now choose between SPICE and VNC for accessing the virtual machine consoles. Plenty of improvements were made: By using spiceterm, a program developed by Proxmox, SPICE can be additionally used for accessing OpenVZ containers or the host shell. A spiceterm console can be resized and offers a fully functional keyboard including special characters like the pipe or @ symbol and function keys like control-c do work. On the client side, it is sufficient to install a SPICE remote viewer. Virt-viewer packages are available for Linux and Windows, and for Android there is compatible App named aSPICE.

**Open vSwitch**
Another enhancement of Proxmox VE in version 3.2 is the introduction of the network software switch Open vSwitch (OVS) on the host network level (a technology preview in 3.2). Open vSwitch is a multilayer virtual switch enabling massive network automation, while still supporting standard management interfaces and protocols as well as distribution across multiple physical servers. It consists of user space tools and kernel modules.

"We are totally excited with the new Proxmox VE 3.2 because it challenges the virtualization market with Ceph storage and SPICE," states Martin Maurer, CEO of Proxmox Server Solutions. "With the integrated Ceph RBD, Proxmox users get a highly available solution with no single points of failure and extreme scalability. This makes it ideal for applications which require high available flexible storage."

**Availability of Proxmox VE 3.2**
Proxmox VE 3.2 is released under the AGPL, v3 and is available as free ISO-image for download at[http://www.proxmox.com/downloads](https://www.proxmox.com/en/component/zoo/?view=frontpage&layout=frontpage&Itemid=120)
For enterprise customers, Proxmox offers subscriptions starting at EUR 49,90 per year and CPU socket.

**Facts and Milestones Proxmox VE**
Proxmox VE is used by more than 57.000 hosts in 140 countries. The GUI is available in 17 languages and the active community counts more than 23.000 forum members.

- Proxmox VE V3.2 with SPICE and spiceterm, Ceph storage system, Open vSwitch, support for VMware™ pvscsi and vmxnet3, new ZFS storage plugin, qemu 1.7 in March 2014.
- Proxmox VE V3.1 with Enterprise-Repository updates via GUI, SPICE, GlusterFS storage plugin in August 2013.
- Proxmox VE V3.0 brings VM templates and cloning, new event driven API server, Debian 7.0 (Wheezy), bootlogd in May 2013.
- Proxmox VE V2.0 with High-Availability (HA) based on Redhat Cluster and Corosync; RESTful web API is released in April 2012.
- Proxmox VE V0.9 - First public release in April 2008. GUI for managing KVM and containers.

**About Proxmox Virtual Environment**
Proxmox Virtual Environment is a open source complete virtualization management solution for servers. It supports KVM-based guests, as well as container-virtualization with OpenVZ and includes strong high-availability (HA) support based on Redhat Cluster and Corosync. You can easily virtualize even the most demanding Linux and Windows application workloads with Proxmox VE. Installation is fast and easy with a bare-metal installer and configuration is done via the integrated web-based management interface. Based on Debian GNU/Linux and fully licensed under the GNU Affero General Public License, Version 3 (AGPL-3.0), Proxmox VE is a solution without restrictions for home and business use.

**About Proxmox Server Solutions GmbH**
Proxmox Server Solutions GmbH is an open source software provider dedicated to develop powerful and efficient server solutions. With its two core products, Proxmox Virtual Environment (Proxmox VE) and Proxmox Mail Gateway, the company offers flexible, affordable and easy-to-use software for businesses implementing secure and open-source IT infrastructures.

Proxmox solutions are widely used in businesses regardless of size, sector or industry as well as in NGOs and in the educational sector. The company also offers services like commercial subscriptions and trainings. A worldwide partner network and a huge active community guarantee business continuity for Proxmox users. Proxmox Server Solutions GmbH is an independent and profitable company based in Vienna, Austria. Website:[http://www.proxmox.com](http://www.proxmox.com)

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-3-2-released
