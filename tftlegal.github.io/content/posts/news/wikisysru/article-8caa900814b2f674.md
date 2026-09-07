---
title: "7. Daemonset"
date: 2026-02-03T19:00:00Z
source: "wikisysru"
original_url: "https://wikisys.ru/7-daemonset/"
summary: "DaemonSet — это объект Kubernetes, который гарантирует запуск одного пода на каждом узле кластера.   Он применяется, когда нужен агент или сервис на каждой ноде, например для мониторинга, сбора логов или сетевых функций.   При добавлении новой ноды в кластер Kubernetes автоматически создаёт соответствующий под на ней.   Если нода удаляется или перестаёт быть частью кластера, её поды DaemonSet удаляются.   В отличие от Deployment, DaemonSet не управляет общим числом реплик, а привязывает поды к узлам.   Поэтому он подходит для системных задач, которые должны выполняться на всех узлах одновременно."
---

# 7. Daemonset

## Краткое содержание

DaemonSet — это объект Kubernetes, который гарантирует запуск одного пода на каждом узле кластера.  
Он применяется, когда нужен агент или сервис на каждой ноде, например для мониторинга, сбора логов или сетевых функций.  
При добавлении новой ноды в кластер Kubernetes автоматически создаёт соответствующий под на ней.  
Если нода удаляется или перестаёт быть частью кластера, её поды DaemonSet удаляются.  
В отличие от Deployment, DaemonSet не управляет общим числом реплик, а привязывает поды к узлам.  
Поэтому он подходит для системных задач, которые должны выполняться на всех узлах одновременно.

## Полная статья

Представим, что нам необходимо мониторить все узлы в кластере Kubernetes. Для того, чтобы снимать с нод информацию, на каждой ноде нам потребуется агент мониторинга. Что нам необходимо от таких агентов на узлах:

- Агенты на всех узлах должны запускаться автоматически
- Должно быть централизованное управление агентами
- Конфигурация агентов должна быть из одной точки, чтобы не ходить по всем узлам, раскладывая конфиги

`DaemonSet`— это контроллер Kubernetes, который гарантирует, что на каждой (или определенной) ноде кластера запущен экземпляр пода.

Можно думать о нем как о «системном демоне» в мире Kubernetes.

Когда мы добавляем новую ноду в кластер, DaemonSet автоматически запускает на ней под. Когда нода удаляется, под удаляется сборщиком мусора.

Для чего используется``DaemonSet``:

- Сбор логов — на каждой ноде нужен агент, который собирает логи со всех контейнеров и отправляет в центральное хранилище. Пример: fluentd или filebeat на каждой ноде.
- Мониторинг****— агенты мониторинга (например, Prometheus Node Exporter), собирающие метрики с каждой ноды.
- Сетевые плагины (CNI) — компоненты сетевых плагинов (Calico, Flannel, Cilium) часто работают как DaemonSet, потому как они должны быть на каждой ноде для настройки сетевых правил.
- Хранилище — демоны для работы с хранилищами (например, создание iSCSI-подключений или монтирование сетевых дисков).
- Безопасность и управление — агенты безопасности, сканеры уязвимостей, инструменты для настройки sysctl параметров ядра.

Манифест`DaemonSet`очень похож на`Deployment`с той лишь разницей, что в нем нет раздела`replicas`. Всё дело в том, что количество экземпляров`DaemonSet`подов по умолчанию равно количеству нод. Если быть точнее, то количество подов`DaemonSet`автоматически поддерживается равным количеству нод, удовлетворяющих условиям селектора и tolerations, которые указаны в манифесте.

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd:latest
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

После создания`DaemonSet`с помощью`kubectl apply -f`или после назначения необходимой метки ноде, под на нодах с нужным`nodeSelector`будет создан автоматически.

`DaemonSet`идеально подходят для инфраструктурных задач, связанных с работой на каждом узле. Вместо того чтобы запускать агента на каждом узле вручную через`systemd`, такие агенты описываются в виде`DaemonSet`, и Kubernetes сам заботится о его размещении и поддержании в рабочем состоянии.

**Полезные команды**

