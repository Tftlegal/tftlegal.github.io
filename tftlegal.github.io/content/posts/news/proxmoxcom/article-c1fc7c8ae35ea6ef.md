---
title: "Proxmox VE 3.3 released with HTML5, Proxmox VE Firewall, Two-factor authentication, ..."
date: 2014-09-15T09:27:59Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-3-3-released"
summary: "Компания Proxmox представила версию Proxmox VE 3.3, ориентированную на усиление безопасности и расширение возможностей управления виртуализацией.   Ключевые нововведения включают распределённый Proxmox VE Firewall и двухфакторную аутентификацию с поддержкой YubiKey и TOTP/OATH.   Firewall интегрирован в веб-интерфейс и кластер, позволяя настраивать правила для хостов, кластера, виртуальных машин и контейнеров, а также обеспечивая высокую пропускную способность без узких мест.   В версии также появились HTML5-консоль noVNC по умолчанию, плагин хранения ZFS и мобильный интерфейс Proxmox VE Mobile для управления системой со смартфонов и планшетов.   Proxmox VE 3.3 распространяется под лицензией AGPL v3, доступна для загрузки в виде ISO, а для корпоративных клиентов предлагаются платные подписки.   Продукция Proxmox уже используется на десятках тысяч серверов по всему миру, а компания продолжает развивать свои решения в области виртуализации и почтовых шлюзов."
---

# Proxmox VE 3.3 released with HTML5, Proxmox VE Firewall, Two-factor authentication, ...

## Краткое содержание

Компания Proxmox представила версию Proxmox VE 3.3, ориентированную на усиление безопасности и расширение возможностей управления виртуализацией.  
Ключевые нововведения включают распределённый Proxmox VE Firewall и двухфакторную аутентификацию с поддержкой YubiKey и TOTP/OATH.  
Firewall интегрирован в веб-интерфейс и кластер, позволяя настраивать правила для хостов, кластера, виртуальных машин и контейнеров, а также обеспечивая высокую пропускную способность без узких мест.  
В версии также появились HTML5-консоль noVNC по умолчанию, плагин хранения ZFS и мобильный интерфейс Proxmox VE Mobile для управления системой со смартфонов и планшетов.  
Proxmox VE 3.3 распространяется под лицензией AGPL v3, доступна для загрузки в виде ISO, а для корпоративных клиентов предлагаются платные подписки.  
Продукция Proxmox уже используется на десятках тысяч серверов по всему миру, а компания продолжает развивать свои решения в области виртуализации и почтовых шлюзов.

## Полная статья

**Vienna – September 15, 2014 –**Proxmox Server Solutions GmbH, developer of the open source server virtualization solution Proxmox Virtual Environment (VE), today released version 3.3. The series of new features focus on security and include the Proxmox VE Firewall and two-factor authentication. A HTML5 console, the ZFS storage plugin and the Proxmox VE Mobile touch interface extend the range of use. Many package updates are included in the release.

