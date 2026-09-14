---
title: "Proxmox Mail Gateway 7.3"
date: 2023-03-28T09:09:10Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-7-3"
summary: "Proxmox Mail Gateway 7.3 — новая версия open-source решения для защиты электронной почты, работающего как почтовый прокси между файрволом и внутренним почтовым сервером. Оно защищает организации от спама, вирусов, троянов и фишинговых писем. Сборка построена на Debian 11.6 с ядром Linux 5.15, опционально поддерживает ядро 6.2 и включает ZFS 2.1.9 и PostgreSQL 13.10. В версии интегрирован Apache SpamAssassin 4.0.0, который теперь умеет обнаруживать спам в вложениях, включая PDF, ODT, DOCX, DOC, RTF и изображения через OCR, а также лучше использует данные DMARC. Другие улучшения: принудительный TLS для входящей почты, улучшенная поддержка SMTPUTF8, тёмная тема интерфейса, загрузка журналов задач и автоматический переход HTTP на HTTPS. Решение распространяется под GNU AGPL v3, доступно как ISO для bare metal или виртуальной машины, а также как контейнерный appliance внутри Proxmox VE. Для корпоративных пользователей Proxmox предлагает подписку на поддержку и доступ к Enterprise Repository, начиная с 165 евро за хост в год."
---

# Proxmox Mail Gateway 7.3

## Краткое содержание

Proxmox Mail Gateway 7.3 — новая версия open-source решения для защиты электронной почты, работающего как почтовый прокси между файрволом и внутренним почтовым сервером.
Оно защищает организации от спама, вирусов, троянов и фишинговых писем.
Сборка построена на Debian 11.6 с ядром Linux 5.15, опционально поддерживает ядро 6.2 и включает ZFS 2.1.9 и PostgreSQL 13.10.
В версии интегрирован Apache SpamAssassin 4.0.0, который теперь умеет обнаруживать спам в вложениях, включая PDF, ODT, DOCX, DOC, RTF и изображения через OCR, а также лучше использует данные DMARC.
Другие улучшения: принудительный TLS для входящей почты, улучшенная поддержка SMTPUTF8, тёмная тема интерфейса, загрузка журналов задач и автоматический переход HTTP на HTTPS.
Решение распространяется под GNU AGPL v3, доступно как ISO для bare metal или виртуальной машины, а также как контейнерный appliance внутри Proxmox VE.
Для корпоративных пользователей Proxmox предлагает подписку на поддержку и доступ к Enterprise Repository, начиная с 165 евро за хост в год.

## Полная статья

**VIENNA, Austria – March 28, 2023 –**Enterprise software developer Proxmox Server Solutions GmbH ("Proxmox") today has released Proxmox Mail Gateway 7.3, the latest version of its open-source email security solution. The anti-spam and anti-virus filtering solution functions as a full featured mail proxy, is deployed between the firewall and the internal mail server. It protects organizations against threats, such as spam, viruses, Trojans, and phishing emails.

### What’s new in Proxmox Mail Gateway 7.3

- Proxmox Mail Gateway is a complete operating system based on Debian 11.6 (“Bullseye”), but using a newer Linux kernel 5.15, and including ZFS 2.1.9 and PostgreSQL 13.10. While the 5.15 kernel is the stable default, Linux kernel 6.2 can optionally be installed for better support of the latest hardware.
- With this version, the Proxmox developers integrate the new major version 4.0.0 of Apache SpamAssassin. It is now possible to detect spam inside of attachments. The detection is implemented for the file types .pdf, .odt, .docx, .doc, .rtf, as well as images (through OCR). Additionally, there is improved support for using information from the Domain-based Message Authentication, Reporting and Conformance (DMARC) policy, for more in-depth analysis. In general, all SpamAssassin configuration files shipped with the pmg-api package were adapted to the new SpamAssassin features.
- TLS-policy improvements: Proxmox Mail Gateway now can enforce TLS encryption for inbound mail.
- The SMTPUTF8 support has been improved and can now be disabled through the API or GUI.
- Dark theme: Like other Proxmox solutions, the Mail Gateway now brings a fully-integrated "Proxmox Dark" theme for the web interface. To detect if a user has requested light or dark color themes, the CSS media feature prefers-color-scheme is used. Through settings in the operating system or their browser, users can indicate their preferences. However, users can switch between the color schemes manually in the web interface as well. Furthermore, the documentation as well as the Proxmox Mail Gateway API Viewer now have a dark mode.
- Task logs can now be downloaded directly as text files for further inspection.
- To avoid the browser error "Connection reset", HTTP requests are automatically redirected to HTTPS. This can prevent confusion, especially after setting up a Proxmox Mail Gateway host for the first time.

### Availability

Proxmox Mail Gateway 7.3 is released under the GNU Affero GPL, v3 and is available as an ISO, which can be installed on bare metal, or on a virtual machine. The ISO contains an installation wizard that automatically installs and configures all the necessary components on the host. This eliminates the need for manual configuration later. The ISO contains the complete feature-set and can be downloaded at[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads)

Additionally, you can install Proxmox Mail Gateway as a container appliance inside Proxmox VE.

For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, as well as technical support. Subscription prices start at EUR 165 per host, per year, for unlimited users and domains. The Enterprise Repository provides regular updates via the web interface, and is recommended for production use.

**About Proxmox Mail Gateway**
Proxmox Mail Gateway is the leading open-source email security solution, protecting your mail server against all email threats from the moment they emerge. Organizations of any size can easily deploy and implement the comprehensive anti-spam and anti-virus platform in just a few minutes. Deploying the full featured mail proxy between the firewall and an internal mail server allows you to control all incoming and outgoing email traffic from the central, web-based interface. Proxmox filters all email traffic at the gateway before it reaches the mail server, protecting businesses against email attacks and other malicious threats. Proxmox Mail Gateway is open-source software, licensed under the GNU AGPL, v3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training services to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**Contact**
Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-7-3
