---
title: "Proxmox introduces Cluster Resource Scheduler for Proxmox Virtual Environment 7.3"
date: 2022-11-10T16:46:03Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-7-3"
summary: "Компания Proxmox объявила о выпуске Proxmox VE 7.3 — обновленной платформы виртуализации на базе Debian 11.5 и Linux-ядра 5.15.   Она объединяет виртуальные машины, контейнеры, программно-определяемое хранилище и сеть в одном инструменте.   В версии добавлена начальная поддержка планировщика ресурсов кластера (CRS) с использованием метода TOPSIS для выбора целевого узла для HA-сервисов.   Также представлена утилита Proxmox Offline Mirror для обновления систем без доступа в интернет и поддержка Ceph Quincy/Pacific.   Среди других улучшений — теги для виртуальных гостевых систем, ZFS dRAID, CPU pinning, улучшенная работа с USB и обновленное мобильное приложение.   Proxmox VE 7.3 распространяется бесплатно как open-source ПО, а для предприятий доступна платная подписка с поддержкой и enterprise-репозиторием."
---

# Proxmox introduces Cluster Resource Scheduler for Proxmox Virtual Environment 7.3

## Краткое содержание

Компания Proxmox объявила о выпуске Proxmox VE 7.3 — обновленной платформы виртуализации на базе Debian 11.5 и Linux-ядра 5.15.  
Она объединяет виртуальные машины, контейнеры, программно-определяемое хранилище и сеть в одном инструменте.  
В версии добавлена начальная поддержка планировщика ресурсов кластера (CRS) с использованием метода TOPSIS для выбора целевого узла для HA-сервисов.  
Также представлена утилита Proxmox Offline Mirror для обновления систем без доступа в интернет и поддержка Ceph Quincy/Pacific.  
Среди других улучшений — теги для виртуальных гостевых систем, ZFS dRAID, CPU pinning, улучшенная работа с USB и обновленное мобильное приложение.  
Proxmox VE 7.3 распространяется бесплатно как open-source ПО, а для предприятий доступна платная подписка с поддержкой и enterprise-репозиторием.

## Полная статья

**VIENNA, Austria – November 22, 2022 –**Enterprise software developer Proxmox Server Solutions (henceforth "Proxmox"), the company behind Proxmox Virtual Environment (VE) today announced enhancements to its server virtualization management platform, now generally available. Users can seamlessly deploy virtual machines and containers, software-defined storage, and networking on a single platform. The new point release Proxmox VE 7.3 is based on Debian 11.5 (“Bullseye”), but using a newer Linux kernel 5.15, and includes updates to the latest versions of leading open-source technologies for virtual environments like QEMU 7.1, LXC 5.0.0, and ZFS 2.1.6. The virtualization platform supports Ceph Quincy 17.2.5 as default for new installations, and optionally Ceph Pacific 16.2.10.

Proxmox Virtual Environment 7.3 comes with initial support for Cluster Resource Scheduling, enables updates for air-gapped systems with the new Proxmox Offline Mirror tool, and has improved UX for various management tasks, as well as interesting storage technologies like ZFS dRAID.

### Highlights in Proxmox Virtual Environment 7.3

**Technology preview: Initial support for a Cluster Resource Scheduler (CRS):**In cases when the Proxmox HA Manager needs to find a new host node for a HA service (e.g. recovery, shutdown-policy, HA group configuration changes), it previously used to only count the number of active HA services for load balancing. This version integrates the TOPSIS (Technique for Order of Preference by Similarity to Ideal Solution) method, a multi-criteria decision making analysis, for finding the best alternative target node.

When the new scheduler mode "Static Load" is applied, the TOPSIS tool uses the "total memory" and "vCPU" properties of the HA resource to guide the decision on which node in the cluster a HA resource is started on. With this CRS foundation established, the Proxmox developers plan to extend it in future releases with a dynamic load scheduler and live load balancing.

