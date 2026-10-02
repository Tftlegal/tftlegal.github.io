---
title: "Проект Argon Forge опубликовал оптимизированные сборки Mesa, Gamescope, DXVK, VKD3D-Proton и Wine"
date: 2026-10-02T07:08:23Z
source: "opennetru"
original_url: "https://www.opennet.ru/opennews/art.shtml?num=66380"
summary: "Argon Forge — проект, создающий оптимизированные сборки ключевых компонентов графического стека Linux: Mesa, Gamescope, DXVK, VKD3D-Proton и Wine. Пакеты распространяются через APT-репозитории, размещённые на GitHub Pages. Сборки ориентированы на процессоры с поддержкой AVX2 (x86-64-v3) и AVX-512 (x86-64-v4). Проект предназначен для пользователей Devuan 6 «Excalibur» и Debian 13 «Trixie», которые хотят получить максимальную производительность в играх. Пакеты используют библиотечную базу этих выпусков и не подходят для более старых версий дистрибутивов. Скрипты сборки и конфигурации GitHub Actions распространяются под лицензией MIT."
---

# Проект Argon Forge опубликовал оптимизированные сборки Mesa, Gamescope, DXVK, VKD3D-Proton и Wine

## Краткое содержание

Argon Forge — проект, создающий оптимизированные сборки ключевых компонентов графического стека Linux: Mesa, Gamescope, DXVK, VKD3D-Proton и Wine. Пакеты распространяются через APT-репозитории, размещённые на GitHub Pages. Сборки ориентированы на процессоры с поддержкой AVX2 (x86-64-v3) и AVX-512 (x86-64-v4). Проект предназначен для пользователей Devuan 6 «Excalibur» и Debian 13 «Trixie», которые хотят получить максимальную производительность в играх. Пакеты используют библиотечную базу этих выпусков и не подходят для более старых версий дистрибутивов. Скрипты сборки и конфигурации GitHub Actions распространяются под лицензией MIT.

## Полная статья

Представлен проект[Argon Forge](https://github.com/argonforge/), в рамках которого формируются оптимизированные сборки ключевых компонентов графического стека Linux:[Mesa](https://github.com/argonforge/mesa-builds),[Gamescope](https://github.com/argonforge/gamescope-builds),[DXVK](https://github.com/argonforge/dxvk-builds),[VKD3D-Proton](https://github.com/argonforge/vkd3d-proton-builds)и[Wine](https://github.com/argonforge/wine-builds). Сборки распространяются через APT-репозитории, размещённые на GitHub Pages, и ориентированы на процессоры с поддержкой AVX2 (x86-64-v3) и AVX-512 (x86-64-v4). Проект нацелен на пользователей дистрибутивов Devuan 6 "Excalibur" и Debian 13 "Trixie", желающих получить максимум производительности от графического стека в играх. Пакеты используют библиотечную базу этих выпусков и не предназначены для установки на более старые версии дистрибутивов. Сборочные скрипты и конфигурации GitHub Actions распространяются под лицензией MIT.

Каждый проект публикуется в двух вариантах, различающихся целевым набором инструкций и суффиксом версии пакета:

- stable-avx2 (суффикс "avx2") - для процессоров с AVX2, BMI и FMA. Подходит для большинства x86-64 CPU с 2013 года и новее.
- stable-avx512 (суффикс "avx512") - для процессоров с AVX-512, включая AMD Zen 4 и новее.

Для установки необходимо импортировать GPG-ключ репозитория и добавить соответствующий codename в "sources.list". Пример для Mesa:

```

   curl -fsSL https://argonforge.github.io/mesa-builds/public.asc \
     | sudo gpg --dearmor -o /usr/share/keyrings/argonforge-mesa.gpg
   echo "deb [signed-by=/usr/share/keyrings/argonforge-mesa.gpg] https://argonforge.github.io/mesa-builds stable-avx512 main" \
     | sudo tee /etc/apt/sources.list.d/argonforge-mesa.list
   sudo apt update
   sudo apt install mesa-libgallium mesa-vulkan-drivers libgl1-mesa-dri
```

Аналогичный подход применяется для остальных проектов: достаточно заменить URL и имя файла ключа. Пакеты собраны с использованием Clang и флагов "-march=x86-64-v3 -mtune=znver3" или "-march=x86-64-v4 -mtune=znver4", а также "-O3" и набора флагов выравнивания. Для Wine дополнительно применяется кросс-компиляция Windows-компонентов через llvm-mingw.

Сборка выполняется автоматически в GitHub Actions при создании тега или ручном запуске. Каждый релиз сопровождается двумя архивами с deb-пакетами для соответствующего варианта ISA. Помимо APT-репозитория, пакеты доступны в разделе "Releases" соответствующих репозиториев. Сборочные скрипты Argon Forge не изменяют лицензии исходных проектов и лишь формируют бинарные пакеты с иными параметрами компиляции.

## Оригинал

https://www.opennet.ru/opennews/art.shtml?num=66380
