---
title: "Установка Helm"
description: "Узнайте, как установить и начать работать с Helm."
weight: 2
---

В этом руководстве рассказывается, как установить Helm CLI. Helm может быть установлен как из
исходного кода, так и с помощью заранее собранных binary релизов.

## Из Проекта Helm

Проект Helm предоставляет два способа получения и установки Helm. Это
официальные методы получения релизов Helm. В дополнение к этому сообщество Helm
предоставляет методы установки Helm через различные менеджеры пакетов.
Установка с помощью этих методов может быть найдена ниже официальных методов.

### Из Binary Релизов

Каждый [релиз](https://github.com/helm/helm/releases) Helm предоставляет различные binary
сборки под разные операционные системы (OSes). Эти binary версии могут быть загружены и установлены вручную.

1. Скачайте нужную вам [версию](https://github.com/helm/helm/releases)
2. Распакуйте её (`tar -zxvf helm-v3.0.0-linux-amd64.tar.gz`)
3. Найдите binary файл `helm` в распакованной директории, и переместите его в нужное место (`mv linux-amd64/helm /usr/local/bin/helm`)

После этого вы сможете запустить клиент Helm и [добавить stable репозиторий](https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository):
`helm help`.

**Примечание:** Автоматические тесты Helm выполняются для Linux AMD64 только во время сборки и выпуска CircleCI.
Тестирование других OSes является обязанностью сообщества, использующего Helm на их OS.

### Из Скрипта

У Helm теперь есть скрипт установки, который автоматически загрузит последнюю версию Helm и 
[установит его локально](https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3).

Вы можете получить этот скрипт, а затем выполнить его локально.
Он хорошо задокументирован, так что вы можете прочесть его и понять что он делает, прежде чем его запускать.

```console
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

Yes, you can `curl
https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash` if
you want to live on the edge.

## Через Менеджеров Пакетов

Сообщество Helm предоставляет возможность установки Helm через
менеджеры пакетов операционной системы. 
Они не поддерживаются проектом Helm и не считаются проверенными.

### Используя Homebrew (macOS)

Члены сообщества Helm внесли свой вклад в создание formula Helm для Homebrew.
Эта formula, почти всегда, актуальна.

```console
brew install helm
```

(**Примечание:**: Существует также formula для emacs-helm, являющаяся другим проектом.)

### Используя Chocolatey (Windows)

Члены сообщества Helm внесли свой вклад в [Helm
пакет](https://chocolatey.org/packages/kubernetes-helm) собранный под
[Chocolatey](https://chocolatey.org/). Данная сборка почти всегда актуальна.

```console
choco install kubernetes-helm
```

### Используя Apt (Debian/Ubuntu)

Члены сообщества Helm внесли свой вклад в [Helm
package](https://helm.baltorepo.com/stable/debian/) для Apt. Данная сборка почти всегда актуальна.

```console
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

### Используя Snap

Сообщество [Snapcrafters](https://github.com/snapcrafters) поддерживает Snap-версию
[Helm пакета](https://snapcraft.io/helm):

```console
sudo snap install helm --classic
```

### Используя pkg (FreeBSD)

Члены сообщества FreeBSD внесли свой вклад в создание [Helm
пакета](https://www.freshports.org/sysutils/helm) под
[FreeBSD Ports Collection](https://man.freebsd.org/ports).
Данная сборка почти всегда актуальна.

```console
pkg install helm
```

### Тестовые Сборки

В дополнение к релизам вы можете скачать или установить тестовые сборки Helm.

### Используя Canary Сборки

Сборки Canary – это версии программного обеспечения Helm, построенные из последней
основной ветви. Они не являются официальными релизами и могут быть нестабильными. Тем не менее,
они предлагают возможность протестировать передовые функции.

Canary Helm binaries сборки хранятся в [get.helm.sh](https://get.helm.sh). Вот несколько
ссылок на общие сборки:

- [Linux AMD64](https://get.helm.sh/helm-canary-linux-amd64.tar.gz)
- [macOS AMD64](https://get.helm.sh/helm-canary-darwin-amd64.tar.gz)
- [Experimental Windows
  AMD64](https://get.helm.sh/helm-canary-windows-amd64.zip)

### Из Исходного Кода (Linux, macOS)

Сборка Helm из исходного кода требует больше усилий, но это лучший способ, если
вы хотите протестировать последнюю (pre-release) версию Helm.

У вас должна быть рабочая среда Go.

```console
$ git clone https://github.com/helm/helm.git
$ cd helm
$ make
```

При необходимости скрипт извлекает зависимости, кэширует их и проверяет
конфигурацию. Затем он скомпилирует `helm` и поместит его в `bin/helm`.

## Заключение

В большинстве случаев установка так же проста, как получение заранее собранного binary файла `helm`.
В этом документе описаны дополнительные случаи для тех, кто хочет делать
с Helm более сложные вещи.

После успешной установки клиента Helm вы можете перейти к использованию
Helm для управления chart-ми и [добавить stable репозиторий](https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository).
