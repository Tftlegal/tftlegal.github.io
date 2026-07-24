---
title: "Big Data analysis"
summary: "Базовые практики для работы с большими данными..."
date: 2026-07-23T20:09:58Z
tldr: "933820173"
---

🧠 Мульти-Агентный ответ:

# PostgreSQL для работы с большими данными: от JOIN до секционирования и триггеров

Что делать когда таблицы содержат миллионы или даже миллиарды строк, а запросы начинают выполняться минутами?
Правильно - идем к профильным специалистам DBA и Analitic. Но в случае если дорога окажется слишком долгой можно пробовать покапаться в них самостоятельно (главное не нарушать закон и корпоративную этику при этом). 
Ниже мы разберём все основные инструменты PostgreSQL для работы с большими объёмами данных:

- **JOIN-ы** — как правильно соединять таблицы, чтобы не убить производительность
- **Секционирование** — как разбить огромную таблицу на управляемые куски
- **Индексы** — какие индексы нужны для больших таблиц и как их создавать без простоя
- **Триггеры и витрины** — как поддерживать агрегированные данные в актуальном состоянии

---

## Часть 1. JOIN-ы: как правильно соединять таблицы

JOIN-ы — это основа реляционных баз данных. 
Левый джойн (LEFT JOIN) и правый джойн (RIGHT JOIN) отличаются тем, какая из таблиц сохраняется полностью, если между ними нет совпадений.LEFT JOIN сохраняет все строки из левой (первой) таблицы.RIGHT JOIN сохраняет все строки из правой (второй) таблицы.Главные особенностиЛевый джойн (LEFT JOIN):Выводит абсолютно все строки из левой таблицы.Из правой таблицы добавляет только те строки, которые подходят под условие. Если пары для левой строки не нашлось, вместо данных из правой таблицы подставляется NULL.
Правый джойн (RIGHT JOIN):Выводит абсолютно все строки из правой таблицы. Из левой таблицы добавляет только те строки, которые подходят под условие. 
Если пары для правой строки не нашлось, вместо данных из левой таблицы подставляется NULL. Эти два оператора полностью зеркальны друг другу. 
Любой RIGHT JOIN можно легко заменить на LEFT JOIN, если просто поменять таблицы местами в запросе.
На практике RIGHT JOIN используют редко, так как LEFT JOIN воспринимается привычнее и читается проще.
Но на больших объёмах данных неправильный JOIN может превратить запрос, который должен выполняться за секунды, в запрос на минуты (или даже часы).

### 1.1. Тестовые данные

Для примеров создадим две таблицы: `students` (ученики) и `grades` (оценки).

```sql
-- Таблица учеников
CREATE TABLE students (
    student_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    class VARCHAR(10)
);

-- Таблица оценок
CREATE TABLE grades (
    grade_id SERIAL PRIMARY KEY,
    student_id INTEGER REFERENCES students(student_id),
    subject VARCHAR(50),
    grade INTEGER,
    date_received DATE
);

-- Заполним тестовыми данными
INSERT INTO students (first_name, last_name, class) VALUES
    ('Иван', 'Петров', '10А'),
    ('Мария', 'Иванова', '10Б'),
    ('Петр', 'Сидоров', '11А');

INSERT INTO grades (student_id, subject, grade, date_received) VALUES
    (1, 'Математика', 5, '2025-09-10'),
    (1, 'Русский язык', 4, '2025-09-11'),
    (2, 'Математика', 3, '2025-09-10');
```

### 1.2. INNER JOIN — только совпадающие записи

`INNER JOIN` возвращает только те строки, для которых есть совпадение в обеих таблицах.

```sql
SELECT
    s.first_name,
    s.last_name,
    g.subject,
    g.grade
FROM students s
INNER JOIN grades g ON s.student_id = g.student_id;
```

**Результат**: увидят только учеников, у которых есть оценки (Иван и Мария). Пётр без оценок не попадёт в результат.

**Когда использовать**: когда вам нужны только данные, имеющие связи в обеих таблицах. Это самый распространённый тип JOIN.

