---
title: "New Release: Tor Browser 16.0.1"
date: 2026-11-10T00:00:00Z
source: "blogtorprojectorg"
original_url: "https://blog.torproject.org/new-release-tor-browser-1601/"
summary: "Tor Browser 16.0.1 стал доступен для скачивания на сайте проекта и в каталоге дистрибутивов. Главное обновление — важные исправления безопасности в базе Firefox (переход на 153.7.0esr), включая бэкпорт патчей из Firefox 160. Также обновлены Tor до 0.4.9.16, NoScript до 13.6.38.1984 и zlib до 1.4. На Android обновлён GeckoView до версии 153.7.0esr. В системе сборки для Android внесены изменения в процесс подписи APK с использованием нового apksigner. Разработчики призывают пользователей сообщать о найденных багах и предлагать улучшения."
---

# New Release: Tor Browser 16.0.1

## Краткое содержание

Tor Browser 16.0.1 стал доступен для скачивания на сайте проекта и в каталоге дистрибутивов. Главное обновление — важные исправления безопасности в базе Firefox (переход на 153.7.0esr), включая бэкпорт патчей из Firefox 160. Также обновлены Tor до 0.4.9.16, NoScript до 13.6.38.1984 и zlib до 1.4. На Android обновлён GeckoView до версии 153.7.0esr. В системе сборки для Android внесены изменения в процесс подписи APK с использованием нового apksigner. Разработчики призывают пользователей сообщать о найденных багах и предлагать улучшения.

## Полная статья

Tor Browser 16.0.1 is now available from the[Tor Browser download page](https://www.torproject.org/download/)and also from our[distribution directory](https://www.torproject.org/dist/torbrowser/16.0.1/).

This version includes important[security updates](https://www.mozilla.org/en-US/security/advisories/)to Firefox.

## Send us your feedback

If you find a bug or have a suggestion for how we could improve this release,[please let us know](https://support.torproject.org/misc/bug-or-feedback/).

## Full changelog

The[full changelog](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/raw/maint-16.0/projects/browser/Bundle-Data/Docs-TBB/ChangeLog.txt)since Tor Browser 16.0 is:

- All Platforms

- Updated Tor to 0.4.9.16
- Updated NoScript to 13.6.38.1984
- Updated zlib to 1.4
- [Bug tor-browser#45439](https://gitlab.torproject.org/tpo/applications/tor-browser/-/issues/45439): Rebase Tor Browser onto 153.7.0esr
- [Bug tor-browser#45442](https://gitlab.torproject.org/tpo/applications/tor-browser/-/issues/45442): Backport Security Fixes from Firefox 160

- Windows + macOS + Linux

- Updated Firefox to 153.7.0esr

- Android

- Updated GeckoView to 153.7.0esr

- Build System

- Android

- [Bug tor-browser-build#41814](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/issues/41814): Use new apksigner from build-tools_r37_linux.zip in signing script
- [Bug tor-browser-build#41912](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/issues/41912): Add`--preserve-alignment true`to apksigner sign calls

## Оригинал

https://blog.torproject.org/new-release-tor-browser-1601/
