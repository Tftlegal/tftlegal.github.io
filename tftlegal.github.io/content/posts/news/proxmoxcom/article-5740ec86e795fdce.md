---
title: "Proxmox Mail Gateway 8.0 based on Debian 12 “Bookworm”"
date: 2023-07-10T13:00:50Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-8-0"
summary: "Proxmox выпустила версию 8.0 своего open-source решения Proxmox Mail Gateway для защиты почты от спама и вирусов. Новая версия построена на Debian 12 с ядром Linux 6.2 и включает ZFS 2.1.12 и PostgreSQL 15.3. Для пользователей версии 7.3 доступны подробная документация по обновлению и скрипт предварительной проверки. В релизе добавлен текстовый установщик TUI на Rust, который помогает избежать проблем с графической установкой на разных системах. Proxmox Mail Gateway работает как почтовый прокси между файрволом и внутренним почтовым сервером, защищая от спама, вирусов, троянов и фишинга. Решение распространяется под GNU AGPL v3, доступно как ISO для bare metal или виртуальной машины, а также как контейнер в Proxmox VE, и для бизнеса предлагается платная поддержка."
---

# Proxmox Mail Gateway 8.0 based on Debian 12 “Bookworm”

## Краткое содержание

Proxmox выпустила версию 8.0 своего open-source решения Proxmox Mail Gateway для защиты почты от спама и вирусов.
Новая версия построена на Debian 12 с ядром Linux 6.2 и включает ZFS 2.1.12 и PostgreSQL 15.3.
Для пользователей версии 7.3 доступны подробная документация по обновлению и скрипт предварительной проверки.
В релизе добавлен текстовый установщик TUI на Rust, который помогает избежать проблем с графической установкой на разных системах.
Proxmox Mail Gateway работает как почтовый прокси между файрволом и внутренним почтовым сервером, защищая от спама, вирусов, троянов и фишинга.
Решение распространяется под GNU AGPL v3, доступно как ISO для bare metal или виртуальной машины, а также как контейнер в Proxmox VE, и для бизнеса предлагается платная поддержка.

## Полная статья

**VIENNA, Austria – June 29, 2023 –**Enterprise software developer Proxmox Server Solutions GmbH (henceforth "Proxmox") today released version 8.0 of its open-source email security solution Proxmox Mail Gateway. The anti-spam and anti-virus filtering solution is a complete operating system now based on Debian 12 (“Bookworm”), but defaulting to a modern Linux kernel 6.2, and includes ZFS 2.1.12, and PostgreSQL 15.3. For users of Proxmox Mail Gateway 7.3 a detailed upgrade documentation is available, helping to enable a smooth upgrade. Additionally, a pre-flight checking script helps to identify potential misconfigurations before the upgrade.

With this major release 8.0, the Proxmox development team adds a new text-based UI (TUI) mode for the installation ISO. The TUI is written in Rust, using the Cursive library. With the new TUI mode, issues with launching the GTK based graphical installer, sometimes observed on both very new and rather old hardware, can be eliminated.

The default settings in many areas of this version have been optimized, reflecting the various deployment possibilities seen among Proxmox Mail Gateway users. This further enhances the out-of-the-box experience and spam detection rates for new users.

Available in the market since 2005, the anti-spam and anti-virus filtering solution Proxmox Mail Gateway functions as a full featured mail proxy, that is deployed between the firewall and the internal mail server. It protects organizations against threats, such as spam, viruses, Trojans, and phishing emails.

### Availability

Proxmox Mail Gateway 8.0 is released under the GNU Affero GPL, v3 and is available as an ISO, which can be installed on bare metal, or a virtual machine. The ISO contains an installation wizard that automatically installs and configures all the necessary components on the host. This eliminates the need for manual configuration later. The ISO can be downloaded at[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads)

A detailed[upgrade documentation from 7 to 8](https://pmg.proxmox.com/wiki/index.php/Upgrade_from_7_to_8)is available.
Additionally, users can install Proxmox Mail Gateway as a container appliance inside Proxmox VE.

For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, as well as technical support. Subscription prices start at EUR 165 per host, per year, for unlimited users and domains. The Enterprise Repository provides regular updates via the web interface, and is recommended for production use.

**About Proxmox Server Solutions**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training services to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**About Proxmox Mail Gateway**
Proxmox Mail Gateway is the leading open-source email security solution, protecting your mail server against all email threats from the moment they emerge. Organizations of any size can easily deploy and implement the comprehensive anti-spam and anti-virus platform in just a few minutes. Deploying the full featured mail proxy between the firewall and an internal mail server allows you to control all incoming and outgoing email traffic from the central, web-based interface. Proxmox filters all email traffic at the gateway before it reaches the mail server, protecting businesses against email attacks and other malicious threats. Proxmox Mail Gateway is open-source software, licensed under the GNU AGPL, v3. Enterprise support subscriptions are available from Proxmox.

**Contact:**Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-8-0
