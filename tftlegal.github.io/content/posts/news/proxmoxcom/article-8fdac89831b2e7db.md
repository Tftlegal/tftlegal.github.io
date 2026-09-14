---
title: "Proxmox Backup Server 3.2 released"
date: 2024-04-25T11:43:28Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-3-2"
summary: "Proxmox объявила о выпуске версии 3.2 своего открытого сервера резервного копирования. Решение предназначено для бэкапа и восстановления виртуальных машин, контейнеров и физических хостов. Новая версия добавила гибкую систему уведомлений с маршрутизацией по правилам и поддержкой Gotify, SMTP и локального Postfix. Появилась автоматизированная установка через инструмент proxmox-auto-install-assistant, что упрощает развертывание на bare-metal. Также улучшены фильтры для синхронизации и ленточного бэкапа, а также добавлен общий обзор задач очистки и сбора мусора по хранилищам. Proxmox Backup Server 3.2 доступна для загрузки, поддерживает обновление через APT и лицензируется под GNU AGPLv3, а платная поддержка начинается от 520 евро за сервер в год."
---

# Proxmox Backup Server 3.2 released

## Краткое содержание

Proxmox объявила о выпуске версии 3.2 своего открытого сервера резервного копирования. Решение предназначено для бэкапа и восстановления виртуальных машин, контейнеров и физических хостов. Новая версия добавила гибкую систему уведомлений с маршрутизацией по правилам и поддержкой Gotify, SMTP и локального Postfix. Появилась автоматизированная установка через инструмент proxmox-auto-install-assistant, что упрощает развертывание на bare-metal. Также улучшены фильтры для синхронизации и ленточного бэкапа, а также добавлен общий обзор задач очистки и сбора мусора по хранилищам. Proxmox Backup Server 3.2 доступна для загрузки, поддерживает обновление через APT и лицензируется под GNU AGPLv3, а платная поддержка начинается от 520 евро за сервер в год.

## Полная статья

VIENNA, Austria – April 25, 2024 – Enterprise software developer Proxmox Server Solutions GmbH (henceforth "Proxmox") today announced version 3.2 of its open-source server backup solution. The enterprise solution for backing up and restoring VMs, containers, and physical hosts supports incremental backups, deduplication, Zstandard compression, and authenticated encryption.

## Enhancements in Proxmox Backup Server 3.2

- **Notification system:**This release introduces a new framework that uses a matcher-based approach to route notifications. It allows users define specific target types as recipients of notifications. Alongside the current local Postfix MTA, supported targets include Gotify servers or remote SMTP servers optionally with SMTP authentication. Notification matchers determine which targets will get notifications for particular events based on predetermined rules. The notification system offers greater flexibility, and precision for defining when, where, and how notifications are sent.
- **Automated and unattended installation:**Proxmox offers a new ‘proxmox-auto-install-assistant’ tool that fully automates the setup process on bare-metal. Automated installation allows for the rapid deployment of Proxmox Backup hosts without the need for manual access to the systems, saving time and reducing the risk of errors. To use this method, an answer file must be prepared with the necessary configuration settings for the installation process. This file can be provided directly in the ISO, on an additional disk such as a USB flash drive, or over the network. Automated installation is useful in various scenarios, such as deploying large-scale infrastructure, automating the setup process, and ensuring consistent configurations across multiple systems.
- **Exclude backup groups from jobs**: Group filters for sync jobs and for tape backup jobs can now exclude particular backup groups. In order to decide whether a job should process a particular backup group, „include“ filters are processed before the „exclude“ filters.
- **Overview of prune and garbage collection jobs:**A new tab in the “Datastore” summary panel shows defined prune and garbage collection jobs for all datastores. This allows to quickly assess whether all datastores are correctly set up to regularly run important maintenance tasks. It also offers more flexibility in adjusting the times and avoiding the simultaneous execution of two or more garbage collection jobs, which could lead to unwanted hardware utilization overload.

Proxmox Backup Server seamlessly integrates into Proxmox Virtual Environment – users just need to add the storage for the Proxmox Backup Server as a new storage backup target to Proxmox VE.

### Availability

The centrally managed software stack Proxmox Backup Server 3.2 is now available for download at[https://www.proxmox.com/downloads.](https://www.proxmox.com/downloads)The downloadable ISO image can be quickly installed on bare-metal using the installation wizard. Distribution upgrades from older versions of Proxmox Backup Server are possible via APT. It is also possible to install Proxmox Backup Server on top of Debian.

License: Proxmox Backup Server is free and open-source software, published under the GNU AGPLv3.

Support Subscriptions: For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 520 per server per year, including unlimited backup storage and unlimited backup clients.

##### Further information:

- Release Notes:[https://pbs.proxmox.com/wiki/index.php/Roadmap](https://pbs.proxmox.com/wiki/index.php/Roadmap#Proxmox_Backup_Server_3.2)
Forum announcement:[https://forum.proxmox.com/](https://forum.proxmox.com/threads/proxmox-backup-server-3-2-available.145816/)

###

**About Proxmox Backup Server**
Proxmox Backup Server is an enterprise backup solution for backing up and restoring virtual machines, containers, and physical hosts. The open-source client-server software supports incremental backups, deduplication, Zstandard compression, and authenticated encryption. To increase productivity, the easy-to-use Proxmox Backup Server allows you to back up your data in a space-efficient manner, restore VMs, archives or single objects in a flash. With the web-based user interface, you can effectively reduce work hours thanks to simplified management. Proxmox Backup Server is licensed under the GNU AGPLv3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions GmbH**
Proxmox provides powerful and user-friendly open-source server software. Enterprises of all sizes and industries use Proxmox solutions to deploy efficient and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support, training services, and an extensive partner ecosystem to ensure business continuity for its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-3-2
