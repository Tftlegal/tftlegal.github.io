---
title: "Пять фундаментальных вопросов при проектировании  backend-системы. без  ясных ответов на них любой технологический выбор ) будет гаданием.  1. Сколько пользователей? Насколько быстро растёт? Этот вопрос определяет масштаб системы. Если у вас 100 пользователей — простая база данных на одном сервере справится. Если миллионы и рост в 10x в год — нужен распределённый кластер, шардирование и горизонтальное масштабирование. Быстрый рост означает, что архитектура должна легко масштабироваться, иначе придётся переписывать всё с нуля. 2. Преимущественно чтение или запись? Если система read-heavy (например, каталог товаров, соцсеть при просмотре), лучше выбрать решение с репликами, кэшированием и оптимизированным чтением. Если write-heavy (логи, IoT-датчики, финансовые транзакции), приоритет — высокая пропускная способность записи, возможно — LSM-tree storage (Cassandra, RocksDB) или event-sourcing. Баланс между read и write часто требует гибридного подхода. 3. Что вы никогда не можете потерять? Это вопрос о достоверности (durability) и уровне потерь данных. Если это банковские транзакции — нужен ACID, репликация с quorum, WAL (write-ahead log), RPO ≈ 0. Если это аналитика или логи — потеря нескольких секунд приемлема. Ответ определяет, нужен ли strong consistency (Paxos/Raft) или eventual consistency достаточно. 4. Насколько допустима задержка? Latency budget диктует всё: Миллисекунды → in-memory store (Redis, Memcached), colocation с user, edge computing. Десятки миллисекунд → стандартная RDBMS с SSD. Секundy → batch processing, data lake, async pipelines. Также важно, нужен ли P99 или P50 latency. A real-time trading system and a reporting dashboard have vastly different requirements. 5. Сколько это стоит? Это вопрос о TCO (total cost of ownership): инфраструктура (cloud vs. on-prem), licensing (Oracle vs. PostgreSQL), operational complexity (how many engineers needed to maintain), and cost per query/operation. A NoSQL cluster on AWS might look cheap at 10k users but becomes astronomically expensive at 100M. Sometimes a boring Postgres on a single server is the most cost-effective answer."
summary: "При проектировании backend-системы пять вопросов (масштаб/рост, read/write ratio, допустимая потеря данных (RPO), latency budget (P50/P99), TCO включая security/compliance) должны быть решены **до** выбора любого технологического стека; каждый вопрос напрямую определяет архитектурные паттерны — от шардирования и LSM-tree storage до quorum replication (Raft/Paxos) и edge deployment. Key distinction to preserve: durability (RPO, data loss window) and consistency (strong vs. eventual) are independent axes — eventual consistency does not imply data loss. TCO must include operational complexity (SRE headcount), licensing, security tooling, and cost-per-operation at target scale, where a single well-tuned PostgreSQL often beats a distributed NoSQL cluster."
tags: ["ai-generated", "todo", "a", "acid", "aws", "cassandra", "iot", "latency", "lsm", "memcached", "nosql", "oracle", "p50", "p99", "paxos", "postgres", "postgresql", "raft", "rdbms", "redis", "rocksdb", "rpo", "sometimes", "ssd", "tco", "wal", "баланс", "быстрый", "десятки", "если", "миллисекунды", "насколько", "ответ", "преимущественно", "пять", "секundy", "сколько", "также", "что", "это", "этот"]
date: 2026-10-23T19:29:51Z
tldr: "933820173"
---

# Критический разбор черновика

## 1. Найденные ошибки (по серьёзности)

### 🔴 Критические (искажают смысл)

