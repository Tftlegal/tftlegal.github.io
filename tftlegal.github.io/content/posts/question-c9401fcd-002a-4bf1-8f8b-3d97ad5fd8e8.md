---
title: "Как настроить DNSOverTLS на актуальных верcиях systemd, RIP Debian 10 и на macOS?"
summary: "В новой версии systemd-resolved для включения DNSOverTLS используется синтаксис поддерживающей формат IP#hostname, он  необходим для передачи SNI и проверки сертификата DNS-сервера."
tags: ["ai-generated", "todo", "dns", "dot", "doh", "DNSOverTLS"]
date: 2026-08-06T18:26:43Z
tldr: "933820173"
---

# DNS over HTTPS (DoH) и DNS over TLS (DoT) — это две разные технологии для шифрования обычных DNS-запросов, которые защищают историю посещенных сайтов от прослушивания и подделки. 

Запрос превращается в зашифрованный трафик с помощью протокола TLS.
Главные отличия DoH и DoTDNS over HTTPS (DoH): Упаковывает DNS-запросы в обычный веб-трафик HTTPS (порт 443). Такое соединение трудно отличить от просмотра веб-страниц, его сложнее заблокировать в сети. Часто встраивается прямо в браузеры.
DNS over TLS (DoT): Отправляет запросы напрямую через защищенный транспортный протокол TLS по выделенному порту 853. 
Системным администраторам и провайдерам легко увидеть и проконтролировать такой специализированный трафик.

Зачем они нужны?

Конфиденциальность: 

- Обычный DNS передает адреса сайтов в открытом виде. 
- Шифрование скрывает их от вашего интернет-провайдера или владельца публичного Wi-Fi.

Безопасность: 

- Защищает от перехвата и подделки DNS-ответов (атак «посредника»)

## Настройка DNSOverTLS на актуальной версии Debian12

Отключаем DNS в NetworkManager

Файл: /etc/NetworkManager/conf.d/dns.conf
```
[main]
dns=none
```

Перезапуск NetworkManager

```
systemctl restart NetworkManager
```

Настройка Systemd-resolved
```
systemctl enable --now systemd-resolved.service
```

Конфигурация

Файл: /etc/systemd/resolved.conf

```
cat /etc/systemd/resolved.conf
```

```
[Resolve]
DNS=94.140.14.14\#dns.adguard-dns.com 94.140.15.15\#dns.adguard-dns.com
FallbackDNS=9.9.9.9\#dns.quad9.net 8.8.8.8\#dns.google 1.1.1.1\#one.one.one.one
DNSSEC=yes
DNSOverTLS=yes
Cache=yes
LLMNR=no
MulticastDNS=no
ReadEtcHosts=yes
```

Файл: /etc/resolv.conf
```
nameserver 127.0.0.53
```

Перезапуск
```
systemctl restart systemd-resolved.service
```

Проверка
```
resolvectl
```
Примечание: в строке “Protocols:” будет +DNSOverTLS DNSSEC=yes/supported

---

## Настройка DNSOverTLS на Debian10


Если вы еще используете устаревшие версии OS  - например Debian10, то там такой конфиг не будет работать

Вы увидите ошибки вроде:
```
Aug 06 15:36:27 master systemd-resolved[18619]: . IN DS 20326 8 2 e06d44b80b8f1d39a95c0b0d7c65d08458e880409bbc683457104237c7f8ec8d
Aug 06 15:36:27 master systemd-resolved[18619]: Negative trust anchors: 10.in-addr.arpa 16.172.in-addr.arpa 17.172.in-addr.arpa 18.172.in-addr.arpa 19.172.
Aug 06 15:36:27 master systemd-resolved[18619]: Failed to add DNS server address '94.140.14.14\#dns.adguard-dns.com', ignoring: Invalid argument
Aug 06 15:36:27 master systemd-resolved[18619]: Failed to add DNS server address '94.140.15.15\#dns.adguard-dns.com', ignoring: Invalid argument
Aug 06 15:36:27 master systemd-resolved[18619]: Failed to add DNS server address '9.9.9.9\#dns.quad9.net', ignoring: Invalid argument
Aug 06 15:36:27 master systemd-resolved[18619]: Failed to add DNS server address '8.8.8.8\#dns.google', ignoring: Invalid argument
Aug 06 15:36:27 master systemd-resolved[18619]: Failed to add DNS server address '1.1.1.1\#one.one.one.one', ignoring: Invalid argument
Aug 06 15:36:27 master systemd-resolved[18619]: /etc/systemd/resolved.conf:5: Failed to parse DNS-over-TLS mode setting, ignoring: yes
Aug 06 15:36:27 master systemd-resolved[18619]: Using system hostname 'master.sytes.ru'.
Aug 06 15:36:27 master systemd[1]: Started Network Name Resolution.
```

