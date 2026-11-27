---
title: "New Alpha Release: Tor Browser 16.0a14"
date: 2026-10-15T00:00:00Z
source: "blogtorprojectorg"
original_url: "https://blog.torproject.org/new-alpha-release-tor-browser-160a14/"
summary: "Вышел Tor Browser версии 16.0a14, доступный для скачивания со страницы загрузки и дистрибутивного каталога. Ключевое обновление — важные исправления безопасности в базе Firefox (переход на 153.5.0esr), а также обновление Tor до 0.4.9.14 и NoScript. Это альфа-версия, предназначенная исключительно для тестирования: она с большей вероятностью содержит ошибки, влияющие на удобство, безопасность и приватность. Пользователям, которым критична надёжная анонимность, рекомендуется пользоваться стабильной веткой. В сборке также обновлены инструменты: GeckoView, Go (до 1.26.9), Lyrebird (до 0.9.0) и отдельные компоненты системы сборки. Разработчики просят сообщество присылать обратную связь и сообщения об ошибках. Полный список изменений доступен в changelog."
---

# New Alpha Release: Tor Browser 16.0a14

## Краткое содержание

Вышел Tor Browser версии 16.0a14, доступный для скачивания со страницы загрузки и дистрибутивного каталога. Ключевое обновление — важные исправления безопасности в базе Firefox (переход на 153.5.0esr), а также обновление Tor до 0.4.9.14 и NoScript. Это альфа-версия, предназначенная исключительно для тестирования: она с большей вероятностью содержит ошибки, влияющие на удобство, безопасность и приватность. Пользователям, которым критична надёжная анонимность, рекомендуется пользоваться стабильной веткой. В сборке также обновлены инструменты: GeckoView, Go (до 1.26.9), Lyrebird (до 0.9.0) и отдельные компоненты системы сборки. Разработчики просят сообщество присылать обратную связь и сообщения об ошибках. Полный список изменений доступен в changelog.

## Полная статья

Tor Browser 16.0a14 is now available from the[Tor Browser download page](https://www.torproject.org/download/alpha/)and also from our[distribution directory](https://www.torproject.org/dist/torbrowser/16.0a14/).

This version includes important[security updates](https://www.mozilla.org/en-US/security/advisories/)to Firefox.

⚠️**Reminder**: The Tor Browser Alpha release-channel is for[testing only](https://community.torproject.org/user-research/become-tester/). As such, Tor Browser Alpha is not intended for general use because it is more likely to include bugs affecting usability, security, and privacy.

If you are an at-risk user, require strong anonymity, or just want a reliably-working browser, please stick with the[stable release channel](https://www.torproject.org/download/).

## Send us your feedback

If you find a bug or have a suggestion for how we could improve this release,[please let us know](https://support.torproject.org/misc/bug-or-feedback/).

## Full changelog

The[full changelog](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/raw/main/projects/browser/Bundle-Data/Docs-TBB/ChangeLog.txt)since Tor Browser 16.0a13 is:

- All Platforms

- Updated NoScript to 13.6.36.90201984
- Updated Tor to 0.4.9.14
- [Bug tor-browser#45380](https://gitlab.torproject.org/tpo/applications/tor-browser/-/issues/45380): Rebase Tor Browser alpha onto 153.5.0esr
- [Bug tor-browser#45382](https://gitlab.torproject.org/tpo/applications/tor-browser/-/issues/45382): Backport Security Fixes from 158

- Windows + macOS + Linux

- Updated Firefox to 153.5.0esr

- Android

- Updated GeckoView to 153.5.0esr

- Build System

- All Platforms

- [Bug tor-browser-build#41885](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/issues/41885): Update changelogs as a last step in relprep.py
- [Bug tor-browser-build#41896](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/issues/41896): Update Lyrebird to 0.9.0
- [Bug tor-browser-build#41898](https://gitlab.torproject.org/tpo/applications/tor-browser-build/-/issues/41898): Separate MB and TB toolchain archives in[https://nightlies.tbb.torproject.org/nightly-builds/toolchains/](https://nightlies.tbb.torproject.org/nightly-builds/toolchains/)

- Windows + Linux + Android

- Updated Go to 1.26.9

## Оригинал

https://blog.torproject.org/new-alpha-release-tor-browser-160a14/
