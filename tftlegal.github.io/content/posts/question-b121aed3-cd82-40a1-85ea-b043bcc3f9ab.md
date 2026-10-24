---
title: "Пять фундаментальных вопросов при проектировании  backend-системы. без  ясных ответов на них любой технологический выбор ) будет гаданием.  1. Сколько пользователей? Насколько быстро растёт? Этот вопрос определяет масштаб системы. Если у вас 100 пользователей — простая база данных на одном сервере справится. Если миллионы и рост в 10x в год — нужен распределённый кластер, шардирование и горизонтальное масштабирование. Быстрый рост означает, что архитектура должна легко масштабироваться, иначе придётся переписывать всё с нуля. 2. Преимущественно чтение или запись? Если система read-heavy (например, каталог товаров, соцсеть при просмотре), лучше выбрать решение с репликами, кэшированием и оптимизированным чтением. Если write-heavy (логи, IoT-датчики, финансовые транзакции), приоритет — высокая пропускная способность записи, возможно — LSM-tree storage (Cassandra, RocksDB) или event-sourcing. Баланс между read и write часто требует гибридного подхода. 3. Что вы никогда не можете потерять? Это вопрос о достоверности (durability) и уровне потерь данных. Если это банковские транзакции — нужен ACID, репликация с quorum, WAL (write-ahead log), RPO ≈ 0. Если это аналитика или логи — потеря нескольких секунд приемлема. Ответ определяет, нужен ли strong consistency (Paxos/Raft) или eventual consistency достаточно. 4. Насколько допустима задержка? Latency budget диктует всё: Миллисекунды → in-memory store (Redis, Memcached), colocation с user, edge computing. Десятки миллисекунд → стандартная RDBMS с SSD. Секundy → batch processing, data lake, async pipelines. Также важно, нужен ли P99 или P50 latency. A real-time trading system and a reporting dashboard have vastly different requirements. 5. Сколько это стоит? Это вопрос о TCO (total cost of ownership): инфраструктура (cloud vs. on-prem), licensing (Oracle vs. PostgreSQL), operational complexity (how many engineers needed to maintain), and cost per query/operation. A NoSQL cluster on AWS might look cheap at 10k users but becomes astronomically expensive at 100M. Sometimes a boring Postgres on a single server is the most cost-effective answer."
summary: "Пять взаимно ограничивающих вопросов (масштаб/рост, R/W ratio, RPO+RTO+consistency model, P99 latency budget, 3-year TCO including people/security/compliance) сужают пространство архитектурных решений; по CAP/PACELC их оптимум is physically incompatible, so the architect must explicitly choose which constraint to sacrifice. Security (OWASP Top 10), data sovereignty (GDPR/HIPAA/152-ФЗ), observability, and team operating capacity are cross-cutting constraints that must be folded into every answer, not bolted on later. The correct stack is the simplest one that satisfies all five constraints simultaneously — often a single PostgreSQL behind Nginx — and complexity should match the team's ability to operate it at 3 AM."
tags: ["ai-generated", "todo", "a", "acid", "aws", "cassandra", "iot", "latency", "lsm", "memcached", "nosql", "oracle", "p50", "p99", "paxos", "postgres", "postgresql", "raft", "rdbms", "redis", "rocksdb", "rpo", "sometimes", "ssd", "tco", "wal", "баланс", "быстрый", "десятки", "если", "миллисекунды", "насколько", "ответ", "преимущественно", "пять", "секundy", "сколько", "также", "что", "это", "этот"]
date: 2026-10-24T11:27:19Z
tldr: "933820173"
---

# Критический разбор и исправленная версия

## Найденные ошибки и баги

