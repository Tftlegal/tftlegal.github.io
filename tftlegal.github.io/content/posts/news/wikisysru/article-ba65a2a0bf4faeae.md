---
title: "8. Kubernetes. ConfigMap. Secret"
date: 2026-03-08T14:10:06Z
source: "wikisysru"
original_url: "https://wikisys.ru/7-kubernetes-configmap-secret/"
summary: "ConfigMap решает проблему тесной связи конфигурации приложения с конкретным окружением или образом.   Вместо помещения файлов вроде application.properties или config.yml внутрь приложения настройки выносятся в отдельный объект Kubernetes.   Это позволяет использовать один и тот же образ в разных средах, меняя только внешние параметры.   ConfigMap упрощает разделение кода и настроек, а также управление версиями и обновлением конфигурации.   Он помогает стандартизировать деплой и снижает риск ошибок при переносе приложения между окружениями.   Таким образом, ConfigMap обеспечивает гибкость, воспроизводимость и удобство обслуживания конфигураций в контейнерных системах."
---

# 8. Kubernetes. ConfigMap. Secret

## Краткое содержание

ConfigMap решает проблему тесной связи конфигурации приложения с конкретным окружением или образом.  
Вместо помещения файлов вроде application.properties или config.yml внутрь приложения настройки выносятся в отдельный объект Kubernetes.  
Это позволяет использовать один и тот же образ в разных средах, меняя только внешние параметры.  
ConfigMap упрощает разделение кода и настроек, а также управление версиями и обновлением конфигурации.  
Он помогает стандартизировать деплой и снижает риск ошибок при переносе приложения между окружениями.  
Таким образом, ConfigMap обеспечивает гибкость, воспроизводимость и удобство обслуживания конфигураций в контейнерных системах.

## Полная статья

Ранее, в эпоху физических серверов или виртуальных машин, конфигурация приложения часто была тесно связана с окружением. Инженеры могли положить файл`application.properties`или`config.yml`прямо в директорию с приложением на сервере. Если нужно было поменять какой-либо параметр в файле конфигурации, инженер заходил на сервер и правил файл вручную.

По мере популяризации технологии контейнеризации и с появлением kubernetes изменился и сам подход к конфигурированию приложений: индустрия пришла к простому и элегантному решению — отделить код от настроек. Теперь образ содержит только программу. Настройки же подкладываются снаружи в момент запуска.

Таким образом, самым популярным способом становится передача переменных окружения (с настройками, параметрами подключения и т.п.) внутрь контейнера. Объясняется это удобством использования одного докер-образа на всех окружениях сразу (например, Prod, Stage и Test), потому как, если бы конфиг был вшит в образ в момент сбора, пришлось бы собирать такой образ для каждого из окружений.

Kubernetes позволяет задавать переменные окружения для каждого контейнера несколькими способами. Когда Kubernetes запускает контейнер из образа, он подключает к нему`ConfigMap`в виде переменных окружения или файла.

Самый простой способ — переменная и значение добавляются прямо внутрь манифеста пода:

![](/news/wikisysru/article-ba65a2a0bf4faeae/image-01.png)

Однако такой подход обладает несколькими минусами. Самый очевидный из них заключается в следующем. Имея такие жестко заданные значения описания пода, нам потребуется иметь отдельные манифесты для разных сред. Нужно будет создать несколько yaml-манифестов одного пода с разными параметрами, например, параметры подключения к базам данных.

Для того, чтобы повторно использовать одну и ту же спецификацию пода в нескольких окружениях, имеет смысл отделить конфигурацию от описания. Для этого и существует`ConfigMap`: он применяется в качестве источника значений переменных среды.

Как и все объекты в kubernetes,`ConfigMap`представляет собой манифест yaml, в котором можно хранить любую информацию. Такая информация в`ConfigMap`хранится в поле`data`.

Базовая структура`ConfigMap`выглядит так:

```
apiVersion: v1kind: ConfigMapmetadata:  name: my-configdata:  # 1. Простая пара "ключ: значение"  app.properties: |    level=info    language=ru  # 2. Отдельные переменные  database.url: "postgres://db:5432"  max.connections: "100"  debug.enabled: "false"
```

