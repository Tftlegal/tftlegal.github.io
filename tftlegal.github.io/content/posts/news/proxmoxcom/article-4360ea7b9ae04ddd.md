---
title: "Proxmox Mail Gateway 6.4 released"
date: 2021-03-22T18:04:45Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-6-4"
summary: "Proxmox Server Solutions выпустила Proxmox Mail Gateway 6.4 — обновлённое решение для защиты электронной почты от спама, вирусов, троянов и фишинга."
---

# Proxmox Mail Gateway 6.4 released

## Краткое содержание

Proxmox Server Solutions выпустила Proxmox Mail Gateway 6.4 — обновлённое решение для защиты электронной почты от спама, вирусов, троянов и фишинга.

## Полная статья

**VIENNA, Austria – March 30, 2021 –**Enterprise software developer Proxmox Server Solutions GmbH ("Proxmox" or the "Company") has released Proxmox Mail Gateway 6.4, the latest version of its open-source email security solution. Proxmox Mail Gateway is a complete operating system based on Debian Buster 10.9, but using Linux kernel 5.4.106, which is under long term support (LTS) status. The anti-spam and anti-virus filtering solution from Proxmox functions as a full featured mail proxy, that is deployed between the firewall and the internal mail server. It protects organizations against threats, such as spam, viruses, Trojans, and phishing emails.

### New Features in Proxmox Mail Gateway 6.4

- General certificate management via the web interface: It is now possible to upload custom certificates from the web interface, or set up a cluster-wide ACME account to automatically get and renew certificates from an ACME provider. The certificate file handling is implemented in the Rust programming language.
Proxmox Mail Gateway now offers full integration of the ACME protocol via the GUI. This enables administrators to create valid and trusted certificates for their domains with the Let's Encrypt certificate authority. The process is easily configurable from the web interface and has full support for the http-01 and dns-01 challenges.
- Support for external SpamAssassin update channels (regular, automated updates): Proxmox Mail Gateway will now fetch verified updates from external rule channels, along with the updates from the SpamAssassin website (updates.spamassassin.org). Providing the channel’s URL and GPG key in a config-file is all that’s necessary. The KAM ruleset channel is now available, and a suitable configuration file is shipped with proxmox-spamassassin.
• Improved Spam Quarantine management for administrators: The admin view of the Spam Quarantine can now display the quarantined emails of all users at once.
- TLS-logging improvements: The Proxmox Message Tracking Center now shows when an outbound connection is established over TLS. The pmg-log-tracker, the binary at the core of the Proxmox Message Tracking Center has already proven to provide optimized performance and more stability, since it’s extension and re-implementation in Rust a year ago (with version 6.2 of Proxmox Mail Gateway).
- Enhancements to the integration of Proxmox Backup Server: If administrators have configured a Proxmox Backup Server-Remote, they can now get notification emails informing them about the result of a scheduled backup. Inclusion of the statistics database is now configurable on a per Remote basis.

**Forum Announcement**
[https://forum.proxmox.com/threads/proxmox-mail-gateway-6-4-released.86760/](https://forum.proxmox.com/threads/proxmox-mail-gateway-6-4-released.86760/)

### Availability

Proxmox Mail Gateway is released under the GNU Affero GPL, v3 and is available as an ISO, which can be installed on bare metal, or a VM. The ISO contains an installation wizard that automatically installs and configures all the necessary components on the host. This eliminates the need for manual configuration later. The ISO contains the complete feature-set and can be downloaded at[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads)

Additionally, you can install Proxmox Mail Gateway as a container appliance inside Proxmox VE.

For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, as well as technical support. Subscription prices start at EUR 139 per host, per year, for unlimited users and domains. The Enterprise Repository provides regular updates via the web interface, and is recommended for production use.

**About Proxmox Mail Gateway**
Proxmox Mail Gateway is the leading open-source email security solution, protecting your mail server against all email threats from the moment they emerge. Organizations of any size can easily deploy and implement the comprehensive anti-spam and anti-virus platform in just a few minutes. Deploying the full featured mail proxy between the firewall and an internal mail server allows you to control all incoming and outgoing email traffic from the central, web-based interface. Proxmox filters all email traffic at the gateway before it reaches the mail server, protecting businesses against email attacks and other malicious threats. Proxmox Mail Gateway is open-source software, licensed under the GNU AGPL, v3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions**
Proxmox is a provider of powerful, yet easy-to-use, open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy agile and efficient IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers professional support and training to ensure business continuity for its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**Contact:**Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-6-4
