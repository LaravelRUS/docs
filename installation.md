git 2166b4af1b9b805d0ef40aac6af90d79707a7453

---

# Установка

- [Установка](#installation)
    - [Требования к серверу](#server-requirements)
    - [Установка Laravel](#installing-laravel)
    - [Настройка](#configuration)
- [Настройка веб-сервера](#web-server-configuration)
    - [Настройка директорий](#directory-configuration)
    - ["Красивые" URL](#pretty-urls)

<a name="installation"></a>
## Установка

<a name="server-requirements"></a>
### Требования к серверу

Фреймворк Laravel предъявляет некоторые системные требования. Конечно же, виртуальная машина [Laravel Homestead](/docs/{{version}}/homestead) соответствует всем этим требованиям, поэтому настоятельно рекомендуется использовать Homestead в качестве основной локальной среды разработки с Laravel.

Однако, если вы не используете Homestead, вам необходимо убедиться, что ваш сервер соответствует следующим требованиям:

<div class="content-list" markdown="1">
- PHP >= 7.2.0
- Расширение PHP BCMath
- Расширение PHP Ctype
- Расширение PHP Fileinfo
- Расширение PHP JSON
- Расширение PHP Mbstring
- Расширение PHP OpenSSL 
- Расширение PHP PDO
- Расширение PHP Mbstring
- Расширение PHP Tokenizer
- Расширение PHP XML
</div>

<a name="installing-laravel"></a>
### Установка Laravel

Laravel использует [Composer](https://getcomposer.org) для управления своими зависимостями, поэтому убедитесь в том, что Composer установлен на вашей машине.

#### С помощью установщика Laravel

Сначала скачайте установщик Laravel с помощью Composer:

    composer global require laravel/installer

Проверьте, чтобы директория composer'а `vendor/bin` находилась в переменной $PATH, что позволит вашей системе найти и выполнить команду `laravel`. Эта директория располагается в разных местах в зависимости от вашей операционной системы, но обычно она находится тут:

<div class="content-list" markdown="1">
- macOS: `$HOME/.composer/vendor/bin`
- Windows: `%USERPROFILE%\AppData\Roaming\Composer\vendor\bin`
- Linux: `$HOME/.config/composer/vendor/bin` или `$HOME/.composer/vendor/bin`
</div>

Также вы можете определить директорию выполнив команду `composer global about` (смотрите первую строчку вывода).

После установки команда `laravel new` создаёт свежую установку Laravel в указанной вами директории. Например, `laravel new blog` создаст директорию с названием `blog`, которая будет содержать свежую установку Laravel со всеми зависимостями:

    laravel new blog

#### С помощью Composer Create-Project

В качестве альтернативы вы можете использовать Composer для установки Laravel с помощью команды `create-project`:

    composer create-project --prefer-dist laravel/laravel blog "6.*"

#### Локальный сервер разработки

Если локально у вас уже установлен PHP и вы хотели бы использовать встроенный сервер для работы вашего приложения, то вы можете использовать команду Artisan `serve`. Эта команда запустит сервер разработки по адресу `http://localhost:8000`:

    php artisan serve

Конечно же, [Homestead](/docs/{{version}}/homestead) и [Valet](/docs/{{version}}/valet) предоставляют наиболее надежные способы локальной разработки.

<a name="configuration"></a>
### Настройка

#### Общедоступная директория

После установки Laravel вам следует указать директорию `public` в качестве корневой директории вашего веб-сервера. Файл `index.php` в этой категории выступает в роли фронт-контроллера всех HTTP-запросов, поступающих в ваше приложение.

#### Файлы настройки

Все файлы настройки фреймворка Laravel расположены в директории `config`. Параметры в каждом из них снабжены комментариями, поэтому не стесняйтесь пройтись по этим файлам и познакомиться с доступными параметрами настройки.

#### Права доступа на директории

Так же, после установки Laravel вам может потребоваться настройка некоторых прав доступа. Директории внутри `storage` и `bootstrap/cache` должны быть доступны для записи веб-сервером, в противном случае Laravel не запустится. Если вы используете виртуальную машину [Homestead](/docs/{{version}}/homestead), то эти права доступа уже установлены.

#### Ключ приложения

Следующее, что вы должны сделать после установки Laravel, это создать ключ шифрования для вашего приложения в виде случайного набора символов. Если вы установили Laravel через Composer или установщик Laravel, то этот ключ уже был создан с помощью команды `php artisan key:generate`.

Как правило, это строка должна быть длиной в 32 символа. Ключ должен быть указан в параметре файла окружения `.env`. Если вы не переименовывали файл `.env.example` в `.env`, то следует сделать это сейчас. **Если ключ приложения не создан, то сессии ваших пользователей и другие шифруемые данные не будут в безопасности!**

#### Дополнительная настройка

Laravel практически не требует настройки из коробки. Вы сразу можете начать разработку! Однако, рекомендуем ознакомиться с файлом `config/app.php` — он содержит в себе несколько параметров, таких как часовой пояс (`timezone`) и локаль (`locale`), которые вы можете изменить согласно потребностям вашего приложения.

Вы также можете настроить некоторые дополнительные компоненты Laravel, такие как:

<div class="content-list" markdown="1">
- [Кэширование](/docs/{{version}}/cache#configuration)
- [База данных](/docs/{{version}}/database#configuration)
- [Сессия](/docs/{{version}}/session#configuration)
</div>

<a name="web-server-configuration"></a>
## Настройка веб-сервера

<a name="directory-configuration"></a>
### Настройка даректорий

Laravel всегда должен располагаться за пределами директории, доступной из web. Не нужно размещать приложение в поддиректории "web root". Попытка сделать это может привести к раскрытию конфиденциальной информации, содержащейся в файлахвашего приложения.

<a name="pretty-urls"></a>
### "Красивые" URL

#### Apache

В Laravel есть файл `public/.htaccess`, который используется для отображения ссылок без указания фронт-контроллера `index.php` в запрашиваемом адресе. Перед началом работы Laravel с сервером Apache, убедитесь, что модуль `mod_rewrite` включен, он необходим для корректной обработки файла `.htaccess`.

Если поставляемый с Laravel файл `.htaccess` не работает с вашим сервером Apache, то попробуйте альтернативу:

    Options +FollowSymLinks -Indexes
    RewriteEngine On

    RewriteCond %{HTTP:Authorization} .
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

#### Nginx

Если вы используете Nginx, то следующая директива в конфигурации вашего сайта направит все запросы на фронт-контроллер `index.php`:

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

При использовании [Homestead](/docs/{{version}}/homestead) или [Valet](/docs/{{version}}/valet), функция "красивых" URL будет работать без дополнительных настроек.