### 1.3. LEFT JOIN — все записи из левой таблицы

`LEFT JOIN` возвращает все строки из левой таблицы, даже если нет совпадений в правой. Для отсутствующих значений подставляется `NULL`.

```sql
SELECT
    s.first_name,
    s.last_name,
    g.subject,
    g.grade
FROM students s
LEFT JOIN grades g ON s.student_id = g.student_id;
```

**Результат**: все ученики, включая Петра (у него `subject` и `grade` будут `NULL`).

**Когда использовать**: когда вам нужно увидеть все записи из основной таблицы, даже если нет связанных данных. Например, список всех учеников с их оценками (если есть).

### 1.4. RIGHT JOIN — все записи из правой таблицы

Работает как LEFT JOIN, но сохраняет все строки из правой таблицы.

```sql
SELECT
    s.first_name,
    s.last_name,
    g.subject,
    g.grade
FROM students s
RIGHT JOIN grades g ON s.student_id = g.student_id;
```

**Результат**: все оценки, даже если ученик был удалён (но в нашей базе таких нет, так как есть внешний ключ).

### 1.5. FULL JOIN — все записи из обеих таблиц

`FULL JOIN` объединяет LEFT и RIGHT JOIN — показывает все строки из обеих таблиц.

```sql
SELECT
    s.first_name,
    s.last_name,
    g.subject,
    g.grade
FROM students s
FULL JOIN grades g ON s.student_id = g.student_id;
```

**Когда использовать**: когда нужно увидеть полную картину — всех учеников и все оценки, даже если они не связаны.

### 1.6. CROSS JOIN — декартово произведение

`CROSS JOIN` создаёт все возможные комбинации строк из двух таблиц. На больших таблицах **опасен** — если в каждой таблице по миллиону строк, результат будет триллион строк.

```sql
SELECT
    s.first_name,
    s.last_name,
    g.subject,
    g.grade
FROM students s
CROSS JOIN grades g;
```

**Когда использовать**: практически никогда на больших данных. Только для генерации тестовых данных или очень специфических задач.

### 1.7. Продвинутая техника: CROSS JOIN + LEFT JOIN

Иногда нужно вывести все возможные комбинации учеников и предметов, а затем подставить реальные оценки (если они есть).

```sql
SELECT
    s.first_name,
    s.last_name,
    subj.subject,
    g.grade
FROM students s
CROSS JOIN (SELECT DISTINCT subject FROM grades) subj
LEFT JOIN grades g ON s.student_id = g.student_id AND subj.subject = g.subject;
```

Этот запрос покажет для каждого ученика все предметы, которые вообще есть в системе, и оценки по ним (или `NULL`, если оценки нет).

---

## Часть 2. Секционирование: разбиваем большие таблицы

Секционирование (partitioning) — это способ разбить логически одну большую таблицу на несколько физических частей. Преимущества:

- **Ускорение запросов** — планировщик может обращаться только к нужным секциям (partition pruning)
- **Упрощение обслуживания** — можно быстро удалить старые данные через `DROP TABLE` вместо `DELETE`
- **Перенос на разные носители** — старые данные можно хранить на медленных дисках

### 2.1. Выбор стратегии секционирования

Для таблицы `bookings` (бронирования авиабилетов) лучше всего подходит **секционирование по диапазону (RANGE)** по полю `book_date`, так как:
- Данные естественным образом группируются по времени
- Большинство запросов ищет недавние бронирования
- Старые данные можно архивировать

### 2.2. Создание секционированной таблицы

```sql
-- Создаём секционированную таблицу
CREATE TABLE bookings_partitioned (
    book_ref CHARACTER(6) NOT NULL,
    book_date TIMESTAMPTZ NOT NULL,
    total_amount NUMERIC(10,2) NOT NULL,
    PRIMARY KEY (book_ref, book_date)
) PARTITION BY RANGE (book_date);
```

**Важно**: первичный ключ должен включать столбец секционирования.

### 2.3. Создание секций

