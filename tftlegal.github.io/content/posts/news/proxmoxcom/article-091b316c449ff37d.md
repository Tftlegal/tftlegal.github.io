---
title: "Proxmox Mail Gateway 7.0 released"
date: 2021-07-14T13:10:59Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-7-0"
summary: "15 июля 2021 года компания Proxmox Server Solutions выпустила новую основную версию 7.0 Proxmox Mail Gateway — открытого решения для защиты электронной почты. Система построена на Debian 11 «Bullseye» с ядром Linux 5.11 и включает обновлённые Apache SpamAssassin 3.4.6 и PostgreSQL 13. Proxmox Mail Gateway работает как полнофункциональный почтовый прокси между межсетевым экраном и внутренним почтовым сервером, защищая организации от спама, вирусов, троянов и фишинга. В версии 7.0 улучшен веб-интерфейс: добавлены более подробная панель состояния, управление APT-репозиториями через GUI, поддержка wildcard-доменов в ACME/Let's Encrypt и ограничения API по IP. Установщик стал удобнее благодаря улучшенному определению ISO и автоматической адаптации интерфейса под HiDPI-экраны. Решение распространяется по лицензии GNU AGPL v3, доступно как ISO для установки на bare metal или в виртуальной машине, а также как контейнерный appliance в Proxmox VE. Для корпоративных пользователей Proxmox предлагает подписку на поддержку и Enterprise Repository по цене от 139 евро за хост в год."
---

# Proxmox Mail Gateway 7.0 released

## Краткое содержание

15 июля 2021 года компания Proxmox Server Solutions выпустила новую основную версию 7.0 Proxmox Mail Gateway — открытого решения для защиты электронной почты. Система построена на Debian 11 «Bullseye» с ядром Linux 5.11 и включает обновлённые Apache SpamAssassin 3.4.6 и PostgreSQL 13. Proxmox Mail Gateway работает как полнофункциональный почтовый прокси между межсетевым экраном и внутренним почтовым сервером, защищая организации от спама, вирусов, троянов и фишинга. В версии 7.0 улучшен веб-интерфейс: добавлены более подробная панель состояния, управление APT-репозиториями через GUI, поддержка wildcard-доменов в ACME/Let's Encrypt и ограничения API по IP. Установщик стал удобнее благодаря улучшенному определению ISO и автоматической адаптации интерфейса под HiDPI-экраны. Решение распространяется по лицензии GNU AGPL v3, доступно как ISO для установки на bare metal или в виртуальной машине, а также как контейнерный appliance в Proxmox VE. Для корпоративных пользователей Proxmox предлагает подписку на поддержку и Enterprise Repository по цене от 139 евро за хост в год.

## Полная статья

**VIENNA, Austria – July 15, 2021 –**Enterprise software developer Proxmox Server Solutions GmbH (henceforth "Proxmox") today released the next major version 7.0 of Proxmox Mail Gateway, its open-source email security solution. The Mail Gateway is a complete operating system based on Debian 11 “Bullseye”, but using the newer Linux kernel 5.11. Proxmox ships the latest upstream release of Apache SpamAssassin 3.4.6, with an updated rule set; PostgreSQL 13 is also included.

The anti-spam and anti-virus filtering solution, available since 2005, functions as a full-featured mail proxy, that is deployed between the firewall and the internal mail server. It protects organizations against threats, such as spam, viruses, Trojans, and phishing emails. As one of the few open-source enterprise email filtering solutions available, Proxmox Mail Gateway continually provides updates to current versions of its underlying Linux distribution, helping companies to stay safe and up-to-date in a security sensitive domain.

### What’s new in Proxmox Mail Gateway 7.0

- This release brings improvements to the web-based user interface, including a more detailed dashboard status panel, that provides a faster system overview to the administrators. It is now possible to manage the APT repositories via the GUI with a new panel in the ‘Administration’ tab. The new ’Repositories’ panel shows an in-depth status report, as well as a list of all configured repositories, and allows users to enable or disable a repository
- The ACME/Let's encrypt now supports using wildcard domains with the DNS plugins.
- API: For improved security, the API proxy daemon can now be restricted to listening on one particular IP through the LISTEN_IP parameter. This enables secure configurations, for example, making it accessible on localhost and accessing it through a VPN only, or preventing it from listening on IPv6, by simply configuring the IPv4 wildcard as LISTEN_IP.
- Improved Proxmox Installer: The Proxmox installer environment now has improved ISO detection, and can automatically detect HiDPI screens and increase console font and GUI scaling accordingly.

### Availability

Proxmox Mail Gateway is released under the GNU Affero GPL, v3 and is available as an ISO, which can be installed on bare metal, or a VM. The ISO contains an installation wizard that automatically installs and configures all the necessary components on the host. This eliminates the need for manual configuration later. The ISO contains the complete feature-set and can be downloaded at[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads)

**Upgrade from Proxmox Mail Gateway 6.x to 7.0:**A detailed step-by-step upgrade guide is available at[https://pmg.proxmox.com/wiki/index.php/Upgrade_from_6.x_to_7.0](https://pmg.proxmox.com/wiki/index.php/Upgrade_from_6.x_to_7.0.)

Additionally, you can install Proxmox Mail Gateway as a container appliance inside Proxmox VE.

For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, as well as technical support. Subscription prices start at EUR 139 per host, per year, for unlimited users and domains. The Enterprise Repository provides regular updates via the web interface, and is recommended for production use.

**About Proxmox Mail Gateway**
Proxmox Mail Gateway is the leading open-source email security solution, protecting your mail server against all email threats from the moment they emerge. Organizations of any size can easily deploy and implement the comprehensive anti-spam and anti-virus platform in just a few minutes. Deploying the full-featured mail proxy between the firewall and an internal mail server allows you to control all incoming and outgoing email traffic from the central, web-based interface. Proxmox Mail Gateway filters all email traffic at the gateway, before it reaches the mail server, protecting businesses against email attacks and other malicious threats. Proxmox Mail Gateway is open-source software, licensed under the GNU AGPL, v3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions**
Proxmox is a provider of powerful, yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training services to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-7-0