В Debian 10 используется старый `systemd-resolved`, поэтому текущая конфигурация содержит две несовместимые вещи:

1. Формат DNS-сервера `IP#hostname` в этой версии не принимается, отсюда `Invalid argument`.
2. Для `DNSOverTLS` в Debian 10 допустимы `false` и `opportunistic`; значение `yes` эта версия не понимает.[^16_1]

## Конфигурация для Debian 10

Замените `/etc/systemd/resolved.conf` на:

```ini
[Resolve]
DNS=94.140.14.14 94.140.15.15
FallbackDNS=9.9.9.9 8.8.8.8 1.1.1.1
DNSSEC=yes
DNSOverTLS=opportunistic
Cache=yes
LLMNR=no
MulticastDNS=no
ReadEtcHosts=yes
```

`DNSOverTLS=opportunistic` означает: `systemd-resolved` попытается использовать DNS-over-TLS, но при невозможности установления TLS может перейти на обычный DNS. В Debian 10 штатными средствами нельзя корректно использовать строгий режим `DNSOverTLS=yes` с именем сертификата через формат `IP#hostname`.

Если приоритет — стабильная работа DNS, можно временно отключить DNS-over-TLS:

```ini
DNSOverTLS=false
```

Но тогда запросы будут отправляться обычным DNS.

## Применение настроек

```bash
sudo systemctl restart systemd-resolved
sudo systemctl enable systemd-resolved
```

Проверьте, куда указывает `/etc/resolv.conf`:

```bash
readlink -f /etc/resolv.conf
```

Для `systemd-resolved` желательно, чтобы он указывал на:

```text
/run/systemd/resolve/stub-resolv.conf
```

Если это не так:

```bash
sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
sudo systemctl restart systemd-resolved
```

Проверка:

```bash
resolvectl status
resolvectl query debian.org
journalctl -u systemd-resolved -b --no-pager
```

В Debian 10 команда иногда называется `systemd-resolve`, а не `resolvectl`:

```bash
systemd-resolve --status
systemd-resolve debian.org
```

## Если нужны строгие DNS-over-TLS

Для строгой проверки сертификата конфигурация обычно выглядит так:

```ini
[Resolve]
DNS=94.140.14.14#dns.adguard-dns.com 94.140.15.15#dns.adguard-dns.com
DNSOverTLS=yes
```

Но это требует более новой версии `systemd-resolved`, поддерживающей формат `IP#hostname`. Такой синтаксис используется в современных версиях `systemd`; он необходим для передачи SNI и проверки сертификата DNS-сервера.

Поэтому для Debian 10 варианты такие:

- использовать `DNSOverTLS=opportunistic` и IP-адреса без `#hostname`;
- обновить Debian до более новой версии;
- установить отдельный DNS-клиент с поддержкой строгого DoT, например `stubby` или `knot-resolver`.

Также учтите, что Debian 10 уже устарел и не является оптимальной базой для современной криптографической конфигурации.

---

## Как настроить DNSOverTLS на macOS?

На macOS системный DNS-over-TLS настраивается не через обычное поле DNS в настройках сети, а через конфигурационный профиль `.mobileconfig`. Простое добавление `94.140.14.14` или `1.1.1.1` в настройках DNS включает только обычный DNS без шифрования.[^17_1]

## Вариант 1: профиль `.mobileconfig`

Поддержка зашифрованного DNS через системные профили появилась в macOS Big Sur 11 и новее. Профиль содержит адреса DNS-серверов, имя TLS-сервера и тип протокола — DoT или DoH.[^17_2][^17_3]

Общий порядок:

1. Скачайте профиль DNS-over-TLS у выбранного провайдера.
2. Откройте файл `.mobileconfig`.
3. Перейдите в настройки установки профиля:
    - macOS Ventura/Sonoma/Sequoia и новее: **Системные настройки → Основные → Управление устройством**;
    - старые версии: **Системные настройки → Профили**.
