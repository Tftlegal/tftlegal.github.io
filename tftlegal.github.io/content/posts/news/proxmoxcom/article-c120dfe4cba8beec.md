---
title: "Proxmox Backup Server 1.1 available with Tape Backup"
date: 2021-03-22T18:08:00Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-1-1-available"
summary: "Proxmox анонсировал выпуск Proxmox Backup Server 1.1 — корпоративного решения для резервного копирования виртуальных машин, контейнеров и физических хостов.   Новая версия построена на Debian Buster 10.9 с ядром 5.4.106, включает ZFS 2.0, а весь программный стек написан на Rust.   В релизе представлен технологический предпросмотр LTO-ленточного бэкапа с поддержкой LTO-4 и новее, аппаратного шифрования, автозагрузчиков и гибких политик хранения.   Также добавлена двухфакторная аутентификация для веб-интерфейса, включая TOTP, WebAuthn и одноразовые recovery keys.   Обновление улучшило производительность и надежность за счет HTTP-сжатия, ZIP-архивов для файловых бэкапов, исправленных ошибок и улучшенной обработки сетевых и дисковых проблем.   Proxmox Backup Server 1.1 распространяется как бесплатное open-source ПО под GNU AGPL v3, а Proxmox предлагает платную подписку на поддержку и enterprise-репозиторий."
---

# Proxmox Backup Server 1.1 available with Tape Backup

## Краткое содержание

Proxmox анонсировал выпуск Proxmox Backup Server 1.1 — корпоративного решения для резервного копирования виртуальных машин, контейнеров и физических хостов.  
Новая версия построена на Debian Buster 10.9 с ядром 5.4.106, включает ZFS 2.0, а весь программный стек написан на Rust.  
В релизе представлен технологический предпросмотр LTO-ленточного бэкапа с поддержкой LTO-4 и новее, аппаратного шифрования, автозагрузчиков и гибких политик хранения.  
Также добавлена двухфакторная аутентификация для веб-интерфейса, включая TOTP, WebAuthn и одноразовые recovery keys.  
Обновление улучшило производительность и надежность за счет HTTP-сжатия, ZIP-архивов для файловых бэкапов, исправленных ошибок и улучшенной обработки сетевых и дисковых проблем.  
Proxmox Backup Server 1.1 распространяется как бесплатное open-source ПО под GNU AGPL v3, а Proxmox предлагает платную подписку на поддержку и enterprise-репозиторий.

## Полная статья

**VIENNA, Austria – April 15, 2021 –**Enterprise software developer Proxmox Server Solutions GmbH (henceforth "Proxmox") has today announced the release of Proxmox Backup Server 1.1. The new version is based on Debian Buster 10.9, but using a newer, long-term supported Linux kernel (5.4.106), and including ZFS 2.0. The whole software stack is written in Rust–a modern, fast, and memory-efficient language.

Proxmox Backup Server is an enterprise backup solution to back up and restore virtual machines, containers, and physical hosts. The client-server software supports incremental, fully deduplicated backups, and significantly reduces network load, required storage space and, as a result, total infrastructure cost. It seamlessly integrates into the virtualization management platform Proxmox Virtual Environment (Proxmox VE), allowing users to simply add a server as a new storage target.

### Proxmox Tape Backup System (Technology Preview)

The new Proxmox tape backup, available as a technology preview in Proxmox Backup Server 1.1, provides an easy way to copy datastore content to tapes. Despite its age, digital magnetic tape continues to provide an easy and economical way for large amounts of data to be archived. Be it for compliance reasons or peace of mind, tape backup makes sense in any effective enterprise backup plan. The existence of WORM (write once read many) tapes, the low cost per unit, and the fast sequential write speeds are all clear benefits of tape based storage.

Proxmox Backup Server supports all Linear Tape-Open generation drives (LTO-4 and later), thus, it includes hardware encryption. Further details of what the Proxmox tape backup system offers are:

