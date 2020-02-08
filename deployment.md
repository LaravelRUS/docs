git 89a3928f9120fcaf8a3a1499d7a5db4d08ea2eee

---

# Развёртывание (деплой)

- [Введение](#introduction)
- [Настройка сервера](#server-configuration)
    - [Nginx](#nginx)
- [Оптимизация](#optimization)
    - [Автозагрузка классов](#autoloader-optimization)
    - [Загрузка конфигурационных файлов](#optimizing-configuration-loading)
    - [Загрузка роутов](#optimizing-route-loading)
- [Развёртывание при помощи Forge](#deploying-with-forge)

<a name="introduction"></a>
## Введение

Есть несколько моментов, на которые нужно обратить внимание во время деплоя приложения на продакшн-сервер.

> {tip} Если вы испытываете трудности в настройке сервера, используйте [Laravel Forge](https://forge.laravel.com)

<a name="server-configuration"></a>
## Настройка сервера

<a name="nginx"></a>
### Nginx

Если на вашем сервере используется nginx, вы можете использовать следующий конфиг для домена:

    server {
        listen 80;
        server_name example.com;
        root /example.com/public;

        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";

        index index.html index.htm index.php;

        charset utf-8;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }

        error_page 404 /index.php;

        location ~ \.php$ {
            fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
            include fastcgi_params;
        }

        location ~ /\.(?!well-known).* {
            deny all;
        }
    }

<a name="optimization"></a>
## Оптимизация

<a name="autoloader-optimization"></a>
### Автозагрузка классов

Убедитесь, что у вас сгенерирован оптимизированный автозагрузчик классов:

    composer install --optimize-autoloader --no-dev

> {tip} Чтобы команда `composer install` отрабатывала быстрее, в репозитории вашего кода должен присутствовать файл `composer.lock`

<a name="optimizing-configuration-loading"></a>
### Загрузка конфигурационных файлов

После выгрузки вашего приложения на продакшн-сервер задайте кэширование конфигов для более быстрой загрузки:

    php artisan config:cache

Эта команда соберёт все файлы конфигов в один быстроисполняемый файл.

> {note} Если вы выполняете `config:cache` в процессе разработки, вы должны быть уверены, что вызываете функцию `env` только из ваших конфигурационных файлов. Как только вы включите кэширование конфигов, функция `env` будет возвращать `null`

<a name="optimizing-route-loading"></a>
### Загрузка роутов

Если в вашем приложении много роутов, вы можете ускорить их загрузку, закешировав их:

    php artisan route:cache

Команда трансформирует стандартный файл роутов в кэшированную версию с уменьшенным количеством регистраций роутов и т.п., которая выполняется быстрее.

> {note} Кешированию поддаются только роуты с использованием контроллеров.

<a name="deploying-with-forge"></a>
## Развёртывание при помощи Forge

Если вы испытываете трудности в установке и настройке сервера, используйте [Laravel Forge](https://forge.laravel.com). Этот сервис позволяет регистрировать VPS таки провайдеров как DigitalOcean, Linode, AWS, настраивать произвольный VPS, устанавливать на нём Nginx, MySQL, Redis, Memcached, Beanstalk и т.п. и предоставлять единую админку для управления VPS. 
