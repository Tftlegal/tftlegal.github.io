---
title: "Proxmox Mail Gateway 9.0 based on Debian 13 “Trixie”"
date: 2025-10-01T09:58:05Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-9-0"
summary: "Proxmox представила Proxmox Mail Gateway 9.0 — обновленное open-source решение для защиты электронной почты от спама, вирусов, троянов и фишинга.   Ключевое обновление — переход на Debian 13 «Trixie» с более новым ядром Linux 6.14, улучшенной поддержкой оборудования и повышенной безопасностью.   В версии обновлены основные инструменты фильтрации, включая ClamAV и SpamAssassin, а также добавлена детальная возможность обновления с версии 8.2.   Разработчики полностью переработали интерфейс карантина для мобильных устройств, сделав управление задержанными письмами быстрее и удобнее.   Улучшены функции аутентификации и SSO, включая конфигурирование OpenID Connect realms через GUI и интеграцию с системами управления доступом.   Proxmox Mail Gateway 9.0 доступна для загрузки бесплатно, а для корпоративных пользователей Proxmox предлагает платную подписку на поддержку и Enterprise Repository."
---

# Proxmox Mail Gateway 9.0 based on Debian 13 “Trixie”

## Краткое содержание

Proxmox представила Proxmox Mail Gateway 9.0 — обновленное open-source решение для защиты электронной почты от спама, вирусов, троянов и фишинга.  
Ключевое обновление — переход на Debian 13 «Trixie» с более новым ядром Linux 6.14, улучшенной поддержкой оборудования и повышенной безопасностью.  
В версии обновлены основные инструменты фильтрации, включая ClamAV и SpamAssassin, а также добавлена детальная возможность обновления с версии 8.2.  
Разработчики полностью переработали интерфейс карантина для мобильных устройств, сделав управление задержанными письмами быстрее и удобнее.  
Улучшены функции аутентификации и SSO, включая конфигурирование OpenID Connect realms через GUI и интеграцию с системами управления доступом.  
Proxmox Mail Gateway 9.0 доступна для загрузки бесплатно, а для корпоративных пользователей Proxmox предлагает платную подписку на поддержку и Enterprise Repository.

## Полная статья

**VIENNA, Austria – October 01, 2025 –**Leading open-source server solutions provider Proxmox Server Solutions GmbH (henceforth “Proxmox”), celebrating its 20th year of innovation, today announced the release of Proxmox Mail Gateway 9.0. Main highlight of the updated email security solution is its modernized core now built upon Debian 13 “Trixie”, ensuring a robust foundation for the open-source platform.

Available in the market since 2005, the anti-spam and antivirus filtering solution Proxmox Mail Gateway functions as a full-featured mail proxy, deployed between the firewall and the internal mail server. It protects organizations against threats such as spam, viruses, Trojans, and phishing emails.

## Highlights in Proxmox Mail Gateway 9.0

Debian 13 “Trixie” at the core

This core update brings the latest Debian 13 “Trixie” release as foundation for Proxmox Mail Gateway including newer packages, improved hardware support, and enhanced security. Proxmox Mail Gateway 9.0 is using a newer Linux kernel 6.14 as stable default, enhancing hardware compatibility and performance. Also, updates to the latest versions of leading open-source technologies for email security like ClamAV 1.4.3 and SpamAssassin 4.0.2 are included. For existing users of version 8.2 , an extensively tested and detailed upgrade path is available to enable a smooth upgrade.

Redesigned quarantine interface for mobile

Users can now manage quarantined messages with a completely rebuilt interface optimized for mobile devices. Developed with the Rust-based Yew framework, the new quarantine UI replaces the previous implementation and provides a faster, cleaner, and more user-friendly experience on smartphones and tablets.

More flexible authentication and SSO

The single sign-on (SSO) and authentication realm features, first introduced in version 8.2, have been significantly improved and expanded. OpenID Connect realms are now fully configurable via the graphical user interface, including claim mappings and default role assignment for auto-provisioned users. This allows seamless integration with popular identity and access management solutions such as Keycloak, Zitadel, or LemonLDAP::NG.

Security enhancements and refined filtering

This version incorporates multiple hardening measures. The Content-Type filtering engine has been adjusted to support the updated MIME type definitions for Microsoft executables, ensuring these high-risk files continue to be reliably blocked.

### Availability

Proxmox Mail Gateway 9.0 is available now for download. The ISO image contains the complete feature-set and can be quickly installed on bare-metal using the installation wizard. Upgrades from version 8.2 to 9.0 are supported and a detailed migration guide is available. It is also possible to install the solution on top of Debian or as a container appliance inside Proxmox VE.

Proxmox Mail Gateway is free and open-source software, published under the GNU AGPLv3. For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, as well as technical support. Subscription prices start at EUR 180 per host, per year, for unlimited users and domains. The Enterprise Repository provides regular updates via the web interface, and is recommended for production use.

Resources:

- ISO Image Download:[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads)
- Forum Announcement:[https://forum.proxmox.com](https://forum.proxmox.com/threads/proxmox-mail-gateway-9-0-released.173159/)
- Roadmap: For published and upcoming features, see the[Release Notes & Roadmap](https://pmg.proxmox.com/wiki/index.php/Roadmap#Proxmox_Mail_Gateway_9.0)

###

**About Proxmox Mail Gateway**
Proxmox Mail Gateway is the leading open-source email security solution, protecting your mail server against all email threats from the moment they emerge. Organizations of any size can easily deploy and implement the comprehensive anti-spam and antivirus platform in just a few minutes. Deploying the full-featured mail proxy between the firewall and an internal mail server allows you to control all incoming and outgoing email traffic from the central, web-based interface. Proxmox filters all email traffic at the gateway before it reaches the mail server, protecting businesses against email attacks and other malicious threats. Proxmox Mail Gateway is open-source software, licensed under the GNU AGPLv3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions**
Proxmox provides powerful and user-friendly open-source server software. For 20 years, enterprises of all sizes and industries use the Proxmox solutions to deploy efficient and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support, training services, and an extensive partner ecosystem to ensure business continuity for its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH, marketing@proxmox.com

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-9-0
