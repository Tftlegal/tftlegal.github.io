---
title: "Proxmox Mail Gateway 6.3 with Proxmox Backup Server integration"
date: 2020-11-13T17:02:45Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-6-3"
summary: "<p><strong>VIENNA, Austria – November 19, 2020 –</strong> Enterprise software developer Proxmox Server Solutions GmbH has today released version 6.3 of its open-source email security solution, Proxmox Mail Gateway. Proxmox Mail Gateway is a complete operating system based on Debian Buster 10.6, but using Linux kernel 5.4, the latest long-term support version. The anti-spam and anti-virus filtering solution functions as a full featured mail proxy, that is deployed between the firewall and the internal mail server. It protects organizations against spam, viruses, Trojans, and phishing emails.</p> <h3>New Features in Proxmox Mail Gateway 6.3</h3> <ul> <li><strong>Proxmox Backup Server Integration:</strong> Proxmox Mail Gateway is fully supported by Proxmox Backup Server 1.0, the new, open-source, enterprise backup solution released on November 11, 2020. In Proxmox Mail Gateway 6.3, users can define multiple remote instances of Proxmox Backup Server to store backups on. In case of a large-scale disaster, they can then be quickly restored. It is also possible to schedule regular backups via the web interface, removing the need for manual backup creation and individual, scripted solutions.</li> <li><strong>Quarantine Link via login-page:</strong> If enabled by the system administrator, users can request mails containing a link to their quarantineview. This allows users to edit their individual blocklists, even if there are no mails in their quarantine. Until now this was only possible for sites using LDAP.</li> <li><strong>Enhancements to the Proxmox Message Tracking Center: </strong>To further improve the usability of Proxmox Mail Gateway, the pmg-log-tracker has been refined: <ul> <li>Case sensitivity has been removed from the search box.</li> <li>In case the pmg-smtp-filter fails to process emails due to misconfiguration, they are marked as rejected.</li> </ul> </li> </ul> <p>Notable bugfixes:</p> <ul> <li>DKIM signing now uses the longest matching domain for the 'd=' tag.</li> <li>Emails held in the Attachment Quarantine are assigned a new Message-ID - fixing interoperability with certain downstream servers (for example, MS Exchange), which silently discard messages with duplicate Message-IDs.</li> </ul> <p>Forum Announcement: <a href=\"https://forum.proxmox.com/threads/proxmox-mail-gateway-6-3-released.79276/\" target=\"_blank\" rel=\"noopener\">https://forum.proxmox.com/threads/proxmox-mail-gateway-6-3-released.79276</a></p> <h3>Availability</h3> <p>Proxmox Mail Gateway is released under the GNU Affero GPL, v3 and is available as an ISO, which can be installed on bare metal. The ISO contains an installation wizard that automatically installs and configures all the necessary components on the host. This eliminates the need for manual configuration later. The ISO contains the complete feature-set and can be downloaded at <a href=\"https://www.proxmox.com//en/component/zoo/?view=frontpage&amp;layout=frontpage&amp;Itemid=120\" target=\"_blank\">https://www.proxmox.com/downloads</a></p> <p>For enterprise users, the company Proxmox offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, as well as technical support. Subscription prices start at EUR 119 per host, per year for unlimited users. The Enterprise Repository provides regular updates via the web interface, and is recommended for production use.</p> <p><strong>About Proxmox Mail Gateway</strong><br />Proxmox Mail Gateway is the leading open-source email security solution, protecting your mail server against all email threats from the moment they emerge. Organizations of any size can easily deploy and implement the comprehensive anti-spam and anti-virus platform in just a few minutes. Deploying the full featured mail proxy between the firewall and an internal mail server allows you to control all incoming and outgoing email traffic from the central, web-based interface. Proxmox filters all email traffic at the gateway before it reaches the mail server, protecting businesses against email attacks and other malicious threats. Proxmox Mail Gateway is open-source software, licensed under the GNU AGPL, v3. Enterprise support subscriptions are available from Proxmox.</p> <p><strong>About Proxmox Server Solutions</strong><br />Proxmox is a provider of powerful, yet easy-to-use, open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training to ensure business continuity for its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.</p> <p><strong>Contact: </strong>Daniela Häsler, Proxmox Server Solutions GmbH</p> <p>&nbsp;</p> <p>&nbsp;</p>"
---