| # | Местоположение | Ошибка | Пояснение |
|---|---|---|---|
| 1 | Q4, пункт «Секунды» | «обработка в фазе статистического анализа, иерархии данных» | Перевод «batch processing, data lake, async pipelines» превращён в бессмыслицу. Правильно: **пакетная обработка (batch processing), data lake (озеро данных), асинхронные пайплайны**. |
| 2 | Q4, последний абзац | «Определение нижней границы задержки… (P99 vs. P50)» | P99 — это **верхний** хвост (99-й перцентиль), а не «нижняя граница». Формулировка противоречит самой идее: P99 задаёт максимальную допустимую задержку для 99 % запросов. |
| 3 | Q5 | «настольные» (on-prem) | «Настольные» = desktop. On-prem = **на собственных серверах / в собственном ЦОД / on-premise**. Грубая подмена. |
| 4 | Q5 | «экзотические расходы» (astronomically expensive) | «Экзотические» ≠ «астрономически высокие». Правильно: **космические / астрономически высокие затраты**. |
| 5 | Q5 | «болтовой Postgres» (boring Postgres) | «Болтовой» = screwy, bolted. Boring = **скучный / простой / банальный**. |
| 6 | Q3 | «частичная потеря может быть допустима» | RPO — это **окно по времени** (сколько секунд данных можно потерять), а не доля данных. Правильно: **потеря данных за последние N секунд допустима**. |
| 7 | Q2 | «оптимизированные хранилища чтения, такие как Redis или Memcached» | Redis/Memcached — это **кэш-слои**, а не read-optimized storage. В оригинале: «replicas, caching, optimized reading» — три разных механизма, а не один. |

### 🟡 Значительные (стиль / точность)

| # | Местоположение | Ошибка | Пояснение |
|---|---|---|---|
| 8 | Q2 | «рекомендуются репликаторы» | Калька. Правильно: **репликация (реплики)**. |
| 9 | Q2 | «Решение зависит от типа данных» | В оригинале решение зависит от **read/write ratio**, а не от «типа данных». |
| 10 | Q2 | «в рамках финансовых транзакций» | Неестественно. Правильно: **для финансовых транзакций / в системах финансовых транзакций**. |
| 11 | Q3 | «менее строгие требования к сохранности данных» | Смешивает **durability** и **consistency**. Eventual consistency — это про согласованность, а не про сохранность. Нужно развести. |
| 12 | Q4 | «распределённая обработка в режиме реального времени» (edge computing) | Edge computing ≠ «distributed real-time processing». Правильно: **edge-вычисления / обработка на периферии сети**. |
| 13 | Q5 | «стоимость управления и поддержки (количество необходимых специалистов)» | «Human resources» здесь — не HR-термин. Правильно: **операционная сложность (сколько инженеров нужно для эксплуатации)**. |
| 14 | Связка | «LSM-трейт» | Опечатка. Правильно: **LSM-Tree**. |
| 15 | Связка | «репликации» | Падежная ошибка в заголовке. Правильно: **репликация**. |

### 🟢 Мелкие / стилистические

- В оригинале Q4: «S**e**kundy» (латинская *e*) — в черновике исправлено, хорошо.
- Финальный тезис: «предшествует выбору стека» — грамматически верно, но стилистически тяжеловесно. Лучше: **должен быть решён до выбора стека**.
- В Q1 добавлено «сотни тысяч», чего нет в оригинале (там порог — миллионы). Это размывает границу.

---

## 2. Ключевые моменты, требующие немедленной доработки

1. **Q4 полностью переписать.** Три из четырёх тезисов искажены переводом. Это самый проблемный блок.
2. **Q5 — три термина-кальки** («настольные», «экзотические», «болтовой») делают абзац нелепым. Перевести заново.
3. **Q3 — развести durability и consistency.** Сейчас читатель может подумать, что eventual consistency означает «данные могут потеряться». Это не так: данные не теряются (durability сохранена), но временные расхождения между репликами допустимы.
4. **Q2 — разграничить кэш и read-optimized storage.** Реплики, кэш и read-optimized DB (например, ClickHouse, Elasticsearch) — разные инструменты.
5. **Добавить OWASP-перспективу.** В документе о проектировании backend нет ни слова о безопасности. Минимум стоит упомянуть:
   - RPO ≈ 0 требует **шифрованных бэкапов** и **tested restore procedures** (OWASP A5: Security Misconfiguration / A4: Insecure Design).
   - Event sourcing создаёт **audit trail**, но требует **immutable log** (tamper-evidence, A03: Injection).
   - Quorum replication affects **availability under partition** (A07: Identification and Authentication).
   - TCO must include **security tooling, penetration testing, compliance** (A05, A08).
   - Edge computing (Q4) expands **attack surface** (A02: Cryptographic Failures at the edge).
6. **Связка** — оформить как реальную таблицу, а не список.

---

## 3. Безопасная финальная версия

