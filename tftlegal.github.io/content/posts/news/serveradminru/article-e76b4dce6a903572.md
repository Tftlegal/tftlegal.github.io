---
title: "Настройка Postfix + Dovecot + Postfixadmin + Roundcube + DKIM на Debian"
date: 2025-11-30T07:00:48Z
source: "serveradminru"
original_url: "https://serveradmin.ru/nastrojka-postfix-dovecot-postfixadmin-roundcube-dkim-na-debian/"
summary: "Текст посвящён тому, что локальные почтовые серверы остаются актуальными, несмотря на развитие облачных сервисов. Автор описывает установку и настройку почтового сервера на ОС Debian. В качестве основных компонентов используются Postfix, Dovecot, Postfixadmin и Roundcube, а также упоминается DKIM. Материал показывает, что классические почтовые решения и электронная почта по-прежнему востребованы. Запись является перепечаткой статьи, впервые опубликованной на сайте Server Admin."
---

# Настройка Postfix + Dovecot + Postfixadmin + Roundcube + DKIM на Debian

## Краткое содержание

Текст посвящён тому, что локальные почтовые серверы остаются актуальными, несмотря на развитие облачных сервисов.
Автор описывает установку и настройку почтового сервера на ОС Debian.
В качестве основных компонентов используются Postfix, Dovecot, Postfixadmin и Roundcube, а также упоминается DKIM.
Материал показывает, что классические почтовые решения и электронная почта по-прежнему востребованы.
Запись является перепечаткой статьи, впервые опубликованной на сайте Server Admin.

## Полная статья

Несмотря на то, что облачные сервисы год за годом вытесняют локальные, некоторые вещи остаются актуальны по сей день. Я расскажу про установку и настройку почтового сервера на базе Postfix, Dovecot, Postfixadmin, Roundcube на ОС Debian. Последнее десятилетие инфраструктура информационных систем стремительно изменяется, но классические почтовые сервера как и сама электронная почта по прежнему актуальны и востребованы.

**Углубленный онлайн-курс по MikroTik**

