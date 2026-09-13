---
title: "Proxmox Mail Gateway 5.2"
date: 2019-03-18T13:55:02Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-5-2"
summary: "Proxmox Server Solutions выпустила Proxmox Mail Gateway 5.2 — обновлённую версию открытого решения для защиты электронной почты.   Продукт представляет собой операционную систему на базе Debian Stretch и работает как почтовый прокси между файрволом и внутренним почтовым сервером.   Новая версия добавила мобильный интерфейс для карантина на базе Framework7, а также упростила управление черными и белыми списками.   Также улучшена интеграция LDAP, включая поддержку FQDN, certificate verification и LDAP+STARTTLS.   Появилась установка в виде Linux Container через новый appliance template и метапакет, что уменьшает размер и ускоряет развертывание.   Дополнительно улучшены логи SpamAssassin, TLS-настройки веб-интерфейса, поддержка IPv6 в кластере и добавлена команда pmg-system-report для диагностики.   Proxmox Mail Gateway 5.2 распространяется под GNU AGPL v3, доступна как ISO, а платная enterprise-поддержка начинается от 99 евро за хост в год."
---

# Proxmox Mail Gateway 5.2

## Краткое содержание

Proxmox Server Solutions выпустила Proxmox Mail Gateway 5.2 — обновлённую версию открытого решения для защиты электронной почты.  
Продукт представляет собой операционную систему на базе Debian Stretch и работает как почтовый прокси между файрволом и внутренним почтовым сервером.  
Новая версия добавила мобильный интерфейс для карантина на базе Framework7, а также упростила управление черными и белыми списками.  
Также улучшена интеграция LDAP, включая поддержку FQDN, certificate verification и LDAP+STARTTLS.  
Появилась установка в виде Linux Container через новый appliance template и метапакет, что уменьшает размер и ускоряет развертывание.  
Дополнительно улучшены логи SpamAssassin, TLS-настройки веб-интерфейса, поддержка IPv6 в кластере и добавлена команда pmg-system-report для диагностики.  
Proxmox Mail Gateway 5.2 распространяется под GNU AGPL v3, доступна как ISO, а платная enterprise-поддержка начинается от 99 евро за хост в год.

## Полная статья

**VIENNA, Austria – March 26, 2019 –**Enterprise software developer Proxmox Server Solutions GmbH has released Proxmox Mail Gateway 5.2, the latest version of its open-source email security solution. The Mail Gateway is a complete operating system based on Debian Stretch 9.8 with a 4.15 kernel. The anti-spam and anti-virus filtering solution, available for over 14 years, functions like a full featured mail proxy deployed between the firewall and the internal mail server and protects organizations against spam, viruses, trojans, and phishing emails.

Proxmox Mail Gateway 5.2 introduces a new mobile interface for the quarantine, making it very handy to check Delivery/Whitelist/Blacklist/Delete emails in the quarantine from any mobile device. The mobile interface is based on Framework7, a full featured open-source HTML framework for building Android and iOS apps.

The Proxmox Mail Gateway 5.2 release also introduces improvements in the LDAP integration, now allowing the use of Fully-Qualified Domain Names (FQDN) instead of IPs in the web user interface. Support for certificate verification (can be enabled for new deployments), and for LDAP+starttls has been added.

A new appliance template enables users to install the Proxmox Mail Gateway 5.2 as a privileged or unprivileged Linux Container. A new 'proxmox-mailgateway-container' Metapackage makes the installation of the template smaller and faster. As it does not depend on a kernel, it results in a reduced size and fewer updates.

The logging has been improved. The pmg-smtp-filter now logs each SpamAssassin rule’s score in addition to the rule names, therefore simplifying the analysis of the spam filter's performance without the need to access the email's source.

With improvements made to the TLS configuration of the web-based user interface, the pmgproxy can be configured via '/etc/default/pmgproxy' to disable or enable certain ciphers, compression, or cipher selection preferences.

### Further improvements in Proxmox Mail Gateway 5.2

- The new command `pmg-system-report` provides an overview of the key characteristics of the setup and performance in Proxmox Mail Gateway, and improves the initial diagnosis for the Enterprise support.
- With the .eml download from the quarantine interface (non-mobile) users can download the complete source of a quarantined message in .eml format for further analysis.
- Improved handling of Blacklist/Whitelist in the Quarantine interface: a ‘multiselect’ option for removing multiple entries at once has been added.
- Added support for custom checks: This functionality enables users to integrate their own custom check logic by providing a defined interface which can be enabled optionally, and runs a custom check before the email gets handed to the virus scanner and the rule system.
- Proxmox Mail Gateway cluster now works with IPv6.
- proxmox-spamassassin: Update the shipped rule sets.

### Availability

Proxmox Mail Gateway is released under the GNU Affero GPL, v3 and is available as a bare-metal ISO install with an installation wizard that automatically installs and configures all necessary components on the host eliminating manual installation configuration. The ISO install contains the full feature-set and can be downloaded at[https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/component/zoo/?view=frontpage&layout=frontpage&Itemid=120).

Enterprise support is available from Proxmox. Subscription prices start at EUR 99 per host and year for unlimited users and provides access to the extensively tested enterprise package repository with regular updates via the web interface. It is recommended for production use.

#### Further information:

- Forum announcement:[https://forum.proxmox.com/forums/announcements.7/](https://forum.proxmox.com/threads/proxmox-mail-gateway-5-2-available.52786/)
- Release Notes:[https://www.proxmox.com/de/downloads/item/proxmox-mail-gateway-5-2-iso-installer](https://www.proxmox.com/de/downloads/item/proxmox-mail-gateway-5-2-iso-installer)

**About Proxmox Mail Gateway**
Proxmox Mail Gateway is the leading open-source email security solution protecting your mail server against all email threats the moment they emerge. Organizations of any size can easily implement and deploy the comprehensive anti-spam and anti-virus platform in only a few minutes. Deploying the full featured mail proxy between the firewall and an internal mail server allows to control all incoming and outgoing email traffic from the central web-based interface. Proxmox filters the whole email traffic at the gateway before it reaches the mail server protecting businesses against email attacks and other malicious threats. Proxmox Mail Gateway is open-source software, licensed under the GNU AGPL, v3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions GmbH**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises regardless of size, sector or industry use the stable, secure, scalable, and open Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**Contact:**Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-5-2