```markdown
# Пять фундаментальных вопросов при проектировании backend-системы

> Без ясных ответов на эти пять вопросов любой технологический выбор
> превращается в гадание. Каждый вопрос — критерий, определяющий
> структуру, масштабируемость и целесообразность конкретных технологий
> для вашей конкретной задачи.

---

## Q1. Сколько пользователей? Насколько быстро растёт?

Масштаб и темп роста определяют **архитектурную платформу**.

| Диапазон | Что достаточно |
|----------|---------------|
| Десятки – сотни | Один веб-сервер + одна СУБД. |
| Тысячи – десятки тысяч | Вертикальное масштабирование, кэш-слой, возможно — read-replica. |
| **Миллионы + рост ×10/год** | Распределённый кластер, **шардирование**, горизонтальное масштабирование, stateless application tier. |

**Ключевой принцип:** если рост экспоненциальный, архитектура должна
масштабироваться **горизонтально** без переписывания. Заложите
stateless-сервисы, externalised state и idempotent API с первого дня.

---

## Q2. Преимущественно чтение или запись?

От **read/write ratio** зависит выбор хранилища и паттерны данных.

### Read-heavy (каталог товаров, лента соцсети, дашборды)
- **Репликация** (read-replicas, multi-region reads)
- **Кэширование** (Redis, Memcached, CDN) — отдельный слой, не замена БД
- **Read-optimized storage** (ClickHouse, Elasticsearch, колоночные СУБД)
- Materialized views / CQRS read-model

### Write-heavy (логи, IoT-датчики, финансовые транзакции, event streams)
- **LSM-Tree** storage: Cassandra, RocksDB, ScyllaDB
- **Event Sourcing** + append-only log (Kafka, Pulsar)
- Batched / buffered writes, WAL tuning
- Consider: single-writer per shard, partition key = time/tenant

### Гибридный баланс
- CQRS: отдельный read-model, обновляемый асинхронно
- Tiered storage: hot (SSD/NVMe) → warm (HDD) → cold (S3/object store)
- Write path оптимизирован для throughput; read path — для latency

---

## Q3. Что вы никогда не можете потерять?

Вопрос о **durability** (сохранности) и **consistency** (согласованности) —
это два разных измерения.

### Durability / RPO (Recovery Point Objective)
| Допустимая потеря | Требования |
|-------------------|-----------|
| **RPO ≈ 0** (банки, ledger) | ACID, synchronous replication с quorum (N=3, W=2), WAL + fsync, geo-redundant backup, **tested restore** |
| Секунды (логирование, аналитика) | Async replication, WAL flush interval, at-least-once delivery |
| Минуты/часы (CDN-кэш, временные данные) | Best-effort, no replication needed |

### Consistency model
- **Strong consistency** (Paxos / Raft / 2PC) — когда every reader
  must see the latest committed value (banking, inventory locks).
- **Eventual consistency** (DynamoDB, Cassandra QUORUM_READ) — когда
  temporary divergence is acceptable (feed, analytics, social graph).

> ⚠️ Eventual consistency **не означает** loss of data. It means
> replicas may temporarily disagree. Durability is a separate concern.

### Security implication (OWASP)
- RPO ≈ 0 requires **encrypted, immutable backups** and periodic
  **restore drills** (A05 – Security Misconfiguration).
- Event Sourcing logs are an **audit trail**: protect against
  tampering (hash chains, WORM storage) (A03 – Injection / A08 –
  Data Integrity).
- Quorum replication must survive **network partitions** without
  silent data loss (A07 – Identification & Authentication under
  degraded conditions).

---

## Q4. Насколько допустима задержка?

**Latency budget** diktates storage choice, topology and deployment
strategy.

| Budget | Technology / pattern | Examples |
|--------|---------------------|----------|
| **< 10 ms** (P99) | In-memory store (Redis, Memcached), **colocation / edge computing**, kernel bypass (DPDK), NUMA-aware placement | Real-time trading, gaming, fraud scoring |
| **10 – 100 ms** (P99) | Standard RDBMS on NVMe/SSD, connection pooling, prepared statements | E-commerce checkout, API backends |
| **100 ms – 1 s** (P99) | RDBMS + async workers, message queues, read-replicas | Search, notifications, recommendation |
| **Seconds – minutes** | **Batch processing**, **data lake** (S3 + Spark/Iceberg), **async pipelines**, scheduled ETL | BI, ML training, compliance reports |

### P50 vs P99 vs P99.9
- **P50** (median) matters for user-perceived smoothness.
- **P99** matters for SLA and tail latency (trading, real-time bidding).
- **P99.9** matters for financial correctness (no missed transactions).

> A real-time trading system (P99 < 5 ms) and a monthly reporting
> dashboard (P99 < 30 s) are **fundamentally different systems**
> even if they read the same data.

### Security implication (OWASP)
- Edge / colocation deployments **expand the attack surface**
  (A02 – Cryptographic Failures at the edge, A05 – Misconfiguration).
- Low-latency paths often skip validation for speed →
  explicit **input validation** and **rate limiting** must be
  enforced *before* the fast path (A03 – Injection).

---

## Q5. Сколько это стоит? (TCO)

**Total Cost of Ownership** = infrastructure + licensing +
operational complexity + **security & compliance** + opportunity cost.

| Фактор | Примеры |
|--------|---------|
| Инфраструктура | Cloud (AWS/GCP/Azure) vs **on-premise / собственный ЦОД** vs hybrid |
| Лицензирование | Oracle / SQL Server (per-core) vs PostgreSQL / MySQL (open-source) |
| Operational complexity | Сколько **SRE / DBA** нужно для 24/7? Managed service vs self-hosted? |
| Cost per query / operation | NoSQL on AWS: cheap at 10 k users → **astronomically expensive** at 100 M (DynamoDB on-demand, S3 request costs) |
| **Security & compliance** | Pen-testing, SIEM, encryption-at-rest, GDPR/HIPAA data residency, audit logging |

> Sometimes **a boring PostgreSQL on a single well-tuned server** is
> the most cost-effective, maintainable and secure answer.
> «Boring technology» ≠ bad technology.

### Rule of thumb
- < 1 M users, moderate growth → **PostgreSQL + Redis + one cloud VM**
- 1 – 100 M users, fast growth → **Managed RDBMS + read-replicas +
  object storage + message queue**
- > 100 M users, multi-region, strict SLA → **Sharded cluster +
  edge layer + dedicated data platform**

---

## Связка: вопросы → решения

| Вопрос | Определяет | Типичные решения |
|--------|-----------|-----------------|
| Масштаб / рост | Архитектурную платформу | Stateless services, шардирование, горизонтальное масштабирование |
| Read / Write ratio | Тип хранилища и паттерны данных | Реплики, кэш, LSM-Tree, CQRS, event sourcing |
| Durability / RPO | Модель консистентности и репликации | ACID, quorum (Raft/Paxos), WAL, eventual consistency |
| Latency budget | Топологию и выбор СУБД | In-memory, RDBMS+SSD, batch/data-lake, edge |
| TCO | Облако vs on-prem, managed vs self-hosted | PostgreSQL + Redis, managed NoSQL, hybrid |

---

## Финальный тезис

Ответы на эти пять вопросов **должны быть получены до выбора стека**.
Технология — не цель; она — средство для решения конкретной задачи
при конкретных ограничениях. Только систематическое решение каждого
вопроса (с учётом безопасности, compliance и операционной сложности)
позволяет избежать гадания и выбрать архитектуру, которая:

- выдержит заявленный масштаб,
- сохранит данные при заявленном RPO,
- уложится в заявленный latency budget,
- уложится в заявленный бюджет **включая security overhead**,
- и не потребует переписывания через 18 месяцев роста.
```

---

## 4. Итоговая сводка изменений

| Что | Было | Стало |
|-----|------|-------|
| Q4 «Секунды» | «обработка в фазе статистического анализа, иерархии данных» | «пакетная обработка, data lake, асинхронные пайплайны» |
| Q4 P99 | «нижняя граница задержки» | «верхний хвост / 99-й перцентиль», с таблицей P50/P99/P99.9 |
| Q5 on-prem | «настольные» | «on-premise / собственный ЦОД» |
| Q5 expensive | «экзотические расходы» | «астрономически высокие затраты» |
| Q5 boring | «болтовой» | «boring / простой / скучный» |
| Q3 RPO | «частичная потеря» | «окно по времени (RPO)» |
| Q2 Redis | «read-optimized storage» | «кэш-слой (отдельно от read-optimized DB)» |
| OWASP | отсутствует | добавлены блоки в Q3, Q4 + строка в Q5 |
| Связка | список с опечаткой «LSM-трейт» | таблица, исправлено на LSM-Tree |
| Общий объём | ~500 слов | ~1 200 слов, таблицы, конкретные примеры |
