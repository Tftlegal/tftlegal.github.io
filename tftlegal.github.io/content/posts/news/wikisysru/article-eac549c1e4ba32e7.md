---
title: "6. Kubernetes. Deployment и replicaset"
date: 2026-02-03T18:00:00Z
source: "wikisysru"
original_url: "https://wikisys.ru/6-kubernetes-deployment-%d0%b8-replicaset/"
summary: "Deployment и ReplicaSet — это два важных контроллера Kubernetes, которые работают под управлением kube-controller-manager.   На нижнем уровне приложения состоят из pod'ов, представляющих объединение контейнеров.   ReplicaSet обеспечивает постоянное наличие заданного количества копий pod'ов.   Deployment управляет ReplicaSet'ами и позволяет задавать целевое состояние приложения.   Это упрощает развертывание, масштабирование и обновление приложений.   Благодаря Deployment можно выполнять постепенные обновления и откат к предыдущим версиям."
---

# 6. Kubernetes. Deployment и replicaset

## Краткое содержание

Deployment и ReplicaSet — это два важных контроллера Kubernetes, которые работают под управлением kube-controller-manager.  
На нижнем уровне приложения состоят из pod'ов, представляющих объединение контейнеров.  
ReplicaSet обеспечивает постоянное наличие заданного количества копий pod'ов.  
Deployment управляет ReplicaSet'ами и позволяет задавать целевое состояние приложения.  
Это упрощает развертывание, масштабирование и обновление приложений.  
Благодаря Deployment можно выполнять постепенные обновления и откат к предыдущим версиям.

## Полная статья

Среди множества kube-controllers, которыми управляет компонент`kube-controller-manager`, существуют 2 важных контроллера, которые обеспечивают развёртывание приложения в kubernetes:

- `deployment-controller`: отвечает за работу с декларативным описанием приложения для его последующего развёртывания и обновления. Сущность, которой оперирует`deployment-controller`называется`deployment`.`deployment`в своей работе использует концепцию набора репликаций —`replicaset`.
- `replicaset-controller`: отвечает за работу с`replicaset`.`replicaset`— сущность kubernetes, отвечающая за поддержку стабильного набора репликаций приложения, работающих в один момент времени.

На самом низком уровне представлены`pods`— объединение контейнеров, вместе представляющих собой приложение. Под — это минимальная и самая простая единица в Kubernetes.

Контроллер, который следит за определенным набором подов —`replicaset`. Он гарантирует, что в любой момент времени работает заданное количество идентичных подов (реплик). Если под умирает,`replicaset`немедленно создает новый. Если подов больше, чем нужно —`replicaset`удалит лишние экземпляры.

`deployment`— это контроллер более высокого уровня, который управляет репликасетами.

Таким образом, работа со всем приложением происходит на уровне деплоймента.

![](/news/wikisysru/article-eac549c1e4ba32e7/image-01.png)

`deployment-controller`способен самостоятельно создать`deployment`на основе образа (docker image), а может использовать готовое описание`deployment`(manifest) в формате yaml или json:

```
kubectl create deployment mariadb --image=mariadb:4.0
```

```
kubectl apply -f nginx-deployment-manifest.yaml
```

`deployment-controller`через сущность`deployment`работает только с объединением контейнеров (`pods`) и наборами их репликаций (`replicasets`).

Для связи`pod`с другими`pods`и внешним миром используется сущность`service`.

`pod`связывается с`service`через`endpoint`. Обеспечением работы с`endpoints`занимается`endpoint-controller`.

Создать`service`можно, выполнив`expose`существующего`deployment`или использовать готовое описание`service`(manifest) в формате yaml или json:

```
kubectl expose deployment mariadb --port=8080
```

```
kubectl apply -f nginx-service-manifest.yaml
```

За счёт связки`deployment`—`replicaset`в кластере kubernetes гарантируется выполнение следующих операций:

- Создание развёртываний
- Обновление без downtime (rolling update)
- Откат текущего обновления (rollback)
- Просмотр истории обновлений (revision)
- Откат приложения до конкретной ревизии (rollback to revision)
- Скалирование и дескалирование (scaling)

## 2. Структура Deployment в Kubernetes

- `apiVersion: apps/v1`— версия API Kubernetes для ресурса Deployment. Определяет, какие поля доступны и как API их обрабатывает.
- `kind: Deployment`— тип ресурса Kubernetes. Указывает, что это именно Deployment, а не Pod, Service, ConfigMap и т.п. Контроллеры Kubernetes смотрят на этот параметр, чтобы понимать, как обрабатывать манифест.
- **Секция metadata (метаданные)**

