---
git: 46c2634ef5a4f15427c94a3157b626cf5bd3937f
---

# Laravel Octane

<a name="introduction"></a>
## Введение

[Laravel Octane](https://github.com/laravel/octane) повышает производительность вашего приложения, обслуживая его с использованием мощных серверов приложений, включая [FrankenPHP](https://frankenphp.dev/), [Open Swoole](https://openswoole.com/), [Swoole](https://github.com/swoole/swoole-src) и [RoadRunner](https://roadrunner.dev). Octane загружает ваше приложение один раз, сохраняет его в памяти, а затем отправляет ему запросы на "сверхзвуковой скорости".

<a name="installation"></a>
## Установка

Octane можно установить через диспетчер пакетов Composer:

```shell
composer require laravel/octane
```

После установки Octane вы можете выполнить Artisan-команду `octane: install`, которая установит файл конфигурации Octane в ваше приложение:

```shell
php artisan octane:install
```

<a name="server-prerequisites"></a>
## Требования к серверу

> [!WARNING]  
> Laravel Octane требует [PHP 8.1+](https://php.net/releases/).

<a name="frankenphp"></a>
### FrankenPHP

> [!WARNING]
> Интеграция Octane c FrankenPHP находится в бета-версии и должна использоваться с осторожностью в продакшене.

[FrankenPHP](https://frankenphp.dev) - это сервер приложений PHP, написанный на Go, который поддерживает современные веб-функции, такие как ранние подсказки и сжатие Zstandard. Когда вы устанавливаете Octane и выбираете FrankenPHP в качестве сервера, Octane автоматически загрузит и установит для вас бинарный файл FrankenPHP.

<a name="frankenphp-via-laravel-sail"></a>

#### FrankenPHP через Laravel Sail

Если вы планируете разрабатывать ваше приложение, используя [Laravel Sail](/docs/{{version}}/sail), вы должны выполнить следующие команды для установки Octane и FrankenPHP:

```shell
./vendor/bin/sail up

./vendor/bin/sail composer require laravel/octane
```

Затем вы должны использовать команду Artisan `octane:install`, чтобы установить бинарный файл FrankenPHP:

```shell
./vendor/bin/sail artisan octane:install --server=frankenphp
```

Наконец, добавьте переменную окружения `SUPERVISOR_PHP_COMMAND` в определение сервиса `laravel.test` в файле `docker-compose.yml` вашего приложения. Эта переменная окружения будет содержать команду, которую Sail будет использовать для обслуживания вашего приложения с использованием Octane вместо сервера разработки PHP:

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=frankenphp --host=0.0.0.0 --admin-port=2019 --port=80" # [tl! add]
```

<a name="roadrunner"></a>
### RoadRunner

[RoadRunner](https://roadrunner.dev) работает на двоичном файле RoadRunner, который создается с использованием Go. При первом запуске сервера Octane на базе RoadRunner Octane предложит загрузить и установить для вас двоичный файл RoadRunner.

<a name="roadrunner-via-laravel-sail"></a>
#### RoadRunner через Laravel Sail

Если вы планируете разрабатывать свое приложение с использованием [Laravel Sail](/docs/{{version}}/sail), вам следует выполнить следующие команды для установки Octane и RoadRunner:

```shell
./vendor/bin/sail up

./vendor/bin/sail composer require laravel/octane spiral/roadrunner-cli spiral/roadrunner-http 
```

Затем вы должны запустить оболочку Sail и использовать исполняемый файл `rr` для получения последней сборки двоичного файла RoadRunner на основе Linux:

```shell
./vendor/bin/sail shell

# через Sail оболочку ...
./vendor/bin/rr get-binary
```

Затем добавьте переменную окружения `SUPERVISOR_PHP_COMMAND` в определение сервиса `laravel.test` в файле `docker-compose.yml` вашего приложения. Эта переменная окружения будет содержать команду, которую Sail будет использовать для обслуживания вашего приложения с использованием Octane вместо сервера разработки PHP:

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=roadrunner --host=0.0.0.0 --rpc-port=6001 --port=80" # [tl! add]
```

Наконец, убедитесь, что двоичный файл `rr` исполняется, и создайте свои образы Sail:

```shell
chmod +x ./rr

./vendor/bin/sail build --no-cache
```

<a name="swoole"></a>
### Swoole

Если вы планируете использовать сервер приложений Swoole для обслуживания приложения Laravel Octane, вы должны установить расширение Swoole PHP. Обычно это можно сделать через PECL:

```shell
pecl install swoole
```

<a name="openswoole"></a>
#### Open Swoole

Если вы хотите использовать сервер приложений Open Swoole для обслуживания вашего приложения Laravel Octane, вам необходимо установить расширение PHP Open Swoole. Обычно это можно сделать через PECL:

```shell
pecl install openswoole
```

Использование Laravel Octane с Open Swoole предоставляет те же функции, что и Swoole, такие как параллельные задачи, тики и интервалы.

<a name="swoole-via-laravel-sail"></a>
#### Swoole через Laravel Sail

> [!WARNING]  
> Перед обслуживанием приложения Octane через Sail убедитесь, что у вас установлена последняя версия [Laravel Sail](/docs/{{version}}/sail), и выполните `./vendor/bin/sail build --no-cache` в корневом каталоге вашего приложения.

В качестве альтернативы вы можете разработать приложение Octane на основе Swoole, используя [Laravel Sail](/docs/{{version}}/sail), официальную среду разработки на основе Docker для Laravel. Laravel Sail по умолчанию включает расширение Swoole. Однако вам все равно нужно будет настроить файл `docker-compose.yml`, используемый Sail.

Чтобы начать, добавьте переменную окружения `SUPERVISOR_PHP_COMMAND` в определение службы `laravel.test` в файле `docker-compose.yml` вашего приложения. Эта переменная окружения будет содержать команду, которую Sail будет использовать для обслуживания вашего приложения с использованием Octane вместо сервера разработки PHP:

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=swoole --host=0.0.0.0 --port=80" # [tl! add]
```

Наконец, создайте свои образы Sail:

```bash
./vendor/bin/sail build --no-cache
```

<a name="swoole-configuration"></a>
#### Конфигурация Swoole

Swoole поддерживает несколько дополнительных параметров конфигурации, которые вы можете добавить в свой файл конфигурации `octane` при необходимости. Поскольку их редко нужно изменять, эти параметры не включены в файл конфигурации по умолчанию:

```php
'swoole' => [
    'options' => [
        'log_file' => storage_path('logs/swoole_http.log'),
        'package_max_length' => 10 * 1024 * 1024,
    ],
],
```

<a name="serving-your-application"></a>
## Запуск приложения

Сервер Octane можно запустить с помощью Artisan-команды `octane:start`. По умолчанию эта команда будет использовать сервер, указанный в параметре конфигурации `server` в файле конфигурации `octane` вашего приложения:

```shell
php artisan octane:start
```

По умолчанию Octane запускает сервер на порту 8000, поэтому вы можете получить доступ к своему приложению в веб-браузере через `http://localhost:8000`.

<a name="serving-your-application-via-https"></a>
### Запуск приложения с HTTPS

По умолчанию приложения, работающие через Octane, генерируют ссылки с префиксом `http://`. Переменная окружения `OCTANE_HTTPS`, используемая в файле конфигурации приложения `config/octane.php`, имеет значение `true` при обслуживании приложения через HTTPS. Если значение конфигурации установлено в значение `true`, Octane укажет Laravel добавлять префикс `https://` ко всем сгенерированным ссылкам:

```php
'https' => env('OCTANE_HTTPS', false),
```

<a name="serving-your-application-via-nginx"></a>
### Запуск приложения с Nginx

> [!NOTE]
> Если вы не совсем готовы управлять конфигурацией своего сервера или вам неудобно настраивать все различные службы, необходимые для запуска надежного приложения Laravel Octane, ознакомьтесь с [Laravel Forge](https://forge.laravel.com).

В производственных средах вы должны обслуживать приложение Octane на традиционном веб-сервере, таком как Nginx или Apache. Это позволит веб-серверу обслуживать ваши статические ресурсы, такие как изображения и таблицы стилей, а также управлять прекращением действия вашего сертификата SSL.

В приведенном ниже примере файла конфигурации, Nginx будет обслуживать статические ресурсы сайта и запросы прокси к серверу Octane, который работает на порту 8000:

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80;
    listen [::]:80;
    server_name domain.com;
    server_tokens off;
    root /home/forge/domain.com/public;

    index index.php;

    charset utf-8;

    location /index.php {
        try_files /not_exists @octane;
    }

    location / {
        try_files $uri $uri/ @octane;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log  /var/log/nginx/domain.com-error.log error;

    error_page 404 /index.php;

    location @octane {
        set $suffix "";

        if ($uri = /index.php) {
            set $suffix ?$query_string;
        }

        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header SERVER_PORT $server_port;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;

        proxy_pass http://127.0.0.1:8000$suffix;
    }
}
```

<a name="watching-for-file-changes"></a>
### Наблюдение за изменениями файлов

Поскольку ваше приложение загружается в память один раз при запуске сервера Octane, любые изменения в файлах вашего приложения не будут отражены при обновлении браузера. Например, определения маршрутов, добавленные в ваш файл `routes/web.php`, не будут отображаться до перезапуска сервера. Для удобства вы можете использовать флаг `--watch`, чтобы дать Octane команду автоматически перезапускать сервер при любых изменениях файла в вашем приложении:

```shell
php artisan octane:start --watch
```

Перед использованием этой функции вы должны убедиться, что [Node](https://nodejs.org) установлен в вашей локальной среде разработки. Кроме того, вы должны установить библиотеку просмотра файлов [Chokidar](https://github.com/paulmillr/chokidar) в свой project:library:

```shell
npm install --save-dev chokidar
```

Вы можете настроить каталоги и файлы, за которыми следует наблюдать, используя параметр конфигурации `watch` в файле конфигурации вашего приложения `config/octane.php`.

<a name="specifying-the-worker-count"></a>
### Указание количества Worker

По умолчанию Octane запускает обработчика запросов приложений для каждого ядра ЦП (Центрального Процессора), предоставленного вашим компьютером. Затем эти воркеры (workers) будут использоваться для обслуживания входящих HTTP-запросов, когда они входят в ваше приложение. Вы можете вручную указать, сколько воркеров вы хотите запустить, используя опцию `--workers` при вызове команды `octane:start`:

```shell
php artisan octane:start --workers=4
```

Если вы используете сервер приложений Swoole, вы также можете указать, сколько ["task workers"](#concurrent-tasks) вы хотите запустить:

```shell
php artisan octane:start --workers=4 --task-workers=6
```

<a name="specifying-the-max-request-count"></a>
### Указание максимального количества запросов

Чтобы предотвратить случайные утечки памяти, Octane корректно перезапустит воркер (worker) после обработки 500 запросов. Чтобы изменить это число, вы можете использовать параметр `--max-requests`:

```shell
php artisan octane:start --max-requests=250
```

<a name="reloading-the-workers"></a>
### Перезагрузка Workers

Вы можете корректно перезапустить рабочие приложения сервера Octane, используя команду `octane:reload`. Как правило, это следует делать после развертывания, чтобы вновь развернутый код загружался в память и использовался для обслуживания последующих запросов:

```shell
php artisan octane:reload
```

<a name="stopping-the-server"></a>
### Остановка сервера

Вы можете остановить сервер Octane, используя Artisan-команду `octane: stop`:

```shell
php artisan octane:stop
```

<a name="checking-the-server-status"></a>
#### Проверка статуса сервера Octane

Вы можете проверить текущий статус сервера Octane, используя Artisan-команду `octane: status`:

```shell
php artisan octane:status
```

<a name="dependency-injection-and-octane"></a>
## Внедрение зависимости и Octane

Поскольку Octane загружает ваше приложение один раз и сохраняет его в памяти при обслуживании запросов, есть несколько предостережений, которые следует учитывать при создании приложения. Например, методы `register` и `boot` поставщиков услуг вашего приложения будут выполняться только один раз при первоначальной загрузке обработчика запросов. При последующих запросах тот же экземпляр приложения будет использоваться повторно.

В свете этого следует проявлять особую осторожность при внедрении контейнера службы приложения или запроса в конструктор любого объекта. Таким образом, этот объект может иметь устаревшую версию контейнера или запроса при последующих запросах.

Octane будет автоматически обрабатывать сброс любого состояния между запросами. Однако Octane не всегда знает, как сбросить глобальное состояние, созданное вашим приложением. Таким образом, вы должны знать, как создать свое приложение, удобное для Octane. Ниже мы обсудим наиболее распространенные ситуации, которые могут вызвать проблемы при использовании Octane.

<a name="container-injection"></a>
### Контейнер для внедрений

В общем случае, следует избегать внедрения контейнера службы приложения или экземпляра HTTP-запроса в конструкторы других объектов. Например, следующая привязка внедряет весь контейнер службы приложения в объект, связанный как синглтон (singleton):

```php
use App\Service;

/**
 * Регистрирует сервисы приложения.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app);
    });
}
```

В этом примере, если экземпляр `Service` предоставляется во время процесса загрузки приложения, контейнер будет внедрен в службу, и этот же контейнер будет удерживаться экземпляром `Service` при последующих запросах. Это **может** не быть проблемой для вашего конкретного приложения; однако это может привести к тому, что в контейнере неожиданно будут отсутствовать привязки, которые были добавлены позже в цикле загрузки или по последующему запросу.

В качестве обходного решения вы можете либо отказаться от регистрации экземпляра как синглтона, либо добавить в инициализацию экземпляра замыкание (closure), которое всегда предоставляет текущий экземпляр своего контейнера:

```php
use App\Service;
use Illuminate\Container\Container;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app);
});

$this->app->singleton(Service::class, function () {
    return new Service(fn () => Container::getInstance());
});
```

Глобальный помощник `app` и метод `Container::getInstance()` всегда будут возвращать последнюю версию контейнера приложения.

<a name="request-injection"></a>
### Запрос на внедрение

В общем случае, следует избегать внедрения контейнера службы приложения или экземпляра HTTP-запроса в конструкторы других объектов. Например, следующая привязка внедряет весь экземпляр запроса в объект, который привязан как синглтон (singleton):

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * Регистрирует сервисы приложения.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app['request']);
    });
}
```

В этом примере, если экземпляр `Service` предоставляется во время процесса загрузки приложения, HTTP-запрос будет внедрен в службу, и тот же самый запрос будет удерживаться экземпляром `Service` при последующих запросах. Следовательно, все заголовки, входные данные и данные строки запроса будут неверными, как и все другие данные запроса.

В качестве обходного решения вы можете либо отказаться от регистрации экземпляра как синглтона, либо добавить в инициализацию экземпляра замыкание (closure), которое всегда предоставляет текущий экземпляр запроса. Или наиболее рекомендуемый подход — просто передать конкретную информацию запроса, необходимую вашему объекту, одному из методов объекта во время выполнения:

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app['request']);
});

$this->app->singleton(Service::class, function (Application $app) {
    return new Service(fn () => $app['request']);
});

// Или ...

$service->method($request->input('name'));
```

Глобальный помощник `request` всегда будет возвращать запрос, который приложение в настоящее время обрабатывает, и поэтому его можно безопасно использовать в вашем приложении.

[!WARNING]
> Допускается вводить подсказку типа `Illuminate\Http\Request` по методам вашего контроллера и замыканиям маршрутов.

<a name="configuration-repository-injection"></a>
### Настройка репозитория внедрения

В общем случае, следует избегать внедрения экземпляра репозитория конфигурации в конструкторы других объектов. Например, следующая привязка вводит репозиторий конфигурации в объект, связанный как синглтон:

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * Регистрирует сервисы приложения.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app->make('config'));
    });
}
```

В этом примере, если значения конфигурации изменяются между запросами, эта служба не будет иметь доступа к новым значениям, поскольку это зависит от исходного экземпляра репозитория.

В качестве обходного решения вы можете либо отказаться от регистрации экземпляра как синглтона, либо добавить в инициализацию экземпляра замыкание (closure):

```php
use App\Service;
use Illuminate\Container\Container;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app->make('config'));
});

$this->app->singleton(Service::class, function () {
    return new Service(fn () => Container::getInstance()->make('config'));
});
```

Глобальный `config` всегда будет возвращать последнюю версию репозитория конфигурации и, следовательно, безопасен для использования в вашем приложении.

<a name="managing-memory-leaks"></a>
### Управление утечкой памяти

Помните, что Octane сохраняет ваше приложение в памяти между запросами; поэтому добавление данных в статически поддерживаемый массив приведет к утечке памяти. Например, следующий контроллер имеет утечку памяти, поскольку каждый запрос к приложению будет продолжать добавлять данные в статический массив `$data`:

```php
use App\Service;
use Illuminate\Http\Request;
use Illuminate\Support\Str;

/**
 * Обработка входящего запроса.
 */
public function index(Request $request): array
{
    Service::$data[] = Str::random(10);

    return [
        // ...
    ];
}
```

При создании приложения следует проявлять особую осторожность, чтобы избежать подобных утечек памяти. Рекомендуется контролировать использование памяти вашим приложением во время разработки, чтобы убедиться, что вы не вносите новые утечки памяти в ваше приложение.

<a name="concurrent-tasks"></a>
## Параллельные задачи

> [!WARNING]
> Для этой функции требуется [Swoole](#swoole).

При использовании Swoole вы можете выполнять операции одновременно с помощью легких фоновых задач. Вы можете сделать это, используя метод Octane `concurrently`. Вы можете комбинировать этот метод с деструктуризацией массива PHP для получения результатов каждой операции:

```php
use App\Models\User;
use App\Models\Server;
use Laravel\Octane\Facades\Octane;

[$users, $servers] = Octane::concurrently([
    fn () => User::all(),
    fn () => Server::all(),
]);
```

Параллельные задачи, обрабатываемые Octane, используют "task workers" Swoole и выполняются в рамках совершенно другого процесса, чем входящий запрос. Количество воркеров (workers), доступных для обработки параллельных задач, определяется директивой `--task-worker` в команде `octane:start`:

```shell
php artisan octane:start --workers=4 --task-workers=6
```

При вызове метода `concurrently` не следует предоставлять более 1024 задач из-за ограничений, накладываемых системой задач Swoole.

<a name="ticks-and-intervals"></a>
## Ticks & Intervals

> [!WARNING]
> Для этой функции требуется [Swoole](#swoole).

При использовании Swoole вы можете зарегистрировать операции "tick", которые будут выполняться каждые заданное количество секунд. Вы можете зарегистрировать обратные вызовы "tick" с помощью метода `tick`. Первым аргументом, предоставленным методу, должна быть строка, представляющая имя операции. Второй аргумент должен быть вызываемой функцией, которая будет вызываться через указанный интервал.

В этом примере мы зарегистрируем замыкание, которое будет вызываться каждые 10 секунд. Обычно метод `tick` должен вызываться внутри метода `boot` одного из поставщиков услуг вашего приложения:

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
        ->seconds(10);
```

Используя метод `immediate`, вы можете указать Octane немедленно вызывать обратный вызов тика при первоначальной загрузке сервера Octane и каждые N секунд после этого:

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
        ->seconds(10)
        ->immediate();
```

<a name="the-octane-cache"></a>
## Кеш Octane

> [!WARNING]
> Для этой функции требуется [Swoole](#swoole).

При использовании Swoole вы можете использовать кэш-драйвер Octane, который обеспечивает скорость чтения и записи до 2 миллионов операций в секунду. Таким образом, этот драйвер кэширования является отличным выбором для приложений, которым требуется экстремальная скорость чтения / записи на уровне кэширования.

Этот драйвер кеширования работает на [Swoole tables](https://www.swoole.co.uk/docs/modules/swoole-table). Все данные, хранящиеся в кеше, доступны всем воркерам (workers) на сервере. Однако кэшированные данные будут сброшены при перезапуске сервера:

```php
Cache::store('octane')->put('framework', 'Laravel', 30);
```
> [!NOTE]
> Максимальное количество записей, разрешенных в кэше Octane, может быть определено в файле конфигурации вашего приложения `octane`.

<a name="cache-intervals"></a>
### Интервалы кеширования

В дополнение к типичным методам, предоставляемым системой кеширования Laravel, драйвер кеширования Octane поддерживает кеширование на основе интервалов. Эти кэши автоматически обновляются с заданным интервалом и должны быть зарегистрированы в методе `boot` одного из поставщиков услуг вашего приложения. Например, следующий кеш будет обновляться каждые пять секунд:

```php
use Illuminate\Support\Str;

Cache::store('octane')->interval('random', function () {
    return Str::random(10);
}, seconds: 5);
```

<a name="tables"></a>
## Таблицы

> [!WARNING]
> Для этой функции требуется [Swoole](#swoole).

При использовании Swoole вы можете определять свои собственные произвольные [таблицы Swoole](https://www.swoole.co.uk/docs/modules/swoole-table) и взаимодействовать с ними. Таблицы Swoole обеспечивают исключительную производительность, и к данным в этих таблицах могут получить доступ все воркеры (workers) на сервере. Однако данные в них будут потеряны при перезапуске сервера.

Таблицы должны быть определены в конфигурационном массиве `tables` конфигурационного файла `octane` вашего приложения. Пример таблицы, которая позволяет не более 1000 строк, уже настроен для вас. Максимальный размер строковых столбцов можно настроить, указав размер столбца после типа столбца, как показано ниже:

```php
'tables' => [
    'example:1000' => [
        'name' => 'string:1000',
        'votes' => 'int',
    ],
],
```

Для доступа к таблице вы можете использовать метод `Octane::table`:

```php
use Laravel\Octane\Facades\Octane;

Octane::table('example')->set('uuid', [
    'name' => 'Nuno Maduro',
    'votes' => 1000,
]);

return Octane::table('example')->get('uuid');
```

> [!WARNING]
> Таблицы Swoole поддерживают следующие типы столбцов: `string`, `int` и `float`.
