---
title: "Proxmox Mail Gateway 6.2 released"
date: 2020-04-21T10:17:11Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-6-2"
summary: "Proxmox Server Solutions выпустила Proxmox Mail Gateway 6.2 — открытое решение для защиты электронной почты от спама, вирусов, троянов и фишинга. Система работает как почтовый прокси между файрволом и внутренним почтовым сервером и построена на Debian 10.3 с ядром Linux 5.4. Ключевые нововведения включают переписанный на Rust Message Tracking Center, удобное включение before-queue фильтрации через веб-интерфейс и гибкую настройку netmask для greylist matching. Также улучшена работа с шаблонами через ucf, интерфейс Spam Quarantine, поддержка нескольких DKIM-селекторов и downstream LMTP-серверов. Proxmox Mail Gateway 6.2 распространяется под лицензией GNU AGPLv3, доступна как ISO для установки, а платная enterprise-поддержка начинается от 119 евро за хост в год."
---

# Proxmox Mail Gateway 6.2 released

## Краткое содержание

Proxmox Server Solutions выпустила Proxmox Mail Gateway 6.2 — открытое решение для защиты электронной почты от спама, вирусов, троянов и фишинга. Система работает как почтовый прокси между файрволом и внутренним почтовым сервером и построена на Debian 10.3 с ядром Linux 5.4. Ключевые нововведения включают переписанный на Rust Message Tracking Center, удобное включение before-queue фильтрации через веб-интерфейс и гибкую настройку netmask для greylist matching. Также улучшена работа с шаблонами через ucf, интерфейс Spam Quarantine, поддержка нескольких DKIM-селекторов и downstream LMTP-серверов. Proxmox Mail Gateway 6.2 распространяется под лицензией GNU AGPLv3, доступна как ISO для установки, а платная enterprise-поддержка начинается от 119 евро за хост в год.

## Полная статья

**VIENNA, Austria – April 28, 2020 –**Proxmox Server Solutions GmbH has released Proxmox Mail Gateway 6.2, the latest version of its open-source email security solution. The Proxmox Mail Gateway, which also celebrates its 15th anniversary this April, is a complete operating system based on the latest stable release of Debian 10.3 (Buster) with a 5.4 Linux kernel. The Proxmox anti-spam and anti-virus filtering solution is a full featured mail proxy deployed between the firewall and the internal mail server. It protects organizations against spam, viruses, Trojans, and phishing emails. Proxmox ships the latest upstream release of Apache SpamAssassin 3.4.4 with an updated and enhanced ruleset (KAM rules added).

### Main new features of Proxmox Mail Gateway 6.2

**Rust for the Proxmox Message Tracking Center:**The pmg-log-tracker, the binary at the core of the Proxmox Message Tracking Center, has been extended and re-implemented in the Rust programming language. pmg-log-tracker is responsible for providing the live searchable and grouped logs displayed in the web-based user interface, where the administrator can overview and control the mail flow from a single screen. The new pmg-log-tracker has support for parsing and grouping logs in before-queue filtering mode. Additionally, the new Rust code base provides optimized performance and more stability.

**Before-Queue filtering mode via the GUI:**With Proxmox Mail Gateway 6.2 the before-queue filtering can now be comfortably enabled with a simple click in the web interface and the logs can be displayed. Instead of accepting and silently discarding unwanted email, the Mail Gateway can optionally reject an email during the SMTP dialogue. By answering with a permanent failure code (554) there is no need to generate a non-delivery report, as this could cause the system to get blacklisted, due to Backscatter.

**Customizable netmask length for greylist matching:**As some cloud-providers send out one email from different IP addresses within a large network this can lead to a rather long delay and sometimes even to a legitimate email being rejected, when greylisting is enabled. With Proxmox Mail Gateway 6.2, the administrator can configure which hosts are considered to be belonging to the same network by setting a larger (or smaller) suffix instead of using “/24”.

**Handling of changes to overridden templates with ucf:**All service configuration templates, copied and modified in /etc/pmg/templates, get registered with ucf. Should an overridden template change with a new package version the administrator is asked and can accept or reject the changes. All users who have their templates in /etc/pmg/templates will be asked about the current changes for the initial registration.

### Further enhancements

- Better usability for the user Spam Quarantine interface: The ‘From’ header and the ‘Subject’ are now displayed on top of the mail body.
- DKIM signing: The DKIM module can handle multiple selectors. With version 6.2 users can comfortably switch between the active selector in the GUI.
- New What object: 'Match Archive Filename'
- Support for downstream LMTP servers.

### Availability

Proxmox Mail Gateway is released under the GNU AGPLv3. It is available as a bare-metal ISO install with an installation wizard that automatically installs and configures all necessary components on the host eliminating manual installation configuration. The ISO install contains the full feature-set and can be downloaded at[https://www.proxmox.com/downloads](https://www.proxmox.com/en/component/zoo/?view=frontpage&layout=frontpage&Itemid=120)

Technical enterprise support is available from Proxmox Server Solutions. Subscription pricing starts at EUR 119 per host and year for unlimited users and provides access to the enterprise package repository with regular updates via the web interface and to support from the Proxmox team.

**About Proxmox Mail Gateway**
Proxmox Mail Gateway is the leading open-source email security solution protecting your mail server against all email threats the moment they emerge. Organizations of any size can easily implement and deploy the comprehensive anti-spam and anti-virus platform in only a few minutes. Deploying the full featured mail proxy between the firewall and an internal mail server allows to control all incoming and outgoing email traffic from the central web-based interface. Proxmox filters the whole email traffic at the gateway before it reaches the mail server protecting businesses against email attacks and other malicious threats. Proxmox Mail Gateway is open-source software, licensed under the GNU AGPLv3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions GmbH**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises regardless of size, sector or industry use the stable, secure, scalable, and open Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**Contact:**Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-6-2
