git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Laravel Homestead

- [Введение](#introduction)
- [Установка и настройка](#installation-and-setup)
    - [Первые шаги](#first-steps)
    - [Настройка Homestead](#configuring-homestead)
    - [Запуск Vagrant Box](#launching-the-vagrant-box)
    - [Установка для проекта](#per-project-installation)
    - [Установка MariaDB](#installing-mariadb)
- [Повседневное использование](#daily-usage)
    - [Глобальный доступ к Homestead](#accessing-homestead-globally)
    - [Подключение через SSH](#connecting-via-ssh)
    - [Подключение к базам данных](#connecting-to-databases)
    - [Добавление дополнительных сайтов](#adding-additional-sites)
    - [Настройка расписания Cron](#configuring-cron-schedules)
    - [Порты](#ports)
    - [Совместное использование вашей среды](#sharing-your-environment)
    - [Несколько версий PHP](#multiple-php-versions)
- [Сетевые интерфейсы](#network-interfaces)
- [Обновление Homestead](#updating-homestead)
- [Старые версии](#old-versions)
- [Специальные настройки провайдера](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## Введение

Laravel стремится преобразить процесс разработки на PHP, это относится и к локальной среде разработки. [Vagrant](https://www.vagrantup.com) обеспечивает простой, элегантный способ настройки и управления виртуальными машинами.

Laravel Homestead — официальный предустановленный Vagrant-бокс, который предоставляет вам замечательную среду разработки без необходимости установки PHP, веб-сервера и любого другого серверного программного обеспечения на ваш компьютер. Можно больше не беспокоиться о том, что ваша операционная система засоряется! Vagrant-боксы очень удобны. Если что-то пошло не так, вы можете уничтожить и пересоздать бокс в считанные минуты!

Homestead запускается на ОС Windows, Mac и Linux, и включает в себя веб-сервер Nginx, PHP 7.1, MySQL, Postgres, Redis, Memcached, Node и все другие полезные штуки, которые вам понадобятся для разработки удивительных Laravel-приложений.

> {note} Если вы используете Windows, возможно, вам необходимо включить аппаратную виртуализацию (VT-x). Она обычно включается через BIOS. Если вы используете Hyper-V на UEFI-системе, вам может понадобиться отключить Hyper-V, для доступа к VT-x.

<a name="included-software"></a>
### Включённое ПО

- Ubuntu 16.04
- Git
- PHP 7.1
- Nginx
- MySQL
- MariaDB
- Sqlite3
- Postgres
- Composer
- Node (с Yarn, Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd
- Mailhog
- ngrok

<a name="installation-and-setup"></a>
## Установка и настройка

<a name="first-steps"></a>
### Первые шаги

Прежде чем запустить среду Homestead, вы должны установить [VirtualBox 5.1](https://www.virtualbox.org/wiki/Downloads), [VMWare](https://www.vmware.com) или [Parallels](http://www.parallels.com/products/desktop/), а также [Vagrant](https://www.vagrantup.com/downloads.html). Эти программные пакеты предоставляют простые в использовании визуальные инсталляторы для всех популярных операционных систем.

Для использования VMWare вам необходимо приобрести и VMware Fusion/Workstation, и [плагин VMware Vagrant](https://www.vagrantup.com/vmware). Хотя он и платный, зато VMware изначально обеспечивает большую скорость работы общих папок.

Для использования провайдера Parallels вам необходимо установить [плагин Parallels Vagrant](https://github.com/Parallels/vagrant-parallels). Он бесплатный.

#### Установка Homestead Vagrant Box

После установки VirtualBox / VMware и Vagrant вы должны добавить бокс `laravel/homestead` в ваш Vagrant, используя следующую команду в вашем терминале. Скачивание образа может занять несколько минут в зависимости от скорости вашего Интернет-подключения:

    vagrant box add laravel/homestead

Если выполнение команды завершится неудачно, проверьте, что у вас установлена свежая версия Vagrant.

#### Установка Homestead

Вы можете установить Homestead просто клонировав репозиторий. Клонируйте репозиторий в папку `Homestead` в директорию "home", потому что коробка Homestead станет хостом всех ваших Laravel-проектов:

    cd ~

    git clone https://github.com/laravel/homestead.git Homestead

Вы должны проверить меченную версию Homestead, так как ветка `master` не всегда может быть стабильна. Самую свежую стабильную сервисю можно найти на [странице релиза в GitHub](https://github.com/laravel/homestead/releases):

    cd Homestead

    // Клонировать желаемый релиз...
    git checkout v5.4.0

После клонирования репозитория Homestead выполните в этой директории команду `bash init.sh`, чтобы создать конфиг `Homestead.yaml`. Файл `Homestead.yaml` будет помещен в директорию Homestead:

    // Mac / Linux...
    bash init.sh

    // Windows...
    init.bat

<a name="configuring-homestead"></a>
### Настройка Homestead

#### Настройка провайдера

Параметр `provider` в вашем файле `Homestead.yaml` указывает на то, какой Vagrant-провайдер следует использовать: `virtualbox`, `vmware_fusion`, `vmware_workstation` или `parallels`. Вы можете задать тот, который предпочитаете:

    provider: virtualbox

#### Настройка общих папок

В свойстве `folders` файла `Homestead.yaml` перечислены все папки, которые вы хотите расшарить для вашей среды Homestead. Поскольку файлы в этих папках будут меняться, они будут синхронизироваться с вашей локальной машиной и средой Homestead. Вы можете настроить столько папок, сколько вам необходимо:

    folders:
        - map: ~/Code
          to: /home/vagrant/Code

Для включения [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html) просто добавьте простой ключ к вашей синхронизируемой папке:

    folders:
        - map: ~/Code
          to: /home/vagrant/Code
          type: "nfs"

> {note} При использовании NFS вам следует рассмотреть возможность установки плагина [vagrant-bindfs](https://github.com/gael-ian/vagrant-bindfs). Этот плагин будет поддерживать верные права пользователя / группы относительно файлов / директорий для файлов и директорий в боксе Homestead.

Также вы можете передавать параметры, поддерживаемые [синхронизируемыми папками](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) Vagrant, указывая их под ключом `options`:

    folders:
        - map: ~/Code
          to: /home/vagrant/Code
          type: "rsync"
          options:
              rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
              rsync__exclude: ["node_modules"]


#### Настройка сайтов Nginx 

Не знакомы с Nginx? Не проблема. Параметр `sites` позволяет легко связать "домен" с папкой в среде Homestead. Типовая конфигурация сайта включена в файл `Homestead.yaml`. И снова, вы можете добавить столько сайтов к своей среде Homestead, сколько необходимо. Homestead может служить удобной виртуальной средой для каждого проекта Laravel, над которым вы работаете:

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public

Если вы измените параметр `sites` после подключения Homestead-бокса, необходимо перезапустить `vagrant reload --provision`, чтобы обновить конфигурацию Nginx на виртуальной машине.

#### Файл Hosts

Вы должны добавить "домены" для своих Nginx-сайтов в файл `hosts` на вашей машине. Файл `hosts` перенаправит запросы к вашим сайтам в вашу машину Homestead. На Mac и Linux этот файл расположен в `/etc/hosts`. На Windows он расположен в `C:\Windows\System32\drivers\etc\hosts`. Строки, которые вы добавляете в этот файл, будут выглядеть примерно так:

    192.168.10.10  homestead.app

Удостоверьтесь, что IP-адрес тот же, что вы установили в своём файле `Homestead.yaml`. Когда вы добавите домен в свой файл `hosts` и запустите Vagrant-бокс, вы можете получить доступ к сайту через свой веб-браузер:

    http://homestead.app

<a name="launching-the-vagrant-box"></a>
### Запуск Vagrant Box

Когда вы отредактировали `Homestead.yaml` по собственному усмотрению, выполните команду `vagrant up` в папке Homestead. Vagrant загрузит виртуальную машину и настроит ваши общие папки и сайты Nginx автоматически.

Чтобы уничтожить машину, вы можете использовать команду `vagrant destroy --force`.

<a name="per-project-installation"></a>
### Установка для проекта

Вместо глобальной установки Homestead и использования одного Homestead-бокса для всех ваших проектов, вы можете настроить отдельный экземпляр Homestead для каждого проекта. Установка Homestead для проекта может быть выгоднее, когда вы хотите поставлять файл `Vagrantfile` вместе с вашим проектом, позволяя тем, кто работает над проектом, просто выполнять `vagrant up`.

Чтобы установить Homestead непосредственно в ваш проект, затребуйте его с помощью Composer:

    composer require laravel/homestead --dev

Когда Homestead установлен, используйте команду `make` для создания `Vagrantfile` и файла `Homestead.yaml` в корне вашего проекта. Команда `make` автоматически настроит директивы `sites` м `folders` в файле `Homestead.yaml`.

Mac / Linux:

    php vendor/bin/homestead make

Windows:

    vendor\\bin\\homestead make

Затем выполните команду `vagrant up`  терминале и зайдите в свой проект по адресу `http://homestead.app` через браузер. Не забывайте, что вам по-прежнему необходимо добавить строку для `homestead.app` или любого другого выбранного вами домена в `/etc/hosts`.

<a name="installing-mariadb"></a>
### Установка MariaDB

Если вы решили использовать MariaDB вместо MySQL, вы можете добавить параметр `mariadb` в свой файл `Homestead.yaml`. Этот параметр удалит MySQL и установит MariaDB. MariaDB является полноценной заменой для MySQL, поэтому вы должны использовать тот же драйвер БД `mysql` в настройках базы данных приложения:

    box: laravel/homestead
    ip: "192.168.20.20"
    memory: 2048
    cpus: 4
    provider: virtualbox
    mariadb: true

<a name="daily-usage"></a>
## Повседневное использование

<a name="accessing-homestead-globally"></a>
### Глобальный доступ к Homestead

Иногда вам может понадобиться выполнить `vagrant up` yвашей Homestead-машины из любого места вашей файловой системы. На системах Mac / Linux это можно сделать добавив Bash-функцию в ваш Bash-профиль. На Windows можно добавить "пакетный" файл к своему `PATH`. Эти скрипты позволят вам выполнять любые команды Vagrant из любого места вашей системы, и автоматически укажет команде на ваш установленный Homestead:

#### Mac / Linux

    function homestead() {
        ( cd ~/Homestead && vagrant $* )
    }

Не забудьте исправить путь `~/Homestead` в функции на реальное расположение вашего установленного Homestead. Когда функция установлена, вы можете выполнять такие команды, как `homestead up` или `homestead ssh` из любого места вашей системы.

#### Windows

Создайте `homestead.bat` пакетный файл из любого места на вашей машине, использовав следующее содержимое:

    @echo off

    set cwd=%cd%
    set homesteadVagrant=C:\Homestead

    cd /d %homesteadVagrant% && vagrant %*
    cd /d %cwd%

    set cwd=
    set homesteadVagrant=

Не забудьте исправить путь `C:\Homestead` в скрипте на реальное расположение вашего установленного Homestead.. После того как вы создали файл, добавьте местоположение своего файла в свой `PATH`. Вы можете выполнять такие команды, как `homestead up` или `homestead ssh` из любого места своей системы.

<a name="connecting-via-ssh"></a>
### Подключение через SSH

Для подключения к своей виртуальной машине по SSH вы можете использовать команду `vagrant ssh` из своей Homestead-директории.

Но поскольку вам, скорее всего, потребуется часто подключаться к вашей Homestead-машине по SSH, будет удобно создать "функцию" на вашей хост-машине для быстрого подключения, как описано выше.

<a name="connecting-to-databases"></a>
### Подключение к базам данных

База `homestead` изначально настроена на использование и MySQL, и Postgres. Для ещё большего удобства файл Laravel `.env` настраивает фреймворк на использование этой БД по умолчанию.

Чтобы подключиться к вашей базе данных MySQL или Postgres через клиент БД с вашей хост-машины, вы должны подключиться к `127.0.0.1` через порт `33060` (MySQL) или `54320` (Postgres). Имя пользователя и пароль для обеих баз данных `homestead` / `secret`.

> {note} Вы должны использовать эти нестандартные порты только подключаясь к базам данных с вашей главной машины. Вы будете использовать порты 3306 и 5432 в вашем конфигурационном файле базы данных Laravel, так как Laravel запущен _на_ виртуальной машине.

<a name="adding-additional-sites"></a>
### Добавление дополнительных сайтов

После настройки и запуска вашей среды Homestead вы можете захотеть добавить дополнительные Nginx-сайты для своих Laravel-приложений. Вы можете запустить в одной среде Homestead столько установок Laravel, сколько захотите. Вы можете просто добавить сайты в свой файл `Homestead.yaml`:

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
        - map: another.app
          to: /home/vagrant/Code/another/public

Если Vagrant не управляет вашим файлом "hosts" автоматически, вам может потребоваться также добавить к тому файлу новый сайт:

    192.168.10.10  homestead.app
    192.168.10.10  another.app

После добавления сайта выполните команду `vagrant reload --provision` из своей директории Homestead.

<a name="site-types"></a>
#### Типы сайтов

Homestead поддерживает несколько типов файлов, что позволяет вам запросто работать с проектами, которые не работаю на Laravel. Например, мы можем запросто добавить Symfony-приложение в Homestead, используя тип сайта `symfony2`:

    sites:
        - map: symfony2.app
          to: /home/vagrant/Code/Symfony/public
          type: symfony2

Доступные типы сайтов: `apache`, `laravel` (по умолчанию), `proxy`, `silverstripe`, `statamic`, `symfony2` и `symfony4`.

<a name="site-parameters"></a>
#### Параметры сайтов

Вы можете добавить дополнительные Nginx-значения `fastcgi_param` вашему сайте через директиву `params`. Например, мы добавим параметр `FOO` со значением `BAR`:

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          params:
              - key: FOO
                value: BAR

<a name="configuring-cron-schedules"></a>
### Настройка расписания Cron

Laravel предоставляет удобный способ для [планирования Cron-задач](/docs/{{version}}/scheduling) путём планирования единственной Artisan-команды `schedule:run` на ежеминутное выполнение. Команда `schedule:run` проверит запланированные задачи, определённые в классе `App\Console\Kernel`, и определит, какие задачи необходимо выполнить.

Если вы хотите выполнить команду `schedule:run` для сайта Homestead, вы можете задать значение `true` для параметра `schedule` при определении сайта:

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          schedule: true

Cron-задача для сайта будет определена в папке `/etc/cron.d` на виртуальной машине.

<a name="ports"></a>
### Порты

По умолчанию следующие порты переадресованы в вашу среду Homestead:

- **SSH:** 2222 &rarr; переадресован в 22
- **HTTP:** 8000 &rarr; переадресован в 80
- **HTTPS:** 44300 &rarr; переадресован в 443
- **MySQL:** 33060 &rarr; переадресован в 3306
- **Postgres:** 54320 &rarr; переадресован в 5432
- **Mailhog:** 8025 &rarr; переадресован в 8025

#### Перенаправление дополнительных портов

По желанию можно переадресовать дополнительные порты в Vagrant-бокс, а также указать их протокол:

    ports:
        - send: 50000
          to: 5000
        - send: 7777
          to: 777
          protocol: udp

<a name="sharing-your-environment"></a>
### Совместное использование вашей среды

Иногда вам может потребоваться поделиться тем, над чем вы работаете в данный момент, с другими разработчиками или клиентом. В Vagrant есть встроенный способ поддержки данного функционала через `vagrant share`; однако, это не будет работать, если в вашем файле `Homestead.yaml` настроено несколько сайтов.

Чтобы решить это проблему у Homestead есть собственная команда `share`. Для начала подключитесь к своей Homested-машине через `vagrant ssh` и выполните `share homestead.app`. Это позволит поделиться сайтом `homestead.app` из вашего конфига `Homestead.yaml`. Конечно же, вы можете заменить любой из других своих настроенных сайтов на `homestead.app`:

    share homestead.app

После запуска команды вы увидите, как появится экран Ngrok, на котором содержится журнал активности и  общедоступные URL для сайта, которым вы поделились. Если вы хотите указать пользовательскую область, поддомен или другую настройку Ngrok, можно добавить их к своей команде `share`:

    share homestead.app -region=eu -subdomain=laravel

> {note} Помните, что Vagrant от природы небезопасен и вы выставляете свою виртуальную машину всему Интернет-пространству во время запуска команды `share`.

<a name="multiple-php-versions"></a>
### Несколько версий PHP

> {note} Совместимо только с Nginx.

Homestead 6 представил поддержку нескольких версий PHP на одной и той же виртуальной машине. Вы можете указать какие версии PHP использовать для заданного сайта в своем файле `Homestead.yaml`. Доступные версии PHP: "5.6", "7.0" и "7.1":

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          php: "5.6"

Дополнительно вы можете пользоваться любой из поддерживаемых версий PHP через CLI:

    php5.6 artisan list
    php7.0 artisan list
    php7.1 artisan list

<a name="network-interfaces"></a>
## Сетевые интерфейсы
Свойство `networks` файла `Homestead.yaml` настраивает сетевые интерфейсы вашей среды Homestead. Вы можете настроить столько интерфейсов, сколько потребуется:

    networks:
        - type: "private_network"
          ip: "192.168.10.20"

Чтобы включить интерфейс с [мостовым соединением](https://www.vagrantup.com/docs/networking/public_network.html), измените настройку `bridge` и измените тип сети на `public_network`:

    networks:
        - type: "public_network"
          ip: "192.168.10.20"
          bridge: "en1: Wi-Fi (AirPort)"

Чтобы включить [DHCP](https://www.vagrantup.com/docs/networking/public_network.html), просто уберите опцию `ip` из своей настройки:

    networks:
        - type: "public_network"
          bridge: "en1: Wi-Fi (AirPort)"

<a name="updating-homestead"></a>
## Обновление Homestead

Для обновления Homestead надо выполнить два простых шага. Во-первых, вам надо обновить Vagrant-бокс с помощью команды `vagrant box update`:

    vagrant box update

Затем вам надо обновить исходный код Homestead. Если вы клонировали репозиторий, то можете просто выполнить `git pull origin master` в то место, куда вы клонировали репозиторий изначально.

Если вы установили Homestead через файл `composer.json`, то должны убедиться, что файл `composer.json` содержит строку `"laravel/homestead": "^4"` и обновить ваши зависимости:

    composer update

<a name="old-versions"></a>
## Старые версии

> {tip} Если вам нужна более старая версия PHP, посмотрите документацию по <a href="#multiple-php-versions">нескольким PHP-версиям</a> прежде чем начнете использовать старую версию Homestead. 

Вы можете легко изменить используемую в Homestead версию бокса, добавив следующую строку в ваш файл `Homestead.yaml`:

    version: 0.6.0

Пример:

    box: laravel/homestead
    version: 0.6.0
    ip: "192.168.20.20"
    memory: 2048
    cpus: 4
    provider: virtualbox

При использовании старых версий бокса Homestead вам надо проверить версию на совместимость с исходным кодом Homestead. Ниже приведена таблица поддерживаемых версий бокса с указанием того, какую версию исходного кода необходимо использовать, и какая в коробке версия PHP:

|   | Версия Homestead | Версия бокса |
|---|---|---|
| PHP 7.0 | 3.1.0 | 0.6.0 |
| PHP 7.1 | 4.0.0 | 1.0.0 |

<a name="provider-specific-settings"></a>
## Специальные настройки провайдера

<a name="provider-specific-virtualbox"></a>
### VirtualBox

По умолчанию Homestead настраивает `natdnshostresolver` как `on`. Это позволяет Homestead использовать настройки DNS вашей операционной системы хоста. Если вы хотите переопределить такое поведение, добавьте следующие строки к своему файлу `Homestead.yaml`:

    provider: virtualbox
    natdnshostresolver: off
