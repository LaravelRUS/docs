git 2cf67bcaacfec590098cefb45af824b74671cfa0

---

# Laravel Scout

- [Введение](#introduction)
- [Установка](#installation)
    - [Требования к драйверам](#driver-prerequisites)
    - [Очередь](#queueing)
- [Настройка](#configuration)
    - [Настройка индексов моделей](#configuring-model-indexes)
    - [Настройка поисковых данных](#configuring-searchable-data)
    - [Настройка идентификатора модели](#configuring-the-model-id)
    - [Идентификация пользователей](#identifying-users)
- [Локальная разработка](#local-development)
- [Индексирование](#indexing)
    - [Пакетный импорт](#batch-import)
    - [Добавление записей](#adding-records)
    - [Обновление записей](#updating-records)
    - [Удаление записей](#removing-records)
    - [Приостановка индексации](#pausing-indexing)
    - [Экземпляры моделей с условным поиском](#conditionally-searchable-model-instances)
- [Поиск](#searching)
    - [Условия Where](#where-clauses)
    - [Постраничная разбивка данных (Pagination)](#pagination)
    - [Псевдоудаление](#soft-deleting)
    - [Настройка поискового движка](#customizing-engine-searches)
- [Разработка поискового движка](#custom-engines)
- [Собственные методы поиска](#builder-macros)

<a name="introduction"></a>
## Введение

[Laravel Scout](https://github.com/laravel/scout) предоставляет простое решение на основе драйверов для добавления полнотекстового поиска в ваши [модели Eloquent](/docs/{{version}}/eloquent). Используя наблюдателей (observers) моделей, Scout будет автоматически синхронизировать поисковые индексы с данными моделей Eloquent.

В настоящее время Scout поставляется с драйверами [Algolia](https://www.algolia.com/) и [MeiliSearch](https://www.meilisearch.com). Кроме того, Scout включает поисковый драйвер «коллекций», предназначенный для использования в локальной разработке и не требующий каких-либо внешних зависимостей или сторонних сервисов. Кроме того, написать собственные драйверы просто, и вы можете свободно расширять Scout своими собственными реализациями поиска.

<a name="installation"></a>
## Установка

Сначала установите Scout через менеджер пакетов Composer:

    composer require laravel/scout

После установки Scout вы должны опубликовать файл конфигурации Scout с помощью Artisan-команды `vendor:publish`. Эта команда добавит файл конфигурации `scout.php` в каталог `config` вашего приложения:

    php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"

Наконец, добавьте трейт (trait) `Laravel\Scout\Searchable` к модели, которую вы хотите сделать доступной для поиска. Этот трейт зарегистрирует наблюдателя модели, который будет автоматически синхронизировать модель с вашим драйвером поиска:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;
    }

<a name="driver-prerequisites"></a>
### Требования к драйверам

<a name="algolia"></a>
#### Algolia

При использовании драйвера Algolia вы должны настроить учетные данные Algolia `id` и `secret` в файле конфигурации `config/scout.php`. После того как ваши учетные данные будут настроены, вам также необходимо будет установить Algolia PHP SDK через диспетчер пакетов Composer:

    composer require algolia/algoliasearch-client-php

<a name="meilisearch"></a>
#### MeiliSearch

[MeiliSearch](https://www.meilisearch.com) это невероятно быстрая поисковая система с открытым исходным кодом. Если вы не знаете, как установить MeiliSearch на свой локальный компьютер, вы можете использовать [Laravel Sail](/docs/{{version}}/sail#meilisearch), официально поддерживаемую Laravel среду разработки Docker.

При использовании драйвера MeiliSearch вам необходимо установить MeiliSearch PHP SDK через менеджер пакетов Composer:

    composer require meilisearch/meilisearch-php http-interop/http-factory-guzzle

Затем установите переменную среды `SCOUT_DRIVER`, а также учетные данные вашего MeiliSearch `host` и `key` в файле` .env` вашего приложения:

    SCOUT_DRIVER=meilisearch
    MEILISEARCH_HOST=http://127.0.0.1:7700
    MEILISEARCH_KEY=masterKey

Для получения дополнительной информации обратитесь к [документации MeiliSearch](https://docs.meilisearch.com/learn/getting_started/quick_start.html).

Кроме того, вы должны убедиться, что вы установили версию `meilisearch/meilisearch-php` которая совместима с вашей двоичной версией MeiliSearch, просмотрев [документацию MeiliSearch относительно двоичной совместимости](https://github.com/meilisearch/meilisearch-php#-compatibility-with-meilisearch).

> {note} При обновлении Scout в приложении, которое использует MeiliSearch, вы всегда должны [просматривать любые дополнительные критические изменения](https://github.com/meilisearch/MeiliSearch/releases) в самой службе MeiliSearch.

<a name="queueing"></a>
### Очередь

Хотя при использовании Scout не является строго обязательным, но вам следует серьезно подумать о настройке [драйвера очереди](/docs/{{version}}/queues) перед использованием библиотеки. Запуск обработчика очереди позволит Scout ставить в очередь все операции, которые синхронизируют информацию вашей модели с вашими поисковыми индексами, обеспечивая гораздо лучшее время отклика для веб-интерфейса вашего приложения.

После того как вы настроили драйвер очереди, установите значение опции `queue` в вашем конфигурационном файле `config/scout.php` равным `true`:

    'queue' => true,

<a name="configuration"></a>
## Настройка

<a name="configuring-model-indexes"></a>
### Настройка индексов моделей

Каждая модель Eloquent синхронизируется с заданным поисковым «индексом», который содержит все доступные для поиска записи для этой модели. Другими словами, вы можете думать о каждом индексе как о таблице MySQL. По умолчанию каждая модель будет сохранена в индексе, соответствующем типичному «табличному» имени модели. Обычно это форма множественного числа от названия модели; однако вы можете настроить индекс, переопределив метод `searchableAs` в модели:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;

        /**
         * Переопределение имени индекса модели по умолчанию
         *
         * @return string
         */
        public function searchableAs()
        {
            return 'posts_index';
        }
    }

<a name="configuring-searchable-data"></a>
### Настройка поисковых данных

По умолчанию вся форма `toArray` данной модели будет сохранена в ее поисковом индексе. Если вы хотите настроить данные, которые синхронизируются с поисковым индексом, вы можете переопределить метод `toSearchableArray` в модели:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;

        /**
         * Переопределение массива индекса модели по умолчанию
         *
         * @return array
         */
        public function toSearchableArray()
        {
            $array = $this->toArray();

            // Customize the data array...

            return $array;
        }
    }

<a name="configuring-the-model-id"></a>
### Настройка идентификатора модели

По умолчанию Scout будет использовать первичный ключ модели в качестве уникального идентификатора / ключа модели, который хранится в поисковом индексе. Если вам нужно настроить это поведение, вы можете переопределить методы `getScoutKey` и `getScoutKeyName` в модели:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class User extends Model
    {
        use Searchable;

        /**
         * Переопределение значения ключа индекса модели по умолчанию
         *
         * @return mixed
         */
        public function getScoutKey()
        {
            return $this->email;
        }

        /**
         * Переопределение имени ключа индекса модели по умолчанию
         *
         * @return mixed
         */
        public function getScoutKeyName()
        {
            return 'email';
        }
    }

<a name="identifying-users"></a>
### Идентификация пользователей

Scout также позволяет автоматически идентифицировать пользователей при использовании [Algolia](https://algolia.com). Связывание аутентифицированного пользователя с операциями поиска может быть полезно при просмотре аналитики поиска на панели инструментов Algolia. Вы можете включить идентификацию пользователя, определив для переменной среды `SCOUT_IDENTIFY` значение `true` в файле `.env` вашего приложения:

    SCOUT_IDENTIFY=true

Включение этой функции также передаст IP-адрес запроса и основной идентификатор вашего аутентифицированного пользователя в Algolia, поэтому эти данные будут связаны с любым поисковым запросом, сделанным пользователем.

<a name="local-development"></a>
## Локальная разработка

Хотя вы можете использовать поисковые системы Algolia или MeiliSearch во время локальной разработки, вам может быть удобнее начать работу с поисковым драйвером «коллекций». Драйвер коллекций данных будет использовать условие «where» и фильтрацию набора результатов из вашей существующей базы данных, чтобы определить применимые результаты поиска для запроса. При использовании этого механизма нет необходимости «индексировать» доступные для поиска модели, поскольку они будут просто извлечены из локальной базы данных.

Чтобы использовать драйвер коллекций, вы можете просто установить для переменной среды `SCOUT_DRIVER` значение `collection` или указать драйвер `collection` непосредственно в файле конфигурации `scout` вашего приложения:

```ini
SCOUT_DRIVER=collection
```

После того как вы указали драйвер коллекции в качестве предпочтительного, вы можете начать [выполнение поисковых запросов](#searching) по вашим моделям. Индексирование поисковой системой, необходимое для заполнения индексов Algolia или MeiliSearch, не требуется при использовании драйвера коллекций.

<a name="indexing"></a>
## Индексирование

<a name="batch-import"></a>
### Пакетный импорт

Если вы устанавливаете Scout в существующий проект, возможно, у вас уже есть записи базы данных, которые необходимо импортировать в индексы. Scout предоставляет Artisan-команду `scout:import`, которую вы можете использовать для импорта всех существующих записей в поисковые индексы:

    php artisan scout:import "App\Models\Post"

Команду `flush` можно использовать для удаления всех записей из поисковых индексов:

    php artisan scout:flush "App\Models\Post"

<a name="modifying-the-import-query"></a>
#### Изменение запроса на импорт

Если вы хотите изменить запрос, который используется для получения моделей для пакетного импорта, вы можете определить метод `makeAllSearchableUsing` в модели. Это отличное место для добавления любых отношений, которые могут потребоваться перед импортом:

    /**
     * Измените запрос, сделав поиск по всем моделям.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $query
     * @return \Illuminate\Database\Eloquent\Builder
     */
    protected function makeAllSearchableUsing($query)
    {
        return $query->with('author');
    }

<a name="adding-records"></a>
### Добавление записей

После того как вы добавили в модель трейт `Laravel\Scout\Searchable`, всё, что вам нужно сделать, это вызвать метод `save` или `create` на экземпляре модели, и она будет автоматически добавлена в поисковый индекс. Если вы настроили Scout для [использования очередей](#queueing), эта операция будет выполняться в фоновом режиме обработчиком очереди:

    use App\Models\Order;

    $order = new Order;

    // ...

    $order->save();

<a name="adding-records-via-query"></a>
#### Добавление записей через запрос

Если вы хотите добавить коллекцию моделей в поисковый индекс с помощью запроса Eloquent, вы можете связать метод `searchable` с запросом Eloquent. Метод `searchable` [разделит результаты](/docs/{{version}}/eloquent#chunking-results) запроса и добавит блоки в поисковый индекс. Опять же, если вы настроили Scout для использования очередей, все блоки будут импортированы в фоновом режиме обработчиком очереди:

    use App\Models\Order;

    Order::where('price', '>', 100)->searchable();

Вы также можете вызвать метод `searchable` для экземпляра коллекции Eloquent:

    $user->orders()->searchable();

Или, если у вас уже есть коллекция Eloquent, вы можете вызвать метод `searchable` для коллекции, чтобы добавить экземпляры моделей в их соответствующий индекс:

    $orders->searchable();

> {tip} Метод `searchable` можно считать операцией "upsert". Другими словами, если запись модели уже есть в поисковом индексе, то она будет обновлена. Если записи нет, она будет добавлена в индекс.

<a name="updating-records"></a>
### Обновление записей

Чтобы обновить поисковый индекс модели, вам нужно только обновить свойства экземпляра модели и вызвать метод `save` для сохранения в базе данных. Scout автоматически сохранит изменения в поисковом индексе:

    use App\Models\Order;

    $order = Order::find(1);

    // Update the order...

    $order->save();

Вы также можете вызвать метод searchable` в экземпляре запроса Eloquent, чтобы обновить коллекцию моделей. Если моделей нет в поисковом индексе, они будут созданы:

    Order::where('price', '>', 100)->searchable();

Если вы хотите обновить записи поискового индекса для всех моделей в коллекции, вы можете вызвать метод `searchable` в цепочке вызова:

    $user->orders()->searchable();

Или, если у вас уже есть коллекция Eloquent, вы можете вызвать метод `searchable` для коллекции, чтобы добавить экземпляры моделей в их соответствующий индекс:

    $orders->searchable();

<a name="removing-records"></a>
### Удаление записей

Чтобы удалить запись из поискового индекса, вы можете вызвать метод `delete` на экземпляре модели для удаления модели из базы данных. Это можно сделать, даже если вы используете [псевдоудаление](/docs/{{version}}/eloquent#soft-deleting) модели:

    use App\Models\Order;

    $order = Order::find(1);

    $order->delete();

Если вы не хотите извлекать модель перед удалением записи, вы можете использовать метод `unsearchable` для экземпляра запроса Eloquent:

    Order::where('price', '>', 100)->unsearchable();

Если вы хотите удалить записи поискового индекса для всех моделей в коллекции, вы можете вызвать метод `unsearchable` на цепочке вызова:

    $user->orders()->unsearchable();

Или, если у вас уже есть коллекция Eloquent, вы можете вызвать метод `unsearchable` для коллекции, чтобы удалить экземпляры моделей из индекса:

    $orders->unsearchable();

<a name="pausing-indexing"></a>
### Приостановка индексации

Иногда вам может потребоваться выполнить некоторые операции Eloquent с моделью без синхронизации данных модели с поисковым индексом. Вы можете сделать это, используя метод `withoutSyncingToSearch`. Этот метод принимает замыкание, которое будет немедленно выполнено. Любые операции модели, которые происходят внутри замыкания, не будут синхронизироваться с индексом модели:

    use App\Models\Order;

    Order::withoutSyncingToSearch(function () {
        // Perform model actions...
    });

<a name="conditionally-searchable-model-instances"></a>
### Экземпляры моделей с условным поиском

Иногда может потребоваться сделать модель доступной для поиска только при определенных условиях. Например, представьте, что у вас есть модель `App\Models\Post`, которая может находиться в одном из двух состояний: «черновик» и «опубликована». Вы можете добавлять в поисковый индекс только «опубликованные» сообщения. Для этого необходимо определить в модели метод `shouldBeSearchable`:

    /**
     * Определите, когда модель должна быть доступной для поиска.
     *
     * @return bool
     */
    public function shouldBeSearchable()
    {
        return $this->isPublished();
    }

Метод `shouldBeSearchable` применяется только при манипуляциях с моделями с помощью методов `save`, `create`, запросов или в цепочке вызовов. Непосредственное добавление моделей или коллекций в поисковый индекс с помощью метода `searchable` переопределит результат метода `shouldBeSearchable`.

<a name="searching"></a>
## Поиск

Вы можете выполнить поиск по модели, используя метод `search`. Метод `search` принимает строку в качестве поискового запроса, которая будет использоваться для поиска по модели. Затем вы должны вызвать метод `get`, чтобы получить модель Eloquent в качестве результата заданного поискового запроса:

    use App\Models\Order;

    $orders = Order::search('Star Trek')->get();

Поскольку поисковые запросы Scout возвращают коллекцию моделей Eloquent, вы можете возвращать результаты непосредственно из маршрута или контроллера, и они будут автоматически преобразованы в JSON:

    use App\Models\Order;
    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return Order::search($request->search)->get();
    });

Если вы хотите получить необработанные результаты поиска до того, как они будут преобразованы в модели Eloquent, вы можете использовать метод `raw`:

    $orders = Order::search('Star Trek')->raw();

<a name="custom-indexes"></a>
#### Пользовательский индекс

Поисковые запросы обычно выполняются по индексу, указанному в методе [searchchableAs](#configuring-model-indexes) модели. Однако вы можете использовать метод `within`, чтобы указать индекс, который следует использовать вместо этого:

    $orders = Order::search('Star Trek')
        ->within('tv_shows_popularity_desc')
        ->get();

<a name="where-clauses"></a>
### Условия Where

Scout позволяет добавлять в поисковые запросы простые условия "where" ("где"). В настоящее время эти условия поддерживают только базовые проверки числового равенства и в первую очередь полезны для определения области поисковых запросов по идентификатору владельца. Поскольку поисковый индекс не является реляционной базой данных, более сложные условия "where" в настоящее время не поддерживаются:

    use App\Models\Order;

    $orders = Order::search('Star Trek')->where('user_id', 1)->get();

<a name="pagination"></a>
### Постраничная разбивка данных (Pagination) (Пагинация)

Помимо получения коллекции моделей, вы можете разбить результаты поиска на страницы, используя метод `paginate`. Этот метод вернет экземпляр `Illuminate\Pagination\LengthAwarePaginator`, как если бы вы [разбили на страницы](/docs/{{version}}/pagination) обычный запрос Eloquent:

    use App\Models\Order;

    $orders = Order::search('Star Trek')->paginate();

Вы можете указать, сколько моделей извлекать на странице, передав количество в качестве аргумента методу `paginate`:

    $orders = Order::search('Star Trek')->paginate(15);

Получив результаты, вы можете отобразить результаты и отобразить ссылки на страницы с помощью [Blade](/docs/{{version}}/blade), как если бы вы разбили на страницы обычный запрос Eloquent:

```html
<div class="container">
    @foreach ($orders as $order)
        {{ $order->price }}
    @endforeach
</div>

{{ $orders->links() }}
```

Конечно, если вы хотите получить результаты разбиения на страницы в виде JSON, вы можете вернуть экземпляр пагинатора прямо из маршрута или контроллера:

    use App\Models\Order;
    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        return Order::search($request->input('query'))->paginate(15);
    });

<a name="soft-deleting"></a>
### Псевдоудаление

Если ваши проиндексированные модели [псевдоудалены](/docs/{{version}}/eloquent#soft-deleting) и вам нужно выполнить поиск по своим псевдоудаленным моделям, установите параметр `soft_delete` в файле `config/scout.php` на `true`:

    'soft_delete' => true,

Когда этот параметр имеет значение `true`, Scout не будет удалять псевдоудаленные модели из поискового индекса. Вместо этого он установит скрытый атрибут `__soft_deleted` для проиндексированной записи. Затем вы можете использовать методы `withTrashed` или `onlyTrashed` для получения псевдоудаленных записей при поиске:

    use App\Models\Order;

    // Использовать удаленные записи при получении результатов ...
    $orders = Order::search('Star Trek')->withTrashed()->get();

    // Использовать только удаленные записи при получении результатов ...
    $orders = Order::search('Star Trek')->onlyTrashed()->get();

> {tip} Когда псевдоудаленная модель будет окончательно удалена с помощью `forceDelete`, Scout автоматически удалит ее из поискового индекса.

<a name="customizing-engine-searches"></a>
### Настройка поискового движка

Если вам нужно выполнить расширенную настройку поведения поискового движка, вы можете передать замыкание в качестве второго аргумента методу `search`. Например, вы можете использовать замыкание, чтобы добавить данные о геолокации в параметры поиска до того, как поисковый запрос будет передан в Algolia:

    use Algolia\AlgoliaSearch\SearchIndex;
    use App\Models\Order;

    Order::search(
        'Star Trek',
        function (SearchIndex $algolia, string $query, array $options) {
            $options['body']['query']['bool']['filter']['geo_distance'] = [
                'distance' => '1000km',
                'location' => ['lat' => 36, 'lon' => 111],
            ];

            return $algolia->search($query, $options);
        }
    )->get();

<a name="custom-engines"></a>
## Разработка поискового движка

<a name="writing-the-engine"></a>
#### Реализация своего поискового механизма

Если одна из встроенных поисковых систем Scout не соответствует вашим потребностям, вы можете написать свой собственный поисковый механизм (поисковый движок) и зарегистрировать его в Scout. Ваш движок должен расширять абстрактный класс `Laravel\Scout\Engines\Engine`. Этот класс содержит восемь методов, которые должен реализовать ваш движок:

    use Laravel\Scout\Builder;

    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function mapIds($results);
    abstract public function map(Builder $builder, $results, $model);
    abstract public function getTotalCount($results);
    abstract public function flush($model);

Возможно, вам будет полезно просмотреть реализации этих методов в классе `Laravel\Scout\Engines\AlgoliaEngine`. Этот класс предоставит вам хорошую отправную точку для изучения того, как реализовать каждый из этих методов в вашем собственном движке.

<a name="registering-the-engine"></a>
#### Регистрация поискового движка

После того как вы написали свой собственный движок, вы можете зарегистрировать его в Scout, используя метод `extend` менеджера Scout. Диспетчер Scout может быть определён из служебного контейнера Laravel. Вы должны вызвать метод `extend` из метода `boot` вашего класса `App\Providers\AppServiceProvider` или любого другого провайдера, используемого вашим приложением:

    use App\ScoutExtensions\MySqlSearchEngine
    use Laravel\Scout\EngineManager;

    /**
     * Загрузка сервисов приложения.
     *
     * @return void
     */
    public function boot()
    {
        resolve(EngineManager::class)->extend('mysql', function () {
            return new MySqlSearchEngine;
        });
    }

После регистрации движка вы можете указать его в качестве `driver` по умолчанию в файле конфигурации `config/scout.php`:

    'driver' => 'mysql',

<a name="builder-macros"></a>
## Собственные методы поиска

Если вы хотите назначить собственный метод построения поиска Scout, вы можете использовать метод `macro` класса `Laravel\Scout\Builder`. Как правило, "макросы" следует определять в методе `boot` [сервисного провайдера](/docs/{{version}}/providers):

    use Illuminate\Support\Facades\Response;
    use Illuminate\Support\ServiceProvider;
    use Laravel\Scout\Builder;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Builder::macro('count', function () {
            return $this->engine()->getTotalCount(
                $this->engine()->search($this)
            );
        });
    }

Функция `macro` принимает имя макроса в качестве первого аргумента и замыкание в качестве второго аргумента. Замыкание макроса будет выполнено при вызове имени макроса из реализации `Laravel\Scout\Builder`:

    use App\Models\Order;

    Order::search('Star Trek')->count();
