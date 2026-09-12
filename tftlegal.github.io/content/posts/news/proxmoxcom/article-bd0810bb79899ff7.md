---
title: "Proxmox VE 4.0 Beta1 available"
date: 2015-06-23T08:25:31Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-4-0-beta1"
summary: "Proxmox Server Solutions объявила о выпуске первой публичной бета-версии Proxmox VE 4.0.   Ключевым нововведением стал новый HA Manager, который заменил rgmanager и автоматически реагирует на сбои виртуальных машин и контейнеров.   В версии также добавлен Proxmox HA Simulator для тестирования функций отказоустойчивости перед развёртыванием в продакшене.   Proxmox VE 4.0 впервые получит полноценную поддержку Linux-контейнеров LXC, а также стабильные пакеты DRBD9 для высокопроизводительных рабочих нагрузок.   Бета-релиз предназначен для широкого тестирования, и скачать его можно по ссылке с официального сайта Proxmox."
---

# Proxmox VE 4.0 Beta1 available

## Краткое содержание

Proxmox Server Solutions объявила о выпуске первой публичной бета-версии Proxmox VE 4.0.  
Ключевым нововведением стал новый HA Manager, который заменил rgmanager и автоматически реагирует на сбои виртуальных машин и контейнеров.  
В версии также добавлен Proxmox HA Simulator для тестирования функций отказоустойчивости перед развёртыванием в продакшене.  
Proxmox VE 4.0 впервые получит полноценную поддержку Linux-контейнеров LXC, а также стабильные пакеты DRBD9 для высокопроизводительных рабочих нагрузок.  
Бета-релиз предназначен для широкого тестирования, и скачать его можно по ссылке с официального сайта Proxmox.

## Полная статья

**VIENNA, Austria – June 23, 2015 –**Proxmox Server Solutions GmbH, developer of the open source server virtualization platform Proxmox Virtual Environment (VE), has announced that the first public beta release of Proxmox VE 4.0 is available for testing.

The[Proxmox VE HA Manager](https://pve.proxmox.com/wiki/High_Availability_Cluster_4.x)(pve-ha-manager), the new resource manager for the high availability cluster is one of the main new features. The pve-ha-manager, developed by the Proxmox team, replaces the former rgmanager. The HA manager monitors all virtual machines and containers on the cluster and automatically gets into action if one of them fails. It works out of the box, and additionally watchdog-based fencing simplifies deployments dramatically. The whole HA settings are configured via GUI.

This beta version also comes with a brand-new Proxmox HA Simulator allowing users to learn and test all the functionality of the Proxmox VE HA solution prior to going into production.

Proxmox VE 4.0 will be the first version to include[Linux containers (LXC)](https://pve.proxmox.com/wiki/Linux_Container). The new container solution for Proxmox VE will be fully integrated into the Proxmox VE frameworks, e.g. this includes also the storage plugins. It works with all modern and latest Linux kernels.

Also integrated in this beta version are the first stable DRBD9 packages. DRBD9 is perfectly suited for high performance workloads, especially when high IOPS are required.

This beta release is made available to allow a broad user base to test and evaluate the next major version of Proxmox VE.

Proxmox VE 4.0 beta1 is available for testing from[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads)[.](https://www.proxmox.com/downloads.)

Official Announcement:[https://forum.proxmox.com/threads/22532-Proxmox-VE-4-0-beta1-released!](https://forum.proxmox.com/threads/22532-Proxmox-VE-4-0-beta1-released!)

**About Proxmox Virtual Environment**
Proxmox Virtual Environment is a complete open source virtualization management solution for servers. It supports KVM full virtualization as well as container-based virtualization and includes strong high-availability (HA) support based on Redhat Cluster and Corosync. Proxmox VE allows to virtualize even the most demanding Linux and Windows application workloads. Installation is fast and easy with a bare-metal installer and configuration is done via the integrated web-based management interface. Based on Debian GNU/Linux and fully licensed under the GNU Affero General Public License, Version 3 (AGPL-3.0), Proxmox VE is a solution without restrictions for home and business use.

**About Proxmox Server Solutions GmbH**
Proxmox Server Solutions GmbH develops open source software providing powerful and efficient server solutions to its customers. With its two core products, Proxmox Virtual Environment (Proxmox VE) and Proxmox Mail Gateway, the company offers flexible, affordable and easy-to-use software for businesses implementing secure and open-source IT infrastructures. Proxmox solutions are widely used in businesses regardless of size, sector or industry as well as in NGOs and in the educational sector. The company also offers services like commercial support subscriptions and trainings. A worldwide partner network and a huge active community guarantee business continuity for Proxmox users. The company is an active member of the Linux Foundation and the Open Virtualization Alliance. Proxmox Server Solutions GmbH is an independent and profitable company based in Vienna, Austria.

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-4-0-beta1
