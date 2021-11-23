---
title: "Краткое Руководство"
description: "How to install and get started with Helm including instructions for distros, FAQs, and plugins."
weight: 1
aliases: ["/docs/quickstart/"]
---

Это руководство описывает, как вы можете быстро начать использовать Helm.

## Необходимые компоненты

Следующие компоненты необходимы для успешного и безопасного использования Helm:

1. Кластер Kubernetes
2. Принятие решения о том, какие конфигурации безопасности должны применяться к вашей установке, если таковые имеются.
3. Установка и настройка Helm

### Установите Kubernetes или получите доступ к кластеру

- У вас должен быть установлен Kubernetes. Для последней версии Helm мы
  рекомендуем последнюю стабильную версию Kubernetes, которая в большинстве случаев является
  второй последней минорной версией.
- Вы также должны иметь локально настроенную копию `kubectl`.

Смотрите [Политику поддержки версий Helm](https://helm.sh/docs/topics/version_skew/) для того, чтобы понимать максимальную версию поддержки между Helm и Kubernetes.

## Установка Helm

Загрузите binary релиз клиента Helm. Вы можете использовать такие инструменты, как `homebrew`,
или заглянуть на [официальную страницу релизов](https://github.com/helm/helm/releases).

Для получения более подробной информации или других вариантов смотрите [руководство по установке]({{< ref
"install.md" >}}).

## Инициализация Helm Chart Repository

Как только Helm будет установлен, вы можете добавить chart репозиторий. Вы можете проверить доступные репозитории Helm chart-ов на [Artifact
Hub](https://artifacthub.io/packages/search?kind=0).

```console
$ helm repo add stable https://charts.helm.sh/stable
```

После добавления репозитория, вы сможете посмотреть charts, которые вам теперь доступны:

```console
$ helm search repo bitnami
NAME                             	CHART VERSION	APP VERSION  	DESCRIPTION
bitnami/bitnami-common           	0.0.9        	0.0.9        	DEPRECATED Chart with custom templates used in ...
bitnami/airflow                  	8.0.2        	2.0.0        	Apache Airflow is a platform to programmaticall...
bitnami/apache                   	8.2.3        	2.4.46       	Chart for Apache HTTP Server
bitnami/aspnet-core              	1.2.3        	3.1.9        	ASP.NET Core is an open-source framework create...
# ... and many more
```

## Пример Установки Chart

Чтобы установить chart необходимо использовать команду `helm install`.
У Helm есть несколько способов поиска и установки chart, но самый простой – это использовать `bitnami` chart.

```console
$ helm repo update              # Для использования последних версий необходимо обновить список chart-ов
$ helm install bitnami/mysql --generate-name
NAME: mysql-1612624192
LAST DEPLOYED: Sat Feb  6 16:09:56 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES: ...
```

В приведенном выше примере был выпущен `bitnami/mysql` chart, а имя нашего нового релиза: `mysql-1612624192`.

Вы получите простое представление о возможностях этого MySQL chart-а, запустив `helm show
chart bitnami/mysql`.
Или вы можете запустить `helm show all stable/mysql` чтобы получить всю информацию о chart-е.

Всякий раз, когда вы устанавливаете chart, создается новый релиз. 
Таким образом, один chart можно установить несколько раз в один и тот же кластер. 
И каждый из них может управляться и обновляться независимо.

`helm install` очень мощная команда со многими возможностями.
Узнать больше можно в [Руководстве по эксплуатации Helm]({{< ref "using_helm.md"
>}})

## Подробнее О Релизах

Можно легко увидеть, что было развёрнуто с помощью Helm:

```console
$ helm list
NAME            	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART      	APP VERSION
mysql-1612624192	default  	1       	2021-02-06 16:09:56.283059 +0100 CET	deployed	mysql-8.3.0	8.0.23
```

Команда `helm list` покажет вам список всех развернутых релизов.

## Удаление Релиза

Чтобы удалить релиз, используйте команду `helm uninstall`:

```console
$ helm uninstall mysql-1612624192
release "mysql-1612624192" uninstalled
```

В данном случае, это удалит `smiling-penguin` из Kubernetes, 
а так же удалит все ресурсы, связанные с этим релизом и саму историю релизов.

Если использовать флаг `--keep-history`, то история релизов будет сохранена.
В этом случае, у вас остается возможность запросить информацию о удаленном ранее релизе:

```console
$ helm status mysql-1612624192
Status: UNINSTALLED
...
```

Так как Helm отслеживает ваши релизы даже после того, как вы их деинсталлировали, 
вы можете проверить историю кластера, и даже восстановить релиз используя `helm rollback`.

## Справка

Чтобы узнать больше о доступных Helm командах, используйте `helm help` или введите
команду за которой следует флаг `-h`:

```console
$ helm get -h
```
