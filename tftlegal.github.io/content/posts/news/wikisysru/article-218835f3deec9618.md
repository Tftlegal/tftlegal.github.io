---
title: "12. Kubernetes. Сетевое взаимодействие подов разных нод. CNI-плагины. NetworkPolicy"
date: 2026-03-08T15:06:00Z
original_url: "https://wikisys.ru/11-kubernetes-%d1%81%d0%b5%d1%82%d0%b5%d0%b2%d0%be%d0%b5-%d0%b2%d0%b7%d0%b0%d0%b8%d0%bc%d0%be%d0%b4%d0%b5%d0%b9%d1%81%d1%82%d0%b2%d0%b8%d0%b5-%d0%bf%d0%be%d0%b4%d0%be%d0%b2-%d1%80%d0%b0%d0%b7%d0%bd/"
---

Кратко: речь о взаимодействии подов на двух воркер-нодах — два виртуальных хоста с адресами 172.16.100.1 и 172.16.100.2, вместе с роутером, находятся в одном широковещательном домене.

## Оригинал

https://wikisys.ru/11-kubernetes-%d1%81%d0%b5%d1%82%d0%b5%d0%b2%d0%be%d0%b5-%d0%b2%d0%b7%d0%b0%d0%b8%d0%bc%d0%be%d0%b4%d0%b5%d0%b9%d1%81%d1%82%d0%b2%d0%b8%d0%b5-%d0%bf%d0%be%d0%b4%d0%be%d0%b2-%d1%80%d0%b0%d0%b7%d0%bd/
