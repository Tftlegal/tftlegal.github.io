---
title: "Proxmox Mail Gateway 9.1 released"
date: 2026-06-11T07:00:00Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-9-1"
summary: "Proxmox Server Solutions 11 июня 2026 года выпустила обновление Proxmox Mail Gateway 9.1 — решение для защиты корпоративной почты от спама, вирусов и фишинга. Платформа обновлена до Debian 13.5 и ядра Linux 7.0, а также включает актуальные версии SpamAssassin, ClamAV, PostgreSQL и ZFS. В разделе карантинного хранения спама улучшено удобство: появились отметки о просмотре для общих ящиков, детализация спама-скоринга, загрузка внешних изображений по запросу и копирование приватной ссылки получателя. Добавлена поддержка шифрования резервных копий при отправке на Proxmox Backup Server, включая чувствительные настройки и статистические данные. Программа распространяется как open-source и доступна для установки через ISO, поверх Debian или в виде LXC-контейнера на Proxmox VE. Для апгрейда с версий 8.2 или 9.0 предусмотрен штатный путь через APT, а для предприятий предлагаются планы поддержки от 190 евро за хост в год."
---

# Proxmox Mail Gateway 9.1 released

## Краткое содержание

Proxmox Server Solutions 11 июня 2026 года выпустила обновление Proxmox Mail Gateway 9.1 — решение для защиты корпоративной почты от спама, вирусов и фишинга.
Платформа обновлена до Debian 13.5 и ядра Linux 7.0, а также включает актуальные версии SpamAssassin, ClamAV, PostgreSQL и ZFS.
В разделе карантинного хранения спама улучшено удобство: появились отметки о просмотре для общих ящиков, детализация спама-скоринга, загрузка внешних изображений по запросу и копирование приватной ссылки получателя.
Добавлена поддержка шифрования резервных копий при отправке на Proxmox Backup Server, включая чувствительные настройки и статистические данные.
Программа распространяется как open-source и доступна для установки через ISO, поверх Debian или в виде LXC-контейнера на Proxmox VE.
Для апгрейда с версий 8.2 или 9.0 предусмотрен штатный путь через APT, а для предприятий предлагаются планы поддержки от 190 евро за хост в год.

## Полная статья

**VIENNA, Austria – June 11, 2026 –**Enterprise software developer Proxmox Server Solutions today announced the release of Proxmox Mail Gateway 9.1. The updated version of its enterprise email security solution introduces updated core components, comprehensive usability improvements to the spam quarantine, and data encryption options for integrated backups.

Proxmox Mail Gateway functions as a full-featured mail proxy deployed between the firewall and internal mail servers. It filters all incoming and outgoing email traffic at the gateway, protecting organizations against threats such as spam, viruses, Trojans, and phishing attacks.

## Key Updates in Proxmox Mail Gateway 9.1

Updated core components

Built on Debian 13.5 “Trixie”, the platform includes updated underlying packages, utilizing a newer Linux kernel 7.0 as its stable default. Proxmox Mail Gateway 9.1 continues to align with the latest major enterprise open-source security components and incorporates stable versions of SpamAssassin 4.0.2 (with continuously updated rulesets), ClamAV 1.4.4, PostgreSQL 17, and ZFS 2.4.

Spam quarantine usability improvements

The web-based quarantine interface features several enhancements to optimize daily administrative and end-user workflows.

- Shared mailboxes: Users can now mark quarantined emails within shared mailboxes as “seen”, preventing duplicate auditing efforts across teams. The status is displayed inline as a checkmark and can be toggled via an action button.
- Granular spam scores: The quarantine overview now displays both the positive and negative components of the spam score simultaneously, providing immediate insight into why an email triggered filtering thresholds.
- On-demand image loading: To enhance privacy and security, external images in quarantined emails can now be configured to load only on demand. Users can then choose to display images by clicking a “Load Images” button in the quarantine view. This ensures email content can be inspected safely without automatically compromising privacy or being exposed to web-based threats.
- Copy Link Functionality: Administrators can now copy a recipient’s private quarantine access link directly from the admin dashboard using a new “Copy Link” option. This provides a secure and convenient way to share the link through any preferred channel or to integrate it in a custom interface.

Encrypted Proxmox Backup Server targets

Version 9.1 adds native encryption support for backups targeted at a Proxmox Backup Server instance. This option ensures that sensitive email configuration settings, user created rule system data, and historic/private statistics data are encrypted client-side before transmission and remain encrypted at rest on the backup storage target.

### Availability

Proxmox Mail Gateway 9.1 is open-source software and immediately available for download. Users can obtain a complete installation image via ISO download, which contains the full feature-set of the solution and can be installed quickly on bare-metal systems using an intuitive installation wizard. The software can be installed on top of an existing Debian installation or as a lightweight Linux Container (LXC) on Proxmox VE. A seamless, fully tested upgrade path from Proxmox Mail Gateway 8.2 or 9.0 is available via the APT package management system.

For production environments, Proxmox offers comprehensive enterprise support plans that provide stable and secure updates and direct access to expert support services. These support contracts offer a cost-effective way to secure enterprise-grade stability. Pricing start at EUR 190 per host per year, including unlimited users and domains.

Resources:

- ISO Image Download:[https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/downloads)
- Forum Announcement:[https://forum.proxmox.com/](https://forum.proxmox.com/threads/proxmox-mail-gateway-9-1-released.184240/)
- Roadmap: For published and upcoming features, see the[Release Notes & Roadmap](https://pmg.proxmox.com/wiki/Roadmap)

[](https://www.proxmox.com)###

**About Proxmox Mail Gateway**
Proxmox Mail Gateway is the leading open-source email security solution, protecting your mail server against all email threats from the moment they emerge. Organizations of any size can easily deploy and implement the comprehensive anti-spam and antivirus platform in just a few minutes. Deploying the full-featured mail proxy between the firewall and an internal mail server allows you to control all incoming and outgoing email traffic from the central, web-based interface. Proxmox filters all email traffic at the gateway before it reaches the mail server, protecting businesses against email attacks and other malicious threats.

**About Proxmox Server Solutions**
Proxmox Server Solutions provides powerful, intuitive open-source server software that guarantees vendor independence and minimizes total cost of ownership. Enterprises of all sizes rely on the company’s reliable vendor support, certified training services, and a global network of 3,000 integration partners to ensure business continuity. Established in 2005 and headquartered in Vienna, Austria, tens of thousands of corporate customers worldwide trust Proxmox solutions to secure their mission-critical IT environments.

Contact:Daniela Häsler, Proxmox Server Solutions GmbH,[press@proxmox.com](mailto:press@proxmox.com)

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-9-1
