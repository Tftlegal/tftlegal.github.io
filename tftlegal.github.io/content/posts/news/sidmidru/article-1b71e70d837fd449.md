---
title: "Openmediavault 8 install debian 13"
date: 2026-07-11T05:34:16Z
source: "sidmidru"
original_url: "https://sidmid.ru/openmediavault-8-install-debian-13/"
summary: "OpenMediaVault 8 представляет собой платформу для создания сетевых хранилищ NAS.   Версия 8 устанавливается только на Debian 13, а версия 7 — на Debian 12.   Платформа имеет веб-интерфейс и поддерживает множество сетевых сервисов и протоколов.   В примере настройки DNS выполняется с помощью команды resolvectl для конкретного сетевого интерфейса.   Текст описывает установку OpenMediaVault 8 на систему Debian 13."
---

# Openmediavault 8 install debian 13

## Краткое содержание

OpenMediaVault 8 представляет собой платформу для создания сетевых хранилищ NAS.  
Версия 8 устанавливается только на Debian 13, а версия 7 — на Debian 12.  
Платформа имеет веб-интерфейс и поддерживает множество сетевых сервисов и протоколов.  
В примере настройки DNS выполняется с помощью команды resolvectl для конкретного сетевого интерфейса.  
Текст описывает установку OpenMediaVault 8 на систему Debian 13.

## Полная статья

Thank you for reading this post, don't forget to subscribe!

8 версия может быть установлена только на debian 13
7 версия на debian 12

платформа для создания сетевых хранилищ OpenMediaVault 8.0

Обновлённая версия решения позволяет развернуть сетевое хранилище (NAS, Network-Attached Storage). Платформа имеет веб-интерфейс и поддерживает множество сетевых сервисов и протоколов

https://docs.openmediavault.org/en/8.x/installation/on_debian.html

|  | apt-get install --yes systemd-resolved psmiscsystemctl enable --now systemd-resolved.servicesystemctl restart systemd-resolved.serviceresolvectl dns <INTERFACE> <DNS_SERVER_IP> |
| --- | --- |

в моём случае это:
resolvectl dns <INTERFACE> <DNS_SERVER_IP>
root@openmediavault:~# resolvectl dns enp0s3 8.8.8.8

|  | apt-get install --yes gnupgwget --quiet --output-document=- https://packages.openmediavault.org/public/archive.key | gpg --dearmor --yes --output "/usr/share/keyrings/openmediavault-archive-keyring.gpg" |
| --- | --- |

|  | cat <<EOF >> /etc/apt/sources.list.d/openmediavault.listdeb [signed-by=/usr/share/keyrings/openmediavault-archive-keyring.gpg] https://packages.openmediavault.org/public synchrony maindeb [signed-by=/usr/share/keyrings/openmediavault-archive-keyring.gpg] https://downloads.sourceforge.net/project/openmediavault/packages synchrony main# Uncomment the following line to add software from the proposed repository.deb [signed-by=/usr/share/keyrings/openmediavault-archive-keyring.gpg] https://packages.openmediavault.org/public synchrony-proposed maindeb [signed-by=/usr/share/keyrings/openmediavault-archive-keyring.gpg] https://downloads.sourceforge.net/project/openmediavault/packages synchrony-proposed main# This software is not part of OpenMediaVault, but is offered by third-party# developers as a service to OpenMediaVault users.deb [signed-by=/usr/share/keyrings/openmediavault-archive-keyring.gpg] https://packages.openmediavault.org/public synchrony partnerdeb [signed-by=/usr/share/keyrings/openmediavault-archive-keyring.gpg] https://downloads.sourceforge.net/project/openmediavault/packages synchrony partnerEOF |
| --- | --- |

ставим:

|  | export LANG=C.UTF-8export DEBIAN_FRONTEND=noninteractiveexport APT_LISTCHANGES_FRONTEND=noneapt-get updateapt-get --yes --auto-remove --show-upgraded \ --allow-downgrades --allow-change-held-packages \ --no-install-recommends \ --option DPkg::Options::="--force-confdef" \ --option DPkg::Options::="--force-confold" \ install openmediavault |
| --- | --- |

|  | omv-salt deploy run systemd-networkd |
| --- | --- |

всё можно заходить:

http://192.168.1.155/

Логин:**admin**
Пароль:**openmediavault**

![](/news/sidmidru/article-1b71e70d837fd449/image-01.png)

далее ставим плагины:

https://github.com/OpenMediaVault-Plugin-Developers/packages/

**wget -O - https://github.com/OpenMediaVault-Plugin-Developers/packages/raw/master/install | bash**

чтобы появился список плагинов делаем следующее:

**omv-upgrade**
**omv-mkaptidx**
**systemctl restart openmediavault-engined nginx**