## 2. Способы подключения ConfigMap к манифесту пода

`ConfigMap`можно использовать с подом двумя способами:

- Можно прописать имя`ConfigMap`в блоке`envFrom`. Тогда все её переменные станут доступны в поде.
- Либо прописать значения в блоке`env`. В поле`- name`добавляется новая переменная, а в поле`valueFrom`прописывается, откуда нужно получить значение — из ключа конфигмапа. Имя конфигмап указано в поле`name`секции`configMapKeyRef`, а его ключ — в поле`key`.

![](/news/wikisysru/article-ba65a2a0bf4faeae/image-02.png)

В этом демонстративном случае мы получим 2 переменные с одинаковым значением: JWT_ISSUER и JWT_ISSUER_SECOND, которая была переопределена из JWT_ISSUER.

![](/news/wikisysru/article-ba65a2a0bf4faeae/image-03.png)

Назначение двух таких способов получения значений из`ConfigMap`следующие:

- `envFrom`— для импорта всех переменных из ConfigMap
- `env.valueFrom`— для импорта только отдельных ключей

## 3. Использование одинакового манифеста пода с разной конфигурацией в разных окружениях

Для того, чтобы получать разные значения переменных одного и того же манифеста пода, но в разных окружениях, необходимо выполнить следующие условия:

- Разные окружения (например, Test и Prod) разнести по разным неймспейсам
- Создать одинаковый манифест пода в обоих неймспейсах
- Описать имя`ConfigMap`, одинаковое для обоих окружений
- Создать в обоих неймспейсах одноименные`ConfigMap`, но с разными параметрами, необходимыми для приложения

В части подключения`ConfigMap`разницы между манифестом Pod и Deployment нет никакой. Deployment — это надстройка над Pod’ом. В манифесте Deployment есть поле`template`, внутри которого описывается Pod. Именно в этом`template`(в секции`spec.containers`) и прописываются`env`и`envFrom`. То есть синтаксис полностью идентичен.

## 4. Secret

`Secret`в Kubernetes — это встроенный объект (API resource), предназначенный для хранения и управления конфиденциальной информацией, такой как пароли, токены, ключи SSH, OAuth-токены или данные подключения к базам данных.

`Secret`закрывает задачи безопасного хранения чувствительных данных, отделяя такие данные от манифестов приложений и образов контейнеров.

```
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque  # Самый распространенный тип для произвольных данных
data:
  # Данные должны быть закодированы в base64
  username: YWRtaW4=  # Это "admin" в base64
  password: MWYyZDFlMmU2N2Rm  # Это "1f2d1e2e67df" (просто пример)
```

base64 — это не шифрование, а просто кодирование. Поэтому, если посторонний получит доступ к API Kubernetes и прочитает`Secret`, он легко может декодировать данные.

Объект в Kubernetes называется именно Secret (`kind: Secret`), но манифест может называться как угодно, например:

- Сам файл может называться как угодно, например:
- `secret.yaml`
- `db-secret.yaml`
- `myapp-secret.yml`
- `secrets.yaml`

## 5. Создание Secret

Если`Secret`создаётся с помощью команды``kubectl create secret``, то данные будут закодированы автоматически в base64.

Если же`Secret`создаётся из YAML-файла, то сперва потребуется самостоятельно закодировать значения в base64 либо воспользоваться полем`stringData`.

**1 способ: из YAML-файла**

```
# Сначала создаем YAML-файл, потом применяем
kubectl apply -f secret.yaml -n <namespace>
```

**2 способ: из файлов**

Такой способ создания секрета подразумевает собой, что имя файла станет ключом, а содержимое — значением.

```
# Например, создаём файлы:
echo -n "admin" > username.txt
echo -n "123456" > password.txt

# Создать секрет из файлов выше:
kubectl create secret generic <secret> \
  --namespace=<namespace> \
  --from-file=/path_to/username.txt \
  --from-file=/path_to/password.txt

# Без .txt:
kubectl create secret generic <secret> -n <namespace> \
  --from-file=username=/path_to/username.txt \
  --from-file=password=/path_to/password.txt
```

**3 способ: из литералов (прямо в командной строке)**