- Tape backup jobs, which back up datastores to a media pool. Multiple datastores can be backed up to the same media set. It is possible to choose between writing all snapshots of a datastore to the media set or only the latest snapshot per group.
- Tape restore jobs restore the contents of a media set to one or more datastores. By doing so, it is possible to restore multiple datastores from a media set, even if the system does not have the free disk space required in a single datastore.
- Flexible retention policies such as: always recycle tapes, never recycle tapes, recycle tapes after a particular calendar event, etc.
- New user space tape driver; written in Rust.
- Support for various tape autoloaders: The Proxmox developers have rewritten the mtx tool in Rust (now called pmtx), thus most autoloaders supported by other tape-backup solutions available on Linux will work with Proxmox Backup Server.
- If a stand-alone tape drive without an attached changer is deployed, users are notified via e-mail about necessary operations such as load/unload.
- Components, jobs, and schedules can be configured via the web-based user interface.
- With the Proxmox LTO Barcode Label Generator, a small web-app, users can generate and print bar-code labels for their tapes, on standard, adhesive label sheets.

### Two-factor authentication (TFA) for the GUI

With Proxmox Backup Server 1.1, users can now set up Two-Factor Authentication (TFA) which adds another layer of security to the authentication process, making it harder for cybercriminals to access the network and take over accounts. The following second factors are available:

- Time-base One-Time Password (TOTP) for clients such as FreeOTP, Google Authenticator, etc.
- Web Authentication (WebAuthn): This is implemented by various security devices, like hardware keys or trusted platform modules (TPM) of computers and mobile phones.
- Recovery keys for single use.

The two-factor authentication can be activated or configured via the GUI, by the users themselves or by a system administrator. TFA is complemented by the existing, token-based authentication for granting access to Proxmox Backup Server resources, for example, when configuring a Proxmox Backup Server storage in a Proxmox VE set up.

### HTTP compression via Content-Encoding

Responses from the API of Proxmox Backup Server can get quite large, but can generally be compressed well. By adding support for deflate Content-Encoding, bandwidth utilization, transfer speed, and response times have been improved, especially over bandwidth constricted links.

### Compression of file-level ZIP archive downloads

Downloading a directory from a file-level backup will now produce a compressed ZIP archive, reducing bandwidth and local space required.

### Notable enhancements and bug fixes

Following the massive increase in deployments after the first stable release (1.0), a couple of bugs were reported by the community, which have now been fixed:

- Improved handling of POSIX ACL entries on files.
- Improved hand-over to new process when upgrading the Proxmox Backup Server packages.
- Use of the local file system to handle synchronization, in order to avoid issues with locking on remote file systems (CIFS/NFS).
- Changed HTTP timeouts to work more robustly, even over high latency and low bandwidth links, which are not uncommon for a remote backup site.
- Error handling during garbage-collection has been improved, to handle cases wherein there is no more free space on a datastore file system.
- Improved user experience when using a GPG master key.

### Availability

Proxmox Backup Server is free and open-source software, published under the GNU Affero General Public License, v3. The downloadable ISO image can be installed on bare-metal. For a production setup, dedicated hardware is recommended. Proxmox Backup Server 1.1 is now available for download at[https://www.proxmox.com/downloads](https://www.proxmox.com/en/component/zoo/?view=frontpage&layout=frontpage&Itemid=120)

The backup server from Proxmox comes stocked with all the essential management tools, as well as an easy-to-use, web-based user interface. This allows for simple, out-of-the-box management of the server, either through the command line or a standard web browser.

For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 449 per server, including unlimited backup storage and unlimited backup-clients.

**Forum Announcement**
[https://forum.proxmox.com/threads/proxmox-backup-server-1-1.87687](https://forum.proxmox.com/threads/proxmox-backup-server-1-1.87687)

**About Proxmox Backup Server**

Proxmox Backup Server is an enterprise backup solution for backing up and restoring virtual machines, containers, and physical hosts. The open-source client-server software supports incremental backups, deduplication, Zstandard compression, and authenticated encryption. To increase productivity, the easy-to-use Proxmox Backup Server allows you to back up your data in a space-efficient manner, restore VMs, archives or single objects in a flash. With the web-based user interface you can effectively reduce work hours thanks to simplified management. Proxmox Backup Server is licensed under the GNU AGPL, v3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions
**Proxmox is a provider of powerful, yet easy-to-use, open-source server software. Enterprises, regardless of size, sector or industry use the stable, secure, and scalable Proxmox solutions to deploy agile and efficient IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support and training to ensure business continuity to its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-1-1-available
