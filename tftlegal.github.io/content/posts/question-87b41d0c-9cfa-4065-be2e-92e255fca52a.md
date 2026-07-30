---
title: "Infrastructure as Code: Полное руководство по Ansible и автоматизации"
summary: "На основе материалов курса Ansible: Infrastructure as Code подготовлен цикл статей, охватывающий все ключевые аспекты управления инфраструктурой через код. Материал структурирован от базовых концепций до продвинутых практик тестирования и безопасности."
tags: [IaC, todo]
date: 2026-07-25T20:30:07Z
tldr: "933820173"
---

---

## Статья 1. Введение в Infrastructure as Code и Ansible

### Что такое Infrastructure as Code?

**Infrastructure as Code (IaC)** — это подход к управлению IT-инфраструктурой, при котором конфигурация серверов, сетей и других компонентов описывается в виде кода. Вместо ручного выполнения команд на серверах, инженеры пишут декларативные или императивные сценарии, которые затем автоматически применяются к целевым системам.

**Ключевые преимущества IaC:**

| Преимущество | Описание |
|--------------|----------|
| **Воспроизводимость** | Один и тот же код гарантированно создаёт идентичные окружения |
| **Версионирование** | Все изменения инфраструктуры хранятся в Git, доступна история и откат |
| **Автоматизация** | Устранение человеческих ошибок при рутинных операциях |
| **Масштабируемость** | Легко управлять сотнями и тысячами серверов единообразно |
| **Документация** | Код сам служит актуальной документацией |

### Место Ansible в экосистеме IaC

**Ansible** — это agentless (безагентный) инструмент управления конфигурациями, написанный на Python. В отличие от Puppet или Chef, Ansible не требует установки агентов на управляемые хосты — он использует SSH для подключения и выполнения задач.

**Архитектура Ansible:**

```
┌─────────────────┐     SSH      ┌──────────────────┐
│  Control Node   │──────────────│   Managed Host   │
│  (Ansible)      │  Python      │   (Target)       │
└─────────────────┘  Модули      └──────────────────┘
```

**Основные компоненты:**

1. **Control Node** — машина с установленным Ansible, с которой выполняются плейбуки
2. **Inventory** — файл, содержащий список управляемых хостов и их параметры
3. **Modules** — готовые скрипты для выполнения конкретных задач (установка пакетов, управление файлами и т.д.)
4. **Playbooks** — сценарии в формате YAML, описывающие желаемое состояние системы
5. **Roles** — переиспользуемые наборы задач, организованные по стандартной структуре

### Почему Ansible?

- **Простота** — используется YAML, не требующий глубоких знаний программирования
- **Идемпотентность** — многократный запуск плейбука не приводит к нежелательным изменениям
- **Обширная экосистема** — тысячи готовых модулей и ролей в Ansible Galaxy
- **Кроссплатформенность** — работает с Linux, Windows, сетевыми устройствами, облачными провайдерами

> **Важно:** Ansible следует парадигме **декларативного подхода** — вы описываете *что* должно быть, а не *как* это сделать. Система сама определяет, нужно ли выполнять действие для достижения желаемого состояния.

### Основные задачи, которые решает Ansible

1. **Установка и настройка ПО** (LEMP-стек, базы данных, веб-серверы)
2. **Управление конфигурациями** — единообразная настройка сотен серверов
3. **Деплой приложений** — автоматизированное развертывание новых версий
4. **Оркестрация** — координация действий на множестве хостов
5. **Проверка состояния** — мониторинг и сбор фактов о системе

---

## Статья 2. Подготовка среды: VirtualBox, Vagrant и первый запуск

### Инструменты для локальной разработки

Для изучения Ansible не требуется сразу поднимать облачные сервера. Локальное окружение позволяет безопасно экспериментировать и отлаживать плейбуки.

#### VirtualBox

**VirtualBox** — гипервизор (система виртуализации) Type 2, позволяющий запускать гостевые ОС внутри основной (host) системы.

**Особенности:**
- Бесплатный и кроссплатформенный
- Поддерживает большинство популярных ОС
- Интегрируется с Vagrant для автоматизации

> **Важно:** Для работы VirtualBox в BIOS/UEFI должна быть включена аппаратная виртуализация (Intel VT-x или AMD-V).

#### Vagrant

