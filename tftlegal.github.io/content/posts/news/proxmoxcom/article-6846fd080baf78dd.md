---
title: "Proxmox Virtual Environment 7.4 released"
date: 2023-03-23T10:09:10Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-7-4"
summary: "Proxmox Virtual Environment 7.4 — это новая версия открытой платформы виртуализации, которая делает упор на улучшение управления через веб-интерфейс.   Выпуск основан на Debian 11.6 и использует ядро Linux 5.15, а при необходимости можно установить более новое ядро 6.2 для поддержки современного оборудования.   Платформа обновлена до актуальных версий ключевых технологий, включая QEMU, LXC, ZFS и Ceph.   Среди главных нововведений — тёмная тема, подробная информация о Ceph OSD, загрузка журналов задач, сортировка ресурсов и расширение возможностей HA-менеджера.   Также улучшены производительность управления доступом, установка, локализация, работа с контейнерами и другие административные функции.   Proxmox VE 7.4 остаётся бесплатным open-source продуктом, а компания Proxmox предлагает платную подписку с технической поддержкой и enterprise-репозиторием."
---

# Proxmox Virtual Environment 7.4 released

## Краткое содержание

Proxmox Virtual Environment 7.4 — это новая версия открытой платформы виртуализации, которая делает упор на улучшение управления через веб-интерфейс.  
Выпуск основан на Debian 11.6 и использует ядро Linux 5.15, а при необходимости можно установить более новое ядро 6.2 для поддержки современного оборудования.  
Платформа обновлена до актуальных версий ключевых технологий, включая QEMU, LXC, ZFS и Ceph.  
Среди главных нововведений — тёмная тема, подробная информация о Ceph OSD, загрузка журналов задач, сортировка ресурсов и расширение возможностей HA-менеджера.  
Также улучшены производительность управления доступом, установка, локализация, работа с контейнерами и другие административные функции.  
Proxmox VE 7.4 остаётся бесплатным open-source продуктом, а компания Proxmox предлагает платную подписку с технической поддержкой и enterprise-репозиторием.

## Полная статья

**VIENNA, Austria – March 23, 2023**– Enterprise software developer Proxmox Server Solutions GmbH (henceforth "Proxmox") released version 7.4 of its server virtualization management platform, Proxmox Virtual Environment, today. The new version includes a focus on improved usability, bringing even more management tasks to the web interface.

### Highlights in Proxmox Virtual Environment 7.4

- Proxmox VE 7.4 is based on**Debian 11.6 (“Bullseye”)**, but uses a newer, long-term supported Linux kernel 5.15 and includes updates to the latest versions of leading open-source technologies for virtual environments, such as QEMU 7.2, LXC 5.0.2, and ZFS 2.1.9. The platform supports Ceph Quincy 17.2.5 as default for new installations or optionally Ceph Pacific 16.2.11.
- By default, Proxmox VE uses a newer, long-term supported Linux kernel 5.15.**Optionally, Linux kernel 6.2**can be installed, providing support for the current features of the latest hardware. The kernel can be installed via the shell by selecting the pve-kernel-6.2 meta package. After a reboot, the new kernel will be activated.
- Dark theme: A fully-integrated**"Proxmox Dark" theme**variant is now available for the web interface and the lovers of darkness. The theme is based on the Crisp light theme. To detect if a user has requested light or dark color themes, the CSS media feature prefers-color-scheme is used. Through settings in the operating system or their browser, users can indicate their preferences. However, users can switch between the color schemes manually in the web interface as well.
- Ceph OSD: The web interface and the API now**display detailed OSD information**. This reduces work for administrators, as they no longer have to gather the necessary information from various places.
- **Task logs can be downloaded**as text files in the GUI, which helps to make further inspection easier.
- **Sorting of the resource tree**: VMs or containers can now be sorted by their names or by VMID, by selecting the appropriate sort-order. Especially in large clusters, sorting all guests by name can provide a better overview for users who rely on them.
- HA Manager: By adding a CRM command, an online node can be**switched manually into maintenance**(without reboot). Also, the support for the HA Cluster Resource Scheduler (CRS) stack was extended and can now**rebalance VMs & containers automatically on start**, not only on recovery.

### Further notable enhancements

- To avoid the browser error "Connection reset", HTTP requests are automatically redirected to HTTPS. This can prevent confusion, especially after setting up a Proxmox VE host for the first time.
- When users select the storage view for a Proxmox Backup Server storage, they can then use the columns for encryption and verification status for sorting.
- Containers: The open-source architectures riscv32 and riscv64 can be used for LXC containers. Additionally, some container templates for amd64 were updated to newer upstream versions.
- Virtual guests: Renaming the "Bulk Stop" action to "Bulk Shutdown" reflects the actual behavior of this action.
- HA Manager: In preparation for the dynamic load scheduling in clusters, the high availability manager abstracts and unifies the needed resources (CPU, memory) of the different HA services (containers, VMs).
- Improved management for Proxmox VE clusters: the Proxmox Offline Mirror tool now supports using HTTP proxies for fetching updates.
- Storage: With the help of the option content-dirs, the specific subdirectories for content-types—for example isos, container templates, backups, guest disks—can be overridden with custom values.
- Access Control: By refactoring the ACL computation, performance significantly improved—up to a factor of 450—in setups with many entries and huge user bases or with many access control rules.
- Notifications for new package updates can optionally be disabled, further improving the administration experience .
- Installation ISO: During installation, the UTC timezone can be selected, facilitating the synchronization of hosts or clusters that are distributed across multiple geographic locations.
- Localization (i18n): Translations for Arabic, French, German, Italian, Japanese, Russian, Slovenian, and Traditional Chinese were improved.

### Availability

Proxmox Virtual Environment is free and open-source software, published under the GNU Affero General Public License, v3. The ISO contains the complete feature-set and can be installed on bare-metal. Proxmox VE 7.4 is available for download at[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads)

The virtualization platform from Proxmox comes stocked with all the essential management tools, as well as an integrated, web-based user interface. This allows for simple out-of-the-box management of the host, either through the command line or a standard web browser.

Distribution upgrades from older versions of Proxmox VE are possible with apt. It’s also possible to install Proxmox VE 7.4 on top of Debian.

For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 105 per year and CPU.

**Forum Announcement**
[https://forum.proxmox.com](https://forum.proxmox.com)

**Video tutorial**
[What's new in Proxmox VE 7.4](https://www.proxmox.com/en/training/video-tutorials/item/what-s-new-in-proxmox-ve-7-4)

**Facts**
The open-source project Proxmox VE has a huge worldwide user base with more than 750.000 hosts. The virtualization platform has been translated into over 26 languages. More than 110.000 active community members in the support forum engage with and help each other. By using Proxmox VE as an alternative to proprietary virtualization management solutions, enterprises are able to centralize and modernize their IT infrastructure, and turn it into a cost-effective and flexible software-defined data center, based on the latest open-source technologies. Tens of thousands of customers rely on a enterprise support subscription from Proxmox Server Solutions GmbH.

**About Proxmox Virtual Environment**
Proxmox Virtual Environment (Proxmox VE) is the leading open-source platform for all-inclusive enterprise virtualization. With the central web interface, you can easily run VMs and containers, manage software-defined storage and networking functionality, high-availability clustering, and multiple integrated out-of-the-box tools like backup/restore, live migration, replication, and the firewall. Enterprises use the powerful yet easy-to-manage, all-in-one solution to meet the core requirements of today’s modern data centers. Proxmox VE allows them to remain adaptable for future growth, thanks to its flexible, modular and open architecture.

**About Proxmox Server Solutions**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training services to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**Media contact**
Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-7-4
