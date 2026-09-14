---
title: "Proxmox Virtual Environment 8.0 with Debian 12 \"Bookworm\" released"
date: 2023-07-10T12:56:02Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-8-0"
summary: "Proxmox представила стабильную версию 8.0 своей платформы виртуализации Proxmox Virtual Environment. Новая версия построена на Debian 12, использует Linux-ядро 6.2 и включает обновлённые QEMU, LXC, ZFS и Ceph Quincy. Для пользователей старых версий предусмотрен протестированный путь обновления до 8.0. Ключевые новинки — стабильный Ceph Enterprise репозиторий, автоматическая синхронизация LDAP/AD, более гибкие права доступа к сетевым ресурсам и устройствам PCI/USB, а также усиленная защита 2FA. Платформа остаётся бесплатным open-source продуктом, управляемым через веб-интерфейс или командную строку. Для бизнеса Proxmox предлагает подписочную поддержку и Enterprise-репозиторий с ценами от 105 евро в год за CPU."
---

# Proxmox Virtual Environment 8.0 with Debian 12 "Bookworm" released

## Краткое содержание

Proxmox представила стабильную версию 8.0 своей платформы виртуализации Proxmox Virtual Environment.
Новая версия построена на Debian 12, использует Linux-ядро 6.2 и включает обновлённые QEMU, LXC, ZFS и Ceph Quincy.
Для пользователей старых версий предусмотрен протестированный путь обновления до 8.0.
Ключевые новинки — стабильный Ceph Enterprise репозиторий, автоматическая синхронизация LDAP/AD, более гибкие права доступа к сетевым ресурсам и устройствам PCI/USB, а также усиленная защита 2FA.
Платформа остаётся бесплатным open-source продуктом, управляемым через веб-интерфейс или командную строку.
Для бизнеса Proxmox предлагает подписочную поддержку и Enterprise-репозиторий с ценами от 105 евро в год за CPU.

## Полная статья

**VIENNA, Austria – June 22, 2023**– Enterprise software developer Proxmox Server Solutions GmbH (henceforth "Proxmox") today released the stable version 8.0 of its server virtualization management platform Proxmox Virtual Environment. This major release is based on the latest Debian 12 (“Bookworm), and comes with an extensively tested and detailed upgrade path for users of Proxmox VE 7.4 or older versions to enable a smooth upgrade. Proxmox VE 8.0 uses a newer Linux kernel 6.2 as stable default, and includes updates to the latest versions of leading open-source technologies for virtual environments like QEMU 8.0.2, LXC 5.0.2, ZFS 2.1.12, and Ceph Quincy 17.2.6.

The virtualization platform from Proxmox comes stocked with all the essential management tools and an easy-to-use, web-based user interface. This allows to conveniently manage individual hosts or the entire data center with out-of-the-box tools - either via a web browser or the command line.

## Further highlights in Proxmox Virtual Environment 8.0

- **New Ceph Enterprise repository:**Proxmox Virtual Environment fully integrates Ceph Quincy, allowing to run and manage Ceph storage directly from any of the cluster nodes and to easily setup and manage a hyper-converged infrastructure. The Ceph source code is packaged by the Proxmox development team and—after extensive tests—delivered in the stable Enterprise repository. This unifies the delivery of Ceph with other components of Proxmox VE. With version 8.0, all Proxmox customers with an active subscription can now access the stable Ceph Enterprise repository recommended for production environments.
- **Authentication realm sync jobs:**The synchronization of users and groups for LDAP-based realms (LDAP & Microsoft Active Directory), can now be configured to**run automatically at regular intervals**. This simplifies management, and removes a source for configuration errors and omissions compared to synchronizing the realm manually.
- **Network resources**defined for Software-defined Networking (SDN) are now also**available as objects in the access control subsystem (ACL)**of Proxmox VE. It is possible to grant fine-grained permissions for host network bridges and VNets to specific users and groups.
- **Resource mappings:**Mappings between resources, such as**PCI(e) or USB devices**, and nodes in a Proxmox VE cluster, can now be created and managed in the API and the web interface. VM guests can get such an abstract resource assigned, which can be matched with concrete resources on each node. This enables offline migrations for VMs with passed-through devices. The mappings are also represented in the ACL system of Proxmox VE, allowing a user to be granted access to one or more specific devices, without requiring root access. In case a conflicting entry is detected, e.g. due to address changes or overlaps, users are informed on VM start.
- **Secure lockout for Two-factor authentication/TOTP:**To further improve security, user accounts with too many login attempts - failing the second factor authentication - are locked out. This protects against attacks where the user password is obtained and a brute-force guess is attempted on the second factor. If TFA fails too many times in a row, the user account is locked out for one hour. If TOTP failed too many times in a row, TOTP is disabled for the user account. The user account can be unlocked again with a recovery key, or manually by an administrator.
- **Text-based user interface (TUI)**for the installer ISO: A text-based user interface has been added and can now be used optionally to gather all required information. This eliminates issues when launching the GTK-based graphical installer that sometimes occur on very new as well as rather old hardware.
- The**x86-64-v2-AES**model is the**new default CPU type for VMs**created via the web interface. It provides important extra features over the qemu64/kvm64, and improves performance of many computing operations.

### Availability

Proxmox Virtual Environment is free and open-source software, published under the GNU Affero General Public License, v3. The ISO contains the complete feature-set and can be installed on bare-metal. Proxmox VE 8.0 is available for download at[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads)

Seamless[upgrade instructions from Proxmox VE 7.x to 8.x](https://pve.proxmox.com/wiki/Upgrade_from_7_to_8)are available.
It’s also possible to install Proxmox VE 8 on top of Debian.

For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 105 per year and CPU.

###

**About Proxmox Server Solutions**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training services to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**Facts Proxmox VE**
The open-source project Proxmox VE has a huge worldwide user base with more than 800,000 hosts. The virtualization platform has been translated into over 26 languages. More than 110,000 active community members in the support forum engage with and help each other. By using Proxmox VE as an alternative to proprietary virtualization management solutions, enterprises are able to centralize and modernize their IT infrastructure, and turn it into a cost-effective and flexible software-defined data center, based on the latest open-source technologies. Tens of thousands of customers rely on a enterprise support subscription from Proxmox Server Solutions GmbH.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-8-0
