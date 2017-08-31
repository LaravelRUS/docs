git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Кэш

- [Настройка](#configuration)
    - [Требования драйвера](#driver-prerequisites)
- [Использование кэша](#cache-usage)
    - [Получение экземпляра кэша](#obtaining-a-cache-instance)
    - [Получение элементов из кэша](#retrieving-items-from-the-cache)
    - [Сохранение элементов в кэш](#storing-items-in-the-cache)
    - [Удаление элементов из кэша](#removing-items-from-the-cache)
    - [Хелпер Cache](#the-cache-helper)
- [Теги кэша](#cache-tags)
    - [Сохранение элементов кэша с тегами](#storing-tagged-cache-items)
    - [Обращение к элементам кэша с тегами](#accessing-tagged-cache-items)
    - [Удаление элементов кэша с тегами](#removing-tagged-cache-items)
- [Добавление своих драйверов кэша](#adding-custom-cache-drivers)
    - [Написание драйвера](#writing-the-driver)
    - [Регистрация драйвера](#registering-the-driver)
- [События](#events)

<a name="configuration"></a>
## Настройка

Laravel предоставляет выразительный, универсальный API для различных систем кэширования. Настройки кэша находятся в файле `config/cache.php`. Здесь вы можете указать драйвер, используемый по умолчанию в вашем приложении. Laravel изначально поддерживает многие популярные системы, такие как [Memcached](https://memcached.org) и [Redis](http://redis.io).

Этот файл также содержит множество других настроек, которые в нём же документированы, поэтому обязательно ознакомьтесь с ними. По умолчанию, Laravel настроен для использования драйвера `file`, который хранит сериализованные объекты кэша в файловой системе. Для больших приложений рекомендуется использование более надёжных драйверов, таких как Memcached или Redis. Вы можете настроить даже несколько конфигураций кэширования для одного драйвера.

<a name="driver-prerequisites"></a>
### Требования драйвера

#### Database

Перед использованием драйвера кэша `database` вам нужно создать таблицу для хранения элементов кэша. Ниже приведён пример объявления структуры `Schema`:

    Schema::create('cache', function ($table) {
        $table->string('key')->unique();
        $table->text('value');
        $table->integer('expiration');
    });

> {tip} Также вы можете использовать Artisan-команду `php artisan cache:table` для генерации миграции с правильной схемой.

#### Memcached

Для использования системы кэширования Memcached необходим установленный [пакет Memcached PECL](https://pecl.php.net/package/memcached). Вы можете перечислить все свои сервера Memcached в конфиге `config/cache.php`:

    'memcached' => [
        [
            'host' => '127.0.0.1',
            'port' => 11211,
            'weight' => 100
        ],
    ],

Вы также можете задать параметр `host` для пути UNIX-сокета. В этом случае в параметр `port` следует записать значение `0`:

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],

#### Redis

Перед тем, как использовать систему Redis необходимо установить пакет `predis/predis` (~1.0) с помощью Composer или установить расширение PhpRedis PHP через PECL.

Для получения дополнительной информации по настройке Redis, загляните на [соответствующую страницу в документации Laravel](/docs/{{version}}/redis#configuration).

<a name="cache-usage"></a>
## Использование кэша

<a name="obtaining-a-cache-instance"></a>
### Получение экземпляра кэша

[Контракты](/docs/{{version}}/contracts) `Illuminate\Contracts\Cache\Factory` и `Illuminate\Contracts\Cache\Repository` предоставляют доступ к службам кэша Laravel. Контракт `Factory` предоставляет доступ ко всем драйверам кэша, определённым для вашего приложения. А контракт `Repository` обычно является реализацией драйвера кэша по умолчанию для вашего приложения, который задан в файле настроек `cache`.

А также вы можете использовать фасад `Cache`, который мы будем использовать в данной статье. Фасад `Cache` обеспечивает удобный и лаконичный способ доступа к лежащим в его основе реализациям контрактов кэша Laravel:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Вывод списка всех пользователей приложения.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

#### Доступ к нескольким хранилищам кэша

Используя фасад `Cache` можно обращаться к разным хранилищам кэша с помощью метода `store`. Передаваемый в метод ключ должен соответствовать одному из хранилищ, перечисленных в массиве `stores` в вашем файле настроек `cache`:

    $value = Cache::store('file')->get('foo');

    Cache::store('redis')->put('bar', 'baz', 10);

<a name="retrieving-items-from-the-cache"></a>
### Получение элементов из кэша

Для получения элементов из кэша используется метод `get` фасада `Cache`. Если элемента в кэше не существует, будет возвращён `null`. При желании вы можете указать другое возвращаемое значение, передав его вторым аргументом метода `get`:

    $value = Cache::get('key');

    $value = Cache::get('key', 'default');

А также вы можете передать замыкание в качестве значения по умолчанию. Тогда, если элемента не существует, будет возвращён результат замыкания. С помощью замыкания вы можете настроить получение значений по умолчанию из базы данных или внешнего сервиса:

    $value = Cache::get('key', function () {
        return DB::table(...)->get();
    });

#### Проверка существования элемента

Для определения существования элемента в кэше используется метод `has`. Он вернёт `false`, если значение равно `null` или `false`:

    if (Cache::has('key')) {
        //
    }

#### Увеличение / уменьшение значений

Для изменения числовых элементов кэша используются методы `increment` и `decrement`. Оба они могут принимать второй необязательный аргумент, определяющий значение, на которое нужно изменить значение элемента:

    Cache::increment('key');
    Cache::increment('key', $amount);
    Cache::decrement('key');
    Cache::decrement('key', $amount);

#### Получение и хранение

Иногда необходимо получить элемент из кэша и при этом сохранить значение по умолчанию, если запрашиваемый элемент не существует. Например, когда необходимо получить всех пользователей из кэша, а если они не существуют, получить их из базы данных и добавить в кэш. Это можно сделать с помощью метода `Cache::remember`:

    $value = Cache::remember('users', $minutes, function () {
        return DB::table('users')->get();
    });

Если элемента нет в кэше, будет выполнено переданное в метод `remember` замыкание, а его результат будет помещён в кэш.

#### Получение и удаление

При необходимости получить элемент и удалить его из кэша используется метод `pull`. Как и метод `get`, данный метод вернёт `null`, если элемента не существует:

    $value = Cache::pull('key');

<a name="storing-items-in-the-cache"></a>
### Сохранение элементов в кэш

Для помещения элементов в кэш используется метод `put` фасада `Cache`. При помещении элемента в кэш необходимо указать, сколько минут его необходимо хранить:

    Cache::put('key', 'value', $minutes);

Вместо указания количества минут можно передать экземпляр PHP-типа `DateTime` для указания времени истечения срока хранения:

    $expiresAt = Carbon::now()->addMinutes(10);

    Cache::put('key', 'value', $expiresAt);

#### Сохранить, если такого нет

Метод `add` просто добавит элемент в кэш, если его там ещё нет. Метод вернёт `true`, если элемент действительно будет добавлен в кэш. Иначе — вернет `false`:

    Cache::add('key', 'value', $minutes);

#### Сохранить элементы навсегда

Для бесконечного хранения элемента кэша используется метод `forever`. Поскольку срок хранения таких элементов не истечёт никогда, они должны удаляться из кэша вручную с помощью метода `forget`:

    Cache::forever('key', 'value');

> {tip} При использовании драйвера Memcached элементы, сохранённые "навсегда", могут быть удалены, когда размер кэша достигнет своего лимита.

<a name="removing-items-from-the-cache"></a>
### Удаление элементов из кэша

Можно удалить элементы из кэша с помощью метода `forget`:

    Cache::forget('key');

Вы можете очистить весь кэш методом `flush`:

    Cache::flush();

> {note} Очистка не поддерживает префикс кэша и удаляет из него все элементы. Учитывайте это при очистке кэша, которым пользуются другие приложения.

<a name="the-cache-helper"></a>
### Хелпер Cache

Помимо использования фасада `Cache` или [контракта кэша](/docs/{{version}}/contracts), вы также можете использовать глобальную функция `cache` для получения данных из кэша и помещения данных в него. При вызове функции `cache` с одним строковым аргументом она вернёт значение данного ключа:

    $value = cache('key');

Если вы передадите в функцию массив пар ключ/значение и время хранения, она сохранит значения в кэше на указанное время:

    cache(['key' => 'value'], $minutes);

    cache(['key' => 'value'], Carbon::now()->addSeconds(10));

> {tip} При тестировании вызова глобальной функции `cache` вы можете использовать метод `Cache::shouldReceive`, как и при [тестировании фасада](/docs/{{version}}/mocking#mocking-facades).

<a name="cache-tags"></a>
## Теги кэша

> {note} Теги кэша не поддерживаются драйверами `file` или `database`. Кроме того, при использовании нескольких тегов для кэшей, хранящихся "навсегда", лучшая производительность будет достигнута при использовании такого драйвера как `memcached`, который автоматически зачищает устаревшие записи.

<a name="storing-tagged-cache-items"></a>
### Сохранение элементов кэша с тегами

Теги кэша позволяют отмечать связанные элементы в кэше, и затем сбрасывать все элементы, которые были отмеченны одним тегом. Вы можете обращаться к кэшу с тегами, передавая упорядоченный массив имён тегов. Например, давайте обратимся к кэшу с тегами и поместим в него значение методом `put`:

    Cache::tags(['people', 'artists'])->put('John', $john, $minutes);

    Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);

<a name="accessing-tagged-cache-items"></a>
### Обращение к элементам кэша с тегами

Для получения элемента с тегом передайте тот же упорядоченный список тегов в метод `tags`, а затем вызовите метод `get` с тем ключом, который необходимо получить:

    $john = Cache::tags(['people', 'artists'])->get('John');

    $anne = Cache::tags(['people', 'authors'])->get('Anne');

<a name="removing-tagged-cache-items"></a>
### Удаление элементов кэша с тегами

Вы можете очистить все элементы с заданным тегом или списком тегов. Например, следующий код удалит все элементы, отмеченные либо тегом `people`, либо `authors`, либо и тем и другим. Поэтому и `Anne`, и `John` будут удалены из кэша:

    Cache::tags(['people', 'authors'])->flush();

В отличие от предыдущего, следующий код удалит только те элементы, которые отмечены тегом `authors`, поэтому `Anne` будет удалён, а `John` - нет:

    Cache::tags('authors')->flush();

<a name="adding-custom-cache-drivers"></a>
## Добавление своих драйверов кэша

<a name="writing-the-driver"></a>
### Написание драйвера

Чтобы создать свой драйвер кэша, нам надо сначала реализовать [контракт](/docs/{{version}}/contracts) `Illuminate\Contracts\Cache\Store`. Итак, наша реализация кэша MongoDB будет выглядеть примерно так:

    <?php

    namespace App\Extensions;

    use Illuminate\Contracts\Cache\Store;

    class MongoStore implements Store
    {
        public function get($key) {}
        public function many(array $keys);
        public function put($key, $value, $minutes) {}
        public function putMany(array $values, $minutes);
        public function increment($key, $value = 1) {}
        public function decrement($key, $value = 1) {}
        public function forever($key, $value) {}
        public function forget($key) {}
        public function flush() {}
        public function getPrefix() {}
    }

Нам просто надо реализовать каждый из этих методов, используя соединение MongoDB. Примеры реализации каждого из этих методов можно найти в `Illuminate\Cache\MemcachedStore` в исходном коде фреймворка. Когда наша реализация готова, мы можем завершить регистрацию нашего драйвера.

    Cache::extend('mongo', function ($app) {
        return Cache::repository(new MongoStore);
    });

> {tip} Если вы задумались о том, где разместить код вашего драйвера, то можете создать пространство имён `Extensions` в директории `app`. Однако, не забывайте, что в Laravel нет жёсткой структуры приложения, и вы можете организовать его как пожелаете.

<a name="registering-the-driver"></a>
### Регистрация драйвера

Чтобы зарегистрировать свой драйвер кэша в Laravel, мы будем использовать метод `extend` фасада `Cache`. Вызов `Cache::extend` можно делать из метода `boot` сервис-провайдера по умолчанию `App\Providers\AppServiceProvider`, который есть в каждом Laravel-приложении. Или вы можете создать свой сервис-провайдер для размещения расширения — только не забудьте зарегистрировать его в массиве провайдеров `config/app.php`:

    <?php

    namespace App\Providers;

    use App\Extensions\MongoStore;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\ServiceProvider;

    class CacheServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Cache::extend('mongo', function ($app) {
                return Cache::repository(new MongoStore);
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Первый аргумент метода `extend` — имя драйвера. Оно будет соответствовать параметру `driver` в вашем конфиге `config/cache.php`. Второй аргумент — замыкание, которое должно возвращать экземпляр `Illuminate\Cache\Repository`. В замыкание будет передан экземпляр `$app`, который является экземпляром [сервис-контейнера](/docs/{{version}}/container).

Когда ваше расширение зарегистрировано, просто укажите его имя в качестве значения параметра `driver` в конфиге `config/cache.php`.

<a name="events"></a>
## События

Для выполнения какого-либо кода при каждой операции с кэшем вы можете прослушивать [события](/docs/{{version}}/events), инициируемые кэшем. Обычно вам необходимо поместить эти слушатели событий в ваш `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Cache\Events\CacheHit' => [
            'App\Listeners\LogCacheHit',
        ],

        'Illuminate\Cache\Events\CacheMissed' => [
            'App\Listeners\LogCacheMissed',
        ],

        'Illuminate\Cache\Events\KeyForgotten' => [
            'App\Listeners\LogKeyForgotten',
        ],

        'Illuminate\Cache\Events\KeyWritten' => [
            'App\Listeners\LogKeyWritten',
        ],
    ];
