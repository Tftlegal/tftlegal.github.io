---
title: "Proxmox Mail Gateway 8.1 with enhanced Rule System"
date: 2024-02-27T15:35:18Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-8-1"
summary: "Proxmox Mail Gateway 8.1 — новая версия открытого решения для защиты почты, выпущенная 29 февраля 2024 года. Платформа работает как полноценный почтовый прокси на базе Debian 12.5 с ядром Linux 6.5 и защищает от спама, вирусов, троянов и фишинга. В версии улучшена система правил, добавлена гибкая DKIM-подпись и поддержка Secure Boot. Веб-интерфейс получил фильтры и поиск для управления крупными разворачиваниями, а также новые и обновленные переводы. Дополнительно устранены проблемы с SMTP smuggling, резервным копированием и логированием, а ядра переименованы в серии proxmox-kernel. Продукт доступен в виде ISO для установки на физический сервер или в виртуальную машину, а также как контейнер, и для бизнеса есть платная поддержка Proxmox."
---

# Proxmox Mail Gateway 8.1 with enhanced Rule System

## Краткое содержание

Proxmox Mail Gateway 8.1 — новая версия открытого решения для защиты почты, выпущенная 29 февраля 2024 года.
Платформа работает как полноценный почтовый прокси на базе Debian 12.5 с ядром Linux 6.5 и защищает от спама, вирусов, троянов и фишинга.
В версии улучшена система правил, добавлена гибкая DKIM-подпись и поддержка Secure Boot.
Веб-интерфейс получил фильтры и поиск для управления крупными разворачиваниями, а также новые и обновленные переводы.
Дополнительно устранены проблемы с SMTP smuggling, резервным копированием и логированием, а ядра переименованы в серии proxmox-kernel.
Продукт доступен в виде ISO для установки на физический сервер или в виртуальную машину, а также как контейнер, и для бизнеса есть платная поддержка Proxmox.

## Полная статья

**VIENNA, Austria – February 29, 2024 –**Enterprise software developer Proxmox Server Solutions GmbH ("Proxmox" or the "Company") has today released Proxmox Mail Gateway 8.1, the latest version of its open-source email security solution. The Mail Gateway platform is a complete operating system based on Debian 12.5, but defaulting to a newer Linux kernel 6.5, and including ZFS 2.2.2 and PostgreSQL 15.6.

The anti-spam and anti-virus filtering solution functions as a full featured mail proxy, that is deployed between the firewall and the internal mail server. It protects organizations against threats, such as spam, viruses, Trojans, and phishing emails.

## What's new in Proxmox Mail Gateway 8.1

- **Enhanced rule system:**Proxmox Mail Gateway is known for its comprehensive object-oriented rule system, where the mail flow can be controlled and filtered. Improvements in this version now allow the selection of a “match-if” mode for entries in What/Who/When objects, and multiple objects in rules, providing flexible control over whether All, Any, Some but not all, or None must match.
- **DKIM signing:**Optionally, DKIM signing can now select the signing domain based on header information instead of the envelope information. This allows for the configuration of more stringent DMARC policies in certain environments.
- **Support for secure boot:**Like the Proxmox virtualization and backup solutions, this version 8.1 is now compatible with Secure Boot. With this security feature, the boot process of a computer can be protected by ensuring that only software with a valid digital signature launches on a machine. Proxmox Mail Gateway now includes a signed shim bootloader trusted by most hardware's UEFI implementations. This allows installing it in environments where Secure Boot is required and enabled. The documentation explains how an existing Proxmox Mail Gateway installation can be switched over to Secure Boot without reinstallation.
- **Management via web interface:**A new filter- and search box added to the web interface simplifies the management of large deployments. Filtering is now possible in Relay Domains, Transport, Networks, and for Objects in the rule system. Additionally, Croatian and Georgian translations have been added, and translations for other languages have been improved.

### Further notable enhancements

- Proxmox ships its own heavily tweaked kernel which all products share. For the Mail Gateway, this is now also reflected in the renaming from pve-kernel and pve-headers to proxmox-kernel and proxmox-headers respectively in all relevant packages. The new proxmox-default-kernel and proxmox-default-headers meta-packages will depend on the currently recommended kernel-series.
- SMTP smuggling mitigation: With a prompt response to the spoofing attack at SMTP protocol level at the end of 2023, the Postfix configuration was secured as quickly as possible against SMTP smuggling in line with the latest recommendations from upstream. Proxmox Mail Gateway users who have not yet updated are encouraged to do so and verify that they have not missed the template change.
- Fixed: Restoring a Mail Gateway backup from a cluster with statistics, does not interfere with creating a fresh cluster on the restored node.
- The Log Tracker tool, which powers the Tracking Center, now uses UTC by default for internal time calculations, eliminating problems with time zone changes such as switching from Standard Time to Daylight Saving Time and vice versa.

### Availability

Proxmox Mail Gateway is released under the GNU Affero GPL, v3 and is available as an ISO download, which can be installed on bare metal or on a virtual machine. The ISO contains an installation wizard that automatically installs and configures all the necessary components on the host. This eliminates the need for manual configuration later. The ISO contains the complete feature-set and can be downloaded at[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads)
Additionally, you can install Proxmox Mail Gateway as a container appliance inside Proxmox VE.

For enterprise use, Proxmox Server Solutions GmbH offers a subscription service that provides access at all levels to the extensively tested, stable Enterprise Repository. This repository provides regular updates via the web interface and is recommended for production use. At the Basic and higher subscription levels, technical support from the Proxmox team is also available. Subscription pricing starts at EUR 175 per host per year for unlimited users and domains.

##### Further information:

- Find all details in the[Release Notes](https://pmg.proxmox.com/wiki/index.php/Roadmap)
- Read the[Forum Announcement](https://forum.proxmox.com/threads/proxmox-mail-gateway-8-1-available.142510/)

###

**About Proxmox Mail Gateway**
Proxmox Mail Gateway is the leading open-source email security solution, protecting your mail server against all email threats from the moment they emerge. Organizations of any size can easily deploy and implement the comprehensive anti-spam and anti-virus platform in just a few minutes. Deploying the full featured mail proxy between the firewall and an internal mail server allows you to control all incoming and outgoing email traffic from the central, web-based interface. Proxmox filters all email traffic at the gateway before it reaches the mail server, protecting businesses against email attacks and other malicious threats. Proxmox Mail Gateway is open-source software, licensed under the AGPLv3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support, training services, and an extensive partner ecosystem to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**Media contact:**Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-8-1
