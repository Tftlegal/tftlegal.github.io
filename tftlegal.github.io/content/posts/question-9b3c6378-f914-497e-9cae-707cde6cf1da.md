---
title: "PostgreSQL для чайников?"
summary: "PostgreSQL для чайников: полное руководство от установки до тюнинга"
date: 2026-07-23T20:24:39Z
tldr: "933820173"
---

# PostgreSQL для чайников: полное руководство от установки до тюнинга

Вы только начали работать с PostgreSQL и уже столкнулись с тем, что «из коробки» он работает медленно? Или не понимаете, как управлять правами, переносить базу на другой диск или разбираться с блокировками? Эта статья — ваш спасительный круг. Мы пройдём путь от базовых настроек до глубокого понимания внутреннего устройства. Всё с примерами SQL-команд, которые можно сразу скопировать и выполнить.

---

## 1. Почему PostgreSQL нужно настраивать?

При первой установке PostgreSQL использует **консервативные настройки**, рассчитанные на запуск даже на самом слабом железе. Например, `shared_buffers` по умолчанию — всего **128 МБ**, а `work_mem` — **4 МБ**. На современном сервере с 32 ГБ ОЗУ и SSD-дисками такие параметры — как тормоз на «Феррари». Поэтому первое, что нужно сделать после установки, — понять тип вашей нагрузки.

### OLTP vs OLAP – определите свою задачу

| Характеристика | OLTP (Online Transaction Processing) | OLAP (Online Analytical Processing) |
|----------------|--------------------------------------|--------------------------------------|
| Что делаем | Много мелких операций (INSERT, UPDATE) | Мало, но очень тяжёлых SELECT-запросов |
| Пример | Интернет-магазин, банковские транзакции | Отчёты за год, анализ продаж |
| Важно | Скорость каждой транзакции — миллисекунды | Скорость одного запроса может быть минуты |
| Пользователей | Много одновременных подключений | Немного аналитиков |

От типа нагрузки зависят все дальнейшие настройки памяти и параллелизма.

---

## 2. Базовые параметры производительности

