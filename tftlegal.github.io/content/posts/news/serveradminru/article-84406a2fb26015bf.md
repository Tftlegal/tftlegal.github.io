---
title: "Защита почтового сервера Postfix + Dovecot с помощью Fail2Ban"
date: 2025-12-02T10:20:39Z
source: "serveradminru"
original_url: "https://serveradmin.ru/fail2ban-postfix-dovecot/"
summary: "Автор поднимает важную тему, которую он часто упускал в своих статьях о настройке почтовых серверов. Речь идёт о защите почтового сервера от перебора паролей. В тексте объясняется, как с помощью Fail2Ban защитить связку Postfix и Dovecot. Цель заключается в предотвращении подбора паролей к почтовым ящикам. Применяется простой и надёжный метод: блокируются IP-адреса, которые совершают подозрительные попытки. Материал опубликован на сайте Server Admin."
---

# Защита почтового сервера Postfix + Dovecot с помощью Fail2Ban

## Краткое содержание

Автор поднимает важную тему, которую он часто упускал в своих статьях о настройке почтовых серверов.
Речь идёт о защите почтового сервера от перебора паролей.
В тексте объясняется, как с помощью Fail2Ban защитить связку Postfix и Dovecot.
Цель заключается в предотвращении подбора паролей к почтовым ящикам.
Применяется простой и надёжный метод: блокируются IP-адреса, которые совершают подозрительные попытки.
Материал опубликован на сайте Server Admin.

## Полная статья