**Vagrant** — утилита от HashiCorp для управления виртуальными машинами через конфигурационный файл `Vagrantfile`. Она позволяет создавать и настраивать ВМ одной командой.

**Ключевые понятия:**

| Термин | Описание |
|--------|----------|
| **Provider** | Система виртуализации (VirtualBox, VMWare) |
| **Box** | Базовый образ виртуальной машины с предустановленной ОС |
| **Vagrantfile** | Ruby-файл с описанием конфигурации ВМ |

**Установка Vagrant на Ubuntu 24.04:**

```bash
# Обновление системы
sudo apt update && sudo apt upgrade -y

# Установка VirtualBox
sudo apt install virtualbox -y

# Добавление пользователя в группу sudo и смена на vagrant
sudo usermod -aG sudo vagrant
su vagrant

# Скачивание и установка Vagrant (актуальная версия 2.4.1)
wget https://hashicorp-releases.yandexcloud.net/vagrant/2.4.1/vagrant_2.4.1-1_amd64.deb
sudo dpkg -i vagrant_2.4.1-1_amd64.deb
```

> **Важно для РФ:** Из-за ограничений доступа к HashiCorp репозиториям, в `Vagrantfile` необходимо указать зеркало:
> ```ruby
> ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'
> ```

### Основные команды Vagrant

| Команда | Действие |
|---------|----------|
| `vagrant init` | Создать Vagrantfile |
| `vagrant up` | Запустить/создать ВМ |
| `vagrant ssh` | Подключиться к ВМ по SSH |
| `vagrant suspend` | Приостановить ВМ |
| `vagrant reload` | Перезагрузить ВМ с применением изменений из Vagrantfile |
| `vagrant destroy` | Удалить ВМ |
| `vagrant status` | Показать статус ВМ |

### Пример Vagrantfile с двумя машинами

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_SERVER_URL'] = "https://vagrant.elab.pro"

Vagrant.configure("2") do |config|
  # Контрольная нода
  config.vm.define "controlnode" do |controlnode|
    controlnode.vm.box = "bento/ubuntu-24.04"
    controlnode.vm.hostname = "controlnode"
    controlnode.vm.network "private_network", ip: "192.168.50.4"
    controlnode.vm.synced_folder "./ansible", "/home/vagrant/ansible"
  end

  # Управляемый хост
  config.vm.define "server" do |server|
    server.vm.box = "bento/ubuntu-24.04"
    server.vm.hostname = "server"
    server.vm.network "private_network", ip: "192.168.50.5"
  end
end
```

> **Примечание:** `synced_folder` позволяет монтировать папку с хостовой машины внутрь ВМ — удобно для разработки плейбуков в редакторе на основной системе.

---

## Статья 3. Ansible: архитектура, установка и первые шаги

### Установка Ansible

**Рекомендуемая версия:** Ansible Community 2.12+ (использует Ansible Core 2.12)

**Установка на Ubuntu/Debian:**

```bash
# Добавление официального PPA
sudo apt-add-repository ppa:ansible/ansible -y
sudo apt update

# Установка Ansible и sshpass (для работы с паролями)
sudo apt install ansible sshpass -y

# Проверка установки
ansible --version
```

**Установка на CentOS/RHEL:**

```bash
# Установка EPEL-репозитория
sudo yum install epel-release -y
# Установка Ansible
sudo yum install ansible -y
```

### Архитектура Ansible в деталях

```
                  ┌─────────────────────────────┐
                  │          User / CLI         │
                  └──────────────┬──────────────┘
                                 │
                  ┌──────────────▼──────────────┐
                  │      Ansible Engine         │
                  ├─────────────────────────────┤
                  │  ┌─────────┐ ┌──────────┐   │
                  │  │Inventory│ │ Playbooks│   │
                  │  └─────────┘ └──────────┘   │
                  │  ┌─────────┐ ┌──────────┐   │
                  │  │ Modules │ │ Plugins  │   │
                  │  └─────────┘ └──────────┘   │
                  └──────────────┬──────────────┘
                                 │ SSH / WinRM
          ┌──────────────────────┼──────────────────────┐
          │                      │                      │