**Proxmox Offline Mirror:**The Proxmox Offline Mirror tool allows to keep Proxmox VE nodes – with restricted or without access to the public internet – up-to-date and running. With the ‘proxmox-offline-mirror’ utility it’s possible to manage a local apt mirror for all package updates for Proxmox and Debian projects. From that mirror, users can create an external medium (USB flash drive or a local network share), and can then update their policy-restricted or air-gapped systems. For subscribers with a Premium and Standard subscription level, Proxmox offers an offline subscription key for its product portfolio.

**Support for Ceph Quincy and Ceph Pacific:**Proxmox Virtual Environment 7.3 supports Ceph Quincy 17.2.5 as default and Ceph Pacific 16.2.10. The preferred Ceph version can be selected during the installation process. The Proxmox developers have added heuristical checks to see if it is safe to stop or remove a service instance (MON, MDS, OSD). If removing or stopping a service would affect the cluster operation, the web interface will now alert users with a warning. Overall, the usability for creating new clusters, including selecting the network and checking duplicate IP, has been improved.

**Tags for virtual guests:**With the option of tagging virtual guests in the web interface, administrators can gain even better overview and simplify management tasks, for example by differentiating between ‘production’ and ‘testing’ VMs. Tags can either be user-modifiable, or restricted to higher-privileged administrator accounts. Tagging has been already available in the configuration (CLI, API), providing users with a possibility for automatization.

### Further notable enhancements

- ZFS dRAID pools: This version comes with support for creating ZFS dRAID pools in the web interface. Declustered RAID (dRAID) – a feature added in OpenZFS 2.1.0 – is a software raid implementation improving disk recovery time in large storage arrays. If a disk drive fails, the hot spare drive(s) participating in the RAID are used for rebuilding. As its layout is more complex than RAIDz, dRAID is an option for large storage arrays.
- Virtual Machines:

- CPU pinning: With the taskset command, a VM can be bind to a certain or a range of CPU core(s). CPU affinity can help optimize cache performance, for example in multiprocessing computers.
- USB devices have been reworked and can now be hot-plugged; also it’s possible to pass through up to 14 USB devices (previous: five) to a virtual machine.

- Proxmox VE Mobile App: The Proxmox VE Android app is based on Flutter 3.0, and allows users to access the Proxmox VE server(s), and manage cluster, nodes, VMs, and containers. The app now supports and targets Android 13. Also feedback about running backup tasks is available.

### Availability

Proxmox Virtual Environment is free and open-source software, published under the GNU Affero General Public License, v3. The ISO contains the complete feature-set and can be installed on bare-metal. Proxmox VE 7.3 is generally[available for download](https://www.proxmox.com/en/component/zoo/?view=frontpage&layout=frontpage&Itemid=120).

The virtualization stack from Proxmox comes stocked with all the essential management tools, as well as an easy-to-use, web-based user interface. This allows for simple, out-of-the-box management of the host, either through the command line or a standard web browser.

Distribution upgrades from older versions of Proxmox VE are possible with apt. It’s also possible to install Proxmox Virtual Environment 7.3 on top of Debian Bullseye.

For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to an Enterprise Repository, with regular updates via the web interface, as well as[technical support directly from the developers](https://www.proxmox.com/en/products/proxmox-virtual-environment/pricing). Prices start at EUR 95 per year and CPU.

**Forum Announcement**
[https://forum.proxmox.com/threads/proxmox-ve-7-3-released.118390/](https://forum.proxmox.com/threads/proxmox-ve-7-3-released.118390/)

**Video tutorial**
[What's new in Proxmox VE 7.3](https://www.proxmox.com/de/training/video-tutorials/item/what-s-new-in-proxmox-ve-7-3)

**Facts**
The open-source project Proxmox VE has a huge worldwide user base with more than 600.000 hosts. The virtualization platform has been translated into over 26 languages. More than 100.000 active community members in the support forum engage with and help each other. By using Proxmox VE as an alternative to proprietary virtualization management solutions, enterprises are able to centralize and modernize their IT infrastructure, and turn it into a cost-effective and flexible software-defined data center, based on the latest open-source technologies. Tens of thousands of customers rely on a enterprise support subscription from Proxmox Server Solutions GmbH.

**About Proxmox Server Solutions**
Proxmox® is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training services to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-7-3
