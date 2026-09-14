---
title: "Proxmox Backup Server 2.3 available"
date: 2022-11-10T16:50:33Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-2-3"
summary: "Proxmox Backup Server 2.3 — обновлённое открытое решение для резервного копирования виртуальных машин, контейнеров и физических хостов. Версия добавила улучшения веб-интерфейса, работу с ленточными носителями и общие улучшения клиента и бэкенда. Теперь систему прунинга можно настраивать по пространствам имён, что упрощает управление сохранением бэкапов. Поддержана отправка метрик производительности в InfluxDB и улучшен восстановление каталогов с лент. Появился инструмент Proxmox Offline Mirror для обновления изолированных систем без доступа в интернет. Продукт остаётся бесплатным под GNU AGPL v3, а для бизнеса доступна платная поддержка с enterprise-репозиторием."
---

# Proxmox Backup Server 2.3 available

## Краткое содержание

Proxmox Backup Server 2.3 — обновлённое открытое решение для резервного копирования виртуальных машин, контейнеров и физических хостов. Версия добавила улучшения веб-интерфейса, работу с ленточными носителями и общие улучшения клиента и бэкенда. Теперь систему прунинга можно настраивать по пространствам имён, что упрощает управление сохранением бэкапов. Поддержана отправка метрик производительности в InfluxDB и улучшен восстановление каталогов с лент. Появился инструмент Proxmox Offline Mirror для обновления изолированных систем без доступа в интернет. Продукт остаётся бесплатным под GNU AGPL v3, а для бизнеса доступна платная поддержка с enterprise-репозиторием.

## Полная статья

**VIENNA, Austria – November 29, 2022 –**Enterprise software developer Proxmox Server Solutions (henceforth "Proxmox"), the company behind Proxmox Backup Server announces enhancements to its open-source server backup solution, which are now generally available. The enterprise solution for backing up and restoring VMs, containers, and physical hosts supports incremental backups, deduplication, Zstandard compression, and authenticated encryption. This version brings enhancements for the web interface, tape backup, as well as countless general client and backend improvements.

### Highlights in Proxmox Backup Server 2.3

- Proxmox Backup Server is based on Debian 11.5 (“Bullseye”), uses the newer Linux kernel 5.15 as stable default and kernel 5.19 as opt-in, and includes ZFS 2.1.6. Features such as fine-grained access control, data integrity verification, and the possibility to create off-site backups through remote sync and tape backups help planning a ransomware defense strategy and ensure that critical data stays protected.
- Pruning namespaces: Pruning lets you specify which backup snapshots you want to keep, in a systematic manner. With this version, the prune job system has been expanded to take namespaces into account: Up until now it has only been possible to add a single-schedule per datastore; now pruning can also be limited to certain namespaces. With the built-in prune simulator users can explore the effect of different retention options with various backup schedules. Namespaces in Proxmox Backup Server (introduced in version 2.2) help organize backups from multiple sites (local and remote) hierarchically, while keeping the required storage space minimal through deduplication. With the new fine-grained control in Proxmox Backup Server 2.3, businesses can determine when and how deeply a particular namespace is pruned.
- Support for sending metrics to InfluxDB: Version 2.3 of Proxmox Backup Server can gather and send critical stats, relevant for measuring performance, to InfluxDB, an open-source database management system for time series. Such metrics are, for example, CPU load averages and IOwait percentages, NIC traffic statistics, filesystem usage or IO for datastores.
- Tape backup improvements: The inventory command, which can be used for disaster recovery, can now optionally restore the catalogs of backups stored on tape, potentially saving critical time in a worst case scenario.
- Proxmox Offline Mirror: The Proxmox Offline Mirror tool allows to keep the Proxmox Backup Server nodes – with restricted or without access to the public internet – up-to-date and running. With the ‘proxmox-offline-mirror’ utility it’s possible to manage a local APT mirror for all package updates for Proxmox and Debian projects. From that mirror, users can create an external medium (USB flash drive or a local network share), and can then update their policy-restricted or air-gapped systems. For subscribers with a Premium or Standard subscription level, Proxmox offers an offline subscription key for its product portfolio.

The whole software stack of Proxmox Backup Server is written in Rust – a modern, fast, and memory-efficient language. Currently, the backup solution seamlessly integrates into Proxmox Virtual Environment – users just need to add the storage for the Proxmox Backup Server as a new storage backup target to the virtualization platform.

#### Availability

Proxmox Backup Server is free and open-source software, published under the GNU AGPL, v3. The downloadable ISO image can be quickly installed on bare-metal using the installation wizard. The centrally managed software stack Proxmox Backup Server 2.3 is now available for download at[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads)

Distribution upgrades from older versions of Proxmox Backup Server are possible via APT. It is also possible to install Proxmox Backup Server on top of Debian.

For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 449 per server, including unlimited backup storage and unlimited backup-clients.

**About Proxmox Backup Server**
Proxmox Backup Server is an enterprise backup solution for backing up and restoring virtual machines, containers, and physical hosts. The open-source client-server software supports incremental backups, deduplication, Zstandard compression, and authenticated encryption. To increase productivity, the easy-to-use Proxmox Backup Server allows you to back up your data in a space-efficient manner, restore VMs, archives or single objects in a flash. With the web-based user interface, you can effectively reduce work hours thanks to simplified management. Proxmox Backup Server is licensed under the GNU AGPL, v3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions**
Proxmox® is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training services to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-2-3