[https://serveradmin.ru/audio/fail2ban-postfix-dovecot.ogg](https://serveradmin.ru/audio/fail2ban-postfix-dovecot.ogg)

У меня есть много[статей](https://serveradmin.ru/tag/mailserver/)про настройку почтового сервера, где я постоянно пропускаю важную тему защиты почтового сервера от перебора паролей. Пришло время это исправить и рассказать, как защитить Postfix и Dovecot с помощью Fail2Ban от подбора паролей к почтовым ящикам. Метод традиционный, простой и надежный - будем банить по IP тех, кто будет пытаться пройти аутентификацию с неверными учетными данными.

**Углубленный онлайн-курс по MikroTik**

Научиться настраивать MikroTik с нуля или систематизировать уже имеющиеся знания можно[на углубленном онлайн-курcе по администрированию MikroTik](https://курсы-по-ит.рф/mikrotik-mtcna?utm_source=serveradminru&utm_medium=cpc&utm_campaign=mikrotik&erid=2SDnjeiuzYr). Автор курcа – сертифицированный тренер MikroTik Дмитрий Скоромнов. Более 40 лабораторных работ по которым дается обратная связь. В три раза больше информации, чем в MTCNA.
Реклама ИП Скоромнов Д.А. ИНН 331403723315

## Введение

**Fail2Ban**- очень старый и известный продукт. У него настолько простая и эффективная функциональность, что за всё время, что я настраиваю сервера на базе Linux, не появилось прямых аналогов. Кроме разве что[CrowdSec](https://serveradmin.ru/ustanovka-i-nastrojka-crowdsec/). Принцип работы Fail2Ban очень простой - анализируем логи сервиса, смотрим там по шаблону строки с неверной аутентификацией, распознаём в этих строках IP адрес и баним их с помощью файрвола.

Данная статья написана на примере настройки почтового сервера по моей статье -[Настройка Postfix + Dovecot + Postfixadmin + Roundcube + DKIM на Debian](https://serveradmin.ru/nastrojka-postfix-dovecot-postfixadmin-roundcube-dkim-na-debian/). Но в контексте описываемого материала это не принципиально, так как Fail2Ban, Postfix и Dovecot имеют одинаковые конфигурации и логи на всех дистрибутивах Linux. Так что представленная в статье информация будет актуальна для любого сервера, где используется это программное обеспечение.

Второй важный момент. Я в своей работе везде использую нативные iptables. Для блокировки IP адресов с помощью Fail2Ban я буду использовать именно этот firewall. Если у вас его нет и вы хотите настроить, то добро пожаловать в мою статью по этой теме -[настройка iptables](https://serveradmin.ru/nastroyka-iptables-v-centos-7/). Далее я не буду останавливаться на этом.

## Установка и настройка Fail2Ban

Установка Fail2Ban на любом дистрибутиве не представляет никаких сложностей, так как продукт популярный и присутствует почти во всех репозиториях популярных дистрибутивов. Ставим через пакетный менеджер. В rpm дистрибутивах пакет обычно живёт в[репозитории epel](https://serveradmin.ru/ustanovka-repozitoriya-epel-rpmforge-v-centos/#_epel_repo_CentOS).

```
# yum install fail2ban
# dnf install fail2ban
```

```
# apt install fail2ban
```

[

![](/news/serveradminru/article-84406a2fb26015bf/image-01.png)

](https://serveradmin.ru/wp-content/uploads/2025/12/fail2ban-debian-postfix-dovecot-01.png)

В Debian в базовой установке по умолчанию не установлен файрвол. Уже много лет существует[nftables](https://serveradmin.ru/bazovye-nastrojki-nftables-dlya-veb-servera-na-debian/), который пришёл на смену iptables и в целом он удобнее. В контексте данной задачи непринципиально, какой будет использоваться файрвол. Я буду использовать iptables, потому больше привык к нему, плюс по этому продукту накопилась огромная база знаний в интернете. Проще и настройку найти, и какие-то проблемы решить. Так что устанавливаем iptables:

[

![](/news/serveradminru/article-84406a2fb26015bf/image-02.png)

](https://serveradmin.ru/wp-content/uploads/2025/12/fail2ban-debian-postfix-dovecot-02.png)

В Debian у Fail2Ban есть стандартный файл конфигурации для этой системы*/etc/fail2ban/jail.d/defaults-debian.conf*. Добавлю настройки iptables туда, а то, что было по умолчанию, закомментирую. Привел его к такому виду.

```
#[DEFAULT]
#banaction = nftables
#banaction_allports = nftables[type=allports]

[DEFAULT]
banaction = iptables-multiport
banaction_allports = iptables-allports
backend = auto
ignoreip = 127.0.0.1/8 10.10.5.21/32

[sshd]
backend = systemd
journalmatch = _SYSTEMD_UNIT=ssh.service + _COMM=sshd
enabled = true
```

Я выделил IP адрес Zabbix сервера, на котором настроен[мониторинг работы почтового сервера](https://serveradmin.ru/monitoring-postfix-v-zabbix/)и мониторинг[tls сертификатов](https://serveradmin.ru/monitoring-sroka-deystviya-ssl-sertifikata-v-zabbix/). Если его не добавить в исключения, то он будет забанен, так как регулярно подключается к почтовому серверу, но не проходит аутентификацию. Ему это не нужно для работы. Со стороны защиты это будет выглядеть подозрительно.

Правило с sshd было по умолчанию, я не стал убирать. Если вам оно не нужно, то закомментируйте. Когда будете редактировать любой конфигурационный файл, не забудьте на всякий случай сохранить оригинал.

Базовая настройка Fail2Ban закончена. Все остальное я оставил по умолчанию. Все основные настройки с правилами будут дальше. Переходим к настройке правил блокировки.

## Защита Postfix с помощью Fail2Ban

Изначально Fail2Ban идет с комплектом готовых настроек и фильтров для защиты большинства популярных сервисов. К ним относится и Postfix. Для этого в Debian есть файл /etc/fail2ban/filter.d/postfix.conf.

[

![](/news/serveradminru/article-84406a2fb26015bf/image-03.png)

](https://serveradmin.ru/wp-content/uploads/2025/12/fail2ban-debian-postfix-dovecot-03.png)

Раз за разом я настраиваю Fail2Ban для Postfix на разных системах и всегда сталкиваюсь с одной и той же проблемой. Стандартные правила не блокируют основные проблемные подключения с перебором паролей. Так вышло и в этот раз. Проверял на версии 3.10.5. В ней перебор паролей в логе выглядит вот так:

```
2025-12-02T07:01:16.393264+03:00 mail postfix/smtpd[217468]: warning: unknown[213.209.157.207]: SASL LOGIN authentication failed: (reason unavailable), sasl_username=sales@zeroxzed.ru
```

Если использовать стандартный фильтр, то он никак не отреагирует на подобную строку. Проверить можно вот так:

```
# fail2ban-regex /var/log/mail.log /etc/fail2ban/filter.d/postfix.conf --print-all-matched
```

[

![](/news/serveradminru/article-84406a2fb26015bf/image-04.png)

](https://serveradmin.ru/wp-content/uploads/2025/12/fail2ban-debian-postfix-dovecot-04.png)

На картинке не видно, так как всё не уместилось, но ниже идут строки с совпадением по маске из файла /etc/fail2ban/filter.d/postfix.conf. Не знаю, зачем, но в Postfix периодически немного меняется формат строки с фразой**SASL LOGIN authentication failed**, из-за этого фильтры приходится править по месту. Я уже третий раз пишу статью по этой теме и каждый раз приходится немного изменять шаблон.

В итоге не стал редактировать стандартный файл postfix.conf, а просто добавил ещё один. Назвал его postfix-sasl.conf. Содержание очень простое:

```
[INCLUDES]
before = common.conf
[Definition]
_daemon = postfix/smtpd
failregex = ^%(__prefix_line)swarning: [-._\w]+\[<HOST>\]: SASL (?:LOGIN|PLAIN|(?:CRAM|DIGEST)-MD5) authentication failed:
ignoreregex =
```

Этот шаблон отлично отлавливает все неудачные аутентификации. Проверяем так же:

```
# fail2ban-regex /var/log/mail.log /etc/fail2ban/filter.d/postfix-sasl.conf --print-all-matched
```

[

![](/news/serveradminru/article-84406a2fb26015bf/image-05.png)

](https://serveradmin.ru/wp-content/uploads/2025/12/fail2ban-debian-postfix-dovecot-05.png)

В моём случае сработал чётко. Проверяйте у себя и изменяйте шаблон, если у вас он не подойдёт. Обычно там немного регулярку подправить надо. Заодно проверьте стандартный шаблон. Он тоже отсекает некоторые проблемные соединения с ошибками. Так что можно оставить и его.

Для удобства добавляем в /etc/fail2ban/jail.d/ отдельный файл конфигурации для Postfix и Dovecot. Я назвал его postfix-dovecot.conf. Пока добавляем туда настройки только по Postfix:

```
[postfix]
enabled = true
filter  = postfix
port    = smtp,465,submission
action  = iptables[name=Postfix, port=smtp, protocol=tcp]
logpath = /var/log/mail.log
bantime = 60m
maxretry = 3
findtime = 60m

[postfix-sasl]
enabled = true
filter  = postfix-sasl
port    = smtp,465,submission
action  = iptables[name=Postfix-sasl, port=smtp, protocol=tcp]
logpath = /var/log/mail.log
bantime = 60m
maxretry = 3
findtime = 60m
```

Параметры bantime, maxretry и findtime подберите под себя. По смыслу понятно, что они значат. В принципе банить можно и подольше, но особо не вижу смысла ужесточать. Даже при таких вводных подбор не словарных паролей будет невозможен. Но имейте ввиду, что при настройке новых клиентов, если 3 раза ошибётесь с учёткой или параметрами TLS, то тоже улетите в бан. На практике такое нередко бывает, так что либо сотрудникам на местах это объясните, либо дайте им доступ к разбану.

Запускаем Fail2Ban и проверяем:

```
# systemctl enable --now fail2ban
```

Лог службы текстовый - /var/log/fail2ban.log. Вся основная информация по распознаванию и банам там есть.

[

![](/news/serveradminru/article-84406a2fb26015bf/image-06.png)

](https://serveradmin.ru/wp-content/uploads/2025/12/fail2ban-debian-postfix-dovecot-06.png)

Если у вас кто-то уже забанен, то вы увидите это в правилах iptables:

```
# iptables -L -v -n
```

[

![](/news/serveradminru/article-84406a2fb26015bf/image-07.png)

](https://serveradmin.ru/wp-content/uploads/2025/12/fail2ban-debian-postfix-dovecot-07.png)

Так же список забаненых IP адресов можно посмотреть через команды**fail2ban-client**:

```
# fail2ban-client status postfix-sasl
Status for the jail: postfix-sasl
|- Filter
|  |- Currently failed: 1
|  |- Total failed:     9
|  `- Journal matches:
`- Actions
   |- Currently banned: 1
   |- Total banned:     3
   `- Banned IP list:   213.209.157.207
```

Можно принудительно кого-то забанить или разбанить:

```
# fail2ban-client set postfix-sasl banip 213.209.157.207
# fail2ban-client set postfix-sasl unbanip 213.209.157.207
```

С защитой Postfix с помощью Fail2Ban всё. Переходим к Dovecot. Там всё то же самое, только шаблоны другие, так что кратенько пройдём по настройкам.

## Защита Dovecot с помощью Fail2Ban

Для защиты от перебора логинов в Dovecot мы будем действовать аналогично. Дефолтный фильтр для Dovecot, который шёл из коробки, так же, как и для Postfix, не сработал. Я его проверил.

```
# fail2ban-regex /var/log/dovecot/info.log /etc/fail2ban/filter.d/dovecot.conf
Lines: 1029 lines, 0 ignored, 0 matched, 1029 missed
```

Несмотря на то, что попытки подборка учёток в логе есть, фильтр ничего не нашёл. Полный лог Dovecot у меня располагается в файле*/var/log/dovecot/info.log*. В параметрах Dovecot у меня добавлено:

```
auth_verbose = yes
```

С этим параметром у вас дополнительно в логе будут следующие строки:

```
Dec 02 09:22:22 auth-worker(testuser@zeroxzed.ru,213.209.157.207)<230299>: request [1]: Info: sql: unknown user
```

По дефолту их нет, а они помогут в нашей задаче по защите от перебора паролей учетных записей Dovecot. Я сходил в репозиторий Fail2Ban забрал оттуда самую свежую версию[правил для Dovecot](https://github.com/fail2ban/fail2ban/blob/master/config/filter.d/dovecot.conf). Эти правила сработали лучше и выявили строки, где идёт подбор пароля к существующим учёткам. Это записи в логе такого вида:

```
Dec 01 09:14:20 pop3-login: Info: Login aborted: Connection closed (auth failed, 1 attempts in 0 secs) (auth_failed): user=<root@zeroxzed.ru>, rip=190.197.9.185, lip=212.193.57.223, session=<f5fF491ElOW5xAi4>
```

Но строки с*unknown user*всё равно почему-то не отлавливались, хотя регулярка была. Помучал поиск, ничего толком не нашёл. По новому синтаксису Dovecot 2.4 очень мало информации. Ещё немного помучал ИИ. Тот постоянно герерил какой-то неработающий бред. В общем, пришлось немного напрячь извилины и написать, как смог, правило самому. Назвал файл dovecot-sql.conf:

```
[Definition]
_daemon = (auth|dovecot(-auth)?|auth-worker)
failregex = ^.+auth-worker\(\S+,\).+sql: (?:unknown user|Password mismatch)$
ignoreregex =
```

Проверил, вроде работает, корректно распознаёт нужные строки:

```
# fail2ban-regex /var/log/dovecot/info.log /etc/fail2ban/filter.d/dovecot-sql.conf
```

[

![](/news/serveradminru/article-84406a2fb26015bf/image-08.png)

](https://serveradmin.ru/wp-content/uploads/2025/12/fail2ban-debian-postfix-dovecot-08.png)

Добавил в /etc/fail2ban/jail.d/postfix-dovecot.conf по аналогии с Postfix оба правила, и стандартное, и своё.

```
[dovecot]
enabled = true
filter  = dovecot
port    = imap,imaps,pop3,pop3s
action  = iptables[name=Dovecot, port=imap, protocol=tcp]
logpath = /var/log/dovecot/info.log
bantime = 60m
maxretry = 3
findtime = 60m

[dovecot-sql]
enabled = true
filter  = dovecot-sql
port    = imap,imaps,pop3,pop3s
action  = iptables[name=Dovecot, port=imap, protocol=tcp]
logpath = /var/log/dovecot/info.log
bantime = 60m
maxretry = 3
findtime = 60m
```

Перезапустил fail2ban:

```
# systemctl restart fail2ban
```

Здесь ничего нового. Всё то же самое, что мы сделали выше. В логе должна появиться информация по работе данного jail'а. А в правилах iptables новые цепочки для Dovecot в случае работы блокировок.

## Отладка работы fail2ban

После добавления правил блокировки IP адресов с помощью Fail2Ban, я некоторое время наблюдаю за сервером, чтобы проверить правильность работы. Для этого делаю вот такое окно в отдельном мониторе и наблюдаю некоторое время.

[

![Отладка работы fail2ban](/news/serveradminru/article-84406a2fb26015bf/image-09.png)

](https://serveradmin.ru/wp-content/uploads/2020/06/fail2ban-postfix-dovecot-07.png)

Здесь открыт лог Postfix, Dovecot и Fail2Ban. Если вижу, что правила отрабатываются корректно, завершаю настройку. На этом этапе могут быть заблокированы корректные IP адреса пользователей, у которых одна из учёток указана с неверным паролем. В итоге он банится по IP и у него вообще перестает работать вся почта. Если это локальные пользователи, то можно всю их подсеть добавить в доверенные, но я бы не рекомендовал так делать. В этом случае вы не узнаете, что кто-то вас перебирает из локальной сети. А это случается нередко.

## Удалить IP адрес из заблокированных Fail2Ban

Вам может понадобиться вручную удалить какой-то IP адрес из списка заблокированных Fail2Ban. Часто в блок попадают IP адреса при настройке учётной записи у пользователя. Пароль может быть перепутан или копироваться с лишними символами. Всякое бывает.

Можно напрямую удалить правило через iptables. Но это будет не очень правильно. Лучше воспользоваться готовым инструментом от Fail2Ban для удаления IP адресов из блокировки.

Смотрим список активных jail'ов.

```
# fail2ban-client status
Status
|- Number of jail: 5
`- Jail list: dovecot, dovecot-sql, postfix, postfix-sasl, sshd
```

Смотрим список заблокированный ip адресов в jail.

```
# fail2ban-client status postfix-sasl
Status for the jail: postfix-sasl
|- Filter
|  |- Currently failed:	1
|  |- Total failed:	169
|  `- File list:	/var/log/mail.log
`- Actions
   |- Currently banned:	27
   |- Total banned:	86
   `- Banned IP list:	46.38.150.193 87.246.7.66 212.70.149.2 141.98.80.150 46.38.150.203 185.143.75.153 212.70.149.18 46.38.150.191 87.246.7.70 185.143.75.81 185.143.72.34 46.38.150.142 46.38.150.190 185.143.72.27 185.143.72.25 185.143.72.23 46.38.145.6 46.38.145.252 46.38.150.188 46.38.145.249 46.38.145.5 46.38.145.250 46.38.145.248 46.38.145.254 46.38.145.253 185.143.72.16 46.38.145.251
```

Теперь разбаним один из адресов в Fail2Ban:

```
# fail2ban-client set postfix-sasl unbanip 46.38.150.193
1
```

Если получите одну из этих ошибок:

```
2020-06-17 20:15:14,809 fail2ban [77578]: ERROR NOK: ('Invalid command (no get action or not yet implemented)',)
2020-06-17 20:13:02,078 fail2ban [77464]: ERROR NOK: ("Invalid command '46.38.150.193' (no set action or not yet implemented)",)
2020-06-17 20:11:48,132 fail2ban [77382]: ERROR   NOK: ('list index out of range',)
```

Значит у вас более старая версия Fail2Ban. Тогда нужно использовать другую команду для разбана ip адреса:

```
# fail2ban-client get postfix-sasl actionunban 46.38.150.193
```

Для проверки можете посмотреть на цепочки правил iptables, чтобы убедиться, в том, что адреса реально удалены из блокировки.

Можете попробовать сымитировать неверную аутентификацию, к примеру, по SMTP. Для этого подойдёт telnet. Подключаемся с его помощью к почтовому серверу:

```
# telnet mail.zeroxzed.ru 25
Trying 212.193.57.223...
Connected to mail.zeroxzed.ru.
Escape character is '^]'.
220 mail.zeroxzed.ru ESMTP Postfix
```

Подключились. Сервер ждёт от нас команд. Отправляем:

```
ehlo zeroxzed.ru
AUTH LOGIN
```

Получили ответ

```
334 VXNlcm5hbWU6
```

Теперь отправляем имя пользователя*admin*, закодированное base64:

```
YWRtaW4=
```

Получаем ответ:

```
334 UGFzc3dvcmQ6
```

Передаём пароль*password*:

```
cGFzc3dvcmQ=
```

Получаем в ответ ошибку:

```
535 5.7.8 Error: authentication failed: (reason unavailable)
```

[

![](/news/serveradminru/article-84406a2fb26015bf/image-10.png)

](https://serveradmin.ru/wp-content/uploads/2025/12/fail2ban-debian-postfix-dovecot-09.png)

В логе mail.log эта попытка аутентификации будет выглядеть так:

```
warning: unknown[95.145.141.246]: SASL LOGIN authentication failed: (reason unavailable), sasl_username=admin@zeroxzed.ru
```

Смотрим лог fail2ban.log:

```
2025-12-02 13:08:41,808 fail2ban.filter [248808]: INFO [postfix-sasl] Found 95.145.141.246 - 2025-12-02 13:08:41
```

Всё корректно отработало. После трёх таких попыток IP адрес должен улететь в бан.

## Заключение

На практике правила в Fail2Ban для Dovecot особо не нужны. Пробивкой учетных записей занимаются боты, которые сразу пробивают smtp и imap порты. Все эти боты первым делом попадают в блокировку Postfix и до правил Dovecot просто не доходят. Но для полноты картины можно оставить и их, хотя бы для того, чтобы выявлять сотрудников с настроенными неактивными учётками.

С помощью Fail2Ban можно так же банить различные почтовые серверы, которые не проходят встроенные проверки Postfix на спам. Но я обычно этого не делаю, так как бывают ложные срабатывания. Потом приходится лишнее время тратить на разбор полетов, так как он усложняется. Так стоит делать, если левые коннекты реально замедляют работу почтового сервера. Обычно это не добавляет каких-то серьезных проблем, в отличие от перебора паролей.

**Онлайн-курс по устройству компьютерных сетей.**

На углубленном курсе "[Архитектура современных компьютерных сетей](https://курсы-по-ит.рф/computer_networks?utm_source=serveradminru&utm_medium=cpc&utm_campaign=asks&erid=2SDnjeiuzYr)" вы с нуля научитесь работать с Wireshark и «под микроскопом» изучите работу сетевых протоколов. На протяжении курса надо будет выполнить более пятидесяти лабораторных работ в Wireshark.
Реклама ИП Скоромнов Д.А. ИНН 331403723315

#### Помогла статья? Подписывайся на[telegram канал](https://serveradmin.ru/telegram-header)автора

Анонсы всех статей, плюс много другой полезной и интересной информации, которая не попадает на сайт.

## Оригинал

https://serveradmin.ru/fail2ban-postfix-dovecot/
