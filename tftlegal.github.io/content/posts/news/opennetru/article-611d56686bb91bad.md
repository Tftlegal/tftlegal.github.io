---
title: "Выпуск miracle-wm 0.11, композитного менеджера на базе Wayland и Mir"
date: 2026-09-17T12:36:49Z
source: "opennetru"
original_url: "https://www.opennet.ru/opennews/art.shtml?num=66296"
summary: "Вышел релиз композитного менеджера miracle-wm версии 0.11. Проект работает под Wayland и использует компоненты Mir. Он поддерживает мозаичную компоновку окон, похожую на i3 и Sway. В качестве панели можно применять Waybar. Код написан на C++ и распространяется под GPLv3. Доступны сборки в формате snap, а также пакеты rpm и deb для Fedora и Ubuntu."
---

# Выпуск miracle-wm 0.11, композитного менеджера на базе Wayland и Mir

## Краткое содержание

Вышел релиз композитного менеджера miracle-wm версии 0.11.
Проект работает под Wayland и использует компоненты Mir.
Он поддерживает мозаичную компоновку окон, похожую на i3 и Sway.
В качестве панели можно применять Waybar.
Код написан на C++ и распространяется под GPLv3.
Доступны сборки в формате snap, а также пакеты rpm и deb для Fedora и Ubuntu.

## Полная статья

[Опубликован](https://github.com/miracle-wm-org/miracle-wm/releases/tag/v0.11.0)выпуск композитного менеджера[miracle-wm 0.11](https://miracle-wm.org/), использующего протокол Wayland и компоненты для построения композитных менеджеров[Mir](https://github.com/canonical/mir). Miracle-wm поддерживает мозаичную (tiling) компоновку окон, схожую с аналогичной в проектах[i3](https://i3wm.org/)и[Sway](https://www.opennet.ru/opennews/art.shtml?num=58388). В качестве панели может применяться[Waybar](https://github.com/Alexays/Waybar). Код проекта написан на языке C++ и[распространяется](https://github.com/mattkae/miracle-wm)под лицензией GPLv3. Готовые сборки сформированы в формате[snap](https://snapcraft.io/miracle-wm), а также в пакетах rpm и deb для[Fedora](https://src.fedoraproject.org/rpms/miracle-wm)и[Ubuntu](https://launchpad.net/~matthew-kosarek/+archive/ubuntu/miracle-wm).

Целью miracle-wm является создание композитного сервера, применяющего мозаичное управление окнами, но более функционального и стильного, чем такие продукты, как[Swayfx](https://github.com/WillPower3309/swayfx). При этом проект позволяет использовать и классические приёмы работы с плавающими окнами, например, можно размещать отдельные окна поверх мозаичной сетки или закреплять окна к определённому месту на рабочем столе. Поддерживается виртуальные рабочие столы с возможностью выставления для каждого рабочего стола своего режима работы с окнами по умолчанию (мозаичная компоновка или плавающие окна).

Предполагается, что miracle-wm может оказаться полезным пользователям, которые отдают предпочтение мозаичной компоновке, но желают получить визуальные эффекты и более яркое графическое оформление с плавными переходами и цветами. Конфигурация определяется в формате[YAML](https://wiki.miracle-wm.org/latest/configuration/introduction/). Для установки miracle-wm можно использовать команду "sudo snap install miracle-wm --classic".

Основные изменения:

- Добавлен обзорный режим, упрощающий переключение между окнами и виртуальными рабочими столами. Вызывается одинарными или двойным нажатием клавиши Menu.
- Добавлена возможность привлечь внимание пользователя к неактивному окну, не переключая при этом фокус ввода с активного окна. Например, при поступлении нового сообщения в почтовом клиенте, он может запросить переключение фокуса, после чего miracle выставит окну и виртуальному рабочему столу флаг[urgent](https://wiki.miracle-wm.org/develop/ipc/get_tree/#reply), который может подхватить панель задач для вывода соответствующего индикатора.
- В плагинах реализована возможность[отправки IPC-событий](https://docs.miracle-wm.org/miracle_plugin/plugin/fn.publish_event)и[обработки IPC-команд](https://docs.miracle-wm.org/miracle_plugin/plugin/trait.Plugin#method.handle_command).
- Добавлена настройка[background_color](https://wiki.miracle-wm.org/develop/configuration/background_color/)для задания цвета, применяемого для очистки содержимого.
- В IPC добавлена команда[GET_KEYBINDS](https://wiki.miracle-wm.org/develop/ipc/get_keybinds/)для определения всех действий, привязанных к комбинациям клавиш.
- Задействованы Wayland-протоколы:

- [ext_image_copy_capture_manager_v1 и ext_output_image_capture_source_manager_v1](https://wayland.app/protocols/ext-image-capture-source-v1)для организация захвата контента, выводимого на экран.
- [ext_foreign_toplevel_list_v1](https://gitlab.freedesktop.org/wayland/wayland-protocols/-/tree/main/staging/ext-foreign-toplevel-list)для получение информации о поверхностях, размещённых на самом верхнем уровне (toplevel), которые позволяют организовать закрепление окон поверх другого содержимого, например, для подключения собственных панелей и переключателей окон.
- [ext_data_control_manager_v1](https://gitlab.freedesktop.org/wayland/wayland-protocols/-/tree/main/staging/ext-data-control?ref_type=heads)для управления обработкой данных, например, для реализации менеджеров буфера обмена.
- [ext_input_trigger_action_manager_v1 и ext_input_trigger_registration_manager_v1](https://github.com/canonical/mir/pull/4328)- для обработки глобальных горячих клавиш, кликов мышью и событий сенсорного экрана.

- Дисплейный сервер[Mir](https://github.com/canonical/mir)обновлён до версии[2.29](https://github.com/canonical/mir/releases/tag/v2.29.0).
[

![](/news/opennetru/article-611d56686bb91bad/image-01.png)

](https://github.com/miracle-wm-org/miracle-wm/raw/develop/resources/screenshot1.png)

## Оригинал

https://www.opennet.ru/opennews/art.shtml?num=66296
