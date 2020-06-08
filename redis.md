git eb78a8b30e07a2f39663e859e8d00afb29788dc0

---

# Redis

- [Введение](#introduction)
    - [Настройка](#configuration)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [Взаимодействие с Redis](#interacting-with-redis)
    - [Конвейер команд](#pipelining-commands)
- [Pub / Sub](#pubsub)

<a name="introduction"></a>
## Введение

[Redis](http://redis.io) - хранилище пар ключ/значение с расширенным функционалом и открытым исходным кодом. Его часто называют сервисом структур данных, так как ключи могут содержать [строки](http://redis.io/topics/data-types#strings), [хэши](http://redis.io/topics/data-types#hashes), [списки](http://redis.io/topics/data-types#lists), [наборы](http://redis.io/topics/data-types#sets) и [сортированные наборы](http://redis.io/topics/data-types#sorted-sets).

Чтобы начать использовать Redis с Laravel, необходимо либо установить пакет `predis/predis` с помощью Composer:

    composer require predis/predis

Либо, вы можете установить расширение для PHP [PhpRedis](https://github.com/phpredis/phpredis) через PECL. Расширение сложнее установить, но оно даёт большую производительность для приложений, которые активно используют Redis.

<a name="configuration"></a>
### Настройка

Настройки вашего подключения к Redis хранятся в конфиге `config/database.php`. В нём вы найдёте массив `redis`, содержащий список серверов, используемых приложением:

    'redis' => [

        'client' => 'predis',

        'default' => [
            'host' => env('REDIS_HOST', 'localhost'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => 0,
        ],

    ],

Значения по умолчанию должны подойти для разработки. Однако вы свободно можете изменять этот массив в зависимости от своей среды. У каждого сервера Redis, определённого в конфиге, должны быть имя, хост и порт.

#### Настройка кластеров

Если ваше приложение использует кластер серверов Redis, вы должны задать эти кластеры в ключе `clusters` своей настройки Redis:

    'redis' => [

        'client' => 'predis',

        'clusters' => [
            'default' => [
                [
                    'host' => env('REDIS_HOST', 'localhost'),
                    'password' => env('REDIS_PASSWORD', null),
                    'port' => env('REDIS_PORT', 6379),
                    'database' => 0,
                ],
            ],
        ],

    ],

По умолчанию кластеры будут выполнять фрагментацию узлов Redis (client-side sharding), что позволит вам обращаться к ним и увеличить доступную RAM. Однако заметьте, что фрагментация не справляется с падениями, поэтому она в основном используется для кэшировании данных, которые доступны из основного источника. Если вы хотите использовать нативную кластеризацию Redis, то вам нужно указать ключ `options` в своей настройке Redis:

    'redis' => [

        'client' => 'predis',

        'options' => [
            'cluster' => 'redis',
        ],

        'clusters' => [
            // ...
        ],

    ],

<a name="predis"></a>
### Predis

Помимо стандартных опций настройки сервера `host`, `port`, `database` и `password` Predis поддерживает дополнительные [параметры подключения](https://github.com/nrk/predis/wiki/Connection-Parameters) , которые можно определить для каждого из ваших серверов Redis. Чтобы использовать эти дополнительные опции, просто добавьте их в конфиг вашего сервера Redis - `config/database.php`:

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_write_timeout' => 60,
    ],

<a name="phpredis"></a>
### PhpRedis

> {note} Если ваше расширение Redis установлено через PECL, вам нужно переименовать псевдоним для Redis в файле `config/app.php`.

Чтобы использовать расширение PhpRedis, вы должны задать опции `client` в конфигурации Redis значение `phpredis`. Эта опция находится в файле `config/database.php`:

    'redis' => [

        'client' => 'phpredis',

        // Остальные настройки Redis...
    ],

Помимо стандартных опций настройки сервера `host`, `port`, `database` и `password` PhpRedis поддерживает следующие дополнительные параметры подключения: `persistent`, `prefix`, `read_timeout` и `timeout`. Вы можете добавить любые из этих опций в настройки своего сервера Redis в конфиге `config/database.php`:

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_timeout' => 60,
    ],

<a name="interacting-with-redis"></a>
## Взаимодействие с Redis

Вы можете взаимодействовать с Redis, вызывая различные методы [фасада](/docs/{{version}}/facades) `Redis`. Фасад `Redis` поддерживает динамические методы, а значит вы можете вызвать любую [Redis-команду](http://redis.io/commands) на фасаде, и команда будет передана прямо в Redis. В этом примере мы вызовем Redis-команду `GET` с помощью вызова метода `get` фасада `Redis`:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Redis;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Показать профиль данного пользователя.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Redis::get('user:profile:'.$id);

            return view('user.profile', ['user' => $user]);
        }
    }

Как уже было сказано, вы можете вызывать любые Redis-команды фасада `Redis`. использует магические методы PHP для передачи команд на сервер Redis, поэтому просто передайте необходимые аргументы Redis-команде:

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);

В качестве альтернативы вы можете передавать команды на сервер методом `command`, который принимает первым аргументом имя команды, а вторым — массив значений:

    $values = Redis::command('lrange', ['name', 5, 10]);

#### Использование нескольких подключений Redis

Вы можете получить экземпляр Redis методом `Redis::connection`:

    $redis = Redis::connection();

Так вы получите экземпляр подключения по умолчанию. Вы также можете передать имя подключения или кластера методу `connection`, чтобы получить определенный сервер или кластер, как определено в вашем файле настроек Redis:

    $redis = Redis::connection('my-connection');

<a name="pipelining-commands"></a>
### Конвейер команд

Конвейер должен использоваться, когда вы отправляете много команд на сервер за одну операцию. Метод  `pipeline` принимает один аргумент - функцию-замыкание, которая получает экземпляр Redis. Вы можете выполнить все ваши команды на этом экземпляре Redis, и все они будут выполнены в рамках одной операции:

    Redis::pipeline(function ($pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });

<a name="pubsub"></a>
## Pub / Sub

Laravel предоставляет удобный интерфейс к Redis-командам `publish` и `subscribe`. Эти команды позволяют прослушивать сообщения на заданном "канале". Вы можете публиковать сообщения в канал из другого приложения или даже при помощи другого языка программирования, что обеспечивает простую связь между приложениями и процессами.

Сначала давайте настроим слушатель канала с помощью метода `subscribe`. Мы поместим вызов этого метода в [Artisan-команду](/docs/{{version}}/artisan), так как вызов метода `subscribe` запускает длительный процесс:

    <?php

    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminate\Support\Facades\Redis;

    class RedisSubscribe extends Command
    {
        /**
         * Название и сигнатура команды консоли.
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';

        /**
         * Описание команды консоли.
         *
         * @var string
         */
        protected $description = 'Subscribe to a Redis channel';

        /**
         * Выполнение команды консоли.
         *
         * @return mixed
         */
        public function handle()
        {
            Redis::subscribe(['test-channel'], function ($message) {
                echo $message;
            });
        }
    }

Теперь мы можем публиковать сообщения в канал методом `publish`:

    Route::get('publish', function () {
        // Route logic...

        Redis::publish('test-channel', json_encode(['foo' => 'bar']));
    });

#### Подписка по маске

С помощью метода `psubscribe` вы можете подписаться на канал по маске, это может быть полезно для отлова всех сообщений на всех каналах. Название канала `$channel` будет передано вторым аргументом в предоставляемую функцию-замыкание:

    Redis::psubscribe(['*'], function ($message, $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function ($message, $channel) {
        echo $message;
    });
