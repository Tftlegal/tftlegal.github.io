---
title: "VirtualBox can't operate in VMX root mode"
date: 2026-07-11T05:34:01Z
source: "sidmidru"
original_url: "https://sidmid.ru/virtualbox-cant-operate-in-vmx-root-mode/"
summary: "На Ubuntu при запуске виртуальной машины в VirtualBox возникает ошибка VERR_VMX_IN_VMX_ROOT_MODE. VirtualBox сообщает, что не может работать в VMX root mode. Для устранения проблемы нужно отключить KVM kernel extension, пересобрать ядро и перезагрузить систему. Ошибка означает, что VirtualBox пытается включить VT-x/AMD-V, но процессор уже занят другим гипервизором. Поэтому необходимо проверить, какая программа или модуль использует аппаратную виртуализацию. После освобождения виртуализации VirtualBox сможет запустить машину."
---

# VirtualBox can't operate in VMX root mode

## Краткое содержание

На Ubuntu при запуске виртуальной машины в VirtualBox возникает ошибка VERR_VMX_IN_VMX_ROOT_MODE.
VirtualBox сообщает, что не может работать в VMX root mode.
Для устранения проблемы нужно отключить KVM kernel extension, пересобрать ядро и перезагрузить систему.
Ошибка означает, что VirtualBox пытается включить VT-x/AMD-V, но процессор уже занят другим гипервизором.
Поэтому необходимо проверить, какая программа или модуль использует аппаратную виртуализацию.
После освобождения виртуализации VirtualBox сможет запустить машину.

## Полная статья

Thank you for reading this post, don't forget to subscribe!

возникла проблема на операционке ubuntu при запуске в virtual box машины я получаю ошибку:

VirtualBox can't operate inVMXroot mode. Please disable theKVMkernel extension, recompile your kernel and reboot (VERR_VMX_IN_VMX_ROOT_MODE).
Код ошибки:
NS_ERROR_FAILURE(0X80004005)
Компонент:
ConsoleWrap
Интерфейс:
IConsole {6ac83d89-6ee7-4e33-8ae6-b257b2e81be8}

VirtualBox пытается включить VT-x/AMD-V, но процессор уже занят другим гипервизором

Проверить, кто держитKVM

|  | root@mid:~# lsmod | grep -E '^kvm|kvm_(intel|amd)'lsof /dev/kvm 2>/dev/null || sudo fuser -v /dev/kvmkvm_intel 487424 0kvm 1425408 1 kvm_intel |
| --- | --- |

Выгрузить модулиKVM“навсегда” (чтобыKVMне поднимался после ребута)

tee /etc/modprobe.d/blacklist-kvm.conf >/dev/null <<'EOF'
blacklist kvm
blacklist kvm_intel
blacklist kvm_amd
EOF

update-initramfs -u
reboot

после этого всё ок.

после обновления ядра 6.17.0-23-generic на ubuntu на intell процессоре перестали запускаться виртуалки

|  | VBoxManage: error: Something is not available or not working properly. (VERR_NOT_AVAILABLE) VBoxManage: error: Details: code NS_ERROR_FAILURE (0x80004005), component ConsoleWrap, interface IConsole Waiting for VM "nfs" to power on... |
| --- | --- |

чего только не пробовал , помогла только переустановка

|  | sudo apt purge -y virtualbox virtualbox-dkms virtualbox-qtsudo apt autoremove --purge -y |
| --- | --- |

|  | sudo install -m 0755 -d /etc/apt/keyringswget -qO- https://www.virtualbox.org/download/oracle_vbox_2016.asc \ | sudo gpg --dearmor -o /etc/apt/keyrings/oracle-virtualbox-2016.gpgecho "deb [arch=amd64 signed-by=/etc/apt/keyrings/oracle-virtualbox-2016.gpg] https://download.virtualbox.org/virtualbox/debian noble contrib" \ | sudo tee /etc/apt/sources.list.d/virtualbox.listsudo apt update |
| --- | --- |

|  | sudo apt install -y virtualbox-7.2 |
| --- | --- |

тачки можно даже не экспортировать - они сейвятся

## Навигация по записям

## https://github.com/midnight47/

## Оригинал

https://sidmid.ru/virtualbox-cant-operate-in-vmx-root-mode/