Для расчёта оптимальных значений удобно использовать онлайн-конфигураторы:  
[PGTune](https://pgtune.leopard.in.ua/), [pgconfigurator](https://pgconfigurator.cybertec.at/). Они сами подберут цифры под ваше железо, но полезно понимать, что они означают.

### 2.1. Настройки CPU (параллелизм)

PostgreSQL умеет выполнять один запрос в несколько потоков. Иерархия параметров:

```
max_worker_processes (общий пул процессов)
 └── max_parallel_workers (сколько из пула можно отдать под параллельные операции)
      ├── max_parallel_workers_per_gather (на один запрос)
      └── max_parallel_maintenance_workers (на одну служебную команду, например, CREATE INDEX)
```

- **max_worker_processes** = число ядер CPU (или +20% для служебных задач)  
- **max_parallel_workers** = max_worker_processes  
- **max_parallel_workers_per_gather** = ядра / 2, но не более 4‑6 для OLTP  
- **max_parallel_maintenance_workers** = 4‑6 (больше — редко даёт прирост)

### 2.2. Память: общая (Shared Memory)

- **shared_buffers** — главный кеш PostgreSQL. Ставьте **25–40% от ОЗУ**. Для 32 ГБ → 8–10 ГБ.
- **effective_cache_size** — подсказка планировщику о размере всего кеша (PostgreSQL + ОС). Ставьте **50–75% ОЗУ**. Для 32 ГБ → 20 ГБ.

Важно: `shared_buffers + effective_cache_size` должно быть **меньше** общего ОЗУ, оставляйте запас для ОС.

### 2.3. Память: на каждое подключение (Backend Memory)

Каждое новое соединение — это отдельный процесс, который потребляет память. Поэтому PostgreSQL не любит тысячи подключений.

- **work_mem** — память для сортировок и хеш-соединений в рамках **одной операции**.  
  - OLTP: **4–16 МБ** (много подключений → осторожно)  
  - OLAP: **64–256 МБ** (мало подключений → можно больше)  

Осторожно: если при 100 подключениях и двух сортировках выставить `work_mem = 256 МБ`, то потенциальное потребление = 100 × 2 × 256 МБ = **51 ГБ** → OOM Killer.

- **maintenance_work_mem** — для служебных операций (VACUUM, CREATE INDEX). Выделяется на один процесс обслуживания. Ставьте **1–4 ГБ** для больших таблиц. Это одна из самых эффективных настроек.

- **temp_buffers** — для временных таблиц. Обычно оставляют 8 МБ.

### 2.4. Дисковая подсистема

- **random_page_cost** — отношение стоимости случайного чтения к последовательному. Для SSD снижайте до **1.1–1.5** (по умолчанию 4.0 — наследие HDD).
- **effective_io_concurrency** — число параллельных операций I/O. Для SSD ставьте **100–200**.
- **fsync = on** — обязательно в production, защищает от потери данных.
- **synchronous_commit** — можно выключить для ускорения, но с риском потери транзакций при сбое.

---

## 3. Физический уровень: где живут данные и как их перенести

Данные PostgreSQL хранятся в каталоге, который называется `data_directory`. Обычно это `/var/lib/postgresql/17/main`. Но иногда нужно перенести этот каталог на отдельный диск, чтобы освободить системный раздел.

### 3.1. Проверяем состояние кластера

```bash
sudo -u postgres pg_lsclusters
```

Вы увидите список кластеров, их версии, порты и состояние.

### 3.2. Создаём новый диск в виртуальной машине

Если вы в VirtualBox, добавьте новый виртуальный диск на 10 ГБ. После перезапуска ВМ:

```bash
lsblk               # смотрим список блочных устройств
sudo parted /dev/sdb mklabel gpt
sudo parted /dev/sdb mkpart primary ext4 0% 100%
sudo mkfs.ext4 /dev/sdb1
sudo mkdir -p /mnt/data
sudo mount /dev/sdb1 /mnt/data
```

Чтобы диск монтировался автоматически после перезагрузки, узнайте его UUID:

```bash
sudo blkid /dev/sdb1
# скопируйте UUID
sudo nano /etc/fstab
# добавьте строку:
UUID=ваш-uuid /mnt/data ext4 defaults 0 2
```

Проверьте монтирование:

```bash
df -h
```

### 3.3. Перенос каталога данных

Остановите кластер:

```bash
sudo -u postgres pg_ctlcluster 17 main stop
```

Сделайте пользователя `postgres` владельцем нового каталога и перенесите данные:

```bash
sudo chown -R postgres:postgres /mnt/data
sudo mv /var/lib/postgresql/17 /mnt/data
```

Теперь укажите PostgreSQL новый путь. Отредактируйте `postgresql.conf`:

```bash
sudo nano /etc/postgresql/17/main/postgresql.conf
# Найдите и измените параметр:
data_directory = '/mnt/data/17/main'
```

Запустите кластер:

```bash
sudo -u postgres pg_ctlcluster 17 main start
```

Если всё сделано правильно, база запустится с новым диском.

---

## 4. Управление базами данных, схемами и правами доступа

PostgreSQL — это не только таблицы, но и слои: кластер → база данных → схема → таблица. По умолчанию все создаются в схеме `public`, но для порядка лучше создавать свои схемы.

### 4.1. Создаём базу и схему

```sql
CREATE DATABASE testdb;
\c testdb
CREATE SCHEMA testnm;
CREATE TABLE testnm.t1 (c1 integer);
INSERT INTO testnm.t1 VALUES (1);
```

### 4.2. Создаём роль и пользователя

Роль — это группа прав, пользователь — это учётная запись с паролем.

```sql
CREATE ROLE readonly;   -- роль только для чтения
CREATE USER testread WITH PASSWORD 'test123';
GRANT readonly TO testread;   -- назначаем роль пользователю
```

Выдаём права на базу и схему:

```sql
GRANT CONNECT ON DATABASE testdb TO readonly;
GRANT USAGE ON SCHEMA testnm TO readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
```

### 4.3. Настройка pg_hba.conf для доступа

Если при подключении возникает ошибка аутентификации, отредактируйте файл `/etc/postgresql/17/main/pg_hba.conf`. Добавьте строку для локальных подключений с паролем:

```
local   all             all                                     md5
```

После этого перечитайте конфиг без перезапуска:

```bash
sudo systemctl reload postgresql
```

Теперь можно подключаться:

```bash
psql -h 127.0.0.1 -U testread -d testdb -W
```

### 4.4. Почему таблица не видна? (search_path и права)

Когда вы выполняете `SELECT * FROM t1` без указания схемы, PostgreSQL ищет таблицу в схемах из параметра `search_path`. По умолчанию это `"$user", public`. Если таблица создана в схеме `testnm`, её нужно либо указывать явно (`testnm.t1`), либо изменить `search_path`:

```sql
SET search_path TO testnm, public;
```

Но также важно: права на таблицы выдаются **на момент выполнения GRANT**. Если таблицу удалить и создать заново, права нужно выдавать повторно. Для автоматической выдачи прав на новые таблицы используйте `ALTER DEFAULT PRIVILEGES`:

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA testnm GRANT SELECT ON TABLES TO readonly;
```

Это будет действовать для будущих таблиц. Для существующих — повторите `GRANT SELECT ON ALL TABLES`.

### 4.5. Ошибка «нет доступа к схеме public» в PostgreSQL 15+

Начиная с версии 15, права на создание таблиц в схеме `public` отозваны у всех пользователей (кроме владельца). Если вы подключаетесь от обычного пользователя и пытаетесь создать таблицу без указания схемы, получите ошибку. Выход — либо создавать таблицы в своей схеме (`CREATE SCHEMA myschema; SET search_path TO myschema;`), либо дать права:

```sql
GRANT CREATE ON SCHEMA public TO testread;
```

Но лучше не использовать `public` для пользовательских таблиц.

---

## 5. MVCC, VACUUM и AUTOVACUUM: борьба с «мусором»

PostgreSQL использует механизм **MVCC** (Multi-Version Concurrency Control). При каждом UPDATE или DELETE старая версия строки не удаляется, а помечается как «мёртвая». Новые версии добавляются. Со временем таблица раздувается, и производительность падает. Очистку выполняет **VACUUM** (и его автоматический вариант — **autovacuum**).

### 5.1. Создаём тестовую нагрузку с pgbench

```bash
pgbench -i postgres   # инициализация тестовых таблиц в БД postgres
pgbench -c8 -P 6 -T 60 -U postgres postgres   # запуск на 60 секунд
```

Смотрите на показатель TPS (транзакций в секунду) — чем выше, тем лучше.

### 5.2. Настройка autovacuum

Иногда после изменения параметров производительность может упасть из-за неверных значений. Например, слишком большой `max_wal_size` или `random_page_cost` для SSD. Для SSD рекомендуется:

```ini
random_page_cost = 1.1
effective_io_concurrency = 200
maintenance_work_mem = 1GB   # для VACUUM
```

### 5.3. Мониторинг мёртвых строк и размера таблицы

Создадим таблицу и намеренно создадим мёртвые строки:

```sql
CREATE TABLE test (id serial, data text);
INSERT INTO test (data) SELECT md5(random()::text) FROM generate_series(1, 1000000);

SELECT pg_size_pretty(pg_total_relation_size('test'));   -- размер таблицы
```

Обновим все строки 5 раз, добавляя символ:

```sql
UPDATE test SET data = data || 'x';   -- повторить 5 раз
```

Теперь посмотрим на мёртвые строки:

```sql
SELECT n_dead_tup, last_autovacuum, last_autoanalyze
FROM pg_stat_all_tables
WHERE relname = 'test';
```

Размер таблицы вырос, потому что старые версии не удалены.

### 5.4. Отключение autovacuum на таблице

```sql
ALTER TABLE test SET (autovacuum_enabled = false);
```

Теперь мёртвые строки не будут очищаться автоматически — размер будет расти до тех пор, пока не включите VACUUM вручную.

---

## 6. Журналы WAL и контрольные точки

PostgreSQL пишет все изменения сначала в журнал предзаписи (WAL). Периодически выполняется **контрольная точка (checkpoint)**, когда все грязные страницы сбрасываются на диск. Частота контрольных точек влияет на нагрузку на дисковую систему.

### 6.1. Настройка интервала контрольных точек

```sql
ALTER SYSTEM SET checkpoint_timeout = '30s';
ALTER SYSTEM SET log_min_messages = 'notice';   -- чтобы видеть чекпоинты в логе
SELECT pg_reload_conf();
```

### 6.2. Измеряем объём WAL за время нагрузки

Запомним текущую позицию в WAL (LSN — Log Sequence Number):

```sql
SELECT pg_current_wal_lsn() AS start_lsn;
```

Запустим нагрузку pgbench на 10 минут:

```bash
pgbench -T 600 postgres
```

После нагрузки снова узнаем LSN:

```sql
SELECT pg_current_wal_lsn() AS end_lsn;
```

Рассчитаем объём сгенерированных WAL-байт:

```sql
SELECT pg_wal_lsn_diff('0/186CF2D8', '0/96EE2B8') AS total_wal_bytes;
```

И средний объём на одну контрольную точку (за 600 секунд при checkpoint_timeout = 30 с будет около 20 точек):

```sql
SELECT
  pg_wal_lsn_diff('0/186CF2D8', '0/96EE2B8') / 20 AS avg_bytes_per_checkpoint;
```

### 6.3. Общая статистика контрольных точек

```sql
SELECT * FROM pg_stat_checkpointer \gx
```

Здесь вы увидите количество выполненных контрольных точек, время записи и синхронизации.

### 6.4. Тест с синхронным и асинхронным коммитом

`synchronous_commit = on` гарантирует запись на диск перед подтверждением транзакции, что снижает TPS. Выключите его для теста:

```sql
ALTER SYSTEM SET synchronous_commit = off;
SELECT pg_reload_conf();
```

Запустите pgbench ещё раз — TPS должен вырасти, но надёжность упадёт.

### 6.5. Повреждение данных и контрольная сумма (data checksums)

При создании нового кластера можно включить контрольные суммы страниц:

```bash
pg_createcluster 17 instance03 -- --data-checksums
```

Это позволит обнаружить повреждения. Создадим таблицу, найдём файл данных:

```sql
CREATE TABLE large_test AS SELECT generate_series(1,1000) as id, repeat(md5(random()::text), 10) as large_data;
SELECT pg_relation_filepath('large_test');   -- вернёт путь, например base/5/16411
```

Остановим кластер и повредим файл:

```bash
sudo pg_ctlcluster 17 instance03 stop
sudo dd if=/dev/zero of=/var/lib/postgresql/17/instance03/base/5/16411 bs=1 count=2 seek=500 conv=notrunc
sudo pg_ctlcluster 17 instance03 start
```

Теперь при чтении таблицы PostgreSQL выдаст ошибку контрольной суммы. Можно игнорировать ошибку и прочитать данные:

```sql
SET ignore_checksum_failure = on;
SELECT * FROM large_test;   -- будут прочитаны неповреждённые страницы
```

Но лучше восстановить данные из бэкапа.

---

## 7. Блокировки: как их обнаружить и разрешить

Блокировки — это нормально, но длительные ожидания могут убить производительность. Научимся их логировать и анализировать.

### 7.1. Включаем логирование блокировок

```sql
ALTER SYSTEM SET log_lock_waits = on;
ALTER SYSTEM SET deadlock_timeout = '200ms';   -- через 200 мс ожидания пишем в лог
SELECT pg_reload_conf();
```

### 7.2. Создаём тестовую таблицу

```sql
CREATE TABLE test_locks (id SERIAL PRIMARY KEY, name VARCHAR(50), value INTEGER);
INSERT INTO test_locks (name, value) VALUES ('record1', 100), ('record2', 200), ('record3', 300);
```

### 7.3. Имитация блокировки

**Сессия 1** (начало транзакции, апдейт без COMMIT):

```sql
BEGIN;
UPDATE test_locks SET value = value + 1 WHERE id = 1;
```

**Сессия 2** (пытается обновить ту же строку):

```sql
BEGIN;
UPDATE test_locks SET value = value + 10 WHERE id = 1;   -- повиснет в ожидании
```

**Сессия 3** (ещё один апдейт той же строки):

```sql
BEGIN;
UPDATE test_locks SET value = value + 100 WHERE id = 1;   -- тоже будет ждать
```

Теперь в **сессии 4** посмотрим, кто кого блокирует:

```sql
SELECT
  locktype,
  relation::regclass,
  transactionid,
  pid,
  mode,
  granted
FROM pg_locks
WHERE relation = 'test_locks'::regclass OR transactionid IS NOT NULL
ORDER BY pid, locktype;
```

Мы увидим цепочку ожиданий: транзакция 3 ждёт транзакцию 2, та ждёт транзакцию 1.

### 7.4. Взаимоблокировка (Deadlock)

Создадим цикл:

- Сессия 1 блокирует строку id=1
- Сессия 2 блокирует строку id=2
- Сессия 3 блокирует строку id=3

Затем:
- Сессия 3 пытается обновить id=1 (ждёт сессию 1)
- Сессия 1 пытается обновить id=2 (ждёт сессию 2)
- Сессия 2 пытается обновить id=3 (ждёт сессию 3)

PostgreSQL обнаружит взаимоблокировку и прервёт одну из транзакций. В логах появится сообщение «deadlock detected». Просмотреть логи:

```bash
grep 'deadlock' /var/log/postgresql/postgresql-*.log
```

### 7.5. Могут ли две транзакции с UPDATE без WHERE заблокировать друг друга?

Нет, потому что такая команда блокирует **всю таблицу** (RowExclusiveLock на уровне таблицы). Первая транзакция захватит блокировку, вторая будет ждать её завершения — deadlock не возникнет. Взаимоблокировка возможна только при разном порядке блокировки строк.

---

## 8. Полезные команды для мониторинга (шпаргалка)

| Что нужно | Команда |
|-----------|---------|
| Размер таблицы | `SELECT pg_size_pretty(pg_total_relation_size('table_name'));` |
| Размер базы данных | `SELECT pg_size_pretty(pg_database_size('dbname'));` |
| Мёртвые строки в таблице | `SELECT n_dead_tup, last_autovacuum FROM pg_stat_all_tables WHERE relname = 'table_name';` |
| Текущие активные запросы | `SELECT pid, usename, query, state, backend_start FROM pg_stat_activity WHERE state = 'active';` |
| Блокировки | `SELECT * FROM pg_locks WHERE NOT granted;` |
| Кто кого блокирует | `SELECT pid, usename, query, waiting FROM pg_stat_activity WHERE waiting = 't';` |
| Перезагрузка конфигурации | `SELECT pg_reload_conf();` |
| Сброс статистики | `SELECT pg_stat_reset();` |

---

## Заключение

PostgreSQL — мощная, но требовательная к настройке СУБД. Этот тернистый  путь от выбора параметров под вашу нагрузку до управления физическим размещением данных, правами доступа, очисткой мёртвых строк, мониторингом журналов и блокировок подсилу не каждому. 
Всегда помните:

1. **Начинайте с конфигураторов** (PGTune), но всегда проверяйте их рекомендации на реальных тестах.
2. **Используйте pgbench** для проверки изменений перед продакшеном.
3. **Не отключайте autovacuum**, но настраивайте его под свои таблицы.
4. **Всегда ведите мониторинг** — без данных вы слепы.
5. **Права доступа** — это не только безопасность, но и правильная работа приложений.

Экспериментируйте, анализируйте логи и не бойтесь копаться в `postgresql.conf`. С этой статьёй у вас теперь есть фундамент, чтобы стать уверенным пользователем PostgreSQL. Удачи!