```
# Посмотреть все DaemonSets
kubectl get daemonsets -A

# Посмотреть, на каких нодах запущен под DaemonSet
kubectl get pods -o wide -l name=fluentd
```

## 2. Особенности манифеста DaemonSet

Поды в шаблоне (`template.spec`) манифеста`DaemonSet`часто используют`hostPath`для доступа к файловой системе ноды,`hostNetwork: true`для использования сети ноды, а также`hostPID: true`для доступа к процессам ноды.

**Раздел hostPath**

`hostPath`монтирует папку с ноды внутрь контейнера с агентом. Это объясняется тем, что агентам нужно читать логи узла, смотреть его параметры или настраивать его файлы. Иными словами, DaemonSet-агент должен иметь возможность зайти на каждую ноду и примонтироваться к её файловой системе для того, чтобы считать логи.

```
volumes:  # Описываем том из файловой системы узла
- name: varlog
  hostPath:
    path: /var/log  # Папка на узле (сервере)
volumeMounts:  # Монтируем этот том в контейнер
- name: varlog
  mountPath: /var/log  # В контейнере эта папка появится здесь
```

Таким образом, когда агент, например, Fluentd (сборщик логов) пишет в`/var/log`внутри контейнера — на самом деле данные сохраняются в`/var/log`на узле. Контейнер видит файлы узла.

![](/news/wikisysru/article-8caa900814b2f674/image-01.png)

- `volumes`— описывает, где и какие данные существуют на узле (или в кластере). Это объявление тома.
- `volumeMounts`— подключает (монтирует) описанный том в конкретную папку внутри контейнера.
- ``hostPath.type``— это механизм проверки существования пути на узле до запуска пода. Если поле`type`не указано (или указано как`type: ""`— пустая строка), то никакой проверки не выполняется. В независимости от того, существует ли путь монтирования, под все равно запустится.

**Раздел****hostNetwork**

По умолчанию раздел`hostNetwork`имеет значение «false», что означает: под получает свой собственный изолированный IP-адрес, т.е. это классическая сетевая изоляция пода. Однако, например, сетевым плагинам (Calico, Flannel), а также Prometheus Node Exporter (для сбора метрик с сетевых интерфейсов самого узла), нужно иметь доступ к сетевым интерфейсам именно на ноде. В ином случае наш агент или плагин будут видеть только виртуальный интерфейс пода. Значит, нам нужен доступ по IP-адресу ноды с пода, на котором расположен контейнер с сетевым плагином или агентом мониторинга. Следовательно, в разделе`hostNetwork`спецификации пода манифеста`DaemonSet`необходимо устанавливать значение`true`:

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-proxy  # Сетевой компонент Kubernetes
spec:
  template:
    spec:
      hostNetwork: true  # КЛЮЧЕВОЙ ПАРАМЕТР
      containers:
      - name: kube-proxy
        image: kube-proxy:latest
```

Таким образом, pod получает IP-адрес самой ноды, а не внутренний IP. Если нода имеет адрес`192.168.1.10`, то и Pod будет доступен по этому же адресу.

**Раздел******hostPID****

С разделом`hostPID`схожая ситуация. Необходимо устанавливать значение`true`для того, чтобы агент в контейнере на поде видел все процессы, запущенные на ноде, а не только свои:

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter  # Сборщик метрик
spec:
  template:
    spec:
      hostPID: true  # PID Namespace Sharing (Общее пространство PID)
      containers:
      - name: node-exporter
        image: prom/node-exporter
        args:
        - --path.procfs=/host/proc  # Смотрим процессы хоста
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
      volumes:
      - name: proc
        hostPath:
          path: /proc  # Каталог с процессами на узле
```

Таким образом, внутри контейнера команда`ps aux`покажет все процессы узла и контейнера.

## 3. Механизмы планирования (Scheduling) запуска подов

**NodeSelector**

`nodeSelector`— это самый простой и понятный способ сказать Kubernetes: «запусти этот Pod только на узлах (Nodes), у которых есть определенная метка». Иными словами это работает так:

- Администратор назначает на ноды лейблы, например:`kubectl label nodes node-name disktype=ssd`
- Затем, в спецификации пода манифеста`DaemonSet`указывается`nodeSelector`как ограничение, на каких именно узлах должны запускаться поды.

