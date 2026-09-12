---
title: "Proxmox Mail Gateway 5.1 released"
date: 2018-10-08T16:31:34Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-5-1"
summary: "<p><strong>VIENNA, Austria – October 09, 2018 – </strong>Proxmox Server Solutions GmbH today released version 5.1 of its open-source email security platform Proxmox Mail Gateway. The Mail Gateway is a complete operating system based on Debian Stretch 9.5 with a 4.15 kernel. The anti-spam and anti-virus filtering solution functions like a full featured mail proxy deployed between the firewall and the internal mail server and protects organizations against spam, viruses, Trojans, and phishing emails.</p> <p>Proxmox Mail Gateway 5.1 comes with Debian security updates, new features, bug fixes, and GUI improvements:</p> <ul> <li>The new Transport Layer Security (TLS) policy provides certificate-based authentication and encrypted sessions. Users can now set a different TLS policy per destination domain, in case they need to prevent e-mail delivery without encryption, or to work around a broken STARTTLS ESMTP implementation. Configuration of the TLS policy can be done via GUI. TLS is also possible on internal SMTP port/traffic.</li> <li>The updated user management now allows a new help desk role enabling help desk staff to access the quarantine.</li> <li>Proxmox Mail Gateway 5.1 supports SMTPUTF8. This will fix the problem with some non delivered Google mails.</li> <li>Smarthost ports: Editing and showing smarthost port is possible.</li> <li>On the web-based user interface, the Proxmox developers improved the “Spam Quarantine” section adding keyboard shortcuts (for ‘Whitelist’, ‘Blacklist’, ‘Deliver’, and ‘Delete’), allowing contextual menus with right click, and enabling multiselection of emails.</li> <li>Graphs and reports on the GUI have been optimized in Proxmox Mail Gateway 5.1.</li> </ul> <h3>Availability</h3> <p>Proxmox Mail Gateway is released under the GNU Affero GPL, v3 and is available as a bare-metal ISO install with an installation wizard that automatically installs and configures all necessary components on the host eliminating manual installation configuration. The ISO install contains the full feature-set and can be downloaded at <a href=\"https://www.proxmox.com\" target=\"_blank\" rel=\"noopener\">https://www.proxmox.com</a>.</p> <p>For enterprise users the company Proxmox offers a subscription-based support model with access to the stable enterprise repository and to technical support. Subscription pricing starts at EUR 99 per host and year for unlimited users. A subscription provides access to the enterprise package repository with regular updates via the web-based user interface, and is recommended for production use.</p> <p>Users of the Proxmox Mail Gateway 4.x can backup/restore the rule database and email statistic from 4 to 5.1. An inplace-upgrade from Proxmox Mail Gateway 4 to 5.1 is not possible.</p> <h5>Further information:</h5> <ul> <li>Forum announcement: <a href=\"https://forum.proxmox.com/threads/proxmox-mail-gateway-5-1-available.47798/\" target=\"_blank\" rel=\"noopener\" title=\"proxmox mail gateway 5.1 is available\">https://forum.proxmox.com/threads/proxmox-mail-gateway-5-1-available.47798/</a></li> <li>Download: <a href=\"https://www.proxmox.com/de/downloads/item/proxmox-mail-gateway-5-1-iso-installer\" target=\"_blank\" rel=\"noopener\" title=\"Download and release notes for Proxmox Mail Gateway 5.1\">https://www.proxmox.com/en/downloads</a></li> </ul> <p><strong>About Proxmox Mail Gateway</strong><br />Proxmox Mail Gateway is the leading open-source email security solution protecting your mail server against all email threats the moment they emerge. Organizations of any size can easily implement and deploy the comprehensive anti-spam and anti-virus platform in a few minutes. Deploying the full featured mail proxy between the firewall and an internal mail server allows to control all incoming and outgoing email traffic from the single web-based interface. Proxmox filters the whole email traffic at the gateway before it reaches the mail server and protects businesses against email attacks and other malicious threats. Proxmox Mail Gateway is open-source software, licensed under the GNU AGPL, v3. Commercial support subscriptions are available from Proxmox.</p> <p><strong>About Proxmox Server Solutions GmbH</strong><br />Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises regardless of size, sector or industry use the stable, secure, scalable, and open Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.</p> <p><strong>Contact: </strong>Daniela Häsler, Proxmox Server Solutions GmbH</p> <p>&nbsp;</p> <p></p>"
---

