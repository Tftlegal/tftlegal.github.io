---
title: "Proxmox Backup Server 4.0 available"
date: 2025-08-06T09:58:26Z
source: "proxmoxcom"
original_url: "https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-4-0"
summary: "<p><strong>VIENNA, Austria – August 6, 2025 –</strong> Leading open-source server solutions provider Proxmox Server Solutions GmbH (henceforth \"Proxmox\"), celebrating its 20th year of innovation, today announced the release of Proxmox Backup Server 4.0. Main highlight of this update is a modernized core built upon Debian 13 “Trixie”, ensuring a robust foundation for the platform, as well as expanding its backup options by introducing native support for S3-compatible object storage.</p> <h2>Highlights in Proxmox Backup Server 4.0</h2> <p class=\"heading4\">Based on Debian 13 “Trixie”</p> <p>This core update is based on Debian 13 “Trixie”, bringing the latest Debian release as foundation for Proxmox Backup Server including newer packages, improved hardware support, and enhanced security. Proxmox Backup Server is using a newer Linux kernel 6.14 as stable default enhancing hardware compatibility and performance, and includes ZFS 2.3.3.</p> <p class=\"heading4\">Support for S3-compatible object storage</p> <p>This version introduces native support for S3-compatible object storage as backend for backups. By supporting the S3 API, Proxmox Backup Server enables users to access highly scalable and cost-effective public and private cloud environments for storing backup data. To improve performance and reduce cost, Proxmox Backup Server leverages a local cache and minimizes API calls to the S3 backend by internally retaining frequently used backup metadata and data chunks. While each S3 datastore is exclusively managed by a single Proxmox Backup Server instance, the underlying object storage’s contents remain reusable if the original instance goes offline, ensuring robust disaster recovery capabilities.</p> <p class=\"heading4\">Live RAIDZ expansion</p> <p>Proxmox Backup Server comes with ZFS 2.3.3 which now allows to expand existing RAIDZ pools with minimal downtime. This enhancement enables organizations to scale their ZFS storage on demand, adding capacity as needs grow without upfront over-provisioning or future disruptions. The ability to incrementally add new drives directly to an existing RAIDZ virtual device (vdev) reduces costs, ensures the continuous availability of data, and significantly increases flexibility.</p> <p class=\"heading4\">Automatic sync jobs for removable datastores</p> <p>This version introduces a significant enhancement for backup automation: Users can now configure sync jobs to run automatically each time a relevant removable datastore is mounted. By simply setting the new “run-on-mount” flag on a sync job, Proxmox Backup Server further simplifies critical backup procedures, ensuring data is consistently synchronized with minimal manual intervention.</p> <p>Proxmox Backup Server seamlessly integrates into Proxmox Virtual Environment – users just need to add a datastore of Proxmox Backup Server as a new storage backup target to Proxmox VE.</p> <h3>Availability</h3> <p>Proxmox Backup Server 4.0 is available now for download. The ISO image contains the complete feature-set and can be quickly installed on bare-metal using the installation wizard. Upgrades from Proxmox Backup Server 3 to 4 are supported and a detailed migration guide is available. It is also possible to install Proxmox Backup Server on top of Debian.&nbsp;Proxmox Backup Server is free and open-source software, published under the GNU AGPLv3.</p> <p>Support Subscriptions: For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 540 per server, including unlimited backup storage and unlimited backup-clients.</p> <p>Resources:</p> <ul> <li>ISO Image Download: <a href=\"https://www.proxmox.com//en/downloads\">https://www.proxmox.com/downloads</a></li> <li>Forum Announcement: <a href=\"https://forum.proxmox.com/threads/proxmox-backup-server-4-0-released.169306/\">https://forum.proxmox.com/</a></li> <li>Roadmap: For published and upcoming features, see the <a href=\"https://pbs.proxmox.com/wiki/Roadmap#Proxmox_Backup_Server_4.0\">Release Notes &amp; Roadmap</a></li> </ul> <p style=\"text-align: center;\">###</p> <p><strong>About Proxmox Backup Server</strong><br />Proxmox Backup Server is an enterprise backup solution for backing up and restoring virtual machines, containers, and physical hosts. The open-source client-server software supports incremental backups, deduplication, Zstandard compression, and authenticated encryption. To increase productivity, the easy-to-use Proxmox Backup Server allows you to back up your data in a space-efficient manner, restore VMs, archives or single objects in a flash. With the web-based user interface, you can effectively reduce work hours thanks to simplified management. Proxmox Backup Server is licensed under the GNU AGPLv3. Enterprise support subscriptions are available from Proxmox.</p> <p><strong>About Proxmox Server Solutions</strong><br />Proxmox provides powerful and user-friendly open-source server software. For 20 years, enterprises of all sizes and industries use the Proxmox solutions to deploy efficient and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support, training services, and an extensive partner ecosystem to ensure business continuity for its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.</p> <p>Contact:&nbsp;Daniela Häsler, Proxmox Server Solutions GmbH, marketing@proxmox.com</p>"
---