проверяем что плагины стали доступны:

![](/news/sidmidru/article-1b71e70d837fd449/image-02.png)

чтобы собрать raid массив нужно поставить плагин**openmediavault-md**

дальше рейд собирается в панели в меню**Multiple Device**

![](/news/sidmidru/article-1b71e70d837fd449/image-03.png)

![](/news/sidmidru/article-1b71e70d837fd449/image-04.png)

![](/news/sidmidru/article-1b71e70d837fd449/image-05.png)

посмотрим информацию по этому raid

![](/news/sidmidru/article-1b71e70d837fd449/image-06.png)

![](/news/sidmidru/article-1b71e70d837fd449/image-07.png)

создадим файловую систему в нашем raid

![](/news/sidmidru/article-1b71e70d837fd449/image-08.png)

![](/news/sidmidru/article-1b71e70d837fd449/image-09.png)

теперь смонтируем это устройство:

![](/news/sidmidru/article-1b71e70d837fd449/image-10.png)

![](/news/sidmidru/article-1b71e70d837fd449/image-11.png)

![](/news/sidmidru/article-1b71e70d837fd449/image-12.png)

по сути всё, дальше можно создавать общие каталоги:

![](/news/sidmidru/article-1b71e70d837fd449/image-13.png)

![](/news/sidmidru/article-1b71e70d837fd449/image-14.png)

![](/news/sidmidru/article-1b71e70d837fd449/image-15.png)

на диске это выглядит вот так:

| 123456789101112131415161718192021222324252627282930 | root@openmediavault:~# df -hFilesystem Size Used Avail Use% Mounted onudev 1.9G 0 1.9G 0% /devtmpfs 393M 756K 392M 1% /run/dev/mapper/debian--vg-root 38G 2.7G 34G 8% /tmpfs 2.0G 84K 2.0G 1% /dev/shmtmpfs 5.0M 0 5.0M 0% /run/locktmpfs 1.0M 0 1.0M 0% /run/credentials/systemd-journald.servicetmpfs 1.0M 0 1.0M 0% /run/credentials/systemd-resolved.servicetmpfs 2.0G 308K 2.0G 1% /tmp/dev/sda1 455M 210M 221M 49% /boottmpfs 1.0M 0 1.0M 0% /run/credentials/systemd-networkd.servicetmpfs 1.0M 0 1.0M 0% /run/credentials/getty@tty1.servicetmpfs 393M 8.0K 393M 1% /run/user/0/dev/md0 20G 2.1M 20G 1% /srv/dev-disk-by-uuid-47085215-ad97-4fb8-bcb0-7581ef0f3ef6root@openmediavault:~# ls -lah /srv/dev-disk-by-uuid-47085215-ad97-4fb8-bcb0-7581ef0f3ef6/total 32Kdrwxr-xr-x 5 root root 4.0K Jan 7 16:50 .drwxr-xr-x 5 root root 4.0K Jan 7 16:42 ..drwx------ 2 root root 16K Jan 7 16:37 lost+founddrwxrwsrwx 2 root users 4.0K Jan 7 16:44 testdrwxrws--- 2 root users 4.0K Jan 7 16:50 test2root@openmediavault:~# ls -lah /srv/dev-disk-by-uuid-47085215-ad97-4fb8-bcb0-7581ef0f3ef6/test2/total 8.0Kdrwxrws--- 2 root users 4.0K Jan 7 16:50 .drwxr-xr-x 5 root root 4.0K Jan 7 16:50 .. |
| --- | --- |

далее можно к этому каталогу примонтировать nfs

![](/news/sidmidru/article-1b71e70d837fd449/image-16.png)

![](/news/sidmidru/article-1b71e70d837fd449/image-17.png)

![](/news/sidmidru/article-1b71e70d837fd449/image-18.png)

![](/news/sidmidru/article-1b71e70d837fd449/image-19.png)

проверяем что nfs нормально прописался:

|  | root@openmediavault:~# cat /etc/exports # This file is auto-generated by openmediavault (https://www.openmediavault.org)# WARNING: Do not edit this file, your changes will get lost.# /etc/exports: the access control list for filesystems which may be exported# to NFS clients. See exports(5)./export/test 192.168.1.0/24(fsid=f7c6fefa-0846-45a4-ac11-584bbcec7535,rw,subtree_check,insecure)/export 192.168.1.0/24(ro,fsid=0,root_squash,subtree_check,insecure) |
| --- | --- |

всё как видим можем из подсети спокойно монтировать устройства.

сбросить пароль можно через консольную команду**omv-firstaid**

я сбросил на**admin admin**

## Навигация по записям

## https://github.com/midnight47/

## Оригинал

https://sidmid.ru/openmediavault-8-install-debian-13/
