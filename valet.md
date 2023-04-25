git 76d29dc81cf208f72b193e8b6f41a9051ea616e4

---

# Laravel Valet

- [Вступление](#introduction)
- [Установка](#installation)
    - [Обновление Valet](#upgrading-valet)
- [Обслуживание сайтов](#serving-sites)
    - [Команда "Park"](#the-park-command)
    - [Команда "Link"](#the-link-command)
    - [Защита сайтов с помощью TLS](#securing-sites)
    - [Обслуживание сайта по умолчанию](#serving-a-default-site)
- [Сайты общего доступа](#sharing-sites)
    - [Общий доступ к сайтам через Ngrok](#sharing-sites-via-ngrok)
    - [Общий доступ к сайтам через Expose](#sharing-sites-via-expose)
    - [Общий доступ к сайтам в вашей локальной сети](#sharing-sites-on-your-local-network)
- [Переменные среды, зависящие от конкретного сайта](#site-specific-environment-variables)
- [Прокси-сервисы](#proxying-services)
- [Пользовательские Valet драйверы](#custom-valet-drivers)
    - [Локальные драйверы](#local-drivers)
- [Другие команды Valet](#other-valet-commands)
- [Директории и файлы Valet](#valet-directories-and-files)

<a name="introduction"></a>
## Вступление

[Laravel Valet](https://github.com/laravel/valet) - это среда разработки для минималистов macOS. Laravel Valet настраивает ваш Mac на постоянный запуск [Nginx](https://www.nginx.com/) в фоновом режиме при запуске вашего компьютера. Затем, используя [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq), Valet проксирует все запросы в домене `*.test`, чтобы указать на сайты, установленные на вашем локальном компьютере.

Другими словами, Valet - это невероятно быстрая среда разработки Laravel, которая использует примерно 7 МБ оперативной памяти. Valet не является полной заменой [Sail](/docs/{{version}}/sail) или [Homestead](/docs/{{version}}/homestead), но предоставляет отличную альтернативу, если вам нужны гибкие основы, вы предпочитаете экстремальную скорость или работаете на компьютере с ограниченным объемом оперативной памяти.

Из коробки, поддержка Valet включает, но не ограничивается этим:

<!-- <style>
    #valet-support > ul {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        line-height: 1.9;
    }
</style>

<div id="valet-support" markdown="1"> -->

- [Laravel](https://laravel.com)
- [Lumen](https://lumen.laravel.com)
- [Bedrock](https://roots.io/bedrock/)
- [CakePHP 3](https://cakephp.org)
- [Concrete5](https://www.concrete5.org/)
- [Contao](https://contao.org/en/)
- [Craft](https://craftcms.com)
- [Drupal](https://www.drupal.org/)
- [ExpressionEngine](https://www.expressionengine.com/)
- [Jigsaw](https://jigsaw.tighten.co)
- [Joomla](https://www.joomla.org/)
- [Katana](https://github.com/themsaid/katana)
- [Kirby](https://getkirby.com/)
- [Magento](https://magento.com/)
- [OctoberCMS](https://octobercms.com/)
- [Sculpin](https://sculpin.io/)
- [Slim](https://www.slimframework.com)
- [Statamic](https://statamic.com)
- Static HTML
- [Symfony](https://symfony.com)
- [WordPress](https://wordpress.org)
- [Zend](https://framework.zend.com)

<!-- </div> -->

Однако вы можете расширить Valet с помощью ваших собственных [пользовательских драйверов](#custom-valet-drivers).

<a name="installation"></a>
## Установка

> {note} для Valet требуется macOS и [Homebrew](https://brew.sh/). Перед установкой вы должны убедиться, что никакие другие программы, такие как Apache или Nginx, не привязаны к порту 80 вашего локального компьютера.

Чтобы начать, сначала вам нужно убедиться, что Homebrew обновлен с помощью команды `update`:

    brew update

Далее вы должны использовать Homebrew для установки PHP:

    brew install php

После установки PHP вы готовы к установке [менеджера пакетов Composer](https://getcomposer.org). Кроме того, вы должны убедиться, что каталог `~/.composer/vendor/bin` находится в "PATH" вашей системы. После установки Composer вы можете установить Laravel Valet в качестве глобального пакета Composer:

    composer global require laravel/valet

Наконец, вы можете выполнить команду Valet `install`. Это позволит настроить и установить Valet и DnsMasq. Кроме того, демоны, от которых зависит Valet, будут настроены для запуска при запуске вашей системы:

    valet install

Как только Valet будет установлен, попробуйте выполнить пинг любого домена `*.test` на вашем терминале, используя такую команду, как `ping foobar.test`. Если Valet установлен правильно, вы должны увидеть, что этот домен отвечает на `127.0.0.1`.

Valet автоматически запускает необходимые службы при каждой загрузке вашего устройства.

<a name="php-versions"></a>
#### Версии PHP

Valet позволяет вам переключать версии PHP с помощью команды `valet use php@version`. Valet установит указанную версию PHP через Homebrew, если она еще не установлена:

    valet use php@7.2

    valet use php

Вы также можете создать файл `.valetphprc` в корневом каталоге вашего проекта. Файл `.valetphprc` должен содержать версию PHP, которую должен использовать сайт:

    php@7.2

Как только этот файл будет создан, вы можете просто выполнить команду `valet use`, и команда определит предпочтительную версию PHP сайта, прочитав файл.

> {note} Valet обслуживает только одну версию PHP одновременно, даже если у вас установлено несколько версий PHP.

<a name="database"></a>
#### База данных

Если вашему приложению нужна база данных, ознакомьтесь с [DBngin](https://dbngin.com). DBngin предоставляет бесплатный универсальный инструмент управления базами данных, который включает MySQL, PostgreSQL и Redis. После установки DBngin вы можете подключиться к своей базе данных по адресу `127.0.0.1`, используя имя пользователя `root` и пустую строку для пароля.

<a name="resetting-your-installation"></a>
#### Сброс вашей установки

Если у вас возникли проблемы с корректным запуском установки Valet, то выполнение команды `composer global update`, за которой следует `valet install`, приведет к сбросу вашей установки и может решить множество проблем. В редких случаях может потребоваться "жесткий сброс" Valet, выполнив команду `valet uninstall --force`, за которой следует `valet install`.

<a name="upgrading-valet"></a>
### Обновление Valet

Вы можете обновить свою установку Valet, выполнив команду `composer global update` в вашем терминале. После обновления рекомендуется выполнить команду `valet install`, чтобы Valet мог при необходимости внести дополнительные обновления в ваши конфигурационные файлы.

<a name="serving-sites"></a>
## Обслуживание сайтов

Как только Valet будет установлен, вы будете готовы начать обслуживать свои приложения Laravel. Valet предоставляет две команды, которые помогут вам обслуживать ваши приложения: `park` и `link`.

<a name="the-park-command"></a>
### Команда `park`

Команда `park` регистрирует каталог на вашем компьютере, содержащий ваши приложения. Как только каталог будет "припаркован" с помощью Valet, все каталоги в этом каталоге будут доступны в вашем веб-браузере по адресу `http://<имя каталога>.test`:

    cd ~/Sites

    valet park

Вот и все, что нужно сделать. Теперь любое приложение, которое вы создадите в своем "припаркованном" каталоге, будет автоматически обслуживаться с использованием соглашения `http://<имя каталога>.test`. Итак, если ваш каталог пакетов содержит каталог с именем "laravel", приложение в этом каталоге будет доступно по адресу `http://laravel.test`. Кроме того, Valet автоматически позволяет вам получить доступ к сайту, используя поддомены с подстановочными знаками (`http://foo.laravel.test`).

<a name="the-link-command"></a>
### Команда `link`

Команда `link` также может быть использована для обслуживания ваших приложений Laravel. Эта команда полезна, если вы хотите обслуживать один сайт в каталоге, а не весь каталог целиком:

    cd ~/Sites/laravel

    valet link

После того, как приложение было связано с Valet с помощью команды `link`, вы можете получить доступ к приложению, используя его имя каталога. Итак, доступ к сайту, на который была дана ссылка в приведенном выше примере, можно получить по адресу `http://laravel.test`. Кроме того, Valet автоматически позволяет вам получить доступ к сайту, используя поддомены с подстановочными знаками (`http://foo.laravel.test`).

Если вы хотите обслуживать приложение под другим именем хоста, вы можете передать имя хоста команде `link`. Например, вы можете выполнить следующую команду, чтобы сделать приложение доступным по адресу `http://application.test`:

    cd ~/Sites/laravel

    valet link application

Вы можете выполнить команду `links`, чтобы отобразить список всех ваших связанных каталогов:

    valet links

Команда `unlink` может быть использована для уничтожения символической ссылки на сайт:

    cd ~/Sites/laravel

    valet unlink

<a name="securing-sites"></a>
### Защита сайтов с помощью TLS

По умолчанию Valet обслуживает сайты по протоколу HTTP. Однако, если вы хотите обслуживать сайт по зашифрованному протоколу TLS с использованием HTTP/2, вы можете использовать команду `secure`. Например, если ваш сайт обслуживается Valet в домене `laravel.test`, вам следует выполнить следующую команду, чтобы обеспечить его безопасность:

    valet secure laravel

Чтобы "снять защиту" с сайта и вернуться к обслуживанию его трафика по обычному протоколу HTTP, используйте команду `unsecure`. Как и команда `secure`, эта команда принимает имя хоста, с которого вы хотите снять защиту:

    valet unsecure laravel

<a name="serving-a-default-site"></a>
### Обслуживание сайта по умолчанию

Иногда вы можете захотеть настроить Valet для обслуживания сайта "по умолчанию" вместо `404` при посещении неизвестного домена `test`. Чтобы выполнить это, вы можете добавить параметр `default` в свой конфигурационный файл `~/.config/valet/config.json`, содержащий путь к сайту, который должен использоваться в качестве вашего сайта по умолчанию:

    "default": "/Users/Sally/Sites/foo",

<a name="sharing-sites"></a>
## Сайты общего доступа

Valet даже включает в себя команду для обмена вашими локальными сайтами со всем миром, предоставляя простой способ протестировать ваш сайт на мобильных устройствах или поделиться им с членами команды и клиентами.

<a name="sharing-sites-via-ngrok"></a>
### Общий доступ к сайтам через Ngrok

Чтобы предоставить общий доступ к сайту, перейдите в каталог сайта в вашем терминале и запустите команду Valet `share`. Общедоступный URL-адрес будет вставлен в ваш буфер обмена и готов для вставки непосредственно в ваш браузер или обмена с вашей командой:

    cd ~/Sites/laravel

    valet share

Чтобы прекратить совместное использование вашего сайта, вы можете нажать `Control + C`. Для предоставления доступа к вашему сайту с помощью Ngrok вам необходимо [создать учетную запись Ngrok](https://dashboard.ngrok.com/signup) и [настройка токена аутентификации](https://dashboard.ngrok.com/get-started/your-authtoken).

> {tip} Вы можете передать дополнительные параметры Ngrok команде `share`, такие как `valet share --region=eu`. Для получения дополнительной информации обратитесь к документации [ngrok](https://ngrok.com/docs).

<a name="sharing-sites-via-expose"></a>
### Общий доступ к сайтам через Expose

Если у вас установлен [Expose](https://expose.dev), вы можете предоставить общий доступ к своему сайту, перейдя в каталог сайта в вашем терминале и выполнив команду `expose`. Обратитесь к [документации Expose](https://expose.dev/docs) для получения информации о дополнительных параметрах командной строки, которые он поддерживает. После предоставления общего доступа к сайту Expose отобразит общедоступный URL-адрес, который вы можете использовать на других своих устройствах или среди членов команды:

    cd ~/Sites/laravel

    expose

Чтобы прекратить совместное использование вашего сайта, вы можете нажать `Control + C`.

<a name="sharing-sites-on-your-local-network"></a>
### Общий доступ к сайтам в вашей локальной сети

Valet по умолчанию ограничивает входящий трафик внутренним интерфейсом `127.0.0.1`, чтобы ваша машина разработки не подвергалась угрозам безопасности из Интернета.

Если вы хотите разрешить другим устройствам в вашей локальной сети получать доступ к сайтам Valet на вашем компьютере через IP-адрес вашего компьютера (например: `192.168.1.10/application.test`), вам нужно будет вручную отредактировать соответствующий файл конфигурации Nginx для этого сайта, чтобы снять ограничение на директиву `listen`. Вам следует удалить префикс `127.0.0.1:` в директиве `listen` для портов 80 и 443.

Если вы не запустили `valet secure` в проекте, вы можете открыть доступ к сети для всех сайтов, отличных от HTTPS, отредактировав файл `/usr/local/etc/nginx/valet/valet.conf`. Однако, если вы обслуживаете сайт проекта по протоколу HTTPS (вы запустили `valet secure` для сайта), то вам следует отредактировать файл `~/.config/valet/Nginx/app-name.test`.

Как только вы обновите конфигурацию Nginx, запустите команду `valet restart`, чтобы применить изменения конфигурации.

<a name="site-specific-environment-variables"></a>
## Переменные среды, зависящие от конкретного сайта

Некоторые приложения, использующие другие фреймворки, могут зависеть от переменных окружения сервера, но не предоставляют способа настройки этих переменных в вашем проекте. Valet позволяет вам настраивать переменные среды для конкретного сайта, добавляя `.valet-env.php` файл в корневом каталоге вашего проекта. Этот файл должен возвращать массив пар переменных сайта / среды, которые будут добавлены в глобальный массив `$_SERVER` для каждого сайта, указанного в массиве:

    <?php

    return [
        // Set $_SERVER['key'] to "value" for the laravel.test site...
        'laravel' => [
            'key' => 'value',
        ],

        // Set $_SERVER['key'] to "value" for all sites...
        '*' => [
            'key' => 'value',
        ],
    ];

<a name="proxying-services"></a>
## Прокси-сервисы

Иногда вы можете захотеть использовать прокси-сервер домена Valet для другой службы на вашем локальном компьютере. Например, иногда вам может понадобиться запустить Valet, одновременно запуская отдельный сайт в Docker; однако Valet и Docker не могут одновременно привязываться к порту 80.

Чтобы решить эту проблему, вы можете использовать команду `proxy` для создания прокси-сервера. Например, вы можете проксировать весь трафик с `http://elasticsearch.test` на `http://127.0.0.1:9200`:

```bash
// Proxy over HTTP...
valet proxy elasticsearch http://127.0.0.1:9200

// Proxy over TLS + HTTP/2...
valet proxy elasticsearch http://127.0.0.1:9200 --secure
```

Вы можете удалить прокси-сервер с помощью команды `unproxy`:

    valet unproxy elasticsearch

Вы можете использовать команду `proxies`, чтобы перечислить все конфигурации сайта, которые проксируются:

    valet proxies

<a name="custom-valet-drivers"></a>
## Пользовательские Valet драйверы

Вы можете написать свой собственный "драйвер" Valet для обслуживания PHP-приложений, работающих на платформе или CMS, которые изначально не поддерживаются Valet. Когда вы устанавливаете Valet, создается каталог `~/.config/valet/Drivers`, который содержит файл `SampleValetDriver.php`. Этот файл содержит пример реализации, демонстрирующий, как написать пользовательский драйвер. Для написания драйвера требуется всего лишь реализовать три метода: `serves`, `isStaticFile` и `frontControllerPath`.

Все три метода получают значения `$sitePath`, `$siteName` и `$uri` в качестве своих аргументов. `$sitePath` - это полный путь к сайту, который обслуживается на вашем компьютере, например `/Users/Lisa/Sites/my-project`. `$siteName` - это часть домена "host" / "название сайта" (`my-project`). `$uri` - это URI входящего запроса (`/foo/bar`).

Как только вы создадите свой пользовательский драйвер Valet, поместите его в каталог `~/.config/valet/Drivers`, используя соглашение об именовании `FrameworkValetDriver.php`. Например, если вы пишете пользовательский драйвер valet для WordPress, ваше имя файла должно быть `WordPressValetDriver.php`.

Давайте взглянем на пример реализации каждого метода, который должен быть реализован вашим пользовательским драйвером Valet.

<a name="the-serves-method"></a>
#### Метод `serves`

Метод `serves` должен возвращать `true`, если ваш драйвер должен обработать входящий запрос. В противном случае метод должен возвращать `false`. Итак, в рамках этого метода вы должны попытаться определить, содержит ли данный `$sitePath` проект того типа, который вы пытаетесь обслуживать.

Например, давайте представим, что мы пишем `WordPressValetDriver`. Наш метод `serves` может выглядеть примерно так:

    /**
     * Determine if the driver serves the request.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return bool
     */
    public function serves($sitePath, $siteName, $uri)
    {
        return is_dir($sitePath.'/wp-admin');
    }

<a name="the-isstaticfile-method"></a>
#### Метод `isStaticFile`

`isStaticFile` должен определять, относится ли входящий запрос к файлу, который является "статическим", такому как изображение или таблица стилей. Если файл статический, метод должен возвращать полный путь к статическому файлу на диске. Если входящий запрос не относится к статическому файлу, метод должен возвращать `false`:

    /**
     * Determine if the incoming request is for a static file.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string|false
     */
    public function isStaticFile($sitePath, $siteName, $uri)
    {
        if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
            return $staticFilePath;
        }

        return false;
    }

> {note} Этот метод `isStaticFile` будет вызван только в том случае, если метод `serves` возвращает `true` для входящего запроса и URI запроса не равен `/`.

<a name="the-frontcontrollerpath-method"></a>
#### Метод `frontControllerPath`

Метод `frontControllerPath` должен возвращать полный путь к "интерфейсному контроллеру" вашего приложения, который обычно является "index.php" файл или его эквивалент:

    /**
     * Get the fully resolved path to the application's front controller.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string
     */
    public function frontControllerPath($sitePath, $siteName, $uri)
    {
        return $sitePath.'/public/index.php';
    }

<a name="local-drivers"></a>
### Локальные драйверы

Если вы хотите определить пользовательский драйвер Valet для отдельного приложения, создайте `LocalValetDriver.php` файл в корневом каталоге приложения. Ваш пользовательский драйвер может расширять базовый класс `ValetDriver` или существующий драйвер для конкретного приложения, такой как `LaravelValetDriver`:

    class LocalValetDriver extends LaravelValetDriver
    {
        /**
         * Determine if the driver serves the request.
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return bool
         */
        public function serves($sitePath, $siteName, $uri)
        {
            return true;
        }

        /**
         * Get the fully resolved path to the application's front controller.
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return string
         */
        public function frontControllerPath($sitePath, $siteName, $uri)
        {
            return $sitePath.'/public_html/index.php';
        }
    }

<a name="other-valet-commands"></a>
## Другие команды Valet

Команда  | Описание
------------- | -------------
`valet forget` | Запустите эту команду из "припаркованного" каталога, чтобы удалить его из списка припаркованных каталогов.
`valet log` | Просмотрите список журналов, которые записываются службами Valet.
`valet paths` | Просмотрите все ваши "припаркованные" пути.
`valet restart` | Перезапустите демонов Valet.
`valet start` | Запустите демонов Valet.
`valet stop` | Остановите демонов Valet.
`valet trust` | Добавьте файлы sudoers для Brew и Valet, чтобы разрешить выполнение команд Valet без запроса вашего пароля.
`valet uninstall` | Удаление Valet: показывает инструкции по удалению вручную. Передайте параметр `--force`, чтобы агрессивно удалить все ресурсы Valet.

<a name="valet-directories-and-files"></a>
## Директории и файлы Valet

Следующая информация о каталоге и файле может оказаться полезной при устранении неполадок в среде Valet:

#### `~/.config/valet`

Содержит всю конфигурацию Valet. Возможно, вы захотите сохранить резервную копию этого каталога.

#### `~/.config/valet/dnsmasq.d/`

Этот каталог содержит конфигурацию DNSMasq.

#### `~/.config/valet/Drivers/`

Этот каталог содержит драйверы Valet. Драйверы определяют, как обслуживается конкретный фреймворк / CMS.

#### `~/.config/valet/Extensions/`

Этот каталог содержит пользовательские расширения / команды Valet.

#### `~/.config/valet/Nginx/`

Этот каталог содержит все конфигурации сайта Valet Nginx. Эти файлы перестраиваются при выполнении команд `install`, `secure` и `tld`.

#### `~/.config/valet/Sites/`

Этот каталог содержит все символические ссылки для ваших [связанных проектов](#the-link-command).

#### `~/.config/valet/config.json`

Этот файл является основным файлом конфигурации Valet.

#### `~/.config/valet/valet.sock`

Этот файл представляет собой сокет PHP-FPM, используемый при установке Nginx от Valet. Это будет существовать только в том случае, если PHP запущен должным образом.

#### `~/.config/valet/Log/fpm-php.www.log`

Этот файл является пользовательским журналом ошибок PHP.

#### `~/.config/valet/Log/nginx-error.log`

Этот файл является пользовательским журналом ошибок Nginx.

#### `/usr/local/var/log/php-fpm.log`

Этот файл является системным журналом ошибок PHP-FPM.

#### `/usr/local/var/log/nginx`

Этот каталог содержит журналы доступов к Nginx и ошибок.

#### `/usr/local/etc/php/X.X/conf.d`

Этот каталог содержит файлы `*.ini` для различных настроек конфигурации PHP.

#### `/usr/local/etc/php/X.X/php-fpm.d/valet-fpm.conf`

Этот файл является файлом конфигурации пула PHP-FPM.

#### `~/.composer/vendor/laravel/valet/cli/stubs/secure.valet.conf`

Этот файл является конфигурацией Nginx по умолчанию, используемой для создания SSL-сертификатов для ваших сайтов.
