---
title: "16. Kubernetes. Работа с шаблонизатором Helm"
date: 2026-03-09T07:00:00Z
source: "wikisysru"
original_url: "https://wikisys.ru/6-kubernetes-%d1%80%d0%b0%d0%b1%d0%be%d1%82%d0%b0-%d1%81-%d1%88%d0%b0%d0%b1%d0%bb%d0%be%d0%bd%d0%b8%d0%b7%d0%b0%d1%82%d0%be%d1%80%d0%be%d0%bc-helm/"
summary: "Шаблонизатор упаковывает приложения для развёртывания и управления в управляемой инфраструктуре.   Helm реализует такой шаблонизатор и менеджер приложений специально для Kubernetes.   Он помогает упаковывать, настраивать и разворачивать приложения в Kubernetes-среде.   Helm упрощает администрирование и повторное использование конфигураций.   Благодаря Helm приложения можно устанавливать, обновлять и управлять ими как едиными сущностями.   Таким образом, Helm позиционируется как лучший инструмент для Kubernetes-развёртывания."
---

# 16. Kubernetes. Работа с шаблонизатором Helm

## Краткое содержание

Шаблонизатор упаковывает приложения для развёртывания и управления в управляемой инфраструктуре.  
Helm реализует такой шаблонизатор и менеджер приложений специально для Kubernetes.  
Он помогает упаковывать, настраивать и разворачивать приложения в Kubernetes-среде.  
Helm упрощает администрирование и повторное использование конфигураций.  
Благодаря Helm приложения можно устанавливать, обновлять и управлять ими как едиными сущностями.  
Таким образом, Helm позиционируется как лучший инструмент для Kubernetes-развёртывания.

## Полная статья

Шаблонизатор — это средство упаковки приложений для последующего развёртывания и администрирования в подконтрольной инфраструктуре.

Helm — это шаблонизатор и менеджер приложений для развёртывания в среде kubernetes.

Шаблонизатор Helm — лучший способ манипуляции над рабочими нагрузками, позволяющий:

- осуществлять поиск
- устанавливать
- обновлять (upgrade)
- удалять приложения

Если бы Kubernetes был операционной системой, то Helm был бы менеджером пакетов. Ubuntu использует apt, CentOS использует yum, а Kubernetes использует Helm.

Helm развертывает пакетные приложения в Kubernetes и структурирует их в чарты (Helm Charts). Чарты содержат все предустановленные ресурсы приложения вместе со всеми версиями, которые помещены в один легко управляемый пакет.

Helm упрощает установку, обновление, вызов зависимостей и настройку развертываний в Kubernetes с помощью простых CLI-команд. Пакеты программного обеспечения находятся в репозиториях или создаются.

Для начала работы с шаблонизатором Helm достаточно установить`helm-client`и добавить chart-репозиторий с необходимым приложением.

`helm-chart`— основная сущность Helm для создания, версионирования, распространения и публикации приложений в kubernetes. Иначе говоря, чарты Helm****— это пакеты Helm, состоящие из файлов и шаблонов YAML, которые преобразуются в файлы манифеста Kubernetes. Чарты могут повторно использоваться кем угодно и в любой среде, что уменьшает сложность и количество дубликатов.

Необходимый`helm-chart`можно получить двумя способами:

- найти в официальном репозитории артефактов
- создать свой собственный

Три основные концепции чартов Helm:

- Чарт — предварительно настроенный шаблон ресурсов Kubernetes.
- Релиз — чарт, развернутый с помощью Helm в кластере Kubernetes.
- Репозиторий — общедоступные чарты.

Рабочий процесс заключается в поиске чартов через репозитории и создании релизов путем установки чартов в кластеры Kubernetes.

Проверяем, установлен ли helm:

```
helm version
```

