git 0f3d5bf05411f37a18087497e89e02555ccda42e

---

# Laravel Sail

- [Введение](#introduction)
- [Установка и настройка](#installation)
    - [Установка Sail в существующее приложение](#installing-sail-into-existing-applications)
    - [Настройка Bash псевдонимов](#configuring-a-bash-alias)
- [Запуск и остановка Sail](#starting-and-stopping-sail)
- [Выполнение команд](#executing-sail-commands)
    - [Выполнение PHP команд](#executing-php-commands)
    - [Выполнение Composer команд](#executing-composer-commands)
    - [Выполнение Artisan команд](#executing-artisan-commands)
    - [Выполнение Node/NPM команд](#executing-node-npm-commands)
- [Взаимодействие с базами данных](#interacting-with-sail-databases)
    - [MySQL](#mysql)
    - [Redis](#redis)
    - [MeiliSearch](#meilisearch)
- [Файловое хранилище](#file-storage)
- [Тестирование](#running-tests)
    - [Laravel Dusk](#laravel-dusk)
- [Предпросмотр писем](#previewing-emails)
- [Контейнер CLI](#sail-container-cli)
- [Версии PHP](#sail-php-versions)
- [Предоставление доступа к сайту](#sharing-your-site)
- [Настройка](#sail-customization)

<a name="introduction"></a>
## Введение

Laravel Sail - это легкий интерфейс командной строки для взаимодействия со средой разработки Docker по умолчанию для Laravel. Sail обеспечивает отличную отправную точку для создания приложения Laravel с использованием PHP, MySQL и Redis без предварительного опыта работы с Docker.

По сути, Sail - это файл `docker-compose.yml` и сценарий `sail`, который хранится в корне вашего проекта. Скрипт `sail` предоставляет CLI с удобными методами для взаимодействия с контейнерами Docker, определенными файлом` docker-compose.yml`.

Laravel Sail поддерживается в macOS, Linux и Windows (через WSL2).

<a name="installation"></a>
## Установка и настройка

Laravel Sail автоматически устанавливается со всеми новыми приложениями Laravel, поэтому вы можете сразу же начать его использовать. Чтобы узнать, как создать новое приложение Laravel, обратитесь к [документации по установке](/docs/{{version}}/installation) Laravel для вашей операционной системы. Во время установки вам будет предложено выбрать, с какими службами, поддерживаемыми Sail, ваше приложение будет взаимодействовать.

<a name="installing-sail-into-existing-applications"></a>
### Установка Sail в существующее приложение

Если вы заинтересованы в использовании Sail с существующим приложением Laravel, вы можете просто установить Sail с помощью диспетчера пакетов Composer. Конечно, эти шаги предполагают, что ваша существующая локальная среда разработки позволяет вам устанавливать зависимости Composer:

    composer require laravel/sail --dev

После установки Sail вы можете запустить Artisan-команду `sail: install`. Эта команда опубликует файл Sail `docker-compose.yml` в корень вашего приложения:

    php artisan sail:install

Наконец, вы можете запустить Sail. Чтобы продолжить изучение использования Sail, продолжайте читать оставшуюся часть этой документации:

    ./vendor/bin/sail up

<a name="configuring-a-bash-alias"></a>
### Настройка Bash псевдонимов

По умолчанию команды Sail вызываются с помощью сценария `vendor/bin/sail`, который включен во все новые приложения Laravel:

```bash
./vendor/bin/sail up
```

Однако вместо того, чтобы многократно вводить `vendor/bin/sail` для выполнения команд Sail, вы можете захотеть настроить псевдоним Bash, который позволит вам более легко выполнять команды Sail:

```bash
alias sail='bash vendor/bin/sail'
```

После настройки псевдонима Bash вы можете выполнять команды Sail, просто набрав `sail`. В остальных примерах из этой документации предполагается, что вы настроили этот псевдоним:

```bash
sail up
```

<a name="starting-and-stopping-sail"></a>
## Запуск и остановка Sail

Файл `docker-compose.yml` Laravel Sail определяет различные контейнеры Docker, которые работают вместе, чтобы помочь вам создавать приложения Laravel. Каждый из этих контейнеров является записью в конфигурации `services` вашего файла`docker-compose.yml`. Контейнер `laravel.test` - это основной контейнер приложения, который будет обслуживать ваше приложение.

Перед запуском Sail убедитесь, что на вашем локальном компьютере не работают другие веб-серверы или базы данных. Чтобы запустить все контейнеры Docker, определенные в файле `docker-compose.yml` вашего приложения, вы должны выполнить команду `up`:

```bash
sail up
```

Чтобы запустить все контейнеры Docker в фоновом режиме, вы можете запустить Sail в "detached" режиме:

```bash
sail up -d
```

После запуска контейнеров приложения вы можете получить доступ к проекту в своем веб-браузере по адресу: http://localhost.

Чтобы остановить все контейнеры, вы можете просто нажать Control + C, чтобы остановить выполнение контейнера. Или, если контейнеры работают в фоновом режиме, вы можете использовать команду `down`:

```bash
sail down
```

<a name="executing-sail-commands"></a>
## Выполнение команд

При использовании Laravel Sail ваше приложение выполняется в контейнере Docker и изолировано от вашего локального компьютера. Однако Sail предоставляет удобный способ запускать различные команды для вашего приложения, такие как произвольные команды PHP, команды Artisan, команды Composer и Node/NPM команды.

**При чтении документации Laravel вы будете часто видеть команды Composer, Artisan и Node/NPM, в которых не упоминается Sail.** В этих примерах предполагается, что эти инструменты установлены на вашем компьютере. Если вы используете Sail для своей локальной среды разработки Laravel, вам следует выполнить эти команды с помощью Sail:

```bash
# Локальное выполнение команд Artisan ...
php artisan queue:work

# Выполнение команд Artisan в Laravel Sail ...
sail artisan queue:work
```

<a name="executing-php-commands"></a>
### Выполнение PHP команд

Команды PHP могут быть выполнены с помощью команды `php`. Конечно, эти команды будут выполняться с использованием версии PHP, настроенной для вашего приложения. Чтобы узнать больше о версиях PHP, доступных для Laravel Sail, обратитесь к [документации версии PHP](#sail-php-versions):

```bash
sail php --version

sail php script.php
```

<a name="executing-composer-commands"></a>
### Выполнение Composer команд

Команды Composer могут быть выполнены с помощью команды `composer`. Контейнер приложения Laravel Sail включает установку Composer 2.x:

```nothing
sail composer require laravel/sanctum
```

<a name="installing-composer-dependencies-for-existing-projects"></a>
#### Установка зависимостей Composer для существующих приложений

Если вы разрабатываете приложение в команде, возможно, вы не тот, кто создал приложение Laravel с нуля. Следовательно, ни одна из зависимостей Composer, включая Sail, не будет установлена после клонирования репозитория приложения на локальный компьютер.

Вы можете установить зависимости приложения, перейдя в каталог приложения и выполнив следующую команду. Эта команда использует небольшой контейнер Docker, содержащий PHP и Composer, для установки зависимостей приложения:

```nothing
docker run --rm \
    -u "$(id -u):$(id -g)" \
    -v $(pwd):/opt \
    -w /opt \
    laravelsail/php80-composer:latest \
    composer install --ignore-platform-reqs
```

<a name="executing-artisan-commands"></a>
### Выполнение Artisan команд

Команды Laravel Artisan могут быть выполнены с помощью команды `artisan`:

```bash
sail artisan queue:work
```

<a name="executing-node-npm-commands"></a>
### Выполнение Node/NPM команд

Команды Node могут выполняться с помощью команды `node`, а команды NPM выполняются с помощью команды `npm`:

```nothing
sail node --version

sail npm run prod
```

<a name="interacting-with-sail-databases"></a>
## Взаимодействие с базами данных

<a name="mysql"></a>
### MySQL

Как вы могли заметить, файл `docker-compose.yml` вашего приложения содержит запись для контейнера MySQL. Этот контейнер использует [том Docker](https://docs.docker.com/storage/volumes/), чтобы данные, хранящиеся в вашей базе данных, сохранялись даже при остановке и перезапуске ваших контейнеров. Вдобавок, когда контейнер MySQL запускается, он гарантирует, что существует база данных, имя которой совпадает со значением вашей переменной окружения `DB_DATABASE`.

После того как вы запустили свои контейнеры, вы можете подключиться к экземпляру MySQL в вашем приложении, установив для переменной среды `DB_HOST` в файле вашего приложения `.env` значение `mysql`.

Чтобы подключиться к базе данных MySQL вашего приложения с вашего локального компьютера, вы можете использовать графическое приложение для управления базой данных, такое как [TablePlus](https://tableplus.com). По умолчанию база данных MySQL доступна через порт 3306 `localhost`.

<a name="redis"></a>
### Redis

Файл `docker-compose.yml` вашего приложения также содержит запись для контейнера [Redis](https://redis.io). Этот контейнер использует [том Docker](https://docs.docker.com/storage/volumes/), чтобы данные, хранящиеся в ваших данных Redis, сохранялись даже при остановке и перезапуске ваших контейнеров. После того как вы запустили свои контейнеры, вы можете подключиться к экземпляру Redis в своем приложении, установив для переменной среды `REDIS_HOST` в файле` .env` вашего приложения значение `redis`.

Чтобы подключиться к базе данных Redis вашего приложения с локального компьютера, вы можете использовать графическое приложение для управления базой данных, такое как [TablePlus](https://tableplus.com). По умолчанию база данных Redis доступна на порту 6379 `localhost`.

<a name="meilisearch"></a>
### MeiliSearch

Если вы выбрали установку службы [MeiliSearch](https://www.meilisearch.com) при установке Sail, файл `docker-compose.yml` вашего приложения будет содержать запись для этой мощной поисковой системы, которая [совместима](https://github.com/meilisearch/meilisearch-laravel-scout) с помощью [Laravel Scout](/docs/{{version}}/scout). После того как вы запустили свои контейнеры, вы можете подключиться к экземпляру MeiliSearch в своем приложении, установив для переменной среды `MEILISEARCH_HOST` значение `http://meilisearch:7700`.

Со своего локального компьютера вы можете получить доступ к веб-панели администрирования MeiliSearch, перейдя по адресу `http://localhost:7700` в своем браузере.

<a name="file-storage"></a>
## Файловое хранилище

Если вы планируете использовать Amazon S3 для хранения файлов при запуске приложения в производственной среде, вы можете установить службу [MinIO](https://min.io) при установке Sail. MinIO предоставляет совместимый с S3 API, который вы можете использовать для локальной разработки с помощью драйвера хранилища файлов Laravel s3, не создавая «тестовых» сегментов хранилища в производственной среде S3. Если вы выберете установку MinIO при установке Sail, раздел конфигурации MinIO будет добавлен в файл `docker-compose.yml` вашего приложения.

По умолчанию файл конфигурации приложения `filesystems` уже содержит конфигурацию диска для диска `s3`. Помимо использования этого диска для взаимодействия с Amazon S3, вы можете использовать его для взаимодействия с любой S3-совместимой службой хранения файлов, такой как MinIO, путем простого изменения связанных переменных среды, которые управляют его конфигурацией. Например, при использовании MinIO конфигурация переменной среды вашей файловой системы должна быть определена следующим образом:

```ini
FILESYSTEM_DRIVER=s3
AWS_ACCESS_KEY_ID=sail
AWS_SECRET_ACCESS_KEY=password
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=local
AWS_ENDPOINT=http://minio:9000
AWS_USE_PATH_STYLE_ENDPOINT=true
```

<a name="running-tests"></a>
## Тестирование

Laravel обеспечивает отличную поддержку тестирования прямо из коробки, и вы можете использовать команду Sail `test` для запуска своих приложений [функциональные и модульные тесты](/docs/{{версия}}/testing). Любые параметры интерфейса командной строки, которые принимает PHPUnit, также могут быть переданы команде `test`:

    sail test

    sail test --group orders

Команда Sail `test` эквивалентна запуску Artisan-команды` test`:

    sail artisan test

<a name="laravel-dusk"></a>
### Laravel Dusk

[Laravel Dusk](/docs/{{version}}/dusk) предоставляет выразительный, простой в использовании API для автоматизации и тестирования браузера. Благодаря Sail вы можете запускать эти тесты, даже не устанавливая Selenium или другие инструменты на свой локальный компьютер. Для начала раскомментируйте службу Selenium в файле `docker-compose.yml` вашего приложения:

```yaml
selenium:
    image: 'selenium/standalone-chrome'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

Затем убедитесь, что служба `laravel.test` в файле `docker-compose.yml` вашего приложения имеет запись `depends_on` для `selenium`:

```yaml
depends_on:
    - mysql
    - redis
    - selenium
```

Наконец, вы можете запустить свой набор тестов Dusk, запустив Sail и выполнив команду `dusk`:

    sail dusk

<a name="previewing-emails"></a>
## Предпросмотр писем

Файл `docker-compose.yml` в Laravel Sail по умолчанию содержит служебную запись для [MailHog](https://github.com/mailhog/MailHog). MailHog перехватывает электронные письма, отправленные вашим приложением во время локальной разработки, и предоставляет удобный веб-интерфейс, чтобы вы могли предварительно просмотреть свои электронные сообщения в браузере. При использовании Sail хостом MailHog по умолчанию является `mailhog` и он доступен через порт 1025:

```bash
MAIL_HOST=mailhog
MAIL_PORT=1025
```

Когда Sail запущен, вы можете получить доступ к веб-интерфейсу MailHog по адресу: http://localhost: 8025

<a name="sail-container-cli"></a>
## Контейнер CLI

Иногда вы можете захотеть запустить сеанс Bash в контейнере вашего приложения. Вы можете использовать команду `shell` для подключения к контейнеру приложения, что позволит вам проверять его файлы и установленные службы, а также выполнять произвольные команды оболочки внутри контейнера:

```nothing
sail shell

sail root-shell
```

Чтобы запустить новый сеанс [Laravel Tinker](https://github.com/laravel/tinker), вы можете выполнить команду `tinker`:

```bash
sail tinker
```

<a name="sail-php-versions"></a>
## Версии PHP

В настоящее время Sail поддерживает обслуживание вашего приложения через PHP 8.0 или PHP 7.4. Чтобы изменить версию PHP, которая используется для обслуживания вашего приложения, вы должны обновить определение `build` контейнера` laravel.test` в файле `docker-compose.yml` вашего приложения:

```yaml
# PHP 8.0
context: ./vendor/laravel/sail/runtimes/8.0

# PHP 7.4
context: ./vendor/laravel/sail/runtimes/7.4
```

Кроме того, вы можете захотеть обновить свое имя `image`, чтобы оно отражало версию PHP, используемую приложением. Этот параметр также определен в файле `docker-compose.yml` приложения:

```yaml
image: sail-8.0/app
```

После обновления файла `docker-compose.yml` вашего приложения вы должны обновить образы контейнеров:

    sail build --no-cache

    sail up

<a name="sharing-your-site"></a>
## Предоставление доступа к сайту

Иногда может потребоваться предоставить общий доступ к своему сайту, например чтобы его посмотрели коллеги или протестировать интеграцию веб-перехватчика с вашим приложением. Чтобы поделиться своим сайтом, вы можете использовать команду `share`. После выполнения этой команды вам будет выдан случайный URL-адрес `laravel-sail.site`, который вы можете использовать для доступа к своему приложению:

    sail share

При совместном использовании сайта с помощью команды `share` вы должны настроить доверенные прокси вашего приложения в промежуточном программном обеспечении `TrustProxies`. В противном случае вспомогательные средства генерации URL, такие как `url` и `route`, не смогут определить правильный HTTP-хост, который следует использовать во время генерации URL:

    /**
     * Надежные прокси для приложения.
     *
     * @var array|string|null
     */
    protected $proxies = '*';

Если вы хотите выбрать поддомен для вашего общего сайта, вы можете указать параметр `subdomain` при выполнении команды `share`:

    sail share --subdomain=my-sail-site

> {tip} Команда `share` поддерживается [Expose](https://github.com/beyondcode/expose), службой туннелирования с открытым исходным кодом от [BeyondCode](https://beyondco.de).

<a name="sail-customization"></a>
## Настройка

Поскольку Sail - это просто Docker-контейнер, вы можете настроить в нём почти всё. Чтобы опубликовать собственные Docker-файлы Sail, вы можете выполнить команду `sail:publish`:

```bash
sail artisan sail:publish
```

После выполнения этой команды файлы Dockerfiles и другие файлы конфигурации, используемые Laravel Sail, будут помещены в каталог `docker` в корневом каталоге вашего приложения. После настройки вашей установки Sail вы можете перестроить контейнеры приложения с помощью команды `build`:

```bash
sail build --no-cache
```
