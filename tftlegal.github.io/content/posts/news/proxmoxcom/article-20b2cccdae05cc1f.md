---
title: "Proxmox Virtual Environment 9.2 with Dynamic Load Balancer released"
date: 2026-05-21T07:32:09Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-9-2"
summary: "Proxmox Server Solutions представила Proxmox Virtual Environment 9.2 — обновлённую open-source платформу для корпоративной виртуализации.   Ключевым нововведением стал динамический балансировщик нагрузки, который автоматически перемещает виртуальные машины для более равномерного использования ресурсов кластера с учётом правил High Availability.   Также расширены возможности SDN: добавлены WireGuard, BGP, фильтрация маршрутов EVPN, поддержка OSPF-фабрик и IPv6 underlay.   Введена удобная настройка пользовательских моделей CPU, включая просмотр поддерживаемых флагов на всех узлах кластера.   Для упрощения обслуживания добавлена возможность временно отключать и включать HA-менеджер, сохраняя состояние ресурсов.   Proxmox VE 9.2 построена на Debian 13.5 и Linux kernel 7.0, поддерживает Ceph Tentacle 20.2 и уже доступна для загрузки и обновления."
---

# Proxmox Virtual Environment 9.2 with Dynamic Load Balancer released

## Краткое содержание

Proxmox Server Solutions представила Proxmox Virtual Environment 9.2 — обновлённую open-source платформу для корпоративной виртуализации.  
Ключевым нововведением стал динамический балансировщик нагрузки, который автоматически перемещает виртуальные машины для более равномерного использования ресурсов кластера с учётом правил High Availability.  
Также расширены возможности SDN: добавлены WireGuard, BGP, фильтрация маршрутов EVPN, поддержка OSPF-фабрик и IPv6 underlay.  
Введена удобная настройка пользовательских моделей CPU, включая просмотр поддерживаемых флагов на всех узлах кластера.  
Для упрощения обслуживания добавлена возможность временно отключать и включать HA-менеджер, сохраняя состояние ресурсов.  
Proxmox VE 9.2 построена на Debian 13.5 и Linux kernel 7.0, поддерживает Ceph Tentacle 20.2 и уже доступна для загрузки и обновления.

## Полная статья

**VIENNA, Austria – May 21, 2026 –**Proxmox Server Solutions GmbH today announced the immediate availability of Proxmox Virtual Environment 9.2, the latest version of its integrated open-source platform for enterprise virtualization. This major update introduces a dynamic load balancer, expanded software-defined networking (SDN) capabilities, and granular management of custom CPU models. By improving resource utilization through dynamic workload balancing and simplifying complex cluster maintenance workflows, Proxmox VE 9.2 enables organizations to scale their infrastructure with higher efficiency and significantly reduced operational complexity.

## Highlights in Proxmox Virtual Environment 9.2

Dynamic Load Balancer

A highlight of version 9.2 is the introduction of the Dynamic Load Balancer, which utilizes an intelligent decision-making framework to optimize guest placement for maximum cluster balance and reliability. Operating in a new dynamic mode, the cluster resource scheduler (CRS) incorporates real-time node and guest resource utilization into every placement decision. The integrated load balancer can automatically migrate guests managed by the High Availability (HA) stack to reduce the imbalance across the cluster nodes while strictly respecting all user-defined HA rules. Administrators maintain granular control through configurable options that define the behavior and sensitivity of the load Balancer through various parameters, providing organizations with superior oversight of resource utilization in highly available environments.

Expanded software-defined networking (SDN)

This release significantly improves its SDN stack to support modern network architectures.

- **New Fabric Protocols:**Native support for WireGuard and BGP has been integrated into the SDN stack.
- **BGP/EVPN filtering:**Support for route maps and prefix lists allows for fine-grained control over route redistribution.

Further additions include route redistribution for OSPF fabrics, additional options for configuring EVPN controllers, and IPv6 underlay support for EVPN.

Custom CPU model management

