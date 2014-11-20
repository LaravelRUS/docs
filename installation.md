git 5812df5ca636f02c5b1581532355cdd6f247041e

---

# Установка

- [Установка Composer](#install-composer)
- [Установка Laravel](#install-laravel)
- [Требования к серверу](#server-requirements)

<a name="install-composer"></a>
## Установка Composer

Laravel использует [Composer](http://getcomposer.org) для управления зависимостями. Для начала, скачайте файл `composer.phar`. Дальше вы можете либо оставить этот Phar-архив в своей локальной папке с проектом, либо переместить его в `usr/local/bin`, чтобы использовать его в рамках всей системы. Для Windows вы можете использовать [официальный установщик](https://getcomposer.org/Composer-Setup.exe).

<a name="install-laravel"></a>
## Установка Laravel

### При помощи установщика Laravel

Для начала, скачайте установщик Laravel используя Composer.

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
- mcrypt PHP Extension
- mbstring PHP Extension

> **Примечание:** Начиная с PHP 5.5, в некоторых операционных системах может понадобиться ручная установка PHP JSON extension. В Ubuntu, например, это можно сделать при помощи `apt-get install php5-json`.