Если не установлен, то загружаем скрипт установки, согласно[**документации по установке**](https://helm.sh/docs/intro/install/#from-script-bash)

Например, можно установить 4-ю версию Helm одним скриптом:

```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4 | bash
```

Helm тесно связан с версией Kubernetes, поэтому важно выбрать версию, совместимую с нашей версией кластера. Ознакомиться с таблицей совместимости версий Helm и Kubernetes можно на странице[**официальной документации**](https://helm.sh/docs/topics/version_skew/?roistat_visit=9017048#supported-version-skew)

## Работа с helm-charts

Рассмотрим особенности работы с helm чартами на примере Grafana.

Создадим неймспейс:

```
kubectl create ns monitoring
```

Теперь добавим репозиторий`Grafana`:

```
helm repo add grafana https://grafana.github.io/helm-charts
```

Проверим список репозиториев:

```
helm repo list
```

![](/news/wikisysru/article-7b3c7bce534ca365/image-01.png)

После подключения репозитория обязательно необходимо обновить кеш, чтобы получить самую свежую информацию о чартах из добавленного репозитория:

```
helm repo update
```

Теперь посмотрим, какие именно чарты для Grafana доступны в этом репозитории:

```
helm search repo grafana/grafana
```

`helm search repo`— команда для поиска по уже добавленным репозиториям. Запись`grafana/grafana`означает: в репозитории с именем`grafana`найти чарт с именем`grafana`. То есть слеш (`/`) работает как разделитель между именем репозитория и именем чарта:

![](/news/wikisysru/article-7b3c7bce534ca365/image-02.png)

`grafana`до слеша — это просто локальный псевдоним, который мы придумали для этого репозитория. Мы могли бы назвать его`my-grafana`— команда выглядела бы соответственно:`my-grafana/grafana`.

Установим чарт в наш кластер:

```
helm install my-grafana grafana/grafana -n monitoring
```

`my-grafana`— это имя, которое мы даём своему «релизу» (экземпляру установки).

После выполнения команды Helm выведет много служебной информации, а в самом конце — важные заметки (Notes), которые подскажут, как получить пароль и как подключиться к Grafana.

![](/news/wikisysru/article-7b3c7bce534ca365/image-03.png)

**Полезные опции для команды helm install**

```
helm install my-grafana grafana/grafana -n monitoring --wait
```

Опция`--wait`заставит Helm не считать установку завершенной, пока все ресурсы приложения действительно не станут рабочими. Команда как бы «зависнет» до тех пор, пока не произойдет одно из двух:

- Все ресурсы перейдут в состояние «Ready»:
— поды в Deployments, StatefulSets и DaemonSets должны быть запущены и пройти проверки готовности (readiness probes);
— PersistentVolumeClaims (PVC) должны перейти в статус`Bound`(подключиться к хранилищу);
— Сервисы типа`LoadBalancer`должны получить внешний IP-адрес.
- Истечет таймаут (по умолчанию — 5 минут). Если за 5 минут ресурсы не станут готовы, установка прервется с ошибкой.

Это критически важно для автоматизации (CI/CD) и написания скриптов. Без`--wait`Helm установит всё «в слепую»: отправит манифесты в кластер и сразу завершится с успехом, даже если поды упадут с ошибкой или PVC не смогут подключиться. Флаг`--wait`гарантирует, что если команда выполнилась успешно — приложение действительно работает.

### Другие полезные опции

| Флаг | Описание | Как применяется |
| --- | --- | --- |
| --timeout | Задает время ожидания для --wait. По умолчанию — 5m (5 минут) . | Всегда используется вместе с --wait. Если приложение тяжелое (например, загружает большой образ), время ожидания увеличивают: --timeout 10m . |
| --atomic | Комбинация --wait + автоматический откат. Если установка (или обновление) провалится, Helm автоматически всё удалит . | Идеально для CI/CD. Гарантирует, что после ошибки не останется «мусора». Пример: helm install myapp ./chart --atomic --timeout 5m. |
| --dry-run | Пробный запуск. Helm покажет, какие манифесты будут созданы, но ничего не отправит в кластер . | «Режим примерки». Проверить, правильно ли сработали шаблоны и переменные, перед реальной установкой. |
| --debug | Включает подробный вывод. Вместе с --dry-run покажет не только итоговые манифесты, но и промежуточные результаты рендеринга шаблонов . | Отладка сложных шаблонов. Используют с --dry-run: helm install ... --dry-run --debug. |
| -f / --values | Указывает файл(ы) с пользовательскими значениями переменных . | Основа настройки. Для разных окружений (dev/stage/prod) делают свои файлы values-dev.yaml, values-prod.yaml. Пример: -f values-prod.yaml. |
| --set | Устанавливает значения переменных прямо в командной строке . | Для быстрых «горячих» правок при тестировании. Пример: --set replicaCount=5. |
| --create-namespace | Автоматически создает namespace, если его еще нет . | Удобно в автоматизации, чтобы не выполнять отдельно kubectl create ns. Пример: -n myapp --create-namespace. |
| --version | Устанавливает конкретную версию чарта . | Фиксация версий. Чтобы гарантированно поставить проверенную версию, а не самую свежую (которая может сломаться). |
| --generate-name | Автоматически генерирует имя для релиза . | Для одноразовых тестов, когда не хочется придумывать имя вручную. |

Для повседневной разработки и тестирования чаще всего используется связка`--wait --timeout`. Для серьезной автоматизации в CI/CD —`--atomic`. А для отладки и проверки перед деплоем —`--dry-run --debug`.

```
Просмотр релизов:
helm list -n monitoring
```

Статус установленного релиза должен быть`deployed`:

![](/news/wikisysru/article-7b3c7bce534ca365/image-04.png)

```
Проверка созданных ресурсов:
kubectl get all -n monitoring
```

![](/news/wikisysru/article-7b3c7bce534ca365/image-05.png)

Эта команда вернёт список подов, сервисов и других ресурсов. Pod Grafana должен быть в статусе`Running`. В моём случае pod не поднялся из-за отсутствия доступа к докерхабу.

```
Удаление релиза:
helm uninstall my-grafana -n monitoring
```

## Переопределение параметров инсталляции

Helm поддерживает установку с переопределением параметров инсталляции, с помощью опции
`--set`:

```
Установка с переопределением параметров:
helm install my-grafana grafana/grafana -n monitoring --set rbac.namespaced=true
```

Перед инсталляцией можно ознакомиться со списком доступных параметров установки:

```
Посмотреть все доступные параметры чарта (с комментариями):
helm show values grafana/grafana
```

![](/news/wikisysru/article-7b3c7bce534ca365/image-06.png)

```
Обновление релиза с переопределением параметров:
helm upgrade my-grafana grafana/grafana -n monitoring \
  --set rbac.namespaced=true \
  --set persistence.enabled=true \
  --set service.type=NodePort \
  -n monitoring
```

```
Сохранить все параметры в файл для изучения:
helm show values grafana/grafana > default-values.yaml

Посмотреть, что реально используется сейчас:
helm get values my-grafana -n monitoring

Посмотреть ВСЕ значения (включая дефолтные):
helm get values my-grafana -n monitoring --all
```

## Создание helm-chart

Helm поддерживает создание собственных helm чартов с помощью команды`helm create <chart>`. При создании такого чарта появится готовый темплейт, предназначенный для кастомизации. В результате выполнения команды`helm create <chart>`будут созданы файлы и директории , т.е. готовый для кастомизации темплейт будет содержать следующие файлы и директории:

- директория`charts/`
- директория`templates/`
- `chart.yaml`
- `values.yaml`

В директории`charts/`могут содержаться зависимые helm чарты, если таковые нужны для нашего кастомного чарта. Helm также управляет зависимостями через команду`helm dependency update`, которая скачивает чарты именно в эту папку (обычно в виде`.tgz`архивов).

В директории`templates/`находятся все основные файлы шаблонов для развёртывания объектов в kubernetes. Все файлы шаблонов, находящиеся в директории`templates/`, подвергнутся механизму рендеринга и отправятся на развёртывание в kubernetes:

- deployment.yaml
- service.yaml
- serviceaccount.yaml
- и другие

Кроме того, в директории`templates/`можно создать и другие объекты (файлы шаблонов). Например, можно создать secret.yaml. Созданные объекты тоже пройдут процедуру рендеринга и будут развёрнуты в kubernetes.

Файл`chart.yaml`содержит описание нашего чарта — метаданные по наименованию, описанию, типу приложения и его версии.

Файл`values.yaml`содержит информацию о переменных значениях развёртывания по умолчанию. При необходимости, эти переменные могут быть переопределены в момент установки helm чарта или при его обновлении.

## Создание helm-chart: MariaDB и adminer

Создадим helm чарт mariadb:

```
helm create mariadb
```

Посмотрим содержимое созданного шаблона:

![](/news/wikisysru/article-7b3c7bce534ca365/image-07.png)

Теперь будем модифицировать те файлы, которые были созданы в ходе создания шаблона нашего чарта. Начнём с файла`values.yaml`.

Созданный`values.yaml`содержит набор стандартных параметров для развертывания тестового приложения (nginx). В нём определены:

- Кол-во реплик —`replicaCount: 1`
- Образ —`repository: nginx`,`tag: ""`,`pullPolicy: IfNotPresent`
- Параметры образа —`imagePullSecrets: []`,`nameOverride: ""`,`fullnameOverride: ""`
- Сервис (Service) —****тип`ClusterIP`, порт`80`
- Ингресс (Ingress) — включен/выключен (`enabled: false`), хосты, пути
- Ресурсы (Resources) — пустые лимиты и запросы (`limits: {}`,`requests: {}`)
- Autoscaling (выключен) — (`enabled: false`), мин/макс реплик, целевая нагрузка
- Сервисный аккаунт (ServiceAccount) — настройки создания и аннотаций
- Политики Pod — Pod Security Context, Pod Disruption Budget
- Переменные окружения: —`env: []`

![](/news/wikisysru/article-7b3c7bce534ca365/image-08.png)

Это общий шаблон для nginx. Для нашей Mariadb придётся его переписать.

Добавим определение namespace — mydb в корневой уровень манифеста, а внутри секции`image`скорректируем название репозитория (mariadb) и укажем тег «latest».

Таким образом мы сообщаем шаблонизатору, что нам нужен образ «mariadb» с тегом «latest».

Далее, в корне манифеста в секциях`nameOverride`и`fullnameOverride`укажем «mymariadb». Это будет использовано, например, для наименования подов.

В корне`values.yaml`зададим значения для`nameOverride`и`fullnameOverride`: «mymariadb». Эти параметры будут использованы для формирования имен создаваемых ресурсов (например, подов и сервисов). Параметры`nameOverride`и`fullnameOverride`переопределяют стандартную логику формирования имен Helm (которая обычно выглядит как`имя_релиза-имя_чарта`). Это удобно, если нужно получить предсказуемые и короткие имена подов, например`mymariadb-xxx`вместо`myrelease-mariadb-xxx`.

![](/news/wikisysru/article-7b3c7bce534ca365/image-09.png)

Перейдём к секции`serviceAccount`. ServiceAccount — это учетная запись в Kubernetes для пода, а не для человека. Она определяет, какие права (RBAC) есть у пода при обращении к API Kubernetes. Это позволяет дать поду разрешение на выполнение действий, например, чтобы под мог прочитать статус другого пода или обновить custom resource.

ServiceAccount привязывается к поду → под получает токен → под использует токен для доступа к API.

Назовём ServiceAccount — name: «mymariadb-sa»:

![](/news/wikisysru/article-7b3c7bce534ca365/image-10.png)

Теперь отредактируем секцию`service`. Тип оставим ClusterIP, а порт укажем 3306 (дефолтный для nariadb):

![](/news/wikisysru/article-7b3c7bce534ca365/image-11.png)

Сохраняем`values.yaml`и открываем теперь для редактирования файл`deployment.yaml`, который находится в директории`templates/`:

![](/news/wikisysru/article-7b3c7bce534ca365/image-12.png)

Добавим неймспейс в секцию`metadata`— после секции metadata.name ниже вставим:

```
namespace: {{ .Values.namespace }}
```

![](/news/wikisysru/article-7b3c7bce534ca365/image-13.png)

Таким образом, из файла переменных будет взято значение секции namespace, а именно, mydb.

Если мы захотим изменить переменные, достаточно будет отредактировать один файл переменных`values.yaml`вместо того, чтобы переписывать все имеющиеся деплойменты, сервисы и т.п.

Перейдём к спецификации, а именно, к секции:`spec.template.spec.containers`. Уберём все пробы, а также — name: http (чуть выше проб):

![](/news/wikisysru/article-7b3c7bce534ca365/image-14.png)

Оставим лишь containerPort, не забыв добавить дефис. Получится следующая структура:

![](/news/wikisysru/article-7b3c7bce534ca365/image-15.png)

В containerPort уже указано значение из`values.yaml`, значит, порт будет 3306.

Теперь создадим директорию secret/ в качестве дополнительного файла в директории`templates/`. В secret/ мы запишем значение SUPERPASS, а в деплойменте мы на него сошлёмся.

Для этого мы добавим секцию`env`в деплоймент. Её нужно вставить внутрь секции`containers`, но вне блоков`with`. Лучше всего вставить её после`imagePullPolicy`и до всех`with`-блоков, либо после всех`with`-блоков, но до закрытия containers:

```
containers:
  - name: {{ .Chart.Name }}
    {{- with .Values.securityContext }}
    securityContext:
      {{- toYaml . | nindent 12 }}
    {{- end }}
    image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
    imagePullPolicy: {{ .Values.image.pullPolicy }}
    ports:
      - containerPort: {{ .Values.service.port }}
        protocol: TCP
    env:                    <-- Вставляем здесь (после ports, до with-блоков)
      - name: MARIADB_ROOT_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mymariadb-secret
            key: mariadb-root-password
    {{- with .Values.resources }}
    resources:
      {{- toYaml . | nindent 12 }}
    {{- end }}
```

![](/news/wikisysru/article-7b3c7bce534ca365/image-16.png)

Сохраняем деплоймент.

В случае ошибок, шаблонизатор Helm укажет на них при валидации, так что ошибиться здесь не страшно.

Теперь отредактируем файл`service.yaml`(mariadb/templates/service.yaml).

Добавим неймспейс:

![](/news/wikisysru/article-7b3c7bce534ca365/image-17.png)

Порт уже указан (будет взят из файла переменных). Укажем тоже самое и для`targetPort`, вместо http. А также удалим name: http, оставив протокол TCP:

![](/news/wikisysru/article-7b3c7bce534ca365/image-18.png)

Сохраняем`service.yaml`.

Редактируем`serviceaccount.yaml`(mariadb/templates/serviceaccount.yaml).

Добавим в этот файл только неймспейс для того, чтобы ServiceAccount был создан в том же неймспейсе, где и mariadb:

![](/news/wikisysru/article-7b3c7bce534ca365/image-19.png)

Сохраняем`serviceaccount.yaml`

Теперь создадим файл`secret.yaml`в директории`templates/`со следующим содержимым:

```
apiVersion: v1
kind: Secret
metadata:
  name: mymariadb-secret
  namespace: {{ .Values.namespace }}
type: Opaque
data:
  mariadb-root-password: MTIzNDU=
```

В секции`data.mariadb-root-password`необходимо указывать хеш, закодированный в base64. Узнать такой хеш для пароля**«**12345″ можно, введя в терминале такую команду:

```
echo -n "12345" | base64
```

![](/news/wikisysru/article-7b3c7bce534ca365/image-20.png)

Также важно убедиться, что указано имя из деплоймента (mymariadb-secret):

![](/news/wikisysru/article-7b3c7bce534ca365/image-21.png)

Сохраняем`secret.yaml`.

Теперь можно приступить к установке кастомного Helm чарта. Но перед этим, создадим второй Helm чарт — adminer:

```
helm create adminer
```

Редактируем`values.yaml`:

![](/news/wikisysru/article-7b3c7bce534ca365/image-22.png)

Добавим в данный файл секцию`namespace`(mydb), изменим образ (`image`) c «nginx» на «adminer» c тегом «latest»,`nameOverride`и`fullnameOverride`укажем, как «myadminer»:

![](/news/wikisysru/article-7b3c7bce534ca365/image-23.png)

Теперь отредактируем секцию`serviceAccount`: укажем имя — myadminer-sa

![](/news/wikisysru/article-7b3c7bce534ca365/image-24.png)

Затем изменим службу (service).

В секции`service.type`изменим ClusterIP на NodePort, а`service.port`— с 80 на 8080:

![](/news/wikisysru/article-7b3c7bce534ca365/image-25.png)

Сохраняем`values.yaml`.

Теперь модифицируем деплоймент.

Добавляем неймспейс:

![](/news/wikisysru/article-7b3c7bce534ca365/image-26.png)

Удаляем пробы и name:

![](/news/wikisysru/article-7b3c7bce534ca365/image-27.png)

Оставляем containerPort:

![](/news/wikisysru/article-7b3c7bce534ca365/image-28.png)

Сохраняем деплоймент.

Приступаем к модификации сервиса:

![](/news/wikisysru/article-7b3c7bce534ca365/image-29.png)

- Добавляем неймспейс —`namespace: {{ .Values.namespace }}`
- Редактируем значение`targetPort`: вместо http указываем`{{ .Values.service.port }}`
- Удаляем`name: http`.

![](/news/wikisysru/article-7b3c7bce534ca365/image-30.png)

Модифицируем ServiceAccount — добавим неймспейс:

![](/news/wikisysru/article-7b3c7bce534ca365/image-31.png)

Сохраняем ServiceAccount.

Приступаем к установке релизов из собственных чартов.

Создадим неймспейс:

```
kubectl create ns mydb
```

Устанавливаем чарт, который назовём mymariadb из директории mariadb/:

```
helm install mymariadb mariadb/
```

Теперь установим adminer:

```
helm install myadminer adminer/
```

![](/news/wikisysru/article-7b3c7bce534ca365/image-32.png)

Проверим, все ли ресурсы в неймспейсе поднялись:

```
kubectl get all -n mydb
```

Проверим сервисаккаунты:

```
kubectl get sa -n mydb
```

Проверим секреты:

```
kubectl get secret -n mydb
```

![](/news/wikisysru/article-7b3c7bce534ca365/image-33.png)

## Оригинал

https://wikisys.ru/6-kubernetes-%d1%80%d0%b0%d0%b1%d0%be%d1%82%d0%b0-%d1%81-%d1%88%d0%b0%d0%b1%d0%bb%d0%be%d0%bd%d0%b8%d0%b7%d0%b0%d1%82%d0%be%d1%80%d0%be%d0%bc-helm/