┌─────────▼─────────┐ ┌─────────▼─────────┐ ┌─────────▼─────────┐
│   Managed Host 1  │ │   Managed Host 2  │ │   Managed Host N  │
│   (Linux/Windows) │ │   (Linux/Windows) │ │   (Linux/Windows) │
└───────────────────┘ └───────────────────┘ └───────────────────┘
```

### Модули vs Плагины

| Аспект | Модули | Плагины |
|--------|--------|---------|
| **Назначение** | Выполняют конкретные задачи на хостах | Расширяют функциональность самого Ansible |
| **Где выполняются** | На удалённых хостах | На контрольной ноде |
| **Примеры** | `apt`, `copy`, `service` | `inventory`, `connection`, `callback` |
| **Язык** | Python (в основном) | Python |

### Первый инвентарь и тестовый запуск

**Файл инвентаря `hosts.ini` (формат INI):**

```ini
# Простой инвентарь
server ansible_host=192.168.50.5 ansible_user=vagrant ansible_password=vagrant

# Группы хостов
[web]
web1 ansible_host=192.168.50.5
web2 ansible_host=192.168.50.6

[db]
db1 ansible_host=192.168.50.7

[all:vars]
ansible_user=vagrant
ansible_password=vagrant
```

**Тестирование подключения (модуль `ping`):**

```bash
# Проверка доступности хостов
ansible all -i hosts.ini -m ping

# Вывод:
# 192.168.50.5 | SUCCESS => {
#     "ansible_facts": {
#         "discovered_interpreter_python": "/usr/bin/python3"
#     },
#     "changed": false,
#     "ping": "pong"
# }
```

### Ad-hoc команды в Ansible

Ad-hoc команды полезны для быстрых операций без написания плейбука:

```bash
# Установка пакета на всех хостах
ansible all -i hosts.ini -m apt -a "name=nginx state=present" --become

# Проверка использования диска
ansible all -i hosts.ini -m shell -a "df -h"

# Перезагрузка сервиса
ansible web -i hosts.ini -m service -a "name=nginx state=restarted" --become
```

---

## Статья 4. Написание плейбуков: от простого к сложному

### Что такое плейбук?

**Плейбук (Playbook)** — это сценарий в формате YAML, описывающий желаемое состояние системы. Каждый плейбук состоит из одного или нескольких **плеев (plays)**, каждый из которых определяет:

- **hosts** — на каких хостах выполнять
- **become** — нужно ли повышать привилегии
- **tasks** — список задач для выполнения
- **vars** — переменные для этого плея

### Структура плейбука

```yaml
---
# Заголовок плейбука
- name: "LEMP Stack Installation"
  hosts: web
  become: true
  vars:
    php_version: 8.1
    nginx_port: 80

  tasks:
    # Задача 1: Установка Nginx
    - name: "Install Nginx"
      ansible.builtin.apt:
        name: nginx
        state: latest
        update_cache: true

    # Задача 2: Удаление дефолтной страницы
    - name: "Remove default website"
      ansible.builtin.file:
        path: /var/www/html
        state: absent

    # Задача 3: Копирование собственного сайта
    - name: "Deploy custom website"
      ansible.builtin.copy:
        src: files/html/
        dest: /var/www/html/
        owner: vagrant
        group: vagrant
        mode: '0644'

    # Задача 4: Установка MySQL
    - name: "Install MySQL server"
      ansible.builtin.apt:
        name: mysql-server
        state: latest
```

### Запуск плейбука

```bash
ansible-playbook -i hosts.ini playbook.yml
```

### Ключевые модули для повседневной работы

#### 1. `apt` / `yum` — управление пакетами

```yaml
# Ubuntu/Debian
- name: "Install packages"
  ansible.builtin.apt:
    name:
      - nginx
      - mysql-server
      - php-fpm
    state: present
    update_cache: true

# CentOS/RHEL
- name: "Install packages"
  ansible.builtin.yum:
    name: nginx
    state: present
```

#### 2. `copy` — копирование файлов

```yaml
- name: "Copy configuration file"
  ansible.builtin.copy:
    src: files/myapp.conf
    dest: /etc/myapp/myapp.conf
    owner: root
    group: root
    mode: '0644'
    backup: yes  # Создаёт резервную копию при изменении
```

#### 3. `template` — копирование с подстановкой переменных (Jinja2)

```yaml
- name: "Deploy nginx config with variables"
  ansible.builtin.template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/sites-available/default.conf
    owner: root
    group: root
    mode: '0644'
