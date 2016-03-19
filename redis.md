git e25837a6d7ac39d1c864ed6f317a323a4fef447d

---

# Redis

- [Введение](#introduction)
- [Использование](#basic-usage)
    - [Команды конвейера](#pipelining-commands)
- [Pub / Sub](#pubsub)

<a name="introduction"></a>
## Введение

[Redis](http://redis.io) - продвинутое хранилище пар ключ/значение. Его часто называют сервисом структур данных, так как ключи могут содержать [строки](http://redis.io/topics/data-types#strings), [хэши](http://redis.io/topics/data-types#hashes), [списки](http://redis.io/topics/data-types#lists), [наборы](http://redis.io/topics/data-types#sets), and [сортированные наборы](http://redis.io/topics/data-types#sorted-sets). Прежде чем использовать Redis вместе с Laravel, вам потребуется установить пакет `predis/predis`  (~1.0) через Composer.

<a name="configuration"></a>
### Настройка

Настройки вашего подключения к Redis хранятся в файле `app/config/database.php`. В нём вы найдёте массив `redis`, содержащий список серверов, используемых приложением:

    'redis' => [

        'cluster' => false,

        'default' => [
            'host'     => '127.0.0.1',
            'port'     => 6379,
            'database' => 0,
        ],

    ],

Конфигурация установленная по умолчанию, должна быть достаточной для разработки. Тем не менее, вы можете изменять настройки в этом массив на вашем сервере разработки. Если у вас несколько серверов, дайте имя каждому подключению к Redis и укажите серверные хост и порт.

Параметр `cluster` ообщает клиенту Redis Laravel, что нужно выполнить фрагментацию узлов Redis (client-side sharding), что позволит вам обращаться к ним и увеличить доступную RAM. Однако заметьте, что фрагментация не справляется с падениями, поэтому она в основном используется для кэшировании данных, которые доступны из основного источника.

Дополнительно, вы можете определить массив настроек `options` в общем массиве настроек подключения вашего Redis, что позволяет определить настройки пакета Predis [документация клиентских настроек](https://github.com/nrk/predis/wiki/Client-Options).

Если ваш сервер Redis требует авторизацию, вы можете указать пароль, добавив к параметрам подключения пару ключ/значение.

> **Внимание:** Если у вас установлено расширение Redis через PECL, вам нужно переименовать псевдоним в файле `config/app.php`.

<a name="basic-usage"></a>
## Использование

Вы можете взаимодействовать с Redis с помощью различных методов используя [facade](/docs/{{version}}/facades) `Redis`. Фассад `Redis` поддерживает динамические методы, что означает что любые [команды Redis](http://redis.io/commands) этого фассада и все комманды будут напрямую отправлены к Redis. К примеру мы вызовем команду `GET` Redis с помощью метода `get` в фассаде `Redis`:

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

Конечно же, как было озвучено ранее, вы можете вызвать любую из команд Redis используя фассад `Redis`. Laravel использует магические методы для передачи команд к серверу Redis, так что просто передайте аргументы команды Redis примерно так:

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);

В качестве альтернативы, вы можете передать команды серверу Redis используя метод `сommand`, который принимает имя команды в качестве первого аргумента, и массив значения в качестве второго аргумента:

    $values = Redis::command('lrange', ['name', 5, 10]);

#### Использование несколько подключений Redis

Вы можете получить экземпляр Redis с помощью вызова метода `Redis::connection`:

    $redis = Redis::connection();

Таким образом вы получите экземпляр сервера, установленного по умолчанию. Если вы не используете кластеризацию, вы можете указать имя сервера Redis в аргументе метода `connection`, для получения экземпляра специфичного сервера. Название сервера должно совпадать с названием, указанным в ваших настройках подключения с Redis:

    $redis = Redis::connection('other');

<a name="pipelining-commands"></a>
### Команды конвейера

Конвейер следует использовать тогда, когда вам нужно послать много команд серверу в одной операции. Метод `pipeline` принимает 1 аргумент: замыкание `Closure`, которое возвращает экземпляр Redis. Вы можете выполнять все ваши команды к этому экземпляру Redis, и оны будут выполнены в одной операции:

    Redis::pipeline(function ($pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });

<a name="pubsub"></a>
## Pub / Sub

Laravel поддерживает удобный интерфейс `publish` и `subscribe` команд. Эти команды позволят вам слушать сообщения конкретного канала. Вы можете публиковать сообщения на канал с помощью другого приложения, или даже использовать другой язык программирования, позволяющий легко связываться между приложниями / процессами.

В начале давайте настроим слушатель (listener) канала с помощью Redis используя метод `subscribe`. Мы поместим вызов этого метода в  [Artisan команду](/docs/{{version}}/artisan), так как вызов метода `subscribe` запускает длительный процесс:

    <?php

    namespace App\Console\Commands;

    use Redis;
    use Illuminate\Console\Command;

    class RedisSubscribe extends Command
    {
        /**
         * Это название и сигнатура консольной команды.
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';

        /**
         * Описание консольной команды.
         *
         * @var string
         */
        protected $description = 'Подписка на канал Redis';

        /**
         * Выполнение команды.
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

Теперь мы можем публиковать сообщения на канал используя метод `publish`:

    Route::get('publish', function () {
        // Логика роута...

        Redis::publish('test-channel', json_encode(['foo' => 'bar']));
    });


#### Подстановки в подписках (Wildcards)

Используйте метод, для подписки на канал с шаблоном подстановки (Wildcard), что очень удобно для приема всех сообщений на всех каналах. Значение имени `$channel` будет помещено во второй аргумент замыкания`Closure`

    Redis::psubscribe(['*'], function($message, $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function($message, $channel) {
        echo $message;
    });