- `name: nginx-deployment`— уникальное имя деплоймента в namespace. Должно быть уникальным в рамках namespace.
- `namespace: production`— пространство имен (логическое разделение). Позволяет изолировать ресурсы (разные окружения, команды). Если не указано — используется неймспейс`default`.
- `labels`— произвольные пары «ключ-значение» для идентификации и группировки. Используются для селекторов (нахождения ресурсов), мониторинга и логирования, связывания ресурсов (Service → Pod). Основные подкатегории`labels`:`app`,``version``,`env`,`component`. Какого-либо предопределенного списка или стандарта не существует, можно использовать любые ключи и значения. Но есть общепринятые соглашения и рекомендации. Например:

```
metadata:
  labels:
    app: nginx
    version: "1.21"
    env: production
    component: frontend
```

где

- `labels:`

- `app`— это имя приложения\микросервиса. Используется для основного поиска и группировки. Часто совпадает с именем Deployment.
- `version`/`release`/`app.kubernetes.io/version`— версия приложения. Используется для развертываний, rollback, мониторинга версий.
- `env`/`environment`/`tier`— окружение (prod) или уровень приложения (frontend).
- `component`— компонент в рамках приложения. То есть в одном приложении (`app: ecommerce`) могут быть разные компоненты.
- `managed-by`/`app.kubernetes.io/managed-by`— инструмент управления (например, helm, terraform, argocd и т.п.)
- `part-of`/`app.kubernetes.io/part-of`— к какой более крупной системе принадлежит.
- ``instance``— конкретный экземпляр (для stateful-приложений). Например,`postgres-primary`,`kafka-broker-1`,`redis-cluster-01`.
- `team`/`owner`— команда-владелец.

**Секция metadata (продолжение разбора данной секции)**

- `metadata:`

- `annotations`— дополнительная информация (не для селекторов). Просто комментарии.

**Секция spec (спецификация)**

- `spec:`

- `replicas`— количество реплик.
- `selector`— механизм, который связывает Deployment с подами, которыми он должен управлять. Это правило поиска для нахождения нужных подов.
- `selector.`matchLabels``— фильтр, который говорит деплойменту управлять всеми подами, у которых точно есть эти лейблы:

![](/news/wikisysru/article-eac549c1e4ba32e7/image-02.png)

То есть деплоймент`nginx`должен управлять всеми подами в namespace`default`, у которых есть лейбл`app`со значением`nginx`.

- ``spec:``

- `template`— шаблон пода.
- `metadata.labels`— это метки, которые будут у создаваемых подов. Эти метки используются сервисами (Service) для маршрутизации трафика.

```
template:  metadata:    labels:      app: nginx        # Должен совпадать с selector!      version: v1      env: prod
```

Если лейблов более одного, то для``matchLabels``под должен будет удовлетворять требованиям всех лейблов одновременно.

- ``spec:``

- `template.spec`— основное тело пода.
- `containers[].name`— имя контейнера внутри пода.
- `template`— шаблон пода.
- `metadata.labels`— это метки, которые будут у создаваемых подов. Эти метки используются сервисами (Service) для маршрутизации трафика.

```
containers:
- name: nginx
```

- ``spec:``

- `template.spec`
- `containers[].image`— Docker-образ контейнера.

```
containers:
- image: nginx:1.21.3-alpine
```

Типы форматов для`containers[].image`:

- `nginx`— последний тег из Docker Hub
- `nginx:1.21`— конкретная версия
- `nginx:1.21-alpine`— версия с Alpine Linux
- `myregistry.com/project/app:v2.1`— приватный registry
- `redis@sha256:abc123...`— по хешу (immutable)

Продолжаем рассматривать секцию``spec:``

- ``spec:``

- `template.spec`
- `containers[].imagePullPolicy`— политика загрузки образа.

Типы политик:

- `Always`— всегда тянет из registry (для latest тега)
- `IfNotPresent`— только если нет локально (по умолчанию для тегов)
- `Never`— использует только локальный образ

Для тега`:latest`по умолчанию применяется`Always`.

- ``spec:``

- `template.spec`
- `containers[].ports`— порт, который слушает контейнер.

```
ports:
- containerPort: 80
  name: http
  protocol: TCP
- containerPort: 443
  name: https
```

Это публичные порты контейнера.

- `containerPort`— порт, который слушает контейнер
- `name`— имя порта (используется в Service)
- `protocol`— TCP (по умолчанию) или UDP
- Не открывает порт на ноде. Только информация для Service.

- ``spec:``

- `template.spec`
- `containers[].env`— переменные окружения.