### Орфография и грамматика
| # | Было | Стало |
|---|------|-------|
| 1 | «100 пользователеми» | «100 пользователями» |
| 2 | «LSM-трейсты» | «LSM-tree (Log-Structured Merge-Tree)» |
| 3 | «асинхронные пиаркуеты» | «асинхронные пайплайны» |
| 4 | «Костей на operations» | «Cost per operation» |
| 5 | «стоимости произведения операций» | «стоимости на операцию/транзакцию» |
| 6 | «по масштабою» | «по масштабу» |
| 7 | «Активно проверить» | «Проверьте актуальность потребности» |
| 8 | «Количеству SRE» | «Количество SRE-инженеров» |
| 9 | «самостоятельность лицензий» | «модель лицензирования» |
| 10 | «Расходы по расположению» | «Региональное ценообразование / data residency» |
| 11 | «инвентаризация, обучение» | «интеграция, миграция, обучение» |
| 12 | «Банк транзакций» | «Банковские транзакции» |
| 13 | «No Order» в матрице | «Read-heavy (batch)» |

### Логические и архитектурные ошибки
| # | Проблема | Пояснение |
|---|----------|-----------|
| 14 | «ACID не гарантирует always eventual consistency» | Смешение двух ортогональных осей. ACID — про атомарность/консистентность *внутри* транзакции. Eventual consistency — про *меж-узеловую* видимость. Они не «гарантируют друг друга». |
| 15 | Матрица: «Банковские транзакции → PostgreSQL + Patroni» | Patroni решает HA/failover, а не strong cross-node consistency. Нужна *synchronous* replication (`synchronous_commit = on`) или Raft-based (CockroachDB, Spanner). |
| 16 | Матрица: столбец «RPO» содержит «ACID» | RPO — это *time* (seconds/minutes). ACID — это *property*. Смешение единиц. |
| 17 | Отсутствует CAP / PACELC | Без упоминания теоремы CAP и PACELC обсуждение consistency vs. availability vs. partition-tolerance неполно. |
| 18 | P50 vs P99 объяснение тривиально | Не указано, *почему* P99 критичен: tail latency → cascading failures, circuit-breaker trips, SLO violations. |
| 19 | ТCO без security-затрат | Шифрование, WAF, SIEM, pen-testing, compliance audits — все это TCO. |
| 20 | Полное отсутствие OWASP / безопасности | Для backend-дизайна это критический пробел. |

---

## Ключевые моменты, требующие немедленной доработки

1. **Безопасность (OWASP Top 10 2021)** — ни один из пяти вопросов не затрагивает A01–A10. Нужно добавить как поперечное ограничение или как 6-й вопрос.
2. **CAP / PACELC** — фундаментальная теорема, без которой выбор consistency model не обоснован.
3. **RTO (Recovery Time Objective)** — в вопросе 3 упомянут только RPO. Для production-систем RTO не менее важен.
4. **Взаимодействие пяти вопросов** — они не независимы. «Millions of users + ms latency + strong consistency + lowest TCO» is physically impossible (CAP).
5. **Data sovereignty / compliance** (GDPR, HIPAA, 152-ФЗ) — влияет на where data lives, which cloud region, encryption requirements.
6. **Observability** — metrics, structured logging, distributed tracing (OpenTelemetry) — не упомянуты, но критичны для ops.
7. **Team expertise & hiring** — «boring Postgres» is best *if* your team knows Postgres.

---

## Финальная исправленная версия