```
# Создать секрет из пары ключ=значение:
kubectl create secret generic <secret> \
  --namespace=<namespace> \
  --from-literal=username=admin \
  --from-literal=password=123456
```

**Применение`stringData`**

Данные из`stringData`автоматически кодируются в base64 при применении манифеста:

```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
stringData:           # ✅ Специальное поле для "сырых" строк
  username: admin     # Kubernetes сам закодирует в base64
  password: "123456"    # при применении манифеста
data:
  # тут можно смешивать с уже закодированными данными
  api-key: SkZqbnMzcWRmYXNkZg==
```

## 6. Подключение Secret

Secrets можно подключить к подам двумя основными способами:

- Как переменные окружения (Environment Variables)
- Как файлы в томе (Volume)

**1.1. Подключение`Secret`, как переменные окружения:**

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: myspace
spec:
  containers:
  - name: my-container
    image: nginx
    env:
    - name: DB_USERNAME           # Имя переменной в контейнере
      valueFrom:
        secretKeyRef:
          name: mysecret           # Имя секрета
          key: username            # Ключ из секрета
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: password
    - name: DB_HOST
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: host
```

**1.2. Подключение с помощью envFrom (более компактно):**

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: myspace
spec:
  containers:
  - name: my-container
    image: nginx
    envFrom:
    - secretRef:
        name: mysecret              # Все ключи секрета станут переменными окружения
```

При этом все ключи из Secret автоматически становятся переменными окружения с именами, равными этим ключам (например,`username`→ в env будет`username`). Если потребуется переименовать ключ (например,`username`из Secret →`DB_USERNAME`в env), то придётся использовать именно`secretKeyRef`в`env`, а не`envFrom`.

**2. Подключение как файлы в томе (Volume):**

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: myspace
spec:
  volumes:
  - name: secret-volume              # Имя тома
    secret:
      secretName: mysecret           # Имя секрета
      # Опционально: можно выбрать только конкретные ключи
      # items:
      # - key: username
      #   path: db-user.txt          # Переименовать файл
      # - key: password
      #   path: db-pass.txt
  containers:
  - name: my-container
    image: nginx
    volumeMounts:
    - name: secret-volume            # Имя тома из volumes
      mountPath: /etc/secrets        # Путь в контейнере
      readOnly: true                 # Рекомендуется readOnly для секретов
```

После монтирования в контейнере появятся файлы:

```
# Зайдем в контейнер и проверим
root@k8s:~# kubectl exec -it <pod> -n <namespace> -- /bin/bash
root@my-pod:/# ls -la /etc/secrets/
total 0
drwxrwxrwt 3 root root  100 Mar  8 12:30 .
drwxr-xr-x 1 root root   21 Mar  8 12:30 ..
drwxr-xr-x 2 root root   80 Mar  8 12:30 ..2025_03_08_12_30_01
lrwxrwxrwx 1 root root   31 Mar  8 12:30 ..data -> ..2025_03_08_12_30_01
lrwxrwxrwx 1 root root   15 Mar  8 12:30 password -> ..data/password
lrwxrwxrwx 1 root root   15 Mar  8 12:30 username -> ..data/username

root@my-pod:/# cat /etc/secrets/username
admin
root@my-pod:/# cat /etc/secrets/password
123456
```

**Способ 3. Особый случай — для доступа к Docker registry:**

```
apiVersion: v1
kind: Pod
metadata:
  name: private-image-pod
  namespace: myspace
spec:
  imagePullSecrets:                    # Специальное поле для registry
  - name: regcred                      # Имя секрета типа docker-registry
  containers:
  - name: my-app
    image: private-registry.com/myapp:latest
```

**Важные особенности:**

- Если секрет обновится, файлы в томе тоже обновятся, но не мгновенно (до ~60 секунд);
- Переменные окружения не обновляются автоматически. Если секрет обновится, переменные окружения не изменятся, для этого потребуется пересоздать под;
- Необходимо устанавливать права доступа`r--------`:

```
volumes:
- name: secret-volume
  secret:
    secretName: mysecret
    defaultMode: 0400           # Установить права доступа (r--------)