4. Нажмите **Установить** и подтвердите пароль администратора.
5. Проверьте активный профиль в разделе управления устройством.

Важно: профиль должен быть получен из доверенного источника. Перед установкой проверьте его содержимое, особенно DNS-серверы и название TLS-хоста.

##  AdGuard DNS

Для AdGuard DNS в профиле должны использоваться соответствующие TLS-имена:

```text
94.140.14.14
dns.adguard-dns.com
```

```text
94.140.15.15
dns.adguard-dns.com
```

В обычных сетевых настройках macOS эти значения нужно указывать только как IP-адреса — поле DNS не понимает запись вида:

```text
94.140.14.14#dns.adguard-dns.com
```

Такая запись является форматом `systemd-resolved`, а не macOS.

## Проверка профиля и DNS

Посмотреть DNS-серверы, используемые macOS:

```bash
scutil --dns
```

Проверить разрешение имени:

```bash
dig example.com
```

Также можно проверить, что профиль установлен:

```bash
profiles status -type enrollment
```

В новых версиях macOS удобнее открыть:

```text
Системные настройки → Основные → Управление устройством
```

и найти установленный DNS-профиль. Подробнее это разобрано в П.4.



## 3. Запуск DNS-over-TLS  через Stubby

Если нужен ручной контроль без `.mobileconfig`, можно запустить локальный DNS-прокси Stubby, который будет принимать запросы на `127.0.0.1` и отправлять их через DoT. Для macOS его можно установить через Homebrew:[^17_4]

```bash
brew install stubby
```

Пример конфигурации:

```yaml
resolution_type: GETDNS_RESOLUTION_STUB
dns_transport_list:
  - GETDNS_TRANSPORT_TLS
tls_authentication: GETDNS_AUTHENTICATION_REQUIRED

listen_addresses:
  - 127.0.0.1@53

upstream_recursive_servers:
  - address_data: 9.9.9.9
    tls_auth_name: "dns.quad9.net"

  - address_data: 1.1.1.1
    tls_auth_name: "cloudflare-dns.com"
```

Запуск:

```bash
brew services start stubby
```

Затем задайте локальный DNS для интерфейса:

```bash
networksetup -listallnetworkservices
```

Для Wi-Fi:

```bash
sudo networksetup -setdnsservers Wi-Fi 127.0.0.1
```

Для Ethernet:

```bash
sudo networksetup -setdnsservers Ethernet 127.0.0.1
```

Проверка:

```bash
dig example.com
```

Чтобы вернуть DNS, выдаваемый DHCP:

```bash
sudo networksetup -setdnsservers Wi-Fi empty
```
Для обычного современного Mac наиболее простой вариант — подписанный `.mobileconfig` профиль. Для ручной настройки, нескольких upstream-серверов и строгой проверки TLS удобнее Stubby.

---

## 4. Создаем профиль с настройками DNS-over-TLS на MAC OS

На macOS 11 Big Sur и новее DNS-over-TLS можно включить системным профилем `.mobileconfig`; обычное добавление DNS-адресов в сетевых настройках шифрование не включает.[^18_1]


Ниже пример профиля для **AdGuard DNS**.


Откройте Terminal и создайте файл:

```bash
cat > ~/adguard-dot.mobileconfig <<'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>PayloadContent</key>
    <array>
        <dict>
            <key>DNSSettings</key>
            <dict>
                <key>DNSProtocol</key>
                <string>TLS</string>

                <key>ServerName</key>
                ```
                <string>dns.adguard-dns.com</string>
                ```

                <key>ServerAddresses</key>
                <array>
                    <string>94.140.14.14</string>
                    <string>94.140.15.15</string>
                </array>
            </dict>

            <key>PayloadDescription</key>
            <string>AdGuard DNS over TLS</string>

            <key>PayloadDisplayName</key>
            ```
            <string>AdGuard DNS-over-TLS</string>
            ```

            <key>PayloadIdentifier</key>
            <string>ru.example.adguard.dnssettings</string>

            <key>PayloadOrganization</key>
            <string>Personal</string>

            <key>PayloadType</key>
            <string>com.apple.dnsSettings.managed</string>

            <key>PayloadUUID</key>
            ```
            <string>7A5F9F36-8E4C-4D3A-9D13-ADGUARD2026</string>
            ```

            <key>PayloadVersion</key>
            <integer>1</integer>
        </dict>
    </array>

    <key>PayloadDisplayName</key>
    ```
    <string>AdGuard DNS-over-TLS</string>
    ```

    <key>PayloadIdentifier</key>
    <string>ru.example.adguard.profile</string>

    <key>PayloadOrganization</key>
    <string>Personal</string>

    <key>PayloadRemovalDisallowed</key>
    <false/>

    <key>PayloadType</key>
    <string>Configuration</string>

    <key>PayloadUUID</key>
    ```
    <string>F7D6F8AA-9A8A-49A6-86B8-ADGUARDPROFILE</string>
    ```

    <key>PayloadVersion</key>
    <integer>1</integer>
</dict>
</plist>
EOF
```

