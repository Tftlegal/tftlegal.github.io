---
title: "Установка k8s со всей обвязкой"
date: 2026-07-11T05:38:43Z
source: "sidmidru"
original_url: "https://sidmid.ru/%d1%83%d1%81%d1%82%d0%b0%d0%bd%d0%be%d0%b2%d0%ba%d0%b0-k8s-%d1%81%d0%be-%d0%b2%d1%81%d0%b5%d0%b9-%d0%be%d0%b1%d0%b2%d1%8f%d0%b7%d0%ba%d0%be%d0%b9/"
summary: "Статья посвящена развертыванию Kubernetes на bare metal с полным набором сопутствующих компонентов. Для начала нужно подготовить несколько серверов с Debian и установить Ansible. Затем клонируют репозиторий ansible-playbook.ginerdctlt и размещают его в /etc/ansible. После этого проверяется файл hosts и запускается роль для установки базовых пакетов и пользователей. Весь процесс автоматизируется через Ansible, что упрощает настройку кластера."
---

# Установка k8s со всей обвязкой

## Краткое содержание

Статья посвящена развертыванию Kubernetes на bare metal с полным набором сопутствующих компонентов. Для начала нужно подготовить несколько серверов с Debian и установить Ansible. Затем клонируют репозиторий ansible-playbook.ginerdctlt и размещают его в /etc/ansible. После этого проверяется файл hosts и запускается роль для установки базовых пакетов и пользователей. Весь процесс автоматизируется через Ansible, что упрощает настройку кластера.

## Полная статья

Thank you for reading this post, don't forget to subscribe!

рассмотрим как на bare metal поднять кубер со всей обвязкой - статья будет масштабной.

что нужно подготовить: несколько серверов на debian

устанавливаете себе ansible выкачиваете следующую репку:
**git clone https://github.com/midnight47/ansible-playbook.ginerdctlt**

я её разместил в /etc/ansible

**cd /etc/ansible/**

**cat hosts**

|  | [all_servers]192.168.1.100 # freeipa1192.168.1.101 # freeipa2192.168.1.102 # nexus192.168.1.103 # vault1192.168.1.104 # vault2192.168.1.105 # vault3192.168.1.106 # gitlab192.168.1.107 # gitlab-runner192.168.1.108 # nfs192.168.1.109 # gluster-fs1192.168.1.110 # gluster-fs2192.168.1.118 # s3-minio1192.168.1.119 # s3-minio2 |
| --- | --- |

первым делом я прогоняю роль по установке базовых пакетов / пользователей

[root@ansible ansible]#**ansible-playbook -u root /etc/ansible/playbooks/roles_play/new_server.yml --ask-pass**

1.[Установка Freeipa](#install_freeipa)
2.[Установка Nexus](#install_nexus)
3.[Установка Vault](#install_vault)
4.[Установка gitlab / gitlab-runner(host)](#gitlab)
5.[УстановкаNFS](#nfs)
6.[Установка Glusterfs](#glusterfs)
7.[Установка s3-minio](#s3minio)
9.[Интеграция Freeipa](#freeipa_int)
9.1[Freeipa -> Nexus](#freeipa_nexus)
9.2[Freeipa -> Gitlab](#freeipa_gitlab)
9.3[Freeipa -> Vault](#freeipa_vault)
9.4[Freeipa -> S3-Minio](#freeipa-s3minio)
10.[Установка kubernetes (kubespray - official)](#install_k8s)
10.1[Доступ с локальногоPCдо кластера](#admin_access)
11[Установка дополнительных компонентов](#additional_component)
11.1[Ingress](#install_ingress)
11.1.1[ingress helm installation](#ingress_helm)(предпочтительный вариант)
11.2[Metallb](#install_metallb)
11.2.1[Metallb helm-installation](#metallb_helm)(предпочтительный вариант)
11.3[NFSprovisioner](#nfs_provisioner)
11.4[GlusterFS provisioner](#glusterfs_provisioner)
11.5[Seaweedfs provisioner](#seaweedfs)(есть проблемы с fuse chown, sync victoria-metrics не поднялась)
11.6[monitoring - Prometheus, grafana, alertmanager](#prometheus-stack)
11.7[monitoring - Victoria-metrics, grafana, alertmanager](#Victoria-metrics)(предпочтительный вариант)
11.7.1[Exporter on node not in the k8s](#Exporteronnodenotinthek8s)
11.7.2[VMServiceScrape - for ingress controller](#scrape_ingress)
11.8[Metrics servers](#metrics_server)
11.9[ПравимCOREDNSиLOCALDNSчтобы работал resolv](#coredns).
11.10[Log - elk](#add_elk)
11.11[Log - loki](#loki)
11.11.1[Log - loki (backend s3-minio без Freeipa -LDAP)](#loki-s3-without-ldap)
11.11.2[Log - loki (backend s3-minio c Freeipa -LDAP)](#loki-s3-with-ldap)
11.11.3[Log - Loki - s3 bucket (seaweedfs)](#loki_s3_seaweedf)
11.11.4[Promail (сборщик логов)](#promtail)
11.11.5[Интеграция grafana с loki](#grafana-loki)
12[Vault - auto unseal](#vault-autounseal)
12.1.1[Установка vault в k8s](#vault-k8s)
12.1.2[Установка autounseal для vault в k8s через cronjob](#k8s-cronjob-autounseal-vault)
12.1.3[Настройка tranzit autounseal vault на физических серверах](#tranzit-autounseal)
12.1.4[обновить токен для распечатывая основного vault](#unseal_update_token)
12.2[Интеграция vault и k8s "Vault Secrets Operator"](#vault-k8s)
12.3[Пример с autoreloader после изменения секрета в vault](#examplt-vault-secret-reloader)
13[Аутентификация, авторизация в k8s (SSOв kubernetes через Freeipa)](#sso_k8s)
13.1[Loft](#Loft)(мне не очень понравилось, не рекомендую)
13.2[Dex - dexK8sAuthenticator](#dex)(нормально работает - дёшево и сердито)
13.3[Rancher](#Rancher)(лучше ставить на отдельной виртуалке а не в кластере, много функционала)
13.3.1[Rancher интеграция с Freeipa](#rancher-freeipa)
13.3.2[Rancher подключение к k8s кластеру](#rancher_k8s)
13.3.3[Rancher проверка авторизации для пользователей k8s](#rancher_user_k8s)
13.4[Keycloak](#Keycloak)(сложная штука, у меня толком с ней ничего не завелось)
13.4.1[Интеграция keycloak c Freeipa](#freeipa_keycloak)
13.4.2[Настройка подключения к k8s](#setting_k8s_keycloak)
13.5[Teleport](#Teleport)
13.5.1[установка teleport](#install_teleport)
13.5.2[Подключение кластера Kubernetes к Teleport](#teleport_connect)
13.5.3[интеграция teleport - keycloak](#teleport_keycloak)
14.0.0[Обновление кластера k8s](#k8s_upgrade)
14.0.1[update 1.24->1.25](#update1.24_1.25)
14.0.2[update 1.25->1.26](#update_1.25_1.26)
14.0.3[update 1.26->1.27](#update_1.26_1.27)
14.0.4[update 1.27->1.28](#update_1.27_1.28)
14.0.5[update 1.28->1.29](#update_1.28_1.29)
14.0.6[update 1.29->1.30](#update_1.29_1.30)
14.0.6[update 1.30->1.31](#update_1.30_1.31)
14.0.7[update 1.31->1.32](#update_1.31_1.32)
14.0.8[update 1.32.5 - > 1.32.9](#update_1.32.9)(пока писал статью новые версии вышли)
14.0.9[update 1.32.9 - > 1.33.5](#update1.33.5)(installPLUTO)
14.0.10[update 1.33.5 -> 1.34.1](#update_1.34.1)
15.1[Gitlab in k8s](#gitlab_k8s)(helm chart)
15.2[Gitlab runner in k8s](#Gitlab_runner_k8s)
15.3[Gitlab helm chart с Freeipa](#gitlab_freeipa)
16.1[Резервное копирование etcd в hostPath](#etcd_hostpath)
16.2[Резервное копирование etcd в s3-minio](#etcd_s3-minio)
16.3[Резервное копирование etcd в s3-minio (LDAPenabled)](#backup_s3_minio_ldap)
17[Cert-manager (self-signed - самоподписанный)](#cert-manager)
18[Argocd](#argocd)
18.1[Argocd add user](#argo_user)
18.2[Argocd интеграция с Freeipa](#agrocd_freeipa)
18.3.1[Argocd создание проекта, настройка деплоя](#argocd_example)
18.3.2[Argocd создание проекта, настройка деплоя Helm](#argo_deploy_helm)
18.3.3[Argocd создание проекта из конфиг файла](#argocd_config)
18.3.4[Argocd создание проекта. настройка деплоя Helm когда values и chart в разных репозиториях](#argo_helm_several_repo)
19[Keda](#keda)
20[Patrony](#patrony)(кластер для postgresql)
21[Helm-chart](#helm)
21.1[Создание дефолтного чарта](#chart_default)
21.2[Добавление секретов из vault](#vault_secret_k8s_chart)
21.3[Добавление Topology Spread Constraints](#topology_spread)
21.4[Добавление RollingUpdate](#RollingUpdate)
21.5[Примеры использования affinity/anti-affinity, nodeSelector, taint/tolerations](#affinity_nodeselector)
21.6[Добавление PodDisruptionBudget](#PodDisruptionBudget)
21.7[Ingress+ cert-manager+resources](#ingres_cert_resources)
21.8[ПроверимHPA](#check_hpa)(HorizontalPodAutoscaler)
21.9[Добавим Keda](#add_keda)(есть примеры с логикой скейлингаИЛИ/ И)
21.9.1[Скейлинг с логикойИЛИ](#logic_or)
21.9.2[Скейлинг с логикой И](#logic_and)
21.10[Контейнеры вPOD](#pod_containers)
21.10.1[обычные контейнеры (sidecar)](#sidecar)
21.10.2[init контейнеры](#init_container)
21.10.3[ephemeral Containers (debug) контейнеры](#ephemeral_container)
21.11[Configmap (делаем связку nginx->php-fpm)](#configmap_nginx_php_fpm)
21.12[volume/ephemeral volume/emptyDir](#volumes_k8s)
21.13[job и cronjob](#cron_crojob)
21.14[probe grpc tcp http](#probe_tcp_grpc_http)
21.15[network policy](#network_policy)
21.16[Canary/Blue-green deployment](#canary_blue_green)
21.16.1[Blue-Green](#blue-green)
21.16.2[Canary](#canary)
22[Pod priority class](#pod_priority_class)
22.1[add priorityClassName promtail](#priorityclass_loki)
22.2[add priorityClassName vault-csi-provider](#priority_class_vault)
22.3[add priorityClassName ingress-nginx-controller](#priority_class_ingress)
23[проверимVPAvertical-pod-autoscaler](#VPA)
23.1[изменим дефолтное значение реплик 2 на 1 дляVPAupdater](#vpa_updater_replica)
24[namespace limitrange](#ns_limitrange)(ограничения для ресурсов на уровне namespace)
25[istio+kiali](#istio_kiali)(service mesh)
25.1[установка istio - используем sidecar](#istio_sidecar)
25.2[установка kiali](#kiali)
26.0.1[Замена nginx ingress controller](#change_ingress_controller)(Envoy Gateway)
26.0.2[Установка Envoy Gateway](#install_envoy_gateway)
26.1[Helm chart - gateway вместо ingress](#helm_gateway)
27[Переезд на новую операционку debian 13](#update_OS)
27.1[проблема с переездом kub-master1 etcd](#problem_etcd)
27.2[проблема с переездом kub-master1 сертификаты](#problem_certificate)
27.3[проблемы с переездом kub-master1 локальный kubctl](#problev_local_kubectl)
27.4[проблемы с переездом kub-master1 scheduller](#problem_scheduller)
27.5[проблемы с переездом - worker node а именно - cluster-info](#problem_cluster-info)
27.6[быстрая диагностика проблем](#diagnostic_kub)
27.7[переезд worker-node](#migration-worker-node)
28.1[Резервное копирование etcd в hostPath (без bitnami)](#etcd_backup_bitnami)
28.2[Резервное копирование etcd в s3-minio (без bitnami)](#etcd_s3_backup_without_bitnami)
29.0[Обновление кластера k8s после переезда на debian 13](#update_k8s_debian-13)
29.1[update v1.34.1 -> 1.34.6](#update-1.34.3)
29.2[update v1.34.6 -> 1.35.4](#update_1.35)
29.3[update v1.35.4 -> 1.36.2](#update_1.36)

### **[]Установка Freeipa**

У меня 2 сервера**CENTOS7**с параметрами
2 ядра 4 гб оперативки - этого хватает на процесс установки - меньше оперативки я бы не ставил так как может прийти OOMkill после того как вся установка пройдёт можно и понизить ресурсы до 1 ядра и 2гб оперативки

Подготавливаем для freeipa inventory файл

**cd /etc/ansible/**

**cat hosts**

| 1234567891011121314151617181920 | [freeipa:children]ipa_serversipa_replicasipa_clients[ipa_servers]freeipa-1.test.local ansible_host=192.168.1.100[ipa_replicas]freeipa-2.test.local ansible_host=192.168.1.101[ipa_clients]nexus.test.local ansible_host=192.168.1.102vault1.test.local ansible_host=192.168.1.103vault2.test.local ansible_host=192.168.1.104vault3.test.local ansible_host=192.168.1.105gitlab.test.local ansible_host=192.168.1.106gitlab-runner.test.local ansible_host=192.168.1.107nfs.test.local ansible_host=192.168.1.108gluster-fs1.test.local ansible_host=192.168.1.109gluster-fs2.test.local ansible_host=192.168.1.110 |
| --- | --- |

у нас будет 2 сервера freeipa для отказоустойчивости на них будет dns сервер отвечающий за зону**test.local**

**cat /etc/ansible/playbooks/roles_play/freeipa.yaml**

|  | ---- hosts: freeipa become: yes vars: domain: test.local realm: TEST.LOCAL admin_password: Secret123 ds_password: Secret123 master_password: Secret123 roles: - freeipa_setup |
| --- | --- |

в данном файле мы обязательно задаём**домен****test.local**и**пароль Secret123**

запускаем установку:

[root@ansible ansible]#**ansible-playbook -u root /etc/ansible/playbooks/roles_play/freeipa.yaml --ask-pass**

дожидаемся окончания и проверяем работу:

**https://freeipa-1.test.local/
**
после можем снизить параметры сервера до 1 ядра и 2гб оперативки

### []Установка Nexus

для ноды с nexus нужно минимум 1 ядро и 2гб оперативки

на сервере ansible ставим:

**ansible-galaxy collection install community.general**

переходим в директорию:

/etc/ansible/roles/nexus/files

**cd /etc/ansible/roles/nexus/files**

объединяем архив

**cat jdk-8u371-linux-x64.tar.gz.part_* > jdk-8u371-linux-x64.tar.gz**

**cd /etc/ansible/**

**cat hosts**

|  | [nexus]nexus.test.local ansible_host=192.168.1.102 |
| --- | --- |

указываем пароль и домен:
- domain: nexus.test.local
- password: Secret123

**cat /etc/ansible/playbooks/roles_play/nexus.yml
**

|  | ---- hosts: nexus become: true ignore_errors: yes become_method: sudo gather_facts: yes vars: - domain: nexus.test.local - password: Secret123 roles: - nexus |
| --- | --- |

[root@ansible ansible]#**ansible-playbook -u root /etc/ansible/playbooks/roles_play/nexus.yml --ask-pass**

дожидаемся окончания установки и проверяем работу:

**http://nexus.test.local:8081/**

### []Установка vault

**cd /etc/ansible/
cat hosts**

|  | [vault]vault1.test.local ansible_host=192.168.1.103vault2.test.local ansible_host=192.168.1.104vault3.test.local ansible_host=192.168.1.105[vault:vars]virtual_hostname=vault.test.localvirtual_address=192.168.1.111 |
| --- | --- |

в файле
**/etc/ansible/roles/vault/defaults/main.yaml**

выставляем параметры для сертификата который будет сгенерен и какие там будут домены

| | vault_certificate_vars: country_name: RU locality_name: Some Country organization_name: Test email_address: master@test.local common_name: test.local subject_alt_name: - DNS:*.test.local - DNS:*.dev.test.local - DNS:*.staging.test.local - DNS:*.prod.test.local - DNS:*.infra.test.local - DNS:vault.test.local - DNS:vault1.test.local - DNS:vault2.test.local - DNS:vault3.test.local - IP:127.0.0.1 |
| --- | --- |

если у вас уже есть сертификат (купленный или самподпианный) то положите его в директорию**/etc/ansible/roles/vault/ssl/**назовите его**my_crt_file.crt**так же нужно положить в эту директорию pem файл my_crt_file.pem имя файлов как и директорию можно задать в переменных**localhost_ssl_dir****vault_certificate_file_name**в файле**/etc/ansible/roles/vault/defaults/main.yaml**если этого не сделать то будет сгенерен самоподписанный сертификат

после установки и распечатки волта в директории**/etc/ansible/roles/vault/unseal_tmp**появится рут токен и ключи для распечатки.

запускаем установку:

[root@ansible ansible]#**ansible-playbook -u root /etc/ansible/playbooks/roles_play/vault.yml --ask-pass**

как и писал выше ключи вот тут:

| | [root@ansible ansible]# cat roles/vault/unseal_tmp/rootkey hvs.UuG0QJvRRfwUHUTGxDTjFaAd[root@ansible ansible]# [root@ansible ansible]# cat roles/vault/unseal_tmp/unseal_key_0 fab3991ca411bf9a7bfef1d99131a983ce1906b10f6e47cbdc8da7e44f0b3a3a3e[root@ansible ansible]# [root@ansible ansible]# cat roles/vault/unseal_tmp/unseal_key_17540f4e2e6084813fd23ad98a00257501e6a8be27dbabf2664b1f807a71d1bf9a3[root@ansible ansible]# [root@ansible ansible]# cat roles/vault/unseal_tmp/unseal_key_2261be9452bc5abbe07942e00d5f3d1adfeb705e6a24733ae9fed514f22b81e6a82[root@ansible ansible]# [root@ansible ansible]# cat roles/vault/unseal_tmp/unseal_key_3c0f1027f89631c403814321e3ad764407e3c204a091f0a30ccc9d1e3bb373d2ca4[root@ansible ansible]# [root@ansible ansible]# cat roles/vault/unseal_tmp/unseal_key_49d8faf0c33a784f7019ab24685e22f9f146c7091674bd868f730993bb02c6c7476[root@ansible ansible]# |
| --- | --- |

### []Установка gitlab / gitlab-runner(host)

по ресурсам нужно для установки

4 ядра 4 оперативки
20 гб диска

**/etc/ansible/hosts**

|  | [gitlab:children]gitlab-servergitlab-runner[gitlab-server]gitlab.test.local ansible_host=192.168.1.106[gitlab-runner]gitlab-runner.test.local ansible_host=192.168.1.107 |
| --- | --- |

**/etc/ansible/playbooks/roles_play/gitlab.yml**

если хотим раннер хостовой то ставим переменную в true

|  | vars: - gitlabrunner: true |
| --- | --- |

запускаем установку:

**ansible-playbook -u root /etc/ansible/playbooks/roles_play/gitlab.yml --ask-pass**

дожидаемся конца установки.
в самом конце ансибл покажет пароль:

|  | TASK [gitlab-gitlab-runner : gitlab show PASSWORD] ****************************************************************************************************************************************************************ok: [gitlab.test.local] => { "gitlab_password.stdout_lines": [ "Password: Ukhr+Al9mi9SutCfItn05Q+MtsYLM9HZ7OkoxpUBxk8=" ]} |
| --- | --- |

проверяем:

**http://gitlab.test.local/**
логин:**root**
пароль:**Ukhr+Al9mi9SutCfItn05Q+MtsYLM9HZ7OkoxpUBxk8=**

заходим и меняем пароль. я меняю на**Secret123**

готово, после установки можем откатить ресурсы на 2 ядра и 3,5 гб оперативки

### []УстановкаNFS

по ресурсам хватит 1 ядро и 512 оперативки

можем поменять директорию в которой будут хранится данные на nfs сервере, а на клиенте к какой директории будет подключено хранилище, или вообще не создавать, если хотим использовать как провижинер на k8s:

**/etc/ansible/playbooks/roles_play/nfs.yml**

|  | vars: - dir_nfs_master: /nfs - dir_nfs_client: /nfs-client - k8s_nfs_provision: false # if true nfs client directory will not create |
| --- | --- |

в моём примере на клиентах я буду создавать директории к которым подключено хранилище.

**/etc/ansible/hosts**

|  | [nfs:children]nfsmasternfsclient[nfsmaster]nfs.test.local ansible_host=192.168.1.108[nfsclient]192.168.1.100192.168.1.101 |
| --- | --- |

**ansible-playbook -u root /etc/ansible/playbooks/roles_play/nfs.yml --ask-pass**

дожидаемся окончания и проверяем:

|  | [root@ansible ansible]# ssh 192.168.1.100root@192.168.1.100's password: [root@freeipa-1 ~]# touch /nfs-client/test-file[root@freeipa-1 ~]# logoutConnection to 192.168.1.100 closed.[root@ansible ansible]# ssh 192.168.1.108root@192.168.1.108's password: root@nfs:~# ls -lah /nfs/total 8.0Kdrwxr-xr-x 2 root root 4.0K Jun 22 10:00 .drwxr-xr-x 19 root root 4.0K Jun 22 09:53 ..-rw-r--r-- 1 root root 0 Jun 22 10:00 test-fileroot@nfs:~# |
| --- | --- |

### []УстановкаGLUSTERFS

можем задать директории для сервера и клиентов и имя для glusterfs tom:

**/etc/ansible/playbooks/roles_play/glusterfs.yml**

|  | vars: - dir_gluster_master: /gluster - dir_gluster_client: /gluster-client - name_of_gluster_tom: gluster-tom |
| --- | --- |

**/etc/ansible/hosts**

|  | [glusterfs:children]glustermasterglusterclient[glustermaster]gluster-fs1.test.local ansible_host=192.168.1.109gluster-fs2.test.local ansible_host=192.168.1.110[glusterclient]192.168.1.100192.168.1.101 |
| --- | --- |

если нужно добавить больше серверов то просто докидывайте в glustermaster. Репликафактор равен 2 всегда. Изменить можно тут:
**/etc/ansible/roles/glusterfs/tasks/add-tom.yml**

запускаем установку:

**ansible-playbook -u root /etc/ansible/playbooks/roles_play/glusterfs.yml --ask-pass**

после первого прохождения запустите ещё раз, - возможны проблемы при первом проходе

Дальше проверяем:

| 1234567891011121314151617181920212223242526272829303132333435363738394041 | root@gluster-fs1:~# gluster peer statusNumber of Peers: 1Hostname: gluster-fs2.test.localUuid: 845b3a30-c48c-4048-b851-2c92c162f9feState: Peer in Cluster (Connected)root@gluster-fs1:~# gluster volume infoVolume Name: gluster-tomType: ReplicateVolume ID: ad32a409-1b0f-4bd6-9420-acd93b0e1a14Status: StartedSnapshot Count: 0Number of Bricks: 1 x 2 = 2Transport-type: tcpBricks:Brick1: gluster-fs1.test.local:/gluster/gv01Brick2: gluster-fs2.test.local:/gluster/gv01Options Reconfigured:cluster.granular-entry-heal: onstorage.fips-mode-rchecksum: ontransport.address-family: inetnfs.disable: onperformance.client-io-threads: offroot@gluster-fs1:~# gluster volume statusStatus of volume: gluster-tomGluster process TCP Port RDMA Port Online Pid------------------------------------------------------------------------------Brick gluster-fs1.test.local:/gluster/gv01 49450 0 Y 615 Brick gluster-fs2.test.local:/gluster/gv01 49701 0 Y 9420 Self-heal Daemon on localhost N/A N/A Y 708 Self-heal Daemon on gluster-fs2.test.local N/A N/A Y 9437 Task Status of Volume gluster-tom------------------------------------------------------------------------------There are no active volume tasks |
| --- | --- |

|  | [root@ansible ansible]# ssh 192.168.1.100root@192.168.1.100's password: [root@freeipa-1 ~]# touch /gluster-client/test_file |
| --- | --- |

и на сервере

|  | root@gluster-fs1:~# ls -lah /gluster/gv01/total 24Kdrwxr-xr-x 4 root root 4.0K Jun 22 12:46 .drwxr-xr-x 3 root root 4.0K Jun 22 12:28 ..drw------- 262 root root 4.0K Jun 22 12:31 .glusterfsdrwxr-xr-x 2 root root 4.0K Jun 22 12:31 .glusterfs-anonymous-inode-ad32a409-1b0f-4bd6-9420-acd93b0e1a14-rw-r--r-- 2 root root 0 Jun 22 12:46 test_fileroot@gluster-fs1:~# ls -lah /gluster-client/total 8.0Kdrwxr-xr-x 4 root root 4.0K Jun 22 12:46 .drwxr-xr-x 20 root root 4.0K Jun 22 12:28 ..-rw-r--r-- 1 root root 0 Jun 22 12:46 test_file |
| --- | --- |

любой из серверов можно спокойно выключать, доступ настроен через fstab

|  | root@gluster-fs1:~# cat /etc/fstab | grep glustergluster-fs1.test.local:/gluster-tom,gluster-fs2.test.local:/gluster-tom /gluster-client glusterfs defaults,_netdev 0 0 |
| --- | --- |

### []Установка S3-minio

Установим хранилищеS3minio в виде кластера чтобы была репликация как бакетов так и пользователей с ролями, + у нас будет виртуальныйIPчтобы мы ходили по одному и тому же адресу.
Приступим:

подготовим 2 сервера debian 121CPU1RAMи нужно будет добавить туда по выделенную разделу , т.е. просто указать директорию не прокатит.

ip будут

192.168.1.118 # s3-minio1

192.168.1.119 # s3-minio2

192.168.1.120 # s3-minio это виртуальныйIP

прогоняем плейбук на установку всех пользователей доп пакетов и т.д.

правим
**/etc/ansible/hosts**

|  | [all_servers]192.168.1.100 # freeipa1192.168.1.101 # freeipa2192.168.1.102 # nexus192.168.1.103 # vault1192.168.1.104 # vault2192.168.1.105 # vault3192.168.1.106 # gitlab192.168.1.107 # gitlab-runner192.168.1.108 # nfs192.168.1.109 # gluster-fs1192.168.1.110 # gluster-fs2192.168.1.118 # s3-minio1192.168.1.119 # s3-minio2 |
| --- | --- |

[root@ansible ansible]#**ansible-playbook -u root /etc/ansible/playbooks/roles_play/new_server.yml --ask-pass**

после этого настраиваем на этих серверах доп диск у меня он будет наLVM

|  | root@debian:~# pvscan PV /dev/sda5 VG debian-vg lvm2 [<9.52 GiB / 24.00 MiB free] Total: 1 [<9.52 GiB] / in use: 1 [<9.52 GiB] / in no VG: 0 [0 ]root@debian:~# vgscan Found volume group "debian-vg" using metadata type lvm2root@debian:~# lvscan ACTIVE '/dev/debian-vg/root' [8.54 GiB] inherit ACTIVE '/dev/debian-vg/swap_1' [976.00 MiB] inherit |
| --- | --- |

|  | pvcreate /dev/sdb && vgextend debian-vg /dev/sdb && lvcreate --name minio -L 10g debian-vg && mkfs.ext4 /dev/mapper/debian--vg-minio && mkdir /minio && mount /dev/debian-vg/minio /minio |
| --- | --- |

Этой командой

создаёмPVpvcreate /dev/sdb
расширяем группу vgextend debian-vg /dev/sdb
создаём том на10GBlvcreate --name minio -L 10g debian-vg
форматриуем том в нужную файловую систему mkfs.ext4 /dev/mapper/debian--vg-minio
создаём директорию в которую будем монтировать том mkdir /minio
монтируем том mount /dev/debian-vg/minio /minio

далее добавим в fstab это

|  | echo "/dev/mapper/debian--vg-minio /minio ext4 errors=remount-ro 0 1" >> /etc/fstab |
| --- | --- |

проверяем

root@debian:~# echo "/dev/mapper/debian--vg-minio /minio ext4 errors=remount-ro 0 1" >> /etc/fstab
root@debian:~# reboot

|  | root@debian:~# df -hFilesystem Size Used Avail Use% Mounted onudev 445M 0 445M 0% /devtmpfs 94M 568K 94M 1% /run/dev/mapper/debian--vg-root 8.4G 2.3G 5.7G 29% /tmpfs 469M 0 469M 0% /dev/shmtmpfs 5.0M 0 5.0M 0% /run/lock/dev/sda1 455M 147M 284M 35% /boot/dev/mapper/debian--vg-minio 9.8G 24K 9.3G 1% /miniotmpfs 94M 0 94M 0% /run/user/0 |
| --- | --- |

на втором сервере делаем тоже самое:

|  | root@debian:~# pvcreate /dev/sdb && vgextend debian-vg /dev/sdb && lvcreate --name minio -L 10g debian-vg && mkfs.ext4 /dev/mapper/debian--vg-minio && mkdir /minio && mount /dev/debian-vg/minio /minioroot@debian:~# echo "/dev/mapper/debian--vg-minio /minio ext4 errors=remount-ro 0 1" >> /etc/fstab |
| --- | --- |

ок сервера подготовили теперь настроим роль:

**/etc/ansible/hosts**

|  | [minio]s3-minio-1.test.local ansible_host=192.168.1.118s3-minio-2.test.local ansible_host=192.168.1.119[minio:vars]virtual_address=192.168.1.120 |
| --- | --- |

**/etc/ansible/roles/minio/defaults/main.yml**

|  | keepalived: trueminio_server_datadirs: - /miniominio_server_cluster_nodes: - s3-minio-1.test.local - s3-minio-2.test.localminio_root_user: "admin"minio_root_password: "Secret123" |
| --- | --- |

указываем что мы будем использовать виртуальныйIPkeepalived: true
указываем нашLVMтом minio_server_datadirs
указываем имена наших s3 нод minio_server_cluster_nodes отмечу что эти имена должны совпадать с теми что указаны в**/etc/ansible/hosts**
указываем рутовый логин minio_root_user
указываем рутовый пароль minio_root_password

готового можно запускать установку:

[root@ansible ansible]#**ansible-playbook -u root /etc/ansible/playbooks/roles_play/s3-minio.yml --ask-pass**

ждём окончания и проверяем:

сначала первый сервер:

**http://s3-minio-1.test.local:9001

![](/news/sidmidru/article-db03824b64600e5b/image-01.png)

**

вводим наши
admin
Secret123

Создаём бакет

![](/news/sidmidru/article-db03824b64600e5b/image-02.png)

![](/news/sidmidru/article-db03824b64600e5b/image-03.png)

создаём пользователя:

![](/news/sidmidru/article-db03824b64600e5b/image-04.png)

![](/news/sidmidru/article-db03824b64600e5b/image-05.png)

проверяем на втором сервере:

**http://s3-minio-2.test.local:9001/login**

![](/news/sidmidru/article-db03824b64600e5b/image-06.png)

![](/news/sidmidru/article-db03824b64600e5b/image-07.png)

как видим на втором сервере и бакет и пользователь доступны - репликация работает, всё ок

### []ИнтеграцияFREEIPA

Для начала создадим группу серверов группы пользователей для сервисов

создаём следующие группы пользователей:

![](/news/sidmidru/article-db03824b64600e5b/image-08.png)

добавим 2 пользователя
user1 - будет админом
user2 - будет обычным пользователем

![](/news/sidmidru/article-db03824b64600e5b/image-09.png)

теперь добавим их по соответствующим группам

![](/news/sidmidru/article-db03824b64600e5b/image-10.png)

![](/news/sidmidru/article-db03824b64600e5b/image-11.png)

так же раскидываем по остальным группам

gitlab, nexus-admins, vault-admins - user1
gitlab. nexus-ro-users, vault-ro-users - user2

### []Freeipa -> Nexus

[root@freeipa-1 ~]#**ipa cert-show 1 --out=/tmp/ipa-ca.crt**
[root@freeipa-1 ~]#**scp /tmp/ipa-ca.crt root@192.168.1.102:/tmp/**
root@nexus:~#**find / -name jre**
/usr/lib/jvm/jdk1.8.0_371/jre/bin
root@nexus:/usr/lib/jvm/jdk1.8.0_371/jre/bin#**./keytool -import -trustcacerts -alias freeipa-ca -file /tmp/ipa-ca.crt -keystore $JAVA_HOME/jre/lib/security/cacerts -storepass changeit**
Trust this certificate? [no]:**yes**
Certificate was added to keystore

Далее нам нужно создать системного пользователя во freeipa с помощью которого nexus сможет ходить в api и читать имена пользователей, но у него не будет прав что то выполнить. для этого на сервере freeipa или его реплике заходим в директорию /etc/ipa и запускаем скрипт. или выкачиваем тут:
**https://github.com/noahbliss/freeipa-sam**

[root@freeipa-1 ipa]#**cd /etc/ipa
**[root@freeipa-1 ipa]#**bash freeipa-sam.sh**

нажимаем**1**и задаём имя нашего сервера:

|  | ### FreeIPA - System Account Manager ###1.) ldapserver=2.) domain= (ldapdomain=)3.) binduser=4.) bindpass=UNSET!5.) ssl=trueActions (conditions not yet met): add | rm | ls | info | passwd | save--- Results ------ End Results ---> 1ldapserver=freeipa-1.test.local |
| --- | --- |

нажимаем**enter**и получаем результат:

|  | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local(ldapdomain=dc=test,dc=local)3.) binduser=4.) bindpass=UNSET!5.) ssl=trueActions (conditions not yet met): add | rm | ls | info | passwd | save--- Results ------ End Results ---> 3 |
| --- | --- |

как видим далее мы нажали**3**
выбираем любого пользователя с админскими правами - я выбрал**admin**

|  | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local(ldapdomain=dc=test,dc=local)3.) binduser=4.) bindpass=UNSET!5.) ssl=trueActions (conditions not yet met): add | rm | ls | info | passwd | save--- Results ------ End Results ---> 3Enter "mgr" for Directory Manager. Otherwise enter the username or full binddn (-D option in ldapsearch)binduser=admin |
| --- | --- |

Далее выбираем**4**и нужно будет ввести пароль

|  | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local(ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=UNSET!5.) ssl=trueActions (conditions not yet met): add | rm | ls | info | passwd | save--- Results ------ End Results ---> 4 |
| --- | --- |

|  | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local(ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=UNSET!5.) ssl=trueActions (conditions not yet met): add | rm | ls | info | passwd | save--- Results ------ End Results ---> 4Enter password (will not echo): |
| --- | --- |

после выбираем**5**чтоб отключитьSSL

|  | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=SET!5.) ssl=falseActions (ready): add | rm | ls | info | passwd | save--- Results ------ End Results ---> |
| --- | --- |

как видим ssl=true поменялось на ssl=false

жмём**enter**и после можем создавать системного пользователя**nexus**для этого набираем команду**add**

|  | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local(ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=SET!5.) ssl=trueActions (ready): add | rm | ls | info | passwd | save--- Results ------ End Results ---> adduid of new user=nexus |
| --- | --- |

далее вводим пароль и нас попросят указать дату окончания работы этого пароля, там ничего не указываем - просто жмём**ENTER**

| 123456789101112131415161718 | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local(ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=SET!5.) ssl=trueActions (ready): add | rm | ls | info | passwd | save--- Results ------ End Results ---> adduid of new user=nexuspassword of new user (blank to generate a password)=password expiration date YYYYMMDD (blank for 20380119)= |
| --- | --- |

командой**ls**можем проверить созданного пользователя

|  | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=SET!5.) ssl=falseActions (ready): add | rm | ls | info | passwd | save--- Results ---dn: uid=sudo,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=nexus,cn=sysaccounts,cn=etc,dc=test,dc=local--- End Results ---> |
| --- | --- |

пароль я кстати задал**Secret123**

логинимся в nexus и переходим в настройкиLDAP

![](/news/sidmidru/article-db03824b64600e5b/image-12.png)

для заполнения используем данные
**uid=nexus,cn=sysaccounts,cn=etc,dc=test,dc=local**

![](/news/sidmidru/article-db03824b64600e5b/image-13.png)

проверяем и нажимаем**NEXT**

![](/news/sidmidru/article-db03824b64600e5b/image-14.png)

далее настраиваем

User relativeDN:**cn=users,cn=accounts
**User subtree:**устанавливаем галку.
**Object class:**inetOrgPerson
**User filter:**(memberOf=cn=nexus-admins,cn=groups,cn=accounts,dc=test,dc=local)
**UserIDattribute:**uid
**Real name attribute:**cn
**Email attribute:**mail
**Password attribute:**оставляем пустым.
**MapLDAPgroups as roles:**устанавливаем галку.
**Group type:**Static Groups
**Group relativeDN:**cn=groups,cn=accounts
**Group subtree:**устанавливаем галку.
**Group object class:**groupOfNames
**GroupIDattribute:**cn
**Group member attribute:**member
**Group member format:**uid=${username},cn=users,cn=accounts,dc=example,dc=org**

![](/news/sidmidru/article-db03824b64600e5b/image-15.png)

![](/news/sidmidru/article-db03824b64600e5b/image-16.png)

![](/news/sidmidru/article-db03824b64600e5b/image-17.png)

![](/news/sidmidru/article-db03824b64600e5b/image-18.png)

![](/news/sidmidru/article-db03824b64600e5b/image-19.png)

всё нажимаем create

идём проверять:

![](/news/sidmidru/article-db03824b64600e5b/image-20.png)

Настраиваем тоже самое для второго сервера - для отказоустойчивости:

![](/news/sidmidru/article-db03824b64600e5b/image-21.png)

Создаём админскую роль чтоб пользователи подтягивались и становились сразу админами:

![](/news/sidmidru/article-db03824b64600e5b/image-22.png)

![](/news/sidmidru/article-db03824b64600e5b/image-23.png)

![](/news/sidmidru/article-db03824b64600e5b/image-24.png)

всё, теперь мотаем в самый низ и сохраняем роль

![](/news/sidmidru/article-db03824b64600e5b/image-25.png)

вот наша роль:

![](/news/sidmidru/article-db03824b64600e5b/image-26.png)

проверяем, для этого идём во freeipa и добавляем нового пользователя:

![](/news/sidmidru/article-db03824b64600e5b/image-27.png)

добавляем его в группу**nexus-admins**

![](/news/sidmidru/article-db03824b64600e5b/image-28.png)

проверяем в nexus

пользователь есть:

![](/news/sidmidru/article-db03824b64600e5b/image-29.png)

и он в нужными правами правами

![](/news/sidmidru/article-db03824b64600e5b/image-30.png)

#### **read only группа**

тут пока в настройках оставляем всё так же

![](/news/sidmidru/article-db03824b64600e5b/image-31.png)

а вот при настройке

User filter мы меняем группу с**nexus-admins**на**nexus-ro-users**

**(memberOf=cn=nexus-ro-users,cn=groups,cn=accounts,dc=test,dc=local)**

![](/news/sidmidru/article-db03824b64600e5b/image-32.png)

и создаём роль с привилегиями - тут я указал привелегии первые попавшиеся, вы можете задавать какие нужны именно вам.

![](/news/sidmidru/article-db03824b64600e5b/image-33.png)

проверяем права у пользователя user2

![](/news/sidmidru/article-db03824b64600e5b/image-34.png)

как видим всё ок

### []Freeipa -> Gitlab

сделаем интеграцию между freeipa и gitlab

к сожалению в бесплатной версии гитлаба нельзя привязывать пользователей к определённым группам в гитлабе, вот дока:
https://docs.gitlab.com/ee/administration/auth/ldap/ldap_synchronization.html

поэтому создадим 1 группу в FreeIPA:

**gitlab**

создадим системного пользователя gitlab у которого будет доступ на чтение групп и пользователей.

для этого на сервере freeipa или его реплике заходим в директорию /etc/ipa и запускаем скрипт. или выкачиваем тут:
**https://github.com/noahbliss/freeipa-sam**

[root@freeipa-1 ipa]#**cd /etc/ipa
**[root@freeipa-1 ipa]#**bash freeipa-sam.sh**

нажимаем**1**и задаём имя нашего сервера freeipa-1.test.local :

вот результат:

|  | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=4.) bindpass=UNSET!5.) ssl=trueActions (conditions not yet met): add | rm | ls | info | passwd | save--- Results ------ End Results ---> 3 |
| --- | --- |

выбрали**3**указываем пользователя**admin**(любой пользователь с правами админа)в нашем freeipa далее выберем**4**и зададим пароль от этого пользователя, после выбираем**5**чтоб отключитьSSL

|  | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=SET!5.) ssl=falseActions (ready): add | rm | ls | info | passwd | save--- Results ------ End Results ---> |
| --- | --- |

после можем создавать системного пользователя**gitlab**для этого набираем команду**add**

нас попросят ввести имя пользователя мы вводим**gitlab**после нас попросят ввести пароль я указал Secret123 далее нас попросят указать дату истечения этого пароль - ничего не указываем, нажимаем**ENTER**

проверяем что пользователь создан для этого набираем**ls**

| 1234567891011121314151617 | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=SET!5.) ssl=falseActions (ready): add | rm | ls | info | passwd | save--- Results ---dn: uid=sudo,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=nexus,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=gitlab,cn=sysaccounts,cn=etc,dc=test,dc=local--- End Results ---> |
| --- | --- |

как видим пользователь создан:

**dn: uid=gitlab,cn=sysaccounts,cn=etc,dc=test,dc=local**

теперь идём на сервер gitlab

[root@ansible ansible]# ssh 192.168.1.106
root@192.168.1.106's password:

редактируем файл:
root@gitlab:~#**nano /etc/gitlab/gitlab.rb**

включаемLDAP
**gitlab_rails['ldap_enabled'] = true**

Затем укажите путь к файлу с настройкамиLDAPдля FreeIPA.

**gitlab_rails['ldap_servers'] =YAML.load_file('/etc/gitlab/freeipa_settings.yml')**
Наконец, создайте файлYAMLдля хранения настроек подключенияIPA.

**cat /etc/gitlab/freeipa_settings.yml**

| 12345678910111213141516171819 | main: label: 'FreeIPA' host: 'freeipa-1.test.local' port: 389 uid: 'uid' method: 'tls' bind_dn: 'uid=gitlab,cn=sysaccounts,cn=etc,dc=test,dc=local' password: 'Secret123' encryption: 'plain' base: 'cn=accounts,dc=test,dc=local' group_base: 'cn=groups,cn=accounts,DC=test,DC=local' user_filter: 'memberOf=cn=gitlab,cn=groups,cn=accounts,dc=test,dc=local' verify_certificates: false attributes: username: ['uid'] email: ['mail'] name: 'displayName' first_name: 'givenName' last_name: 'sn' |
| --- | --- |

вот тут:
user_filter: 'memberOf=cn=gitlab,cn=groups,cn=accounts,dc=test,dc=local'
мы ограничиваем пользователей группой gitlab

bind_dn: 'uid=gitlab,cn=sysaccounts,cn=etc,dc=test,dc=local'
а тут мы используем системный аккаунт который создали ранее

и запускаем реконфигурацию:

root@gitlab:~#**gitlab-ctl reconfigure**

проверяем:

http://gitlab.test.local/users/sign_in

![](/news/sidmidru/article-db03824b64600e5b/image-35.png)

![](/news/sidmidru/article-db03824b64600e5b/image-36.png)

как видим всё ок.

дальше можем настраивать группы и добавлять пользователей внутри гитлаба

### []Freeipa -> Vault

создадим системного пользователя vault у которого будет доступ на чтение групп и пользователей.

для этого на сервере freeipa или его реплике заходим в директорию /etc/ipa и запускаем скрипт. или выкачиваем тут:
**https://github.com/noahbliss/freeipa-sam**

[root@freeipa-1 ipa]#**cd /etc/ipa
**[root@freeipa-1 ipa]#**bash freeipa-sam.sh**

нажимаем**1**и задаём имя нашего сервера**freeipa-1.test.local**
нажимаем**3**и вводим имя пользователя**admin**
нажимаем**4**и вводим пароль**Secret123**
нажимаем**5**- выключаем ssl

далее нажимаем**add**предложат ввести имя системного пользователя, вводим**vault**предложат ввести пароль для него, я использую**Secret123**далее попросят ввести дату истечения пароля, ничего не вводим, нажимаем**Enter**

пользователь создан, проверяем, нажимаем**ls**
вот результат:

| 123456789101112131415161718 | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=SET!5.) ssl=falseActions (ready): add | rm | ls | info | passwd | save--- Results ---dn: uid=sudo,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=nexus,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=gitlab,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=vault,cn=sysaccounts,cn=etc,dc=test,dc=local--- End Results ---> |
| --- | --- |

со стороны freeipa всё,

теперь настраиваем vault:

на всех тачках добавляем:

**echo "192.168.1.100 freeipa-1.test.local" >> /etc/hosts**

еслиDNSне настроен.

заходим в vault

**https://vault.test.local:8200/**

напомню что root_token hvs.*****************************

переходим:
**Access -> Enable new Method ->LDAP-> Enable Method**

![](/news/sidmidru/article-db03824b64600e5b/image-37.png)

![](/news/sidmidru/article-db03824b64600e5b/image-38.png)

![](/news/sidmidru/article-db03824b64600e5b/image-39.png)

![](/news/sidmidru/article-db03824b64600e5b/image-40.png)

теперь настраиваем
**URL= ldap://freeipa-1.test.local:389**

![](/news/sidmidru/article-db03824b64600e5b/image-41.png)

в разделе**LDAPoptions**: раскрываем и меняем User atrribut на**uid**

![](/news/sidmidru/article-db03824b64600e5b/image-42.png)

![](/news/sidmidru/article-db03824b64600e5b/image-43.png)

Следующий раздел**customize user search**— тут мы как раз и настраиваем адрес кем биндимся, а так же где искать пользователей, для freeipa это будут такие параметры

**Name of Object to bind (binddn) = uid=vault,cn=sysaccounts,cn=etc,dc=test,dc=local****
****UserDN= cn=users,cn=accounts,dc=test,dc=local
****Bindpass = Secret123**

![](/news/sidmidru/article-db03824b64600e5b/image-44.png)

Так же настроим поиск по группам

**Group Filter = (|(member={{.UserDN}})(uniqueMember={{.UserDN}}))
Group Attribute = cn
****GroupDN= cn=groups,cn=accounts,dc=test,dc=local**

![](/news/sidmidru/article-db03824b64600e5b/image-45.png)

теперь создадим policy для админов и пользователей

![](/news/sidmidru/article-db03824b64600e5b/image-46.png)

![](/news/sidmidru/article-db03824b64600e5b/image-47.png)

![](/news/sidmidru/article-db03824b64600e5b/image-48.png)

и ещё одну

![](/news/sidmidru/article-db03824b64600e5b/image-49.png)

вот сами policy:
vault-admins

|  | path "*" { capabilities = ["create", "read", "update", "delete", "list", "sudo"]} |
| --- | --- |

vault-ro-users

|  | path "*" { capabilities = ["read", "list"]} |
| --- | --- |

теперь группы вLDAP

![](/news/sidmidru/article-db03824b64600e5b/image-50.png)

![](/news/sidmidru/article-db03824b64600e5b/image-51.png)

группа**vault-admins**
policy**vault-admins**

![](/news/sidmidru/article-db03824b64600e5b/image-52.png)

![](/news/sidmidru/article-db03824b64600e5b/image-53.png)

добавляем вторую группу
группа**vault-ro-users**
policy**vault-ro-users**

![](/news/sidmidru/article-db03824b64600e5b/image-54.png)

![](/news/sidmidru/article-db03824b64600e5b/image-55.png)

проверяем:

![](/news/sidmidru/article-db03824b64600e5b/image-56.png)

![](/news/sidmidru/article-db03824b64600e5b/image-57.png)

![](/news/sidmidru/article-db03824b64600e5b/image-58.png)

![](/news/sidmidru/article-db03824b64600e5b/image-59.png)

![](/news/sidmidru/article-db03824b64600e5b/image-60.png)

как видимKVуспешно создан

теперь зайдём под**user2**который напомним что находится в группе**vault-ro-users**

![](/news/sidmidru/article-db03824b64600e5b/image-61.png)

![](/news/sidmidru/article-db03824b64600e5b/image-62.png)

![](/news/sidmidru/article-db03824b64600e5b/image-63.png)

![](/news/sidmidru/article-db03824b64600e5b/image-64.png)

![](/news/sidmidru/article-db03824b64600e5b/image-65.png)

как видим нам не хватает прав на создание новогоKV

настройка закончена, далее можно изменять policy как нам требуется.

**ниже указано как произвести всё тоже самое но консольными командами**

заходим на vault и логинимся

**vault login**

вводим рут токен

|  | root@vault1:~# vault loginWARNING! VAULT_ADDR and -address unset. Defaulting to https://127.0.0.1:8200.Token (will be hidden): Success! You are now authenticated. The token information displayed belowis already stored in the token helper. You do NOT need to run "vault login"again. Future Vault requests will automatically use this token.Key Value--- -----token hvs.b1aFMPa9WenUgwIkUgqtFtICtoken_accessor a87f7TLRK4mCRnrg5RDh4Pt4token_duration ∞token_renewable falsetoken_policies ["root"]identity_policies []policies ["root"] |
| --- | --- |

включаемLDAP

**vault authenableldap**

|  | root@vault1:~# vault auth enable ldapWARNING! VAULT_ADDR and -address unset. Defaulting to https://127.0.0.1:8200.Success! Enabled ldap auth method at: ldap/ |
| --- | --- |

настраиваем подключение

**vault write auth/ldap/config \**
**url="ldap://freeipa-1.test.local:389" \**
**binddn="uid=vault,cn=sysaccounts,cn=etc,dc=test,dc=local" \**
**bindpass='Secret123' \**
**userdn="cn=users,cn=accounts,dc=test,dc=local" \**
**userattr="uid" \**
**groupdn="cn=groups,cn=accounts,dc=test,dc=local" \**
**groupfilter="(|(member={{.UserDN}})(uniqueMember={{.UserDN}}))" \**
**groupattr="cn"**

|  | root@vault1:~# vault write auth/ldap/config \ url="ldap://freeipa-1.test.local:389" \ binddn="uid=vault,cn=sysaccounts,cn=etc,dc=test,dc=local" \ bindpass='Secret123' \ userdn="cn=users,cn=accounts,dc=test,dc=local" \ userattr="uid" \ groupdn="cn=groups,cn=accounts,dc=test,dc=local" \ groupfilter="(|(member={{.UserDN}})(uniqueMember={{.UserDN}}))" \ groupattr="cn"WARNING! VAULT_ADDR and -address unset. Defaulting to https://127.0.0.1:8200.Success! Data written to: auth/ldap/config |
| --- | --- |

создаём policy
vault-admins

**vault policy write vault-admins - <<EOFpath "*" { capabilities = ["create", "read", "update", "delete", "list", "sudo"] }EOF**

и vault-ro-users

**vault policy write vault-ro-users - <<EOFpath "*" { capabilities = ["read", "list"] }EOF**

|  | root@vault1:~# vault policy write vault-admins - <<EOFpath "*" { capabilities = ["create", "read", "update", "delete", "list", "sudo"]}EOFSuccess! Uploaded policy: vault-admins |
| --- | --- |

|  | root@vault1:~# vault policy write vault-ro-users - <<EOFpath "*" { capabilities = ["read", "list"]}EOFSuccess! Uploaded policy: vault-ro-users |
| --- | --- |

создаём группы в ldap и привязываем к ним policy

**vault write auth/ldap/groups/vault-admins policies=vault-admins**
**vault write auth/ldap/groups/vault-ro-users policies=vault-ro-users**

|  | root@vault1:~# vault write auth/ldap/groups/vault-admins policies=vault-adminsWARNING! VAULT_ADDR and -address unset. Defaulting to https://127.0.0.1:8200.Success! Data written to: auth/ldap/groups/vault-admins |
| --- | --- |

|  | root@vault1:~# vault write auth/ldap/groups/vault-ro-users policies=vault-ro-usersWARNING! VAULT_ADDR and -address unset. Defaulting to https://127.0.0.1:8200.Success! Data written to: auth/ldap/groups/vault-ro-users |
| --- | --- |

### []Freeipa -> S3-Minio

Настраиваем интеграцию Freeipa с S3-Minio

**/etc/ansible/hosts**

| 1234567891011121314151617181920 | [freeipa:children]ipa_serversipa_replicasipa_clients[ipa_servers]freeipa-1.test.local ansible_host=192.168.1.100[ipa_replicas]freeipa-2.test.local ansible_host=192.168.1.101[ipa_clients]nexus.test.local ansible_host=192.168.1.102vault1.test.local ansible_host=192.168.1.103vault2.test.local ansible_host=192.168.1.104vault3.test.local ansible_host=192.168.1.105gitlab.test.local ansible_host=192.168.1.106gitlab-runner.test.local ansible_host=192.168.1.107nfs.test.local ansible_host=192.168.1.108gluster-fs1.test.local ansible_host=192.168.1.109gluster-fs2.test.local ansible_host=192.168.1.110s3-minio-1.test.local ansible_host=192.168.1.118s3-minio-2.test.local ansible_host=192.168.1.119 |
| --- | --- |

[root@ansible ansible]#**ansible-playbook -u root /etc/ansible/playbooks/roles_play/freeipa.yaml --ask-pass**

идём в freeipa

**https://freeipa-1.test.local/**

создаём группу узлов:

![](/news/sidmidru/article-db03824b64600e5b/image-66.png)

![](/news/sidmidru/article-db03824b64600e5b/image-67.png)

Добавляем туда наши сервера

![](/news/sidmidru/article-db03824b64600e5b/image-68.png)

![](/news/sidmidru/article-db03824b64600e5b/image-69.png)

![](/news/sidmidru/article-db03824b64600e5b/image-70.png)

создаём группы пользователей admin и read only

![](/news/sidmidru/article-db03824b64600e5b/image-71.png)

![](/news/sidmidru/article-db03824b64600e5b/image-72.png)

![](/news/sidmidru/article-db03824b64600e5b/image-73.png)

![](/news/sidmidru/article-db03824b64600e5b/image-74.png)

![](/news/sidmidru/article-db03824b64600e5b/image-75.png)

![](/news/sidmidru/article-db03824b64600e5b/image-76.png)

![](/news/sidmidru/article-db03824b64600e5b/image-77.png)

![](/news/sidmidru/article-db03824b64600e5b/image-78.png)

user1 у нас будет в группе админов
user2 у нас будет в группе read only пользователей

Далее нам нужно создать системного пользователя во freeipa с помощью которого s3-minio сможет ходить в api и читать имена пользователей, но у него не будет прав что то выполнить. для этого на сервере freeipa или его реплике заходим в директорию**/etc/ipa**и запускаем скрипт. или выкачиваем тут:
**https://github.com/noahbliss/freeipa-sam**

[root@freeipa-1 ipa]#**cd /etc/ipa**
[root@freeipa-1 ipa]#**bash freeipa-sam.sh**

нажимаем**1**и задаём имя нашего сервера freeipa-1.test.local :

вот результат:

|  | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=4.) bindpass=UNSET!5.) ssl=trueActions (conditions not yet met): add | rm | ls | info | passwd | save--- Results ------ End Results ---> 3 |
| --- | --- |

выбрали**3**указываем пользователя**admin**(любой пользователь с правами админа)в нашем freeipa далее выберем**4**и зададим пароль от этого пользователя, после выбираем**5**чтоб отключитьSSL

|  | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=SET!5.) ssl=falseActions (ready): add | rm | ls | info | passwd | save--- Results ------ End Results ---> |
| --- | --- |

после можем создавать системного пользователя**s3minio**для этого набираем команду**add**

нас попросят ввести имя пользователя мы вводим**s3minio**после нас попросят ввести пароль я указал**Secret123**далее нас попросят указать дату истечения этого пароль - ничего не указываем, нажимаем**ENTER**

проверяем что пользователь создан для этого набираем**ls**

| 12345678910111213141516171819 | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=SET!5.) ssl=falseActions (ready): add | rm | ls | info | passwd | save--- Results ---dn: uid=sudo,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=nexus,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=gitlab,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=vault,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=s3minio,cn=sysaccounts,cn=etc,dc=test,dc=local--- End Results ---> |
| --- | --- |

как видим пользователь создан:

**dn: uid=s3minio,cn=sysaccounts,cn=etc,dc=test,dc=local**

дальше настройка из консоли:

1 создаём подключение:

|  | root@s3-minio-1:~# mc alias set myminio http://s3-minio-1.test.local:9000 admin Secret123mc: Configuration written to `/root/.mc/config.json`. Please update your access credentials.mc: Successfully created `/root/.mc/share`.mc: Initialized share uploads `/root/.mc/share/uploads.json` file.mc: Initialized share downloads `/root/.mc/share/downloads.json` file.Added `myminio` successfully. |
| --- | --- |

2. проверяем какие бакеты есть

|  | root@s3-minio-1:~# mc ls myminio[2024-08-31 14:47:41 +06] 0B test-bucket/ |
| --- | --- |

3. настраиваем подключение к ldap

|  | mc admin config set myminio identity_ldap \ enabled="true" \ server_addr="freeipa-1.test.local:389" \ lookup_bind_dn="uid=s3minio,cn=sysaccounts,cn=etc,dc=test,dc=local" \ lookup_bind_password="Secret123" \ user_dn_search_base_dn="cn=users,cn=accounts,dc=test,dc=local" \ user_dn_search_filter="(&(uid=%s)(objectClass=inetOrgPerson))" \ group_search_base_dn="cn=groups,cn=accounts,dc=test,dc=local" \ group_search_filter="(&(objectclass=groupOfNames)(member=%d))" \ server_insecure="true" \ tls_skip_verify="true" |
| --- | --- |

рестартуем

|  | mc admin service restart myminio |
| --- | --- |

подвязываем политику**consoleAdmin**к роли админов**s3-minio-admins**

|  | mc idp ldap policy attach myminio consoleAdmin --group='cn=s3-minio-admins,cn=groups,cn=accounts,dc=test,dc=local' |
| --- | --- |

подвязываем политики**readwrite**и**diagnostics**к роли пользователей**s3-minio-ro-users**я вижу что роль называетсяROно мне лень сейчас и скрины переделывать и роль переименовывать.

|  | mc idp ldap policy attach myminio readwrite --group='cn=s3-minio-ro-users,cn=groups,cn=accounts,dc=test,dc=local'mc idp ldap policy attach myminio diagnostics --group='cn=s3-minio-ro-users,cn=groups,cn=accounts,dc=test,dc=local' |
| --- | --- |

проверяем в админ панели minio что доступы есть:

![](/news/sidmidru/article-db03824b64600e5b/image-79.png)

проверяем авторизацию напомню что user1 у нас в группе админов а user2 в группе пользователей, + логин пароль используем из freeipa

y user1 админка не отличается от пользователя admin

вот админка пользователя user2

![](/news/sidmidru/article-db03824b64600e5b/image-80.png)

ну всё ок. логин проверили.

интеграция успешно настроена.

### []Install kubernetes (kubespray official)

Подготовим 6 серверов 3 под мастеров 3 под воркеры

операционка debian 12

192.168.1.112 # kub-master1

192.168.1.113 # kub-master2

192.168.1.114 # kub-master3

192.168.1.115 # kub-worker1

192.168.1.116 # kub-worker2

192.168.1.117 # kub-worker3

для мастеров 2 ядра 4 оперативки
для воркеров 4 ядра 4 оперативки

так же подготовим диски минимум 30 гб

**pvcreate /dev/sdb&&vgextend debian-vg /dev/sdb&&lvextend -L +15G/dev/debian-vg/root&&resize2fs /dev/debian-vg/root**

| 12345678910111213141516171819202122232425262728293031323334 | [root@ansible ansible]# ssh 192.168.1.112root@192.168.1.112's password: Linux debian 5.10.0-30-amd64 #1 SMP Debian 5.10.218-1 (2024-06-01) x86_64root@debian:~# pvscan PV /dev/sda5 VG debian-vg lvm2 [<14.52 GiB / 0 free] Total: 1 [<14.52 GiB] / in use: 1 [<14.52 GiB] / in no VG: 0 [0 ]root@debian:~# pvcreate /dev/sdb Physical volume "/dev/sdb" successfully created.root@debian:~# vgextend debian-vg /dev/sdb Volume group "debian-vg" successfully extendedroot@debian:~# pvscan PV /dev/sda5 VG debian-vg lvm2 [<14.52 GiB / 0 free] PV /dev/sdb VG debian-vg lvm2 [<30.00 GiB / <30.00 GiB free] Total: 2 [<44.52 GiB] / in use: 2 [<44.52 GiB] / in no VG: 0 [0 ]root@debian:~# lvextend -L +15G /dev/debian-vg/root Size of logical volume debian-vg/root changed from 13.56 GiB (3472 extents) to 28.56 GiB (7312 extents). Logical volume debian-vg/root successfully resized.root@debian:~# resize2fs /dev/debian-vg/root resize2fs 1.46.2 (28-Feb-2021)Filesystem at /dev/debian-vg/root is mounted on /; on-line resizing requiredold_desc_blocks = 2, new_desc_blocks = 4The filesystem on /dev/debian-vg/root is now 7487488 (4k) blocks long. |
| --- | --- |

обновим и доустановим все необходимые пакеты:

/etc/ansible/hosts

| 123456789101112131415161718 | [all_servers]192.168.1.100 # freeipa1192.168.1.101 # freeipa2192.168.1.102 # nexus192.168.1.103 # vault1192.168.1.104 # vault2192.168.1.105 # vault3192.168.1.106 # gitlab192.168.1.107 # gitlab-runner192.168.1.108 # nfs192.168.1.109 # gluster-fs1192.168.1.110 # gluster-fs2192.168.1.112 # kub-master1192.168.1.113 # kub-master2192.168.1.114 # kub-master3192.168.1.115 # kub-worker1192.168.1.116 # kub-worker2192.168.1.117 # kub-worker3 |
| --- | --- |

[root@ansible ansible]#**ansible-playbook -u root /etc/ansible/playbooks/roles_play/new_server.yml --ask-pass**

**ЭТООПЦИОНАЛЬНО**добавим эти сервера сразу к freeipa. nexus

(можно и не добавлять если в вашем варианте это не требуется)

**/etc/ansible/hosts**

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647 | [freeipa:children]ipa_serversipa_replicasipa_clients[ipa_servers]freeipa-1.test.local ansible_host=192.168.1.100[ipa_replicas]freeipa-2.test.local ansible_host=192.168.1.101[ipa_clients]nexus.test.local ansible_host=192.168.1.102vault1.test.local ansible_host=192.168.1.103vault2.test.local ansible_host=192.168.1.104vault3.test.local ansible_host=192.168.1.105gitlab.test.local ansible_host=192.168.1.106gitlab-runner.test.local ansible_host=192.168.1.107nfs.test.local ansible_host=192.168.1.108gluster-fs1.test.local ansible_host=192.168.1.109gluster-fs2.test.local ansible_host=192.168.1.110kub-master1.test.local ansible_host=192.168.1.112 kub-master2.test.local ansible_host=192.168.1.113 kub-master3.test.local ansible_host=192.168.1.114 kub-worker1.test.local ansible_host=192.168.1.115 kub-worker2.test.local ansible_host=192.168.1.116 kub-worker3.test.local ansible_host=192.168.1.117 [nexus:children]nexus_servernexus_clients[nexus_server]nexus.test.local ansible_host=192.168.1.102[nexus_clients]freeipa-1.test.local ansible_host=192.168.1.100freeipa-2.test.local ansible_host=192.168.1.101vault1.test.local ansible_host=192.168.1.103vault2.test.local ansible_host=192.168.1.104vault3.test.local ansible_host=192.168.1.105gitlab.test.local ansible_host=192.168.1.106gitlab-runner.test.local ansible_host=192.168.1.107nfs.test.local ansible_host=192.168.1.108gluster-fs1.test.local ansible_host=192.168.1.109gluster-fs2.test.local ansible_host=192.168.1.110kub-master1.test.local ansible_host=192.168.1.112 kub-master2.test.local ansible_host=192.168.1.113 kub-master3.test.local ansible_host=192.168.1.114 kub-worker1.test.local ansible_host=192.168.1.115 kub-worker2.test.local ansible_host=192.168.1.116 kub-worker3.test.local ansible_host=192.168.1.117 |
| --- | --- |

[root@ansible ansible]#**ansible-playbook -u root /etc/ansible/playbooks/roles_play/freeipa.yaml --ask-pass**

#### теперь можно приступать к установке k8s

нам нужен будет python3.8 у меня на ansible используется операционка centos7 с python3.6 поэтому я ставлю так:

**yum install -y gcc openssl-devel bzip2-devel libffi-devel zlib-devel**
**cd /usr/src**
**wget https://www.python.org/ftp/python/3.8.10/Python-3.8.10.tgz**
**tar xzf Python-3.8.10.tgz**
**cd Python-3.8.10**
**./configure --enable-optimizations**
**make altinstall**
**python3.8 -m venv ~/venv**
**source ~/venv/bin/activate**

вот тут официальный чарт kubespray

https://github.com/kubernetes-incubator/kubespray.git

в моём репозитории тоже используется официальный kubespray но так в моём репозитории используется старый kubespray

Ставим все нужные зависимости в ansible:

root@ansible:/etc/ansible#**cd /etc/ansible/kubespray-official/kubespray**
root@ansible:/etc/ansible/kubespray-official/kubespray#**apt install python3-pip -y**
root@ansible:/etc/ansible/kubespray-official/kubespray#**pip install -r requirements.txt**
root@ansible:/etc/ansible/kubespray-official/kubespray#**pip install -U jinja2**

заполняем inventory

**/etc/ansible/kubespray-official/kubespray/inventory/sample/inventory.ini**

| 1234567891011121314151617181920212223 | [all]kub-master1.test.local ansible_host=192.168.1.112 ip=192.168.1.112kub-master2.test.local ansible_host=192.168.1.113 ip=192.168.1.113kub-master3.test.local ansible_host=192.168.1.114 ip=192.168.1.114kub-worker1.test.local ansible_host=192.168.1.115 ip=192.168.1.115kub-worker2.test.local ansible_host=192.168.1.116 ip=192.168.1.116kub-worker3.test.local ansible_host=192.168.1.117 ip=192.168.1.117[kube_control_plane]kub-master1.test.localkub-master2.test.localkub-master3.test.local[etcd]kub-master1.test.localkub-master2.test.localkub-master3.test.local[kube_node]kub-worker1.test.localkub-worker2.test.localkub-worker3.test.local |
| --- | --- |

дополнительные аддоны мы можем включить тут:

**inventory/sample/group_vars/k8s_cluster/addons.yml**

|  | [root@ansible kubespray]# cat inventory/sample/group_vars/k8s_cluster/addons.yml | grep -i true | grep -v '#'helm_enabled: truelocal_volume_provisioner_enabled: true |
| --- | --- |

я выключаю всё кроме**helm_enabled**и**local_volume_provisioner_enabled**

Далее правим
**vim inventory/sample/group_vars/all/all.yml**

|  | ## Upstream dns serversupstream_dns_servers: - 192.168.1.100 - 192.168.1.101 - 8.8.8.8 - 8.8.4.4 |
| --- | --- |

#тут указываем наши днс сервера, я использую гугловые, можно оставить по умолчанию. ниже указываем наш прокси если он используется http_proxy: "http://proxy_ip:3128" https_proxy: "http://proxy_ip:3128"

я добавляю ещё мои 2 dns сервера с freeipa это 192,168,1,100 и 192,168,1,101

**vim inventory/sample/group_vars/k8s_cluster/k8s-cluster.yml
**включим все варианты аутентификации (обратите внимание на отступы в yml файлах, модули должны быть без отступов)

**kube_oidc_auth: true**
**kube_token_auth: true**

|  | [root@ansible kubespray]# cat inventory/sample/group_vars/k8s_cluster/k8s-cluster.yml | grep -E 'kube_oidc_auth|kube_basic_auth|kube_token_auth'kube_oidc_auth: truekube_token_auth: true |
| --- | --- |

в файле
**/etc/ansible/kubespray-official/kubespray/inventory/sample/group_vars/k8s_cluster/k8s-cluster.ym**l
настраиваем:

версию кластера:
**kube_version**: v1.24.0
плагин сети
**kube_network_plugin**: calico
подсеть для service
**kube_service_addresses**: 10.233.0.0/18
подсеть для pod
**kube_pods_subnet**: 10.233.64.0/18
подсеть node
**kube_network_node_prefix**: 24
выключим поддержку IPv6
**enable_dual_stack_networks**: false
имя кластера
**cluster_name**: cluster.local
можно настроить авто обновление сертификатов controlplane, но я это буду делать в ручном режиме.
**auto_renew_certificates**: false

на debian я запускал так:

|  | python3 -m venv .venv-ansiblesource .venv-ansible/bin/activatepip install --upgrade pip setuptools wheelsed -i '/ruamel.yaml.clib/d' requirements.txtpip install --upgrade pip setuptools wheelpip install -r requirements.txtpip install netaddr pip install cryptography pip install jmespath pip install jsonschemamkdir -p filter_pluginscp ~/.ansible/collections/ansible_collections/ansible/utils/plugins/filter/ipaddr.py filter_plugins/ |
| --- | --- |

запускаем установку:

**ansible-playbook -u root -i inventory/sample/inventory.ini cluster.yml -b --ask-pass**

ждём минут 30-40 зависит от сети и системы

проверяем:

[root@ansible ansible]#**ssh 192.168.1.112**
root@kub-master1:~#**kubectl get nodes**

|  | root@kub-master1:~# kubectl get nodesNAME STATUS ROLES AGE VERSIONkub-master1.test.local Ready control-plane 5d16h v1.24.0kub-master2.test.local Ready control-plane 5d16h v1.24.0kub-master3.test.local Ready control-plane 5d16h v1.24.0kub-worker1.test.local Ready <none> 5d16h v1.24.0kub-worker2.test.local Ready <none> 5d16h v1.24.0kub-worker3.test.local Ready <none> 5d16h v1.24.0 |
| --- | --- |

### []Доступ с локальногоPCдо кластера

допустим у насPCс операционкой debian и ip адресом**192.168.1.120**

root@debian:~#**curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"**

root@debian:~#**chmod +x kubectl**

root@debian:~#**cp ./kubectl /usr/bin/**

копируем конфиг на наш клиент

root@kub-master1:~#**scp /etc/kubernetes/admin.conf root@192.168.1.120:~/**

|  | root@kub-master1:~# scp /etc/kubernetes/admin.conf root@192.168.1.120:~/The authenticity of host '192.168.1.120 (192.168.1.120)' can't be established.ED25519 key fingerprint is SHA256:soNRFix0hNHB30IqzBzqt9BV9LTjHRE6V5uCvOVNxm8.This key is not known by any other names.Are you sure you want to continue connecting (yes/no/[fingerprint])? yesWarning: Permanently added '192.168.1.120' (ED25519) to the list of known hosts.root@192.168.1.120's password: admin.conf |
| --- | --- |

root@debian:~#**mkdir -p ~/.kube**
root@debian:~#**mv admin.conf ~/.kube/config**

так как в этом конфиге указан**127.0.0.1**в качестве сервера, заменим на наш ip**192.168.1.112**

root@debian:~#**sed -i 's/127.0.0.1/192.168.1.112/' ~/.kube/config**

можем проверять

|  | root@debian:~# kubectl get nodesNAME STATUS ROLES AGE VERSIONkub-master1.test.local Ready control-plane 6d v1.24.0kub-master2.test.local Ready control-plane 6d v1.24.0kub-master3.test.local Ready control-plane 6d v1.24.0kub-worker1.test.local Ready <none> 5d23h v1.24.0kub-worker2.test.local Ready <none> 6d v1.24.0kub-worker3.test.local Ready <none> 5d23h v1.24.0 |
| --- | --- |

настроим ещё автодополнения
root@debian:~#**apt-get install bash-completion -y**
root@debian:~#**echo 'source <(kubectl completion bash)' >>~/.bashrc**

установим на наш локальныйPC**HELM**

root@debian:~#**apt-get install gpg -y**
root@debian:~#**curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | tee /usr/share/keyrings/helm.gpg > /dev/null**
root@debian:~#**apt-get install apt-transport-https --yes**
root@debian:~#**echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | tee /etc/apt/sources.list.d/helm-stable-debian.list**
root@debian:~#**apt-get update**
root@debian:~#**apt-get install helm -y**

добавим ещё плагин diff - пригодится
**helm plugin install https://github.com/databus23/helm-diff**

### []Установка дополнительных компонентов

#### []Ставим ingress controller

сразу включим и метрики и snippet-annotations

**curl -sL https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.3/deploy/static/provider/cloud/deploy.yaml \
| sed 's/--enable-metrics=false/--enable-metrics=true/g; s/allow-snippet-annotations: "false"/allow-snippet-annotations: "true"/g' \
| kubectl apply -f -
**

| 1234567891011121314151617181920212223 | root@kub-master1:~# curl -sL https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.3/deploy/static/provider/cloud/deploy.yaml \| sed 's/--enable-metrics=false/--enable-metrics=true/g; s/allow-snippet-annotations: "false"/allow-snippet-annotations: "true"/g' \| kubectl apply -f -namespace/ingress-nginx createdserviceaccount/ingress-nginx createdserviceaccount/ingress-nginx-admission createdrole.rbac.authorization.k8s.io/ingress-nginx createdrole.rbac.authorization.k8s.io/ingress-nginx-admission createdclusterrole.rbac.authorization.k8s.io/ingress-nginx createdclusterrole.rbac.authorization.k8s.io/ingress-nginx-admission createdrolebinding.rbac.authorization.k8s.io/ingress-nginx createdrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission createdclusterrolebinding.rbac.authorization.k8s.io/ingress-nginx createdclusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission createdconfigmap/ingress-nginx-controller createdservice/ingress-nginx-controller createdservice/ingress-nginx-controller-admission createddeployment.apps/ingress-nginx-controller createdjob.batch/ingress-nginx-admission-create createdjob.batch/ingress-nginx-admission-patch createdingressclass.networking.k8s.io/nginx created |
| --- | --- |

проверяем:

|  | root@kub-master1:~# kubectl get pod -n ingress-nginx NAME READY STATUS RESTARTS AGEingress-nginx-admission-create-v94wt 0/1 Completed 0 3m2singress-nginx-admission-patch-8zwp2 0/1 Completed 2 3m2singress-nginx-controller-686556747b-dqxdg 1/1 Running 0 3m2s |
| --- | --- |

### []Ingress controller helm install

или можем поставить через helm вот оф values

**https://github.com/kubernetes/ingress-nginx/blob/helm-chart-4.12.2/charts/ingress-nginx/values.yaml**

вот values
cat values.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104 | controller: enabled: true kind: Deployment # DaemonSet replicaCount: 2 # CPU/memory limits for the controller pods resources: limits: cpu: 1000m memory: 1Gi requests: cpu: 100m memory: 256Mi # CRD for the nginx ingress class ingressClassResource: default: false enabled: true name: nginx controllerValue: k8s.io/ingress-nginx # Enable metrics endpoint metrics: enabled: true extraArgs: "enable-ssl-passthrough": "true"# enable-ssl-passthrough: true# passthrough:# enabled: true # All valid ConfigMap keys must be in kebab-case config: disable-ipv6: "true" disable-ipv6-dns: "true" enable-access-log-for-default-backend: "false" http2-max-field-size: "8k" large-client-header-buffers: "16 64k" limit-conn-status-code: "429" limit-req-status-code: "429" load-balance: ewma keepalive: "65" client-max-body-size: "300m" proxy-body-size: "300m" error-log-level: error log-format-escape-json: "true" log-format-upstream: >- {"bytes_sent":"$bytes_sent","vhost":"$host"," request_proto":"$server_protocol","remote_addr":"$remote_addr", "proxy_add_x_forwarded_for":"$proxy_add_x_forwarded_for","remote_user":"$remote_user", "time_local":"$time_local","request_method":"$request_method", "request_uri":"$uri","request_args":"$args","request":"$request", "status":"$status","body_bytes_sent":"$body_bytes_sent", "http_referer":"$http_referer","http_user_agent":"$http_user_agent", "request_length":"$request_length","request_time":"$request_time", "upstream_addr":"$upstream_addr","upstream_response_length":"$upstream_response_length", "upstream_response_time":"$upstream_response_time","upstream_status":"$upstream_status", "X-Business-Error":"$upstream_http_x_business_error","upstream_header_time":"$upstream_header_time", "upstream_connect_time":"$upstream_connect_time","connections_waiting":"$connections_waiting", "connections_active":"$connections_active"} map-hash-bucket-size: "128" server-tokens: "false" ssl-protocols: "TLSv1.2 TLSv1.3" ssl-session-cache: "true" ssl-session-cache-size: "20m" ssl-session-timeout: "30m" use-forwarded-headers: "true" use-gzip: "true" use-proxy-protocol: "false" worker-cpu-affinity: auto worker-processes: "2" allow-snippet-annotations: "true" annotations-risk-level: Critical # If you need raw TCP/SNI pass-through, un-comment and adjust: # extraArgs: # enable-ssl-passthrough: "true" # tcp: # "443": "default/ssl-passthrough:443" ingressClass: nginx # Service definition for the controller service: type: LoadBalancer externalTrafficPolicy: Cluster # Turn on the admission webhook deployment admissionWebhooks: enabled: true tolerations: - key: "node-role.kubernetes.io/master" operator: "Exists" effect: "NoSchedule" - key: "node-role.kubernetes.io/control-plane" operator: "Exists" effect: "NoSchedule"defaultBackend: enabled: false |
| --- | --- |

ставим командой:

**helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx**

**helm repo update**

|  | helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \ --namespace ingress-nginx --create-namespace \ --version 4.12.1 \ -f values.yaml |
| --- | --- |

#### []Metalllb

**kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.9/config/manifests/metallb-native.yaml**

| 12345678910111213141516171819202122232425 | root@kub-master1:~# kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.9/config/manifests/metallb-native.yamlnamespace/metallb-system createdcustomresourcedefinition.apiextensions.k8s.io/addresspools.metallb.io createdcustomresourcedefinition.apiextensions.k8s.io/bfdprofiles.metallb.io createdcustomresourcedefinition.apiextensions.k8s.io/bgpadvertisements.metallb.io createdcustomresourcedefinition.apiextensions.k8s.io/bgppeers.metallb.io createdcustomresourcedefinition.apiextensions.k8s.io/communities.metallb.io createdcustomresourcedefinition.apiextensions.k8s.io/ipaddresspools.metallb.io createdcustomresourcedefinition.apiextensions.k8s.io/l2advertisements.metallb.io createdserviceaccount/controller createdserviceaccount/speaker createdrole.rbac.authorization.k8s.io/controller createdrole.rbac.authorization.k8s.io/pod-lister createdclusterrole.rbac.authorization.k8s.io/metallb-system:controller createdclusterrole.rbac.authorization.k8s.io/metallb-system:speaker createdrolebinding.rbac.authorization.k8s.io/controller createdrolebinding.rbac.authorization.k8s.io/pod-lister createdclusterrolebinding.rbac.authorization.k8s.io/metallb-system:controller createdclusterrolebinding.rbac.authorization.k8s.io/metallb-system:speaker createdsecret/webhook-server-cert createdservice/webhook-service createddeployment.apps/controller createddaemonset.apps/speaker createdvalidatingwebhookconfiguration.admissionregistration.k8s.io/metallb-webhook-configuration createdroot@kub-master1:~# |
| --- | --- |

правим конфигмапу

**kubectl get configmap kube-proxy -n kube-system -o yaml | sed -e "s/strictARP: false/strictARP: true/" | kubectl apply -f - -n kube-system**

|  | root@kub-master1:~# kubectl get configmap kube-proxy -n kube-system -o yaml | sed -e "s/strictARP: false/strictARP: true/" | kubectl apply -f - -n kube-systemWarning: resource configmaps/kube-proxy is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.configmap/kube-proxy configured |
| --- | --- |

возвращаемся на**ansible**и идём в директорию:**/etc/ansible/kubespray-official/**

|  | [root@ansible ansible]# cd /etc/ansible/kubespray-official/[root@ansible kubespray-official]# pwd/etc/ansible/kubespray-official[root@ansible kubespray-official]# |
| --- | --- |

так как нам нужно отредактировать файлы и применить их а доступ на ансибл мы давать не хотим то скопируем эту директорию на наш master и уже оттуда применим.

**rsync -avh --progress metallb root@192.168.1.112:/tmp/**

|  | [root@ansible kubespray-official]# rsync -avh --progress metallb root@192.168.1.112:/tmp/root@192.168.1.112's password: sending incremental file listmetallb/metallb/README.md 2.44K 100% 0.00kB/s 0:00:00 (xfr#1, to-chk=2/4)metallb/ip-pool.yaml 267 100% 260.74kB/s 0:00:00 (xfr#2, to-chk=1/4)metallb/lb-ingress-controller-svc.yaml 537 100% 524.41kB/s 0:00:00 (xfr#3, to-chk=0/4)sent 3.52K bytes received 77 bytes 2.40K bytes/sectotal size is 3.24K speedup is 0.90 |
| --- | --- |

переходим в мастер и заходим в директорию**/tmp/metallb**и редактируем файл**ip-pool.yaml**

root@kub-master1:~#**cd /tmp/metallb/**
root@kub-master1:/tmp/metallb#**vim ip-pool.yaml**

В файл ip-pool.yaml добавляем или диапазонIPадресов, или 1IPуказав его подсеть /32 (по этому адресу будет доступен кластер)
я добавил - 192.168.1.191/32 и 192.168.1.192/32

|  | apiVersion: metallb.io/v1beta1kind: IPAddressPoolmetadata: name: first-pool namespace: metallb-systemspec: addresses: - 192.168.1.191/32 - 192.168.1.192/32 |
| --- | --- |

применяем

**kubectl apply -f ip-pool.yaml**

|  | root@kub-master1:/tmp/metallb# kubectl apply -f ip-pool.yamlipaddresspool.metallb.io/first-pool createdl2advertisement.metallb.io/example created |
| --- | --- |

Добавляем сервис для ingres-controller типа LoadBalancer

**kubectl apply -f lb-ingress-controller-svc.yaml**

|  | root@kub-master1:/tmp/metallb# kubectl apply -f lb-ingress-controller-svc.yamlservice/ingress-nginx-lb created |
| --- | --- |

проверяем:

|  | root@kub-master1:/tmp/metallb# kubectl get pod -n metallb-systemNAME READY STATUS RESTARTS AGEcontroller-c6c466d64-f79d8 1/1 Running 1 (18m ago) 19mspeaker-4t9hf 1/1 Running 0 19mspeaker-9p9sd 1/1 Running 0 19mspeaker-b8m9m 1/1 Running 0 19mspeaker-kfkfz 1/1 Running 0 19mspeaker-rwcfg 1/1 Running 0 19mspeaker-x2g8d 1/1 Running 0 19mroot@kub-master1:/tmp/metallb# kubectl get svc -n ingress-nginxNAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGEingress-nginx-controller LoadBalancer 10.233.32.234 192.168.1.191 80:31034/TCP,443:30100/TCP 27mingress-nginx-controller-admission ClusterIP 10.233.6.121 <none> 443/TCP 27mingress-nginx-lb LoadBalancer 10.233.14.234 192.168.1.192 80:31990/TCP,443:31405/TCP 41s |
| --- | --- |

как видимIP192.168.1.191 и 192.168.1.192 являютсяEXTERNAL-IP

### []Metallb helm-installation

оффициальный репозиторий

https://github.com/metallb/metallb/tree/main/charts/metallb

**helm repo add metallb https://metallb.github.io/metallb**

cat values.yaml

| 1234567891011121314151617181920212223242526272829303132 | controller: enabled: true resources: requests: cpu: "50m" memory: "80Mi" limits: cpu: 100m memory: 100Mispeaker: enabled: true logLevel: info resources: requests: cpu: "50m" memory: "50Mi" limits: cpu: 100m memory: 100Mi frr: enabled: true resources: requests: cpu: "10m" memory: "50Mi" limits: cpu: 50m memory: 100Micrds: enabled: true |
| --- | --- |

**helm install metallb metallb/metallb -n metallb-system --create-namespace -f values.yaml**

cat ip-pool.yaml

| 123456789101112131415161718 | apiVersion: metallb.io/v1beta1kind: IPAddressPoolmetadata: name: first-pool namespace: metallb-systemspec: addresses: - 192.168.1.191/32---apiVersion: metallb.io/v1beta1kind: L2Advertisementmetadata: name: first-l2-advertisement namespace: metallb-systemspec: ipAddressPools: - first-pool |
| --- | --- |

**kubectl apply -f ip-pool.yaml**

так же включаем strictArp

|  | kubectl get configmap kube-proxy -n kube-system -o yaml | \sed -e "s/strictARP: false/strictARP: true/" | \kubectl apply -f - -n kube-system |
| --- | --- |

проверить можем так

cat 1.yaml

| 1234567891011121314151617181920 | apiVersion: networking.k8s.io/v1kind: Ingressmetadata: name: test-ingress namespace: ingress-nginx annotations: nginx.ingress.kubernetes.io/rewrite-target: /spec: ingressClassName: nginx rules: - host: test.test.local http: paths: - path: / pathType: Prefix backend: service: name: ingress-nginx-defaultbackend port: number: 80 |
| --- | --- |

kubectl apply -f 1.yaml

|  | kubectl get ingress -ANAMESPACE NAME CLASS HOSTS ADDRESS PORTS AGEingress-nginx test-ingress nginx test.test.local 192.168.1.191 80 20s |
| --- | --- |

### []NFSprovisioner

добавляем доступ для серверов k8s к nfs серверу

**/etc/ansible/hosts**

|  | [nfs:children]nfsmasternfsclient[nfsmaster]nfs.test.local ansible_host=192.168.1.108[nfsclient]192.168.1.100192.168.1.101192.168.1.112192.168.1.113192.168.1.114192.168.1.115192.168.1.116192.168.1.117 |
| --- | --- |

[root@ansible ansible]#**ansible-playbook -u root /etc/ansible/playbooks/roles_play/nfs.yml --ask-pass**

копируем
[root@ansible ansible]#**scp -r /etc/ansible/kubespray-official/nfs-provision root@192.168.1.120:~/**

с клиента заходим и правим конфиг файл:

root@debian:~#**cd ~/nfs-provision/**
root@debian:~/nfs-provision#**nano nfs_provision.yaml**

| 12345678910111213141516171819202122232425262728293031323334353637383940 | apiVersion: apps/v1kind: Deploymentmetadata: name: nfs-client-provisioner labels: app: nfs-client-provisioner # replace with namespace where provisioner is deployed namespace: kube-systemspec: replicas: 1 strategy: type: Recreate selector: matchLabels: app: nfs-client-provisioner template: metadata: labels: app: nfs-client-provisioner spec: serviceAccountName: nfs-client-provisioner containers: - name: nfs-client-provisioner image: k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2 volumeMounts: - name: nfs-client-root mountPath: /persistentvolumes env: - name: PROVISIONER_NAME value: k8s-sigs.io/nfs-subdir-external-provisioner - name: NFS_SERVER value: 192.168.1.108 - name: NFS_PATH value: /nfs volumes: - name: nfs-client-root nfs: server: 192.168.1.108 path: /nfs |
| --- | --- |

устанавливаем ip адрес нашего nfs сервера
**192.168.1.108**

всё, можно ставить все компоненты
root@debian:~/nfs-provision#**kubectl apply -f rbac.yaml -f nfs_class.yaml -f nfs_provision.yaml**
если что, внутри этой директории есть**README.md**

проверяем:

|  | root@debian:~# kubectl get storageclasses.storage.k8s.ioNAME PROVISIONER RECLAIMPOLICY VOLUMEBINDINGMODE ALLOWVOLUMEEXPANSION AGElocal-storage kubernetes.io/no-provisioner Delete WaitForFirstConsumer false 6dnfs-client k8s-sigs.io/nfs-subdir-external-provisioner Delete Immediate false 18m |
| --- | --- |

запросим тестовые volume

root@debian:~/nfs-provision#**kubectl apply -f test-claim.yaml**

|  | root@debian:~/nfs-provision# kubectl get pvcNAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGEtest-claim Bound pvc-2c3d964b-ceb5-482b-a8a0-902ff3117c51 50Mi RWX nfs-client 44sroot@debian:~/nfs-provision# kubectl get pvNAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORAGECLASS REASON AGEpvc-2c3d964b-ceb5-482b-a8a0-902ff3117c51 50Mi RWX Delete Bound default/test-claim nfs-client 49s |
| --- | --- |

### []GlusterFS provisioner

Добавляем доступ для наших k8s серверов:

/etc/ansible/hosts

|  | [glusterfs:children]glustermasterglusterclient[glustermaster]gluster-fs1.test.local ansible_host=192.168.1.109gluster-fs2.test.local ansible_host=192.168.1.110[glusterclient]192.168.1.100192.168.1.101192.168.1.112192.168.1.113192.168.1.114192.168.1.115192.168.1.116192.168.1.117 |
| --- | --- |

[root@ansible ansible]#**ansible-playbook -u root /etc/ansible/playbooks/roles_play/glusterfs.yml --ask-pass**

копируем файлы на клиента

[root@ansible ansible]#**scp -r /etc/ansible/kubespray-official/glusterfs-provision root@192.168.1.120:~/**

заходим на наш клиент и правим файл:

root@debian:~#**cd glusterfs-provision/**
root@debian:~/glusterfs-provision#**nano endpont.yml**

|  | subsets: - addresses: #Тут указываем ip наших серверов - ip: 192.168.1.109 ports: #Тут просто можно оставить 1, порт роли не играет - port: 1 - addresses: - ip: 192.168.1.110 ports: - port: 1 |
| --- | --- |

добавляемIPнаших glusterfs серверов

192.168.1.109
192.168.1.110

root@debian:~/glusterfs-provision#**helm repo add olli-ai https://olli-ai.github.io/helm-charts/
**root@debian:~/glusterfs-provision#**helm repo update
**

проверяем:

|  | root@debian:~/glusterfs-provision# helm search repo glusterfs-clientNAME CHART VERSION APP VERSION DESCRIPTIONolli-ai/glusterfs-client-provisioner 1.0.2 v1.0.2 glusterfs-client-provisioner is an automatic pr... |
| --- | --- |

ставим

|  | helm install glusterfs-client olli-ai/glusterfs-client-provisioner \--namespace kube-system \--set glusterfs.server="{192.168.1.109,192.168.1.110}" \--set glusterfs.volume=gluster-tom \--set glusterfs.path=/ |
| --- | --- |

**glusterfs.volume**: gluster-tom - это том который мы задавали при настройке плейбука:
**glusterfs.path**/ - это путь который в самом glusterfs будет смотреть в самый корень т.е. вот сюда:

|  | root@gluster-fs2:~# ls -lah /gluster/gv01/total 32Kdrwxr-xr-x 4 root root 4.0K Jun 22 13:06 .drwxr-xr-x 3 root root 4.0K Jun 22 12:28 ..-rw-r--r-- 2 root root 0 Jun 22 13:06 1111111111111-rw-r--r-- 2 root root 0 Jun 22 12:58 111212drw------- 262 root root 4.0K Jun 22 12:31 .glusterfsdrwxr-xr-x 2 root root 4.0K Jun 22 12:31 .glusterfs-anonymous-inode-ad32a409-1b0f-4bd6-9420-acd93b0e1a14-rw-r--r-- 2 root root 0 Jun 22 12:46 test_file |
| --- | --- |

**/etc/ansible/playbooks/roles_play/glusterfs.yml**

|  | - name_of_gluster_tom: gluster-tom |
| --- | --- |

ответ примерно такой

|  | root@debian:~/glusterfs-provision# helm install glusterfs-client olli-ai/glusterfs-client-provisioner \--namespace kube-system \--set glusterfs.server="{192.168.1.109,192.168.1.110}" \--set glusterfs.volume=/NAME: glusterfs-clientLAST DEPLOYED: Sat Jul 6 13:00:39 2024NAMESPACE: kube-systemSTATUS: deployedREVISION: 1TEST SUITE: None |
| --- | --- |

проверяем

|  | root@debian:~/glusterfs-provision# kubectl get storageclasses.storage.k8s.ioNAME PROVISIONER RECLAIMPOLICY VOLUMEBINDINGMODE ALLOWVOLUMEEXPANSION AGEglusterfs-client cluster.local/glusterfs-client-glusterfs-client-provisioner Delete Immediate true 3m34slocal-storage kubernetes.io/no-provisioner Delete WaitForFirstConsumer false 6d2hnfs-client k8s-sigs.io/nfs-subdir-external-provisioner Delete Immediate false 129m |
| --- | --- |

root@debian:~/glusterfs-provision#**kubectl apply -f test-claim.yaml**

|  | root@debian:~/glusterfs-provision# kubectl get pvcNAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGEtest-claim Bound pvc-2c3d964b-ceb5-482b-a8a0-902ff3117c51 50Mi RWX nfs-client 112mtest-claim-gluster Bound pvc-6ac72dee-d061-459b-8dc1-b9fb90e249ae 50Mi RWX glusterfs-client 9sroot@debian:~/glusterfs-provision# kubectl get pvNAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORAGECLASS REASON AGEpvc-2c3d964b-ceb5-482b-a8a0-902ff3117c51 50Mi RWX Delete Bound default/test-claim nfs-client 112mpvc-6ac72dee-d061-459b-8dc1-b9fb90e249ae 50Mi RWX Delete Bound default/test-claim-gluster glusterfs-client 15s |
| --- | --- |

### []Seaweedfs provisioner

сразу скажу что есть проблемы с подключением по fuse

- не работает chown (пришлось отключать у grafana init контейнер)
- если делать репликацию 2 то тогда metadata у vmsingle victoria-metrics не работает

так что как то так - для victoria-metrics не подошло, возможно отстрелит ещё что то.

https://github.com/seaweedfs/seaweedfs-csi-driver/tree/master/deploy/helm/seaweedfs-csi-driver

ставим сам seaweedfs

/etc/ansible/hosts

|  | [seaweed:children]seaweed_masterseaweed_filerseaweed_volume[seaweed_master]192.168.1.121192.168.1.122192.168.1.123[seaweed_filer]192.168.1.124192.168.1.125[seaweed_volume]192.168.1.126192.168.1.127 |
| --- | --- |

root@ansible:/etc/ansible#**ansible-playbook playbooks/roles_play/seaweedfs.yml --ask-pass**

теперь ставим provisioner

/etc/ansible/kubespray-official/seaweedfs-provision/values.yaml

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253 | # Значение seaweedfsFiler должно указывать на все доступные Filer-сервера# Формат: "<host1>:<port>,<host2>:<port>,…"seaweedfsFiler: "192.168.1.124:8888,192.168.1.125:8888"# Имя StorageClass, через который Pods будут брать PVstorageClassName: "seaweedfs-storage"# Сделать ли этот StorageClass по-умолчанию в кластереisDefaultStorageClass: false# (опционально) Если у вас TLS между CSI-плагином и Filer-ами,# можно указать здесь имя Kubernetes Secret с сертификатомtlsSecret: ""# Настройки образов CSI-компонентов можно оставить по-умолчаниюimagePullPolicy: "IfNotPresent"csiProvisioner: image: registry.k8s.io/sig-storage/csi-provisioner:v3.5.0csiResizer: image: registry.k8s.io/sig-storage/csi-resizer:v1.8.0csiAttacher: enabled: true image: registry.k8s.io/sig-storage/csi-attacher:v4.3.0csiNodeDriverRegistrar: image: registry.k8s.io/sig-storage/csi-node-driver-registrar:v2.8.0csiLivenessProbe: image: registry.k8s.io/sig-storage/livenessprobe:v2.10.0seaweedfsCsiPlugin: image: chrislusf/seaweedfs-csi-driver:latest securityContext: privileged: true capabilities: add: ["SYS_ADMIN"] allowPrivilegeEscalation: truedriverName: seaweedfs-csi-drivercontroller: replicas: 1# Опция dataLocality можно оставить none,# если не требуется привязка к топологиям нодdataLocality: "none"node: enabled: true |
| --- | --- |

**helm repo add seaweedfs-csi-driver https://seaweedfs.github.io/seaweedfs-csi-driver/helm**

**helm repo update**

**helm install seaweedfs-csi-driver seaweedfs-csi-driver/seaweedfs-csi-driver --namespace seaweedfs-csi-driver --create-namespace -f values.yaml --version 0.2.2**

для проверки можем использовать:

/etc/ansible/kubespray-official/seaweedfs-provision/pvc-test.yaml

|  | apiVersion: v1kind: PersistentVolumeClaimmetadata: name: seaweedfs-csi-pvcspec: accessModes: - ReadWriteOnce resources: requests: storage: 5Gi storageClassName: seaweedfs-storage |
| --- | --- |

kubectl apply -f pvc-test.yaml -n seaweedfs-csi-driver

|  | root@kub-master1:~/seaweedfs-provision# kubectl get pvc -n seaweedfs-csi-driver NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS VOLUMEATTRIBUTESCLASS AGEseaweedfs-csi-pvc Bound pvc-d4435cb0-2469-45bc-9d4e-79507eda7dc6 5Gi RWO seaweedfs-storage <unset> 10s |
| --- | --- |

### []Prometheus, grafana, alertmanager

копируем values который будем использовать для развёртывания prom stack

[root@ansible ansible]#**scp -r /etc/ansible/kubespray-official/prometheus root@192.168.1.120:~/**

далее на нашемPCс которого есть доступ до кластера ставим всё необходимое:
namespace:
root@debian:~#**kubectl create ns monitoring**

репозиторий
root@debian:~#**helm repo add prometheus-community https://prometheus-community.github.io/helm-charts**
root@debian:~#**helm repo update**
смотрим версии чартов
root@debian:~#**helm search repo prometheus**

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647 | NAME CHART VERSION APP VERSION DESCRIPTIONprometheus-community/kube-prometheus-stack 61.2.0 v0.75.0 kube-prometheus-stack collects Kubernetes manif...prometheus-community/prometheus 25.22.1 v2.53.0 Prometheus is a monitoring system and time seri...prometheus-community/prometheus-adapter 4.10.0 v0.11.2 A Helm chart for k8s prometheus adapterprometheus-community/prometheus-blackbox-exporter 8.17.0 v0.25.0 Prometheus Blackbox Exporterprometheus-community/prometheus-cloudwatch-expo... 0.25.3 0.15.5 A Helm chart for prometheus cloudwatch-exporterprometheus-community/prometheus-conntrack-stats... 0.5.10 v0.4.18 A Helm chart for conntrack-stats-exporterprometheus-community/prometheus-consul-exporter 1.0.0 0.4.0 A Helm chart for the Prometheus Consul Exporterprometheus-community/prometheus-couchdb-exporter 1.0.0 1.0 A Helm chart to export the metrics from couchdb...prometheus-community/prometheus-druid-exporter 1.1.0 v0.11.0 Druid exporter to monitor druid metrics with Pr...prometheus-community/prometheus-elasticsearch-e... 6.0.0 v1.7.0 Elasticsearch stats exporter for Prometheusprometheus-community/prometheus-fastly-exporter 0.4.0 v8.1.0 A Helm chart for the Prometheus Fastly Exporterprometheus-community/prometheus-ipmi-exporter 0.4.0 v1.8.0 This is an IPMI exporter for Prometheus.prometheus-community/prometheus-json-exporter 0.13.0 v0.6.0 Install prometheus-json-exporterprometheus-community/prometheus-kafka-exporter 2.10.0 v1.7.0 A Helm chart to export the metrics from Kafka i...prometheus-community/prometheus-memcached-exporter 0.3.2 v0.14.3 Prometheus exporter for Memcached metricsprometheus-community/prometheus-modbus-exporter 0.1.2 0.4.1 A Helm chart for prometheus-modbus-exporterprometheus-community/prometheus-mongodb-exporter 3.5.0 0.40.0 A Prometheus exporter for MongoDB metricsprometheus-community/prometheus-mysql-exporter 2.5.3 v0.15.1 A Helm chart for prometheus mysql exporter with...prometheus-community/prometheus-nats-exporter 2.17.0 0.15.0 A Helm chart for prometheus-nats-exporterprometheus-community/prometheus-nginx-exporter 0.2.1 0.11.0 A Helm chart for the Prometheus NGINX Exporterprometheus-community/prometheus-node-exporter 4.37.0 1.8.1 A Helm chart for prometheus node-exporterprometheus-community/prometheus-opencost-exporter 0.1.1 1.108.0 Prometheus OpenCost Exporterprometheus-community/prometheus-operator 9.3.2 0.38.1 DEPRECATED - This chart will be renamed. See ht...prometheus-community/prometheus-operator-admiss... 0.14.0 0.75.1 Prometheus Operator Admission Webhookprometheus-community/prometheus-operator-crds 13.0.1 v0.75.0 A Helm chart that collects custom resource defi...prometheus-community/prometheus-pgbouncer-exporter 0.3.0 v0.8.0 A Helm chart for prometheus pgbouncer-exporterprometheus-community/prometheus-pingdom-exporter 2.5.0 20190610-1 A Helm chart for Prometheus Pingdom Exporterprometheus-community/prometheus-pingmesh-exporter 0.4.0 v1.2.1 Prometheus Pingmesh Exporterprometheus-community/prometheus-postgres-exporter 6.0.0 v0.15.0 A Helm chart for prometheus postgres-exporterprometheus-community/prometheus-pushgateway 2.14.0 v1.9.0 A Helm chart for prometheus pushgatewayprometheus-community/prometheus-rabbitmq-exporter 1.12.0 v0.29.0 Rabbitmq metrics exporter for prometheusprometheus-community/prometheus-redis-exporter 6.3.0 v1.61.0 Prometheus exporter for Redis metricsprometheus-community/prometheus-smartctl-exporter 0.10.0 v0.12.0 A Helm chart for Kubernetesprometheus-community/prometheus-snmp-exporter 5.5.0 v0.26.0 Prometheus SNMP Exporterprometheus-community/prometheus-sql-exporter 0.1.0 v0.5.4 Prometheus SQL Exporterprometheus-community/prometheus-stackdriver-exp... 4.5.1 v0.15.1 Stackdriver exporter for Prometheusprometheus-community/prometheus-statsd-exporter 0.13.1 v0.26.1 A Helm chart for prometheus stats-exporterprometheus-community/prometheus-systemd-exporter 0.2.2 0.6.0 A Helm chart for prometheus systemd-exporterprometheus-community/prometheus-to-sd 0.4.2 0.5.2 Scrape metrics stored in prometheus format and ...prometheus-community/prometheus-windows-exporter 0.3.1 0.25.1 A Helm chart for prometheus windows-exporterprometheus-community/alertmanager 1.11.0 v0.27.0 The Alertmanager handles alerts sent by client ...prometheus-community/alertmanager-snmp-notifier 0.3.0 v1.5.0 The SNMP Notifier handles alerts coming from Pr...prometheus-community/jiralert 1.7.1 v1.3.0 A Helm chart for Kubernetes to install jiralertprometheus-community/kube-state-metrics 5.21.0 2.12.0 Install kube-state-metrics to generate and expo...prometheus-community/prom-label-proxy 0.9.0 v0.10.0 A proxy that enforces a given label in a given ... |
| --- | --- |

мы будем использовать prometheus-community/kube-prometheus-stack версии 61.2.0

root@debian:~#**cd prometheus/**

в файле**my-values.yaml**правим доменные имена:

alertmanager.test.local

grafana.test.local

prometheus.test.local

дальше правим

**storageClassName**в моём случае это**nfs-client**

|  | root@debian:~/prometheus# kubectl get storageclasses.storage.k8s.ioNAME PROVISIONER RECLAIMPOLICY VOLUMEBINDINGMODE ALLOWVOLUMEEXPANSION AGEglusterfs-client cluster.local/glusterfs-client-glusterfs-client-provisioner Delete Immediate true 17hlocal-storage kubernetes.io/no-provisioner Delete WaitForFirstConsumer false 6d19hnfs-client k8s-sigs.io/nfs-subdir-external-provisioner Delete Immediate false 19h |
| --- | --- |

выставляем необходимые объёмы для persistant volume

в полях storage

и size

так же если мы хотим собирать метрики только с определённых неймспейсов мы можем использовать функционал**serviceMonitorNamespaceSelector**

|  | prometheusSpec: serviceMonitorNamespaceSelector: matchLabels: prometheus: enabled |
| --- | --- |

т.е. нужно будет добавлять label к тем неймспейсам с которых хотим собирать метрики

применимCRD

|  | kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.75.0/example/prometheus-operator-crd/monitoring.coreos.com_alertmanagerconfigs.yaml --force-conflictskubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.75.0/example/prometheus-operator-crd/monitoring.coreos.com_alertmanagers.yaml --force-conflictskubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.75.0/example/prometheus-operator-crd/monitoring.coreos.com_podmonitors.yaml --force-conflictskubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.75.0/example/prometheus-operator-crd/monitoring.coreos.com_probes.yaml --force-conflictskubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.75.0/example/prometheus-operator-crd/monitoring.coreos.com_prometheusagents.yaml --force-conflictskubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.75.0/example/prometheus-operator-crd/monitoring.coreos.com_prometheuses.yaml --force-conflictskubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.75.0/example/prometheus-operator-crd/monitoring.coreos.com_prometheusrules.yaml --force-conflictskubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.75.0/example/prometheus-operator-crd/monitoring.coreos.com_scrapeconfigs.yaml --force-conflictskubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.75.0/example/prometheus-operator-crd/monitoring.coreos.com_servicemonitors.yaml --force-conflictskubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.75.0/example/prometheus-operator-crd/monitoring.coreos.com_thanosrulers.yaml --force-conflicts |
| --- | --- |

после этого можно запускать установку:

root@debian:~/prometheus#**helm install prometheus prometheus-community/kube-prometheus-stack --version 61.2.0 -n monitoring -f my-values.yaml**

|  | root@debian:~/prometheus# helm install prometheus prometheus-community/kube-prometheus-stack --version 61.2.0 -n monitoring -f my-values.yamlNAME: prometheusLAST DEPLOYED: Sun Jul 7 06:30:44 2024NAMESPACE: monitoringSTATUS: deployedREVISION: 1NOTES:kube-prometheus-stack has been installed. Check its status by running: kubectl --namespace monitoring get pods -l "release=prometheus"Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator. |
| --- | --- |

ставим леблы на все неймспейсы:

root@debian:~/prometheus#**kubectl label namespace --all "prometheus=enabled"**

|  | root@debian:~/prometheus# kubectl label namespace --all "prometheus=enabled"namespace/cattle-fleet-system labelednamespace/cattle-system labelednamespace/cert-manager labelednamespace/default labelednamespace/fleet-default labelednamespace/fleet-local labelednamespace/ingress-nginx labelednamespace/kube-node-lease labelednamespace/kube-public labelednamespace/kube-system labelednamespace/local labelednamespace/metallb-system labelednamespace/monitoring labeled |
| --- | --- |

отмечу что есть проблема с**prometheus-kube-proxy**он стартует на**127,0,0,1**

а прометеус лезет на айпишник т.е. щимится на ноды а там ни кто не отвечает.

для исправления делаем,**НАМАСТЕРАХ**правим:

**vim /etc/kubernetes/kubeadm-config.yaml**

c

**metricsBindAddress: 127.0.0.1:10249**

на

**metricsBindAddress: 0.0.0.0:10249**

мы воспользуемся скриптом:

**for ip in 192.168.1.112 192.168.1.113 192.168.1.114; do ssh root@$ip "sed -i 's/metricsBindAddress: 127.0.0.1:10249/metricsBindAddress: 0.0.0.0:10249/' /etc/kubernetes/kubeadm-config.yaml"; done**

|  | root@debian:~/prometheus# for ip in 192.168.1.112 192.168.1.113 192.168.1.114; do ssh root@$ip "sed -i 's/metricsBindAddress: 127.0.0.1:10249/metricsBindAddress: 0.0.0.0:10249/' /etc/kubernetes/kubeadm-config.yaml"; done |
| --- | --- |

редактируем конфигмап, так же правим:

metricsBindAddress: 127.0.0.1:10249

на

metricsBindAddress: 0.0.0.0:10249

root@debian:~/prometheus#**kubectl -n kube-system get cm kube-proxy -o yaml | sed 's/metricsBindAddress: 127.0.0.1:10249/metricsBindAddress: 0.0.0.0:10249/' | kubectl apply -f -
**

после чего перезапускаем все kube-proxy:

**for i in `kubectl get pod -n kube-system | grep kube-proxy | awk '{print $1}'`; do kubectl delete pod -n kube-system $i; done**

проверяем:

| 12345678910111213141516171819 | root@debian:~/prometheus# kubectl get pod -n monitoringNAME READY STATUS RESTARTS AGEalertmanager-prometheus-kube-prometheus-alertmanager-0 2/2 Running 0 153mprometheus-grafana-797c9f4574-2fgmk 3/3 Running 0 154mprometheus-kube-prometheus-operator-669b6cb6d6-ntl7t 1/1 Running 0 154mprometheus-kube-state-metrics-fdb4588c9-cxv6n 1/1 Running 0 154mprometheus-prometheus-kube-prometheus-prometheus-0 2/2 Running 0 2m51sprometheus-prometheus-node-exporter-2d9mf 1/1 Running 0 154mprometheus-prometheus-node-exporter-6wq9x 1/1 Running 0 154mprometheus-prometheus-node-exporter-7bddg 1/1 Running 4 154mprometheus-prometheus-node-exporter-8gdjv 1/1 Running 0 154mprometheus-prometheus-node-exporter-pjwcm 1/1 Running 2 (70m ago) 154mprometheus-prometheus-node-exporter-ztqts 1/1 Running 4 (26m ago) 154mroot@debian:~/prometheus# kubectl get ingress -n monitoringNAME CLASS HOSTS ADDRESS PORTS AGEprometheus-grafana nginx grafana.test.local 192.168.1.191 80 154mprometheus-kube-prometheus-alertmanager nginx alertmanager.test.local 192.168.1.191 80 154mprometheus-kube-prometheus-prometheus nginx prometheus.test.local 192.168.1.191 80 154m |
| --- | --- |

http://grafana.test.local/

по умолчанию логин**admin**пароль**prom-operator**

![](/news/sidmidru/article-db03824b64600e5b/image-81.png)

### []Victoria-metrics, grafana, alertmanager

в отличие от prometheus потребляет меньше ресурсов.

**kubectl create ns monitoring**

ставимCRD

**helm repo add vm https://victoriametrics.github.io/helm-charts/**

**helm repo update**

**helm search repo vm/victoria-metrics-operator-crds -l**

|  | NAME CHART VERSION APP VERSION DESCRIPTION vm/victoria-metrics-operator-crds 0.2.1 v0.59.2 VictoriaMetrics Operator CRDs vm/victoria-metrics-operator-crds 0.2.0 v0.59.0 VictoriaMetrics Operator CRDs vm/victoria-metrics-operator-crds 0.1.2 v0.58.0 Victoria Metrics Operator CRDsvm/victoria-metrics-operator-crds 0.1.1 v0.57.0 Victoria Metrics Operator CRDsvm/victoria-metrics-operator-crds 0.1.0 v0.56.0 Victoria Metrics Operator CRDsvm/victoria-metrics-operator-crds 0.0.3 v0.55.0 Victoria Metrics Operator CRDsvm/victoria-metrics-operator-crds 0.0.2 v0.55.0 Victoria Metrics Operator CRDsvm/victoria-metrics-operator-crds 0.0.1 v0.54.1 Victoria Metrics Operator CRDs |
| --- | --- |

**helm install vmoc vm/victoria-metrics-operator-crds -n monitoring --version 0.2.1**

смотрим последние версии

|  | helm search repo vm/victoria-metrics-k8s-stack -lNAME CHART VERSION APP VERSION DESCRIPTION vm/victoria-metrics-k8s-stack 0.52.0 v1.119.0 Kubernetes monitoring on VictoriaMetrics stack....vm/victoria-metrics-k8s-stack 0.51.0 v1.119.0 Kubernetes monitoring on VictoriaMetrics stack....vm/victoria-metrics-k8s-stack 0.50.1 v1.118.0 Kubernetes monitoring on VictoriaMetrics stack....vm/victoria-metrics-k8s-stack 0.50.0 v1.118.0 Kubernetes monitoring on VictoriaMetrics stack....vm/victoria-metrics-k8s-stack 0.49.0 v1.118.0 Kubernetes monitoring on VictoriaMetrics stack....vm/victoria-metrics-k8s-stack 0.48.1 v1.117.1 Kubernetes monitoring on VictoriaMetrics stack.... |
| --- | --- |

создаём сертификат:

/etc/ansible/kubespray-official/victoria-metrics/certs/ca_openssl.cnf

|  | [ v3_ca ]subjectAltName = @alt_names[ alt_names ]DNS.1 = vmsingle.test.localDNS.2 = alertmanager.test.localDNS.3 = grafana.test.localDNS.4 = vmalert.test.local |
| --- | --- |

/etc/ansible/kubespray-official/victoria-metrics/certs/vm_openssl.cnf

| 123456789101112131415161718192021222324 | [ req ]default_bits = 4096prompt = nodefault_md = sha256distinguished_name = dnreq_extensions = req_ext[ dn ]C = RUST = YourStateL = YourCityO = YourOrganizationCN = vmsingle.test.local[ req_ext ]subjectAltName = @alt_names[ alt_names ]DNS.1 = vmsingle.test.localDNS.2 = alertmanager.test.localDNS.3 = grafana.test.localDNS.4 = vmalert.test.local |
| --- | --- |

**openssl genrsa -out ca.key 4096**

**openssl req -x509 -sha256 -new -key ca.key -days 10000 -out ca.crt**

**openssl genrsa -out vmsingle.key 4096**

**openssl req -new -key vmsingle.key -out vmsingle.csr -config vm_openssl.cnf**

**openssl x509 -req -in vmsingle.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out vmsingle.crt -days 10000 -sha256 -extfile ca_openssl.cnf -extensions v3_ca**

добавляем сертификат в секреты

|  | kubectl create secret generic vm-tls \ --from-file=tls.crt=./vmsingle.crt \ --from-file=tls.key=./vmsingle.key \ --from-file=ca.crt=./ca.crt \ -n monitoring |
| --- | --- |

/etc/ansible/kubespray-official/victoria-metrics/values.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210211212213214215216217218219220221222223224225226227228 | defaultRules: create: true rules: general: false kubernetesSystem: false disabled: TooHighChurnRate: true TooManyScrapeErrors: truevictoria-metrics-operator: enabled: true resources: limits: cpu: "500m" memory: "500Mi" requests: cpu: "100m" memory: "100Mi" crd: create: false cleanup: enabled: false operator: disable_prometheus_converter: false enable_converter_ownership: true useCustomConfigReloader: true psp_auto_creation_enabled: falsevmsingle: enabled: true # spec for VMSingle crd rbac: create: true # https://github.com/VictoriaMetrics/operator/blob/master/docs/api.MD#vmsinglespec ingress: enabled: true ingressClassName: nginx annotations: nginx.ingress.kubernetes.io/force-ssl-redirect: "true" path: / hosts: - vmsingle.test.local tls: - secretName: vm-tls hosts: - vmsingle.test.local spec: retentionPeriod: "12d" replicaCount: 1 storage: storageClassName: nfs-client accessModes: - ReadWriteOnce resources: requests: storage: "10Gi" resources: limits: cpu: "2" memory: "1500Mi" requests: cpu: "1" memory: "1000Mi" extraArgs: maxLabelsPerTimeseries: "50" memory.allowedPercent: "90" search.maxUniqueTimeseries: "5000000" search.maxConcurrentRequests: "100" search.maxQueryDuration: "300s" search.maxSeries: "5000000" search.cacheTimestampOffset: "12m" search.logSlowQueryDuration: "25s"vmagent: enabled: true spec: selectAllByDefault: true extraArgs: promscrape.streamParse: "true" promscrape.dropOriginalLabels: "true" promscrape.maxScrapeSize: "3355443200" resources: limits: cpu: "500m" memory: "500Mi" requests: cpu: "100m" memory: "100Mi"kube-state-metrics: enabled: true resources: limits: cpu: "500m" memory: "500Mi" requests: cpu: "100m" memory: "100Mi"kubeScheduler: enabled: falsekubeControllerManager: enabled: falsekubeEtcd: enabled: falsekubeProxy: enabled: falsegrafana: enabled: true adminPassword: "Secret123" deploymentStrategy: type: Recreate rbac: pspUseAppArmor: false ingress: enabled: true ingressClassName: nginx annotations: nginx.ingress.kubernetes.io/force-ssl-redirect: "true" path: / hosts: - grafana.test.local tls: - secretName: vm-tls hosts: - grafana.test.local env: GF_SERVER_ROOT_URL: https://grafana.test.local GF_USERS_ALLOW_SIGN_UP: false resources: limits: cpu: "500m" memory: "500Mi" requests: cpu: "100m" memory: "200Mi" persistence: enabled: true type: pvc size: "5Gi" storageClassName: nfs-client accessModes: - ReadWriteOnce# Alertmanager parametersalertmanager: enabled: true rbac: create: true externalURL: "https://alertmanager.test.local" ingress: enabled: true ingressClassName: nginx annotations: nginx.ingress.kubernetes.io/force-ssl-redirect: "true" path: / hosts: - alertmanager.test.local tls: - secretName: vm-tls hosts: - nginx spec: storage: volumeClaimTemplate: spec: storageClassName: nfs-client accessModes: - ReadWriteOnce resources: requests: storage: "1Gi" resources: limits: cpu: "500m" memory: "500Mi" requests: cpu: "100m" memory: "100Mi"#Vmalert configurationvmalert: enabled: true externalURL: "https://alertmanager.test.local" ingress: enabled: true ingressClassName: nginx annotations: nginx.ingress.kubernetes.io/force-ssl-redirect: "true" path: / hosts: - vmalert.test.local tls: - secretName: vm-tls hosts: - vmalert.test.local spec: resources: limits: cpu: "500m" memory: "500Mi" requests: cpu: "100m" memory: "100Mi"prometheus-node-exporter: enabled: true rbac: create: true resources: limits: cpu: "500m" memory: "500Mi" requests: cpu: "50m" memory: "50Mi" service: enabled: true type: ClusterIP port: 9100 targetPort: 9100 |
| --- | --- |

**helm upgrade --install vmks vm/victoria-metrics-k8s-stack -f values.yaml -n monitoring --version 0.52.0**

|  | kubectl get ingress -n monitoring NAME CLASS HOSTS ADDRESS PORTS AGEvmalert-vmks-victoria-metrics-k8s-stack nginx vmalert.test.local 192.168.1.191 80, 443 26hvmalertmanager-vmks-victoria-metrics-k8s-stack nginx alertmanager.test.local 192.168.1.191 80, 443 26hvmks-grafana nginx grafana.test.local 192.168.1.191 80, 443 26hvmsingle-vmks-victoria-metrics-k8s-stack nginx vmsingle.test.local 192.168.1.191 80, 443 26h |
| --- | --- |

#### []Exporter on node not in the k8s

на нодах которые не в кубере мы можем запустить композник чтобы собирать с них метрики

/etc/ansible/kubespray-official/victoria-metrics/docker-compose-vmagent/prometheus.yml

|  | global: # как часто опрашивать targets scrape_interval: 15s # сколько ждать ответа от target scrape_timeout: 10sscrape_configs: - job_name: 'node-vault' metrics_path: /metrics static_configs: - targets: ['192.168.1.103:9100'] |
| --- | --- |

тут указываем job_name: и группируем его по 'node-vault' , т.е. в списке мы будем получать все ноды под этой группой

тут указываем targets: ['192.168.1.103:9100'] это гдеIPадрес на котором запущен сбор метрик

/etc/ansible/kubespray-official/victoria-metrics/docker-compose-vmagent/docker-compose.yml

| 12345678910111213141516171819202122232425262728293031 | version: '3.5'services: node-exporter: image: prom/node-exporter volumes: - /proc:/host/proc:ro - /sys:/host/sys:ro - /:/rootfs:ro command: - '--path.procfs=/host/proc' - '--path.sysfs=/host/sys' - '--path.rootfs=/rootfs' - '--collector.filesystem.mount-points-exclude' - "^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)" expose: - 9100 restart: always network_mode: host vmagent: image: victoriametrics/vmagent volumes: - ./data/vmagentdata:/vmagentdata - ./prometheus.yml:/etc/prometheus/prometheus.yml command: - '--promscrape.config=/etc/prometheus/prometheus.yml' - '--remoteWrite.url=http://vmsingle.test.local/api/v1/write' - '--remoteWrite.showURL' - '--remoteWrite.tlsInsecureSkipVerify=true' restart: always network_mode: host |
| --- | --- |

тут указываем для vmagent '--remoteWrite.url=http://vmsingle.test.local/api/v1/write' это адрес нашего vmsingle.

если нетуDNSвнутреннего то нужно подкинуть в хосты

cat /etc/hosts | grep vms

192.168.1.191 vmsingle.test.local

всё после этого можно стартовать

**docker-compose up -d**

мы получаем группированные сервера, (группы и ip адреса мы можем выбирать)

![](/news/sidmidru/article-db03824b64600e5b/image-82.png)

![](/news/sidmidru/article-db03824b64600e5b/image-83.png)

### []VMServiceScrape - for ingress controller

чтобы собирать метрики с ingress controller во первых на самом контроллере должно быть включено:

|  | metrics: enabled: true port: 10254 service: enabled: true annotations: prometheus.io/scrape: "true" prometheus.io/port: "10254" |
| --- | --- |

а чтобы метрики попали в victoria-metrics нужно использовать следующий конфиг:

/etc/ansible/kubespray-official/victoria-metrics/scrape-ingress-controller.yaml

| 1234567891011121314151617 | apiVersion: operator.victoriametrics.com/v1beta1kind: VMServiceScrapemetadata: name: ingress-nginx namespace: ingress-nginxspec: selector: matchLabels: app.kubernetes.io/name: ingress-nginx namespaceSelector: matchNames: - ingress-nginx endpoints: - port: metrics path: /metrics interval: 15s |
| --- | --- |

применяем
**kubectl apply -f scrape-ingress-controller.yaml**

далее будут доступны такие метрики как:

nginx_ingress_controller_requests
nginx_ingress_controller_nginx_process_connections
и другие.

### []Metrics servers

root@debian:~/prometheus#**helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/**
root@debian:~/prometheus#**helm upgrade --install metrics-server metrics-server/metrics-server --set args={"--kubelet-insecure-tls"} -n monitoring
**проверяем:

| 123456789101112131415161718192021222324 | root@debian:~/prometheus# kubectl top pod -n monitoringNAME CPU(cores) MEMORY(bytes)alertmanager-prometheus-kube-prometheus-alertmanager-0 1m 43Mimetrics-server-dff86cf4f-t5g2h 7m 21Miprometheus-grafana-797c9f4574-2fgmk 2m 239Miprometheus-kube-prometheus-operator-669b6cb6d6-ntl7t 2m 29Miprometheus-kube-state-metrics-fdb4588c9-cxv6n 4m 21Miprometheus-prometheus-kube-prometheus-prometheus-0 46m 435Miprometheus-prometheus-node-exporter-2d9mf 5m 13Miprometheus-prometheus-node-exporter-6wq9x 2m 12Miprometheus-prometheus-node-exporter-7bddg 2m 16Miprometheus-prometheus-node-exporter-8gdjv 5m 12Miprometheus-prometheus-node-exporter-pjwcm 2m 18Miprometheus-prometheus-node-exporter-ztqts 2m 18Miroot@debian:~/prometheus# kubectl top nodeNAME CPU(cores) CPU% MEMORY(bytes) MEMORY%kub-master1.test.local 226m 7% 2332Mi 83%kub-master2.test.local 217m 7% 2530Mi 90%kub-master3.test.local 235m 7% 2321Mi 82%kub-worker1.test.local 158m 3% 2334Mi 62%kub-worker2.test.local 127m 3% 2186Mi 58%kub-worker3.test.local 125m 3% 2034Mi 54% |
| --- | --- |

### []ПравимCOREDNSиLOCALDNSчтобы работал resolv

**cat > coredns.yaml**

| 1234567891011121314151617181920212223242526272829303132 | apiVersion: v1kind: ConfigMapmetadata: name: coredns namespace: kube-systemdata: Corefile: | test.local:53 { forward . 192.168.1.100:53 192.168.1.101:53 } .:53 { errors health { lameduck 5s } ready kubernetes cluster.local in-addr.arpa ip6.arpa { pods insecure fallthrough in-addr.arpa ip6.arpa } prometheus :9153 forward . 192.168.1.100 192.168.1.101 8.8.8.8 8.8.4.4 { prefer_udp max_concurrent 1000 } cache 30 loop reload loadbalance } |
| --- | --- |

**kubectl apply -f coredns.yaml**

тут мы указали test.local чтоб искал ip в нашем freeipa 192.168.1.100 192.168.1.101

ещё правил nodelocaldns

**kubectl edit configmaps -n kube-system nodelocaldns**

| 12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455 | apiVersion: v1data: Corefile: | test.local:53 { errors cache 30 bind 169.254.25.10 forward . 192.168.1.100 192.168.1.101 } cluster.local:53 { errors cache { success 9984 30 denial 9984 5 } reload loop bind 169.254.25.10 forward . 10.233.0.3 { force_tcp } prometheus :9253 health 169.254.25.10:9254 } in-addr.arpa:53 { errors cache 30 reload loop bind 169.254.25.10 forward . 10.233.0.3 { force_tcp } prometheus :9253 } ip6.arpa:53 { errors cache 30 reload loop bind 169.254.25.10 forward . 10.233.0.3 { force_tcp } prometheus :9253 } .:53 { errors cache 30 reload loop bind 169.254.25.10 forward . 8.8.8.8 8.8.4.4 prometheus :9253 } |
| --- | --- |

мы добавляем туда:

|  | test.local:53 { errors cache 30 bind 169.254.25.10 forward . 192.168.1.100 192.168.1.101 } |
| --- | --- |

и рестартуем:

**kubectl rollout restart -n kube-system daemonset nodelocaldns**

### []Логирование - elk

[root@ansible ansible]#**scp -r /etc/ansible/kubespray-official/elk/ root@192.168.1.120:~/**

заходим на нашего клиента

root@debian:~#**cd elk/**

создаём неймспейс

root@debian:~/elk#**kubectl create ns elk**

добавляем репозиторий

root@debian:~/elk#**helm repo add elastic https://helm.elastic.co**

сгенерим сертификаты на основе который всё будет работать

root@debian:~/elk#**cd certs/**

чтоб не захламлять систему собирать всё будем в докере

ставим докер, если его нет:

**apt-get update**
**apt-get install ca-certificates curl**
**install -m 0755 -d /etc/apt/keyrings**
**curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc**
**chmod a+r /etc/apt/keyrings/docker.asc**

|  | echo \ "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \ $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \ tee /etc/apt/sources.list.d/docker.list > /dev/null |
| --- | --- |

**apt-get update**

root@debian:~/elk/certs#**apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y**

root@debian:~/elk/certs#**systemctl start docker.socket**

командой ниже будет сгенерены самоподписанные сертификаты имена задаём через команду --dns

и выполняем команду ниже:

|  | docker run --name elastic-helm-charts-certs -i -w /tmp \ elasticsearch:8.5.1 \ /bin/sh -c " \ elasticsearch-certutil ca --out /tmp/elastic-stack-ca.p12 --pass '' && \ elasticsearch-certutil cert --name security-master --dns elasticsearch-master --dns kibana-kibana --dns kibana.test.local --dns elasticsearch-master* --dns logstash-logstash-headless --dns logstash* --days 7000 --ca /tmp/elastic-stack-ca.p12 --pass '' --ca-pass '' --out /tmp/elastic-certificates.p12" && \docker cp elastic-helm-charts-certs:/tmp/elastic-certificates.p12 ./ && \docker rm -f elastic-helm-charts-certs && \openssl pkcs12 -nodes -passin pass:'' -in elastic-certificates.p12 -out elastic-certificate.pem && \openssl x509 -outform der -in elastic-certificate.pem -out elastic-certificate.crt && \kubectl create secret -n elk generic elastic-certificates --from-file=elastic-certificates.p12 && \kubectl create secret -n elk generic elastic-certificate-pem --from-file=elastic-certificate.pem && \kubectl create secret -n elk generic elastic-certificate-crt --from-file=elastic-certificate.crt |
| --- | --- |

проверяем что сертификаты созданы

|  | root@debian:~/elk/certs# kubectl get secrets -n elkNAME TYPE DATA AGEelastic-certificate-crt Opaque 1 60selastic-certificate-pem Opaque 1 60selastic-certificates Opaque 1 60s |
| --- | --- |

root@debian:~/elk/certs#**openssl pkey -in elastic-certificate.pem -out cert.key**
root@debian:~/elk/certs#**openssl crl2pkcs7 -nocrl -certfile elastic-certificate.pem | openssl pkcs7 -print_certs -out cert.crt**

root@debian:~/elk/certs#**kubectl create secret tls tls-kibana --namespace elk --key cert.key --cert cert.crt**

root@debian:~/elk/certs#**cd ../**
правим файл**elastic.yaml**

можем задать хранилище, я буду использовать

storageClassName:**nfs-client**

можем задать оперативку и т.д. в данном примере я оставлю всё по дефолту

root@debian:~/elk#**helm upgrade --install elasticsearch elastic/elasticsearch --version 8.5.1 -n elk -f elastic.yaml**

дожидаемся установки, проверяем:

|  | root@debian:~# kubectl get pod -n elk -o wideNAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATESelasticsearch-master-0 1/1 Running 0 56m 10.233.79.23 kub-worker3.test.local <none> <none>elasticsearch-master-1 1/1 Running 0 56m 10.233.67.240 kub-worker1.test.local <none> <none>elasticsearch-master-2 1/1 Running 0 56m 10.233.107.87 kub-worker2.test.local <none> <none> |
| --- | --- |

настраиваем**kibana**, для этого в файле**kibana.yaml**правим домен на наш kibana.test.local и ставим:

root@debian:~/elk#**helm upgrade --install kibana elastic/kibana -n elk -f kibana.yaml**

ставим**logstash**

root@debian:~/elk#**helm upgrade --install logstash elastic/logstash -n elk -f logstash.yaml**

ставим**flebeat**

root@debian:~/elk#**helm upgrade --install filebeat elastic/filebeat -n elk -f filebeat.yaml**

проверяем

|  | root@debian:~/elk# kubectl get pod -n elkNAME READY STATUS RESTARTS AGEelasticsearch-master-0 1/1 Running 0 17melasticsearch-master-1 1/1 Running 1 (18m ago) 86melasticsearch-master-2 1/1 Running 1 (17m ago) 140mfilebeat-filebeat-25grp 1/1 Running 3 (17m ago) 67mfilebeat-filebeat-dvwkz 1/1 Running 2 (17m ago) 67mfilebeat-filebeat-fd9q6 1/1 Running 1 (18m ago) 67mkibana-kibana-57c7f6b879-7vfm8 1/1 Running 1 (18m ago) 80mlogstash-logstash-0 1/1 Running 0 8m25s |
| --- | --- |

|  | root@debian:~# kubectl get ingress -n elkNAME CLASS HOSTS ADDRESS PORTS AGEkibana-kibana nginx kibana.test.local 192.168.1.191 80, 443 5m58s |
| --- | --- |

смотрим пароль:

root@debian:~/elk#**kubectl get secrets --namespace=elk elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d**
**k57UN6bFQ01tLjhU**

заходим

**https://kibana.test.local/**

в качестве логина используем**elastic**
в качестве пароля**k57UN6bFQ01tLjhU**

![](/news/sidmidru/article-db03824b64600e5b/image-84.png)

![](/news/sidmidru/article-db03824b64600e5b/image-85.png)

проверим что данные попадают в индекс:

![](/news/sidmidru/article-db03824b64600e5b/image-86.png)

![](/news/sidmidru/article-db03824b64600e5b/image-87.png)

чтобы посмотреть логи нужно создать**data views**

![](/news/sidmidru/article-db03824b64600e5b/image-88.png)

![](/news/sidmidru/article-db03824b64600e5b/image-89.png)

проверяем

![](/news/sidmidru/article-db03824b64600e5b/image-90.png)

![](/news/sidmidru/article-db03824b64600e5b/image-91.png)

создадим пользователя с правами админа

![](/news/sidmidru/article-db03824b64600e5b/image-92.png)

![](/news/sidmidru/article-db03824b64600e5b/image-93.png)

всё, можно логиниться под новыми кредами admin Secret123

### []Loki

в качестве альтернативы рассмотрим использование loki, в качестве хранилища у нас будет s3

Рассмотрим 2 варианта
1 когда s3-minio использует логин пароль (без ldap)
2 когда для s3-minio включён ldap

### **[]1 Вариант - у s3 minioНЕвключён ldap и мы создаём обычного пользователя.**

создаём бакет

![](/news/sidmidru/article-db03824b64600e5b/image-94.png)

![](/news/sidmidru/article-db03824b64600e5b/image-95.png)

создаём policy:

![](/news/sidmidru/article-db03824b64600e5b/image-96.png)

![](/news/sidmidru/article-db03824b64600e5b/image-97.png)

| 1234567891011121314151617181920212223242526272829 | { "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Action": [ "s3:ListBucket", "s3:ListBucketMultipartUploads", "s3:GetBucketLocation" ], "Resource": [ "arn:aws:s3:::loki-bucket" ] }, { "Effect": "Allow", "Action": [ "s3:ListMultipartUploadParts", "s3:PutObject", "s3:AbortMultipartUpload", "s3:DeleteObject", "s3:GetObject" ], "Resource": [ "arn:aws:s3:::loki-bucket/*" ] } ]} |
| --- | --- |

создаём пользователя с нашими access-key и secret-key

![](/news/sidmidru/article-db03824b64600e5b/image-98.png)

![](/news/sidmidru/article-db03824b64600e5b/image-99.png)

создаём ключи доступа

- **Access Key (Ключ доступа):**Уникальный идентификатор пользователя.
- **Secret Key (Секретный ключ):**Пароль для доступа.

![](/news/sidmidru/article-db03824b64600e5b/image-100.png)

![](/news/sidmidru/article-db03824b64600e5b/image-101.png)

![](/news/sidmidru/article-db03824b64600e5b/image-102.png)

получаем

**access key**iXToAdDvt0gx0fEJ1bXl
**secret key**8bpWxY7fNKi0zEL85UI9Sg22floLjqMhPdxs2kVS

проверяем что нужный к нашему бакету добавлен нужный пользователь

![](/news/sidmidru/article-db03824b64600e5b/image-103.png)

теперь приступим к настройкеLOKI

вот наш values

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210 | loki: server: grpc_server_max_recv_msg_size: 8388608 grpc_server_max_send_msg_size: 8388608 useTestSchema: false auth_enabled: false structuredConfig: memberlist: cluster_label: "loki" schema_config: configs: - from: "2020-09-07" store: tsdb object_store: "s3" schema: "v13" index: period: "24h" prefix: "loki_index_" limits_config: retention_period: 14d ingestion_rate_mb: 4 ingestion_burst_size_mb: 6 # compactor: # shared_store: s3 # compaction_interval: 10m # retention_enabled: true # retention_delete_delay: 2h # retention_delete_worker_count: 150 storage: # filesystem: null # type: s3 bucketNames: chunks: loki-bucket ruler: loki-bucket admin: loki-bucket s3: endpoint: http://192.168.1.120:9000 # http://s3-minio.test.local:9000 accessKeyId: "ARMMLGAU2B2Q4K7H92OU" secretAccessKey: "4AkkOeozat6SRS7yZbdIwuWnND3wa1LD3q+4vtmm" insecure: true s3ForcePathStyle: true region: null sse_encryption: false http_config: idle_conn_timeout: 90s response_header_timeout: 0s insecure_skip_verify: true ingester: autoforget_unhealthy: true storage_config: memcached: chunk_cache: enabled: false results_cache: enabled: false commonConfig: replication_factor: 1serviceAccount: create: true name: loki-service-account annotations: {}monitoring: dashboards: enabled: false rules: enabled: true alerting: true labels: release: kube-prometheus-stack serviceMonitor: enabled: true labels: release: kube-prometheus-stack interval: 15s lokiCanary: resources: requests: cpu: 100m memory: 256Mi limits: cpu: 200m memory: 512Mi selfMonitoring: grafanaAgent: resources: requests: cpu: 200m memory: 256Mi limits: cpu: 500m memory: 512Miwrite: replicas: 1 extraArgs: - "-log.level=info" resources: requests: cpu: 100m memory: 300Mi limits: cpu: 500m memory: 800Mi persistence: enabled: true storageClass: nfs-client size: 10Giread: replicas: 1 extraArgs: - "-log.level=info" resources: requests: cpu: 100m memory: 300Mi limits: cpu: 100m memory: 300Mi # persistence: # enabled: false # storageClass: nfs-client # size: 10Gibackend: replicas: 1 extraArgs: - "-log.level=info" resources: requests: cpu: 100m memory: 300Mi limits: cpu: 100m memory: 300Mi persistence: enabled: true storageClass: nfs-client size: 10GilokiCanary: enabled: false resources: requests: cpu: 100m memory: 300Mi limits: cpu: 100m memory: 300Migateway: replicas: 1 resources: requests: cpu: 100m memory: 300Mi limits: cpu: 100m memory: 300Mi nginxConfig: resolver: "coredns.kube-system.svc.cluster.local"# memberlist:# join_members:# - loki-memberlist.loki.svc.cluster.localmemberlist: service: publishNotReadyAddresses: false join_members: - loki-memberlist.loki.svc.cluster.local:7946 #- loki-memberlist.loki.svc.cluster.local# Disable cachesmemcached: enabled: false# If specific sections are usedmemcached_chunks: enabled: falsememcached_frontend: enabled: falsechunksCache: enabled: falseresultsCache: enabled: falsesidecar: rules: enabled: falsetest: enabled: falseenterprise: enabled: false# gateway:# enabled: false |
| --- | --- |

в этом values endpoint используемIPа не хостнейм потому что pod loki backend не резолвит домен - хотя все остальные pod-ы нормально резолвят домены с freeipa

root@debian:~#**helm repo add grafana https://grafana.github.io/helm-charts**
root@debian:~#**helm repo update**

**kubectl create ns loki**

**helm upgrade --install loki grafana/loki --namespace loki --version 6.30.1 -f loki.yaml**

### **[]2. Второй вариант когда у s3-minio включён ldap**

идём во**https://freeipa-1.test.local/**и создаём пользователя**user-loki**

![](/news/sidmidru/article-db03824b64600e5b/image-104.png)

и добавляем его в группу

![](/news/sidmidru/article-db03824b64600e5b/image-105.png)

проверяем что ldap нормально всё подтянул, для этого идём в s3-minio

**http://s3-minio.test.local:9001/**

![](/news/sidmidru/article-db03824b64600e5b/image-106.png)

как видим всё ок. пользователь подтянулся.

теперь нам надо назначить ему policy которую мы создавали ранее, вот она:

![](/news/sidmidru/article-db03824b64600e5b/image-107.png)

| 1234567891011121314151617181920212223242526272829 | { "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Action": [ "s3:ListBucket", "s3:ListBucketMultipartUploads", "s3:GetBucketLocation" ], "Resource": [ "arn:aws:s3:::loki-bucket" ] }, { "Effect": "Allow", "Action": [ "s3:ListMultipartUploadParts", "s3:PutObject", "s3:AbortMultipartUpload", "s3:DeleteObject", "s3:GetObject" ], "Resource": [ "arn:aws:s3:::loki-bucket/*" ] } ]} |
| --- | --- |

как видим добавить пользователя мы не можем это происходит потому что включёнLDAP

теперь назначим policy для пользователя, для этого идём на сервер s3-minio

[root@ansible ]#**ssh 192.168.1.118**

root@s3-minio-1:~#

root@s3-minio-1:~#**mc idp ldap policy attach myminio loki-policy --user=user-loki**

ответ должен быть такой:

|  | root@s3-minio-1:~# mc idp ldap policy attach myminio loki-policy --user=user-lokiAttached Policies: [loki-policy]To User: user-loki |
| --- | --- |

а теперь создаём access-key и secret-key

root@s3-minio-1:~#**mc admin user svcacct add myminio user-loki**

|  | root@s3-minio-1:~# mc admin user svcacct add myminio user-lokiAccess Key: ARMMLGAU2B2Q4K7H92OUSecret Key: 4AkkOeozat6SRS7yZbdIwuWnND3wa1LD3q+4vtmmExpiration: no-expiry |
| --- | --- |

всё, теперь правим наш values c новыми access и secret key

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209 | loki: server: grpc_server_max_recv_msg_size: 8388608 grpc_server_max_send_msg_size: 8388608 useTestSchema: false auth_enabled: false structuredConfig: memberlist: cluster_label: "loki" schema_config: configs: - from: "2020-09-07" store: tsdb object_store: "s3" schema: "v13" index: period: "24h" prefix: "loki_index_" limits_config: retention_period: 14d ingestion_rate_mb: 4 ingestion_burst_size_mb: 6 # compactor: # shared_store: s3 # compaction_interval: 10m # retention_enabled: true # retention_delete_delay: 2h # retention_delete_worker_count: 150 storage: # filesystem: null # type: s3 bucketNames: chunks: loki-bucket ruler: loki-bucket admin: loki-bucket s3: endpoint: http://192.168.1.120:9000 # http://s3-minio.test.local:9000 accessKeyId: "ARMMLGAU2B2Q4K7H92OU" secretAccessKey: "4AkkOeozat6SRS7yZbdIwuWnND3wa1LD3q+4vtmm" insecure: true s3ForcePathStyle: true region: null sse_encryption: false http_config: idle_conn_timeout: 90s response_header_timeout: 0s insecure_skip_verify: true ingester: autoforget_unhealthy: true storage_config: memcached: chunk_cache: enabled: false results_cache: enabled: false commonConfig: replication_factor: 1serviceAccount: create: true name: loki-service-account annotations: {}monitoring: dashboards: enabled: false rules: enabled: true alerting: true labels: release: kube-prometheus-stack serviceMonitor: enabled: true labels: release: kube-prometheus-stack interval: 15s lokiCanary: resources: requests: cpu: 100m memory: 256Mi limits: cpu: 200m memory: 512Mi selfMonitoring: grafanaAgent: resources: requests: cpu: 200m memory: 256Mi limits: cpu: 500m memory: 512Miwrite: replicas: 1 extraArgs: - "-log.level=info" resources: requests: cpu: 100m memory: 300Mi limits: cpu: 100m memory: 300Mi persistence: enabled: true storageClass: nfs-client size: 10Giread: replicas: 1 extraArgs: - "-log.level=info" resources: requests: cpu: 100m memory: 300Mi limits: cpu: 100m memory: 300Mi persistence: enabled: true storageClass: nfs-client size: 10Gibackend: replicas: 1 extraArgs: - "-log.level=info" resources: requests: cpu: 100m memory: 300Mi limits: cpu: 100m memory: 300Mi persistence: enabled: true storageClass: nfs-client size: 10GilokiCanary: enabled: false resources: requests: cpu: 100m memory: 300Mi limits: cpu: 100m memory: 300Migateway: replicas: 1 resources: requests: cpu: 100m memory: 300Mi limits: cpu: 100m memory: 300Mi nginxConfig: resolver: "coredns.kube-system.svc.cluster.local"# memberlist:# join_members:# - loki-memberlist.loki.svc.cluster.localmemberlist: service: publishNotReadyAddresses: true join_members: - loki-memberlist.loki.svc.cluster.local# Disable cachesmemcached: enabled: false# If specific sections are usedmemcached_chunks: enabled: falsememcached_frontend: enabled: falsechunksCache: enabled: falseresultsCache: enabled: falsesidecar: rules: enabled: falsetest: enabled: falseenterprise: enabled: false# gateway:# enabled: false |
| --- | --- |

и инсталлим:

root@client:~/LOKI#**helm upgrade --install loki grafana/loki --namespace loki --version 6.30.1 -f loki.yaml**

теперь ставим сборщик логов promtail

### []Loki - s3 bucket (seaweedfs)

заходим на мастер

root@ansible:/etc/ansible#**ssh 192.168.1.121**

подключаемся

root@debian:~#**weed shell**

проверяем что есть:

|  | > s3.bucket.list pvc-5f6c6f0c-5f1a-44ec-be45-0542b83712a4 size:16 chunk:0 pvc-a52116d8-4e15-4f55-8413-5f3e0ba91e90 size:27674376 chunk:415 pvc-ab8f5b44-84d1-4224-b9d4-3a563824f748 size:7299778472 chunk:4061 test-bucket size:200680 chunk:1 |
| --- | --- |

создаём новый бакет loki-bucket:

>**s3.bucket.create --name loki-bucket**

|  | > s3.bucket.create --name loki-bucketcreate bucket under /bucketscreated bucket loki-bucket> s3.bucket.list loki-bucket size:0 chunk:0 pvc-5f6c6f0c-5f1a-44ec-be45-0542b83712a4 size:16 chunk:0 pvc-a52116d8-4e15-4f55-8413-5f3e0ba91e90 size:27794664 chunk:415 pvc-ab8f5b44-84d1-4224-b9d4-3a563824f748 size:7355498792 chunk:4061 test-bucket size:200680 chunk:1 |
| --- | --- |

ок, настраиваем values

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209 | loki: server: grpc_server_max_recv_msg_size: 8388608 grpc_server_max_send_msg_size: 8388608 useTestSchema: false auth_enabled: false structuredConfig: memberlist: cluster_label: "loki" schema_config: configs: - from: "2020-09-07" store: tsdb object_store: "s3" schema: "v13" index: period: "24h" prefix: "loki_index_" limits_config: retention_period: 14d ingestion_rate_mb: 4 ingestion_burst_size_mb: 6 # compactor: # shared_store: s3 # compaction_interval: 10m # retention_enabled: true # retention_delete_delay: 2h # retention_delete_worker_count: 150 storage: # filesystem: null # type: s3 bucketNames: chunks: loki-bucket ruler: loki-bucket admin: loki-bucket s3: endpoint: http://192.168.1.124:8333 # http://s3-minio.test.local:9000 accessKeyId: "dummy" secretAccessKey: "dummy" insecure: true s3ForcePathStyle: true region: null sse_encryption: false http_config: idle_conn_timeout: 90s response_header_timeout: 0s insecure_skip_verify: true ingester: autoforget_unhealthy: true storage_config: memcached: chunk_cache: enabled: false results_cache: enabled: false commonConfig: replication_factor: 1serviceAccount: create: true name: loki-service-account annotations: {}monitoring: dashboards: enabled: false rules: enabled: true alerting: true labels: release: kube-prometheus-stack serviceMonitor: enabled: true labels: release: kube-prometheus-stack interval: 15s lokiCanary: resources: requests: cpu: 100m memory: 256Mi limits: cpu: 200m memory: 512Mi selfMonitoring: grafanaAgent: resources: requests: cpu: 200m memory: 256Mi limits: cpu: 500m memory: 512Miwrite: replicas: 1 extraArgs: - "-log.level=info" resources: requests: cpu: 100m memory: 300Mi limits: cpu: 100m memory: 300Mi persistence: enabled: true storageClass: seaweedfs-storage size: 5Giread: replicas: 1 extraArgs: - "-log.level=info" resources: requests: cpu: 100m memory: 300Mi limits: cpu: 100m memory: 300Mi persistence: enabled: true storageClass: seaweedfs-storage size: 5Gibackend: replicas: 1 extraArgs: - "-log.level=info" resources: requests: cpu: 100m memory: 300Mi limits: cpu: 100m memory: 300Mi persistence: enabled: true storageClass: seaweedfs-storage size: 5GilokiCanary: enabled: false resources: requests: cpu: 100m memory: 300Mi limits: cpu: 100m memory: 300Migateway: replicas: 1 resources: requests: cpu: 100m memory: 300Mi limits: cpu: 100m memory: 300Mi nginxConfig: resolver: "coredns.kube-system.svc.cluster.local"# memberlist:# join_members:# - loki-memberlist.loki.svc.cluster.localmemberlist: service: publishNotReadyAddresses: true join_members: - loki-memberlist.loki.svc.cluster.local# Disable cachesmemcached: enabled: false# If specific sections are usedmemcached_chunks: enabled: falsememcached_frontend: enabled: falsechunksCache: enabled: falseresultsCache: enabled: falsesidecar: rules: enabled: falsetest: enabled: falseenterprise: enabled: false# gateway:# enabled: false |
| --- | --- |

helm upgrade --install loki grafana/loki --namespace loki --version 6.30.1 -f loki-seaweedfs-s3.yaml

### **[]Promtail**

Promtail — агент, который читает и отправляет логи на сервер. Он устанавливается как daemonset так как логи собираются со всех нод в кластере.

| 123456789101112131415161718192021 | daemonset: enabled: trueresources: limits: cpu: 200m memory: 128Mi requests: cpu: 100m memory: 128MiserviceMonitor: enabled: true labels: release: kube-prometheus-stack prometheusRule: enabled: falseconfig: logLevel: info clients: - url: http://loki-gateway.loki.svc.cluster.local/loki/api/v1/push tenant_id: 1 |
| --- | --- |

**helm repo add prometheus-community https://prometheus-community.github.io/helm-charts**

**helm repo update**

**helm install prometheus-crds prometheus-community/prometheus-operator-crds --namespace loki**

root@client:~/LOKI#**helm upgrade --install promtail grafana/promtail --namespace loki --values promtail.yaml**

проверяем:

|  | root@client:~/LOKI# kubectl get pod -n loki -o wideNAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATESloki-backend-0 1/1 Running 1 (18h ago) 18h 10.233.79.43 kub-worker3.test.local <none> <none>loki-gateway-5456868ddc-kdddj 1/1 Running 2 (133m ago) 18h 10.233.79.1 kub-worker3.test.local <none> <none>loki-read-6f49c674b7-pzh69 1/1 Running 1 (18h ago) 18h 10.233.79.60 kub-worker3.test.local <none> <none>loki-write-0 1/1 Running 1 (18h ago) 18h 10.233.79.63 kub-worker3.test.local <none> <none>promtail-6r9g8 1/1 Running 0 103m 10.233.79.10 kub-worker3.test.local <none> <none>promtail-9ft9z 1/1 Running 0 102m 10.233.126.121 kub-master3.test.local <none> <none>promtail-j8wmw 1/1 Running 0 103m 10.233.107.120 kub-worker2.test.local <none> <none>promtail-mz487 1/1 Running 0 103m 10.233.66.194 kub-master1.test.local <none> <none>promtail-w8w52 1/1 Running 0 104m 10.233.122.200 kub-master2.test.local <none> <none>promtail-xrvfl 1/1 Running 0 104m 10.233.67.251 kub-worker1.test.local <none> <none> |
| --- | --- |

как видим promtail установлен на каждой ноде в кластере. Проверим так же наш бакет:

![](/news/sidmidru/article-db03824b64600e5b/image-108.png)

как видим логи собираются.

### []Grafana - Loki

Идём в нашу grafana

http://grafana.test.local/login

настраиваем datasource

![](/news/sidmidru/article-db03824b64600e5b/image-109.png)

![](/news/sidmidru/article-db03824b64600e5b/image-110.png)

для ConnectionURLуказываем

**http://loki-read.loki.svc.cluster.local:3100**

![](/news/sidmidru/article-db03824b64600e5b/image-111.png)

всё этого хватает, дальше скролим вниз и нажимаем save&test

![](/news/sidmidru/article-db03824b64600e5b/image-112.png)

как видим получаем сообщение
Data source successfully connected.

проверяем работу дашборды

![](/news/sidmidru/article-db03824b64600e5b/image-113.png)

можем получить такую проблему:

![](/news/sidmidru/article-db03824b64600e5b/image-114.png)

можно попробовать поковырять настройки самой борды но мне было лень я просто поставил новую

https://grafana.com/grafana/dashboards/15324-loki-logs-dashboard/

вотIDэтой борды 15324

далее импортируем её:

![](/news/sidmidru/article-db03824b64600e5b/image-115.png)

![](/news/sidmidru/article-db03824b64600e5b/image-116.png)

![](/news/sidmidru/article-db03824b64600e5b/image-117.png)

проверяем саму борду

![](/news/sidmidru/article-db03824b64600e5b/image-118.png)

как видим всё ок, за последний час в namespace monitoring логи были.

на этом с логами из loki всё. Можете выбирать что вам удобнейELK/EFK/Grafana-Loki

### []Auto unseal для vault кластера

в чём идея, у нас уже есть кластер vault

vault1.test.local =192.168.1.103

vault2.test.local =192.168.1.104

vault3.test.local =192.168.1.105

нам нужно сделать его авторазблокировку в случае рестарта одной из node

идея в том что в k8s будет так же поднят vault который будет настроен для разблокировки нашего кластера, а vault в кластере будет разблокироваться cronjob.

**ЗАПОМНИТЕ!!!!НЕЛЬЗЯНАСТРАИВАТЬAUTOUNSEALДРУГДРУГА.ЕСЛИВВАШЕМДЦОТКЛЮЧАТСВЕТИ УВАСОДНОВРЕМЕННОВЫКЛЮЧАТЬСЯОБАКЛАСТЕРА,ОНИНЕСМОГУТСТАРТАНУТИРАСПЕЧАТАТЬДРУГДРУГА.**

Отмечу что все используемые домены у нас резолвятся через freeipa или через hosts - кому как удобней

**[]Установка vault в k8s**

не забываем что мы всё ещё используем

**https://github.com/midnight47/ansible-playbook.git**

который я перетащил в /etc/ansible

перевходим в

[root@ansible ~]#**cd /etc/ansible/kubespray-official/vault-autounseal/**

создаём неймспейс

root@client:~#**kubectl create ns vault
**добавляем репозиторй

root@client:~#**helm repo add hashicorp https://helm.releases.hashicorp.com
**устанавливаем vault, отмечу что я использую в качестве бэкенда raft а для него использую диски от провиженера

**nfs-client**так что если у вас будет другой провижинер то используйте его.

root@client:~/vault-autounseal#**helm upgrade --install -n vault vault hashicorp/vault -f values.yaml**

создаём ключи и записываем их в файл:

root@client:~/vault-autounseal#**kubectl -n vault exec vault-0 -- vault operator init -key-shares=5 -key-threshold=3 -format=json > cluster-keys.json**

ставим jq

root@client:~/vault-autounseal#**apt-get install jq -y**

делаем unseal первого vault

root@client:~/vault-autounseal#**for i in `cat cluster-keys.json | jq -r ".unseal_keys_b64[]"`; do kubectl -n vault exec vault-0 -- vault operator unseal $i; done**

собираем raft в кластер:

root@client:~/vault-autounseal#**kubectl -n vault exec -ti vault-1 -- vault operator raft join http://vault-0.vault-internal:8200**

root@client:~/vault-autounseal#**kubectl -n vault exec -ti vault-2 -- vault operator raft join http://vault-0.vault-internal:8200**

делаем unseal для второго и третьего vault

root@client:~/vault-autounseal#**for i in `cat cluster-keys.json | jq -r ".unseal_keys_b64[]"`; do kubectl -n vault exec vault-1 -- vault operator unseal $i; done**

root@client:~/vault-autounseal#**for i in `cat cluster-keys.json | jq -r ".unseal_keys_b64[]"`; do kubectl -n vault exec vault-2 -- vault operator unseal $i; done**

**[]Auto unseal через cronjob**

сделаем сразу cronjob которая будет расшифровывать vault который у нас запущен в кубере:

создаём конфигмап

root@client:~/vault-autounseal#**kubectl create configmap vault-unseal-keys --from-file=cluster-keys.json -n vault**

теперь можем применять нашу cronjob:

root@client:~/vault-autounseal#**kubectl apply -f auto-unseal-cronjob.yaml**

сам скрипт выглядит вот так:

| 12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152 | apiVersion: batch/v1kind: CronJobmetadata: name: vault-unseal-cronjob namespace: vaultspec: schedule: "*/2 * * * *" # Запускать каждые 2 минуты successfulJobsHistoryLimit: 1 # Хранить только последнее успешное задание failedJobsHistoryLimit: 1 # Хранить только последнее неудачное задание jobTemplate: spec: template: spec: containers: - name: vault-unseal image: alpine:latest command: - /bin/sh - -c - | apk add --no-cache curl jq bind-tools netcat-openbsd KEYS=$(jq -r '.unseal_keys_b64[]' /config/cluster-keys.json) for POD in vault-0 vault-1 vault-2 do POD_ADDRESS="$POD.vault-internal.vault.svc.cluster.local" echo "Checking if $POD_ADDRESS is sealed…" SEALED=$(curl -s http://$POD_ADDRESS:8200/v1/sys/health | jq -r '.sealed') if [ "$SEALED" = "true" ]; then echo "$POD_ADDRESS is sealed. Proceeding to unseal." for KEY in $KEYS do RESPONSE=$(curl --silent --request POST --data "{\"key\":\"$KEY\"}" http://$POD_ADDRESS:8200/v1/sys/unseal) echo "Unseal response from $POD: $RESPONSE" SEALED=$(echo "$RESPONSE" | jq -r '.sealed') if [ "$SEALED" = "false" ]; then echo "$POD_ADDRESS is now unsealed." break fi done else echo "$POD_ADDRESS is already unsealed." fi done volumeMounts: - name: config mountPath: /config restartPolicy: OnFailure volumes: - name: config configMap: name: vault-unseal-keys |
| --- | --- |

**[]теперь приступим к настройке autounseal с помощью tranzit**

root@client:~/vault-autounseal#**kubectl exec -it -n vault vault-0 -- sh**

/ $**vault login**

/ $**vault secrets enable transit**
/ $**vault write -f transit/keys/vault-unseal-key**
/ $**cd /tmp/**
/tmp $**cat > transit-policy.yaml**

|  | path "transit/encrypt/vault-unseal-key" { capabilities = [ "update" ]}path "transit/decrypt/vault-unseal-key" { capabilities = [ "update" ]}path "transit/keys/vault-unseal-key" { capabilities = ["read"] } |
| --- | --- |

/tmp $**vault policy write unseal-policy transit-policy.yaml**
/tmp $**vault token create -policy=unseal-policy**

получаем token

|  | /tmp $ vault token create -policy=unseal-policyKey Value--- -----token s.UAiqZBv9lIwDS6ZzOpDdf3XWtoken_accessor kYcVArPi5nVu9o0lKRqdNbNEtoken_duration 768htoken_renewable truetoken_policies ["default" "unseal-policy"]identity_policies []policies ["default" "unseal-policy"] |
| --- | --- |

теперь идём в конфигурации наших vault на виртуалках и добавляем следующее:

|  | seal "transit" { address = "http://vault-unseal.test.local:80" token = "s.UAiqZBv9lIwDS6ZzOpDdf3XW" key_name = "vault-unseal-key" mount_path = "transit/" disable_renewal = "false" tls_skip_verify = "true" } |
| --- | --- |

root@client:~/vault-autounseal#**ssh 192.168.1.103
**root@vault1:~#**nano /etc/vault.d/vault.hcl
**

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950 | cluster_addr = "https://vault1.test.local:8201"api_addr = "https://vault1.test.local:8200"disable_mlock = trueui = truelistener "tcp" { address = "0.0.0.0:8200" tls_ca_cert_file = "/opt/vault/tls/my_crt_file.pem" tls_cert_file = "/opt/vault/tls/my_crt_file.pem" tls_key_file = "/opt/vault/tls/my_crt_file.pem"}storage "raft" { path = "/opt/vault/data" node_id = "vault1.test.local" retry_join { leader_tls_servername = "vault1.test.local" leader_api_addr = "https://vault1.test.local:8200" leader_ca_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_key_file = "/opt/vault/tls/my_crt_file.pem" } retry_join { leader_tls_servername = "vault2.test.local" leader_api_addr = "https://vault2.test.local:8200" leader_ca_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_key_file = "/opt/vault/tls/my_crt_file.pem" } retry_join { leader_tls_servername = "vault3.test.local" leader_api_addr = "https://vault3.test.local:8200" leader_ca_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_key_file = "/opt/vault/tls/my_crt_file.pem" } } seal "transit" { address = "http://vault-unseal.test.local:80" token = "s.UAiqZBv9lIwDS6ZzOpDdf3XW" key_name = "vault-unseal-key" mount_path = "transit/" disable_renewal = "false" tls_skip_verify = "true" } |
| --- | --- |

root@client:~/vault-autounseal#**ssh 192.168.1.104**
root@vault2:~#**nano /etc/vault.d/vault.hcl**

| 12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849 | cluster_addr = "https://vault2.test.local:8201"api_addr = "https://vault2.test.local:8200"disable_mlock = trueui = truelistener "tcp" { address = "0.0.0.0:8200" tls_ca_cert_file = "/opt/vault/tls/my_crt_file.pem" tls_cert_file = "/opt/vault/tls/my_crt_file.pem" tls_key_file = "/opt/vault/tls/my_crt_file.pem"}storage "raft" { path = "/opt/vault/data" node_id = "vault2.test.local" retry_join { leader_tls_servername = "vault1.test.local" leader_api_addr = "https://vault1.test.local:8200" leader_ca_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_key_file = "/opt/vault/tls/my_crt_file.pem" } retry_join { leader_tls_servername = "vault2.test.local" leader_api_addr = "https://vault2.test.local:8200" leader_ca_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_key_file = "/opt/vault/tls/my_crt_file.pem" } retry_join { leader_tls_servername = "vault3.test.local" leader_api_addr = "https://vault3.test.local:8200" leader_ca_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_key_file = "/opt/vault/tls/my_crt_file.pem" } } seal "transit" { address = "http://vault-unseal.test.local:80" token = "s.UAiqZBv9lIwDS6ZzOpDdf3XW" key_name = "vault-unseal-key" mount_path = "transit/" disable_renewal = "false" tls_skip_verify = "true" } |
| --- | --- |

root@client:~/vault-autounseal#**ssh 192.168.1.105**
root@vault3:~#**nano /etc/vault.d/vault.hcl**

| 12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849 | cluster_addr = "https://vault3.test.local:8201"api_addr = "https://vault3.test.local:8200"disable_mlock = trueui = truelistener "tcp" { address = "0.0.0.0:8200" tls_ca_cert_file = "/opt/vault/tls/my_crt_file.pem" tls_cert_file = "/opt/vault/tls/my_crt_file.pem" tls_key_file = "/opt/vault/tls/my_crt_file.pem"}storage "raft" { path = "/opt/vault/data" node_id = "vault3.test.local" retry_join { leader_tls_servername = "vault1.test.local" leader_api_addr = "https://vault1.test.local:8200" leader_ca_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_key_file = "/opt/vault/tls/my_crt_file.pem" } retry_join { leader_tls_servername = "vault2.test.local" leader_api_addr = "https://vault2.test.local:8200" leader_ca_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_key_file = "/opt/vault/tls/my_crt_file.pem" } retry_join { leader_tls_servername = "vault3.test.local" leader_api_addr = "https://vault3.test.local:8200" leader_ca_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_cert_file = "/opt/vault/tls/my_crt_file.pem" leader_client_key_file = "/opt/vault/tls/my_crt_file.pem" } } seal "transit" { address = "http://vault-unseal.test.local:80" token = "s.UAiqZBv9lIwDS6ZzOpDdf3XW" key_name = "vault-unseal-key" mount_path = "transit/" disable_renewal = "false" tls_skip_verify = "true" } |
| --- | --- |

Добавляем в хосты наш сервер(который в кубере)
root@vault1:~#**echo "192.168.1.191 vault-unseal.test.local" >> /etc/hosts**
root@vault2:~#**echo "192.168.1.191 vault-unseal.test.local" >> /etc/hosts**
root@vault3:~#**echo "192.168.1.191 vault-unseal.test.local" >> /etc/hosts**

[root@vault1 ~]#**systemctl restart vault**
[root@vault2 ~]#**systemctl restart vault**
[root@vault3 ~]#**systemctl restart vault**

теперь нужно распечатать наше хранилище с параметром:
'**-migrate**'

после каждой команды нужно ввести ключ для разблокировки

[root@vault1 ~]# vault operator unseal -migrate
[root@vault1 ~]# vault operator unseal -migrate
[root@vault1 ~]# vault operator unseal -migrate

[root@vault2 ~]# vault operator unseal -migrate
[root@vault2 ~]# vault operator unseal -migrate
[root@vault2 ~]# vault operator unseal -migrate

[root@vault3 ~]# vault operator unseal -migrate
[root@vault3 ~]# vault operator unseal -migrate
[root@vault3 ~]# vault operator unseal -migrate

напомню вот мои ключи

|  | 184bdc202f516bee92f449ec14a2e92ca22b05aea7032f74badb77303863cd8aebc9726a282be011b8e21a62ab307d30104e1bea0701c7d9406e05d4ae68b57344af 8034b00602cc135c602025b9117815f0ea1a38d2ce7c812d3b740b0071b60e0696 1304a0e6cea51b5192465d6beccab9efaa7b856f9511c81f9ea6690a43cf9e3d38 4bb452f31133f4724df04db888611cb88b19014f71b32f3fb2ad58d0f7bdbb218a |
| --- | --- |

всё, можем проверять что наш кластер успешно распечатывается:

| 12345678910111213141516171819202122 | root@vault2:~# vault statusWARNING! VAULT_ADDR and -address unset. Defaulting to https://127.0.0.1:8200.Key Value--- -----Seal Type transitRecovery Seal Type shamirInitialized trueSealed falseTotal Recovery Shares 5Threshold 3Version 1.17.0Build Date 2024-06-10T10:11:34ZStorage Type raftCluster Name vault-cluster-011de0c1Cluster ID 69604c77-2116-9a8d-4c2e-fced06b9b578HA Enabled trueHA Cluster https://vault2.test.local:8201HA Mode activeActive Since 2024-11-18T19:58:34.395069102+06:00Raft Committed Index 10335Raft Applied Index 10335 |
| --- | --- |

главный сервер https://vault2.test.local:8201 проверяем что и виртуальный ip на этом сервере:

| 1234567891011121314151617181920212223 | root@vault2:~# ifconfigenp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500 inet 192.168.1.104 netmask 255.255.255.0 broadcast 192.168.1.255 inet6 fe80::a00:27ff:fe6d:1798 prefixlen 64 scopeid 0x20<link> ether 08:00:27:6d:17:98 txqueuelen 1000 (Ethernet) RX packets 418231 bytes 61354583 (58.5 MiB) RX errors 0 dropped 0 overruns 0 frame 0 TX packets 192759 bytes 37057036 (35.3 MiB) TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0enp0s3:vip: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500 inet 192.168.1.111 netmask 255.255.255.255 broadcast 0.0.0.0 ether 08:00:27:6d:17:98 txqueuelen 1000 (Ethernet)lo: flags=73<UP,LOOPBACK,RUNNING> mtu 65536 inet 127.0.0.1 netmask 255.0.0.0 inet6 ::1 prefixlen 128 scopeid 0x10<host> loop txqueuelen 1000 (Local Loopback) RX packets 242954 bytes 49866779 (47.5 MiB) RX errors 0 dropped 0 overruns 0 frame 0 TX packets 242954 bytes 49866779 (47.5 MiB) TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0 |
| --- | --- |

рестартуем vault и проверяем статус:

| 1234567891011121314151617181920212223242526272829303132333435363738394041 | root@vault2:~# systemctl restart vaultroot@vault2:~# vault statusWARNING! VAULT_ADDR and -address unset. Defaulting to https://127.0.0.1:8200.Key Value--- -----Seal Type transitRecovery Seal Type shamirInitialized trueSealed falseTotal Recovery Shares 5Threshold 3Version 1.17.0Build Date 2024-06-10T10:11:34ZStorage Type raftCluster Name vault-cluster-011de0c1Cluster ID 69604c77-2116-9a8d-4c2e-fced06b9b578HA Enabled trueHA Cluster https://vault1.test.local:8201HA Mode standbyActive Node Address https://vault1.test.local:8200Raft Committed Index 10376Raft Applied Index 10376root@vault2:~# ifconfigenp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500 inet 192.168.1.104 netmask 255.255.255.0 broadcast 192.168.1.255 inet6 fe80::a00:27ff:fe6d:1798 prefixlen 64 scopeid 0x20<link> ether 08:00:27:6d:17:98 txqueuelen 1000 (Ethernet) RX packets 422662 bytes 62139374 (59.2 MiB) RX errors 0 dropped 0 overruns 0 frame 0 TX packets 199484 bytes 38118335 (36.3 MiB) TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0lo: flags=73<UP,LOOPBACK,RUNNING> mtu 65536 inet 127.0.0.1 netmask 255.0.0.0 inet6 ::1 prefixlen 128 scopeid 0x10<host> loop txqueuelen 1000 (Local Loopback) RX packets 245286 bytes 50343368 (48.0 MiB) RX errors 0 dropped 0 overruns 0 frame 0 TX packets 245286 bytes 50343368 (48.0 MiB) TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0 |
| --- | --- |

как видим после рестарта кластер успешно распечатан
**Sealed false**
основной нодой является https://vault1.test.local:8201
виртуальный адрес переехал на другую ноду.

как видим auto unseal успешно настроен.

напомню что все домены у нас заведены через freeipa

![](/news/sidmidru/article-db03824b64600e5b/image-119.png)

### []обновить токен для распечатывая основного vault

если токен сдох и основной волт не распечатывается то нужно его обновить, для этого смотрим основной рут токен:

root@kub-master1:~#**kubectl get configmaps -n vault vault-unseal-keys -o yaml | grep root**
"root_token": "hvs.hOrdYvRpvieSUF1yuWDIeegB"

root@kub-master1:~#**kubectl exec -ti -n vault vault-0 -- sh**

/ $**vault login**

проверяем что транзит создан

/ $**vault secrets list | grep transit**

transit/ transit transit_1622485e n/a

создаём новый токен:

|  | / $ vault token create -policy=unseal-policy -period=90000h -orphanWARNING! The following warnings were returned from Vault: * period of "90000h" exceeded the effective max_ttl of "768h"; period value is capped accordinglyKey Value--- -----token hvs.CAESIJ4oY7ZiA7rRwJIBUmIafMpEa--RPzdZMjjFzJdfN0-3Gh4KHGh2cy4wOTduMTB5OG1SenIxNk9YdTJHZmhDOFMtoken_accessor Bl3c2lPN6OzkkAuhfKPZwk5htoken_duration 768htoken_renewable truetoken_policies ["default" "unseal-policy"]identity_policies []policies ["default" "unseal-policy"] |
| --- | --- |

токен максимум создаётся на**768h**изменим это число:

/ $**vault auth tune -max-lease-ttl=90000h token/**

Success! Tuned the auth method at: token/

и создадим новый токен:

**vault token create -policy=unseal-policy -period=90000h -orphan**

|  | Key Value--- -----token hvs.CAESIJGcJoJ3xhrepURua7Rvte2u3TCGJ7t1q3XIMaZFPkjwGh4KHGh2cy5nRHZUWVZ0elowYmE3REZpbFVkS2JCVGwtoken_accessor bkfa8CPHegDJbziHIOFWDdSstoken_duration 90000htoken_renewable truetoken_policies ["default" "unseal-policy"]identity_policies []policies ["default" "unseal-policy"] |
| --- | --- |

теперь этот токен раскидаем по всем волтам:

vault1 vault2 vault3 в блок seal "transit"
**/etc/vault.d/vault.hcl**

|  | seal "transit" { address = "http://vault-unseal.test.local:80" token = "hvs.CAESIJGcJoJ3xhrepURua7Rvte2u3TCGJ7t1q3XIMaZFPkjwGh4KHGh2cy5nRHZUWVZ0elowYmE3REZpbFVkS2JCVGw" key_name = "vault-unseal-key" mount_path = "transit/" disable_renewal = "false" tls_skip_verify = "true" } |
| --- | --- |

всё можно ребутать тачки волтов ну или рестартить волты

### []Добавим интеграцию vault(виртуалки) в k8s "Vault Secrets Operator"

вот официальная репка:

https://github.com/hashicorp/vault-secrets-operator

ставим

**helm repo add hashicorp https://helm.releases.hashicorp.com**

добавляем адрес нашего vault в values чтоб установить
**cat vault-secrets-operator.yaml**

| 1234567891011121314151617181920212223242526272829303132333435 | controller: replicas: 2 affinity: podAntiAffinity: requiredDuringSchedulingIgnoredDuringExecution: - labelSelector: matchExpressions: - key: app operator: In values: - vault-secrets-operator topologyKey: "kubernetes.io/hostname" kubeRbacProxy: resources: limits: cpu: 150m memory: 150Mi requests: cpu: 50m memory: 100Mi manager: resources: limits: cpu: 150m memory: 150Mi requests: cpu: 50m memory: 100MidefaultVaultConnection: enabled: true skipTLSVerify: true address: "https://192.168.1.111:8200" |
| --- | --- |

адрес лучше конечно указать vault.test.local хоть оно и резолвит:

|  | / $ nslookup vault.test.localServer: 169.254.25.10Address: 169.254.25.10:53Name: vault.test.localAddress: 192.168.1.111 |
| --- | --- |

но могут возникать проблемы:

|  | Warning VaultClientConfigError 6s (x12 over 86s) VaultStaticSecret Failed to get Vault auth login: Put "https://vault.test.local:8200/v1/auth/kubernetes/login": dial tcp: lookup vault.test.local on 169.254.25.10:53: no such host |
| --- | --- |

поэтому я оставил ip address 192.168.1.111

ставим:

**helm upgrade --install --namespace vault vault-secrets-operator hashicorp/vault-secrets-operator --version 0.9.0 -f vault-secrets-operator.yaml**

дальше нам надо настроить интеграцию с нашим vault кластером, который на виртуалках, для этого нам понадобится сертификат поэтому выполняем команду:

root@client:~/vault-autounseal#**kubectl get cm kube-root-ca.crt -o jsonpath="{['data']['ca\.crt']}"**

получаем наш серт:

|  | root@client:~/vault-autounseal# kubectl get cm kube-root-ca.crt -o jsonpath="{['data']['ca\.crt']}"-----BEGIN CERTIFICATE-----MIIC/jCCAeagAwIBAgIBADANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwprdWJlcm5ldGVzMB4XDTI0MDYzMDE0MjIxMFoXDTM0MDYyODE0MjIxMFowFTETMBEGA1UEAxMKa3ViZXJuZXRlczCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAPsSLBw3lg/4zrpycC1xJprK7XRIZK6AyL7DIClHPhAWn+h+m72r4PfZib9wu7/VS0e0SkNhVhEhjTZx1yMkJfP4zwd6IUuFAptZfCgVHnJQnkPeadWUqj/zEIh2ubcgbRwsjG1nh8yNRbtrxd/dpXPveBCFptOi5CqdUYodhoBDic5mYwbk8AbMf71FBhhdq8+8y5TiFlXPglITHxe/0VHCXO3ANUvNJDbDoYUGU5I0Kv3AIwffxSrNlyVhgzX70b7eKaurkmGW577hXCEZapRjHlxl2wUwW+BCKZ7X8MCFbwUVVgVLP0nroKvEVUxQNimalip/RtaqiKrf3PypuUsCAwEAAaNZMFcwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB/wQFMAMBAf8wHQYDVR0OBBYEFB/OXFsCKolo5DCMnaFCxrXMBamSMBUGA1UdEQQOMAyCCmt1YmVybmV0ZXMwDQYJKoZIhvcNAQELBQADggEBAKQ3XdnYLdoS9OeETDMGj3nHXyjZsfc7TupYIpHwUH27Jh9LSzl0oxK1dWoRVizF6P1VKlKoZV7tzPPFEjqQfeE1cDXErY4acWq6v6ta3Og65TH6d9/rIqcUfr7UqZw+oMRi4UGkCP4PGoG8KmcpmGmWBLmzBD/ChB0zE9gN2vsKgGWSYUdZoO3KbrMRE/bdH1jDABXpkXvcnQ+gPaqPw1ZNOcADFWwITbKaRAwWTCspgqKd7D1IAuPRR/kIDPIWCc0rLdvPPuKTmx4Ah3j0TQvc8UW/gQX7OV2iReE0sZ1n/RRR4Fa+9mtUleaODao0MvqHVo+9IYFCTTi1fvRCySE=-----END CERTIFICATE----- |
| --- | --- |

создаём сервис аккаунт

root@client:~/vault-autounseal#**kubectl create serviceaccount vault-auth -n kube-system**

на основе этого сервис аккаунта получаем jwt token устанавливаю время на**100**лет**876000h**

root@client:~/vault-autounseal#**kubectl create token vault-auth -n kube-system --duration 876000h**

|  | eyJ************************************Jlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjo0ODg1NzAzNDUwLCJpYXQiOjE3MzIxMDM0NTAsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJ2YXVsdC1hdXRoIiwidWlkIjoiZDRhZmY3YzktY2Q4My00ODEwLWEyYzYtYWRjZmU5M2VlY2MwIn19LCJuYmYiOjE3MzIxMDM0NTAsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTp2YXVsdC1hdXRoIn0.jwJulDCsUGDnTgQcMqcV00V_JW1OxfEJZleB_qggqjSlTWjlWW9y_4ijs8-o66IeZFi_tIx2lC80G1uAkxnpenGdqC36fEiOzMW3WosYWsXhdNVENBYLEaFUWHuJPNnDJsOg9bVPyLlyVigLfAgp5RdDltaheo2Voxw4P_JWCynYwNKNsdWmZ97CnB5SagHgccsYnr_eacL_P8Ewb82HFyqkcN6dk6lhzWkD2zI4Nqg1gNpkl95La8DX9-MckkMxyu5dh9foksWT4YjqJIsmHb76agTEbTF3r0N2DCDBLUQrVUQfT-a-8hXMCp1N68BjxANkyk-sDSmOp6xa4spzjg |
| --- | --- |

далее подключаемся к нашему vault который на виртуалках

root@client:~/vault-autounseal#**ssh 192.168.1.103**

напоминаю root token

hvs.***********************

root@vault1:~#**vault login**

|  | Key Value--- -----token hvs.UuG0Q***token_accessor a**********4token_duration ∞token_renewable falsetoken_policies ["root"]identity_policies []policies ["root"] |
| --- | --- |

создаём секрет который будем подкидывать:

root@vault1:~#**vault secrets enable -path=test/secret/ kv
**
добавляем туда ключ значение:

root@vault1:~#**vault kv put test/secret/namespace-test/first-app password="db-secret-password"**

включаем аутентификацию в k8s

root@vault1:~#**vault auth enable kubernetes**

сертификат который получили ранее, кладём в файл**/root/ca.crt**

дальше присваиваем переменнойTOKEN_REVIEWER_JWTнаш jwt токен который мы создали выше

root@vault1:~#

|  | export TOKEN_REVIEWER_JWT='eyJhbGciOiJSUzI1N************************************0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjo0ODg1NzAzNDUwLCJpYXQiOjE3MzIxMDM0NTAsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJ2YXVsdC1hdXRoIiwidWlkIjoiZDRhZmY3YzktY2Q4My00ODEwLWEyYzYtYWRjZmU5M2VlY2MwIn19LCJuYmYiOjE3MzIxMDM0NTAsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTp2YXVsdC1hdXRoIn0.jwJulDCsUGDnTgQcMqcV00V_JW1OxfEJZleB_qggqjSlTWjlWW9y_4ijs8-o66IeZFi_tIx2lC80G1uAkxnpenGdqC36fEiOzMW3WosYWsXhdNVENBYLEaFUWHuJPNnDJsOg9bVPyLlyVigLfAgp5RdDltaheo2Voxw4P_JWCynYwNKNsdWmZ97CnB5SagHgccsYnr_eacL_P8Ewb82HFyqkcN6dk6lhzWkD2zI4Nqg1gNpkl95La8DX9-MckkMxyu5dh9foksWT4YjqJIsmHb76agTEbTF3r0N2DCDBLUQrVUQfT-a-8hXMCp1N68BjxANkyk-sDSmOp6xa4spzjg' |
| --- | --- |

теперь настраиваем auth к кластеру

root@vault1:~#

|  | vault write auth/kubernetes/config \ kubernetes_host="https://192.168.1.112:6443" \ kubernetes_ca_cert=@/root/ca.crt \ token_reviewer_jwt="$TOKEN_REVIEWER_JWT" \ disable_iss_validation="true" |
| --- | --- |

создаём policy "test-policy" на чтениеТОЛЬКОнашего секрета

|  | vault policy write test-policy - <<EOFpath "test/secret/namespace-test/first-app" { capabilities = ["read"]}EOF |
| --- | --- |

создаём роль "**test-role**"

Роль связывает учетную запись службы Kubernetes(serviceaccaunt), которую назовём**test-serviceaccaunt**(но лучше использовать уже существующий сервис аккаунт нашего приложения ) в пространстве имен**test**с политикой Vault,**test-policy**Токены, возвращенные после аутентификации, действительны в течение 10 минут

|  | vault write auth/kubernetes/role/test-role \ bound_service_account_names=test-serviceaccount \ bound_service_account_namespaces=test \ policies=test-policy \ ttl=10m |
| --- | --- |

- **`bound_service_account_names`**: Имя сервисного аккаунта, которому разрешено аутентифицироваться.
- **`bound_service_account_namespaces`**: Пространство имён, в котором находится сервисный аккаунт.

создаём namespace test

root@client:~#**kubectl create ns test**

создаём serviceaccaunt

**kubectl create serviceaccount test-serviceaccount -n test**

создаём объект**VaultConnection**который будет использоваться для подключения к vault во всех неймспейсах

|  | apiVersion: secrets.hashicorp.com/v1beta1kind: VaultConnectionmetadata: name: vault-connection namespace: kube-systemspec: address: "https://192.168.1.111:8200" skipTLSVerify: true |
| --- | --- |

как видим он использует namespace: kube-system
192.168.1.111 - это виртуальный ip vault кластера

теперь создаём объект**VaultAuth**

|  | apiVersion: secrets.hashicorp.com/v1beta1kind: VaultAuthmetadata: name: vault-auth-test namespace: testspec: vaultConnectionRef: kube-system/vault-connection method: kubernetes mount: kubernetes kubernetes: role: test-role serviceAccount: test-serviceaccount |
| --- | --- |

как видим для подключения к VaultConnection используется запись: kube-system/vault-connection
так же тут указываем роль созданную в vault test-role и сервис аккаунт test-serviceoccount - который так же должен совпадать с тем что мы указали в vault и с тем что у нас уже создан в k8s

теперь создаём объект**VaultStaticSecret**

|  | apiVersion: secrets.hashicorp.com/v1beta1kind: VaultStaticSecretmetadata: name: app-db-creds-test namespace: testspec: vaultAuthRef: vault-auth-test mount: test/secret type: kv-v1 path: namespace-test/first-app refreshAfter: 10s destination: create: true overwrite: true name: test-secret-k8s |
| --- | --- |

тут мы указываем как будет называться secret и как часто его обновлять.

теперь создаём несколько объектов**ClusterRole**и**ClusterRoleBinding**

| 123456789101112131415161718192021222324252627282930313233 | apiVersion: rbac.authorization.k8s.io/v1kind: ClusterRolemetadata: name: vault-token-reviewerrules: - apiGroups: ["authentication.k8s.io"] resources: ["tokenreviews"] verbs: ["create"] - apiGroups: [""] resources: ["configmaps"] verbs: ["get"] - apiGroups: [""] resources: ["secrets"] verbs: ["get"] - apiGroups: [""] resources: ["serviceaccounts"] verbs: ["get"]--- apiVersion: rbac.authorization.k8s.io/v1kind: ClusterRoleBindingmetadata: name: vault-token-reviewer-bindingroleRef: apiGroup: rbac.authorization.k8s.io kind: ClusterRole name: vault-token-reviewersubjects: - kind: ServiceAccount name: vault-auth namespace: kube-system |
| --- | --- |

как мы видим ClusterRoleBinding смотрит на ServiceAccount vault-auth расположенный kube-system мы его создавали вручную и jwt token создавали на его основе.

[root@ansible ansible]#**scp -r /etc/ansible/kubespray-official/vault-autounseal/ root@192.168.1.121:~/**

теперь запускаем по очереди:

root@client:~/vault-autounseal#**kubectl apply -f vault-secrets-operator-rbac.yaml
**root@client:~/vault-autounseal#**kubectl apply -f vault-secrets-operator-VaultConnection.yaml
**root@client:~/vault-autounseal#**kubectl apply -f vault-secrets-operator-VaultAuth.yaml
**root@client:~/vault-autounseal#****kubectl apply -f vault-secrets-operator-VaultSecret.yaml****

проверяем:

| 12345678910111213141516171819202122232425 | root@client:~/vault-autounseal# kubectl describe vaultconnections.secrets.hashicorp.com -n kube-system vault-connectionName: vault-connectionNamespace: kube-systemLabels: <none>Annotations: <none>API Version: secrets.hashicorp.com/v1beta1Kind: VaultConnectionMetadata: Creation Timestamp: 2024-11-20T11:53:23Z Finalizers: vaultconnection.secrets.hashicorp.com/finalizer Generation: 1 Resource Version: 2383949 UID: 0a8e370d-2f46-4069-8fb6-9d760f782643Spec: Address: https://192.168.1.111:8200 Skip TLS Verify: trueStatus: Valid: trueEvents: Type Reason Age From Message ---- ------ ---- ---- ------- Warning VaultClientError 2m15s VaultConnection Failed to check Vault seal status: Get "https://192.168.1.111:8200/v1/sys/seal-status": dial tcp 192.168.1.111:8200: connect: no route to host Normal Accepted 2m13s VaultConnection VaultConnection accepted |
| --- | --- |

| 123456789101112131415161718192021222324252627282930 | root@client:~/vault-autounseal# kubectl describe vaultauths.secrets.hashicorp.com -n test vault-auth-testName: vault-auth-testNamespace: testLabels: <none>Annotations: <none>API Version: secrets.hashicorp.com/v1beta1Kind: VaultAuthMetadata: Creation Timestamp: 2024-11-20T11:53:23Z Finalizers: vaultauth.secrets.hashicorp.com/finalizer Generation: 1 Resource Version: 2383940 UID: db80cd4b-5cbc-4e7f-8bac-fc0099b205feSpec: Kubernetes: Role: test-role Service Account: test-serviceaccount Token Expiration Seconds: 600 Method: kubernetes Mount: kubernetes Vault Connection Ref: kube-system/vault-connectionStatus: Spec Hash: e70e9199edd25d059ad9898fa1142805fdd5df041e490bf1ca1a735c65ee7fd5 Valid: trueEvents: Type Reason Age From Message ---- ------ ---- ---- ------- Normal Accepted 2m56s VaultAuth Successfully handled VaultAuth resource request |
| --- | --- |

| 123456789101112131415161718192021222324252627282930313233343536 | root@client:~/vault-autounseal# kubectl describe vaultstaticsecrets.secrets.hashicorp.com -n test app-db-creds-testName: app-db-creds-testNamespace: testLabels: <none>Annotations: <none>API Version: secrets.hashicorp.com/v1beta1Kind: VaultStaticSecretMetadata: Creation Timestamp: 2024-11-20T11:53:23Z Finalizers: vaultstaticsecret.secrets.hashicorp.com/finalizer Generation: 2 Resource Version: 2383969 UID: 67395e52-f0b0-464f-9622-93f3c68c3f16Spec: Destination: Create: true Name: test-secret-k8s Overwrite: false Transformation: Hmac Secret Data: true Mount: test/secret Path: namespace-test/first-app Refresh After: 10s Type: kv-v1 Vault Auth Ref: vault-auth-testStatus: Last Generation: 2 Secret MAC: vz7xmB0zNnANWvtdar3CZTYGB6Y+7SwkZJRqCfdAyxw=Events: Type Reason Age From Message ---- ------ ---- ---- ------- Warning VaultClientConfigError 4m12s VaultStaticSecret Failed to get Vault auth login: Put "https://192.168.1.111:8200/v1/auth/kubernetes/login": dial tcp 192.168.1.111:8200: connect: no route to host Normal SecretSynced 4m7s VaultStaticSecret Secret synced Normal SecretRotated 4m7s VaultStaticSecret Secret synced |
| --- | --- |

проверяем созданный секрет:

| 123456789101112131415161718192021222324 | root@client:~/vault-autounseal# kubectl get secret -n test test-secret-k8s -o yamlapiVersion: v1data: _raw: eyJwYXNzd29yZCI6ImRiLXNlY3JldC1wYXNzd29yZCJ9 password: ZGItc2VjcmV0LXBhc3N3b3Jkkind: Secretmetadata: creationTimestamp: "2024-11-20T11:53:39Z" labels: app.kubernetes.io/component: secret-sync app.kubernetes.io/managed-by: hashicorp-vso app.kubernetes.io/name: vault-secrets-operator secrets.hashicorp.com/vso-ownerRefUID: 67395e52-f0b0-464f-9622-93f3c68c3f16 name: test-secret-k8s namespace: test ownerReferences: - apiVersion: secrets.hashicorp.com/v1beta1 kind: VaultStaticSecret name: app-db-creds-test uid: 67395e52-f0b0-464f-9622-93f3c68c3f16 resourceVersion: "2383963" uid: eecfaae7-0275-4019-9898-a6f99aab2d35type: Opaque |
| --- | --- |

а теперь добавим в vault секрет пару новый значений

![](/news/sidmidru/article-db03824b64600e5b/image-120.png)

| 1234567891011121314151617181920212223242526 | root@client:~/vault-autounseal# kubectl get secret -n test test-secret-k8s -o yamlapiVersion: v1data: _raw: eyIxMjMiOiIxMjMiLCJwYXNzd29yZCI6ImRiLXNlY3JldC1wYXNzd29yZCIsInRlc3QiOiJ0ZXN0In0= "123": MTIz password: ZGItc2VjcmV0LXBhc3N3b3Jk test: dGVzdA==kind: Secretmetadata: creationTimestamp: "2024-11-20T11:53:39Z" labels: app.kubernetes.io/component: secret-sync app.kubernetes.io/managed-by: hashicorp-vso app.kubernetes.io/name: vault-secrets-operator secrets.hashicorp.com/vso-ownerRefUID: 67395e52-f0b0-464f-9622-93f3c68c3f16 name: test-secret-k8s namespace: test ownerReferences: - apiVersion: secrets.hashicorp.com/v1beta1 kind: VaultStaticSecret name: app-db-creds-test uid: 67395e52-f0b0-464f-9622-93f3c68c3f16 resourceVersion: "2385699" uid: eecfaae7-0275-4019-9898-a6f99aab2d35type: Opaque |
| --- | --- |

как видим дополнительные key - value 123 и test добавились в секрет.

### []Пример с autoreloader после изменения секрета в vault

сделаем следующую структуру у секретов.
/dev/service/app1(app2/app3)
/prod/service/app1(app2/app3)

![](/news/sidmidru/article-db03824b64600e5b/image-121.png)

![](/news/sidmidru/article-db03824b64600e5b/image-122.png)

![](/news/sidmidru/article-db03824b64600e5b/image-123.png)

не забываем выставлять версию секретов я использую 1ую версию

![](/news/sidmidru/article-db03824b64600e5b/image-124.png)

![](/news/sidmidru/article-db03824b64600e5b/image-125.png)

![](/news/sidmidru/article-db03824b64600e5b/image-126.png)

в итоге получаем:

![](/news/sidmidru/article-db03824b64600e5b/image-127.png)

теперь создаём полиси.
сделаем одну полиси которая сможет читать все секреты в dev/service и вторую которая будет читать всё в prod/service

вот эти policy**read-dev-service**и**read-prod-service**

|  | path "dev/service/*" { capabilities = ["read", "list"]} |
| --- | --- |

|  | path "prod/service/*" { capabilities = ["read", "list"]} |
| --- | --- |

![](/news/sidmidru/article-db03824b64600e5b/image-128.png)

![](/news/sidmidru/article-db03824b64600e5b/image-129.png)

![](/news/sidmidru/article-db03824b64600e5b/image-130.png)

теперь создаём roles - нужно знать как будет называться serviceaccaunt у нашего приложения. если не использовать

fullnameOverride то имя serviceaccount будет состоять из имени релиза и имени чарта, в моём случае это:

root@client:~/vault-autounseal# helm upgrade --install --namespace dev**first-app**./common-chart/ --values ./**common-chart**/values.yaml

поэтому мой сервис аккаунт будет выглядеть так:

**first-app-common-chart**

В одну роль можно запихнуть несколько аккаунтов, но мы создадим каждому приложению свою роль

|  | vault write auth/kubernetes/role/dev-read-role-app1 \ bound_service_account_names=first-app-common-chart\ bound_service_account_namespaces=dev \ policies=read-dev-service \ ttl=10m |
| --- | --- |

сразу сделаем для второй апки в dev и апки в prod схема запуска будет такой же

|  | vault write auth/kubernetes/role/dev-read-role-app2 \ bound_service_account_names=second-app-common-chart\ bound_service_account_namespaces=dev \ policies=read-dev-service \ ttl=10m |
| --- | --- |

|  | vault write auth/kubernetes/role/prod-read-role-app1 \ bound_service_account_names=first-app-common-chart\ bound_service_account_namespaces=prod \ policies=read-prod-service \ ttl=10m |
| --- | --- |

проверить можно так:

**vault list auth/kubernetes/role**

|  | vault list auth/kubernetes/role Keys dev-read-role-app1 dev-read-role-app2 prod-read-role-app1 test-role |
| --- | --- |

детально смотрим пару ролей
**vaultreadauth/kubernetes/role/ИМЯ_РОЛИ**

| 123456789101112131415161718192021222324252627282930313233343536 | vault read auth/kubernetes/role/dev-read-role-app1Key Value alias_name_source serviceaccount_uid bound_service_account_names ["first-app-common-chart"]bound_service_account_namespace_selector bound_service_account_namespaces ["dev"] policies ["read-dev-service"] token_bound_cidrs [] token_explicit_max_ttl 0 token_max_ttl 0 token_no_default_policy false token_num_uses 0 token_period 0 token_policies ["read-dev-service"] token_ttl 600 token_type default ttl 600 vault read auth/kubernetes/role/prod-read-role-app1Key Value alias_name_source serviceaccount_uid bound_service_account_names ["first-app-common-chart"]bound_service_account_namespace_selector bound_service_account_namespaces ["prod"] policies ["read-prod-service"] token_bound_cidrs [] token_explicit_max_ttl 0 token_max_ttl 0 token_no_default_policy false token_num_uses 0 token_period 0 token_policies ["read-prod-service"] token_ttl 600 token_type default ttl 600 |
| --- | --- |

а вот values, я тут покажу только отличия в values для секретов:

/etc/ansible/kubespray-official/vault-autounseal/common-chart/values-dev-app1.yaml

|  | vault_secret: mount: "dev/service" path: "app1" type: "kv-v1" refreshAfter: "10s" role: "dev-read-role-app1" vaultConnectionRef: "kube-system/vault-connection" secret_name: "first-app-secret" |
| --- | --- |

/etc/ansible/kubespray-official/vault-autounseal/common-chart/values-dev-app2.yaml

|  | vault_secret: mount: "dev/service" path: "app2" type: "kv-v1" refreshAfter: "10s" role: "dev-read-role-app2" vaultConnectionRef: "kube-system/vault-connection" secret_name: "second-app-secret" |
| --- | --- |

/etc/ansible/kubespray-official/vault-autounseal/common-chart/values-prod-app1.yaml

|  | vault_secret: mount: "prod/service" path: "app1" type: "kv-v1" refreshAfter: "10s" role: "prod-read-role-app1" vaultConnectionRef: "kube-system/vault-connection" secret_name: "first-app-secret" |
| --- | --- |

у нас будет 2 namespace**dev**и**prod**

root@client:~/vault-autounseal#**kubectl create ns dev**
root@client:~/vault-autounseal#**kubectl create ns prod**

ставим теперь наши апки:

root@client:~/vault-autounseal#**helm upgrade --install --namespace dev first-app ./common-chart/ --values ./common-chart/values-dev-app1.yaml**

root@client:~/vault-autounseal#**helm upgrade --install --namespace dev second-app ./common-chart/ --values ./common-chart/values-dev-app2.yaml**

root@client:~/vault-autounseal#**helm upgrade --install --namespace prod first-app ./common-chart/ --values ./common-chart/values-prod-app1.yaml**

что нужно добавить в helm template

/etc/ansible/kubespray-official/vault-autounseal/common-chart/templates/vault-secrets-operator-VaultAuth.yaml

|  | apiVersion: secrets.hashicorp.com/v1beta1kind: VaultAuthmetadata: name: "vaultauth-{{ .Values.vault_secret.secret_name }}"spec: vaultConnectionRef: {{ .Values.vault_secret.vaultConnectionRef }} method: kubernetes mount: kubernetes kubernetes: role: {{ .Values.vault_secret.role }} serviceAccount: {{ include "common-chart.serviceAccountName" . }} |
| --- | --- |

/etc/ansible/kubespray-official/vault-autounseal/common-chart/templates/vault-secrets-operator-VaultSecret.yaml

| 12345678910111213141516171819 | apiVersion: secrets.hashicorp.com/v1beta1kind: VaultStaticSecretmetadata: name: "vaultsecret-{{ .Values.vault_secret.secret_name }}"spec: vaultAuthRef: "vaultauth-{{ .Values.vault_secret.secret_name }}" mount: {{ .Values.vault_secret.mount }} type: {{ .Values.vault_secret.type }} path: {{ .Values.vault_secret.path }} refreshAfter: {{ .Values.vault_secret.refreshAfter }} destination: create: true overwrite: true name: {{ .Values.vault_secret.secret_name }} rolloutRestartTargets: - kind: Deployment name: {{ include "common-chart.fullname" . }} |
| --- | --- |

rolloutRestartTargets - будет рестартовать деплоймент при изменении секрета

и нужно отредактировать deployment

/etc/ansible/kubespray-official/vault-autounseal/common-chart/templates/deployment.yaml

добавив:

|  | envFrom: - secretRef: name: {{ .Values.vault_secret.secret_name }} |
| --- | --- |

всё теперь подкидывая новые или удаляя старые секреты - будет рестартоваться и деплоймент

### []Аутентификация, авторизация в k8s (SSOв kubernetes)

В Kubernetes есть два типа пользователей:

- *Service Accounts*— аккаунты, управляемые KubernetesAPI;
- *Users*— «нормальные» пользователи, управляемые внешними, независимыми сервисами.

Основное отличие этих типов в том, что для Service Accounts существуют специальные объекты в KubernetesAPI(они так и называются —`ServiceAccounts`), которые привязаны к пространству имён и набору авторизационных данных, хранящихся в кластере в объектах типа Secrets. Такие пользователи (Service Accounts) предназначены в основном для управления правами доступа к KubernetesAPIпроцессов, работающих в кластере Kubernetes.

Обычные же Users не имеют записей в KubernetesAPI: управление ими должно осуществляться внешними механизмами. Они предназначены для людей или процессов, живущих вне кластера.

Каждый запрос кAPIпривязан либо к Service Account, либо к User, либо считается анонимным.

Аутентификационные данные пользователя включают в себя:

- *Username*— имя пользователя (зависит от регистра!);
- *UID*— машинно-читаемая строка идентификации пользователя, которая «более консистентна и уникальна, чем имя пользователя»;
- *Groups*— список групп, к которым принадлежит пользователь;
- *Extra*— дополнительные поля, которые могут быть использованы механизмом авторизации.

Kubernetes может использовать большое количество механизмов аутентификации: сертификатыX509, Bearer-токены, аутентифицирующий прокси,HTTPBasic Auth. При помощи этих механизмов можно реализовать большое количество схем авторизации: от статичного файла с паролями до OpenID OAuth2.

Более того, допускается использование нескольких схем авторизации одновременно. По умолчанию в кластере используются:

- service account tokens — для Service Accounts;
- X509— для Users.

### []Loft

вот официальный блог с инструкцией

https://www.loft.sh/blog/kubernetes-and-ldap-enterprise-authentication-for-kubernetes

подготавливаем values.yaml который расположен у меня тут:

cd /etc/ansible/kubespray-official/sso-loft/

| 12345678910111213141516171819202122232425262728293031 | admin: create: true username: admin password: "Secret123"service: type: ClusterIPingress: enabled: true name: loft-ingress host: loft.test.local ingressClass: nginx path: / tls: enabled: false # Установите true, если используете TLS secret: loft-tlspersistence: enabled: true size: 10Gi storageClassName: nfs-client # Замените на ваш StorageClass accessModes: ["ReadWriteOnce"]serviceAccount: name: loft create: true clusterRole: cluster-admin |
| --- | --- |

тут мы указываем логин пароль с которым будем заходить
admin
Secret123

настраиваем persistant volume и ingress
loft.test.local

ставим

root@client:~#**kubectl create ns loft**
root@client:~#**helm repo add loft https://charts.loft.sh**
root@client:~#**helm repo update**
root@client:~#**helm search repo loft**
последний командой смотрим последнюю версию,

| 12345678910111213141516171819202122232425262728 | root@client:~# helm search repo loftNAME CHART VERSION APP VERSION DESCRIPTIONloft/loft 4.1.0 Secure Cluster Sharing, Self-Service Namespace ...loft/loft-agent 3.2.4 Loft Cluster Agentloft/loft-direct-cluster-endpoint 1.14.0 Direct Secure Cluster Loft Access Pointloft/loft-grafana-dashboards 0.1.2 Grafana dashboards for loftloft/central-hostpath-mapper 0.2.6 0.0.1 A centralized version of the hostpath mapperloft/cert-issuer 0.0.4 ClusterIssuer for cert-managerloft/component-chart 0.9.1 A general purpose helm chart for deploying comp...loft/devpod-pro 4.1.0 DevPod.Pro - Developer Environments as Codeloft/devspace-cloud 0.3.3 The DevSpace Cloud control plane.loft/isolation-templates 0.1.0 Smart defaults for isolations including Network...loft/jspolicy 0.2.2 JavaScript Policies for Kubernetesloft/kiosk 0.2.11 Multi-Tenancy Extension For Kubernetesloft/vcluster 0.21.1 0.21.1 vcluster - Virtual Kubernetes Clustersloft/vcluster-control-plane 3.4.9 vCluster.Pro - Virtual Kubernetes Clustersloft/vcluster-eks 0.19.8 0.19.8 vcluster - Virtual Kubernetes Clusters (eks)loft/vcluster-hpm 0.1.2 1.20.0 A Helm chart to resolve the correct virtual pod...loft/vcluster-k0s 0.19.8 0.19.8 vcluster - Virtual Kubernetes Clusters (k0s)loft/vcluster-k8s 0.19.8 0.19.8 vcluster - Virtual Kubernetes Clusters (k8s)loft/vcluster-platform 4.1.0 vCluster Platform - Virtual Kubernetes Clustersloft/vcluster-pro 0.2.0 vcluster-pro - Virtual Kubernetes Clustersloft/vcluster-pro-eks 0.2.0 vcluster-pro - Virtual Kubernetes Clusters (eks)loft/vcluster-pro-k0s 0.2.0 vcluster-pro - Virtual Kubernetes Clusters (k0s)loft/vcluster-pro-k8s 0.2.0 vcluster-pro - Virtual Kubernetes Clusters (k8s)loft/vcluster-pro-rancher-plugin 0.0.2 0.0.2 vCluster.Pro plugin for Rancherloft/virtualcluster 0.0.28 A virtual kubernetes cluster |
| --- | --- |

всё теперь может устанавливать:
root@client:~/sso-loft#**helm install loft loft/loft --namespace loft -f values.yaml --version 4.1.0**

ждём окончания установки и заходим:

http://loft.test.local/login

![](/news/sidmidru/article-db03824b64600e5b/image-131.png)

далее нужно заполнить данные о компании

![](/news/sidmidru/article-db03824b64600e5b/image-132.png)

![](/news/sidmidru/article-db03824b64600e5b/image-133.png)

эта панель позволяет создавать виртуальные кластера внутри нашего кластера но этот функционал нам не нужен, так как сама панель она триальная:

![](/news/sidmidru/article-db03824b64600e5b/image-134.png)

создаём пользователя:

![](/news/sidmidru/article-db03824b64600e5b/image-135.png)

![](/news/sidmidru/article-db03824b64600e5b/image-136.png)

![](/news/sidmidru/article-db03824b64600e5b/image-137.png)

![](/news/sidmidru/article-db03824b64600e5b/image-138.png)

### []Dex - dexK8sAuthenticator

**Dex**— сервер аутентификации, реализующий протоколы OpenID Connect (OIDC) и OAuth 2.0. Он выступает в качестве посредника между различными внешними провайдерами аутентификации и приложениями, которым требуется аутентификация черезOIDC.

**Функциональность Dex:**

- **Аутентификация пользователей:**Dex позволяет интегрировать различные внешние источники аутентификации, такие какLDAP,SAML, GitHub, Google и другие.
- **ПровайдерOIDC:**Предоставляет стандартный интерфейсOIDCдля приложений, требующих аутентификации.
- **(SSO):**Позволяет реализовать единую точку входа для аутентификации пользователей в различных приложениях и сервисах.

**В контексте Kubernetes:**

- **Аутентификация кластера:**Dex может использоваться как OIDC-провайдер для Kubernetes, позволяя пользователям аутентифицироваться в кластере через внешний источник.
- **Интеграция с kubectl:**Пользователи могут получать токены доступа для взаимодействия с кластером через`kubectl`.

**dex-auth**— это сторонний проект или инструмент, который позволяет настроить интеграцию Dex с Kubernetes для более удобного использования, например, для входа в Kubernetes с помощью Dex и управленияOIDCтокенами. Он обычно используется для автоматизации и упрощения аутентификации через Dex.

**Основные возможности`dexK8sAuthenticator`:**

- **Упрощение процесса входа**:

- Пользователь может войти в Kubernetes через Dex, не выполняя сложных операций с токенамиOIDCвручную.
- Предоставляет интерфейс для ввода логина/пароля или использования внешнего браузера для аутентификации.

- **Автоматическое управление токенами**:

- После успешного входа автоматически получаетOIDCтокен (ID-токен, Access-токен).
- Токен добавляется в kubeconfig, чтобы пользователь мог работать с Kubernetes, используя стандартные инструменты (`kubectl`,`helm`и т.д.).

- **Интеграция с Dex**:

- Работает как клиент для Dex.
- ИспользуетOIDCпротокол для получения токенов.

**Создаем корневой сертификат**

root@client:~#**mkdir certs**
root@client:~#**cd certs/**

root@client:~/certs#**openssl genrsa -out ca.key 4096**

root@client:~/certs#**openssl req -x509 -sha256 -new -key ca.key -days 10000 -out ca.crt**

root@client:~/certs#**openssl genrsa -out dex.key 4096**

**cat dex_openssl.cnf**

| 123456789101112131415161718192021 | [ req ]default_bits = 4096prompt = nodefault_md = sha256distinguished_name = dnreq_extensions = req_ext[ dn ]C = RUST = YourStateL = YourCityO = YourOrganizationCN = dex.test.local # Основное доменное имя[ req_ext ]subjectAltName = @alt_names[ alt_names ]DNS.1 = dex.test.localDNS.2 = dex-auth.test.local |
| --- | --- |

root@client:~/certs#**openssl req -new -key dex.key -out dex.csr -config dex_openssl.cnf**

**cat ca_openssl.cnf**

|  | [ v3_ca ]subjectAltName = @alt_names[ alt_names ]DNS.1 = dex.test.localDNS.2 = dex-auth.test.local |
| --- | --- |

root@client:~/certs#**openssl x509 -req -in dex.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out dex.crt -days 10000 -sha256 -extfile ca_openssl.cnf -extensions v3_ca**

проверяем:

|  | root@client:~/certs# openssl x509 -in dex.crt -text -noout | grep -i test Subject: C = RU, ST = YourState, L = YourCity, O = YourOrganization, CN = dex.test.local DNS:dex.test.local, DNS:dex-auth.test.local |
| --- | --- |

как видим оба домена есть в сертификатах

создаём неймспейсы:

root@client:~/certs#**kubectl create ns dex**

создаём секреты с этими сертификатами:

root@client:~/certs#**kubectl create secret tls dex-tls --key dex.key --cert dex.crt --namespace dex**

и секрет для ca.crt

root@client:~/certs#**kubectl create secret generic dex-ca-cert --from-file=ca.crt=./ca.crt -n dex**

теперь создадим во freeipa группы для доступа к k8s кластеру а так же системного пользователя:

![](/news/sidmidru/article-db03824b64600e5b/image-139.png)

[root@freeipa-1 ~]# cd /etc/ipa
[root@freeipa-1 ipa]# bash freeipa-sam.sh

| 123456789101112131415161718 | ### FreeIPA - System Account Manager ###1.) ldapserver=2.) domain= (ldapdomain=)3.) binduser=4.) bindpass=UNSET!5.) ssl=trueActions (conditions not yet met): add | rm | ls | info | passwd | save--- Results ------ End Results ---> 1 |
| --- | --- |

вводим домен ldap сервера freeipa-1.test.local

3 вводим admin
4 вводим пароль Secret123
5 выключаем ssl
теперь создадим пользователя k8s-access с паролем Secret123

|  | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=SET!5.) ssl=falseActions (ready): add | rm | ls | info | passwd | save> adduid of new user=k8s-accesspassword of new user (blank to generate a password)=password expiration date YYYYMMDD (blank for 20380119)= |
| --- | --- |

теперь подготовим values

вот**dex.yaml**

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293 | replicaCount: 1https: enabled: trueconfigSecret: create: true name: "dex-config"config: issuer: https://dex.test.local storage: type: kubernetes config: inCluster: true web: https: 0.0.0.0:5556 tlsCert: /etc/dex/tls/tls.crt tlsKey: /etc/dex/tls/tls.key connectors: - type: ldap id: freeipa name: FreeIPA config: host: freeipa-1.test.local:389 insecureNoSSL: true startTLS: false bindDN: uid=k8s-access,cn=sysaccounts,cn=etc,dc=test,dc=local bindPW: Secret123 usernamePrompt: "Username" userSearch: baseDN: cn=users,cn=accounts,dc=test,dc=local filter: "(objectClass=person)" username: uid idAttr: uid emailAttr: mail nameAttr: cn groupSearch: baseDN: cn=groups,cn=accounts,dc=test,dc=local filter: "(objectClass=groupOfNames)" userMatchers: - userAttr: uid groupAttr: cn staticClients: - id: kubernetes redirectURIs: - 'https://dex-auth.test.local/callback' name: 'Kubernetes' secret: 'kubernetes-secret' oauth2: skipApprovalScreen: trueingress: enabled: true className: "nginx" annotations: cert-manager.io/cluster-issuer: "letsencrypt" hosts: - host: dex.test.local paths: - path: / pathType: Prefix tls: - secretName: dex-tls hosts: - dex.test.localrbac: create: true createClusterScoped: truevolumes: - name: dex-tls secret: secretName: dex-tlsvolumeMounts: - name: dex-tls mountPath: /etc/dex/tls readOnly: trueresources: requests: memory: "128Mi" cpu: "100m" limits: memory: "256Mi" cpu: "500m" |
| --- | --- |

и**dex-auth.yaml**

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374757677787980 | replicaCount: 1image: repository: mintel/dex-k8s-authenticator tag: latest pullPolicy: IfNotPresentdex: url: https://dex.test.local # clientID: kubernetes # clientSecret: kubernetes-secret redirectURI: https://dex-auth.test.local/callbackkubernetes: apiServerURL: https://192.168.1.112 caCert: "" # Если используется собственный CA, добавьте его сюдаrbac: enabled: true roles: admin: groups: - k8s-devops edit: groups: - k8s-users-owners view: groups: - k8s-users-roingress: enabled: true annotations: kubernetes.io/ingress.class: "nginx" path: / hosts: - dex-auth.test.local tls: - secretName: dex-tls hosts: - dex-auth.test.localconfig: logLevel: debugresources: requests: memory: 128Mi cpu: 100m limits: memory: 256Mi cpu: 500mdexK8sAuthenticator: port: 5555 debug: true web_path_prefix: / tlsCert: /etc/dex/tls/tls.crt tlsKey: /etc/dex/tls/tls.key clusters: - name: cluster-local short_description: "My Cluster" description: "Example Cluster Long Description…" client_secret: kubernetes-secret issuer: https://dex.test.local k8s_master_uri: https://192.168.1.112 client_id: kubernetes redirect_uri: https://dex-auth.test.local/callback # k8s_ca_uri: https://url-to-your-ca.crtcaCerts: enabled: true secrets: - name: dex-ca-cert filename: ca.crt value |
| --- | --- |

есть особенность для caCerts

в value добавляем наш**ca.crt**в base64

ставим:

root@client:~/certs#**helm repo add dex https://charts.dexidp.io**
root@client:~/certs#**helm repo update**

root@client:~/autentification-dex-dex-auth#**helm upgrade --install dex dex/dex -f dex.yaml --namespace dex --version 0.19.1**

root@client:~/autentification-dex-dex-auth#**helm upgrade --install dex-auth wiremind/dex-k8s-authenticator -n dex -f dex-auth.yaml**

есть ещё**проблема с резолвом**адресов у dex поэтому надо ещё поправить**деплойменты**dex и dex-auth а именно**заменить dnspolicy: ClusterFirst**

на

|  | dnsPolicy: None dnsConfig: nameservers: - 10.233.0.3 searches: - cluster.local - svc.cluster.local - test.local |
| --- | --- |

|  | root@client:~/autentification-dex-dex-auth# kubectl get svc -n kube-system | grep corednscoredns ClusterIP 10.233.0.3 <none> 53/UDP,53/TCP,9153/TCP 167d |
| --- | --- |

root@client:~/autentification-dex-dex-auth#**kubectl edit deployments.apps -n dex dex**
root@client:~/autentification-dex-dex-auth#**kubectl edit deployments.apps -n dex dex-auth-dex-k8s-authenticator**

результат выглядит вот так:

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144 | root@client:~/autentification-dex-dex-auth# kubectl get deployments.apps -n dex dex -o yamlapiVersion: apps/v1kind: Deploymentmetadata: annotations: deployment.kubernetes.io/revision: "2" meta.helm.sh/release-name: dex meta.helm.sh/release-namespace: dex creationTimestamp: "2024-12-15T10:28:37Z" generation: 2 labels: app.kubernetes.io/instance: dex app.kubernetes.io/managed-by: Helm app.kubernetes.io/name: dex app.kubernetes.io/version: 2.41.1 helm.sh/chart: dex-0.19.1 name: dex namespace: dex resourceVersion: "2908992" uid: d23d3213-a2ba-4e88-a8a7-f117c28a03e0spec: progressDeadlineSeconds: 600 replicas: 1 revisionHistoryLimit: 10 selector: matchLabels: app.kubernetes.io/instance: dex app.kubernetes.io/name: dex strategy: rollingUpdate: maxSurge: 25% maxUnavailable: 25% type: RollingUpdate template: metadata: annotations: checksum/config: 4f0ce13e695a73a4cce73dd6ce01a6b1852776f6eee0b6eb2e5325b7323112cc creationTimestamp: null labels: app.kubernetes.io/instance: dex app.kubernetes.io/name: dex spec: containers: - args: - dex - serve - --web-http-addr - 0.0.0.0:5556 - --web-https-addr - 0.0.0.0:5554 - --telemetry-addr - 0.0.0.0:5558 - /etc/dex/config.yaml image: ghcr.io/dexidp/dex:v2.41.1 imagePullPolicy: IfNotPresent livenessProbe: failureThreshold: 3 httpGet: path: /healthz/live port: telemetry scheme: HTTP periodSeconds: 10 successThreshold: 1 timeoutSeconds: 1 name: dex ports: - containerPort: 5556 name: http protocol: TCP - containerPort: 5554 name: https protocol: TCP - containerPort: 5558 name: telemetry protocol: TCP readinessProbe: failureThreshold: 3 httpGet: path: /healthz/ready port: telemetry scheme: HTTP periodSeconds: 10 successThreshold: 1 timeoutSeconds: 1 resources: limits: cpu: 500m memory: 256Mi requests: cpu: 100m memory: 128Mi securityContext: {} terminationMessagePath: /dev/termination-log terminationMessagePolicy: File volumeMounts: - mountPath: /etc/dex name: config readOnly: true - mountPath: /etc/dex/tls name: dex-tls readOnly: true dnsConfig: nameservers: - 10.233.0.3 searches: - cluster.local - svc.cluster.local - test.local dnsPolicy: None restartPolicy: Always schedulerName: default-scheduler securityContext: {} serviceAccount: dex serviceAccountName: dex terminationGracePeriodSeconds: 30 volumes: - name: config secret: defaultMode: 420 secretName: dex-config - name: dex-tls secret: defaultMode: 420 secretName: dex-tlsstatus: availableReplicas: 1 conditions: - lastTransitionTime: "2024-12-15T10:28:40Z" lastUpdateTime: "2024-12-15T10:28:40Z" message: Deployment has minimum availability. reason: MinimumReplicasAvailable status: "True" type: Available - lastTransitionTime: "2024-12-15T10:28:37Z" lastUpdateTime: "2024-12-15T10:51:02Z" message: ReplicaSet "dex-548c7df66c" has successfully progressed. reason: NewReplicaSetAvailable status: "True" type: Progressing observedGeneration: 2 readyReplicas: 1 replicas: 1 updatedReplicas: 1 |
| --- | --- |

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132 | root@client:~/autentification-dex-dex-auth# kubectl get deployments.apps -n dex dex-auth-dex-k8s-authenticator -o yamlapiVersion: apps/v1kind: Deploymentmetadata: annotations: deployment.kubernetes.io/revision: "5" meta.helm.sh/release-name: dex-auth meta.helm.sh/release-namespace: dex creationTimestamp: "2024-12-15T10:29:04Z" generation: 5 labels: app: dex-k8s-authenticator app.kubernetes.io/managed-by: Helm chart: dex-k8s-authenticator-1.7.0 env: dev heritage: Helm release: dex-auth name: dex-auth-dex-k8s-authenticator namespace: dex resourceVersion: "2907244" uid: c6c3787a-9e24-465e-a734-1d4013158b4aspec: progressDeadlineSeconds: 600 replicas: 1 revisionHistoryLimit: 10 selector: matchLabels: app: dex-k8s-authenticator env: dev release: dex-auth strategy: rollingUpdate: maxSurge: 25% maxUnavailable: 25% type: RollingUpdate template: metadata: annotations: checksum/config: 2fb5ed5e0c4df2a6343b4dedc4af4f78e970a2a9219c1f888c784cbf2a30e994 creationTimestamp: null labels: app: dex-k8s-authenticator env: dev release: dex-auth spec: containers: - args: - --config - config.yaml image: mintel/dex-k8s-authenticator:latest imagePullPolicy: IfNotPresent livenessProbe: failureThreshold: 3 httpGet: path: /healthz port: http scheme: HTTP periodSeconds: 10 successThreshold: 1 timeoutSeconds: 1 name: dex-k8s-authenticator ports: - containerPort: 5555 name: http protocol: TCP readinessProbe: failureThreshold: 3 httpGet: path: /healthz port: http scheme: HTTP periodSeconds: 10 successThreshold: 1 timeoutSeconds: 1 resources: limits: cpu: 500m memory: 256Mi requests: cpu: 100m memory: 128Mi terminationMessagePath: /dev/termination-log terminationMessagePolicy: File volumeMounts: - mountPath: /app/config.yaml name: config subPath: config.yaml - mountPath: /certs/ca.crt name: dex-auth-dex-k8s-authenticator-dex-ca-cert subPath: dex-ca-cert dnsConfig: nameservers: - 10.233.0.3 searches: - cluster.local - svc.cluster.local - test.local dnsPolicy: None restartPolicy: Always schedulerName: default-scheduler securityContext: {} terminationGracePeriodSeconds: 30 volumes: - configMap: defaultMode: 420 name: dex-auth-dex-k8s-authenticator name: config - name: dex-auth-dex-k8s-authenticator-dex-ca-cert secret: defaultMode: 420 secretName: dex-auth-dex-k8s-authenticator-dex-ca-certstatus: availableReplicas: 1 conditions: - lastTransitionTime: "2024-12-15T10:45:09Z" lastUpdateTime: "2024-12-15T10:45:09Z" message: Deployment has minimum availability. reason: MinimumReplicasAvailable status: "True" type: Available - lastTransitionTime: "2024-12-15T10:45:05Z" lastUpdateTime: "2024-12-15T10:45:10Z" message: ReplicaSet "dex-auth-dex-k8s-authenticator-7b787957f7" has successfully progressed. reason: NewReplicaSetAvailable status: "True" type: Progressing observedGeneration: 5 readyReplicas: 1 replicas: 1 updatedReplicas: 1 |
| --- | --- |

проверяем:

|  | root@client:~/autentification-dex-dex-auth# kubectl get ingress -n dexNAME CLASS HOSTS ADDRESS PORTS AGEdex nginx dex.test.local 192.168.1.191 80, 443 73mdex-auth-dex-k8s-authenticator <none> dex-auth.test.local 192.168.1.191 80, 443 73m |
| --- | --- |

идём по адресу:

**https://dex-auth.test.local**

нас сразу перекинет на

**https://dex.test.local/**auth/freeipa/login?back=&state=c365acph2k6v35bqqcbhig72d

![](/news/sidmidru/article-db03824b64600e5b/image-140.png)

тут вводим логин пароль нашего пользователя из freeipa

user1

user1

попадаем во внутрь и видим

![](/news/sidmidru/article-db03824b64600e5b/image-141.png)

данные с freeipa подтянулись и есть инструкция как сделать подключение

нам нужно добавить сертификат k8s на наш клиент:

идём на мастер сервер:

root@kub-master1:~#**sudo cat /etc/kubernetes/pki/ca.crt**

| 1234567891011121314151617181920 | -----BEGIN CERTIFICATE-----MIIC/jCCAeagAwIBAgIBADANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwprdWJlcm5ldGVzMB4XDTI0MDYzMDE0MjIxMFoXDTM0MDYyODE0MjIxMFowFTETMBEGA1UEAxMKa3ViZXJuZXRlczCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAPsSLBw3lg/4zrpycC1xJprK7XRIZK6AyL7DIClHPhAWn+h+m72r4PfZib9wu7/VS0e0SkNhVhEhjTZx1yMkJfP4zwd6IUuFAptZfCgVHnJQnkPeadWUqj/zEIh2ubcgbRwsjG1nh8yNRbtrxd/dpXPveBCFptOi5CqdUYodhoBDic5mYwbk8AbMf71FBhhdq8+8y5TiFlXPglITHxe/0VHCXO3ANUvNJDbDoYUGU5I0Kv3AIwffxSrNlyVhgzX70b7eKaurkmGW577hXCEZapRjHlxl2wUwW+BCKZ7X8MCFbwUVVgVLP0nroKvEVUxQNimalip/RtaqiKrf3PypuUsCAwEAAaNZMFcwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB/wQFMAMBAf8wHQYDVR0OBBYEFB/OXFsCKolo5DCMnaFCxrXMBamSMBUGA1UdEQQOMAyCCmt1YmVybmV0ZXMwDQYJKoZIhvcNAQELBQADggEBAKQ3XdnYLdoS9OeETDMGj3nHXyjZsfc7TupYIpHwUH27Jh9LSzl0oxK1dWoRVizF6P1VKlKoZV7tzPPFEjqQfeE1cDXErY4acWq6v6ta3Og65TH6d9/rIqcUfr7UqZw+oMRi4UGkCP4PGoG8KmcpmGmWBLmzBD/ChB0zE9gN2vsKgGWSYUdZoO3KbrMRE/bdH1jDABXpkXvcnQ+gPaqPw1ZNOcADFWwITbKaRAwWTCspgqKd7D1IAuPRR/kIDPIWCc0rLdvPPuKTmx4Ah3j0TQvc8UW/gQX7OV2iReE0sZ1n/RRR4Fa+9mtUleaODao0MvqHVo+9IYFCTTi1fvRCySE=-----END CERTIFICATE----- |
| --- | --- |

добавляем его на нашем клиенте

root@client:~#**cat > /usr/local/share/ca-certificates/myca.crt**

| 1234567891011121314151617181920 | -----BEGIN CERTIFICATE-----MIIC/jCCAeagAwIBAgIBADANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwprdWJlcm5ldGVzMB4XDTI0MDYzMDE0MjIxMFoXDTM0MDYyODE0MjIxMFowFTETMBEGA1UEAxMKa3ViZXJuZXRlczCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAPsSLBw3lg/4zrpycC1xJprK7XRIZK6AyL7DIClHPhAWn+h+m72r4PfZib9wu7/VS0e0SkNhVhEhjTZx1yMkJfP4zwd6IUuFAptZfCgVHnJQnkPeadWUqj/zEIh2ubcgbRwsjG1nh8yNRbtrxd/dpXPveBCFptOi5CqdUYodhoBDic5mYwbk8AbMf71FBhhdq8+8y5TiFlXPglITHxe/0VHCXO3ANUvNJDbDoYUGU5I0Kv3AIwffxSrNlyVhgzX70b7eKaurkmGW577hXCEZapRjHlxl2wUwW+BCKZ7X8MCFbwUVVgVLP0nroKvEVUxQNimalip/RtaqiKrf3PypuUsCAwEAAaNZMFcwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB/wQFMAMBAf8wHQYDVR0OBBYEFB/OXFsCKolo5DCMnaFCxrXMBamSMBUGA1UdEQQOMAyCCmt1YmVybmV0ZXMwDQYJKoZIhvcNAQELBQADggEBAKQ3XdnYLdoS9OeETDMGj3nHXyjZsfc7TupYIpHwUH27Jh9LSzl0oxK1dWoRVizF6P1VKlKoZV7tzPPFEjqQfeE1cDXErY4acWq6v6ta3Og65TH6d9/rIqcUfr7UqZw+oMRi4UGkCP4PGoG8KmcpmGmWBLmzBD/ChB0zE9gN2vsKgGWSYUdZoO3KbrMRE/bdH1jDABXpkXvcnQ+gPaqPw1ZNOcADFWwITbKaRAwWTCspgqKd7D1IAuPRR/kIDPIWCc0rLdvPPuKTmx4Ah3j0TQvc8UW/gQX7OV2iReE0sZ1n/RRR4Fa+9mtUleaODao0MvqHVo+9IYFCTTi1fvRCySE=-----END CERTIFICATE----- |
| --- | --- |

обновляем

так же добавляем сертификат с dex

root@client:~#**kubectl get secret -n dex dex-tls -o yaml**

| 12345678910111213141516171819202122232425 | root@client:~# kubectl get secret -n dex dex-tls -o yamlapiVersion: v1data: tls.crt tls.keykind: Secretmetadata: annotations: cert-manager.io/alt-names: dex.test.local,dex-auth.test.local cert-manager.io/common-name: dex.test.local cert-manager.io/ip-sans: "" cert-manager.io/subject-countries: RU cert-manager.io/subject-localities: YourCity cert-manager.io/subject-organizations: YourOrganization cert-manager.io/subject-provinces: YourState cert-manager.io/uri-sans: "" creationTimestamp: "2024-12-15T08:55:51Z" labels: controller.cert-manager.io/fao: "true" name: dex-tls namespace: dex resourceVersion: "2902166" uid: 2d5bed1d-1eff-472c-aea2-03a6f1ef0ae9type: kubernetes.io/tls |
| --- | --- |

берём только
**tls.crt**

вот наш сертификат:

| 123456789101112131415161718192021222324252627282930313233 | -----BEGIN CERTIFICATE-----MIIFrzCCA5egAwIBAgIUCH/P7Zinjm1wVdFQhCkWDtqT0+AwDQYJKoZIhvcNAQELBQAwRTELMAkGA1UEBhMCa2cxEzARBgNVBAgMClNvbWUtU3RhdGUxITAfBgNVBAoMGEludGVybmV0IFdpZGdpdHMgUHR5IEx0ZDAgFw0yNDEyMTUwODU1MThaGA8yMDUyMDUwMjA4NTUxOFowaDELMAkGA1UEBhMCUlUxEjAQBgNVBAgMCVlvdXJTdGF0ZTERMA8GA1UEBwwIWW91ckNpdHkxGTAXBgNVBAoMEFlvdXJPcmdhbml6YXRpb24xFzAVBgNVBAMMDmRleC50ZXN0LmxvY2FsMIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEA11xPhtHcJPjMApLBtYx2DjG0FwbB5QB87KrQE1DOc7TI6WeA8t51wPlELbiqiZMD1R+R3ZohQVktwhpWDkrOvQl/pLbX3/GFOuW5Y4EXusVu/tvJEM9VUsSmCkVt4Xk1vpzgO1dnXh/nYBQHzru60jlgtp0CGN79mICMQbm+iiHj/4vku3Bk++4xSyQ07SX6LT8yPJnCAlV/Swp6Vu0MN2I0dmgl9Azz1OCZ/7PTZc+Uz8KBsi4RXfOBCvc/+CF6rTKMJq4FjLfAPhqrlxfgoKq2XKBCFFQh/qAhplS9NUFb20oOPLOLqRwmCK78CA/gzWbIAVDUtIh6eQILn9/LAtffWYHkV7mvyAWO1TAwDeESsvXXiIfFQ23KnQ7gf/86OY3XWOsLGPd4GxNyzqYef427MQq9X37ebtQf61OyR4bSPjL/O1+bMTdio7Mhg4bRaSnGG/hlS0tRdZdTANQMMgoDTeR+DIEus/kwIbnAJ6mD+imxb47urICfVOUADIjnC1RbAlBTLckjFpPpbdOg4FgTWOwjE91cFDpPL2832bcKy2AnCuWcjzJIIr6rH48YjdWGXAt8w1Ov8VDDV2n3NMqb0mx/VDK9lfE2aSM/X8rMagpOICQTaBkdvjKTFaCV5nrNkmOpMA1GhADZ8V5GpJr1NHm5Wk2Kgkkwqksdk68CAwEAAaNyMHAwLgYDVR0RBCcwJYIOZGV4LnRlc3QubG9jYWyCE2RleC1hdXRoLnRlc3QubG9jYWwwHQYDVR0OBBYEFKR85gNF3SLlxBiwOIYA0bCYj8hNMB8GA1UdIwQYMBaAFEHzZYwMZOMjNeMQC21old/vFjvqMA0GCSqGSIb3DQEBCwUAA4ICAQBUJd3Yr1OYZgonjwjGpiUumQ53cuNKWztX5fY7WXU+MBlgzRy+8UAE9ZNB3m07YXjUO3YIuDLLSiwlpCErl4q6wnkCuzlwpbNQPJAsdYGrkRt+kTJMBTAfvt6bNcoLCixu43FJBviwxxp3xUwv0xiM0oCfITuVPddKmse06sj88ZsemhOZH9SSSaSlt9cLGAThIiogZCwBdrNxFv8Vbex4xlO+tVkUfNTIkyomzvY3/StACxKd9aUwuegsUilq1NE8S/DiyLO/7vN3Mh+Kuy0zWtcFyLfB503vRZh9ScivaAEtIVDVG7FmSjODo9GghibAL8TaR10alcoqyp47Jh/Yw4f9o2+uRkiEj+RGuo+wpUMMC0DqGF2hwF5Pc0ypGBBIRbkZGFNgvjahsY0iN3YhOA0ar75T8qD1EBt6CSyxtUdquFsfS/SAL8OWFzgxN8jkwxBa7lTMfmeOwz8hCi17MY+3dfTYGW6pZdnAwwyl/8/vL02pr5n+qexoCuNN60Uc0afH+5fwhn90S1luAkOgxz5BhKyMiMc66tOGuzDjDTjgThfCYmYEg7agjFf3lXH1PvO6IzXy2lcb9gV5IRsaSNOwfiz6wgN5bQ8RA5RF7deZ+jYEvDSYKApuJ8AsoUSKpKr8ZLriDHQQpdc3uY6JCc65wQl+wV+tGp6k51qtNQ==-----END CERTIFICATE----- |
| --- | --- |

root@client:~#**sudo update-ca-certificates**

после того как выполним все команды в https://dex-auth.test.local/ под нашим тестовым пользователем**test1**

нужно поправить конфиг

test1@client:~$**nano ~/.kube/config**

в строке с сервером правим порт

**server: https://192.168.1.112:6443**

проверим кстати token который получаем из dex

test1@client:~$**echo**eyJhbGciOiJSUzI1NiIsImtpZCI6IjkxOWZiNzM4OGI5NmRjYjI4YTdjZjZhZDk5YjJiNzczODgwOTNlMjAifQ.eyJpc3MiOiJodHRwczovL2RleC50ZXN0LmxvY2FsIiwic3ViIjoiQ2dWMWMyVnlNUklIWm5KbFpXbHdZUSIsImF1ZCI6Imt1YmVybmV0ZXMiLCJleHAiOjE3MzQ5NTUzNzYsImlhdCI6MTczNDg2ODk3NiwiYXRfaGFzaCI6IlFuODNpRWdvV1ZyLTRHRXlWdERoeHciLCJjX2hhc2giOiJpaG4wSkxqeGo0c0FnV004YVNsczVRIiwiZW1haWwiOiJ1c2VyMUB0ZXN0LmxvY2FsIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsImdyb3VwcyI6WyJpcGF1c2VycyIsIm5leHVzLWFkbWlucyIsInZhdWx0LWFkbWlucyIsInMzLW1pbmlvLWFkbWlucyIsIms4cy1kZXZvcHMiXSwibmFtZSI6InVzZXIxIHVzZXIxIn0.PUTW-3sZI48JXHqAb-Xhtut2F0Mxv0FvmsR3_QtPL6sB5hVq_ZqwwekWiJQtoSLiv0_0hr62ZJK3zipRK19fNL3mIpYHbRicFPSpfnm560tNxn-5yVq9IkJR_NGIkKGy2AsG6GYOE-jXWDovde4FRw5k4wh5LDopPxpBgSrq6AGK4H591YZnH7SNyZgTotu7HjVC_fzd6LQwuK7b5YiQskiYe6CBCScJnvEB4bwML1HJ8AnuW7UGYPeONyNlU1dThy5M2dA2GsIDEBkL8cBi49JA9ydtuY1pYdk8KQ92AEcZbLPV7HOHXV3JL7qLKbArdWwnEm2gA-xBhHVx7LGx3g |**cut -d '.' -f2 | base64 -d | jq**

получаем такой результат:

| 1234567891011121314151617181920 | { "iss": "https://dex.test.local", "sub": "CgV1c2VyMRIHZnJlZWlwYQ", "aud": "kubernetes", "exp": 1734955376, "iat": 1734868976, "at_hash": "Qn83iEgoWVr-4GEyVtDhxw", "c_hash": "ihn0JLjxj4sAgWM8aSls5Q", "email": "user1@test.local", "email_verified": true, "groups": [ "ipausers", "nexus-admins", "vault-admins", "s3-minio-admins", "k8s-devops" ], "name": "user1 user1"} |
| --- | --- |

видимо что в группе подтянулись все группы из freeipa

|  | "groups": [ "ipausers", "nexus-admins", "vault-admins", "s3-minio-admins", "k8s-devops" ], |
| --- | --- |

теперь надо поправить на мастерах api для oidc

root@kub-master1:~#**cat /etc/kubernetes/manifests/kube-apiserver.yaml**

вот весь конфиг:

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155 | root@kub-master1:~# cat /etc/kubernetes/manifests/kube-apiserver.yamlapiVersion: v1kind: Podmetadata: annotations: kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.1.112:6443 creationTimestamp: null labels: component: kube-apiserver tier: control-plane name: kube-apiserver namespace: kube-systemspec: containers: - command: - kube-apiserver - --advertise-address=192.168.1.112 - --allow-privileged=true - --anonymous-auth=True - --apiserver-count=3 - --authorization-mode=Node,RBAC - --bind-address=0.0.0.0 - --client-ca-file=/etc/kubernetes/ssl/ca.crt - --default-not-ready-toleration-seconds=300 - --default-unreachable-toleration-seconds=300 - --enable-admission-plugins=NodeRestriction - --enable-aggregator-routing=False - --enable-bootstrap-token-auth=true - --endpoint-reconciler-type=lease - --etcd-cafile=/etc/ssl/etcd/ssl/ca.pem - --etcd-certfile=/etc/ssl/etcd/ssl/node-kub-master1.test.local.pem - --etcd-keyfile=/etc/ssl/etcd/ssl/node-kub-master1.test.local-key.pem - --etcd-servers=https://192.168.1.112:2379,https://192.168.1.113:2379,https://192.168.1.114:2379 - --event-ttl=1h0m0s - --kubelet-client-certificate=/etc/kubernetes/ssl/apiserver-kubelet-client.crt - --kubelet-client-key=/etc/kubernetes/ssl/apiserver-kubelet-client.key - --kubelet-preferred-address-types=InternalDNS,InternalIP,Hostname,ExternalDNS,ExternalIP - --profiling=False - --proxy-client-cert-file=/etc/kubernetes/ssl/front-proxy-client.crt - --proxy-client-key-file=/etc/kubernetes/ssl/front-proxy-client.key - --request-timeout=1m0s - --requestheader-allowed-names=front-proxy-client - --requestheader-client-ca-file=/etc/kubernetes/ssl/front-proxy-ca.crt - --requestheader-extra-headers-prefix=X-Remote-Extra- - --requestheader-group-headers=X-Remote-Group - --requestheader-username-headers=X-Remote-User - --secure-port=6443 - --service-account-issuer=https://kubernetes.default.svc.cluster.local - --service-account-key-file=/etc/kubernetes/ssl/sa.pub - --service-account-lookup=True - --service-account-signing-key-file=/etc/kubernetes/ssl/sa.key - --service-cluster-ip-range=10.233.0.0/18 - --service-node-port-range=30000-32767 - --storage-backend=etcd3 - --tls-cert-file=/etc/kubernetes/ssl/apiserver.crt - --tls-private-key-file=/etc/kubernetes/ssl/apiserver.key - --oidc-issuer-url=https://dex.test.local - --oidc-client-id=kubernetes # берём отсюда dexK8sAuthenticator -> clusters -> client_id - --oidc-ca-file=/etc/kubernetes/ssl/dex.crt - --oidc-username-claim=email - --oidc-groups-claim=groups image: registry.k8s.io/kube-apiserver:v1.24.0 imagePullPolicy: IfNotPresent livenessProbe: failureThreshold: 8 httpGet: host: 192.168.1.112 path: /livez port: 6443 scheme: HTTPS initialDelaySeconds: 10 periodSeconds: 10 timeoutSeconds: 15 name: kube-apiserver readinessProbe: failureThreshold: 3 httpGet: host: 192.168.1.112 path: /readyz port: 6443 scheme: HTTPS periodSeconds: 1 timeoutSeconds: 15 resources: requests: cpu: 250m startupProbe: failureThreshold: 30 httpGet: host: 192.168.1.112 path: /livez port: 6443 scheme: HTTPS initialDelaySeconds: 10 periodSeconds: 10 timeoutSeconds: 15 volumeMounts: - mountPath: /etc/ssl/certs name: ca-certs readOnly: true - mountPath: /etc/ca-certificates name: etc-ca-certificates readOnly: true - mountPath: /etc/pki name: etc-pki readOnly: true - mountPath: /etc/ssl/etcd/ssl name: etcd-certs-0 readOnly: true - mountPath: /etc/kubernetes/ssl name: k8s-certs readOnly: true - mountPath: /usr/local/share/ca-certificates name: usr-local-share-ca-certificates readOnly: true - mountPath: /usr/share/ca-certificates name: usr-share-ca-certificates readOnly: true hostNetwork: true priorityClassName: system-node-critical securityContext: seccompProfile: type: RuntimeDefault volumes: - hostPath: path: /etc/ssl/certs type: DirectoryOrCreate name: ca-certs - hostPath: path: /etc/ca-certificates type: DirectoryOrCreate name: etc-ca-certificates - hostPath: path: /etc/pki type: DirectoryOrCreate name: etc-pki - hostPath: path: /etc/ssl/etcd/ssl type: DirectoryOrCreate name: etcd-certs-0 - hostPath: path: /etc/kubernetes/ssl type: DirectoryOrCreate name: k8s-certs - hostPath: path: /usr/local/share/ca-certificates type: DirectoryOrCreate name: usr-local-share-ca-certificates - hostPath: path: /usr/share/ca-certificates type: "" name: usr-share-ca-certificatesstatus: {} |
| --- | --- |

мы же добавляем только oidc

root@kub-master1:~#**cat /etc/kubernetes/manifests/kube-apiserver.yaml| grep -i oidc**

|  | - --oidc-issuer-url=https://dex.test.local - --oidc-client-id=kubernetes # берём отсюда dexK8sAuthenticator -> clusters -> client_id - --oidc-ca-file=/etc/kubernetes/ssl/dex.crt - --oidc-username-claim=email - --oidc-groups-claim=groups |
| --- | --- |

добавляем наш dex сертификат: /etc/kubernetes/ssl/dex.crt и данные изменения по oidc на все мастер сервера:

**- --oidc-client-id**=<client_id>
**- --oidc-username-claim**=email
**- --oidc-groups-claim**=groups
**- --oidc-ca-file**=/etc/kubernetes/ssl/dex.crt

теперь нам нужно создать**rbac**роли:

- для админов/девопсов будем использовать уже существующий ClusterRole - cluster-admin

| 123456789101112131415161718192021222324 | root@client:~# kubectl get clusterroles.rbac.authorization.k8s.io cluster-admin -o yamlapiVersion: rbac.authorization.k8s.io/v1kind: ClusterRolemetadata: annotations: rbac.authorization.kubernetes.io/autoupdate: "true" creationTimestamp: "2024-06-30T14:22:26Z" labels: kubernetes.io/bootstrapping: rbac-defaults name: cluster-admin resourceVersion: "77" uid: 34ce792f-9e65-4b05-aa48-2023e2e6adc9rules:- apiGroups: - '*' resources: - '*' verbs: - '*'- nonResourceURLs: - '*' verbs: - '*' |
| --- | --- |

он уже есть в системе, поэтому для него нужно создать только**ClusterRoleBinding**

**group-devops.yaml**

|  | apiVersion: rbac.authorization.k8s.io/v1kind: ClusterRoleBindingmetadata: name: k8s-devops-cluster-adminsubjects: - kind: Group name: k8s-devops apiGroup: rbac.authorization.k8s.ioroleRef: kind: ClusterRole name: cluster-admin apiGroup: rbac.authorization.k8s.io |
| --- | --- |

root@client:~/autentification-dex-dex-auth#**kubectl apply -f group-devops.yaml**

проверяем:

|  | test1@client:~$ kubectl get nodesNAME STATUS ROLES AGE VERSIONkub-master1.test.local Ready control-plane 181d v1.24.0kub-master2.test.local Ready control-plane 181d v1.24.0kub-master3.test.local Ready control-plane 181d v1.24.0kub-worker1.test.local Ready <none> 181d v1.24.0kub-worker2.test.local Ready <none> 76d v1.24.0kub-worker3.test.local Ready <none> 181d v1.24.0 |
| --- | --- |

как видим под пользователем**test1**@client нам теперь доступен вывод серверов.

2. создадим теперь Role и RoleBinding для доступа пользователей в определённый namespace в нашем случае это dev. Вот файл**group-k8s-users-ro.yaml**

| 1234567891011121314151617181920212223242526 | apiVersion: rbac.authorization.k8s.io/v1kind: Rolemetadata: namespace: dev name: dev-read-onlyrules: - apiGroups: ["", "apps", "batch", "extensions", "networking.k8s.io"] resources: ["*"] verbs: ["get", "list", "watch"]--- apiVersion: rbac.authorization.k8s.io/v1kind: RoleBindingmetadata: name: k8s-users-ro-read-only namespace: devsubjects: - kind: Group name: k8s-users-ro apiGroup: rbac.authorization.k8s.ioroleRef: kind: Role name: dev-read-only apiGroup: rbac.authorization.k8s.io |
| --- | --- |

применим его:

root@client:~/autentification-dex-dex-auth#**kubectl apply -f group-k8s-users-ro.yaml**

добавим пользователя**user2**в группу**k8s-users-ro**в нашAD**Freeipa**

![](/news/sidmidru/article-db03824b64600e5b/image-142.png)

логинимся в**https://dex-auth.test.local/**под нашим user2

![](/news/sidmidru/article-db03824b64600e5b/image-143.png)

далее на нашем клиенте создадим нового пользователя - пусть будет test2

root@client:~#**adduser test2**

root@client:~#**su - test2**

далее копируем данные из dex

|  | kubectl config set-cluster cluster-local \ --server=https://192.168.1.112 |
| --- | --- |

|  | kubectl config set-credentials user2-cluster-local \ --auth-provider=oidc \ --auth-provider-arg="idp-issuer-url=https://dex.test.local" \ --auth-provider-arg="client-id=kubernetes" \ --auth-provider-arg="client-secret=kubernetes-secret" \ --auth-provider-arg="refresh-token=ChlpamVxN2c2d3dkNGZhYWozNXF6dzNhN202EhlkZ2lnYXBndmNta29uNGg2d2V5N3R1ZWF6" \ --auth-provider-arg="id-token=eyJhbGciOiJSUzI1NiIsImtpZCI6IjdlNDdlNTNlMzg4Yzg1YzY1YjkzOGFiOGNiMWZiYjllYTA0ZjIyMzcifQ.eyJpc3MiOiJodHRwczovL2RleC50ZXN0LmxvY2FsIiwic3ViIjoiQ2dWMWMyVnlNaElIWm5KbFpXbHdZUSIsImF1ZCI6Imt1YmVybmV0ZXMiLCJleHAiOjE3MzU1NTY1MzYsImlhdCI6MTczNTQ3MDEzNiwiYXRfaGFzaCI6IlZYWG51ck5pVDVpcm5ZUzFITGE5WXciLCJjX2hhc2giOiIzX3ZwWlJQWUdMZVBScVVVVnJCQ053IiwiZW1haWwiOiJ1c2VyMkB0ZXN0LmxvY2FsIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsImdyb3VwcyI6WyJpcGF1c2VycyIsIm5leHVzLXJvLXVzZXJzIiwidmF1bHQtcm8tdXNlcnMiLCJzMy1taW5pby1yby11c2VycyIsIms4cy11c2Vycy1ybyJdLCJuYW1lIjoidXNlcjIgdXNlcjIifQ.XY5tBN1JDow-6QhP1ir0TcIxOiyjtLSM4d_-K6Cm6qxDOmL1k7hhrUrI58ZdRoqcN55mmPLq99BoH9yO4be_H8PLMqaHhzsw0vr46jRiS-gGQLJUKJtKxSRwPuNjIu-Fz48jF2TFU7XfbbcnzNQwFiPw3_grqoWl0quf9InJozxVpHc0JkNLk9mpvknp7vp7qswfXCS1gSc54deY8g29c2OBKJeB46GlyNMVYLEEboLcjU7mjyby1M8K9lUqm76h6wI6uYP_MgZgkNN4a5zHJFi-pqOx1PX-Al5KZE90CeNc5rUByd1Tj7csd66FKsoi_ZEyxLjx55EC9CErKAdi-Q" |
| --- | --- |

|  | kubectl config set-context user2-cluster-local \ --cluster=cluster-local \ --user=user2-cluster-local |
| --- | --- |

|  | kubectl config use-context user2-cluster-local |
| --- | --- |

далее надо поправить порт для кластера потому что у меня используется 6443

test2@client:~$**nano ~/.kube/config**

| 1234567891011121314151617181920212223242526 | apiVersion: v1clusters:- cluster: server: https://192.168.1.112:6443 name: cluster-localcontexts:- context: cluster: cluster-local user: user2-cluster-local name: user2-cluster-localcurrent-context: user2-cluster-localkind: Configpreferences: {}users:- name: user2-cluster-local user: auth-provider: config: client-id: kubernetes client-secret: kubernetes-secret id-token: eyJhbGciOiJSUzI1NiIsImtpZCI6IjdlNDdlNTNlMzg4Yzg1YzY1YjkzOGFiOGNiMWZiYjllYTA0ZjIyMzcifQ.eyJpc3MiOiJodHRwczovL2RleC50ZXN0LmxvY2FsIiwic3ViIjoiQ2dWMWMyVnlNaElIWm5KbFpXbHdZUSIs> idp-issuer-url: https://dex.test.local refresh-token: ChlpamVxN2c2d3dkNGZhYWozNXF6dzNhN202EhlkZ2lnYXBndmNta29uNGg2d2V5N3R1ZWF6 name: oidc |
| --- | --- |

проверяем:

|  | test2@client:~$ kubectl get nodesError from server (Forbidden): nodes is forbidden: User "user2@test.local" cannot list resource "nodes" in API group "" at the cluster scopetest2@client:~$ kubectl get pod -n devNAME READY STATUS RESTARTS AGEfirst-app-common-chart-6cdf48bf88-5785g 1/1 Running 14 (21h ago) 36dsecond-app-common-chart-6b8457fbfb-sgmh8 1/1 Running 12 (21h ago) 36dtest2@client:~$ kubectl get pod -n dexError from server (Forbidden): pods is forbidden: User "user2@test.local" cannot list resource "pods" in API group "" in the namespace "dex" |
| --- | --- |

как видим мы не можем вывести ни ноды ни поды из неймспейса dex, но нам доступны поды в неймспейсе dev

### []rancher

**kubectl create namespace cattle-system**
**helm repo add rancher-stable https://releases.rancher.com/server-charts/stable**
**helm repo update**

[root@ansible ansible]# cd kubespray-official/autentification-keycloak/certs-rancher/

ca_openssl.cnf

|  | [ v3_ca ]subjectAltName = @alt_names[ alt_names ]DNS.1 = rancher.test.localDNS.2 = *.test.local |
| --- | --- |

rancher_openssl.cnf

| 123456789101112131415161718192021 | [ req ]default_bits = 4096prompt = nodefault_md = sha256distinguished_name = dnreq_extensions = req_ext[ dn ]C = RUST = YourStateL = YourCityO = YourOrganizationCN = rancher.test.local # Основное доменное имя[ req_ext ]subjectAltName = @alt_names[ alt_names ]DNS.1 = rancher.test.localDNS.2 = *.test.local |
| --- | --- |

[root@ansible kubespray-official]#**scp -r /etc/ansible/kubespray-official/autentification-keycloak/ root@192.168.1.121:~/**

root@client:~#**cd ~/autentification-keycloak/certs-rancher/**

root@client:~/autentification-keycloak/certs-rancher#**openssl genrsa -out ca.key 4096**

root@client:~/autentification-keycloak/certs-rancher#**openssl req -x509 -sha256 -new -key ca.key -days 10000 -out ca.crt**

root@client:~/autentification-keycloak/certs-rancher#**openssl genrsa -out rancher.key 4096**

root@client:~/autentification-keycloak/certs-rancher#**openssl req -new -key rancher.key -out rancher.csr -config rancher_openssl.cnf**

root@client:~/autentification-keycloak/certs-rancher#**openssl x509 -req -in rancher.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out rancher.crt -days 10000 -sha256 -extfile ca_openssl.cnf -extensions v3_ca**

создаём секрет

root@client:~/autentification-keycloak/certs-rancher#**kubectl -n cattle-system create secret tls tls-rancher-ingress --cert=./rancher.crt --key=./rancher.key**

запускаем установку:

values-rancher.yaml

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071 | # Основной домен для доступа к Rancherhostname: rancher.test.localreplicas: 1privateCA: truecaCerts: secretName: tls-ca # ваш секрет с CA сертификатом API# extraEnv:# - name: CATTLE_K8S_API_ENDPOINT# value: "https://192.168.1.112:6443"# - name: KUBERNETES_SERVICE_HOST# value: "192.168.1.112"# - name: KUBERNETES_SERVICE_PORT# value: "6443"ingress: # If set to false, ingress will not be created # Defaults to true # options: true, false enabled: true includeDefaultExtraAnnotations: true extraAnnotations: {} ingressClassName: "nginx" # Certain ingress controllers will require the pathType or path to be set to a different value. pathType: ImplementationSpecific path: "/" # backend port number servicePort: 80 tls: # options: rancher, letsEncrypt, secret source: secret secretName: tls-rancher-ingressresources: requests: cpu: "100m" memory: "500Mi" limits: cpu: "1000m" memory: "2024Mi"persistence: enabled: true # Включаем использование persistent volume (если необходимо) storageClass: "nfs-client" # Укажите storageClass, подходящий для вашего кластера accessModes: - ReadWriteOnce size: 10GistartupProbe: enabled: true initialDelaySeconds: 60 periodSeconds: 20 timeoutSeconds: 5 failureThreshold: 30 # Это даст Rancher до 10 минут на запускreadinessProbe: initialDelaySeconds: 30 periodSeconds: 20 timeoutSeconds: 5 failureThreshold: 10livenessProbe: initialDelaySeconds: 300 periodSeconds: 30 timeoutSeconds: 5 failureThreshold: 5 |
| --- | --- |

root@client:~/autentification-keycloak#**scp root@192.168.1.112:/etc/kubernetes/ssl/ca.crt ./ca.crt**

root@client:~/autentification-keycloak#**kubectl -n cattle-system create secret generic tls-ca --from-file=cacerts.pem=./ca.crt**

root@client:~/autentification-keycloak#**helm install rancher rancher-stable/rancher -n cattle-system --version 2.9.3 -f values-rancher.yaml--setbootstrapPassword=ddjjKKSSlldd345hhRRsd**

kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{ "\n" }}'

echo https://rancher.test.local/dashboard/?setup=$(kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}')

получаем

https://rancher.test.local/dashboard/?setup=ddjjKKSSlldd345hhRRsd

![](/news/sidmidru/article-db03824b64600e5b/image-144.png)

================================================

так как rancher в кластере жрёт слишком много ресурсов и у меня тупо сетка отваливалась, я поднял его на отдельной виртуалке:

192.168.1.128

первоначальная установка 4 ядра 4 гб оперативки.

в докере

|  | docker run -d --name=v2.11-head --restart=unless-stopped --privileged -p 80:80 -p 443:443 -v /root/rancher:/var/lib/rancher rancher/rancher:v2.11-head --debug |
| --- | --- |

смотрим пароль

root@debian:~#**docker logs v2.11-head 2>&1 | grep "Bootstrap Password:"**
2025/03/29 06:46:41 [INFO] Bootstrap Password: 9ppkpqdbmwv9txs9w8png5jc4h2btzhktdn6km6b2rp96gk9s9rngv

идём в панель

https://192.168.1.128/dashboard/auth/login

![](/news/sidmidru/article-db03824b64600e5b/image-145.png)

задаём пароль

![](/news/sidmidru/article-db03824b64600e5b/image-146.png)

Password must be at least 12 characters

так что задаём пароль длинней

после установки ресурсы можно порезать до 2 ядер и 2гб оперативки.

### []Rancher интеграция с Freeipa

создаём системного пользователя во freeipa

[root@freeipa-1 ~]#**cd /etc/ipa**
[root@freeipa-1 ipa]#**bash freeipa-sam.sh**

выбираем**1**
вводим**freeipa-1.test.local**
выбираем**3**
вводим**admin**
выбираем**4**
вводим**Secret123**

дальше можем добавлять нового пользователя,
вводим**add**
имя пользователя**rancher**
пароль**Secret123**(можно любой но я делаю везде одинаковый)

проверяем пользователей нажимаем**ls
**

| 1234567891011121314151617181920212223 | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=SET!5.) ssl=trueActions (ready): add | rm | ls | info | passwd | save--- Results ---dn: uid=sudo,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=nexus,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=gitlab,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=vault,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=s3minio,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=k8s-access,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=keycloak,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=rancher,cn=sysaccounts,cn=etc,dc=test,dc=local--- End Results ---> |
| --- | --- |

![](/news/sidmidru/article-db03824b64600e5b/image-147.png)

заполняем поля:

**Hostname/IP**192.168.1.100,192.168.1.100

**Service AccountDN**uid=rancher,cn=sysaccounts,cn=etc,dc=test,dc=local
**User Search Base**cn=users,cn=accounts,dc=test,dc=local
**Group Search Base**cn=groups,cn=accounts,dc=test,dc=local
**Object Class (Users)**inetorgperson
**Login Attribute**uid
**Username Attribute**uid
**User Member Attribute**memberOf
**Object Class (Groups**) groupofnames
**Group Name Attribute**cn
**Group Member Attribute**member
**GroupDNAttribute**entrydn

в самом низу user1 это из freeipa пользователь, под которого вы сразу переключитесь если всё правильно настроено.

![](/news/sidmidru/article-db03824b64600e5b/image-148.png)

![](/news/sidmidru/article-db03824b64600e5b/image-149.png)

![](/news/sidmidru/article-db03824b64600e5b/image-150.png)

проверяем:

![](/news/sidmidru/article-db03824b64600e5b/image-151.png)

как видим появилась возможность авторизации через freeipa

так же можем авторизовываться и через локального пользователя.

![](/news/sidmidru/article-db03824b64600e5b/image-152.png)

![](/news/sidmidru/article-db03824b64600e5b/image-153.png)

как видим подключение успешно произведено.

### []Rancher подключение к k8s кластеру

заходим в rancher под рутом далее

Import -> Generic

![](/news/sidmidru/article-db03824b64600e5b/image-154.png)

![](/news/sidmidru/article-db03824b64600e5b/image-155.png)

**cluster-name**: cluster-local

![](/news/sidmidru/article-db03824b64600e5b/image-156.png)

получаем список команд

![](/news/sidmidru/article-db03824b64600e5b/image-157.png)

так как у меня самоподписанный сертификат то я использую вторую команду:

**curl --insecure -sfL https://192.168.1.128/v3/import/57pzg7t2dnxvtjqn6tj7w2bhwttkr7gddgzcwc9xvgxgsfzw65ml74_c-sv925.yaml | kubectl apply -f -**

результат команды такой

|  | clusterrole.rbac.authorization.k8s.io/proxy-clusterrole-kubeapiserver createdclusterrolebinding.rbac.authorization.k8s.io/proxy-role-binding-kubernetes-master creatednamespace/cattle-system createdserviceaccount/cattle createdclusterrolebinding.rbac.authorization.k8s.io/cattle-admin-binding createdsecret/cattle-credentials-4b82eaf createdclusterrole.rbac.authorization.k8s.io/cattle-admin createdWarning: spec.template.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution.nodeSelectorTerms[0].matchExpressions[0].key: beta.kubernetes.io/os is deprecated since v1.14; use "kubernetes.io/os" insteaddeployment.apps/cattle-cluster-agent createdservice/cattle-cluster-agent created |
| --- | --- |

можем посмотреть состояние процесса:

![](/news/sidmidru/article-db03824b64600e5b/image-158.png)

тут можем посмотреть какие сервера добавились в кластер:

![](/news/sidmidru/article-db03824b64600e5b/image-159.png)

дальше можем посмотреть разные данные по кластеру сторадж классы, ноды и т.д.

одну ноду я выключил так как у меня не хватало ресурсов на моём компе

![](/news/sidmidru/article-db03824b64600e5b/image-160.png)

![](/news/sidmidru/article-db03824b64600e5b/image-161.png)

![](/news/sidmidru/article-db03824b64600e5b/image-162.png)

добавляем группы, заходим из под пользователя user1 так как из под рута будет недоступно - х.з. почему

![](/news/sidmidru/article-db03824b64600e5b/image-163.png)

![](/news/sidmidru/article-db03824b64600e5b/image-164.png)

группа**k8s-devops**будет с полными доступами

добавим группу**k8s-users-ro**для пользователей:

![](/news/sidmidru/article-db03824b64600e5b/image-165.png)

![](/news/sidmidru/article-db03824b64600e5b/image-166.png)

так же ограничиваем доступ группами

![](/news/sidmidru/article-db03824b64600e5b/image-167.png)

чтобы ограничить группу неймспейсами - создадим проект с названием**limited-access-project**

![](/news/sidmidru/article-db03824b64600e5b/image-168.png)

![](/news/sidmidru/article-db03824b64600e5b/image-169.png)

создали проект теперь добавим к нему группу**k8s-users-ro**и удалим пользователя user1

![](/news/sidmidru/article-db03824b64600e5b/image-170.png)

![](/news/sidmidru/article-db03824b64600e5b/image-171.png)

![](/news/sidmidru/article-db03824b64600e5b/image-172.png)

![](/news/sidmidru/article-db03824b64600e5b/image-173.png)

теперь переместим в этот проект нужный нам namespace

![](/news/sidmidru/article-db03824b64600e5b/image-174.png)

![](/news/sidmidru/article-db03824b64600e5b/image-175.png)

![](/news/sidmidru/article-db03824b64600e5b/image-176.png)

любому проекту можно назначать несколько групп с разными правами, например добавим группу k8s-devops как owner

![](/news/sidmidru/article-db03824b64600e5b/image-177.png)

можно кастомайзить доступы при добавлении группы:

![](/news/sidmidru/article-db03824b64600e5b/image-178.png)

так же всегда можно добавлятьRBACруками - rancher нормально это подтягивает.

### []Rancher проверка авторизации для пользователей k8s

во freeipa есть пользователь user-2 в группе k8s-users-ro

![](/news/sidmidru/article-db03824b64600e5b/image-179.png)

авторизуемся с этим пользователем в rancher

![](/news/sidmidru/article-db03824b64600e5b/image-180.png)

теперь нужно скачать kubeconfig

там на выбор можно файл скачать можно в буфер обмена скопировать

![](/news/sidmidru/article-db03824b64600e5b/image-181.png)

теперь идём в консоль

создаём директорию:
user2@client:~$**mkdir ~/.kube**

создаём файл:
user2@client:~$**cat > .kube/config**

и вставляем содержимое которое скопировали:

| 123456789101112131415161718192021222324252627282930313233 | apiVersion: v1kind: Configclusters:- name: "cluster-local" cluster: server: "https://192.168.1.128/k8s/clusters/c-sv925" certificate-authority-data: "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJ2RENDQ\ VdPZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQkdNUnd3R2dZRFZRUUtFeE5rZVc1aGJXbGoKY\ kdsemRHVnVaWEl0YjNKbk1TWXdKQVlEVlFRRERCMWtlVzVoYldsamJHbHpkR1Z1WlhJdFkyRkFNV\ GMwTXpJegpNRGd3TXpBZUZ3MHlOVEF6TWprd05qUTJORE5hRncwek5UQXpNamN3TmpRMk5ETmFNR\ Vl4SERBYUJnTlZCQW9UCkUyUjVibUZ0YVdOc2FYTjBaVzVsY2kxdmNtY3hKakFrQmdOVkJBTU1IV\ 1I1Ym1GdGFXTnNhWE4wWlc1bGNpMWoKWVVBeE56UXpNak13T0RBek1Ga3dFd1lIS29aSXpqMENBU\ VlJS29aSXpqMERBUWNEUWdBRVc0MitBTFYrMTB5NQpNb0h6SFpZRnljazlIcHdKVy9QK3h2SUYwd\ 2RkNmV6NnVGZ2N4cnVrM3lqelFKZEdBaTQ2S0NrcCtNaGlwTHhtCjBPZjEwVUZ6MzZOQ01FQXdEZ\ 1lEVlIwUEFRSC9CQVFEQWdLa01BOEdBMVVkRXdFQi93UUZNQU1CQWY4d0hRWUQKVlIwT0JCWUVGR\ GVnZXBuK01lQmVJdWw1LzZ1cWh5alNvT2pBTUFvR0NDcUdTTTQ5QkFNQ0EwY0FNRVFDSUVkSApzT\ 2FJZEVTS0ZSZkhwNTdBek41SHhMeHkrdGJITWlXUGhHQkcrOVgyQWlBaGo2dmpiOUZjbVhtUjExS\ zhVSE52CnNmSDBmYk9KQklxYzVYTnY1WFJBVFE9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0t"users:- name: "cluster-local" user: token: "kubeconfig-u-kvtgq3ddgxnt9qk:njmk6v2w8cq57gqnwzn8qq598vnb9vqmvxhwtr9mssgzrrl6bftqws"contexts:- name: "cluster-local" context: user: "cluster-local" cluster: "cluster-local"current-context: "cluster-local" |
| --- | --- |

пытаем отобразить всеPOD

user2@client:~$**kubectl get pod -A
**получаем ответ

|  | Error from server (Forbidden): pods is forbidden: User "u-kvtgq3ddgx" cannot list resource "pods" in API group "" at the cluster scope |
| --- | --- |

смотрим наш неймспейс dev там всё ок

|  | user2@client:~$ kubectl get pod -n devNAME READY STATUS RESTARTS AGEfirst-app-common-chart-6cdf48bf88-q9db6 1/1 Running 7 (174m ago) 71dsecond-app-common-chart-6b8457fbfb-fqm8w 1/1 Running 6 (174m ago) 71d |
| --- | --- |

как видим всё работает

### удаление rancher из кластера

|  | # Get the resource types for both namespace and cluster-levelNS_TYPES=$(kubectl api-resources --verbs=list -o name --namespaced=true | grep "cattle.io")CLUSTER_TYPES=$(kubectl api-resources --verbs=list -o name --namespaced=false | grep "cattle.io")echo "Removing finalizers from namespaced Rancher resources"for type in $NS_TYPES; do echo "Removing finalizers for $type" kubectl get $type --all-namespaces -o custom-columns='NAMESPACE:.metadata.namespace','NAME:.metadata.name' --no-headers | awk '{print $1 " " $2}' | xargs -L1 bash -c "kubectl patch --dry-run=client -n \$0 $type/\$1 --type=merge -p \$(kubectl get -n \$0 $type/\$1 -o json | jq -Mcr '.metadata.finalizers // [] | {metadata:{finalizers:map(select(. | (contains(\"controller.cattle.io/\") or contains(\"wrangler.cattle.io/\")) | not ))}}')"doneecho "Removing finalizers from cluster Rancher resources"for type in $CLUSTER_TYPES; do echo "Removing finalizers for $type" kubectl get $type -o name --show-kind --no-headers | awk '{print $1 }' | xargs -L1 bash -c "kubectl patch --dry-run=client \$0 --type=merge -p \$(kubectl get \$0 -o json | jq -Mcr '.metadata.finalizers // [] | {metadata:{finalizers:map(select(. | (contains(\"controller.cattle.io/\") or contains(\"wrangler.cattle.io/\")) | not ))}}')"done |
| --- | --- |

| 123456789101112131415161718192021222324252627282930313233343536 | #!/bin/bashNAMESPACES=$(for ns in $(kubectl get ns -o jsonpath='{.items[*].metadata.name}'); do if kubectl get ns "$ns" -o json | grep -q 'wrangler.cattle.io'; then echo "$ns" fidone)echo "Removing finalizers from namespaces"for ns in $NAMESPACES; do echo "Removing finalizers from namespace $ns" PATCH=$(kubectl get namespace "$ns" -o json | jq -Mc ' { metadata: { finalizers: ( .metadata.finalizers // [] | map( select( (. | contains("controller.cattle.io/") | not) and (. | contains("wrangler.cattle.io/") | not) ) ) ) } } ') if [[ -n "$PATCH" && "$PATCH" != "{}" ]]; then echo "Patching: $PATCH" kubectl patch namespace "$ns" --type=merge -p "$PATCH" else echo "Nothing to patch for $ns" fidone |
| --- | --- |

для удаления crd используем:

|  | for crd in $(kubectl get crd -o name | grep cattle.io | cut -d'/' -f2); do echo "Patching $crd to remove finalizers…"; kubectl patch crd "$crd" --type=json -p='[{"op": "remove", "path": "/metadata/finalizers"}]'; kubectl delete crd "$crd"; done |
| --- | --- |

как только процесс остановится отменяем процесс и перезапускаем и так до победного

### []Keycloak

Keycloak - это приложение для реализации единой точки аутентификации и авторизации. Данную технологию единого входа также называют Single Sign-On или, сокращенно,SSO. А подобные сервисы имеют общее название "Система управления идентификацией и доступом" или Identity and Access Management (IAM).

Keycloak может предоставить возможность пользователям получать права для различных приложений, пройдя один раз процесс аутентификации. Разработчикам не нужно для этого писать много кода. А инженеры DevOps могут настроить аутентификацию через общую базу пользователей для приложений, у которых нет дополнительных механизмов интеграции.

Среди функций и возможностей выделяют:

- SSO.
- Выдачу токенов.
- Двухфакторную аутентификацию.
- Авторизацию через социальные сети.
- Возможность интеграции со службами каталогов.
- Автоматическую аутентификацию с использованием тикетов Kerberos.
- Управление разными изолированными средами (Realm) со своими настройками.
- Свой интерфейс для регистрации и аутентификации пользователей с возможностью настройки внешнего вида.

Keycloak имеет клиент-серверную инфраструктуру. В качестве сервера используется готовый пакет, устанавливаемый на операционную систему (есть поддержка Linux, Windows) или docker-приложение. В качестве клиента используется адаптер — блок кода, который должен использовать разработчик для интеграции своего приложения с сервером.

Поддерживается два стандарта обмена данными аутентификации и авторизации: OpenID Connect иSAML. В зависимости от данного стандарта Keycloak предлагает готовые шаблоны адаптеров для языков программирования, платформ или приложений а также их фреймворков/расширений:

1. Для OpenID Connect:

- Java (Spring Boot, Wildfly ElytronOIDC).
- JavaScript.
- Node.js.
- C#.
- Python.
- Android/iOS.
- Apache Web Server (mod_auth_openidc).

2. ДляSAML:

- Java.
- Apache Web Server (mod_auth_mellon).

Подробнее о поддерживаемых языках можно почитать в[официальной документации](https://www.keycloak.org/docs/latest/securing_apps/).

ставим keycloak в k8s вот офф helm чарт

https://github.com/codecentric/helm-charts/tree/master

добавляем репозиторий

**helm repo add codecentric https://codecentric.github.io/helm-charts**

создаём namespace

**kubectl create ns keycloak**

создаём секрет с логином и паролем - для keycloak:

|  | kubectl create secret generic keycloak-admin-settings -n keycloak \ --from-literal=admin-password=Secret123 \ --from-literal=admin-username=admin |
| --- | --- |

создаём сертификаты чтоб работы по https

root@client:~#**cd ~/autentification-keycloak/certs/**

root@client:~#**cat ca_openssl.cnf**

|  | [ v3_ca ]subjectAltName = @alt_names[ alt_names ]DNS.1 = keycloak.test.local |
| --- | --- |

root@client:~#**cat keycloak_openssl.cnf**

| 123456789101112131415161718192021 | [ req ]default_bits = 4096prompt = nodefault_md = sha256distinguished_name = dnreq_extensions = req_ext[ dn ]C = RUST = YourStateL = YourCityO = YourOrganizationCN = keycloak.test.local # Основное доменное имя[ req_ext ]subjectAltName = @alt_names[ alt_names ]DNS.1 = keycloak.test.local |
| --- | --- |

root@client:~/autentification-keycloak/certs#**openssl genrsa -out ca.key 4096**

root@client:~/autentification-keycloak/certs#**openssl req -x509 -sha256 -new -key ca.key -days 10000 -out ca.crt**

root@client:~/autentification-keycloak/certs#**openssl genrsa -out keycloak.key 4096**

root@client:~/autentification-keycloak/certs#**openssl req -new -key keycloak.key -out keycloak.csr -config keycloak_openssl.cnf**

root@client:~/autentification-keycloak/certs#**openssl x509 -req -in keycloak.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out****keycloak.crt -days 10000 -sha256 -extfile ca_openssl.cnf -extensions v3_ca**

создаём секреты

root@client:~/autentification-keycloak/certs#**kubectl create secret tls keycloak-tls --key keycloak.key --cert keycloak.crt --namespace keycloak**

root@client:~/autentification-keycloak/certs#**kubectl create secret generic keycloak-ca-cert --from-file=ca.crt=./ca.crt -n keycloak**

вот values

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374757677787980818283 | extraEnv: | - name: KEYCLOAK_USER valueFrom: secretKeyRef: name: keycloak-admin-settings key: admin-username - name: KEYCLOAK_PASSWORD valueFrom: secretKeyRef: name: keycloak-admin-settings key: admin-password - name: PROXY_ADDRESS_FORWARDING value: "true" - name: KEYCLOAK_EXTRA_ARGS value: "-Dkeycloak.frontendUrl=https://keycloak.test.local/" - name: KEYCLOAK_ENABLE_HTTPS value: 'true'resources: requests: cpu: "500m" memory: "1024Mi" limits: cpu: "500m" memory: "1024Mi"ingress: enabled: true ingressClassName: "nginx" servicePort: http # annotations: # # Это чтобы nginx всегда редиректил http на https # nginx.ingress.kubernetes.io/force-ssl-redirect: "true" rules: - host: 'keycloak.test.local' paths: - path: / pathType: Prefix tls: - hosts: - 'keycloak.test.local' secretName: "keycloak-tls" console: enabled: true ingressClassName: "nginx" rules: - host: 'keycloak.test.local' paths: - path: /auth/admin/ pathType: Prefix tls: - hosts: - 'keycloak.test.local' secretName: "keycloak-tls"postgresql: enabled: true # PostgreSQL User to create postgresqlUsername: keycloak # PostgreSQL Password for the new user postgresqlPassword: keycloak # PostgreSQL Database to create postgresqlDatabase: keycloak resources: requests: cpu: 500m memory: 500Mi limits: cpu: "1" memory: 1000Mi persistence: enabled: true volumeName: "data" existingClaim: "" mountPath: /bitnami/postgresql subPath: "" storageClass: nfs-client accessModes: - ReadWriteOnce size: 5Gi |
| --- | --- |

тут я настраивал:

ingressClassName
ingress
postgresql.persistence.storageClass
postgresql.persistence.size

ставим:

root@client:~/**cd /etc/ansible/kubespray-official/autentification-keycloak/**

root@client:~/autentification-keycloak#**helm upgrade --install keycloak oci://ghcr.io/codecentric/helm-charts/keycloak --version 18.9.0 -n keycloak -f values.yaml**

дальше ждём пока всё установится и можем заходить:

https://keycloak.test.local/

![](/news/sidmidru/article-db03824b64600e5b/image-182.png)

![](/news/sidmidru/article-db03824b64600e5b/image-183.png)

![](/news/sidmidru/article-db03824b64600e5b/image-184.png)

напомню

логин**admin**
пароль**Secret123**

### []интеграция c Freeipa

для начала создадим системного пользователя во freeipa

[root@freeipa-1 ~]# cd /etc/ipa
[root@freeipa-1 ipa]# bash freeipa-sam.sh

| 1234567891011121314151617 | ### FreeIPA - System Account Manager ###1.) ldapserver=2.) domain= (ldapdomain=)3.) binduser=4.) bindpass=UNSET!5.) ssl=trueActions (conditions not yet met): add | rm | ls | info | passwd | save--- Results ------ End Results ---> 1ldapserver=freeipa-1.test.local |
| --- | --- |

| 123456789101112131415161718 | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=4.) bindpass=UNSET!5.) ssl=trueActions (conditions not yet met): add | rm | ls | info | passwd | save--- Results ------ End Results ---> 3Enter "mgr" for Directory Manager. Otherwise enter the username or full binddn (-D option in ldapsearch)binduser=admin |
| --- | --- |

| 1234567891011121314151617 | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=UNSET!5.) ssl=trueActions (conditions not yet met): add | rm | ls | info | passwd | save--- Results ------ End Results ---> 4Enter password (will not echo): |
| --- | --- |

|  | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=SET!5.) ssl=trueActions (ready): add | rm | ls | info | passwd | save--- Results ------ End Results ---> 5 |
| --- | --- |

| 12345678910111213141516171819 | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=SET!5.) ssl=falseActions (ready): add | rm | ls | info | passwd | save--- Results ------ End Results ---> adduid of new user=keycloakpassword of new user (blank to generate a password)=password expiration date YYYYMMDD (blank for 20380119)= |
| --- | --- |

создал пользователя**keycloak**а пароль**Secret123**

проверим всем пользователей:

| 12345678910111213141516171819202122 | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=SET!5.) ssl=falseActions (ready): add | rm | ls | info | passwd | save--- Results ---dn: uid=sudo,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=nexus,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=gitlab,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=vault,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=s3minio,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=k8s-access,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=keycloak,cn=sysaccounts,cn=etc,dc=test,dc=local--- End Results ---> |
| --- | --- |

всё готово.

## Настройка федерации

Первым делом создадим новый realm. Realm — это пространство нашего приложения. Каждое приложение может иметь свой realm с разными пользователями и настройками авторизации. Master realm используется самим Keycloak и использовать его для чего-нибудь еще неправильно.

Нажимаем**Add realm**

![](/news/sidmidru/article-db03824b64600e5b/image-185.png)

![](/news/sidmidru/article-db03824b64600e5b/image-186.png)

![](/news/sidmidru/article-db03824b64600e5b/image-187.png)

Kubernetes по умолчанию проверяет подтвержден у пользователя email или нет. Так как мы используем собственный LDAP-сервер, то тут эта проверка почти всегда будет возвращать`false`. Давайте отключим представление этого параметра в Kubernetes:

**Client scopes**-->**Email**-->**Mappers**-->**Email verified**(Delete)

![](/news/sidmidru/article-db03824b64600e5b/image-188.png)

![](/news/sidmidru/article-db03824b64600e5b/image-189.png)

![](/news/sidmidru/article-db03824b64600e5b/image-190.png)

Теперь настроим федерацию, для этого перейдем в:

**User federation**-->**Add provider…**-->**ldap**

![](/news/sidmidru/article-db03824b64600e5b/image-191.png)

Приведу пример настройки для FreeIPA:

**Console Display Name**- freeipa-1.test.local
**Edit Mode**-READ_ONLY
**Vendor**- Red Hat Directory Server
**ConnectionURL**- ldap://freeipa-1.test.local:389
**UUIDLDAPattribute**- ipaUniqueID**
**UsersDN**- cn=users,cn=accounts,dc=test,dc=local
**BindDN-**uid=keycloak ,cn=sysaccounts,cn=etc,dc=test,dc=local
**Bind Credential -**Secret123
**Allow Kerberos authentication**- on
**Kerberos Realm -**TEST.LOCAL
**Server Principal**-HTTP/freeipa-1.test.local@TEST.LOCAL
**KeyTab**- /etc/krb5.keytab

![](/news/sidmidru/article-db03824b64600e5b/image-192.png)

![](/news/sidmidru/article-db03824b64600e5b/image-193.png)

теперь перейдём:

**User federation**-->**freeipa.test.local**-->**Mappers**-->**First Name**

![](/news/sidmidru/article-db03824b64600e5b/image-194.png)

![](/news/sidmidru/article-db03824b64600e5b/image-195.png)

**Ldap attribure**- givenName

![](/news/sidmidru/article-db03824b64600e5b/image-196.png)

Теперь включим маппинг групп:

**User federation**-->**freeipa.test.local**-->**Mappers**-->**Create**

![](/news/sidmidru/article-db03824b64600e5b/image-197.png)

**Name**- groups
**Mapper type**- group-ldap-mapper
**LDAPGroups****DN**- cn=groups,cn=accounts,dc=test,dc=local
**User Groups Retrieve Strategy**-GET_GROUPS_FROM_USER_MEMBEROF_ATTRIBUTE

![](/news/sidmidru/article-db03824b64600e5b/image-198.png)

На этом настройка федерации закончена, перейдем к настройке клиента.

## Настройка клиента

Создадим нового клиента (приложение которое будет получать пользователей из Keycloak). Переходим:

**Clients**-->**Create**

![](/news/sidmidru/article-db03824b64600e5b/image-199.png)

**ClientID-**kubernetes
**Client Protocol**- openid-connect
**RootURL**- http://kubernetes.test.local/

![](/news/sidmidru/article-db03824b64600e5b/image-200.png)

**Access Type**- confidential
**Valid Redirect URIs**- http://kubernetes.test.local/*
**AdminURL**- http://kubernetes.test.local/

![](/news/sidmidru/article-db03824b64600e5b/image-201.png)

Так же создадим**scope**для групп:

**Client Scopes**-->**Create**

**Name**groups

![](/news/sidmidru/article-db03824b64600e5b/image-202.png)

![](/news/sidmidru/article-db03824b64600e5b/image-203.png)

И настроим mapper для них:

**Client Scopes**-->**groups**-->**Mappers**-->**Create**

**Name**- groups
**Mapper Type**- Group membership
**Toke****n Claim Name**- groups

![](/news/sidmidru/article-db03824b64600e5b/image-204.png)

![](/news/sidmidru/article-db03824b64600e5b/image-205.png)

Теперь нам нужно включить маппинг груп в нашем client scope:

**Clients**-->**kubernetes**-->**Client Scopes**-->**Default Client Scopes**

![](/news/sidmidru/article-db03824b64600e5b/image-206.png)

Выбираем**groups**в**Available Client Scopes**, нажимаем**Add selected**

![](/news/sidmidru/article-db03824b64600e5b/image-207.png)

![](/news/sidmidru/article-db03824b64600e5b/image-208.png)

Теперь настроим аутентификацию нашего приложения, переходим:

**Clients**-->**kubernetes**

**Authorization Enabled**-ON

![](/news/sidmidru/article-db03824b64600e5b/image-209.png)

Нажимем**save**и на этом настройка клиента завершена,

проверим что в keycloak подтягиваются группы и пользователи с Freeipa

![](/news/sidmidru/article-db03824b64600e5b/image-210.png)

![](/news/sidmidru/article-db03824b64600e5b/image-211.png)

теперь на вкладке

**Clients**-->**kubernetes**-->**Credentials**

вы сможете получить**Secret**который мы будем использовать в дальнейшем.

## []Настройка Kubernetes

Настройка Kubernetes для OIDC-авторизации. Все что вам нужно это положить CA-сертификат вашего OIDC-сервера в`/etc/kubernetes/ssl/keycloak.crt`и добавить необходимые опции для kube-apiserver.
Для этого обновите`/etc/kubernetes/manifests/kube-apiserver.yaml`на всех ваших мастерах:

сначала раскидаем сертификаты по всем серверам:

root@client:~/autentification-keycloak/certs#**scp keycloak.crt root@192.168.1.112:/etc/kubernetes/ssl/keycloak.crt**
root@client:~/autentification-keycloak/certs#**scp keycloak.crt root@192.168.1.113:/etc/kubernetes/ssl/keycloak.crt**
root@client:~/autentification-keycloak/certs#**scp keycloak.crt root@192.168.1.114:/etc/kubernetes/ssl/keycloak.crt**
root@client:~/autentification-keycloak/certs#**scp keycloak.crt root@192.168.1.115:/etc/kubernetes/ssl/keycloak.crt**
root@client:~/autentification-keycloak/certs#**scp keycloak.crt root@192.168.1.116:/etc/kubernetes/ssl/keycloak.crt**
root@client:~/autentification-keycloak/certs#**scp keycloak.crt root@192.168.1.117:/etc/kubernetes/ssl/keycloak.crt**

теперь на мастерах правим файл**/etc/kubernetes/manifests/kube-apiserver.yaml**

добавляем следующее:

|  | - --oidc-issuer-url=https://keycloak.test.local/auth/realms/kubernetes - --oidc-client-id=kubernetes - --oidc-ca-file=/etc/kubernetes/ssl/keycloak.crt - --oidc-username-claim=email - --oidc-groups-claim=groups |
| --- | --- |

root@kub-master1:~#**nano /etc/kubernetes/manifests/kube-apiserver.yaml**

весь файл выглядит вот так:

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154 | apiVersion: v1kind: Podmetadata: annotations: kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.1.112:6443 creationTimestamp: null labels: component: kube-apiserver tier: control-plane name: kube-apiserver namespace: kube-systemspec: containers: - command: - kube-apiserver - --advertise-address=192.168.1.112 - --allow-privileged=true - --anonymous-auth=True - --apiserver-count=3 - --authorization-mode=Node,RBAC - --bind-address=0.0.0.0 - --client-ca-file=/etc/kubernetes/ssl/ca.crt - --default-not-ready-toleration-seconds=300 - --default-unreachable-toleration-seconds=300 - --enable-admission-plugins=NodeRestriction - --enable-aggregator-routing=False - --enable-bootstrap-token-auth=true - --endpoint-reconciler-type=lease - --etcd-cafile=/etc/ssl/etcd/ssl/ca.pem - --etcd-certfile=/etc/ssl/etcd/ssl/node-kub-master1.test.local.pem - --etcd-keyfile=/etc/ssl/etcd/ssl/node-kub-master1.test.local-key.pem - --etcd-servers=https://192.168.1.112:2379,https://192.168.1.113:2379,https://192.168.1.114:2379 - --event-ttl=1h0m0s - --kubelet-client-certificate=/etc/kubernetes/ssl/apiserver-kubelet-client.crt - --kubelet-client-key=/etc/kubernetes/ssl/apiserver-kubelet-client.key - --kubelet-preferred-address-types=InternalDNS,InternalIP,Hostname,ExternalDNS,ExternalIP - --profiling=False - --proxy-client-cert-file=/etc/kubernetes/ssl/front-proxy-client.crt - --proxy-client-key-file=/etc/kubernetes/ssl/front-proxy-client.key - --request-timeout=1m0s - --requestheader-allowed-names=front-proxy-client - --requestheader-client-ca-file=/etc/kubernetes/ssl/front-proxy-ca.crt - --requestheader-extra-headers-prefix=X-Remote-Extra- - --requestheader-group-headers=X-Remote-Group - --requestheader-username-headers=X-Remote-User - --secure-port=6443 - --service-account-issuer=https://kubernetes.default.svc.cluster.local - --service-account-key-file=/etc/kubernetes/ssl/sa.pub - --service-account-lookup=True - --service-account-signing-key-file=/etc/kubernetes/ssl/sa.key - --service-cluster-ip-range=10.233.0.0/18 - --service-node-port-range=30000-32767 - --storage-backend=etcd3 - --tls-cert-file=/etc/kubernetes/ssl/apiserver.crt - --tls-private-key-file=/etc/kubernetes/ssl/apiserver.key - --oidc-issuer-url=https://keycloak.test.local/auth/realms/kubernetes - --oidc-client-id=kubernetes - --oidc-ca-file=/etc/kubernetes/ssl/keycloak.crt - --oidc-username-claim=email - --oidc-groups-claim=groups image: registry.k8s.io/kube-apiserver:v1.24.0 imagePullPolicy: IfNotPresent livenessProbe: failureThreshold: 8 httpGet: host: 192.168.1.112 path: /livez port: 6443 scheme: HTTPS initialDelaySeconds: 10 periodSeconds: 10 timeoutSeconds: 15 name: kube-apiserver readinessProbe: failureThreshold: 3 httpGet: host: 192.168.1.112 path: /readyz port: 6443 scheme: HTTPS periodSeconds: 1 timeoutSeconds: 15 resources: requests: cpu: 250m startupProbe: failureThreshold: 30 httpGet: host: 192.168.1.112 path: /livez port: 6443 scheme: HTTPS initialDelaySeconds: 10 periodSeconds: 10 timeoutSeconds: 15 volumeMounts: - mountPath: /etc/ssl/certs name: ca-certs readOnly: true - mountPath: /etc/ca-certificates name: etc-ca-certificates readOnly: true - mountPath: /etc/pki name: etc-pki readOnly: true - mountPath: /etc/ssl/etcd/ssl name: etcd-certs-0 readOnly: true - mountPath: /etc/kubernetes/ssl name: k8s-certs readOnly: true - mountPath: /usr/local/share/ca-certificates name: usr-local-share-ca-certificates readOnly: true - mountPath: /usr/share/ca-certificates name: usr-share-ca-certificates readOnly: true hostNetwork: true priorityClassName: system-node-critical securityContext: seccompProfile: type: RuntimeDefault volumes: - hostPath: path: /etc/ssl/certs type: DirectoryOrCreate name: ca-certs - hostPath: path: /etc/ca-certificates type: DirectoryOrCreate name: etc-ca-certificates - hostPath: path: /etc/pki type: DirectoryOrCreate name: etc-pki - hostPath: path: /etc/ssl/etcd/ssl type: DirectoryOrCreate name: etcd-certs-0 - hostPath: path: /etc/kubernetes/ssl type: DirectoryOrCreate name: k8s-certs - hostPath: path: /usr/local/share/ca-certificates type: DirectoryOrCreate name: usr-local-share-ca-certificates - hostPath: path: /usr/share/ca-certificates type: "" name: usr-share-ca-certificatesstatus: {} |
| --- | --- |

root@kub-master2:~#**nano /etc/kubernetes/manifests/kube-apiserver.yaml**
root@kub-master3:~#**nano /etc/kubernetes/manifests/kube-apiserver.yaml**

А так-же поправим kubeadm конфиг в кластере, что бы не потерять эти настройки при обновлении:

root@client:~#**kubectl edit -n kube-system configmaps kubeadm-config**

|  | ...data: ClusterConfiguration: | apiServer: extraArgs: oidc-ca-file: /etc/kubernetes/ssl/keycloak.crt oidc-client-id: kubernetes oidc-groups-claim: groups oidc-issuer-url: https://keycloak.test.local/auth/realms/kubernetes oidc-username-claim: email... |
| --- | --- |

вот так выглядит весь конфиг:

root@client:~#**kubectl get -n kube-system configmaps kubeadm-config -o yaml**

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104 | apiVersion: v1data: ClusterConfiguration: | apiServer: certSANs: - kubernetes - kubernetes.default - kubernetes.default.svc - kubernetes.default.svc.cluster.local - 10.233.0.1 - localhost - 127.0.0.1 - kub-master1.test.local - kub-master2.test.local - kub-master3.test.local - lb-apiserver.kubernetes.local - 192.168.1.112 - 192.168.1.113 - 192.168.1.114 - kub-master1 - kub-master2 - kub-master3 extraArgs: allow-privileged: "true" anonymous-auth: "True" apiserver-count: "3" authorization-mode: Node,RBAC bind-address: 0.0.0.0 default-not-ready-toleration-seconds: "300" default-unreachable-toleration-seconds: "300" enable-aggregator-routing: "False" endpoint-reconciler-type: lease event-ttl: 1h0m0s kubelet-preferred-address-types: InternalDNS,InternalIP,Hostname,ExternalDNS,ExternalIP profiling: "False" request-timeout: 1m0s service-account-lookup: "True" service-cluster-ip-range: 10.233.0.0/18 service-node-port-range: 30000-32767 storage-backend: etcd3 oidc-ca-file: /etc/kubernetes/ssl/keycloak.crt oidc-client-id: kubernetes oidc-groups-claim: groups oidc-issuer-url: https://keycloak.test.local/auth/realms/kubernetes oidc-username-claim: email extraVolumes: - hostPath: /usr/share/ca-certificates mountPath: /usr/share/ca-certificates name: usr-share-ca-certificates readOnly: true timeoutForControlPlane: 5m0s apiVersion: kubeadm.k8s.io/v1beta3 certificatesDir: /etc/kubernetes/ssl clusterName: cluster.local controlPlaneEndpoint: 192.168.1.112:6443 controllerManager: extraArgs: bind-address: 0.0.0.0 cluster-cidr: 10.233.64.0/18 configure-cloud-routes: "false" leader-elect-lease-duration: 15s leader-elect-renew-deadline: 10s node-cidr-mask-size: "24" node-monitor-grace-period: 40s node-monitor-period: 5s profiling: "False" service-cluster-ip-range: 10.233.0.0/18 terminated-pod-gc-threshold: "12500" dns: imageRepository: registry.k8s.io/coredns imageTag: v1.8.6 etcd: external: caFile: /etc/ssl/etcd/ssl/ca.pem certFile: /etc/ssl/etcd/ssl/node-kub-master1.test.local.pem endpoints: - https://192.168.1.112:2379 - https://192.168.1.113:2379 - https://192.168.1.114:2379 keyFile: /etc/ssl/etcd/ssl/node-kub-master1.test.local-key.pem imageRepository: registry.k8s.io kind: ClusterConfiguration kubernetesVersion: v1.24.0 networking: dnsDomain: cluster.local podSubnet: 10.233.64.0/18 serviceSubnet: 10.233.0.0/18 scheduler: extraArgs: bind-address: 0.0.0.0 config: /etc/kubernetes/kubescheduler-config.yaml extraVolumes: - hostPath: /etc/kubernetes/kubescheduler-config.yaml mountPath: /etc/kubernetes/kubescheduler-config.yaml name: kubescheduler-config readOnly: truekind: ConfigMapmetadata: creationTimestamp: "2024-06-30T14:22:29Z" name: kubeadm-config namespace: kube-system resourceVersion: "3343644" uid: 8f3983c4-d932-4a02-8928-fd084b04b401 |
| --- | --- |

### []Teleport

**Teleport**– это инструмент для безопасного подключения к Kubernetes-кластерам, а также к другомуПО. Помимо Kubernetes, Teleport можно использовать для аутентификации с такими системами как:

- Облачные провайдеры – Amazon, Google Cloud, Microsoft Azure;
- Операционные системы – Windows, Linux;
- СУБД– Redis, CockroachDB, MongoDB;
- Системы для поиска и анализа данных – Elasticsearch

достоинства Teleport:

#### Способы подключения

Teleport позволяет удаленно подключаться к Kubernetes-кластерам с использованием протоколовSSHилиTLS. Также присутствует встроенный веб-интерфейс.

#### Аудит

Teleport отслеживает все действия, которые пользователь совершает внутри системы (кластера), и предоставляет расширенные функции, такие как фильтры, хронология, уведомления, управление учетными записями и т. д.

#### Аутентификация

Teleport использует многофакторную аутентификацию, чтобы удостоверить личность пользователя.

#### Управление учетными записями

Teleport может управлять учетными записями для того, чтобы проверить, что пользователь имеет доступ только к разрешенным ресурсам.

## Архитектура и принцип работы Teleport

Teleport написан на языке Go и состоит из трех независимых исполняемых файлов:

- **tsh**(клиент командной строки);
- **tctl**(инструмент администрирования);
- **teleport**(серверный демон).

Представляет собой прокси-сервер, предназначенный для доступа и аутентификации к требуемым системам (операционным системам,СУБДи т. д.). Управление доступом реализовано на основе ролейRBAC.

Клиент командной строки tsh предназначен для входа на конечные ресурсы и выполнения команд.

Инструмент администрирования tctl используется для создания пользователей, ключей сертификатов, а также может применяться для изменения динамической конфигурации кластеров Kubernetes, например, для создания новых ролей.

Демон сервера teleport может работать в трех режимах:

- **Node**. В этом режиме демон предоставляетSSHи Kubernetes доступ к серверу, на котором он работает;
- **Прокси-сервер**. В этом режиме демон действует как удостоверяющий личность прокси для всех протоколов, поддерживаемых Teleport (SSH,HTTPS, KubernetesAPI);
- **Сервер аутентификации**. В этом режиме демон действует как центр сертификации, который выдает сертификаты для пользователей. Также хранит журнал аудита.

![](/news/sidmidru/article-db03824b64600e5b/image-212.png)

работает Teleport следующим образом:

- Пользователь выбирает один из нескольких способов подключения к кластеру Kubernetes (например tsh);
- Далее запрос переходит к серверному демону teleport, который, в свою очередь, отправляет запрос Identity Provider – системе, предназначенной для создания и хранения цифровых идентификационных данных (логин, пароль и т. д). В качестве провайдеров Teleport поддерживает следующие системы:

- Azure Active Directory;
- Active Directory;
- Google Workspace;
- GitHub;
- GitLab;
- OneLogin;
- OIDC;
- SSO

- После того как в Identity Provider найдены аутентификационные данные пользователя, запрос возвращается демону teleport, который обрабатывает поступивший запрос и разрешает доступ к кластеру Kubernetes или другой конечной системе.

### []установка teleport

офф документация

https://goteleport.com/docs/admin-guides/deploy-a-cluster/helm-deployments/custom/

вот мой values

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374757677787980818283848586878889909192 | clusterName: teleport.test.localproxyListenerMode: multiplexproxyProtocol: "off"log: level: DEBUGteleport: dataDir: /var/lib/teleport authService: enabled: true storage: type: dir proxyService: enabled: true publicAddr: teleport.test.local:443 web_listen_addr: 0.0.0.0:443 # монтируем CA, чтобы Teleport доверял своему же TLS-сертификату extraVolumes: - name: teleport-ca secret: secretName: teleport-ca-cert extraVolumeMounts: - name: teleport-ca mountPath: /etc/ssl/certs readOnly: true env: - name: SSL_CERT_FILE value: /etc/ssl/certs/ca.pem kubeService: enabled: true kubeClusterName: "cluster.local" sshService: enabled: falseservice: type: ClusterIPpersistence: enabled: true accessModes: - ReadWriteMany size: 2Gi storageClassName: nfs-clientingress: enabled: true spec: ingressClassName: nginx rules: - host: teleport.test.local http: paths: - path: / pathType: Prefix backend: service: name: teleport-cluster port: number: 443# Глобальные аннотации (chart автоматически подхватит annotations.ingress для metadata.ingress.annotations)annotations: ingress: nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"# Секрет с TLS-сертификатом для Teleport proxy (chart смонтирует его в /etc/teleport/certs)tls: existingSecretName: teleport-tls existingCASecretName: teleport-ca-certauth: tls: enabled: true certFile: /etc/teleport/certs/tls.crt keyFile: /etc/teleport/certs/tls.key caFile: /etc/teleport/certs/ca.crt extraVolumes: - name: teleport-tls secret: secretName: teleport-tls extraVolumeMounts: - name: teleport-tls mountPath: /etc/teleport/certs readOnly: true |
| --- | --- |

домен
teleport.test.local

**helm repo add teleport https://charts.releases.teleport.dev**

**helm repo update**

**kubectl create ns teleport**

создаём сертификат

cat > ca_openssl.cnf

|  | [ v3_ca ]subjectAltName = @alt_names[ alt_names ]DNS.1 = teleport.test.localDNS.2 = *.teleport.test.local |
| --- | --- |

cat > teleport_openssl.cnf

| 123456789101112131415161718192021 | [ req ]default_bits = 4096prompt = nodefault_md = sha256distinguished_name = dnreq_extensions = req_ext[ dn ]C = RUST = YourStateL = YourCityO = YourOrganizationCN = teleport.test.local[ req_ext ]subjectAltName = @alt_names[ alt_names ]DNS.1 = teleport.test.localDNS.2 = *.teleport.test.local |
| --- | --- |

**openssl genrsa -out ca.key 4096**

**openssl req -x509 -sha256 -new -key ca.key -days 10000 -out ca.crt**

**openssl genrsa -out teleport.key 4096**

**openssl req -new -key teleport.key -out teleport.csr -config teleport_openssl.cnf**

**openssl x509 -req -in teleport.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out teleport.crt -days 10000 -sha256 -extfile ca_openssl.cnf -extensions v3_ca**

создаём секрет в k8s

**kubectl -n teleport create secret generic teleport-ca-cert --from-file=ca.pem=./ca.crt**

|  | kubectl create secret generic teleport-tls \ --from-file=tls.crt=./teleport.crt \ --from-file=tls.key=./teleport.key \ --from-file=ca.crt=./ca.crt \ -n teleport |
| --- | --- |

на клиентах добавляем сертификаты в доверенные

**mkdir /usr/local/share/ca-certificates/teleport/**

**cp teleport.crt ca.crt /usr/local/share/ca-certificates/teleport/**

**update-ca-certificates**

ставим сам чарт

**helm upgrade --install teleport-cluster teleport/teleport-cluster -n teleport -f values.yaml --version 17.4.5**

| 123456789101112131415161718 | root@client:~/autentification-teleport# kubectl get ingress -n teleportNAME CLASS HOSTS ADDRESS PORTS AGEteleport-cluster-proxy nginx teleport.test.local,*.teleport.test.local 192.168.1.191 80, 443 6d19hroot@client:~/autentification-teleport# kubectl get pod -n teleportNAME READY STATUS RESTARTS AGEteleport-cluster-auth-f45664b6b-kp5s8 1/1 Running 2 (151m ago) 6d19hteleport-cluster-proxy-f856f76d9-9gdgg 1/1 Running 1 (151m ago) 6d4hteleport-cluster-proxy-f856f76d9-xbgjj 1/1 Running 1 (151m ago) 6d4hroot@client:~/autentification-teleport# kubectl get ingress -n teleportNAME CLASS HOSTS ADDRESS PORTS AGEteleport-cluster-proxy nginx teleport.test.local,*.teleport.test.local 192.168.1.191 80, 443 6d19hroot@client:~/autentification-teleport# kubectl get pvc -n teleportNAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGEteleport-cluster Bound pvc-f3a8dadb-7a60-4009-b8de-2c5cad5c932b 10Gi RWO nfs-client 6d19h |
| --- | --- |

заходим

**https://teleport.test.local/**

![](/news/sidmidru/article-db03824b64600e5b/image-213.png)

создаём админа

**kubectl exec -it -n teleport deploy/teleport-cluster-auth -- tctl users add admin --roles=editor,auditor,access**

|  | root@client:~/autentification-teleport# kubectl exec -it -n teleport deploy/teleport-cluster-auth -- tctl users add admin --roles=editor,auditor,accessUser "admin" has been created but requires a password. Share this URL with the user to complete user setup, link is valid for 1h:https://teleport.test.local:443/web/invite/01ea692b034d44afb0ea6e61a6c6b3a5NOTE: Make sure teleport.test.local:443 points at a Teleport proxy which users can access. |
| --- | --- |

получаем токен, по которому заходим

https://teleport.test.local:443/web/invite/01ea692b034d44afb0ea6e61a6c6b3a5

![](/news/sidmidru/article-db03824b64600e5b/image-214.png)

пароль минимум 12 символов

![](/news/sidmidru/article-db03824b64600e5b/image-215.png)

![](/news/sidmidru/article-db03824b64600e5b/image-216.png)

мы можем использовать только passkey -там нужно использовать usb флешку или mfa через мобилу приложение на андройде называется**AUTHENTICATOR**

там сканим наш код

![](/news/sidmidru/article-db03824b64600e5b/image-217.png)

![](/news/sidmidru/article-db03824b64600e5b/image-218.png)

![](/news/sidmidru/article-db03824b64600e5b/image-219.png)

**teleportНЕПОДДЕРЖИВАЕТ**интеграцию с**freeipa**напрямую, возможно будет в платной версии

## []Подключение кластера Kubernetes к Teleport

Для того чтобы использовать Teleport для входа в кластер Kubernetes, необходимо установить агент. Для начала проверим, работает ли аутентификация в Teleport при помощи следующей команды:

ставим teleport-connect, вот офф сайт

https://goteleport.com/download/?product=connect

версия сервера 17.4.5 такую же версию ставим и для клиента

вот команда

**curl https://goteleport.com/static/install-connect.sh | bash -s 17.4.5**

проверим коннект, для этого получим команду вот тут:

![](/news/sidmidru/article-db03824b64600e5b/image-220.png)

![](/news/sidmidru/article-db03824b64600e5b/image-221.png)

tsh login --proxy=teleport.test.local:443 --auth=local --user=admin teleport.test.local

|  | user1@client:~$ tsh login --proxy=teleport.test.local:443 --auth=local --user=admin teleport.test.localEnter password for Teleport user admin:Enter an OTP code from a device:> Profile URL: https://teleport.test.local:443 Logged in as: admin Cluster: teleport.test.local Roles: access, auditor, editor Kubernetes: enabled Valid until: 2025-05-03 23:46:47 +0600 +06 [valid for 12h0m0s] Extensions: login-ip, permit-agent-forwarding, permit-port-forwarding, permit-pty, private-key-policy |
| --- | --- |

нужно будет ввести пароль от нашего админа иOTPкод с мобилы.

чтобы был доступ к кластеру нужно добавить прав,

Создадим роль**k8s-admin**

![](/news/sidmidru/article-db03824b64600e5b/image-222.png)

![](/news/sidmidru/article-db03824b64600e5b/image-223.png)

![](/news/sidmidru/article-db03824b64600e5b/image-224.png)

![](/news/sidmidru/article-db03824b64600e5b/image-225.png)

в группу добавим
**system:masters**

![](/news/sidmidru/article-db03824b64600e5b/image-226.png)

![](/news/sidmidru/article-db03824b64600e5b/image-227.png)

![](/news/sidmidru/article-db03824b64600e5b/image-228.png)

роль создана:

![](/news/sidmidru/article-db03824b64600e5b/image-229.png)

теперь добавим нашему пользователю эту роль:

![](/news/sidmidru/article-db03824b64600e5b/image-230.png)

![](/news/sidmidru/article-db03824b64600e5b/image-231.png)

![](/news/sidmidru/article-db03824b64600e5b/image-232.png)

![](/news/sidmidru/article-db03824b64600e5b/image-233.png)

после добавления новой роли нужно пользователю перелогиниться:

|  | user1@client:~$ tsh login --proxy=teleport.test.local:443 --auth=local --user=admin teleport.test.local> Profile URL: https://teleport.test.local:443 Logged in as: admin Cluster: teleport.test.local Roles: access, auditor, editor Kubernetes: enabled Valid until: 2025-05-03 23:46:47 +0600 +06 [valid for 10h44m0s] Extensions: login-ip, permit-agent-forwarding, permit-port-forwarding, permit-pty, private-key-policy |
| --- | --- |

как видим при обычном логине Roles остаются старыми поэтому делаем логаут

|  | user1@client:~$ tsh logoutLogged out all users from all proxies.user1@client:~$ tsh login --proxy=teleport.test.local:443 --auth=local --user=admin teleport.test.localEnter password for Teleport user admin:Enter an OTP code from a device:> Profile URL: https://teleport.test.local:443 Logged in as: admin Cluster: teleport.test.local Roles: access, auditor, editor, k8s-admin Kubernetes: enabled Kubernetes users: admin Valid until: 2025-05-04 01:04:20 +0600 +06 [valid for 12h0m0s] Extensions: login-ip, permit-agent-forwarding, permit-port-forwarding, permit-pty, private-key-policy |
| --- | --- |

всё, теперь как видим в наших ролях есть k8s-admin

возвращаемся сюда:

![](/news/sidmidru/article-db03824b64600e5b/image-234.png)

мы уже залогинились, выполним следующие команды:

**exportKUBECONFIG=${HOME?}/teleport-kubeconfig.yaml**

**tsh kube login teleport.test.local**

получим следующее уведомление:

|  | user1@client:~$ tsh kube login teleport.test.localLogged into Kubernetes cluster "teleport.test.local".Your Teleport cluster runs behind a layer 7 load balancer or reverse proxy.To access the cluster, use "tsh kubectl" which is a fully featured "kubectl"command that works when the Teleport cluster is behind layer 7 load balancer orreverse proxy. To run the Kubernetes client, use: tsh kubectl versionOr, start a local proxy with "tsh proxy kube" and use the kubeconfigprovided by the local proxy with your native Kubernetes clients: tsh proxy kube -p 8443 |
| --- | --- |

это говорит о том что так как мы используем ingress т.е. 7 уровень модели osi (application layer - прикладной уровень)
использовать напрямую утилиту kubectl будет невозможно, поэтому есть 2 варианта 1 это добавлять**tsh перед kubectl**или запустить в одной консоли
**tsh proxy kube -p 8443**

а в другой использовать kubectl.
проверим:

| 1234567891011121314151617181920212223242526 | user1@client:~$ tsh kubectl get nsNAME STATUS AGEcattle-system Active 19dcattle-ui-plugin-system Active 42dcert-manager Active 300dcluster-fleet-local-local-1a3d67d0a899 Active 47ddefault Active 306ddev Active 161ddex Active 145delk Active 294dfleet-default Active 300dfleet-local Active 300dingress-nginx Active 19dkeycloak Active 118dkube-node-lease Active 306dkube-public Active 306dkube-system Active 306dlocal Active 300dloki Active 166dmetallb-system Active 301dmonitoring Active 299dprod Active 162dteleport Active 19dtest Active 163dvault Active 165d |
| --- | --- |

как видим работает.

### Проверим ограниченный доступ только к одному из namespace например это будет пользователь user2

создадим пользователя

user2

![](/news/sidmidru/article-db03824b64600e5b/image-235.png)

![](/news/sidmidru/article-db03824b64600e5b/image-236.png)

![](/news/sidmidru/article-db03824b64600e5b/image-237.png)

https://teleport.test.local/web/invite/70e8b53c1468f5ed82efa35480dc13ef

получив ссылку проходим по ней:

![](/news/sidmidru/article-db03824b64600e5b/image-238.png)

![](/news/sidmidru/article-db03824b64600e5b/image-239.png)

![](/news/sidmidru/article-db03824b64600e5b/image-240.png)

![](/news/sidmidru/article-db03824b64600e5b/image-241.png)

![](/news/sidmidru/article-db03824b64600e5b/image-242.png)

привязываемся поOTP

создадим ещё одну роль с доступом к namespace dev

![](/news/sidmidru/article-db03824b64600e5b/image-243.png)

![](/news/sidmidru/article-db03824b64600e5b/image-244.png)

![](/news/sidmidru/article-db03824b64600e5b/image-245.png)

![](/news/sidmidru/article-db03824b64600e5b/image-246.png)

![](/news/sidmidru/article-db03824b64600e5b/image-247.png)

добавим пользователю роль

![](/news/sidmidru/article-db03824b64600e5b/image-248.png)

![](/news/sidmidru/article-db03824b64600e5b/image-249.png)

теперь нам нужно создатьRBACв кластере, с админ ролью можно было его не создавать так как такой rbac есть по умолчанию в k8s

**cat > dev-admins.yaml**

|  | apiVersion: rbac.authorization.k8s.io/v1kind: RoleBindingmetadata: name: dev-admin-binding namespace: devroleRef: apiGroup: rbac.authorization.k8s.io kind: ClusterRole name: adminsubjects: - kind: Group name: dev-admins # ← Должно точно совпадать с тем, что указано в Teleport (kubernetes_groups) apiGroup: rbac.authorization.k8s.io |
| --- | --- |

root@client:~#**kubectl apply -f****dev-admins.yaml**

проверим:

создам нового пользователя:

root@client:~#**adduser user2**

root@client:~#**su - user2**

получим адрес для подключения:

![](/news/sidmidru/article-db03824b64600e5b/image-250.png)

tsh login --proxy=teleport.test.local:443 --auth=local --user=user2 teleport.test.local

|  | user2@client:~$ tsh login --proxy=teleport.test.local:443 --auth=local --user=user2 teleport.test.localEnter password for Teleport user user2:Enter an OTP code from a device:> Profile URL: https://teleport.test.local:443 Logged in as: user2 Cluster: teleport.test.local Roles: access, auditor, dev Kubernetes: enabled Kubernetes groups: dev-admins Valid until: 2025-05-05 04:49:15 +0600 +06 [valid for 12h0m0s] Extensions: login-ip, permit-agent-forwarding, permit-port-forwarding, permit-pty, private-key-policy |
| --- | --- |

user2@client:~$**exportKUBECONFIG=${HOME?}/teleport-kubeconfig.yaml
**user2@client:~$**tsh kube login teleport.test.local**

получаем уведомление:

|  | user2@client:~$ tsh kube login teleport.test.localLogged into Kubernetes cluster "teleport.test.local".Your Teleport cluster runs behind a layer 7 load balancer or reverse proxy.To access the cluster, use "tsh kubectl" which is a fully featured "kubectl"command that works when the Teleport cluster is behind layer 7 load balancer orreverse proxy. To run the Kubernetes client, use: tsh kubectl versionOr, start a local proxy with "tsh proxy kube" and use the kubeconfigprovided by the local proxy with your native Kubernetes clients: tsh proxy kube -p 8443Learn more at https://goteleport.com/docs/architecture/tls-routing/#working-with-layer-7-load-balancers-or-reverse-proxies-preview |
| --- | --- |

вот результат запросов:

|  | user2@client:~$ tsh kubectl get pod -n devNAME READY STATUS RESTARTS AGEfirst-app-common-chart-6cdf48bf88-kfwc2 1/1 Running 6 (26h ago) 20dsecond-app-common-chart-6b8457fbfb-fqm8w 1/1 Running 13 (26h ago) 92duser2@client:~$ tsh kubectl get pod -n prodError from server (Forbidden): pods is forbidden: User "user2" cannot list resource "pods" in API group "" in the namespace "prod"user2@client:~$ tsh kubectl get nsError from server (Forbidden): namespaces is forbidden: User "user2" cannot list resource "namespaces" in API group "" at the cluster scope |
| --- | --- |

как видим в dev всё ок, а посмотреть в другом namespace или отобразить все ns не возможно

### []Интеграция teleport - keycloak

### []Обновление кластера k8s

ранее я использовал:

https://github.com/kubernetes-incubator/kubespray.git

сейчас он переехал:

https://github.com/kubernetes-sigs/kubespray

#### []обновление 1,24 на 1,25

чтобы обновиться нужная новая версия ansible

**cd /etc/ansible/kubespray-official/kubespray-new**

**git clone https://github.com/kubernetes-sigs/kubespray.git**

**cd /etc/ansible/kubespray-official/kubespray-new/kubespray**

**git checkout release-2.20**

**apt-get update&&apt-get install -y python3-venv python3-pip rsync**

**apt-get install -y build-essential python3-dev libyaml-dev**

**python3 -m venv .venv-ansible**

**source .venv-ansible/bin/activate**

**pip install --upgrade pip setuptools wheel**

**sed -i '/ruamel.yaml.clib/d' requirements.txt**

**pip install --upgrade pip setuptools wheel**

**pip install -r requirements.txt**

|  | ansible-galaxy collection install \ community.general \ community.kubernetes \ kubernetes.core \ amazon.aws \ azure.azcollection \ ansible.posix \ ansible.utils \ community.crypto \ ansible.netcommon |
| --- | --- |

**mkdir -p filter_plugins**

**cp ~/.ansible/collections/ansible_collections/ansible/utils/plugins/filter/ipaddr.py filter_plugins/**

**pip install netaddr**

**pip install cryptography**

**pip install jmespath**

**pip install jsonschema**

в файле

/etc/ansible/kubespray-official/kubespray-new/kubespray/roles/kubernetes/preinstall/vars/debian.yml

**меняем**python-apt на python3-apt и комментим aufs-tools

|  | ---required_pkgs: - python3-apt #- aufs-tools - apt-transport-https - software-properties-common - conntrack - apparmor - libseccomp2 |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/inventory.ini

| 12345678910111213141516171819202122232425262728293031323334353637 | [all]kub-master1.test.local ansible_host=192.168.1.112 ip=192.168.1.112kub-master2.test.local ansible_host=192.168.1.113 ip=192.168.1.113kub-master3.test.local ansible_host=192.168.1.114 ip=192.168.1.114kub-worker1.test.local ansible_host=192.168.1.115 ip=192.168.1.115kub-worker2.test.local ansible_host=192.168.1.116 ip=192.168.1.116kub-worker3.test.local ansible_host=192.168.1.117 ip=192.168.1.117# ## configure a bastion host if your nodes are not directly reachable# [bastion]# bastion ansible_host=x.x.x.x ansible_user=some_user[kube_control_plane]kub-master1.test.localkub-master2.test.localkub-master3.test.local[etcd]kub-master1.test.localkub-master2.test.localkub-master3.test.local[kube_node]kub-worker1.test.localkub-worker2.test.localkub-worker3.test.local[calico_rr][k8s_cluster:children]kube_control_planekube_nodecalico_rr |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/k8s-cluster.yml

|  | kube_oidc_auth: truekube_token_auth: truekube_proxy_strict_arp: true |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/addons.yml

|  | helm_enabled: truelocal_path_provisioner_enabled: true |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/all/all.yml

|  | upstream_dns_servers: - 8.8.8.8 - 8.8.4.4 |
| --- | --- |

и запускаем установку:

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**ansible-playbook -i inventory/sample/inventory.ini upgrade-cluster.yml -b --become-user=root --ask-pass**

проверяем:

|  | root@kub-master1:~# kubectl get nodesNAME STATUS ROLES AGE VERSIONkub-master1.test.local Ready control-plane 25h v1.24.6kub-master2.test.local Ready control-plane 25h v1.24.6kub-master3.test.local Ready control-plane 25h v1.24.6kub-worker1.test.local Ready <none> 25h v1.24.6kub-worker2.test.local Ready <none> 25h v1.24.6kub-worker3.test.local Ready <none> 25h v1.24.6 |
| --- | --- |

как видим обновились до**v1.24.6**

обновляемся дальше

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**deactivate**

root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**rm -rf .venv-ansible/**
root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**git add .**
root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**git commit -m "20"**

root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**git checkout release-2.21**

нужно сейчас так же поправить инвентори и будем обновлять дальше на**v1.25.6**

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/k8s-cluster.yml

|  | kube_oidc_auth: truekube_token_auth: truekube_proxy_strict_arp: true |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/addons.yml

|  | helm_enabled: truelocal_volume_provisioner_enabled: true |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/all/all.yml

|  | upstream_dns_servers: - 8.8.8.8 - 8.8.4.4 |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/inventory.ini

| 1234567891011121314151617181920212223242526272829303132333435 | [all]kub-master1.test.local ansible_host=192.168.1.112 ip=192.168.1.112kub-master2.test.local ansible_host=192.168.1.113 ip=192.168.1.113kub-master3.test.local ansible_host=192.168.1.114 ip=192.168.1.114kub-worker1.test.local ansible_host=192.168.1.115 ip=192.168.1.115kub-worker2.test.local ansible_host=192.168.1.116 ip=192.168.1.116kub-worker3.test.local ansible_host=192.168.1.117 ip=192.168.1.117# ## configure a bastion host if your nodes are not directly reachable# [bastion]# bastion ansible_host=x.x.x.x ansible_user=some_user[kube_control_plane]kub-master1.test.localkub-master2.test.localkub-master3.test.local[etcd]kub-master1.test.localkub-master2.test.localkub-master3.test.local[kube_node]kub-worker1.test.localkub-worker2.test.localkub-worker3.test.local[calico_rr][k8s_cluster:children]kube_control_planekube_nodecalico_rr |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/roles/kubernetes/preinstall/vars/debian.yml

|  | ---required_pkgs: - python3-apt #- aufs-tools - apt-transport-https - software-properties-common - conntrack - apparmor - libseccomp2 |
| --- | --- |

**python3 -m venv .venv-ansible**

**source .venv-ansible/bin/activate**

**pip install --upgrade pip setuptools wheel**

**sed -i '/ruamel.yaml.clib/d' requirements.txt**

**pip install --upgrade pip setuptools wheel**

**pip install -r requirements.txt**

|  | ansible-galaxy collection install \ community.general \ community.kubernetes \ kubernetes.core \ amazon.aws \ azure.azcollection \ ansible.posix \ ansible.utils \ community.crypto \ ansible.netcommon |
| --- | --- |

**mkdir -p filter_plugins**

**cp ~/.ansible/collections/ansible_collections/ansible/utils/plugins/filter/ipaddr.py filter_plugins/**

**pip install netaddr**

**pip install cryptography**

**pip install jmespath**

**pip install jsonschema**

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**ansible-playbook -i inventory/sample/inventory.ini upgrade-cluster.yml -b --become-user=root --ask-pass**

#### []обновляемся дальше до 1,26

root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**rm -rf .venv-ansible/**
root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**git add .**

root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**git commit -m "21"**

root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**python3 -m venv .venv-ansible**
root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**source .venv-ansible/bin/activate**
(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**pip install --upgrade pip setuptools wheel**

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**sed -i '/ruamel.yaml.clib/d' requirements.txt**

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**pip install -r requirements.txt**

|  | ansible-galaxy collection install \ community.general \ community.kubernetes \ kubernetes.core \ amazon.aws \ azure.azcollection \ ansible.posix \ ansible.utils \ community.crypto \ ansible.netcommon |
| --- | --- |

**mkdir -p filter_plugins**

**cp ~/.ansible/collections/ansible_collections/ansible/utils/plugins/filter/ipaddr.py filter_plugins/**

**pip install netaddr**

**pip install cryptography**

**pip install jmespath**

**pip install jsonschema**

тут правим инвентори и все файлы которые и прошлые разы правили.

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**ansible-playbook -i inventory/sample/inventory.ini upgrade-cluster.yml -b --become-user=root --ask-pass**

|  | root@kub-master1:~# kubectl get nodes NAME STATUS ROLES AGE VERSIONkub-master1.test.local Ready control-plane 2d v1.26.11kub-master2.test.local Ready control-plane 2d v1.26.11kub-master3.test.local Ready control-plane 2d v1.26.11kub-worker1.test.local Ready <none> 2d v1.26.11kub-worker2.test.local Ready <none> 2d v1.26.11kub-worker3.test.local Ready <none> 2d v1.26.11 |
| --- | --- |

#### []обновляем до 1,27

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**deactivate**
root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**rm -rf .venv-ansible/**
root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**git add .**
root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**git branch**
master
release-2.20
release-2.21
* release-2.22
root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**git commit -m "22"**
root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**git checkout release-2.23**

правим все те же файлы - заполняем inventory, но**в версии release-2.23 есть отличие,**там появился playbook для debian 12

/etc/ansible/kubespray-official/kubespray-new/kubespray/roles/kubernetes/preinstall/vars/**debian-12.yml**

поэтому можно не править файл**debian.yaml**

root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**python3 -m venv .venv-ansible**
root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**source .venv-ansible/bin/activate**
(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**pip install --upgrade pip setuptools wheel**

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**sed -i '/ruamel.yaml.clib/d' requirements.txt**

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**pip install -r requirements.txt**

|  | ansible-galaxy collection install \ community.general \ community.kubernetes \ kubernetes.core \ amazon.aws \ azure.azcollection \ ansible.posix \ ansible.utils \ community.crypto \ ansible.netcommon |
| --- | --- |

**mkdir -p filter_plugins**

**cp ~/.ansible/collections/ansible_collections/ansible/utils/plugins/filter/ipaddr.py filter_plugins/**

**pip install netaddr**

**pip install cryptography**

**pip install jmespath**

**pip install jsonschema**

запускаем upgrade

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**ansible-playbook -i inventory/sample/inventory.ini upgrade-cluster.yml -b --become-user=root --ask-pass**

|  | root@kub-master2:~# kubectl get nodesNAME STATUS ROLES AGE VERSIONkub-master1.test.local Ready control-plane 2d22h v1.27.10kub-master2.test.local Ready control-plane 2d22h v1.27.10kub-master3.test.local Ready control-plane 2d22h v1.27.10kub-worker1.test.local Ready <none> 2d22h v1.27.10kub-worker2.test.local Ready <none> 2d22h v1.27.10kub-worker3.test.local Ready <none> 2d22h v1.27.10 |
| --- | --- |

#### []обновляем до 1,28

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**deactivate**
root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**rm -rf .venv-ansible/**
root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**git add .**
root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**git commit -m "23"**
root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**git checkout release-2.24**

не забываем править:

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/inventory.ini
/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/all/all.yml
/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/addons.yml
/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/k8s-cluster.yml

root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**python3 -m venv .venv-ansible**
root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**source .venv-ansible/bin/activate**
(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**pip install --upgrade pip setuptools wheel**

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**sed -i '/ruamel.yaml.clib/d' requirements.txt**

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**pip install -r requirements.txt**

|  | ansible-galaxy collection install \ community.general \ community.kubernetes \ kubernetes.core \ amazon.aws \ azure.azcollection \ ansible.posix \ ansible.utils \ community.crypto \ ansible.netcommon |
| --- | --- |

**mkdir -p filter_plugins**

**cp ~/.ansible/collections/ansible_collections/ansible/utils/plugins/filter/ipaddr.py filter_plugins/**

**pip install netaddr**

**pip install cryptography**

**pip install jmespath**

**pip install jsonschema**

запускаем upgrade

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**ansible-playbook -i inventory/sample/inventory.ini upgrade-cluster.yml -b --become-user=root --ask-pass**

|  | root@kub-master1:~# kubectl get nodesNAME STATUS ROLES AGE VERSIONkub-master1.test.local Ready control-plane 3d6h v1.28.10kub-master2.test.local Ready control-plane 3d6h v1.28.10kub-master3.test.local Ready control-plane 3d6h v1.28.10kub-worker1.test.local Ready <none> 3d6h v1.28.10kub-worker2.test.local Ready <none> 3d6h v1.28.10kub-worker3.test.local Ready <none> 3d6h v1.28.10 |
| --- | --- |

#### []обновляем до 1,29

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**deactivate**
**rm -rf .venv-ansible/**
**git add .**
**git commit -m "24"**
**git checkout release-2.25**
**python3 -m venv .venv-ansible**
**source .venv-ansible/bin/activate**
**pip install --upgrade pip setuptools wheel**
**sed -i '/ruamel.yaml.clib/d' requirements.txt**
**pip install -r requirements.txt**

|  | ansible-galaxy collection install \ community.general \ community.kubernetes \ kubernetes.core \ amazon.aws \ azure.azcollection \ ansible.posix \ ansible.utils \ community.crypto \ ansible.netcommon |
| --- | --- |

**mkdir -p filter_plugins**

**cp ~/.ansible/collections/ansible_collections/ansible/utils/plugins/filter/ipaddr.py filter_plugins/**

**pip install netaddr**

**pip install cryptography**

**pip install jmespath**

**pip install jsonschema**

не забываем править:

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/inventory.ini
/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/all/all.yml
/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/addons.yml
/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/k8s-cluster.yml

и запускаем апгрейд

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**ansible-playbook -i inventory/sample/inventory.ini upgrade-cluster.yml -b --become-user=root --ask-pass**

|  | root@kub-master1:~# kubectl get nodesNAME STATUS ROLES AGE VERSIONkub-master1.test.local Ready control-plane 3d7h v1.29.10kub-master2.test.local Ready control-plane 3d7h v1.29.10kub-master3.test.local Ready control-plane 3d7h v1.29.10kub-worker1.test.local Ready <none> 3d7h v1.29.10kub-worker2.test.local Ready <none> 3d7h v1.29.10kub-worker3.test.local Ready <none> 3d7h v1.29.10 |
| --- | --- |

#### []обновляем до 1,30

**deactivate**
**rm -rf .venv-ansible/**
**git add .**
**git commit -m "24"**
**git checkout release-2.26**
**python3 -m venv .venv-ansible**
**source .venv-ansible/bin/activate**
**pip install --upgrade pip setuptools wheel**
**sed -i '/ruamel.yaml.clib/d' requirements.txt**
**pip install -r requirements.txt**

|  | ansible-galaxy collection install \ community.general \ community.kubernetes \ kubernetes.core \ amazon.aws \ azure.azcollection \ ansible.posix \ ansible.utils \ community.crypto \ ansible.netcommon |
| --- | --- |

**mkdir -p filter_plugins**

**cp ~/.ansible/collections/ansible_collections/ansible/utils/plugins/filter/ipaddr.py filter_plugins/**

**pip install netaddr**

**pip install cryptography**

**pip install jmespath**

**pip install jsonschema**

не забываем править:

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/inventory.ini
/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/all/all.yml
/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/addons.yml
/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/k8s-cluster.yml

и запускаем апгрейд

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**ansible-playbook -i inventory/sample/inventory.ini upgrade-cluster.yml -b --become-user=root --ask-pass**

адейт прошёл не совсем по плану:

|  | root@kub-master1:~# kubectl get nodesNAME STATUS ROLES AGE VERSIONkub-master1.test.local Ready control-plane 3d21h v1.30.4kub-master2.test.local Ready,SchedulingDisabled control-plane 3d21h v1.30.4kub-master3.test.local Ready control-plane 3d21h v1.30.4kub-worker1.test.local Ready <none> 3d21h v1.30.4kub-worker2.test.local Ready <none> 3d21h v1.30.4kub-worker3.test.local Ready <none> 3d21h v1.30.4 |
| --- | --- |

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374757677 | root@kub-master1:~# kubectl describe node kub-master2.test.localName: kub-master2.test.localRoles: control-planeLabels: beta.kubernetes.io/arch=amd64 beta.kubernetes.io/os=linux kubernetes.io/arch=amd64 kubernetes.io/hostname=kub-master2.test.local kubernetes.io/os=linux node-role.kubernetes.io/control-plane= node.kubernetes.io/exclude-from-external-load-balancers=Annotations: kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock node.alpha.kubernetes.io/ttl: 0 projectcalico.org/IPv4Address: 192.168.1.113/24 projectcalico.org/IPv4VXLANTunnelAddr: 10.233.122.192 volumes.kubernetes.io/controller-managed-attach-detach: trueCreationTimestamp: Wed, 21 May 2025 12:53:37 +0600Taints: node-role.kubernetes.io/control-plane:NoSchedule node.kubernetes.io/unschedulable:NoScheduleUnschedulable: trueLease: HolderIdentity: kub-master2.test.local AcquireTime: <unset> RenewTime: Sun, 25 May 2025 10:21:29 +0600Conditions: Type Status LastHeartbeatTime LastTransitionTime Reason Message ---- ------ ----------------- ------------------ ------ ------- NetworkUnavailable False Sat, 24 May 2025 23:21:24 +0600 Sat, 24 May 2025 23:21:24 +0600 CalicoIsUp Calico is running on this node MemoryPressure False Sun, 25 May 2025 10:21:29 +0600 Sat, 24 May 2025 21:40:58 +0600 KubeletHasSufficientMemory kubelet has sufficient memory available DiskPressure False Sun, 25 May 2025 10:21:29 +0600 Sat, 24 May 2025 21:40:58 +0600 KubeletHasNoDiskPressure kubelet has no disk pressure PIDPressure False Sun, 25 May 2025 10:21:29 +0600 Sat, 24 May 2025 21:40:58 +0600 KubeletHasSufficientPID kubelet has sufficient PID available Ready True Sun, 25 May 2025 10:21:29 +0600 Sat, 24 May 2025 23:21:04 +0600 KubeletReady kubelet is posting ready statusAddresses: InternalIP: 192.168.1.113 Hostname: kub-master2.test.localCapacity: cpu: 4 ephemeral-storage: 69549756Ki hugepages-2Mi: 0 memory: 3978024Ki pods: 110Allocatable: cpu: 3800m ephemeral-storage: 64097055024 hugepages-2Mi: 0 memory: 3351336Ki pods: 110System Info: Machine ID: 10e2ab9ef8014ab8b59603f47fdc2db7 System UUID: c68dffeb-951c-c943-a81c-0471fa140e09 Boot ID: b74446fc-ce4e-48b3-9e0d-e6f5900aee44 Kernel Version: 6.1.0-35-amd64 OS Image: Debian GNU/Linux 12 (bookworm) Operating System: linux Architecture: amd64 Container Runtime Version: containerd://1.7.23 Kubelet Version: v1.30.4 Kube-Proxy Version: v1.30.4PodCIDR: 10.233.65.0/24PodCIDRs: 10.233.65.0/24Non-terminated Pods: (6 in total) Namespace Name CPU Requests CPU Limits Memory Requests Memory Limits Age --------- ---- ------------ ---------- --------------- ------------- --- kube-system calico-node-m5kvp 150m (3%) 300m (7%) 64M (1%) 500M (14%) 12h kube-system kube-apiserver-kub-master2.test.local 250m (6%) 0 (0%) 0 (0%) 0 (0%) 12h kube-system kube-controller-manager-kub-master2.test.local 200m (5%) 0 (0%) 0 (0%) 0 (0%) 12h kube-system kube-proxy-cnjkw 0 (0%) 0 (0%) 0 (0%) 0 (0%) 12h kube-system kube-scheduler-kub-master2.test.local 100m (2%) 0 (0%) 0 (0%) 0 (0%) 12h kube-system nodelocaldns-jvcdh 100m (2%) 0 (0%) 70Mi (2%) 200Mi (6%) 14hAllocated resources: (Total limits may be over 100 percent, i.e., overcommitted.) Resource Requests Limits -------- -------- ------ cpu 800m (21%) 300m (7%) memory 137400320 (4%) 709715200 (20%) ephemeral-storage 0 (0%) 0 (0%) hugepages-2Mi 0 (0%) 0 (0%)Events: <none> |
| --- | --- |

root@kub-master1:~#**kubectl uncordon kub-master2.test.local**
node/kub-master2.test.local uncordoned

|  | root@kub-master1:~# kubectl get nodesNAME STATUS ROLES AGE VERSIONkub-master1.test.local Ready control-plane 3d21h v1.30.4kub-master2.test.local Ready control-plane 3d21h v1.30.4kub-master3.test.local Ready control-plane 3d21h v1.30.4kub-worker1.test.local Ready <none> 3d21h v1.30.4kub-worker2.test.local Ready <none> 3d21h v1.30.4kub-worker3.test.local Ready <none> 3d21h v1.30.4 |
| --- | --- |

#### []обновляем до 1,31

**deactivate**
**rm -rf .venv-ansible/**
**git add .**
**git commit -m "26"**
**git checkout release-2.27
****python3 -m venv .venv-ansible
****source .venv-ansible/bin/activate
****pip install --upgrade pip setuptools wheel
****sed -i '/ruamel.yaml.clib/d' requirements.txt
****pip install -r requirements.txt**

|  | ansible-galaxy collection install \ community.general \ community.kubernetes \ kubernetes.core \ amazon.aws \ azure.azcollection \ ansible.posix \ ansible.utils \ community.crypto \ ansible.netcommon |
| --- | --- |

**mkdir -p filter_plugins
****cp ~/.ansible/collections/ansible_collections/ansible/utils/plugins/filter/ipaddr.py filter_plugins/**

**pip install netaddr**

**pip install cryptography**

**pip install jmespath**

**pip install jsonschema
**

не забываем править:

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/inventory.ini
/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/all/all.yml
/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/addons.yml
/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/k8s-cluster.yml

и запускаем апгрейд

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**ansible-playbook -i inventory/sample/inventory.ini upgrade-cluster.yml -b --become-user=root --ask-pass**

|  | root@kub-master1:~# kubectl get nodesNAME STATUS ROLES AGE VERSIONkub-master1.test.local Ready control-plane 4d1h v1.31.7kub-master2.test.local Ready control-plane 4d1h v1.31.7kub-master3.test.local Ready control-plane 4d1h v1.31.7kub-worker1.test.local Ready <none> 4d1h v1.31.7kub-worker2.test.local Ready <none> 4d1h v1.31.7kub-worker3.test.local Ready <none> 4d1h v1.31.7 |
| --- | --- |

#### []обновляем до 1,32

**deactivate**
**rm -rf .venv-ansible/**
**git add .**
**git commit -m "27"**
**git checkout release-2.28
****python3 -m venv .venv-ansible
****source .venv-ansible/bin/activate
****pip install --upgrade pip setuptools wheel
****sed -i '/ruamel.yaml.clib/d' requirements.txt
****pip install -r requirements.txt**

|  | ansible-galaxy collection install \ community.general \ community.kubernetes \ kubernetes.core \ amazon.aws \ azure.azcollection \ ansible.posix \ ansible.utils \ community.crypto \ ansible.netcommon |
| --- | --- |

**mkdir -p filter_plugins
****cp ~/.ansible/collections/ansible_collections/ansible/utils/plugins/filter/ipaddr.py filter_plugins/**

**pip install netaddr**

**pip install cryptography**

**pip install jmespath**

**pip install jsonschema
**

не забываем править:

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/inventory.ini

| 1234567891011121314151617181920212223242526272829303132333435 | [all]kub-master1.test.local ansible_host=192.168.1.112 ip=192.168.1.112kub-master2.test.local ansible_host=192.168.1.113 ip=192.168.1.113kub-master3.test.local ansible_host=192.168.1.114 ip=192.168.1.114kub-worker1.test.local ansible_host=192.168.1.115 ip=192.168.1.115kub-worker2.test.local ansible_host=192.168.1.116 ip=192.168.1.116kub-worker3.test.local ansible_host=192.168.1.117 ip=192.168.1.117# ## configure a bastion host if your nodes are not directly reachable# [bastion]# bastion ansible_host=x.x.x.x ansible_user=some_user[kube_control_plane]kub-master1.test.localkub-master2.test.localkub-master3.test.local[etcd]kub-master1.test.localkub-master2.test.localkub-master3.test.local[kube_node]kub-worker1.test.localkub-worker2.test.localkub-worker3.test.local[calico_rr][k8s_cluster:children]kube_control_planekube_nodecalico_rr |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/all/all.yml

|  | upstream_dns_servers: - 8.8.8.8 - 8.8.4.4 |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/addons.yml

|  | helm_enabled: truelocal_volume_provisioner_enabled: true |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/k8s-cluster.yml

|  | kube_oidc_auth: truekube_token_auth: truekube_proxy_strict_arp: true |
| --- | --- |

и запускаем апгрейд

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**ansible-playbook -i inventory/sample/inventory.ini upgrade-cluster.yml -b --become-user=root --ask-pass**

|  | root@kub-master1:~# kubectl get nodesNAME STATUS ROLES AGE VERSIONkub-master1.test.local Ready control-plane 4d3h v1.32.5kub-master2.test.local Ready control-plane 4d3h v1.32.5kub-master3.test.local Ready control-plane 4d3h v1.32.5kub-worker1.test.local Ready <none> 4d3h v1.32.5kub-worker2.test.local Ready <none> 4d3h v1.32.5kub-worker3.test.local Ready <none> 4d3h v1.32.5 |
| --- | --- |

#### []обновляем до 1.32.9

пока писал всю статью прилетели ещё обновления поэтому сейчас нужно обновиться с 1.32.5 до 1.32.9

поэтому выполняем все действия что и на предыдущем шаге и запускаем апдейт

я выкачивал
https://github.com/kubernetes-sigs/kubespray/tree/release-2.28
в директорию:

/etc/ansible/kubespray-official/kub-new

потом переименую и верну как было

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kub-new/kubespray/kubespray#**ansible-playbook -i inventory/sample/inventory.ini upgrade-cluster.yml -b --become-user=root --ask-pass**

ждём и проверяем:

|  | root@kub-master1:~# kubectl get nodesNAME STATUS ROLES AGE VERSIONkub-master1.test.local Ready control-plane 166d v1.32.9kub-master2.test.local Ready control-plane 166d v1.32.9kub-master3.test.local Ready control-plane 166d v1.32.9kub-worker1.test.local Ready <none> 166d v1.32.9kub-worker2.test.local Ready <none> 166d v1.32.9kub-worker3.test.local Ready <none> 166d v1.32.9 |
| --- | --- |

готово.
обновляемся дальше:

#### []обновляем до 1.33.5 (installPLUTO)

перед обновлением поставим ещё утилиту pluto чтобы проверить есть проблемы с совместимостью api

https://github.com/FairwindsOps/pluto/releases

на момент написания статьи последняя версия

https://github.com/FairwindsOps/pluto/releases/download/v5.22.6/pluto_5.22.6_linux_amd64.tar.gz

качаем её
**wget https://github.com/FairwindsOps/pluto/releases/download/v5.22.6/pluto_5.22.6_linux_amd64.tar.gz**

root@kub-master1:~#**tar -xvf pluto_5.22.6_linux_amd64.tar.gz**

root@kub-master1:~#**chmod +x pluto**

root@kub-master1:~#**mv ./pluto /usr/bin/**

можно проверить как api в кластере:

|  | root@kub-master1:~# pluto detect-helm -owideThere were no resources found with known deprecated apiVersions. |
| --- | --- |

так и все файлы:

|  | root@kub-master1:~# pluto detect-files -d .NAME KIND VERSION REPLACEMENT REMOVED DEPRECATED REPL AVAIL nginx Deployment apps/v1beta1 apps/v1 true true true nginx Deployment apps/v1beta1 apps/v1 true true true |
| --- | --- |

как видим найдена старая версия api apps/v1beta1 и её нужно заменить на apps/v1, других проблем у меня не найдено поэтому продолжим обновление:

**deactivate**
**rm -rf .venv-ansible/**
**git add .**
**git commit -m "28"**
**git checkout release-2.29
****python3 -m venv .venv-ansible
****source .venv-ansible/bin/activate
****pip install --upgrade pip setuptools wheel
****sed -i '/ruamel.yaml.clib/d' requirements.txt
****pip install -r requirements.txt**

|  | ansible-galaxy collection install \ community.general \ community.kubernetes \ kubernetes.core \ amazon.aws \ azure.azcollection \ ansible.posix \ ansible.utils \ community.crypto \ ansible.netcommon |
| --- | --- |

**mkdir -p filter_plugins
****cp ~/.ansible/collections/ansible_collections/ansible/utils/plugins/filter/ipaddr.py filter_plugins/**

**pip install netaddr**

**pip install cryptography**

**pip install jmespath**

**pip install jsonschema
**

не забываем править:

/etc/ansible/kubespray-official/kub-new/kubespray/kubespray/inventory/sample/inventory.ini

потом я эту директорию переделаю в:

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/inventory.ini

| 1234567891011121314151617181920212223242526272829303132333435 | [all]kub-master1.test.local ansible_host=192.168.1.112 ip=192.168.1.112kub-master2.test.local ansible_host=192.168.1.113 ip=192.168.1.113kub-master3.test.local ansible_host=192.168.1.114 ip=192.168.1.114kub-worker1.test.local ansible_host=192.168.1.115 ip=192.168.1.115kub-worker2.test.local ansible_host=192.168.1.116 ip=192.168.1.116kub-worker3.test.local ansible_host=192.168.1.117 ip=192.168.1.117# ## configure a bastion host if your nodes are not directly reachable# [bastion]# bastion ansible_host=x.x.x.x ansible_user=some_user[kube_control_plane]kub-master1.test.localkub-master2.test.localkub-master3.test.local[etcd]kub-master1.test.localkub-master2.test.localkub-master3.test.local[kube_node]kub-worker1.test.localkub-worker2.test.localkub-worker3.test.local[calico_rr][k8s_cluster:children]kube_control_planekube_nodecalico_rr |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/all/all.yml

|  | upstream_dns_servers: - 8.8.8.8 - 8.8.4.4 |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/addons.yml

|  | helm_enabled: truelocal_volume_provisioner_enabled: true |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/k8s-cluster.yml

|  | kube_oidc_auth: truekube_token_auth: truekube_proxy_strict_arp: true |
| --- | --- |

и запускаем апгрейд

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**ansible-playbook -i inventory/sample/inventory.ini upgrade-cluster.yml -b --become-user=root --ask-pass**

Проверяем:

|  | root@kub-master1:~# kubectl get nodes NAME STATUS ROLES AGE VERSIONkub-master1.test.local Ready control-plane 166d v1.33.5kub-master2.test.local Ready control-plane 166d v1.33.5kub-master3.test.local Ready control-plane 166d v1.33.5kub-worker1.test.local Ready <none> 166d v1.33.5kub-worker2.test.local Ready <none> 166d v1.33.5kub-worker3.test.local Ready <none> 166d v1.33.5 |
| --- | --- |

#### []обновляем до 1.34.1

**deactivate**
**rm -rf .venv-ansible/**
**git add .**
**git commit -m "29"**

так как ещё нет последнего апдейта минорного и новой версии то переключаемся на мастера
**git checkout master
****python3 -m venv .venv-ansible
****source .venv-ansible/bin/activate
****pip install --upgrade pip setuptools wheel
****sed -i '/ruamel.yaml.clib/d' requirements.txt
****pip install -r requirements.txt**

|  | ansible-galaxy collection install \ community.general \ community.kubernetes \ kubernetes.core \ amazon.aws \ azure.azcollection \ ansible.posix \ ansible.utils \ community.crypto \ ansible.netcommon |
| --- | --- |

**mkdir -p filter_plugins
****cp ~/.ansible/collections/ansible_collections/ansible/utils/plugins/filter/ipaddr.py filter_plugins/**

**pip install netaddr**

**pip install cryptography**

**pip install jmespath**

**pip install jsonschema
**

не забываем править:

/etc/ansible/kubespray-official/kub-new/kubespray/kubespray/inventory/sample/inventory.ini

потом я эту директорию переделаю в:

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/inventory.ini

| 1234567891011121314151617181920212223242526272829303132333435 | [all]kub-master1.test.local ansible_host=192.168.1.112 ip=192.168.1.112kub-master2.test.local ansible_host=192.168.1.113 ip=192.168.1.113kub-master3.test.local ansible_host=192.168.1.114 ip=192.168.1.114kub-worker1.test.local ansible_host=192.168.1.115 ip=192.168.1.115kub-worker2.test.local ansible_host=192.168.1.116 ip=192.168.1.116kub-worker3.test.local ansible_host=192.168.1.117 ip=192.168.1.117# ## configure a bastion host if your nodes are not directly reachable# [bastion]# bastion ansible_host=x.x.x.x ansible_user=some_user[kube_control_plane]kub-master1.test.localkub-master2.test.localkub-master3.test.local[etcd]kub-master1.test.localkub-master2.test.localkub-master3.test.local[kube_node]kub-worker1.test.localkub-worker2.test.localkub-worker3.test.local[calico_rr][k8s_cluster:children]kube_control_planekube_nodecalico_rr |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/all/all.yml

|  | upstream_dns_servers: - 8.8.8.8 - 8.8.4.4 |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/addons.yml

|  | helm_enabled: truelocal_volume_provisioner_enabled: true |
| --- | --- |

/etc/ansible/kubespray-official/kubespray-new/kubespray/inventory/sample/group_vars/k8s_cluster/k8s-cluster.yml

|  | kube_oidc_auth: truekube_token_auth: truekube_proxy_strict_arp: true |
| --- | --- |

и запускаем апгрейд

(.venv-ansible) root@ansible:/etc/ansible/kubespray-official/kubespray-new/kubespray#**ansible-playbook -i inventory/sample/inventory.ini upgrade-cluster.yml -b --become-user=root --ask-pass**

Проверяем:

|  | root@kub-master1:~# kubectl get nodesNAME STATUS ROLES AGE VERSIONkub-master1.test.local Ready control-plane 166d v1.34.1kub-master2.test.local Ready control-plane 166d v1.34.1kub-master3.test.local Ready control-plane 166d v1.34.1kub-worker1.test.local Ready <none> 166d v1.34.1kub-worker2.test.local Ready <none> 166d v1.34.1kub-worker3.test.local Ready <none> 166d v1.34.1 |
| --- | --- |

### []Gitlab in k8s

берём оф чарт:

https://gitlab.com/gitlab-org/charts/gitlab/-/tree/master/charts/gitlab

используем самоподписанный сертификат:

/etc/ansible/kubespray-official/gitlab-gitlab-runner/certs/ca_openssl.cnf

|  | [ v3_ca ]subjectAltName = @alt_names[ alt_names ]DNS.1 = gitlab.test.localDNS.2 = kas.test.local |
| --- | --- |

/etc/ansible/kubespray-official/gitlab-gitlab-runner/certs/gitlab_openssl.cnf

| 123456789101112131415161718192021 | [ req ]default_bits = 4096prompt = nodefault_md = sha256distinguished_name = dnreq_extensions = req_ext[ dn ]C = RUST = YourStateL = YourCityO = YourOrganizationCN = gitlab.test.local[ req_ext ]subjectAltName = @alt_names[ alt_names ]DNS.1 = gitlab.test.localDNS.2 = kas.test.local |
| --- | --- |

создаём сертификат

|  | openssl genrsa -out ca.key 4096openssl req -x509 -sha256 -new -key ca.key -days 10000 -out ca.crtopenssl genrsa -out gitlab.key 4096openssl req -new -key gitlab.key -out gitlab.csr -config gitlab_openssl.cnfopenssl x509 -req -in gitlab.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out gitlab.crt -days 10000 -sha256 -extfile ca_openssl.cnf -extensions v3_ca |
| --- | --- |

создаём namespace

создаём секрет с нашим сертификатом:

|  | kubectl create secret generic gitlab-tls \ --from-file=tls.crt=./gitlab.crt \ --from-file=tls.key=./gitlab.key \ --from-file=ca.crt=./ca.crt \ -n gitlab |
| --- | --- |

созданный сертификат раскидываем по всем тачкам:

|  | for node in $(kubectl get nodes -o wide | awk 'NR>1{print $6}' | grep -v INTERNAL); do scp gitlab.crt root@$node:/usr/local/share/ca-certificates/gitlab.crt \ && ssh root@$node update-ca-certificatesdone |
| --- | --- |

будем использовать встроенную базу данных, можно использовать и внешнюю базу для этого нужно раскомментировать и заполнить psql часть, а postgresql выключить ( это в values)

нам нужно несколько s3 бакетов (встроенный minio не будет использовать)

![](/news/sidmidru/article-db03824b64600e5b/image-251.png)

делаем для них policy

![](/news/sidmidru/article-db03824b64600e5b/image-252.png)

| 12345678910111213141516171819202122232425262728293031323334 | { "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Action": [ "s3:ListBucket", "s3:GetBucketLocation" ], "Resource": [ "arn:aws:s3:::gitlab-lfs", "arn:aws:s3:::gitlab-packages", "arn:aws:s3:::gilab-uploads", "arn:aws:s3:::gitlab-artifacts", "arn:aws:s3:::gitlab-backups" ] }, { "Effect": "Allow", "Action": [ "s3:GetObject", "s3:PutObject", "s3:DeleteObject" ], "Resource": [ "arn:aws:s3:::gilab-uploads/*", "arn:aws:s3:::gitlab-artifacts/*", "arn:aws:s3:::gitlab-backups/*", "arn:aws:s3:::gitlab-lfs/*", "arn:aws:s3:::gitlab-packages/*" ] } ]} |
| --- | --- |

создаём пользователя

и асайним на него нашу policy

![](/news/sidmidru/article-db03824b64600e5b/image-253.png)

создаём ключи для него:

![](/news/sidmidru/article-db03824b64600e5b/image-254.png)

создаём секрет с доступами к бакету, можно делать отдельные секреты но я решил использовать 1 на все

/etc/ansible/kubespray-official/gitlab-gitlab-runner/s3-conf.yml

|  | provider: AWSaws_access_key_id: "biZiP3COrMLYTzflHWH6"aws_secret_access_key: "LejhJ03w8wqsMJrDf8qeGgcHCPs3B8ml9s73GYg4"region: us-east-1 # обязательный, даже если MinIO, любое значение допустимоendpoint: "http://192.168.1.120:9000"path_style: true |
| --- | --- |

**kubectl -n gitlab create secret generic gitlab-s3-config --from-file=s3.yml=./s3-conf.yml**

ок вот наш values

/etc/ansible/kubespray-official/gitlab-gitlab-runner/gitlab.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210211212213214215216217218219220221222223224225226227228229230231232233234235236237238239240241242243244245246247248249250251252253254255256257258259260261262263264265266267268269270271272273274275276277278279280281282283284285286287288289290291292293294295296297298299300301302303304305306307 | nginx-ingress: enabled: falseglobal: hosts: # Домен, по которому будет доступен GitLab domain: "test.local" ingress: enabled: true configureCertmanager: false provider: nginx class: "nginx" annotations: nginx.ingress.kubernetes.io/force-ssl-redirect: "true" tls: secretName: "gitlab-tls" hosts: - gitlab.test.local # psql: # main: # # Параметры подключения к внешней базе данных для основной части GitLab # host: "${var.rds_address[local.gitlab.rds_infra_name]}" # port: 5432 # username: "gitlab" # password: # secret: "${kubernetes_secret.gitlab_db_secret.metadata[0].name}" # key: "${keys(kubernetes_secret.gitlab_db_secret.data)[0]}" # database: "gitlab" # ci: # # Параметры подключения для базы данных CI # host: "${var.rds_address[local.gitlab.rds_infra_name]}" # port: 5432 # username: "gitlab" # password: # secret: "${kubernetes_secret.gitlab_db_secret.metadata[0].name}" # key: "${keys(kubernetes_secret.gitlab_db_secret.data)[0]}" # database: "gitlab" appConfig: object_store: enabled: true connection: secret: "gitlab-s3-config" key: "s3.yml" minio: enabled: false serviceAccount: enabled: true email: display_name: 'GitLab' from: "gitlab@test.local" reply_to: "noreply@test.local" smtp: enabled: false address: "" port: 587 domain: "gitlab@test.local" user_name: "" password: secret: "" # имя секрета, где хранится пароль key: "" #password authentication: "login" # обычно используется "login" или "plain" starttls_auto: true registry: enabled: false shell: port: 2222 # так же открываем порт на ingress controller tcp: 2222: "gitlab/gitlab-gitlab-shell:2222:PROXY" tcp: proxyProtocol: false # appConfig: # lfs: # bucket: "gitlab-lfs" # path: "lfs" # connection: # secret: "biZiP3COrMLYTzflHWH6" # key: "LejhJ03w8wqsMJrDf8qeGgcHCPs3B8ml9s73GYg4" # artifacts: # bucket: "gitlab-artifacts" # path: "artifacts" # connection: # secret: "biZiP3COrMLYTzflHWH6" # key: "LejhJ03w8wqsMJrDf8qeGgcHCPs3B8ml9s73GYg4" # uploads: # bucket: "gilab-uploads" # path: "uploads" # connection: # secret: "biZiP3COrMLYTzflHWH6" # key: "LejhJ03w8wqsMJrDf8qeGgcHCPs3B8ml9s73GYg4" # packages: # bucket: "gitlab-packages" # path: "packages" # connection: # secret: "biZiP3COrMLYTzflHWH6" # key: "LejhJ03w8wqsMJrDf8qeGgcHCPs3B8ml9s73GYg4" # backups: # bucket: "gitlab-backups" # path: "backups" # tmpBucket: "gitlab-backups" # tmpPath: "tmp" # гугловая авторизация # omniauth: # enabled: true # allowSingleSignOn: ['google_oauth2'] # syncProfileAttributes: ['email'] # autoLinkSamlUser: true # blockAutoCreatedUsers: false # autoLinkUser: ['google_oauth2'] # providers: # - secret: "${kubernetes_secret.gitlab_secret.metadata[0].name}" # key: "${keys(kubernetes_secret.gitlab_secret.data)[0]}"redis: install: true master: resources: requests: cpu: "100m" memory: "100Mi" limits: cpu: "500m" memory: "300Mi" persistence: enabled: true storageClass: nfs-client size: 5Gi# Отключаем certmanagercertmanager: installCRDs: false install: false# ОтклюВключаем чаем установку встроенного PostgreSQLpostgresql: install: true primary: resources: requests: cpu: "300m" memory: "300Mi" limits: cpu: "1000m" memory: "1Gi" persistence: enabled: true storageClass: nfs-client size: 10Gi# Registry – хранение Docker-образов (если используется)registry: enabled: false# Prometheus – мониторинг GitLab (если устанавливается вместе с GitLab)prometheus: install: false# Основные компоненты GitLab (Rails/Sidekiq/Webservice)gitlab: webservice: hpa: enabled: false replicaCount: 1 minReplicas: 1 maxReplicas: 1 ingress: proxyBodySize: "5000m" proxyConnectTimeout: "8000" proxyReadTimeout: "8000" resources: requests: cpu: "500m" memory: "1500Mi" limits: cpu: "1000m" memory: "3Gi" sidekiq: hpa: enabled: true cpu: targetType: Value targetAverageValue: 400m replicaCount: 1 minReplicas: 1 maxReplicas: "40" concurrency: "20" resources: requests: cpu: "400m" memory: "1Gi" limits: cpu: "1500m" memory: "2Gi" toolbox: resources: requests: cpu: "100m" memory: "200Mi" limits: cpu: "500m" memory: "1Gi" # backups: # cron: # enabled: "false" # schedule: "@daily" # resources: # requests: # cpu: "200m" # memory: "200Mi" # limits: # cpu: "1000m" # memory: "2Gi" # persistence: # enabled: "true" # accessMode: ReadWriteOnce # useGenericEphemeralVolume: false # storageClass: "nfs-client" # size: 40Gi # objectStorage: # config: # secret: "biZiP3COrMLYTzflHWH6" # key: "LejhJ03w8wqsMJrDf8qeGgcHCPs3B8ml9s73GYg4" # Gitaly – хранит Git-репозитории GitLab gitaly: resources: requests: cpu: "300m" memory: "300Mi" limits: cpu: "1000m" memory: "1Gi" persistence: enabled: true storageClass: nfs-client size: 20Gi gitlab-shell: hpa: enabled: false replicaCount: 1 minReplicas: 1 maxReplicas: 1 service: enabled: true type: ClusterIP port: 2222 resources: requests: cpu: "100m" memory: "100Mi" limits: cpu: "500m" memory: "300Mi" kas: minReplicas: 1 maxReplicas: 1 resources: requests: cpu: "100m" memory: "100Mi" limits: cpu: "200m" memory: "200Mi" gitlab-exporter: resources: requests: cpu: "100m" memory: "100Mi" limits: cpu: "500m" memory: "200Mi" appConfig: lfs: bucket: "gitlab-lfs" path: "lfs" connection: secret: "gitlab-s3-config" key: "s3.yml" artifacts: bucket: "gitlab-artifacts" path: "artifacts" connection: secret: "gitlab-s3-config" key: "s3.yml" uploads: bucket: "gitlab-uploads" path: "uploads" connection: secret: "gitlab-s3-config" key: "s3.yml" packages: bucket: "gitlab-packages" path: "packages" connection: secret: "gitlab-s3-config" key: "s3.yml" backups: bucket: "gitlab-backups" path: "backups" tmpBucket: "gitlab-backups" tmpPath: "tmp"gitlab-runner: install: false |
| --- | --- |

ставим:

**helm repo add gitlab https://charts.gitlab.io/**

**helm repo update**

**helm upgrade --install gitlab gitlab/gitlab -n gitlab --version 8.9.2 --values gitlab.yaml**

ждём пока всё установится,

|  | root@kub-master1:~# kubectl get pod -n gitlab NAME READY STATUS RESTARTS AGEgitlab-gitaly-0 1/1 Running 0 18hgitlab-gitlab-exporter-6f6d6598d-xbhgk 1/1 Running 0 18hgitlab-gitlab-shell-b985b95ff-j4xz2 1/1 Running 0 18hgitlab-kas-95b98b8b6-bs79r 1/1 Running 3 (18h ago) 18hgitlab-postgresql-0 2/2 Running 3 (5h36m ago) 18hgitlab-redis-master-0 2/2 Running 6 (5h38m ago) 18hgitlab-sidekiq-all-in-1-v2-8448db474-6ltn9 1/1 Running 0 18hgitlab-toolbox-56d756dc6c-fssp9 1/1 Running 0 18hgitlab-webservice-default-54b9669ddd-w84j2 2/2 Running 7 (5h36m ago) 18h |
| --- | --- |

и смотрим пароль от админки:

root@kub-master1:~#**kubectl get secrets -n gitlab gitlab-gitlab-initial-root-password -o yaml | grep password: | awk '{print $2}' | base64 -d**

чтобы работал git clone так как мы указали порт 2222 то добавим в ingress controller

**/etc/ansible/kubespray-official/ingress-controller/values.yaml**

|  | tcp: 2222: "gitlab/gitlab-gitlab-shell:2222" |
| --- | --- |

весь values для ingress выглядит так:

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106 | controller: enabled: true kind: Deployment # DaemonSet replicaCount: 2 # CPU/memory limits for the controller pods resources: limits: cpu: 1000m memory: 1Gi requests: cpu: 100m memory: 256Mi # CRD for the nginx ingress class ingressClassResource: default: false enabled: true name: nginx controllerValue: k8s.io/ingress-nginx # Enable metrics endpoint metrics: enabled: true port: 10254 service: enabled: true annotations: prometheus.io/scrape: "true" prometheus.io/port: "10254" extraArgs: "enable-ssl-passthrough": "true"# enable-ssl-passthrough: true# passthrough:# enabled: true # All valid ConfigMap keys must be in kebab-case config: disable-ipv6: "true" disable-ipv6-dns: "true" enable-access-log-for-default-backend: "false" http2-max-field-size: "8k" large-client-header-buffers: "16 64k" limit-conn-status-code: "429" limit-req-status-code: "429" load-balance: ewma keepalive: "65" client-max-body-size: "300m" proxy-body-size: "300m" error-log-level: error log-format-escape-json: "true" log-format-upstream: >- {"bytes_sent":"$bytes_sent","vhost":"$host"," request_proto":"$server_protocol","remote_addr":"$remote_addr", "proxy_add_x_forwarded_for":"$proxy_add_x_forwarded_for","remote_user":"$remote_user", "time_local":"$time_local","request_method":"$request_method", "request_uri":"$uri","request_args":"$args","request":"$request", "status":"$status","body_bytes_sent":"$body_bytes_sent", "http_referer":"$http_referer","http_user_agent":"$http_user_agent", "request_length":"$request_length","request_time":"$request_time", "upstream_addr":"$upstream_addr","upstream_response_length":"$upstream_response_length", "upstream_response_time":"$upstream_response_time","upstream_status":"$upstream_status", "X-Business-Error":"$upstream_http_x_business_error","upstream_header_time":"$upstream_header_time", "upstream_connect_time":"$upstream_connect_time","connections_waiting":"$connections_waiting", "connections_active":"$connections_active"} map-hash-bucket-size: "128" server-tokens: "false" ssl-protocols: "TLSv1.2 TLSv1.3" ssl-session-cache: "true" ssl-session-cache-size: "20m" ssl-session-timeout: "30m" use-forwarded-headers: "true" use-gzip: "true" use-proxy-protocol: "false" worker-cpu-affinity: auto worker-processes: "2" allow-snippet-annotations: "true" annotations-risk-level: Critical ingressClass: nginx # Service definition for the controller service: type: LoadBalancer externalTrafficPolicy: Cluster # Turn on the admission webhook deployment admissionWebhooks: enabled: true tolerations: - key: "node-role.kubernetes.io/master" operator: "Exists" effect: "NoSchedule" - key: "node-role.kubernetes.io/control-plane" operator: "Exists" effect: "NoSchedule"defaultBackend: enabled: falsetcp: 2222: "gitlab/gitlab-gitlab-shell:2222" |
| --- | --- |

обновляем новую конфигурацию values

**helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \**

**--namespace ingress-nginx --create-namespace \**

**--version 4.12.1 \**

**-f values.yaml**

проверяем что порт на ingress добавился:

|  | kubectl get svc -n ingress-nginx NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGEingress-nginx-controller LoadBalancer 10.233.9.103 192.168.1.191 80:31891/TCP,443:30190/TCP,2222:31213/TCP 3d5hingress-nginx-controller-admission ClusterIP 10.233.50.207 <none> 443/TCP 3d5hingress-nginx-controller-metrics ClusterIP 10.233.40.60 <none> 10254/TCP 3d5h |
| --- | --- |

всё, дальше можно добавлять наш публичный ssh ключ и работать с репозиториями:

git clone ssh://git@gitlab.test.local:2222/test/test-project.git

### []Gitlab runner

создаём секрет с нашим самоподписанным сертификатом:

root@kub-master1:~/gitlab-gitlab-runner/certs#**kubectl create secret generic gitlab-ssl --namespace gitlab --from-file=gitlab.test.local.crt=./gitlab.crt**

при установке helm chart gitlab я выключил установку gitlab-runner, буду ставить его отдельно, нормально работает как с self hosted так и с облачным gitlab

создаём token в панели gitlab

![](/news/sidmidru/article-db03824b64600e5b/image-255.png)

![](/news/sidmidru/article-db03824b64600e5b/image-256.png)

![](/news/sidmidru/article-db03824b64600e5b/image-257.png)

получаем токен:

glrt-t3_sFjXrYwuz-xDWPx_A-5a

вот values:

/etc/ansible/kubespray-official/gitlab-gitlab-runner/gitlab-runner.yaml

| 12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273747576 | imagePullPolicy: "Always"gitlabUrl: "https://gitlab.test.local"runnerRegistrationToken: "glrt-t3_sFjXrYwuz-xDWPx_A-5a" # токен регистрации gitlab runner создаётся вручную в gitlabconcurrent: "20"checkInterval: "1"terminationGracePeriodSeconds: "3600"certsSecretName: "gitlab-ssl" # тут сапольный сертификат для gitlabrbac: create: "true" rules: - resources: ["configmaps", "pods", "pods/attach", "secrets", "services"] verbs: ["get", "list", "watch", "create", "patch", "update", "delete"] - apiGroups: [""] resources: ["pods/exec"] verbs: ["create", "patch", "delete"] serviceAccount: name: runner-gitlab-runnermetrics: enabled: "true" serviceMonitor: enabled: true interval: 15s labels: release: kube-prometheus-stackservice: enabled: true#nodeSelector:serviceAccount: create: true name: runner-gitlab-runnerrunners: serviceAccountName: runner-gitlab-runner config: | [[runners]] [runners.kubernetes] namespace = "{{.Release.Namespace}}" image = "ubuntu:22.04" # build container cpu = "300m" memory = "300Mi" cpu_limit = "500m" memory_limit = "500Mi" cpu_limit_overwrite_max_allowed = "500m" memory_limit_overwrite_max_allowed = "500Mi" # service containers service_cpu_limit = "100m" service_memory_limit = "100Mi" service_cpu_limit_overwrite_max_allowed = "100m" service_memory_limit_overwrite_max_allowed = "100Mi" # helper container helper_cpu_limit = "100m" helper_memory_limit = "100Mi" helper_cpu_limit_overwrite_max_allowed = "100m" helper_memory_limit_overwrite_max_allowed = "100Mi" service_account = "runner-gitlab-runner" image: "ubuntu:22.04" imagePullPolicy: "always" privileged: true tags: "test-runner" runUntagged: trueresources: requests: cpu: "200m" memory: "150Mi" ephemeral-storage: "300Mi" limits: cpu: "400m" memory: "350Mi" ephemeral-storage: "600Mi" |
| --- | --- |

root@kub-master1:~/gitlab-gitlab-runner#**helm upgrade --install gitlab-runner gitlab/gitlab-runner -n gitlab --version 0.74.1 --values gitlab-runner.yaml**

проверяем:

![](/news/sidmidru/article-db03824b64600e5b/image-258.png)

как видим всё ок.

создаём файл:

.gitlab-ci.yml

| 123456789101112131415161718192021222324252627282930313233 | workflow: rules: - if: '$CI_COMMIT_BRANCH == "main"' when: always - when: neverstages: - create - printcreate_file: stage: create image: alpine:latest tags: - test-runner script: - echo "hello world" > hello.txt artifacts: paths: - hello.txt expire_in: 1 hourprint_file: stage: print image: alpine:latest tags: - test-runner dependencies: - create_file script: - echo "Содержимое hello.txt:" - cat hello.txt |
| --- | --- |

2 этапа и создаётся артифакт.

![](/news/sidmidru/article-db03824b64600e5b/image-259.png)

проверяем что всё ок:

![](/news/sidmidru/article-db03824b64600e5b/image-260.png)

проверяем что артефакт создался

![](/news/sidmidru/article-db03824b64600e5b/image-261.png)

![](/news/sidmidru/article-db03824b64600e5b/image-262.png)

![](/news/sidmidru/article-db03824b64600e5b/image-263.png)

теперь проверим что данный файл добавился в s3-minio

![](/news/sidmidru/article-db03824b64600e5b/image-264.png)

как видим всё ок

### []Gitlab helm chart с Freeipa

для интеграции с freeipa используем ранее созданный sysaccount

uid=gitlab,cn=sysaccounts,cn=etc,dc=test,dc=local

|  | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=SET!5.) ssl=falseActions (ready): add | rm | ls | info | passwd | save--- Results ---dn: uid=sudo,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=nexus,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=gitlab,cn=sysaccounts,cn=etc,dc=test,dc=local--- End Results --- |
| --- | --- |

создаём секрет с паролем:

/etc/ansible/kubespray-official/gitlab-gitlab-runner/gitlab-ldap-secret.yaml

|  | apiVersion: v1kind: Secretmetadata: name: gitlab-ldap-secret namespace: gitlabtype: OpaquestringData: # bindPW — здесь указываете ваш пароль ldap.bindPW: "Secret123" |
| --- | --- |

kubectl apply -f gitlab-ldap-secret.yaml

вот наш values:

/etc/ansible/kubespray-official/gitlab-gitlab-runner/gitlab-ldap.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210211212213214215216217218219220221222223224225226227228229230231232233234235236237238239240241242243244245246247248249250251252253254255256257258259260261262263264265266267268269270271272273274275276277278279280281282283284285286287288289290291292293294295296297298299300301302303304305306307308309310311312313314315316317318319320321322323324325326327328329330331332 | nginx-ingress: enabled: falseglobal: hosts: # Домен, по которому будет доступен GitLab domain: "test.local" ingress: enabled: true configureCertmanager: false provider: nginx class: "nginx" annotations: nginx.ingress.kubernetes.io/force-ssl-redirect: "true" tls: secretName: "gitlab-tls" hosts: - gitlab.test.local # psql: # main: # # Параметры подключения к внешней базе данных для основной части GitLab # host: "${var.rds_address[local.gitlab.rds_infra_name]}" # port: 5432 # username: "gitlab" # password: # secret: "${kubernetes_secret.gitlab_db_secret.metadata[0].name}" # key: "${keys(kubernetes_secret.gitlab_db_secret.data)[0]}" # database: "gitlab" # ci: # # Параметры подключения для базы данных CI # host: "${var.rds_address[local.gitlab.rds_infra_name]}" # port: 5432 # username: "gitlab" # password: # secret: "${kubernetes_secret.gitlab_db_secret.metadata[0].name}" # key: "${keys(kubernetes_secret.gitlab_db_secret.data)[0]}" # database: "gitlab" appConfig: object_store: enabled: true connection: secret: "gitlab-s3-config" key: "s3.yml" ldap: enabled: true servers: main: label: "FreeIPA LDAP" host: "192.168.1.100" port: 389 uid: "uid" # в FreeIPA имя юзера в атрибуте uid bind_dn: "uid=gitlab,cn=sysaccounts,cn=etc,dc=test,dc=local" password: secret: gitlab-ldap-secret key: ldap.bindPW # Шифрование: plain – без TLS; start_tls – через StartTLS; simple_tls – LDAPS encryption: "plain" verify_certificates: false # Базовая точка поиска пользователей в FreeIPA base: "cn=accounts,dc=test,dc=local" # Ограничиваем вход только членами группы gitlab user_filter: "(memberOf=cn=gitlab,cn=groups,cn=accounts,dc=test,dc=local)" # По логину используем uid (или поставьте allow_username_or_email_login: true) allow_username_or_email_login: false block_auto_created_users: false minio: enabled: false serviceAccount: enabled: true email: display_name: 'GitLab' from: "gitlab@test.local" reply_to: "noreply@test.local" smtp: enabled: false address: "" port: 587 domain: "gitlab@test.local" user_name: "" password: secret: "" # имя секрета, где хранится пароль key: "" #password authentication: "login" # обычно используется "login" или "plain" starttls_auto: true registry: enabled: false shell: port: 2222 # так же открываем порт на ingress controller tcp: 2222: "gitlab/gitlab-gitlab-shell:2222:PROXY" tcp: proxyProtocol: false # appConfig: # lfs: # bucket: "gitlab-lfs" # path: "lfs" # connection: # secret: "biZiP3COrMLYTzflHWH6" # key: "LejhJ03w8wqsMJrDf8qeGgcHCPs3B8ml9s73GYg4" # artifacts: # bucket: "gitlab-artifacts" # path: "artifacts" # connection: # secret: "biZiP3COrMLYTzflHWH6" # key: "LejhJ03w8wqsMJrDf8qeGgcHCPs3B8ml9s73GYg4" # uploads: # bucket: "gilab-uploads" # path: "uploads" # connection: # secret: "biZiP3COrMLYTzflHWH6" # key: "LejhJ03w8wqsMJrDf8qeGgcHCPs3B8ml9s73GYg4" # packages: # bucket: "gitlab-packages" # path: "packages" # connection: # secret: "biZiP3COrMLYTzflHWH6" # key: "LejhJ03w8wqsMJrDf8qeGgcHCPs3B8ml9s73GYg4" # backups: # bucket: "gitlab-backups" # path: "backups" # tmpBucket: "gitlab-backups" # tmpPath: "tmp" # гугловая авторизация # omniauth: # enabled: true # allowSingleSignOn: ['google_oauth2'] # syncProfileAttributes: ['email'] # autoLinkSamlUser: true # blockAutoCreatedUsers: false # autoLinkUser: ['google_oauth2'] # providers: # - secret: "${kubernetes_secret.gitlab_secret.metadata[0].name}" # key: "${keys(kubernetes_secret.gitlab_secret.data)[0]}"redis: install: true master: resources: requests: cpu: "100m" memory: "100Mi" limits: cpu: "500m" memory: "300Mi" persistence: enabled: true storageClass: nfs-client size: 5Gi# Отключаем certmanagercertmanager: installCRDs: false install: false# ОтклюВключаем чаем установку встроенного PostgreSQLpostgresql: install: true primary: resources: requests: cpu: "300m" memory: "300Mi" limits: cpu: "1000m" memory: "1Gi" persistence: enabled: true storageClass: nfs-client size: 10Gi# Registry – хранение Docker-образов (если используется)registry: enabled: false# Prometheus – мониторинг GitLab (если устанавливается вместе с GitLab)prometheus: install: false# Основные компоненты GitLab (Rails/Sidekiq/Webservice)gitlab: webservice: hpa: enabled: false replicaCount: 1 minReplicas: 1 maxReplicas: 1 ingress: proxyBodySize: "5000m" proxyConnectTimeout: "8000" proxyReadTimeout: "8000" resources: requests: cpu: "500m" memory: "800Mi" limits: cpu: "1000m" memory: "3Gi" sidekiq: hpa: enabled: true cpu: targetType: Value targetAverageValue: 400m replicaCount: 1 minReplicas: 1 maxReplicas: "40" concurrency: "20" resources: requests: cpu: "400m" memory: "900Mi" limits: cpu: "1500m" memory: "2Gi" toolbox: resources: requests: cpu: "100m" memory: "200Mi" limits: cpu: "500m" memory: "1Gi" # backups: # cron: # enabled: "false" # schedule: "@daily" # resources: # requests: # cpu: "200m" # memory: "200Mi" # limits: # cpu: "1000m" # memory: "2Gi" # persistence: # enabled: "true" # accessMode: ReadWriteOnce # useGenericEphemeralVolume: false # storageClass: "nfs-client" # size: 40Gi # objectStorage: # config: # secret: "biZiP3COrMLYTzflHWH6" # key: "LejhJ03w8wqsMJrDf8qeGgcHCPs3B8ml9s73GYg4" # Gitaly – хранит Git-репозитории GitLab gitaly: resources: requests: cpu: "300m" memory: "300Mi" limits: cpu: "1000m" memory: "1Gi" persistence: enabled: true storageClass: nfs-client size: 20Gi gitlab-shell: hpa: enabled: false replicaCount: 1 minReplicas: 1 maxReplicas: 1 service: enabled: true type: ClusterIP port: 2222 resources: requests: cpu: "100m" memory: "100Mi" limits: cpu: "500m" memory: "300Mi" kas: minReplicas: 1 maxReplicas: 1 resources: requests: cpu: "100m" memory: "100Mi" limits: cpu: "200m" memory: "200Mi" gitlab-exporter: resources: requests: cpu: "100m" memory: "100Mi" limits: cpu: "500m" memory: "200Mi" appConfig: lfs: bucket: "gitlab-lfs" path: "lfs" connection: secret: "gitlab-s3-config" key: "s3.yml" artifacts: bucket: "gitlab-artifacts" path: "artifacts" connection: secret: "gitlab-s3-config" key: "s3.yml" uploads: bucket: "gitlab-uploads" path: "uploads" connection: secret: "gitlab-s3-config" key: "s3.yml" packages: bucket: "gitlab-packages" path: "packages" connection: secret: "gitlab-s3-config" key: "s3.yml" backups: bucket: "gitlab-backups" path: "backups" tmpBucket: "gitlab-backups" tmpPath: "tmp"gitlab-runner: install: false |
| --- | --- |

от предыдущего отличается вот этой частью:

| 123456789101112131415161718192021222324 | global: appConfig: ldap: enabled: true servers: main: label: "FreeIPA LDAP" host: "192.168.1.100" port: 389 uid: "uid" # в FreeIPA имя юзера в атрибуте uid bind_dn: "uid=gitlab,cn=sysaccounts,cn=etc,dc=test,dc=local" password: secret: gitlab-ldap-secret key: ldap.bindPW # Шифрование: plain – без TLS; start_tls – через StartTLS; simple_tls – LDAPS encryption: "plain" verify_certificates: false # Базовая точка поиска пользователей в FreeIPA base: "cn=accounts,dc=test,dc=local" # Ограничиваем вход только членами группы gitlab user_filter: "(memberOf=cn=gitlab,cn=groups,cn=accounts,dc=test,dc=local)" # По логину используем uid (или поставьте allow_username_or_email_login: true) allow_username_or_email_login: false block_auto_created_users: false |
| --- | --- |

во freeipa у нас есть группа gitlab и пользовать user1

![](/news/sidmidru/article-db03824b64600e5b/image-265.png)

пробуем залогиниться с ним:

![](/news/sidmidru/article-db03824b64600e5b/image-266.png)

![](/news/sidmidru/article-db03824b64600e5b/image-267.png)

как видим всё ок.

### []Резервное копирование etcd в hostPath

так как мы на своём железе нам нужно делать бэкапы etcd, тут мы будет делать snapshot etcd и сохранять его локально на мастерах

для этого первым делом создадим на всем мастерах директорию под бэкапы:

**for node in $(kubectl get nodes -o wide | grep -i master | awk '{print $6}' ); do ssh root@$node mkdir -p /var/backups/etcd; done**

скрипт будет запускать каждый день в 2 часа ночи, бэкап будет сохраняться на той тачке где запустился job

/etc/ansible/kubespray-official/etcd-backup/backup-to-host/backup.yaml

| 12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152 | apiVersion: batch/v1kind: CronJobmetadata: name: etcd-backup namespace: kube-systemspec: schedule: "0 2 * * *" # каждый день в 02:00 jobTemplate: spec: template: spec: hostNetwork: true dnsPolicy: ClusterFirstWithHostNet nodeSelector: node-role.kubernetes.io/control-plane: "" # только на мастерах tolerations: - key: node-role.kubernetes.io/control-plane operator: Exists effect: NoSchedule securityContext: runAsUser: 0 restartPolicy: OnFailure volumes: - name: etcd-ssl hostPath: path: /etc/ssl/etcd/ssl type: Directory - name: backup-dir hostPath: path: /var/backups/etcd type: DirectoryOrCreate containers: - name: etcd-backup image: bitnami/etcd:3 volumeMounts: - name: etcd-ssl mountPath: /ssl readOnly: true - name: backup-dir mountPath: /backup command: - /bin/sh - -c - | export ETCDCTL_API=3 etcdctl \ --endpoints=https://127.0.0.1:2379 \ --cacert=/ssl/ca.pem \ --cert=/ssl/admin-$(hostname -f).pem \ --key=/ssl/admin-$(hostname -f)-key.pem \ snapshot save /backup/etcd-$(date +%Y%m%dT%H%M%S).db |
| --- | --- |

**kubectl apply -f backup.yaml**

вручную запустим проверим:

kubectl create job etcd-backup-now1 --from=cronjob/etcd-backup -n kube-system

|  | root@kub-master1:/var/backups/etcd# ls -lahtotal 56Mdrwxr-xr-x 2 root root 4.0K Jun 29 16:25 .drwxr-xr-x 12 root root 4.0K Jun 29 16:16 ..-rw------- 1 root root 56M Jun 29 16:25 etcd-20250629T102518.dbroot@kub-master1:/var/backups/etcd# kubectl logs job/etcd-backup-now1 -n kube-system{"level":"info","ts":1751192718.184084,"caller":"snapshot/v3_snapshot.go:68","msg":"created temporary db file","path":"/backup/etcd-20250629T102518.db.part"}{"level":"info","ts":1751192718.1902862,"logger":"client","caller":"v3/maintenance.go:211","msg":"opened snapshot stream; downloading"}{"level":"info","ts":1751192718.1904283,"caller":"snapshot/v3_snapshot.go:76","msg":"fetching snapshot","endpoint":"https://127.0.0.1:2379"}{"level":"info","ts":1751192718.6031828,"logger":"client","caller":"v3/maintenance.go:219","msg":"completed snapshot read; closing"}{"level":"info","ts":1751192718.6250348,"caller":"snapshot/v3_snapshot.go:91","msg":"fetched snapshot","endpoint":"https://127.0.0.1:2379","size":"58 MB","took":"now"}{"level":"info","ts":1751192718.6251087,"caller":"snapshot/v3_snapshot.go:100","msg":"saved","path":"/backup/etcd-20250629T102518.db"}Snapshot saved at /backup/etcd-20250629T102518.db |
| --- | --- |

ок снапшот создаётся.

### []Резервное копирование etcd в s3-minio

так же создаём снапшот но будет хранить его в s3-minio

для этого сначала создадим bucket**etcd-snapshots**

вот его policy

| 12345678910111213141516171819202122232425262728293031 | { "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Action": [ "s3:ListBucket", "s3:GetBucketLocation" ], "Resource": [ "arn:aws:s3:::etcd-snapshots", "arn:aws:s3:::etcd-snapshots/*" ] }, { "Effect": "Allow", "Action": [ "s3:AbortMultipartUpload", "s3:DeleteObject", "s3:GetObject", "s3:ListBucketMultipartUploads", "s3:ListMultipartUploadParts", "s3:PutObject" ], "Resource": [ "arn:aws:s3:::etcd-snapshots", "arn:aws:s3:::etcd-snapshots/*" ] } ]} |
| --- | --- |

пользователь**etcd**и вот креды пользователя:

access: YcGmA6wqztm5FX4Pfwnf
secret: t0TkdM2Om2P4ZX4mgXx6Sxy9E9SsYnYO9MkIJsCI

![](/news/sidmidru/article-db03824b64600e5b/image-268.png)

![](/news/sidmidru/article-db03824b64600e5b/image-269.png)

![](/news/sidmidru/article-db03824b64600e5b/image-270.png)

теперь создаём секрет

kubectl create secret generic s3-credentials \
--from-literal=access-key=ВАШ_ACCESS_KEY\
--from-literal=secret-key=ВАШ_SECRET_KEY\
-n kube-system

в моём случае это:

|  | kubectl create secret generic s3-credentials \ --from-literal=access-key=YcGmA6wqztm5FX4Pfwnf \ --from-literal=secret-key=t0TkdM2Om2P4ZX4mgXx6Sxy9E9SsYnYO9MkIJsCI \ -n kube-system |
| --- | --- |

а вот cronjob

/etc/ansible/kubespray-official/etcd-backup/backup-to-s3-minio/backup.yaml

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374757677787980818283 | apiVersion: batch/v1kind: CronJobmetadata: name: etcd-backup-to-s3 namespace: kube-systemspec: schedule: "0 */12 * * *" # каждые 12 часов jobTemplate: spec: template: spec: hostNetwork: true dnsPolicy: ClusterFirstWithHostNet nodeSelector: node-role.kubernetes.io/control-plane: "" tolerations: - key: node-role.kubernetes.io/control-plane operator: Exists effect: NoSchedule securityContext: runAsUser: 0 restartPolicy: OnFailure volumes: - name: etcd-ssl hostPath: path: /etc/ssl/etcd/ssl type: Directory - name: temp-backup emptyDir: {} initContainers: - name: snapshot image: bitnami/etcd:3 volumeMounts: - name: etcd-ssl mountPath: /ssl readOnly: true - name: temp-backup mountPath: /snapshot command: - /bin/sh - -c - | export ETCDCTL_API=3 etcdctl \ --endpoints=https://127.0.0.1:2379 \ --cacert=/ssl/ca.pem \ --cert=/ssl/admin-$(hostname -f).pem \ --key=/ssl/admin-$(hostname -f)-key.pem \ snapshot save /snapshot/etcd-$(date +%Y%m%dT%H%M%S).db containers: - name: uploader image: amazon/aws-cli:latest env: - name: AWS_ACCESS_KEY_ID valueFrom: secretKeyRef: name: s3-credentials key: access-key - name: AWS_SECRET_ACCESS_KEY valueFrom: secretKeyRef: name: s3-credentials key: secret-key - name: AWS_REGION value: us-east-1 volumeMounts: - name: temp-backup mountPath: /snapshot command: - /bin/sh - -c - | LATEST=$(ls /snapshot/etcd-*.db | sort | tail -n1) echo "Uploading $LATEST to etcd-snapshots…" aws s3api put-object \ --bucket etcd-snapshots \ --key "$(basename $LATEST)" \ --body "$LATEST" \ --endpoint-url http://192.168.1.120:9000 |
| --- | --- |

тут нам нужно поправить
**--bucket**указываем наш созданный бакет
**--endpoint-url**указываем адрес нашего s3-minio

алпаим

**kubectl apply -f etcd-backup/backup-to-s3-minio/backup.yaml**

проверяем
kubectl create job etcd-backup-s3 --from=cronjob/etcd-backup-to-s3 -n kube-system

|  | kubectl logs -n kube-system etcd-backup-s3-jj7zc Defaulted container "uploader" out of: uploader, snapshot (init)Uploading /snapshot/etcd-20250629T115320.db to etcd-snapshots...{ "ETag": "\"2b47552c13a06a1d94e5ee8f02948ad4\""} |
| --- | --- |

всё норм.
проверяем бакет

![](/news/sidmidru/article-db03824b64600e5b/image-271.png)

всё норм

#### []Резервное копирование etcd в s3-minio (LDAPenabled)

не забываем если включёнLDAPгенерить токен нужно в консоли, для начала создадим пользователя в freeipa, добавляем его в группу s3-minio-admins (выше по статье я всё описывал как мы его создавали)

![](/news/sidmidru/article-db03824b64600e5b/image-272.png)

проверяем что пользователь подтягивается

![](/news/sidmidru/article-db03824b64600e5b/image-273.png)

при первой проверке ну будет видно что пользователю приатачена policy, поэтому идём в консоль на сервер

**ssh 192.168.1.118**

root@s3-minio-1:~#**mc idp ldap policy attach myminio etcd-snapshots --user=user-etcd**
Attached Policies: [etcd-snapshots]
To User: user-etcd

генерим токен

root@s3-minio-1:~#**mc admin user svcacct add myminio user-etcd**
Access Key:O0TQHSCRWF7ZXQXGLUGG
Secret Key: ZBCUUUSl+xmOQRXK4KqczWfUKks6xzXm9sovPPIl
Expiration: no-expiry

проверяем в панели всё ок:

![](/news/sidmidru/article-db03824b64600e5b/image-274.png)

ну и создаём секрет:

|  | kubectl create secret generic s3-credentials \ --from-literal=access-key=O0TQHSCRWF7ZXQXGLUGG \ --from-literal=secret-key=ZBCUUUSl+xmOQRXK4KqczWfUKks6xzXm9sovPPIl \ -n kube-system |
| --- | --- |

проверяем что cronjob создаётся и работает:

root@kub-master1:~#**kubectl create job etcd-backup-s3 --from=cronjob/etcd-backup-to-s3 -n kube-system**

|  | root@kub-master1:~# kubectl logs -n kube-system etcd-backup-s3-vfc6d Defaulted container "uploader" out of: uploader, snapshot (init)Uploading /snapshot/etcd-20250704T081714.db to etcd-snapshots...{ "ETag": "\"cae1f28b2b4e13cfb5d2e566dc9df774\""} |
| --- | --- |

### []Cert-manager (self-signed - самоподписанный)

https://github.com/cert-manager/cert-manager/tree/master/deploy/charts/cert-manager

чтобы каждый раз не бегать и руками не создать сертификат и секреты - можно использовать cert-manager

**kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.18.2/cert-manager.crds.yaml**

**helm repo add jetstack https://charts.jetstack.io --force-update**

**kubectl create ns cert-manager**

/etc/ansible/kubespray-official/cert-manager/values.yaml

| 1234567891011121314151617181920212223242526 | resources: requests: cpu: 10m memory: 32Mi limits: cpu: 20m memory: 50Miwebhook: resources: requests: cpu: 10m memory: 32Mi limits: cpu: 20m memory: 50Micainjector: resources: requests: cpu: 10m memory: 32Mi limits: cpu: 20m memory: 50Mi |
| --- | --- |

**helm upgrade --install cert-manager --namespace cert-manager --version v1.18.2 jetstack/cert-manager --values values.yaml**

/etc/ansible/kubespray-official/cert-manager/issuer-selfsigned.yaml

|  | apiVersion: cert-manager.io/v1kind: Issuermetadata: name: ca-selfsign namespace: cert-managerspec: selfSigned: {} |
| --- | --- |

/etc/ansible/kubespray-official/cert-manager/certificate-root-ca.yaml

|  | apiVersion: cert-manager.io/v1kind: Certificatemetadata: name: test-local-root-ca namespace: cert-managerspec: isCA: true commonName: "test.local" secretName: test-local-root-ca-secret duration: 8760h # 1 год renewBefore: 360h # обновлять за 15 дней до окончания issuerRef: name: ca-selfsign kind: Issuer |
| --- | --- |

/etc/ansible/kubespray-official/cert-manager/issuer-ca.yaml

|  | apiVersion: cert-manager.io/v1kind: ClusterIssuermetadata: name: ca-issuerspec: ca: secretName: test-local-root-ca-secret |
| --- | --- |

/etc/ansible/kubespray-official/cert-manager/certificate-wildcard.yaml

| 123456789101112131415161718 | apiVersion: cert-manager.io/v1kind: Certificatemetadata: name: wildcard-test-local namespace: cert-managerspec: secretName: wildcard-test-local-tls commonName: "*.test.local" dnsNames: - "*.test.local" - "test.local" # если нужен и корневой домен duration: 720h # 30 дней renewBefore: 168h # за неделю до окончания issuerRef: name: ca-issuer kind: Issuer |
| --- | --- |

мы будем выписывать самоподписанный сертификат *.test.local

**kubectl apply -f issuer-selfsigned.yaml**

**kubectl apply -f certificate-root-ca.yaml**

**kubectl apply -f issuer-ca.yaml**

**kubectl apply -f certificate-wildcard.yaml**

Issuer selfSigned (**issuer-selfsigned.yaml**)

Генерирует корневой ключ/сертификатCAв первом Certificate-шаге.

Certificate Root-CA (**certificate-root-ca.yaml**)

Сам Certificate с isCA: true — он подписывается первым Issuer’ом и сохраняет в Secret ваш корневойCA.

IssuerCA(**issuer-ca.yaml**)

Берёт этот корневой CA-секрет (ключ + cert) и становится «настоящим» подписывающим Issuer’ом для любых downstream-запросов.

Certificate wildcard (**certificate-wildcard.yaml**)

Запрос сертификата *.test.local, подписанного уже вашим CA-Issuer’ом.

чтобы получать сертификат - нужно добавить в аннотации

|  | annotations: cert-manager.io/cluster-issuer: ca-issuer |
| --- | --- |

вот пример:

**kubectl create ns argocd**

/etc/ansible/kubespray-official/cert-manager/example-cert.yaml

| 123456789101112131415161718192021222324252627282930313233343536 | apiVersion: networking.k8s.io/v1kind: Ingressmetadata: name: argo-ingress namespace: argocd annotations: cert-manager.io/cluster-issuer: ca-issuerspec: tls: - hosts: - argo.test.local - www.argo.test.local secretName: argo-tls-secret rules: - host: argo.test.local http: paths: - path: / pathType: Prefix backend: service: name: argo-server port: number: 80 - host: www.argo.test.local http: paths: - path: / pathType: Prefix backend: service: name: argo-server port: number: 80 |
| --- | --- |

**kubectl apply -f example-cert.yaml**

|  | kubectl get secrets -n argocd NAME TYPE DATA AGEargo-tls-secret kubernetes.io/tls 3 16m |
| --- | --- |

там будут

**tls.crt**: # публичный сертификат *.test.local

**tls.key**: # приватный ключ к нему

**ca.crt**: # корневой CA-сертификат, на базе которого был подписан tls.crt

на этом всё.

### []Argocd

**kubectl create ns argocd**

**helm repo add argo-cd https://argoproj.github.io/argo-helm**

/etc/ansible/kubespray-official/argocd/values.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596 | global: domain: argocd.test.localconfigs: params: server.insecure: trueserver: replicas: 1 ingress: enabled: true ingressClassName: nginx annotations: cert-manager.io/cluster-issuer: ca-issuer nginx.ingress.kubernetes.io/force-ssl-redirect: "true" nginx.ingress.kubernetes.io/backend-protocol: "HTTP" extraTls: - hosts: - argocd.test.local # Based on the ingress controller used secret might be optional secretName: argocd-tls resources: limits: cpu: 100m memory: 128Mi requests: cpu: 50m memory: 64Miredis-ha: enabled: true haproxy: resources: limits: cpu: 100m memory: 128Mi requests: cpu: 50m memory: 64Mi redis: resources: limits: cpu: 100m memory: 128Mi requests: cpu: 50m memory: 64Mi sentinel: resources: requests: memory: 200Mi cpu: 100m limits: memory: 200Mi cpu: 500mcontroller: replicas: 1 resources: limits: cpu: 500m memory: 512Mi requests: cpu: 250m memory: 150MirepoServer: replicas: 2 resources: limits: cpu: 50m memory: 128Mi requests: cpu: 10m memory: 64MiapplicationSet: replicas: 2 resources: limits: cpu: 100m memory: 128Mi requests: cpu: 100m memory: 128Midex: resources: limits: cpu: 50m memory: 64Mi requests: cpu: 10m memory: 32Mi |
| --- | --- |

**helm upgrade --install argo-cd argo-cd/argo-cd -n argocd --values values.yaml --version 8.1.3**

ждём пока установится, потом смотрим пароль:

**kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d**

![](/news/sidmidru/article-db03824b64600e5b/image-275.png)

![](/news/sidmidru/article-db03824b64600e5b/image-276.png)

### []Argocd add user

# 1) Генерим пароль

**kubectl exec -ti -n argocd argo-cd-argocd-server-84c56df477-h6mnl -- bash**

**argocd account bcrypt --password '4CL8do3zkZzPHfuk22'**

получаем:

$2a$10$R1OkCf90A7Kg/aMA.DD3DOv9y12FPLP6eh0ViukCt/NIf.c/Q/oDy

# 2) полученный hash присваиваем переменной

**RAW_HASH='$2a$10$R1OkCf90A7Kg/aMA.DD3DOv9y12FPLP6eh0ViukCt/NIf.c/Q/oDy'**

# 3) Кодируем его в base64 (это и будет ваше accounts.test.password)

**HASH_B64=$(echo -n "$RAW_HASH" | base64 -w0)**

# 4) Задаём правильный RFC3339-штамп:

**RAW_MTIME='2025-07-14T12:48:50Z'**

# 5) Кодируем штамп в base64 (это будет ваше accounts.test.passwordMtime)

**MTIME_B64=$(echo -n "$RAW_MTIME" | base64 -w0)**

# 6) Добавляем пароль для пользователя test

|  | kubectl patch secret argocd-secret -n argocd --type=json -p="[ { \"op\":\"add\", \"path\":\"/data/accounts.test.password\", \"value\":\"$HASH_B64\" }, { \"op\":\"add\", \"path\":\"/data/accounts.test.passwordMtime\", \"value\":\"$MTIME_B64\" }]" |
| --- | --- |

# 7) Добавляем самого пользователя test в конфигмап

|  | kubectl patch configmap argocd-cm -n argocd --type=merge -p '{ "data": { "accounts.test":"login", "accounts.test.enabled":"true" }}' |
| --- | --- |

# 8) Выдать доступы, например пользователь test будет админом а пользователь alice будет иметь доступы readonly

|  | kubectl patch configmap argocd-rbac-cm -n argocd --type=merge -p '{ "data": { "policy.csv": "p, role:readonly, applications, get, *, allow\ng, alice, role:readonly\np, role:admin, *, *, *, allow\ng, test, role:admin\n" }}' |
| --- | --- |

проверяем:

|  | root@kub-master1:~/argocd# kubectl -n argocd get configmap argocd-rbac-cm -o yamlapiVersion: v1data: policy.csv: | p, role:readonly, applications, get, *, allow g, alice, role:readonly p, role:admin, *, *, *, allow g, test, role:admin policy.default: "" policy.matchMode: glob scopes: '[groups]' |
| --- | --- |

# 9) Рестартуем и можем проверять аутентификацию

**kubectl rollout restart -n argocd deployment argo-cd-argocd-server**

проверяем:

Заходим под пользователем test - у которого есть все права

![](/news/sidmidru/article-db03824b64600e5b/image-277.png)

![](/news/sidmidru/article-db03824b64600e5b/image-278.png)

![](/news/sidmidru/article-db03824b64600e5b/image-279.png)

![](/news/sidmidru/article-db03824b64600e5b/image-280.png)

![](/news/sidmidru/article-db03824b64600e5b/image-281.png)

как видим проект успешно создан

теперь зайдём под пользователем alice и так же попробуем создать проект:

![](/news/sidmidru/article-db03824b64600e5b/image-282.png)

![](/news/sidmidru/article-db03824b64600e5b/image-283.png)

![](/news/sidmidru/article-db03824b64600e5b/image-284.png)

![](/news/sidmidru/article-db03824b64600e5b/image-285.png)

как видим всё ок. доступов не хватает - как и задумано.

### []Argocd интеграция с Freeipa

для начала создаём sysaccount во freeipa
[root@freeipa-1 ]#**cd /etc/ipa**
[root@freeipa-1 ipa]#**bash freeipa-sam.sh**

| 123456789101112131415161718192021 | ### FreeIPA - System Account Manager ###1.) ldapserver=freeipa-1.test.local2.) domain=test.local (ldapdomain=dc=test,dc=local)3.) binduser=uid=admin,cn=users,cn=accounts,dc=test,dc=local4.) bindpass=SET!5.) ssl=falseActions (ready): add | rm | ls | info | passwd | save--- Results ---dn: uid=sudo,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=nexus,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=gitlab,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=vault,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=s3minio,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=k8s-access,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=keycloak,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=rancher,cn=sysaccounts,cn=etc,dc=test,dc=localdn: uid=argocd,cn=sysaccounts,cn=etc,dc=test,dc=local--- End Results --- |
| --- | --- |

создаём 2 группы

![](/news/sidmidru/article-db03824b64600e5b/image-286.png)

в админах у меня user1
в read only user2

проверить наличие пользователя в группах мы можем вот такой командой:

**ldapsearch -H ldap://192.168.1.100 -D "uid=argocd,cn=sysaccounts,cn=etc,dc=test,dc=local" -W -b "uid=user1,cn=users,cn=accounts,dc=test,dc=local" "(objectClass=*)" memberOf**

| 12345678910111213141516171819202122232425 | [root@freeipa-1 ipa]# ldapsearch -H ldap://192.168.1.100 -D "uid=argocd,cn=sysaccounts,cn=etc,dc=test,dc=local" -W -b "uid=user1,cn=users,cn=accounts,dc=test,dc=local" "(objectClass=*)" memberOfEnter LDAP Password: # extended LDIF## LDAPv3# base <uid=user1,cn=users,cn=accounts,dc=test,dc=local> with scope subtree# filter: (objectClass=*)# requesting: memberOf ## user1, users, accounts, test.localdn: uid=user1,cn=users,cn=accounts,dc=test,dc=localmemberOf: cn=ipausers,cn=groups,cn=accounts,dc=test,dc=localmemberOf: cn=nexus-admins,cn=groups,cn=accounts,dc=test,dc=localmemberOf: cn=vault-admins,cn=groups,cn=accounts,dc=test,dc=localmemberOf: cn=s3-minio-admins,cn=groups,cn=accounts,dc=test,dc=localmemberOf: cn=k8s-devops,cn=groups,cn=accounts,dc=test,dc=localmemberOf: cn=argocd-admin,cn=groups,cn=accounts,dc=test,dc=local# search resultsearch: 2result: 0 Success# numResponses: 2# numEntries: 1 |
| --- | --- |

проверить какие пользователи есть в группе вот такой командой:

**ldapsearch -H ldap://192.168.1.100 -D "uid=argocd,cn=sysaccounts,cn=etc,dc=test,dc=local" -W -b "cn=argocd-admin,cn=groups,cn=accounts,dc=test,dc=local" "(objectClass=*)" member**

| 1234567891011121314151617181920 | [root@freeipa-1 ipa]# ldapsearch -H ldap://192.168.1.100 -D "uid=argocd,cn=sysaccounts,cn=etc,dc=test,dc=local" -W -b "cn=argocd-admin,cn=groups,cn=accounts,dc=test,dc=local" "(objectClass=*)" memberEnter LDAP Password: # extended LDIF## LDAPv3# base <cn=argocd-admin,cn=groups,cn=accounts,dc=test,dc=local> with scope subtree# filter: (objectClass=*)# requesting: member ## argocd-admin, groups, accounts, test.localdn: cn=argocd-admin,cn=groups,cn=accounts,dc=test,dc=localmember: uid=user1,cn=users,cn=accounts,dc=test,dc=local# search resultsearch: 2result: 0 Success# numResponses: 2# numEntries: 1 |
| --- | --- |

пароль для sysaccount будем хранить в secret

/etc/ansible/kubespray-official/argocd/argocd-ldap-secret.yaml

|  | apiVersion: v1kind: Secretmetadata: name: argocd-ldap-secret namespace: argocdtype: OpaquestringData: # bindDN вашего LDAP‑пользователя ldap.bindDN: "uid=argocd,cn=sysaccounts,cn=etc,dc=test,dc=local" # bindPW — здесь указываете ваш пароль ldap.bindPW: "Secret123" |
| --- | --- |

root@kub-master1:~/argocd#**kubectl apply -f argocd-ldap-secret.yaml**

вот values
/etc/ansible/kubespray-official/argocd/values-ldap.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160 | global: domain: argocd.test.localconfigs: params: server.insecure: true # 1) ConfigMap argocd-cm cm: create: true annotations: {} # по надобности url: https://argocd.test.local dex.config: | connectors: - type: ldap id: freeipa name: FreeIPA config: host: 192.168.1.100:389 insecureNoSSL: true insecureSkipVerify: true bindDN: $LDAP_BIND_DN bindPW: $LDAP_BIND_PW usernamePrompt: Username userSearch: baseDN: "cn=users,cn=accounts,dc=test,dc=local" filter: "(&(objectClass=posixAccount)(|(memberOf=cn=argocd-admin,cn=groups,cn=accounts,dc=test,dc=local)(memberOf=cn=argocd-ro-users,cn=groups,cn=accounts,dc=test,dc=local)))" username: uid idAttr: uid emailAttr: uid nameAttr: displayName groupSearch: baseDN: cn=groups,cn=accounts,dc=test,dc=local filter: "(objectClass=groupOfNames)" userMatchers: - userAttr: DN groupAttr: member nameAttr: cn rbac: create: true policy.csv: | p, role:admin, applications, *, *, allow g, "argocd-admin", role:admin p, role:readonly, applications, get, *, allow g, "argocd-ro-users", role:readonly policy.default: "" policy.matchMode: glob scopes: '[groups]'server: replicas: 1 ingress: enabled: true ingressClassName: nginx annotations: cert-manager.io/cluster-issuer: ca-issuer nginx.ingress.kubernetes.io/force-ssl-redirect: "true" nginx.ingress.kubernetes.io/backend-protocol: "HTTP" extraTls: - hosts: - argocd.test.local # Based on the ingress controller used secret might be optional secretName: argocd-tls resources: limits: cpu: 100m memory: 128Mi requests: cpu: 50m memory: 64Miredis-ha: enabled: true haproxy: resources: limits: cpu: 100m memory: 128Mi requests: cpu: 50m memory: 64Mi redis: resources: limits: cpu: 100m memory: 128Mi requests: cpu: 50m memory: 64Mi sentinel: resources: requests: memory: 200Mi cpu: 100m limits: memory: 200Mi cpu: 500mcontroller: replicas: 1 resources: limits: cpu: 500m memory: 512Mi requests: cpu: 250m memory: 150MirepoServer: replicas: 2 resources: limits: cpu: 50m memory: 128Mi requests: cpu: 10m memory: 64MiapplicationSet: replicas: 2 resources: limits: cpu: 100m memory: 128Mi requests: cpu: 100m memory: 128Midex: resources: limits: cpu: 50m memory: 64Mi requests: cpu: 10m memory: 32Mi env: - name: LDAP_BIND_DN valueFrom: secretKeyRef: name: argocd-ldap-secret key: ldap.bindDN - name: LDAP_BIND_PW valueFrom: secretKeyRef: name: argocd-ldap-secret key: ldap.bindPW |
| --- | --- |

тут:

host: 192.168.1.100:389 адрес freeipa сервера
bindDN: $LDAP_BIND_DNлогин из секрета
bindPW: $LDAP_BIND_PWпароль из секрета
filter: "(&(objectClass=posixAccount)(|(memberOf=cn=argocd-admin,cn=groups,cn=accounts,dc=test,dc=local)(memberOf=cn=argocd-ro-users,cn=groups,cn=accounts,dc=test,dc=local)))" ограничиваем доступ 2мя группами argocd-admin и argocd-ro-users

|  | policy.csv: | p, role:admin, applications, *, *, allow g, "argocd-admin", role:admin p, role:readonly, applications, get, *, allow g, "argocd-ro-users", role:readonly |
| --- | --- |

а тут выдаём rbac доступы для групп
dex:
env:
- name:LDAP_BIND_DN

тут подтягиваем в переменные логин и пароль

можем ставить

root@kub-master1:~/argocd#**helm upgrade --install argo-cd argo-cd/argo-cd -n argocd --values values-ldap.yaml --version 8.1.3**

проверяем:

![](/news/sidmidru/article-db03824b64600e5b/image-287.png)

![](/news/sidmidru/article-db03824b64600e5b/image-288.png)

как видим пользователь user3 не имеет доступа

проверим user2

![](/news/sidmidru/article-db03824b64600e5b/image-289.png)

![](/news/sidmidru/article-db03824b64600e5b/image-290.png)

как видим мы можем зайти но ничего создать не можем

проверим теперь user1

![](/news/sidmidru/article-db03824b64600e5b/image-291.png)

можем увидеть в каких группах состоит этот пользователь

создадим группу:

![](/news/sidmidru/article-db03824b64600e5b/image-292.png)

![](/news/sidmidru/article-db03824b64600e5b/image-293.png)

как видим всё ок - успешно создаётся, прав хватает.

### []Argocd создание проекта, настройка деплоя

в моей репке
https://gitlab.test.local/argo/test-app/app1.git
есть только 1 файл:

deployment.yaml

| 1234567891011121314151617181920212223242526272829303132333435363738 | apiVersion: apps/v1kind: Deploymentmetadata: name: nginx-deploymentspec: replicas: 2 selector: matchLabels: app: nginx template: metadata: labels: app: nginx spec: containers: - name: nginx image: nginx:latest resources: requests: cpu: "50m" memory: "64Mi" limits: cpu: "100m" memory: "128Mi" livenessProbe: httpGet: path: / port: 80 initialDelaySeconds: 15 periodSeconds: 20 failureThreshold: 3 readinessProbe: httpGet: path: / port: 80 initialDelaySeconds: 5 periodSeconds: 10 failureThreshold: 3 |
| --- | --- |

которые запускает деплоймент в 2х репликах с минимальными ресурсами и liveness readiness пробами.

создаём новый проект

![](/news/sidmidru/article-db03824b64600e5b/image-294.png)

![](/news/sidmidru/article-db03824b64600e5b/image-295.png)

добавляем целевой кластер

![](/news/sidmidru/article-db03824b64600e5b/image-296.png)

указываем что проекту будет разрешено

![](/news/sidmidru/article-db03824b64600e5b/image-297.png)

для подключения репозитория будем использовать access token

токен делается на уровне группы:

![](/news/sidmidru/article-db03824b64600e5b/image-298.png)

создаю argo-token из доступов хватает read_repository но я ещё тыкнул на api лень было пересоздавать оставил так.
после создания будет показан пароль который показывается только 1 раз, сохраните его

![](/news/sidmidru/article-db03824b64600e5b/image-299.png)

возвращаемся в argocd

![](/news/sidmidru/article-db03824b64600e5b/image-300.png)

![](/news/sidmidru/article-db03824b64600e5b/image-301.png)

![](/news/sidmidru/article-db03824b64600e5b/image-302.png)

заполняем все указанные поля, и не забываем включить Skip server verification

кстати подключить репозиторий можно командой:

|  | argocd@argo-cd-argocd-server-6c68fc4594-xhjn4:~$ argocd repo add https://gitlab.test.local/argo/test-app/app1.git \ --username argo-token \ --password glpat-Ssh************** \ --insecure-skip-server-verification |
| --- | --- |

как видим репозиторий подключен

![](/news/sidmidru/article-db03824b64600e5b/image-303.png)

теперь создадим application который будет автоматически синхронизироваться

![](/news/sidmidru/article-db03824b64600e5b/image-304.png)

выбираем наш репозиторий указываем что по branch должен синкаться, можно ещё выбрать по тэгам, и в path ставим точку (если прям в корне лежат файлы которые нужно аплаить, а если в какой то другой директории то указываем до неё путь, можно ещё включить и рекурсивную проходу по всем вложенными директориям в path)

![](/news/sidmidru/article-db03824b64600e5b/image-305.png)

сохраняем и получаем:

![](/news/sidmidru/article-db03824b64600e5b/image-306.png)

как видим сразу пошёл процесс синхронизации, так как мы указали Sync Policy автоматически.

как видим всё ок, и оба наших pod задеплоились

![](/news/sidmidru/article-db03824b64600e5b/image-307.png)

проверим:

|  | root@kub-master1:~# kubectl get pod -n testNAME READY STATUS RESTARTS AGEnginx-deployment-fd447d4f5-nlkzv 1/1 Running 0 4m10snginx-deployment-fd447d4f5-x74w4 1/1 Running 0 4m10s |
| --- | --- |

мы можем скейлить наш деплоймент в любое количество - argocd будет возвращать к исходному в 2 реплики.

для примера выключуSELFHEALи деплоймент заскейлю в 1.

выбираю апку

![](/news/sidmidru/article-db03824b64600e5b/image-308.png)

прометываем ниже:

![](/news/sidmidru/article-db03824b64600e5b/image-309.png)

выключаемSELFHEAL

![](/news/sidmidru/article-db03824b64600e5b/image-310.png)

приводим к такому виду:

![](/news/sidmidru/article-db03824b64600e5b/image-311.png)

скейлим апку вниз:

|  | root@kub-master1:~# kubectl scale deployment -n test nginx-deployment --replicas 1deployment.apps/nginx-deployment scaledroot@kub-master1:~# kubectl get pod -n testNAME READY STATUS RESTARTS AGEnginx-deployment-fd447d4f5-x74w4 1/1 Running 0 9m27s |
| --- | --- |

как видим только 1 pod

проверяем:

![](/news/sidmidru/article-db03824b64600e5b/image-312.png)

как видимSYNCSTATUS= OutOfSync

если нажать на OutOfSync откроется diff

![](/news/sidmidru/article-db03824b64600e5b/image-313.png)

который покажет различие:

![](/news/sidmidru/article-db03824b64600e5b/image-314.png)

для восстановления может нажать sync

![](/news/sidmidru/article-db03824b64600e5b/image-315.png)

![](/news/sidmidru/article-db03824b64600e5b/image-316.png)

![](/news/sidmidru/article-db03824b64600e5b/image-317.png)

как видим всё ок - синканулось

### []Argocd создание проекта, настройка деплоя Helm

Теперь с помощью argocd установим апку которая использует helm.

у нас есть репка
https://gitlab.test.local/argo/test-helm/app1.git

я создал дефолтный chart
**helm create helm-app1**

и пушнул его в gitlab repo

![](/news/sidmidru/article-db03824b64600e5b/image-318.png)

добавляем этот репозиторий в argocd

![](/news/sidmidru/article-db03824b64600e5b/image-319.png)

напоминаю что для моего проекта project-dev можно деплоить в любой неймспейс

![](/news/sidmidru/article-db03824b64600e5b/image-320.png)

![](/news/sidmidru/article-db03824b64600e5b/image-321.png)

теперь создадим application

![](/news/sidmidru/article-db03824b64600e5b/image-322.png)

будем автоматически ставить без ручной синхронизации + включим создание namespace

![](/news/sidmidru/article-db03824b64600e5b/image-323.png)

указываем нашу репку имя неймспейса test2 и путь где лежит чарт

![](/news/sidmidru/article-db03824b64600e5b/image-324.png)

дальше появляется меню где можем выставить настройки для values которыеНЕзакомменчены

![](/news/sidmidru/article-db03824b64600e5b/image-325.png)

как видим всё ок.

![](/news/sidmidru/article-db03824b64600e5b/image-326.png)

![](/news/sidmidru/article-db03824b64600e5b/image-327.png)

### []Argocd создание проекта из конфиг файла

Всё ок, мы натыкали application в панели, но есть проблема если мы удалим его в панели то с включённым финалайзером удалится и апка и потом надо будет вручную добавлять всё.
поэтому нужно хранить апликейшены в конфигах.

вытащить конфиг файл можем так:

|  | kubectl get applications.argoproj.io app1 -n argocd -o json \ | yq -y 'del( .status, .metadata.managedFields, .metadata.annotations."kubectl.kubernetes.io/last-applied-configuration", .metadata.creationTimestamp, .metadata.resourceVersion, .metadata.uid, .metadata.generation )' |
| --- | --- |

конфиг получаем следующий:

| 1234567891011121314151617181920 | apiVersion: argoproj.io/v1alpha1kind: Applicationmetadata: finalizers: - resources-finalizer.argocd.argoproj.io name: app1 namespace: argocdspec: destination: namespace: test server: https://kubernetes.default.svc project: project-dev source: path: . repoURL: https://gitlab.test.local/argo/test-app/app1.git targetRevision: HEAD syncPolicy: automated: prune: true selfHeal: true |
| --- | --- |

helm app вот такой:

| 12345678910111213141516171819202122 | apiVersion: argoproj.io/v1alpha1kind: Applicationmetadata: finalizers: - resources-finalizer.argocd.argoproj.io name: helm-app1 namespace: argocdspec: destination: namespace: test2 server: https://kubernetes.default.svc project: project-dev source: path: helm-app1 repoURL: https://gitlab.test.local/argo/test-helm/app1.git targetRevision: HEAD syncPolicy: automated: prune: true selfHeal: true syncOptions: - CreateNamespace=true |
| --- | --- |

сложим эти конфиги в репку

https://gitlab.test.local/argo/argo-config.git

файлы будут лежать тут:

argo-config/app1/app1.yaml
argo-config/helm-app1/helm-app1.yaml

далее удаляем наши applications

![](/news/sidmidru/article-db03824b64600e5b/image-328.png)

настроем теперь репку, так же как и остальные

![](/news/sidmidru/article-db03824b64600e5b/image-329.png)

теперь настроим application

![](/news/sidmidru/article-db03824b64600e5b/image-330.png)

![](/news/sidmidru/article-db03824b64600e5b/image-331.png)

обязательно отмечаемDIRECTORYRECURSE

![](/news/sidmidru/article-db03824b64600e5b/image-332.png)

проверяем что всё ок:

![](/news/sidmidru/article-db03824b64600e5b/image-333.png)

![](/news/sidmidru/article-db03824b64600e5b/image-334.png)

как видим ок - мы запускаем наши applications с помощью другого application

### []Argocd создание проекта. настройка деплоя Helm когда values и chart в разных репозиториях

у нас есть репка где лежит основной helm chart:
https://gitlab.test.local/argo/test-helm/app1.git

а вот в эту репку:
https://gitlab.test.local/argo/test-helm/app2.git

положим только values
helm/values.yaml

|  | replicaCount: 2nameOverride: ""fullnameOverride: "second-app" |
| --- | --- |

добавляем репку

![](/news/sidmidru/article-db03824b64600e5b/image-335.png)

дальше есть проблема так какUIу ArgoCD не умеет добавлять к application несколько репозиториев, мы будем настраивать это через конфиг test123.yaml

| 12345678910111213141516171819202122232425262728293031 | apiVersion: argoproj.io/v1alpha1kind: Applicationmetadata: name: test123 namespace: argocdspec: project: project-dev destination: server: https://kubernetes.default.svc namespace: test123 syncPolicy: automated: prune: true selfHeal: true syncOptions: - CreateNamespace=true sources: # 1) сам Helm‑чарт - repoURL: https://gitlab.test.local/argo/test-helm/app1.git targetRevision: HEAD path: helm-app1 helm: valueFiles: # -- здесь мы ссылаемся на внешний values-файл - $valuesRepo/helm/values.yaml # 2) репо только для values (без `path`) - repoURL: https://gitlab.test.local/argo/test-helm/app2.git targetRevision: HEAD ref: valuesRepo |
| --- | --- |

применим его

**kubectl apply -f test123.yaml**

проверяем:

![](/news/sidmidru/article-db03824b64600e5b/image-336.png)

![](/news/sidmidru/article-db03824b64600e5b/image-337.png)

как видим при такой конфигурации у нас появилась дополнительная вкладкаSOURCESкоторой не было при запуске из панели

![](/news/sidmidru/article-db03824b64600e5b/image-338.png)

по сути всё, готово. у нас есть основной репозиторий в котором находится helm chart и у нас может быть множество других репозиториев в которых может быть только values.

всё разруливается на уровне конфига в sources

|  | sources: # 1) сам Helm‑чарт - repoURL: https://gitlab.test.local/argo/test-helm/app1.git targetRevision: HEAD path: helm-app1 helm: valueFiles: # -- здесь мы ссылаемся на внешний values-файл - $valuesRepo/helm/values.yaml # 2) репо только для values (без `path`) - repoURL: https://gitlab.test.local/argo/test-helm/app2.git targetRevision: HEAD ref: valuesRepo |
| --- | --- |

вот официальная дока:
https://argo-cd.readthedocs.io/en/latest/user-guide/multiple_sources/

### []Keda

кеда позволяет скейлить приложения на основе разных метрик из promethtus или victoria-metrics

офф чарт

https://github.com/kedacore/charts/tree/main/keda

создаём тестовую апку:

**helm create test-app-keda**

**kubectl create ns app1**

**cd ~/keda/example**

**helm upgrade --install -n app1 app1 ./test-app-keda/**

|  | root@kub-master1:~/keda/example# kubectl get pod -n app1 NAME READY STATUS RESTARTS AGEapp1-test-app-keda-65cbf4f4c7-c5zcx 1/1 Running 0 14s |
| --- | --- |

создаём тестовый домен

|  | root@kub-master1:~/keda/example# kubectl get ingress -n app1 NAME CLASS HOSTS ADDRESS PORTS AGEapp1-test-app-keda nginx test.test.local 192.168.1.191 80 69s |
| --- | --- |

применяем файл /etc/ansible/kubespray-official/keda/example/scale-object.yaml

| 123456789101112131415161718192021222324252627 | apiVersion: keda.sh/v1alpha1kind: ScaledObjectmetadata: name: v2-ingress-requests namespace: app1spec: scaleTargetRef: kind: Deployment name: app1-test-app-keda pollingInterval: 15 cooldownPeriod: 30 minReplicaCount: 1 maxReplicaCount: 5 triggers: - type: prometheus metadata: serverAddress: http://vmsingle-vmks-victoria-metrics-k8s-stack.monitoring.svc.cluster.local:8428 metricName: ingress_requests query: >- sum( increase( nginx_ingress_controller_requests{ingress="app1-test-app-keda"}[2m] ) ) threshold: "100" |
| --- | --- |

он будет смотреть в викторию и в случае если за 2 минуты будет больше 100 запросов он будет скейлить апку вверх

для проверки запускаем

root@kub-master1:~/keda/example#**for i in {1..1000}; do curl -Ik test.test.local ; sleep 1; done**

результат такой:

|  | root@kub-master1:~# kubectl get hpa -n app1 -wNAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGEkeda-hpa-v2-ingress-requests Deployment/app1-test-app-keda 0/100 (avg) 1 5 1 13mkeda-hpa-v2-ingress-requests Deployment/app1-test-app-keda 797/100 (avg) 1 5 1 14mkeda-hpa-v2-ingress-requests Deployment/app1-test-app-keda 590500m/100 (avg) 1 5 4 15mroot@kub-master1:~# kubectl get pod -n app1 NAME READY STATUS RESTARTS AGEapp1-test-app-keda-685c6c455c-mw2nf 1/1 Running 0 38sapp1-test-app-keda-685c6c455c-mzn27 1/1 Running 0 38sapp1-test-app-keda-685c6c455c-rgf2h 1/1 Running 0 23sapp1-test-app-keda-685c6c455c-wqhcg 1/1 Running 0 39mapp1-test-app-keda-685c6c455c-wrt7s 1/1 Running 0 38s |
| --- | --- |

сам запрос к victoriametrics может быть любой

как видим всё ок, апка заскейлилась в максимум

если выключим цикл для curl и подождём, то увидим что апка заскейлилась вниз:

|  | keda-hpa-v2-ingress-requests Deployment/app1-test-app-keda 9/100 (avg) 1 5 2 30mkeda-hpa-v2-ingress-requests Deployment/app1-test-app-keda 2/100 (avg) 1 5 2 30mkeda-hpa-v2-ingress-requests Deployment/app1-test-app-keda 0/100 (avg) 1 5 2 31mkeda-hpa-v2-ingress-requests Deployment/app1-test-app-keda 0/100 (avg) 1 5 2 34mkeda-hpa-v2-ingress-requests Deployment/app1-test-app-keda 0/100 (avg) 1 5 1 34m |
| --- | --- |

### []Patrony

это кластер для бд postgresql

позволяет переключать мастер реплика на лету. будем использовать ansible role

https://github.com/midnight47/ansible-playbook/tree/master/roles/patroni

Протестировано на серверах debian

схема такая нужны 3 ноды для etcd базы будет 2, они будут располагаться на тех же нодах что и etcd, на них же будет haproxy который будет отправлять весь трафик на мастер ноду, реплика не будет получать трафик. так же для удобства настроен keepalived с виртуальным ip на который мы будем обращаться.

нам нужно будет настроить следующие переменные:

/etc/ansible/roles/patroni/defaults/main.yml

| 12345678910111213141516171819202122232425262728293031 | ---# Default variables for Patroni rolepatroni_cluster_name: "my_patroni_cluster" replication_password: replication_passwordsuperuser_password: superuser_passwordhaproxy_port: 5432# PostgreSQL listener portpostgresql_port: 5430# Subnet for replication in pg_hbapg_hba_subnet: 192.168.1.0/24# Patroni REST API portpatroni_restapi_port: 8008# Watchdog mode: off | automatic | requiredwatchdog_mode: "off"# Major version of PostgreSQLpostgresql_major_version: 17# Virtual IP managed by Keepalived / HAProxy binds herevirtual_ip: 192.168.1.155# Keepalived instance ID per nodekeepalived_id: 1 |
| --- | --- |

тут мы можем задать порты для postgresql/haproxy/patroni
подсеть в которой будет расположен кластер
версия postgres
виртуальныйIP

/etc/ansible/roles/patroni/handlers/main.yml

|  | ---- name: restart ntpd service: name=ntpd state=restarted enabled=yes- name: restart keepalived service: name=keepalived state=restarted enabled=yes- name: restart haproxy service: name=haproxy state=restarted enabled=yes- name: restart etcd service: name=etcd state=restarted enabled=yes- name: Restart Patroni become: true service: name: patroni state: restarted enabled: yes |
| --- | --- |

/etc/ansible/roles/patroni/tasks/install_etcd.yml

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465 | - name: Install etcd and dependencies on Debian/Ubuntu apt: name: - etcd-server - etcd-client - gcc - python3-dev - python3-requests - python3-urllib3 state: present update_cache: yes when: ansible_os_family == 'Debian'- name: Install etcd and dependencies on RedHat yum: name: [ 'gcc', 'python-devel', 'etcd', 'python-requests', 'python-urllib3' ] state: present when: ansible_os_family == 'RedHat'################################ configure etcd ################################- name: set etcd node name set_fact: etcd_check: "{{ inventory_hostname }}"- name: ensure etcd configuration directory exists on RedHat file: path: /etc/etcd state: directory owner: root group: root mode: '0755' when: - ansible_os_family == 'RedHat'- name: copy etcd configuration template: src: ../templates/etcd.conf dest: /etc/default/etcd backup: yes notify: - restart etcd register: etcd_conf- name: Перезапустить etcd, если конфиг изменился service: name: etcd state: restarted enabled: yes when: - etcd_conf is changed- name: Wait for ETCD to accept connections wait_for: host: "{{ ansible_host }}" port: 2379 state: started delay: 5 timeout: 60 when: inventory_hostname in groups['etcd'] |
| --- | --- |

/etc/ansible/roles/patroni/tasks/install_haproxy.yml

| 1234567891011121314151617181920212223242526272829303132 | ---# Install HAProxy on both Debian/Ubuntu and RedHat systems. HAProxy is used# as a TCP load balancer for the Patroni-managed PostgreSQL cluster. We# differentiate package managers based on the OS family.- name: Install haproxy on Debian/Ubuntu apt: name: haproxy state: present update_cache: yes notify: - restart haproxy when: ansible_os_family == 'Debian'- name: Install haproxy on RedHat yum: name: haproxy state: present notify: - restart haproxy when: ansible_os_family == 'RedHat'- name: Render HAProxy configuration become: true template: src: haproxy.cfg.j2 dest: /etc/haproxy/haproxy.cfg owner: root group: root mode: '0644' notify: restart haproxy |
| --- | --- |

/etc/ansible/roles/patroni/tasks/install_keepalived.yml

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344 | ---# Install Keepalived, which provides VRRP-based failover between HAProxy instances.- name: Install keepalived on Debian/Ubuntu apt: name: keepalived state: present update_cache: yes notify: - restart keepalived when: ansible_os_family == 'Debian'- name: Install keepalived on RedHat yum: name: keepalived state: present notify: - restart keepalived when: ansible_os_family == 'RedHat'- name: put keepalived service version 2 copy: src: ../templates/keepalived dest: /usr/sbin/ backup: yes # Only overwrite the keepalived binary on RedHat-based systems. when: ansible_os_family == 'RedHat'- name: assign keepalived role # Determine whether the current host should act as MASTER or BACKUP. # The first host in the `patroni` group is elected MASTER; all others become BACKUP. set_fact: keepalived_check: "{{ 'MASTER' if inventory_hostname == groups['patroni'][0] else 'BACKUP' }}" keepalived_priority_check: "{{ 100 if inventory_hostname == groups['patroni'][0] else 90 }}"- name: put keepalived_config template: src: ../templates/keepalived.conf dest: /etc/keepalived/keepalived.conf backup: yes notify: - restart keepalived |
| --- | --- |

/etc/ansible/roles/patroni/tasks/install_patroni.yml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126 | - name: install pip on Debian/Ubuntu apt: name: python3-pip state: present update_cache: yes environment: # Разрешаем pip «ломать» системные пакеты (PEP 668) PIP_BREAK_SYSTEM_PACKAGES: '1' when: ansible_os_family == 'Debian'- name: install pip on RedHat yum: name: python-pip state: present environment: PIP_BREAK_SYSTEM_PACKAGES: '1' when: ansible_os_family == 'RedHat'- name: set pip executable fact set_fact: pip_executable: "{{ 'pip' if ansible_os_family == 'RedHat' else 'pip3' }}"- name: upgrade pip to latest pip: name: pip executable: "{{ pip_executable }}" state: latest environment: PIP_BREAK_SYSTEM_PACKAGES: '1'- name: install setuptools pip: name: setuptools executable: "{{ pip_executable }}" environment: PIP_BREAK_SYSTEM_PACKAGES: '1'- name: upgrade setuptools pip: name: setuptools extra_args: --upgrade executable: "{{ pip_executable }}" environment: PIP_BREAK_SYSTEM_PACKAGES: '1'- name: install python packages flake8 pip: name: "flake8" executable: "{{ pip_executable }}" environment: PIP_BREAK_SYSTEM_PACKAGES: '1'- name: install python packages psycopg2-binary pip: name: "psycopg2-binary" executable: "{{ pip_executable }}" environment: PIP_BREAK_SYSTEM_PACKAGES: '1'- name: install python packages urllib3 patroni[etcd] pip: name: - 'urllib3>=1.19.1' - 'patroni[etcd]' - 'cryptography' - 'certifi' - 'idna' - 'pyOpenSSL' executable: "{{ pip_executable }}" environment: PIP_BREAK_SYSTEM_PACKAGES: '1'- name: Render Patroni configuration become: true template: src: patroni.yml.j2 dest: /etc/patroni.yml owner: postgres group: postgres mode: '0644'- name: put patroni.service systemd unit template: src: ../templates/patroni.service dest: /etc/systemd/system/patroni.service backup: yes register: patroni_service- name: Reload daemon definitions command: /usr/bin/systemctl daemon-reload tags: patroni when: - patroni_service is changed- name: Start Patroni on primary node service: name: patroni state: started enabled: yes when: inventory_hostname == groups['patroni'][0]- name: Wait for primary Postgres to accept connections wait_for: host: "{{ ansible_host }}" port: "{{ postgresql_port }}" state: started delay: 5 timeout: 60 when: inventory_hostname == groups['patroni'][0]# 2) Then start Patroni on the replica nodes- name: Start Patroni on replica nodes service: name: patroni state: started enabled: yes when: inventory_hostname != groups['patroni'][0]# 3) Finally, ensure Patroni is running everywhere- name: Ensure Patroni is running service: name: patroni state: started enabled: yes when: inventory_hostname in groups['patroni'] |
| --- | --- |

/etc/ansible/roles/patroni/tasks/install_postgres.yml

| 12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273747576 | - name: install PGDG repository on RedHat/CentOS yum: name: https://download.postgresql.org/pub/repos/yum/pgdg-redhat-repo-latest.noarch.rpm state: present when: ansible_os_family == 'RedHat'- name: Install PostgreSQL {{ postgresql_major_version }} and dependencies on RedHat/CentOS" yum: name: "{{ packages }}" state: present vars: packages: - "postgresql{{ postgresql_major_version }}" - "postgresql{{ postgresql_major_version }}-contrib" - "postgresql{{ postgresql_major_version }}-server" - python-psycopg2 - "repmgr{{ postgresql_major_version }}" when: ansible_os_family == 'RedHat'- name: Stop and disable default PostgreSQL on RedHat/CentOS systemd: name: postgresql-{{ postgresql_major_version }} state: stopped enabled: no masked: yes # чтобы PostgreSQL гарантированно не запустился сам when: ansible_os_family == "RedHat"########################################################## debian ##########################################################- name: Install repository prerequisites on Debian/Ubuntu apt: name: - curl - ca-certificates - gnupg - lsb-release state: present update_cache: yes when: ansible_os_family == 'Debian'- name: Add PostgreSQL apt repository key on Debian/Ubuntu apt_key: url: https://www.postgresql.org/media/keys/ACCC4CF8.asc state: present when: ansible_os_family == 'Debian'- name: Add PostgreSQL apt repository on Debian/Ubuntu apt_repository: repo: "deb https://apt.postgresql.org/pub/repos/apt {{ ansible_distribution_release }}-pgdg main" state: present filename: "pgdg" when: ansible_os_family == 'Debian'- name: Update apt cache after adding PGDG repository on Debian/Ubuntu apt: update_cache: yes when: ansible_os_family == 'Debian'- name: Install PostgreSQL {{ postgresql_major_version }} on Debian/Ubuntu apt: name: - "postgresql-{{ postgresql_major_version }}" - "postgresql-client-{{ postgresql_major_version }}" - postgresql-contrib - python3-psycopg2 state: present when: ansible_os_family == 'Debian'- name: Stop and disable default PostgreSQL (Debian/Ubuntu) systemd: name: postgresql state: stopped enabled: no masked: yes # чтобы PostgreSQL гарантированно не запустился сам when: ansible_os_family == "Debian" |
| --- | --- |

/etc/ansible/roles/patroni/tasks/ntp.yml

| 12345678910111213141516171819202122 | - name: update time command: "ntpdate 0.debian.pool.ntp.org" when: ansible_distribution in ['Debian', 'Ubuntu']- name: ntpd stop service: name: ntpd state: stopped enabled: yes when: ansible_distribution in ['RedHat', 'CentOS']- name: update time command: "ntpd -gq" when: ansible_distribution in ['RedHat', 'CentOS']- name: ntpd start service: name: ntpd state: started enabled: yes when: ansible_distribution in ['RedHat', 'CentOS'] |
| --- | --- |

/etc/ansible/roles/patroni/tasks/main.yml

| 1234567891011121314151617181920212223242526272829 | ---- import_tasks: ntp.yml tags: update time - import_tasks: install_etcd.yml tags: install etcd package when: inventory_hostname in groups['etcd']- import_tasks: install_postgres.yml tags: install postgres package when: - inventory_hostname in groups['patroni']- import_tasks: install_patroni.yml tags: install patroni when: - inventory_hostname in groups['patroni']- import_tasks: install_haproxy.yml tags: install haproxy when: inventory_hostname in groups['patroni']- import_tasks: install_keepalived.yml tags: install keepalived when: inventory_hostname in groups['patroni'] |
| --- | --- |

/etc/ansible/roles/patroni/templates/etcd.conf

|  | ETCD_NAME="{{ etcd_check }}"ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"ETCD_ADVERTISE_CLIENT_URLS="http://{{ ansible_default_ipv4.address }}:2379"ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"ETCD_INITIAL_ADVERTISE_PEER_URLS="http://{{ ansible_default_ipv4.address }}:2380"ETCD_INITIAL_CLUSTER_TOKEN="etcdPatroniCluster"ETCD_INITIAL_CLUSTER="{% for host in groups['etcd'] | sort %}{{ host }}=http://{{ hostvars[host]['ansible_host'] }}:2380{% if not loop.last %},{% endif %}{% endfor %}"ETCD_INITIAL_CLUSTER_STATE="new"ETCD_DATA_DIR="/var/lib/etcd"ETCD_ELECTION_TIMEOUT="5000"ETCD_HEARTBEAT_INTERVAL="1000"ETCD_ENABLE_V2="true" |
| --- | --- |

/etc/ansible/roles/patroni/templates/haproxy.cfg.j2

| 1234567891011121314151617181920212223242526272829303132333435 | global log /dev/log local0 maxconn 4096 user haproxy group haproxydefaults log global mode tcp option tcplog retries 3 timeout connect 5s timeout client 60s timeout server 60sfrontend postgres bind *:{{ haproxy_port }} default_backend postgres_backendbackend postgres_backend mode tcp balance roundrobin # шлём только строку запроса option httpchk GET /master HTTP/1.1 # отдельно даём заголовок Host http-check send hdr Host localhost\r\n # проверяем, что код 200 http-check expect status 200{% for host in groups['patroni'] %} server {{ host }} {{ hostvars[host].ansible_host }}:{{ postgresql_port }} check port {{ patroni_restapi_port }} inter 3s rise 2 fall 2{% endfor %} |
| --- | --- |

/etc/ansible/roles/patroni/templates/keepalived.conf

| 12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849 | ! Configuration File for keepalivedglobal_defs { script_user root script_security on!notification_email {! an@test.ru!}!notification_email_from keepalived@{{ ansible_hostname }}.test.ru! smtp_server 192.168.1.56! smtp_connect_timeout 30}# 1) Проверяем haproxyvrrp_script chk_haproxy { script "/usr/bin/pgrep haproxy" interval 2 fall 2 rise 1 weight 20 # при успехе добавляем +20 к priority}vrrp_instance VI_1 { state {{ keepalived_check }} # Use the primary network interface detected by Ansible. This avoids # hard‑coding interface names like enp0s3 and makes the configuration # portable across different distributions and cloud providers. interface {{ ansible_default_ipv4.interface }} garp_master_refresh 15 virtual_router_id {{ keepalived_id }} priority {{ keepalived_priority_check }} advert_int 5 smtp_alert authentication { auth_type PASS auth_pass 111werfgfgqwer3gfh567 } virtual_ipaddress { {{ virtual_ip }} dev {{ ansible_default_ipv4.interface }} label {{ ansible_default_ipv4.interface }}:vip } track_script { chk_haproxy }} |
| --- | --- |

/etc/ansible/roles/patroni/templates/patroni.service

| 1234567891011121314151617 | [Unit]Description=Patroni PostgreSQL High-AvailabilityAfter=syslog.target network.target[Service]Type=simpleUser=postgres Group=postgres ExecStart=/usr/local/bin/patroni /etc/patroni.ymlExecReload=/bin/kill -s HUP $MAINPIDKillMode=processTimeoutSec=30Restart=on-failure[Install]WantedBy=multi-user.target |
| --- | --- |

/etc/ansible/roles/patroni/templates/patroni.yml.j2

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374757677787980 | ---scope: "{{ patroni_cluster_name }}"name: "{{ inventory_hostname }}"watchdog: mode: "{{ watchdog_mode }}"restapi: listen: "0.0.0.0:{{ patroni_restapi_port }}" connect_address: "{{ ansible_host }}:{{ patroni_restapi_port }}"etcd: hosts:{% for host in groups['etcd'] | sort %} - "{{ hostvars[host]['ansible_host'] | default(host) }}:2379"{% endfor %} protocol: http use_proxies: falsebootstrap: dcs: ttl: 30 loop_wait: 10 retry_timeout: 10 maximum_lag_on_failover: 1048576 postgresql: use_pg_rewind: true use_slots: true parameters: archive_mode: "on" wal_level: hot_standby max_wal_senders: 10 wal_keep_segments: 8 archive_timeout: "1800s" max_replication_slots: 5 hot_standby: "on" wal_log_hints: "on" initdb: - encoding: UTF8 - data-checksumspg_hba: - local all postgres peer - host replication repl {{ pg_hba_subnet }} md5 - host replication repl 127.0.0.1/32 trust - host all all 0.0.0.0/0 md5authentication: replication: username: repl password: "{{ replication_password }}" superuser: username: postgres password: "{{ superuser_password }}"postgresql: pgpass: /var/lib/postgresql/{{ postgresql_major_version }}/.pgpass listen: "0.0.0.0:{{ postgresql_port }}" connect_address: "{{ ansible_host }}:{{ postgresql_port }}" data_dir: "/var/lib/postgresql/{{ postgresql_major_version }}/data/" bin_dir: "/usr/lib/postgresql/{{ postgresql_major_version }}/bin/" create_replica_methods: - basebackup use_unix_socket: true parameters: unix_socket_directories: '/var/run/postgresql' pg_rewind: username: postgres password: "{{ superuser_password }}" pg_hba: - local all postgres peer - host replication repl {{ pg_hba_subnet }} md5 - host replication repl 127.0.0.1/32 trust - host all all 0.0.0.0/0 md5 replication: username: repl password: "{{ replication_password }}" superuser: username: postgres password: "{{ superuser_password }}" |
| --- | --- |

/etc/ansible/playbooks/roles_play/patroni.yml

|  | - name: Deploy highly available PostgreSQL cluster hosts: patronicluster become: yes roles: - patroni |
| --- | --- |

/etc/ansible/hosts

|  | [patronicluster:children]etcdpatroni[etcd]etcd1 ansible_host=192.168.1.152etcd2 ansible_host=192.168.1.153etcd3 ansible_host=192.168.1.154[patroni]db1 ansible_host=192.168.1.152db2 ansible_host=192.168.1.153# db3 ansible_host=10.0.0.13 # если добавите ещё реплики |
| --- | --- |

установку запускаем так:

root@ansible:/etc/ansible#**ansible-playbook playbooks/roles_play/patroni.yml --ask-pass**

проверить что etcd кластер запущен

**etcdctl member list**

проверить что патрони запущен:

**patronictl -c /etc/patroni.yml list**

ответ должен быть такой:

|  | root@debian:~# patronictl -c /etc/patroni.yml list+ Cluster: my_patroni_cluster (7534615147125032841) +----+-----------+| Member | Host | Role | State | TL | Lag in MB |+--------+--------------------+---------+-----------+----+-----------+| db1 | 192.168.1.152:5430 | Leader | running | 9 | || db2 | 192.168.1.153:5430 | Replica | streaming | 9 | 0 |+--------+--------------------+---------+-----------+----+-----------+ |
| --- | --- |

если нужно сменить лидера:

|  | patronictl -c /etc/patroni.yml switchover \ --leader db2 \ --candidate db1 \ --force |
| --- | --- |

в интерактивном режиме команда

patronictl -c /etc/patroni.yml failover

он там сам предложит варианты

проверить подключение:

**psql -h 192.168.1.155 -p 5432 -U postgres -c "SELECTpg_is_in_recovery();"**

(ip виртуальный а пароль тот который задан в superuser_password)

посмотреть логи:

**journalctl -u patroni.service -b --no-pager | tail -n50**

в целом всё, можно подключаться по виртуальному ip**192.168.1.155**haproxy сам найдёт кто мастер и будет слать трафик туда.

### []Helm-chart

**Helm**- это менеджер пакетов для Kubernetes. Этот инструмент позволяет нам обернуть Kubernetes приложения в удобные пакеты, называемые чартами, которые можно легко развертывать, обновлять и управлять ими в любой момент времени.

**Чарты**– это пакеты, которые могут включать в себя все для запуска приложения в Kubernetes, от deployments до services. Все это дает возможность работать с приложениями как с единой сущностью, а не как с набором отдельных ресурсов, которые еще и в ручную нужно настраивать…

Так же Helm**упрощает управление зависимостями**между приложениями, позволяет легко параметризировать настройки приложений через файлы values.yaml и дает возможность повторного использования чартов с помощью шаблонизации.

### []Создание дефолтного чарта

root@kub-master1:~#**helm create common-chart**

**kubectl create ns ap**p

root@kub-master1:~/helm-charts/1_default#**helm upgrade --install app -n app ./common-chart/ --values ./common-chart/values.yaml**

|  | root@kub-master1:~/helm-charts/1_default# kubectl get pod -n appNAME READY STATUS RESTARTS AGEapp-common-chart-749cf6bc54-h5jnn 1/1 Running 0 8sroot@kub-master1:~/helm-charts/1_default# helm list -n appNAME NAMESPACE REVISION UPDATED STATUS CHART APP VERSIONapp app 1 2025-08-07 12:55:32.737050736 +0600 +06 deployed common-chart-0.1.0 1.16.0 |
| --- | --- |

Чтобы убрать из имени пода суффикс common-chart, нужно переопределить шаблон полного имени ресурса в Helm. В стандартном чарте common-chart это делается через значение fullnameOverride.

Добавьте в ваш**values.yaml**(/etc/ansible/kubespray-official/helm-charts/1_default/common-chart/values.yaml) строку:

**fullnameOverride: app**

Это заставит Helm генерировать имя Deployment (и, соответственно, подов) ровно как <Release.Name>, без добавления имени чарта.

|  | root@kub-master1:~/helm-charts/1_default# helm upgrade --install app -n app ./common-chart/ --values ./common-chart/values.yaml root@kub-master1:~/helm-charts/1_default# kubectl get pod -n appNAME READY STATUS RESTARTS AGEapp-5d588f6bc8-qvwqz 1/1 Running 0 4s |
| --- | --- |

как видим имя сменилось.

### []Добавление секретов из vault

настроим подключение vault secret к нашему чарту

не забудем что у нас должен быть установлен
["Vault Secrets Operator"](#vault-k8s)

ну настроем ещё разок

https://github.com/hashicorp/vault-secrets-operator/blob/main/chart/Chart.yaml

| 1234567891011121314151617181920212223242526272829303132333435 | root@kub-master1:~/vault-autounseal# cat vault-secrets-operator.yamlcontroller: replicas: 2 affinity: podAntiAffinity: requiredDuringSchedulingIgnoredDuringExecution: - labelSelector: matchExpressions: - key: app operator: In values: - vault-secrets-operator topologyKey: "kubernetes.io/hostname" kubeRbacProxy: resources: limits: cpu: 150m memory: 150Mi requests: cpu: 50m memory: 100Mi manager: resources: limits: cpu: 150m memory: 150Mi requests: cpu: 50m memory: 100MidefaultVaultConnection: enabled: true skipTLSVerify: true address: "https://192.168.1.111:8200" |
| --- | --- |

ставим

**helm repo add hashicorp https://helm.releases.hashicorp.com**

**helm upgrade --install --namespace vault vault-secrets-operator hashicorp/vault-secrets-operator --version 0.10.0 -f vault-secrets-operator.yaml**

дальше нам надо настроить интеграцию с нашим vault кластером, который на виртуалках, для этого нам понадобится сертификат поэтому выполняем команду:

**kubectl get cm kube-root-ca.crt -o jsonpath="{['data']['ca\.crt']}"**

получаем наш серт

| 1234567891011121314151617181920 | root@kub-master1:~# kubectl get cm kube-root-ca.crt -o jsonpath="{['data']['ca\.crt']}"-----BEGIN CERTIFICATE-----MIIC/jCCAeagAwIBAgIBADANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwprdWJlcm5ldGVzMB4XDTI1MDUyMTA2NTMwOFoXDTM1MDUxOTA2NTMwOFowFTETMBEGA1UEAxMKa3ViZXJuZXRlczCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAOK8T4rAvr9V4TcXsOfuuT0Jew14EgAekHNHOuqnkb9E2O9dTMfcAZK+jHTJx7NNFtHdJxcVQeoQGDQxLI21PW50cr48Tw24tvpYEWNeJJnMeo59kcn00knegAgIP5EqyTfjFIuozkQWXtT0BE0791nRJt82U6wh35pxN8UJA31in7Qo5Ga/VV9yp7F4yy4ZeLTcBVYssK1vdHWd8mb43pGqZs4vSP16oVr4JLJMImlqTuZQqJs6eNHhuBOR1Sxq5gWeTQlfSmSMRAg5ctmYmgG8QvW6phkHMH5DaBz1ZKlpHy0bGtEDoWqShSrdo08+3GNHATRVh6rFGX7vI6ATOZcCAwEAAaNZMFcwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB/wQFMAMBAf8wHQYDVR0OBBYEFLIi6aXVZDS6deO1oMXEPzX0gc/bMBUGA1UdEQQOMAyCCmt1YmVybmV0ZXMwDQYJKoZIhvcNAQELBQADggEBANVO3epULTmHP09zQxkMR984HWpqYSpCd+zho3fzIpXfWgGbk8Kx6l2sBw3PKQB9MA9y/cE5t8mKTkZcO2sEgtpCLRC+TA3652HrSWyHgbCGPh9+SZvctLkPeZSz8Z22D6Nq1rZ5lf8RXsbm9vCwlVtS/K3LRsuk/mO7ZTLPQ2Fp8pkkQ00pW42Y8BifFSrzPewc6LO9jSxe2UlbthwtoFj02IxKY4PSmcORZ5qAcYz9DbowjzXxmHZOUZTIQaEruVtqObbFy5BJFsfTYhsjlILMas+xb5Oibpx2R+LT6L1jm4kT/GkQoi1PnFr8wAB5Nhs/INC1B94AolonlX/Ai5o=-----END CERTIFICATE----- |
| --- | --- |

создаём сервис аккаунт

**kubectl create serviceaccount vault-auth -n kube-system**

на основе этого сервис аккаунта получаем jwt token устанавливаю время на 100 лет 876000h

**kubectl create token vault-auth -n kube-system --duration 876000h**

|  | root@kub-master1:~# kubectl create token vault-auth -n kube-system --duration 876000heyJhbGciOiJSUzI1NiIsImtpZCI6Im8wVXVCeFB2ZHpscW1IcWpOeUh6M0NHbjRDcW8xOVhicm1XVHF5VUx2blUifQ..X3DXNViu8lumFeEwF13ESyQpXCDGvrK9cVtMd_DIE-EZteeHmfyyQi7D_KO95NpTYLbeLzDhQFV9v9PUbMxBaXtLMkBrJMpcx5txMXYYdQrJHRR6Ft7jXPHdCI73HEPfpbcnzfLoyFloISWzDP8gd62F5WMMGEi18ubIBWLNIn04X8T7bpPSTUdrFOnJPwu5rtvp15yYMLcgrzlGOaE_8AKkkozLuOGcpQL8zH1CIdhz6u7Ed-tjOMPZUmDQPI9tHq2ZZXMcvkaGybSWYMRVeMcGAM6bMqKY4JQG7ZDF6_k-_her_Ci1-baNHmOfe-_O_-lM3j8AE8wki1HAuNQFwA |
| --- | --- |

далее подключаемся к нашему vault который на виртуалках

**ssh 192.168.1.103**

напоминаю root token

hvs.UuG0QJvRRfwUHUTGxDTjFaAd

root@vault1:~#**vault login**

создаём секрет который будем подкидывать:

root@vault1:~#**vault secrets enable -path=test/secret/ kv**

добавляем туда ключ значение:

root@vault1:~#**vault kv put test/secret/app/first-app**password="db-secret-password"

включаем аутентификацию в k8s

root@vault1:~#**vault auth enable kubernetes**

сертификат который получили ранее командой**kubectl get cm kube-root-ca.crt -o jsonpath="{['data']['ca\.crt']}"**, кладём в файл /root/ca.crt

дальше присваиваем переменнойTOKEN_REVIEWER_JWTнаш jwt токен который мы создали выше

root@vault1:~#**exportTOKEN_REVIEWER_JWT=''**

теперь настраиваем auth к кластеру

|  | vault write auth/kubernetes/config \ kubernetes_host="https://192.168.1.112:6443" \ kubernetes_ca_cert=@/root/ca.crt \ token_reviewer_jwt="$TOKEN_REVIEWER_JWT" \ disable_iss_validation="true" |
| --- | --- |

создаём policy "test-policy" на чтениеТОЛЬКОнашего секрета

|  | vault policy write test-policy - <<EOFpath "test/secret/app/first-app" { capabilities = ["read"]}EOF |
| --- | --- |

создаём роль "test-role"

|  | root@kub-master1:~# kubectl get serviceaccounts -n appNAME SECRETS AGEapp 0 16ddefault 0 16d |
| --- | --- |

Роль связывает учетную запись службы Kubernetes(serviceaccaunt), которую назовём app (но лучше использовать уже существующий сервис аккаунт нашего приложения ) в пространстве имен app с политикой Vault, test-policy Токены, возвращенные после аутентификации, действительны в течение 10 минут

На роли должен быть задан audience — с Vault ≥1.21 он обязателен. Значение должно совпадать с aud вJWTсервис-аккаунта, которым логинится pod. Пример обновления роли:

|  | vault write auth/kubernetes/role/test-role \ bound_service_account_names=app \ bound_service_account_namespaces=app \ policies=test-policy \ audience="vault" \ ttl=10m |
| --- | --- |

bound_service_account_names: Имя сервисного аккаунта, которому разрешено аутентифицироваться.

bound_service_account_namespaces: Пространство имён, в котором находится сервисный аккаунт.

Как выбрать audience:

Самый надёжный вариант — выдать pod’у projected токен с нужной audience и поставить её же в роли Vault:

|  | # фрагмент Pod/Deploymentspec: serviceAccountName: app volumes: - name: sa-token projected: sources: - serviceAccountToken: path: token audience: vault # <— то же значение укажете в роли expirationSeconds: 3600 |
| --- | --- |

создаём объект VaultConnection который будет использоваться для подключения к vault во всех неймспейсах

**kubectl apply -f vault-connection.yaml**

|  | apiVersion: secrets.hashicorp.com/v1beta1kind: VaultConnectionmetadata: name: vault-connection namespace: kube-systemspec: address: "https://192.168.1.111:8200" skipTLSVerify: true |
| --- | --- |

**kubectl apply -f vault-auth.yaml
**

|  | apiVersion: secrets.hashicorp.com/v1beta1kind: VaultAuthmetadata: name: vault-auth-test namespace: appspec: vaultConnectionRef: kube-system/vault-connection method: kubernetes mount: kubernetes kubernetes: role: test-role serviceAccount: app audiences: - vault # <-- та же строка должна быть в роли Vault |
| --- | --- |

для подключения к VaultConnection используется запись: kube-system/vault-connection

так же тут указываем

роль созданную в vault test-role и

сервис аккаунт app - который так же должен совпадать с тем что мы указали в vault и с тем что у нас уже создан в k8s

**kubectl apply -f vault-static-secret.yaml**

| 1234567891011121314151617 | apiVersion: secrets.hashicorp.com/v1beta1kind: VaultStaticSecretmetadata: name: app-creds namespace: appspec: vaultAuthRef: vault-auth-test mount: test/secret type: kv-v1 path: app/first-app refreshAfter: 10s destination: create: true overwrite: true name: app-secret |
| --- | --- |

тут мы указываем как будет называться secret и как часто его обновлять.

теперь создаём несколько объектов ClusterRole и ClusterRoleBinding

**kubectl apply -f vault-rbac.yaml**

| 123456789101112131415161718192021222324252627282930313233 | apiVersion: rbac.authorization.k8s.io/v1kind: ClusterRolemetadata: name: vault-token-reviewerrules: - apiGroups: ["authentication.k8s.io"] resources: ["tokenreviews"] verbs: ["create"] - apiGroups: [""] resources: ["configmaps"] verbs: ["get"] - apiGroups: [""] resources: ["secrets"] verbs: ["get"] - apiGroups: [""] resources: ["serviceaccounts"] verbs: ["get"]--- apiVersion: rbac.authorization.k8s.io/v1kind: ClusterRoleBindingmetadata: name: vault-token-reviewer-bindingroleRef: apiGroup: rbac.authorization.k8s.io kind: ClusterRole name: vault-token-reviewersubjects: - kind: ServiceAccount name: vault-auth namespace: kube-system |
| --- | --- |

ClusterRoleBinding смотрит на ServiceAccount vault-auth расположенный kube-system мы его создавали вручную и jwt token создавали на его основе.

проверить что всё ок можно командами

kubectl describe vaultconnections.secrets.hashicorp.com -n kube-system vault-connection

kubectl describe vaultauths.secrets.hashicorp.com -n app vault-auth-test

kubectl describe vaultstaticsecrets.secrets.hashicorp.com -n app app-creds

kubectl get secret -n app app-secret -o yaml

============================

постараемся привести к виду когда не нужно будет каждый раз создавать полиси или роли в vault

accessor Kubernetes-аутентификации

|  | export K8S_ACC=$(vault auth list -format=json | jq -r '."kubernetes/".accessor') |
| --- | --- |

Универсальная политика (KVv1)

политика: разрешает читать только <ns>/service/*

|  | vault policy write read-service-by-namespace - <<EOFpath "{{identity.entity.aliases.${K8S_ACC}.metadata.service_account_namespace}}/service/*" { capabilities = ["read","list"]}EOF |
| --- | --- |

Одна роль на все namespace

Разрешаем любой service account в любом namespace, а ограничение — через политику выше (по подставленному ns):

|  | vault write auth/kubernetes/role/ns-scoped-read \ bound_service_account_names="*" \ bound_service_account_namespaces="*" \ policies="read-service-by-namespace" \ audience="https://kubernetes.default.svc" \ ttl=10m |
| --- | --- |

Если хочется чуть строже — вместо "*" можно перечислить допустимые ns:
bound_service_account_namespaces="dev,prod,stage,pre".

Если когда-нибудь нужно кросс-ns (например, pod в`app`, а секреты из`dev/service/*`) — добавь отдельную политику (`read-dev-service`) и пришьёшь её к этой же роли в`policies=...`

теперь рассмотрим чарт:

/etc/ansible/kubespray-official/helm-charts/2_secret/common-chart/templates/deployment.yaml

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374757677787980818283848586878889909192 | apiVersion: apps/v1kind: Deploymentmetadata: name: {{ include "common-chart.fullname" . }} labels: {{- include "common-chart.labels" . | nindent 4 }} {{- if and .Values.reloader.enabled .Values.vault_secret.enabled .Values.vault_secret.name }} annotations: secret.reloader.stakater.com/reload: "{{ .Values.vault_secret.name }}" {{- end }}spec: {{- if not .Values.autoscaling.enabled }} replicas: {{ .Values.replicaCount }} {{- end }} selector: matchLabels: {{- include "common-chart.selectorLabels" . | nindent 6 }} template: metadata: labels: {{- include "common-chart.selectorLabels" . | nindent 8 }} {{- with .Values.podLabels }} {{- toYaml . | nindent 8 }} {{- end }} {{- with .Values.podAnnotations }} annotations: {{- toYaml . | nindent 8 }} {{- end }} spec: serviceAccountName: {{ include "common-chart.serviceAccountName" . }} securityContext: {{- toYaml .Values.podSecurityContext | nindent 8 }} containers: - name: {{ .Values.container.name }} image: "{{ .Values.image.repository }}{{ if .Values.image.tag }}:{{ .Values.image.tag }}{{ end }}" imagePullPolicy: {{ .Values.image.pullPolicy }} securityContext: {{- toYaml .Values.securityContext | nindent 12 }} {{- if and .Values.vault_secret.enabled .Values.vault_secret.attach.asEnv .Values.vault_secret.name }} envFrom: - secretRef: name: {{ .Values.vault_secret.name }} {{- end }} ports: {{- toYaml .Values.container.ports | nindent 12 }} livenessProbe: {{- toYaml .Values.livenessProbe | nindent 12 }} readinessProbe: {{- toYaml .Values.readinessProbe | nindent 12 }} resources: {{- toYaml .Values.resources | nindent 12 }} {{- $needSecretVol := and .Values.vault_secret.enabled .Values.vault_secret.attach.asVolume .Values.vault_secret.name }} {{- if or $needSecretVol .Values.volumeMounts }} volumeMounts: {{- if $needSecretVol }} - name: vault-secret mountPath: {{ .Values.vault_secret.attach.mountPath }} readOnly: true {{- end }} {{- with .Values.volumeMounts }} {{- toYaml . | nindent 12 }} {{- end }} {{- end }} {{- $needSecretVol := and .Values.vault_secret.enabled .Values.vault_secret.attach.asVolume .Values.vault_secret.name }} {{- if or $needSecretVol .Values.volumes }} volumes: {{- if $needSecretVol }} - name: vault-secret secret: secretName: {{ .Values.vault_secret.name }} {{- end }} {{- with .Values.volumes }} {{- toYaml . | nindent 8 }} {{- end }} {{- end }} {{- with .Values.imagePullSecrets }} imagePullSecrets: {{- toYaml . | nindent 8 }} {{- end }} {{- with .Values.nodeSelector }} nodeSelector: {{- toYaml . | nindent 8 }} {{- end }} {{- with .Values.affinity }} affinity: {{- toYaml . | nindent 8 }} {{- end }} {{- with .Values.tolerations }} tolerations: {{- toYaml . | nindent 8 }} {{- end }} |
| --- | --- |

/etc/ansible/kubespray-official/helm-charts/2_secret/common-chart/templates/vault-secrets-operator-VaultAuth.yaml

| 1234567891011121314151617181920212223 | {{- if .Values.vault_secret.enabled }}apiVersion: secrets.hashicorp.com/v1beta1kind: VaultAuthmetadata: name: "{{ include "common-chart.fullname" . }}-va" labels: {{- include "common-chart.labels" . | nindent 4 }}spec: vaultConnectionRef: {{ .Values.vault_secret.vaultConnectionRef }} method: kubernetes mount: {{ .Values.vault_secret.authMount }} kubernetes: role: {{ default "ns-scoped-read" .Values.vault_secret.role }} serviceAccount: {{ include "common-chart.serviceAccountName" . }} {{- if .Values.vault_secret.audiences }} audiences: {{- toYaml .Values.vault_secret.audiences | nindent 6 }} {{- else }} audiences: - "https://kubernetes.default.svc" {{- end }}{{- end }} |
| --- | --- |

/etc/ansible/kubespray-official/helm-charts/2_secret/common-chart/templates/vault-secrets-operator-VaultSecret.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748 | {{- if .Values.vault_secret.enabled }}{{/* Разбор vault_secret.secretFullPath в формате "<ns>/service/<app[/subpath…]>". Принято: mount = "<ns>/service" (первые 2 сегмента), path = остальное. Если хочешь задать mount/path явно — можно указать .Values.vault_secret.mount и .Values.vault_secret.path, тогда они переопределят вычисленные значения.*/}}{{- $full := required "values.vault_secret.secretFullPath is required (e.g. 'dev/service/app1')" .Values.vault_secret.secretFullPath -}}{{- $parts := splitList "/" $full -}}{{- $plen := len $parts -}}{{- if lt $plen 3 -}} {{- fail (printf "vault_secret.secretFullPath must have at least 3 segments like 'dev/service/app1', got '%s'" $full) -}}{{- end -}}{{- $computedMount := printf "%s/%s" (index $parts 0) (index $parts 1) -}}{{- $computedPath := join "/" (slice $parts 2 $plen) -}}{{- $mount := default $computedMount .Values.vault_secret.mount -}}{{- $path := default $computedPath .Values.vault_secret.path -}}apiVersion: secrets.hashicorp.com/v1beta1kind: VaultStaticSecretmetadata: name: "{{ include "common-chart.fullname" . }}-vss" labels: {{- include "common-chart.labels" . | nindent 4 }}spec: vaultAuthRef: "{{ include "common-chart.fullname" . }}-va" mount: {{ $mount }} type: {{ .Values.vault_secret.type }} path: {{ $path }} refreshAfter: {{ .Values.vault_secret.refreshAfter }} destination: create: true overwrite: true name: {{ required "values.vault_secret.name is required (k8s Secret name)" .Values.vault_secret.name }}{{- if .Values.vault_secret.rolloutRestart.enabled }} rolloutRestartTargets:{{- range .Values.vault_secret.rolloutRestart.targets }} - kind: {{ .kind }} name: {{ tpl .name $ }}{{- end }}{{- end }}{{- end }} |
| --- | --- |

/etc/ansible/kubespray-official/helm-charts/2_secret/common-chart/values.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117 | replicaCount: 1image: repository: nginx pullPolicy: IfNotPresent tag: ""imagePullSecrets: []nameOverride: ""fullnameOverride: "app"serviceAccount: create: true automount: true annotations: {} name: ""podAnnotations: {}podLabels: {}podSecurityContext: {}securityContext: {}service: type: ClusterIP port: 80ingress: enabled: false className: "" annotations: {} hosts: - host: chart-example.local paths: - path: / pathType: ImplementationSpecific tls: []resources: {}livenessProbe: httpGet: path: / port: httpreadinessProbe: httpGet: path: / port: httpautoscaling: enabled: false minReplicas: 1 maxReplicas: 100 targetCPUUtilizationPercentage: 80 # targetMemoryUtilizationPercentage: 80# Дополнительные (пользовательские) тома/маунты, если нужныvolumes: []volumeMounts: []nodeSelector: {}tolerations: []affinity: {}container: name: app ports: - name: http containerPort: 80 protocol: TCP# ===================== VAULT + VSO =====================# Минимум, что нужно заполнить: secret.name и vault.secretFullPath.vault_secret: enabled: true # Имя k8s Secret, куда VSO запишет данные, и который подключим в Pod name: "app1-secret" # <— ТЫ МЕНЯЕШЬ # Полный путь секрета в Vault вида "<mount>/<…>/<leaf>" # примеры: "dev/service/app1" или "prod/service/app1" secretFullPath: "app/service/app1" # <— ТЫ МЕНЯЕШЬ # Тип KV в Vault: kv-v1 или kv-v2 type: "kv-v1" # Ссылка на VaultConnection (ns/name или просто name, если в том же ns) vaultConnectionRef: "kube-system/vault-connection" # Точка монтирования метода аутентификации в Vault authMount: "kubernetes" # Роль в Vault. По умолчанию используем универсальную роль для всех ns # (см. команды ниже) — ns-scoped-read role: "ns-scoped-read" # Как часто перечитывать секрет refreshAfter: "30s" audiences: - "https://kubernetes.default.svc" # Подключение секрета в Pod attach: asEnv: true # добавить envFrom: secretRef asVolume: false # смонтировать как том mountPath: /etc/app/secret # Авто-рестарт Deployment при изменении секрета через VSO rolloutRestart: enabled: true targets: - kind: Deployment name: "{{ include \"common-chart.fullname\" . }}"# Если хочешь вместо rolloutRestartTargets — stakater/reloaderreloader: enabled: false |
| --- | --- |

минимально что нужно добавить в values:

|  | vault_secret: enabled: true name: app2-secret # имя k8s Secret, который получит VSO secretFullPath: app/service/app2 # путь в Vault: <ns>/service/<app> type: kv-v1 # у меня сейчас kv-v1 role: ns-scoped-read # универсальная роль в vault для всех ns audiences: - "https://kubernetes.default.svc" |
| --- | --- |

Этого достаточно, чтобы:

- 

VSOзалогинился в Vault с нужной`aud`,

- 

прочитал`app/service/app2`,

- 

создал/обновлял Secret`app2-secret`,

- 

и «пнул» Deployment на рестарт при изменении.

Примечание: если когда-нибудь перейдёшь наKVv2 с (например, kv/…), тогда укажи type: kv-v2 и либо:
задай vault_secret.mount и vault_secret.path явно либо подготовь secretFullPath так, как ожидает шаблон (первые два сегмента).

для проверки создадим ещё секрет:

![](/news/sidmidru/article-db03824b64600e5b/image-339.png)

и запустим эту же апку но другим релизом, вот мой минимальный values
/etc/ansible/kubespray-official/helm-charts/2_secret/common-chart/values-minimal.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134 | # Default values for common-chart.# This is a YAML-formatted file.# Declare variables to be passed into your templates.# This will set the replicaset count more information can be found here: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/replicaCount: 1# This sets the container image more information can be found here: https://kubernetes.io/docs/concepts/containers/images/image: repository: nginx # This sets the pull policy for images. pullPolicy: IfNotPresent # Overrides the image tag whose default is the chart appVersion. tag: ""# This is for the secretes for pulling an image from a private repository more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/imagePullSecrets: []# This is to override the chart name.nameOverride: ""fullnameOverride: "app-minimal"#This section builds out the service account more information can be found here: https://kubernetes.io/docs/concepts/security/service-accounts/serviceAccount: # Specifies whether a service account should be created create: true # Automatically mount a ServiceAccount's API credentials? automount: true # Annotations to add to the service account annotations: {} # The name of the service account to use. # If not set and create is true, a name is generated using the fullname template name: ""# This is for setting Kubernetes Annotations to a Pod.# For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/ podAnnotations: {}# This is for setting Kubernetes Labels to a Pod.# For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/podLabels: {}podSecurityContext: {} # fsGroup: 2000securityContext: {} # capabilities: # drop: # - ALL # readOnlyRootFilesystem: true # runAsNonRoot: true # runAsUser: 1000# This is for setting up a service more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/service: # This sets the service type more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types type: ClusterIP # This sets the ports more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#field-spec-ports port: 80# This block is for setting up the ingress for more information can be found here: https://kubernetes.io/docs/concepts/services-networking/ingress/ingress: enabled: false className: "" annotations: {} # kubernetes.io/ingress.class: nginx # kubernetes.io/tls-acme: "true" hosts: - host: chart-example.local paths: - path: / pathType: ImplementationSpecific tls: [] # - secretName: chart-example-tls # hosts: # - chart-example.localresources: {} # We usually recommend not to specify default resources and to leave this as a conscious # choice for the user. This also increases chances charts run on environments with little # resources, such as Minikube. If you do want to specify resources, uncomment the following # lines, adjust them as necessary, and remove the curly braces after 'resources:'. # limits: # cpu: 100m # memory: 128Mi # requests: # cpu: 100m # memory: 128Mi# This is to setup the liveness and readiness probes more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/livenessProbe: httpGet: path: / port: httpreadinessProbe: httpGet: path: / port: http#This section is for setting up autoscaling more information can be found here: https://kubernetes.io/docs/concepts/workloads/autoscaling/autoscaling: enabled: false minReplicas: 1 maxReplicas: 100 targetCPUUtilizationPercentage: 80 # targetMemoryUtilizationPercentage: 80# Additional volumes on the output Deployment definition.volumes: []# - name: foo# secret:# secretName: mysecret# optional: false# Additional volumeMounts on the output Deployment definition.volumeMounts: []# - name: foo# mountPath: "/etc/foo"# readOnly: truenodeSelector: {}tolerations: []affinity: {}vault_secret: enabled: true name: app2-secret # имя k8s Secret, который получит VSO secretFullPath: app/service/app2 # путь в Vault: <ns>/service/<app> type: kv-v1 # у меня сейчас kv-v1 role: ns-scoped-read # универсальная роль в vault для всех ns # audiences: # - "https://kubernetes.default.svc" |
| --- | --- |

root@kub-master1:~/helm-charts/2_secret#**helm upgrade --install app2 -n app ./common-chart -f ./common-chart/values-minimal.yaml**

# смотрим VSO-ресурсы
kubectl -n app describe vaultauth app-va
kubectl -n app describe vaultstaticsecret app-vss | egrep 'Mount:|Path:|Type:'

# появился ли секрет и ключи?
kubectl -n app get secret app1-secret -o json | jq -r '.data | keys[]'

# env внутри pod’а
kubectl -n app rollout status deploy/app
kubectl -n app exec deploy/app -- env | egrep '^(test|test1|tttt)='

вот результат:

| 12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455 | root@kub-master1:~/helm-charts/2_secret# kubectl -n app describe vaultauth app-vaName: app-vaNamespace: appLabels: app.kubernetes.io/instance=app app.kubernetes.io/managed-by=Helm app.kubernetes.io/name=common-chart app.kubernetes.io/version=1.16.0 helm.sh/chart=common-chart-0.1.0Annotations: meta.helm.sh/release-name: app meta.helm.sh/release-namespace: appAPI Version: secrets.hashicorp.com/v1beta1Kind: VaultAuthMetadata: Creation Timestamp: 2025-08-23T14:20:28Z Finalizers: vaultauth.secrets.hashicorp.com/finalizer Generation: 2 Resource Version: 22751828 UID: 3b22c2a5-fbe7-4ff7-b24a-dbef00a5e3aeSpec: Kubernetes: Audiences: https://kubernetes.default.svc Role: ns-scoped-read Service Account: app Token Expiration Seconds: 600 Method: kubernetes Mount: kubernetes Vault Connection Ref: kube-system/vault-connectionStatus: Spec Hash: 21ae278d9888a4ed421fed213ce3dfbcbb48f22ee0c43cffd162743b5c07f213 Valid: trueEvents: <none>root@kub-master1:~/helm-charts/2_secret# kubectl -n app describe vaultstaticsecret app-vss | egrep 'Mount:|Path:|Type:' Mount: app/service Path: app1 Type: kv-v1root@kub-master1:~/helm-charts/2_secret# kubectl -n app get secret app1-secret -o json | jq -r '.data | keys[]'_rawgggggggtest1ttttroot@kub-master1:~/helm-charts/2_secret# kubectl -n app rollout status deploy/appdeployment "app" successfully rolled outroot@kub-master1:~/helm-charts/2_secret# kubectl -n app exec deploy/app -- env | egrep '^(test|test1|tttt)='tttt=dddddtest1=dsfsdf |
| --- | --- |

### []Добавление Topology Spread Constraints

**Topology Spread Constraints (TSC)**— это механизм Kubernetes, который заставляет планировщик**равномерно распределять Pod’ы**по доменам отказа (узлам, зонам, регионам и т.п.), чтобы повысить доступность и избежать «скученности».

Что настраивается:

- 

`topologyKey`— по какому домену равномерно раскладывать (например,`topology.kubernetes.io/zone`или`kubernetes.io/hostname`).

- 

`maxSkew`— допустимая «косина», т.е. разница в количестве Pod’ов между самыми заполненным и самым пустым доменами.

- 

`whenUnsatisfiable`— что делать, если условие нельзя соблюсти:

- 

`DoNotSchedule`— лучше не запускать Pod, чем нарушить правило (жёстко).

- 

`ScheduleAnyway`— запустить, даже если распределение получится неровным (мягко).

- 

`labelSelector`— какие Pod’ы считать «своими» для подсчёта баланса.

- 

Дополнительно:`minDomains`(требуемый минимум доступных доменов),`nodeAffinityPolicy`,`nodeTaintsPolicy`.

в spec нужно добавить

|  | spec: topologySpreadConstraints: - maxSkew: 1 topologyKey: topology.kubernetes.io/zone whenUnsatisfiable: DoNotSchedule labelSelector: matchLabels: app: mysvc |
| --- | --- |

в темплейте это будет выглядеть так:

| 1234567891011121314151617181920 | {{- if and .Values.topologySpread.enabled .Values.topologySpread.constraints }} topologySpreadConstraints: {{- range .Values.topologySpread.constraints }} - maxSkew: {{ .maxSkew | default 1 }} topologyKey: {{ .topologyKey | quote }} whenUnsatisfiable: {{ .whenUnsatisfiable | default "DoNotSchedule" | quote }} {{- if hasKey . "minDomains" }} minDomains: {{ .minDomains }} {{- end }} {{- if .nodeAffinityPolicy }} nodeAffinityPolicy: {{ .nodeAffinityPolicy | quote }} {{- end }} {{- if .nodeTaintsPolicy }} nodeTaintsPolicy: {{ .nodeTaintsPolicy | quote }} {{- end }} labelSelector: matchLabels: {{- include "common-chart.selectorLabels" $ | nindent 14 }} {{- end }} {{- end }} |
| --- | --- |

полный template будет таки:

/etc/ansible/kubespray-official/helm-charts/3_topolgy_spread_constraints/common-chart/templates/deployment.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127 | apiVersion: apps/v1kind: Deploymentmetadata: name: {{ include "common-chart.fullname" . }} labels: {{- include "common-chart.labels" . | nindent 4 }}spec: {{- if not .Values.autoscaling.enabled }} replicas: {{ .Values.replicaCount }} {{- end }} selector: matchLabels: {{- include "common-chart.selectorLabels" . | nindent 6 }} template: metadata: # Аннотация reloader добавляется автоматически, если включён vault_secret и указан имя секрета annotations: {{- if and .Values.vault_secret.enabled .Values.vault_secret.name }} secret.reloader.stakater.com/reload: "{{ .Values.vault_secret.name }}" {{- end }} {{- with .Values.podAnnotations }} {{- toYaml . | nindent 8 }} {{- end }} labels: {{- include "common-chart.labels" . | nindent 8 }} {{- with .Values.podLabels }} {{- toYaml . | nindent 8 }} {{- end }} spec: {{- with .Values.imagePullSecrets }} imagePullSecrets: {{- toYaml . | nindent 8 }} {{- end }} serviceAccountName: {{ include "common-chart.serviceAccountName" . }} {{- if .Values.podSecurityContext }} securityContext: {{- toYaml .Values.podSecurityContext | nindent 8 }} {{- else }} securityContext: {} {{- end }} {{/* Настройки подключения секрета: безопасные дефолты (без хомпинга '-') */}} {{ $attach := default (dict) .Values.vault_secret.attach }} {{ $asEnv := default true (get $attach "asEnv") }} {{ $asVol := default false (get $attach "asVolume") }} {{ $mountPath := default "/etc/app/secret" (get $attach "mountPath") }} containers: - name: {{ .Chart.Name }} securityContext: {{- toYaml .Values.securityContext | nindent 12 }} image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}" imagePullPolicy: {{ .Values.image.pullPolicy }} ports: - name: http containerPort: {{ .Values.service.port }} protocol: TCP {{- if and .Values.vault_secret.enabled $asEnv .Values.vault_secret.name }} envFrom: - secretRef: name: {{ .Values.vault_secret.name }} {{- end }} livenessProbe: {{- toYaml .Values.livenessProbe | nindent 12 }} readinessProbe: {{- toYaml .Values.readinessProbe | nindent 12 }} resources: {{- toYaml .Values.resources | nindent 12 }} {{- $needSecretVol := and .Values.vault_secret.enabled $asVol .Values.vault_secret.name }} {{- if or $needSecretVol .Values.volumeMounts }} volumeMounts: {{- if $needSecretVol }} - name: vault-secret mountPath: {{ $mountPath }} readOnly: true {{- end }} {{- with .Values.volumeMounts }} {{- toYaml . | nindent 12 }} {{- end }} {{- end }} {{- $needSecretVol := and .Values.vault_secret.enabled $asVol .Values.vault_secret.name }} {{- if or $needSecretVol .Values.volumes }} volumes: {{- if $needSecretVol }} - name: vault-secret secret: secretName: {{ .Values.vault_secret.name }} {{- end }} {{- with .Values.volumes }} {{- toYaml . | nindent 8 }} {{- end }} {{- end }} {{- with .Values.nodeSelector }} nodeSelector: {{- toYaml . | nindent 8 }} {{- end }} {{- with .Values.affinity }} affinity: {{- toYaml . | nindent 8 }} {{- end }} {{- if and .Values.topologySpread.enabled .Values.topologySpread.constraints }} topologySpreadConstraints: {{- range .Values.topologySpread.constraints }} - maxSkew: {{ .maxSkew | default 1 }} topologyKey: {{ .topologyKey | quote }} whenUnsatisfiable: {{ .whenUnsatisfiable | default "DoNotSchedule" | quote }} {{- if hasKey . "minDomains" }} minDomains: {{ .minDomains }} {{- end }} labelSelector: matchLabels: {{- include "common-chart.selectorLabels" $ | nindent 14 }} {{- end }} {{- end }} {{- with .Values.tolerations }} tolerations: {{- toYaml . | nindent 8 }} {{- end }} |
| --- | --- |

я ещё поправил:

/etc/ansible/kubespray-official/helm-charts/3_topolgy_spread_constraints/common-chart/templates/vault-secrets-operator-VaultAuth.yaml

| 12345678910111213141516171819202122 | {{- if .Values.vault_secret.enabled }}apiVersion: secrets.hashicorp.com/v1beta1kind: VaultAuthmetadata: name: "{{ include "common-chart.fullname" . }}-va" labels: {{- include "common-chart.labels" . | nindent 4 }}spec: vaultConnectionRef: {{ .Values.vault_secret.vaultConnectionRef | default "kube-system/vault-connection" }} method: kubernetes mount: {{ .Values.vault_secret.authMount | default "kubernetes" }} kubernetes: role: {{ .Values.vault_secret.role | default "ns-scoped-read" }} serviceAccount: {{ include "common-chart.serviceAccountName" . }} {{- /* безопасно формируем audiences: берём из values, иначе дефолт */ -}} {{- $aud := default (list "https://kubernetes.default.svc") (get .Values.vault_secret "audiences") }} audiences: {{- range $aud }} - {{ . | quote }} {{- end }}{{- end }} |
| --- | --- |

/etc/ansible/kubespray-official/helm-charts/3_topolgy_spread_constraints/common-chart/templates/vault-secrets-operator-VaultSecret.yaml

| 12345678910111213141516171819202122232425262728293031323334 | {{- if .Values.vault_secret.enabled }}{{/* Требуемый full path: "<ns>/service/<leaf>" */}}{{- $full := required "values.vault_secret.secretFullPath is required (e.g. 'dev/service/app1')" .Values.vault_secret.secretFullPath -}}{{- $parts := splitList "/" $full -}}{{- $plen := len $parts -}}{{- if lt $plen 3 -}} {{- fail (printf "vault_secret.secretFullPath must have at least 3 segments like 'dev/service/app1', got '%s'" $full) -}}{{- end -}}{{- $computedMount := printf "%s/%s" (index $parts 0) (index $parts 1) -}}{{- $computedPath := join "/" (slice $parts 2 $plen) -}}{{- $mount := default $computedMount .Values.vault_secret.mount -}}{{- $path := default $computedPath .Values.vault_secret.path -}}apiVersion: secrets.hashicorp.com/v1beta1kind: VaultStaticSecretmetadata: name: "{{ include "common-chart.fullname" . }}-vss" labels: {{- include "common-chart.labels" . | nindent 4 }}spec: vaultAuthRef: "{{ include "common-chart.fullname" . }}-va" mount: {{ $mount }} type: {{ .Values.vault_secret.type | default "kv-v1" }} path: {{ $path }} refreshAfter: {{ .Values.vault_secret.refreshAfter | default "30s" }} destination: create: true overwrite: true name: {{ required "values.vault_secret.name is required (k8s Secret name)" .Values.vault_secret.name }}{{- end }} |
| --- | --- |

минимальный values

|  | topologySpread: enabled: true constraints: - topologyKey: "kubernetes.io/hostname" # по серверам maxSkew: 1 whenUnsatisfiable: "DoNotSchedule" |
| --- | --- |

**range**— это цикл, в шаблоне он перебирает массив topologySpread.constraints и на лету рендерит 0…N объектов topologySpreadConstraints.
Без range был бы жёстко зашит ровно один constraint

Что даёт range здесь:

Можно указать несколько правил одновременно (например, «ровно по зонам» + «желательно по нодам»).

Можно иметь разные наборы правил для dev/prod просто через разные values — шаблон не трогаем.

Можно легко отключить блок (пустой список → ничего не срендерится).

Внутри range текущий элемент — это «локальный» . (один constraint), а $ — ссылка на верхний контекст чарта. Поэтому для labelSelector мы делали {{ include "common-chart.selectorLabels" $ }}, чтобы взять метки из верхнего контекста, а не из элемента.

**Примеры values и что получится**

1) Один constraint: равномерно по зонам (жёстко)

|  | topologySpread: enabled: true constraints: - topologyKey: topology.kubernetes.io/zone maxSkew: 1 whenUnsatisfiable: DoNotSchedule |
| --- | --- |

Результат (фрагмент PodSpec):

|  | topologySpread: enabled: true constraints: - topologyKey: topology.kubernetes.io/zone maxSkew: 1 whenUnsatisfiable: DoNotSchedule - topologyKey: kubernetes.io/hostname maxSkew: 1 whenUnsatisfiable: ScheduleAnyway |
| --- | --- |

Результат:

общий values получается таким:

/etc/ansible/kubespray-official/helm-charts/3_topolgy_spread_constraints/common-chart/values.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143 | # Default values for common-chart.# This is a YAML-formatted file.# Declare variables to be passed into your templates.# This will set the replicaset count more information can be found here: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/replicaCount: 1# This sets the container image more information can be found here: https://kubernetes.io/docs/concepts/containers/images/image: repository: nginx # This sets the pull policy for images. pullPolicy: IfNotPresent # Overrides the image tag whose default is the chart appVersion. tag: ""# This is for the secretes for pulling an image from a private repository more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/imagePullSecrets: []# This is to override the chart name.nameOverride: ""fullnameOverride: "app"#This section builds out the service account more information can be found here: https://kubernetes.io/docs/concepts/security/service-accounts/serviceAccount: # Specifies whether a service account should be created create: true # Automatically mount a ServiceAccount's API credentials? automount: true # Annotations to add to the service account annotations: {} # The name of the service account to use. # If not set and create is true, a name is generated using the fullname template name: ""# This is for setting Kubernetes Annotations to a Pod.# For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/ podAnnotations: {}# This is for setting Kubernetes Labels to a Pod.# For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/podLabels: {}podSecurityContext: {} # fsGroup: 2000securityContext: {} # capabilities: # drop: # - ALL # readOnlyRootFilesystem: true # runAsNonRoot: true # runAsUser: 1000# This is for setting up a service more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/service: # This sets the service type more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types type: ClusterIP # This sets the ports more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#field-spec-ports port: 80# This block is for setting up the ingress for more information can be found here: https://kubernetes.io/docs/concepts/services-networking/ingress/ingress: enabled: false className: "" annotations: {} # kubernetes.io/ingress.class: nginx # kubernetes.io/tls-acme: "true" hosts: - host: chart-example.local paths: - path: / pathType: ImplementationSpecific tls: [] # - secretName: chart-example-tls # hosts: # - chart-example.localresources: {} # We usually recommend not to specify default resources and to leave this as a conscious # choice for the user. This also increases chances charts run on environments with little # resources, such as Minikube. If you do want to specify resources, uncomment the following # lines, adjust them as necessary, and remove the curly braces after 'resources:'. # limits: # cpu: 100m # memory: 128Mi # requests: # cpu: 100m # memory: 128Mi# This is to setup the liveness and readiness probes more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/livenessProbe: httpGet: path: / port: httpreadinessProbe: httpGet: path: / port: http#This section is for setting up autoscaling more information can be found here: https://kubernetes.io/docs/concepts/workloads/autoscaling/autoscaling: enabled: false minReplicas: 1 maxReplicas: 100 targetCPUUtilizationPercentage: 80 # targetMemoryUtilizationPercentage: 80# Additional volumes on the output Deployment definition.volumes: []# - name: foo# secret:# secretName: mysecret# optional: false# Additional volumeMounts on the output Deployment definition.volumeMounts: []# - name: foo# mountPath: "/etc/foo"# readOnly: truenodeSelector: {}tolerations: []affinity: {}vault_secret: enabled: true name: app-secret # имя k8s Secret, который получит VSO secretFullPath: app/service/app1 # путь в Vault: <ns>/service/<app> type: kv-v1 # у меня сейчас kv-v1 role: ns-scoped-read # универсальная роль в vault для всех ns # audiences: # - "https://kubernetes.default.svc"topologySpread: enabled: true constraints: - topologyKey: "kubernetes.io/hostname" # по серверам maxSkew: 1 whenUnsatisfiable: "DoNotSchedule" |
| --- | --- |

ставим

root@kub-master1:~/helm-charts/3_topolgy_spread_constraints# helm upgrade --install app -n app ./common-chart -f ./common-chart/values.yaml

если увеличим количество реплик то получим следующую картину:

|  | root@kub-master1:~/helm-charts/3_topolgy_spread_constraints# kubectl get pod -n app -o wideNAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATESapp-557cc644d8-5hwpf 1/1 Running 0 17m 10.233.79.11 kub-worker3.test.local <none> <none>app-557cc644d8-6twmc 0/1 Pending 0 28s <none> <none> <none> <none>app-557cc644d8-bqbw2 0/1 Pending 0 28s <none> <none> <none> <none>app-557cc644d8-btb7n 0/1 Pending 0 28s <none> <none> <none> <none>app-557cc644d8-gfqdx 1/1 Running 0 28s 10.233.107.171 kub-worker2.test.local <none> <none>app-557cc644d8-rxhxv 0/1 Pending 0 28s <none> <none> <none> <none>app-557cc644d8-ssmpm 1/1 Running 0 28s 10.233.67.227 kub-worker1.test.local <none> <none> |
| --- | --- |

вот по этой причине

Warning FailedScheduling 45s (x2 over 47s) default-scheduler 0/6 nodes are available: 3 node(s) didn't match pod topology spread constraints, 3 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }. preemption: 0/6 nodes are available: 3 No preemption victims found for incoming pod, 3 Preemption is not helpful for scheduling.

но если мы поменяем:

maxSkew: 2

то получим на каждой ноде по 2 пода максимум:

|  | root@kub-master1:~/helm-charts/3_topolgy_spread_constraints# kubectl get pod -n app -o wideNAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATESapp-999f4cf-2vcx6 1/1 Running 0 10s 10.233.79.63 kub-worker3.test.local <none> <none>app-999f4cf-58lwd 0/1 Pending 0 8s <none> <none> <none> <none>app-999f4cf-bzp4b 0/1 Pending 0 9s <none> <none> <none> <none>app-999f4cf-qqr95 0/1 Pending 0 8s <none> <none> <none> <none>app-999f4cf-qsgvc 1/1 Running 0 10s 10.233.67.224 kub-worker1.test.local <none> <none>app-999f4cf-wkcrq 1/1 Running 0 10s 10.233.107.180 kub-worker2.test.local <none> <none>app-f9885f844-7wnxc 1/1 Running 0 118s 10.233.67.229 kub-worker1.test.local <none> <none>app-f9885f844-nl8n4 1/1 Running 0 48s 10.233.107.189 kub-worker2.test.local <none> <none>app-f9885f844-xz4jh 1/1 Running 0 48s 10.233.79.60 kub-worker3.test.local <none> <none> |
| --- | --- |

но если поправим на более мягкие правила(ScheduleAnyway):

|  | topologySpread: enabled: true constraints: - topologyKey: "kubernetes.io/hostname" # по серверам maxSkew: 2 whenUnsatisfiable: "ScheduleAnyway" # или "DoNotSchedule" |
| --- | --- |

то получим:

|  | root@kub-master1:~/helm-charts/3_topolgy_spread_constraints# kubectl get pod -n app -o wideNAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATESapp-6c68fcc9f8-46vjz 1/1 Running 0 7s 10.233.79.24 kub-worker3.test.local <none> <none>app-6c68fcc9f8-784jm 1/1 Running 0 6s 10.233.107.155 kub-worker2.test.local <none> <none>app-6c68fcc9f8-9r4fg 1/1 Running 0 8s 10.233.67.248 kub-worker1.test.local <none> <none>app-6c68fcc9f8-gvsms 1/1 Running 0 8s 10.233.107.170 kub-worker2.test.local <none> <none>app-6c68fcc9f8-j4rvl 1/1 Running 0 6s 10.233.67.221 kub-worker1.test.local <none> <none>app-6c68fcc9f8-n5qqt 1/1 Running 0 5s 10.233.79.53 kub-worker3.test.local <none> <none>app-6c68fcc9f8-psn9v 1/1 Running 0 8s 10.233.79.45 kub-worker3.test.local <none> <none> |
| --- | --- |

### []Добавление RollingUpdate

**RollingUpdate**— стандартная стратегия выката`Deployment`, при которой старые Pod’ы постепенно заменяются новыми, сохраняя доступность сервиса.

Ключевые настройки:

- 

`maxSurge`— сколько**дополнительных**Pod’ов (сверх`replicas`) можно поднять во время обновления. Проценты или число (например,`25%`или`1`).

- 

`maxUnavailable`— сколько Pod’ов может быть**временно недоступно**из желаемого числа во время обновления (например,`0`для минимального даунтайма).

- 

`minReadySeconds`— сколько секунд Pod должен быть в`Ready`, прежде чем считается «действительно готовым» для прогресса выката (страхует от «ложной готовности»).

- 

`progressDeadlineSeconds`— дедлайн на прогресс выката; если не выполняется (например, Pod’ы не становятся`Ready`), Deployment помечает выкат как застрявший.

- 

`revisionHistoryLimit`— сколько старых ReplicaSet’ов хранить для отката.

в spec нужно добавить:

|  | spec: # --- RollingUpdate strategy + параметры выката --- {{- if .Values.updateStrategy.enabled }} strategy: type: {{ .Values.updateStrategy.type | default "RollingUpdate" }} {{- if ne (.Values.updateStrategy.type | default "RollingUpdate") "Recreate" }} rollingUpdate: maxSurge: {{ .Values.updateStrategy.rollingUpdate.maxSurge | default "25%" }} maxUnavailable: {{ .Values.updateStrategy.rollingUpdate.maxUnavailable | default 0 }} {{- end }} minReadySeconds: {{ .Values.updateStrategy.minReadySeconds | default 0 }} revisionHistoryLimit: {{ .Values.updateStrategy.revisionHistoryLimit | default 10 }} progressDeadlineSeconds: {{ .Values.updateStrategy.progressDeadlineSeconds | default 600 }} {{- end }} # --- /RollingUpdate --- |
| --- | --- |

в values докидываем:

|  | updateStrategy: enabled: true type: RollingUpdate # или Recreate rollingUpdate: maxSurge: 25% # число или проценты; напр. 1 или "25%" maxUnavailable: 0 # число или проценты; 0 = без даунтайма по репликам minReadySeconds: 10 # сколько pod должен быть Ready перед следующим шагом revisionHistoryLimit: 10 # сколько старых ReplicaSet хранить progressDeadlineSeconds: 600 # дедлайн на прогресс выката |
| --- | --- |

выкатываем:

root@kub-master1:~/helm-charts/4_rollingupdate# helm upgrade --install app -n app ./common-chart -f ./common-chart/values.yaml

### []Примеры использования affinity/anti-affinity, nodeSelector, taint/tolerations

**nodeSelector**

Самый простой способ «прилипнуть» к нужным нодам. Это просто логическоеANDпо ярлыкам ноды (точные совпадения).

Когда использовать: есть специальный пул нод (GPU/ssd/arm64/комплаенс), и поды обязательно должны запускаться только там.

Плюсы: простой и быстрый.

Минусы: только точное равенство, без “или/не”, без весов/приоритетов.

|  | spec: template: spec: nodeSelector: disktype: ssd nodepool: batch |
| --- | --- |

**Node affinity**

Как nodeSelector, но богаче:

requiredDuringSchedulingIgnoredDuringExecution — жёсткое требование (как hard-rule).

preferredDuringSchedulingIgnoredDuringExecution — мягкое пожелание (веса).

Операторы: In/NotIn/Exists/DoesNotExist/Gt/Lt.

Когда использовать: «хочу только linux&amd64», «предпочитаю зону b», «не ставься на ноды с ярлыком env=dev».

| 12345678910111213141516171819 | spec: template: spec: affinity: nodeAffinity: requiredDuringSchedulingIgnoredDuringExecution: nodeSelectorTerms: - matchExpressions: - key: kubernetes.io/os operator: In values: ["linux"] preferredDuringSchedulingIgnoredDuringExecution: - weight: 50 preference: matchExpressions: - key: topology.kubernetes.io/zone operator: In values: ["b"] |
| --- | --- |

**Pod affinity / anti-affinity**

Правила «рядом с/врозь от» других подов (по их labels) на уровне топологии (topologyKey: хостнейм, зона, регион).

podAffinity — стараемся ко-локировать сервисы (например, веб ближе к кэшу).

podAntiAffinity — распределяем реплики по узлам/зонам (избегаем «все яйца в одну корзину»).

Есть жёсткие (required…) и мягкие (preferred…) варианты.

Замечание: anti-affinity может поддушить размещение в маленьком кластере → чаще ставят preferred… либо комбинируют с topologySpreadConstraints.

|  | spec: template: metadata: labels: { app: checkout } spec: affinity: podAntiAffinity: requiredDuringSchedulingIgnoredDuringExecution: - labelSelector: matchLabels: { app: checkout } topologyKey: kubernetes.io/hostname |
| --- | --- |

**Taints / Tolerations**

Taint на ноде «отталкивает» все поды, пока у пода нет соответствующей toleration.
Эффекты:

NoSchedule — не размещать новые поды без толерации.

PreferNoSchedule — по возможности избегать.

NoExecute — ещё и выселяет уже запущенные поды без толерации.

пример:
отмечаем ноду как gpu-пул (запрет для всех без толерации)
kubectl taint nodes nodeA pool=gpu:NoSchedule

|  | # под допускает gpu-таинтspec: tolerations: - key: "pool" operator: "Equal" value: "gpu" effect: "NoSchedule" |
| --- | --- |

Важно: toleration не «притягивает» под на ноду — она только разрешает игнорировать «запрет». Чтобы реально направить под в пул, обычно комбинируют:
taint на ноде,
toleration у пода,
и nodeSelector / nodeAffinity для целенаправленного выбора.

**Рассмотрим nodeselector:**

|  | root@kub-master1:~# kubectl get nodes --show-labels NAME STATUS ROLES AGE VERSION LABELSkub-master1.test.local Ready control-plane 101d v1.32.5 beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub-master1.test.local,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=kub-master2.test.local Ready control-plane 101d v1.32.5 beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub-master2.test.local,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=kub-master3.test.local Ready control-plane 101d v1.32.5 beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub-master3.test.local,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=kub-worker1.test.local Ready <none> 101d v1.32.5 beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub-worker1.test.local,kubernetes.io/os=linuxkub-worker2.test.local Ready <none> 101d v1.32.5 beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub-worker2.test.local,kubernetes.io/os=linuxkub-worker3.test.local Ready <none> 101d v1.32.5 beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub-worker3.test.local,kubernetes.io/os=linux |
| --- | --- |

разместим апку на сервере
kub-worker1.test.local

вот его label

kubernetes.io/hostname=kub-worker1.test.local

/etc/ansible/kubespray-official/helm-charts/5_affinity_nodeselector_tollerations/common-chart/values-nodeselector.yaml

|  | nodeSelector: kubernetes.io/hostname: kub-worker1.test.local |
| --- | --- |

root@kub-master1:~/helm-charts/5_affinity_nodeselector_tollerations#**helm upgrade --install app -n app ./common-chart -f ./common-chart/values-nodeselector.yaml**

|  | root@kub-master1:~/helm-charts/5_affinity_nodeselector_tollerations# kubectl describe pod -n app app-d44db8d94-cx8jw | grep Node-SelectorsNode-Selectors: kubernetes.io/hostname=kub-worker1.test.local |
| --- | --- |

как видим всё ок, nodeselector добавился.

**Рассмотрим affinity**

Разнести реплики по нодам/зонам (HA) — podAntiAffinity

|  | replicaCount: 4affinity: podAntiAffinity: requiredDuringSchedulingIgnoredDuringExecution: - labelSelector: matchLabels: app.kubernetes.io/instance: app # ← подставь своё имя приложения topologyKey: kubernetes.io/hostname # или topology.kubernetes.io/zone |
| --- | --- |

при этом topologyspread выключен:

|  | topologySpread: enabled: false constraints: - topologyKey: "kubernetes.io/hostname" # по серверам maxSkew: 2 whenUnsatisfiable: "ScheduleAnyway" # или "DoNotSchedule" |
| --- | --- |

root@kub-master1:~/helm-charts/5_affinity_nodeselector_tollerations# helm upgrade --install app -n app ./common-chart -f ./common-chart/values-affinity.yaml

вот результат:

|  | root@kub-master1:~/helm-charts/5_affinity_nodeselector_tollerations# kubectl get pod -n app -o wideNAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATESapp-764b587b5d-59qqk 1/1 Running 0 9s 10.233.107.187 kub-worker2.test.local <none> <none>app-764b587b5d-lcl7g 1/1 Running 0 9s 10.233.67.255 kub-worker1.test.local <none> <none>app-764b587b5d-nfzkn 1/1 Running 0 9s 10.233.79.9 kub-worker3.test.local <none> <none>app-764b587b5d-tp6k6 0/1 Pending 0 9s <none> <none> <none> <none> |
| --- | --- |

как видим одна реплика не может заехать из-за правил requiredDuringSchedulingIgnoredDuringExecution

поменяем на мягкие правила:

preferredDuringSchedulingIgnoredDuringExecution

|  | affinity: podAntiAffinity: preferredDuringSchedulingIgnoredDuringExecution: - weight: 50 podAffinityTerm: labelSelector: matchLabels: app.kubernetes.io/instance: app # ← подставь своё имя приложения topologyKey: kubernetes.io/hostname # или topology.kubernetes.io/zone |
| --- | --- |

|  | root@kub-master1:~/helm-charts/5_affinity_nodeselector_tollerations# kubectl get pod -n app -o wideNAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATESapp-67f8c856cd-6npmr 1/1 Running 0 3s 10.233.79.36 kub-worker3.test.local <none> <none>app-67f8c856cd-jhz7q 1/1 Running 0 3s 10.233.79.10 kub-worker3.test.local <none> <none>app-67f8c856cd-np9hf 1/1 Running 0 3s 10.233.107.175 kub-worker2.test.local <none> <none>app-67f8c856cd-rf6n8 1/1 Running 0 3s 10.233.67.233 kub-worker1.test.local <none> <none> |
| --- | --- |

**Рассмотрим nodeAffinity**

|  | affinity: nodeAffinity: requiredDuringSchedulingIgnoredDuringExecution: nodeSelectorTerms: - matchExpressions: - key: kubernetes.io/arch operator: In values: ["amd64"] - key: kubernetes.io/hostname operator: In values: ["kub-worker1.test.local"] |
| --- | --- |

root@kub-master1:~/helm-charts/5_affinity_nodeselector_tollerations# helm upgrade --install app -n app ./common-chart -f ./common-chart/values-node-affinity.yaml

|  | root@kub-master1:~/helm-charts/5_affinity_nodeselector_tollerations# kubectl get pod -n app -o wideNAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATESapp-6cd994bf7-x9f42 1/1 Running 0 101s 10.233.67.215 kub-worker1.test.local <none> <none> |
| --- | --- |

**Рассмотрим taint/tolerations**

напомню label на нода:

|  | root@kub-master1:~/helm-charts/5_affinity_nodeselector_tollerations# kubectl get nodes --show-labels NAME STATUS ROLES AGE VERSION LABELSkub-master1.test.local Ready control-plane 101d v1.32.5 beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub-master1.test.local,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=kub-master2.test.local Ready control-plane 101d v1.32.5 beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub-master2.test.local,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=kub-master3.test.local Ready control-plane 101d v1.32.5 beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub-master3.test.local,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=kub-worker1.test.local Ready <none> 101d v1.32.5 beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub-worker1.test.local,kubernetes.io/os=linuxkub-worker2.test.local Ready <none> 101d v1.32.5 beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub-worker2.test.local,kubernetes.io/os=linuxkub-worker3.test.local Ready <none> 101d v1.32.5 beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=kub-worker3.test.local,kubernetes.io/os=linux |
| --- | --- |

возьмём как пример ноду мастера:

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687 | kubectl describe node kub-master1.test.local Name: kub-master1.test.localRoles: control-planeLabels: beta.kubernetes.io/arch=amd64 beta.kubernetes.io/os=linux kubernetes.io/arch=amd64 kubernetes.io/hostname=kub-master1.test.local kubernetes.io/os=linux node-role.kubernetes.io/control-plane= node.kubernetes.io/exclude-from-external-load-balancers=Annotations: kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock node.alpha.kubernetes.io/ttl: 0 projectcalico.org/IPv4Address: 192.168.1.112/24 projectcalico.org/IPv4VXLANTunnelAddr: 10.233.66.192 volumes.kubernetes.io/controller-managed-attach-detach: trueCreationTimestamp: Wed, 21 May 2025 12:53:14 +0600Taints: node-role.kubernetes.io/control-plane:NoScheduleUnschedulable: falseLease: HolderIdentity: kub-master1.test.local AcquireTime: <unset> RenewTime: Sun, 31 Aug 2025 12:24:50 +0600Conditions: Type Status LastHeartbeatTime LastTransitionTime Reason Message ---- ------ ----------------- ------------------ ------ ------- NetworkUnavailable False Sun, 31 Aug 2025 10:35:32 +0600 Sun, 31 Aug 2025 10:35:32 +0600 CalicoIsUp Calico is running on this node MemoryPressure False Sun, 31 Aug 2025 12:24:51 +0600 Sun, 31 Aug 2025 07:38:48 +0600 KubeletHasSufficientMemory kubelet has sufficient memory available DiskPressure False Sun, 31 Aug 2025 12:24:51 +0600 Sun, 24 Aug 2025 11:47:47 +0600 KubeletHasNoDiskPressure kubelet has no disk pressure PIDPressure False Sun, 31 Aug 2025 12:24:51 +0600 Sun, 24 Aug 2025 11:47:47 +0600 KubeletHasSufficientPID kubelet has sufficient PID available Ready True Sun, 31 Aug 2025 12:24:51 +0600 Sun, 31 Aug 2025 08:06:01 +0600 KubeletReady kubelet is posting ready statusAddresses: InternalIP: 192.168.1.112 Hostname: kub-master1.test.localCapacity: cpu: 4 ephemeral-storage: 69549756Ki hugepages-2Mi: 0 memory: 2465324Ki pods: 110Allocatable: cpu: 3400m ephemeral-storage: 63048479024 hugepages-2Mi: 0 memory: 1576492Ki pods: 110System Info: Machine ID: 10e2ab9ef8014ab8b59603f47fdc2db7 System UUID: bcb81750-b634-af4f-9e83-b2b6a5e11578 Boot ID: e9b7ac88-c37e-4eba-8395-2311a9906eb4 Kernel Version: 6.1.0-37-amd64 OS Image: Debian GNU/Linux 12 (bookworm) Operating System: linux Architecture: amd64 Container Runtime Version: containerd://2.0.5 Kubelet Version: v1.32.5 Kube-Proxy Version: v1.32.5PodCIDR: 10.233.64.0/24PodCIDRs: 10.233.64.0/24Non-terminated Pods: (13 in total) Namespace Name CPU Requests CPU Limits Memory Requests Memory Limits Age --------- ---- ------------ ---------- --------------- ------------- --- app app-f6d978c88-kvwfs 0 (0%) 0 (0%) 0 (0%) 0 (0%) 101s kube-system calico-kube-controllers-588d6df6c9-r8l7t 30m (0%) 1 (29%) 64M (3%) 256M (15%) 42d kube-system calico-node-fn5tb 150m (4%) 0 (0%) 64M (3%) 500M (30%) 97d kube-system coredns-7d6ddb4b69-x8p5j 100m (2%) 0 (0%) 70Mi (4%) 300Mi (19%) 59d kube-system dns-autoscaler-79f85f486f-dj988 20m (0%) 0 (0%) 10Mi (0%) 0 (0%) 42d kube-system kube-apiserver-kub-master1.test.local 250m (7%) 0 (0%) 0 (0%) 0 (0%) 97d kube-system kube-controller-manager-kub-master1.test.local 200m (5%) 0 (0%) 0 (0%) 0 (0%) 97d kube-system kube-proxy-5hwjn 0 (0%) 0 (0%) 0 (0%) 0 (0%) 70d kube-system kube-scheduler-kub-master1.test.local 100m (2%) 0 (0%) 0 (0%) 0 (0%) 97d kube-system nodelocaldns-kcvkl 100m (2%) 0 (0%) 70Mi (4%) 200Mi (12%) 63d loki promtail-64b25 100m (2%) 200m (5%) 128Mi (8%) 128Mi (8%) 4h18m metallb-system metallb-speaker-g65w6 50m (1%) 100m (2%) 50Mi (3%) 100Mi (6%) 4h18m monitoring vmks-prometheus-node-exporter-zjp2z 50m (1%) 500m (14%) 50Mi (3%) 500Mi (32%) 4h18mAllocated resources: (Total limits may be over 100 percent, i.e., overcommitted.) Resource Requests Limits -------- -------- ------ cpu 1150m (33%) 1800m (52%) memory 524361728 (32%) 2043651328 (126%) ephemeral-storage 0 (0%) 0 (0%) hugepages-2Mi 0 (0%) 0 (0%)Events: Type Reason Age From Message ---- ------ ---- ---- ------- Normal RegisteredNode 8m42s node-controller Node kub-master1.test.local event: Registered Node kub-master1.test.local in Controller Normal RegisteredNode 4m25s node-controller Node kub-master1.test.local event: Registered Node kub-master1.test.local in Controller |
| --- | --- |

на ней есть вот такой taint

Taints: node-role.kubernetes.io/control-plane:NoSchedule

нам нужно добавить tolerations для подов и укажем эту ноду для размещения пода:

|  | nodeSelector: kubernetes.io/hostname: kub-master1.test.localtolerations: - key: "node-role.kubernetes.io/control-plane" operator: "Exists" effect: "NoSchedule" |
| --- | --- |

root@kub-master1:~/helm-charts/5_affinity_nodeselector_tollerations# helm upgrade --nstall app -n app ./common-chart -f ./common-chart/values-taint-toleration.yaml

### []Добавление PodDisruptionBudget

PodDisruptionBudget (PDB) — это механизм в Kubernetes, который позволяет администраторам контролировать, сколько подов (pods) приложения могут быть одновременно недоступны во время плановых или внеплановых событий, таких как:
— Обновления узлов (node drain, node upgrades)
— Масштабирование кластера
— Ручное удаление подов (например, `kubectl delete pod`)

PDBгарантирует, что критически важные сервисы останутся работоспособными даже при обслуживании кластера.

Как работает PodDisruptionBudget?
PDBопределяет **минимальное количество доступных подов (`minAvailable`) или максимальное количество недоступных подов (`maxUnavailable`) для приложения.

для работы добавим новый темплейт:

/etc/ansible/kubespray-official/helm-charts/6_poddisruptionbudget/common-chart/templates/pdb.yaml

| 1234567891011121314151617181920212223242526 | {{- /*PodDisruptionBudget for the Deployment pods*/ -}}{{- if .Values.pdb.enabled }}apiVersion: policy/v1kind: PodDisruptionBudgetmetadata: name: {{ include "common-chart.fullname" . }} labels: {{- include "common-chart.labels" . | nindent 4 }}spec: {{- if hasKey .Values.pdb "minAvailable" }} minAvailable: {{ .Values.pdb.minAvailable }} {{- else if hasKey .Values.pdb "maxUnavailable" }} maxUnavailable: {{ .Values.pdb.maxUnavailable }} {{- else }} minAvailable: 1 {{- end }} {{- with .Values.pdb.unhealthyPodEvictionPolicy }} unhealthyPodEvictionPolicy: {{ . | quote }} {{- end }} selector: matchLabels: {{- include "common-chart.selectorLabels" . | nindent 6 }}{{- end }} |
| --- | --- |

в values докинем:

/etc/ansible/kubespray-official/helm-charts/6_poddisruptionbudget/common-chart/values.yaml

|  | pdb: enabled: true # Выбери ОДИН из параметров ниже: minAvailable: 1 #maxUnavailable: 1 #Замечание: при replicaCount: 1 и minAvailable: 1 drain ноды будет блокироваться — это ожидаемое поведение. Если это не нужно, ставьте maxUnavailable: 1 или увеличьте число реплик |
| --- | --- |

можно использовать или minAvailable или maxUnavailable

root@kub-master1:~/helm-charts/6_poddisruptionbudget# helm upgrade --install app -n app ./common-chart -f ./common-chart/values.yaml

проверяем:

|  | root@kub-master1:~/helm-charts/6_poddisruptionbudget# kubectl get poddisruptionbudgets.policy -n appNAME MIN AVAILABLE MAX UNAVAILABLE ALLOWED DISRUPTIONS AGEapp 1 N/A 0 4m34s |
| --- | --- |

### []Ingress+ cert-manager+resources

у нас уже установлен[cert-manager](#cert-manager)т.е. нам нужно добавить только аннотацию с нужным issue

|  | root@kub-master1:~/helm-charts/7_ingress_cermanager# kubectl get clusterissuers.cert-manager.io NAME READY AGEca-issuer True 50d |
| --- | --- |

в values будет выглядеть так:

|  | ingress: enabled: true className: "nginx" annotations: cert-manager.io/cluster-issuer: ca-issuer hosts: - host: test-app.test.local paths: - path: / pathType: ImplementationSpecific tls: - secretName: test-app-tls hosts: - test-app.test.local |
| --- | --- |

для сертификата аннотация
annotations: cert-manager.io/cluster-issuer: ca-issuer
для ingress указываем наш класс
className: "nginx"
для ssl указываем имя секрета в который будет положен серт сгенерированный cert-manager
secretName: test-app-tls

так же сразу добавляем ресурсы для нашей апки:

|  | resources: limits: cpu: 100m memory: 128Mi requests: cpu: 100m memory: 128Mi |
| --- | --- |

ставим:
root@kub-master1:~/helm-charts/7_ingress_cermanager# helm upgrade --install app -n app ./common-chart -f ./common-chart/values.yaml

проверяем сертификаты и секрет:

|  | root@kub-master1:~/helm-charts/7_ingress_cermanager# kubectl get certificaterequests.cert-manager.io -n appNAME APPROVED DENIED READY ISSUER REQUESTER AGEtest-app-tls-1 True True ca-issuer system:serviceaccount:cert-manager:cert-manager 6m36sroot@kub-master1:~/helm-charts/7_ingress_cermanager# kubectl get certificates.cert-manager.io -n appNAME READY SECRET AGEtest-app-tls True test-app-tls 7m31sroot@kub-master1:~/helm-charts/7_ingress_cermanager# kubectl get secret -n app | grep test-apptest-app-tls kubernetes.io/tls 3 7m41s |
| --- | --- |

проверяем домен:

|  | root@kub-master1:~/helm-charts/7_ingress_cermanager# kubectl get ingress -n appNAME CLASS HOSTS ADDRESS PORTS AGEapp nginx test-app.test.local 192.168.1.191 80, 443 9m1s |
| --- | --- |

проверим курлом:

| 123456789101112131415161718 | curl -k -IL test-app.test.localHTTP/1.1 308 Permanent RedirectDate: Sun, 31 Aug 2025 08:48:23 GMTContent-Type: text/htmlContent-Length: 164Connection: keep-aliveLocation: https://test-app.test.localHTTP/2 200 date: Sun, 31 Aug 2025 08:48:23 GMTcontent-type: text/htmlcontent-length: 612vary: Accept-Encodinglast-modified: Tue, 23 Apr 2019 10:18:21 GMTetag: "5cbee66d-264"accept-ranges: bytesstrict-transport-security: max-age=31536000; includeSubDomains |
| --- | --- |

проверим серт:

|  | openssl s_client -connect test-app.test.local:443 \ -servername test-app.test.local -showcerts </dev/null 2>/dev/null \| openssl x509 -noout -subject -issuer -dates -serial \ -fingerprint -sha256 -ext subjectAltName |
| --- | --- |

вот результат:

|  | subject=issuer=CN = test.localnotBefore=Aug 31 08:36:42 2025 GMTnotAfter=Nov 29 08:36:42 2025 GMTserial=8D19F8466BABE8820FE1F63DD9087930sha256 Fingerprint=77:92:25:35:90:DB:D9:70:3F:94:19:0B:76:8C:EB:E2:EE:10:48:DA:8F:D7:F0:4C:FC:89:F7:67:CB:3A:69:A3X509v3 Subject Alternative Name: critical DNS:test-app.test.local |
| --- | --- |

если хотим получить полный сертификат:

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103 | root@kub-master1:~/helm-charts/7_ingress_cermanager# openssl s_client -connect test-app.test.local:443CONNECTED(00000003)depth=0 verify error:num=20:unable to get local issuer certificateverify return:1depth=0 verify error:num=21:unable to verify the first certificateverify return:1depth=0 verify return:1---Certificate chain 0 s: i:CN = test.local a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256 v:NotBefore: Aug 31 08:36:42 2025 GMT; NotAfter: Nov 29 08:36:42 2025 GMT---Server certificate-----BEGIN CERTIFICATE-----MIIDBDCCAeygAwIBAgIRAI0Z+EZrq+iCD+H2PdkIeTAwDQYJKoZIhvcNAQELBQAwFTETMBEGA1UEAxMKdGVzdC5sb2NhbDAeFw0yNTA4MzEwODM2NDJaFw0yNTExMjkwODM2NDJaMAAwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDhGpv3gODzfgPkm2aMXIUKQ8EnzSSsJDECAN6nJr9AeQBonqcuYQpwqaNaMLnw523b45KAbyrtL22VMN+dnEKmaimK/jSGP09AdyKkArRSyAQGlg55qnJ9ildt2W1IgiOiJicgYWzbOy/PBEGICcVNIuC4YeGOG0kXO/lcS5LhMuz6UYb7HQ9wXvc/+h/FTECZ72IML6etRKE8mI+GaEUoNsTNXqJhA1eVdDwT02ddr5yJ12F6F633X+kucMSpgMhi8aIiGZGIdaY+ZZIladtZItK0TX4HWfr8ke/hBetEs+kGxobvyacI8JGc/JV1F8/YxbEJA1ikyPLHvUXKVBBZAgMBAAGjZDBiMA4GA1UdDwEB/wQEAwIFoDAMBgNVHRMBAf8EAjAAMB8GA1UdIwQYMBaAFDnbkugU2kHHb3aZmFiqIJBsixcRMCEGA1UdEQEB/wQXMBWCE3Rlc3QtYXBwLnRlc3QubG9jYWwwDQYJKoZIhvcNAQELBQADggEBAJiu+A5fcSSaspRinQlUzYtFwGgXX1TtrOc7CxV90fvmvfBYbUSojjd+fWiWBbaYmFkSL+MKwJgWuRn83eyrfRfSGJwV8fX43qB7S6y6GURaq9t/dr7jLm/sq1/+h67dFbRTKNcKtNDAC7JDG6fjWfmwO3DgPDYA+Vs+35kjJxnjXtMm/XidMtqwNoaIJwt/pez/nZN/8LsT7TjZVU7heXhgm5uU7Lq0WrfFYRbJLywmTHaZFb19MsdXKNklQb9c7RZANFB+E/uS3kI+FWTNiCF8wml3a2DLCfYUAK7vfREPE3W0fUA38yo0J+/DQEI1sqMxLeWC9agf996h3/8NpwE=-----END CERTIFICATE-----subject=issuer=CN = test.local---No client certificate CA names sentPeer signing digest: SHA256Peer signature type: RSA-PSSServer Temp Key: X25519, 253 bits---SSL handshake has read 1336 bytes and written 405 bytesVerification error: unable to verify the first certificate---New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384Server public key is 2048 bitSecure Renegotiation IS NOT supportedCompression: NONEExpansion: NONENo ALPN negotiatedEarly data was not sentVerify return code: 21 (unable to verify the first certificate)------Post-Handshake New Session Ticket arrived:SSL-Session: Protocol : TLSv1.3 Cipher : TLS_AES_256_GCM_SHA384 Session-ID: E494640BD528EC5DDF9A98801D8945DD90A11CCA28C7A57D276628FCA0EAF7B4 Session-ID-ctx: Resumption PSK: 8508F87E12A8B8EAF4B1CB9DC2CDBE58461EFB8A7FC4A6CF5792A50EDA5F8F97D5D6620E4A20FCB48B7717E2AB72804F PSK identity: None PSK identity hint: None SRP username: None TLS session ticket lifetime hint: 1800 (seconds) TLS session ticket: 0000 - 8a 33 c9 bc fb d5 10 c1-35 9a a7 c9 f6 c8 14 2c .3......5......, 0010 - 63 5f 89 ad 43 23 43 4a-95 80 59 fd bf 96 f7 c5 c_..C#CJ..Y.…. Start Time: 1756630827 Timeout : 7200 (sec) Verify return code: 21 (unable to verify the first certificate) Extended master secret: no Max Early Data: 0---read R BLOCK---Post-Handshake New Session Ticket arrived:SSL-Session: Protocol : TLSv1.3 Cipher : TLS_AES_256_GCM_SHA384 Session-ID: E3FAA1435BB042D5E7C0C0062A39B18B4128305B603B5113FD48CFC4233E3648 Session-ID-ctx: Resumption PSK: 4DC5B6E28BF76BF2D7ED650019C226C9B29219BF8B1339D92ED4896ECC87750EE9982BB0ED88B89F29ECB5E40F436524 PSK identity: None PSK identity hint: None SRP username: None TLS session ticket lifetime hint: 1800 (seconds) TLS session ticket: 0000 - 0e 89 59 77 a9 32 3d 49-f0 cf 95 4f 52 42 44 79 ..Yw.2=I...ORBDy 0010 - 2c e0 e0 1e 15 89 00 d4-3b b4 c4 e3 66 6d a6 a7 ,.......;...fm.. Start Time: 1756630827 Timeout : 7200 (sec) Verify return code: 21 (unable to verify the first certificate) Extended master secret: no Max Early Data: 0---read R BLOCK |
| --- | --- |

проверяем что ресурсы выставлены:

|  | root@kub-master1:~/helm-charts/7_ingress_cermanager# kubectl describe pod -n app app-c76d6bd47-nmd9q | grep -E 'Limits|Requests' -A2 Limits: cpu: 100m memory: 128Mi Requests: cpu: 100m memory: 128Mi |
| --- | --- |

### []ПроверимHPA

HPAберётCPU/Memory из`metrics.k8s.io`(обычно даёт metrics-server). Без него авто-скейла не будет.

для проверки поCPUи Memory будем использовать следующие конструкции для нагрузки:

нагрузить апку по процу:

|  | kubectl -n app exec -it deploy/app -- sh -c 'yes > /dev/null' |
| --- | --- |

нагрузить по оперативке да и процу каждый под через эфимерные контейнеры:

|  | for p in $(kubectl -n app get po -l app.kubernetes.io/instance=app,app.kubernetes.io/name=common-chart -o name); do kubectl -n app debug "$p" \ --target=common-chart \ --image=polinux/stress \ -- /bin/sh -lc 'stress --vm 1 --vm-bytes 100M --vm-keep --timeout 600s'done |
| --- | --- |

в дескрайбе они выглядят так:

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115 | kubectl -n app describe pod app-c76d6bd47-nmd9qName: app-c76d6bd47-nmd9qNamespace: appPriority: 0Service Account: appNode: kub-worker1.test.local/192.168.1.115Start Time: Sun, 31 Aug 2025 14:36:29 +0600Labels: app.kubernetes.io/instance=app app.kubernetes.io/managed-by=Helm app.kubernetes.io/name=common-chart app.kubernetes.io/version=1.16.0 helm.sh/chart=common-chart-0.1.7 pod-template-hash=c76d6bd47Annotations: cni.projectcalico.org/containerID: df3fd70a2df8b7bf628b3f9c657efe9454270c2551ade1a4f854cf67bfc492e3 cni.projectcalico.org/podIP: 10.233.67.217/32 cni.projectcalico.org/podIPs: 10.233.67.217/32 kubectl.kubernetes.io/restartedAt: 2025-08-31T11:42:43+06:00 secret.reloader.stakater.com/reload: app-secret vso.secrets.hashicorp.com/restartedAt: 2025-08-23T16:25:33ZStatus: RunningIP: 10.233.67.217IPs: IP: 10.233.67.217Controlled By: ReplicaSet/app-c76d6bd47Containers: common-chart: Container ID: containerd://85f4f725e9547afe1d586703fae8e240557dca85fb2f66f12876118673273957 Image: nginx:1.16.0 Image ID: docker.io/library/nginx@sha256:3e373fd5b8d41baeddc24be311c5c6929425c04cabf893b874ac09b72a798010 Port: 80/TCP Host Port: 0/TCP State: Running Started: Sun, 31 Aug 2025 14:36:30 +0600 Ready: True Restart Count: 0 Limits: cpu: 100m memory: 128Mi Requests: cpu: 100m memory: 128Mi Liveness: http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3 Readiness: http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3 Environment Variables from: app-secret Secret Optional: false Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-wrfw6 (ro)Ephemeral Containers: debugger-qhwnh: Container ID: containerd://b0a6a234c9db693e56ce2ab0e5512daa2c4f45f310aef87e9659421e6cc599be Image: polinux/stress Image ID: docker.io/polinux/stress@sha256:b6144f84f9c15dac80deb48d3a646b55c7043ab1d83ea0a697c09097aaad21aa Port: <none> Host Port: <none> Command: /bin/sh -lc stress --vm 1 --vm-bytes 50M --vm-keep --timeout 600s State: Running Started: Sun, 31 Aug 2025 15:30:08 +0600 Ready: False Restart Count: 0 Environment: <none> Mounts: <none> debugger-t9mk9: Container ID: containerd://2194d5f52ce50e8dd5809163f27a0d5412b712444b7ec5fe854e4f8c3f97254a Image: polinux/stress Image ID: docker.io/polinux/stress@sha256:b6144f84f9c15dac80deb48d3a646b55c7043ab1d83ea0a697c09097aaad21aa Port: <none> Host Port: <none> Command: /bin/sh -lc stress --vm 1 --vm-bytes 50M --vm-keep --timeout 600s State: Running Started: Sun, 31 Aug 2025 15:30:57 +0600 Ready: False Restart Count: 0 Environment: <none> Mounts: <none>Conditions: Type Status PodReadyToStartContainers True Initialized True Ready True ContainersReady True PodScheduled True Volumes: kube-api-access-wrfw6: Type: Projected (a volume that contains injected data from multiple sources) TokenExpirationSeconds: 3607 ConfigMapName: kube-root-ca.crt ConfigMapOptional: <nil> DownwardAPI: trueQoS Class: GuaranteedNode-Selectors: <none>Tolerations: node.kubernetes.io/not-ready:NoExecute op=Exists for 300s node.kubernetes.io/unreachable:NoExecute op=Exists for 300sTopology Spread Constraints: kubernetes.io/hostname:ScheduleAnyway when max skew 2 is exceeded for selector app.kubernetes.io/instance=app,app.kubernetes.io/name=common-chartEvents: Type Reason Age From Message ---- ------ ---- ---- ------- Normal Scheduled 57m default-scheduler Successfully assigned app/app-c76d6bd47-nmd9q to kub-worker1.test.local Normal Pulled 57m kubelet Container image "nginx:1.16.0" already present on machine Normal Created 57m kubelet Created container: common-chart Normal Started 57m kubelet Started container common-chart Normal Pulling 3m53s kubelet Pulling image "polinux/stress" Normal Pulled 3m47s kubelet Successfully pulled image "polinux/stress" in 5.655s (5.655s including waiting). Image size: 4041495 bytes. Normal Created 3m47s kubelet Created container: debugger-qhwnh Normal Started 3m47s kubelet Started container debugger-qhwnh Normal Pulling 2m59s kubelet Pulling image "polinux/stress" Normal Pulled 2m58s kubelet Successfully pulled image "polinux/stress" in 1.448s (1.448s including waiting). Image size: 4041495 bytes. Normal Created 2m58s kubelet Created container: debugger-t9mk9 Normal Started 2m58s kubelet Started container debugger-t9mk9 |
| --- | --- |

чтобы включитьHPAкоторый есть по дефолту включаем в

/etc/ansible/kubespray-official/helm-charts/8_horizontalpodautoscaler/common-chart/values.yaml

|  | autoscaling: enabled: true minReplicas: 1 maxReplicas: 5 targetCPUUtilizationPercentage: 80 # targetMemoryUtilizationPercentage: 80 |
| --- | --- |

ставим

root@kub-master1:~/helm-charts/8_horizontalpodautoscaler# helm upgrade --install app -n app ./common-chart -f ./common-chart/values.yaml

после чего запускаем нагрузку и смотрим:

|  | root@kub-master1:~/helm-charts/8_horizontalpodautoscaler# kubectl get horizontalpodautoscalers.autoscaling -n appNAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGEapp Deployment/app cpu: 0%/80% 1 5 1 31s |
| --- | --- |

сначала ничего но потом:

|  | root@kub-master1:~/helm-charts/8_horizontalpodautoscaler# kubectl get horizontalpodautoscalers.autoscaling -n app -wNAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGEapp Deployment/app cpu: 1%/80% 1 5 1 2m44sapp Deployment/app cpu: 102%/80% 1 5 1 3mapp Deployment/app cpu: 103%/80% 1 5 2 3m15sapp Deployment/app cpu: 51%/80% 1 5 2 3m51s |
| --- | --- |

как видим я запустил нагрузку она выросла до 102 дальше запустилась реплика и нагрузка ушла до 51
и так и продолжает висеть, а всё потому что сейчас нагружен фактически только 1PODа метрика снимается с деплоймента.

вот доказательство:

|  | root@kub-master1:~/helm-charts/8_horizontalpodautoscalerkubectl get pod -n appNAME READY STATUS RESTARTS AGEapp-68665864b8-82str 1/1 Running 0 3m46sapp-68665864b8-9w68d 1/1 Running 0 46s |
| --- | --- |

![](/news/sidmidru/article-db03824b64600e5b/image-340.png)

![](/news/sidmidru/article-db03824b64600e5b/image-341.png)

2 реплики 1 загружена максимально вторая пустая но нагрузка снимается с деплоймента поэтому она в hpa 50%

когда нагрузка падает всё откатывается назад:

|  | root@kub-master1:~/helm-charts/8_horizontalpodautoscaler# kubectl get horizontalpodautoscalers.autoscaling -n app -wNAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGEapp Deployment/app cpu: 1%/80% 1 5 2 17mapp Deployment/app cpu: 1%/80% 1 5 1 18m |
| --- | --- |

чтобы использовать behavior https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#configurable-scaling-behavior
нужно расширить template

/etc/ansible/kubespray-official/helm-charts/8_horizontalpodautoscaler/common-chart/templates/hpa.yaml

| 1234567891011121314151617181920212223242526272829303132333435363738 | {{- if .Values.autoscaling.enabled }}apiVersion: autoscaling/v2kind: HorizontalPodAutoscalermetadata: name: {{ include "common-chart.fullname" . }} labels: {{- include "common-chart.labels" . | nindent 4 }}spec: scaleTargetRef: apiVersion: apps/v1 kind: Deployment name: {{ include "common-chart.fullname" . }} minReplicas: {{ .Values.autoscaling.minReplicas }} maxReplicas: {{ .Values.autoscaling.maxReplicas }} metrics: {{- if .Values.autoscaling.targetCPUUtilizationPercentage }} - type: Resource resource: name: cpu target: type: Utilization averageUtilization: {{ .Values.autoscaling.targetCPUUtilizationPercentage }} {{- end }} {{- if .Values.autoscaling.targetMemoryUtilizationPercentage }} - type: Resource resource: name: memory target: type: Utilization averageUtilization: {{ .Values.autoscaling.targetMemoryUtilizationPercentage }} {{- end }} {{- with .Values.autoscaling.behavior }} behavior: {{- toYaml . | nindent 4 }} {{- end }}{{- end }} |
| --- | --- |

а вот values
/etc/ansible/kubespray-official/helm-charts/8_horizontalpodautoscaler/common-chart/values.yaml

| 1234567891011121314151617181920212223242526 | autoscaling: enabled: true minReplicas: 1 maxReplicas: 5 targetCPUUtilizationPercentage: 80 # targetMemoryUtilizationPercentage: 80 behavior: scaleUp: stabilizationWindowSeconds: 0 selectPolicy: Max # допустимо: Max | Min | Disabled policies: - type: Percent # не более +100% за 60 сек value: 100 periodSeconds: 60 - type: Pods # и не более +4 пода за 60 сек value: 4 periodSeconds: 60 scaleDown: stabilizationWindowSeconds: 30 selectPolicy: Max # для жёсткого ограничения выбирай Min policies: - type: Percent # не более -10% за 60 сек value: 10 periodSeconds: 60 |
| --- | --- |

**`behavior.scaleUp`**— как быстро можноРАСТИ.

- 

`stabilizationWindowSeconds: 0`
Нет окна стабилизации на апскейл: как только метрика выше цели — масштабируемся сразу (дефолт именно так).[Kubernetes](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)

- 

`policies`— «ограничители скорости» роста за скользящее окно`periodSeconds`.
В примере заданы два лимита одновременно:

- 

`type: Percent, value: 100, periodSeconds: 60`→ не более**+100%**от текущего числа реплик**за 60с**;

- 

`type: Pods, value: 4, periodSeconds: 60`→ не более**+4 пода**за 60с.

- 

`selectPolicy: Max`
Если задано несколько политик, берётся та, которая**разрешает самое большое**изменение (для scaleUp — которая добавит больше подов).

**`behavior.scaleDown`**— как быстро можноУМЕНЬШАТЬ.

- 

`stabilizationWindowSeconds: 30`
При даунскейлеHPAзапоминает прошлые «желательные» размеры и берёт**наибольший**из них в окне 30с — это сглаживает «дёрганье». (По умолчанию окно даунскейла 300с, у меня ускорено до 30с.)[Kubernetes](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)

- 

`policies: [{type: Percent, value: 10, periodSeconds: 60}]`
Можно уменьшать не больше чем на**10% текущих реплик за 60с**.

- 

`selectPolicy: Max`
При нескольких политиках выбрали бы самую «быструю» (удаляет больше подов). Чтобы ограничивать жёстче — ставь`Min`

смотрим как после наших изменений происходит скейлинг вниз:

|  | kubectl get horizontalpodautoscalers.autoscaling -n app -wNAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGEapp Deployment/app cpu: 76%/80% 1 5 4 45mapp Deployment/app cpu: 1%/80% 1 5 4 51mapp Deployment/app cpu: 1%/80% 1 5 4 51mapp Deployment/app cpu: 1%/80% 1 5 3 52mapp Deployment/app cpu: 1%/80% 1 5 3 52mapp Deployment/app cpu: 1%/80% 1 5 2 53mapp Deployment/app cpu: 1%/80% 1 5 2 53mapp Deployment/app cpu: 1%/80% 1 5 1 54m |
| --- | --- |

### []Добавим Keda

как ставить keda я уже описывал вот[тут](#keda)

нужно добавить template

/etc/ansible/kubespray-official/helm-charts/9_keda/common-chart/templates/keda-scaledobject.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172 | {{- if .Values.keda.enabled }}apiVersion: keda.sh/v1alpha1kind: ScaledObjectmetadata: name: {{ include "common-chart.fullname" . }}{{- if .Values.keda.nameSuffix }}-{{ .Values.keda.nameSuffix }}{{- end }} labels: {{- include "common-chart.labels" . | nindent 4 }}spec: scaleTargetRef: apiVersion: {{ default "apps/v1" .Values.keda.scaleTargetRef.apiVersion }} kind: {{ default "Deployment" .Values.keda.scaleTargetRef.kind }} name: {{ default (include "common-chart.fullname" .) .Values.keda.scaleTargetRef.name }} {{- with .Values.keda.pollingInterval }} pollingInterval: {{ . }} {{- end }} {{- with .Values.keda.cooldownPeriod }} cooldownPeriod: {{ . }} {{- end }} {{- with .Values.keda.minReplicaCount }} minReplicaCount: {{ . }} {{- end }} {{- with .Values.keda.maxReplicaCount }} maxReplicaCount: {{ . }} {{- end }} {{- with .Values.keda.restoreToOriginalReplicaCount }} restoreToOriginalReplicaCount: {{ . }} {{- end }} {{- with .Values.keda.fallback }} {{- if or (hasKey . "failureThreshold") (hasKey . "replicas") }} fallback: {{- if hasKey . "failureThreshold" }}failureThreshold: {{ .failureThreshold }}{{- end }} {{- if hasKey . "replicas" }}replicas: {{ .replicas }}{{- end }} {{- end }} {{- end }} {{- if .Values.keda.advanced }} advanced: {{- with .Values.keda.advanced.horizontalPodAutoscalerConfig }} horizontalPodAutoscalerConfig: {{- with .behavior }} behavior: {{- toYaml . | nindent 8 }} {{- end }} {{- end }} {{- with .Values.keda.advanced.scalingStrategy }} scalingStrategy: {{- toYaml . | nindent 6 }} {{- end }} {{- end }} triggers: {{- range $i, $t := .Values.keda.triggers }} - type: {{ $t.type | quote }} {{- if $t.name }} name: {{ $t.name | quote }} {{- end }} {{- if $t.metricType }} metricType: {{ $t.metricType | quote }} {{- end }} metadata: {{- /* метаданные триггера как есть (serverAddress, query, threshold и т.п.) */}} {{- toYaml $t.metadata | nindent 8 }} {{- with $t.authenticationRef }} authenticationRef: {{- if .name }}name: {{ .name | quote }}{{- end }} {{- if .kind }}kind: {{ .kind | quote }}{{- end }} {{/* TriggerAuthentication | ClusterTriggerAuthentication */}} {{- end }} {{- end }}{{- end }} |
| --- | --- |

values будет таким:

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566 | keda: enabled: true # имя цели по умолчанию = fullname релиза, но можно переопределить scaleTargetRef: apiVersion: apps/v1 kind: Deployment name: "" # пусто => {{ include "common-chart.fullname" . }} # как часто KEDA опрашивает метрику и сколько держит реплики после падения нагрузки pollingInterval: 15 # сек, дефолт KEDA ~30 cooldownPeriod: 30 # сек, дефолт KEDA ~300 # границы масштаба (если не заданы, KEDA не создаёт HPA-ограничения) minReplicaCount: 1 maxReplicaCount: 5 # вернуть ли реплики к исходному числу при отключении триггера restoreToOriginalReplicaCount: false # fallback — сколько держать реплик, если источник метрик «упал» fallback: # включается автоматически, если задать поля ниже # failureThreshold: 3 # после скольких «ошибочных» циклов метрик включить fallback # replicas: 2 # во сколько реплик перейти на fallback # продвинутые настройки (пробрасываются в HPA, который создаёт KEDA) advanced: # стратегия при нескольких триггерах: max | min | average scalingStrategy: multipleScalersCalculation: max # полностью совместимо с autoscaling/v2 .spec.behavior (как у HPA) horizontalPodAutoscalerConfig: behavior: scaleUp: stabilizationWindowSeconds: 0 selectPolicy: Max policies: - type: Percent value: 100 periodSeconds: 60 - type: Pods value: 4 periodSeconds: 60 scaleDown: stabilizationWindowSeconds: 20 selectPolicy: Max policies: - type: Percent value: 10 periodSeconds: 60 # список триггеров (можно несколько). Ниже — пример для Prometheus/VictoriaMetrics triggers: - type: prometheus # name: "ingress-requests" # опционально — имя триггера metadata: serverAddress: "http://vmsingle-vmks-victoria-metrics-k8s-stack.monitoring.svc.cluster.local:8428" metricName: "ingress_requests" query: |- sum( increase( nginx_ingress_controller_requests{ingress="app"}[2m] ) ) threshold: "100" # строки обязательны для большинства триггеров # activationThreshold: "10" # опционально — «включаться» только после X # ignoreNullValues: "true" # для некоторых скейлеров unsafeSsl: "true" # если Prometheus с self-signed # authenticationRef: # name: keda-prometheus-auth # TriggerAuthentication или ClusterTriggerAuthentication # kind: TriggerAuthentication |
| --- | --- |

Пояснения к ключам в values

scaleTargetRef.* — на что скейлим (обычно Deployment).

pollingInterval — как частоKEDAтянет метрику. Меньше → быстрее реакция, но выше нагрузка на источник.

cooldownPeriod — «тормоз» после падения метрики: столько секунд держит увеличенные реплики перед даунскейлом.

minReplicaCount/maxReplicaCount — нижняя/верхняя граница для создаваемогоHPA.

restoreToOriginalReplicaCount — если true, когда триггеры «молчат»,KEDAвернёт реплики к тому числу, которое было до включенияKEDA.

fallback — если метрика не читается failureThreshold циклов подряд,KEDAвыставит фиксированное число replicas.

advanced.horizontalPodAutoscalerConfig.behavior — 1:1 как вHPAautoscaling/v2: окна стабилизации и лимиты скорости роста/сжатия.

advanced.scalingStrategy.multipleScalersCalculation — что делать, если триггеров несколько (берём максимум/минимум/среднее желаемых реплик).

triggers[] — сами скейлеры. Для Prometheus/VictoriaMetrics главные поля: serverAddress, query или metricName+доп.параметры, threshold. Часто полезны activationThreshold, unsafeSsl.

authenticationRef — если Prometheus защищён (базовая auth, токен,TLSи т.п.), тут указываете ссылку на TriggerAuthentication/ClusterTriggerAuthentication (можно добавить отдельный шаблон при необходимости).

проверяем как идёт скейлинг:

так как мы ориентируемся на количество запросов, то вот так я дёргаю:
for i in {1..5500}; do curl -Ik https://test-app.test.local; done

а вот результат скейлинга:

| 123456789101112131415161718192021222324 | root@kub-master1:~/helm-charts/9_keda# kubectl get horizontalpodautoscalers.autoscaling -n app -wNAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGEkeda-hpa-app Deployment/app 4772/100 (avg) 1 5 1 11mkeda-hpa-app Deployment/app 806400m/100 (avg) 1 5 5 11mkeda-hpa-app Deployment/app 638800m/100 (avg) 1 5 5 12mkeda-hpa-app Deployment/app 482200m/100 (avg) 1 5 5 12mkeda-hpa-app Deployment/app 433/100 (avg) 1 5 5 12mkeda-hpa-app Deployment/app 496600m/100 (avg) 1 5 5 12mkeda-hpa-app Deployment/app 558400m/100 (avg) 1 5 5 13mkeda-hpa-app Deployment/app 758600m/100 (avg) 1 5 5 13mkeda-hpa-app Deployment/app 974600m/100 (avg) 1 5 5 13mkeda-hpa-app Deployment/app 1100/100 (avg) 1 5 5 13mkeda-hpa-app Deployment/app 996/100 (avg) 1 5 5 14mkeda-hpa-app Deployment/app 777800m/100 (avg) 1 5 5 14mkeda-hpa-app Deployment/app 558200m/100 (avg) 1 5 5 15mkeda-hpa-app Deployment/app 341400m/100 (avg) 1 5 5 15mkeda-hpa-app Deployment/app 125400m/100 (avg) 1 5 5 15mkeda-hpa-app Deployment/app 0/100 (avg) 1 5 5 15mkeda-hpa-app Deployment/app 0/100 (avg) 1 5 5 16mkeda-hpa-app Deployment/app 0/100 (avg) 1 5 4 16mkeda-hpa-app Deployment/app 0/100 (avg) 1 5 4 17mkeda-hpa-app Deployment/app 0/100 (avg) 1 5 3 17mkeda-hpa-app Deployment/app 0/100 (avg) 1 5 3 18mkeda-hpa-app Deployment/app 0/100 (avg) 1 5 2 18m |
| --- | --- |

как видим он увидел превышение и сразу заскейлил 5 подов а вниз даже с отсутствием нагрузки он скейлит постепенно, ну и главное ориентируется на запросы.

можно ещё организоваться логику для нескольких тригеров когда срабатывает**И**/**ИЛИ**

#### []Скейлинг с логикойИЛИ

чтобы работали 2 триггера используем values:

/etc/ansible/kubespray-official/helm-charts/9_keda/common-chart/values_2_keda_trigger.yaml

| 12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273 | keda: enabled: true # имя цели по умолчанию = fullname релиза, но можно переопределить scaleTargetRef: apiVersion: apps/v1 kind: Deployment name: "" # пусто => {{ include "common-chart.fullname" . }} # как часто KEDA опрашивает метрику и сколько держит реплики после падения нагрузки pollingInterval: 15 # сек, дефолт KEDA ~30 cooldownPeriod: 30 # сек, дефолт KEDA ~300 # границы масштаба (если не заданы, KEDA не создаёт HPA-ограничения) minReplicaCount: 1 maxReplicaCount: 5 # вернуть ли реплики к исходному числу при отключении триггера restoreToOriginalReplicaCount: false # fallback — сколько держать реплик, если источник метрик «упал» fallback: # включается автоматически, если задать поля ниже # failureThreshold: 3 # после скольких «ошибочных» циклов метрик включить fallback # replicas: 2 # во сколько реплик перейти на fallback # продвинутые настройки (пробрасываются в HPA, который создаёт KEDA) advanced: # стратегия при нескольких триггерах: max | min | average scalingStrategy: multipleScalersCalculation: max # полностью совместимо с autoscaling/v2 .spec.behavior (как у HPA) horizontalPodAutoscalerConfig: behavior: scaleUp: stabilizationWindowSeconds: 0 selectPolicy: Max policies: - type: Percent value: 100 periodSeconds: 60 - type: Pods value: 4 periodSeconds: 60 scaleDown: stabilizationWindowSeconds: 20 selectPolicy: Max policies: - type: Percent value: 10 periodSeconds: 60 # список триггеров (можно несколько). Ниже — пример для Prometheus/VictoriaMetrics triggers: - type: prometheus # name: "ingress-requests" # опционально — имя триггера metadata: serverAddress: "http://vmsingle-vmks-victoria-metrics-k8s-stack.monitoring.svc.cluster.local:8428" metricName: "ingress_requests" query: |- sum( increase( nginx_ingress_controller_requests{ingress="app"}[2m] ) ) threshold: "100" # строки обязательны для большинства триггеров # activationThreshold: "10" # опционально — «включаться» только после X # ignoreNullValues: "true" # для некоторых скейлеров unsafeSsl: "true" # если Prometheus с self-signed # authenticationRef: # name: keda-prometheus-auth # TriggerAuthentication или ClusterTriggerAuthentication # kind: TriggerAuthentication # 2) CPU > 60% - type: cpu name: cpu metricType: Utilization metadata: value: "60" # целевая загрузка CPU в % # containerName: common-chart # опционально, если надо таргетить конкретный контейнер |
| --- | --- |

ставим

root@kub-master1:~/helm-charts/9_keda# helm upgrade --install app -n app ./common-chart -f ./common-chart/values_2_keda_trigger.yaml

запускаем нагрузку по cpu

kubectl -n app exec -it deploy/app -- sh -c 'yes > /dev/null'

|  | root@kub-master1:~# kubectl get horizontalpodautoscalers.autoscaling -n app -wNAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGEkeda-hpa-app Deployment/app 0/100 (avg), cpu: 1%/60% 1 5 1 6d20hkeda-hpa-app Deployment/app 0/100 (avg), cpu: 49%/60% 1 5 1 6d20hkeda-hpa-app Deployment/app 0/100 (avg), cpu: 101%/60% 1 5 1 6d20hkeda-hpa-app Deployment/app 0/100 (avg), cpu: 101%/60% 1 5 2 6d20hkeda-hpa-app Deployment/app 0/100 (avg), cpu: 50%/60% 1 5 2 6d20h |
| --- | --- |

как видим всё ок скейлится.

появляется 2 таргера и если срабатывает один из них то срабатывает скейлинг

#### []Скейлинг с логикой И

для этого надо поправить template

/etc/ansible/kubespray-official/helm-charts/9_keda/common-chart/templates/keda-scaledobject.yaml

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374757677787980818283848586878889 | {{- if .Values.keda.enabled }}apiVersion: keda.sh/v1alpha1kind: ScaledObjectmetadata: name: {{ include "common-chart.fullname" . }}{{- if .Values.keda.nameSuffix }}-{{ .Values.keda.nameSuffix }}{{- end }} labels: {{- include "common-chart.labels" . | nindent 4 }}spec: scaleTargetRef: apiVersion: {{ default "apps/v1" .Values.keda.scaleTargetRef.apiVersion }} kind: {{ default "Deployment" .Values.keda.scaleTargetRef.kind }} name: {{ default (include "common-chart.fullname" .) .Values.keda.scaleTargetRef.name }} {{- with .Values.keda.pollingInterval }} pollingInterval: {{ . }} {{- end }} {{- with .Values.keda.cooldownPeriod }} cooldownPeriod: {{ . }} {{- end }} {{- with .Values.keda.minReplicaCount }} minReplicaCount: {{ . }} {{- end }} {{- with .Values.keda.maxReplicaCount }} maxReplicaCount: {{ . }} {{- end }} {{- with .Values.keda.restoreToOriginalReplicaCount }} restoreToOriginalReplicaCount: {{ . }} {{- end }} {{- with .Values.keda.fallback }} {{- if or (hasKey . "failureThreshold") (hasKey . "replicas") }} fallback: {{- if hasKey . "failureThreshold" }} failureThreshold: {{ .failureThreshold }} {{- end }} {{- if hasKey . "replicas" }} replicas: {{ .replicas }} {{- end }} {{- end }} {{- end }} {{- with .Values.keda.scalingStrategy }} scalingStrategy: {{- toYaml . | nindent 4 }} {{- end }} {{- $adv := .Values.keda.advanced }} {{- $sm := and $adv $adv.scalingModifiers }} {{- if or $adv $sm }} advanced: {{- with $adv.horizontalPodAutoscalerConfig }} horizontalPodAutoscalerConfig: {{- toYaml . | nindent 6 }} {{- end }} {{- if and $sm $sm.formula }} scalingModifiers: formula: {{ $sm.formula | quote }} {{- with $sm.target }} target: {{ . | quote }} {{- end }} {{- with $sm.activationTarget }} activationTarget: {{ . | quote }} {{- end }} {{- with $sm.metricType }} metricType: {{ . | quote }} {{- end }} {{- end }} {{- end }} triggers: {{- range $i, $t := .Values.keda.triggers }} - type: {{ $t.type | quote }} {{- if $t.name }} name: {{ $t.name | quote }} {{- end }} {{- if $t.metricType }} metricType: {{ $t.metricType | quote }} {{- end }} metadata: {{- toYaml $t.metadata | nindent 8 }} {{- with $t.authenticationRef }} authenticationRef: {{- if .name }}name: {{ .name | quote }}{{- end }} {{- if .kind }}kind: {{ .kind | quote }}{{- end }} {{- end }} {{- end }}{{- end }} |
| --- | --- |

добавляется

сделаем срабатывание этих же тригеров но**по логике И**

для этого после правки template добавим в values:

|  | keda: advanced: scalingModifiers: formula: "req >= 100 && cpu >= 60 ? max(req/100, cpu/60) : 0" target: "1" # масштабируемся, когда формула >= 1 activationTarget: "0" # при 0 не активируемся metricType: "AverageValue" |
| --- | --- |

полный values:

/etc/ansible/kubespray-official/helm-charts/9_keda/common-chart/values_2_keda_trigger_logic_AND.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210211212213214215216217218219220221222223224225226227228229230231232233234235236237238239240241242243244245246247248249250251252253254255256257258259260261262263264265266267268269270271272273274275276 | # Default values for common-chart.# This is a YAML-formatted file.# Declare variables to be passed into your templates.# This will set the replicaset count more information can be found here: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/replicaCount: 1# This sets the container image more information can be found here: https://kubernetes.io/docs/concepts/containers/images/image: repository: nginx # This sets the pull policy for images. pullPolicy: IfNotPresent # Overrides the image tag whose default is the chart appVersion. tag: ""# This is for the secretes for pulling an image from a private repository more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/imagePullSecrets: []# This is to override the chart name.nameOverride: ""fullnameOverride: "app"#This section builds out the service account more information can be found here: https://kubernetes.io/docs/concepts/security/service-accounts/serviceAccount: # Specifies whether a service account should be created create: true # Automatically mount a ServiceAccount's API credentials? automount: true # Annotations to add to the service account annotations: {} # The name of the service account to use. # If not set and create is true, a name is generated using the fullname template name: ""# This is for setting Kubernetes Annotations to a Pod.# For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/ podAnnotations: {}# This is for setting Kubernetes Labels to a Pod.# For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/podLabels: {}podSecurityContext: {} # fsGroup: 2000securityContext: {} # capabilities: # drop: # - ALL # readOnlyRootFilesystem: true # runAsNonRoot: true # runAsUser: 1000# This is for setting up a service more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/service: # This sets the service type more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types type: ClusterIP # This sets the ports more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#field-spec-ports port: 80# This block is for setting up the ingress for more information can be found here: https://kubernetes.io/docs/concepts/services-networking/ingress/ingress: enabled: true className: "nginx" annotations: cert-manager.io/cluster-issuer: ca-issuer hosts: - host: test-app.test.local paths: - path: / pathType: ImplementationSpecific tls: - secretName: test-app-tls hosts: - test-app.test.localresources: limits: cpu: 100m memory: 128Mi requests: cpu: 100m memory: 128Mi# This is to setup the liveness and readiness probes more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/livenessProbe: httpGet: path: / port: httpreadinessProbe: httpGet: path: / port: http#This section is for setting up autoscaling more information can be found here: https://kubernetes.io/docs/concepts/workloads/autoscaling/autoscaling: enabled: false minReplicas: 1 maxReplicas: 5 targetCPUUtilizationPercentage: 80 # targetMemoryUtilizationPercentage: 80 behavior: scaleUp: stabilizationWindowSeconds: 0 selectPolicy: Max # допустимо: Max | Min | Disabled policies: - type: Percent # не более +100% за 60 сек value: 100 periodSeconds: 60 - type: Pods # и не более +4 пода за 60 сек value: 4 periodSeconds: 60 scaleDown: stabilizationWindowSeconds: 30 selectPolicy: Max # для жёсткого ограничения выбирай Min policies: - type: Percent # не более -10% за 60 сек value: 10 periodSeconds: 60# Additional volumes on the output Deployment definition.volumes: []# - name: foo# secret:# secretName: mysecret# optional: false# Additional volumeMounts on the output Deployment definition.volumeMounts: []# - name: foo# mountPath: "/etc/foo"# readOnly: truenodeSelector: {}tolerations: []affinity: {}vault_secret: enabled: true name: app-secret # имя k8s Secret, который получит VSO secretFullPath: app/service/app1 # путь в Vault: <ns>/service/<app> type: kv-v1 # у меня сейчас kv-v1 role: ns-scoped-read # универсальная роль в vault для всех ns # audiences: # - "https://kubernetes.default.svc"topologySpread: enabled: true constraints: - topologyKey: "kubernetes.io/hostname" # по серверам maxSkew: 2 whenUnsatisfiable: "ScheduleAnyway" # или "DoNotSchedule"updateStrategy: enabled: true type: RollingUpdate # или Recreate rollingUpdate: maxSurge: 25% # число или проценты; напр. 1 или "25%" maxUnavailable: 0 # число или проценты; 0 = без даунтайма по репликам minReadySeconds: 10 # сколько pod должен быть Ready перед следующим шагом revisionHistoryLimit: 10 # сколько старых ReplicaSet хранить progressDeadlineSeconds: 600 # дедлайн на прогресс выкатаpdb: enabled: true # Выбери ОДИН из параметров ниже: minAvailable: 1 #maxUnavailable: 1 #Замечание: при replicaCount: 1 и minAvailable: 1 drain ноды будет блокироваться — это ожидаемое поведение. Если это не нужно, ставьте maxUnavailable: 1 или увеличьте число репликkeda: enabled: true # Кого скейлим (по умолчанию — имя релиза) scaleTargetRef: apiVersion: apps/v1 kind: Deployment name: "" # пусто => {{ include "common-chart.fullname" . }} # Периоды опроса/остывания pollingInterval: 15 cooldownPeriod: 30 # Границы minReplicaCount: 1 maxReplicaCount: 5 # Поведение при сбое источника метрик (опционально) fallback: # failureThreshold: 3 # replicas: 2 # Алгоритм объединения желаемых реплик при нескольких триггерах (если НЕТ формулы) — max|min|average scalingStrategy: multipleScalersCalculation: max # Проброс behavior в создаваемый HPA (опционально) advanced: horizontalPodAutoscalerConfig: behavior: scaleUp: stabilizationWindowSeconds: 0 selectPolicy: Max policies: - type: Percent value: 100 periodSeconds: 60 - type: Pods value: 4 periodSeconds: 60 scaleDown: stabilizationWindowSeconds: 20 selectPolicy: Max policies: - type: Percent value: 10 periodSeconds: 60 # === AND-логика через формулу === # req — это метрика запросов, cpu — Prometheus-метрика CPU-процентов. scalingModifiers: formula: "req >= 100 && cpu >= 60 ? max(req/100, cpu/60) : 0" target: "1" # масштабируемся, когда формула >= 1 activationTarget: "0" # при 0 не активируемся metricType: "AverageValue" # === Триггеры (два Prometheus-триггера: ingress-requests и CPU-проценты) === triggers: # 1) Ingress requests за 2 минуты - type: prometheus name: req metadata: serverAddress: "http://vmsingle-vmks-victoria-metrics-k8s-stack.monitoring.svc.cluster.local:8428" metricName: "ingress_requests" query: |- sum( increase( nginx_ingress_controller_requests{ingress="app"}[2m] ) ) threshold: "100" unsafeSsl: "true" # 2) CPU-проценты как Prometheus-метрика (средняя загрузка по подам деплоймента "app" в ns "app") # Требует kube-state-metrics. Формула: 100 * usage_cores / requested_cores. - type: prometheus name: cpu metadata: serverAddress: "http://vmsingle-vmks-victoria-metrics-k8s-stack.monitoring.svc.cluster.local:8428" metricName: "cpu_utilization_pct" query: |- 100 * sum by (namespace) ( rate(container_cpu_usage_seconds_total{ namespace="app", pod=~"app-.*", container!="POD", image!="" }[1m]) ) / sum by (namespace) ( kube_pod_container_resource_requests{ namespace="app", pod=~"app-.*", resource="cpu" } ) threshold: "60" unsafeSsl: "true" |
| --- | --- |

на каждом target должно быть name чтобы сработала

formula: "req >= 100&&cpu >= 60 ? max(req/100, cpu/60) : 0"

hpa выглядит таким образом:

|  | root@kub-master1:~# kubectl get hpa -n app -wNAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGEkeda-hpa-app Deployment/app 0/1 (avg) 1 5 1 6d23h |
| --- | --- |

скейлинг начнётсяТОЛЬКОесли оба тригера сработают, а чтобы это произошло нужно запустить:

kubectl -n app exec -it deploy/app -- sh -c 'yes > /dev/null'
for i in {1..5500}; do curl -Ik https://test-app.test.local; done

и результат:

|  | root@kub-master1:~# kubectl get hpa -n app -wkeda-hpa-app Deployment/app 2474m/1 (avg) 1 5 5 6d23h |
| --- | --- |

чтобы вернуться к**логикеИЛИ**когда скейлинг будет срабатывать на любой тригер, можно в values закомментировать

|  | scalingModifiers: formula: "req >= 100 && cpu >= 60 ? max(req/100, cpu/60) : 0" target: "1" # масштабируемся, когда формула >= 1 activationTarget: "0" # при 0 не активируемся metricType: "AverageValue" |
| --- | --- |

### []Контейнеры вPOD

в Pod бывают четыре «типа» контейнеров (три — на уровнеAPI, один — служебный уCRI)

##### 1) Обычные контейнеры (spec.containers[])

- 

Основное приложение Pod’а, могут быть**несколько**.

- 

Запускаются**после**всех init-контейнеров.

- 

Имеют порты, probes, ресурсы (requests/limits), volumeMounts.

- 

Перезапускаются по`pod.spec.restartPolicy`(обычно`Always`).

- 

«Сайдкары» (лог-агенты, сервис-mesh прокси) — это**просто обычные контейнеры**, работающие рядом с основным.

Мини-пример:

|  | spec: containers: - name: app image: nginx ports: [{containerPort: 80}] - name: sidecar-log image: fluent/fluent-bit volumeMounts: [{name: logs, mountPath: /var/log/app}] volumes: - name: logs emptyDir: {} |
| --- | --- |

##### 2) Init-контейнеры (spec.initContainers[])

- 

Запускаются**последовательно**и**должны завершиться**успешно перед стартом обычных контейнеров.

- 

Используются для миграций, прогрева кэша, ожидания зависимостей.

- 

Не экспонируют порты/readiness (им «готовность» не нужна — им надо**закончить**).

- 

При сбое рестартуют по`restartPolicy`Pod’а (с backoff).

Пример:

|  | spec: initContainers: - name: wait-db image: busybox command: ["sh","-c","until nc -z db 5432; do sleep 2; done"] |
| --- | --- |

##### 3) Эфемерные (debug) контейнеры (spec.ephemeralContainers[])

- 

Добавляются**на лету**для отладки:`kubectl debug …`.

- 

**Не входят**в шаблон контроллеров (Deployment/…); не влияют на`READY`.

- 

Не перезапускаются, нет probes/портов; разделяют сеть/IPC/volumes Pod’а.

- 

Их использование учитывается в фактическом потреблении Pod’а (CPU/Mem).

Пример запуска:

|  | kubectl debug pod/<pod-name> \ --target=<имя-обычного-контейнера> \ --image=busybox --container=debug -- /bin/sh |
| --- | --- |

##### 4) «Pause»/sandbox контейнер (служебный)

- 

СоздаётсяCRI(containerd/docker)**внутренне**для каждого Pod’а.

- 

Держит сетевые/IPC-неймспейсы иIPPod’а.

- 

Не управляется манифестами, в`kubectl`обычно не показывается.

#### Важные нюансы

- 

**ОбщийIP/порты/loopback**— все контейнеры Pod’а делят один сетевой неймспейс.

- 

**Общие тома**— через`spec.volumes`и`volumeMounts`.

- 

**Последовательность старта**: init → обычные (вместе) → при необходимости добавляешь ephemeral.

- 

**Завершение**:`preStop`/graceful shutdown вызывается у**обычных**контейнеров; init уже завершились, ephemeral — отладочные.

- 

**HPA/VPA**: метрики берутся по всему Pod’у; сайдкар/эфемерный контейнер повышает суммарное потребление.

### []обычные контейнеры (sidecar)

как уже было сказано sidecar container запускается рядом с основным

чтобы добавлять несколько sidecar контейнеров используем range и поправим deployment

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374757677787980 | {{- /* ===== САЙДКАРЫ: несколько контейнеров через range ===== */}} {{- with .Values.sidecars }} {{- range $i, $sc := . }} - name: {{ default (printf "%s-sidecar-%d" $.Chart.Name $i) $sc.name }} {{- with $sc.securityContext }} securityContext: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.image }} image: "{{ .repository }}{{- if .tag }}:{{ .tag }}{{- end }}" {{- if .pullPolicy }} imagePullPolicy: {{ .pullPolicy }} {{- end }} {{- end }} {{- with $sc.command }} command: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.args }} args: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.workingDir }} workingDir: {{ . | quote }} {{- end }} {{- with $sc.ports }} ports: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.env }} env: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.envFrom }} envFrom: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.livenessProbe }} livenessProbe: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.readinessProbe }} readinessProbe: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.startupProbe }} startupProbe: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.resources }} resources: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.volumeMounts }} volumeMounts: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.lifecycle }} lifecycle: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.terminationMessagePath }} terminationMessagePath: {{ . | quote }} {{- end }} {{- with $sc.terminationMessagePolicy }} terminationMessagePolicy: {{ . | quote }} {{- end }} {{- with $sc.extraContainerSpec }} {{- toYaml . | nindent 10 }} {{- end }} {{- end }} {{- end }} {{- /* ===== /САЙДКАРЫ ===== */}} |
| --- | --- |

полностью деплоймент будет выглядеть так:

/etc/ansible/kubespray-official/helm-charts/10_containers_1_sidecar/common-chart/templates/deployment.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210211212213214215216217218219220221222223224225226227 | apiVersion: apps/v1kind: Deploymentmetadata: name: {{ include "common-chart.fullname" . }} labels: {{- include "common-chart.labels" . | nindent 4 }}spec: {{- if not .Values.autoscaling.enabled }} replicas: {{ .Values.replicaCount }} {{- end }} # --- RollingUpdate strategy + параметры выката --- {{- if .Values.updateStrategy.enabled }} strategy: type: {{ .Values.updateStrategy.type | default "RollingUpdate" }} {{- if ne (.Values.updateStrategy.type | default "RollingUpdate") "Recreate" }} rollingUpdate: maxSurge: {{ .Values.updateStrategy.rollingUpdate.maxSurge | default "25%" }} maxUnavailable: {{ .Values.updateStrategy.rollingUpdate.maxUnavailable | default 0 }} {{- end }} minReadySeconds: {{ .Values.updateStrategy.minReadySeconds | default 0 }} revisionHistoryLimit: {{ .Values.updateStrategy.revisionHistoryLimit | default 10 }} progressDeadlineSeconds: {{ .Values.updateStrategy.progressDeadlineSeconds | default 600 }} {{- end }} # --- /RollingUpdate --- selector: matchLabels: {{- include "common-chart.selectorLabels" . | nindent 6 }} template: metadata: # Аннотация reloader добавляется автоматически, если включён vault_secret и указан имя секрета annotations: {{- if and .Values.vault_secret.enabled .Values.vault_secret.name }} secret.reloader.stakater.com/reload: "{{ .Values.vault_secret.name }}" {{- end }} {{- with .Values.podAnnotations }} {{- toYaml . | nindent 8 }} {{- end }} labels: {{- include "common-chart.labels" . | nindent 8 }} {{- with .Values.podLabels }} {{- toYaml . | nindent 8 }} {{- end }} spec: {{- with .Values.imagePullSecrets }} imagePullSecrets: {{- toYaml . | nindent 8 }} {{- end }} serviceAccountName: {{ include "common-chart.serviceAccountName" . }} {{- if .Values.podSecurityContext }} securityContext: {{- toYaml .Values.podSecurityContext | nindent 8 }} {{- else }} securityContext: {} {{- end }} {{/* Настройки подключения секрета: безопасные дефолты (без хомпинга '-') */}} {{ $attach := default (dict) .Values.vault_secret.attach }} {{ $asEnv := default true (get $attach "asEnv") }} {{ $asVol := default false (get $attach "asVolume") }} {{ $mountPath := default "/etc/app/secret" (get $attach "mountPath") }} containers: - name: {{ .Chart.Name }} securityContext: {{- toYaml .Values.securityContext | nindent 12 }} image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}" imagePullPolicy: {{ .Values.image.pullPolicy }} ports: - name: http containerPort: {{ .Values.service.port }} protocol: TCP {{- if and .Values.vault_secret.enabled $asEnv .Values.vault_secret.name }} envFrom: - secretRef: name: {{ .Values.vault_secret.name }} {{- end }} livenessProbe: {{- toYaml .Values.livenessProbe | nindent 12 }} readinessProbe: {{- toYaml .Values.readinessProbe | nindent 12 }} resources: {{- toYaml .Values.resources | nindent 12 }} {{- $needSecretVol := and .Values.vault_secret.enabled $asVol .Values.vault_secret.name }} {{- if or $needSecretVol .Values.volumeMounts }} volumeMounts: {{- if $needSecretVol }} - name: vault-secret mountPath: {{ $mountPath }} readOnly: true {{- end }} {{- with .Values.volumeMounts }} {{- toYaml . | nindent 12 }} {{- end }} {{- end }} {{- /* ===== САЙДКАРЫ: несколько контейнеров через range ===== */}} {{- with .Values.sidecars }} {{- range $i, $sc := . }} - name: {{ default (printf "%s-sidecar-%d" $.Chart.Name $i) $sc.name }} {{- with $sc.securityContext }} securityContext: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.image }} image: "{{ .repository }}{{- if .tag }}:{{ .tag }}{{- end }}" {{- if .pullPolicy }} imagePullPolicy: {{ .pullPolicy }} {{- end }} {{- end }} {{- with $sc.command }} command: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.args }} args: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.workingDir }} workingDir: {{ . | quote }} {{- end }} {{- with $sc.ports }} ports: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.env }} env: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.envFrom }} envFrom: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.livenessProbe }} livenessProbe: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.readinessProbe }} readinessProbe: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.startupProbe }} startupProbe: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.resources }} resources: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.volumeMounts }} volumeMounts: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.lifecycle }} lifecycle: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.terminationMessagePath }} terminationMessagePath: {{ . | quote }} {{- end }} {{- with $sc.terminationMessagePolicy }} terminationMessagePolicy: {{ . | quote }} {{- end }} {{- with $sc.extraContainerSpec }} {{- toYaml . | nindent 10 }} {{- end }} {{- end }} {{- end }} {{- /* ===== /САЙДКАРЫ ===== */}} {{- $needSecretVol := and .Values.vault_secret.enabled $asVol .Values.vault_secret.name }} {{- if or $needSecretVol .Values.volumes }} volumes: {{- if $needSecretVol }} - name: vault-secret secret: secretName: {{ .Values.vault_secret.name }} {{- end }} {{- with .Values.volumes }} {{- toYaml . | nindent 8 }} {{- end }} {{- end }} {{- with .Values.nodeSelector }} nodeSelector: {{- toYaml . | nindent 8 }} {{- end }} {{- with .Values.affinity }} affinity: {{- toYaml . | nindent 8 }} {{- end }} {{- if and .Values.topologySpread.enabled .Values.topologySpread.constraints }} topologySpreadConstraints: {{- range .Values.topologySpread.constraints }} - maxSkew: {{ .maxSkew | default 1 }} topologyKey: {{ .topologyKey | quote }} whenUnsatisfiable: {{ .whenUnsatisfiable | default "DoNotSchedule" | quote }} {{- if hasKey . "minDomains" }} minDomains: {{ .minDomains }} {{- end }} labelSelector: matchLabels: {{- include "common-chart.selectorLabels" $ | nindent 14 }} {{- end }} {{- end }} {{- with .Values.tolerations }} tolerations: {{- toYaml . | nindent 8 }} {{- end }} |
| --- | --- |

values будет выглядеть так:

/etc/ansible/kubespray-official/helm-charts/10_containers_1_sidecar/common-chart/values.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210211212213214215216217218219220221222223224225226227228229230231232233234235236237238239240241242243244245246247248249250251252253254255256257258259260261262263264265266267268269270271272273274275276277278279280281282283284285286287288289290291292293294295296297298299300301302303304305306307308309310311312313314315316317318319320321322323324325326327328329330331332333334335336337338339340341342343344345346 | # Default values for common-chart.# This is a YAML-formatted file.# Declare variables to be passed into your templates.# This will set the replicaset count more information can be found here: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/replicaCount: 1# This sets the container image more information can be found here: https://kubernetes.io/docs/concepts/containers/images/image: repository: nginx # This sets the pull policy for images. pullPolicy: IfNotPresent # Overrides the image tag whose default is the chart appVersion. tag: ""# This is for the secretes for pulling an image from a private repository more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/imagePullSecrets: []# This is to override the chart name.nameOverride: ""fullnameOverride: "app"#This section builds out the service account more information can be found here: https://kubernetes.io/docs/concepts/security/service-accounts/serviceAccount: # Specifies whether a service account should be created create: true # Automatically mount a ServiceAccount's API credentials? automount: true # Annotations to add to the service account annotations: {} # The name of the service account to use. # If not set and create is true, a name is generated using the fullname template name: ""# This is for setting Kubernetes Annotations to a Pod.# For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/ podAnnotations: {}# This is for setting Kubernetes Labels to a Pod.# For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/podLabels: {}podSecurityContext: {} # fsGroup: 2000securityContext: {} # capabilities: # drop: # - ALL # readOnlyRootFilesystem: true # runAsNonRoot: true # runAsUser: 1000# This is for setting up a service more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/service: # This sets the service type more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types type: ClusterIP # This sets the ports more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#field-spec-ports port: 80# This block is for setting up the ingress for more information can be found here: https://kubernetes.io/docs/concepts/services-networking/ingress/ingress: enabled: true className: "nginx" annotations: cert-manager.io/cluster-issuer: ca-issuer hosts: - host: test-app.test.local paths: - path: / pathType: ImplementationSpecific tls: - secretName: test-app-tls hosts: - test-app.test.localresources: limits: cpu: 100m memory: 128Mi requests: cpu: 100m memory: 128Mi# This is to setup the liveness and readiness probes more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/livenessProbe: httpGet: path: / port: httpreadinessProbe: httpGet: path: / port: http#This section is for setting up autoscaling more information can be found here: https://kubernetes.io/docs/concepts/workloads/autoscaling/autoscaling: enabled: false minReplicas: 1 maxReplicas: 5 targetCPUUtilizationPercentage: 80 # targetMemoryUtilizationPercentage: 80 behavior: scaleUp: stabilizationWindowSeconds: 0 selectPolicy: Max # допустимо: Max | Min | Disabled policies: - type: Percent # не более +100% за 60 сек value: 100 periodSeconds: 60 - type: Pods # и не более +4 пода за 60 сек value: 4 periodSeconds: 60 scaleDown: stabilizationWindowSeconds: 30 selectPolicy: Max # для жёсткого ограничения выбирай Min policies: - type: Percent # не более -10% за 60 сек value: 10 periodSeconds: 60# Additional volumes on the output Deployment definition.volumes: []# - name: foo# secret:# secretName: mysecret# optional: false# Additional volumeMounts on the output Deployment definition.volumeMounts: []# - name: foo# mountPath: "/etc/foo"# readOnly: truenodeSelector: {}tolerations: []affinity: {}vault_secret: enabled: true name: app-secret # имя k8s Secret, который получит VSO secretFullPath: app/service/app1 # путь в Vault: <ns>/service/<app> type: kv-v1 # у меня сейчас kv-v1 role: ns-scoped-read # универсальная роль в vault для всех ns # audiences: # - "https://kubernetes.default.svc"topologySpread: enabled: true constraints: - topologyKey: "kubernetes.io/hostname" # по серверам maxSkew: 2 whenUnsatisfiable: "ScheduleAnyway" # или "DoNotSchedule"updateStrategy: enabled: true type: RollingUpdate # или Recreate rollingUpdate: maxSurge: 25% # число или проценты; напр. 1 или "25%" maxUnavailable: 0 # число или проценты; 0 = без даунтайма по репликам minReadySeconds: 10 # сколько pod должен быть Ready перед следующим шагом revisionHistoryLimit: 10 # сколько старых ReplicaSet хранить progressDeadlineSeconds: 600 # дедлайн на прогресс выкатаpdb: enabled: true # Выбери ОДИН из параметров ниже: minAvailable: 1 #maxUnavailable: 1 #Замечание: при replicaCount: 1 и minAvailable: 1 drain ноды будет блокироваться — это ожидаемое поведение. Если это не нужно, ставьте maxUnavailable: 1 или увеличьте число репликkeda: enabled: true # Кого скейлим (по умолчанию — имя релиза) scaleTargetRef: apiVersion: apps/v1 kind: Deployment name: "" # пусто => {{ include "common-chart.fullname" . }} # Периоды опроса/остывания pollingInterval: 15 cooldownPeriod: 30 # Границы minReplicaCount: 1 maxReplicaCount: 5 # Поведение при сбое источника метрик (опционально) fallback: # failureThreshold: 3 # replicas: 2 # Алгоритм объединения желаемых реплик при нескольких триггерах (если НЕТ формулы) — max|min|average scalingStrategy: multipleScalersCalculation: max # Проброс behavior в создаваемый HPA (опционально) advanced: horizontalPodAutoscalerConfig: behavior: scaleUp: stabilizationWindowSeconds: 0 selectPolicy: Max policies: - type: Percent value: 100 periodSeconds: 60 - type: Pods value: 4 periodSeconds: 60 scaleDown: stabilizationWindowSeconds: 20 selectPolicy: Max policies: - type: Percent value: 10 periodSeconds: 60 # === AND-логика через формулу === чтобы вернуть логику ИЛИ нужно закомментировать блок scalingModifiers # req — это метрика запросов, cpu — Prometheus-метрика CPU-процентов. scalingModifiers: formula: "req >= 100 && cpu >= 60 ? max(req/100, cpu/60) : 0" target: "1" # масштабируемся, когда формула >= 1 activationTarget: "0" # при 0 не активируемся metricType: "AverageValue" # === Триггеры (два Prometheus-триггера: ingress-requests и CPU-проценты) === triggers: # 1) Ingress requests за 2 минуты - type: prometheus name: req metadata: serverAddress: "http://vmsingle-vmks-victoria-metrics-k8s-stack.monitoring.svc.cluster.local:8428" metricName: "ingress_requests" query: |- sum( increase( nginx_ingress_controller_requests{ingress="app"}[2m] ) ) threshold: "100" unsafeSsl: "true" # 2) CPU-проценты как Prometheus-метрика (средняя загрузка по подам деплоймента "app" в ns "app") # Требует kube-state-metrics. Формула: 100 * usage_cores / requested_cores. - type: prometheus name: cpu metadata: serverAddress: "http://vmsingle-vmks-victoria-metrics-k8s-stack.monitoring.svc.cluster.local:8428" metricName: "cpu_utilization_pct" query: |- 100 * sum by (namespace) ( rate(container_cpu_usage_seconds_total{ namespace="app", pod=~"app-.*", container!="POD", image!="" }[1m]) ) / sum by (namespace) ( kube_pod_container_resource_requests{ namespace="app", pod=~"app-.*", resource="cpu" } ) threshold: "60" unsafeSsl: "true"sidecars: - name: curl-pinger image: repository: curlimages/curl tag: "latest" # или зафиксируй конкретную версию pullPolicy: IfNotPresent command: - sh - -c - | while true; do code=$(curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:80/ || true) echo "$(date -u +%FT%TZ) GET / -> ${code}" sleep 5 done resources: requests: cpu: 5m memory: 16Mi limits: cpu: 50m memory: 64Mi - name: wget-pinger image: repository: busybox tag: "1.36" command: - sh - -c - | while true; do wget -q -S -O /dev/null http://127.0.0.1:80/ 2>&1 | awk 'NR==1{print strftime("%Y-%m-%dT%H:%M:%SZ"), $0}' sleep 5 done - name: php-fpm image: repository: php tag: "8.2-fpm-alpine" pullPolicy: IfNotPresent command: - sh - -c - | mkdir -p /var/www/html [ -f /var/www/html/index.php ] || echo '<?php echo "Hello from PHP-FPM @ " . date("c"); ?>' > /var/www/html/index.php # слушаем на 0.0.0.0:9000 (явно) sed -ri 's/^;?listen\s*=.*$/listen = 0.0.0.0:9000/' /usr/local/etc/php-fpm.d/zz-docker.conf exec php-fpm -F ports: - name: fpm containerPort: 9000 protocol: TCP livenessProbe: tcpSocket: port: fpm initialDelaySeconds: 5 periodSeconds: 10 readinessProbe: tcpSocket: port: fpm initialDelaySeconds: 2 periodSeconds: 5 resources: requests: cpu: 50m memory: 64Mi limits: cpu: 200m memory: 256Mi |
| --- | --- |

мы добавили 3 контейнера

ставим:

helm upgrade --install app -n app ./common-chart -f ./common-chart/values.yaml

далее проверяем логи:

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166 | root@kub-master1:~/helm-charts/10_containers_1_sidecar# kubectl get pod -n appNAME READY STATUS RESTARTS AGEapp-564d8bd5b6-w2tmj 4/4 Running 0 56sroot@kub-master1:~/helm-charts/10_containers_1_sidecar# root@kub-master1:~/helm-charts/10_containers_1_sidecar# kubectl describe pod -n app app-564d8bd5b6-w2tmj Name: app-564d8bd5b6-w2tmjNamespace: appPriority: 0Service Account: appNode: kub-worker1.test.local/192.168.1.115Start Time: Sun, 14 Sep 2025 12:52:05 +0600Labels: app.kubernetes.io/instance=app app.kubernetes.io/managed-by=Helm app.kubernetes.io/name=common-chart app.kubernetes.io/version=1.16.0 helm.sh/chart=common-chart-0.1.10 pod-template-hash=564d8bd5b6Annotations: cni.projectcalico.org/containerID: be796f8e2f25db66df23508f09c77c7eb8730f0b6142316d75a783ca75ba4ce7 cni.projectcalico.org/podIP: 10.233.67.242/32 cni.projectcalico.org/podIPs: 10.233.67.242/32 kubectl.kubernetes.io/restartedAt: 2025-08-31T11:42:43+06:00 secret.reloader.stakater.com/reload: app-secret vso.secrets.hashicorp.com/restartedAt: 2025-08-23T16:25:33ZStatus: RunningIP: 10.233.67.242IPs: IP: 10.233.67.242Controlled By: ReplicaSet/app-564d8bd5b6Containers: common-chart: Container ID: containerd://8c4ee184203141e85fdb56fd4c22be89e63df0de51b82f9650579044dc05f6d2 Image: nginx:1.16.0 Image ID: docker.io/library/nginx@sha256:3e373fd5b8d41baeddc24be311c5c6929425c04cabf893b874ac09b72a798010 Port: 80/TCP Host Port: 0/TCP State: Running Started: Sun, 14 Sep 2025 12:52:05 +0600 Ready: True Restart Count: 0 Limits: cpu: 100m memory: 128Mi Requests: cpu: 100m memory: 128Mi Liveness: http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3 Readiness: http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3 Environment Variables from: app-secret Secret Optional: false Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kqtcf (ro) curl-pinger: Container ID: containerd://843494e04cf96d21ddb8d2862669d64c5e0bf657f861bd18244ca4521a986433 Image: curlimages/curl:latest Image ID: docker.io/curlimages/curl@sha256:463eaf6072688fe96ac64fa623fe73e1dbe25d8ad6c34404a669ad3ce1f104b6 Port: <none> Host Port: <none> Command: sh -c while true; do code=$(curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:80/ || true) echo "$(date -u +%FT%TZ) GET / -> ${code}" sleep 5 done State: Running Started: Sun, 14 Sep 2025 12:52:05 +0600 Ready: True Restart Count: 0 Limits: cpu: 50m memory: 64Mi Requests: cpu: 5m memory: 16Mi Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kqtcf (ro) wget-pinger: Container ID: containerd://37d5784c1caebbaea571f8f34d3e17a3ef2e5e3105bb50751cf9e810ff47d8bf Image: busybox:1.36 Image ID: docker.io/library/busybox@sha256:303337d3d52898b33018796d760d3a01c2dcc87973dd66ec22fe78ae1e8429c9 Port: <none> Host Port: <none> Command: sh -c while true; do wget -q -S -O /dev/null http://127.0.0.1:80/ 2>&1 | awk 'NR==1{print strftime("%Y-%m-%dT%H:%M:%SZ"), $0}' sleep 5 done State: Running Started: Sun, 14 Sep 2025 12:52:05 +0600 Ready: True Restart Count: 0 Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kqtcf (ro) php-fpm: Container ID: containerd://f5e06dd50dba60d0d109c452f600be9cc8bca5dca0fd3595c49ff09c76e5021f Image: php:8.2-fpm-alpine Image ID: docker.io/library/php@sha256:b7355fb38ef93fd0ef6ccbecee006a25f9bb7fbb65a9f6b7a4721582b75c073a Port: 9000/TCP Host Port: 0/TCP Command: sh -c mkdir -p /var/www/html [ -f /var/www/html/index.php ] || echo '<?php echo "Hello from PHP-FPM @ " . date("c"); ?>' > /var/www/html/index.php # слушаем на 0.0.0.0:9000 (явно) sed -ri 's/^;?listen\s*=.*$/listen = 0.0.0.0:9000/' /usr/local/etc/php-fpm.d/zz-docker.conf exec php-fpm -F State: Running Started: Sun, 14 Sep 2025 12:52:05 +0600 Ready: True Restart Count: 0 Limits: cpu: 200m memory: 256Mi Requests: cpu: 50m memory: 64Mi Liveness: tcp-socket :fpm delay=5s timeout=1s period=10s #success=1 #failure=3 Readiness: tcp-socket :fpm delay=2s timeout=1s period=5s #success=1 #failure=3 Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kqtcf (ro)Conditions: Type Status PodReadyToStartContainers True Initialized True Ready True ContainersReady True PodScheduled True Volumes: kube-api-access-kqtcf: Type: Projected (a volume that contains injected data from multiple sources) TokenExpirationSeconds: 3607 ConfigMapName: kube-root-ca.crt ConfigMapOptional: <nil> DownwardAPI: trueQoS Class: BurstableNode-Selectors: <none>Tolerations: node.kubernetes.io/not-ready:NoExecute op=Exists for 300s node.kubernetes.io/unreachable:NoExecute op=Exists for 300sTopology Spread Constraints: kubernetes.io/hostname:ScheduleAnyway when max skew 2 is exceeded for selector app.kubernetes.io/instance=app,app.kubernetes.io/name=common-chartEvents: Type Reason Age From Message ---- ------ ---- ---- ------- Normal Scheduled 63s default-scheduler Successfully assigned app/app-564d8bd5b6-w2tmj to kub-worker1.test.local Normal Pulled 63s kubelet Container image "nginx:1.16.0" already present on machine Normal Created 63s kubelet Created container: common-chart Normal Started 63s kubelet Started container common-chart Normal Pulled 63s kubelet Container image "curlimages/curl:latest" already present on machine Normal Created 63s kubelet Created container: curl-pinger Normal Started 63s kubelet Started container curl-pinger Normal Pulled 63s kubelet Container image "busybox:1.36" already present on machine Normal Created 63s kubelet Created container: wget-pinger Normal Started 63s kubelet Started container wget-pinger Normal Pulled 63s kubelet Container image "php:8.2-fpm-alpine" already present on machine Normal Created 63s kubelet Created container: php-fpm Normal Started 63s kubelet Started container php-fpm |
| --- | --- |

|  | root@kub-master1:~/helm-charts/10_containers_1_sidecar# kubectl logs -f -n app app-564d8bd5b6-w2tmj Defaulted container "common-chart" out of: common-chart, curl-pinger, wget-pinger, php-fpm127.0.0.1 - - [14/Sep/2025:06:52:05 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/8.16.0" "-"127.0.0.1 - - [14/Sep/2025:06:52:05 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"192.168.1.115 - - [14/Sep/2025:06:52:06 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.32" "-"127.0.0.1 - - [14/Sep/2025:06:52:10 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"127.0.0.1 - - [14/Sep/2025:06:52:10 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/8.16.0" "-"192.168.1.115 - - [14/Sep/2025:06:52:15 +0000] "GET / HTTP/1.1" 200 612 "-" "kube-probe/1.32" "-" |
| --- | --- |

как видим всё ок, curl и wget работают

### []init контейнеры

напоминаю что init контейнеры запускаются последовательно.
их задача выполнить действие и завершиться.

в темплейт деплоймента нужно докинуть:

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253 | {{- with .Values.initContainers }} initContainers: {{- range $i, $ic := . }} - name: {{ default (printf "%s-init-%d" $.Chart.Name $i) $ic.name }} {{- with $ic.securityContext }} securityContext: {{- toYaml . | nindent 12 }} {{- end }} {{- with $ic.image }} image: "{{ .repository }}{{- if .tag }}:{{ .tag }}{{- end }}" {{- if .pullPolicy }} imagePullPolicy: {{ .pullPolicy }} {{- end }} {{- end }} {{- with $ic.command }} command: {{- toYaml . | nindent 12 }} {{- end }} {{- with $ic.args }} args: {{- toYaml . | nindent 12 }} {{- end }} {{- with $ic.workingDir }} workingDir: {{ . | quote }} {{- end }} {{- with $ic.env }} env: {{- toYaml . | nindent 12 }} {{- end }} {{- with $ic.envFrom }} envFrom: {{- toYaml . | nindent 12 }} {{- end }} {{- with $ic.resources }} resources: {{- toYaml . | nindent 12 }} {{- end }} {{- with $ic.volumeMounts }} volumeMounts: {{- toYaml . | nindent 12 }} {{- end }} {{- with $ic.extraContainerSpec }} {{- toYaml . | nindent 10 }} {{- end }} {{- end }}{{- end }} |
| --- | --- |

итоговый деплоймент выглядит так:

/etc/ansible/kubespray-official/helm-charts/10_containers_2_init/common-chart/templates/deployment.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210211212213214215216217218219220221222223224225226227228229230231232233234235236237238239240241242243244245246247248249250251252253254255256257258259260261262263264265266267268269270271272273274275276277278279280 | apiVersion: apps/v1kind: Deploymentmetadata: name: {{ include "common-chart.fullname" . }} labels: {{- include "common-chart.labels" . | nindent 4 }}spec: {{- if not .Values.autoscaling.enabled }} replicas: {{ .Values.replicaCount }} {{- end }} # --- RollingUpdate strategy + параметры выката --- {{- if .Values.updateStrategy.enabled }} strategy: type: {{ .Values.updateStrategy.type | default "RollingUpdate" }} {{- if ne (.Values.updateStrategy.type | default "RollingUpdate") "Recreate" }} rollingUpdate: maxSurge: {{ .Values.updateStrategy.rollingUpdate.maxSurge | default "25%" }} maxUnavailable: {{ .Values.updateStrategy.rollingUpdate.maxUnavailable | default 0 }} {{- end }} minReadySeconds: {{ .Values.updateStrategy.minReadySeconds | default 0 }} revisionHistoryLimit: {{ .Values.updateStrategy.revisionHistoryLimit | default 10 }} progressDeadlineSeconds: {{ .Values.updateStrategy.progressDeadlineSeconds | default 600 }} {{- end }} # --- /RollingUpdate --- selector: matchLabels: {{- include "common-chart.selectorLabels" . | nindent 6 }} template: metadata: # Аннотация reloader добавляется автоматически, если включён vault_secret и указан имя секрета annotations: {{- if and .Values.vault_secret.enabled .Values.vault_secret.name }} secret.reloader.stakater.com/reload: "{{ .Values.vault_secret.name }}" {{- end }} {{- with .Values.podAnnotations }} {{- toYaml . | nindent 8 }} {{- end }} labels: {{- include "common-chart.labels" . | nindent 8 }} {{- with .Values.podLabels }} {{- toYaml . | nindent 8 }} {{- end }} spec: {{- with .Values.imagePullSecrets }} imagePullSecrets: {{- toYaml . | nindent 8 }} {{- end }} serviceAccountName: {{ include "common-chart.serviceAccountName" . }} {{- if .Values.podSecurityContext }} securityContext: {{- toYaml .Values.podSecurityContext | nindent 8 }} {{- else }} securityContext: {} {{- end }} {{/* Настройки подключения секрета: безопасные дефолты (без хомпинга '-') */}} {{ $attach := default (dict) .Values.vault_secret.attach }} {{ $asEnv := default true (get $attach "asEnv") }} {{ $asVol := default false (get $attach "asVolume") }} {{ $mountPath := default "/etc/app/secret" (get $attach "mountPath") }} {{- /* ===== INIT CONTAINERS ===== */}} {{- with .Values.initContainers }} initContainers: {{- range $i, $ic := . }} - name: {{ default (printf "%s-init-%d" $.Chart.Name $i) $ic.name }} {{- with $ic.securityContext }} securityContext: {{- toYaml . | nindent 12 }} {{- end }} {{- with $ic.image }} image: "{{ .repository }}{{- if .tag }}:{{ .tag }}{{- end }}" {{- if .pullPolicy }} imagePullPolicy: {{ .pullPolicy }} {{- end }} {{- end }} {{- with $ic.command }} command: {{- toYaml . | nindent 12 }} {{- end }} {{- with $ic.args }} args: {{- toYaml . | nindent 12 }} {{- end }} {{- with $ic.workingDir }} workingDir: {{ . | quote }} {{- end }} {{- with $ic.env }} env: {{- toYaml . | nindent 12 }} {{- end }} {{- with $ic.envFrom }} envFrom: {{- toYaml . | nindent 12 }} {{- end }} {{- with $ic.resources }} resources: {{- toYaml . | nindent 12 }} {{- end }} {{- with $ic.volumeMounts }} volumeMounts: {{- toYaml . | nindent 12 }} {{- end }} {{- with $ic.extraContainerSpec }} {{- toYaml . | nindent 10 }} {{- end }} {{- end }} {{- end }} {{- /* ===== /INIT CONTAINERS ===== */}} containers: - name: {{ .Chart.Name }} securityContext: {{- toYaml .Values.securityContext | nindent 12 }} image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}" imagePullPolicy: {{ .Values.image.pullPolicy }} ports: - name: http containerPort: {{ .Values.service.port }} protocol: TCP {{- if and .Values.vault_secret.enabled $asEnv .Values.vault_secret.name }} envFrom: - secretRef: name: {{ .Values.vault_secret.name }} {{- end }} livenessProbe: {{- toYaml .Values.livenessProbe | nindent 12 }} readinessProbe: {{- toYaml .Values.readinessProbe | nindent 12 }} resources: {{- toYaml .Values.resources | nindent 12 }} {{- $needSecretVol := and .Values.vault_secret.enabled $asVol .Values.vault_secret.name }} {{- if or $needSecretVol .Values.volumeMounts }} volumeMounts: {{- if $needSecretVol }} - name: vault-secret mountPath: {{ $mountPath }} readOnly: true {{- end }} {{- with .Values.volumeMounts }} {{- toYaml . | nindent 12 }} {{- end }} {{- end }} {{- /* ===== САЙДКАРЫ: несколько контейнеров через range ===== */}} {{- with .Values.sidecars }} {{- range $i, $sc := . }} - name: {{ default (printf "%s-sidecar-%d" $.Chart.Name $i) $sc.name }} {{- with $sc.securityContext }} securityContext: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.image }} image: "{{ .repository }}{{- if .tag }}:{{ .tag }}{{- end }}" {{- if .pullPolicy }} imagePullPolicy: {{ .pullPolicy }} {{- end }} {{- end }} {{- with $sc.command }} command: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.args }} args: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.workingDir }} workingDir: {{ . | quote }} {{- end }} {{- with $sc.ports }} ports: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.env }} env: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.envFrom }} envFrom: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.livenessProbe }} livenessProbe: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.readinessProbe }} readinessProbe: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.startupProbe }} startupProbe: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.resources }} resources: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.volumeMounts }} volumeMounts: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.lifecycle }} lifecycle: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.terminationMessagePath }} terminationMessagePath: {{ . | quote }} {{- end }} {{- with $sc.terminationMessagePolicy }} terminationMessagePolicy: {{ . | quote }} {{- end }} {{- with $sc.extraContainerSpec }} {{- toYaml . | nindent 10 }} {{- end }} {{- end }} {{- end }} {{- /* ===== /САЙДКАРЫ ===== */}} {{- $needSecretVol := and .Values.vault_secret.enabled $asVol .Values.vault_secret.name }} {{- if or $needSecretVol .Values.volumes }} volumes: {{- if $needSecretVol }} - name: vault-secret secret: secretName: {{ .Values.vault_secret.name }} {{- end }} {{- with .Values.volumes }} {{- toYaml . | nindent 8 }} {{- end }} {{- end }} {{- with .Values.nodeSelector }} nodeSelector: {{- toYaml . | nindent 8 }} {{- end }} {{- with .Values.affinity }} affinity: {{- toYaml . | nindent 8 }} {{- end }} {{- if and .Values.topologySpread.enabled .Values.topologySpread.constraints }} topologySpreadConstraints: {{- range .Values.topologySpread.constraints }} - maxSkew: {{ .maxSkew | default 1 }} topologyKey: {{ .topologyKey | quote }} whenUnsatisfiable: {{ .whenUnsatisfiable | default "DoNotSchedule" | quote }} {{- if hasKey . "minDomains" }} minDomains: {{ .minDomains }} {{- end }} labelSelector: matchLabels: {{- include "common-chart.selectorLabels" $ | nindent 14 }} {{- end }} {{- end }} {{- with .Values.tolerations }} tolerations: {{- toYaml . | nindent 8 }} {{- end }} |
| --- | --- |

вельюс выглядит так:

/etc/ansible/kubespray-official/helm-charts/10_containers_2_init/common-chart/values.yaml

| 12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061 | ######### инит контейнеры ########## Общий том (если нужно готовить файлы/права в init)volumes: - name: app-code emptyDir: {}# Смонтировать общий том в основной контейнер (опционально)volumeMounts: - name: app-code mountPath: /var/www/html# Набор init-контейнеровinitContainers: # 1) Подготовка файлов - name: init-files image: repository: busybox tag: "1.36" pullPolicy: IfNotPresent command: - sh - -c - | mkdir -p /var/www/html echo "init generated $(date -u +%FT%TZ)" > /var/www/html/README.txt volumeMounts: - name: app-code mountPath: /var/www/html resources: requests: cpu: 5m memory: 16Mi limits: cpu: 50m memory: 64Mi # 2) Подготовка прав - name: init-permissions image: repository: busybox tag: "1.36" pullPolicy: IfNotPresent command: - sh - -c - | chown -R 101:101 /var/www/html || true ls -lah /var/www/ echo "" cat /var/www/html/README.txt volumeMounts: - name: app-code mountPath: /var/www/html resources: requests: cpu: 5m memory: 16Mi limits: cpu: 50m memory: 64Mi |
| --- | --- |

ставим

root@kub-master1:~/helm-charts/10_containers_2_init# helm upgrade --install app -n app ./common-chart -f ./common-chart/values.yaml

проверяем:

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210211212213214215216217218219220221222223224225226227228229230231232233234235236237 | root@kub-master1:~/helm-charts/10_containers_2_init# kubectl get pod -n appNAME READY STATUS RESTARTS AGEapp-5ff65f7547-cjt7x 4/4 Running 0 84sroot@kub-master1:~/helm-charts/10_containers_2_init# kubectl describe pod -n app app-5ff65f7547-cjt7x Name: app-5ff65f7547-cjt7xNamespace: appPriority: 0Service Account: appNode: kub-worker2.test.local/192.168.1.116Start Time: Sun, 14 Sep 2025 13:14:15 +0600Labels: app.kubernetes.io/instance=app app.kubernetes.io/managed-by=Helm app.kubernetes.io/name=common-chart app.kubernetes.io/version=1.16.0 helm.sh/chart=common-chart-0.1.10 pod-template-hash=5ff65f7547Annotations: cni.projectcalico.org/containerID: d46590beb6ab610529412a1da62a6c68304dff1efa834b3f7005995c6a201e37 cni.projectcalico.org/podIP: 10.233.107.178/32 cni.projectcalico.org/podIPs: 10.233.107.178/32 kubectl.kubernetes.io/restartedAt: 2025-08-31T11:42:43+06:00 secret.reloader.stakater.com/reload: app-secret vso.secrets.hashicorp.com/restartedAt: 2025-08-23T16:25:33ZStatus: RunningIP: 10.233.107.178IPs: IP: 10.233.107.178Controlled By: ReplicaSet/app-5ff65f7547Init Containers: init-files: Container ID: containerd://58e57d7e545042a07132b848ca0e52455fa52881351bc56afa485ec043a39cc7 Image: busybox:1.36 Image ID: docker.io/library/busybox@sha256:303337d3d52898b33018796d760d3a01c2dcc87973dd66ec22fe78ae1e8429c9 Port: <none> Host Port: <none> Command: sh -c mkdir -p /var/www/html echo "init generated $(date -u +%FT%TZ)" > /var/www/html/README.txt State: Terminated Reason: Completed Exit Code: 0 Started: Sun, 14 Sep 2025 13:14:16 +0600 Finished: Sun, 14 Sep 2025 13:14:16 +0600 Ready: True Restart Count: 0 Limits: cpu: 50m memory: 64Mi Requests: cpu: 5m memory: 16Mi Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro) /var/www/html from app-code (rw) init-permissions: Container ID: containerd://4481d48f22ba6dfb1f4b146c57e526e299c8824778d4ad6368538f34606c19ea Image: busybox:1.36 Image ID: docker.io/library/busybox@sha256:303337d3d52898b33018796d760d3a01c2dcc87973dd66ec22fe78ae1e8429c9 Port: <none> Host Port: <none> Command: sh -c chown -R 101:101 /var/www/html || true ls -lah /var/www/ echo "" cat /var/www/html/README.txt State: Terminated Reason: Completed Exit Code: 0 Started: Sun, 14 Sep 2025 13:14:17 +0600 Finished: Sun, 14 Sep 2025 13:14:17 +0600 Ready: True Restart Count: 0 Limits: cpu: 50m memory: 64Mi Requests: cpu: 5m memory: 16Mi Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro) /var/www/html from app-code (rw)Containers: common-chart: Container ID: containerd://2975418db6fe279fe67dc28204f73d8ba312f6cc319c8de7db0d9e0714806fb9 Image: nginx:1.16.0 Image ID: docker.io/library/nginx@sha256:3e373fd5b8d41baeddc24be311c5c6929425c04cabf893b874ac09b72a798010 Port: 80/TCP Host Port: 0/TCP State: Running Started: Sun, 14 Sep 2025 13:14:18 +0600 Ready: True Restart Count: 0 Limits: cpu: 100m memory: 128Mi Requests: cpu: 100m memory: 128Mi Liveness: http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3 Readiness: http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3 Environment Variables from: app-secret Secret Optional: false Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro) /var/www/html from app-code (rw) curl-pinger: Container ID: containerd://ac246c761f62f941a3f78485596e8c0011f3b44bcffbe8fd155732dfcdf629b8 Image: curlimages/curl:latest Image ID: docker.io/curlimages/curl@sha256:463eaf6072688fe96ac64fa623fe73e1dbe25d8ad6c34404a669ad3ce1f104b6 Port: <none> Host Port: <none> Command: sh -c while true; do code=$(curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:80/ || true) echo "$(date -u +%FT%TZ) GET / -> ${code}" sleep 5 done State: Running Started: Sun, 14 Sep 2025 13:14:18 +0600 Ready: True Restart Count: 0 Limits: cpu: 50m memory: 64Mi Requests: cpu: 5m memory: 16Mi Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro) wget-pinger: Container ID: containerd://120af387522574748d26dae37144f86e80c625683c2fd12f8f03f0df10fd5b9d Image: busybox:1.36 Image ID: docker.io/library/busybox@sha256:303337d3d52898b33018796d760d3a01c2dcc87973dd66ec22fe78ae1e8429c9 Port: <none> Host Port: <none> Command: sh -c while true; do wget -q -S -O /dev/null http://127.0.0.1:80/ 2>&1 | awk 'NR==1{print strftime("%Y-%m-%dT%H:%M:%SZ"), $0}' sleep 5 done State: Running Started: Sun, 14 Sep 2025 13:14:18 +0600 Ready: True Restart Count: 0 Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro) php-fpm: Container ID: containerd://744ee7f76cd1417ad1a24ee04ee8f1f7447736beb4b815979b09ec2b8c2aa58d Image: php:8.2-fpm-alpine Image ID: docker.io/library/php@sha256:b7355fb38ef93fd0ef6ccbecee006a25f9bb7fbb65a9f6b7a4721582b75c073a Port: 9000/TCP Host Port: 0/TCP Command: sh -c mkdir -p /var/www/html [ -f /var/www/html/index.php ] || echo '<?php echo "Hello from PHP-FPM @ " . date("c"); ?>' > /var/www/html/index.php # слушаем на 0.0.0.0:9000 (явно) sed -ri 's/^;?listen\s*=.*$/listen = 0.0.0.0:9000/' /usr/local/etc/php-fpm.d/zz-docker.conf exec php-fpm -F State: Running Started: Sun, 14 Sep 2025 13:14:18 +0600 Ready: True Restart Count: 0 Limits: cpu: 200m memory: 256Mi Requests: cpu: 50m memory: 64Mi Liveness: tcp-socket :fpm delay=5s timeout=1s period=10s #success=1 #failure=3 Readiness: tcp-socket :fpm delay=2s timeout=1s period=5s #success=1 #failure=3 Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro)Conditions: Type Status PodReadyToStartContainers True Initialized True Ready True ContainersReady True PodScheduled True Volumes: app-code: Type: EmptyDir (a temporary directory that shares a pod's lifetime) Medium: SizeLimit: <unset> kube-api-access-7flqn: Type: Projected (a volume that contains injected data from multiple sources) TokenExpirationSeconds: 3607 ConfigMapName: kube-root-ca.crt ConfigMapOptional: <nil> DownwardAPI: trueQoS Class: BurstableNode-Selectors: <none>Tolerations: node.kubernetes.io/not-ready:NoExecute op=Exists for 300s node.kubernetes.io/unreachable:NoExecute op=Exists for 300sTopology Spread Constraints: kubernetes.io/hostname:ScheduleAnyway when max skew 2 is exceeded for selector app.kubernetes.io/instance=app,app.kubernetes.io/name=common-chartEvents: Type Reason Age From Message ---- ------ ---- ---- ------- Normal Scheduled 95s default-scheduler Successfully assigned app/app-5ff65f7547-cjt7x to kub-worker2.test.local Normal Pulled 94s kubelet Container image "busybox:1.36" already present on machine Normal Created 94s kubelet Created container: init-files Normal Started 94s kubelet Started container init-files Normal Pulled 93s kubelet Container image "busybox:1.36" already present on machine Normal Created 93s kubelet Created container: init-permissions Normal Started 93s kubelet Started container init-permissions Normal Pulled 92s kubelet Container image "nginx:1.16.0" already present on machine Normal Created 92s kubelet Created container: common-chart Normal Started 92s kubelet Started container common-chart Normal Pulled 92s kubelet Container image "curlimages/curl:latest" already present on machine Normal Created 92s kubelet Created container: curl-pinger Normal Started 92s kubelet Started container curl-pinger Normal Pulled 92s kubelet Container image "busybox:1.36" already present on machine Normal Created 92s kubelet Created container: wget-pinger Normal Started 92s kubelet Started container wget-pinger Normal Pulled 92s kubelet Container image "php:8.2-fpm-alpine" already present on machine Normal Created 92s kubelet Created container: php-fpm Normal Started 92s kubelet Started container php-fpm |
| --- | --- |

как видим 2 init container

посмотрим всё ли они выполнили:

| 12345678910111213141516171819202122 | root@kub-master1:~/helm-charts/10_containers_2_init# kubectl exec -ti -n app app-5ff65f7547-cjt7x -- shDefaulted container "common-chart" out of: common-chart, curl-pinger, wget-pinger, php-fpm, init-files (init), init-permissions (init)# bashroot@app-5ff65f7547-cjt7x:/# ls -lah /var/www/total 16Kdrwxr-xr-x 3 root root 4.0K Sep 14 07:14 .drwxr-xr-x 1 root root 4.0K Sep 14 07:14 ..drwxrwxrwx 2 nginx nginx 4.0K Sep 14 07:14 htmlroot@app-5ff65f7547-cjt7x:/# ls -lah /var/www/html/total 12Kdrwxrwxrwx 2 nginx nginx 4.0K Sep 14 07:14 .drwxr-xr-x 3 root root 4.0K Sep 14 07:14 ..-rw-r--r-- 1 nginx nginx 36 Sep 14 07:14 README.txtroot@app-5ff65f7547-cjt7x:/# cat /var/www/html/README.txt init generated 2025-09-14T07:14:16Z |
| --- | --- |

как видим один init контейнер подготовил директорию и файл а второй контейнер назначил права

напоминание:

- 

Любой`volumeMounts[].name`в init-контейнере должен существовать в`volumes[]`.

- 

Init-контейнеры выполняются**последовательно**; основной Pod стартует только после их завершения.

### []ephemeral Containers (debug) контейнеры

как уже говорилось это контейнеры подкидываются к существующим подам. они не добавляются отдельно к деплойменту.

root@kub-master1:~/helm-charts/10_containers_2_init# kubectl get pod -n app
NAMEREADYSTATUSRESTARTSAGE
app-5ff65f7547-cjt7x 4/4 Running 0 86m
root@kub-master1:~/helm-charts/10_containers_2_init#NS=app
root@kub-master1:~/helm-charts/10_containers_2_init#TARGET=common-chart
root@kub-master1:~/helm-charts/10_containers_2_init#POD=app-5ff65f7547-cjt7x

|  | # Лёгкий busybox shell в неймспейсах TARGETkubectl debug -n $NS pod/$POD -it \ --image=busybox:1.36 \ --target=$TARGET \ -- /bin/sh |
| --- | --- |

проверим:

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210211212213214215216217218219220221222223224225226227228229230231232233234235236 | root@kub-master1:~# kubectl get pod -n appNAME READY STATUS RESTARTS AGEapp-5ff65f7547-cjt7x 4/4 Running 0 87mroot@kub-master1:~# kubectl describe pod -n app app-5ff65f7547-cjt7x Name: app-5ff65f7547-cjt7xNamespace: appPriority: 0Service Account: appNode: kub-worker2.test.local/192.168.1.116Start Time: Sun, 14 Sep 2025 13:14:15 +0600Labels: app.kubernetes.io/instance=app app.kubernetes.io/managed-by=Helm app.kubernetes.io/name=common-chart app.kubernetes.io/version=1.16.0 helm.sh/chart=common-chart-0.1.10 pod-template-hash=5ff65f7547Annotations: cni.projectcalico.org/containerID: d46590beb6ab610529412a1da62a6c68304dff1efa834b3f7005995c6a201e37 cni.projectcalico.org/podIP: 10.233.107.178/32 cni.projectcalico.org/podIPs: 10.233.107.178/32 kubectl.kubernetes.io/restartedAt: 2025-08-31T11:42:43+06:00 secret.reloader.stakater.com/reload: app-secret vso.secrets.hashicorp.com/restartedAt: 2025-08-23T16:25:33ZStatus: RunningIP: 10.233.107.178IPs: IP: 10.233.107.178Controlled By: ReplicaSet/app-5ff65f7547Init Containers: init-files: Container ID: containerd://58e57d7e545042a07132b848ca0e52455fa52881351bc56afa485ec043a39cc7 Image: busybox:1.36 Image ID: docker.io/library/busybox@sha256:303337d3d52898b33018796d760d3a01c2dcc87973dd66ec22fe78ae1e8429c9 Port: <none> Host Port: <none> Command: sh -c mkdir -p /var/www/html echo "init generated $(date -u +%FT%TZ)" > /var/www/html/README.txt State: Terminated Reason: Completed Exit Code: 0 Started: Sun, 14 Sep 2025 13:14:16 +0600 Finished: Sun, 14 Sep 2025 13:14:16 +0600 Ready: True Restart Count: 0 Limits: cpu: 50m memory: 64Mi Requests: cpu: 5m memory: 16Mi Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro) /var/www/html from app-code (rw) init-permissions: Container ID: containerd://4481d48f22ba6dfb1f4b146c57e526e299c8824778d4ad6368538f34606c19ea Image: busybox:1.36 Image ID: docker.io/library/busybox@sha256:303337d3d52898b33018796d760d3a01c2dcc87973dd66ec22fe78ae1e8429c9 Port: <none> Host Port: <none> Command: sh -c chown -R 101:101 /var/www/html || true ls -lah /var/www/ echo "" cat /var/www/html/README.txt State: Terminated Reason: Completed Exit Code: 0 Started: Sun, 14 Sep 2025 13:14:17 +0600 Finished: Sun, 14 Sep 2025 13:14:17 +0600 Ready: True Restart Count: 0 Limits: cpu: 50m memory: 64Mi Requests: cpu: 5m memory: 16Mi Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro) /var/www/html from app-code (rw)Containers: common-chart: Container ID: containerd://2975418db6fe279fe67dc28204f73d8ba312f6cc319c8de7db0d9e0714806fb9 Image: nginx:1.16.0 Image ID: docker.io/library/nginx@sha256:3e373fd5b8d41baeddc24be311c5c6929425c04cabf893b874ac09b72a798010 Port: 80/TCP Host Port: 0/TCP State: Running Started: Sun, 14 Sep 2025 13:14:18 +0600 Ready: True Restart Count: 0 Limits: cpu: 100m memory: 128Mi Requests: cpu: 100m memory: 128Mi Liveness: http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3 Readiness: http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3 Environment Variables from: app-secret Secret Optional: false Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro) /var/www/html from app-code (rw) curl-pinger: Container ID: containerd://ac246c761f62f941a3f78485596e8c0011f3b44bcffbe8fd155732dfcdf629b8 Image: curlimages/curl:latest Image ID: docker.io/curlimages/curl@sha256:463eaf6072688fe96ac64fa623fe73e1dbe25d8ad6c34404a669ad3ce1f104b6 Port: <none> Host Port: <none> Command: sh -c while true; do code=$(curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:80/ || true) echo "$(date -u +%FT%TZ) GET / -> ${code}" sleep 5 done State: Running Started: Sun, 14 Sep 2025 13:14:18 +0600 Ready: True Restart Count: 0 Limits: cpu: 50m memory: 64Mi Requests: cpu: 5m memory: 16Mi Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro) wget-pinger: Container ID: containerd://120af387522574748d26dae37144f86e80c625683c2fd12f8f03f0df10fd5b9d Image: busybox:1.36 Image ID: docker.io/library/busybox@sha256:303337d3d52898b33018796d760d3a01c2dcc87973dd66ec22fe78ae1e8429c9 Port: <none> Host Port: <none> Command: sh -c while true; do wget -q -S -O /dev/null http://127.0.0.1:80/ 2>&1 | awk 'NR==1{print strftime("%Y-%m-%dT%H:%M:%SZ"), $0}' sleep 5 done State: Running Started: Sun, 14 Sep 2025 13:14:18 +0600 Ready: True Restart Count: 0 Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro) php-fpm: Container ID: containerd://744ee7f76cd1417ad1a24ee04ee8f1f7447736beb4b815979b09ec2b8c2aa58d Image: php:8.2-fpm-alpine Image ID: docker.io/library/php@sha256:b7355fb38ef93fd0ef6ccbecee006a25f9bb7fbb65a9f6b7a4721582b75c073a Port: 9000/TCP Host Port: 0/TCP Command: sh -c mkdir -p /var/www/html [ -f /var/www/html/index.php ] || echo '<?php echo "Hello from PHP-FPM @ " . date("c"); ?>' > /var/www/html/index.php # слушаем на 0.0.0.0:9000 (явно) sed -ri 's/^;?listen\s*=.*$/listen = 0.0.0.0:9000/' /usr/local/etc/php-fpm.d/zz-docker.conf exec php-fpm -F State: Running Started: Sun, 14 Sep 2025 13:14:18 +0600 Ready: True Restart Count: 0 Limits: cpu: 200m memory: 256Mi Requests: cpu: 50m memory: 64Mi Liveness: tcp-socket :fpm delay=5s timeout=1s period=10s #success=1 #failure=3 Readiness: tcp-socket :fpm delay=2s timeout=1s period=5s #success=1 #failure=3 Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro)Ephemeral Containers: debugger-r2w6q: Container ID: containerd://5dba57c05cfc5adf60e049f8cd5f4c16830f86e42ea259843868909944715734 Image: busybox:1.36 Image ID: docker.io/library/busybox@sha256:303337d3d52898b33018796d760d3a01c2dcc87973dd66ec22fe78ae1e8429c9 Port: <none> Host Port: <none> Command: /bin/sh State: Running Started: Sun, 14 Sep 2025 14:41:21 +0600 Ready: False Restart Count: 0 Environment: <none> Mounts: <none>Conditions: Type Status PodReadyToStartContainers True Initialized True Ready True ContainersReady True PodScheduled True Volumes: app-code: Type: EmptyDir (a temporary directory that shares a pod's lifetime) Medium: SizeLimit: <unset> kube-api-access-7flqn: Type: Projected (a volume that contains injected data from multiple sources) TokenExpirationSeconds: 3607 ConfigMapName: kube-root-ca.crt ConfigMapOptional: <nil> DownwardAPI: trueQoS Class: BurstableNode-Selectors: <none>Tolerations: node.kubernetes.io/not-ready:NoExecute op=Exists for 300s node.kubernetes.io/unreachable:NoExecute op=Exists for 300sTopology Spread Constraints: kubernetes.io/hostname:ScheduleAnyway when max skew 2 is exceeded for selector app.kubernetes.io/instance=app,app.kubernetes.io/name=common-chartEvents: Type Reason Age From Message ---- ------ ---- ---- ------- Normal Pulled 47s kubelet Container image "busybox:1.36" already present on machine Normal Created 47s kubelet Created container: debugger-r2w6q Normal Started 47s kubelet Started container debugger-r2w6q |
| --- | --- |

как видим появился

Ephemeral Containers:
debugger-r2w6q:

можено ещё запустить:

|  | # netshoot с кучей сетевых утилитkubectl debug -n $NS pod/$POD -it \ --image=nicolaka/netshoot \ --target=$TARGET \ -- bash |
| --- | --- |

проверим дескрайб

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210211212213214215216217218219220221222223224225226227228229230231232233234235236237238239240241242243244245246247248249250251252253254 | root@kub-master1:~# kubectl describe pod -n app app-5ff65f7547-cjt7x Name: app-5ff65f7547-cjt7xNamespace: appPriority: 0Service Account: appNode: kub-worker2.test.local/192.168.1.116Start Time: Sun, 14 Sep 2025 13:14:15 +0600Labels: app.kubernetes.io/instance=app app.kubernetes.io/managed-by=Helm app.kubernetes.io/name=common-chart app.kubernetes.io/version=1.16.0 helm.sh/chart=common-chart-0.1.10 pod-template-hash=5ff65f7547Annotations: cni.projectcalico.org/containerID: d46590beb6ab610529412a1da62a6c68304dff1efa834b3f7005995c6a201e37 cni.projectcalico.org/podIP: 10.233.107.178/32 cni.projectcalico.org/podIPs: 10.233.107.178/32 kubectl.kubernetes.io/restartedAt: 2025-08-31T11:42:43+06:00 secret.reloader.stakater.com/reload: app-secret vso.secrets.hashicorp.com/restartedAt: 2025-08-23T16:25:33ZStatus: RunningIP: 10.233.107.178IPs: IP: 10.233.107.178Controlled By: ReplicaSet/app-5ff65f7547Init Containers: init-files: Container ID: containerd://58e57d7e545042a07132b848ca0e52455fa52881351bc56afa485ec043a39cc7 Image: busybox:1.36 Image ID: docker.io/library/busybox@sha256:303337d3d52898b33018796d760d3a01c2dcc87973dd66ec22fe78ae1e8429c9 Port: <none> Host Port: <none> Command: sh -c mkdir -p /var/www/html echo "init generated $(date -u +%FT%TZ)" > /var/www/html/README.txt State: Terminated Reason: Completed Exit Code: 0 Started: Sun, 14 Sep 2025 13:14:16 +0600 Finished: Sun, 14 Sep 2025 13:14:16 +0600 Ready: True Restart Count: 0 Limits: cpu: 50m memory: 64Mi Requests: cpu: 5m memory: 16Mi Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro) /var/www/html from app-code (rw) init-permissions: Container ID: containerd://4481d48f22ba6dfb1f4b146c57e526e299c8824778d4ad6368538f34606c19ea Image: busybox:1.36 Image ID: docker.io/library/busybox@sha256:303337d3d52898b33018796d760d3a01c2dcc87973dd66ec22fe78ae1e8429c9 Port: <none> Host Port: <none> Command: sh -c chown -R 101:101 /var/www/html || true ls -lah /var/www/ echo "" cat /var/www/html/README.txt State: Terminated Reason: Completed Exit Code: 0 Started: Sun, 14 Sep 2025 13:14:17 +0600 Finished: Sun, 14 Sep 2025 13:14:17 +0600 Ready: True Restart Count: 0 Limits: cpu: 50m memory: 64Mi Requests: cpu: 5m memory: 16Mi Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro) /var/www/html from app-code (rw)Containers: common-chart: Container ID: containerd://2975418db6fe279fe67dc28204f73d8ba312f6cc319c8de7db0d9e0714806fb9 Image: nginx:1.16.0 Image ID: docker.io/library/nginx@sha256:3e373fd5b8d41baeddc24be311c5c6929425c04cabf893b874ac09b72a798010 Port: 80/TCP Host Port: 0/TCP State: Running Started: Sun, 14 Sep 2025 13:14:18 +0600 Ready: True Restart Count: 0 Limits: cpu: 100m memory: 128Mi Requests: cpu: 100m memory: 128Mi Liveness: http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3 Readiness: http-get http://:http/ delay=0s timeout=1s period=10s #success=1 #failure=3 Environment Variables from: app-secret Secret Optional: false Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro) /var/www/html from app-code (rw) curl-pinger: Container ID: containerd://ac246c761f62f941a3f78485596e8c0011f3b44bcffbe8fd155732dfcdf629b8 Image: curlimages/curl:latest Image ID: docker.io/curlimages/curl@sha256:463eaf6072688fe96ac64fa623fe73e1dbe25d8ad6c34404a669ad3ce1f104b6 Port: <none> Host Port: <none> Command: sh -c while true; do code=$(curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:80/ || true) echo "$(date -u +%FT%TZ) GET / -> ${code}" sleep 5 done State: Running Started: Sun, 14 Sep 2025 13:14:18 +0600 Ready: True Restart Count: 0 Limits: cpu: 50m memory: 64Mi Requests: cpu: 5m memory: 16Mi Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro) wget-pinger: Container ID: containerd://120af387522574748d26dae37144f86e80c625683c2fd12f8f03f0df10fd5b9d Image: busybox:1.36 Image ID: docker.io/library/busybox@sha256:303337d3d52898b33018796d760d3a01c2dcc87973dd66ec22fe78ae1e8429c9 Port: <none> Host Port: <none> Command: sh -c while true; do wget -q -S -O /dev/null http://127.0.0.1:80/ 2>&1 | awk 'NR==1{print strftime("%Y-%m-%dT%H:%M:%SZ"), $0}' sleep 5 done State: Running Started: Sun, 14 Sep 2025 13:14:18 +0600 Ready: True Restart Count: 0 Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro) php-fpm: Container ID: containerd://744ee7f76cd1417ad1a24ee04ee8f1f7447736beb4b815979b09ec2b8c2aa58d Image: php:8.2-fpm-alpine Image ID: docker.io/library/php@sha256:b7355fb38ef93fd0ef6ccbecee006a25f9bb7fbb65a9f6b7a4721582b75c073a Port: 9000/TCP Host Port: 0/TCP Command: sh -c mkdir -p /var/www/html [ -f /var/www/html/index.php ] || echo '<?php echo "Hello from PHP-FPM @ " . date("c"); ?>' > /var/www/html/index.php # слушаем на 0.0.0.0:9000 (явно) sed -ri 's/^;?listen\s*=.*$/listen = 0.0.0.0:9000/' /usr/local/etc/php-fpm.d/zz-docker.conf exec php-fpm -F State: Running Started: Sun, 14 Sep 2025 13:14:18 +0600 Ready: True Restart Count: 0 Limits: cpu: 200m memory: 256Mi Requests: cpu: 50m memory: 64Mi Liveness: tcp-socket :fpm delay=5s timeout=1s period=10s #success=1 #failure=3 Readiness: tcp-socket :fpm delay=2s timeout=1s period=5s #success=1 #failure=3 Environment: <none> Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7flqn (ro)Ephemeral Containers: debugger-r2w6q: Container ID: containerd://5dba57c05cfc5adf60e049f8cd5f4c16830f86e42ea259843868909944715734 Image: busybox:1.36 Image ID: docker.io/library/busybox@sha256:303337d3d52898b33018796d760d3a01c2dcc87973dd66ec22fe78ae1e8429c9 Port: <none> Host Port: <none> Command: /bin/sh State: Terminated Reason: Error Exit Code: 130 Started: Sun, 14 Sep 2025 14:41:21 +0600 Finished: Sun, 14 Sep 2025 14:44:31 +0600 Ready: False Restart Count: 0 Environment: <none> Mounts: <none> debugger-h5dtc: Container ID: containerd://34d6a8aaddf28d566a616bd670e219327b176b1148d7ea97543b56fe219c9d66 Image: nicolaka/netshoot Image ID: docker.io/nicolaka/netshoot@sha256:7f08c4aff13ff61a35d30e30c5c1ea8396eac6ab4ce19fd02d5a4b3b5d0d09a2 Port: <none> Host Port: <none> Command: bash State: Running Started: Sun, 14 Sep 2025 14:45:11 +0600 Ready: False Restart Count: 0 Environment: <none> Mounts: <none>Conditions: Type Status PodReadyToStartContainers True Initialized True Ready True ContainersReady True PodScheduled True Volumes: app-code: Type: EmptyDir (a temporary directory that shares a pod's lifetime) Medium: SizeLimit: <unset> kube-api-access-7flqn: Type: Projected (a volume that contains injected data from multiple sources) TokenExpirationSeconds: 3607 ConfigMapName: kube-root-ca.crt ConfigMapOptional: <nil> DownwardAPI: trueQoS Class: BurstableNode-Selectors: <none>Tolerations: node.kubernetes.io/not-ready:NoExecute op=Exists for 300s node.kubernetes.io/unreachable:NoExecute op=Exists for 300sTopology Spread Constraints: kubernetes.io/hostname:ScheduleAnyway when max skew 2 is exceeded for selector app.kubernetes.io/instance=app,app.kubernetes.io/name=common-chartEvents: Type Reason Age From Message ---- ------ ---- ---- ------- Normal Pulled 4m1s kubelet Container image "busybox:1.36" already present on machine Normal Created 4m1s kubelet Created container: debugger-r2w6q Normal Started 4m1s kubelet Started container debugger-r2w6q Normal Pulling 46s kubelet Pulling image "nicolaka/netshoot" Normal Pulled 11s kubelet Successfully pulled image "nicolaka/netshoot" in 34.367s (34.367s including waiting). Image size: 207893848 bytes. Normal Created 11s kubelet Created container: debugger-h5dtc Normal Started 11s kubelet Started container debugger-h5dtc |
| --- | --- |

как видим старый (busybox) debugger-r2w6q из которого мы вышли теперь в статусе Terminated, а новый debugger-h5dtc nicolaka/netshoot успешно запущен

### []Configmap (делаем связку nginx->php-fpm)

задача в одномPODу нас будет nginx и php-fpm по разным контейнерам. Эта часть уже сделана в примерах с sidecar контейнерами, дальше нам нужно добавить создание configmap и подкидывание этих конфигов к nginx и php-fpm

для начала сделаем чтобы наш common helm-chart создавал конфиги из файлов

/etc/ansible/kubespray-official/helm-charts/10_containers_3_configmap/common-chart/templates/configmaps-from-files.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748 | {{- /*Шаблон создаёт по одному ConfigMap на каждый элемент из .Values.configmaps.items.везде используем корневой контекст `$` для Files/Chart/Release.*/ -}}{{- if and .Values.configmaps .Values.configmaps.enabled }}{{- $items := default (list) .Values.configmaps.items }}{{- range $i, $cm := $items }} {{- $rawName := required (printf "configmaps.items[%d].name is required" $i) $cm.name }} {{- $file := required (printf "configmaps.items[%d].file is required" $i) $cm.file }} {{- if not ($.Files.Get $file) }} {{- fail (printf "File '%s' not found in chart for configmaps.items[%d]" $file $i) }} {{- end }} {{- $key := default (base $file) $cm.key }} {{- $isBin := default false $cm.binary }} {{- /* Нормализация имени под DNS-1123 */ -}} {{- $name := $rawName | replace "_" "-" | replace "." "-" | lower | trunc 63 | trimSuffix "-" }}apiVersion: v1kind: ConfigMapmetadata: name: {{ $name | quote }} namespace: {{ $.Release.Namespace }} labels: app.kubernetes.io/managed-by: "Helm" app.kubernetes.io/part-of: {{ $.Chart.Name | quote }} helm.sh/chart: "{{ $.Chart.Name }}-{{ $.Chart.Version }}" {{- with $cm.labels }} {{- toYaml . | nindent 4 }} {{- end }} {{- with $cm.annotations }} annotations: {{- toYaml . | nindent 4 }} {{- end }}{{- if not $isBin }}data: {{ $key }}: |-{{ $.Files.Get $file | indent 4 }}{{- else }}binaryData: {{ $key }}: {{ $.Files.GetBytes $file | toString | b64enc }}{{- end }}---{{- end }}{{- end }} |
| --- | --- |

вот краткое описание:

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113 | описание темплейта configmaps-from-files.yaml---{{- if and .Values.configmaps .Values.configmaps.enabled }}{{- $items := default (list) .Values.configmaps.items }}if and … — создаём ресурсы только если есть секция configmaps и флаг enabled: true.$items := default (list) … — локальная переменная со списком элементов. Если список не задан, подставится пустой (list) — цикл ниже просто ничего не сделает.------Цикл по элементам{{- range $i, $cm := $items }}Идём по каждому объекту-описанию конфигмапа из values:$i — индекс (0,1,2,…), удобен для сообщений об ошибке.$cm — сам элемент (map с полями name, file, key, binary, labels, annotations).------Обязательные поля и fail-fast{{- $rawName := required (printf "configmaps.items[%d].name is required" $i) $cm.name }}{{- $file := required (printf "configmaps.items[%d].file is required" $i) $cm.file }}{{- if not ($.Files.Get $file) }} {{- fail (printf "File '%s' not found in chart for configmaps.items[%d]" $file $i) }}{{- end }}required(msg, value) — если value пустое/не задано, рендер оборвётся с msg. Так валидируем, что у каждого элемента есть name и file.$.Files.Get $file — читаем файл из директории чарта (ключевой момент: путь относителен корня чарта).Используем корневой контекст $, потому что внутри range . уже указывает на $cm, а не на чарт.if not (…) fail(…) — если файла нет в чарте то фейлится------Необязательные поля и нормализация имени{{- $key := default (base $file) $cm.key }}{{- $isBin := default false $cm.binary }}{{- /* Нормализация имени под DNS-1123 */ -}}{{- $name := $rawName | replace "_" "-" | replace "." "-" | lower | trunc 63 | trimSuffix "-" }}$key — имя ключа в data/binaryData. По умолчанию — basename(file) (например, из files/conf/app.yaml получится app.yaml).$isBin — флаг «использовать binaryData» (для бинарных/не-UTF8 файлов).$name — итоговое имя ресурса:заменяем _ и . на -, приводим к нижнему регистру,trunc 63 — ограничиваем длину до 63 символов (требование DNS-1123),trimSuffix "-" — убираем - в конце, если после обрезки он остался.------Метаданные ресурсаapiVersion: v1kind: ConfigMapmetadata: name: {{ $name | quote }} namespace: {{ $.Release.Namespace }} labels: app.kubernetes.io/managed-by: "Helm" app.kubernetes.io/part-of: {{ $.Chart.Name | quote }} helm.sh/chart: "{{ $.Chart.Name }}-{{ $.Chart.Version }}" {{- with $cm.labels }} {{- toYaml . | nindent 4 }} {{- end }} {{- with $cm.annotations }} annotations: {{- toYaml . | nindent 4 }} {{- end }}name — берём нормализованное имя.namespace — неймспейс релиза (из helm install … -n), через корень $.Release.Namespace.Базовые лейблы — полезные метки для отладки/поиска.with $cm.labels / with $cm.annotations — вставляем блоки только если они есть в values.toYaml . | nindent 4 — красивое форматирование под 4 пробела.------Содержимое ConfigMap{{- if not $isBin }}data: {{ $key }}: |-{{ $.Files.Get $file | indent 4 }}{{- else }}binaryData: {{ $key }}: {{ $.Files.GetBytes $file | toString | b64enc }}{{- end }}Ветвление по типу:Текстовый ($isBin == false): кладём в data многострочной строкой (|- сохраняет переносы, indent 4 — отступ).Бинарный ($isBin == true): кладём в binaryData как base64.$.Files.GetBytes → toString (Go-tmpl хочет строку) → b64enc.для бинарей лучше реально использовать binaryData, иначе Kubernetes может ругаться на невалидный UTF-8 в data.------Разделитель документов и закрывающие теги---{{- end }}{{- end }}--- — YAML-разделитель между несколькими ресурсами (каждый элемент списка создаёт свой документ).Два end — закрывают range и внешний if.--- |
| --- | --- |

а вот сами файлы из которых мы будем делать configmap

/etc/ansible/kubespray-official/helm-charts/10_containers_3_configmap/common-chart/configmap-files/test.yaml
fgdfgdfg

/etc/ansible/kubespray-official/helm-charts/10_containers_3_configmap/common-chart/configmap-files/test2.yaml
gfdgfg11111111

/etc/ansible/kubespray-official/helm-charts/10_containers_3_configmap/common-chart/configmap-files/test3.crt
fds1213123

вот values

/etc/ansible/kubespray-official/helm-charts/10_containers_3_configmap/common-chart/values.yaml

| 12345678910111213141516171819 | configmaps: enabled: true items: - name: app-config-test # имя ConfigMap file: configmap-files/test.yaml # путь к файлу внутри чарта key: app.conf # (опционально) ключ в .data; если не задан — basename(file) labels: # (опционально) tier: backend annotations: # (опционально) reloader.stakater.com/match: "true" - name: logging file: configmap-files/test2.yaml # бинарный пример -> попадёт в binaryData (base64) - name: ca-bundle file: configmap-files/test3.crt binary: true # (опционально) по умолчанию false |
| --- | --- |

ставим

helm upgrade --install app -n app ./common-chart -f ./common-chart/values.yaml

получаем:

|  | root@kub-master1:~/helm-charts/10_containers_3_configmap# kubectl get configmaps -n appNAME DATA AGEapp-config-test 1 3sca-bundle 1 55skube-root-ca.crt 1 28dlogging 1 55s |
| --- | --- |

теперь посмотрим содержимое каждого конфигмапа:

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465 | root@kub-master1:~/helm-charts/10_containers_3_configmap# kubectl get configmaps -n app app-config-test -o yamlapiVersion: v1data: app.conf: fgdfgdfgkind: ConfigMapmetadata: annotations: meta.helm.sh/release-name: app meta.helm.sh/release-namespace: app reloader.stakater.com/match: "true" creationTimestamp: "2025-09-21T10:45:27Z" labels: app.kubernetes.io/managed-by: Helm app.kubernetes.io/part-of: common-chart helm.sh/chart: common-chart-0.1.10 tier: backend name: app-config-test namespace: app resourceVersion: "33494910" uid: 81d8109d-59f6-4d33-9cbd-4d3c387a0d51root@kub-master1:~/helm-charts/10_containers_3_configmap# kubectl get configmaps -n app logging -o yamlapiVersion: v1data: test2.yaml: gfdgfg11111111kind: ConfigMapmetadata: annotations: meta.helm.sh/release-name: app meta.helm.sh/release-namespace: app creationTimestamp: "2025-09-21T10:44:35Z" labels: app.kubernetes.io/managed-by: Helm app.kubernetes.io/part-of: common-chart helm.sh/chart: common-chart-0.1.10 name: logging namespace: app resourceVersion: "33494641" uid: e0c9c0da-854c-4826-a3c4-b2231ed17f52root@kub-master1:~/helm-charts/10_containers_3_configmap# kubectl get configmaps -n app ca-bundle -o yamlapiVersion: v1binaryData: test3.crt: ZmRzMTIxMzEyMw==kind: ConfigMapmetadata: annotations: meta.helm.sh/release-name: app meta.helm.sh/release-namespace: app creationTimestamp: "2025-09-21T10:44:35Z" labels: app.kubernetes.io/managed-by: Helm app.kubernetes.io/part-of: common-chart helm.sh/chart: common-chart-0.1.10 name: ca-bundle namespace: app resourceVersion: "33494642" uid: e5f660ae-0d55-4eb0-9452-4dc79bb612ecroot@kub-master1:~/helm-charts/10_containers_3_configmap# echo ZmRzMTIxMzEyMw== | base64 -d |
| --- | --- |

как видим все 3 создались.

теперь нам нужно подкинуть правильный конфиг в правильный sidecar

создаём нужные нам конфиги:

/etc/ansible/kubespray-official/helm-charts/10_containers_3_configmap/common-chart/configmap-files/nginx/default.conf

| 12345678910111213141516171819202122232425 | server { listen 80; server_name _; root /var/www/html; index index.php index.html index.htm; access_log /var/log/nginx/access.log; error_log /var/log/nginx/error.log; location / { try_files $uri $uri/ /index.php?$args; } location ~ \.php$ { include fastcgi_params; fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name; fastcgi_param PATH_INFO $fastcgi_path_info; fastcgi_index index.php; fastcgi_buffers 16 16k; fastcgi_buffer_size 32k; fastcgi_pass 127.0.0.1:9000; }} |
| --- | --- |

/etc/ansible/kubespray-official/helm-charts/10_containers_3_configmap/common-chart/configmap-files/php/index.php

добавляем эти 2 записи в values configmap

| 1234567891011121314151617181920212223242526 | configmaps: enabled: true items: - name: app-config-test # имя ConfigMap file: configmap-files/test.yaml # путь к файлу внутри чарта key: app.conf # (опционально) ключ в .data; если не задан — basename(file) labels: # (опционально) tier: backend annotations: # (опционально) reloader.stakater.com/match: "true" - name: logging file: configmap-files/test2.yaml # бинарный пример -> попадёт в binaryData (base64) - name: ca-bundle file: configmap-files/test3.crt binary: true # (опционально) по умолчанию false # для nginx и php-fpm чтобы запрос прилетал на nginx и проксировался на php-fpm - name: nginx-conf file: configmap-files/nginx/default.conf key: default.conf - name: php-index file: configmap-files/php/index.php key: index.php |
| --- | --- |

есть несколько добавлений для values

/etc/ansible/kubespray-official/helm-charts/10_containers_3_configmap/common-chart/values.yaml

добавляем volumes и volumeMounts это nginx-conf и php-index

| 123456789101112131415161718192021222324252627 | volumes: - name: app-code emptyDir: {} # NGINX конфиг: - name: nginx-conf configMap: name: nginx-conf items: - key: default.conf path: default.conf # PHP код (index.php): - name: php-index configMap: name: php-index items: - key: index.php path: index.phpvolumeMounts: - name: app-code mountPath: /var/www/html - name: nginx-conf mountPath: /etc/nginx/conf.d/default.conf subPath: default.conf - name: php-index mountPath: /var/www/html/index.php subPath: index.php |
| --- | --- |

а так же к

sidecars:

- name: php-fpm

добавляем

volumeMounts

| 123456789101112131415161718192021222324252627282930313233343536 | sidecars: - name: php-fpm image: repository: php tag: "8.2-fpm-alpine" pullPolicy: IfNotPresent command: - sh - -c - | exec php-fpm -F ports: - name: fpm containerPort: 9000 protocol: TCP livenessProbe: tcpSocket: port: fpm initialDelaySeconds: 5 periodSeconds: 10 readinessProbe: tcpSocket: port: fpm initialDelaySeconds: 2 periodSeconds: 5 resources: requests: cpu: 50m memory: 64Mi limits: cpu: 200m memory: 256Mi volumeMounts: # <--- ДОБАВЛЕНО - name: php-index mountPath: /var/www/html/index.php subPath: index.php |
| --- | --- |

вот полный вид values

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210211212213214215216217218219220221222223224225226227228229230231232233234235236237238239240241242243244245246247248249250251252253254255256257258259260261262263264265266267268269270271272273274275276277278279280281282283284285286287288289290291292293294295296297298299300301302303304305306307308309310311312313314315316317318319320321322323324325326327328329330331332333334335336337338339340341342343344345346347348349350351352353354355356357358359360361362363364365366367368369370371372373374375376377378379380381382383384385386387388389390391392393394395396397398399400401402403404405406407408409410411412413414415416417418419420421422423424425426427428429430431432433434435436437438439440441442443444445446447448449450451452453454455456457458 | # Default values for common-chart.# This is a YAML-formatted file.# Declare variables to be passed into your templates.# This will set the replicaset count more information can be found here: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/replicaCount: 1# This sets the container image more information can be found here: https://kubernetes.io/docs/concepts/containers/images/image: repository: nginx # This sets the pull policy for images. pullPolicy: IfNotPresent # Overrides the image tag whose default is the chart appVersion. tag: ""# This is for the secretes for pulling an image from a private repository more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/imagePullSecrets: []# This is to override the chart name.nameOverride: ""fullnameOverride: "app"#This section builds out the service account more information can be found here: https://kubernetes.io/docs/concepts/security/service-accounts/serviceAccount: # Specifies whether a service account should be created create: true # Automatically mount a ServiceAccount's API credentials? automount: true # Annotations to add to the service account annotations: {} # The name of the service account to use. # If not set and create is true, a name is generated using the fullname template name: ""# This is for setting Kubernetes Annotations to a Pod.# For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/ podAnnotations: {}# This is for setting Kubernetes Labels to a Pod.# For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/podLabels: {}podSecurityContext: {} # fsGroup: 2000securityContext: {} # capabilities: # drop: # - ALL # readOnlyRootFilesystem: true # runAsNonRoot: true # runAsUser: 1000# This is for setting up a service more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/service: # This sets the service type more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types type: ClusterIP # This sets the ports more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#field-spec-ports port: 80# This block is for setting up the ingress for more information can be found here: https://kubernetes.io/docs/concepts/services-networking/ingress/ingress: enabled: true className: "nginx" annotations: cert-manager.io/cluster-issuer: ca-issuer hosts: - host: test-app.test.local paths: - path: / pathType: ImplementationSpecific tls: - secretName: test-app-tls hosts: - test-app.test.localresources: limits: cpu: 100m memory: 128Mi requests: cpu: 100m memory: 128Mi# This is to setup the liveness and readiness probes more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/livenessProbe: httpGet: path: / port: httpreadinessProbe: httpGet: path: / port: http#This section is for setting up autoscaling more information can be found here: https://kubernetes.io/docs/concepts/workloads/autoscaling/autoscaling: enabled: false minReplicas: 1 maxReplicas: 5 targetCPUUtilizationPercentage: 80 # targetMemoryUtilizationPercentage: 80 behavior: scaleUp: stabilizationWindowSeconds: 0 selectPolicy: Max # допустимо: Max | Min | Disabled policies: - type: Percent # не более +100% за 60 сек value: 100 periodSeconds: 60 - type: Pods # и не более +4 пода за 60 сек value: 4 periodSeconds: 60 scaleDown: stabilizationWindowSeconds: 30 selectPolicy: Max # для жёсткого ограничения выбирай Min policies: - type: Percent # не более -10% за 60 сек value: 10 periodSeconds: 60# # Additional volumes on the output Deployment definition.# volumes: []# # - name: foo# # secret:# # secretName: mysecret# # optional: false# # Additional volumeMounts on the output Deployment definition.# volumeMounts: []# # - name: foo# # mountPath: "/etc/foo"# # readOnly: truenodeSelector: {}tolerations: []affinity: {}vault_secret: enabled: true name: app-secret # имя k8s Secret, который получит VSO secretFullPath: app/service/app1 # путь в Vault: <ns>/service/<app> type: kv-v1 # у меня сейчас kv-v1 role: ns-scoped-read # универсальная роль в vault для всех ns # audiences: # - "https://kubernetes.default.svc"topologySpread: enabled: true constraints: - topologyKey: "kubernetes.io/hostname" # по серверам maxSkew: 2 whenUnsatisfiable: "ScheduleAnyway" # или "DoNotSchedule"updateStrategy: enabled: true type: RollingUpdate # или Recreate rollingUpdate: maxSurge: 25% # число или проценты; напр. 1 или "25%" maxUnavailable: 0 # число или проценты; 0 = без даунтайма по репликам minReadySeconds: 10 # сколько pod должен быть Ready перед следующим шагом revisionHistoryLimit: 10 # сколько старых ReplicaSet хранить progressDeadlineSeconds: 600 # дедлайн на прогресс выкатаpdb: enabled: true # Выбери ОДИН из параметров ниже: minAvailable: 1 #maxUnavailable: 1 #Замечание: при replicaCount: 1 и minAvailable: 1 drain ноды будет блокироваться — это ожидаемое поведение. Если это не нужно, ставьте maxUnavailable: 1 или увеличьте число репликkeda: enabled: true # Кого скейлим (по умолчанию — имя релиза) scaleTargetRef: apiVersion: apps/v1 kind: Deployment name: "" # пусто => {{ include "common-chart.fullname" . }} # Периоды опроса/остывания pollingInterval: 15 cooldownPeriod: 30 # Границы minReplicaCount: 1 maxReplicaCount: 5 # Поведение при сбое источника метрик (опционально) fallback: # failureThreshold: 3 # replicas: 2 # Алгоритм объединения желаемых реплик при нескольких триггерах (если НЕТ формулы) — max|min|average scalingStrategy: multipleScalersCalculation: max # Проброс behavior в создаваемый HPA (опционально) advanced: horizontalPodAutoscalerConfig: behavior: scaleUp: stabilizationWindowSeconds: 0 selectPolicy: Max policies: - type: Percent value: 100 periodSeconds: 60 - type: Pods value: 4 periodSeconds: 60 scaleDown: stabilizationWindowSeconds: 20 selectPolicy: Max policies: - type: Percent value: 10 periodSeconds: 60 # === AND-логика через формулу === чтобы вернуть логику ИЛИ нужно закомментировать блок scalingModifiers # req — это метрика запросов, cpu — Prometheus-метрика CPU-процентов. scalingModifiers: formula: "req >= 100 && cpu >= 60 ? max(req/100, cpu/60) : 0" target: "1" # масштабируемся, когда формула >= 1 activationTarget: "0" # при 0 не активируемся metricType: "AverageValue" # === Триггеры (два Prometheus-триггера: ingress-requests и CPU-проценты) === triggers: # 1) Ingress requests за 2 минуты - type: prometheus name: req metadata: serverAddress: "http://vmsingle-vmks-victoria-metrics-k8s-stack.monitoring.svc.cluster.local:8428" metricName: "ingress_requests" query: |- sum( increase( nginx_ingress_controller_requests{ingress="app"}[2m] ) ) threshold: "100" unsafeSsl: "true" # 2) CPU-проценты как Prometheus-метрика (средняя загрузка по подам деплоймента "app" в ns "app") # Требует kube-state-metrics. Формула: 100 * usage_cores / requested_cores. - type: prometheus name: cpu metadata: serverAddress: "http://vmsingle-vmks-victoria-metrics-k8s-stack.monitoring.svc.cluster.local:8428" metricName: "cpu_utilization_pct" query: |- 100 * sum by (namespace) ( rate(container_cpu_usage_seconds_total{ namespace="app", pod=~"app-.*", container!="POD", image!="" }[1m]) ) / sum by (namespace) ( kube_pod_container_resource_requests{ namespace="app", pod=~"app-.*", resource="cpu" } ) threshold: "60" unsafeSsl: "true"sidecars: - name: curl-pinger image: repository: curlimages/curl tag: "latest" # или зафиксируй конкретную версию pullPolicy: IfNotPresent command: - sh - -c - | while true; do code=$(curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:80/ || true) echo "$(date -u +%FT%TZ) GET / -> ${code}" sleep 5 done resources: requests: cpu: 5m memory: 16Mi limits: cpu: 50m memory: 64Mi - name: wget-pinger image: repository: busybox tag: "1.36" command: - sh - -c - | while true; do wget -q -S -O /dev/null http://127.0.0.1:80/ 2>&1 | awk 'NR==1{print strftime("%Y-%m-%dT%H:%M:%SZ"), $0}' sleep 5 done - name: php-fpm image: repository: php tag: "8.2-fpm-alpine" pullPolicy: IfNotPresent command: - sh - -c - | exec php-fpm -F ports: - name: fpm containerPort: 9000 protocol: TCP livenessProbe: tcpSocket: port: fpm initialDelaySeconds: 5 periodSeconds: 10 readinessProbe: tcpSocket: port: fpm initialDelaySeconds: 2 periodSeconds: 5 resources: requests: cpu: 50m memory: 64Mi limits: cpu: 200m memory: 256Mi volumeMounts: # <--- ДОБАВЛЕНО - name: php-index mountPath: /var/www/html/index.php subPath: index.php######### инит контейнеры ########## Общий том (если нужно готовить файлы/права в init)volumes: - name: app-code emptyDir: {} # NGINX конфиг: - name: nginx-conf configMap: name: nginx-conf items: - key: default.conf path: default.conf # PHP код (index.php): - name: php-index configMap: name: php-index items: - key: index.php path: index.php# Смонтировать общий том в основной контейнер (опционально)volumeMounts: - name: app-code mountPath: /var/www/html - name: nginx-conf mountPath: /etc/nginx/conf.d/default.conf subPath: default.conf - name: php-index mountPath: /var/www/html/index.php subPath: index.php# Набор init-контейнеровinitContainers: # 1) Подготовка файлов - name: init-files image: repository: busybox tag: "1.36" pullPolicy: IfNotPresent command: - sh - -c - | mkdir -p /var/www/html echo "init generated $(date -u +%FT%TZ)" > /var/www/html/README.txt volumeMounts: - name: app-code mountPath: /var/www/html resources: requests: cpu: 5m memory: 16Mi limits: cpu: 50m memory: 64Mi # 2) Подготовка прав - name: init-permissions image: repository: busybox tag: "1.36" pullPolicy: IfNotPresent command: - sh - -c - | chown -R 101:101 /var/www/html || true ls -lah /var/www/ echo "" cat /var/www/html/README.txt volumeMounts: - name: app-code mountPath: /var/www/html resources: requests: cpu: 5m memory: 16Mi limits: cpu: 50m memory: 64Miconfigmaps: enabled: true items: - name: app-config-test # имя ConfigMap file: configmap-files/test.yaml # путь к файлу внутри чарта key: app.conf # (опционально) ключ в .data; если не задан — basename(file) labels: # (опционально) tier: backend annotations: # (опционально) reloader.stakater.com/match: "true" - name: logging file: configmap-files/test2.yaml # бинарный пример -> попадёт в binaryData (base64) - name: ca-bundle file: configmap-files/test3.crt binary: true # (опционально) по умолчанию false # для nginx и php-fpm чтобы запрос прилетал на nginx и проксировался на php-fpm - name: nginx-conf file: configmap-files/nginx/default.conf key: default.conf - name: php-index file: configmap-files/php/index.php key: index.php |
| --- | --- |

проверяем:

root@kub-master1:~/helm-charts/10_containers_3_configmap#**kubectl exec -ti -n app app-5dcd6669f6-5fkzp -- bash**
Defaulted container "common-chart" out of: common-chart, curl-pinger, wget-pinger, php-fpm, init-files (init), init-permissions (init)

|  | root@app-5dcd6669f6-5fkzp:/# ls -lah /var/www/html/total 16Kdrwxrwxrwx 2 nginx nginx 4.0K Sep 21 11:49 .drwxr-xr-x 3 root root 4.0K Sep 21 11:49 ..-rw-r--r-- 1 nginx nginx 36 Sep 21 11:49 README.txt-rw-r--r-- 1 root root 16 Sep 21 11:49 index.php |
| --- | --- |

|  | root@app-5dcd6669f6-5fkzp:/# cat /var/www/html/index.php <?phpphpinfo(); |
| --- | --- |

| 12345678910111213141516171819202122232425 | root@app-5dcd6669f6-5fkzp:/# cat /etc/nginx/conf.d/default.conf server { listen 80; server_name _; root /var/www/html; index index.php index.html index.htm; access_log /var/log/nginx/access.log; error_log /var/log/nginx/error.log; location / { try_files $uri $uri/ /index.php?$args; } location ~ \.php$ { include fastcgi_params; fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name; fastcgi_param PATH_INFO $fastcgi_path_info; fastcgi_index index.php; fastcgi_buffers 16 16k; fastcgi_buffer_size 32k; fastcgi_pass 127.0.0.1:9000; }} |
| --- | --- |

зайдём на сам сайт:

![](/news/sidmidru/article-db03824b64600e5b/image-342.png)

как видим всё ок.

вот логи:

root@kub-master1:~/helm-charts/10_containers_3_configmap#**kubectl -n app logs deploy/app -c common-chart -f**

|  | 192.168.1.116 - - [21/Sep/2025:12:09:43 +0000] "GET / HTTP/1.1" 200 65456 "-" "kube-probe/1.32"127.0.0.1 - - [21/Sep/2025:12:09:44 +0000] "GET / HTTP/1.1" 200 85744 "-" "curl/8.16.0"192.168.1.116 - - [21/Sep/2025:12:09:46 +0000] "GET / HTTP/1.1" 200 65456 "-" "kube-probe/1.32"127.0.0.1 - - [21/Sep/2025:12:09:46 +0000] "GET / HTTP/1.1" 200 85748 "-" "Wget"127.0.0.1 - - [21/Sep/2025:12:09:49 +0000] "GET / HTTP/1.1" 200 85744 "-" "curl/8.16.0"10.233.67.223 - - [21/Sep/2025:12:09:50 +0000] "GET / HTTP/1.1" 200 92550 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36"10.233.67.223 - - [21/Sep/2025:12:09:51 +0000] "GET /favicon.ico HTTP/1.1" 200 92134 "https://test-app.test.local/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36"127.0.0.1 - - [21/Sep/2025:12:09:51 +0000] "GET / HTTP/1.1" 200 85748 "-" "Wget"10.233.67.223 - - [21/Sep/2025:12:09:51 +0000] "GET / HTTP/1.1" 200 92552 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36"10.233.67.223 - - [21/Sep/2025:12:09:52 +0000] "GET / HTTP/1.1" 200 92552 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36"10.233.67.223 - - [21/Sep/2025:12:09:52 +0000] "GET / HTTP/1.1" 499 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36"10.233.67.223 - - [21/Sep/2025:12:09:52 +0000] "GET / HTTP/1.1" 200 92560 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36"10.233.67.223 - - [21/Sep/2025:12:09:52 +0000] "GET / HTTP/1.1" 200 92552 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36"10.233.67.223 - - [21/Sep/2025:12:09:53 +0000] "GET / HTTP/1.1" 499 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36"10.233.67.223 - - [21/Sep/2025:12:09:53 +0000] "GET / HTTP/1.1" 200 92552 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36" |
| --- | --- |

как видим nginx показывает где дёргает сайдкар контейнеры (curl/wget) а где проверки kube-probe остальное это я с браузера дёргал.

а вот логи с php-fpm

root@kub-master1:~/helm-charts/10_containers_3_configmap#**kubectl logs -f -n app app-5dcd6669f6-5fkzp -c php-fpm**

| 123456789101112131415161718192021 | 127.0.0.1 - 21/Sep/2025:12:13:22 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:23 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:26 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:27 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:27 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:32 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:32 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:33 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:36 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:37 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:37 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:42 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:42 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:43 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:46 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:47 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:47 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:52 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:52 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:53 +0000 "GET /index.php" 200127.0.0.1 - 21/Sep/2025:12:13:56 +0000 "GET /index.php" 200 |
| --- | --- |

тут мы видим только запросы с localhost - т.е. в рамках одного pod

### []volume/ephemeral volume/emptyDir

**Volume**— это способ дать контейнеру файловую систему, живущую дольше, чем сам процесс контейнера. Объявляется в PodSpec (`spec.volumes`) и монтируется в контейнеры (`volumeMounts`). Бывают**персистентные**(черезPV/PVC, живут независимо от Pod) и**эфемерные**(живут ровно столько, сколько Pod).

##### Эфемерные тома (ephemeral volumes)

Общее свойство:**жизненный цикл = жизнь Pod’а**. Данные теряются при удалении/эвикте Pod’а. К эфемерным относят:

- 

**emptyDir**— самый простой временный том.

- 

**CSIinline ephemeral**— том от CSI-драйвера, описывается прямо в Pod’e.

- 

**Generic Ephemeral Volume**— динамически создаётся по шаблонуPVCна время Pod’а (даёт плюшки StorageClass: квоты, снапшоты, расширение, политики).

##### emptyDir

- 

Создаётся пустым при старте Pod’а.

- 

Хранится**на узле**: по умолчанию на диске; можно`medium: Memory`для tmpfs (быстро, но считает вRAM).

- 

Общий для всех контейнеров Pod’а, пока Pod жив.

- 

Стирается при удалении/перезапуске Pod’а или его эвикте на другой узел.

##### Когда использовать emptyDir

- 

Временный рабочий каталог/скретч-данные (build/tmp).

- 

Общий обмен файлами между контейнерами одного Pod’а (nginx ↔ php-fpm, sidecar-логгер и т.п.).

- 

Кэш, который**можно потерять**(artifact cache, download cache).

- 

Быстрый tmpfs (`medium: Memory`) для очень горячих временных файлов/IPC.

##### КогдаНЕиспользовать emptyDir

- 

Когда данные должны пережить рестарт Pod’а/пересоздание на другом узле → берите**PersistentVolume (PVC)**.

- 

Когда нужны квоты/снапшоты/resize/политики хранения →**Generic Ephemeral Volume**(через PVC-template) или обычныйPVC.

##### CSIinline&Generic Ephemeral — когда уместны

- 

**CSIinline ephemeral**: нужен спец-том отCSI(например, токены HW-модулей, небольшие девайс-маунты) без отдельногоPVC.

- 

**Generic Ephemeral Volume**: нужен “как emptyDir, но с возможностями StorageClass” — лимиты, снапшоты, расширение, разные классы дисков. Живёт как Pod, но управляется какPVCпод капотом.

##### Быстрые рекомендации

- 

**Нужен временный общий каталог в Pod, можно потерять данные?**→`emptyDir`.

- 

**Нужен быстрый временный RAM-том?**→`emptyDir.medium: Memory`.

- 

**Нужны фичи хранилища, но том должен умирать вместе с Pod?**→ Generic Ephemeral (PVC-template).

- 

**Данные должны переживать Pod/перенос узла?**→ обычный**PVC(PersistentVolume)**.

в предыдущих пример я использовал только`emptyDir`

чтоб можно было подключать pvc который доступен и после перезагрузки используем следующий template

/etc/ansible/kubespray-official/helm-charts/10_containers_4_volume/common-chart/templates/pvc.yaml

| 1234567891011121314151617 | {{- if .Values.pvc.enabled }}apiVersion: v1kind: PersistentVolumeClaimmetadata: name: {{ include "common-chart.fullname" . }}-app-data labels: {{- include "common-chart.labels" . | nindent 4 }}spec: accessModes: - {{ .Values.pvc.accessMode | default "ReadWriteMany" }} storageClassName: {{ .Values.pvc.storageClassName | default "nfs-client" | quote }} resources: requests: storage: {{ .Values.pvc.size | default "1Gi" }}{{- end }} |
| --- | --- |

мы уже подключали сетевое хранилище nfs-client

values будет таким:

/etc/ansible/kubespray-official/helm-charts/10_containers_4_volume/common-chart/values.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445 | # === volumes: подключаем созданный PVC ===volumes: - name: app-data persistentVolumeClaim: claimName: app-app-data # (опционально: конфиг nginx и index.php из ConfigMap, если используешь) - name: nginx-conf configMap: name: nginx-conf items: - key: default.conf path: default.conf - name: php-index configMap: name: php-index items: - key: index.php path: index.php# === монтирование в основной контейнер (nginx) ===volumeMounts: - name: app-data mountPath: /var/www/html # (опционально — если конфиг из ConfigMap) - name: nginx-conf mountPath: /etc/nginx/conf.d/default.conf subPath: default.conf # (опционально — если index.php из ConfigMap; иначе файл ляжет на NFS как артефакт деплоя) - name: php-index mountPath: /var/www/html/index.php subPath: index.phppvc: enabled: true storageClassName: nfs-client accessMode: ReadWriteMany size: 1Gi# Рекомендуется для NFS, чтобы контейнеры могли писать:podSecurityContext: fsGroup: 101 # подстраховка для nginx (часто uid/gid 101) fsGroupChangePolicy: "OnRootMismatch" |
| --- | --- |

вот весь values:

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210211212213214215216217218219220221222223224225226227228229230231232233234235236237238239240241242243244245246247248249250251252253254255256257258259260261262263264265266267268269270271272273274275276277278279280281282283284285286287288289290291292293294295296297298299300301302303304305306307308309310311312313314315316317318319320321322323324325326327328329330331332333334335336337338339340341342343344345346347348349350351352353354355356357358359360361362363364365366367368369370371372373374375376377378379380381382383384385386387388389390391392393394395396397398399400401402403404405406407408409410411412413414415416417418419420421422423424425426427428429430431432433434435436437438439440441442443444445446447448449450451452453454455456457458459460461462463464465466467468469470471472473474475476 | # Default values for common-chart.# This is a YAML-formatted file.# Declare variables to be passed into your templates.# This will set the replicaset count more information can be found here: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/replicaCount: 1# This sets the container image more information can be found here: https://kubernetes.io/docs/concepts/containers/images/image: repository: nginx # This sets the pull policy for images. pullPolicy: IfNotPresent # Overrides the image tag whose default is the chart appVersion. tag: ""# This is for the secretes for pulling an image from a private repository more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/imagePullSecrets: []# This is to override the chart name.nameOverride: ""fullnameOverride: "app"#This section builds out the service account more information can be found here: https://kubernetes.io/docs/concepts/security/service-accounts/serviceAccount: # Specifies whether a service account should be created create: true # Automatically mount a ServiceAccount's API credentials? automount: true # Annotations to add to the service account annotations: {} # The name of the service account to use. # If not set and create is true, a name is generated using the fullname template name: ""# This is for setting Kubernetes Annotations to a Pod.# For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/ podAnnotations: {}# This is for setting Kubernetes Labels to a Pod.# For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/podLabels: {}# podSecurityContext: {}# # fsGroup: 2000securityContext: {} # capabilities: # drop: # - ALL # readOnlyRootFilesystem: true # runAsNonRoot: true # runAsUser: 1000# This is for setting up a service more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/service: # This sets the service type more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types type: ClusterIP # This sets the ports more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#field-spec-ports port: 80# This block is for setting up the ingress for more information can be found here: https://kubernetes.io/docs/concepts/services-networking/ingress/ingress: enabled: true className: "nginx" annotations: cert-manager.io/cluster-issuer: ca-issuer hosts: - host: test-app.test.local paths: - path: / pathType: ImplementationSpecific tls: - secretName: test-app-tls hosts: - test-app.test.localresources: limits: cpu: 100m memory: 128Mi requests: cpu: 100m memory: 128Mi# This is to setup the liveness and readiness probes more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/livenessProbe: httpGet: path: / port: httpreadinessProbe: httpGet: path: / port: http#This section is for setting up autoscaling more information can be found here: https://kubernetes.io/docs/concepts/workloads/autoscaling/autoscaling: enabled: false minReplicas: 1 maxReplicas: 5 targetCPUUtilizationPercentage: 80 # targetMemoryUtilizationPercentage: 80 behavior: scaleUp: stabilizationWindowSeconds: 0 selectPolicy: Max # допустимо: Max | Min | Disabled policies: - type: Percent # не более +100% за 60 сек value: 100 periodSeconds: 60 - type: Pods # и не более +4 пода за 60 сек value: 4 periodSeconds: 60 scaleDown: stabilizationWindowSeconds: 30 selectPolicy: Max # для жёсткого ограничения выбирай Min policies: - type: Percent # не более -10% за 60 сек value: 10 periodSeconds: 60# # Additional volumes on the output Deployment definition.# volumes: []# # - name: foo# # secret:# # secretName: mysecret# # optional: false# # Additional volumeMounts on the output Deployment definition.# volumeMounts: []# # - name: foo# # mountPath: "/etc/foo"# # readOnly: truenodeSelector: {}tolerations: []affinity: {}vault_secret: enabled: true name: app-secret # имя k8s Secret, который получит VSO secretFullPath: app/service/app1 # путь в Vault: <ns>/service/<app> type: kv-v1 # у меня сейчас kv-v1 role: ns-scoped-read # универсальная роль в vault для всех ns # audiences: # - "https://kubernetes.default.svc"topologySpread: enabled: true constraints: - topologyKey: "kubernetes.io/hostname" # по серверам maxSkew: 2 whenUnsatisfiable: "ScheduleAnyway" # или "DoNotSchedule"updateStrategy: enabled: true type: RollingUpdate # или Recreate rollingUpdate: maxSurge: 25% # число или проценты; напр. 1 или "25%" maxUnavailable: 0 # число или проценты; 0 = без даунтайма по репликам minReadySeconds: 10 # сколько pod должен быть Ready перед следующим шагом revisionHistoryLimit: 10 # сколько старых ReplicaSet хранить progressDeadlineSeconds: 600 # дедлайн на прогресс выкатаpdb: enabled: true # Выбери ОДИН из параметров ниже: minAvailable: 1 #maxUnavailable: 1 #Замечание: при replicaCount: 1 и minAvailable: 1 drain ноды будет блокироваться — это ожидаемое поведение. Если это не нужно, ставьте maxUnavailable: 1 или увеличьте число репликkeda: enabled: true # Кого скейлим (по умолчанию — имя релиза) scaleTargetRef: apiVersion: apps/v1 kind: Deployment name: "" # пусто => {{ include "common-chart.fullname" . }} # Периоды опроса/остывания pollingInterval: 15 cooldownPeriod: 30 # Границы minReplicaCount: 1 maxReplicaCount: 5 # Поведение при сбое источника метрик (опционально) fallback: # failureThreshold: 3 # replicas: 2 # Алгоритм объединения желаемых реплик при нескольких триггерах (если НЕТ формулы) — max|min|average scalingStrategy: multipleScalersCalculation: max # Проброс behavior в создаваемый HPA (опционально) advanced: horizontalPodAutoscalerConfig: behavior: scaleUp: stabilizationWindowSeconds: 0 selectPolicy: Max policies: - type: Percent value: 100 periodSeconds: 60 - type: Pods value: 4 periodSeconds: 60 scaleDown: stabilizationWindowSeconds: 20 selectPolicy: Max policies: - type: Percent value: 10 periodSeconds: 60 # === AND-логика через формулу === чтобы вернуть логику ИЛИ нужно закомментировать блок scalingModifiers # req — это метрика запросов, cpu — Prometheus-метрика CPU-процентов. scalingModifiers: formula: "req >= 100 && cpu >= 60 ? max(req/100, cpu/60) : 0" target: "1" # масштабируемся, когда формула >= 1 activationTarget: "0" # при 0 не активируемся metricType: "AverageValue" # === Триггеры (два Prometheus-триггера: ingress-requests и CPU-проценты) === triggers: # 1) Ingress requests за 2 минуты - type: prometheus name: req metadata: serverAddress: "http://vmsingle-vmks-victoria-metrics-k8s-stack.monitoring.svc.cluster.local:8428" metricName: "ingress_requests" query: |- sum( increase( nginx_ingress_controller_requests{ingress="app"}[2m] ) ) threshold: "100" unsafeSsl: "true" # 2) CPU-проценты как Prometheus-метрика (средняя загрузка по подам деплоймента "app" в ns "app") # Требует kube-state-metrics. Формула: 100 * usage_cores / requested_cores. - type: prometheus name: cpu metadata: serverAddress: "http://vmsingle-vmks-victoria-metrics-k8s-stack.monitoring.svc.cluster.local:8428" metricName: "cpu_utilization_pct" query: |- 100 * sum by (namespace) ( rate(container_cpu_usage_seconds_total{ namespace="app", pod=~"app-.*", container!="POD", image!="" }[1m]) ) / sum by (namespace) ( kube_pod_container_resource_requests{ namespace="app", pod=~"app-.*", resource="cpu" } ) threshold: "60" unsafeSsl: "true"sidecars: - name: curl-pinger image: repository: curlimages/curl tag: "latest" # или зафиксируй конкретную версию pullPolicy: IfNotPresent command: - sh - -c - | while true; do code=$(curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:80/ || true) echo "$(date -u +%FT%TZ) GET / -> ${code}" sleep 5 done resources: requests: cpu: 5m memory: 16Mi limits: cpu: 50m memory: 64Mi - name: wget-pinger image: repository: busybox tag: "1.36" command: - sh - -c - | while true; do wget -q -S -O /dev/null http://127.0.0.1:80/ 2>&1 | awk 'NR==1{print strftime("%Y-%m-%dT%H:%M:%SZ"), $0}' sleep 5 done - name: php-fpm image: repository: php tag: "8.2-fpm-alpine" pullPolicy: IfNotPresent command: - sh - -c - | exec php-fpm -F ports: - name: fpm containerPort: 9000 protocol: TCP livenessProbe: tcpSocket: port: fpm initialDelaySeconds: 5 periodSeconds: 10 readinessProbe: tcpSocket: port: fpm initialDelaySeconds: 2 periodSeconds: 5 resources: requests: cpu: 50m memory: 64Mi limits: cpu: 200m memory: 256Mi volumeMounts: - name: app-data mountPath: /var/www/html # (опционально — если index.php из ConfigMap) - name: php-index mountPath: /var/www/html/index.php subPath: index.php######### инит контейнеры ########## === volumes: подключаем созданный PVC ===volumes: - name: app-data persistentVolumeClaim: claimName: app-app-data # (опционально: конфиг nginx и index.php из ConfigMap, если используешь) - name: nginx-conf configMap: name: nginx-conf items: - key: default.conf path: default.conf - name: php-index configMap: name: php-index items: - key: index.php path: index.php# === монтирование в основной контейнер (nginx) ===volumeMounts: - name: app-data mountPath: /var/www/html # (опционально — если конфиг из ConfigMap) - name: nginx-conf mountPath: /etc/nginx/conf.d/default.conf subPath: default.conf # (опционально — если index.php из ConfigMap; иначе файл ляжет на NFS как артефакт деплоя) - name: php-index mountPath: /var/www/html/index.php subPath: index.php# Набор init-контейнеровinitContainers: # 1) Подготовка файлов - name: init-files image: repository: busybox tag: "1.36" pullPolicy: IfNotPresent command: - sh - -c - | mkdir -p /var/www/html echo "init generated $(date -u +%FT%TZ)" > /var/www/html/README.txt volumeMounts: - name: app-data mountPath: /var/www/html resources: requests: cpu: 5m memory: 16Mi limits: cpu: 50m memory: 64Mi # 2) Подготовка прав - name: init-permissions image: repository: busybox tag: "1.36" pullPolicy: IfNotPresent command: - sh - -c - | chown -R 101:101 /var/www/html || true ls -lah /var/www/ echo "" cat /var/www/html/README.txt volumeMounts: - name: app-data mountPath: /var/www/html resources: requests: cpu: 5m memory: 16Mi limits: cpu: 50m memory: 64Miconfigmaps: enabled: true items: - name: app-config-test # имя ConfigMap file: configmap-files/test.yaml # путь к файлу внутри чарта key: app.conf # (опционально) ключ в .data; если не задан — basename(file) labels: # (опционально) tier: backend annotations: # (опционально) reloader.stakater.com/match: "true" - name: logging file: configmap-files/test2.yaml # бинарный пример -> попадёт в binaryData (base64) - name: ca-bundle file: configmap-files/test3.crt binary: true # (опционально) по умолчанию false # для nginx и php-fpm чтобы запрос прилетал на nginx и проксировался на php-fpm - name: nginx-conf file: configmap-files/nginx/default.conf key: default.conf - name: php-index file: configmap-files/php/index.php key: index.php################### pvcpvc: enabled: true storageClassName: nfs-client accessMode: ReadWriteMany size: 1Gi# Рекомендуется для NFS, чтобы контейнеры могли писать:podSecurityContext: fsGroup: 101 # подстраховка для nginx (часто uid/gid 101) fsGroupChangePolicy: "OnRootMismatch" |
| --- | --- |

ставим

helm upgrade --install app -n app ./common-chart -f ./common-chart/values.yaml

проверяем:

|  | root@kub-master1:~/helm-charts/10_containers_4_volume# kubectl get pvc -n appNAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS VOLUMEATTRIBUTESCLASS AGEapp-app-data Bound pvc-c7d5623c-ffea-4973-a800-a0a2f3ac73f8 1Gi RWX nfs-client <unset> 17m |
| --- | --- |

диск создался.

проверяем что он подкинулся:

| 123456789101112131415161718192021222324 | root@kub-master1:~/helm-charts/10_containers_4_volume# kubectl exec -ti -n app app-745d64f8-t7b4x -- bashDefaulted container "common-chart" out of: common-chart, curl-pinger, wget-pinger, php-fpm, init-files (init), init-permissions (init)root@app-745d64f8-t7b4x:/# root@app-745d64f8-t7b4x:/# ls -lah /var/www/html/total 16Kdrwxrwxrwx 2 nginx nginx 4.0K Sep 21 12:49 .drwxr-xr-x 3 root root 4.0K Sep 21 12:49 ..-rw-r--r-- 1 nginx nginx 36 Sep 21 12:49 README.txt-rw-r--r-- 1 root nginx 16 Sep 21 12:49 index.phproot@app-745d64f8-t7b4x:/# df -hFilesystem Size Used Avail Use% Mounted onoverlay 67G 36G 28G 57% /tmpfs 64M 0 64M 0% /dev/dev/mapper/debian--vg-root 67G 36G 28G 57% /etc/hostsshm 64M 0 64M 0% /dev/shm192.168.1.108:/nfs/app-app-app-data-pvc-c7d5623c-ffea-4973-a800-a0a2f3ac73f8 37G 15G 21G 41% /var/www/htmltmpfs 3.8G 12K 3.8G 1% /run/secrets/kubernetes.io/serviceaccounttmpfs 2.3G 0 2.3G 0% /proc/asoundtmpfs 2.3G 0 2.3G 0% /proc/acpitmpfs 2.3G 0 2.3G 0% /sys/firmwaretmpfs 2.3G 0 2.3G 0% /sys/devices/virtual/powercap |
| --- | --- |

создадим файл, удалимPODзайдём в новый и проверим что файл на месте:

| 123456789101112131415161718192021222324252627 | root@app-745d64f8-t7b4x:/# echo test > /var/www/html/test.txtroot@app-745d64f8-t7b4x:/# ls -lah /var/www/html/total 20Kdrwxrwxrwx 2 nginx nginx 4.0K Sep 21 12:55 .drwxr-xr-x 3 root root 4.0K Sep 21 12:49 ..-rw-r--r-- 1 nginx nginx 36 Sep 21 12:49 README.txt-rw-r--r-- 1 root nginx 16 Sep 21 12:49 index.php-rw-r--r-- 1 root root 5 Sep 21 12:55 test.txtroot@app-745d64f8-t7b4x:/# exitcommand terminated with exit code 130root@kub-master1:~/helm-charts/10_containers_4_volume# kubectl delete pod -n app app-745d64f8-t7b4x pod "app-745d64f8-t7b4x" deletedroot@kub-master1:~/helm-charts/10_containers_4_volume# kubectl exec -ti -n app app-745d64f8-7fxmt -- bashDefaulted container "common-chart" out of: common-chart, curl-pinger, wget-pinger, php-fpm, init-files (init), init-permissions (init)root@app-745d64f8-7fxmt:/# ls -lah /var/www/html/total 20Kdrwxrwxrwx 2 nginx nginx 4.0K Sep 21 12:55 .drwxr-xr-x 3 root root 4.0K Sep 21 13:01 ..-rw-r--r-- 1 nginx nginx 36 Sep 21 13:01 README.txt-rw-r--r-- 1 root nginx 16 Sep 21 13:01 index.php-rw-r--r-- 1 nginx nginx 5 Sep 21 12:55 test.txt |
| --- | --- |

как видим всё ок.

### []job и cronjob

##### Job

**Что это:**одноразовая задача. Контроллер следит, чтобы Pod(ы) завершились**успешно**нужное число раз.

**Ключевые поля:**

- 

`spec.template`— Pod-шаблон (контейнеры, env, volumes и т.д.).

- 

`spec.completions`— сколько**успешных завершений**нужно (по умолчанию 1).

- 

`spec.parallelism`— сколько Pod’ов выполнять**параллельно**.

- 

`spec.backoffLimit`— сколько раз перезапускать при неуспехе (по умолчанию 6).

- 

`spec.activeDeadlineSeconds`— общий дедлайн для всей Job.

- 

`spec.ttlSecondsAfterFinished`— авто-удаление Job после завершения.

|  | apiVersion: batch/v1kind: Jobmetadata: name: migrate-dbspec: backoffLimit: 2 ttlSecondsAfterFinished: 600 template: spec: restartPolicy: Never containers: - name: migrate image: alpine command: ["sh", "-c", "echo run migrations && sleep 10 && exit 0"] |
| --- | --- |

##### CronJob

**Что это:**задача по**расписанию**. По cron-графику создаёт Job.

**Ключевые поля:**

- 

`spec.schedule`— cron-строка (например,`"*/5 * * * *"`).

- 

`spec.timeZone`— часовой пояс расписания (1.27+), например`Asia/Bishkek`.

- 

`spec.concurrencyPolicy`— поведение при совпадении запусков:

- 

`Allow`— разрешать параллельно (по умолчанию)

- 

`Forbid`— не запускать новый, если старый не завершён

- 

`Replace`— отменить старый и запустить новый

- 

`spec.startingDeadlineSeconds`— дедлайн на пропущенные запуски.

- 

`spec.successfulJobsHistoryLimit`/`failedJobsHistoryLimit`— сколько историй хранить.

- 

`spec.suspend`— пауза (не создавать новые Job).

- 

`spec.jobTemplate`— шаблон Job (то же, что у Job в`spec`).

| 12345678910111213141516171819202122 | apiVersion: batch/v1kind: CronJobmetadata: name: nightly-backupspec: schedule: "0 2 * * *" # каждый день в 02:00 timeZone: "Asia/Bishkek" concurrencyPolicy: Forbid startingDeadlineSeconds: 600 successfulJobsHistoryLimit: 3 failedJobsHistoryLimit: 1 jobTemplate: spec: backoffLimit: 1 template: spec: restartPolicy: OnFailure containers: - name: backup image: alpine command: ["sh","-c","echo backup && date"] |
| --- | --- |

вот дополнение к нашему темплейту:

/etc/ansible/kubespray-official/helm-charts/11_jobs_cronjob/common-chart/templates/cronjobs.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121 | {{- if and .Values.cronjobs .Values.cronjobs.enabled }}{{- $root := . }}{{- range $i, $cj := .Values.cronjobs.items }} {{- $name := required (printf "cronjobs.items[%d].name is required" $i) $cj.name }} {{- $sched := required (printf "cronjobs.items[%d].schedule is required" $i) $cj.schedule }} {{- $job := required (printf "cronjobs.items[%d].job is required" $i) $cj.job }} {{- $img := required (printf "cronjobs.items[%d].job.image.repository is required" $i) (get $job.image "repository") }}apiVersion: batch/v1kind: CronJobmetadata: name: {{ printf "%s-%s" (include "common-chart.fullname" $root) $name | trunc 63 | trimSuffix "-" }} namespace: {{ $root.Release.Namespace }} labels: {{- include "common-chart.labels" $root | nindent 4 }} cronjob.kubernetes.io/name: {{ $name | quote }} {{- $meta := get $cj "metadata" }} {{- if and $meta (kindIs "map" $meta) (get $meta "annotations") }} annotations: {{- toYaml (get $meta "annotations") | nindent 4 }} {{- end }}spec: schedule: {{ $sched | quote }} {{- with $cj.timeZone }} timeZone: {{ . | quote }} {{- end }} {{- with $cj.suspend }} suspend: {{ . }} {{- end }} {{- with $cj.concurrencyPolicy }} concurrencyPolicy: {{ . }} {{- end }} {{- with $cj.startingDeadlineSeconds }} startingDeadlineSeconds: {{ . }} {{- end }} {{- with $cj.successfulJobsHistoryLimit }} successfulJobsHistoryLimit: {{ . }} {{- end }} {{- with $cj.failedJobsHistoryLimit }} failedJobsHistoryLimit: {{ . }} {{- end }} jobTemplate: spec: {{- with $job.backoffLimit }} backoffLimit: {{ . }} {{- end }} {{- with $job.activeDeadlineSeconds }} activeDeadlineSeconds: {{ . }} {{- end }} {{- with $job.ttlSecondsAfterFinished }} ttlSecondsAfterFinished: {{ . }} {{- end }} template: metadata: labels: {{- include "common-chart.labels" $root | nindent 12 }} cronjob.kubernetes.io/name: {{ $name | quote }} {{- with (get $job "podAnnotations") }} annotations: {{- toYaml . | nindent 12 }} {{- end }} spec: serviceAccountName: {{ include "common-chart.serviceAccountName" $root }} {{- with $root.Values.podSecurityContext }} securityContext: {{- toYaml . | nindent 12 }} {{- end }} {{- with $root.Values.imagePullSecrets }} imagePullSecrets: {{- toYaml . | nindent 12 }} {{- end }} restartPolicy: {{ default "OnFailure" $job.restartPolicy }} containers: - name: {{ $name | trunc 63 | trimSuffix "-" }} image: "{{ $img }}{{- if (get $job.image "tag") }}:{{ get $job.image "tag" }}{{- end }}" {{- with (get $job.image "pullPolicy") }} imagePullPolicy: {{ . }} {{- end }} {{- with $job.command }} command: {{- toYaml . | nindent 16 }} {{- end }} {{- with $job.args }} args: {{- toYaml . | nindent 16 }} {{- end }} {{- with $job.env }} env: {{- toYaml . | nindent 16 }} {{- end }} {{- with $job.envFrom }} envFrom: {{- toYaml . | nindent 16 }} {{- end }} {{- with ($job.resources | default $root.Values.resources) }} resources: {{- toYaml . | nindent 16 }} {{- end }} {{- with $job.volumeMounts }} volumeMounts: {{- toYaml . | nindent 16 }} {{- end }} {{- with $job.volumes }} volumes: {{- toYaml . | nindent 12 }} {{- end }} {{- with ($job.nodeSelector | default $root.Values.nodeSelector) }} nodeSelector: {{- toYaml . | nindent 12 }} {{- end }} {{- with ($job.tolerations | default $root.Values.tolerations) }} tolerations: {{- toYaml . | nindent 12 }} {{- end }} {{- with ($job.affinity | default $root.Values.affinity) }} affinity: {{- toYaml . | nindent 12 }} {{- end }}---{{- end }}{{- end }} |
| --- | --- |

/etc/ansible/kubespray-official/helm-charts/11_jobs_cronjob/common-chart/templates/jobs.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104 | {{- if and .Values.jobs .Values.jobs.enabled }}{{- $root := . }}{{- range $i, $j := .Values.jobs.items }} {{- $name := required (printf "jobs.items[%d].name is required" $i) $j.name }} {{- $img := required (printf "jobs.items[%d].image.repository is required" $i) (get $j.image "repository") }}apiVersion: batch/v1kind: Jobmetadata: name: {{ printf "%s-%s" (include "common-chart.fullname" $root) $name | trunc 63 | trimSuffix "-" }} namespace: {{ $root.Release.Namespace }} labels: {{- include "common-chart.labels" $root | nindent 4 }} job.kubernetes.io/name: {{ $name | quote }} {{- $meta := get $j "metadata" }} {{- if and $meta (kindIs "map" $meta) (get $meta "annotations") }} annotations: {{- toYaml (get $meta "annotations") | nindent 4 }} {{- end }}spec: {{- with $j.parallelism }} parallelism: {{ . }} {{- end }} {{- with $j.completions }} completions: {{ . }} {{- end }} {{- with $j.backoffLimit }} backoffLimit: {{ . }} {{- end }} {{- with $j.activeDeadlineSeconds }} activeDeadlineSeconds: {{ . }} {{- end }} {{- with $j.ttlSecondsAfterFinished }} ttlSecondsAfterFinished: {{ . }} {{- end }} template: metadata: labels: {{- include "common-chart.labels" $root | nindent 8 }} job.kubernetes.io/name: {{ $name | quote }} {{- with (get $j "podAnnotations") }} annotations: {{- toYaml . | nindent 8 }} {{- end }} spec: serviceAccountName: {{ include "common-chart.serviceAccountName" $root }} {{- with $root.Values.imagePullSecrets }} imagePullSecrets: {{- toYaml . | nindent 8 }} {{- end }} {{- with $root.Values.podSecurityContext }} securityContext: {{- toYaml . | nindent 8 }} {{- end }} restartPolicy: {{ default "OnFailure" $j.restartPolicy }} containers: - name: {{ $name | trunc 63 | trimSuffix "-" }} image: "{{ $img }}{{- if (get $j.image "tag") }}:{{ get $j.image "tag" }}{{- end }}" {{- with (get $j.image "pullPolicy") }} imagePullPolicy: {{ . }} {{- end }} {{- with $j.command }} command: {{- toYaml . | nindent 12 }} {{- end }} {{- with $j.args }} args: {{- toYaml . | nindent 12 }} {{- end }} {{- with $j.env }} env: {{- toYaml . | nindent 12 }} {{- end }} {{- with $j.envFrom }} envFrom: {{- toYaml . | nindent 12 }} {{- end }} {{- with ($j.resources | default $root.Values.resources) }} resources: {{- toYaml . | nindent 12 }} {{- end }} {{- with $j.volumeMounts }} volumeMounts: {{- toYaml . | nindent 12 }} {{- end }} {{- with $j.volumes }} volumes: {{- toYaml . | nindent 8 }} {{- end }} {{- with ($j.nodeSelector | default $root.Values.nodeSelector) }} nodeSelector: {{- toYaml . | nindent 8 }} {{- end }} {{- with ($j.tolerations | default $root.Values.tolerations) }} tolerations: {{- toYaml . | nindent 8 }} {{- end }} {{- with ($j.affinity | default $root.Values.affinity) }} affinity: {{- toYaml . | nindent 8 }} {{- end }}---{{- end }}{{- end }} |
| --- | --- |

а вот values в котором у нас 2 job и 2 cronjob

/etc/ansible/kubespray-official/helm-charts/11_jobs_cronjob/common-chart/values.yaml

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384 | jobs: enabled: true items: - name: db-migrate image: repository: alpine tag: "3.20" pullPolicy: IfNotPresent command: ["sh","-c"] args: ["echo 'migrate' && sleep 5 && exit 0"] backoffLimit: 2 ttlSecondsAfterFinished: 600 resources: requests: { cpu: 50m, memory: 64Mi } limits: { cpu: 200m, memory: 128Mi } env: - name: ENV value: "prod" # volumes/volumeMounts по желанию: # volumes: # - name: data # emptyDir: {} # volumeMounts: # - name: data # mountPath: /work - name: db-migrate2 image: repository: alpine tag: "3.19" pullPolicy: IfNotPresent command: ["sh","-c"] args: ["echo 'migrate2' && sleep 40 && exit 0"] backoffLimit: 2 ttlSecondsAfterFinished: 600 resources: requests: { cpu: 52m, memory: 50Mi } limits: { cpu: 200m, memory: 128Mi } env: - name: ENV value: "prod"cronjobs: enabled: true items: - name: nightly-backup schedule: "0 2 * * *" timeZone: "Asia/Bishkek" concurrencyPolicy: Forbid startingDeadlineSeconds: 600 successfulJobsHistoryLimit: 3 failedJobsHistoryLimit: 1 job: image: repository: busybox tag: "1.36" command: ["sh","-c"] args: ["echo backup $(date -u +%FT%TZ) && sleep 10"] backoffLimit: 1 resources: requests: { cpu: 10m, memory: 16Mi } limits: { cpu: 100m, memory: 64Mi } # env/envFrom/volumes/volumeMounts/restartPolicy при необходимости - name: nightly-backup2 schedule: "0 3 * * *" timeZone: "Asia/Bishkek" concurrencyPolicy: Forbid startingDeadlineSeconds: 600 successfulJobsHistoryLimit: 2 failedJobsHistoryLimit: 1 job: image: repository: busybox tag: "1.35" command: ["sh","-c"] args: ["echo backup2 $(date -u +%FT%TZ) && sleep 10"] backoffLimit: 1 resources: requests: { cpu: 22m, memory: 22Mi } limits: { cpu: 100m, memory: 64Mi } # env/envFrom/volumes/volumeMounts/restartPolicy при необходимости |
| --- | --- |

ставим:

helm upgrade --install app -n app ./common-chart -f ./common-chart/values.yaml

проверяем:

|  | root@kub-master1:~/helm-charts/11_jobs_cronjob# kubectl get pod -n appNAME READY STATUS RESTARTS AGEapp-745d64f8-7fxmt 4/4 Running 23 (151m ago) 5d20happ-db-migrate-cmfth 0/1 Completed 0 10sapp-db-migrate2-9l9ns 1/1 Running 0 10sroot@kub-master1:~/helm-charts/11_jobs_cronjob# kubectl logs -f -n app app-db-migrate-cmfth migrateroot@kub-master1:~/helm-charts/11_jobs_cronjob# kubectl logs -f -n app app-db-migrate2-9l9ns migrate2 |
| --- | --- |

|  | root@kub-master1:~/helm-charts/11_jobs_cronjob# kubectl get cronjobs.batch -n appNAME SCHEDULE TIMEZONE SUSPEND ACTIVE LAST SCHEDULE AGEapp-nightly-backup 0 2 * * * Asia/Bishkek False 0 <none> 13mapp-nightly-backup2 0 3 * * * Asia/Bishkek False 0 <none> 11m |
| --- | --- |

как видим всё ок.

### []probe grpc tcp http

##### HTTPprobe

- 

**Как работает:**делает HTTP-запрос (`GET`по умолчанию) на`path`/`port`.

- 

**Успех:**код ответа**200–399**.

- 

**Когда использовать:**у сервиса есть HTTP-эндпоинт здоровья (`/healthz`,`/ready`).

- 

**Пример:**

|  | httpGet: path: /healthz port: 8080 scheme: HTTP # или HTTPS |
| --- | --- |

##### TCPprobe

- 

**Как работает:**пытается открыть TCP-соединение к`port`.

- 

**Успех:**порт слушает и соединение установлено.

- 

**Когда использовать:**нетHTTP/gRPC-эндпоинта, но есть открытый порт (DB, простые TCP-сервисы).

- 

**Пример:**

##### gRPC probe (1.24+)

- 

**Как работает:**выполняет gRPC Health Checking Protocol к`service`/`port`.

- 

**Успех:**ответ`SERVING`.

- 

**Когда использовать:**сервис на gRPC и реализован стандартный health-сервис.

- 

**Пример:**

|  | grpc: port: 9090 service: my.grpc.Health # опционально; по умолчанию "" (default service) |
| --- | --- |

##### Общие параметры (для любой пробы)

- 

`initialDelaySeconds`— задержка перед первым проверочным запросом.

- 

`periodSeconds`— интервал между проверками.

- 

`timeoutSeconds`— таймаут одной проверки.

- 

`failureThreshold`— сколько подряд фейлов = «нездоров».

- 

`successThreshold`— для readiness/startup: сколько подряд успехов = «готов».

##### Как выбирать

- 

ЕстьHTTPendpoint? →**HTTP**(даёт семантику кода ответа и путь).

- 

gRPC-сервис с health-чеком? →**gRPC**(точнее, чемTCP).

- 

Нет эндпоинтов здоровья? →**TCP**(минимальная проверка «порт жив»).

в шаблонах ничего править не нужно.
для основного дейплоймена стоит

|  | livenessProbe: {{- toYaml .Values.livenessProbe | nindent 12 }} readinessProbe: {{- toYaml .Values.readinessProbe | nindent 12 }} |
| --- | --- |

для сайдкаров уже есть и стартап пробы

|  | {{- with $sc.livenessProbe }} livenessProbe: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.readinessProbe }} readinessProbe: {{- toYaml . | nindent 12 }} {{- end }} {{- with $sc.startupProbe }} startupProbe: {{- toYaml . | nindent 12 }} {{- end }} |
| --- | --- |

так что туда можно передавать то что нам требуется.

### []network policy

##### Когда применяют NetworkPolicy

- 

**Изоляция между командами/тенантами.**Несколько приложений в одном кластере → включают`default deny`на namespace и по-правилу открывают только нужные направления: фронт→бэк, бэк→БД.

- 

**Zero-trust по умолчанию.**«Открыто только то, что явно разрешено». Любой новый Pod ничего никуда не видит, пока ему не добавят allow-политику.

- 

**Доступ кБДтолько от приложений.**Блокируют прямой доступ к PostgreSQL/Redis от всего, кроме Pod’ов с меткой`role=backend`.

- 

**Граница внешнего трафика.**Разрешают вход**только**от ingress-контроллера (namespace`ingress-nginx`/`cilium`и т.п.), чтобы миновать случайные прямые обращения к PodIP.

- 

**Контроль исходящего трафика (egress).**
— Разрешить толькоDNSи конкретные внешниеAPI/подсети.
— Запретить обращения к облачному metadata-endpoint (169.254.169.254).
— Разрешить выгрузку бэкапов только вS3/VPN-сеть.

- 

**Мониторинг/обсервабилити.**Дать Prometheus (или VictoriaMetrics Agent) читать только нужные target’ы/порты; запретить прочее сканирование в кластере.

- 

**Секьюринг stateful-кластеров.**Для Kafka/Elasticsearch/ZooKeeper разрешают только необходимые peer-to-peer порты между членами кластера и нужным клиентам.

- 

**Разделение сред.**В одном кластере есть`dev/stage/prod`→ политики запрещают межсредовой трафик по умолчанию.

- 

**Сервис-аккаунты и вебхуки.**Разрешают аписерверу (или контроллерам) стучаться в вебхук/оператор, но блокируют всех остальных.

- 

**Карантин инцидентов.**Быстро закрыть namespace (`deny all egress/ingress`) и постепенно открывать безопасные направления.

- 

**Защита от ошибочных сервисов.**Новое приложение «случайно» делает широковещательные/сканирующие вызовы — политики режут такие исходящие коннекты.

##### Что умеют (и чего нет)

- 

**Матчить по:**

- 

Pod-лейблам (внутри/между namespace’ами),

- 

Namespace-лейблам,

- 

`ipBlock`(CIDR’ы для внешних сетей),

- 

Портам (TCP/UDP/SCTP, можно по имени порта из контейнера).

- 

**Выборочно включать только Ingress, только Egress или оба.**

- 

**Аддитивность.**Несколько политик суммируют разрешения (чем больше allow-правил — тем больше разрешено).

- 

**Открыть порт «для всех».**В Ingress/Egress можно указать порт без секции from/to — это значит «любой источник/назначение», но только на этот порт.

- 

**Ограничения:**

- 

Стандартные NetworkPolicy — этоL3/L4(IP/порт).**L7(URI/метод/хост) — нет**(исключение: расширения конкретныхCNI, например Cilium может делатьL7).

- 

Нельзя «отрицательно» матчить (типа «разрешить всем, кроме X») — делайте позитивные allow-правила.

- 

ICMPне регулируется стандартом (толькоTCP/UDP/SCTP).

- 

Политики действуют на трафик**к Pod’ам**; трафик kubelet/apiserver тоже может попадать под них, учитывайте это при default-deny.

##### Типовые ситуации и решения

- 

**Readiness/liveness-пробы не проходят после default-deny.**
Открой нужный порт (или путь) для «любого источника» либо дляIPнод/ingress-контроллера — иначе kubelet/ingress не достучится.

- 

**DNSрезолвинг сломался.**
При egress-deny — явно разрешите выход на kube-dns (обычноUDP/TCP:53 в`kube-system`).

- 

**Prometheus перестал снимать метрики.**
Дайте ingress-право на порт метрик**только**от Pod’ов Prometheus (или по namespaceSelector).

- 

**Нужно чтобы сервис видел только свой же app.**
Разрешение ingress «только от Pod’ов с тем же`app=<имя>`».

- 

**Доступ из кластера к внешнейБД/API.**
Egress-allow к нужнымCIDR/портам, остальное запрещено.

- 

**Строгие регуляторные требования.**
Default-deny + белые списки по всем путям данных + аудит/ревью политик.

посмотрим примеры:

**1) Политика «default-deny ingress» для всех Pod’ов релиза**

(входящий трафик запрещён, исходящий — без ограничений)

|  | networkPolicy: enabled: true items: - name: default-deny-ingress policyTypes: ["Ingress"] ingress: [] # ничего не разрешаем |
| --- | --- |

Ожидание: любой внешний pod не дойдёт до app.

проверяем:

# из dev →ДОЛЖНОпровалиться (timeout)

|  | root@kub-master1:~# kubectl -n dev run tmp --rm -it --image=curlimages/curl -- \ sh -c 'curl -sS -m 3 -o /dev/null -w "%{http_code}\n" http://app.app.svc.cluster.local || echo FAILED'If you don't see a command prompt, try pressing enter.curl: (28) Connection timed out after 3001 milliseconds000FAILEDSession ended, resume using 'kubectl attach tmp -c tmp -i -t' command when the pod is runningpod "tmp" deleted |
| --- | --- |

# TCP-connect тоже должен падать

|  | root@kub-master1:~# kubectl -n dev run tmp --rm -it --image=curlimages/curl -- \ sh -c 'nc -vz -w 3 app.app.svc.cluster.local 80 || echo "connect failed"'If you don't see a command prompt, try pressing enter.nc: app.app.svc.cluster.local (10.233.25.136:80): Operation timed outconnect failedSession ended, resume using 'kubectl attach tmp -c tmp -i -t' command when the pod is runningpod "tmp" deleted |
| --- | --- |

# из app → тоже 403/timeout (если нет allow внутри ns)

|  | root@kub-master1:~# kubectl -n app run tmp --rm -it --image=curlimages/curl -- \ sh -c 'curl -sS -m 3 http://app.app.svc.cluster.local || echo FAILED'If you don't see a command prompt, try pressing enter.curl: (28) Connection timed out after 3002 millisecondsFAILEDSession ended, resume using 'kubectl attach tmp -c tmp -i -t' command when the pod is runningpod "tmp" deleted |
| --- | --- |

**2) Разрешить вход только через Ingress-контроллер наHTTP80**

чтоб каждый раз не дёргать поды создадим сразу несколько в разных неймспейсах:

root@kub-master2:~# kubectl run -n**ingress-nginx**test-ing --rm -it --image=curlimages/curl -- sh
root@kub-master1:~# kubectl run -n**dev**test-dev --rm -it --image=curlimages/curl -- sh
root@kub-master3:~# kubectl run -n**app**test-app --rm -it --image=curlimages/curl -- sh

|  | networkPolicy: enabled: true items: - name: allow-from-ingress policyTypes: ["Ingress"] ingress: - from: - namespaceSelector: matchLabels: kubernetes.io/metadata.name: ingress-nginx ports: - protocol: TCP port: 80 |
| --- | --- |

Ожидание:

из ingress-nginx → проходит
из других ns → блок

# из ingress-namespace →OK(должен вернуть 200/…)

|  | root@kub-master2:~# kubectl run -n ingress-nginx test-ing --rm -it --image=curlimages/curl -- shIf you don't see a command prompt, try pressing enter.~ $ curl -sS -m 3 -I http://app.app.svc.cluster.localHTTP/1.1 200 OKServer: nginx/1.16.0Date: Sun, 28 Sep 2025 11:17:09 GMTContent-Type: text/html; charset=UTF-8Connection: keep-aliveX-Powered-By: PHP/8.2.29 |
| --- | --- |

из остальных неймспейсов отваливается по таймауту.

|  | ~ $ curl -sS -m 3 -I http://app.app.svc.cluster.localcurl: (28) Connection timed out after 3000 milliseconds |
| --- | --- |

**3) Базовый «default-deny all» + доступ с Ingress + доступ VictoriaMetrics (scrape)**

| 1234567891011121314151617181920212223242526272829303132 | networkPolicy: enabled: true items: # 1) Полный запрет всего (по умолчанию ничего не ходит) - name: default-deny-all policyTypes: ["Ingress","Egress"] ingress: [] egress: [] # 2) Разрешить веб-трафик к приложению только с ingress-nginx (порт 80) - name: allow-from-ingress policyTypes: ["Ingress"] ingress: - from: - namespaceSelector: matchLabels: kubernetes.io/metadata.name: ingress-nginx ports: - { protocol: TCP, port: 80 } # 3) Разрешить scrape со стороны VictoriaMetrics (порт метрик приложения, напр. 9113) # Вариант через namespaceSelector (проще и надёжнее) - name: allow-vm-scrape policyTypes: ["Ingress"] ingress: - from: - namespaceSelector: matchLabels: kubernetes.io/metadata.name: monitoring ports: - { protocol: TCP, port: 9113 } # замени на свой порт метрик / имя "metrics" |
| --- | --- |

проверяем

|  | root@kub-master3:~# kubectl run -n monitoring test-monitor --rm -it --image=curlimages/curl -- sh~ $ curl -sS -m 3 -I http://app.app.svc.cluster.localcurl: (28) Connection timed out after 3002 milliseconds~ $ curl -sS -m 5 http://app.app.svc.cluster.local:9113/metrics | head -n 5 || echo FAILEDcurl: (7) Failed to connect to app.app.svc.cluster.local port 9113 after 4 ms: Could not connect to server |
| --- | --- |

как видим из неймспейса monitoring по 80 порту мы не можем подключиться так как мы не разрешали доступ по этому порту из неймспейса monitoring
но вот по порту 9113 у нас другой ответ так как самого порта у меня нет в этом поде то получаем ошибку Could not connect to server

**4) Разрешить vmagent скрейпить метрики твоего приложения**

| 12345678910111213141516171819202122232425 | networkPolicy: enabled: true items: - name: allow-from-ingress policyTypes: ["Ingress"] ingress: - from: - namespaceSelector: matchLabels: kubernetes.io/metadata.name: ingress-nginx ports: - { protocol: TCP, port: 80 } # разрешаем scrape из namespace monitoring на порт(ы) метрик приложения - name: allow-vmagent-scrape policyTypes: ["Ingress"] ingress: - from: - namespaceSelector: matchLabels: kubernetes.io/metadata.name: monitoring ports: - { protocol: TCP, port: 9113 } # <— замени на порт(ы) твоего экспортера - { protocol: TCP, port: 9153 } # можно добавить ещё порты, если нужно - { protocol: TCP, port: 80 } # можно добавить ещё порты, если нужно |
| --- | --- |

ну и для проверки я докинул несколько портов чтоб проверить из неймспейса monitoring

|  | root@kub-master3:~# kubectl run -n monitoring test-mon --rm -it --image=curlimages/curl -- sh~ $ curl -sS -m 3 -I http://app.app.svc.cluster.localHTTP/1.1 200 OK |
| --- | --- |

как видим всё ок

### []Canary/Blue-green

#### Blue-Green (два параллельных пула, мгновенный свитч)

##### Идея

Держим**две**версии одновременно:*blue*(текущая) и*green*(новая). Трафик идёт только в одну. После проверки — мгновенно переключаем всё на новую версию. Роллбэк — такой же мгновенный.

##### Базовая схема

- 

2 Deployment’а:`app-blue`и`app-green`(разные метки, одинаковые порты/пробы).

- 

1 Service, который указывает**ровно на один цвет**через selector.

| 12345678910111213141516171819202122232425262728293031323334 | # Service (селектор указывает на текущий цвет)apiVersion: v1kind: Servicemetadata: name: appspec: selector: app: app color: blue # <— переключаем на green при релизе ports: - name: http port: 80 targetPort: 80---# Deployment BLUEapiVersion: apps/v1kind: Deploymentmetadata: { name: app-blue }spec: selector: { matchLabels: { app: app, color: blue } } template: metadata: { labels: { app: app, color: blue } } spec: { containers: [{ name: app, image: repo/app:1.0 }] }---# Deployment GREENapiVersion: apps/v1kind: Deploymentmetadata: { name: app-green }spec: selector: { matchLabels: { app: app, color: green } } template: metadata: { labels: { app: app, color: green } } spec: { containers: [{ name: app, image: repo/app:1.1 }] } |
| --- | --- |

##### Процесс

- 

Катим`app-green`(реплики поднялись, пробыOK).

- 

Переключаем`Service.spec.selector.color: green`.

- 

Наблюдаем метрики/логи.

- 

Роллбэк при необходимости — вернуть`color: blue`.

##### Когда удобно

- 

Требуется**мгновенный свитч/роллбэк**.

- 

Нельзя смешивать трафик между версиями.

- 

Позволяет полноценно прогреть кэш, выполнить миграции заранее (осторожно с**несовместимыми**миграциямиБД).

#### Canary (постепенная доля трафика на новую версию)

##### Идея

Запускаем**canary-пул**(малое число подов) и отправляем туда**часть**трафика. По метрикам/ошибкам постепенно увеличиваем долю. Роллбэк — вернуть долю к 0%.

##### Как делить трафик

Service сам**не умеет взвешивать**бэкэнды. Нужен:

- 

**Ingress-контроллер с канареечными аннотациями**(NGINXIngress, Traefik), или

- 

**Service Mesh**(Istio/Linkerd) с weighted routing, или

- 

ВнешнийLB/ API-шлюз.

##### Пример с**NGINXIngress**

- 

2 Deployment’а:`app-stable`(основной) и`app-canary`(несколько подов).

- 

1 обычный Ingress → stable Service.

- 

1 canary Ingress → canary Service + аннотации веса.

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051 | # Stable Service + IngressapiVersion: v1kind: Servicemetadata: { name: app-stable }spec: selector: { app: app, track: stable } ports: [{ port: 80, targetPort: 80 }]---apiVersion: networking.k8s.io/v1kind: Ingressmetadata: name: app annotations: kubernetes.io/ingress.class: nginxspec: rules: - host: app.example.com http: paths: - path: / pathType: Prefix backend: { service: { name: app-stable, port: { number: 80 } } }---# Canary Service + Canary Ingress (вес)apiVersion: v1kind: Servicemetadata: { name: app-canary }spec: selector: { app: app, track: canary } ports: [{ port: 80, targetPort: 80 }]---apiVersion: networking.k8s.io/v1kind: Ingressmetadata: name: app-canary annotations: kubernetes.io/ingress.class: nginx nginx.ingress.kubernetes.io/canary: "true" nginx.ingress.kubernetes.io/canary-weight: "10" # 10% трафика # альтернативы: # nginx.ingress.kubernetes.io/canary-by-header: X-Canary # nginx.ingress.kubernetes.io/canary-by-cookie: canary=alwaysspec: rules: - host: app.example.com http: paths: - path: / pathType: Prefix backend: { service: { name: app-canary, port: { number: 80 } } } |
| --- | --- |

##### Процесс

- 

Поднимаем`app-stable`(v1) и`app-canary`(v2, 1–2 пода).

- 

Включаем canary Ingress с малым весом (1–5%).

- 

Смотрим ошибки/латентность/бизнес-метрики → постепенно повышаем вес (10 → 25 → 50 → 100).

- 

Когда всё ок — обновляем stable на v2, выключаем canary.

- 

Роллбэк — снизить вес до 0 или отключить canary.

##### Когда удобно

- 

Нужно**поэтапно**валидировать новую версию на реальном трафике.

- 

Есть поддержка взвешенного роутинга (Ingress/Mesh).

- 

Удобно подключать**canary-анализ**(автопромоут/автороллбэк) — Flagger/Argo Rollouts.

### []Blue-Green

вот полные файлы для работы с blueGreen

/etc/ansible/kubespray-official/helm-charts/13_blue_green/common-chart/templates/deployment.yaml

## Оригинал

https://sidmid.ru/%d1%83%d1%81%d1%82%d0%b0%d0%bd%d0%be%d0%b2%d0%ba%d0%b0-k8s-%d1%81%d0%be-%d0%b2%d1%81%d0%b5%d0%b9-%d0%be%d0%b1%d0%b2%d1%8f%d0%b7%d0%ba%d0%be%d0%b9/
