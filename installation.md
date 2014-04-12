git 71802e293e6738247fe5485816e5f4be66311558
---
# Установка

- [Установка Composer](#install-composer)
- [Установка Laravel](#install-laravel)
- [Загрузка архива](#server-requirements)
- [Настройка](#configuration)
- [Красивые URL](#pretty-urls)

<a name="install-composer"></a>
## Установка Composer

Laravel использует [Composer](http://getcomposer.org) для управления зависимостями. Для начала скачайте файл `composer.phar`. Дальше вы можете либо оставить этот Phar-архив в своей локальной папке с проектом, либо переместить его в `usr/local/bin`, чтобы использовать его в рамках всей системы. Для Windows вы можете использовать [официальный установщик](https://getcomposer.org/Composer-Setup.exe).

<a name="install-laravel"></a>
## Установка Laravel

### При помощи установщика Laravel

Загрузите [Laravel installer PHAR archive](http://laravel.com/laravel.phar). Если у вас Mac OS или Linux, переименуйте его в `laravel` и положите в `/usr/local/bin`. Если у вас Windows, положите `laravel.phar` в папку, которая у вас прописана в PATH, плюс в этой же папке создайте файл `laravel.bat` со следующим содержимым:

	@php "%~dp0laravel.phar" %*

Теперь, к примеру, если вы исполните в терминале команду `laravel new blog`, будет создана папка `blog`, в которую будет загружен фреймворк с уже подтянутыми зависимостями composer. Этот способ установки Laravel наиболее быстрый, особенно на Windows, где Composer работает довольно медленно.

### При помощи Composer 

Вы можете установить Laravel с помощью команды `create-project`:

	composer create-project laravel/laravel --prefer-dist

### Загрузка архива

Cкачайте [последнюю версию фреймворка](https://github.com/laravel/laravel/archive/master.zip) и извлеките архив в папку на вашем сервере. Далее скачайте [Composer](http://getcomposer.org) в эту же папку и выполните в этой папке `php composer.phar install` (или `composer install`, если composer у вас уже установлен в системе глобально) для установки всех зависимостей библиотеки. Этот процесс также требует, чтобы у вас был установлен [Git](http://git-scm.com/).

Если вы хотите обновить Laravel выполните команду `php composer.phar update`.

<a name="server-requirements"></a>
## Требования к серверу

У Laravel всего несколько требований к вашему серверу:

- PHP >= 5.3.7
- MCrypt PHP Extension

<a name="configuration"></a>
## Настройка

Laravel практически не требует начальной настройки - вы можете сразу начинать разработку. Однако вам может пригодиться файл `app/config/app.php` и его документация - он содержит несколько настроек вроде `timezone` и `locale`, которые вам может потребоваться изменить в соответствии с нуждами вашего приложения.

> **Примечание:** В Laravel 3 и в ранних версиях Laravel 4 единственная настройка, которую вам нужно было изменить - `key` в файле `app/config/app.php`. Это значение должно быть случайной строкой длиной 32 символа. Оно используется при шифровании и зашифрованные строки не будут безопасными, пока вы не измените эту настройку. Теперь в Laravel 4 это делается автоматически. Вы также можете быстро его установить с помощью следующей команды: `php artisan key:generate`.


<a name="permissions"></a>
### Права доступа
Laravel требует, чтобы у сервера были права на запись в папку `app/storage`.

<a name="paths"></a>
### Пути

Некоторые системные пути Laravel - настраиваемые; для этого обратитесь к файлу `bootstrap/paths.php`.

> **Примечание:** Laravel спроектирован так, чтобы защитить код вашего приложения и локальное хранилище - для этого общедоступные файлы помещаются в папку `public`. Подразумевается, что эта папка является корневой папкой вашего сайта (DocumentRoot в Apache).

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

Если вы используете в качествет веб-сервера Nginx, то используйте для ЧПУ следующую конструкцию:

	location / {
		try_files $uri $uri/ /index.php?$query_string;
	}