To provide greater flexibility for specialized workloads, Proxmox VE 9.2 introduces a dedicated management interface for custom CPU models. Administrators can now create, edit, and remove custom CPU profiles directly in the web interface under the “Datacenter” section. This makes it easier to tailor the virtual CPU features exposed to VMs, ensuring optimal workload performance. Additionally, the integrated CPU flags selector provides instant visibility into supported flags across all cluster nodes, helping administrators identify potential cluster-wide compatibility issues during the configuration phase.

Confident maintenance with HA Arm/Disarm

Addressing common administrative challenges during maintenance windows, Proxmox VE 9.2 introduces the ability to "disarm" and "arm" the HA Manager cluster-wide. Administrators can temporarily suspend the HA stack during planned cluster maintenance to prevent unwanted actions, such as fencing nodes. HA resource states are preserved during these disarm and arm cycles, ensuring HA resources return to their previous state and node placement automatically once maintenance is completed.

Updated technology stack

Proxmox Virtual Environment 9.2 is based on Debian 13.5 "Trixie" and features Linux kernel 7.0 as the new stable default. Along with the latest versions of QEMU 11.0, LXC 7.0, and ZFS 2.4, this release offers a high-performance open-source architecture for modern infrastructure.

As a complete data center ecosystem engineered for high-density virtualization and disaster recovery, version 9.2 provides businesses with a seamless management environment for compute, storage, and backup. This includes updated support for the storage layer, with Ceph Tentacle 20.2. now available as a stable option alongside Ceph Squid 19.2.

### Availability

Proxmox Virtual Environment 9.2 is open-source software and immediately available for download at the official website. Users can obtain a complete installation image via ISO download, which contains the full feature set of the solution and can be installed quickly on bare-metal systems using an intuitive installation wizard.

Seamless distribution upgrades from older versions of Proxmox Virtual Environment are possible using the standard APT package management system. Furthermore, it is also possible to install Proxmox Virtual Environment on top of an existing Debian installation.

For enterprise environments, Proxmox offers comprehensive support plans that provide direct access to expert support services and stable and secure updates. These support contracts offer a cost-effective way to secure enterprise-grade stability, with pricing starting at EUR 120 per year and CPU.

Resources:

- ISO Image Download:[https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/downloads)
- Forum Announcement:[https://forum.proxmox.com/](https://forum.proxmox.com/)
- Video tutorial:[What’s new in Proxmox VE 9.2](https://www.proxmox.com/en/services/training-courses/videos/proxmox-virtual-environment/whats-new-in-proxmox-ve-9-2)
- Roadmap: For published and upcoming features, see the[Release Notes & Roadmap](https://pve.proxmox.com/wiki/Roadmap#Proxmox_VE_9.2)

[](https://www.proxmox.com)###

**About Proxmox Virtual Environment**
Powering over 2 million hosts globally, Proxmox Virtual Environment is a complete open-source platform for enterprise virtualization and hyper-converged infrastructure. It natively unifies KVM virtualization, LXC containers, software-defined storage, and networking on a single platform. Alongside its dedicated Backup Server and Datacenter Manager, the Proxmox ecosystem eliminates multi-site complexity as well as dependency on proprietary stacks. Backed by a global community of over 225,000 members, the platform serves as a scalable, cost-effective foundation for modern data centers.

**About Proxmox Server Solutions**
Proxmox Server Solutions provides powerful, intuitive open-source server software that guarantees vendor independence and minimizes total cost of ownership. Enterprises of all sizes rely on the company’s reliable vendor support, certified training services, and a global network of 3,000 integration partners to ensure business continuity. Established in 2005 and headquartered in Vienna, Austria, tens of thousands of corporate customers worldwide trust Proxmox solutions to secure their mission-critical IT environments. To learn more visit[https://www.proxmox.com](https://www.proxmox.com)or follow us on[LinkedIn](https://www.linkedin.com/company/proxmox)and[YouTube.](https://www.youtube.com/user/ProxmoxVE)

Contact:Daniela Häsler, Proxmox Server Solutions GmbH,[marketing@proxmox.com](mailto:marketing@proxmox.com)

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-9-2