Создаём секции для трёх месяцев:

```sql
CREATE TABLE bookings_202509 PARTITION OF bookings_partitioned
    FOR VALUES FROM ('2025-09-01') TO ('2025-10-01');

CREATE TABLE bookings_202510 PARTITION OF bookings_partitioned
    FOR VALUES FROM ('2025-10-01') TO ('2025-11-01');

CREATE TABLE bookings_202511 PARTITION OF bookings_partitioned
    FOR VALUES FROM ('2025-11-01') TO ('2025-12-01');

-- DEFAULT-секция для всех остальных данных (запасной вариант)
CREATE TABLE bookings_default PARTITION OF bookings_partitioned DEFAULT;
```

### 2.4. Миграция данных и переименование

```sql
-- Переносим данные из старой таблицы
INSERT INTO bookings_partitioned SELECT * FROM bookings;

-- Проверяем распределение по секциям
SELECT
    tableoid::regclass AS partition_name,
    COUNT(*) AS row_count
FROM bookings_partitioned
GROUP BY partition_name
ORDER BY partition_name;

-- Переименовываем таблицы
ALTER TABLE bookings RENAME TO bookings_old;
ALTER TABLE bookings_partitioned RENAME TO bookings;
```

### 2.5. Тестирование производительности

**До секционирования:**

```sql
EXPLAIN ANALYZE
SELECT COUNT(*) FROM bookings_old
WHERE book_date BETWEEN '2025-10-01' AND '2025-10-07';
```

**После секционирования:**

```sql
EXPLAIN ANALYZE
SELECT COUNT(*) FROM bookings
WHERE book_date BETWEEN '2025-10-01' AND '2025-10-07';
```

Планировщик выполнит **partition pruning** — обратится только к секции `bookings_202510`. Ускорение может быть в **2–50 раз** в зависимости от объёма данных.

### 2.6. Агрегация по месяцам

**До секционирования:**

```sql
EXPLAIN ANALYZE
SELECT
    TO_CHAR(book_date, 'YYYY-MM') AS month,
    COUNT(*) AS bookings_count,
    AVG(total_amount) AS avg_amount
FROM bookings_old
GROUP BY TO_CHAR(book_date, 'YYYY-MM')
ORDER BY month;
```

**После секционирования:**

```sql
EXPLAIN ANALYZE
SELECT
    TO_CHAR(book_date, 'YYYY-MM') AS month,
    COUNT(*) AS bookings_count,
    AVG(total_amount) AS avg_amount
FROM bookings
GROUP BY TO_CHAR(book_date, 'YYYY-MM')
ORDER BY month;
```

### 2.7. Автоматическое создание секций

Для больших систем можно автоматизировать создание секций. Например, с помощью расширения `pg_partman` или через функцию, запускаемую по расписанию:

```sql
-- Функция для создания секции на следующий месяц
CREATE OR REPLACE FUNCTION create_next_month_partition()
RETURNS VOID AS $$
DECLARE
    next_month DATE;
    partition_name TEXT;
BEGIN
    next_month := DATE_TRUNC('month', NOW() + INTERVAL '1 month');
    partition_name := 'bookings_' || TO_CHAR(next_month, 'YYYYMM');
    
    EXECUTE format('
        CREATE TABLE IF NOT EXISTS %I PARTITION OF bookings
        FOR VALUES FROM (%L) TO (%L)',
        partition_name,
        next_month,
        next_month + INTERVAL '1 month'
    );
END;
$$ LANGUAGE plpgsql;
```

---

## Часть 3. Индексы для больших таблиц

Индексы — это главный инструмент ускорения SELECT-запросов. Но на больших таблицах создание индекса может занять часы и заблокировать запись.

### 3.1. Базовые команды для работы с индексами

```sql
-- Создание индекса (блокирует таблицу на запись)
CREATE INDEX idx_bookings_date ON bookings(book_date);

-- Создание индекса без блокировки (рекомендуется для production)
CREATE INDEX CONCURRENTLY idx_bookings_date ON bookings(book_date);

-- Удаление индекса
DROP INDEX idx_bookings_date;

-- Просмотр всех индексов таблицы
SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'bookings';

-- Размер индекса
SELECT pg_size_pretty(pg_relation_size('idx_bookings_date'));
```

