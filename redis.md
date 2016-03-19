git e25837a6d7ac39d1c864ed6f317a323a4fef447d

---

# Redis

- [Introduction](#introduction)
- [Basic Usage](#basic-usage)
    - [Pipelining Commands](#pipelining-commands)
- [Pub / Sub](#pubsub)

<a name="introduction"></a>
## Introduction

[Redis](http://redis.io) is an open source, advanced key-value store. It is often referred to as a data structure server since keys can contain [strings](http://redis.io/topics/data-types#strings), [hashes](http://redis.io/topics/data-types#hashes), [lists](http://redis.io/topics/data-types#lists), [sets](http://redis.io/topics/data-types#sets), and [sorted sets](http://redis.io/topics/data-types#sorted-sets). Before using Redis with Laravel, you will need to install the `predis/predis` package (~1.0) via Composer.

<a name="configuration"></a>
### Configuration

The Redis configuration for your application is located in the `config/database.php` configuration file. Within this file, you will see a `redis` array containing the Redis servers used by your application:

    'redis' => [

        'cluster' => false,

        'default' => [
            'host'     => '127.0.0.1',
            'port'     => 6379,
            'database' => 0,
        ],

    ],

The default server configuration should suffice for development. However, you are free to modify this array based on your environment. Simply give each Redis server a name, and specify the host and port used by the server.

The `cluster` option will tell the Laravel Redis client to perform client-side sharding across your Redis nodes, allowing you to pool nodes and create a large amount of available RAM. However, note that client-side sharding does not handle failover; therefore, is primarily suited for cached data that is available from another primary data store.

Additionally, you may define an `options` array value in your Redis connection definition, allowing you to specify a set of Predis [client options](https://github.com/nrk/predis/wiki/Client-Options).

If your Redis server requires authentication, you may supply a password by adding a `password` configuration item to your Redis server configuration array.

> **Note:** If you have the Redis PHP extension installed via PECL, you will need to rename the alias for Redis in your `config/app.php` file.

<a name="basic-usage"></a>
## Basic Usage

You may interact with Redis by calling various methods on the `Redis` [facade](/docs/{{version}}/facades). The `Redis` facade supports dynamic methods, meaning you may call any [Redis command](http://redis.io/commands) on the facade and the command will be passed directly to Redis. In this example, we will call the `GET` command on Redis by calling the `get` method on the `Redis` facade:

    <?php

    namespace App\Http\Controllers;

    use Redis;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
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

Of course, as mentioned above, you may call any of the Redis commands on the `Redis` facade. Laravel uses magic methods to pass the commands to the Redis server, so simply pass the arguments the Redis command expects:

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);

Alternatively, you may also pass commands to the server using the `command` method, which accepts the name of the command as its first argument, and an array of values as its second argument:

    $values = Redis::command('lrange', ['name', 5, 10]);

#### Using Multiple Redis Connections

You may get a Redis instance by calling the `Redis::connection` method:

    $redis = Redis::connection();

This will give you an instance of the default Redis server. If you are not using server clustering, you may pass the server name to the `connection` method to get a specific server as defined in your Redis configuration:

    $redis = Redis::connection('other');

<a name="pipelining-commands"></a>
### Pipelining Commands

Pipelining should be used when you need to send many commands to the server in one operation. The `pipeline` method accepts one argument: a `Closure` that receives a Redis instance. You may issue all of your commands to this Redis instance and they will all be executed within a single operation:

    Redis::pipeline(function ($pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });

<a name="pubsub"></a>
## Pub / Sub

Laravel also provides a convenient interface to the Redis `publish` and `subscribe` commands. These Redis commands allow you to listen for messages on a given "channel". You may publish messages to the channel from another application, or even using another programming language, allowing easy communication between applications / processes.

First, let's setup a listener on a channel via Redis using the `subscribe` method. We will place this method call within an [Artisan command](/docs/{{version}}/artisan) since calling the `subscribe` method begins a long-running process:

    <?php

    namespace App\Console\Commands;

    use Redis;
    use Illuminate\Console\Command;

    class RedisSubscribe extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Subscribe to a Redis channel';

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            Redis::subscribe(['test-channel'], function($message) {
                echo $message;
            });
        }
    }

Now, we may publish messages to the channel using the `publish` method:

    Route::get('publish', function () {
        // Route logic...

        Redis::publish('test-channel', json_encode(['foo' => 'bar']));
    });

#### Wildcard Subscriptions

Using the `psubscribe` method, you may subscribe to a wildcard channel, which is useful for catching all messages on all channels. The `$channel` name will be passed as the second argument to the provided callback `Closure`:

    Redis::psubscribe(['*'], function($message, $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function($message, $channel) {
        echo $message;
    });



# Redis

- [Введение](#introduction)
- [Настройка](#configuration)
- [Использование](#usage)
- [Конвейер](#pipelining)

<a name="introduction"></a>
## Введение

[Redis](http://redis.io) - продвинутое хранилище пар ключ/значение. Его часто называют сервисом структур данных, так как ключи могут содержать [строки](http://redis.io/topics/data-types#strings), [хэши](http://redis.io/topics/data-types#hashes), [списки](http://redis.io/topics/data-types#lists), [наборы](http://redis.io/topics/data-types#sets), and [сортированные наборы](http://redis.io/topics/data-types#sorted-sets).

Прежде чем использовать Redis, необходимо установить пакет `predis/predis` версии `~1.0` через Composer.

> **Внимание:** Если у вас установлено расширение Redis через PECL, вам нужно переименовать псевдоним в файле `config/app.php`.

<a name="configuration"></a>
## Настройка

Настройки вашего подключения к Redis хранятся в файле `app/config/database.php`. В нём вы найдёте массив `redis`, содержащий список серверов, используемых приложением:

	'redis' => [

		'cluster' => true,

		'default' => ['host' => '127.0.0.1', 'port' => 6379],

	],

Если у вас Redis установлен на других портах, или есть несколько redis-серверов, дайте имя каждому подключению к Redis и укажите серверные хост и порт.

Параметр `cluster` ообщает клиенту Redis Laravel, что нужно выполнить фрагментацию узлов Redis (client-side sharding), что позволит вам обращаться к ним и увеличить доступную RAM. Однако заметьте, что фрагментация не справляется с падениями, поэтому она в основном используется для кэшировании данных, которые доступны из основного источника.

Если ваш сервер Redis требует авторизацию, вы можете указать пароль, добавив к параметрам подключения пару ключ/значение `password`.

<a name="usage"></a>
## Использование

Вы можете получить экземпляр Redis методом `Redis::connection`:

	$redis = Redis::connection();

Так вы получите экземпляр подключения по умолчанию. Если вы не используете фрагментацию, то можно передать этому методу имя сервера для получения конкретного подключения, как оно определено в файле настроек.

	$redis = Redis::connection('other');

Как только у вас есть экземпляр клиента Redis вы можете выполнить любую [команду Redis](http://redis.io/commands).Laravel использует магические методы PHP для передачи команд на сервер:

	$redis->set('name', 'Тейлор');

	$name = $redis->get('name');

	$values = $redis->lrange('names', 5, 10);

Как вы видите, параметры команд просто передаются магическому методу. Конечно, вам не обязательно использовать эти методы - вы можете передавать команды на сервер методом `command`:

	$values = $redis->command('lrange', array(5, 10));

Если у вас в конфиге определено одно дефолтное подключение, то вы можете использовать статические методы:

	Redis::set('name', 'Тейлор');

	$name = Redis::get('name');

	$values = Redis::lrange('names', 5, 10);

> **Примечание:** Laravel поставляется с драйверами Redis для [кэширования](/docs/{{version}}/cache) и [сессий](/docs/{{version}}/session).

<a name="pipelining"></a>
## Конвейер

Конвейер (pipelining) должен использоваться, когда вы отправляете много команд на сервер за одну операцию. Для начала выполните команду `pipeline`:

#### Отправка конвейером набора команд на сервер

	Redis::pipeline(function($pipe)
	{
		for ($i = 0; $i < 1000; $i++)
		{
			$pipe->set("key:$i", $i);
		}
	});
