git 4deba2bfca6636d5cdcede3f2068eff3b59c15ce

---

# Установка

- [Установка Composer](#install-composer)
- [Установка Laravel](#install-laravel)
- [Требования к серверу](#server-requirements)
- [Настройка](#configuration)

<a name="install-composer"></a>
## Установка Composer

Laravel использует [Composer](http://getcomposer.org) для управления зависимостями. Поэтому прежде чем ставить Laravel вы должны установить Composer.

<a name="install-laravel"></a>
## Установка Laravel

### При помощи установщика Laravel

Используя Composer скачайте установщик Laravel.

	composer global require "laravel/installer=~1.1"

Указав в качестве PATH директорию `~/.composer/vendor/bin`, станет возможным использование команды `laravel`.

После установки, простая команда `laravel new` создаст свеженькое Laravel приложение в директории, которую вы укажете. Например, `laravel new blog` создаст директорию `blog` и установит туда Laravel со всеми зависимостями. Этот метод установки намного быстрее, чем установка через Composer:

	laravel new blog

### При помощи Composer 

Вы также можете установить Laravel используя команду Composer `create-project`:

	composer create-project laravel/laravel --prefer-dist

<a name="server-requirements"></a>
## Требования к серверу

У Laravel всего несколько требований к вашему серверу:

- PHP >= 5.4
- Mcrypt PHP Extension
- OpenSSL PHP Extension
- Mbstring PHP Extension

Начиная с PHP 5.5, в некоторых операционных системах может понадобиться ручная установка PHP JSON extension. В Ubuntu, например, это можно сделать при помощи `sudo apt-get install php5-json`.

<a name="configuration"></a>
## Настройка

Первое, что вы должны сделать после установки Laravel - установить ключ шифрования сессий и кук. Это случайная строка из 32 символов, находится в файле `.env`, параметр 'APP_KEY'. Если вы устанавливали Laravel при помощи Composer, то ключ уже сгенерен. Вы можете сгенерить его вручную artisan-командой `key:generate`. **Если ключ шифрования отсутствует, ваши сессии, куки другая шифруемая информация не будет зашифрована надежным образом.**.

Laravel практически не требует другой начальной настройки - вы можете сразу начинать разработку. Однако может быть полезным изучить файл `config/app.php` - он содержит несколько настроек вроде `timezone` и `locale`, которые вам может потребоваться изменить в соответствии с нуждами вашего приложения.

Далее вы можете сконфигурить [настройки среды выполнения](/docs/{{version}}/configuration#environment-configuration).

> **Примечание:** Никогда не устанавливайте настройку `app.debug` в `true` на рабочем (продакшн) окружении.

### Права на запись

Папки внутри `storage` должны быть доступны веб-серверу для записи. Если вы устанавливаете фреймворк на Linux или MacOS - открыть папки на запись можно командой `chmod -R 777 storage`

<a name="pretty-urls"></a>
## Красивые URL

### Apache

Laravel поставляется вместе с файлом `public/.htaccess`, который настроен для обработки URL без указания `index.php`. Если вы используете Apache в качестве веб-сервера обязательно включите модуль `mod_rewrite`.

Если стандартный `.htaccess` не работает для вашего Apache, попробуйте следующий:

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]

### Nginx

Если вы используете в качестве веб-сервера Nginx, то используйте для ЧПУ следующую конструкцию:

	location / {
		try_files $uri $uri/ /index.php?$query_string;
	}

Если вы используете [Homestead](/docs/{{version}}/homestead), то вам ничего делать не нужно, там всё это уже настроено.
