---
title: "AWS organization несколько аккаунтов"
date: 2026-07-11T05:35:45Z
source: "sidmidru"
original_url: "https://sidmid.ru/aws-organization-%d0%bd%d0%b5%d1%81%d0%ba%d0%be%d0%bb%d1%8c%d0%ba%d0%be-%d0%b0%d0%ba%d0%ba%d0%b0%d1%83%d0%bd%d1%82%d0%be%d0%b2/"
summary: "В тексте описан процесс создания нескольких аккаунтов в AWS. Сначала нужно войти в корневой аккаунт и перейти в AWS Organizations. Там добавляется новый аккаунт, для которого указывается адрес электронной почты, не совпадающий с root-почтой. Затем в IAM Identity Center создаются новая группа и новый пользователь. На указанный адрес приходит ссылка для активации аккаунта. Пользователь открывает ссылку, задаёт пароль и настраивает аутентификацию через приложение, отсканировав QR-код."
---

# AWS organization несколько аккаунтов

## Краткое содержание

В тексте описан процесс создания нескольких аккаунтов в AWS.
Сначала нужно войти в корневой аккаунт и перейти в AWS Organizations.
Там добавляется новый аккаунт, для которого указывается адрес электронной почты, не совпадающий с root-почтой.
Затем в IAM Identity Center создаются новая группа и новый пользователь.
На указанный адрес приходит ссылка для активации аккаунта.
Пользователь открывает ссылку, задаёт пароль и настраивает аутентификацию через приложение, отсканировав QR-код.

## Полная статья

Thank you for reading this post, don't forget to subscribe!

в рутовом аккаунте заходим вAWSOrganizations и добавляем новый аккаунт

![](/news/sidmidru/article-2eaac5d82fa28109/image-01.png)

добавляем новый аккаунт

![](/news/sidmidru/article-2eaac5d82fa28109/image-02.png)

email не может совпадать с root email

![](/news/sidmidru/article-2eaac5d82fa28109/image-03.png)

далее идём вIAMIdentity Center создаём новую группу:

![](/news/sidmidru/article-2eaac5d82fa28109/image-04.png)

![](/news/sidmidru/article-2eaac5d82fa28109/image-05.png)

создаём теперь пользователя:

![](/news/sidmidru/article-2eaac5d82fa28109/image-06.png)

![](/news/sidmidru/article-2eaac5d82fa28109/image-07.png)

![](/news/sidmidru/article-2eaac5d82fa28109/image-08.png)

после добавления на почту будет выслана ссылка переходим по ней.

![](/news/sidmidru/article-2eaac5d82fa28109/image-09.png)

задаём пароль

![](/news/sidmidru/article-2eaac5d82fa28109/image-10.jpg)

выбираем аутентификацию через приложение:

![](/news/sidmidru/article-2eaac5d82fa28109/image-11.jpg)

сканируем qr код приложением, далее полученный код вводим в поле autentification code

![](/news/sidmidru/article-2eaac5d82fa28109/image-12.jpg)

всё, далее можно заходить вводим логин

![](/news/sidmidru/article-2eaac5d82fa28109/image-13.jpg)

пароль:

![](/news/sidmidru/article-2eaac5d82fa28109/image-14.jpg)

проверяем в панели:

![](/news/sidmidru/article-2eaac5d82fa28109/image-15.png)

создаём permission set

![](/news/sidmidru/article-2eaac5d82fa28109/image-16.png)

![](/news/sidmidru/article-2eaac5d82fa28109/image-17.png)

добавляем полиси

![](/news/sidmidru/article-2eaac5d82fa28109/image-18.png)

![](/news/sidmidru/article-2eaac5d82fa28109/image-19.png)

![](/news/sidmidru/article-2eaac5d82fa28109/image-20.png)

я добавил 2 группы:

![](/news/sidmidru/article-2eaac5d82fa28109/image-21.png)

теперь добавляем группу к аккаунту:

![](/news/sidmidru/article-2eaac5d82fa28109/image-22.png)

![](/news/sidmidru/article-2eaac5d82fa28109/image-23.png)

![](/news/sidmidru/article-2eaac5d82fa28109/image-24.png)

![](/news/sidmidru/article-2eaac5d82fa28109/image-25.png)

добавили теперь у аккаунта есть группу

![](/news/sidmidru/article-2eaac5d82fa28109/image-26.png)

получить ссылку для входа в организацию вы можете тут:

![](/news/sidmidru/article-2eaac5d82fa28109/image-27.png)

логинимся по ссылке вводим аутентификационный код

![](/news/sidmidru/article-2eaac5d82fa28109/image-28.png)

настроенSSOнажимаем на имя группы**test**и попадаем в панель

![](/news/sidmidru/article-2eaac5d82fa28109/image-29.png)

## Навигация по записям

## https://github.com/midnight47/

## Оригинал

https://sidmid.ru/aws-organization-%d0%bd%d0%b5%d1%81%d0%ba%d0%be%d0%bb%d1%8c%d0%ba%d0%be-%d0%b0%d0%ba%d0%ba%d0%b0%d1%83%d0%bd%d1%82%d0%be%d0%b2/