# Proxmox Mail Gateway 6.3 with Proxmox Backup Server integration

## Краткое содержание

<p><strong>VIENNA, Austria – November 19, 2020 –</strong> Enterprise software developer Proxmox Server Solutions GmbH has today released version 6.3 of its open-source email security solution, Proxmox Mail Gateway. Proxmox Mail Gateway is a complete operating system based on Debian Buster 10.6, but using Linux kernel 5.4, the latest long-term support version. The anti-spam and anti-virus filtering solution functions as a full featured mail proxy, that is deployed between the firewall and the internal mail server. It protects organizations against spam, viruses, Trojans, and phishing emails.</p>
<h3>New Features in Proxmox Mail Gateway 6.3</h3>
<ul>
<li><strong>Proxmox Backup Server Integration:</strong> Proxmox Mail Gateway is fully supported by Proxmox Backup Server 1.0, the new, open-source, enterprise backup solution released on November 11, 2020. In Proxmox Mail Gateway 6.3, users can define multiple remote instances of Proxmox Backup Server to store backups on. In case of a large-scale disaster, they can then be quickly restored. It is also possible to schedule regular backups via the web interface, removing the need for manual backup creation and individual, scripted solutions.</li>
<li><strong>Quarantine Link via login-page:</strong> If enabled by the system administrator, users can request mails containing a link to their quarantineview. This allows users to edit their individual blocklists, even if there are no mails in their quarantine. Until now this was only possible for sites using LDAP.</li>
<li><strong>Enhancements to the Proxmox Message Tracking Center: </strong>To further improve the usability of Proxmox Mail Gateway, the pmg-log-tracker has been refined:
<ul>
<li>Case sensitivity has been removed from the search box.</li>
<li>In case the pmg-smtp-filter fails to process emails due to misconfiguration, they are marked as rejected.</li>
</ul>
</li>
</ul>
<p>Notable bugfixes:</p>
<ul>
<li>DKIM signing now uses the longest matching domain for the 'd=' tag.</li>
<li>Emails held in the Attachment Quarantine are assigned a new Message-ID - fixing interoperability with certain downstream servers (for example, MS Exchange), which silently discard messages with duplicate Message-IDs.</li>
</ul>
<p>Forum Announcement: <a href="https://forum.proxmox.com/threads/proxmox-mail-gateway-6-3-released.79276/" target="_blank" rel="noopener">https://forum.proxmox.com/threads/proxmox-mail-gateway-6-3-released.79276</a></p>
<h3>Availability</h3>
<p>Proxmox Mail Gateway is released under the GNU Affero GPL, v3 and is available as an ISO, which can be installed on bare metal. The ISO contains an installation wizard that automatically installs and configures all the necessary components on the host. This eliminates the need for manual configuration later. The ISO contains the complete feature-set and can be downloaded at <a href="https://www.proxmox.com//en/component/zoo/?view=frontpage&amp;layout=frontpage&amp;Itemid=120" target="_blank">https://www.proxmox.com/downloads</a></p>
<p>For enterprise users, the company Proxmox offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, as well as technical support. Subscription prices start at EUR 119 per host, per year for unlimited users. The Enterprise Repository provides regular updates via the web interface, and is recommended for production use.</p>
<p><strong>About Proxmox Mail Gateway</strong><br />Proxmox Mail Gateway is the leading open-source email security solution, protecting your mail server against all email threats from the moment they emerge. Organizations of any size can easily deploy and implement the comprehensive anti-spam and anti-virus platform in just a few minutes. Deploying the full featured mail proxy between the firewall and an internal mail server allows you to control all incoming and outgoing email traffic from the central, web-based interface. Proxmox filters all email traffic at the gateway before it reaches the mail server, protecting businesses against email attacks and other malicious threats. Proxmox Mail Gateway is open-source software, licensed under the GNU AGPL, v3. Enterprise support subscriptions are available from Proxmox.</p>
<p><strong>About Proxmox Server Solutions</strong><br />Proxmox is a provider of powerful, yet easy-to-use, open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training to ensure business continuity for its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.</p>
<p><strong>Contact: </strong>Daniela Häsler, Proxmox Server Solutions GmbH</p>
<p>&nbsp;</p>
<p>&nbsp;</p>

