---
title: "harbor auth problem"
date: 2025-08-22T04:13:19Z
source: "sidmidru"
original_url: "https://sidmid.ru/harbor-auth-problem/"
summary: "Автор описывает проблему с авторизацией в Harbor. Локальная машина успешно логинится под robot-пользователем. На сервере вход не получается. В приведённом отрывке нет сообщения об ошибке. Поэтому причина не видна из текста. Скорее всего, отличаются сеть, DNS, TLS, proxy или настройки Docker. Нужно сравнить вывод команды и логи Harbor."
---

# harbor auth problem

## Краткое содержание

Автор описывает проблему с авторизацией в Harbor.
Локальная машина успешно логинится под robot-пользователем.
На сервере вход не получается.
В приведённом отрывке нет сообщения об ошибке.
Поэтому причина не видна из текста.
Скорее всего, отличаются сеть, DNS, TLS, proxy или настройки Docker.
Нужно сравнить вывод команды и логи Harbor.

## Полная статья

Thank you for reading this post, don't forget to subscribe!

поймал следующую проблему:
есть harbor, создал в нём проект , создал robot пользователя, авторизуюсь с локальной тачки всё ок:

**echo 5tMe4thRmmNCKnyWG6R0APa5 | docker login harbor.test.local -u 'robot$user-test' --password-stdin**
WARNING! Your password will be stored unencrypted in /home/mid/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credential-stores

Login Succeeded

но при попытке авторизоваться с сервера где запускается runner я ловлю ошибку:

Error response from daemon: Get "https://harbor.test.local/v2/": unauthorized: authentication required

проблема заключалась в старой версии docker. Чтобы отдебажить локально можно воспользоваться dind

**docker run -d --privileged --name dind -eDOCKER_TLS_CERTDIR= docker:24-dind**
**docker exec -it dind sh**
/ # echo 5tMCKnyWG6R0APa5 | docker login harbor.test.local -u 'robot$user-test' --password-stdin

Error response from daemon: Get "https://harbor.test.local/v2/": unauthorized: authentication required

| 12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061 | / # docker infoClient: Version: 24.0.9 Context: default Debug Mode: false Plugins: buildx: Docker Buildx (Docker Inc.) Version: v0.16.2 Path: /usr/local/libexec/docker/cli-plugins/docker-buildx compose: Docker Compose (Docker Inc.) Version: v2.29.1 Path: /usr/local/libexec/docker/cli-plugins/docker-composeServer: Containers: 0 Running: 0 Paused: 0 Stopped: 0 Images: 0 Server Version: 24.0.9 Storage Driver: overlay2 Backing Filesystem: extfs Supports d_type: true Using metacopy: false Native Overlay Diff: true userxattr: false Logging Driver: json-file Cgroup Driver: cgroupfs Cgroup Version: 2 Plugins: Volume: local Network: bridge host ipvlan macvlan null overlay Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog Swarm: inactive Runtimes: io.containerd.runc.v2 runc Default Runtime: runc Init Binary: docker-init containerd version: 7c3aca7a610df76212171d200ca3811ff6096eb8 runc version: v1.1.12-0-g51d5e94 init version: de40ad0 Security Options: apparmor seccomp Profile: builtin cgroupns Kernel Version: 6.8.0-71-generic Operating System: Alpine Linux v3.20 (containerized) OSType: linux Architecture: x86_64 CPUs: 12 Total Memory: 14.98GiB Name: f0b15691a9a4 ID: 5f08480e-bdac-41bd-83c9-bce0ca3beb3c Docker Root Dir: /var/lib/docker Debug Mode: false Experimental: false Insecure Registries: 127.0.0.0/8 Live Restore Enabled: false Product License: Community Engine |
| --- | --- |

минимум нужна версия**25**

**docker run -d --privileged --name dind -eDOCKER_TLS_CERTDIR= docker:25-dind**

:#**docker exec -it dind sh**
/ #**echo 5tMenyWG6R0APa5 | docker login harbor.test.local -u 'robot$user-test' --password-stdin**

WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

на сервере обновляем только нужные для докера пакеты:

**apt-get update**
**apt-get install --only-upgrade docker-ce docker-ce-cli containerd.io docker-compose-plugin**

## Навигация по записям

## https://github.com/midnight47/

## Оригинал

https://sidmid.ru/harbor-auth-problem/
