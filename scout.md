git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Laravel Scout

- [Введение](#introduction)
- [Установка](#installation)
    - [Очередь](#queueing)
    - [Условия для работы драйвера](#driver-prerequisites)
- [Настройка](#configuration)
    - [Настройка индексов модели](#configuring-model-indexes)
    - [Настройка данных для поиска](#configuring-searchable-data)
- [Индексирование](#indexing)
    - [Пакетный импорт](#batch-import)
    - [Добавление записей](#adding-records)
    - [Обновление записей](#updating-records)
    - [Удаление записей](#removing-records)
    - [Приостановка индексирования](#pausing-indexing)
- [Поиск](#searching)
    - [Условия Where](#where-clauses)
    - [Страничный вывод](#pagination)
- [Пользовательские движки](#custom-engines)

<a name="introduction"></a>
## Введение

Laravel Scout предоставляет простое решение на основе драйверов для добавления полнотекстового поиска в ваши [Eloquent-модели](/docs/{{version}}/eloquent). С помощью наблюдателей за моделями Scout будет автоматически синхронизировать ваши поисковые индексы с вашими записями Eloquent.

Сейчас Scout поставляется с драйвером [Algolia](https://www.algolia.com/); однако, написать свой драйвер довольно просто и вы можете дополнить Scout своей собственной реализацией поиска.

<a name="installation"></a>
## Установка

Сначала установите Scout через менеджер пакетов Composer:

    composer require laravel/scout

Затем добавьте `ScoutServiceProvider` в массив `providers` вашего конфига `config/app.php`:

    Laravel\Scout\ScoutServiceProvider::class,

После регистрации сервис-провайдера Scout опубликуйте его конфигурацию с помощью Artisan-команды `vendor:publish`. Эта команда опубликует файл настроек `scout.php` в ваш каталог `config`:

    php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"

И наконец, добавьте трейт `Laravel\Scout\Searchable` в модель, которую хотите сделать доступной для поиска. Этот трейт зарегистрирует наблюдателя модели, чтобы синхронизировать модель с вашим драйвером поиска:

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;
    }

<a name="queueing"></a>
### Очередь

Вам нужно изучить настройку [драйвера очереди](/docs/{{version}}/queues) перед использованием этой библиотеки, не смотря на то, что использовать очереди в Scout не обязательно. После запуска обработчика очереди Scout сможет ставить в очередь все операции синхронизации информации вашей модели с вашими поисковыми индексами, обеспечивая намного лучшее время отклика для веб-интерфейса вашего приложения.

После настройки драйвера очереди задайте значение `true` параметру `queue` в конфиге `config/scout.php`:

    'queue' => true,

<a name="driver-prerequisites"></a>
### Условия для работы драйвера

#### Algolia

При использовании драйвера Algolia задайте ваши учётные данные Algolia `id` и `secret` в своем конфиге `config/scout.php`. После этого установите Algolia PHP SDK через менеджер пакетов Composer:

    composer require algolia/algoliasearch-client-php

<a name="configuration"></a>
## Настройка

<a name="configuring-model-indexes"></a>
### Настройка индексов модели

Каждая модель Eloquent синхронизируется с заданным поисковым "индексом", который содержит все записи для поиска этой модели. Другими словами, можно представить каждый индекс как таблицу MySQL. По-умолчанию для каждой модели будет назначен индекс, совпадающий с именем "таблицы" модели. Как правило, это множественная форма имени модели, но вы можете изменить индекс модели, переопределив для неё метод `searchableAs`:

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * Получить имя индекса для модели.
         *
         * @return string
         */
        public function searchableAs()
        {
            return 'posts_index';
        }
    }

<a name="configuring-searchable-data"></a>
### Настройка данных для поиска

По-умолчанию в индексе модели будут все её данные, аналогично результату `toArray`. Если вы хотите настроить то, какие данные будут синхронизироваться с поисковым индексом, переопределите метод модели `toSearchableArray`:

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * Получить индексируемый массив данных для модели.
         *
         * @return array
         */
        public function toSearchableArray()
        {
            $array = $this->toArray();

            // Customize array...

            return $array;
        }
    }

<a name="indexing"></a>
## Индексирование

<a name="batch-import"></a>
### Пакетный импорт

Если вы устанавливаете Scout в существующий проект, у вас уже могут быть записи БД, которые вам надо импортировать в ваш драйвер поиска. Scout предоставляет Artisan-команду `import`, которую можно использовать для импорта всех существующих записей в ваши поисковые индексы:

    php artisan scout:import "App\Post"

<a name="adding-records"></a>
### Добавление записей

После добавления в модель типажа `Laravel\Scout\Searchable` вам остаётся только сохранить (`save`) экземпляр модели, и она автоматически добавится в поисковый индекс. Если вы настроили Scout на [использование очередей](#queueing), то эта операция будет выполнена в фоне вашим обработчиком очереди:

    $order = new App\Order;

    // ...

    $order->save();

#### Добавление через запрос

Если вы хотите добавить коллекцию моделей в свой поисковый индекс через запрос Eloquent, вы можете добавить метод `searchable` к запросу Eloquent. Метод `searchable` [разделит на части результаты](/docs/{{version}}/eloquent#chunking-results) запроса и добавит записи в ваш поисковый индекс. Если вы настроили Scout на использование очередей, то все части будут добавлены в фоне обработчиками очереди:

    // Добавление через запрос Eloquent...
    App\Order::where('price', '>', 100)->searchable();

    // Вы также можете добавить записи через отношения...
    $user->orders()->searchable();

    // Вы также можете добавить записи через коллекции...
    $orders->searchable();

Метод `searchable` можно рассматривать как операцию "upsert" (обновить или вставить). Другими словами, если запись модели уже в вашем индексе, она будет обновлена. А если её нет в поисковом индексе, то она будет добавлена в него.

<a name="updating-records"></a>
### Обновление записей

Для обновления модели, предназначенной для поиска, вам надо обновить только свойства экземпляра модели и сохранить (`save`) модель в БД. Scout автоматически внесёт изменения в ваш поисковый индекс:

    $order = App\Order::find(1);

    // Обновить заказ...

    $order->save();

Вы также можете использовать метод `searchable` на запросе Eloquent для обновления коллекции моделей. Если моделей нет в поисковом индексе, они будут созданы:

    // Обновление через запрос Eloquent...
    App\Order::where('price', '>', 100)->searchable();

    // Вы также можете обновить через отношения...
    $user->orders()->searchable();

    // Вы также можете обновить через коллекции...
    $orders->searchable();

<a name="removing-records"></a>
### Удаление записей

Для удаления записи из индекса просто удалите (`delete`) модель из БД. Этот способ удаления совместим даже с [мягко удаляемыми](/docs/{{version}}/eloquent#soft-deleting) моделями:

    $order = App\Order::find(1);

    $order->delete();

Если вам не надо получать модель перед удалением записи, используйте метод `unsearchable` на экземпляре запроса Eloquent или коллекции:

    // Удаление через запрос Eloquent...
    App\Order::where('price', '>', 100)->unsearchable();

    // Также вы можете удалять через отношения...
    $user->orders()->unsearchable();

    // Также вы можете удалять через коллекции...
    $orders->unsearchable();

<a name="pausing-indexing"></a>
### Приостановка индексирования

Иногда бывает необходимо выполнить на модели группу операций Eloquent, при этом не синхронизируя данные модели с поисковым индексом. Это можно сделать методом `withoutSyncingToSearch`. Этот метод принимает единственную анонимную функцию, которая будет выполнен немедленно. Все операции на модели, сделанные в функции, не будут синхронизированы с индексом модели:

    App\Order::withoutSyncingToSearch(function () {
        // Выполнение действий над моделью...
    });

<a name="searching"></a>
## Поиск

Вы можете начать поиск модели методом `search`. Он принимает единственную строку, которая используется для поиска для поиска моделей. Затем вам надо добавить метод `get` к поисковому запросу для получения моделей Eloquent, которые будут найдены по этому запросу:

    $orders = App\Order::search('Star Trek')->get();

Поскольку поиски Scout возвращают коллекцию моделей Eloquent, вы можете вернуть результаты даже напрямую из роута или контроллера и они будут автоматически конвертированы в JSON:

    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return App\Order::search($request->search)->get();
    });

Если вы бы хотели получить необработанные результаты прежде, чем они будут сконвертированы в модели Eloquent, следует использовать метод `raw`:

    $orders = App\Order::search('Star Trek')->raw();

Поисковые запросы, как правило, будут выполняться в индексе, указываемом методом модели [`searchableAs`](#configuring-model-indexes) method. Однако, вы можете использовать метод `within`, чтобы указать собственный индекс:

    $orders = App\Order::search('Star Trek')
        ->within('tv_shows_popularity_desc')
        ->get();

<a name="where-clauses"></a>
### Условия Where

Scout позволяет добавлять простые условия "where" в ваши поисковые запросы. На данный момент эти условия поддерживают только базовые числовые сравнения на равенство и в основном полезны для ограничения поисковых запросов определённым ID. Поскольку поисковые индексы не являются реляционной БД, то более сложные условия "where" пока не поддерживаются:

    $orders = App\Order::search('Star Trek')->where('user_id', 1)->get();

<a name="pagination"></a>
### Страничный вывод

Помимо получения коллекции моделей вы можете разделить результаты поиска на страницы методом `paginate`. Этот метод вернёт экземпляр `Paginator`, как и в случае [страничного вывода обычного запроса Eloquent](/docs/{{version}}/pagination):

    $orders = App\Order::search('Star Trek')->paginate();

Вы можете указать, сколько моделей необходимо размещать на одной странице, передав количество первым аргументом методу `paginate`:

    $orders = App\Order::search('Star Trek')->paginate(15);

Когда вы получите результаты, вы можете отобразить их и вывести страничные ссылки с помощью [Blade](/docs/{{version}}/blade), как и в случае страничного вывода обычного запроса Eloquent:

    <div class="container">
        @foreach ($orders as $order)
            {{ $order->price }}
        @endforeach
    </div>

    {{ $orders->links() }}

<a name="custom-engines"></a>
## Пользовательские движки

#### Создание движка

Если ни один из встроенных поисковых движков Scout вам не подходит, вы можете написать свой собственный движок и зарегистрировать его в Scout. Ваш движок должен наследовать абстрактный класс `Laravel\Scout\Engines\Engine`. Этот абстрактный класс содержит пять методов, которые должен реализовать ваш движок:

    use Laravel\Scout\Builder;

    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function map($results, $model);

Вам может быть полезно посмотреть на реализацию этих методов в классе `Laravel\Scout\Engines\AlgoliaEngine`. Этот класс предоставит вам хорошую отправную точку для изучения того, как реализуется каждый из этих методов.

#### Регистрация движка

После создания своего движка вы можете зарегистрировать его в Scout с помощью метода `extend` менеджера движков Scout. Вам надо вызвать метод `extend` из метода `boot` вашего `AppServiceProvider` или любого другого сервис-провайдера в вашем приложении. Например, если вы создали `MySqlSearchEngine`, зарегистрируйте его так:

    use Laravel\Scout\EngineManager;

    /**
     * Начальная загрузка всех сервисов приложения.
     *
     * @return void
     */
    public function boot()
    {
        resolve(EngineManager::class)->extend('mysql', function () {
            return new MySqlSearchEngine;
        });
    }

После регистрации движка вы можете указать его в качестве драйвера (`driver`) Scout по-умолчанию в конфиге `config/scout.php`:

    'driver' => 'mysql',
