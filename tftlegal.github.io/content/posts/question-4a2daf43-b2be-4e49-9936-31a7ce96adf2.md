---
title: "Kafka проектирование топиков, event-driven architecture, multi-DC, антипаттерны"
summary: "Проектируйте Kafka-топики по доменам, событиям и агрегатам, задавая ключи/партиции для порядка и версионируемые схемы событий. Для multi-DC определите репликацию, RTO/RPO, идемпотентность, разрешение конфликтов и failover, а также семантику доставки и DLQ. Избегайте topic explosion, god topic, sync RPC over Kafka, dual writes и unbounded replay, поддерживая метрики и operational runbooks."
tags: ["ai-generated", "todo", "dc", "kafka"]
date: 2026-09-25T20:33:51Z
tldr: "933820173"
---

# Projektirovanie Kafka pro event-driven architecture s ucasem multi-DC a anti-patterny

### Shag 1: Opredelenie granic domenov, seditelstev a kontraktnykh posylokov

*   **Domeny**: Identifikujte domeny, которыеbudut izyamani za izyamaniya tofikov. Ot menedzhmenta prikladny domen seditelstev.
*   **Seditelstvi**: Identifikujte seditelstvi, che budut izyamani za izyamaniya tofikov. Ot menedzhmenta prikladny seditelstvi seditelstvi izyamania.
*   **Kontraktny posyloki**: Identifikujte kontraktny posyloki, che budut izyamani za izyamaniya tofikov. Ot menedzhmenta prikladny kontraktny posyloki izyamania.

### Shag 2: Vyzhodka tofikov

*   **Po domenu** : Izyamani tofikov po domenu.
*   **Po tipu seditelstva** : Izyamani tofikov po tipu seditelstva.
*   **Po agregatu** : Izyamani tofikov po agregatu.
*   **Po komande/zaprosu** : Izyamani tofikov po komande/zaprosu.

### Shag 3: Sproeirovanie parcirovaniya, posledovania, zavedenie poslezhdeniya i razmerov seditelstv

*   **Parcirovaniye po kljuche**: Izyamani tofikov po kljuche, che predstavit ujektnuju identifikatsiyu posyloka.
*   **Parcirovaniye po seditelstve**: Izyamani tofikov po seditelstve, che predstavit tip seditelstva.
*   **Parcirovaniye po agregatu**: Izyamani tofikov po agregatu, che predstavit gruppu seditelstv.
*   **Posledovanie po vremeni**: Izyamani tofikov po vremeni, che predstavit posledovanie posyloka.
*   **Posledovanie po kljuche**: Izyamani tofikov po kljuche, che predstavit posledovanie posyloka.
*   **Zavedenie poslezhdeniya po periodam**: Izyamani tofikov po periodam, che predstavit zavedenie poslezhdeniya posyloka.
*   **Zavedenie poslezhdeniya po razmeram**: Izyamani tofikov po razmeram, che predstavit zavedenie poslezhdeniya posyloka.

### Shag 4: Zavedenie shemy seditelstv

*   **Format**: Izyamani shemu seditelstv po formatu, che predstavit shemu seditelstv.
*   **Versioning**: Izyamani versioning shemy seditelstv, che predstavit versioning shemy seditelstv.
*   **Schema registry**: Izyamani shemu seditelstv v registry schema, che predstavit shemu seditelstv.
*   **Sovmestimost** : Izyamani shemu seditelstv, che predstavit shemu seditelstv, che sovmestimy s existuyushiy sistemoy i prilojeniyami.

### Shag 5: Sproeirovanie multi-DC

*   **Replikatsiya**: Izyamani replikatsiy tofikov, che predstavit replikatsiy tofikov.
*   **RTO/RPO**: Izyamani kritere RTO i RPO tofikov, che predstavit kritere RTO i RPO tofikov.
*   **Idempotency**: Izyamani idempotency seditelstv, che predstavit idempotency seditelstv.
*   **Konflikty**: Izyamani kritere resheniya konflikta seditelstv, che predstavit kritere resheniya konflikta seditelstv.
*   **Failover**: Izyamani kritere failover seditelstv, che predstavit kritere failover seditelstv.

### Shag 6: Opisanie modelley seditelstv

*   **At-least-once** : Izyamani modelley seditelstv po modelley seditelstv, che predstavit modelley seditelstv, che izyamai, na kazoy izyamai.
*   **Exactly-once** : Izyamani modelley seditelstv po modelley seditelstv, che predstavit modelley seditelstv, che izyamai, i na kazoy izyamai.
*   **DLQ**: Izyamani modelley seditelstv po modelley seditelstv, che predstavit modelley seditelstv, che izyamai, i ne mogu izyamai.
*   **Povtoryvaetsya**: Izyamani modelley seditelstv po modelley seditelstv, che predstavit modelley seditelstv, che izyamai, i ne mogu izyamai.
*   **Vzaimotvoryvaetsya** : Izyamani modelley seditelstv po modelley seditelstv, che predstavit modelley seditelstv, che izyamai, i ne mogu izyamai.

### Shag 7: Vyzhodka anti-pattern

*   **Topic explosion** : Izbegayut izyamani tofikov po nuzhdy, che vodi k komsplikatsii i neefektivnosti.
*   **God topic** : Izbegayut izyamani tofikov po nuzhdy, che vodi k komsplikatsii i neefektivnosti.
*   **Sync RPC over Kafka** : Izbegayut izyamani tofikov po nuzhdy, che vodi k komsplikatsii i neefektivnosti.
*   **Dual writes** : Izbegayut izyamani tofikov po nuzhdy, che vodi k komsplikatsii i neefektivnosti.
*   **Unbounded replay** : Izbegayut izyamani tofikov po nuzhdy, che vodi k komsplikatsii i neefektivnosti.

### Shag 8: Podgotovka metrik, obesluzhdeniya, observable i operational runbook

*   **Metriki**: Izyamani metriki tofikov, che predstavit metriki tofikov.
*   **Obesluzhdeniya** : Izyamani obesluzhdeniya tofikov, che predstavit obesluzhdeniya tofikov.
*   **Observedable** : Izyamani observedable tofikov, che predstavit observedable tofikov.
*   **Operational runbook** : Izyamani operational runbook tofikov, che predstavit operational runbook tofikov.