### 3.2. Частичные индексы (Partial Indexes)

Частичный индекс содержит только строки, удовлетворяющие условию `WHERE`. Это экономит место и ускоряет запросы к часто используемым подмножествам данных.

```sql
-- Индекс только для дорогих бронирований (> 10000) в текущем месяце
CREATE INDEX CONCURRENTLY idx_bookings_202511_high_amount
ON bookings_202511 (total_amount)
WHERE total_amount > 10000;

-- Частичные индексы для каждой секции
CREATE INDEX idx_bookings_202509_very_high
ON bookings_202509 (book_date, total_amount)
WHERE total_amount > 50000;

CREATE INDEX idx_bookings_202510_very_high
ON bookings_202510 (book_date, total_amount)
WHERE total_amount > 50000;

CREATE INDEX idx_bookings_202511_very_high
ON bookings_202511 (book_date, total_amount)
WHERE total_amount > 50000;
```

### 3.3. Тестирование производительности

**До создания индексов:**

```sql
EXPLAIN ANALYZE
SELECT * FROM bookings_202511
WHERE total_amount > 10000
ORDER BY total_amount DESC
LIMIT 20;
```

**После создания индексов:**

```sql
EXPLAIN ANALYZE
SELECT * FROM bookings_202511
WHERE total_amount > 10000
ORDER BY total_amount DESC
LIMIT 20;
```

### 3.4. BRIN-индексы для очень больших таблиц

Для таблиц с естественно упорядоченными данными (например, по времени) **BRIN-индексы** (Block Range Indexes) занимают гораздо меньше места, чем B-tree, и хорошо работают на очень больших таблицах.

```sql
-- BRIN-индекс занимает килобайты вместо гигабайт для B-tree
CREATE INDEX idx_bookings_date_brin ON bookings USING BRIN(book_date);
```

### 3.5. Best practices для создания индексов на больших таблицах

1. **Всегда используйте `CONCURRENTLY`** в production, чтобы не блокировать запись
2. **Временно увеличьте `maintenance_work_mem`** перед созданием индекса
3. **Создавайте индексы в часы минимальной нагрузки**
4. **Мониторьте ресурсы** (CPU, память, диск) во время создания
5. **Тестируйте на staging-окружении** перед production
6. **Всегда делайте бэкап** перед массовыми изменениями схемы
7. **Используйте частичные индексы** для часто запрашиваемых подмножеств

Пример настройки перед созданием индекса:

```sql
-- Временно увеличиваем память для обслуживания
SET maintenance_work_mem = '2GB';

-- Создаём индекс без блокировки
CREATE INDEX CONCURRENTLY idx_bookings_total_amount ON bookings(total_amount);

-- Возвращаем значение по умолчанию
RESET maintenance_work_mem;
```

---

## Часть 4. Триггеры и поддержка витрин данных

Витрина данных (data mart) — это таблица с агрегированными данными, которая обновляется при изменении исходных данных. Триггеры позволяют поддерживать витрину в актуальном состоянии автоматически.

### 4.1. Создание витрины

Предположим, у нас есть таблицы `goods` (товары) и `sales` (продажи). Создадим витрину `good_sum_mart` с общей суммой продаж по каждому товару.

```sql
-- Таблица товаров
CREATE TABLE goods (
    goods_id SERIAL PRIMARY KEY,
    good_name VARCHAR(100),
    good_price NUMERIC(10,2)
);

-- Таблица продаж
CREATE TABLE sales (
    sales_id SERIAL PRIMARY KEY,
    good_id INTEGER REFERENCES goods(goods_id),
    sales_qty INTEGER,
    sale_date TIMESTAMP DEFAULT NOW()
);

-- Витрина (агрегированные данные)
CREATE TABLE good_sum_mart (
    good_name VARCHAR(100) PRIMARY KEY,
    sum_sale NUMERIC(10,2) DEFAULT 0
);
```

