git 05366c9cb2b0d6934953026d474532d06b4abe5b

---

# Laravel Valet

- [Введение](#introduction)
    - [Valet или Homestead](#valet-or-homestead)
- [Установка](#installation)
    - [Обновление](#upgrading)
- [Обслуживание сайтов](#serving-sites)
    - [Команда "Park"](#the-park-command)
    - [Команда "Link"](#the-link-command)
    - [Защита сайтов при помощи TLS](#securing-sites)
- [Общий доступ к сайтам](#sharing-sites)
- [Пользовательские драйверы Valet](#custom-valet-drivers)
    - [Локальные драйверы](#local-drivers)
- [Другие команды Valet](#other-valet-commands)

<a name="introduction"></a>
## Введение

Valet — среда для разработки в Laravel для минималистов, работающих на Mac. Без Vagrant, без файла `/etc/hosts`. Можно даже расшаривать сайты в общий доступ через локальные туннели. _Да, нам тоже это нравится._

Laravel Valet включает на вашем Mac фоновую автозагрузку [nginx](https://www.nginx.com/). Затем с помощью [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq), Valet настраивает прокси для всех запросов к домену `*.dev` для переадресации их на сайты на вашей локальной машине.

Другими словами, это молниеносное окружение для разработки в Laravel, которое использует около 7 Мб RAM. Valet не является полной заменой для Vagrant или Homestead, но предоставляет отличную альтернативу, когда вам нужна база для гибкой настройки, максимальная скорость, или вы работаете на машине с ограниченным объёмом RAM.

Изначально Valet поддерживает, но не ограничивается только ими:

<div class="content-list" markdown="1">
- [Laravel](https://laravel.com)
- [Lumen](https://lumen.laravel.com)
- [Bedrock](https://roots.io/bedrock/)
- [CakePHP 3](https://cakephp.org)
- [Concrete5](http://www.concrete5.org/)
- [Contao](https://contao.org/en/)
- [Craft](https://craftcms.com)
- [Drupal](https://www.drupal.org/)
- [Jigsaw](http://jigsaw.tighten.co)
- [Joomla](https://www.joomla.org/)
- [Katana](https://github.com/themsaid/katana)
- [Kirby](https://getkirby.com/)
- [Magento](https://magento.com/)
- [Sculpin](https://sculpin.io/)
- [Slim](https://www.slimframework.com)
- [Statamic](https://statamic.com)
- Статический HTML
- [Symfony](https://symfony.com)
- [WordPress](https://wordpress.org)
- [Zend](https://framework.zend.com)
</div>

Также вы можете дополнить Valet своими [собственными драйверами](#custom-valet-drivers).

<a name="valet-or-homestead"></a>
### Valet или Homestead

Как вы знаете, Laravel предлагает [Homestead](/docs/{{version}}/homestead) — другую локальную среду разработки. Valet и Homestead отличаются целевой аудиторией и подходом к локальной разработке. Homestead включает в себя целую виртуальную машину с Ubuntu и автоматической настройкой Nginx. Homestead — отличный выбор, если вам нужна полностью виртуальная среда разработки на Linux или на Windows / Linux.

Valet поддерживает только Mac и требует установки PHP и сервера базы данных непосредственно на вашу локальную машину. Это легко делается при помощи [Homebrew](http://brew.sh/) такими командами, как `brew install php71` и `brew install mysql`. Valet обеспечивает молниеносную среду разработки с минимальным потреблением ресурсов, поэтому идеально подходит для разработчиков, использующих только PHP / MySQL и не нуждающихся в полностью виртуальной среде разработки.

И Valet, и Homestead являются отличным выбором для настройки среды разработки в Laravel. Выбор зависит от ваших личных предпочтений и потребностей вашей команды.

<a name="installation"></a>
## Установка

**Valet требует macOS и [Homebrew](http://brew.sh/). Перед установкой необходимо убедиться, что другие программы, такие как Apache и Nginx, не используют 80 порт вашей локальной машины.**

<div class="content-list" markdown="1">
- Установите или обновите [Homebrew](http://brew.sh/) до последней версии с помощью `brew update`.
- Установите PHP 7.1 с помощью Homebrew командой `brew install homebrew/php/php71`.
- Установите Valet через Composer при помощи команды `composer global require laravel/valet`. Убедитесь, что в вашей системной переменной "PATH" есть директория `~/.composer/vendor/bin`.
- Выполните команду `valet install`. Она установит и настроит Valet и DnsMasq, и пропишет демон Valet в автозагрузку.
</div>

Послу установки Valet, попробуйте запинговать любой домен `*.dev` на своем терминале при помощи команды `ping foobar.dev`. Если Valet установлен корректно, то вы увидите, что этот домен соответствует `127.0.0.1`.

Valet автоматически запустит своего демона при запуске ОС. После начальной установки Valet больше не потребуется выполнять `valet start` или `valet install`.

#### Использование другого домена

По умолчанию Valet использует для ваших проектов домен верхнего уровня `.dev`. Если вы хотите использовать другой домен, используйте команду `valet domain tld-name`.

Например, чтобы использовать `.app` вместо `.dev`, выполните `valet domain app`, и Valet начнет атоматически использовать ваши проекты на `*.app`.

#### База данных

Если вам нужна база данных, попробуйте MySQL с помощью команды `brew install mysql`. После установки MySQL, вы можете запустить её командой `brew services start mysql`. Затем вы можете подключиться к БД на `127.0.0.1`, используя имя пользователя `root` и пароль - пустая строка.

<a name="upgrading"></a>
### Обновление

Вы можете обновить установленный Valet терминальной командой `composer global update`. После обновления рекомендуется выполнить команду `valet install`, чтобы Valet сделал дополнительные обновления ваших конфигурационных файлов при необходимости.

#### Обновление до Valet 2.0

В Valet 2.0 изменён базовый веб-сервер с Caddy на Nginx. Перед обновлением на эту версию вы должны выполнить следующие команды, чтобы остановить и удалить существующий демон Caddy:

    valet stop
    valet uninstall

Далее вам надо обновить Valet до последней версии. Это делается с помощью Git или Composer, в зависимости от того, как вы устанавливали Valet изначально. Если вы устанавливали его через Composer, то вам надо использовать следующие команды для обновления до последней мажорной версии:

    composer global require laravel/valet

Когда будет скачан свежий исходный код Valet, вы должны выполнить команду `install`:

    valet install
    valet restart

После обновления может понадобиться пере-разместить или пере-привязать ваши сайты.

<a name="serving-sites"></a>
## Обслуживание сайтов

После установки Valet можно начать обслуживать сайты. Valet предоставляет две команды для помощи в обслуживании Laravel-сайтов: `park` и `link`.

<a name="the-park-command"></a>
**Команда `park`**

<div class="content-list" markdown="1">
- Создайте новый каталог на своём Mac с помощью команды `mkdir ~/Sites`. Далее, перейдите к ней  `cd ~/Sites` и выполните `valet park`. Эта команда зарегистрирует текущий каталог как путь, по которому Valet будет искать сайты.
- Теперь создайте новый Laravel-сайт в этом каталоге: `laravel new blog`.
- Откройте `http://blog.dev` в своём браузере.
</div>

**Вот и всё.** Теперь все проекты, которые вы разместите в каталоге "парковки", будут обслуживаться автоматически с адресами в соответствии с названиями их папок в конвенции `http://folder-name.dev` convention.

<a name="the-link-command"></a>
**Команда `link`**

Для обслуживания Laravel-сайтов также можно использовать команду `link`. Эта команда полезна, когда вы хотите обслуживать один сайт в каталоге, а не весь каталог.

<div class="content-list" markdown="1">
- Для использования команды перейдите к одному из своих проектов и выполните `valet link app-name` в терминале. Valet создаст в `~/.valet/Sites` символьную ссылку на ваш текущий каталог.
- После запуска команды `link` вы можете перейти на сайт `http://app-name.dev` в своём браузере.
</div>

Для просмотра списка всех привязанных каталогов выполните команд `valet links`. Для удаления символьной ссылки используйте `valet unlink app-name`.

> {tip} Вы можете использовать команду `valet link` для обслуживания одного проекта из нескольких (под)доменов. Чтобы добавить поддомен или другой домен в свой проект, выполните `valet link subdomain.app-name` в папке проекта.

<a name="securing-sites"></a>
**Защита сайтов при помощи TLS**

По умолчанию Valet обслуживает сайты через чистый HTTP. Но при желании вы можете включить шифрование TLS, используя HTTP/2, с помощью команды `secure`. Например, если Valet обслуживает ваш сайт на домене `laravel.dev`, то для его защиты вам надо выполнить команду:

    valet secure laravel

Чтобы отключить защиту и перевести трафик обратно на чистый HTTP, используйте команду `unsecure`. Как и команда `secure`, эта команда принимает имя хоста, для которого необходимо отключить защиту:

    valet unsecure laravel

<a name="sharing-sites"></a>
## Общий доступ к сайтам

Valet имеет команду для открытия доступа к вашим локальным сайтам для всех. Кроме Valet не нужно никакое ПО.

Для открытия доступа к сайту перейдите в терминале в его каталог и выполните команду `valet share`. URL для общего доступа будет помещён в буфер обмена, и его можно вставить в браузер. Вот и всё.

Для закрытия общего доступа нажмите `Control + C`, чтобы отменить процесс.

> {note} В настоящее время `valet share` не поддерживает общий доступ к сайтам, которые были защищены командой `valet secure`.

<a name="custom-valet-drivers"></a>
## Пользовательские драйверы Valet

Вы можете написать свой собственный «драйвер» Valet для обслуживания PHP-приложений, работающих на другом фреймворке или CMS, изначально не поддерживаемых Valet. При установке Valet создаётся папка `~/.valet/Drivers`, содержащая файл `SampleValetDriver.php`. В этом файле находится пример реализации драйвера для демонстрации. Для написания драйвера необходимо реализовать всего три метода: `serves`, `isStaticFile` и `frontControllerPath`.

Все три метода принимают в качестве аргументов `$sitePath`, `$siteName` и `$uri`. Параметр `$sitePath` — полный путь к сайту на вашей машине, например, `/Users/Lisa/Sites/my-project`. Параметр `$siteName` — часть домена "хост" / "имя сайта" (`my-project`). Параметр `$uri` — URI входящих запросов (`/foo/bar`).

Когда вы завершили написание своего драйвера Valet, поместите его в директорию `~/.valet/Drivers`, используя принцип именования `FrameworkValetDriver.php`. Например, если вы написали драйвер для WordPress, то имя файла должно быть `WordPressValetDriver.php`.

Давайте рассмотрим примеры реализации каждого из этих методов.

#### Метод `serves`

Метод `serves` должен возвращать `true`, если ваш драйвер должен обрабатывать входящие запросы. Иначе метод должен возвращать `false`. В этом методе вы должны попытаться определить, содержит ли данный `$sitePath` проект того типа, который вы хотите обслуживать.

Например, давайте предположим, что мы пишем `WordPressValetDriver`. Наш метод `serves` должен выглядеть примерно так:

    /**
     * Определить, обслуживает ли драйвер запрос.
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

#### Метод `isStaticFile`

Метод `isStaticFile` должен определять, является ли входящий запрос запросом к статическому файлу, такому как изображение или таблица стилей. Если файл статический, метод должен вернуть полный путь к этому файлу на диске. Если входящий запрос не к статическому файлу, метод должен вернуть `false`:

    /**
     * Определить, является ли входящий запрос запросом к статическому файлу.
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

> {note} Метод `isStaticFile` будет вызван, только если метод `serves` возвращает `true` для входящего запроса, и значение URI запроса не равно `/`.

#### Метод `frontControllerPath`

Метод `frontControllerPath` должен вернуть полный путь к "первичному контроллеру" вашего приложения, которым обычно является ваш файл "index.php" или его эквивалент:

    /**
     * Получение полного пути к первичному контроллеру приложения.
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

Если вы хотите определить пользовательский драйвер Valet для одного приложения, создайте `LocalValetDriver.php` в корневой директории приложения. Ваш драйвер может расширить базовый класс `ValetDriver` или расширить существующий определенный драйвер приложения, такой как `LaravelValetDriver`:

    class LocalValetDriver extends LaravelValetDriver
    {
        /**
         * Определить, обслуживает ли драйвер запрос.
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
`valet forget` | Выполните эту команду в каталоге "парковки" для удаления его из списка каталогов парковки.
`valet paths` | Просмотр всех путей "парковки".
`valet restart` | Перезапуск демона Valet.
`valet start` | Запуск демона Valet.
`valet stop` | Остановка демона Valet.
`valet uninstall` | Полное удаление демона Valet.