# Proxmox Mail Gateway 5.1 released

## Краткое содержание

<p><strong>VIENNA, Austria – October 09, 2018 – </strong>Proxmox Server Solutions GmbH today released version 5.1 of its open-source email security platform Proxmox Mail Gateway. The Mail Gateway is a complete operating system based on Debian Stretch 9.5 with a 4.15 kernel. The anti-spam and anti-virus filtering solution functions like a full featured mail proxy deployed between the firewall and the internal mail server and protects organizations against spam, viruses, Trojans, and phishing emails.</p>
<p>Proxmox Mail Gateway 5.1 comes with Debian security updates, new features, bug fixes, and GUI improvements:</p>
<ul>
<li>The new Transport Layer Security (TLS) policy provides certificate-based authentication and encrypted sessions. Users can now set a different TLS policy per destination domain, in case they need to prevent e-mail delivery without encryption, or to work around a broken STARTTLS ESMTP implementation. Configuration of the TLS policy can be done via GUI. TLS is also possible on internal SMTP port/traffic.</li>
<li>The updated user management now allows a new help desk role enabling help desk staff to access the quarantine.</li>
<li>Proxmox Mail Gateway 5.1 supports SMTPUTF8. This will fix the problem with some non delivered Google mails.</li>
<li>Smarthost ports: Editing and showing smarthost port is possible.</li>
<li>On the web-based user interface, the Proxmox developers improved the “Spam Quarantine” section adding keyboard shortcuts (for ‘Whitelist’, ‘Blacklist’, ‘Deliver’, and ‘Delete’), allowing contextual menus with right click, and enabling multiselection of emails.</li>
<li>Graphs and reports on the GUI have been optimized in Proxmox Mail Gateway 5.1.</li>
</ul>
<h3>Availability</h3>
<p>Proxmox Mail Gateway is released under the GNU Affero GPL, v3 and is available as a bare-metal ISO install with an installation wizard that automatically installs and configures all necessary components on the host eliminating manual installation configuration. The ISO install contains the full feature-set and can be downloaded at <a href="https://www.proxmox.com" target="_blank" rel="noopener">https://www.proxmox.com</a>.</p>
<p>For enterprise users the company Proxmox offers a subscription-based support model with access to the stable enterprise repository and to technical support. Subscription pricing starts at EUR 99 per host and year for unlimited users. A subscription provides access to the enterprise package repository with regular updates via the web-based user interface, and is recommended for production use.</p>
<p>Users of the Proxmox Mail Gateway 4.x can backup/restore the rule database and email statistic from 4 to 5.1. An inplace-upgrade from Proxmox Mail Gateway 4 to 5.1 is not possible.</p>
<h5>Further information:</h5>
<ul>
<li>Forum announcement: <a href="https://forum.proxmox.com/threads/proxmox-mail-gateway-5-1-available.47798/" target="_blank" rel="noopener" title="proxmox mail gateway 5.1 is available">https://forum.proxmox.com/threads/proxmox-mail-gateway-5-1-available.47798/</a></li>
<li>Download: <a href="https://www.proxmox.com/de/downloads/item/proxmox-mail-gateway-5-1-iso-installer" target="_blank" rel="noopener" title="Download and release notes for Proxmox Mail Gateway 5.1">https://www.proxmox.com/en/downloads</a></li>
</ul>
<p><strong>About Proxmox Mail Gateway</strong><br />Proxmox Mail Gateway is the leading open-source email security solution protecting your mail server against all email threats the moment they emerge. Organizations of any size can easily implement and deploy the comprehensive anti-spam and anti-virus platform in a few minutes. Deploying the full featured mail proxy between the firewall and an internal mail server allows to control all incoming and outgoing email traffic from the single web-based interface. Proxmox filters the whole email traffic at the gateway before it reaches the mail server and protects businesses against email attacks and other malicious threats. Proxmox Mail Gateway is open-source software, licensed under the GNU AGPL, v3. Commercial support subscriptions are available from Proxmox.</p>
<p><strong>About Proxmox Server Solutions GmbH</strong><br />Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises regardless of size, sector or industry use the stable, secure, scalable, and open Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.</p>
<p><strong>Contact: </strong>Daniela Häsler, Proxmox Server Solutions GmbH</p>
<p>&nbsp;</p>
<p></p>