### 4.2. Первоначальное заполнение витрины

```sql
INSERT INTO good_sum_mart (good_name, sum_sale)
SELECT
    G.good_name,
    SUM(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.good_id = G.goods_id
GROUP BY G.good_name;
```

### 4.3. Создание триггерной функции

Функция должна обрабатывать три типа событий:

- **INSERT** — добавить сумму продажи к итогу по товару
- **DELETE** — вычесть сумму продажи из итога по товару
- **UPDATE** — вычесть старую сумму и прибавить новую

```sql
CREATE OR REPLACE FUNCTION update_good_sum_mart()
RETURNS TRIGGER AS $$
BEGIN
    -- Обработка DELETE и UPDATE: вычитаем старую запись
    IF TG_OP = 'DELETE' OR TG_OP = 'UPDATE' THEN
        UPDATE good_sum_mart
        SET sum_sale = sum_sale - (G.good_price * OLD.sales_qty)
        FROM goods G
        WHERE G.goods_id = OLD.good_id
          AND good_sum_mart.good_name = G.good_name;
    END IF;

    -- Обработка INSERT и UPDATE: прибавляем новую запись
    IF TG_OP = 'INSERT' OR TG_OP = 'UPDATE' THEN
        -- Если товара ещё нет в витрине — вставляем новую строку
        IF NOT EXISTS (
            SELECT 1 FROM good_sum_mart
            WHERE good_name = (SELECT good_name FROM goods WHERE goods_id = NEW.good_id)
        ) THEN
            INSERT INTO good_sum_mart (good_name, sum_sale)
            SELECT good_name, good_price * NEW.sales_qty
            FROM goods
            WHERE goods_id = NEW.good_id;
        ELSE
            -- Если есть — обновляем существующую
            UPDATE good_sum_mart
            SET sum_sale = sum_sale + (G.good_price * NEW.sales_qty)
            FROM goods G
            WHERE G.goods_id = NEW.good_id
              AND good_sum_mart.good_name = G.good_name;
        END IF;
    END IF;

    RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql;
```

### 4.4. Создание триггера

```sql
CREATE TRIGGER sales_mart_trigger
AFTER INSERT OR UPDATE OR DELETE ON sales
FOR EACH ROW
EXECUTE FUNCTION update_good_sum_mart();
```

### 4.5. Тестирование триггера

```sql
-- 1. Исходное состояние витрины
SELECT * FROM good_sum_mart;

-- 2. INSERT: добавляем новую продажу
INSERT INTO sales (good_id, sales_qty) VALUES (1, 10);
SELECT * FROM good_sum_mart;  -- сумма должна увеличиться

-- 3. UPDATE: изменяем количество (с 10 на 5)
UPDATE sales SET sales_qty = 5 WHERE sales_id = (SELECT MAX(sales_id) FROM sales);
SELECT * FROM good_sum_mart;  -- сумма должна уменьшиться на 5 * price

-- 4. DELETE: удаляем продажу
DELETE FROM sales WHERE sales_id = (SELECT MAX(sales_id) FROM sales);
SELECT * FROM good_sum_mart;  -- сумма должна вернуться к исходной
```

### 4.6. Важное замечание: цена товара в витрине

**Почему триггер — это правильно?**

Представьте: цена товара в таблице `goods` может измениться. Если бы мы каждый раз пересчитывали отчёт «на лету», то старые продажи посчитались бы по **новой** цене, что дало бы неверную сумму выручки.

Триггер же в момент совершения продажи фиксирует сумму **с текущей ценой** в витрине. Последующее изменение цены в `goods` никак не влияет на уже сохранённые исторические данные в `good_sum_mart`. Это гарантирует **корректность** отчётов.

---

## Часть 5. Дополнительные Best Practices для работы с большими данными

### 5.1. Статистика и планировщик

Для больших таблиц важно, чтобы у планировщика была актуальная статистика.

