---
git: 8dacb4ca63361ccb80fb45e1c482d1fee39ecebd
---

# Laravel Scout

<a name="introduction"></a>
## Введение

[Laravel Scout](https://github.com/laravel/scout) предоставляет простое решение на основе драйверов для добавления полнотекстового поиска в ваши [модели Eloquent](/docs/{{version}}/eloquent). Используя наблюдателей (observers) моделей, Scout будет автоматически синхронизировать поисковые индексы с данными моделей Eloquent.

В настоящее время Scout поставляется с драйверами [Algolia](https://www.algolia.com/), [Meilisearch](https://www.meilisearch.com), [Typesense](https://typesense.org) и MySQL / PostgreSQL (`database`) . Кроме того, Scout включает поисковый драйвер «коллекций», предназначенный для использования в локальной разработке и не требующий каких-либо внешних зависимостей или сторонних сервисов. Кроме того, написать собственные драйверы просто, и вы можете свободно расширять Scout своими собственными реализациями поиска.

<a name="installation"></a>
## Установка

Сначала установите Scout через менеджер пакетов Composer:

```shell
composer require laravel/scout
```

После установки Scout вы должны опубликовать файл конфигурации Scout с помощью Artisan-команды `vendor:publish`. Эта команда добавит файл конфигурации `scout.php` в каталог `config` вашего приложения:

```shell
php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
```

Наконец, добавьте трейт (trait) `Laravel\Scout\Searchable` к модели, которую вы хотите сделать доступной для поиска. Этот трейт зарегистрирует наблюдателя модели, который будет автоматически синхронизировать модель с вашим драйвером поиска:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;
    }


<a name="queueing"></a>
### Использование очереди

Хотя не обязательно использовать Scout с настройками очередей, рекомендуется настроить [драйвер очереди](/docs/{{version}}/queues) перед использованием библиотеки. Запуск рабочего процесса очереди позволит Scout помещать все операции, синхронизирующие информацию модели с поисковыми индексами, в очередь, что обеспечит более быстрое время ответа для веб-интерфейса вашего приложения.

После настройки драйвера очереди установите значение параметра `queue` в вашем конфигурационном файле `config/scout.php` в `true`:

    'queue' => true,

Даже когда параметр `queue` установлен в `false`, важно помнить, что некоторые драйверы Scout, такие как Algolia и Meilisearch, всегда индексируют записи асинхронно. Это означает, что даже если операция индексации завершена в вашем приложении Laravel, сама поисковая система может не сразу отразить новые и обновленные записи.

Чтобы указать соединение и очередь, которые используют ваши задания Scout, вы можете определить параметр конфигурации `queue` как массив:

    'queue' => [
        'connection' => 'redis',
        'queue' => 'scout'
    ],

Конечно, если вы настроите соединение и очередь, которые используют задания Scout, вам следует запустить обработчик очереди для обработки заданий в этом соединении и очереди:

    php artisan queue:work redis --queue=scout

<a name="driver-prerequisites"></a>
## Требования к драйверам

<a name="algolia"></a>
### Algolia

При использовании драйвера Algolia вы должны настроить учетные данные Algolia `id` и `secret` в файле конфигурации `config/scout.php`. После того как ваши учетные данные будут настроены, вам также необходимо будет установить Algolia PHP SDK через диспетчер пакетов Composer:

```shell
composer require algolia/algoliasearch-client-php
```

<a name="meilisearch"></a>
#### Meilisearch

[Meilisearch](https://www.meilisearch.com) это невероятно быстрая поисковая система с открытым исходным кодом. Если вы не знаете, как установить Meilisearch на свой локальный компьютер, вы можете использовать [Laravel Sail](/docs/{{version}}/sail#meilisearch), официально поддерживаемую Laravel среду разработки Docker.

При использовании драйвера Meilisearch вам необходимо установить Meilisearch PHP SDK через менеджер пакетов Composer:

    composer require meilisearch/meilisearch-php http-interop/http-factory-guzzle

Затем установите переменную среды `SCOUT_DRIVER`, а также учетные данные вашего Meilisearch `host` и `key` в файле` .env` вашего приложения:

```ini
SCOUT_DRIVER=meilisearch
MEILISEARCH_HOST=http://127.0.0.1:7700
MEILISEARCH_KEY=masterKey
```

Для получения дополнительной информации обратитесь к [документации MeiliSearch](https://docs.meilisearch.com/learn/getting_started/quick_start.html).

Кроме того, вы должны убедиться, что вы установили версию `meilisearch/meilisearch-php` которая совместима с вашей двоичной версией Meilisearch, просмотрев [документацию Meilisearch относительно двоичной совместимости](https://github.com/meilisearch/meilisearch-php#-compatibility-with-meilisearch).


> [!WARNING]  
> При обновлении Scout в приложении, которое использует MeiliSearch, вы всегда должны [просматривать любые дополнительные критические изменения](https://github.com/meilisearch/MeiliSearch/releases) в самой службе Meilisearch.

<a name="typesense"></a>
### Typesense

[Typesense](https://typesense.org) - это быстрый и открытый поисковый движок, поддерживающий поиск по ключевым словам, семантический поиск, гео-поиск и векторный поиск.

Вы можете [разместить Typesense у себя](https://typesense.org/docs/guide/install-typesense.html#option-2-local-machine-self-hosting) или использовать [Typesense Cloud](https://cloud.typesense.org).

Чтобы начать использовать Typesense с Scout, установите Typesense PHP SDK через менеджер пакетов Composer:

```shell
composer require typesense/typesense-php
```

Затем установите переменную среды `SCOUT_DRIVER`, а также укажите адрес вашего Typesense и ключ API в файле `.env` вашего приложения:

```env
SCOUT_DRIVER=typesense
TYPESENSE_API_KEY=masterKey
TYPESENSE_HOST=localhost
```

При необходимости вы также можете указать порт, путь и протокол вашей установки:

```env
TYPESENSE_PORT=8108
TYPESENSE_PATH=
TYPESENSE_PROTOCOL=http
```

Дополнительные настройки и определения схемы для коллекций Typesense можно найти в конфигурационном файле вашего приложения `config/scout.php`. Для получения дополнительной информации о Typesense, пожалуйста, обратитесь к [документации Typesense](https://typesense.org/docs/guide/#quick-start).

<a name="preparing-data-for-storage-in-typesense"></a>
#### Подготовка данных для хранения в Typesense

При использовании Typesense ваши модели для поиска должны определить метод `toSearchableArray`, который преобразует основной ключ вашей модели в строку и дату создания в метку времени UNIX:

```php
/**
 * Получить индексируемый массив данных для модели.
 *
 * @return array<string, mixed>
 */
public function toSearchableArray()
{
    return array_merge($this->toArray(),[
        'id' => (string) $this->id,
        'created_at' => $this->created_at->timestamp,
    ]);
}
```

Вы также должны определить схемы коллекций Typesense в файле конфигурации вашего приложения `config/scout.php`. Схема коллекции описывает типы данных каждого поля, которые можно искать с помощью Typesense. Для получения дополнительной информации о всех доступных параметрах схемы обратитесь к [документации Typesense](https://typesense.org/docs/latest/api/collections.html#schema-parameters).

Если вам необходимо изменить схему коллекции Typesense после её определения, вы можете либо выполнить команды `scout:flush` и `scout:import`, которые удалят все существующие индексированные данные и создадут схему заново. Либо вы можете использовать API Typesense для изменения схемы коллекции без удаления каких-либо индексированных данных.

Если ваша модель для поиска поддерживает мягкое удаление, вы должны определить поле `__soft_deleted` в схеме соответствующей модели Typesense в файле конфигурации вашего приложения `config/scout.php`.


```php
User::class => [
    'collection-schema' => [
        'fields' => [
            // ...
            [
                'name' => '__soft_deleted',
                'type' => 'int32',
                'optional' => true,
            ],
        ],
    ],
],
```


<a name="typesense-dynamic-search-parameters"></a>
#### Динамические параметры поиска

Typesense позволяет вам динамически изменять [параметры поиска](https://typesense.org/docs/latest/api/search.html#search-parameters) при выполнении операции поиска с помощью метода `options`:


```php
use App\Models\Todo;

Todo::search('Groceries')->options([
    'query_by' => 'title, description'
])->get();
```

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
         */
         public function searchableAs(): string
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
         * @return array<string, mixed>
         */
        public function toSearchableArray(): array
        {
            $array = $this->toArray();

            // Customize the data array...

            return $array;
        }
    }

Некоторые поисковые движки, такие как Meilisearch, выполнят операции фильтрации (`>`, `<` и т. д.) только для данных правильного типа. Поэтому, при использовании таких поисковых движков и настройке вашего поискового контента, убедитесь, что числовые значения преобразованы в правильный тип:

    public function toSearchableArray()
    {
        return [
            'id' => (int) $this->id,
            'name' => $this->name,
            'price' => (float) $this->price,
        ];
    }
<a name="configuring-filterable-data-for-meilisearch"></a>
#### Настройка фильтруемых данных и параметров индекса (Meilisearch)

В отличие от других драйверов Scout, Meilisearch требует предварительного определения настроек поиска индекса, таких как фильтруемые атрибуты, сортируемые атрибуты и [другие поддерживаемые поля настроек](https://docs.meilisearch.com/reference/api/settings.html).

Фильтруемые атрибуты - это любые атрибуты, по которым вы планируете фильтровать при вызове метода `where` Scout, в то время как сортируемые атрибуты - это любые атрибуты, по которым вы планируете сортировать при вызове метода `orderBy` Scout. Чтобы определить настройки вашего индекса, отредактируйте раздел `index-settings` в записи `meilisearch` в файле конфигурации `scout` вашего приложения:

```php
use App\Models\User;
use App\Models\Flight;

'meilisearch' => [
    'host' => env('MEILISEARCH_HOST', 'http://localhost:7700'),
    'key' => env('MEILISEARCH_KEY', null),
    'index-settings' => [
        User::class => [
            'filterableAttributes'=> ['id', 'name', 'email'],
            'sortableAttributes' => ['created_at'],
            // Other settings fields...
        ],
        Flight::class => [
            'filterableAttributes'=> ['id', 'destination'],
            'sortableAttributes' => ['updated_at'],
        ],
    ],
],
```

Если модель, лежащая в основе определенного индекса, поддерживает мягкое удаление и включена в массив `index-settings`, Scout автоматически включит поддержку фильтрации для мягко удаленных моделей в этом индексе. Если у вас нет других атрибутов, по которым можно фильтровать или сортировать для определения индекса модели с мягким удалением, вы можете просто добавить пустую запись в массив `index-settings` для этой модели:

```php
'index-settings' => [
    Flight::class => []
],
```

После настройки параметров индекса вашего приложения необходимо вызвать команду Artisan `scout:sync-index-settings`. Эта команда сообщит Meilisearch о ваших текущих настройках индекса. Для удобства вы можете включить эту команду в ваш процесс развертывания:

```shell
php artisan scout:sync-index-settings
```

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
         */
        public function getScoutKey(): mixed
        {
            return $this->email;
        }

        /**
         * Переопределение имени ключа индекса модели по умолчанию
         */
        public function getScoutKeyName(): mixed
        {
            return 'email';
        }
    }


<a name="configuring-search-engines-per-model"></a>
### Настройка поисковых драйверов для каждой модели

При выполнении поиска Scout обычно использует поисковый драйвер, указанный по умолчанию в файле конфигурации `scout` вашего приложения. Однако поисковый движок для определенной модели можно изменить, переопределив метод `searchableUsing` в модели:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Engines\Engine;
    use Laravel\Scout\EngineManager;
    use Laravel\Scout\Searchable;

    class User extends Model
    {
        use Searchable;

        /**
         * Get the engine used to index the model.
         */
        public function searchableUsing(): Engine
        {
            return app(EngineManager::class)->engine('meilisearch');
        }
    }

<a name="identifying-users"></a>
### Идентификация пользователей

Scout также позволяет автоматически идентифицировать пользователей при использовании [Algolia](https://algolia.com). Связывание аутентифицированного пользователя с операциями поиска может быть полезно при просмотре аналитики поиска на панели инструментов Algolia. Вы можете включить идентификацию пользователя, определив для переменной среды `SCOUT_IDENTIFY` значение `true` в файле `.env` вашего приложения:

```ini
SCOUT_IDENTIFY=true
```

Включение этой функции также передаст IP-адрес запроса и основной идентификатор вашего аутентифицированного пользователя в Algolia, поэтому эти данные будут связаны с любым поисковым запросом, сделанным пользователем.


<a name="database-and-collection-engines"></a>
## Драйвер базы данных/колекции

<a name="database-engine"></a>
### Драйвер базы данных

>[!WARNING]
> В настоящее время драйвер базы данных поддерживает MySQL и PostgreSQL.

Если ваше приложение взаимодействует с небольшими или средними базами данных или имеет легкую нагрузку, вам может быть удобнее начать с драйвера "database" Scout. Драйвер будет использовать запросы "where like" и полнотекстовые индексы при фильтрации результатов из вашей существующей базы данных для определения применимых результатов поиска для вашего запроса.

Чтобы использовать драйвер базы данных, вы можете просто установить значение переменной среды `SCOUT_DRIVER` равным `database`, или указать драйвер `database` непосредственно в конфигурационном файле `scout` вашего приложения:

```ini
SCOUT_DRIVER=database
```

Как только вы укажете драйвер базы данных в качестве предпочтительного, вам необходимо [настроить свои данные для поиска](#configuring-searchable-data). Затем вы можете начать [выполнять поисковые запросы](#searching) к вашим моделям. Некоторые поисковые движки, такие как Algolia, Meilisearch или Typesense, требуют предворительную индексацию. При использовании баз данных индексация не требуется

#### Настройка стратегий поиска в базе данных

По умолчанию драйвер базы данных будет выполнять запрос "where like" к каждому атрибуту модели, который вы [настроили для поиска](#configuring-searchable-data). Однако в некоторых ситуациях это может привести к низкой производительности. Поэтому стратегия поиска может быть настроена так, чтобы некоторые столбцы использовали запросы полнотекстового поиска или только условия "where like" для поиска префиксов строк (`пример%`) вместо поиска внутри всей строки (`%пример%`).

Чтобы определить это поведение, вы можете назначить атрибуты PHP методу `toSearchableArray` вашей модели. Любые столбцы, которым не назначены дополнительные стратегии поиска, будут продолжать использовать стратегию "where like" по умолчанию:

```php
use Laravel\Scout\Attributes\SearchUsingFullText;
use Laravel\Scout\Attributes\SearchUsingPrefix;

/**
 * Get the indexable data array for the model.
 *
 * @return array<string, mixed>
 */
#[SearchUsingPrefix(['id', 'email'])]
#[SearchUsingFullText(['bio'])]
public function toSearchableArray(): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'bio' => $this->bio,
    ];
}
```

> [!WARNING]
> Перед тем как указывать, что столбец должен использовать ограничения полнотекстового запроса, убедитесь, что столбцу был назначен [полнотекстовый индекс](/docs/{{version}}/migrations#available-index-types).

<a name="collection-engine"></a>
### Драйвер коллекции

Хотя вы можете использовать поисковые системы Algolia, Meilisearch или Typesense во время локальной разработки, вам может быть удобнее начать работу с поисковым драйвером «коллекций». Драйвер коллекций данных будет использовать условие «where» и фильтрацию набора результатов из вашей существующей базы данных, чтобы определить применимые результаты поиска для запроса. При использовании этого механизма нет необходимости «индексировать» доступные для поиска модели, поскольку они будут просто извлечены из локальной базы данных.

Чтобы использовать драйвер коллекций, вы можете просто установить для переменной среды `SCOUT_DRIVER` значение `collection` или указать драйвер `collection` непосредственно в файле конфигурации `scout` вашего приложения:

```ini
SCOUT_DRIVER=collection
```

После того как вы указали драйвер коллекции в качестве предпочтительного, вы можете начать [выполнение поисковых запросов](#searching) по вашим моделям. Индексирование поисковой системой, необходимое для заполнения индексов Algolia или MeiliSearch, не требуется при использовании драйвера коллекций.

#### Отличия от Движка Базы Данных

На первый взгляд, драйверы "database" и "collections" достаточно похожи. Оба они взаимодействуют непосредственно с вашей базой данных для получения результатов поиска. Однако драйвер "collections" не использует полнотекстовые индексы или `LIKE` операторы для поиска соответствующих записей. Вместо этого он извлекает все возможные записи и использует вспомогательную функцию `Str::is` Laravel для определения того, существует ли строка поиска в значениях атрибутов модели.

Драйвер "collections" является наиболее портативным, так как он работает с любыми реляционными базами данных, поддерживаемыми Laravel (включая SQLite и SQL Server); однако он менее эффективен, чем драйвер базы данных.

<a name="indexing"></a>
## Индексирование

<a name="batch-import"></a>
### Пакетный импорт

Если вы устанавливаете Scout в существующий проект, возможно, у вас уже есть записи базы данных, которые необходимо импортировать в индексы. Scout предоставляет Artisan-команду `scout:import`, которую вы можете использовать для импорта всех существующих записей в поисковые индексы:

```shell
php artisan scout:import "App\Models\Post"
```

Команду `flush` можно использовать для удаления всех записей из поисковых индексов:

```shell
php artisan scout:flush "App\Models\Post"
```

<a name="modifying-the-import-query"></a>
#### Изменение запроса на импорт

Если вы хотите изменить запрос, который используется для получения моделей для пакетного импорта, вы можете определить метод `makeAllSearchableUsing` в модели. Это отличное место для добавления любых отношений, которые могут потребоваться перед импортом:


    use Illuminate\Database\Eloquent\Builder;

    /**
     * Измените запрос, сделав поиск по всем моделям.
     */
    protected function makeAllSearchableUsing(Builder $query): Builder
    {
        return $query->with('author');
    }

> [!WARNING]  
> Метод `makeAllSearchableUsing` может оказаться не применимым при использовании очереди для пакетного импорта моделей. Связи [не восстанавливаются](/docs/{{version}}/queues#handling-relationships) при обработке коллекций моделей в заданиях.

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

> [!NOTE]
> Метод `searchable` можно считать операцией "upsert". Другими словами, если запись модели уже есть в поисковом индексе, то она будет обновлена. Если записи нет, она будет добавлена в индекс.

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


<a name="modifying-records-before-importing"></a>
#### Изменение Записей Перед Импортом

Иногда вам может потребоваться подготовить коллекцию моделей перед тем, как они станут доступны для поиска. Например, вы можете захотеть предварительно загрузить отношение, чтобы данные они могли быть эффективно добавлены в ваш индекс поиска. Для достижения этой цели определите метод `makeSearchableUsing` в соответствующей модели:

    use Illuminate\Database\Eloquent\Collection;

    /**
     * Modify the collection of models being made searchable.
     */
    public function makeSearchableUsing(Collection $models): Collection
    {
        return $models->load('author');
    }

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
     */
    public function shouldBeSearchable(): bool
    {
        return $this->isPublished();
    }

Метод `shouldBeSearchable` применяется только при манипуляциях с моделями с помощью методов `save`, `create`, запросов или в цепочке вызовов. Непосредственное добавление моделей или коллекций в поисковый индекс с помощью метода `searchable` переопределит результат метода `shouldBeSearchable`.

> [!WARNING]  
> Метод `shouldBeSearchable` не применим при использовании драйвера "database" в Scout, так как все данные для поиска всегда хранятся в базе данных. Для достижения аналогичного поведения при использовании драйвера базы данных следует использовать [Условия Where](#where-clauses) вместо этого.

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

Кроме того, метод `whereIn` может быть использован для проверки того, содержится ли значение заданного столбца в указанном массиве:
   
     $orders = Order::search('Star Trek')->whereIn(
        'status', ['open', 'paid']
    )->get();

Метод `whereNotIn` проверяет, что значение заданного столбца не содержится в указанном массиве:

    $orders = Order::search('Star Trek')->whereNotIn(
        'status', ['closed']
    )->get();

Поскольку поисковый индекс не является реляционной базой данных, более сложные "where" условия в настоящее время не поддерживаются.

> [!WARNING]  
> Если ваше приложение использует Meilisearch, вам необходимо настроить [фильтруемые атрибуты](#configuring-filterable-data-for-meilisearch) вашего приложения перед использованием условий "where" Scout.

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

> [!WARNING]  
> Поскольку поисковые движки не осведомлены о глобальных определениях области видимости вашей Eloquent-модели, вы не должны использовать глобальные области видимости в приложениях, которые используют пагинацию Scout. Или же вы должны воссоздать ограничения глобальной области видимости при поиске через Scout.

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


> [!NOTE]  
> Когда псевдоудаленная модель будет окончательно удалена с помощью `forceDelete`, Scout автоматически удалит ее из поискового индекса.

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

Поскольку это обратный вызов вызывается после того, как соответствующие модели уже были извлечены из поискового движка вашего приложения, метод `query` не следует использовать для "фильтрации" результатов. Вместо этого вы должны использовать [условия where Scout](#where-clauses).

<a name="customizing-the-eloquent-results-query"></a>

#### Настройка Запроса Результатов Eloquent

После того, как Scout получает список соответствующих моделей Eloquent из поискового движка вашего приложения, для извлечения всех соответствующих моделей по их первичным ключам используется Eloquent. Вы можете настроить этот запрос, вызвав метод `query`. Метод `query` принимает замыкание, которое получит экземпляр построителя запросов Eloquent в качестве аргумента:

```php
use App\Models\Order;
use Illuminate\Database\Eloquent\Builder;

$orders = Order::search('Star Trek')
    ->query(fn (Builder $query) => $query->with('invoices'))
    ->get();
```

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

    use App\ScoutExtensions\MySqlSearchEngine;
    use Laravel\Scout\EngineManager;

    /**
     * Загрузка сервисов приложения.
     */
    public function boot(): void
    {
        resolve(EngineManager::class)->extend('mysql', function () {
            return new MySqlSearchEngine;
        });
    }

После регистрации движка вы можете указать его в качестве `driver` по умолчанию в файле конфигурации `config/scout.php`:

    'driver' => 'mysql',
