---
title: "Proxmox Virtual Environment 9.1 available"
date: 2025-11-19T09:25:53Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-9-1"
summary: "Proxmox объявила о выпуске Proxmox Virtual Environment 9.1, которая расширяет возможности контейнеризации, безопасности виртуальных машин и программно-определяемых сетей.   Теперь можно создавать LXC-контейнеры из OCI-образов, загружая их из реестров или вручную, включая оптимизированные application containers.   Обновление добавляет поддержку сохранения состояния vTPM в qcow2, что позволяет делать полные снимки виртуальных машин даже с активным виртуальным TPM.   Введена более точная настройка вложенной виртуализации через новый vCPU-флаг, полезный для вложенных гипервизоров и Windows VBS.   Интерфейс получил улучшенную отчетность SDN: отображение подключенных гостей, VNets, EVPN-зон, Fabrics, IP-VRF и MAC-VRF для упрощения мониторинга.   Proxmox VE 9.1 доступна сразу в виде ISO, поддерживает обновление через APT и установку поверх Debian, а enterprise-подписка начинается от 115 евро в год за CPU."
---

# Proxmox Virtual Environment 9.1 available

## Краткое содержание

Proxmox объявила о выпуске Proxmox Virtual Environment 9.1, которая расширяет возможности контейнеризации, безопасности виртуальных машин и программно-определяемых сетей.  
Теперь можно создавать LXC-контейнеры из OCI-образов, загружая их из реестров или вручную, включая оптимизированные application containers.  
Обновление добавляет поддержку сохранения состояния vTPM в qcow2, что позволяет делать полные снимки виртуальных машин даже с активным виртуальным TPM.  
Введена более точная настройка вложенной виртуализации через новый vCPU-флаг, полезный для вложенных гипервизоров и Windows VBS.  
Интерфейс получил улучшенную отчетность SDN: отображение подключенных гостей, VNets, EVPN-зон, Fabrics, IP-VRF и MAC-VRF для упрощения мониторинга.  
Proxmox VE 9.1 доступна сразу в виде ISO, поддерживает обновление через APT и установку поверх Debian, а enterprise-подписка начинается от 115 евро в год за CPU.

## Полная статья

**VIENNA, Austria – November 19, 2025 –**Leading open-source server solutions provider Proxmox Server Solutions GmbH (henceforth "Proxmox"), today announced the immediate availability of Proxmox Virtual Environment 9.1. The new version introduces significant enhancements across container deployment, virtual machine security, and software-defined networking, offering businesses greater flexibility, performance, and operational control.

## Highlights in Proxmox Virtual Environment 9.1

Create LXC containers from OCI images

Proxmox VE 9.1 integrates support for Open Container Initiative (OCI) images, a standard format for container distribution. Users can now download widely-adopted OCI images directly from registries or upload them manually to use as templates for LXC containers. Depending on the image, these containers are provisioned as full system containers or lean application containers. Application containers are a distinct and optimized approach that ensures minimal footprint and better resource utilization for microservices. This new functionality means administrators can now deploy standardized applications (e.g., a specific database or API service) from existing container build pipelines quickly and seamlessly through the Proxmox VE GUI or command line.

Support for TPM state in qcow2 format

This version introduces the ability to store the state of a virtual Trusted Platform Module (vTPM) in the qcow2 disk image format. This allows users to perform full VM snapshots, even with an active vTPM, across diverse storage types like NFS/CIFS. LVM storages with snapshots as volume chains now support taking offline snapshots of VMs with vTPM states. This advancement improves operational agility for security-sensitive workloads, such as Windows deployments that require a vTPM.

Fine-grained control of nested virtualization

Proxmox VE now offers enhanced control for nested virtualization in specialized VMs. This feature is especially useful for workloads such as nested hypervisors or Windows environments with Virtualization-based Security (VBS). A new vCPU flag allows to conveniently and precisely enable virtualization extensions for nested virtualization. This flexible option gives IT administrators more control and offers an optimized alternative to simply exposing the full host CPU type to the guest.

Enhanced SDN status reporting

Version 9.1 comes with an improved Software-Defined Networking (SDN) stack, including detailed monitoring and reporting in the web interface. The GUI now offers more visibility into the SDN stack, displaying all guests connected to local bridges or VNets. EVPN zones additionally report the learned IPs and MAC addresses. Fabrics are integrated into the resource tree, showing routes, neighbors, and interfaces. The updated GUI offers visibility into key network components like IP-VRFs and MAC-VRFs. This enhanced observability simplifies cluster-wide network troubleshooting and monitoring of complex network topologies, without the need for the command line.

### Availability

Proxmox Virtual Environment 9.1 is immediately available for download. Users can obtain a complete installation image via ISO download, which contains the full feature-set of the solution and can be installed quickly on bare-metal systems using an intuitive installation wizard.

Seamless distribution upgrades from older versions of Proxmox Virtual Environment are possible using the standard APT package management system. Furthermore, it is also possible to install Proxmox Virtual Environment on top of an existing Debian installation. As Free/Libre and Open Source Software (FLOSS), the entire solution is published under the GNU AGPLv3.

For enterprise users, Proxmox Server Solutions GmbH offers professional support through subscription plans. Pricing for these subscriptions starts at EUR 115 per year and CPU. A subscription provides access to the stable Enterprise Repository with timely updates via the web interface, as well as to certified technical support and is recommended for production use.

Resources:

- ISO Image Download:[https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/downloads)
- Forum Announcement:[https://forum.proxmox.com/](https://forum.proxmox.com/threads/proxmox-virtual-environment-9-1-available.176255/)
- Video tutorial:[What’s new in Proxmox VE 9.1](https://www.proxmox.com/en/services/training-courses/videos/proxmox-virtual-environment/whats-new-in-proxmox-ve-9-1)
- Roadmap: For published and upcoming features, see the[Release Notes & Roadmap](https://pve.proxmox.com/wiki/Roadmap#Proxmox_VE_9.1)

[](https://www.proxmox.com)###

**Facts**
The open-source project Proxmox VE has a huge worldwide user base with more than 1.6 million hosts. The virtualization platform has been translated into over 31 languages. More than 225,000 active community members in the support forum engage with and help each other. By using Proxmox VE as an alternative to proprietary virtualization management solutions, enterprises are able to centralize and modernize their IT infrastructure, and turn it into a cost-effective and flexible software-defined data center, based on the latest open-source technologies. Tens of thousands of customers rely on enterprise support subscriptions from Proxmox Server Solutions GmbH.

**About Proxmox Server Solutions**
Proxmox provides powerful and user-friendly open-source server software. Enterprises of all sizes and industries use the Proxmox solutions to deploy efficient and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support, training services, and an extensive partner ecosystem to ensure business continuity for its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH, marketing@proxmox.com

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-9-1
