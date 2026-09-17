---
title: "Выпуск браузера Pale Moon 35.0"
date: 2026-09-17T13:28:26Z
source: "opennetru"
original_url: "https://www.opennet.ru/opennews/art.shtml?num=66297"
summary: "Вышел релиз веб-браузера Pale Moon версии 35.0.0. Проект создан как ответвление от кодовой базы Firefox. Основная цель — повысить эффективность работы и уменьшить потребление памяти. Разработчики также сохраняют классический интерфейс и расширяют возможности настройки. Сборки доступны для Windows и Linux x86_64, а код распространяется под лицензией MPLv2."
---

# Выпуск браузера Pale Moon 35.0

## Краткое содержание

Вышел релиз веб-браузера Pale Moon версии 35.0.0.
Проект создан как ответвление от кодовой базы Firefox.
Основная цель — повысить эффективность работы и уменьшить потребление памяти.
Разработчики также сохраняют классический интерфейс и расширяют возможности настройки.
Сборки доступны для Windows и Linux x86_64, а код распространяется под лицензией MPLv2.

## Полная статья

[Опубликован](https://forum.palemoon.org/viewtopic.php?f=1&t=33764)релиз web-браузера[Pale Moon 35.0.0](https://www.palemoon.org/), ответвившегося от кодовой базы Firefox для обеспечения более высокой эффективности работы, сохранения классического интерфейса, минимизации потребления памяти и предоставления дополнительных возможностей по настройке. Сборки Pale Moon формируются для[Windows](https://www.palemoon.org/download.shtml)и[Linux](http://linux.palemoon.org/)(x86_64). Код проекта[распространяется](https://repo.palemoon.org/MoonchildProductions/Pale-Moon)под лицензией MPLv2 (Mozilla Public License).

Проект[придерживается](https://www.palemoon.org/technical.shtml)классической организации интерфейса, без перехода к интегрированным в Firefox 29 и 57 интерфейсам Australis и Photon, и с предоставлением широких возможностей кастомизации. Из удалённых компонентов можно отметить DRM, Social API, WebRTC, PDF-просмотрщик, Сrash Reporter, код для сбора статистики, средства для родительского контроля и людей с ограниченными возможностями. По сравнению с Firefox, в браузер возвращена поддержка расширений, использующих XUL, и сохранена возможность применения как полноценных, так и легковесных тем оформления.

Основные[изменения](https://www.palemoon.org/releasenotes.shtml):

- Включены по умолчанию прослойки для обеспечения совместимости с API Document.elementFromPoint, GetAnimations, image.decode(), Intl.DisplayNames, Intl.ListFormat, Intl.RelativeTimeFormat и Streams.
- В CSS реализованы ключевое слово[revert-layer](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/revert-layer)и псевдо-класс ":has()".
- В объекте[CSSStyleSheet](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleSheet/CSSStyleSheet)реализован метод replaceSync.
- Добавлено свойство[adoptedStyleSheets](https://developer.mozilla.org/en-US/docs/Web/API/Document/adoptedStyleSheets).
- В режиме совместимости с Firefox по умолчанию выставлена версия 140.
- Улучшена поддержка архитектур LoongArch64 и Mac/PPC.
- Для Linux и FreeBSD добавлена опция для сборки поддержки NPAPI без GTK2.
- На платформе Linux повышена производительность набора данных в формах на некоторых сложных страницах.

## Оригинал

https://www.opennet.ru/opennews/art.shtml?num=66297