Научиться настраивать MikroTik с нуля или систематизировать уже имеющиеся знания можно[на углубленном онлайн-курcе по администрированию MikroTik](https://курсы-по-ит.рф/mikrotik-mtcna?utm_source=serveradminru&utm_medium=cpc&utm_campaign=mikrotik&erid=2SDnjdSUXCF). Автор курcа – сертифицированный тренер MikroTik Дмитрий Скоромнов. Более 40 лабораторных работ по которым дается обратная связь. В три раза больше информации, чем в MTCNA.
Реклама ООО "Курсы по ИТ" ИНН 5029266531

## Цели статьи

- Рассказать о настройке DNS записей для почтового сервера.
- Установить и настроить базовую функциональность почтового сервера на базе Debian 13 с помощью Postfix и Dovecot.
- Настроить панель управления почтовым сервером и аккаунтами пользователей Postfixadmin.
- Настроить веб интерфейс Roundcube, а так же его плагины acl, managesieve и некоторые другие.
- Рассказать о способах борьбы со спамом средствами Postfix.

Данная статья является частью единого цикла статьей про сервер[Debian](https://serveradmin.ru/debian/).

## Введение

Это уже не помню, какая по счёту версия статьи про почтовый сервер на Linux. В ней будет использоваться версия Dovecot 2.4, в которой очень сильно поменялся синтаксис конфигурационного файла. Я не стал оставлять в каком-то виде прошлую версию статьи, чтобы на сайте не было путаницы и задвоения похожего материала. Прошлая статья хоть и похожа на эту, но уже совершенно не подходит для свежих версий систем и пакетов. Сохранил её и выложил в виде[архива](https://serveradmin.ru/files/Debian12+Postfix+Dovecot2.3.zip)со всеми комментариями на момент сохранения.

Изначально меня побудило к написанию очередной статьи по настройке почтового сервера Linux информация о том, что Яндекс больше не будет бесплатно предоставлять услугу по использованию почты со своим доменом. Это очень популярный сервис, которым я сам активно пользовался и всем его рекомендовал. Теперь становится понятно, что скорее всего больше такой услуги не будет в бесплатном виде ни у кого. После Яндекса через пару лет эту услугу сделал платным и сервис Mail Ru, предварительно собрав всех переехавших. Легко читалось это намерение. Я сразу знал, что так и будет.

В связи с этим какая-то часть пользователей подобных сервисов переехала на свой почтовый сервер. Думаю, статья по его настройке будет актуальна и своевременна и сейчас. А у меня, ко всему прочему, есть хороший опыт по установке, настройке и эксплуатации почтовых серверов.

Как я уже сказал, настраивать почтовый сервер буду на ОС на базе Linux, а точнее на Debian 13. За основу будет взят**Postfix**, который присутствует в этой системе. Инструкция получится универсальной, можно использовать и для других дистрибутивов. Все основные конфигурации легко переносятся на разные системы, требуя минимальной правки, в основном путей. Думаю, что вообще без какой-то дополнительной правки эта же статья будет полностью актуальна для Ubuntu.

Я напишу статью на самом что ни на есть реальном примере, без какой-либо правки доменов, IP адресов и прочего, чтобы не ошибиться и показать максимально возможный реальный пример. У меня есть технический домен zeroxzed.ru. Я буду использовать его в своей работе. Почтовый сервер будет иметь имя mail.zeroxzed.ru.

## Проверка хостинга и IP адресов

Любую настройку почтового сервера рекомендую начинать с проверки технической возможности отправлять почту с данного сервера и хостера в целом. Очень многие провайдеры по умолчанию блокируют трафик по исходящему TCP порту 25, который почтовые серверы используют для взаимодействия друг с другом. Прежде чем что-либо настраивать, убедитесь, что этот порт открыт с вашего сервера.

Сделать такую проверку очень просто. Можно с помощью**telnet**обратиться на любой публичный почтовый сервер:

```
# telnet relay.1c.ru 25
Trying 185.12.152.55...
Connected to relay.1c.ru.
Escape character is '^]'.
220 relay.1c.ru ESMTP Wed, 26 Nov 2025 16:36:45 +0300
```

[

![Проверка доступности 25 TCP порта для работы почтового сервера](/news/serveradminru/article-e76b4dce6a903572/image-01.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-02.png)

В этом примере видно, что порт 25 открыт, мы без проблем подключились к серверу, который ждёт от нас дальнейших команд. Если порт закрыт, то вы увидите примерно следующую картину:

```
# telnet relay.1c.ru 25
Trying 185.12.152.55...
```

И на этом всё. Через некоторое время запросы прекратятся и вы увидите ошибку, что подключение не установлено. Даже если у вас закрыт порт 25, скорее всего это не проблема. Просто напишите своему провайдеру в техподдержку, что вы хотите организовать почтовый сервер. Чаще всего провайдер либо сразу открывает отправку, либо попросит пройти какую-то свою внутреннюю процедуру для проверки того, что вы действительно реальный клиент с реальным доменом, а не спамер, который хочет организовать массовую рассылку.

Вторым этапом стоит проверить выделенный IP адрес на наличие его в публичных списках блокировок и репутации. Для этого есть много инструментов. Один из бесплатных вариантов -[hetrixtools.com](https://hetrixtools.com/). Открываем раздел**Blacklist Monitor**и проверяем свой IP адрес.

[

![Проверка IP адреса в чёрных списках](/news/serveradminru/article-e76b4dce6a903572/image-02.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-03.png)

Мне провайдер выделил полностью чистый IP адрес, с которым не должно быть проблем. Если у вас всё в прядке, то переходите к настройке DNS для почтового сервера. Я встречал очень много отзывов на различные проблемы, связанные с хостингом и IP адресом. Если заранее их не проверить, то потом можно потратить очень много времени на отладку и решение проблем. Будет всё настроено правильно, а почта не отправляется. И поди разберись, в чём тут проблема, если ты делаешь это впервые и только учишься.

## Настройка DNS для почтового сервера

Начнем потихоньку подготовку к настройке нашего почтового сервера. Первым делом нужно подготовить DNS записи. Это очень важно. Без правильной наcтройки DNS нормально работать сервер не будет. Вернее, работать то он будет, но ваша почта может улетать в спам у принимающей стороны, либо полностью отклоняться. Неправильная настройка DNS всегда прибавляет очень много баллов для любого антиспам фильтра.

Для написания этой статьи, я сделал поддомен mail.zeroxzed.ru. Для работы почты нет никакой разницы, используется домен первого, второго или третьего уровня. Почтовые ящики у меня будут вида user1@zeroxzed.ru. Внешний IP адрес, на котором будет работать сервер - 212.193.57.223.

Для подготовки к настройке почтового сервера вам нужно добавить следующие DNS записи в панели управления DNS вашего домена:

| Имя | Тип | Значение |
| --- | --- | --- |
| mail.zeroxzed.ru | A | 212.193.57.223 |
| @ | MX | mail.zeroxzed.ru. |

Подробнее о типах DNS записей читайте на[википедии](https://ru.wikipedia.org/wiki/%D0%A2%D0%B8%D0%BF%D1%8B_%D1%80%D0%B5%D1%81%D1%83%D1%80%D1%81%D0%BD%D1%8B%D1%85_%D0%B7%D0%B0%D0%BF%D0%B8%D1%81%D0%B5%D0%B9_DNS). У меня в итоге получилось вот так:

[

![DNS записи для почтового сервера](/news/serveradminru/article-e76b4dce6a903572/image-03.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-01.png)

Но это не всё. К сожалению, указанных DNS записей недостаточно. Для вашего внешнего IP адреса должна быть прописана обратная зона - PTR. В обратной зоне IP адресу ставится в соответствие доменное имя. То есть вашему IP адресу должно соответствовать доменное имя mail.zeroxzed.ru. Это идеальная ситуация, когда обратная зона соответствует имени почтового сервера и названию домена, который он обслуживает. Не всегда это возможно. Например, один почтовый сервер может обслуживать множество доменов. Важно, чтобы PTR зона была прописана и соответствовала реально существующей DNS записи без признаков динамического назначения имён.

Настройками обратной зоны во многих случаях вы не можете управлять напрямую. Иногда это возможно через панель управления провайдера, если это какой-то VPS или выделенный сервер. Вот пример собственноручной настройки PTR:

[

![PTR запись для почтового сервера](/news/serveradminru/article-e76b4dce6a903572/image-04.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-04.png)

В остальных случаях нужно обратиться к вашему провайдеру, который выделил вам внешний IP и сказать ему, что вы хотите настроить почтовый сервер и вам необходимо прописать обратную зону с именем mail.zeroxzed.ru на внешний IP адрес. Это обычная просьба, и техподдержка без проблем сделает то, что вы просите, если они предоставляют такую услугу. Чаще всего провайдеры, работающие с юридическими лицами, такую услугу предоставляют. Домовые провайдеры для физиков - нет, хотя и бывают исключения.

Проверить обратную зону для домена можно в консоли сервера с помощью команды**host**. Вот пример для моего внешнего IP 212.193.57.223:

```
# host 212.193.57.223
223.57.193.212.in-addr.arpa domain name pointer mail.zeroxzed.ru.
```

Выделенное является PTR записью. В идеале тут должен быть прописан не просто какой-то домен, а полное DNS имя вашего**почтового**сервера. Вот в итоге что получилось у меня со всеми необходимыми DNS записями:

[

![Правильно настроенные DNS записи почтового сервера](/news/serveradminru/article-e76b4dce6a903572/image-05.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-05.png)

Для примера ещё приведу полностью корректно настроенные DNS записи.

[

![Проверка DNS](/news/serveradminru/article-e76b4dce6a903572/image-06.png)

](https://serveradmin.ru/wp-content/uploads/2022/12/debian-postfix-dovecot-02.png)

Тут тоже полное соответствие всех записей.**MX**запись соответствует**A**записи, а IP адрес**A**записи резолвится в доменное имя, указанное в**MX**. Вам в идеале нужно сделать так же.

Иногда люди путают, называют, к примеру, почтовый сервер mail.site.ru, а обратную зону делают просто site.ru или srv.site.ru. Вроде всё правильно, домен один и тот же, но соответствие не полное. Если есть возможность, сделайте полное соответствие всех записей - A, MX, PTR. Очень много спам фильтров используют эти записи для проверки домена отправителя на полное соответствие. Отклонение от идеала добавляет лишние баллы, но не означает, что почта не будет доставлена.

Для непосредственно работы почтового сервера обратная зона не нужна. Если у вас тестовый сервер, и вы просто тренируетесь в настройках, можно обойтись без обратной зоны. Почта будет ходить. Но если вы готовите рабочий сервер, то обратную зону сделайте обязательно. Без нее некоторые серверы вообще не будут принимать вашу почту, а большинство будут либо считать спамом, либо накидывать очень много балов в своих спам фильтрах. Это важный элемент в борьбе со спамом, поэтому без обратной записи для принимающей стороны вы по умолчанию становитесь спамером и вам придется доказывать, что вы на самом деле не он.

## Подготовка сервера

После того, как закончите с DNS, можно приступать к настройке самого сервера. Рекомендую воспользоваться услугами провайдера[Selectel](https://serveradmin.ru/hoster-selectel). У него помимо очень дешёвых выделенных серверов, есть в том числе и бесплатная услуга по хостингу DNS. Я обычно там заказываю выделенный сервер, устанавливаю туда[Proxmox](https://serveradmin.ru/ustanovka-i-nastroyka-proxmox/), создаю виртуальные машины. И одну из них использую под почтовый сервер.

Если у вас еще не настроен сервер с Debian, рекомендую мои материалы на эту тему:

- [Установка Debian на сервер](https://serveradmin.ru/kak-skachat-i-ustanovit-debian-10-buster/)
- [Базовая настройка Debian после установки](https://serveradmin.ru/debian-nastroyka-servera/)

Отдельно потратьте время на[настройку iptables](https://serveradmin.ru/debian-nastroyka-servera/#Nastrojka_firewall_iptables_v_Debian),[nftables](https://serveradmin.ru/bazovye-nastrojki-nftables-dlya-veb-servera-na-debian/)или любого другого файрвола. Я не буду касаться этого вопроса в данной статье. Удобнее, когда всё по отдельности рассказано и описано с должной глубиной. Сваливать всё в одну кучу не хочется. Если вы только учитесь настраивать почтовые сервера, то firewall можно вообще отключить, чтобы лишний раз не мешал. Пока вы не захотите настроить какие-то ограничения доступа к серверу или заблокировать переборщиков паролей, файрвол вам не будет нужен. Настройте сначала почтовый сервер, убедитесь, что он нормально работает. А потом уже настраивайте ограничения.

Из обязательных предварительных настроек, которые наиболее важны, отмечу здесь следующие:

- Настройка имени сервера.
- Настройка времени и часового пояса.

Всё остальное делайте по желанию.

```
# hostnamectl set-hostname mail.zeroxzed.ru
# timedatectl set-timezone Europe/Moscow
# timedatectl timesync-status
```

В выводе последней команды посмотрите, чтобы время синхронизировалось. По умолчанию это обычно уже настроено в Debian, но может не быть доступа к серверам времени. Тогда их нужно будет заменить.

[

![Статус синхронизации времени на сервере](/news/serveradminru/article-e76b4dce6a903572/image-07.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-06.png)

После этого перезагрузите сервер:

```
# reboot
```

## Установка Postfixadmin

Начнем с установки и настройки панели управления почтовым сервером Postfix —**Postfixadmin**. Без него начинать что-то делать неудобно, так как управлять пользователями, ящиками, алиасами будет нечем. По своей сути Postfixadmin - набор php скриптов для управления записями в базе данных MySQL, которую использует сервер Postfix во время своей работы. Соответственно, для работы Postfixadmin нам нужен классический веб сервер с php.

Подробно на настройке веб сервера я останавливаться не буду. Быстро установим всё необходимое для дальнейшей работы, в том числе и веб интерфейса с TLS сертификатами. Привожу только команды, с минимумом комментариев. Вы можете настроить любой веб сервер привычным вам образом: из пакетов, в отдельных базовых контейнерах или какой-то готовый проект в docker compose. Либо можете взять готовый[docker контейнер с postfixadmin](https://github.com/postfixadmin/docker). Я привожу универсальный пример веб сервера с установкой из пакетов.

Установка Angie:

```
# apt update
# apt install ca-certificates curl
# curl -o /etc/apt/trusted.gpg.d/angie-signing.gpg https://angie.software/keys/angie-signing.gpg
# echo "deb https://download.angie.software/angie/$(. /etc/os-release && echo "$ID/$VERSION_ID $VERSION_CODENAME") main" | tee /etc/apt/sources.list.d/angie.list > /dev/null
# apt update
# apt install angie
```

Перейдите по IP адресу сервера и убедитесь, что Angie установлен и запущен.

[

![Веб сервер Angie](/news/serveradminru/article-e76b4dce6a903572/image-08.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-07.png)

Установка php-fpm со всеми модулями, которые нам понадобятся во время настройки по данному руководству:

```
# apt install php-cli php-mysql php-json php-gd php-ldap php-odbc php-common php8.4-opcache php-pear php-xml php-xmlrpc php-mbstring php-snmp php-soap php-zip php-sqlite3 php php-fpm
```

Настроим запуск php-fpm под пользователем angie. Для этого в файле*/etc/php/8.4/fpm/pool.d/www.conf*меняем параметры:

```
user = angie
group = angie
listen.owner = angie
listen.group = angie
```

Запускаем php-fpm:

```
# systemctl enable --now php8.4-fpm
```

Установка MariaDB:

```
# apt install mariadb-server
# systemctl enable mariadb
# mariadb-secure-installation
```

Для простоты откажитесь от unix_socket authentication и задайте пароль root. Не хочется в этой статье останавливаться на настройке MariaDB.

Предлагаю сразу поставить Phpmyadmin, с ним удобно работать с базой. В нашем случае все пользователи будут храниться в MySQL, иногда может понадобиться туда заглянуть.

```
# apt install phpmyadmin
```

По умолчанию у Phpmyadmin нет готовой настройки под Angie, поэтому настроим вручную. Сделаем символьную ссылку и выдадим необходимые разрешения:

```
# ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
# chown angie:angie /var/lib/phpmyadmin/tmp
```

Приведём стандартную конфигурацию Angie*/etc/angie/http.d/default.conf*к следующему виду:

```
server {
    listen       80 default_server;
    server_name  _;
    root         /var/www/html;
    index  index.html index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        fastcgi_pass   unix:/run/php/php8.4-fpm.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

}
```

Проверяем и перезапускаем Angie:

```
# angie -t
# angie -s reload
```

При переходе по адресу http://ip-сервера/phpmyadmin вы должны попадать в интерфейс панели управления.

[

![PHPMyAdmin](/news/serveradminru/article-e76b4dce6a903572/image-09.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-08.png)

Можно зайти под учётной записью root. Если вы настроили веб сервер на внешнем интерфейсе, то оставлять Phpmyadmin доступным для всех крайне нежелательно. Нужно ограничить доступ тем или иным способом. Можно по IP, либо паролем. Закроем паролем:

```
# apt install apache2-utils
# htpasswd -c /etc/angie/.htpasswd pmauser01
```

Создали пользователя pmauser01. Теперь включим аутентификацию в Angie. Добавляем новые параметры.

```
server {
    listen       80 default_server;
    server_name  _;
    root         /var/www/html;
    index  index.html index.php;
    auth_basic "Administrator’s Area";
    auth_basic_user_file /etc/angie/.htpasswd;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        fastcgi_pass   unix:/run/php/php8.4-fpm.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

}
```

```
# angie -s reload
```

Теперь наш веб сервер скрыт от посторонних глаз. Пригодится и на будущее, чтобы закрыть доступ к панели управления почтовым сервером.

Устанавливаем Postfixadmin. Он есть в базовых репозиториях Debian, но старой версии. На момент написания статьи уже вышла версия 4, а в репозитории была до сих пор третья. Поэтому скачаем и установим свежую версию вручную. Адрес для загрузки возьмём в[репозитории на github](https://github.com/postfixadmin/postfixadmin/releases). Попутно установим один необходимый нам пакет. Если же вас устроит более старая версия, то устанавливайте её через apt.

```
# apt install acl
# wget https://github.com/postfixadmin/postfixadmin/archive/refs/tags/v4.0.1.tar.gz
# tar xzvf v4.0.1.tar.gz
# mv postfixadmin-4.0.1 /var/www/html/postfixadmin
# chown -R angie:angie /var/www/html/postfixadmin
```

Копируем дефолтный конфиг postfixadmin для того, чтобы вносить туда свои изменения.

```
# cp /var/www/html/postfixadmin/config.inc.php /var/www/html/postfixadmin/config.local.php
```

Убедитесь, что файл config.local.php содержит следующие исправленные параметры.

```
<?php
$CONF['configured'] = true;
$CONF['default_language'] = 'ru';
$CONF['database_type'] = 'mysqli';
$CONF['database_host'] = 'localhost';
$CONF['database_user'] = 'postfix';
$CONF['database_password'] = 'password';
$CONF['database_name'] = 'postfix';
$CONF['admin_email'] = 'root@zeroxzed.ru';
$CONF['encrypt'] = 'md5crypt';
$CONF['domain_path'] = 'YES';
$CONF['domain_in_mailbox'] = 'YES';
$CONF['transport_default'] = 'virtual';
$CONF['show_footer_text'] = 'YES';
$CONF['footer_text'] = 'Return to http://212.193.57.223/padmin/public/';
$CONF['footer_link'] = 'http://212.193.57.223/padmin/public/';
$CONF['default_aliases'] = array (
 'abuse' => 'root',
 'hostmaster' => 'root',
 'postmaster' => 'root',
 'webmaster' => 'root'
);
```

Это не все параметры, которые стоит изменить, а список минимально необходимых, с которыми можно нормально управлять почтовым сервером. Посмотрите внимательно все параметры и измените по своим потребностям. Они хорошо прокомментированы и в целом понятны.

Обращаю внимание на выделенный параметр. Он указывает на то, в каком виде хранить пароли пользователей в базе данных. Конечно, хранить обычным текстом без шифрования это дурной тон и может быть опасно. Я указал хранение в шифрованном виде. Но если мы говорим о небольшой компании без публичного доступа к серверу, можно использовать нешифрованные пароли. Для этого указываем значение параметра**cleartext**. Я сам так часто делаю просто из соображений удобства. Объясню, в чём удобство.

К примеру, у пользователя несколько устройств подключены к почте и он забыл свой пароль. Админ при создании почему-то тоже его никуда не записал, или забыл, или потерял. Вам придется сбросить пароль и перенастроить все устройства. Если же у вас пароль хранится в открытом виде, вы просто смотрите в базу и говорите пользователю пароль. Пароли в открытом виде удобно просто выгрузить дампом из базы, если понадобится кому-то все учётки передать. Но тут как посмотреть :) С одной стороны плюс, с другой минус - кто-то очень просто может спереть все ваши пароли. В общем, тут от ситуации зависит, решайте сами, как вам удобнее хранить пароли.

У меня распространены ситуации, когда я удаленно администрирую сервера, а на месте поддержка работает с пользователями. Чаще всего это не очень аккуратные и ответственные люди, иначе они бы работали с серверами. Они часто забывают записать пароль, путают что-то и т.д. В итоге, когда один человек увольняется и приходит другой, оказывается, что найти пароли на некоторые ящики просто невозможно. Тут очень выручает возможность посмотреть пароль в базе. Я новому админу либо пароль говорю, либо весь дамп сразу отдаю, пусть работает.

Если вы сами работаете с сервером и всё аккуратно ведёте, записываете, например, в KeePass все пароли от почтовых ящиков, то смело шифруйте все пароли, так будет спокойнее.

Последние 2 параметра**domain_path**и**domain_in_mailbox**указывайте по своему усмотрению. В файле конфигурации в комментариях расписано, за что они отвечают и в чём отличие. Мне кажется, удобно хранить директории именно в таком виде, как я указал. Получится следующий путь до ящика, если у вас архив почты будет жить, к примеру, в директории /mnt/mail —/mnt/mail/zeroxzed.ru/root@zeroxzed.ru. Если доменов будет несколько, в такой иерархии удобно работать.

В базе данных Mysql необходимо создать указанные выше в конфигурации пользователя и базу postfix. Если устанавливали Phpmyadmin, то можете это сделать там.

С параметрами разобрались. Сохраняем конфигурацию. Ещё один маленький нюанс. Для работы панели управления нужна директория*templates_c*с правами на запись для web сервера. Сделаем ее и дадим права.

```
# mkdir /var/www/html/postfixadmin/templates_c
# chown -R angie:angie /var/www/html/postfixadmin/templates_c
```

Запускаем установочный скрипт, который скачает composer и установит необходимые зависимости:

```
# cd /var/www/html/postfixadmin
# export COMPOSER_ALLOW_SUPERUSER=1
# /bin/bash install.sh
```

[

![Установка Postfixadmin](/news/serveradminru/article-e76b4dce6a903572/image-10.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-09.png)

Скрипт должен отработать без ошибок и всё установить. У меня сначала были ошибки. Не хватало некоторых пакетов c php в системе, плюс, надо было разрешить работу от root. Пакеты я установил и уже добавил их список, который использовался ранее. Но, возможно, в следующих версиях зависимостей будет больше и вам придётся их разрешить.

Идем по адресу http://ip-сервера/postfixadmin/public/setup.php и начинаем установку Postfixadmin. Первым делом идет проверка всех необходимых для установки и работы компонентов. Для продолжения установки у вас должна быть такая картинка.

[

![Создание пароля установки](/news/serveradminru/article-e76b4dce6a903572/image-11.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-10.png)

Нам необходимо указать пароль установки, на основе которого будет сгенерирован hash, который нужно будет добавить в*config.local.php*. Для этого в форму**Generate setup_password**введите два раза пароль, который обязательно должен содержать буквы и 2 цифры.

После генерации пароля, под формой появится строка для конфигурационного файла:

```
$CONF['setup_password'] = '$2y$10$BGiyhSLns.dyd1mPT68xR.nTSdhcl2PPA15EzFrGZKptfGMnyEBPi';
```

Её нужно добавить в*config.local.php*в директории postfixadmin. Достаточно заменить строку, которая там есть по умолчанию со значением changeme.

После того, как это сделаете, обновите страницу в браузере. Информация там изменится. У вас не должно быть замечаний к конфигурации, кроме предупреждений по поводу PostgreSQL, так как она нам не нужна и мы её не настраивали. А все остальные проверки должны быть с зелёными индикаторами. Если есть какие-то ошибки, то исправьте их. Обычно забывают или неправильно указывают параметры доступа к MySQL, копируя конфигурацию из статьи.

[

![Проверка сервера для postfixadmin](/news/serveradminru/article-e76b4dce6a903572/image-12.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-11.png)

В самом низу страницы, используя созданный ранее пароль установки, нужно добавить учётную запись администратора панели управления.

[

![Добавление администратора в postfixadmin](/news/serveradminru/article-e76b4dce6a903572/image-13.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-12.png)

Используя созданную учётную запись, можно авторизоваться в панели управления. Для этого перейдите по ссылке http://ip-сервера/postfixadmin/public/login.php

[

![Аутентификация в интерфейсе управления почтовым сервером](/news/serveradminru/article-e76b4dce6a903572/image-14.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-13.png)

После входа увидите панель управления.

[

![Внешний вид postfixadmin](/news/serveradminru/article-e76b4dce6a903572/image-15.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-14.png)

Сразу же добавим наш домен в список почтовых доменов. Идем в раздел**Список доменов -> Новый домен**и добавляем свой домен. Я обычно не ставлю никаких ограничений.

[

![Добавление почтового домена](/news/serveradminru/article-e76b4dce6a903572/image-16.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-15.png)

При создании домена были добавлены стандартные алиасы, получателя для которых мы указали ещё в конфигурации - ящик root@zeroxzed.ru. Создание таких алиасов - требование стандартов, но по факту, кроме спама, вы скорее всего ничего не будете получать по этим адресам. Так что их создание оставляйте на свое усмотрения. Я обычно их не делаю, так как ящик для этих алиасов всё равно никто не читает.

Далее создадим почтовый ящик администратора - admin@zeroxzed.ru. Для этого идем в раздел**Обзор -> Создать ящик**и заполняем поля.

[

![Создание почтового ящика](/news/serveradminru/article-e76b4dce6a903572/image-17.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-16.png)

Вы получите ошибку, что невозможно отправить сообщение. Это нормально, так как непосредственно почтового сервера у нас ещё нет, только панель управления для него. Сам ящик на диске создан тоже не будет, но запись в базе данных появится. Это можно проверить через Phpmyadmin.

[

![Запись о почтовом ящике в mysql базе](/news/serveradminru/article-e76b4dce6a903572/image-18.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-17.png)

Как вы видите, пароль зашифрован. На этом установку и настройку Postfixadmin завершаем. Интерфейс для управления почтовым сервером мы подготовили. Теперь можно заняться непосредственно настройкой Postfix.

Сердце нашего почтового сервера на Linux - Postfix. В дистрибутиве Debian в минимальной установке он по умолчанию отсутствует. Так что сначала устанавливаем Postfix.

```
# apt install postfix postfix-mysql
```

Во время установки будет задан вопрос по поводу назначения этого сервера. Тут не важно, что выбрать. Всё равно конфигурацию мы полностью составим сами. Так что можете оставить тот вариант, что будет по умолчанию. Имя сервера укажите как в DNS записи. В моём случае - mail.zeroxzed.ru.

Приводим конфиг postfix*/etc/postfix/main.cf*к следующему виду, предварительно сохранив оригинальный (рекомендую всегда так делать).

```
# cp /etc/postfix/main.cf /etc/postfix/main.cf.orig
```

```
soft_bounce = no
queue_directory = /var/spool/postfix
command_directory = /usr/sbin
daemon_directory = /usr/lib/postfix/sbin
data_directory = /var/lib/postfix
mail_owner = postfix

myhostname = mail.zeroxzed.ru
mydomain = zeroxzed.ru
myorigin = $myhostname

inet_interfaces = all
inet_protocols = ipv4

mydestination = localhost.$mydomain, localhost
unknown_local_recipient_reject_code = 550
mynetworks = 127.0.0.0/8

alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases

smtpd_banner = $myhostname ESMTP $mail_name

debug_peer_level = 2
# Строки с PATH и ddd должны быть с отступом в виде табуляции от начала строки
debugger_command =
    PATH=/bin:/usr/bin:/usr/local/bin:/usr/X11R6/bin
    ddd $daemon_directory/$process_name $process_id & sleep 5

setgid_group = postdrop
html_directory = no

relay_domains = mysql:/etc/postfix/mysql/relay_domains.cf
virtual_alias_maps = mysql:/etc/postfix/mysql/virtual_alias_maps.cf,
 mysql:/etc/postfix/mysql/virtual_alias_domain_maps.cf
virtual_mailbox_domains = mysql:/etc/postfix/mysql/virtual_mailbox_domains.cf
virtual_mailbox_maps = mysql:/etc/postfix/mysql/virtual_mailbox_maps.cf

smtpd_discard_ehlo_keywords = etrn, silent-discard
smtpd_forbidden_commands = CONNECT GET POST
broken_sasl_auth_clients = yes
smtpd_delay_reject = yes
smtpd_helo_required = yes
smtp_always_send_ehlo = yes
disable_vrfy_command = yes

smtpd_helo_restrictions = permit_mynetworks,
 permit_sasl_authenticated,
 reject_non_fqdn_helo_hostname,
 reject_invalid_helo_hostname

smtpd_data_restrictions = permit_mynetworks,
 permit_sasl_authenticated,
 reject_unauth_pipelining,
 reject_multi_recipient_bounce,

smtpd_sender_restrictions = permit_mynetworks,
 permit_sasl_authenticated,
 reject_non_fqdn_sender,
 reject_unknown_sender_domain

smtpd_recipient_restrictions = permit_mynetworks,
 permit_sasl_authenticated,
 reject_non_fqdn_recipient,
 reject_unknown_recipient_domain,
 reject_multi_recipient_bounce,
 reject_unauth_destination,

smtp_tls_security_level = may
smtp_tls_loglevel = 1
smtpd_tls_security_level = may
smtpd_tls_loglevel = 1
smtpd_tls_received_header = yes
smtpd_tls_session_cache_timeout = 3600s
smtp_tls_session_cache_database = btree:$data_directory/smtp_tls_session_cache
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
smtpd_tls_key_file = /etc/postfix/certs/key.pem
smtpd_tls_cert_file = /etc/postfix/certs/cert.pem
tls_random_source = dev:/dev/urandom
smtpd_tls_mandatory_ciphers = low
smtpd_tls_ciphers = low
smtpd_tls_mandatory_protocols = !SSLv2,!SSLv3
smtp_tls_mandatory_protocols  = !SSLv2,!SSLv3
smtp_tls_ciphers = low
smtp_tls_mandatory_ciphers = low
smtp_tls_protocols = !SSLv2,!SSLv3
smtp_tls_policy_maps = hash:/etc/postfix/tls_policy_maps
# фиксировать в логе имена серверов, выдающих сообщение STARTTLS, поддержка TLS для которых не включена
smtp_tls_note_starttls_offer = yes

# Ограничение максимального размера письма в байтах
message_size_limit = 20000000
smtpd_soft_error_limit = 10
smtpd_hard_error_limit = 15
smtpd_error_sleep_time = 20
anvil_rate_time_unit = 60s
smtpd_client_connection_count_limit = 20
smtpd_client_connection_rate_limit = 30
smtpd_client_message_rate_limit = 30
smtpd_client_event_limit_exceptions = 127.0.0.0/8
smtpd_client_connection_limit_exceptions = 127.0.0.0/8

maximal_queue_lifetime = 1d
bounce_queue_lifetime = 1d

smtpd_sasl_auth_enable = yes
smtpd_sasl_security_options = noanonymous
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/dovecot-auth

# Директория для хранения почты
virtual_mailbox_base = /mnt/mail
virtual_minimum_uid = 1100
virtual_uid_maps = static:1100
virtual_gid_maps = static:1100
virtual_transport = dovecot
dovecot_destination_recipient_limit = 1

sender_bcc_maps = hash:/etc/postfix/sender_bcc_maps
recipient_bcc_maps = hash:/etc/postfix/recipient_bcc_maps

compatibility_level=2
```

Я выделил жирным имя домена и путь для директории с почтовыми ящиками. Не забудьте поменять эти параметры на свои. Сохраняем конфигурационный файл и продолжаем настройку. В таком виде сервер ещё не готов. Нужно теперь создать всё то, что описано в файле конфигурации. Создаем папку для файлов с конфигурацией подключения к MySQL и сами файлы подключения.

```
# mkdir /etc/postfix/mysql && cd /etc/postfix/mysql
```

```
# mcedit relay_domains.cf

hosts = 127.0.0.1:3306
user = postfix
password = password
dbname = postfix
query = SELECT domain FROM domain WHERE domain='%s' and backupmx = '1'
```

```
# mcedit virtual_alias_domain_maps.cf

hosts = 127.0.0.1:3306
user = postfix
password = password
dbname = postfix
query = SELECT goto FROM alias,alias_domain WHERE alias_domain.alias_domain = '%d' and alias.address = CONCAT('%u', '@', alias_domain.target_domain) AND alias.active = 1
```

```
# mcedit virtual_alias_maps.cf

hosts = 127.0.0.1:3306
user = postfix
password = password
dbname = postfix
query = SELECT goto FROM alias WHERE address='%s' AND active = '1'
```

```
# mcedit virtual_mailbox_domains.cf

hosts = 127.0.0.1:3306
user = postfix
password = password
dbname = postfix
query = SELECT domain FROM domain WHERE domain='%s' AND backupmx = '0' AND active = '1'
```

```
# mcedit virtual_mailbox_maps.cf

hosts = 127.0.0.1:3306
user = postfix
password = password
dbname = postfix
query = SELECT maildir FROM mailbox WHERE username='%s' AND active = '1'
```

Создадим файл*tls_policy_maps*, с помощью которого можно вручную задавать версию TLS протокола для общения серверов, либо полностью отключать его. Это иногда бывает нужно, если подключаешься к очень старому серверу, который не поддерживает современные протоколы. Приходится использовать нешифрованное подключение.

```
# mcedit /etc/postfix/tls_policy_maps
```

```
93.175.56.122		none
77.221.87.91		encrypt protocols=TLSv1
```

```
# postmap /etc/postfix/tls_policy_maps
```

Также не стоит забывать, что в современных ОС зачастую на уровне всей системы запрещено использование старых протоколов шифрования, таких как TLS 1.0 или SSL 3. Чтобы разрешить их, недостаточно настроек почтового сервера. Надо менять системные настройки. Например, вот так это делается в Centos и производных rpm системах -[Centos 8 и TLS 1.0 и 1.1](https://serveradmin.ru/centos-8-tls-v1_0/).

Продолжаем настройку почтового сервера Postfix. Редактируем файл*/etc/postfix/master.cf*. Нам надо добавить строки, касающиеся настройки Submission для того, чтобы почтовый сервер работал на 587 порту. Смартфоны очень часто при настройке используют этот порт по умолчанию, где-то даже без возможности изменить эту настройку. Приводим секцию, отвечающую за эту работу, к следующему виду.

```
# cp /etc/postfix/master.cf /etc/postfix/master.cf.orig
```

```
submission inet n       -       n       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_tls_auth_only=yes
  -o smtpd_reject_unlisted_recipient=no
  -o smtpd_recipient_restrictions=
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
```

Обращаю внимание на пробел в начале строки, начиная со второй. Его надо обязательно оставить. Добавляем еще настройки для того, чтобы наш сервер поддерживал протокол SSL/TLS и слушал порт 465

```
smtps inet n - n - - smtpd
 -o syslog_name=postfix/smtps
 -o smtpd_tls_wrappermode=yes
 -o smtpd_sasl_auth_enable=yes
 -o smtpd_recipient_restrictions=permit_mynetworks,permit_sasl_authenticated,reject
 -o smtpd_relay_restrictions=permit_mynetworks,permit_sasl_authenticated,defer_unauth_destination
 -o milter_macro_daemon_name=ORIGINATING
```

В этот же файл добавляем еще одну настройку, которая будет указывать Postfix, что доставкой почты у нас будет заниматься Dovecot, который мы настроим следом. Добавляем в master.cf в самый конец.

```
dovecot unix - n n - - pipe
 flags=DRhu user=vmail:vmail argv=/usr/lib/dovecot/deliver -f ${sender} -d ${recipient}
```

Сгенерируем самоподписанные TLS сертификаты для нашего почтового сервера. В настоящее время повсеместно распространены бесплатные сертификаты от Let's Encrypt. Удобнее будет взять сразу их. Позже отдельным пунктом я расскажу, как это сделать. Но так как почтовый сервер может использоваться и в закрытом контуре, и со своими сертификатами, и Let's Encrypt могут отключить или заблокировать. Поэтому чтобы инструкция была универсальной, показываю быструю настройку Postfix на использование своих сертификатов, которые уже указаны в конфигурационном файле.

Создаем директорию и сами сертификаты:

```
# mkdir /etc/postfix/certs
# openssl req -new -x509 -days 3650 -nodes -out /etc/postfix/certs/cert.pem -keyout /etc/postfix/certs/key.pem
```

Для генерации вам зададут несколько вопросов по поводу данных о сертификате. В принципе, можете там писать все, что угодно. Вот мои данные.

[

![Создание самоподписанного сертификата для почтового сервера](/news/serveradminru/article-e76b4dce6a903572/image-19.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-18.png)

Создадим файлы для информации об ящиках, куда будет собираться вся входящая и исходящая почта.

```
# mcedit /etc/postfix/recipient_bcc_maps
```

```
@zeroxzed.ru all_in@zeroxzed.ru
```

```
# mcedit /etc/postfix/sender_bcc_maps
```

```
@zeroxzed.ru all_out@zeroxzed.ru
```

Создаём индексированные базы данных из этих файлов. Это нужно делать каждый раз, после изменения.

```
# postmap /etc/postfix/recipient_bcc_maps /etc/postfix/sender_bcc_maps
```

Теперь создайте два почтовых ящика all_in@zeroxzed.ru и all_out@zeroxzed.ru через Postfixadmin.

Немного поясню по этим ящикам, для чего они нужны. Изначально я их делал, когда пользователи использовали протокол pop3 без сохранения писем на сервере. Это позволяло организовать бэкап всей переписки. Эти ящики очень быстро заполняются и занимают огромный объем, поэтому их обязательно надо чистить. Я просто скриптами регулярно собирал всю почту в архивы с именами в виде дат. Если нужно было какое-то письмо найти, то просто распаковывал нужный архив. Пример прямого поиска писем по дате я показывал в статье про[обслуживание почтовой базы](https://serveradmin.ru/ochistka-i-obsluzhivanie-pochtovoy-bazyi-postfix/#i-5).

В случае с imap роль бэкапа отпадает, так как вся почта хранится на сервере. Но эти ящики всё равно бывают полезны, когда пользователь, к примеру, удалил какое-то важное письмо и потом делает вид, что его и не было. Если это письмо пришло только сегодня и еще не успело улететь в бэкап, то кроме записи в логах об этом письме, вы не увидите само содержимое. А с такими ящиками всё сразу будет понятно, и вопросы отпадут. Последнее применение - служба безопасности. Если у вас есть кто-то, кому положено читать всю переписку, то реализовать эту функциональность можно таким простым способом.

Все основные настройки для Postfix мы сделали. Некоторые из них завязаны на работу с Dovecot, который мы еще не настроили. Поэтому больше Postfix не трогаем, не перезапускаем. Идем настраивать Dovecot - imap сервер нашей почтовой системы.

## Настройка Dovecot

Займемся настройкой dovecot - сервер доставки почты пользователю по протоколам pop3 и imap. Я не вижу причин использовать pop3. Он неудобен по сравнению с imap. Чаще всего pop3 отключаю вовсе. Но это уже на ваше усмотрение. Приведу пример с настройкой обоих протоколов. Помимо основной функциональности по доставке почты, я настрою несколько полезных плагинов. Расскажу о них поподробнее:

- **Sieve**— выполняет фильтрацию почты по заданным правилам в момент локальной доставки на почтовом сервере. Удобство такого подхода в том, что вы один раз можете настроить правило сортировки, и оно будет работать во всех клиентах, которыми вы будете получать почту по imap. Правила создаются, хранятся и исполняются на самом сервере.
- **Acl**— позволяет пользователям расшаривать папки в своем почтовом ящике и предоставлять доступ к этим папкам другим пользователям. Нечасто видел, чтобы эту возможность настраивали и использовали. Думаю, просто по незнанию. По мне так очень удобная и полезная функциональность.

Часто вижу, что люди настраивают плагин**quota**, который позволяет ограничивать максимальный размер почтового ящика. Я лично в своей работе его не использую. Возможно, когда у тебя клиентов сотни и тысячи это имеет значение и надо обязательно настроить ограничение. Когда же ящиков меньше, нет смысла напрягать людей постоянной чисткой. Сейчас диски стоят не так дорого. Мне кажется, проще и дешевле увеличить место на сервере, нежели постоянно беспокоить пользователей необходимостью чистки ящика. Лучше ограничить максимальный размер письма, скажем 20-ю мегабайтами. Тогда сильно забить ящик даже при большом желании быстро не получится. А почта все-таки важный инструмент в работе. Мне кажется, её лучше хранить как можно дольше.

Есть еще один полезный плагин -**expire**, который позволяет удалять устаревшие письма в определенных папках. Например, удалять все письма старше 30-ти дней в корзине и папке со спамом. Но реально пользоваться им не получается по простой причине. Разные почтовые клиенты создают различные папки для корзины и спама. Thunderbird создает папки с латинскими именами trash и spam, Outlook - с русскими, которые на почтовом сервере преобразуются в кодировку UTF7, мобильные клиенты тоже используют разные имена папок. В итоге нет единообразия, плагин полноценно не работает.

Я рассказал об этих плагинах для наводки. Сам их не настраиваю, но если вам захочется реализовать описанные возможности, можете сами разобраться и настроить.

Небольшую теорию я дал, теперь переходим к практике. Устанавливаем необходимые для Dovecot пакеты.

```
# apt install dovecot-imapd dovecot-pop3d dovecot-mysql dovecot-sieve dovecot-managesieved dovecot-lmtpd
```

Изначально конфигурация Dovecot разбита на отдельные сегменты и лежат они в директории*/etc/dovecot/conf.d*. Каждый файл - отдельная функциональность. Мне не нравится прыгать по файлам, поэтому я храню всё в едином общем файле конфигурации*/etc/dovecot/dovecot.conf*. С ним мы и будем работать. Обращаю внимание, что это только вопрос удобства. Можете оставить все файлы отдельности, если вам так привычнее. Приводим конфигурацию к следующему виду.

```
# cp /etc/dovecot/dovecot.conf /etc/dovecot/dovecot.conf.orig
```

```
dovecot_config_version = 2.4.0
dovecot_storage_version = 2.4.0

listen = *

mail_plugins = acl
protocols = imap pop3 sieve lmtp

mail_uid = 1100
mail_gid = 1100

first_valid_uid = 1100
last_valid_uid = 1100

auth_verbose = yes
log_path = /var/log/dovecot/main.log
info_log_path = /var/log/dovecot/info.log
log_debug = category=ssl
log_debug=category=mail
mail_plugins {
   notify = yes
   mail_log = yes
}

ssl_min_protocol = TLSv1
ssl_cipher_list = ALL:!SSLv2:!SSLv3:!eNULL:!aNULL
ssl_server_prefer_ciphers = server
ssl_server_cert_file = /etc/postfix/certs/cert.pem
ssl_server_key_file = /etc/postfix/certs/key.pem

auth_allow_cleartext = no

mail_driver = maildir
mail_path = /mnt/mail/%{user|domain}/%{user|username}@%{user|domain}
mail_home = /mnt/mail/%{user|domain}/%{user|username}@%{user|domain}
mailbox_list_layout = fs

auth_default_domain = zeroxzed.ru

auth_mechanisms = PLAIN LOGIN

service auth {
 unix_listener /var/spool/postfix/private/dovecot-auth {
 user = postfix
 group = postfix
 mode = 0666
 }
unix_listener auth-master {
 user = vmail
 group = vmail
 mode = 0666
 }

unix_listener auth-userdb {
 user = vmail
 group = vmail
 mode = 0660
 }
}

service lmtp {
 unix_listener /var/spool/postfix/private/dovecot-lmtp {
 user = postfix
 group = postfix
 mode = 0600
 }
}

sql_driver = mysql

mysql localhost {
 user = postfix
 password = password
 dbname = postfix
}

passdb sql {
 default_password_scheme = CRYPT
 query = SELECT username as user, '%{user|domain}' as domain, password, '/mnt/mail/%{user|domain}/%{user|username}@%{user|domain}' as userdb_home, 'maildir:/mnt/mail/%{user|domain}/%{user|username}@%{user|domain}' as userdb_mail, 1100 as userdb_uid, 1100 as userdb_gid FROM mailbox WHERE username = '%{user}' AND domain = '%{user|domain}' AND active = '1'
}

userdb sql {
 query = SELECT '/mnt/mail/%{user|domain}/%{user|username}@%{user|domain}' as home, 'maildir:/mnt/mail/%{user|domain}/%{user|username}@%{user|domain}' as mail, 1100 AS uid, 1100 AS gid, concat('dirsize:storage=',  quota) AS quota FROM mailbox WHERE username = '%{user}' AND domain = '%{user|domain}' AND active = '1'
}

auth_master_user_separator = *

auth_socket_path = /var/run/dovecot/auth-master

acl_driver = vfile
acl_sharing_map {
 dict file {
 path = /mnt/mail/shared-folders/shared-mailboxes.db
 }
}

sieve_script personal {
  driver = file
  path = ~/.sieve
  active_path = ~/.dovecot.sieve
}

protocol lda {
 mail_plugins {
  acl = yes
  sieve = yes
 }
 auth_socket_path = /var/run/dovecot/auth-master
 deliver_log_format = from=%{from} msgid=%{msgid}: %{message} (subject=%{subject} size=%{size})
 log_path = /var/log/dovecot/lda-errors.log
 info_log_path = /var/log/dovecot/lda-deliver.log
 lda_mailbox_autocreate = yes
 lda_mailbox_autosubscribe = yes
# postmaster_address = root
}

protocol lmtp {
 info_log_path = /var/log/dovecot/lmtp.log
 mail_plugins = quota sieve
 postmaster_address = postmaster
 lmtp_save_to_detail_mailbox = yes
 recipient_delimiter = +
}

protocol imap {
 mail_plugins {
  imap_acl = yes
 }
 imap_client_workarounds = tb-extra-mailbox-sep
 mail_max_userip_connections = 30
}

protocol pop3 {
 mail_plugins = $mail_plugins
 pop3_client_workarounds = outlook-no-nuls oe-ns-eoh
 pop3_uidl_format = %{uid | hex(8)}%{uidvalidity | hex(8)}
 mail_max_userip_connections = 10
}

service imap-login {
  process_limit = 100
  client_limit = 10
 }

service pop3-login {
  process_limit = 100
  client_limit = 10
 }

service managesieve-login {
 inet_listener sieve {
 port = 4190
 }
}

service stats {
    unix_listener stats-reader {
        user = vmail
        group = vmail
        mode = 0660
    }

    unix_listener stats-writer {
        user = vmail
        group = vmail
        mode = 0660
    }
}

namespace inbox {
 type = private
 separator = /
 prefix =
 inbox = yes

 mailbox Sent {
 auto = subscribe
 special_use = \Sent
 }
 mailbox "Sent Messages" {
 auto = no
 special_use = \Sent
 }
 mailbox "Sent Items" {
 auto = no
 special_use = \Sent
 }
 mailbox Drafts {
 auto = subscribe
 special_use = \Drafts
 }
 mailbox Trash {
 auto = subscribe
 special_use = \Trash
 }
 mailbox "Deleted Messages" {
 auto = no
 special_use = \Trash
 }
 mailbox Junk {
 auto = subscribe
 special_use = \Junk
 }
 mailbox Spam {
 auto = no
 special_use = \Junk
 }
 mailbox "Junk E-mail" {
 auto = no
 special_use = \Junk
 }
 mailbox Archive {
 auto = no
 special_use = \Archive
 }
 mailbox Archives {
 auto = no
 special_use = \Archive
 }
}

namespace shared {
 type = shared
 separator = /
 prefix = Shared/$user/
 list = children
 subscriptions = yes
 mail_driver = maildir
 mailbox_list_layout = fs
 mail_path = %{owner_home}
 mail_index_path = ~/Shared/%{owner_user}
}
```

Обращаю внимание на параметр шифрования паролей в запросах к базе данных:

```
default_password_scheme = CRYPT
```

Если пароли не зашифрованы, то параметр будет**PLAIN**.

Ещё привлеку ваше внимание к параметру**ssl_cipher_list**. С ним довольно муторно разбираться. Dovecot использует обозначения шифров из openssl. Посмотреть их можно в[документации](https://docs.openssl.org/master/man1/openssl-ciphers/#tls-v10-cipher-suites). Шифры должны соответствовать минимальному протоколу в параметре**ssl_min_protocol**. Например, по умолчанию в конфигурации Dovecot в шифрах указан запрет на использование шифров на основе MD5. В конфигурации на это указывает запись !MD5. При этом если вы разрешите протокол TLS v1.0, часть его шифров не будут работать из-за этой настройки, если не поменяете и её. Я в итоге оставил настройку, которая разрешает все шифры TLS 1.0 и выше:

```
ssl_cipher_list = 'ALL:!SSLv2:!SSLv3:!eNULL:!aNULL'
```

Если вы будете использовать только TLS 1.3, то для него нужно будет указать параметр:

```
ssl_server_prefer_ciphers = client
```

Я рекомендую не жестить с отключением старых версий TLS и шифров. Почтовые сервера - очень консервативная область. Можете получить проблемы с подключением каких-то старых клиентов или устройств. Если у вас нет обоснованных опасений, что кто-то будет перехватывать и расшифровывать вашу почту, то нет смысла тут сильно урезать старые протоколы.

Создаем группу и пользователя с указанными в конфигурации uid 1100. Если у вас уже занят этот uid, то везде замените его на другой или удалите пользователя с uid 1100, если он вам не нужен.

```
# groupadd -g 1100 vmail
# useradd -d /mnt/mail/ -g 1100 -u 1100 vmail
# usermod -a -G dovecot vmail
# mkdir /mnt/mail
# chown vmail:vmail /mnt/mail
```

Создадим директорию и файлы для логов.

```
# mkdir /var/log/dovecot
# cd /var/log/dovecot && touch main.log info.log lda-errors.log lda-deliver.log lmtp.log
# chown -R vmail:dovecot /var/log/dovecot
```

Создаем служебную папку для плагина acl.

```
# mkdir /mnt/mail/shared-folders
# chown -R vmail:vmail /mnt/mail
```

На этом основная настройка почтового сервера на базе Postfix и Dovecot завершена. Можно запускать службы и проверять работу системы. Но перед этим сделаем ещё одну небольшую настройку, которая касается удобства обслуживания почтового сервера. По умолчанию в современных системах все логи перенесены в systemd, что не всегда удобно для отладки. Вернём так же текстовые логи. Для этого достаточно установить пакет с rsyslog:

```
# apt install rsyslog
```

Теперь можно перезапускать службы и проверять их логи:

```
# systemctl restart postfix
# systemctl restart dovecot
# systemctl enable postfix
# systemctl enable dovecot
```

Обе службы должны запуститься без ошибок. Если это не так, то разбирайтесь в чём проблема. Скорее всего где-то ошиблись в конфигурационных файлах. Если у вас другой дистрибутив, то в первую очередь проверяйте все пути и доступы к сокетам. Они могут иметь разные имена и права доступа.

### Настройка Logrotate для Dovecot

Для того, чтобы логи Dovecot не разрослись до бесконечности, их имеет смысл периодически ротировать с помощью logrotate. Для этого создадим файл конфигурации*/etc/logrotate.d/dovecot*следующего содержания:

```
/var/log/dovecot/*.log {
    weekly
    rotate 3
    compress
    missingok
    notifempty
    postrotate
	doveadm log reopen
    endscript
    create 0644 vmail dovecot
}
```

Для логов Postfix этого делать не нужно, так как они обрабатываются с помощью системной службы rsyslog, для которой ротация обычно уже настроена в системе по умолчанию.

## Проверка работы почтового сервера

Самый простой и быстрый способ проверить работу почтового сервера - отправить на него письмо. Я буду отправлять со своего почтового адреса zeroxzed@gmail.com на адрес admin@zeroxzed.ru. Вот что должно быть в логе, если у вас всё правильно настроено и почтовый сервер нормально работает.

```
# tail -f /var/log/mail.log
```

```
2025-11-27T14:34:36.871935+03:00 mail postfix/smtpd[137233]: connect from mail-ej1-f47.google.com[209.85.218.47]
2025-11-27T14:34:37.081352+03:00 mail postfix/smtpd[137233]: Anonymous TLS connection established from mail-ej1-f47.google.com[209.85.218.47]: TLSv1.3 with cipher TLS_AES_128_GCM_SHA256 (128/128 bits) key-exchange x25519 server-signature RSA-PSS (2048 bits) server-digest SHA256
2025-11-27T14:34:37.235569+03:00 mail postfix/smtpd[137233]: 39564411A7: client=mail-ej1-f47.google.com[209.85.218.47]
2025-11-27T14:34:37.242795+03:00 mail postfix/cleanup[137256]: 39564411A7: message-id=<CAHWPLcNnZ2kTzBDYfOTAY8m43Gw+vySfZ2HNfknqqLz0cpFu=w@mail.gmail.com>
2025-11-27T14:34:37.253663+03:00 mail postfix/qmgr[45166]: 39564411A7: from=<zeroxzed@gmail.com>, size=3580, nrcpt=2 (queue active)
2025-11-27T14:34:37.339150+03:00 mail postfix/smtpd[137233]: disconnect from mail-ej1-f47.google.com[209.85.218.47] ehlo=2 starttls=1 mail=1 rcpt=1 bdat=1 quit=1 commands=7
2025-11-27T14:34:37.457744+03:00 mail postfix/pipe[137258]: 39564411A7: to=<admin@zeroxzed.ru>, relay=dovecot, delay=0.26, delays=0.06/0.02/0/0.18, dsn=2.0.0, status=sent (delivered via dovecot service)
2025-11-27T14:34:37.480576+03:00 mail postfix/pipe[137259]: 39564411A7: to=<all_in@zeroxzed.ru>, relay=dovecot, delay=0.28, delays=0.06/0.04/0/0.18, dsn=2.0.0, status=sent (delivered via dovecot service)
2025-11-27T14:34:37.481766+03:00 mail postfix/qmgr[45166]: 39564411A7: removed
```

Пояснять тут нечего, по логу все понятно. Было подключение от сервера mail-ej1-f47.google.com[209.85.218.47] с использованием протокола шифрования и шифров TLSv1.3 with cipher TLS_AES_128_GCM_SHA256.

Письмо было доставлено в указанный ящик. Это можно и в логе Dovecot посмотреть, в lda-deliver.log:

```
Nov 27 14:34:37 lda(admin@zeroxzed.ru)<137260>: Info: Mailbox INBOX: save: uid=6, msgid=<CAHWPLcNnZ2kTzBDYfOTAY8m43Gw+vySfZ2HNfknqqLz0cpFu=w@mail.gmail.com>, size=3584
Nov 27 14:34:37 lda(admin@zeroxzed.ru)<137260>: Info: from=zeroxzed@gmail.com msgid=<CAHWPLcNnZ2kTzBDYfOTAY8m43Gw+vySfZ2HNfknqqLz0cpFu=w@mail.gmail.com>: saved mail to INBOX (subject=Проверка связи size=3584)
Nov 27 14:34:37 lda(all_in@zeroxzed.ru)<137261>: Info: Mailbox INBOX: save: uid=6, msgid=<CAHWPLcNnZ2kTzBDYfOTAY8m43Gw+vySfZ2HNfknqqLz0cpFu=w@mail.gmail.com>, size=3585
Nov 27 14:34:37 lda(all_in@zeroxzed.ru)<137261>: Info: from=zeroxzed@gmail.com msgid=<CAHWPLcNnZ2kTzBDYfOTAY8m43Gw+vySfZ2HNfknqqLz0cpFu=w@mail.gmail.com>: saved mail to INBOX (subject=Проверка связи size=3585)
```

В директории*/mnt/mail*была создана директория с именем домена zeroxzed.ru, а в ней создана папка с именем ящика admin@zeroxzed.ru. В итоге новое, ещё не прочитанное письмо, лежит здесь -*/mnt/mail/zeroxzed.ru/admin@zeroxzed.ru/new*. Можно зайти и прочитать его прямо через консоль. Это обычный файл в текстовом формате.

Параллельно с этим письмо было скопировано в ящик all_in@zeroxzed.ru. Без особой необходимости не рекомендую собирать копию почты. У вас объём постовой базы будет расти с двойной скоростью. Либо отключайте эту возможность в настройках Postfix, либо регулярно очищайте этот ящик.

Директории с почтовыми ящиками создаются в момент получения первого письма. Непрочитанное письмо помещается в директорию**new**в почтовом ящике. После прочтения переносится в**cur**.

Попробуем теперь подключиться к почтовому ящику по imap, прочитать письмо и отправить ответ. Настроим любой почтовый клиент для проверки работы настроенного почтового сервера. Я для этих целей буду использовать**Thunderbird**. Из всех почтовых клиентов мне он нравится больше всего. В основном из-за его портированной версии. Указываем следующие настройки.

[

![Настройка Thunderbird](/news/serveradminru/article-e76b4dce6a903572/image-20.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-19.png)

Клиенты сейчас умные, пытаются подобрать настройки на основе DNS записей и тестовых подключений к серверу. Им, кстати, можно помочь дополнительными DNS записями. Сейчас уже не буду на этом останавливаться, так как материал и так очень большой. Не всегда и не все определяют настройки правильно. У меня получились две ошибки: имя SMTP сервера почему-то неправильное, и имя пользователя просто admin, а надо полноценный почтовый ящик.

[

![Автоматическая конфигурация в Thunderbird](/news/serveradminru/article-e76b4dce6a903572/image-21.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-20.png)

Исправил это:

[

![Ручная настройка Thunderbird](/news/serveradminru/article-e76b4dce6a903572/image-22.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-21.png)

Так как мы используем самоподписанный сертификат, автоматическая проверка будет завершаться ошибкой. Это нормально, потому что наш сертификат не добавлен в локальное хранилище Thunderbird. Чтобы это произошло, достаточно нажать ещё раз на пункт**Дополнительная настройка**. У вас появится информационное сообщение.

[

![Создание учётной записи без проверок](/news/serveradminru/article-e76b4dce6a903572/image-23.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-22.png)

После согласия, данная учётная запись будет добавлена без проверки. И тут же при попытке открыть этот ящик по imap, вы получите предупреждение на то, что сертификату нет доверия.

[

![Предупреждение об отсутствии доверия к сертификату](/news/serveradminru/article-e76b4dce6a903572/image-24.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-23.png)

Нажимаем на**Дополнительные сведения**и видим предложение вручную настроить доверие.

[

![Добавление сертификата imap сервера в исключения](/news/serveradminru/article-e76b4dce6a903572/image-25.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-24.png)

Соглашаемся и попадаем в свой почтовый ящик.

[

![Почтовый ящик в Thunderbird](/news/serveradminru/article-e76b4dce6a903572/image-26.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-25.png)

Видим тут все свои тестовые сообщения, которые отправляли ранее. Ещё одно такое же предупреждение вы получите во время отправки первого сообщения.

[

![Добавление сертификата smtp сервера в исключения](/news/serveradminru/article-e76b4dce6a903572/image-27.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-26.png)

Также соглашаемся. После этого почта будет нормально работать через Thunderbird. Позже получим и настроим нормальный сертификат. Некоторые клиенты вообще не позволяют подключаться к серверу без доверия к сертификату. Я специально всё настраиваю последовательно, чтобы было понимание, как всё это настраивать и потом обслуживать. Иногда возникают ошибки с сертификатами, версиями TLS, шифрами.

На этом этапе чаще всего и возникают проблемы. Если не получается подключиться к ящику, смотрите логи Postfix и Dovecot. Можете показать ошибки в комментариях, я подскажу, в какую сторону смотреть.

Я подключился к почтовому ящику и увидел тестовые письма. Отвечу на одно из них и посмотрю в логе, как прошла отправка. Как я уже показал, у меня ещё раз выскочило окно с предупреждением о небезопасном сертификате. Ещё раз добавляю его в исключения. Это нормально, сертификат проверяется во время получения почты в Dovecot и во время отправки в Postfix. Это разные службы, у них могут быть и разные сертификаты. Так что нужны 2 подтверждения. Более того, сервера imap и smtp могут иметь разные DNS имена, разнесены на разные машины.

Отправляю письмо еще раз и смотрю лог.

```
# tail -f /var/log/mail.log
```

```
2025-11-27T16:26:19.175732+03:00 mail postfix/submission/smtpd[147650]: connect from unknown[185.21.46.169]
2025-11-27T16:26:19.272295+03:00 mail postfix/submission/smtpd[147650]: Anonymous TLS connection established from unknown[185.21.46.169]: TLSv1.3 with cipher TLS_AES_128_GCM_SHA256 (128/128 bits) key-exchange x25519 server-signature RSA-PSS (2048 bits) server-digest SHA256
2025-11-27T16:26:19.504798+03:00 mail postfix/submission/smtpd[147650]: 7B05841236: client=unknown[185.21.46.169], sasl_method=PLAIN, sasl_username=admin@zeroxzed.ru
2025-11-27T16:26:19.519353+03:00 mail postfix/cleanup[147661]: 7B05841236: message-id=<63b9cb43-9850-4b0f-be5f-bf617dd42611@zeroxzed.ru>
2025-11-27T16:26:19.689966+03:00 mail postfix/qmgr[139397]: 7B05841236: from=<admin@zeroxzed.ru>, size=1043, nrcpt=2 (queue active)
2025-11-27T16:26:20.011243+03:00 mail postfix/pipe[147665]: 7B05841236: to=<all_out@zeroxzed.ru>, relay=dovecot, delay=0.54, delays=0.22/0.04/0/0.28, dsn=2.0.0, status=sent (delivered via dovecot service)
2025-11-27T16:26:20.077489+03:00 mail postfix/smtp[147664]: Trusted TLS connection established to gmail-smtp-in.l.google.com[64.233.165.27]:25: TLSv1.3 with cipher TLS_AES_256_GCM_SHA384 (256/256 bits) key-exchange x25519 server-signature ECDSA (prime256v1) server-digest SHA256
2025-11-27T16:26:20.562039+03:00 mail postfix/smtp[147664]: 7B05841236: to=<zeroxzed@gmail.com>, relay=gmail-smtp-in.l.google.com[64.233.165.27]:25, delay=0.96, delays=0.22/0.11/0.4/0.24, dsn=5.7.26, status=bounced (host gmail-smtp-in.l.google.com[64.233.165.27] said: 550-5.7.26 Your email has been blocked because the sender is unauthenticated. 550-5.7.26 Gmail requires all senders to authenticate with either SPF or DKIM. 550-5.7.26 550-5.7.26 Authentication results: 550-5.7.26 DKIM = did not pass 550-5.7.26 SPF [zeroxzed.ru] with ip: [212.193.57.223] = did not pass 550-5.7.26 550-5.7.26 For instructions on setting up authentication, go to 550 5.7.26 https://support.google.com/mail/answer/81126#authentication 2adb3069b0e04-596bfa54ac2si297978e87.866 - gsmtp (in reply to end of DATA command))
2025-11-27T16:26:20.570160+03:00 mail postfix/cleanup[147661]: 8A80341270: message-id=<20251127132620.8A80341270@mail.zeroxzed.ru>
2025-11-27T16:26:20.577496+03:00 mail postfix/bounce[147667]: 7B05841236: sender non-delivery notification: 8A80341270
2025-11-27T16:26:20.577779+03:00 mail postfix/qmgr[139397]: 7B05841236: removed
```

В целом всё в порядке. Видно подключение с моего IP, успешную SASL аутентификацию, формирование письма на сервере, присваивание ему message-id, отправка в ящик получателя, копирование в ящик all_out (не забудьте отключить, если вам это не надо). Другое дело, что сервер Gmail моё письмо не принял, ответив, что он не принимает письма с доменов, для которых не настроены DKIM и SPF. Это нормальное поведение. Какие-то сервера игнорируют эти настройки, какие-то нет. Главное, что сам почтовый сервер работает нормально на основе текущих настроек. Далее мы его донастроим и проблем не будет.

Расскажу, куда еще надо смотреть для отладки почтовой системы. Да и не только отладки, во время работы периодически придется разбираться, куда ушло то или иное письмо, кто и когда подключался к ящику. Разные ситуации бывают. В файле*/var/log/dovecot/lda-deliver.log*содержится информация обо всех пришедших письмах - когда, от кого и в какой ящик было положено.

```
Nov 27 15:05:12 lda(admin@zeroxzed.ru)<140282><mAhXMnc+KGn6IwIAmJkchQ>: Info: Mailbox INBOX: save: uid=7, msgid=<042999fc-4efd-4536-a653-c8a6f679ac9a@rocky-linux.ru>, size=1901
Nov 27 15:05:12 lda(admin@zeroxzed.ru)<140282><mAhXMnc+KGn6IwIAmJkchQ>: Info: from=root@rocky-linux.ru msgid=<042999fc-4efd-4536-a653-c8a6f679ac9a@rocky-linux.ru>: saved mail to INBOX (subject=Re: 123 size=1901)
```

Тут происходит дублирование одной и той же информации. За вторую строку отвечает настройка**deliver_log_format**. Вы можете настроить своё отображение, какое посчитаете нужным. Формат описан в[разделе документации](https://doc.dovecot.org/2.4.2/core/summaries/settings.html#deliver_log_format).

В*/var/log/dovecot/info.log*информация о подключениях к почтовым ящикам — кто, когда, откуда и каким способом подключался к серверу. Вот пример успешной аутентификации:

```
Nov 27 15:20:45 imap-login: Info: Logged in: user=<admin@zeroxzed.ru>, method=PLAIN, rip=185.21.46.169, lip=212.193.57.223, mpid=141887, TLS, session=<h5jSipJEH5K5FC6p>
```

А вот неудачной:

```
Nov 27 04:14:25 auth-worker(all_in@zeroxzed.ru,45.92.33.82)<82365>: request [1]: Info: sql: Password mismatch
```

Тут кто-то пытается подключиться, используя несуществующий ящик:

```
Nov 27 00:23:08 auth-worker(office@zeroxzed.ru,178.16.52.107)<62690>: request [24]: Info: sql: unknown user
```

Эти три лога основные для разбора ситуаций:

- postfix: mail.log
- dovecot: lda-deliver.log
- dovecot: info.log

На текущий момент почтовый сервер на базе Postfix и Dovecot полностью работоспособен с некоторыми оговорками. В таком виде им без проблем можно пользоваться. Но функциональности мало. Использовать плагины sieve и acl через удаленные почтовые клиенты неудобно. Проще всего их настроить через веб интерфейс**Roundcube**. Установим эту веб панель на наш почтовый сервер.

## Установка веб интерфейса Roundcube

Установим и настроим самый популярный веб интерфейс для Postfix - Roundcube. Вы часто его можете видеть в услугах почтового сервера у провайдеров или в готовых сборках почтового сервера. Вообще говоря, его не обязательно ставить на почтовый сервер. Более того, я бы рекомендовал его ставить в другое место. Если в инфраструктуре имеется отдельный веб сервер, лучше поставить Roundcube туда. Но в рамках этой статьи, я всё буду ставить на один сервер для простоты и наглядности. Если у вас всё получится на одном сервере, потом сможете не спеша разнести по разным.

[Скачиваем](https://roundcube.net/download/)исходники последней LTS или STABLE версии Roundcube. Я обычно использую именно LTS версию, хотя она достаточно старая. Но моя практика показывает, что именно в ней всё работает стабильно и сразу, без лишней возни. К примеру, когда я готовил эту статью, у меня не получилось настроить работу автоответчика, о котором я расскажу отдельно ниже. Я его включал, а он просто не работает. За разумный срок (1 час) я не смог выяснить причину этого и опять откатился на LTS версию, где все заработало сразу.

Если есть желание, попробуйте последнюю версию. Может всё, что вам надо, заработает. Но в целом, там во всех версиях всё примерно одинаковое. В новых версиях есть адаптивный скин elastic, но кто-то по-прежнему использует стандартный larry.

```
# cd ~ && wget https://github.com/roundcube/roundcubemail/releases/download/1.6.11/roundcubemail-1.6.11-complete.tar.gz
```

Распаковываем и перемещаем на веб-сервер. Я установлю веб интерфейс для почты на отдельный поддомен webmail, а исходники положу в директорию*/var/www/webmail*. Это стандартная структура каталогов для веб сервера. Вы можете сделать, как вам удобно. Не обязательно полностью повторять за мной.

```
# tar xzvf roundcubemail-1.6.11-complete.tar.gz
# mkdir /var/www/webmail
# cp -R roundcubemail-1.6.11/* /var/www/webmail
# chown -R angie:angie /var/www/webmail
```

**Не забудьте проверить в момент установки номер актуальной версии roundcube и заменить его в приведенных выше командах.

Отмечу, что если для вас некритично использовать максимально свежую или LTS версию, можете установить Roundcube из стандартного репозитория Debian. Там сейчас располагаются stable версии. На момент написания статьи в репозитории находилась версия stable 1.6.11, точно такая же, как и на сайте. Обычно в репозиториях версии отстают, но так как я настраиваю на свежей Debian 13, отставания пока нет. Обычно всё же в репозиториях более старая. Так что я предпочитаю вручную выбирать и устанавливать желаемую версию.

Установим необходимые для работы roundcube зависимости:

```
# apt install aspell aspell-en fonts-glyphicons-halflings libaspell15 libjs-bootstrap libjs-bootstrap4 libjs-jquery-minicolors libjs-jstimezonedetect libjs-popper.js libjs-sizzle node-jquery php-auth-sasl php-mail-mime php-masterminds-html5 php-net-sieve php-net-smtp php-net-socket php-pspell php-intl php-imagick
```

Меняем некоторые параметры в php. Редактируем или добавляем в*/etc/php/8.4/fpm/php.ini*:

```
post_max_size = 32M
upload_max_filesize = 32M
date.timezone = Europe/Moscow
```

Перезапускаем php-fpm.

```
# systemctl restart php8.4-fpm
```

Создаём конфигурацию виртуального хоста для Roundcube в файле*/etc/angie/http.d/webmail.conf*. Если вы планируете заходить в веб почту по ip адресу, как в phpmyadmin или postfixadmin, то ничего настраивать не надо. Просто положите исходники в папку веб сервера*/var/www/html*и заходите по ip.

```
server {
    listen 80;
    server_name webmail.zeroxzed.ru;
    root  /var/www/webmail;
    index index.php index.html;

    access_log /var/log/angie/webmail-access.log;
    error_log /var/log/angie/webmail-error.log;

    location / {
        try_files  $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass   unix:/run/php/php8.4-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

Если есть возможность и желание, закройте доступ к Roundcube дополнительным паролем через basic auth, как я это сделал в самом начале с phpmyadmin. Это не обязательно делать. Roundcube вполне безопасная панель и её можно оставлять в открытом доступе. У многих хостеров она в открытом доступе для услуги почтового сервера. Но и в ней находили критические уязвимости. Так что за ней нужно внимательно следить и регулярно обновлять, защищать от перебора учёток и т.д. Если закрыть доступ отдельным паролем, то можно следить не так внимательно. Как делать - решать вам. Для конечных пользователей лишняя аутентификация всегда нежелательна.

Проверяем конфигурацию Angie и перезапускаем:

```
# angie -t && angie -s reload
```

Если всё в порядке, то можно идти по адресу http://webmail.zeroxzed.ru/installer/ и выполнять настройку Roundcube. Вы должны увидеть интерфейс установщика и все проверки в состоянииOK, кроме тех, что касаются сторонних баз. Нам нужна только MySQL.

[

![Установка Roundcube](/news/serveradminru/article-e76b4dce6a903572/image-28.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-27.png)

В самом низу нажимайте**NEXT**. Здесь нужно будет указать параметры подключения к базе данных. Создайте отдельную базу данных для roundcube и укажите данные подключения.

Также на этой странице нужно будет указать несколько параметров:

- Параметры доступа к базе данных в разделе**Database setup**
- **smtp_host**= 127.0.0.1:25
- **Use the current IMAP username and password for SMTP authentication**— галочка должна стоять
- **language**— ru_RU
- **skin**— elastic (можете поменять на larry, если вам он больше нравится)
- Выбираем**плагины**— acl, managesieve, userinfo. Остальные на свое усмотрение.

Остальное можно оставить по умолчанию. Жмем**CREATE CONFIG**. Должны увидеть сообщение:

```
The config file was saved successfully into /var/www/webmail/config directory of your Roundcube installation.
```

Жмем**CONTINUE**. Открывается страница с проверкой настроек. Нам нужно инициализировать базу данных. Для этого жмите на соответствующую кнопку.

[

![Создание базы данных для Roundcube](/news/serveradminru/article-e76b4dce6a903572/image-29.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-28.png)

Тут проверять отправку почты неудобно, можно этого не делать. Зайдем в почтовый ящик и там всё проверим. Если что, конфигурационный файл потом всё равно можно вручную отредактировать. Папку*/var/www/webmail/installer*удаляем. Заходим в почтовый ящик через roundcube. Набирать нужно полное имя ящика и пароль. Если всё сделали правильно, должны попасть в свой почтовый ящик.

[

![Веб интерфейс Roundcube для почтового ящика](/news/serveradminru/article-e76b4dce6a903572/image-30.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-29.png)

Видим всю свою переписку и тестовые сообщения, которые делали ранее. Можете попробовать написать и отправить письмо. В настройках можно изменить некоторые параметры web интерфейса. Если будут какие-то ошибки, можно посмотреть лог самого roundcube -*/var/www/webmail/logs/errors.log*Он достаточно информативен. В этой же директории будет вестись лог всех отправленных писем через web интерфейс.

Если вы хотите, чтобы пользователи могли самостоятельно изменять свой пароль от почтового ящика, смотрите отдельную статью, как это настроить -[изменение почтового пароля пользователем через roundcube](https://serveradmin.ru/izmenenie-pochtovogo-parolya-polzovatelem-cherez-roundcube/).

## Настройка TLS сертификатов в Postfix и Dovecot

Веб клиент для почтового сервера настроили, убедились, что он работает. Пришло время получить доверенные TLS сертификаты. Сделаем это с помощью модуля ACME в Angie. Он обеспечивает автоматическое получение сертификатов, которые потом будет использовать и почтовый сервер.

Для этого добавляем в секцию http в конфигурационном файле*/etc/angie/angie.conf*несколько параметров:

```
http {
...
resolver 62.76.76.62 62.76.62.76;
acme_client webmail https://acme-v02.api.letsencrypt.org/directory;
acme_client mail https://acme-v02.api.letsencrypt.org/directory;
...
```

- 62.76.76.62 62.76.62.76 - публичные DNS сервера, можете использовать любые
- webmail и mail тоже могут быть любыми словами, я указал по названиям поддоменов.

Далее создаём два конфигурационных файла в*/etc/angie/http.d*. Я назвал их по именам поддоменов -*mail.conf*и*webmail.conf*. Их содержимое:

```
server {
    listen 443 ssl;
    server_name mail.zeroxzed.ru;

    root  /var/www/mail;
    index index.php index.html;

    acme mail;
    ssl_certificate $acme_cert_mail;
    ssl_certificate_key $acme_cert_key_mail;

    access_log /var/log/angie/mail-access.log;
    error_log /var/log/angie/mail-error.log;

    location / {
        try_files  $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass   unix:/run/php/php8.4-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}

server {
    listen 80;
    server_name mail.zeroxzed.ru;
    return 301 https://$host$request_uri;
}
```

```
server {
    listen 443 ssl;
    server_name webmail.zeroxzed.ru;

    root  /var/www/webmail;
    index index.php index.html;

    acme webmail;
    ssl_certificate $acme_cert_webmail;
    ssl_certificate_key $acme_cert_key_webmail;

    access_log /var/log/angie/webmail-access.log;
    error_log /var/log/angie/webmail-error.log;

    location / {
        try_files  $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass   unix:/run/php/php8.4-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}

server {
    listen 80;
    server_name webmail.zeroxzed.ru;
    return 301 https://$host$request_uri;
}
```

Раннее директорию*/var/www/mail*для поддомена mail.zeroxzed.ru мы не создавали и никак не использовали. Можете там postfixadmin разместить или ничего не делать. В данном случае это не важно. Мы добавили виртуальный хост с этим поддоменом, чтобы получить для него сертификат, который будет использовать почтовый сервер в лице Postfix и Dovecot.

Проверяем конфигурацию Angie и перечитываем её:

```
# angie -t && angie -s reload
```

В логе /var/log/angie/error.log должны увидеть примерно такие строки:

```
2025/11/27 17:24:28 [notice] 154580#154580: creating implicit client block for "@acme"
2025/11/27 17:24:28 [notice] 154580#154580: no certificate, renewal scheduled now, ACME client: webmail
2025/11/27 17:24:29 [notice] 154581#154581: creating implicit client block for "@acme"
2025/11/27 17:24:29 [notice] 154581#154581: signal process started
2025/11/27 17:24:29 [notice] 31749#31749: signal 1 (SIGHUP) received from 154581, reconfiguring
2025/11/27 17:24:29 [notice] 31749#31749: reconfiguring
2025/11/27 17:24:29 [notice] 31749#31749: creating implicit client block for "@acme"
2025/11/27 17:24:29 [notice] 31749#31749: no certificate, renewal scheduled now, ACME client: webmail
2025/11/27 17:24:29 [notice] 31749#31749: using the "epoll" event method
2025/11/27 17:24:29 [notice] 31749#31749: start worker processes
2025/11/27 17:24:29 [notice] 31749#31749: start worker process 154582
2025/11/27 17:24:29 [notice] 154371#154371: gracefully shutting down
2025/11/27 17:24:29 [notice] 154371#154371: exiting
2025/11/27 17:24:29 [notice] 154371#154371: exit
2025/11/27 17:24:29 [notice] 31749#31749: signal 17 (SIGCHLD) received from 154371
2025/11/27 17:24:29 [notice] 31749#31749: worker process 154371 exited with code 0
2025/11/27 17:24:29 [notice] 31749#31749: signal 29 (SIGIO) received
2025/11/27 17:24:31 [notice] 154582#154582: ACME account ID: "https://acme-v02.api.letsencrypt.org/acme/acct/2832575756", ACME client: webmail
2025/11/27 17:24:47 [notice] 154582#154582: certificate renewed, next renewal date: Mon Jan 26 16:26:15 2026, ACME client: webmail
```

Сертификат для поддомена webmail успешно получен. Сами файлы сертификата лежат в директории*/var/lib/angie/acme/webmail*:

```
# ls /var/lib/angie/acme/webmail
account.key certificate.pem private.key
```

Для домена mail, соответственно, в*/var/lib/angie/acme/mail*. Именно они нам и понадобятся для использования в почтовом сервере. Меняем параметры в Postfix на использование новых сертификатов. Для этого в файле*/etc/postfix/main.cf*указываем:

```
smtpd_tls_key_file = /var/lib/angie/acme/mail/private.key
smtpd_tls_cert_file = /var/lib/angie/acme/mail/certificate.pem
```

В*/etc/dovecot/dovecot.conf*меняем параметры:

```
ssl_server_cert_file = /var/lib/angie/acme/mail/certificate.pem
ssl_server_key_file = /var/lib/angie/acme/mail/private.key
```

Перезапускаем службы:

```
# systemctl restart postfix dovecot
```

На этом с настройкой сертификатов всё. Теперь можно спокойно подключаться любым клиентом. Не должно быть никаких предупреждений. Теперь даже Thunderbird правильно автоматически определил адреса и imap, и smtp сервера. Взял их из данных в сертификатах.

## Настройка фильтра почты sieve

Sieve очень удобная штука, но вот хорошего интерфейса для управления через почтовый клиент я не знаю. Существует плагин для Thunderbird, который так и называется - sieve. Но лично мне он не понравился вообще, так как предлагает писать правила определенным кодом. Для этого надо знать синтаксис, тратить время. Можете сами на него посмотреть —[https://github.com/thsmi/sieve](https://github.com/thsmi/sieve).

К счастью, есть удобный способ писать правила фильтрации для sieve через Roundcube. Там это реализовано отдельным плагином**managesieve**, который мы активировали во время установки. Его конфигурация находится в директории*plugins/managesieve*. Скопируйте конфигурацию по умолчанию и проследите, чтобы там были следующие настройки:

```
# cp config.inc.php.dist config.inc.php
```

```
$config['managesieve_port'] = 4190;
$config['managesieve_host'] = '127.0.0.1';
$config['managesieve_usetls'] = false;
```

Для создания правила фильтрации, зайдите в свой почтовый ящик через roundcube. Переходите в раздел**Настройки -> Фильтры**и создавайте новое правило. Если оно будет содержать перемещение в какую-то директорию, то её нужно создать заранее в разделе**Настройки -> Папки**.

[

![Настройка фильтра почты sieve](/news/serveradminru/article-e76b4dce6a903572/image-31.png)

](https://serveradmin.ru/wp-content/uploads/2022/12/debian-postfix-dovecot-17.png)

После этого письма будут обрабатываться правилом сразу после поступления в почтовый сервер, вне зависимости от вашего подключения к ящику. В ящике в папке .*sieve*появился файл*managesieve.sieve*с текстовым описанием настроенного правила. Можете познакомиться с синтаксисом. Он несложный.

[

![Содержимое фильтра sieve](/news/serveradminru/article-e76b4dce6a903572/image-32.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-30.png)

С помощью этих файлов можно автоматизировать создание правил различным пользователям.

## Настройка автоответчика

В Roundcube есть замечательная возможность настроить автоответчик в почтовом ящике. Это актуально, к примеру, если вы уходите в отпуск. Вы можете сами настроить автоответчик, который будет отправлять письмо с указанной вами информацией всем, от кого будут приходить письма в ваш ящик. Возможность эта реализована на базе того же плагина managesieve. По умолчанию она отключена. Активировать ее нужно вручную.

Для того, чтобы модуль автоответчика заработал, отредактируйте конфигурационный файл плагина. Для этого открываем его в моём случае в директории с плагином*/plugins/managesieve/config.inc.php.*Изменяем там параметр:

```
$config['managesieve_vacation'] = 1;
```

После этого достаточно просто обновить веб интерфейс Roundcube, и появятся новые настройки по адресу**Настройки -> Вне офиса**.

[

![Настройка автоответчика для Postfix](/news/serveradminru/article-e76b4dce6a903572/image-33.png)

](https://serveradmin.ru/wp-content/uploads/2022/12/debian-postfix-dovecot-18.png)

По своей сути настройка автоответчика - это просто создание еще одного правила sieve для всей входящей почты.

## Общие папки по imap

Рассмотрим настройку необычной и полезной функциональности в виде общих папок. С их помощью один пользователь почтового ящика может предоставить другому пользователю доступ к папке внутри своего почтового ящика. Где и как использовать эту функциональность, каждый может придумать сам, в зависимости от своих потребностей. Мне кажется это удобным в том случае, когда создан какой-то общий ящик, на который только поступает информация и нет необходимости писать ответ от его имени. То есть по сути работает как обычный почтовый алиас. Но в случае с алиасом и несколькими почтовыми ящиками, письмо падает в каждый ящик. Если таких писем и получателей много, то идет большое дублирование одного и того же письма в рамках почтового сервера. Если сделать ящик и настроить общий доступ к папке, подключить её всем пользователям, то дублирования почты не будет. Каждый сможет прочитать письмо, без необходимости его доставки в каждый конкретный ящик.

Настроим общую папку imap. Хотя настраивать нам по сути нечего. Мы уже всё настроили ранее. Добавили соответствующие настройки в Dovecot и активировали плагин**acl**в Roundcube. Теперь нужно просто сделать папку и открыть ее для другого пользователя. Для этого идем в раздел**Настройки -> Папки**. Создаем там любую папку. В моем случае это папка с названием Общая. После того, как создали папку, открываем ее еще раз.

[

![Общие imap папки](/news/serveradminru/article-e76b4dce6a903572/image-34.png)

](https://serveradmin.ru/wp-content/uploads/2022/12/debian-postfix-dovecot-19.png)

Добавляем необходимый доступ либо всем ящикам в домене, либо какому-то конкретному. Так же можно указать, какого рода это будет доступ:

- чтение
- запись
- удаление

[

![Настройка прав для общих папок по imap](/news/serveradminru/article-e76b4dce6a903572/image-35.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-31.png)

Если тут же нажать на кнопку**Действия**и выбрать**Экспертный режим**, то настроек прав будет значительно больше.

[

![Расширенные права доступа по imap](/news/serveradminru/article-e76b4dce6a903572/image-36.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-32.png)

Заходим в ящик, которому добавили общий доступ и проверяем.

[

![Подключение общей imap папки](/news/serveradminru/article-e76b4dce6a903572/image-37.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-33.png)

Все в порядке, общая папка imap настроена и подключена. В папке*/mnt/mail/shared-folders*появился файл с настроенным выше правилом.

На этом настройка пользовательской функциональности закончена. В принципе, почтовый сервер полностью готов к работе. Но мы сделаем еще несколько необходимых настроек на стороне сервера.

## Настройка DKIM

Напишу своими словами как я понимаю работу**DKIM**. С помощью DKIM вся исходящая почта сервера подписывается электронной цифровой подписью, связанной с именем домена. Открытый ключ шифрования с помощью DNS публикуется в записи типа TXT. Таким образом, удаленный сервер, при получении письма от вас, сравнивает цифровую подпись с опубликованным в DNS открытым ключом вашего домена. Если всё в порядке, то считает, что ваше письмо в самом деле пришло от вас, а не от мошенников. То есть с помощью этой технологии можно однозначно идентифицировать отправителя.

Не стоит путать эту технологию с защитой от спама. Защиты тут нет. Любой спамер так же может себе сделать DKIM подпись к своему домену. Спамеру не составит большого труда настроить на своем сервере DKIM и отправлять спам, но подписанный электронной цифровой подписью. Теоретически, DKIM помогает защититься от подделки адреса отправителя, когда письмо якобы от вас шлет совсем другой сервер. Но с этим можно бороться и другими способами.

Для настройки DKIM устанавливаем соответствующие пакеты:

```
# apt install opendkim opendkim-tools
```

Создаем директорию для хранения ключей:

```
# mkdir -p /etc/postfix/dkim && cd /etc/postfix/dkim
```

Генерируем ключи для домена:

```
# opendkim-genkey -D /etc/postfix/dkim/ -d zeroxzed.ru -s mail
```

| zeroxzed.ru | имя почтового домена |
| --- | --- |
| mail | селектор, я обычно указываю как имя самого сервера |

На выходе получаете пару файлов - закрытый (приватный) и открытый ключ. Закрытый останется на сервере, открытый будет опубликован в DNS. Переименуем их сразу, чтобы не путаться, если у вас будет несколько доменов. Ключи нужно будет делать для каждого домена.

```
# mv /etc/postfix/dkim/mail.private /etc/postfix/dkim/mail.zeroxzed.ru.private
# mv /etc/postfix/dkim/mail.txt /etc/postfix/dkim/mail.zeroxzed.ru.txt
```

Создаем файл с таблицей ключей, в которой будут описаны все домены. В данном случае только один.

```
# mcedit /etc/postfix/dkim/keytable
```

```
mail._domainkey.zeroxzed.ru zeroxzed.ru:mail:/etc/postfix/dkim/mail.zeroxzed.ru.private
```

Тут же создаем еще один файл, в котором будет описано, каким ключом подписывать письма каждого домена. У нас один домен, поэтому только одна запись.

```
# mcedit /etc/postfix/dkim/signingtable
```

```
*@zeroxzed.ru mail._domainkey.zeroxzed.ru
```

Выставляем права доступа на все файлы:

```
# chown root:opendkim /etc/postfix/dkim/*
# chmod u=rw,g=r,o= /etc/postfix/dkim/*
```

Рисуем конфигурацию службы.

```
# cp /etc/opendkim.conf /etc/opendkim.conf.orig
# mcedit /etc/opendkim.conf
```

```
AutoRestart yes
AutoRestartRate 10/1h
Umask 022
Syslog yes
SyslogSuccess yes
LogWhy yes
Mode sv
UserID opendkim:opendkim
Socket inet:8891@localhost
PidFile /var/run/opendkim/opendkim.pid
Canonicalization relaxed/relaxed
Selector default
MinimumKeyBits 1024
KeyFile /etc/postfix/dkim/mail.zeroxzed.ru.private
KeyTable /etc/postfix/dkim/keytable
SigningTable refile:/etc/postfix/dkim/signingtable
```

Добавляем в конфигурационный файл Postfix в самый конец следующие параметры:

```
# mcedit /etc/postfix/main.cf
```

```
smtpd_milters = inet:127.0.0.1:8891
non_smtpd_milters = $smtpd_milters
milter_default_action = accept
milter_protocol = 2
```

Перезапускаем службы postfix и dkim, последний добавляем в автозагрузку.

```
# systemctl restart postfix
# systemctl restart opendkim.service
# systemctl enable opendkim.service
```

Теперь нам надо добавить открытый ключ в DNS. Идём в консоль управления DNS и добавляем новую TXT запись. Её содержание берём из файла*/etc/postfix/dkim/mail.zeroxzed.ru.txt*

```
# cat /etc/postfix/dkim/mail.zeroxzed.ru.txt
```

```
mail._domainkey	IN	TXT	( "v=DKIM1; h=sha256; k=rsa; "
	  "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAodU6tspBDprFmj0aloGo2AH0vwq9OTyyfNRjeHsIFvZUKyBWvDtYZNuPqnQXi+DLrxjYW8rkEXqipoV29/GQaNUbzKqeUF+C6JS5ehxR8p3dS1E5sR8lrHOrmwfiQN/Bgn9HPAXc8mvJ7m2Y2DJbIgpFEas7g76AtphXZOKGZECmiaQHPpA4I1bPLFdg46tNK0S4vrmNWDXvLR"
	  "5Q2culydzdfduUjbaRfUjsIw05uObxO2zV4FivSghgeEfsHsVt48QhXM5wU+pwOji+cDTc6a6FuzU+t8Xx0Wrxg1WyRyRAA1CnqN5siokTMsyXG5Q4hCOiIW4DdD+uBaFpaV12RwIDAQAB" )  ; ----- DKIM key mail for zeroxzed.ru
```

Убираем кавычки, лишние пробелы и вставляем. Должно получиться вот так:

```
v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAodU6tspBDprFmj0aloGo2AH0vwq9OTyyfNRjeHsIFvZUKyBWvDtYZNuPqnQXi+DLrxjYW8rkEXqipoV29/GQaNUbzKqeUF+C6JS5ehxR8p3dS1E5sR8lrHOrmwfiQN/Bgn9HPAXc8mvJ7m2Y2DJbIgpFEas7g76AtphXZOKGZECmiaQHPpA4I1bPLFdg46tNK0S4vrmNWDXvLR5Q2culydzdfduUjbaRfUjsIw05uObxO2zV4FivSghgeEfsHsVt48QhXM5wU+pwOji+cDTc6a6FuzU+t8Xx0Wrxg1WyRyRAA1CnqN5siokTMsyXG5Q4hCOiIW4DdD+uBaFpaV12RwIDAQAB
```

[

![DNS запись для DKIM](/news/serveradminru/article-e76b4dce6a903572/image-38.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-34.png)

Корректность настроек можно проверить с помощью какого-то публичного онлайн сервиса. Например с помощью[https://dkimcore.org/tools/keycheck.html](https://dkimcore.org/tools/keycheck.html). Указываете Selector, Имя домена и жмёте Check.

[

![Онлайн проверка DKIM](/news/serveradminru/article-e76b4dce6a903572/image-39.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-35.png)

Проверим работу. Отправляю письмо на gmail и смотрю лог почтового сервера:

```
# tail -f /var/log/mail.log
```

```
2025-11-29T07:56:49.660196+03:00 mail postfix/smtpd[351116]: connect from localhost[127.0.0.1]
2025-11-29T07:56:49.842929+03:00 mail postfix/smtpd[351116]: CDB9F43623: client=localhost[127.0.0.1], sasl_method=LOGIN, sasl_username=root@zeroxzed.ru
2025-11-29T07:56:49.891980+03:00 mail postfix/cleanup[351122]: CDB9F43623: message-id=<f511cb495bf36189fbc864539c45b864@zeroxzed.ru>
2025-11-29T07:56:49.904746+03:00 mail opendkim[349940]: CDB9F43623: DKIM-Signature field added (s=mail, d=zeroxzed.ru)
2025-11-29T07:56:49.950178+03:00 mail postfix/qmgr[349830]: CDB9F43623: from=<root@zeroxzed.ru>, size=677, nrcpt=2 (queue active)
2025-11-29T07:56:49.953111+03:00 mail postfix/smtpd[351116]: disconnect from localhost[127.0.0.1] ehlo=1 auth=1 mail=1 rcpt=1 data=1 quit=1 commands=6
2025-11-29T07:56:50.309026+03:00 mail postfix/pipe[351125]: CDB9F43623: to=<all_out@zeroxzed.ru>, relay=dovecot, delay=0.49, delays=0.13/0.05/0/0.31, dsn=2.0.0, status=sent (delivered via dovecot service)
2025-11-29T07:56:50.371767+03:00 mail postfix/smtp[351126]: Trusted TLS connection established to gmail-smtp-in.l.google.com[173.194.222.26]:25: TLSv1.3 with cipher TLS_AES_256_GCM_SHA384 (256/256 bits) key-exchange x25519 server-signature ECDSA (prime256v1) server-digest SHA256
2025-11-29T07:56:50.714689+03:00 mail postfix/smtp[351126]: CDB9F43623: to=<zeroxzed@gmail.com>, relay=gmail-smtp-in.l.google.com[173.194.222.26]:25, delay=0.89, delays=0.13/0.15/0.31/0.3, dsn=2.0.0, status=sent (250 2.0.0 OK 1764392210 2adb3069b0e04-596bfa479b7si1226551e87.508 - gsmtp)
2025-11-29T07:56:50.714908+03:00 mail postfix/qmgr[349830]: CDB9F43623: removed
```

[

![Проверка DKIM в консоли](/news/serveradminru/article-e76b4dce6a903572/image-40.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-36.png)

Всё в порядке, письмо подписано цифровой подписью. Gmail, кстати, письмо успешно принял, но поместил его в директорию СПАМ. То есть для него наиболее критичным было именно наличие DKIM. Несмотря на то, что мы ещё не настроили SPF и DMARC, почта уже нормально ходит. А попадание в СПАМ для нового почтового сервера и нового IP скорее норма, чем исключение. С этим ничего не поделать. У сервера репутации 0.

Проверим, как Gmail отреагировал на нашу подпись:

[

![Отчёт gmail по письму](/news/serveradminru/article-e76b4dce6a903572/image-41.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-37.png)

Тоже всё в порядке. Подпись выполнена корректно, проверку прошла.

## Настройка SPF

Настроим еще одно средство для повышения доверия к нашей почте со стороны других серверов —**SPF**. Расскажу опять своими словами для чего это нужно. SPF запись добавляется в виде TXT записи в DNS вашего домена. С помощью этой записи вы указываете, какие IP адреса имеют право отправлять почту от вашего имени. Если кто-то из спамеров будет использовать ваше имя домена при рассылке спама, он не пройдет проверку по SPF и скорее всего будет идентифицирован как спам.

Можно указать конкретные IP адреса в записи, а можно сказать, чтобы IP адреса проверялись по спискам A и MX записей. У нас простой случай и только 1 сервер с одним IP, поэтому укажем конкретно этот адрес. Идём в панель управления DNS и добавляем новую TXT запись.

```
zeroxzed.ru. TXT v=spf1 ip4:212.193.57.223 ~all
```

[

![Настройка SPF записи в DNS](/news/serveradminru/article-e76b4dce6a903572/image-42.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-38.png)

Больше ничего делать не надо. Можно снова отправить письмо на Gmail и проверить. Обращаю внимание, что Gmail может и без настройки SPF в DNS указать, что проверка SPF прошла, хотя TXT запись еще не была создана. Гугл умный. Думаю, он автоматом сопоставляет все DNS записи домена и убеждается, что отправка идет с доверенного сервера, к которому привязана A запись и MX запись.

Но отправка может идти не только с почтового сервера. К примеру, может быть отдельно веб сервер с интернет магазином. Он по каким-то причинам может отправлять почту сам (нет модуля для smtp отправки, не работает smtp аутентификация, разработчики странные и хотят использовать php_mail и т.д.), а не через настроенный почтовый сервер. Так иногда бывает. Тогда нужно обязательно добавить в SPF запись IP адрес этого веб сервера, с которого будет идти отправка.

## Настройка DMARC

Для настройки DMARC на самом почтовом сервере ничего делать не надо. По своей сути это просто указание другим, что делать с письмами от вас, не прошедшими проверки DKIM и SPF (которые являются подделками, если у вас всё настроено правильно). Для этого сам принимающий почтовый сервер должен поддерживать работу в соответствии с DMARC. Плюс, для вашего домена должны быть настроены правила, что делать в том или ином случае.

Есть три типа действий, которые можно настроить с помощью DMARC:

- Отклонить письмо.
- Пометить письмо как спам.
- Ничего не делать.

При этом можно настроить при каждом действии формирование отчета и отправку его на какой-то email адрес. Я очень осторожно отношусь к этим правилам и никогда не настраиваю блокировку или пометку о спаме. Так можно выстрелить себе в ногу и загубить всю свою почту из-за какой-нибудь ошибки. Мне кажется, разумнее всего настроить пропускание всех писем, но сделать отправку отчетов на свой существующий адрес. Если вы заметите какую-то подозрительную активность по отчетам, то можно будет временно изменить настройки, чтобы помечать все поддельные письма, к примеру, как спам.

Указанные правила мы сейчас и добавим с помощью TXT записи в DNS. Запись будет такая:

```
v=DMARC1; p=none; rua=mailto:dmarc@zeroxzed.ru
```

[

![Настройка DMARC в DNS](/news/serveradminru/article-e76b4dce6a903572/image-43.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-39.png)

Отчёты будут приходить в xml формате на ящик dmarc@zeroxzed.ru. Нужно будет еще потрудиться, чтобы в них разобраться. В общем случае, я вообще не слежу за DMARC. Думаю, это актуально только для крупных компаний, где есть отдельные люди, которые занимаются обслуживанием почты.

С учётом добавленных всех необходимых DNS записей, отправим письмо в Gmail ещё раз и посмотрим его проверки.

[

![Проверка DKIM, SPF, DMARC](/news/serveradminru/article-e76b4dce6a903572/image-44.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-40.png)

Всё чётко. Почтовый сервер полностью готов к работе. Обычно достаточно нескольким людям пометить письма, как НЕ СПАМ и проблема со спамом пропадает. Если, конечно, вы не делаете каких-то рассылок с него. С новым сервером нужно очень аккуратно начать работать. Если не будете делать массовые рассылки, то далее проблем особых не возникнет. Нужно понимать, что новый сервер в глазах крупных почтовых сервисов всегда будет выглядеть подозрительно. Надо заработать репутацию.

## Дополнительная функциональность почтового сервера Postfix

Я рассмотрел и настроил наиболее актуальную с моей точки зрения функциональность почтового сервера. В статье я основывался исключительно на своём опыте работы с почтой в малых и средних организациях, поэтому не претендую на истину в последней инстанции. Рекомендую осмысленно подходить к настройке своего сервера и решать, что нужно именно вам. Будет хорошо, если кто-то укажет на мои ошибки или подскажет какие-то более удобные и эффективные приёмы для решения затронутых задач.

В данном виде почтовый сервер представляет собой готовое и законченное решение, но есть еще несколько вещей, которые ему бы не помешали. Я их сейчас перечислю, а затем постараюсь постепенно писать статьи на указанные темы и ставить на них ссылки в этой теме. Вот список того, что по моему мнению нужно еще настроить на почтовом сервере:

- [Защиту от подбора паролей с помощью fail2ban](https://serveradmin.ru/fail2ban-postfix-dovecot/).
- [Мониторинг почтового сервера Postfix с помощью Zabbix](https://serveradmin.ru/monitoring-postfix-v-zabbix/).
- [Автоматическая загрузка почтовых вложений в Nextcloud](https://serveradmin.ru/avtomaticheskaya-zagruzka-pochtovyh-vlozhenij-v-nextcloud-pri-otpravke-cherez-roundcube-i-thunderbird/).
- Просмотр и анализ логов с помощью[webmin](https://serveradmin.ru/ustanovka-webmin-na-centos-7/).
- [Регулярная очистка служебных почтовых ящиков](https://serveradmin.ru/ochistka-i-obsluzhivanie-pochtovoy-bazyi-postfix/).
- Бэкап всей почтовой базы.
- Сбор логов с почтовых серверов в[ELK Stack](https://serveradmin.ru/ustanovka-i-nastroyka-elasticsearch-logstash-kibana-elk-stack/).

Расскажу, почему я не настраиваю некоторые популярные программы, которые используют на почтовых серверах:

- **Clamav**— известный антивирус. Считаю, что сейчас он не актуален, так как вирусов, от которых он способен защитить, я уже давно не видел. Сейчас вирусная эпидемия шифровальщиков. От них он не защищает. К тому же на момент написания статьи доступ к антвирусным базам с IP адресов РФ заблокирован. Нужно использовать прокси.
- **Spamassasin**или**spamd**— популярные бесплатные антиспам фильтры. Скажу честно, работал с ними очень мало и могу быть необъективен. Насколько я видел их настройку и работу - они требуют к себе некоторого внимания, калибровки, особенно на начальном этапе. Мне обычно не хочется этим заниматься. Вместо этого я использую либо платный антиспам от компании Kaspersky, либо перед почтовым сервером ставлю**[Proxmox Mail Gateway](https://www.proxmox.com/en/proxmox-mail-gateway)**, который имеет все фильтры уже в настроенном виде с управлением через веб интерфейс. Рекомендую это бесплатное решение.
- **Graylist**— эффективное средство борьбы со спамом. Я уже подробно его рассматривал, когда писал про[Iredmail](https://serveradmin.ru/ustanovka-i-nastroyka-iredmail/#Graylist), так что не буду повторяться. Скажу лишь, что режет спам очень эффективно и бесплатно, но есть существенные неудобства, которые по моему мнению не перекрывают плюсы. Поэтому я не использую.

В качестве антиспама я предпочитаю коммерческое решение -**Kaspersky Anti-Spam**. Я знаю этот продукт уже лет 15, наверное даже больше. Он действительно отлично фильтрует спам. Ложных срабатываний вообще не припомню, 95% спама фильтрует, может больше. Стоит он недорого, можно купить лицензию на меньшее количество ящиков, чем реально используется в системе. Этот вопрос никак не отслеживается и на качество работы не влияет. Но нужно понимать, что это уже нарушение лицензионного соглашения. Но можно всякие хитрости придумать, чтобы и фильтровать и не нарушать.

## Борьба со спамом средствами Postfix

Сначала хотел сразу все настройки Postfix разместить в соответствующем разделе в едином файле конфигурации, но потом передумал и решил всё же вынести этот вопрос на отдельное рассмотрение. Возможно, не каждому захочется сразу в эту тему углубляться. Всё, что рассказано выше, позволит настроить полноценный почтовый сервер, который будет успешно принимать почту и доставлять её пользователям. В таком виде он будет принимать слишком много спама, но зато не будет проблем с тем, что от кого-то что-то не придёт. Как ни крути, но все средства борьбы со спамом так или иначе несут накладные расходы в виде ложных срабатываний с той или иной вероятностью. Если вы решите не заморачиваться и купить Kaspersky Anti-Spam, можете этот раздел не читать. Он сам реализует все те проверки, что мы будем делать. Если же хотите своими силами бороться со спамом средствами Postfix, то давайте дальше разбираться.

Я буду использовать штатные возможности Postfix, позволяющие отсеять спам по тем или иным параметрам ещё до получения письма. Это очень эффективный способ с точки зрения производительности. Благодаря этому правильно настроенный на отсев спама Postfix часто ставят перед Exchange, чтобы снизить на него нагрузку. Сразу дам ссылки на официальную документацию с описанием параметров, которые я буду использовать:

- [smtpd_helo_restrictions](http://www.postfix.org/postconf.5.html#smtpd_helo_restrictions)
- [smtpd_sender_restrictions](http://www.postfix.org/postconf.5.html#smtpd_sender_restrictions)
- [smtpd_recipient_restrictions](http://www.postfix.org/postconf.5.html#smtpd_recipient_restrictions)
- [smtpd_data_restrictions](http://www.postfix.org/postconf.5.html#smtpd_data_restrictions)
- [smtpd_client_restrictions](http://www.postfix.org/postconf.5.html#smtpd_client_restrictions)

Есть еще парочка — smtpd_etrn_restrictions и smtpd_end_of_data_restrictions, но я ими не пользуюсь.

**Обращаю внимание на то, что нужно очень аккуратно работать с настройками, о которых пойдёт речь. Нужно чётко понимать как, зачем и что вы делаете. Неверные настройки могут нарушить нормальное хождение почты. Нужно уметь анализировать лог файл почтового сервера и понимать, что там происходит.

Долго думал, как лучше всего представить информацию по этому разделу. В итоге решил просто написать часть конфигурации, которая отвечает за restrictions с комментариями. Более подробную информацию по каждой настройке вы можете найти в официальной документации Postfix, ссылки я привел выше, либо в поисковиках или ИИ. Данные настройки - моя многолетняя калькуляция различных параметров, собранных из черновиков и рабочих серверов. Не везде всё было настроено именно в таком виде, так как ситуации бывают разные. Сейчас я постарался всё собрать в одном месте и аккуратно описать. Те проверки в борьбе со спамом, что вам не нужны, просто закомментируйте. В конце я еще пройдусь по некоторым из них и поделюсь своим опытом.

```
#Описание списков исключений
smtpd_restriction_classes = white_client_ip,
                            black_client_ip,
                            block_dsl,
                            white_client,
                            white_helo,
                            black_client,
                            mx_access

# IP адреса, которые нужно пропускать всегда
white_client_ip		= check_client_access hash:/etc/postfix/lists/white_client_ip

# IP адреса, которые нужно блокировать всегда
black_client_ip		= check_client_access hash:/etc/postfix/lists/black_client_ip

# E-mail, которые нужно пропускать всегда
white_client		= check_sender_access hash:/etc/postfix/lists/white_client

# E-mail, которые нужно блокировать всегда
black_client		= check_sender_access hash:/etc/postfix/lists/black_client

# Неправильные значения HELO, которые мы тем не менее принимаем
white_helo = check_sender_access hash:/etc/postfix/lists/white_helo
# Правила для блокировки различных динамических ip.
block_dsl 		= check_client_access regexp:/etc/postfix/lists/block_dsl

# Список приватных сетей, которые не могут быть использованы в качестве IP для MX записей
mx_access		= check_sender_mx_access cidr:/etc/postfix/lists/mx_access

# Проверки на основе данных, переданных в HELO/EHLO hostname
smtpd_helo_restrictions =	permit_mynetworks,
		permit_sasl_authenticated,
		white_client_ip,
                white_helo,
		black_client_ip,
		block_dsl,
		# Отказываем серверам, у которых в HELO несуществующий или не FQDN адрес 
		reject_invalid_helo_hostname,
		reject_non_fqdn_helo_hostname,
		# Запрещаем приём писем от серверов, представляющихся адресом, для которого не существует A или MX записи.
		reject_unknown_helo_hostname

# Проверки клиентского компьютера или другого почтового сервера, который соединяется с сервером postfix для отправки письма
smtpd_client_restrictions = 	permit_mynetworks,
		permit_sasl_authenticated,
		# Отвергает запрос, когда клиент отправляет команды SMTP раньше времени, еще не зная, поддерживает ли Postfix конвейерную обработку команд ESMTP
		reject_unauth_pipelining,
		# Блокируем клиентов с адресами from, домены которых не имеют A/MX записей
		reject_unknown_address,
		reject_unknown_client_hostname

# Проверки исходящей или пересылаемой через нас почты на основе данных MAIL FROM
smtpd_sender_restrictions =	permit_mynetworks,
		permit_sasl_authenticated,
		white_client,
		black_client,
		# Запрет отправки писем, когда адрес MAIL FROM не совпадает с логином пользователя
		reject_authenticated_sender_login_mismatch,
		# Отклоняем письма от несуществующих доменов
		reject_unknown_sender_domain,
		# Отклоняем письма от доменов в не FQDN формате
		reject_non_fqdn_sender,
		# Отклонение писем с несуществующим адресом отправителя
		reject_unlisted_sender,
		reject_unauth_destination,
		# Отклонять сообщения от отправителей, ящики которых не существуют, использовать аккуратно
		#reject_unverified_sender,
		mx_access

# Правила приема почты нашим сервером на основе данных RCPT TO
smtpd_recipient_restrictions =  permit_mynetworks,
		permit_sasl_authenticated,
		# Отклоняет всю почту, что адресована не для наших доменов
		reject_unauth_destination,
		# Отклонение писем с несуществующим адресом получателя
		reject_unlisted_recipient,
		# Отклоняет сообщения на несуществующие домены
		reject_unknown_recipient_domain,
		# Отклоняет сообщения если получатель не в формате FQDN
		reject_non_fqdn_recipient,
		# Отклоняем прием от отправителя с пустым адресом письма, предназначенным нескольким получателям.
		reject_multi_recipient_bounce
```

У меня во всех ограничениях первыми правилами стоят разрешения для mynetworks и аутентифицировавшихся пользователей (permit_sasl_authenticated). Важно понимать, что это значит и для чего сделано. Ограничения читаются последовательно в порядке их перечисления. Таким образом, мы своих пользователей пускаем мимо ограничений, а для всех остальных выполняются проверки.

Теперь важные комментарии по указанным параметрам. Если бы все почтовые сервера всех системных администраторов были настроены по правилам, то эти комментарии не были бы нужны. Пройдемся по некоторым ограничениям, которые нужно включать осторожно:

- reject_invalid_helo_hostname и reject_unknown_helo_hostname — под эти правила иногда попадают почтовые серверы клиентов, которые не очень хорошо настроены. У них бывают неправильные адреса, кривые записи DNS, отсутствие обратных зон и т.д. Их немного, но попадаются. Это не страшно, если вы регулярно следите за сервером. Отправитель получит сразу сообщение о том, что его письмо не дошло до вас. Если он как-то сообщит вам о проблеме, вы легко добавите его в белый список и всё будет нормально. Если вам не хочется следить за сервером, лучше не указывайте эти ограничения. Но спам они отсекают хорошо. Сюда попадают все завирусованные компьютеры и сервера без нормальных настроек DNS (а их чаще всего и нет).
- reject_unverified_sender — специально его закомментировал. Я тестировал этот параметр. В принципе, работает нормально, но есть, как обычно, нюансы. Поясню, что делает этот параметр. Когда вам кто-то шлёт письмо, ваш сервер обращается к серверу отправителю и спрашивает его стандартной командой, есть ли на сервере такой отправитель. Если удаленный сервер отвечает, что есть, то никаких проблем — письмо принимается. Если удаленный сервер не отвечает или говорит, что такого адресата нет — письмо отклоняем. Очевидно, что такие проверки создают дополнительную постоянную нагрузку. Это нужно учитывать. К тому же, у вас почтовый лог постоянно будет забит этими проверками, особенно, если вам приходит много спама. На каждое спамовое письмо будет идти проверка, а сервера отправителя скорее всего либо нет, либо он неправильный, либо не отвечает и т.д. Всё это будет постоянно проверяться и фиксироваться. В общем, я не использую.

На время отладки ограничений, рекомендую пользоваться параметром:

```
soft_bounce = yes
```

Когда он включен, все ответы сервера с кодами ошибок 5XX, заменяются на 4ХХ. То есть постоянная ошибка, которая сразу отклоняет письмо, заменяется на временную, которая предлагает повторить отправку позже. Таким образом, вы увидите работу всех ограничений, но письма не будут отклонены навсегда. Сервер отправителя через некоторое время снова придет к вам с новой попыткой доставки почты. Письмо безвозвратно не отклоняется. Вы можете проанализировать работу фильтра и решить, ставить его на постоянную работу или с ним что-то не так.

Создадим теперь файлы с белыми и чёрными (надеюсь, так ещё можно писать) списками.

```
cd /etc/postfix/lists && touch white_client_ip black_client_ip white_client black_client white_helo block_dsl mx_access
```

Ниже пример содержания этих файлов. Вы можете менять по своему усмотрению.

```
# cat white_client_ip
195.28.34.162 OK
141.197.4.160 OK
```

```
# cat black_client_ip
205.201.130.163 REJECT You IP are blacklisted!
198.2.129.162 REJECT You IP are blacklisted!
```

```
# cat white_client
# Принимать всю почту с домена яндекс
yandex.ru OK
# Разрешить конкретный ящик
spammer@mail.ru OK
```

```
# cat black_client
# Блокировать всю почту с домена mail.ru
mail.ru REJECT You domain are blacklisted!
# Блокировать конкретный ящик
spam@rambler.ru REJECT You e-mail are blacklisted!
```

```
# cat white_helo
# Могут попадаться вот такие адреса, которые не пройдут наши проверки
ka-s-ex01.itk.local     OK
exchange.elcom.local    OK
```

```
# cat block_dsl
/^dsl.*\..*\..*/i                               553 AUTO_DSL spam
/dsl.*\..*\..*/i                                553 AUTO_DSL1 spam
/[ax]dsl.*\..*\..*/i                            553 AUTO_XDSL spam
/client.*\..*\..*/i                             553 AUTO_CLIENT spam
/cable.*\..*\..*/i                              553 AUTO_CABLE spam
/pool.*\..*\..*/i                               553 AUTO_POOL spam
/dial.*\..*\..*/i                               553 AUTO_DIAL spam
/ppp.*\..*\..*/i                                553 AUTO_PPP spam
/dslam.*\..*\..*/i                              553 AUTO_DSLAM spam
/node.*\..*\..*/i                               553 AUTO_NODE spam
/([0-9]*-){3}[0-9]*(\..*){2,}/i                 553 SPAM_ip-add-rr-ess_networks
/([0-9]*\.){4}(.*\.){3,}.*/i                    553 SPAM_ip-add-rr-ess_networks
/.*\.pppool\..*/i                               553 SPAM_POOL
/[0-9]*-[0-9]*-[0-9]*-[0-9]*-tami\.tami\.pl/i   553 SPAM_POOL
/pool-[0-9]*-[0-9]*-[0-9]*-[0-9]*\..*/i         553 SPAM_POOL
/.*-[0-9]*-[0-9]*-[0-9]*-[0-9]*\.gtel.net.mx/i  553 SPAM_POOL
/dhcp.*\..*\..*/i                               553 SPAM_DHCP
```

```
# cat mx_access
127.0.0.1      DUNNO 
127.0.0.2      550 Domains not registered properly
0.0.0.0/8      REJECT Domain MX in broadcast network 
10.0.0.0/8     REJECT Domain MX in RFC 1918 private network 
127.0.0.0/8    REJECT Domain MX in loopback network 
169.254.0.0/16 REJECT Domain MX in link local network 
172.16.0.0/12  REJECT Domain MX in RFC 1918 private network 
192.0.2.0/24   REJECT Domain MX in TEST-NET network 
192.168.0.0/16 REJECT Domain MX in RFC 1918 private network 
224.0.0.0/4    REJECT Domain MX in class D multicast network 
240.0.0.0/5    REJECT Domain MX in class E reserved network 
248.0.0.0/5    REJECT Domain MX in reserved network
```

По сути файлы белых и чёрных списков не отличаются друг от друга. Можно использовать только один файл и в нём в каждой отдельной строке указывать либо запрет, либо разрешение. Я разделил просто для удобства восприятия и редактирования. Возможно вам будет удобнее с одним файлом.

После редактирования файлов обязательно выполняем команду на перестроение базы данных. Я перестрою сразу все файлы:

```
cd /etc/postfix/lists && postmap white_client_ip black_client_ip white_client black_client white_helo block_dsl mx_access
```

Ещё упомяну о таком популярном явлении в спамерских письмах, как подделка адреса отправителя. Причем не просто подделка на абы кого, а именно на ваше имя домена. Пользователь получает спам письмо и в почтовом клиенте видит, что оно отправлено с вашего домена. Только по заголовкам можно определить реального отправителя. Такой подход позволяет обходить некоторые антиспам системы, которые не фильтруют письма внутреннего домена. Бороться с подменой адреса отправителя в нашем случае очень просто. Об этом я рассказал отдельно -[Запрет писем с поддельным полем From](https://serveradmin.ru/zapret-pisem-s-poddelnyim-polem-from-ili-spam-ot-sebya-k-sebe-v-postfix/).

Приведу в завершении описания методов борьбы со спамом простой пример. Добавим в black_client почтовый адрес и отправим с него письмо.

```
# cat black_client
zeroxzed@gmail.com REJECT Your e-mail was banned!
```

```
# postmap black_client
```

Отправляем сообщение и проверяем почтовый лог.

```
# tail -f /var/log/mail.log
```

```
Feb 11 16:21:31 mail postfix/smtpd[20115]: connect from mail-ua1-f48.google.com[209.85.222.48]
Feb 11 16:21:31 mail postfix/smtpd[20115]: Anonymous TLS connection established from mail-ua1-f48.google.com[209.85.222.48]: TLSv1.3 with cipher TLS_AES_128_GCM_SHA256 (128/128 bits)
Feb 11 16:21:32 mail postfix/smtpd[20115]: NOQUEUE: reject: RCPT from mail-ua1-f48.google.com[209.85.222.48]: 554 5.7.1 <zeroxzed@gmail.com>: Sender address rejected: Your e-mail was banned!; from=<zeroxzed@gmail.com> to=<root@zeroxzed.ru> proto=ESMTP helo=
Feb 11 16:21:32 mail postfix/smtpd[20115]: disconnect from mail-ua1-f48.google.com[209.85.222.48] ehlo=2 starttls=1 mail=1 rcpt=0/1 data=0/1 quit=1 commands=5/7
```

Вот и результат. На этом по борьбе со спамом всё.

## Проверка почтового сервера

Проверить настроенный почтовый сервер можно с помощью онлайн сервиса[https://www.mail-tester.com](https://www.mail-tester.com/). Не факт, что получите максимальный бал, но все недочёты будут указаны. Критичное нужно исправить (например, если обратная зона неправильная), некритичное можно пропустить. Я проверил сервер, настроенный строго по этой статье. Получил 10/10.

[

![Онлайн проверка почтового сервера](/news/serveradminru/article-e76b4dce6a903572/image-45.png)

](https://serveradmin.ru/wp-content/uploads/2025/11/mailserver-debian-postfix-dovecot-41.png)

Замечания были только непосредственно к содержимому:

> 

HTML-версии сообщения в Вашем письме нет.

Ваше сообщение не содержит заголовок List-Unsubscribe

Письмо было в обычном текстовом формате. А так как это не рассылка, то и соответствующего тэга тоже не было.

Некоторые другие сервисы по проверке почтового сервера описал в отдельной статье -[Онлайн сервисы для проверки работы почтового сервера](https://serveradmin.ru/onlajn-servisy-dlya-proverki-raboty-pochtovogo-servera/).

## Заключение

Кажется всё написал, что знал по поводу почтового сервера на Linux в небольших и средних организациях. У некоторых может возникнуть вопрос, а зачем свой почтовый сервер? Почему бы не воспользоваться средствами корпоративной почты, которую представляют популярные почтовые сервисы бесплатно или за деньги? У меня есть определенный опыт на этот счет, в том числе негативный. И если некоторое время назад я считал, что свои почтовые серверы в небольших организациях уже не актуальны, то сейчас я так не думаю, поэтому и появилась эта статья.

Также вам могут быть полезны мои материалы на тему Postfix:

- [Как изменить тему письма и адрес отправителя через postfix](https://serveradmin.ru/kak-izmenit-temu-pisma-i-adres-otpravitelya-cherez-postfix/)
- [Настройка relayhost, отдельный для каждого домена](https://serveradmin.ru/postfix-nastrojka-relayhost-otdelnyj-dlya-kazhdogo-domena/)
- [Перенос почтового сервера postfix](https://serveradmin.ru/perenos-pochtovogo-servera-postfix/)

Буду рад замечаниям по делу и советам в комментариях. Напоминаю, что данная статья является частью единого цикла статьей про сервер[Debian](https://serveradmin.ru/debian/).

**Онлайн-курс по устройству компьютерных сетей.**

На углубленном курсе "[Архитектура современных компьютерных сетей](https://курсы-по-ит.рф/computer_networks?utm_source=serveradminru&utm_medium=cpc&utm_campaign=asks&erid=2SDnjdSUXCF)" вы с нуля научитесь работать с Wireshark и «под микроскопом» изучите работу сетевых протоколов. На протяжении курса надо будет выполнить более пятидесяти лабораторных работ в Wireshark.
Реклама ООО "Курсы по ИТ" ИНН 5029266531

#### Помогла статья? Подписывайся на[telegram канал](https://serveradmin.ru/telegram-header)автора

Анонсы всех статей, плюс много другой полезной и интересной информации, которая не попадает на сайт.

## Оригинал

https://serveradmin.ru/nastrojka-postfix-dovecot-postfixadmin-roundcube-dkim-na-debian/