# Proxmox Backup Server 4.0 available

## Краткое содержание

<p><strong>VIENNA, Austria – August 6, 2025 –</strong> Leading open-source server solutions provider Proxmox Server Solutions GmbH (henceforth "Proxmox"), celebrating its 20th year of innovation, today announced the release of Proxmox Backup Server 4.0. Main highlight of this update is a modernized core built upon Debian 13 “Trixie”, ensuring a robust foundation for the platform, as well as expanding its backup options by introducing native support for S3-compatible object storage.</p>
<h2>Highlights in Proxmox Backup Server 4.0</h2>
<p class="heading4">Based on Debian 13 “Trixie”</p>
<p>This core update is based on Debian 13 “Trixie”, bringing the latest Debian release as foundation for Proxmox Backup Server including newer packages, improved hardware support, and enhanced security. Proxmox Backup Server is using a newer Linux kernel 6.14 as stable default enhancing hardware compatibility and performance, and includes ZFS 2.3.3.</p>
<p class="heading4">Support for S3-compatible object storage</p>
<p>This version introduces native support for S3-compatible object storage as backend for backups. By supporting the S3 API, Proxmox Backup Server enables users to access highly scalable and cost-effective public and private cloud environments for storing backup data. To improve performance and reduce cost, Proxmox Backup Server leverages a local cache and minimizes API calls to the S3 backend by internally retaining frequently used backup metadata and data chunks. While each S3 datastore is exclusively managed by a single Proxmox Backup Server instance, the underlying object storage’s contents remain reusable if the original instance goes offline, ensuring robust disaster recovery capabilities.</p>
<p class="heading4">Live RAIDZ expansion</p>
<p>Proxmox Backup Server comes with ZFS 2.3.3 which now allows to expand existing RAIDZ pools with minimal downtime. This enhancement enables organizations to scale their ZFS storage on demand, adding capacity as needs grow without upfront over-provisioning or future disruptions. The ability to incrementally add new drives directly to an existing RAIDZ virtual device (vdev) reduces costs, ensures the continuous availability of data, and significantly increases flexibility.</p>
<p class="heading4">Automatic sync jobs for removable datastores</p>
<p>This version introduces a significant enhancement for backup automation: Users can now configure sync jobs to run automatically each time a relevant removable datastore is mounted. By simply setting the new “run-on-mount” flag on a sync job, Proxmox Backup Server further simplifies critical backup procedures, ensuring data is consistently synchronized with minimal manual intervention.</p>
<p>Proxmox Backup Server seamlessly integrates into Proxmox Virtual Environment – users just need to add a datastore of Proxmox Backup Server as a new storage backup target to Proxmox VE.</p>
<h3>Availability</h3>
<p>Proxmox Backup Server 4.0 is available now for download. The ISO image contains the complete feature-set and can be quickly installed on bare-metal using the installation wizard. Upgrades from Proxmox Backup Server 3 to 4 are supported and a detailed migration guide is available. It is also possible to install Proxmox Backup Server on top of Debian.&nbsp;Proxmox Backup Server is free and open-source software, published under the GNU AGPLv3.</p>
<p>Support Subscriptions: For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 540 per server, including unlimited backup storage and unlimited backup-clients.</p>
<p>Resources:</p>
<ul>
<li>ISO Image Download: <a href="https://www.proxmox.com//en/downloads">https://www.proxmox.com/downloads</a></li>
<li>Forum Announcement: <a href="https://forum.proxmox.com/threads/proxmox-backup-server-4-0-released.169306/">https://forum.proxmox.com/</a></li>
<li>Roadmap: For published and upcoming features, see the <a href="https://pbs.proxmox.com/wiki/Roadmap#Proxmox_Backup_Server_4.0">Release Notes &amp; Roadmap</a></li>
</ul>
<p style="text-align: center;">###</p>
<p><strong>About Proxmox Backup Server</strong><br />Proxmox Backup Server is an enterprise backup solution for backing up and restoring virtual machines, containers, and physical hosts. The open-source client-server software supports incremental backups, deduplication, Zstandard compression, and authenticated encryption. To increase productivity, the easy-to-use Proxmox Backup Server allows you to back up your data in a space-efficient manner, restore VMs, archives or single objects in a flash. With the web-based user interface, you can effectively reduce work hours thanks to simplified management. Proxmox Backup Server is licensed under the GNU AGPLv3. Enterprise support subscriptions are available from Proxmox.</p>
<p><strong>About Proxmox Server Solutions</strong><br />Proxmox provides powerful and user-friendly open-source server software. For 20 years, enterprises of all sizes and industries use the Proxmox solutions to deploy efficient and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support, training services, and an extensive partner ecosystem to ensure business continuity for its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.</p>
<p>Contact:&nbsp;Daniela Häsler, Proxmox Server Solutions GmbH, marketing@proxmox.com</p>

