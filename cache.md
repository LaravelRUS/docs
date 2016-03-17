git c4c1d7973767dbd19d4095b1b88c2dfe01ef367a

---

# Кэш

- [Настройка](#configuration)
- [Использование кэша](#cache-usage)
    - [Получение экземпляра класса кэша](#obtaining-a-cache-instance)
    - [Чтение элементов из кэша](#retrieving-items-from-the-cache)
    - [Сохранение элементов в кэш](#storing-items-in-the-cache)
    - [Удаление элементов из кэша](#removing-items-from-the-cache)
- [Тэги кэша](#cache-tags)
    - [Сохранение элементов в кэш по тегам](#storing-tagged-cache-items)
    - [Получение элементов из кэша с тэгами](#accessing-tagged-cache-items)
- [Добавление пользовательских драйверов для кэша](#adding-custom-cache-drivers)
- [События](#events)

<a name="configuration"></a>
## Настройка

Laravel предоставляет унифицированное API для различных систем кэширования. Настройки кэша содержатся в файле `config/cache.php`. Там вы можете указать драйвер, который будет использоваться для кэширования. Laravel "из коробки" поддерживает многие популярные системы, такие как [Memcached](http://memcached.org) и [Redis](http://redis.io).

Этот файл также содержит множество других настроек, которые в нём же документированы, поэтому обязательно ознакомьтесь с ними. По умолчанию Laravel настроен для использования драйвера `File`, который хранит сериализованные объекты кэша в файловой системе. Для больших приложений рекомендуется использование систем кэширования в памяти - таких как Memcached или APC. Вы так же можете создать несколько разных конфигураций для одного драйвера.

### Предварительная подготовка кэша

#### База данных

Перед использовании драйвера `database` вам понадобится создать таблицу для хранения элементов кэша. Ниже приведён пример `Schema` её структуры в виде миграции Laravel:

    Schema::create('cache', function($table) {
        $table->string('key')->unique();
        $table->text('value');
        $table->integer('expiration');
    });

Вы также можете использовать Artisan команду `php artisan cache:table`, чтобы сгенерировать миграцию с правильной схемой.

#### Memcached

Для использования Memcached кэша необходимо установить [Memcached PECL package](http://pecl.php.net/package/memcached).

По умолчанию [настройка](#configuration) TCP/IP используется на базе [Memcached::addServer](http://php.net/manual/en/memcached.addserver.php):

    'memcached' => [
        [
            'host' => '127.0.0.1',
            'port' => 11211,
            'weight' => 100
        ],
    ],

Вы также можете коннектиться не по IP, а через UNIX socket. В этом случае поставьте `port` в '0':

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],

#### Redis

Прежде чем использовать Redis, необходимо установить пакет `predis/predis` версии (~1.0) с помощью Composer. 

Для получения дополнительной информации о настройке Redis, обратитесь к соответсвующему [разделу документации Laravel](/docs/{{version}}/redis#configuration).

<a name="cache-usage"></a>
## Использование кэша

<a name="obtaining-a-cache-instance"></a>
### Получение экземпляра класса Cache

 `Illuminate\Contracts\Cache\Factory` и `Illuminate\Contracts\Cache\Repository` [contracts](/docs/{{version}}/contracts) обеспечивают доступ к сервисам кэша в Laravel. Интерфейс `Factory` обеспечивает доступ ко всем драйверам кэша вашего приложения. Интерфейс `Repository` обычно устанавливает по умолчанию драйвер кеша из вашего конфигурационного файла `cache`, для вашего приложения.

Тем не менее, вы можете также использовать `Cache` фасад, который мы будем использовать в этой документации. Фасад `Cache` обеспечивает удобный и лаконичный доступ к выполнению контрактов кэша Laravel.

Например, давайте импортировать фасад `Cache` в контроллер:

    <?php

    namespace App\Http\Controllers;

    use Cache;

    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

#### Доступ к определённому хранилищу

Используя фасад `Cache`, вы получаете доступ к различным хранилищам кэша с помощью метода `store`. Ключ передается методу `store`, который должен соответствовать одному из хранилищ, перечисленных в  списке массива `stores` в вашем конфигурационном файле `cache`:

    $value = Cache::store('file')->get('foo');

    Cache::store('redis')->put('bar', 'baz', 10);

<a name="retrieving-items-from-the-cache"></a>
### Чтение элементов из кэша

Метод `get` фасада `Cache` используется для получения элементов из кэша. Если элемент не существует в кэше, то будет возвращен `null`. Вы также можете передать второй аргумент методу `get` с указанием значения по умолчанию, который вы хотите чтобы был возвращен, если элемент не существует:

    $value = Cache::get('key');

    $value = Cache::get('key', 'default');

Вы даже можете передать `Closure` в качестве значения по умолчанию. Результат `Closure` будет возвращен, если указанный элемент не существует в кэше. Передача `Closure` позволяет отложить извлечение значения по умолчанию из базы данных или другого внешнего сервиса:

    $value = Cache::get('key', function() {
        return DB::table(...)->get();
    });

#### Проверка существования элемента

Метод `has` может быть использован для определения, существует ли элемент в кэше:

    if (Cache::has('key')) {
        //
    }

#### Увеличение / уменьшение значений

Методы `increment` и `decrement` могут быть использованы для корректировки числовых значений элементов в кэше. Оба этих метода принимают необязательный второй аргумент, который указывает на сколько необходимо увеличить или уменьшить значение элемента:

    Cache::increment('key');

    Cache::increment('key', $amount);

    Cache::decrement('key');

    Cache::decrement('key', $amount);

#### Получить или обновить

Иногда бывает нужно не просто получить элемент из кэша, но и сохранить значение по умолчанию, если запрошенного элемента не существует. Например, вы хотите получить всех пользователей из кэша и если их нету, то извлечь их из базы данных и добавить обратно в кэш. Вы можете сделать это с помощью метода `Cache::remember`:

    $value = Cache::remember('users', $minutes, function() {
        return DB::table('users')->get();
    });

Если элемент не существует в кэше, то `Closure` передается в метод `remember`, кторый будет выполнен и его результат будет помещен в кэш.

Вы также можете комбинировать методы `remember` и forever`:

    $value = Cache::rememberForever('users', function() {
        return DB::table('users')->get();
    });

#### Получить и удалить

Если вам необходимо получить элемент из кэша, а затем удалить его, то вы можете использовать метод `pull`. Подобно `get` методу, будет возвращен `null`, если элемент не существует в кэше:

    $value = Cache::pull('key');

<a name="storing-items-in-the-cache"></a>
### Сохранение элементов в Кэш

Вы можете использовать метод `put` в фасаде `Cache` для сохранения элемента в кэше. При сохранении элементов в кэше, вам нужно будет указать количество минут, на сколько необходимо за кэшировать значения:

    Cache::put('key', 'value', $minutes);

Вместо простой передачи количества минут, вы можете также передать PHP `DateTime` экземпляр класса, представляющий время истечения срока действия кэшированного элемента:

    $expiresAt = Carbon::now()->addMinutes(10);

    Cache::put('key', 'value', $expiresAt);

Метод `add` добавит элемент в хранилище кэша, если такого элемента не существует. Метод вернёт `true`, если элемент на самом деле будет добавлен в кэш. В противном случае, этот метод вернёт `false`:

    Cache::add('key', 'value', $minutes);

Метод `forever` может быть использован для записи элемента в кэш, на постоянное хранение. Такие значения должны быть удалены вручную из кэша с помощью метода `forget`:

    Cache::forever('key', 'value');

<a name="removing-items-from-the-cache"></a>
### Удаление элементов из кэша

Вы можете удалить элементы из кэша с помощью фасада `Cache` и метода `forget`:

    Cache::forget('key');

Вы можете очистить весь кэш, используя метод `flush`:

    Cache::flush();

Полная очистка кэша ** не ** соблюдает префиксы кэша и удалит все записи из кэша. Учитывайте это внимательно при очистке кэша, который совместно используется другими приложениями.

<a name="cache-tags"></a>
## Тэги кэша

> ** Примечание: **тэги кэша не поддерживаются драйверами `file` или` database`. Кроме того, если вы используете мультитэги для "вечных" элементов кэша (сохраненных как "forever"), наиболее подходящим с точки зрения производительности будет драйвер типа `memcached`, который автоматически очищает устаревшие записи.

<a name="storing-tagged-cache-items"></a>
### Сохранение элементов в Кэш по тегам

При помощи тэгов вы можете объединять элементы кэша в группы, а затем очищать всю группу целиком по названию тэга. Вы можете получить доступ к тэгам кэша, путем получения их в упорядоченном массиве имен тегов. Для примера, давайте добавим к тегам кэша значение с помощью метода `put`:

    Cache::tags(['people', 'artists'])->put('John', $john, $minutes);

    Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);

Тем не менее, вы не ограничены методом `put`. Вы можете использовать любой метод сохранения в кэш, во время работы с тегами.

<a name="accessing-tagged-cache-items"></a>
### Получение элементов из кэша с тэгами

Чтобы получить элемент кэша, вы должны указать все тэги к методу `tags`, под которыми он был сохранен:

    $john = Cache::tags(['people', 'artists'])->get('John');

    $anne = Cache::tags(['people', 'authors'])->get('Anne');

Вы можете очистить все элементы по тэгу или списку тэгов. Например, это выражение удалит все элементы кэша с тэгом `people` или `authors`, или в обоих сразу. Таким образом, и `Anne`, и `John` будут удалены из кэша:

    Cache::tags(['people', 'authors'])->flush();

Для сравнения, это выражение удалит только элементы с тэгом `authors`, таким образом `Anne` будет удалена, а `John` нет:

    Cache::tags('authors')->flush();

<a name="adding-custom-cache-drivers"></a>
## Добавление пользовательских драйверов для кэша

Для того, чтобы расширить кэш Laravel с пользовательским драйвером, мы будем использовать метод `extend` у фасада `Cache`, который используется для привязки распознавания пользовательского драйвера к менеджеру. Как правило, это делается в рамках [service provider](/docs/{{version}}/providers).

Например, чтобы зарегистрировать новый драйвер кэша с именем "mongo":

    <?php

    namespace App\Providers;

    use Cache;
    use App\Extensions\MongoStore;
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
            Cache::extend('mongo', function($app) {
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

Первый аргумент переданный в метод `extend` указывает на имя драйвера. Это будет соответствовать вашей опцией `driver` в конфигурационном фале `config/cache.php`. Второй аргумент является `Closure`, который должен возвращать экземпляр класса `Illuminate\Cache\Repository`. В `Closure` будет передан экземпляр класса `$app`, который является экземпляром класса [service container](/docs/{{version}}/container).

Вызов `Cache::extend` может быть завершён в методе `boot` по умолчанию `App\Providers\AppServiceProvider`, который поставляется со свежими приложениями Laravel, или вы можете, для расширения, создать свой собственный service provider - и не забудьте зарегистрировать провайдер в `config/app.php`.

Для того, чтобы создать наш драйвер пользовательского кэша, сначала мы должны реализовать интерфейс `Illuminate\Contracts\Cache\Store` [contract](/docs/{{version}}/contracts). Таким образом, наша реализация кэша MongoDB будет выглядеть примерно так:

    <?php

    namespace App\Extensions;

    class MongoStore implements \Illuminate\Contracts\Cache\Store
    {
        public function get($key) {}
        public function put($key, $value, $minutes) {}
        public function increment($key, $value = 1) {}
        public function decrement($key, $value = 1) {}
        public function forever($key, $value) {}
        public function forget($key) {}
        public function flush() {}
        public function getPrefix() {}
    }

Нам просто нужно реализовать каждый из этих методов, использующих соединение с MongoDB. После того, как наша реализация будет завершена, мы можем закончить нашу регистрацию пользовательского драйвера:

    Cache::extend('mongo', function($app) {
        return Cache::repository(new MongoStore);
    });

После того, как ваше расширение будет завершено, просто обновите в файле `config/cache.php` опцию `driver` на имя вашего расширения.

Если вам хочется разместить ваш код драйвера для кэша, примите во внимание возможность сделать его доступным на Packagist! Или, вы можете создать `Extensions` пространства имен в пределах вашего `app` каталога. Однако, имейте в виду, что Laravel не имеет жесткой структуры приложения, и вы можете организовать ваше приложение в соответствии с вашими предпочтениями.

<a name="events"></a>
## События

Для выполнения кода над каждой операции кэша, вы можете слушать [события](/docs/{{version}}/events). Как правило, вы должны разместить обработчики событий в вашем `EventServiceProvider`:

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