```markdown
# Пять фундаментальных вопросов при проектировании backend-системы

Без ясных ответов на пять вопросов ниже **любой** технологический выбор
становится гаданием. Каждый вопрос сужает пространство решений; вместе
они образуют систему ограничений, из которой архитектура вытекает
почти детерминированно.

> **Важно:** вопросы не независимы. «Миллионы пользователей + миллисекунд-
> ная латентность + strong consistency + минимальный TCO» — это
> физически недостижимая комбинация (см. CAP / PACELC). Задача
> архитектора — осознанно выбрать, *какое* ограничение жертвовать.

---

## 1. Сколько пользователей? Насколько быстро растёт?

**Что определяет:** масштаб, тип хранилища, необходимость шардирования.

| Пользователи | Рост/год | Типичный подход |
|---|---|---|
| < 1 000 | < 2× | Один PostgreSQL/MySQL, один VM, managed DB |
| 1 k – 1 M | 2–5× | Managed RDBMS + read replicas + Redis cache |
| 1 M – 100 M | 5–10× | Sharded cluster (CockroachDB / Vitess / Cassandra) + CDN + edge cache |
| > 100 M | 10×+ | Multi-region, CQRS, event-driven microservices, custom storage |

**Ключевые следствия:**
- Быстрый рост → проектируйте под **horizontal scaling** с самого начала
  (stateless API tier, externalized sessions, partitioned data).
- Уточните **concurrent** users, не только registered. 10 M registered
  ≠ 10 M concurrent.
- Пиковые нагрузки (Black Friday, viral event) могут быть 100× average.

---

## 2. Преимущественно чтение или запись?

**Что определяет:** тип движка, репликация topology, caching strategy.

### Read-heavy (каталоги, соцсети-лента, CMS)
- Read replicas + application-level / CDN caching.
- Denormalization, materialized views, CQRS read-model.
- Target: sub-10 ms P95 read.

### Write-heavy (IoT-телеметрия, event streams, финансовых ledgers)
- LSM-tree storage (Cassandra, RocksDB, ScyllaDB) для high-throughput writes.
- Event sourcing / append-only log (Kafka, Pulsar) + periodic compaction.
- Target: millions of writes/sec, relaxed read latency.

### Hybrid / mixed
- CQRS: separate read and write models, different storage engines.
- Tiered caching: L1 (in-process) → L2 (Redis) → L3 (DB).
- Write path: WAL → async commit → eventual read-model update.

> **OWASP A04 (Insecure Design):** Decide *where* authorization checks
> happen (edge / API-gateway / service / DB row-level). Read-heavy
> systems with public data still need per-user ACLs on write paths.

---

## 3. Что вы никогда не можете потерять? (Durability, RPO, RTO, Consistency)

Вопрос распадается на **четыре** под-вопроса:

| Под-вопрос | Вопрос | Пример |
|---|---|---|
| **RPO** (Recovery Point Objective) | Сколько данных можно потерять? | Bank: RPO ≈ 0. Analytics: RPO ≤ 5 min. |
| **RTO** (Recovery Time Objective) | How fast must the system be back? | Trading: < 1 s. E-commerce: < 30 min. |
| **Consistency model** | Strong (linearizable) or eventual? | Bank balance: linearizable. Feed: eventual. |
| **Durability mechanism** | WAL, sync replication, quorum? | `synchronous_commit = on`, Raft quorum (3/5). |

### Consistency spectrum (PACELC)

```
Linearizable (strong)  →  Sequential  →  Causal  →  Eventual  →  Bounded staleness
        Raft / 2PC              CQRS + seq. no.         Kafka offsets          TTL cache
