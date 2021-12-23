git 2cf67bcaacfec590098cefb45af824b74671cfa0

---

# Laravel Homestead

- [Введение](#introduction)
- [Установка и настройка](#installation-and-setup)
    - [Первые шаги](#first-steps)
    - [Настройка Homestead](#configuring-homestead)
    - [Настройка Nginx](#configuring-nginx-sites)
    - [Настройка сервисов](#configuring-services)
    - [Запуск Vagrant Box](#launching-the-vagrant-box)
    - [Подготовка к установке](#per-project-installation)
    - [Установка дополнительных пакетов](#installing-optional-features)
    - [Псевдонимы](#aliases)
- [Обновление Homestead](#updating-homestead)
- [Ежедневное использование](#daily-usage)
    - [Подключение через SSH](#connecting-via-ssh)
    - [Добавление сайтов](#adding-additional-sites)
    - [Настройка окружения](#environment-variables)
    - [Порты](#ports)
    - [Версии PHP](#php-versions)
    - [Соединение с базой данных](#connecting-to-databases)
    - [Резервные копии базы данных](#database-backups)
    - [Настройка расписания Cron](#configuring-cron-schedules)
    - [Настройка MailHog](#configuring-mailhog)
    - [Настройка Minio](#configuring-minio)
    - [Laravel Dusk](#laravel-dusk)
    - [Совместное использование](#sharing-your-environment)
- [Отладка и профилирование](#debugging-and-profiling)
    - [Отладка веб-запросов](#debugging-web-requests)
    - [Отладка CLI-приложений](#debugging-cli-applications)
    - [Профилирование приложения с Blackfire](#profiling-applications-with-blackfire)
- [Сетевые интерфейсы](#network-interfaces)
- [Добавление команд Homestead](#extending-homestead)
- [Специфичные настройки](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## Введение

Laravel стремится сделать весь процесс разработки PHP приятным, включая вашу локальную среду разработки. [Laravel Homestead](https://github.com/laravel/homestead) - это официальный предварительно упакованный пакет Vagrant, который предоставляет вам прекрасную среду разработки, не требуя установки PHP, веб-сервера и любого другого серверного программного обеспечения на вашем локальном компьютере.

[Vagrant](https://www.vagrantup.com) предоставляет простой и элегантный способ управления виртуальными машинами и их подготовки. Vagrant-контейнеры полностью одноразовые. Если что-то пойдет не так, вы можете уничтожить и воссоздать контейнер за считанные минуты!

Homestead работает в любой системе Windows, macOS или Linux и включает Nginx, PHP, MySQL, PostgreSQL, Redis, Memcached, Node и все другое программное обеспечение, необходимое для разработки потрясающих приложений Laravel.

> {note} Если вы используете Windows, вам может потребоваться включить аппаратную виртуализацию (VT-x). Обычно его можно включить в BIOS. Если вы используете Hyper-V в системе UEFI, вам может дополнительно потребоваться отключить Hyper-V, чтобы получить доступ к VT-x.

<a name="included-software"></a>
### Включенное в набор программное обеспечение

- Ubuntu 20.04
- Git
- PHP 8.1
- PHP 8.0
- PHP 7.4
- PHP 7.3
- PHP 7.2
- PHP 7.1
- PHP 7.0
- PHP 5.6
- Nginx
- MySQL 8.0
- lmm
- Sqlite3
- PostgreSQL 13
- Composer
- Node (With Yarn, Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd
- Mailhog
- avahi
- ngrok
- Xdebug
- XHProf / Tideways / XHGui
- wp-cli

<a name="optional-software"></a>
### Дополнительное программное обеспечение

- Apache
- Blackfire
- Cassandra
- Chronograf
- CouchDB
- Crystal & Lucky Framework
- Docker
- Elasticsearch
- EventStoreDB
- Gearman
- Go
- Grafana
- InfluxDB
- MariaDB
- Meilisearch
- MinIO
- MongoDB
- Neo4j
- Oh My Zsh
- Open Resty
- PM2
- Python
- R
- RabbitMQ
- RVM (Ruby Version Manager)
- Solr
- TimescaleDB
- Trader <small>(PHP extension)</small>
- Webdriver & Laravel Dusk Utilities

<a name="installation-and-setup"></a>
## Установка и настройка

<a name="first-steps"></a>
### Первые шаги

Перед запуском среды Homestead необходимо установить [Vagrant](https://www.vagrantup.com/downloads.html), а также одного из следующих поддерживаемых провайдеров:

- [VirtualBox 6.1.x](https://www.virtualbox.org/wiki/Downloads)
- [Parallels](https://www.parallels.com/products/desktop/)

Все эти программные пакеты предоставляют простые в использовании визуальные установщики для всех популярных операционных систем.

Чтобы использовать провайдер Parallels, вам необходимо установить бесплатный плагин [Parallels Vagrant](https://github.com/Parallels/vagrant-parallels).

<a name="installing-homestead"></a>
#### Установка Homestead

Вы можете установить Homestead, клонировав репозиторий Homestead на свой компьютер. Рассмотрите возможность клонирования репозитория в папку `Homestead` в вашем домашнем каталоге, поскольку виртуальная машина Homestead будет служить хостом для всех ваших приложений Laravel. В этой документации мы будем называть этот каталог - «каталогом Homestead»:

```bash
git clone https://github.com/laravel/homestead.git ~/Homestead
```

После клонирования репозитория Laravel Homestead вы должны проверить ветку `release`. Эта ветка всегда содержит последний стабильный выпуск Homestead:

    cd ~/Homestead

    git checkout release

Затем выполните команду `bash init.sh` из каталога Homestead, чтобы создать файл конфигурации `Homestead.yaml`. Файл `Homestead.yaml` - это то место, где вы настраиваете все параметры установки Homestead. Этот файл будет помещен в каталог Homestead:

    // macOS / Linux...
    bash init.sh

    // Windows...
    init.bat

<a name="configuring-homestead"></a>
### Настройка Homestead

<a name="setting-your-provider"></a>
#### Настройка провайдера

Ключ `provider` в файле `Homestead.yaml` указывает, какой провайдер Vagrant следует использовать: `virtualbox` или `parallels`:

    provider: virtualbox

> {note} Если вы используете Apple Silicon, вам следует добавить `box: laravel/homestead-arm` в ваш файл `Homestead.yaml`. Apple Silicon требуется поставщик Parallels.

<a name="configuring-shared-folders"></a>
#### Настройка общих папок

Параметр `folder` файла `Homestead.yaml` перечисляет все директории, которыми вы хотите поделиться со своей виртуальной средой Homestead. При изменении файлов в этих папках они будут синхронизироваться между вашим локальным компьютером и средой Homestead. Вы можете настроить столько общих директорий, сколько необходимо:

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
```

> {note} Пользователи Windows, при указании пути не должны использовать синтаксис `~/`, а вместо этого должны указать полный путь к своему проекту от корня диска, например `C:\Users\user\Code\project1`.

Вы всегда должны сопоставлять каждое ваше приложение с его собственной отдельной директорией вместо назначения одного большого каталога, содержащего все ваши приложения. При назначении папки приложению виртуальная машина должна отслеживать все операции ввода-вывода на диске для *каждого* файла в папке. Поэтому у вас может снизиться производительность среды, если в папке много файлов:

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
    - map: ~/code/project2
      to: /home/vagrant/project2
```

> {note} Вы никогда не должны монтировать `.` (текущий каталог) при использовании Homestead. Это приводит к тому, что Vagrant не отображает текущую папку в `/vagrant`, что нарушает работу дополнительных функций и приводит к неожиданным результатам при подготовке.

Чтобы включить [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html), вы можете добавить параметр `type` при сопоставлении папок:

    folders:
        - map: ~/code/project1
          to: /home/vagrant/project1
          type: "nfs"

> {note} При использовании NFS в Windows вам следует рассмотреть возможность установки подключаемого модуля [vagrant-winnfsd](https://github.com/winnfsd/vagrant-winnfsd). Этот плагин будет поддерживать правильные разрешения пользователя / группы для файлов и каталогов на виртуальной машине Homestead.

Вы также можете передать любые параметры, поддерживаемые [общими папками Vagrant](https://www.vagrantup.com/docs/synced-folders/basic_usage.html), указав их под ключом options:

    folders:
        - map: ~/code/project1
          to: /home/vagrant/project1
          type: "rsync"
          options:
              rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
              rsync__exclude: ["node_modules"]

<a name="configuring-nginx-sites"></a>
### Настройка Nginx

Не знаком с Nginx? Нет проблем! Свойство `sites` файла `Homestead.yaml` позволяет легко сопоставить "домен" с папкой в среде Homestead. Пример конфигурации сайта включен в файл `Homestead.yaml`. Опять же, вы можете добавить столько сайтов в среду Homestead, сколько необходимо. Homestead может служить удобной виртуальной средой для каждого приложения Laravel, над которым вы работаете:

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public

Если вы измените свойство `sites` после подготовки виртуальной машины Homestead, вы должны выполнить команду `vagrant reload --provision` в своем терминале, чтобы обновить конфигурацию Nginx на виртуальной машине.

> {note} Скрипты Homestead созданы максимально [идемпотентными](https://ru.wikipedia.org/wiki/Идемпотентность). Однако, если у вас возникли проблемы во время подготовки, вам следует удалить и повторно запустить виртуальную машину, выполнив команду `vagrant destroy && vagrant up`.

<a name="hostname-resolution"></a>
#### Определение имени хоста

Homestead публикует имена хостов, используя `mDNS` для автоматического определения хостов. Если вы установите `hostname: homestead` в вашем файле `Homestead.yaml`, хост будет доступен по адресу `homestead.local`. Настольные дистрибутивы macOS, iOS и Linux по умолчанию включают поддержку `mDNS`. Если вы используете Windows, вы должны установить [Bonjour Print Services для Windows](https://support.apple.com/kb/DL999?viewlocale=en_US&locale=en_US).

Настройку имен хостов лучше всего проводить при [подготовке к установке](#per-project-installation) Homestead. Если вы размещаете несколько сайтов на одном экземпляре Homestead, вы можете добавить домены для своих веб-сайтов в файл `hosts` на вашем компьютере. Файл `hosts` будет перенаправлять запросы для ваших сайтов Homestead на вашу виртуальную машину Homestead. В macOS и Linux этот файл находится в `/etc/hosts`. В Windows он находится в `C:\Windows\System32\drivers\etc\hosts`. Строки, которые вы добавляете в этот файл, будут выглядеть следующим образом:

    192.168.56.56  homestead.test

Убедитесь, что в списке указан IP-адрес, указанный в вашем файле `Homestead.yaml`. После того как вы добавили домен в файл `hosts` и запустили Vagrant-контейнер, вы сможете получить доступ к сайту через свой веб-браузер:

```bash
http://homestead.test
```

<a name="configuring-services"></a>
### Настройка сервисов

По умолчанию Homestead запускает несколько сервисов; однако вы можете настроить, какие службы будут включены или отключены во время подготовки. Например, вы можете включить PostgreSQL и отключить MySQL, изменив параметр `services` в файле `Homestead.yaml`:

```yaml
services:
    - enabled:
        - "postgresql"
    - disabled:
        - "mysql"
```

Указанные службы будут запускаться или останавливаться в зависимости от их порядка в директивах `enabled`и `disabled`.

<a name="launching-the-vagrant-box"></a>
### Запуск Vagrant Box

После того как вы отредактировали файл `Homestead.yaml` по своему вкусу, запустите команду `vagrant up` из каталога Homestead. Vagrant загрузит виртуальную машину и автоматически настроит ваши общие папки и сайты Nginx.

Чтобы удалить машину, вы можете использовать команду `vagrant destroy`.

<a name="per-project-installation"></a>
### Подготовка к установке

Вместо того чтобы устанавливать Homestead глобально и использовать одну и ту же виртуальную машину Homestead для всех ваших проектов, вы можете настроить экземпляр Homestead для каждого проекта, которым вы управляете. Установка Homestead для каждого проекта может быть полезной, если вы хотите опубликовать `Vagrantfile` вместе с вашим проектом, позволяя другим, работающим над проектом, пользоваться проектом сразу после клонирования репозитория проекта.

Вы можете установить Homestead в свой проект с помощью диспетчера пакетов Composer:

```bash
composer require laravel/homestead --dev
```

Как только Homestead будет установлен, вызовите команду Homestead `make`, чтобы сгенерировать файлы `Vagrantfile` и `Homestead.yaml` для вашего проекта. Эти файлы будут помещены в корень проекта. Команда `make` автоматически настроит директивы `sites` и `folder` в файле `Homestead.yaml`:

    // macOS / Linux...
    php vendor/bin/homestead make

    // Windows...
    vendor\\bin\\homestead make

Затем запустите команду `vagrant up` в вашем терминале и войдите в проект по адресу `http://homestead.test` в браузере. Помните, что вам все равно нужно будет добавить запись файла `/etc/hosts` для `homestead.test` или домена по вашему выбору, если вы не используете автоматическое [определение имени хоста](#hostname-resolution).

<a name="installing-optional-features"></a>
### Установка дополнительных пакетов

Дополнительное программное обеспечение устанавливается с помощью опции `features` в файле `Homestead.yaml`. Большинство функций можно включить или отключить с помощью логического значения, в то время как некоторые функции позволяют использовать несколько параметров конфигурации:

    features:
        - blackfire:
            server_id: "server_id"
            server_token: "server_value"
            client_id: "client_id"
            client_token: "client_value"
        - cassandra: true
        - chronograf: true
        - couchdb: true
        - crystal: true
        - docker: true
        - elasticsearch:
            version: 7.9.0
        - eventstore: true
            version: 21.2.0
        - gearman: true
        - golang: true
        - grafana: true
        - influxdb: true
        - mariadb: true
        - meilisearch: true
        - minio: true
        - mongodb: true
        - neo4j: true
        - ohmyzsh: true
        - openresty: true
        - pm2: true
        - python: true
        - r-base: true
        - rabbitmq: true
        - rvm: true
        - solr: true
        - timescaledb: true
        - trader: true
        - webdriver: true

<a name="elasticsearch"></a>
#### Elasticsearch (поисковая система)

Вы можете указать поддерживаемую версию Elasticsearch, которая должна быть точным номером версии (major.minor.patch). При установке по умолчанию будет создан кластер с именем «homestead». Никогда не следует отдавать Elasticsearch больше половины памяти операционной системы, поэтому убедитесь, что на вашей виртуальной машине Homestead выделено как минимум вдвое больше памяти Elasticsearch.

> {tip} Ознакомьтесь с [документацией Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current), чтобы узнать, как настроить свою конфигурацию.

<a name="mariadb"></a>
#### MariaDB

Включение MariaDB удалит MySQL и установит MariaDB. MariaDB обычно служит заменой MySQL, поэтому вам все равно следует использовать драйвер базы данных `mysql` в конфигурации базы данных вашего приложения.

<a name="mongodb"></a>
#### MongoDB

При установке MongoDB по умолчанию для имени пользователя базы данных будет установлено значение `homestead`, а для соответствующего пароля - `secret`.

<a name="neo4j"></a>
#### Neo4j

При установке Neo4j по умолчанию для имени пользователя базы данных будет установлено значение `homestead`, а для соответствующего пароля - `secret`. Чтобы получить доступ к браузеру Neo4j, зайдите на сайт `http://homestead.test:7474` в своем браузере. Порты `7687` (Bolt), `7474` (HTTP) и `7473` (HTTPS) готовы обслуживать запросы от клиента Neo4j.

<a name="aliases"></a>
### Псевдонимы

Вы можете добавить псевдонимы Bash на свою виртуальную машину Homestead, изменив файл `aliases` в каталоге Homestead:

    alias c='clear'
    alias ..='cd ..'

После обновления файла `aliases` вам следует повторно подготовить виртуальную машину Homestead с помощью команды `vagrant reload --provision`. Это обеспечит доступность ваших новых псевдонимов на машине.

<a name="updating-homestead"></a>
## Обновление Homestead

Перед тем, как начать обновление Homestead, убедитесь, что вы удалили текущую виртуальную машину, выполнив следующую команду в каталоге Homestead:

    vagrant destroy

Затем вам нужно обновить исходный код Homestead. Если вы клонировали репозиторий, вы можете выполнить следующие команды в том месте, где вы изначально клонировали репозиторий:

    git fetch

    git pull origin release

Эти команды скачивают последний код Homestead из репозитория GitHub, извлекают теги, а затем проверяют выпуск с тегами. Вы можете найти последнюю стабильную версию выпуска Homestead на [странице релизов GitHub](https://github.com/laravel/homestead/releases).

Если вы установили Homestead через файл `composer.json` вашего проекта, вы должны убедиться, что файл `composer.json` содержит `"laravel/homestead": "^12"` и обновите ваши зависимости:

    composer update

Затем вы должны обновить поле Vagrant с помощью команды `vagrant box update`:

    vagrant box update

После обновления Vagrant вы должны запустить команду `bash init.sh` из каталога Homestead, чтобы обновить дополнительные файлы конфигурации Homestead. Затем вас спросят, хотите ли вы перезаписать существующие файлы `Homestead.yaml`, `after.sh` и `aliases`:

    // macOS / Linux...
    bash init.sh

    // Windows...
    init.bat

Наконец, вам нужно будет обновить виртуальную машину Homestead, чтобы использовать последнюю установку Vagrant:

    vagrant up

<a name="daily-usage"></a>
## Ежедневное использование

<a name="connecting-via-ssh"></a>
### Подключение через SSH

Вы можете подключиться к вашей виртуальной машине по SSH, выполнив команду терминала `vagrant ssh` из вашего каталога Homestead.

<a name="adding-additional-sites"></a>
### Добавление сайтов

После того как ваша среда Homestead подготовлена и запущена, вы можете добавить дополнительные сайты Nginx для других ваших проектов Laravel. Вы можете запускать столько проектов Laravel, сколько хотите, в одной среде Homestead. Чтобы добавить дополнительный сайт, добавьте его в файл `Homestead.yaml`.

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public
        - map: another.test
          to: /home/vagrant/project2/public

> {note} Перед добавлением сайта убедитесь, что вы настроили [сопоставление папок](#configuring-shared-folders) для каталога проекта.

Если Vagrant не управляет вашим файлом «hosts» автоматически, вам может потребоваться также добавить новый сайт в этот файл. В macOS и Linux этот файл находится в `/etc/hosts`. В Windows он находится в `C:\Windows\System32\drivers\etc\hosts`:

    192.168.56.56  homestead.test
    192.168.56.56  another.test

После добавления сайта выполните команду терминала `vagrant reload --provision` из каталога Homestead.

<a name="site-types"></a>
#### Типы сайтов

Homestead поддерживает несколько «типов» сайтов, которые позволяют легко запускать проекты, не основанные на Laravel. Например, мы можем легко добавить приложение [Statamic](https://statamic.com/) в Homestead, используя тип сайта `statamic`:

```yaml
sites:
    - map: statamic.test
      to: /home/vagrant/my-symfony-project/web
      type: "statamic"
```

Доступные типы сайтов: `apache`, `apigility`, `expressive`, `laravel` (по умолчанию), `proxy`, `silverstripe`, `statamic`, `symfony2`, `symfony4`, and `zf`.

<a name="site-parameters"></a>
#### Параметры сайтов

Вы можете добавить дополнительные значения `fastcgi_param` Nginx на свой сайт с помощью директивы сайта `params`:

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public
          params:
              - key: FOO
                value: BAR

<a name="environment-variables"></a>
### Настройка окружения

Вы можете определить глобальные переменные окружения, добавив их в свой файл `Homestead.yaml`:

    variables:
        - key: APP_ENV
          value: local
        - key: FOO
          value: bar

После обновления файла `Homestead.yaml` не забудьте перезагрузить виртуальную машину, выполнив команду `vagrant reload --provision`. Это обновит конфигурацию PHP-FPM для всех установленных версий PHP, а также обновит среду для пользователя `vagrant`.

<a name="ports"></a>
### Порты

По умолчанию в среду Homestead перенаправляются следующие порты:

- **HTTP:** 8000 &rarr; перенаправляется на 80
- **HTTPS:** 44300 &rarr; перенаправляется на 443

<a name="forwarding-additional-ports"></a>
#### Перенаправление дополнительных портов

Вы можете перенаправить дополнительные порты в контейнер Vagrant, указав запись `ports` в конфигурационном файле `Homestead.yaml`. После обновления файла `Homestead.yaml` не забудьте повторно перезагрузить виртуальную машину, выполнив команду` vagrant reload --provision`:

    ports:
        - send: 50000
          to: 5000
        - send: 7777
          to: 777
          protocol: udp

Ниже приведен список дополнительных сервисных портов Homestead, которые вы, возможно, захотите перенаправить с вашего хост-компьютера на ваш Vagrant box:

- **SSH:** 2222 &rarr; 22
- **ngrok UI:** 4040 &rarr; 4040
- **MySQL:** 33060 &rarr; 3306
- **PostgreSQL:** 54320 &rarr; 5432
- **MongoDB:** 27017 &rarr; 27017
- **Mailhog:** 8025 &rarr; 8025
- **Minio:** 9600 &rarr; 9600

<a name="php-versions"></a>
### Версии PHP

В Homestead 6 появилась поддержка запуска нескольких версий PHP на одной виртуальной машине. Вы можете указать, какую версию PHP использовать для данного сайта в файле `Homestead.yaml`. Доступные версии PHP: «5.6», «7.0», «7.1», «7.2», «7.3», «7.4», «8.0» и "8.1" (по умолчанию):

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public
          php: "7.1"

[На виртуальной машине Homestead](#connected-via-ssh) вы можете использовать любую из поддерживаемых версий PHP через интерфейс командной строки:

    php5.6 artisan list
    php7.0 artisan list
    php7.1 artisan list
    php7.2 artisan list
    php7.3 artisan list
    php7.4 artisan list
    php8.0 artisan list
    php8.1 artisan list

Вы можете изменить версию PHP по умолчанию, используемую CLI, выполнив следующие команды на своей виртуальной машине Homestead:

    php56
    php70
    php71
    php72
    php73
    php74
    php80
    php81

<a name="connecting-to-databases"></a>
### Соединение с базой данных

База данных `homestead` настраивается как для MySQL, так и для PostgreSQL из коробки. Чтобы подключиться к вашей базе данных MySQL или PostgreSQL из клиента вашего хост-компьютера, вы должны подключиться к `127.0.0.1` через порт `33060` (MySQL) или `54320` (PostgreSQL). Имя пользователя и пароль для обеих баз данных - `homestead` / `secret`.

> {note} Вы должны использовать эти нестандартные порты только при подключении к базам данных с вашего хост-компьютера. Вы будете использовать порты 3306 и 5432 по умолчанию в файле конфигурации вашего приложения Laravel `database`, поскольку Laravel работает _внутри_ виртуальной машины.

<a name="database-backups"></a>
### Резервные копии базы данных

Homestead может автоматически создавать резервную копию базы данных, когда ваша виртуальная машина Homestead будет удалена. Чтобы использовать эту функцию, вы должны использовать Vagrant 2.1.0 или выше. Или, если вы используете старую версию Vagrant, вы должны установить плагин `vagrant-triggers`. Чтобы включить автоматическое резервное копирование базы данных, добавьте следующую строку в ваш файл `Homestead.yaml`:

    backup: true

После настройки Homestead будет экспортировать ваши базы данных в каталоги `mysql_backup` и `postgres_backup` при выполнении команды `vagrant destroy`. Эти каталоги можно найти в папке, в которую вы установили Homestead, или в корне вашего проекта, если вы использовали метод [подготовка к установке](#per-project-installation).

<a name="configuring-cron-schedules"></a>
### Настройка расписания Cron

Laravel предоставляет удобный способ [запланировать задания cron](/docs/{{version}}/scheduling), через выполнение Artisan-команды `schedule: run` каждую минуту. Команда `schedule: run` проверяет расписание заданий, записанные в классе `App\Console\Kernel`, чтобы определить, какие запланированные задачи нужно выполнить.

Если вы хотите, чтобы команда `schedule: run` запускалась для сайта Homestead, вы можете установить для параметра `schedule` значение `true` при определении сайта:

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      schedule: true
```

Задание cron для сайта будет записано в каталоге `/etc/cron.d` виртуальной машины Homestead.

<a name="configuring-mailhog"></a>
### Настройка MailHog

[MailHog](https://github.com/mailhog/MailHog) позволяет вам перехватывать исходящую электронную почту и проверять ее, не отправляя ее получателям. Для начала обновите файл `.env` вашего приложения, чтобы использовать следующие настройки почты:

    MAIL_MAILER=smtp
    MAIL_HOST=localhost
    MAIL_PORT=1025
    MAIL_USERNAME=null
    MAIL_PASSWORD=null
    MAIL_ENCRYPTION=null

После настройки MailHog вы можете получить доступ к панели управления MailHog по адресу `http://localhost:8025`.

<a name="configuring-minio"></a>
### Настройка Minio

[Minio](https://github.com/minio/minio) - это сервер хранения объектов с открытым исходным кодом и API, совместимый с Amazon S3. Чтобы установить Minio, обновите файл `Homestead.yaml`, указав следующую опцию конфигурации в разделе [features](#install-optional-features):

    minio: true

По умолчанию Minio доступен через порт 9600. Вы можете получить доступ к панели управления Minio, посетив `http://localhost:9600`. Ключ доступа по умолчанию - `homestead`, а секретный ключ по умолчанию - `secretkey`. При доступе к Minio вы всегда должны использовать регион `us-east-1`.

Чтобы использовать Minio, вам необходимо настроить конфигурацию диска S3 в файле конфигурации вашего приложения `config/filesystems.php`. Вам нужно будет добавить параметр `use_path_style_endpoint` в конфигурацию диска, а также изменить ключ `url` на `endpoint`:

    's3' => [
        'driver' => 's3',
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION'),
        'bucket' => env('AWS_BUCKET'),
        'endpoint' => env('AWS_URL'),
        'use_path_style_endpoint' => true,
    ]

Наконец, убедитесь, что ваш файл `.env` содержит следующие параметры:

```bash
AWS_ACCESS_KEY_ID=homestead
AWS_SECRET_ACCESS_KEY=secretkey
AWS_DEFAULT_REGION=us-east-1
AWS_URL=http://localhost:9600
```

Чтобы подготовить контейнеры "S3" на базе Minio, добавьте директиву `buckets` в ваш файл `Homestead.yaml`. После определения контейнеров вы должны выполнить команду `vagrant reload --provision` в терминале:

```yaml
buckets:
    - name: your-bucket
      policy: public
    - name: your-private-bucket
      policy: none
```

Поддерживаемые значения `policy` включают в себя: `none`, `download`, `upload`, and `public`.

<a name="laravel-dusk"></a>
### Laravel Dusk

Чтобы запустить тесты [Laravel Dusk](/docs/{{version}}/dusk) в Homestead, вы должны включить [`webdriver` feature](#installing-optional-features) в вашей конфигурации Homestead:

```yaml
features:
    - webdriver: true
```

После включения функции `webdriver` вы должны выполнить команду `vagrant reload --provision` в терминале.

<a name="sharing-your-environment"></a>
### Совместное использование

Иногда вы захотите поделиться тем, над чем сейчас работаете, с коллегами или клиентом. Vagrant имеет встроенную поддержку для этого с помощью команды `vagrant share`; однако это не сработает, если в файле `Homestead.yaml` настроено несколько сайтов.

Чтобы решить эту проблему, Homestead включает собственную команду `share`. Для начала [подключитесь через SSH к вашей виртуальной машине Homestead](#connecting-via-ssh) через `vagrant ssh` и выполните команду `share homestead.test`. Эта команда предоставит общий доступ к сайту `homestead.test` из файла конфигурации` Homestead.yaml`. Вы можете заменить любой из других настроенных вами сайтов на `homestead.test`:

    share homestead.test

После выполнения команды вы увидите экран Ngrok, который содержит журнал активности и общедоступные URL-адреса для общего сайта. Если вы хотите указать настраиваемый регион, поддомен или другую опцию Ngrok, вы можете добавить их в команду `share`:

    share homestead.test -region=eu -subdomain=laravel

> {note} Помните, что Vagrant по своей сути небезопасен, и вы открываете свою виртуальную машину для доступа из Интернета, выполняя команду `share`.

<a name="debugging-and-profiling"></a>
## Отладка и профилирование

<a name="debugging-web-requests"></a>
### Отладка веб-запросов

Homestead включает поддержку пошаговой отладки с использованием [Xdebug](https://xdebug.org). Например, вы можете получить доступ к странице в своем браузере, и PHP подключится к вашей среде IDE, чтобы разрешить проверку и изменение выполняемого кода.

По умолчанию Xdebug уже запущен и готов принимать подключения. Если вам нужно включить Xdebug в CLI, выполните команду `sudo phpenmod xdebug` на виртуальной машине Homestead. Затем следуйте инструкциям IDE, чтобы включить отладку. Наконец, настройте свой браузер для запуска Xdebug с расширением или [букмарклетом](https://www.jetbrains.com/phpstorm/marklets/).

> {note} Xdebug заставляет PHP работать значительно медленнее. Чтобы отключить Xdebug, запустите `sudo phpdismod xdebug` на виртуальной машине Homestead и перезапустите службу FPM.

<a name="autostarting-xdebug"></a>
#### Автозапуск Xdebug

При отладке функциональных тестов, которые отправляют запросы к веб-серверу, проще автоматически запускать отладку, чем изменять тесты для прохождения через настраиваемый заголовок или файл cookie для запуска отладки. Чтобы заставить Xdebug запускаться автоматически, измените файл `/etc/php/7.x/fpm/conf.d/20-xdebug.ini` внутри виртуальной машины Homestead и добавьте следующую конфигурацию:

```ini
; If Homestead.yaml contains a different subnet for the IP address, this address may be different...
xdebug.remote_host = 192.168.10.1
xdebug.remote_autostart = 1
```

<a name="debugging-cli-applications"></a>
### Отладка CLI-приложений

Чтобы отладить CLI-приложение PHP, используйте псевдоним оболочки `xphp` внутри вашей виртуальной машины Homestead:

    xphp /path/to/script

<a name="profiling-applications-with-blackfire"></a>
### Профилирование приложения с Blackfire

[Blackfire](https://blackfire.io/docs/introduction) - это сервис для профилирования веб-запросов и CLI-приложений. Он предлагает интерактивный пользовательский интерфейс, который отображает данные профиля в виде графиков вызовов и временных шкал. Он создан для использования в разработке, тестировании и производстве без дополнительных затрат для конечных пользователей. Кроме того, Blackfire обеспечивает проверку производительности, качества и безопасности кода и параметров конфигурации `php.ini`.

[Blackfire Player](https://blackfire.io/docs/player/index) - это приложение с открытым исходным кодом для веб-сканирования, веб-тестирования и веб-скрапинга, которое может работать совместно с Blackfire для создания сценариев профилирования.

Чтобы включить Blackfire, используйте параметр "features" в файле конфигурации Homestead:

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
```

Учетные данные сервера Blackfire и учетные данные клиента [требуют наличие аккаунта Blackfire](https://blackfire.io/signup). Blackfire предлагает различные варианты профилирования приложения, включая инструмент командной строки и расширение браузера. Пожалуйста, [просмотрите документацию Blackfire для получения более подробной информации](https://blackfire.io/docs/cookbooks/index).

<a name="network-interfaces"></a>
## Сетевые интерфейсы

Свойство `networks` файла `Homestead.yaml` настраивает сетевые интерфейсы для вашей виртуальной машины Homestead. Вы можете настроить столько интерфейсов, сколько необходимо:

```yaml
networks:
    - type: "private_network"
      ip: "192.168.10.20"
```

Чтобы включить [мостовой (bridge)](https://www.vagrantup.com/docs/networking/public_network.html) интерфейс, настройте параметр `bridge` для сети и измените тип сети на `public_network`:

```yaml
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
```

Чтобы включить [DHCP](https://www.vagrantup.com/docs/networking/public_network.html), просто удалите параметр ip из вашей конфигурации:

```yaml
networks:
    - type: "public_network"
      bridge: "en1: Wi-Fi (AirPort)"
```

<a name="extending-homestead"></a>
## Добавление команд Homestead

Вы можете расширить Homestead, используя сценарий `after.sh` в корне каталога Homestead. В этот файл вы можете добавить любые команды оболочки, которые необходимы для правильной настройки вашей виртуальной машины.

При настройке Homestead, Ubuntu может спросить вас, сохранять ли исходную конфигурацию пакета или перезаписать ее новым файлом конфигурации. Чтобы избежать этого, вы должны использовать следующую команду, чтобы избежать перезаписи любой конфигурации, ранее записанной Homestead:

    sudo apt-get -y \
        -o Dpkg::Options::="--force-confdef" \
        -o Dpkg::Options::="--force-confold" \
        install package-name

<a name="user-customizations"></a>
### Пользовательские настройки

При использовании Homestead вместе со своей командой вы можете настроить Homestead, чтобы он лучше соответствовал вашему личному стилю разработки. Для этого вы можете создать файл `user-customizations.sh` в корне каталога Homestead (тот же каталог, где находится ваш файл` Homestead.yaml`). В этом файле вы можете сделать любую настройку, какую захотите; однако файл `user-customizations.sh` должен быть добавлен в список игнорируемых, если вы пользуетесь системами контроля версий ([VCS](https://ru.wikipedia.org/wiki/Система_управления_версиями)).

<a name="provider-specific-settings"></a>
## Специфичные настройки

<a name="provider-specific-virtualbox"></a>
### VirtualBox

<a name="natdnshostresolver"></a>
#### `natdnshostresolver`

По умолчанию Homestead устанавливает для параметра `natdnshostresolver` значение` on`. Это позволяет Homestead использовать настройки DNS вашей операционной системы. Если вы хотите изменить это поведение, добавьте следующие параметры конфигурации в ваш файл `Homestead.yaml`:

```yaml
provider: virtualbox
natdnshostresolver: 'off'
```

<a name="symbolic-links-on-windows"></a>
#### Символические ссылки в Windows

Если символические ссылки не работают должным образом на вашем компьютере с Windows, вам может потребоваться добавить следующий блок в ваш `Vagrantfile`:

```ruby
config.vm.provider "virtualbox" do |v|
    v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
end
```
