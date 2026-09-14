---
title: "Proxmox Mail Gateway 7.1 available with Multi-Factor Authentication via GUI"
date: 2021-11-26T13:47:18Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-7-1"
summary: "Компания Proxmox выпустила версию Proxmox Mail Gateway 7.1 — открытое решение для защиты электронной почты, работающее на базе Debian Bullseye 11.1, Linux 5.13 и OpenZFS 2.1. Система размещается между файрволом и внутренним почтовым сервером и фильтрует входящий и исходящий трафик, защищая от спама, вирусов, троянов и фишинга. В новой версии обновлены Apache SpamAssassin 3.4.6 и PostgreSQL 13.5, улучшен API, добавлена настройка IP через DHCP и расширена поддержка delivery status notifications. Веб-интерфейс стал удобнее: улучшена настройка LDAP, отредактированы APT-репозитории для администраторов, обновлены переводы, а для GUI добавлена многофакторная аутентификация с WebAuthn, TOTP и recovery keys. Proxmox Mail Gateway распространяется по лицензии GNU AGPL v3, устанавливается из ISO на bare-metal или VM, а также может развертываться как контейнер в Proxmox Virtual Environment. Для предприятий Proxmox предлагает подписку на поддержку и Enterprise Repository с регулярными обновлениями, начиная с 139 евро за хост в год."
---

# Proxmox Mail Gateway 7.1 available with Multi-Factor Authentication via GUI

## Краткое содержание

Компания Proxmox выпустила версию Proxmox Mail Gateway 7.1 — открытое решение для защиты электронной почты, работающее на базе Debian Bullseye 11.1, Linux 5.13 и OpenZFS 2.1.
Система размещается между файрволом и внутренним почтовым сервером и фильтрует входящий и исходящий трафик, защищая от спама, вирусов, троянов и фишинга.
В новой версии обновлены Apache SpamAssassin 3.4.6 и PostgreSQL 13.5, улучшен API, добавлена настройка IP через DHCP и расширена поддержка delivery status notifications.
Веб-интерфейс стал удобнее: улучшена настройка LDAP, отредактированы APT-репозитории для администраторов, обновлены переводы, а для GUI добавлена многофакторная аутентификация с WebAuthn, TOTP и recovery keys.
Proxmox Mail Gateway распространяется по лицензии GNU AGPL v3, устанавливается из ISO на bare-metal или VM, а также может развертываться как контейнер в Proxmox Virtual Environment.
Для предприятий Proxmox предлагает подписку на поддержку и Enterprise Repository с регулярными обновлениями, начиная с 139 евро за хост в год.

## Полная статья

**VIENNA, Austria – November 30, 2021 –**Enterprise software developer Proxmox Server Solutions GmbH ("Proxmox" or the "Company") has today released Proxmox Mail Gateway 7.1, the latest version of its open-source email security solution. The Mail Gateway is a complete operating system based on Debian Bullseye 11.1, but using a newer Linux kernel 5.13, and OpenZFS 2.1.

The anti-spam and anti-virus filtering solution, available since 2005, functions as a full-featured mail proxy, that is deployed between the firewall and the internal mail server. It protects organizations against threats, such as spam, viruses, Trojans, and phishing emails. As one of the few open-source enterprise email filtering solutions available, Proxmox Mail Gateway continually provides updates to current versions of its underlying Linux distribution, helping companies to stay safe and up-to-date in a security sensitive domain.

### What’s new in Version 7.1

Proxmox ships the latest upstream release of Apache SpamAssassin 3.4.6 with an updated rule-set for its Mail Gateway. PostgreSQL 13.5 is also included.

- This release brings enhancements to the API. Support for IP configuration via DHCP has been added. While email still needs working DNS records, users can now manage and configure the IP for Proxmox Mail Gateway in the DHCP configuration. When adding a new entry to a ‘Who’ object, a duplicate check is performed before saving. Proxmox Mail Gateway uses the first search domain from /etc/resolv.conf as domain name - Version 7.1 can now also handle entries in domain names that have a trailing dot. Finally, support for delivery status notifications (DSN, RFC 3461) has been made available in before-queue filtering mode and improved for after-queue filtering.
- Improved web interface: The configuration of LDAP backends via GUI has been improved. Changes can now be applied without specifying a password. Furthermore, the APT repository configuration, which was restricted to 'root', is now visible and editable by all users with 'Administrator' privileges. Some translations have been updated including Arabic, Basque, Brazilian Portuguese, French, German, Simplified Chinese, Traditional Chinese, and Turkish.
- Multi-Factor Authentication for the GUI: Multiple second factors can now be configured for a single account. Web Authentication (WebAuthn), the general standard for authentication, one-time recovery keys and Time-based OATH (TOTP) are all available as second factors.

### Availability

Proxmox Mail Gateway is released under the GNU Affero GPL, v3. The downloadable ISO image can be installed on bare-metal or a VM. The ISO contains an installation wizard that automatically installs and configures all the necessary components on the host. This eliminates the need for manual configuration later. The ISO contains the complete feature-set and can be downloaded at[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads)

Additionally, you can install Proxmox Mail Gateway as a container appliance inside Proxmox Virtual Environment.

For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, as well as technical support. Subscription prices start at EUR 139 per host, per year, for unlimited users and domains. The Enterprise Repository provides regular updates via the web interface, and is recommended for production use.

**Forum Announcement**
[https://forum.proxmox.com/threads/proxmox-mail-gateway-7-1-available.100637](https://forum.proxmox.com/threads/proxmox-mail-gateway-7-1-available.100637/)

**About Proxmox Mail Gateway**
Proxmox Mail Gateway is the leading open-source email security solution, protecting your mail server against all email threats from the moment they emerge. Organizations of any size can easily deploy and implement the comprehensive anti-spam and anti-virus platform in just a few minutes. Deploying the full featured mail proxy between the firewall and an internal mail server allows you to control all incoming and outgoing email traffic from the central, web-based interface. Proxmox filters all email traffic at the gateway before it reaches the mail server, protecting businesses against email attacks and other malicious threats. Proxmox Mail Gateway is open-source software, licensed under the GNU AGPL, v3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training services to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-7-1
