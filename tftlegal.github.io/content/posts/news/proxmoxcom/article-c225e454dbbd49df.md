---
title: "Proxmox Virtual Environment Version 2.3 with new KVM Live Backup Technology"
date: 2013-06-04T11:32:40Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-version-2-3-available"
summary: "Proxmox Server Solutions объявила о выпуске версии 2.3 открытой платформы виртуализации Proxmox Virtual Environment. Ключевым нововведением стала поддержка живого резервного копирования и восстановления для KVM-виртуальных машин, работающая с различными типами хранилищ, включая NFS, iSCSI, Ceph RBD и Sheepdog. Версия также добавила GUI-поддержку хранения дисков KVM на Ceph RBD и включила улучшения QEMU/KVM 1.4. В Proxmox VE 2.3 улучшено управление памятью благодаря KSM и автоматическому ballooning, что позволяет эффективнее использовать физическую память хоста. Proxmox VE 2.3 позиционируется как бесплатная открытая бизнес-альтернатива VMware vSphere, Microsoft Hyper-V, Oracle VM и Citrix XenServer и доступна для скачивания."
---

# Proxmox Virtual Environment Version 2.3 with new KVM Live Backup Technology

## Краткое содержание

Proxmox Server Solutions объявила о выпуске версии 2.3 открытой платформы виртуализации Proxmox Virtual Environment.
Ключевым нововведением стала поддержка живого резервного копирования и восстановления для KVM-виртуальных машин, работающая с различными типами хранилищ, включая NFS, iSCSI, Ceph RBD и Sheepdog.
Версия также добавила GUI-поддержку хранения дисков KVM на Ceph RBD и включила улучшения QEMU/KVM 1.4.
В Proxmox VE 2.3 улучшено управление памятью благодаря KSM и автоматическому ballooning, что позволяет эффективнее использовать физическую память хоста.
Proxmox VE 2.3 позиционируется как бесплатная открытая бизнес-альтернатива VMware vSphere, Microsoft Hyper-V, Oracle VM и Citrix XenServer и доступна для скачивания.

## Полная статья

**VIENNA, March 4, 2013*****-***Proxmox Server Solutions GmbH, developer of the open-source virtualization platform Proxmox Virtual Environment, today announced the release of version 2.3. The version brings new compelling features like KVM live backup technology as well as the integration of the Ceph RBD (RADOS Block Device) as storage plugin.

**KVM backup and restore**
The key feature of Proxmox VE 2.3 is the new KVM backup and restore, replacing LVM snapshots. The benefit of KVM live backup is that it works for all storage types including VM images on NFS, iSCSI LUN, Ceph RBD or Sheepdog. The new backup format is optimized for storing VM backups fast and effective (sparse files, out of order data, minimized I/O).
The V2.3 also includes the latest QEMU/KVM version 1.4 with many improvements as well as GUI support for the storage of KVM VM disks on Ceph RADOS Block Device (RBD) storage system.

**Dynamic Memory Management**
Optimized and effective memory management is a key factor in virtualization environments. KSM and Auto-Ballooning enables sophisticated and economic configurations for physical RAM utilization. Memory ballooning (KVM only) allows you to have your guest dynamically change it’s memory usage by evicting unused memory during run time. It reduces the impact your guest can have on memory usage of your host by giving up unused memory back to the host.

„Proxmox Virtual Environment 2.3 is the open-source business alternative to VMware vSphere, Microsoft Hyper-V, Oracle VM or Citrix XenServer“, says Martin Maurer, CEO of Proxmox Server Solutions GmbH. In addition to regular product updates, a huge and active community (more than 20.000 community forum members) and a worldwide partner network Proxmox VE users can benefit from services like business support subscriptions and trainings.

**Availability**
Proxmox Virtual Environment 2.3 is available as free ISO-image download at[http://www.proxmox.com/downloads/proxmox-ve]

**About Proxmox Virtual Environment 2.3**
Proxmox Virtual Environment is a complete open-source virtualization management solution for servers. It combines Kernel-based Virtual Machine (KVM) and OpenVZ containers on one platform and is manageable via a fast web GUI. With the HA cluster, Proxmox VE meets the high availability requirements of companies.
Proxmox Virtual Environment is based on the stable Debian GNU/Linux with a modified RHEL6 kernel. It is licensed under the GNU Affero General Public License v3 (AGPL v3) which makes it to a free available solution for businesses. The Proxmox VE 2.3 GUI is available for more than 14 languages.

**About Proxmox Server Solutions GmbH**
Proxmox Server Solutions GmbH is a software provider dedicated to develop powerful and easy-to-use server solutions that ease the work of people and companies. With its two core products – Proxmox Virtual Environment (PVE) and Proxmox Mail Gateway (PMG) – the company offers flexible, affordable and easy-to-use software for implementing a secure open-source IT infrastructure.

Proxmox solutions are widely used in businesses regardless of size, sector or industry as well as in NGOs and in the educational sector. With a worldwide partner network and a huge active community, Proxmox users benefit from excellent and fast support. Regular updates, commercial support subscriptions as well as trainings guarantee business continuity. Proxmox is an active member of the Open Virtualization Alliance, the worldwide KVM consortium. Proxmox Server Solutions GmbH is a privately held corporation based in Vienna, Austria.
Website: www.proxmox.com

**Additional information:**
Website: www.proxmox.com
Project page Proxmox VE:[https://pve.proxmox.com](https://pve.proxmox.com)
Downloads:[https://www.proxmox.com/downloads](https://www.proxmox.com/en/downloads)

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-version-2-3-available
