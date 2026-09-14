---
title: "Proxmox Backup Server 3.0 with Debian 12 “Bookworm” available"
date: 2023-07-10T12:58:58Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-3-0"
summary: "Proxmox представила крупное обновление Proxmox Backup Server 3.0 — открытое решение для резервного копирования серверов. Новая версия построена на Debian 12 «Bookworm», использует ядро Linux 6.2 и ZFS 2.1.12, а также предлагает проверенный путь обновления с 2.x. Среди улучшений — более удобное управление ленточными носителями, гибкая синхронизация с параметром «transfer-last», блокировка учетных записей при повторных неудачах TFA/TOTP и текстовый интерфейс установщика. Система поддерживает инкрементальные дедуплицированные бэкапы виртуальных машин"
---

# Proxmox Backup Server 3.0 with Debian 12 “Bookworm” available

## Краткое содержание

Proxmox представила крупное обновление Proxmox Backup Server 3.0 — открытое решение для резервного копирования серверов. Новая версия построена на Debian 12 «Bookworm», использует ядро Linux 6.2 и ZFS 2.1.12, а также предлагает проверенный путь обновления с 2.x. Среди улучшений — более удобное управление ленточными носителями, гибкая синхронизация с параметром «transfer-last», блокировка учетных записей при повторных неудачах TFA/TOTP и текстовый интерфейс установщика. Система поддерживает инкрементальные дедуплицированные бэкапы виртуальных машин

## Полная статья

**VIENNA, Austria – June 28, 2023 –**Enterprise software developer Proxmox Server Solutions GmbH (henceforth "Proxmox") today announced a new major release 3.0 of its open-source server backup solution. Proxmox Backup Server is based on Debian 12 “Bookworm”, but uses the newer Linux kernel 6.2 and includes ZFS 2.1.12. For users of the Proxmox Backup Server 2.x family, there is an extensively tested and well-documented upgrade path available to enable a smooth upgrade from 2.x to 3.

## Further enhancements in Proxmox Backup Server 3.0

- **Countless improvements for tape handling:**Including the tape backup/restore tasks in the in the “task-summary” of the GUI dashboard provides a better overview. When restoring a single snapshot, the Proxmox Backup solution now shows a list of the required tapes. When restoring backups, the task does not abort the job if a tape is missing in the changer but waits for the correct tape to be inserted.
- **Flexible synchronisation:**On the client side, a “transfer-last” parameter for sync jobs is now supported. This provides more flexibility by making it possible to specify the number of most recent backups to be transferred.
- **Secure lockout for TFA/TOTP:**To further improve security, user accounts with repeatedly failed login attempts—failing the second factor authentication—are locked out. This protects against attacks where the user password is obtained and a brute-force guess is attempted on the second factor. With a recovery key, or manually by an administrator, the user account can be unlocked again.
- **Text-based user interface (TUI)**for the installer ISO: A text-based user interface has been added and can now be used optionally to gather all required information. This addresses potential issues when launching the GTK-based graphical installer on very new and rather old hardware.

The enterprise backup solution, capable of backing up and restoring VMs, containers, and physical hosts, supports incremental, fully deduplicated backups, significantly reducing network load, required storage space and, as a result, total cost of infrastructure. With the integrated web interface, users can easily manage and monitor all backup tasks from a single pane of glass and quickly recover single files, entire VMs, and archives in case of a disaster.

Proxmox Backup Server seamlessly integrates into Proxmox Virtual Environment—users just need to add the storage for the Proxmox Backup Server as a new storage backup target to Proxmox VE.

### Availability

Proxmox Backup Server is free and open-source software, published under the GNU AGPL, v3. The downloadable ISO image can be quickly installed on bare-metal using the installation wizard. The centrally managed software stack Proxmox Backup Server 3.0 is now available for download at[https://www.proxmox.com/downloads](https://www.proxmox.com/downloads)

Seamless[upgrade instructions from Proxmox Backup Server 2 to 3](https://pbs.proxmox.com/wiki/index.php/Upgrade_from_2_to_3)are available. It is also possible to install Proxmox Backup Server on top of Debian.

For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 495 per server, including unlimited backup storage and unlimited backup-clients.

**About Proxmox Server Solutions**
Proxmox is a provider of powerful yet easy-to-use open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy efficient, agile and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training services to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

**About Proxmox Backup Server**
Proxmox Backup Server is an enterprise backup solution for backing up and restoring virtual machines, containers, and physical hosts. The open-source client-server software supports incremental backups, deduplication, Zstandard compression, and authenticated encryption. To increase productivity, the easy-to-use Proxmox Backup Server allows you to back up your data in a space-efficient manner, restore VMs, archives or single objects in a flash. With the web-based user interface, you can effectively reduce work hours thanks to simplified management. Proxmox Backup Server is licensed under the GNU AGPL, v3. Enterprise support subscriptions are available from Proxmox.

**Contact**
Daniela Häsler, Proxmox Server Solutions GmbH

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-3-0