## Полная статья

**VIENNA, Austria – August 6, 2025 –**Leading open-source server solutions provider Proxmox Server Solutions GmbH (henceforth "Proxmox"), celebrating its 20th year of innovation, today announced the release of Proxmox Backup Server 4.0. Main highlight of this update is a modernized core built upon Debian 13 “Trixie”, ensuring a robust foundation for the platform, as well as expanding its backup options by introducing native support for S3-compatible object storage.

## Highlights in Proxmox Backup Server 4.0

Based on Debian 13 “Trixie”

This core update is based on Debian 13 “Trixie”, bringing the latest Debian release as foundation for Proxmox Backup Server including newer packages, improved hardware support, and enhanced security. Proxmox Backup Server is using a newer Linux kernel 6.14 as stable default enhancing hardware compatibility and performance, and includes ZFS 2.3.3.

Support for S3-compatible object storage

This version introduces native support for S3-compatible object storage as backend for backups. By supporting the S3 API, Proxmox Backup Server enables users to access highly scalable and cost-effective public and private cloud environments for storing backup data. To improve performance and reduce cost, Proxmox Backup Server leverages a local cache and minimizes API calls to the S3 backend by internally retaining frequently used backup metadata and data chunks. While each S3 datastore is exclusively managed by a single Proxmox Backup Server instance, the underlying object storage’s contents remain reusable if the original instance goes offline, ensuring robust disaster recovery capabilities.

Live RAIDZ expansion

Proxmox Backup Server comes with ZFS 2.3.3 which now allows to expand existing RAIDZ pools with minimal downtime. This enhancement enables organizations to scale their ZFS storage on demand, adding capacity as needs grow without upfront over-provisioning or future disruptions. The ability to incrementally add new drives directly to an existing RAIDZ virtual device (vdev) reduces costs, ensures the continuous availability of data, and significantly increases flexibility.

Automatic sync jobs for removable datastores

This version introduces a significant enhancement for backup automation: Users can now configure sync jobs to run automatically each time a relevant removable datastore is mounted. By simply setting the new “run-on-mount” flag on a sync job, Proxmox Backup Server further simplifies critical backup procedures, ensuring data is consistently synchronized with minimal manual intervention.

Proxmox Backup Server seamlessly integrates into Proxmox Virtual Environment – users just need to add a datastore of Proxmox Backup Server as a new storage backup target to Proxmox VE.

### Availability

Proxmox Backup Server 4.0 is available now for download. The ISO image contains the complete feature-set and can be quickly installed on bare-metal using the installation wizard. Upgrades from Proxmox Backup Server 3 to 4 are supported and a detailed migration guide is available. It is also possible to install Proxmox Backup Server on top of Debian. Proxmox Backup Server is free and open-source software, published under the GNU AGPLv3.

Support Subscriptions: For enterprise users, Proxmox Server Solutions GmbH offers a subscription-based support model, which provides access to the extensively tested Enterprise Repository, with regular updates via the web interface, as well as technical support on a subscription basis. Prices start at EUR 540 per server, including unlimited backup storage and unlimited backup-clients.

Resources:

- ISO Image Download:[https://www.proxmox.com/downloads](https://www.proxmox.com/en/downloads)
- Forum Announcement:[https://forum.proxmox.com/](https://forum.proxmox.com/threads/proxmox-backup-server-4-0-released.169306/)
- Roadmap: For published and upcoming features, see the[Release Notes & Roadmap](https://pbs.proxmox.com/wiki/Roadmap#Proxmox_Backup_Server_4.0)

###

**About Proxmox Backup Server**
Proxmox Backup Server is an enterprise backup solution for backing up and restoring virtual machines, containers, and physical hosts. The open-source client-server software supports incremental backups, deduplication, Zstandard compression, and authenticated encryption. To increase productivity, the easy-to-use Proxmox Backup Server allows you to back up your data in a space-efficient manner, restore VMs, archives or single objects in a flash. With the web-based user interface, you can effectively reduce work hours thanks to simplified management. Proxmox Backup Server is licensed under the GNU AGPLv3. Enterprise support subscriptions are available from Proxmox.

**About Proxmox Server Solutions**
Proxmox provides powerful and user-friendly open-source server software. For 20 years, enterprises of all sizes and industries use the Proxmox solutions to deploy efficient and simplified IT infrastructures, minimize total cost of ownership, and avoid vendor lock-in. Proxmox also offers commercial support, training services, and an extensive partner ecosystem to ensure business continuity for its customers. Proxmox Server Solutions GmbH was established in 2005 and is headquartered in Vienna, Austria.

Contact: Daniela Häsler, Proxmox Server Solutions GmbH, marketing@proxmox.com

## Оригинал

https://www.proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-4-0
