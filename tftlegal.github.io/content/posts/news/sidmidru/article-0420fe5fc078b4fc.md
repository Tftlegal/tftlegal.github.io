---
title: "3x-ui vless"
date: 2025-05-29T04:56:45Z
source: "sidmidru"
original_url: "https://sidmid.ru/3x-ui-vless/"
summary: "В тексте описывается настройка сервера в AWS VPC. Задача — обеспечить доступ не только в интернет, но и во внутреннюю сеть. Для этого создаётся VPC с peering-соединениями в другие AWS-аккаунты. Настраиваются внутренняя таблица маршрутизации и security group. Затем запускается инстанс, который остаётся в приватной сети, но получает публичный адрес. Ссылка указывает на материал по 3x-ui VLESS."
---

# 3x-ui vless

## Краткое содержание

В тексте описывается настройка сервера в AWS VPC.
Задача — обеспечить доступ не только в интернет, но и во внутреннюю сеть.
Для этого создаётся VPC с peering-соединениями в другие AWS-аккаунты.
Настраиваются внутренняя таблица маршрутизации и security group.
Затем запускается инстанс, который остаётся в приватной сети, но получает публичный адрес.
Ссылка указывает на материал по 3x-ui VLESS.

## Полная статья

Thank you for reading this post, don't forget to subscribe!

задача - нужно чтобы доступ был и во внутреннюю сеть, не только в мир

поднимаем сервер вVPCс которого у нас есть пиринг во все остальные aws аккаунты

![](/news/sidmidru/article-0420fe5fc078b4fc/image-01.png)

вот internal route table

![](/news/sidmidru/article-0420fe5fc078b4fc/image-02.png)

вот security group

![](/news/sidmidru/article-0420fe5fc078b4fc/image-03.png)

![](/news/sidmidru/article-0420fe5fc078b4fc/image-04.png)

vpc настроено так:

![](/news/sidmidru/article-0420fe5fc078b4fc/image-05.png)

теперь поднимаем сам Instance

![](/news/sidmidru/article-0420fe5fc078b4fc/image-06.png)

![](/news/sidmidru/article-0420fe5fc078b4fc/image-07.png)

всё можно запускать - будет в приватной сети и будет назначен public ip - потом можно добавить elastic ip

yum update -y

yum install -y docker git

usermod -a -G docker ec2-user

curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose

sysctl -w net.ipv4.ip_forward=1

nano /etc/sysctl.conf

добавляем net.ipv4.ip_forward=1 и сохраняем

sysctl -p

git clone https://github.com/MHSanaei/3x-ui.git

cd 3x-ui

systemctl status docker

systemctl enable docker

systemctl start docker

docker-compose up -d

заходим в панель

http://3.68.114.223:2053/

admin
admin
и меняем пароль

![](/news/sidmidru/article-0420fe5fc078b4fc/image-08.png)

теперь настраиваем подключение:

![](/news/sidmidru/article-0420fe5fc078b4fc/image-09.png)

![](/news/sidmidru/article-0420fe5fc078b4fc/image-10.png)

теперь можем добавлять пользователей:

![](/news/sidmidru/article-0420fe5fc078b4fc/image-11.png)

![](/news/sidmidru/article-0420fe5fc078b4fc/image-12.png)

![](/news/sidmidru/article-0420fe5fc078b4fc/image-13.png)

клиентам можно высылать или qr или конфиг.

описание многих параметров есть вот в этом pdf

[Инструкция по настройке своего Xray-сервера (VLESSXTLS-Reality+3X-UIнаVPS) в 2024-2025г](https://sidmid.ru/wp-content/uploads/2025/05/Инструкция-по-настройке-своего-Xray-сервера-VLESS-XTLS-Reality3X-UI-на-VPS-в-2024-2025г.pdf)

### Подключение клиентов

android

ставим приложение поддерживающее vless протокол - я использую hiddify

![](/news/sidmidru/article-0420fe5fc078b4fc/image-14.jpg)

![](/news/sidmidru/article-0420fe5fc078b4fc/image-15.jpg)

![](/news/sidmidru/article-0420fe5fc078b4fc/image-16.jpg)

![](/news/sidmidru/article-0420fe5fc078b4fc/image-17.jpg)

Linux

идём на

[https://hiddify.com/app/](https://hiddify.com/app/)

![](/news/sidmidru/article-0420fe5fc078b4fc/image-18.png)

скачивается файл - делаем его испольняемым

chmod +x Hiddify-Linux-x64.AppImage

после нужно запустить из под рута
sudo ./Hiddify-Linux-x64.AppImage

![](/news/sidmidru/article-0420fe5fc078b4fc/image-19.png)

![](/news/sidmidru/article-0420fe5fc078b4fc/image-20.png)

идём на сервер, и копируем конфиг для подключения

![](/news/sidmidru/article-0420fe5fc078b4fc/image-21.png)

возвращемся в апку и нажимаем на кнопку добавить из буфера обмена

далее меняем режим работы на vpn

![](/news/sidmidru/article-0420fe5fc078b4fc/image-22.png)

всё ок, дальше можно подключаться

![](/news/sidmidru/article-0420fe5fc078b4fc/image-23.png)

для mac ios можно так же использовать hiddify

но есть альтернативы

[https://apps.apple.com/ru/app/v2box-v2ray-client/id6446814690](https://apps.apple.com/ru/app/v2box-v2ray-client/id6446814690)
[https://apps.apple.com/ru/app/foxray/id6448898396](https://apps.apple.com/ru/app/foxray/id6448898396)
[https://apps.apple.com/ru/app/streisand/id6450534064](https://apps.apple.com/ru/app/streisand/id6450534064)
[https://github.com/tzmax/V2RayXS](https://github.com/tzmax/V2RayXS)
[https://github.com/abbasnaqdi/nekoray-macos](https://github.com/abbasnaqdi/nekoray-macos)
[https://github.com/LorenEteval/Furious/](https://github.com/LorenEteval/Furious/)

## Навигация по записям

## https://github.com/midnight47/

## Оригинал

https://sidmid.ru/3x-ui-vless/
