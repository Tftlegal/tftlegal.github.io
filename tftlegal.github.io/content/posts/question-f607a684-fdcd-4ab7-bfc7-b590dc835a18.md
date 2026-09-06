---
title: "Как корректно установить  client_body_buffer_size и client_body_single_buffer на nginx где наблюдаетются повторяющиеся «тяжёлые» POST/PUT запросы, скачивание крупных JSON-документтов и очень часто, каждый запрос /ai/models/v1/message/async/v1  > буфера (судя по частоте — батч/генератора запросов с большими телами)"
summary: "Для location `/ai/models/v1/message/async/v1` set `client_body_buffer_size` to at least the expected maximum body size (e.g. `10m`), enable `client_body_single_buffer on`, and set `client_max_body_size` no smaller than that buffer; also configure `client_body_temp_path` on a disk with enough free space for overflow.  Validate the configuration with `nginx -t`, reload nginx, then monitor memory, disk I/O, the error log, and upstream behavior, adjusting buffer size or concurrency if needed."
tags: ["ai-generated", "todo", "json", "post", "put", "как"]
date: 2026-09-06T17:32:42Z
tldr: "933820173"
---

**Решение проблемы: установка optimal `client_body_buffer_size` и `client_body_single_buffer` для nginx при работе с повторяющимися "тяжёлыми" POST/PUT запросами**

**Шаг 1: Определите реальный максимум/типовой размер тела для `/ai/models/v1/message/async/v1`**

Чтобы определить реальный максимум/типовой размер тела для `/ai/models/v1/message/async/v1`, мы можем проанализировать логовые данные или метрики приложения. Мы можем использовать инструменты, такие как Nagios, Prometheus или Grafana, чтобы собрать данные о размере тела для каждого запроса.

**Шаг 2: Выберите `client_body_buffer_size` не меньше ожидаемого тела**

После анализа логов мы можем определить ожидаемый максимальный размер тела для `/ai/models/v1/message/async/v1`. Например, если максимальный размер тела составляет 1 GB, мы можем выбрать `client_body_buffer_size` не меньше 10m, чтобы обеспечитьufficientный буфер для каждого запроса.

**Шаг 3: Включите одиночный буфер: `client_body_single_buffer on;`**

Чтобы избежать блокировки процесса nginx, мы можем включить одиночный буфер для каждого запроса:

```bash
location /ai/models/v1/message/async/v1 {
    client_body_buffer_size 10m;
    client_body_single_buffer on;
}
```

**Шаг 4: Установите `client_body_buffer_size` и убедитесь, что `client_max_body_size` не меньше этого значения**

Чтобы обеспечить безопасность, нам нужно убедиться, что `client_max_body_size` не меньше `client_body_buffer_size`:

```bash
location /ai/models/v1/message/async/v1 {
    client_body_buffer_size 10m;
    client_max_body_size 10m;
}
```

**Шаг 5: Проверьте ограничения и пути для перелива тела: `client_body_temp_path`, диск space, лимиты**

Чтобы избежать ошибок, нам нужно убедиться, что у нас есть достаточно диск space и лимиты на диске:

```bash
client_body_temp_path /var/tmp;
```

**Шаг 6: Проверьте синтаксис: `nginx -t` и перезагрузите nginx**

Чтобы проверить синтаксис конфигурации nginx, мы можем использовать команду `nginx -t`:

```bash
nginx -t
```

Если синтаксис корректен, nginx не будет выдавать ошибки.

**Шаг 7: Наблюдайте за памятью, дисковым I/O, error log и upstream behavior; при необходимости скорректируйте размер или concurrency**

ЧтобыmonitorитьPerformance nginx, мы можем использовать инструменты, такие как Nagios, Prometheus или Grafana, чтобы отслеживать показатели performance, такие как:

*   Память: `netstat -an | grep 80`
*   Дисковое I/O: `top -n 10 | grep %cpu`
*   Error log: `error.log` на диске
*   Behavior upstream: `upstream` на диске

**Правильный конфигурный файл nginx:**

```bash
http {
    ...
    server {
        listen 80;
        server_name example.com;

        location /ai/models/v1/message/async/v1 {
            client_body_buffer_size 10m;
            client_body_single_buffer on;
            client_max_body_size 10m;
            client_body_temp_path /var/tmp;
        }
    }
}
```

**Наблюдение и корректировка:**

*   Наблюдайте за памятью, дисковым I/O, error log и upstream behavior.
*   Скорректируйте `client_body_buffer_size` или `client_max_body_size` если необходимо.
*   Увеличите конкуренцию, если необходимо.

**Безопасность:**

*   Убедитесь, что `client_max_body_size` не меньше `client_body_buffer_size`.
*   Убедитесь, что `client_body_temp_path` указывает на диск с sufficient space.
*   Убедитесь, что лимиты на диске не превышены.

**Машина-вершина:**

*   Убедитесь, что nginx выставляет лимиты на диске не более 70%.
*   Убедитесь, что nginx не превышает 80% usage CPU.