```

- **ACID ≠ eventual consistency** — это ортогональные оси.
  ACID гарантирует атомарность/консистентность *внутри* транзакции.
  Eventual consistency описывает *меж-узеловую* видимость.
- Для strong cross-node consistency: CockroachDB, Spanner, PostgreSQL
  with `synchronous_commit = remote_apply` + Patroni (HA) — но Patroni
  alone does **not** give linearizability; it gives fast failover.
- For eventual: Cassandra (QUORUM reads), DynamoDB (strongly consistent
  optional), event-sourcing with read-model lag.

> **OWASP A08 (Software & Data Integrity Failures):** If RPO ≈ 0,
> you need tamper-evident logs (hash-chained WAL), signed snapshots,
> and backup integrity verification.

---

## 4. Насколько допустима задержка? (Latency budget)

**Что определяет:** storage engine, network topology, edge vs. central.

| Latency budget | Typical tech | Notes |
|---|---|---|
| < 1 ms (P99) | In-memory (Redis, Memcached), colocation, edge PoP | GPU/accelerated for ML inference |
| 1 – 10 ms (P99) | NVMe-backed RDBMS, in-memory cache layer | Most SaaS APIs |
| 10 – 100 ms (P99) | Standard RDBMS on SSD, managed DB | Acceptable for most CRUD |
| 100 ms – 1 s (P99) | Cross-region replication, OLAP queries | Dashboards, reporting |
| > 1 s (P99) | Batch, data lake, async pipelines | ETL, ML training, analytics |

### P50 vs P95 vs P99 vs P99.9

- **P50** tells you the "typical" user experience.
- **P99 / P99.9** reveals tail latency → cascading failures,
  circuit-breaker trips, SLO violations.
- A real-time trading system cares about P99.9 < 5 ms.
  A reporting dashboard cares about P50 < 2 s.
- **Security overhead adds latency:** TLS handshake, JWT verification,
  authorization checks, WAF inspection. Budget 1–5 ms for these.
  If your budget is < 5 ms total, you need session caching,
  pre-verified tokens, or mTLS at the mesh level.

> **OWASP A07 (Identification & Authentication Failures):**
> Multi-factor auth, session management, and token validation
> must fit within your latency budget.

---

## 5. Сколько это стоит? (TCO — Total Cost of Ownership)

TCO = **Infrastructure** + **Licensing** + **People** + **Security & Compliance** + **Opportunity cost**

| Компонент | Что включить |
|---|---|
| **Infrastructure** | Compute, storage, network, egress, CDN, multi-AZ/region. Cloud vs. on-prem vs. hybrid. Regional pricing (data residency). |
| **Licensing** | Oracle vs. PostgreSQL, commercial vs. OSS, per-core vs. per-GB. Hidden: support contracts, cloud egress fees. |
| **People / Ops** | # of SRE/DBA engineers, on-call burden, training, runbooks. A 3-node Cassandra cluster needs a different skill set than one Postgres. |
| **Security & Compliance** | WAF, SIEM, DLP, pen-testing, GDPR/HIPAA/152-ФЗ audits, encryption (at rest + in transit), secret management (Vault, KMS). |
| **Opportunity cost** | Time-to-market, vendor lock-in, migration cost if you outgrow the stack. |

### Примеры

- **10 k users:** Managed PostgreSQL (RDS/Aurora) + Redis + S3.
  TCO ≈ $200–500/mo + 0.5 FTE. Boring and correct.
- **100 M users, write-heavy:** Cassandra cluster (15+ nodes) + Kafka
  + Elasticsearch + Redis. TCO ≈ $50 k–200 k/mo + 5–8 FTE.
  A single Postgres *might* work if queries are simple and data
  fits in RAM, saving 90 % of cost.
- **Bank, strong consistency, RPO = 0:** CockroachDB / Spanner /
  PostgreSQL with synchronous replication across 3 AZs +
  geo-redundant backup. TCO is high, but a $1 M error is
  cheaper than a lost transaction.

> **OWASP A05 (Security Misconfiguration):** Default cloud
> configurations (public S3, open DB ports, no VPC peering
> restrictions) are a leading cause of breaches. Security
> hardening has a *cost* — include it in TCO.

---

## 6. Поперечные ограничения (не «шестой вопрос», а рамка)

Эти факторы пересекают все пять вопросов:

| Фактор | Влияние |
|---|---|
| **CAP / PACELC** | Partition tolerance is non-negotiable in distributed systems. You trade C vs. A. |
| **Data sovereignty / compliance** | GDPR, 152-ФЗ, HIPAA → data must reside in specific regions, encryption mandatory, right-to-be-forgotten complicates event sourcing. |
| **Multi-tenancy** | Row-level security, per-tenant quotas, noisy-neighbor isolation. |
| **Observability** | Metrics (Prometheus), structured logs (ELK/Loki), distributed tracing (OpenTelemetry). Without it, you can't debug production. |
| **Team expertise** | The best stack is the one your team can operate at 3 AM. «Boring technology» wins. |
| **OWASP Top 10 (2021)** | A01 Broken Access Control, A02 Cryptographic Failures, A03 Injection, A04 Insecure Design, A05 Misconfiguration, A06 Vulnerable Components, A07 Auth Failures, A08 Data Integrity, A09 Logging/Monitoring, A10 SSRF. Each maps to a design decision. |

---

## Матрица решений (примеры)

| Сценарий | Scale | R/W | Consistency | RPO / RTO | Latency (P99) | TCO | Рекомендация |
|---|---|---|---|---|---|---|---|
| Internal tool | < 1 k | Mixed | Strong (single-node) | 24 h / 4 h | 100 ms | Minimal | PostgreSQL + Nginx + S3 |
| SaaS CRUD | 1 M | 70/30 R/W | Strong (single-region) | 1 min / 15 min | 50 ms | Low–Med | PostgreSQL (Aurora) + Redis + CDN |
| Social feed | 50 M | 90/10 R/W | Eventual | 5 min / 1 h | 20 ms | Medium | Cassandra + Redis + Kafka + S3 |
| IoT telemetry | 100 M devices | 95/5 W | Eventual | 1 min / 30 min | 500 ms | Medium | Kafka + TimescaleDB / InfluxDB + S3 |
| Banking ledger | 10 M | 50/50 | Linearizable | 0 / 5 s | 10 ms | High | CockroachDB / PostgreSQL sync-repl (3 AZ) + HSM |
| BI / Analytics | 10 M rows | 99/1 R (batch) | Eventual | 1 h / 24 h | 5 s | Medium | Snowflake / BigQuery / ClickHouse |

---

## Анти-паттерны

1. **Choosing NoSQL solely because "it scales horizontally."**
   Verify: do you actually need > 1 TB per shard or > 100 k writes/s
   per partition? Often a sharded Postgres (Vitess / Citus) is simpler
   and cheaper.

2. **Ignoring P99 / P99.9.**
   Designing for P50 while your SLO is P99 < 100 ms guarantees
   production surprises. Tail latency kills user trust.

3. **Treating security as an afterthought.**
   Encryption, authn/authz, injection prevention, logging must be
   in the *design*, not bolted on. OWASP A04 (Insecure Design) is
   the #1 root cause of critical vulnerabilities.

4. **Underestimating TCO.**
   Cloud egress fees, licensing, on-call engineer cost, and
   compliance audits can 3–5× the "compute" line item.

5. **Single-AZ / single-region without a plan.**
   If RTO < 1 h, you need multi-AZ. If RTO < 15 min, you need
   multi-region with automated failover.

6. **Ignoring the team.**
   A 5-person team running a 15-node Cassandra + Kafka + ES cluster
   will drown in operational toil. Match complexity to headcount.

---

## Чек-лист перед выбором стека (7 вопросов)

- [ ] **Scale & growth:** How many concurrent users now and in 2 years?
      What's the peak-to-average ratio?
- [ ] **R/W ratio:** What's the read:write ratio? Is it stable or
      does it shift by time-of-day / event?
- [ ] **Durability & consistency:** What is my RPO? RTO? Do I need
      linearizable or is eventual acceptable? (PACELC: C or A on
      partition?)
- [ ] **Latency budget:** What is P99 (not P50)? Does security
      overhead (TLS, authn, WAF) fit within the budget?
- [ ] **TCO:** What's the 3-year total cost including people,
      licensing, security tooling, and egress?
- [ ] **Compliance & security:** Which regulations apply (GDPR,
      HIPAA, 152-ФЗ)? Which OWASP Top 10 risks are highest for
      this domain?
- [ ] **Team & ops:** Can my team operate this stack at 3 AM?
      What's the hiring difficulty? What's the migration cost if
      I outgrow it in 18 months?

---

## Заключение

Пять вопросов — это не «опросник», а **система взаимно
ограничивающих условий**. Ответ на каждый сужает пространство
допустимых архитектур. Ответы на все пять *одновременно*
определяют, что именно вы можете build, how fast, how reliably,
and how securely — and, critically, **what you must sacrifice**.

The best architecture is not the one with the flashiest technology.
It is the one where every constraint is explicitly acknowledged,
every trade-off is documented, and the team can operate the system
without heroic effort. Sometimes that means a single PostgreSQL
instance behind Nginx. That is not a failure of ambition — it is
a correct engineering answer.
```