```

**Шаблон Jinja2 (`nginx.conf.j2`):**
```nginx
server {
    listen {{ nginx_port }};
    server_name {{ domain_name }};
    root {{ web_root }};
    index index.html index.php;
}
```

#### 4. `file` — управление файлами и директориями

```yaml
- name: "Create application directory"
  ansible.builtin.file:
    path: /var/www/myapp
    state: directory
    owner: www-data
    group: www-data
    mode: '0755'
```

#### 5. `service` — управление сервисами

```yaml
- name: "Start and enable Nginx"
  ansible.builtin.service:
    name: nginx
    state: started    # started / stopped / restarted / reloaded
    enabled: true     # добавить в автозагрузку
```

#### 6. `lineinfile` / `blockinfile` — редактирование файлов

```yaml
# Замена строки в конфигурационном файле
- name: "Update listen address"
  ansible.builtin.lineinfile:
    path: /etc/mysql/mysql.conf.d/mysqld.cnf
    regexp: '^bind-address'
    line: 'bind-address = 0.0.0.0'
    backup: yes

# Добавление блока конфигурации
- name: "Add database replication config"
  ansible.builtin.blockinfile:
    path: /etc/postgresql/14/main/postgresql.conf
    block: |
      wal_level = replica
      max_wal_senders = 3
      wal_keep_segments = 8
    marker: "# {mark} REPLICATION CONFIG"
```

### Циклы в плейбуках

```yaml
- name: "Install multiple PHP extensions"
  ansible.builtin.apt:
    name: "{{ item }}"
    state: present
  with_items:
    - php-fpm
    - php-mysql
    - php-curl
    - php-gd
    - php-mbstring
```

### Условное выполнение (`when`)

```yaml
- name: "Install MySQL on Ubuntu"
  ansible.builtin.apt:
    name: mysql-server
    state: present
  when: ansible_os_family == "Debian"

- name: "Install MariaDB on CentOS"
  ansible.builtin.yum:
    name: mariadb-server
    state: present
  when: ansible_os_family == "RedHat"
```

---

## Статья 5. Переменные, Jinja2 и масштабирование с помощью ролей

### Переменные в Ansible: где и как их использовать

**Приоритет переменных (от высшего к низшему):**

1. Переданные через командную строку (`-e`)
2. Переменные инвентаря (`host_vars`, `group_vars`)
3. Переменные плейбука (`vars`)
4. Переменные ролей (`vars`)
5. Переменные по умолчанию (`defaults`)

**Примеры объявления переменных:**

```yaml
# В инвентаре (hosts.ini)
web1 ansible_host=192.168.50.5 app_port=8080

# В групповых переменных (group_vars/web.yml)
---
nginx_port: 80
php_version: 8.1

# В плейбуке
---
- hosts: web
  vars:
    domain: example.com
    app_env: production
```

### Работа с фактами (Facts)

Ansible автоматически собирает информацию о хостах (facts) перед выполнением задач:

```yaml
- name: "Print system information"
  ansible.builtin.debug:
    msg: |
      OS: {{ ansible_os_family }}
      Distribution: {{ ansible_distribution }}
      Version: {{ ansible_distribution_version }}
      IP: {{ ansible_default_ipv4.address }}
      Memory: {{ ansible_memtotal_mb }} MB
      CPU: {{ ansible_processor_cores }} cores
```

### Jinja2 — шаблонизатор для динамических конфигураций

**Основные конструкции Jinja2:**

| Конструкция | Назначение | Пример |
|-------------|------------|--------|
| `{{ variable }}` | Вывод переменной | `{{ domain }}` |
| `{% for %}` | Цикл | `{% for host in hosts %}` |
| `{% if %}` | Условие | `{% if env == 'prod' %}` |
| `{% set %}` | Объявление переменной | `{% set port = 8080 %}` |
| `{{ var \| default('value') }}` | Значение по умолчанию | `{{ port \| default(80) }}` |
| `{{ var \| upper }}` | Фильтр (верхний регистр) | `{{ name \| upper }}` |

**Пример шаблона для генерации конфигурации балансировщика:**

```nginx
upstream app_backend {
    {% for server in servers %}
    server {{ server.ip }}:{{ server.port }};
    {% endfor %}
}