```

- Способ 1.1 (`secretKeyRef`в`env`) — корректный и стандартный.
- Способ 1.2 через`envFrom`(`secretRef`) — тоже корректный и рабочий, но даёт меньше контроля над именами переменных.
- Способ 2 (Secret как volume) — полностью соответствует официальной практике;`readOnly: true`для секретных томов является хорошей практикой.
- Способ 3 используется для доступа к Docker registry.

## 7. Типы Secrets

```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
```

- `Opaque`— самый распространенный тип. Данные хранятся в виде пар «ключ-значение». Подходит для хранения любых пользовательских данных: паролей, ключей API, токенов и т.д., которые не требуют специальной обработки.
- `bootstrap.kubernetes.io/token`— используется в процессе начальной загрузки (bootstrap) кластера. Хранит токены, которые применяются новыми узлами для подключения к кластеру и установления доверия с control plane. Как правило, администраторы редко работают с этим типом напрямую, так как он автоматически создается инструментами вроде`kubeadm`.
- `kubernetes.io/service-account-token`— хранение токена для ServiceAccount. Этот тип секрета автоматически создается для каждой служебной учетной записи (ServiceAccount) и содержит токен, который монтируется в под для аутентификации в API-сервере Kubernetes. В современных версиях Kubernetes рекомендуется использовать более безопасный механизм (TokenRequest API).
- `kubernetes.io/tls`— хранение TLS-сертификатов и ключей. Используется для настройки HTTPS на сайтах (Ingress), шифрования трафика внутри кластера или для любых других целей, где требуется пара «сертификат-ключ». Содержит два ключа:`tls.crt`(сертификат) и`tls.key`(закрытый ключ).
- `kubernetes.io/basic-auth`— хранение учетных данных для HTTP Basic Authentication. Это удобный и стандартизированный способ передать приложению логин и пароль для базовой аутентификации. Обычно содержит два ключа:`username`и`password`.
- `kubernetes.io/ssh-auth`— хранение учетных данных для SSH-аутентификации. Используется, когда приложению нужно подключиться к удаленному серверу по SSH. Содержит ключ`ssh-privatekey`.
- `kubernetes.io/dockerconfigjson`(пришел на смену старому`kubernetes.io/dockercfg`) — хранение учетных данных для доступа к частному Docker-реестру (например, к Docker Hub, Harbor, Nexus) для скачивания образов. Kubernetes использует этот секрет в поле`imagePullSecrets`Pod’а, чтобы аутентифицироваться и скачать образ из приватного репозитория. Содержит данные в формате JSON, включая адрес сервера, имя пользователя и пароль.

## 8. Полезные команды

Проверка, какие секреты использует под

```
# Через describe
root@k8s:~# kubectl describe pod <pod> -n <namespace>
...
Environment:
  DB_USERNAME:  <set to the key 'username' in secret 'mysecret'>  Optional: false
  DB_PASSWORD:  <set to the key 'password' in secret 'mysecret'>  Optional: false
Mounts:
  /etc/secrets from secret-volume (rw)

# Через get
root@k8s:~# kubectl get pod <pod> -n <namespace> -o yaml | grep -A 5 -B 5 "secret"
```

```
Просмотр списка секретов:
# Посмотреть все секреты в указанном namespace
kubectl get secrets -n <namespace>

Просмотр деталей секрета (describe):
# Показать общую информацию о секрете (значения скрыты)
kubectl describe secret <secret> -n <namespace>

Просмотр и декодирование значений секрета:
# Посмотреть сырые данные (в base64)
kubectl get secret <secret> -n <namespace> -o yaml

# Посмотреть конкретное поле и сразу декодировать
kubectl get secret <secret> -n <namespace> -o jsonpath='{.data.password}' | base64 --decode

# Или декодировать вручную
echo "MWYyZDFlMmU2N2Rm" | base64 --decode

# Просмотр переменных окружения в поде
kubectl exec -it <pod> -n <namespace> -- printenv
```

Стоит заметить, что такой способ хранения секретов является не самым безопасным. На практике обычно используют сторонние решения, например, Hashicorp Vault либо иные способы. Главный недостаток секретов k8s в том, что любой пользователь кластера с нужными правами может зайти и увидеть их значения.

## Оригинал

https://wikisys.ru/7-kubernetes-configmap-secret/
