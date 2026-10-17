---
title: "Алерты Zabbix в Telegram из России: контейнер в Нидерландах"
date: 2026-10-12T21:33:36Z
source: "serveradminru"
original_url: "https://serveradmin.ru/alerty-zabbix-v-telegram-iz-rossii-kontejner-v-niderlandah/"
summary: "Zabbix по умолчанию отправляет уведомления в Telegram напрямую с сервера мониторинга. Из России адрес api.telegram.org недоступен: запросы зависают или обрываются, что приводит к errors in the delivery log. Setting up a proxy or VPN on the monitoring server just for one bot is impractical, as it creates an extra infrastructure layer that requires ongoing maintenance. The article proposes an alternative solution: running a container in the Netherlands to relay Telegram alerts. This approach avoids the need for a full proxy/VPN setup on the monitoring host. The solution was published on the Server Admin blog."
---

# Алерты Zabbix в Telegram из России: контейнер в Нидерландах

## Краткое содержание

Zabbix по умолчанию отправляет уведомления в Telegram напрямую с сервера мониторинга. Из России адрес api.telegram.org недоступен: запросы зависают или обрываются, что приводит к errors in the delivery log. Setting up a proxy or VPN on the monitoring server just for one bot is impractical, as it creates an extra infrastructure layer that requires ongoing maintenance. The article proposes an alternative solution: running a container in the Netherlands to relay Telegram alerts. This approach avoids the need for a full proxy/VPN setup on the monitoring host. The solution was published on the Server Admin blog.

## Полная статья

Штатный способ оповещений Zabbix ходит в*api.telegram.org*с того сервера, где стоит мониторинг. С хоста в России этот адрес не открывается: запрос зависает или обрывается, в журнал действий падает ошибка доставки. Прокси и VPN на сервере мониторинга ради одного бота тащат за собой отдельный контур, который ещё и надо сопровождать.

**Углубленный онлайн-курс по MikroTik**

