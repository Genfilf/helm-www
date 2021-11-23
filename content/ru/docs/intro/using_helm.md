---
title: "Использование Helm"
description: "Объясняет основы работы с Helm."
weight: 3
---

В этом руководстве объясняются основы использования Helm для управления пакетами на вашем
кластере Kubernetes. Руководство предполагает, что вы уже [установили]({{< ref
"install.md" >}}) Helm CLI.

Если вас просто интересно запустить несколько быстрый комманд, то можно начать с [Краткого Руководства]({{< ref "quickstart.md" >}}). В этой главе
описаны особенности команд Helm, и объясняет как использовать Helm.

## Три Больших Концепта

*Chart* – это пакет Helm.
Он содержит описания ресурсов, необходимые для запуска приложения, инструментов, или службы внутри кластера Kubernetes. Подумайте об этом, как об аналоге formula в homebrew, пакетах apt, или пакетах rpm, только для Kubernetes. 

*Репозиторий* это место, где можно загружать charts-ы и делиться ими. Очень похоже на
Perl's [CPAN archive](https://www.cpan.org) или [Fedora Package
Database](https://src.fedoraproject.org/), но для пакетов Kubernetes. 

*Релиз* это экземпляр chart-а, работающего в кластере Kubernetes. 
Один chart часто может быть установлен много раз в одном и том же кластере.
Каждый раз, когда он устанавливается, создается новый _release_. Рассмотрим chart MySQL.
Если вам в кластере необходимы две базы данных, вы можете просто установить chart дважды. 
Каждый установленный chart будет иметь свой _release_, который в свою очередь будет 
иметь свой _release_name_.   

Зная эти понятия, объяснить работу Helm можно так:

Helm устанавливает _charts_ в Kubernetes, создавая новый _release_ для каждой установки. 
А для поиска новых chart-ов можно воспользоваться _repositories_ Helm chart-ов.

## 'helm search': Поиск Charts

В Helm есть мощная поисковая команда. 
Он может быть использован для поиска двух различных типов источников:

- `helm search hub` ищет в [the Artifact Hub](https://artifacthub.io), который агрегирует списки из различных репозиториев.
- `helm search repo` поиск в репозиториях, которые вы добавили в свой локальный helm client (используя `helm repo add`).
  Этот поиск выполняется по локальным данным, не используя подключение к публичной сети.

Вы можете найти общедоступные chart-ы, запустив `helm search hub`:

```console
$ helm search hub wordpress
URL                                                 CHART VERSION APP VERSION DESCRIPTION
https://hub.helm.sh/charts/bitnami/wordpress        7.6.7         5.2.4       Web publishing platform for building blogs and ...
https://hub.helm.sh/charts/presslabs/wordpress-...  v0.6.3        v0.6.3      Presslabs WordPress Operator Helm Chart
https://hub.helm.sh/charts/presslabs/wordpress-...  v0.7.1        v0.7.1      A Helm chart for deploying a WordPress site on ...
```

Выше приведен поиск всех chart-ов `wordpress` на Artifact Hub.

Если не указывать фильтры, то `helm search hub` вернёт список всех доступных chart-ов.

Используя `helm search repo`, вы можете найти названия chart-ов в
уже добавленных вами репозиториях:


```console
$ helm repo add brigade https://brigadecore.github.io/charts
"brigade" has been added to your repositories
$ helm search repo brigade
NAME                          CHART VERSION APP VERSION DESCRIPTION
brigade/brigade               1.3.2         v1.2.1      Brigade provides event-driven scripting of Kube...
brigade/brigade-github-app    0.4.1         v0.2.1      The Brigade GitHub App, an advanced gateway for...
brigade/brigade-github-oauth  0.2.0         v0.20.0     The legacy OAuth GitHub Gateway for Brigade
brigade/brigade-k8s-gateway   0.1.0                     A Helm chart for Kubernetes
brigade/brigade-project       1.0.0         v1.0.0      Create a Brigade project
brigade/kashti                0.4.0         v0.4.0      A Helm chart for Kubernetes
```

Helm search использует приближённый поиск подстроки (fuzzy string matching), поэтому вы можете вводить части
слов или фраз:

```console
$ helm search repo kash
NAME            CHART VERSION APP VERSION DESCRIPTION
brigade/kashti  0.4.0         v0.4.0      A Helm chart for Kubernetes
```

Поиск – это хороший способ найти доступные пакеты. Как только вы нашли пакет
, который хотите установить, вы можете использовать `helm install` для его установки.

## 'helm install': Установка пакета

Чтобы установить новый пакет, используйте команду `helm install`. В простейшем случае она
ожидает два аргумента: выбранное вами имя выпуска `_release_name_`, и имя chart-а, который вы
хотите установить.

```console
$ helm install happy-panda bitnami/wordpress
NAME: happy-panda
LAST DEPLOYED: Tue Jan 26 10:27:17 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
** Please be patient while the chart is being deployed **

Your WordPress site can be accessed through the following DNS name from within your cluster:

    happy-panda-wordpress.default.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w happy-panda-wordpress'

   export SERVICE_IP=$(kubectl get svc --namespace default happy-panda-wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: user
  echo Password: $(kubectl get secret --namespace default happy-panda-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
```

После установки `wordpress` chart-а обратите внимание, что при установке создается
новый объект _release_. Текущей релиз называется `happy-panda`. (Если вы хотите
чтобы Helm сгенерировал имя за вас, пропустите название релиза и используйте
аргумент `--generate-name`.)

Во время установки `helm` клиент напечатает полезную информацию о том, какие
ресурсы были созданы, каково состояние релиза, а также есть ли
дополнительные шаги настройки, которые вам необходимо выполнить.

Helm устанавливает ресурсы в следующем порядке:

- Namespace
- NetworkPolicy
- ResourceQuota
- LimitRange
- PodSecurityPolicy
- PodDisruptionBudget
- ServiceAccount
- Secret
- SecretList
- ConfigMap
- StorageClass
- PersistentVolume
- PersistentVolumeClaim
- CustomResourceDefinition
- ClusterRole
- ClusterRoleList
- ClusterRoleBinding
- ClusterRoleBindingList
- Role
- RoleList
- RoleBinding
- RoleBindingList
- Service
- DaemonSet
- Pod
- ReplicationController
- ReplicaSet
- Deployment
- HorizontalPodAutoscaler
- StatefulSet
- Job
- CronJob
- Ingress
- APIService

Helm не ждет, пока все ресурсы будут запущены, прежде чем он завершит свою работу. Для многих
chart-ов требуются образ Docker-а размером более 600M, и их установка в кластер может занять много
времени.

Чтобы отслеживать состояние релиза или ещё раз прочесть информацию о конфигурации, вы
можете использовать `helm status`:

```console
$ helm status happy-panda
NAME: happy-panda
LAST DEPLOYED: Tue Jan 26 10:27:17 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
** Please be patient while the chart is being deployed **

Your WordPress site can be accessed through the following DNS name from within your cluster:

    happy-panda-wordpress.default.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w happy-panda-wordpress'

   export SERVICE_IP=$(kubectl get svc --namespace default happy-panda-wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: user
  echo Password: $(kubectl get secret --namespace default happy-panda-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
```

Выше показано текущее состояние вашего релиза.

### Кастомизация Chart-а Перед установкой

Изначально Helm использует параметры конфигурации по умолчанию. 
Скорее всего вам захочется кастомизировать chart для использования
собственной конфигурации.

Чтобы увидеть, какие параметры возможно изменить в chart-е, используйте `helm show values`:

```console
$ helm show values bitnami/wordpress
## Global Docker image parameters
## Please, note that this will override the image parameters, including dependencies, configured to use the global value
## Current available global Docker image parameters: imageRegistry and imagePullSecrets
##
# global:
#   imageRegistry: myRegistryName
#   imagePullSecrets:
#     - myRegistryKeySecretName
#   storageClass: myStorageClass

## Bitnami WordPress image version
## ref: https://hub.docker.com/r/bitnami/wordpress/tags/
##
image:
  registry: docker.io
  repository: bitnami/wordpress
  tag: 5.6.0-debian-10-r35
  [..]
```

Вы можете переопределить любой из этих параметров в файле формата YAML, а затем передать этот файл во время установки.

```console
$ echo '{mariadb.auth.database: user0db, mariadb.auth.username: user0}' > values.yaml
$ helm install -f values.yaml bitnami/wordpress --generate-name
```

Команды выше создадут пользователя для MariaDB с именем `user0` и предоставит
этому пользователю доступ к созданной базе данных `user0db`, но использует остальные значения chart-а по умолчанию.

Существует два способа передачи конфигурационных данных во время установки:

- `--values` (или `-f`): Укажите YAML файл с конфигурацией. Это можно сделать несколько раз, и самый правый файл будет иметь приоритет
- `--set`: переопределения в командной строке.

Если используются оба параметра, то значения `--set` объединяются в `--values` с более высоким
приоритетом. Переопределения, указанные с помощью `--set`, сохраняются в ConfigMap.
Чтобы узнать какие значения были указаны командой `--set` для релиза, можно воспользоваться 
командой `helm get values <release-name>`. Значения `--set` можно очистить, выполнив
`helm upgrade` с аргументом `--reset-values`.



#### Формат и Ограничения`--set`

Параметр `--set` принимает ноль или более пар имя/значение. В простейшем случае он
используется следующим образом: `--set name=value`. Пример в YAML формате:

```yaml
name: value
```

Несколько значений разделяются символами `,`. Например `--set a=b,c=d` в YAML формате будет выглядеть так:

```yaml
a: b
c: d
```

Поддерживаются и более сложные выражения. Например, `--set outer.inner=value` превратится в:

```yaml
outer:
  inner: value
```

Списки могут быть предоставлены путем включения значений в `{` и `}`. Например, `--set
name={a, b, c}` превратится в:

```yaml
name:
  - a
  - b
  - c
```

Начиная с Helm 2.5.0, доступ к элементам списка возможен с помощью
синтаксиса индекса массива. Например, `--set servers[0].port=80` станет:

```yaml
servers:
  - port: 80
```

Этим способом можно задать несколько значений. Строка `--set
servers[0].port=80,servers[0].host=example` становится:

```yaml
servers:
  - port: 80
    host: example
```

Иногда вам нужно использовать специальные символы в `--set`. Вы можете использовать
обратную косую черту для экранирования символов; `--set name=value1\,value2` станет:

```yaml
name: "value1,value2"
```

Точно так же вы можете экранировать точки, что может пригодиться, когда
chart-ы используют функцию `toYaml` для анализа аннотаций, меток и
селекторов узлов. Пример синтаксиса `--set nodeSelector."kubernetes\.io/role"=master`:

```yaml
nodeSelector:
  kubernetes.io/role: master
```

Структуры с глубокой вложенностью может быть сложно задать с помощью `--set`. 
Разработчикам chart-ов рекомендуется учитывать использование `--set` при 
разработке файла `values.yaml` (узнать больше о [Файлах значений](https://helm.sh/docs/chart_template_guide/values_files/)).

### Дополнительные Методы Установки

  `helm install` может установить chart из нескольких источников:

- Репозиторий chart-ов (как мы видели выше)
- Локальное хранилище chart-ов (`helm install foo foo-0.1.1.tgz`)
- Распакованная директория с chart-ами (`helm install foo path/to/foo`)
- Полный URL до chart-а (`helm install foo https://example.com/charts/foo-1.2.3.tgz`)

## 'helm upgrade' и 'helm rollback': Обновление Релиза и Восстановление После Сбоя

Когда выпускается новая версия chart-а или когда вы хотите изменить
конфигурацию своего релиза, вы можете использовать команду `helm upgrade`.

`upgrade` вносит предоставленные вами изменения в текущий релиз. Из-за того, что
chart-ы для Kubernetes могут быть большими и сложными, Helm при обновлении старается
затронуть как можно меньше, обновляя только то, что изменилось с прошлого релиза.  

```console
$ helm upgrade -f panda.yaml happy-panda bitnami/wordpress
```

В приведенном выше случае релиз `happy-panda` обновляется с тем же chart,
но с новым файлом YAML:

```yaml
mariadb.auth.username: user1
```

Вы можете воспользоваться `helm get values`, чтобы увидеть применилась ли конфигурация.

```console
$ helm get values happy-panda
mariadb:
  auth:
    username: user1
```

Команда `helm get` - это полезный инструмент для просмотра релизов в кластере.
И как мы видим выше, она показывает что наши новые значения из `panda.yaml` были
развернуты в кластере.

Теперь, если что-то пойдёт не так как планировалось во время развёртывания,  то можно легко
откатиться к предыдущему релизу, используя `helm rollback [RELEASE] [REVISION]`.

```console
$ helm rollback happy-panda 1
```

Пример выше откатывает happy-panda к самой первой редакции релиза. Редакция
релиза является инкрементным значением. Каждый раз, когда происходит установка, обновление или
откат, номер редакции увеличивается на 1. Первый
номер редакции всегда равен 1. Мы можем использовать `helm history [RELEASE]`, чтобы увидеть
номера редакций для определенного релиза.

## Полезные Опции для Установки/Обновления/Отката

Существует несколько других полезных параметров, которые вы можете указать для настройки
поведения Helm во время установки/обновления/отката. Обратите внимание, что это
не полный список флагов cli. Чтобы увидеть описание всех флагов, просто запустите `helm
<command> --help`.

- `--timeout`: [Go duration](https://golang.org/pkg/time/#ParseDuration) время ожидания 
  выполнения Kubernetes команд. Значение по-умолчанию `5m0s`.
- `--wait`: Ожидание пока все Pods перейдут в режим готовности, примонтируются PVC,
  в Deployments будет развёрнут необходимый минимум Pods (`Desired` минус `maxUnavailable`), 
  и Services получит IP адрес (так же Ingree, если указан `LoadBalancer`).
  После этого релиз считается успешно развёрнутым. Время ожидания указывается значение `--timeout`.
  По истечению таймаута релиз будет помечен как `FAILED`, что означает что запуск прошёл неуспешно.
  Обратите внимание: в рамках rolling update, если в Deployment значение `replicas` равно 1, а значение `maxUnavailable` 
  не равно 0, `--wait` отработает сразу, как только будет минимальное количество Pod в готовом состоянии. 
- `--no-hooks`: Данной командой можно пропустить запуск hooks
- `--recreate-pods` (доступно только для `upgrade` и `rollback`): данным флагом можно вызвать пересоздание
  всех Pods (за исключением тех Pods, которые относятся к Deployments). (УСТАРЕЛО в Helm 3)

## 'helm uninstall': Удаление Релиза

Когда потребуется удалить релиз из кластера, используйте команду `helm uninstall`:

```console
$ helm uninstall happy-panda
```

Это приведет к удалению релиза из кластера. Вы можете просмотреть все ваши текущие
развернутые релизы с помощью команды `helm list`:

```console
$ helm list
NAME            VERSION UPDATED                         STATUS          CHART
inky-cat        1       Wed Sep 28 12:59:46 2016        DEPLOYED        alpine-0.1.0
```

Из вывода выше видно, что релиз `happy-panda` был удален.

В предыдущих версиях Helm, когда релиз удалялся, запись о его
удалении оставалась. В Helm 3 удаление релиза также удаляет и запись о нём.
Если вы хотите сохранить запись об удалении выпуска, используйте `helm uninstall
--keep-history`. Использование `helm list --uninstalled` покажет только те выпуски, которые
были удалены с флагом `--keep-history`.

Флаг `helm list --all` покажет вам все релизы,
сохраненные Helm, включая записи для неудачных или удаленных элементов (если был указан параметр `--keep-history`
):

```console
$  helm list --all
NAME            VERSION UPDATED                         STATUS          CHART
happy-panda     2       Wed Sep 28 12:47:54 2016        UNINSTALLED     wordpress-10.4.5.6.0
inky-cat        1       Wed Sep 28 12:59:46 2016        DEPLOYED        alpine-0.1.0
kindred-angelf  2       Tue Sep 27 16:16:10 2016        UNINSTALLED     alpine-0.1.0
```

Обратите внимание, что поскольку релизы теперь удаляются по умолчанию, откат удаленного ресурса больше невозможен.

## 'helm repo': Работа с Репозиториями

Helm 3 больше не поставляется со стандартным хранилищем chart-ов. Команды `helm repo`
предоставляет возможности добавления, отображения и удаления репозиториев.

Узнать о настроенных репозитории можно с помощью `helm repo list`:

```console
$ helm repo list
NAME            URL
stable          https://charts.helm.sh/stable
mumoshu         https://mumoshu.github.io/charts
```

Новые репозитории могут быть добавлены с помощью `helm repo add`:

```console
$ helm repo add dev https://example.com/dev-charts
```

Поскольку репозитории диаграмм часто меняются, в любой момент вы можете убедиться,
что ваш клиент Helm обновлен, запустив `helm repo update`.

Репозитории могут быть удалены с помощью `helm repo remove`.

## Создание Собственных Chart-ов

[Руководство по разработке Chart-ов](https://helm.sh/docs/topics/charts/) объясняет, как
создавать свои собственные chart-ы. Но вы можете быстро начать работу с помощью команды `helm
create`:

```console
$ helm create deis-workflow
Creating deis-workflow
```

Теперь у вас есть chart в `./deis-workflow`. Вы можете редактировать его и создавать свои собственные
шаблоны.


У Helm есть линтер для проверки корректности chart-а, воспользоваться им можно с помощью `helm
lint`.

Когда придет время упаковать chart в пакет для распространения вам поможет команда `helm
package`

```console
$ helm package deis-workflow
deis-workflow-0.1.0.tgz
```

Теперь вы можете с легкостью поставить этот chart в ваш Helm с помощью `helm install`

```console
$ helm install deis-workflow ./deis-workflow-0.1.0.tgz
...
```

Упакованные chart-ы могут быть загружены в репозиторий chart-ов. Для подробных сведений
просмотрите документацию по [Helm chart репозитории](https://helm.sh/docs/topics/charts/) .

## Заключение

В этой главе рассмотрены основные схемы использования клиента `helm`,
включая поиск, установку, обновление и удаление. Он также
охватывает полезные служебные команды, такие как `helm status`, `helm get`, и `helm repo`.

Для получения дополнительной информации об этих командах обратитесь к встроенной справке Helm: `helm help`.

В [следующей главе](../howto/charts_tips_and_tricks/) мы рассмотрим процесс разработки chart-ов 