Highlight of the new release is the[Proxmox VE Firewall](http://pve.proxmox.com/wiki/Proxmox_VE_Firewall). It has a distributed nature and is designed to protect the whole IT infrastructure. Completely integrated into the web-GUI and the cluster stack, it allows the user to setup firewall rules for all hosts, the cluster, virtual machines and containers. To keep these tasks simple, the Proxmox VE Firewall comes with features like firewall macros, security groups, IP sets and aliases. While configuration is stored on the cluster file system layer, the iptables run on each cluster node providing full isolation between individual virtual machines. In contrast to a central firewall solution, the distributed nature of the Proxmox VE Firewall provides a higher bandwidth, and at the same time avoids bottlenecks.

The newly integrated[two-factor authentication](http://pve.proxmox.com/wiki/Two-Factor_Authentication)will add an extra layer of security to Proxmox VE. With version 3.3, login with a one-time password (OTP) can be enabled to the username/password interface login – this works for all authentication realms, including LDAP or Active Directory. The Proxmox developers offer two different methods: YubiKey from Yubico and Time Based One-Time Passwords (with OATH). These secure login processes shall help to prevent unauthorized persons or programs to access the virtualization servers.

The new HTML5 console (noVNC) is set as the default in Proxmox VE 3.3. It works on every platform (Windows/Linux/OSX), even on mobile devices and supersedes the installation of the Java plugin or SPICE viewer.

In addition to the full featured web interface,[Proxmox VE Mobile](http://pve.proxmox.com/wiki/Proxmox_VE_Mobile)is a touch interface designed specifically for the use on mobile devices (phones and tablets). It is just a HTML5 app built with Sencha Touch and it runs on any mobile with a modern browser. Proxmox VE Mobile includes a lot of key functionalities needed to manage virtualization on the go, including the HTML5 and the SPICE console. It is meant as an additional help for the admin, but not to replace the full admin interface. Proxmox VE Mobile also supports two-factor authentication like a YubiKey with NFC.

**Availability of Proxmox VE 3.3:**
Proxmox VE 3.3 is released under the AGPL, v3 and is available as ISO-image for download at[http://www.proxmox.com/downloads](https://www.proxmox.com/en/component/zoo/?view=frontpage&layout=frontpage&Itemid=120)For enterprise customers, Proxmox offers subscriptions starting at EUR 49.90 per year and CPU socket.

**Facts and Milestones Proxmox VE**
Proxmox VE is used by 62.000 hosts in 140 countries. The GUI is available in 17 languages and the active community counts more than 24.000 forum members.

- Proxmox VE 3.3 with HTML5 console, Proxmox VE Firewall, Two-factor authentication, ZFS storage plugin, and a touch interface Proxmox VE Mobile, qemu 2.1 in September 2014
- Proxmox VE 3.2 with SPICE and spiceterm, Ceph storage system, Open vSwitch, support for VMware™ pvscsi and vmxnet3, new ZFS storage plugin, qemu 1.7 in March 2014.
- Proxmox VE V3.1 with Enterprise-Repository updates via GUI, SPICE, GlusterFS storage plugin in August 2013.
- Proxmox VE V3.0 brings VM templates and cloning, new event driven API server, Debian 7.0 (Wheezy), bootlogd in May 2013.
- Proxmox VE V2.0 with High-Availability (HA) based on Redhat Cluster and Corosync; RESTful web API is released in April 2012.
- Proxmox VE V0.9 - First public release in April 2008. GUI for managing KVM and containers.

**About Proxmox Virtual Environment**
Proxmox Virtual Environment is an open source virtualization management solution for servers. It supports KVM-based guests, as well as container-virtualization with OpenVZ and includes strong high-availability (HA) support based on Redhat Cluster and Corosync. You can easily virtualize even the most demanding Linux and Windows application workloads with Proxmox VE. Installation is fast and easy with a bare-metal installer and configuration is done via the integrated web-based management interface. Based on Debian GNU/Linux and fully licensed under the GNU Affero General Public License, Version 3 (AGPL-3.0), Proxmox VE is a solution without restrictions for home and business use.

**About Proxmox Server Solutions GmbH**
Proxmox Server Solutions GmbH is an open source software provider dedicated to develop powerful and efficient server solutions. With its two core products – Proxmox Virtual Environment (Proxmox VE) and Proxmox Mail Gateway – the company offers flexible, affordable and easy-to-use software for businesses implementing secure and open-source IT infrastructures. Proxmox solutions are widely used in businesses regardless of size, sector or industry as well as in NGOs and in the educational sector. The company also offers services like commercial subscriptions and trainings. A worldwide partner network and a huge active community guarantee business continuity for Proxmox users. Proxmox is member of the Linux Foundation and the Open Virtualization Alliance. Proxmox Server Solutions GmbH is an independent and profitable company based in Vienna, Austria.

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-ve-3-3-released