## Полная статья

**VIENNA, Austria – November 19, 2020 –**Enterprise software developer Proxmox Server Solutions GmbH has today released version 6.3 of its open-source email security solution, Proxmox Mail Gateway. Proxmox Mail Gateway is a complete operating system based on Debian Buster 10.6, but using Linux kernel 5.4, the latest long-term support version. The anti-spam and anti-virus filtering solution functions as a full featured mail proxy, that is deployed between the firewall and the internal mail server. It protects organizations against spam, viruses, Trojans, and phishing emails.

### New Features in Proxmox Mail Gateway 6.3

- **Proxmox Backup Server Integration:**Proxmox Mail Gateway is fully supported by Proxmox Backup Server 1.0, the new, open-source, enterprise backup solution released on November 11, 2020. In Proxmox Mail Gateway 6.3, users can define multiple remote instances of Proxmox Backup Server to store backups on. In case of a large-scale disaster, they can then be quickly restored. It is also possible to schedule regular backups via the web interface, removing the need for manual backup creation and individual, scripted solutions.
- **Quarantine Link via login-page:**If enabled by the system administrator, users can request mails containing a link to their quarantineview. This allows users to edit their individual blocklists, even if there are no mails in their quarantine. Until now this was only possible for sites using LDAP.
- **Enhancements to the Proxmox Message Tracking Center:**To further improve the usability of Proxmox Mail Gateway, the pmg-log-tracker has been refined:

- Case sensitivity has been removed from the search box.
- In case the pmg-smtp-filter fails to process emails due to misconfiguration, they are marked as rejected.

Notable bugfixes:

- DKIM signing now uses the longest matching domain for the 'd=' tag.
- Emails held in the Attachment Quarantine are assigned a new Message-ID - fixing interoperability with certain downstream servers (for example, MS Exchange), which silently discard messages with duplicate Message-IDs.

Forum Announcement:[https://forum.proxmox.com/threads/proxmox-mail-gateway-6-3-released.79276](https://forum.proxmox.com/threads/proxmox-mail-gateway-6-3-released.79276/)

### Availability

Proxmox Mail Gateway is released under the GNU Affero GPL, v3 and is available as an ISO, which can be installed on bare metal. The ISO contains an installation wizard that automatically installs and configures all the necessary components on the host. This eliminates the need for manual configuration later. The ISO contains the complete feature-set and can be downloaded at[https://www.proxmox.com/downloads](https://www.proxmox.com/en/component/zoo/?view=frontpage&layout=frontpage&Itemid=120)

For enterprise users, the company Proxmox offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, as well as technical support. Subscription prices start at EUR 119 per host, per year for unlimited users. The Enterprise Repository provides regular updates via the web interface, and is recommended for production use.

**About Proxmox Mail Gateway**
Proxmox Mail Gateway is the leading open-source email security solution, protecting your mail server against all email threats from the moment they emerge. Organizations of any size can easily deploy and implement the comprehensive anti-spam and anti-virus platform in just a few minutes. Deploying the full featured mail proxy between the firewall and an internal mail server allows you to control all incoming and outgoing email traffic from the central, web-based interface. Proxmox filters all email traffic at the gateway before it reaches the mail server, protecting businesses against email attacks and other malicious threats. Proxmox Mail Gateway is open-source software, licensed under the GNU AGPL, v3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions**
Proxmox is a provider of powerful, yet easy-to-use, open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training to ensure business continuity for its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**Contact:**Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-mail-gateway-6-3
