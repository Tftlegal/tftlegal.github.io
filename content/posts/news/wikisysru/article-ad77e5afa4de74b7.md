---
title: "11. Kubernetes. Сетевое взаимодействие подов одной ноды"
date: 2026-03-08T15:05:00Z
original_url: "https://wikisys.ru/10-kubernetes-%d1%81%d0%b5%d1%82%d0%b5%d0%b2%d0%be%d0%b5-%d0%b2%d0%b7%d0%b0%d0%b8%d0%bc%d0%be%d0%b4%d0%b5%d0%b9%d1%81%d1%82%d0%b2%d0%b8%d0%b5-%d0%bf%d0%be%d0%b4%d0%be%d0%b2-%d0%be%d0%b4%d0%bd%d0%be/"
---

Кратко: в Kubernetes у каждого пода есть свой IP-адрес внутри кластера, поэтому поды могут общаться друг с другом напрямую, без NAT, а сервисы и ноды обеспечивают стабильный доступ и балансировку.

## Оригинал

https://wikisys.ru/10-kubernetes-%d1%81%d0%b5%d1%82%d0%b5%d0%b2%d0%be%d0%b5-%d0%b2%d0%b7%d0%b0%d0%b8%d0%bc%d0%be%d0%b4%d0%b5%d0%b9%d1%81%d1%82%d0%b2%d0%b8%d0%b5-%d0%bf%d0%be%d0%b4%d0%be%d0%b2-%d0%be%d0%b4%d0%bd%d0%be/