server {
    listen 80;
    server_name {{ domain }};
    location / {
        proxy_pass http://app_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Роли (Roles) — организация кода для переиспользования

**Структура роли:**

```
roles/nginx/
├── tasks/
│   └── main.yml          # Основные задачи
├── handlers/
│   └── main.yml          # Обработчики
├── defaults/
│   └── main.yml          # Переменные по умолчанию
├── vars/
│   └── main.yml          # Переменные (высокий приоритет)
├── files/                # Статические файлы
├── templates/            # Jinja2-шаблоны
└── meta/
    └── main.yml          # Метаданные и зависимости
```

**Пример роли для Nginx (`roles/nginx/tasks/main.yml`):**

```yaml
---
# Роль: установка и настройка Nginx
- name: "Install Nginx"
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: true
  notify: restart nginx

- name: "Remove default site"
  ansible.builtin.file:
    path: /etc/nginx/sites-enabled/default
    state: absent
  notify: restart nginx

- name: "Deploy site configuration"
  ansible.builtin.template:
    src: site.conf.j2
    dest: "/etc/nginx/sites-available/{{ domain }}.conf"
  notify: restart nginx

- name: "Enable site"
  ansible.builtin.file:
    src: "/etc/nginx/sites-available/{{ domain }}.conf"
    dest: "/etc/nginx/sites-enabled/{{ domain }}.conf"
    state: link
  notify: restart nginx
```

**Обработчики (`roles/nginx/handlers/main.yml`):**

```yaml
---
- name: restart nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
```

**Переменные по умолчанию (`roles/nginx/defaults/main.yml`):**

```yaml
---
nginx_port: 80
domain: localhost
web_root: /var/www/html
```

**Использование роли в плейбуке:**

```yaml
---
- name: "Setup web server"
  hosts: web
  become: true
  vars:
    domain: myapp.example.com
    web_root: /var/www/myapp

  roles:
    - nginx
    - mysql
    - php
```

### Ansible Galaxy — каталог готовых ролей

```bash
# Поиск роли
ansible-galaxy search postgresql

# Установка роли
ansible-galaxy role install geerlingguy.postgresql

# Установка из файла зависимостей (requirements.yml)
ansible-galaxy install -r requirements.yml
```

**Файл `requirements.yml`:**
```yaml
---
roles:
  - src: geerlingguy.postgresql
    version: 4.0.2
  - src: https://github.com/UnderGreen/ansible-role-mongodb.git
    name: green.mongodb

collections:
  - name: community.postgresql
    version: 3.3.0
```

---

## Статья 6. Продвинутые техники: работа с базами данных и тестирование

### Управление MySQL через Ansible

**Установка и настройка MySQL:**

```yaml
---
- name: "MySQL setup"
  hosts: db
  become: true

  tasks:
    - name: "Install MySQL and PyMySQL"
      ansible.builtin.apt:
        name:
          - mysql-server
          - python3-pymysql
        state: present
        update_cache: true

    - name: "Set root password"
      community.mysql.mysql_user:
        name: root
        password: "{{ mysql_root_password }}"
        login_unix_socket: /var/run/mysqld/mysqld.sock
        check_implicit_admin: true

    - name: "Remove anonymous users"
      community.mysql.mysql_user:
        name: ''
        host_all: yes
        state: absent
        login_user: root
        login_password: "{{ mysql_root_password }}"

    - name: "Create application database"
      community.mysql.mysql_db:
        name: "{{ app_db_name }}"
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"

    - name: "Create application user"
      community.mysql.mysql_user:
        name: "{{ app_db_user }}"
        password: "{{ app_db_password }}"
        priv: "{{ app_db_name }}.*:ALL"
        host: '%'
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"
```

### Обработка ошибок с `block` / `rescue` / `always`

```yaml
- name: "Database migration with rollback"
  block:
    - name: "Apply migrations"
      community.postgresql.postgresql_query:
        db: myapp
        query: "{{ lookup('file', 'migrations/001_create_users.sql') }}"

    - name: "Verify migration"
      community.postgresql.postgresql_query:
        db: myapp
        query: "SELECT COUNT(*) FROM users"
      register: result
      fail:
        when: result.query_result[0].count < 100

  rescue:
    - name: "Rollback migration"
      community.postgresql.postgresql_query:
        db: myapp
        query: "DROP TABLE IF EXISTS users"
      notify: report_failure

  always:
    - name: "Clean up temporary files"
      ansible.builtin.file:
        path: /tmp/migrations
        state: absent
```

### Тестирование ролей с Molecule

**Molecule** — инструмент для тестирования Ansible ролей, поддерживающий различные драйверы (Docker, Vagrant, AWS, Podman).

**Установка:**

```bash
pip install molecule molecule-docker
```

**Инициализация роли с тестами:**

```bash
molecule init role myapp.nginx
```

**Структура сценария Molecule:**

```
molecule/
└── default/
    ├── molecule.yml      # Конфигурация теста
    ├── converge.yml      # Плейбук для применения роли
    ├── verify.yml        # Проверки после применения
    └── prepare.yml       # Подготовка окружения
```

**Файл `molecule/default/molecule.yml`:**

```yaml
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: instance
    image: quay.io/centos/centos:stream8
    pre_build_image: true
    privileged: true
provisioner:
  name: ansible
verifier:
  name: ansible
```

**Файл `molecule/default/converge.yml`:**

```yaml
---
- name: "Apply role"
  hosts: all
  become: true
  vars:
    nginx_port: 8080
    domain: test.local

  roles:
    - myapp.nginx
```

**Файл `molecule/default/verify.yml` (проверки):**

```yaml
---
- name: "Verify Nginx installation"
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: "Check Nginx is running"
      ansible.builtin.service:
        name: nginx
        state: started
      register: service_status
      failed_when: not service_status.status.SubState == 'running'

    - name: "Check Nginx listens on port"
      ansible.builtin.wait_for:
        port: 8080
        timeout: 10

    - name: "Check website responds"
      ansible.builtin.uri:
        url: http://localhost:8080
        status_code: 200

    - name: "Check Nginx config syntax"
      ansible.builtin.shell: nginx -t
      changed_when: false
      register: config_test
      failed_when: "'test is successful' not in config_test.stdout"
```

**Запуск тестов:**

```bash
# Полный цикл тестирования
molecule test

# Отдельные этапы
molecule create   # Создать инстансы
molecule converge # Применить роль
molecule verify   # Запустить проверки
molecule destroy  # Уничтожить инстансы
```

---

## Статья 7. Безопасность: Ansible Vault и защита данных

### Ansible Vault — шифрование чувствительных данных

**Ansible Vault** позволяет шифровать переменные и файлы, содержащие пароли, ключи и другие секреты.

#### Шифрование строковых переменных

```bash
# Шифрование одной переменной
ansible-vault encrypt_string --vault-id dev@pass 'super_secret_password' --name 'db_password'

# Результат:
db_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          66386439653236336... (зашифрованные данные)
```

**Использование в плейбуке:**
```yaml
---
- hosts: db
  become: true
  vars:
    db_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          66386439653236336...

  tasks:
    - name: "Set MySQL password"
      community.mysql.mysql_user:
        name: app_user
        password: "{{ db_password }}"
        priv: '*.*:ALL'
```

#### Шифрование целых файлов

```bash
# Зашифровать файл
ansible-vault encrypt group_vars/production --vault-id prod@pass

# Расшифровать (для редактирования)
ansible-vault decrypt group_vars/production --vault-id prod@pass

# Редактировать зашифрованный файл
ansible-vault edit group_vars/production --vault-id prod@pass

# Сменить пароль
ansible-vault rekey group_vars/production --vault-id dev@pass --new-vault-id prod@pass
```

#### Запуск плейбука с расшифровкой

```bash
# С использованием файла с паролем
ansible-playbook -i production playbook.yml --vault-id prod@password-file

# С запросом пароля в интерактивном режиме
ansible-playbook -i production playbook.yml --vault-id prod@prompt
```

### Лучшие практики безопасности

1. **Храните пароль Vault отдельно** — не в репозитории
2. **Используйте разные vault-id** для разных окружений
3. **Применяйте `no_log: true`** для задач, выводящих секреты:

```yaml
- name: "Set database password"
  community.mysql.mysql_user:
    name: app_user
    password: "{{ db_password }}"
  no_log: true  # Не показывать в выводе
```

4. **Используйте динамическое получение паролей** через скрипты:

```bash
# Вместо статического файла с паролем
ansible-playbook playbook.yml --vault-id vault@/usr/local/bin/get_vault_password.sh
```

---

## Статья 8. Оптимизация производительности Ansible

### Ключевые методы ускорения

#### 1. Pipelining (SSH-конвейер)

Сокращает число SSH-соединений, выполняя копирование и выполнение модуля в одном соединении.

**Включение в `ansible.cfg`:**
```ini
[ssh_connection]
pipelining = True
```

> **Внимание:** Может конфликтовать с `requiretty` в `/etc/sudoers`. Отключите `requiretty` для пользователя Ansible.

#### 2. Увеличение количества потоков (`forks`)

```ini
[defaults]
forks = 50  # Количество параллельных соединений
```

Или через командную строку:
```bash
ansible-playbook playbook.yml -f 50
```

#### 3. Стратегии выполнения (strategies)

| Стратегия | Описание |
|-----------|----------|
| `linear` | По умолчанию — ждёт завершения задачи на всех хостах |
| `free` | Задачи выполняются на хостах независимо, без ожидания |
| `host_pinned` | Все задачи выполняются на хосте, затем переходят к следующему |
| `debug` | Linear + отладчик |

```yaml
---
- name: "Fast deployment"
  hosts: all
  strategy: free
  serial: 30%  # Запуск на 30% хостов одновременно
  max_fail_percentage: 20%  # Остановка при 20% ошибок

  tasks:
    - name: "Install common packages"
      ansible.builtin.apt:
        name: "{{ item }}"
        state: present
      loop:
        - htop
        - git
        - curl
```

#### 4. Мультиплексирование SSH

Повторное использование SSH-соединений:

```ini
[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
control_path = ~/.ansible/cp/%%h-%%p-%%r
```

#### 5. Асинхронные задачи

```yaml
- name: "Long running task (async)"
  ansible.builtin.shell: |
    /usr/local/bin/long_running_script.sh
  async: 3600        # Максимальное время выполнения (сек)
  poll: 0            # 0 = не ждать, запустить в фоне
  register: long_task

- name: "Check task status"
  ansible.builtin.async_status:
    jid: "{{ long_task.ansible_job_id }}"
  register: job_result
  until: job_result.finished
  retries: 60        # Количество попыток проверки
  delay: 10          # Интервал между проверками
```

#### 6. Сбор фактов с умом

```yaml
---
- name: "Optimized fact gathering"
  hosts: all
  gather_facts: true
  gather_subset:
    - '!all'          # Не собирать всё
    - '!hardware'     # Не собирать информацию о железе
    - '!network'      # Не собирать сетевые данные
    - min             # Собрать только минимальный набор

  tasks:
    - name: "Use limited facts"
      ansible.builtin.debug:
        msg: "OS: {{ ansible_distribution }}"
```

### Плагин Mitogen — ускорение до 7 раз

**Установка:**
```bash
pip install mitogen
```

**Настройка в `ansible.cfg`:**
```ini
[defaults]
strategy_plugins = /usr/local/lib/python3.10/dist-packages/ansible_mitogen/plugins/strategy
strategy = mitogen_linear  # или mitogen_free
```

### Callback-плагины для мониторинга производительности

```ini
[defaults]
# Включение таймера и профилирования ролей
callback_whitelist = timer, profile_roles
```

**Использование:**
```bash
ansible-playbook playbook.yml
# В конце вывода появится статистика времени выполнения
```

---

## Статья 9. Интеграция и автоматизация в CI/CD

### Ansible в GitLab CI/CD

**Пример `.gitlab-ci.yml`:**

```yaml
---
image: quay.io/ansible/creator-ee:v24.2.0

stages:
  - test
  - deploy

variables:
  ANSIBLE_HOST_KEY_CHECKING: "false"
  ANSIBLE_FORCE_COLOR: "true"

workflow:
  rules:
    - if: $CI_MERGE_REQUEST_ID
      when: never
    - if: $CI_COMMIT_BRANCH == "main"

# Тестирование Molecule
test-molecule:
  stage: test
  script:
    - microdnf -y install sshpass
    - ansible-galaxy collection install community.mysql
    - pip install molecule molecule-docker
    - cd roles/myapp
    - molecule test
  only:
    changes:
      - roles/**/*

# Деплой в staging
deploy-staging:
  stage: deploy
  script:
    - ansible-playbook -i inventory/staging playbook.yml
  environment:
    name: staging
  only:
    - main

# Деплой в production (только с ручным подтверждением)
deploy-production:
  stage: deploy
  script:
    - ansible-playbook -i inventory/production playbook.yml --vault-id prod@$VAULT_PASSWORD_FILE
  environment:
    name: production
  when: manual
  only:
    - main
```

### Динамический инвентарь (Yandex Cloud пример)

**Плагин инвентаря `plugins/inventory/yandex_cloud.py`:**

```python
from ansible.plugins.inventory import BaseInventoryPlugin
import requests
import json

class InventoryModule(BaseInventoryPlugin):
    NAME = 'yandex_cloud'

    def verify_file(self, path):
        return path.endswith('yandex_cloud.yml')

    def parse(self, inventory, loader, path, cache=True):
        super(InventoryModule, self).parse(inventory, loader, path, cache)

        config = self._read_config_data(path)
        token = config.get('yandex_token')
        folder_id = config.get('folder_id')

        # Получение списка ВМ через API Yandex Cloud
        url = f"https://compute.api.cloud.yandex.net/compute/v1/instances?folderId={folder_id}"
        headers = {"Authorization": f"Bearer {token}"}

        response = requests.get(url, headers=headers)
        if response.status_code != 200:
            self.display.warning(f"Failed to fetch VMs: {response.text}")
            return

        instances = response.json().get('instances', [])

        for vm in instances:
            # Извлечение IP-адреса
            address = vm['networkInterfaces'][0]['primaryV4Address']['address']

            # Добавление хоста в инвентарь
            self.inventory.add_host(vm['id'])
            self.inventory.set_variable(vm['id'], 'ansible_host', address)

            # Добавление в группу по меткам
            for label, value in vm.get('labels', {}).items():
                self.inventory.add_group(label)
                self.inventory.add_child(label, vm['id'])
```

**Конфигурационный файл для плагина (`inventory/yandex_cloud.yml`):**

```yaml
---
plugin: yandex_cloud
yandex_token: "{{ vault_yandex_token }}"
folder_id: "b1gu10dj1kuvvscc6mch"
```

### Ansible Pull — подход "pull-based"

**Ansible Pull** позволяет хостам самостоятельно забирать плейбуки из Git и применять их:

```bash
# Базовое использование
ansible-pull -U https://github.com/company/ansible.git -d /etc/ansible playbook.yml

# С SSH-ключом (приватный репозиторий)
ansible-pull -U git@github.com:company/ansible.git -d /etc/ansible -i inventory/production playbook.yml

# По расписанию (cron)
# /etc/cron.d/ansible-pull
0 * * * * root /usr/bin/ansible-pull -U git@github.com:company/ansible.git -d /etc/ansible playbook.yml
```

### Ansible Tower / AWX

**AWX** — open-source версия Ansible Tower, предоставляющая:

- Веб-интерфейс для управления плейбуками
- REST API для интеграции
- RBAC (управление доступом)
- Расписание задач
- Журналирование и аудит

**Основные компоненты:**

1. **Projects** — коллекции плейбуков из Git
2. **Inventories** — управление хостами (статический/динамический)
3. **Credentials** — безопасное хранение учетных данных
4. **Job Templates** — шаблоны запуска плейбуков
5. **Workflows** — последовательное выполнение нескольких шаблонов

**Пример API-запроса для запуска задания:**

```bash
curl -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"limit": "web-servers"}' \
  https://awx.example.com/api/v2/job_templates/5/launch/
```

---

## Заключение

Infrastructure as Code с использованием Ansible предоставляет мощный инструментарий для автоматизации управления IT-инфраструктурой. От локальных тестов с Vagrant до масштабных деплоев в облаках — Ansible остаётся одним из наиболее доступных и эффективных решений.

**Ключевые выводы:**

1. **Идемпотентность** — фундаментальное свойство, гарантирующее стабильность при многократных запусках
2. **Декларативный подход** — описание желаемого состояния проще и надёжнее императивных скриптов
3. **Тестирование** — Molecule и CI/CD обеспечивают качество и предотвращают регрессии
4. **Безопасность** — Ansible Vault и управление доступом защищают чувствительные данные
5. **Масштабируемость** — от одной машины до тысяч хостов с помощью правильных стратегий и оптимизаций

Инструмент эволюционирует, но принципы остаются неизменными: автоматизация, воспроизводимость и документированность инфраструктуры через код.



