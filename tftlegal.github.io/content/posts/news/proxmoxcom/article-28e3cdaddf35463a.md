---
title: "Proxmox Mail Gateway 7.2 released"
date: 2022-11-10T16:54:17Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-7-2"
summary: "Proxmox Server Solutions выпустила версию 7.2 Proxmox Mail Gateway — открытого решения для корпоративной защиты электронной почты. Продукт работает как почтовый прокси между файрволом и внутренним почтовым сервером, фильтруя спам, вирусы, трояны и фишинговые письма. Версия 7.2 построена на Debian 11.5 и включает обновлённые компоненты, включая Linux 5.15, ZFS 2.1.6, SpamAssassin 3.4.6 и PostgreSQL 13.8. Обновление улучшает поддержку Unicode и SMTPUTF8, а также делает интерфейс карантина удобнее для администраторов и пользователей. Добавлен инструмент offline mirror, позволяющий обновлять изолированные или ограниченные системы через локальный репозиторий. Proxmox Mail Gateway распространяется под лицензией AGPL v3 и доступен как ISO-образ или контейнер, с возможностью платной корпоративной поддержки."
---

# Proxmox Mail Gateway 7.2 released

## Краткое содержание

Proxmox Server Solutions выпустила версию 7.2 Proxmox Mail Gateway — открытого решения для корпоративной защиты электронной почты. Продукт работает как почтовый прокси между файрволом и внутренним почтовым сервером, фильтруя спам, вирусы, трояны и фишинговые письма. Версия 7.2 построена на Debian 11.5 и включает обновлённые компоненты, включая Linux 5.15, ZFS 2.1.6, SpamAssassin 3.4.6 и PostgreSQL 13.8. Обновление улучшает поддержку Unicode и SMTPUTF8, а также делает интерфейс карантина удобнее для администраторов и пользователей. Добавлен инструмент offline mirror, позволяющий обновлять изолированные или ограниченные системы через локальный репозиторий. Proxmox Mail Gateway распространяется под лицензией AGPL v3 и доступен как ISO-образ или контейнер, с возможностью платной корпоративной поддержки.

## Полная статья

**VIENNA, Austria – November 30, 2022 –**Enterprise software developer Proxmox Server Solutions, the company behind Proxmox Mail Gateway, announces today point release 7.2 of its open-source email security solution. Proxmox Mail Gateway, available since 2005, is one of few enterprise-grade open-source email filtering solutions. It protects organizations against threats such as spam, viruses, Trojans, and phishing emails and functions as a fully-featured mail proxy that is deployed between the firewall and the internal mail server.

### Enhancements in Proxmox Mail Gateway 7.2

- Proxmox Mail Gateway 7.2 is a complete operating system based on Debian 11.5 (“Bullseye”), using the newer Linux kernel 5.15, as well as ZFS 2.1.6. The Proxmox developers ship the latest upstream release of Apache SpamAssassin 3.4.6 with an updated rule-set for its Mail Gateway. PostgreSQL 13.8 is also included.
- Enhancements in the rule system: With the comprehensive object-oriented rule system of Proxmox Mail Gateway, users can create customized rules for their mail environment. This version improves the handling of international emails by adding support for Unicode characters and better handles SMTPUTF8 emails.
- The Quarantine interface brings improved usability for administrators and end-users alike:

- In the administrator view, the admin can now select multiple emails, the ‘Receiver’ information is displayed, and there is a context menu in the mail-listing, for the ‘Attachment’ and ‘Virus’ quarantines.
- The SpamAssassin rules can now be visually sorted by scores in the SpamInfo grid, thus helping to accelerate the decision on which ones to select and remove. The 'Deliver' and 'Delete' actions are now colorized, improving intuitive handling of common actions.
- By additionally displaying attachments in the ‘Spam’ and ‘Virus’ quarantine, administrators get a more complete overview of the email in question.
- ‘Virus’ and ‘Attachment’ quarantine can now optionally be filtered by ‘Receiver’ – a function especially helpful in larger deployments.
- The Proxmox Offline Mirror tool allows keeping Proxmox Mail Gateway nodes – with restricted or without access to the public internet – up-to-date and running. With the ‘proxmox-offline-mirror’ utility, it is possible to manage a local apt mirror for all package updates for Proxmox and Debian projects. From that mirror, users can create an external medium (USB flash drive or a local network share) and can then update their policy-restricted or air-gapped systems. For subscribers with a Premium or Standard subscription level, Proxmox offers an offline subscription key for its product portfolio.

### Availability

Proxmox Mail Gateway is released under the GNU Affero GPL, v3 and is available as an ISO, which can be installed on bare metal, or a VM. The ISO contains an installation wizard that automatically installs and configures all the necessary components on the host. This eliminates the need for manual configuration later. The ISO contains the complete feature-set and can be downloaded[here](https://www.proxmox.com/en/component/zoo/?view=frontpage&layout=frontpage&Itemid=120)

Additionally, you can install Proxmox Mail Gateway as a container appliance inside Proxmox VE.

Proxmox Server Solutions GmbH offers enterprise support contracts on a subscription basis, which provides access to an extensively tested Enterprise Repository, regular updates via the web interface, and to technical support. Prices start at EUR 149 per year and host, and each subscription includes unlimited users and unlimited domains.

**About Proxmox Mail Gateway**
Proxmox Mail Gateway is the leading open-source email security solution, protecting your mail server against all email threats from the moment they emerge. Organizations of any size can easily deploy and implement the comprehensive anti-spam and anti-virus platform in just a few minutes. Deploying the full featured mail proxy between the firewall and an internal mail server allows you to control all incoming and outgoing email traffic from the central, web-based interface. Proxmox filters all email traffic at the gateway before it reaches the mail server, protecting businesses against email attacks and other malicious threats. Proxmox Mail Gateway is open-source software, licensed under the GNU AGPL, v3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training services to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-7-2