Научиться настраивать MikroTik с нуля или систематизировать уже имеющиеся знания можно[на углубленном онлайн-курcе по администрированию MikroTik](https://курсы-по-ит.рф/mikrotik-mtcna?utm_source=serveradminru&utm_medium=cpc&utm_campaign=mikrotik&erid=2SDnjdSUXCF). Автор курcа – сертифицированный тренер MikroTik Дмитрий Скоромнов. Более 40 лабораторных работ по которым дается обратная связь. В три раза больше информации, чем в MTCNA.
Реклама ООО "Курсы по ИТ" ИНН 5029266531

Ниже Zabbix из России шлёт обычный HTTPS на ваш домен, а сообщение в Telegram уходит уже из контейнера в Нидерландах, где API Telegram доступно. Отдельный VPS в Европе не нужен: репозиторий деплоится на[Bothost](https://bothost.ru/), нода в NL, домен и сертификат появляются вместе с контейнером.

Путь такой:

```
Zabbix (Россия) --HTTPS--> имя.bothost.tech (Нидерланды) --api.telegram.org--> Telegram
```

До сервера мониторинга*api.telegram.org*не требуется. Ему нужен только исходящий 443 до**.bothost.tech*. Токен бота тоже остаётся в контейнере в NL, на российском хосте его нет.

Подойдёт Zabbix 6.0 и новее. На выходе: проблема приходит в Telegram красным, восстановление — зелёным, исходник бота лежит в Git.

## Что будет в репозитории

Два файла. Сторонних библиотек нет, хватает стандартной библиотеки Python.

*main.py*:

```
#!/usr/bin/env python3
import json
import os
import urllib.request
from http.server import BaseHTTPRequestHandler, ThreadingHTTPServer

TOKEN = os.environ["BOT_TOKEN"]
CHAT_ID = os.environ["TELEGRAM_CHAT_ID"]
SECRET = os.environ["ZABBIX_WEBHOOK_SECRET"]
PORT = int(os.environ.get("PORT", "3000"))

def esc(value) -> str:
    return (
        str(value)
        .replace("&", "&")
        .replace("<", "<") .replace(">", ">")
    )

def send_telegram(text: str) -> None:
    url = f"https://api.telegram.org/bot{TOKEN}/sendMessage"
    data = json.dumps({
        "chat_id": CHAT_ID,
        "text": text,
        "parse_mode": "HTML",
        "disable_web_page_preview": True,
    }).encode()
    req = urllib.request.Request(
        url, data=data, headers={"Content-Type": "application/json"}
    )
    with urllib.request.urlopen(req, timeout=15) as resp:
        resp.read()

class Handler(BaseHTTPRequestHandler):
    def _send(self, code: int, body: str = "ok") -> None:
        raw = body.encode()
        self.send_response(code)
        self.send_header("Content-Type", "text/plain; charset=utf-8")
        self.send_header("Content-Length", str(len(raw)))
        self.end_headers()
        self.wfile.write(raw)

    def do_GET(self):
        path = self.path.split("?", 1)[0]
        if path == "/health":
            self._send(200, "ok")
        else:
            self._send(404, "not found")

    def do_POST(self):
        path = self.path.split("?", 1)[0]
        if path != "/zabbix":
            self._send(404, "not found")
            return
        if self.headers.get("X-Webhook-Secret", "") != SECRET:
            self._send(403, "forbidden")
            return

        length = int(self.headers.get("Content-Length", "0"))
        raw = self.rfile.read(length)
        try:
            payload = json.loads(raw.decode() or "{}")
        except json.JSONDecodeError:
            self._send(400, "bad json")
            return

        status = str(payload.get("status", "PROBLEM")).upper()
        mark = "🔴" if status == "PROBLEM" else "🟢"
        lines = [
            f"{mark} {esc(payload.get('status', ''))} {esc(payload.get('severity', ''))}",
            f"{esc(payload.get('name', ''))}",
            f"Хост: {esc(payload.get('host', ''))}",
        ]
        opdata = payload.get("opdata") or ""
        if opdata:
            lines.append(esc(opdata))
        try:
            send_telegram("\n".join(lines))
        except Exception as exc:
            print("telegram error:", exc)
            self._send(502, "telegram error")
            return
        self._send(200, "ok")

    def log_message(self, fmt, *args):
        print("%s - %s" % (self.address_string(), fmt % args))

if __name__ == "__main__":
    server = ThreadingHTTPServer(("0.0.0.0", PORT), Handler)
    print(f"listening on 0.0.0.0:{PORT}")
    server.serve_forever()
```

*requirements.txt*оставьте пустым. Секретов в репозитории нет: токен бота, id чата и пароль вебхука придут переменными окружения.

Сервис слушает*0.0.0.0*и порт из*PORT*. Именно так до него доходит внешний HTTPS.
*GET /health*нужен, чтобы увидеть, что контейнер жив.*POST /zabbix*принимает алерт и отвечает сразу, до повторных попыток Zabbix.

## Бот в Telegram

В[@BotFather](https://t.me/BotFather)создайте бота командой /newbot и сохраните токен. Токен в Git не кладётся.

[

![BotFather: создать нового бота](/news/serveradminru/article-e819df1dfed3d080/image-01.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/botfather-menu.png)

[

![BotFather: имя и username](/news/serveradminru/article-e819df1dfed3d080/image-02.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/botfather-username.png)

[

![BotFather: токен после создания](/news/serveradminru/article-e819df1dfed3d080/image-03.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/botfather-token.png)

Напишите боту*/start*со своего аккаунта. Пока этого сообщения нет, Bot API не отдаст id чата, и бот не сможет писать вам первым.

## Деплой на Bothost

- Залейте репозиторий на GitHub или GitLab.
- арегистрируйтесь на[bothost.ru](https://bothost.ru/)и откройте создание бота.
- Укажите название, платформу Telegram, ссылку на репозиторий и ветку*main*.
- Вставьте токен от BotFather. Платформа положит его в переменную*BOT_TOKEN*.
- Включите веб-интерфейс, чтобы выдался домен вида*https://имя.bothost.tech*и сертификат. Порт в панели оставьте*3000:*приложение читает его из*PORT*.
- Добавьте переменные окружения:
*TELEGRAM_CHAT_ID*— id чата или группы, куда слать алерты;
*ZABBIX_WEBHOOK_SECRET*— длинная случайная строка, её же укажете в Zabbix.

[

![Bothost: название, платформа и токен](/news/serveradminru/article-e819df1dfed3d080/image-04.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/bothost-create.png)[

![нода в Нидерландах, репозиторий и ветка main](/news/serveradminru/article-e819df1dfed3d080/image-05.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/bothost-repo.png)

[

![домен и порт 3000](/news/serveradminru/article-e819df1dfed3d080/image-06.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/bothost-web.png)[

![переменные окружения](/news/serveradminru/article-e819df1dfed3d080/image-07.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/bothost-env.png)Дождитесь статуса контейнера Up и откройте*https://имя.bothost.tech/health*. Ответ ok значит, что маршрут и процесс на месте.

[

![Bothost: бот в статусе «Работает»](/news/serveradminru/article-e819df1dfed3d080/image-08.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/bothost-dashboard.png)

[

![Браузер: /health отвечает ok](/news/serveradminru/article-e819df1dfed3d080/image-09.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/health-ok.png)

Id чата можно снять до деплоя. Откройте в браузере (токен подставьте свой):

```
https://api.telegram.org/bot<ТОКЕН>/getUpdates
```

После*/start*в ответе будет*"chat":{"id":123456789}*. Для группы id отрицательный. Это значение и есть*TELEGRAM_CHAT_ID*.

Для личного чата id можно снять и через стороннего бота: перешлите ему любое своё сообщение, он ответит числом. В личном чате с вашим ботом это число совпадает с id чата. Токен своего бота ему не отправляйте.

[

![Сторонний бот показывает id пользователя](/news/serveradminru/article-e819df1dfed3d080/image-10.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/telegram-user-id.png)

Проверка приёмника с любой машины, где есть выход в интернет:

```
curl -sS -o /dev/null -w "%{http_code}\n" \
  -H "Content-Type: application/json" \
  -H "X-Webhook-Secret: ВАШ_СЕКРЕТ" \
  -d '{"status":"PROBLEM","severity":"High","name":"Тест с curl","host":"zabbix","opdata":"ping"}' \
  https://имя.bothost.tech/zabbix
```

Код*200*и сообщение в Telegram.*403*— секрет в заголовке не совпал с переменной.*502*— контейнер жив, но Telegram не принял сообщение: проверьте токен и что вы нажали*/start*.

## Медиатип в Zabbix

Есть два пути: импортировать готовый файл или собрать медиатип руками.

**Быстрый путь.**Администрирование → Способы оповещений → Импорт, файл*media_type_telegram_bothost.yaml*. После импорта откройте медиатип и замените в параметрах*url*на свой*https://имя.bothost.tech/zabbix*и*secret*на свою строку. Шаблоны сообщений и скрипт уже внутри.

**Вручную.**Администрирование → Способы оповещений → Создать → тип Webhook. Имя, например,*Telegram via Bothost*.

В списке видно штатный*Telegram*и наш. Штатный ходит в API напрямую, наш идёт через контейнер.

[

![Список способов оповещений в Zabbix](/news/serveradminru/article-e819df1dfed3d080/image-11.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/01-mediatype-list.png)

Параметры медиатипа:

| Параметр | Значение |
| --- | --- |
| url | https://имя.bothost.tech/zabbix |
| secret | та же строка, что в ZABBIX_WEBHOOK_SECRET |
| status | {EVENT.STATUS} |
| severity | {EVENT.SEVERITY} |
| name | {EVENT.NAME} |
| host | {HOST.NAME} |
| opdata | {EVENT.OPDATA} |

[

![Параметры медиатипа](/news/serveradminru/article-e819df1dfed3d080/image-12.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/02-mediatype-params.png)В поле «Скрипт» вставьте код. Тайм-аут оставьте*30s*.

```
try {
    var params = JSON.parse(value),
        req = new HttpRequest(),
        resp;

    req.addHeader('Content-Type: application/json');
    req.addHeader('X-Webhook-Secret: ' + params.secret);

    resp = req.post(params.url, JSON.stringify({
        status: params.status,
        severity: params.severity,
        name: params.name,
        host: params.host,
        opdata: params.opdata
    }));

    if (req.getStatus() != 200) {
        throw 'Response code: ' + req.getStatus() + ' body: ' + resp;
    }

    return 'OK';
}
catch (error) {
    Zabbix.log(4, '[ Bothost telegram ] ' + error);
    throw 'Failed: ' + error;
}
```

[

![Скрипт медиатипа](/news/serveradminru/article-e819df1dfed3d080/image-13.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/03-mediatype-script.png)

Строка*return 'OK'*; обязательна. Без неё Zabbix считает отправку неудачной (*undefined return value*) и повторяет её, хотя бот уже принял сообщение. В Telegram каждое оповещение приходит по три раза.

*{EVENT.STATUS}*в Zabbix 6+ принимает значения*PROBLEM*и*RESOLVED*. По ним бот выбирает цвет сообщения.

**Шаблоны сообщений.**Откройте вкладку «Шаблоны сообщений» и добавьте два шаблона: «Проблема» и «Восстановление проблемы». Текст может быть любым, скрипт его не использует, бот строит сообщение из параметров. Но пустые шаблоны нельзя: оповещение упадёт с ошибкой*No message defined for media type*, и скрипт даже не будет вызван.

Подойдёт такой текст.

Для «Проблема»:

```
Тема: Problem: {EVENT.NAME}
Сообщение:
Problem started at {EVENT.TIME} on {EVENT.DATE}
Host: {HOST.NAME}
Severity: {EVENT.SEVERITY}
```

Для «Восстановление проблемы»:

```
Тема: Resolved: {EVENT.NAME}
Сообщение:
Problem has been resolved at {EVENT.RECOVERY.TIME} on {EVENT.RECOVERY.DATE}
Host: {HOST.NAME}
```

[

![Шаблоны сообщений в Zabbix](/news/serveradminru/article-e819df1dfed3d080/image-14.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/04-mediatype-templates.png)Сохраните медиатип.

## Проверка медиатипа

Кнопка «Тест» находится не в форме медиатипа, а в списке: Администрирование → Способы оповещений, колонка «Действие» в строке вашего медиатипа.

В окне подставьте живые значения вместо макросов:*host*—*web-01*,*name*— любое название,*opdata*— любой текст,*severity*—*High*,*status*—*PROBLEM*. Поля*url*и*secret*уже заполнены из медиатипа. Если оставить*{EVENT.NAME}*как есть, в Telegram уйдёт сам макрос.

Успешный тест показывает «Успешное тестирование способа оповещения», в поле «Ответ» стоит*OK*, а сообщение приходит в тот же чат, что и от*curl*.

[

![Тест медиатипа](/news/serveradminru/article-e819df1dfed3d080/image-15.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/05-mediatype-test.png)С российского Zabbix проверяется только доступ до*имя.bothost.tech*. Доступ до*api.telegram.org*с этого сервера можно не настраивать: его вызывает контейнер в Нидерландах. Если с хоста мониторинга закрыт исходящий HTTPS целиком, вебхук не уйдёт, хотя контейнер при этом здоров.

## Способ оповещения у пользователя

Администрирование → Пользователи → нужный пользователь → вкладка «Оповещения» → Добавить. Выберите ваш медиатип. В поле «Отправлять на» можно написать любое слово, например telegram: для webhook оно не используется, но не должно быть пустым. Получатель без способа оповещения действие не получит.

[

![Способ оповещения у пользователя](/news/serveradminru/article-e819df1dfed3d080/image-16.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/06-user-media.png)

## Действие на триггеры

Настройка → Действия → Действия триггеров → Создать действие.

Условие оставьте широким, например «Важность триггеров больше или равно Предупреждение».

[

![Действие триггера: условия](/news/serveradminru/article-e819df1dfed3d080/image-17.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/07-action-conditions.png)

На вкладке «Операции» добавьте отправку через ваш медиатип нужному пользователю в блоках «Операции» и «Операции восстановления». Без второго блока зелёных сообщений не будет.

[

![Действие триггера: операции](/news/serveradminru/article-e819df1dfed3d080/image-18.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/08-action-operations.png)

Сгенерируйте тестовую проблему на любом узле. В «Отчёты → Журнал действий» статус отправки должен быть «Отправлено», и для проблемы, и для восстановления.

[

![Журнал действий в Zabbix](/news/serveradminru/article-e819df1dfed3d080/image-19.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/09-action-log.png)

В Telegram проблема приходит красной, восстановление зелёной.

Повторная доставка с тем же текстом возможна: бот не хранит id событий и каждый POST превращает в новое сообщение.

## Если не доходит

- *health*не открывается. Контейнер не в статусе Up, либо порт в панели не совпадает с*PORT*, либо процесс слушает*127.0.0.1*. В логах контейнера на Bothost должна быть строка*listening on 0.0.0.0:3000*. Если переменную не задали до старта, в логах повторяется*KeyError*.

[

![Логи: нет переменной TELEGRAM_CHAT_ID](/news/serveradminru/article-e819df1dfed3d080/image-20.png)

](https://serveradmin.ru/wp-content/uploads/2026/10/bothost-logs-keyerror.png)

- *curl*даёт 403. Секрет в заголовке и в переменной окружения различаются, в том числе пробелом в конце.
- *curl*даёт 200, Telegram молчит. Чат id от другого аккаунта, бот не добавлен в группу или у группы включена история, в которую бот не попал. Напишите боту*/start*ещё раз и обновите*TELEGRAM_CHAT_ID*.
- В журнале действий ошибка*No message defined for media type*. У медиатипа пустые шаблоны сообщений, добавьте их на вкладке «Шаблоны сообщений».
- В журнале ошибка*undefined return value*, а в Telegram приходит по три одинаковых сообщения. В конце скрипта нет*return 'OK'*;.
- Тест медиатипа в Zabbix падает, в тексте ошибки таймаут до*api.telegram.org*. Это штатный медиатип Telegram: он ходит в API напрямую с сервера мониторинга. Для российского хоста нужен медиатип из этой статьи, с URL на**.bothost.tech*.
- Тест медиатипа падает с кодом не 200 по адресу Bothost. Смотрите текст ошибки в окне теста и лог*zabbix_server*. Частая причина — сервер мониторинга не резолвит*bothost.tech*или не пускает наружу 443.
- *curl*с российского сервера до*https://api.telegram.or*g не проходит, а до*https://имя.bothost.tech/health*открывается. Так и должно быть: Telegram API с России недоступно, домен контейнера в Нидерландах доступен.
- В Telegram приходит сырой JSON или пустое имя. В действии не подставлены макросы из таблицы, и в POST уходит пустой*name*.

Дальше тот же приёмник можно кормить не только из Zabbix. Любой сервис, который умеет POST JSON с заголовком*X-Webhook-Secret*, попадёт в тот же чат.

**Онлайн-курс по устройству компьютерных сетей.**

На углубленном курсе "[Архитектура современных компьютерных сетей](https://курсы-по-ит.рф/computer_networks?utm_source=serveradminru&utm_medium=cpc&utm_campaign=asks&erid=2SDnjdSUXCF)" вы с нуля научитесь работать с Wireshark и «под микроскопом» изучите работу сетевых протоколов. На протяжении курса надо будет выполнить более пятидесяти лабораторных работ в Wireshark.
Реклама ООО "Курсы по ИТ" ИНН 5029266531

## Оригинал

https://serveradmin.ru/alerty-zabbix-v-telegram-iz-rossii-kontejner-v-niderlandah/