Без`nodeSelector``DaemonSet`запустит под на каждой ноде в кластере. С`nodeSelector``DaemonSet`запустит под только на нодах, соответствующих селектору.

```
Показать лейблы на ноде:
kubectl get node <node> --show-labels

# Показать лейблы на всех нодах кластера:
kubectl get nodes --show-labels

# Компактный вывод (только имена нод и их лейблы):
kubectl get nodes -o custom-columns=NAME:.metadata.name,LABELS:.metadata.labels

# Показать ноды с конкретным лейблом
kubectl get nodes -L <key>

# Например, лейбл disktype=ssd:
kubectl get nodes -L disktype
```

![](/news/wikisysru/article-8caa900814b2f674/image-02.png)

Назначение лейблов на ноды происходит так:

```
Назначить лейбл на ноду:
kubectl label node <node> <key=value>

# Пример:
kubectl label node k3d-node2-0 disktype=ssd

# Переписать существующий лейбл:
kubectl label node <node> <key=value> --overwrite
```

Манифест пода с`nodeSelector`выглядит так:

```
spec:
  template:
    spec:
      nodeSelector:
        disktype: ssd   # Pod запустится только на узлах с этой меткой
```

На практике, когда кластеры становятся большими и сложными, используются более гибкие инструменты:`Affinity`и`Anti-Affinity`, а также`Taints`и`Tolerations`.

**Taints и Tolerations**

![](/news/wikisysru/article-8caa900814b2f674/image-03.png)

`Taints`и`Tolerations`— это механизм, с помощью которого можно запретить подам запускаться на определенных нодах.

`Taint`можно перевести, как «зараза», а`Tolerations`— как «иммунитет» или «сопротивляемость».

На ноде назначается`Taint`, например:

- Имя`Taint`— node-role.kubernetes.io/master
- Значение «true»
- effect — NoSchedule

Теперь шедуллер не будет распределять на этой ноде поды, у которых нет «иммунитета» к «заразе» с именем node-role.kubernetes.io/master. Эта сопротивляемость указывается в разделе`Toleration`манифеста`DaemonSet`.

Существует ещё более сильный effect — NoExecute. Если эффект NoSchedule действует только на запуск новых подов, то, при объявлении эффекта NoExecute, поды, которые уже работают на ноде, но не имеют нужных`Tolerations`, будут с этого узла «эвакуированы».

Сводная таблица эффектов:

| Эффект | Поведение | Пример использования |
| --- | --- | --- |
| NoSchedule | Новые поды без toleration не назначаются | Обычные ноды не пускают случайные поды |
| PreferNoSchedule | Мягкая версия, стараться избегать | Редко используется |
| NoExecute | + выселяет существующие поды без toleration | Срочные работы на ноде, эвакуация |

Назначение Taint на ноду выполняется с помощью команды`kubectl taint nodes`:

```
Синтаксис:

kubectl taint nodes <node-name> <key>=<value>:<effect>
```

```
# Назначаем Taint на ноду master-node:
kubectl taint nodes master-node node-role.kubernetes.io/master=true:NoSchedule

# Назначаем Taint с эффектом NoExecute:
kubectl taint nodes special-node dedicated=special:NoExecute

# Посмотреть Taints на ноде:
kubectl describe node master-node | grep Taints
```

| Часть | Значение | Пояснение |
| --- | --- | --- |
| key: dedicated | Имя «заразы» | Например: тип оборудования, назначение ноды |
| value: special | Конкретное значение | Например: special, gpu, monitoring |
| effect: NoExecute | Действие | Выселять поды без toleration |

Значения`key`и`value`позволяют создавать разные категории «зараз». Например:

| Taint | Значение | Эффект | Что означает |
| --- | --- | --- | --- |
| dedicated | special | NoExecute | Нода только для special-задач, все остальные выселить |
| dedicated | gpu | NoSchedule | Нода только для GPU-задач, новые без toleration не пускать |
| dedicated | ssd | PreferNoSchedule | Желательно не пускать сюда обычные поды |

**Operator**