```sql
-- Увеличиваем объём собираемой статистики для конкретной таблицы
ALTER TABLE bookings SET STATISTICS 1000;

-- Собираем статистику вручную
ANALYZE bookings;

-- Просмотр статистики по колонке
SELECT attname, n_distinct, most_common_vals
FROM pg_stats
WHERE tablename = 'bookings';
```

### 5.2. Мониторинг размера таблиц и индексов

```sql
-- Размер всех таблиц в базе
SELECT
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS total_size
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Размер индексов для конкретной таблицы
SELECT
    indexname,
    pg_size_pretty(pg_relation_size(indexname)) AS index_size
FROM pg_indexes
WHERE tablename = 'bookings';
```

### 5.3. Поиск неиспользуемых индексов

Индексы ускоряют SELECT, но замедляют INSERT/UPDATE/DELETE. Если индекс не используется — его нужно удалить.

```sql
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
WHERE idx_scan = 0   -- индекс ни разу не использовался
ORDER BY idx_tup_read DESC;
```

### 5.4. Временные таблицы для сложных отчётов

Для сложных аналитических запросов иногда эффективнее использовать временные таблицы:

```sql
-- Создаём временную таблицу с промежуточными данными
CREATE TEMP TABLE temp_monthly_stats AS
SELECT
    DATE_TRUNC('month', book_date) AS month,
    COUNT(*) AS bookings_count
FROM bookings
GROUP BY DATE_TRUNC('month', book_date);

-- Используем временную таблицу в дальнейших расчётах
SELECT * FROM temp_monthly_stats
WHERE bookings_count > 1000;
```

### 5.5. Объединение мелких индексов в составные

Вместо двух отдельных индексов на `book_date` и `total_amount` лучше создать один составной:

```sql
-- Вместо:
CREATE INDEX idx_bookings_date ON bookings(book_date);
CREATE INDEX idx_bookings_amount ON bookings(total_amount);

-- Лучше (если запросы фильтруют по обоим полям):
CREATE INDEX idx_bookings_date_amount ON bookings(book_date, total_amount);
```

Составной индекс может использоваться для фильтрации по первому столбцу, по первым двум вместе и по всем трём.

### 5.6. Использование EXPLAIN для анализа запросов

Перед оптимизацией любого запроса всегда смотрите план выполнения:

```sql
-- Базовый план
EXPLAIN SELECT * FROM bookings WHERE book_date > '2025-10-01';

-- С фактическим выполнением (осторожно на больших таблицах!)
EXPLAIN (ANALYZE, BUFFERS, COSTS, TIMING)
SELECT * FROM bookings WHERE book_date > '2025-10-01';
```

Ключевые моменты в плане:
- **Seq Scan** — полное сканирование таблицы (плохо для больших таблиц)
- **Index Scan** — сканирование по индексу (хорошо)
- **Bitmap Heap Scan** — комбинированный доступ (тоже хорошо)
- **Nested Loop** — может быть проблемой при соединении больших таблиц

---

## Заключение

Работа с большими данными в требует комплексного подхода:

1. **Правильные JOIN-ы** — используйте INNER JOIN для связанных данных, LEFT JOIN для сохранения всех записей из основной таблицы, избегайте CROSS JOIN на больших данных.

2. **Секционирование** — разбивайте огромные таблицы по времени или другим естественным диапазонам. Это даёт ускорение в 2–50 раз и упрощает обслуживание.

3. **Индексы** — создавайте их через `CONCURRENTLY`, используйте частичные и BRIN-индексы для больших таблиц, удаляйте неиспользуемые.

4. **Триггеры и витрины** — поддерживайте агрегированные данные в актуальном состоянии, чтобы сложные отчёты выполнялись мгновенно.

5. **Мониторинг** — всегда знайте размер таблиц, статистику использования индексов и планы выполнения запросов.

Помните: оптимизация — это итеративный процесс. Начните с самых тяжёлых запросов, проанализируйте их планы, добавьте индексы и секционирование, проверьте результат. И никогда не забывайте про бэкапы перед массовыми изменениями!