Проверьте XML:

```bash
plutil -lint ~/adguard-dot.mobileconfig
```

Ожидаемый результат:

```text
~/adguard-dot.mobileconfig: OK
```


## Установка профиля

Откройте файл:

```bash
open ~/adguard-dot.mobileconfig
```

Далее установите профиль в системных настройках:

- macOS Ventura/Sonoma/Sequoia: **Системные настройки → Основные → Управление устройством**;
- в некоторых версиях: **Системные настройки → Конфиденциальность и безопасность → Профили**;
- выберите профиль `AdGuard DNS-over-TLS`;
- нажмите **Установить**;
- подтвердите установку паролем администратора.

После установки профиль должен появиться среди активных профилей DNS. Профили `.mobileconfig` являются штатным способом настройки DoT/DoH на macOS.[^18_2]

## Проверка

Посмотрите активную DNS-конфигурацию:

```bash
scutil --dns
```

Проверьте разрешение имени:

```bash
dig example.com
```

Также проверьте, что DNS-запросы действительно идут через AdGuard:

```bash
curl https://www.dnsleaktest.com/
```

Для более практичной проверки можно открыть:

```text
https://www.dnsleaktest.com/
```


## Профиль для Quad9

Для Quad9 замените в профиле:

```xml
```

<string>dns.adguard-dns.com</string>

```
```

на:

```xml
<string>dns.quad9.net</string>
```

и адреса:

```xml
<string>9.9.9.9</string>
<string>149.112.112.112</string>
```

Для Cloudflare:

```xml
<key>ServerName</key>
```

<string>cloudflare-dns.com</string>

```
```

```xml
<key>ServerAddresses</key>
<array>
    <string>1.1.1.1</string>
    <string>1.0.0.1</string>
</array>
```

`ServerName` обязателен: это имя используется для проверки TLS-сертификата DNS-сервера. Поэтому на macOS нельзя просто указать `94.140.14.14#dns.adguard-dns.com`, как в `systemd-resolved`.

Удалить профиль можно через **Системные настройки → Основные → Управление устройством**, выбрав профиль и нажав **Удалить профиль**.

<span style="display:none">[^18_10][^18_11][^18_12][^18_13][^18_14][^18_15][^18_3][^18_4][^18_5][^18_6][^18_7][^18_8][^18_9]</span>

<div align="center">⁂</div>

[^18_1]: https://quad9.net/news/blog/ios-mobile-provisioning-profiles/

[^18_2]: https://simpledns.plus/kb/201-how-to-enable-dns-over-tls-dot-dns-over-https-doh-in-macos-v11

[^18_3]: https://github.com/undirectlookable/apple-encrypted-dns-profile

[^18_4]: https://github.com/cntrump/dns-mobileconfig

[^18_5]: https://github.com/mullvad/encrypted-dns-profiles

[^18_6]: https://upset.dev/dns-profile-generator/

[^18_7]: https://dns.senkl.eu/

[^18_8]: https://simpledns.plus/apple-dot-doh

[^18_9]: https://dns-over-https.org/ko/guides/how-to-setup-doh-in-macos/

[^18_10]: https://dnsgratis.com/ru/nastroika/macos/

[^18_11]: https://joshspicer.com/dns-over-https-ios

[^18_12]: https://north.engineer/Networking/DNS/apple-configuration-profile

[^18_13]: https://support.apple.com/ru-ru/guide/mac-help/mh14127/mac

[^18_14]: https://publicdns.info/guides/encrypted-dns/macos.html

[^18_15]: https://infotoast.org/site/index.php/2024/09/12/how-to-activate-dns-over-https-on-mac-and-ios/