`operator`определяет правило сравнения. Его можно использовать для`tolerations`,`nodeAffinity`и`nodeSelector`.`operator`говорит Kubernetes, как именно сравнивать ключ (key) и значение (value) лейбла или «заразы» (taint).

- 1.`Equal`— значение должнно совпадать (по умолчанию):

```
tolerations:
- key: "disktype"
  operator: "Equal"    # Требует точного совпадения значения
  value: "ssd"         # Сработает только если на ноде есть taint disktype=ssd
  effect: "NoSchedule"
```

- 2.`Exists`— проверяет только наличие ключа (значение не важно):

```
tolerations:
- key: "disktype"
  operator: "Exists"   # НЕ смотрит на значение! Подходит ЛЮБОЕ disktype
  effect: "NoSchedule" # Сработает и для disktype=ssd, и для disktype=hdd
```

- 3. Без key — toleration для любых taints с любым именем и значением

```
tolerations:
- operator: "Exists"   # Без key —  толерантен к любым taints (опасно!)
  # Такой под запустится вообще везде, даже на сломанных нодах
```

Или толерантен к любым taints с любым именем и значением, но с эффектом NoSchedule:

```
tolerations:
- operator: "Exists"
  effect: "NoSchedule"  # Только для taints с эффектом NoSchedule
```

Таким образом под будет иметь сопротивляемость к любым taints, у которых эффект равен NoSchedule.

| Ситуация | Что используем | Почему |
| --- | --- | --- |
| На мастер-нодах стандартная «зараза» node-role.kubernetes.io/master:NoSchedule без значения | operator: Exists | Неизвестно, какое там значение (его вообще нет), нужно просто игнорировать сам факт наличия «заразы» |
| Своя «зараза» fast-storage=ssd:NoSchedule | operator: Equal | Нужно пропустить только те поды, которые явно просят SSD |
| Хотим запустить под везде, игнорируя все «заразы» | operator: Exists (без key) | Аварийный отладчик, который должен попасть на любую ноду |

Ещё один пример для закрепления:

```
tolerations:
- key: "node.kubernetes.io/unreachable"
  operator: "Exists"    # Игнорируем, что нода недоступна
  effect: "NoExecute"
  tolerationSeconds: 60 # Но только на 60 секунд
```

Пример манифеста DaemonSet с`Tolerations`для master-нод:

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-master
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      tolerations:                    # ИММУНИТЕТ к "заразам"
      - key: node-role.kubernetes.io/master
        operator: Equal
        value: "true"
        effect: NoSchedule             # Иммунитет к NoSchedule
      
      - key: dedicated
        operator: Equal
        value: "special"
        effect: NoExecute              # Иммунитет к NoExecute
      
      - key: node.kubernetes.io/not-ready
        operator: Exists
        effect: NoExecute
        tolerationSeconds: 300         # 5 минут терпения
        
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.16
```

Разные варианты Tolerations:

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-agent
spec:
  selector:
    matchLabels:
      app: monitor
  template:
    metadata:
      labels:
        app: monitor
    spec:
      tolerations:
      # Вариант 1: Точное совпадение
      - key: node-role.kubernetes.io/master
        operator: Equal
        value: "true"
        effect: NoSchedule
      
      # Вариант 2: Любое значение (главное, чтобы ключ был)
      - key: gpu
        operator: Exists      # Достаточно наличия ключа
        effect: NoSchedule
      
      # Вариант 3: Все эффекты с ключом
      - key: dedicated
        operator: Exists
        effect: ""            # Пустой effect = любой эффект
      
      # Вариант 4: Терпимость к недоступности ноды
      - key: node.kubernetes.io/unreachable
        operator: Exists
        effect: NoExecute
        tolerationSeconds: 60  # 60 секунд терпения до эвакуации
      
      containers:
      - name: agent
        image: prom/node-exporter
```

**Проверка результатов:**

```
# 1. Проверить Taints на нодах
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints

# 2. Проверить, запустился ли DaemonSet на нужных нодах
kubectl get pods -l app=unified-agent -o wide

# 3. Увидеть tolerations в работающем поде
kubectl get pod unified-agent-xxxxx -o yaml | grep tolerations -A 10
```

## Оригинал

https://wikisys.ru/7-daemonset/