```
env:
- name: DATABASE_URL
  value: "postgres://user:pass@db:5432/app"
- name: LOG_LEVEL
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: log-level
- name: SECRET_KEY
  valueFrom:
    secretKeyRef:
      name: app-secrets
      key: api-key
- name: MY_POD_IP
  valueFrom:
    fieldRef:
      fieldPath: status.podIP
```

Объяснение:

- Прямое значение через`value`
- Из ConfigMap через`valueFrom.configMapKeyRef`
- Из Secret через`valueFrom.secretKeyRef`
- Из полей Pod через`valueFrom.fieldRef`:

- ``spec:``

- `template.spec`
- `containers[].resources`— запросы и лимиты CPU/памяти.

```
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

``requests``— гарантированные ресурсы. Используется шедуллером для размещения на нодах.

``limits``— максимальные ресурсы. При превышении CPU — throttling (замедление). Приложение не крашится, но начинает работать медленнее. При превышении памяти — OOM Kill (контейнер немедленно убивается).

- ``spec:``

- `template.spec`
- `containers[].livenessProbe`— проверка, живо ли приложение

```
livenessProbe:
  httpGet:
    path: /health
    port: 8080
    httpHeaders:
    - name: Custom-Header
      value: Awesome
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
  successThreshold: 1
```

- ``spec:``

- `template.spec`
- `containers[].readinessProbe`— проверка готовности принимать трафик.

```
readinessProbe:
  tcpSocket:
    port: 3306
  initialDelaySeconds: 5
  periodSeconds: 10
```

- ``spec:``

- `template.spec`
- `containers[].startupProbe`— проверка, что приложение запустилось.

```
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

`startupProbe`используется для медленно стартующих приложений. Пока`startupProbe`не пройдет — liveness/readiness не запускаются.

- ``spec:``

- `template.spec`
- `restartPolicy`— политика перезапуска подов.

```
spec:
  restartPolicy: Always
```

- `Always`— всегда перезапускать (по умолчанию для Deployment)
- `OnFailure`— перезапускать только при ошибке
- `Never`— никогда не перезапускать

- **Секция**strategy**(****стратегия обновления****)**.

- `type:`RollingUpdate``— указывает, как обновлять поды при изменении Deployment. По умолчанию, да и в 90% случаев используется стратегия`RollingUpdate`.

```
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 25%
    maxUnavailable: 25%
```

Такую стратегию стоит использовать, если приложение может одновременно работать как со старой, так и с новой версией.

`RollingUpdate`работает следующим образом. Сперва`Deployment`создает (через`ReplicaSet`) новый Pod с новой версией. Ждет, пока он станет Ready. После этого удаляет старый под. Затем это действие повторяется для всех остальных подов. Плюсы такого подхода заключаются в том, что приложение обновляется без простоя по времени и продолжает оставаться работоспособным.

То есть`Deployment`создает новый`ReplicaSet`с новым шаблоном пода (новой версией).`ReplicaSet`нового поколения создает новые поды с новой версией.`Deployment`управляет масштабированием двух`ReplicaSet`: постепенно увеличивает количество реплик в новом`ReplicaSet`и постепенно уменьшает количество реплик в старом`ReplicaSet`.

`Kubelet`на каждом узле отвечает за здоровье подов (liveness/readiness probes).

Когда все поды нового`ReplicaSet`готовы (`Ready`), старый`ReplicaSet`масштабируется до 0.

- `maxSurge`— максимальное количество подов относительно текущего значения`replicas`. То есть, если в поле`replicas`было задано 2 реплики, а`maxSurge: 1`, значит, в рамках данной стратегии обновления, можно добавить одну новую реплику поверх двух старых. После прохождения проб новой реплики, одна старая будет удалена. Затем, при необходимости, будет добавлена новая одна реплика и так далее.

- Число:`1`,`2`
- Процент:`25%`(значение по умолчанию),`50%`
- Пример: при`replicas: 4`и`maxSurge: 1`— максимум 5 подов

- `maxUnavailable`— максимальное количество недоступных подов. Это означает, на сколько реплик можно уменьшить текущее количество работающих подов относительно желаемого состояния (`replicas`) во время обновления.

- `0`— гарантированная доступность (но медленнее)
- `1`или`25%`— быстрее, но возможна частичная недоступность

В случае, если желаемое состояние`replicas`— несколько подов, имеет смысл обновлять по нескольку подов сразу. Например, наше значение`replicas: 10`. Тогда с`maxUnavailable: 1`и`maxSurge: 1`обновление займет 10 циклов. Каждый цикл = создание пода + пробы + удаление пода ≈ 30-60 секунд. Итого: 5-10 минут обновления, и всегда минимум 9 из 10 подов будут доступны. Рекомендуемый баланс в данном случае:

```
replicas: 10
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 2    # 20% могут быть недоступны
    maxSurge: 2          # +20% дополнительных подов
```

Таким образом мы добьёмся сокращения времени обновления в два раза: до 5 циклов вместо 10, что займёт 2,5-5 минут вместо 5-10 при условии всегда минимум 8 работающих подов (80% capacity).
Это лишь пример того, насколько данные параметры могут играть важную роль на скорость обновления деплоймента. Однако для продакшена, где важна доступность, рекомендованный сценарий для 10 реплик будет такой:

```
maxUnavailable: 1    # Максимум 1 недоступный (90% capacity)
maxSurge: 2          # +2 для быстрого прогрева
minReadySeconds: 30  # Даем прогреть кэши
```

**Стратегия`Recreate`— простая, но агрессивная стратегия развертывания**.

Применяется для stateful или миграций БД, а также на тестовых и dev-средах (для ускорения обновления), где постоянная доступность не так важна. Также используется в тех случаях, когда приложение не поддерживает параллельное выполнение нескольких версий и требует полной остановки старой версии перед использованием новой.

- `type:`Recreate``

```
strategy:
  type: Recreate
```

Принцип работы:`Deployment`удаляет все старые поды, затем создаёт новые. В отличие от`RollingUpdate`, влечёт за собой downtime на время обновления.

- `revisionHistoryLimit`— сколько хранить старых ревизий`ReplicaSet`.

Каждое изменение`Deployment`создает новый`ReplicaSet`. Старые`ReplicaSet`позволяют сделать rollback на предыдущие доступные версии ревизий. По умолчанию хранится 10 ревизий.

- `minReadySeconds`— минимальное время, которое под должен быть Ready.

Этот параметр даёт приложению время «прогреться», предотвращает преждевременное удаление старых подов. На практике применяется для для JVM-приложений.

- `progressDeadlineSeconds`— максимальное время на обновление`Deployment`.

Если обновление не завершилось за это время — считается failed. По умолчанию: 600 секунд (10 минут). После этого можно сделать rollback.

Полный пример`Deployment`с комментариями:

```
apiVersion: apps/v1
kind: Deployment
metadata:
name: web-api
namespace: production
labels:
app: api-server
version: v2.1
annotations:
git.commit: "abc123def"
deployment.timestamp: "2024-01-15T10:30:00Z"
spec:
replicas: 3  # Три копии для отказоустойчивости
  selector:
matchLabels:
app: api-server  # Ищет поды с этой меткой
revisionHistoryLimit: 5  # Храним 5 последних версий для rollback
  strategy:
type: RollingUpdate
rollingUpdate:
maxSurge: 1        # Одновременно может быть на 1 под больше
maxUnavailable: 0  # Все поды должны быть доступны
  minReadySeconds: 30    # Под должен быть готов 30 секунд
  progressDeadlineSeconds: 300  # 5 минут на деплой
  template:
metadata:
labels:
app: api-server  # Должно совпадать с selector
        version: v2.1
env: production
spec:
containers:
- name: api
image: myregistry.com/api:v2.1.5
imagePullPolicy: IfNotPresent
ports:
- name: http
containerPort: 8080
protocol: TCP
env:
- name: NODE_ENV
value: "production"
- name: DB_HOST
valueFrom:
configMapKeyRef:
name: app-config
key: database.host
resources:
requests:
memory: "256Mi"
cpu: "200m"
limits:
memory: "512Mi"
cpu: "500m"
livenessProbe:
httpGet:
            path: /health
port: 8080
initialDelaySeconds: 45
periodSeconds: 15
readinessProbe:
httpGet:
path: /ready
port: 8080
initialDelaySeconds: 5
periodSeconds: 5
startupProbe:
httpGet:
path: /startup
port: 8080
failureThreshold: 30
periodSeconds: 10
        volumeMounts:
- name: config-volume
mountPath: /etc/config
volumes:
- name: config-volume
configMap:
name: app-config
restartPolicy: Always
terminationGracePeriodSeconds: 30
```

Рекомендации:

- Всегда указывайте тег образа (не`:latest`)
- Обязательно настраивайте ресурсы (requests/limits)
- Настраивайте probes (liveness/readiness)
- Используйте RollingUpdate с maxUnavailable: 0 для критичных сервисов
- Метки должны совпадать между selector и template
- Храните секреты в Secret, конфиги в ConfigMap
- Указывайте namespace явно
- Используйте readinessProbe для stateful сервисов (БД, кэши)

## Оригинал

https://wikisys.ru/6-kubernetes-deployment-%d0%b8-replicaset/
