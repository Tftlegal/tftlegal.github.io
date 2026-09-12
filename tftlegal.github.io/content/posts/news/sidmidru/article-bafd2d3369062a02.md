---
title: "testrail - docker swarm(compose)"
date: 2026-07-11T05:33:25Z
source: "sidmidru"
original_url: "https://sidmid.ru/testrail-docker-swarmcompose/"
summary: "Текст представляет собой короткий фрагмент веб-страницы с перечнем файлов конфигурации для TestRail. Указаны ссылки на два файла: testrail-config.php_v4 и testrail-apache-prefork.conf. Эти файлы, скорее всего, служат примерами настроек TestRail и Apache в режиме prefork. Далее сообщается, что статья о TestRail в Docker Swarm и Compose появилась на сайте sidmid.ru. Материал связан с развертыванием или настройкой TestRail в Docker-окружении."
---

# testrail - docker swarm(compose)

## Краткое содержание

Текст представляет собой короткий фрагмент веб-страницы с перечнем файлов конфигурации для TestRail.
Указаны ссылки на два файла: testrail-config.php_v4 и testrail-apache-prefork.conf.
Эти файлы, скорее всего, служат примерами настроек TestRail и Apache в режиме prefork.
Далее сообщается, что статья о TestRail в Docker Swarm и Compose появилась на сайте sidmid.ru.
Материал связан с развертыванием или настройкой TestRail в Docker-окружении.

## Полная статья

| 12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273747576777879 | version: '3.7'networks: testrail: driver: overlayvolumes: cassandra_data:configs: config.php: external: true name: testrail-config.php_v4 prefork.conf: external: true name: testrail-apache-prefork.confservices: testrail: image: registry.com/platform/testrail-faketime:7.5.3 networks: - testrail configs: - source: config.php target: /var/www/testrail/config/config.php - source: prefork.conf target: /etc/apache2/mods-available/mpm_prefork.conf environment: FAKETIME: -5184000 MYSQL_USER: testrail MYSQL_PASSWORD: j0-k34 MYSQL_DATABASE: testrail MYSQL_HOST_OLD: HOST-rc1a-6 MYSQL_HOST: HOST-c-c9q2 ports: - target: 80 published: 7598 protocol: tcp mode: host logging: driver: "json-file" options: max-size: "10m" max-file: "3" deploy: restart_policy: condition: on-failure delay: 30s replicas: 1 placement: constraints: - "node.hostname == swarm1" cassandra: image: testrail/cassandra:latest networks: - testrail volumes: - 'cassandra_data:/var/lib/cassandra' environment: - HEAP_NEWSIZE=128M - MAX_HEAP_SIZE=512M logging: driver: "json-file" options: max-size: "10m" max-file: "3" deploy: resources: limits: memory: 1500M restart_policy: condition: on-failure max_attempts: 50 replicas: 1 placement: constraints: - "node.hostname == swarm1" |
| --- | --- |

Thank you for reading this post, don't forget to subscribe!

testrail-config.php_v4

| 1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859 | <?php/*|--------------------------------------------------------------------| DATABASE CONFIGURATION|--------------------------------------------------------------------|| Please specify the database connection settings below. Currently| supported databases are SQL Server (2005, 2008, 2012) and MySql (5).*/define('DB_DRIVER', 'mysql'); // sqlsrv or mysqldefine('DB_HOSTNAME', 'HOST-rc1a-6h24);define('DB_DATABASE', 'testrail');define('DB_USERNAME', 'testrail');define('DB_PASSWORD', 'fej0-k3lji');/*|--------------------------------------------------------------------| DIAGNOSTICS|--------------------------------------------------------------------|| The following settings configure the logging and error behavior| of TestRail.*/define('LOG_PATH', '/usr/local/testrail/logs/');/*|--------------------------------------------------------------------| OPTIMIZATIONS|--------------------------------------------------------------------|| You can choose whether to optimize the delivery of style sheet and| javascript files and the handling of language files. The following| optimization settings are available:|| DEPLOY_OPTIMIZE_LANG: If enabled, TestRail uses a single combined| language file named 'all_lang' instead of| multiple language files.|| DEPLOY_OPTIMIZE_CSS: If enabled, a single combined style sheet| is served to the clients.|| DEPLOY_OPTIMIZE_JS: If enabled, a single combined javascript| file is served to the clients.*/define('DEPLOY_OPTIMIZE_LANG', true);define('DEPLOY_OPTIMIZE_CSS', true);define('DEPLOY_OPTIMIZE_JS', true);define('CASSANDRA_HOSTNAME', 'cassandra');define('CASSANDRA_PORT', 9042);define('CASSANDRA_KEYSPACE', 'testrail');define('CASSANDRA_USERNAME', 'cassandra');define('CASSANDRA_PASSWORD', 'cassandra');define('CASSANDRA_SCHEMA_VERSION', 2); |
| --- | --- |

testrail-apache-prefork.conf

|  | <IfModule mpm_prefork_module> StartServers 1 MinSpareServers 1 MaxSpareServers 2 MaxRequestWorkers 2 MaxConnectionsPerChild 1000</IfModule> |
| --- | --- |

## Навигация по записям

## https://github.com/midnight47/

## Оригинал

https://sidmid.ru/testrail-docker-swarmcompose/
