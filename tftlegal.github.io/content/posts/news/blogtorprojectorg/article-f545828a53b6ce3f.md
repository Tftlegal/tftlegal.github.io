---
title: "New Release: Tor Browser 15.0.26"
date: 2026-10-21T00:00:00Z
source: "blogtorprojectorg"
original_url: "https://blog.torproject.org/new-release-tor-browser-15026/"
summary: "Выпущена новая версия Tor Browser 15.0.26, доступная для скачивания со страницы загрузки и каталога дистрибутивов. Ключевое обновление этой версии — блокировка скомпрометированных промежуточных сертификатов eMudhra. При первом запуске браузера обновлённая база отзыва сертификатов может требовать нескольких минут на построение. Разработчики приглашают пользователей отправлять обратную связь, сообщать об ошибках и делиться предложениями по улучшению. Полный список изменений по сравнению с версией 15.0.25 включает обновление CA-промежуточных сертификатов и других конфигурационных настроек для всех платформ."
---

# New Release: Tor Browser 15.0.26

## Краткое содержание

Выпущена новая версия Tor Browser 15.0.26, доступная для скачивания со страницы загрузки и каталога дистрибутивов. Ключевое обновление этой версии — блокировка скомпрометированных промежуточных сертификатов eMudhra. При первом запуске браузера обновлённая база отзыва сертификатов может требовать нескольких минут на построение. Разработчики приглашают пользователей отправлять обратную связь, сообщать об ошибках и делиться предложениями по улучшению. Полный список изменений по сравнению с версией 15.0.25 включает обновление CA-промежуточных сертификатов и других конфигурационных настроек для всех платформ.

## Полная статья

Tor Browser 15.0.26 is now available from the[Tor Browser download page](https://www.torproject.org/download/)and also from our[distribution directory](https://www.torproject.org/dist/torbrowser/15.0.26/).

This version[blocks compromised intermediate eMudhra certificates](https://www.mozilla.org/en-US/security/advisories/mfsa2026-112/#CVE-2026-113470).

Note: the updated revocation database can take some minutes to build on first browser startup.

## Send us your feedback

If you find a bug or have a suggestion for how we could improve this release,[please let us know](https://support.torproject.org/misc/bug-or-feedback/).

## Full changelog

The[full changelog](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/raw/maint-15.0/projects/browser/Bundle-Data/Docs-TBB/ChangeLog.txt)since Tor Browser 15.0.25 is:

- All Platforms

- [Bug tor-browser#45397](https://gitlab.torproject.org/tpo/applications/tor-browser/-/issues/45397): Update CA intermediates and other settings dumps

## Оригинал

https://blog.torproject.org/new-release-tor-browser-15026/