## Полная статья

**VIENNA, Austria – October 09, 2018 –**Proxmox Server Solutions GmbH today released version 5.1 of its open-source email security platform Proxmox Mail Gateway. The Mail Gateway is a complete operating system based on Debian Stretch 9.5 with a 4.15 kernel. The anti-spam and anti-virus filtering solution functions like a full featured mail proxy deployed between the firewall and the internal mail server and protects organizations against spam, viruses, Trojans, and phishing emails.

Proxmox Mail Gateway 5.1 comes with Debian security updates, new features, bug fixes, and GUI improvements:

- The new Transport Layer Security (TLS) policy provides certificate-based authentication and encrypted sessions. Users can now set a different TLS policy per destination domain, in case they need to prevent e-mail delivery without encryption, or to work around a broken STARTTLS ESMTP implementation. Configuration of the TLS policy can be done via GUI. TLS is also possible on internal SMTP port/traffic.
- The updated user management now allows a new help desk role enabling help desk staff to access the quarantine.
- Proxmox Mail Gateway 5.1 supports SMTPUTF8. This will fix the problem with some non delivered Google mails.
- Smarthost ports: Editing and showing smarthost port is possible.
- On the web-based user interface, the Proxmox developers improved the “Spam Quarantine” section adding keyboard shortcuts (for ‘Whitelist’, ‘Blacklist’, ‘Deliver’, and ‘Delete’), allowing contextual menus with right click, and enabling multiselection of emails.
- Graphs and reports on the GUI have been optimized in Proxmox Mail Gateway 5.1.

### Availability

Proxmox Mail Gateway is released under the GNU Affero GPL, v3 and is available as a bare-metal ISO install with an installation wizard that automatically installs and configures all necessary components on the host eliminating manual installation configuration. The ISO install contains the full feature-set and can be downloaded at[https://www.proxmox.com](https://www.proxmox.com).

For enterprise users the company Proxmox offers a subscription-based support model with access to the stable enterprise repository and to technical support. Subscription pricing starts at EUR 99 per host and year for unlimited users. A subscription provides access to the enterprise package repository with regular updates via the web-based user interface, and is recommended for production use.

Users of the Proxmox Mail Gateway 4.x can backup/restore the rule database and email statistic from 4 to 5.1. An inplace-upgrade from Proxmox Mail Gateway 4 to 5.1 is not possible.

##### Further information:

- Forum announcement:[https://forum.proxmox.com/threads/proxmox-mail-gateway-5-1-available.47798/](https://forum.proxmox.com/threads/proxmox-mail-gateway-5-1-available.47798/)
- Download:[https://www.proxmox.com/en/downloads](https://www.proxmox.com/de/downloads/item/proxmox-mail-gateway-5-1-iso-installer)

**About Proxmox Mail Gateway**
Proxmox Mail Gateway is the leading open-source email security solution protecting your mail server against all email threats the moment they emerge. Organizations of any size can easily implement and deploy the comprehensive anti-spam and anti-virus platform in a few minutes. Deploying the full featured mail proxy between the firewall and an internal mail server allows to control all incoming and outgoing email traffic from the single web-based interface. Proxmox filters the whole email traffic at the gateway before it reaches the mail server and protects businesses against email attacks and other malicious threats. Proxmox Mail Gateway is open-source software, licensed under the GNU AGPL, v3. Commercial support subscriptions are available from Proxmox.

**About Proxmox Server Solutions GmbH**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises regardless of size, sector or industry use the stable, secure, scalable, and open Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**Contact:**Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-5-1
