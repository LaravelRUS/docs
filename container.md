git 3653b7ab270221d7e45f9d47031faeb4a2cfbeb9

---

# Сервис-контейнер

- [Введение](#introduction)
- [Связывание](#binding)
    - [Основы связывания](#binding-basics)
    - [Связывание интерфейса с реализацией](#binding-interfaces-to-implementations)
    - [Контекстное связывание](#contextual-binding)
    - [Тегирование](#tagging)
- [Применение на практике](#resolving)
    - [Метод Make](#the-make-method)
    - [Автоматическое внедрение](#automatic-injection)
- [События контейнера](#container-events)

<a name="introduction"></a>
## Введение

Сервис-контейнер в Laravel — это мощное средство для управления зависимостями классов и внедрения зависимостей. Внедрение зависимостей — это распространенный термин, который означает добавление других классов в этот класс через конструктор или, в некоторых случаях, метод-сеттер.

Давайте взглянем на простой пример:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Repositories\UserRepository;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Внедрение репозитория пользователя.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Создание нового экземпляра контроллера.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Показать профиль переданного пользователя.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            $user = $this->users->find($id);

            return view('user.profile', ['user' => $user]);
        }
    }

В этом примере `UserController` должен получить пользователей из хранилища данных. Поэтому мы будем **внедрять** сервис, который может получить пользователей. В данном контексте наш `UserRepository` скорее всего использует [Eloquent](/docs/{{version}}/eloquent) для получения информации пользователя из базы данных. Однако, так как внедряется репозиторий, мы можем легко подменить его с другой реализацией. Также можно легко создать "заглушку" или фиктивную реализацию `UserRepository` при тестировании нашего приложения.

Глубокое понимание сервис-контейнера Laravel важно для создания мощного, высокопроизводительного приложения, а также для работы с самим ядром Laravel.

<a name="binding"></a>
## Связывание

<a name="binding-basics"></a>
### Основы связывания

Поскольку почти все ваши привязки сервис-контейнеров будут зарегистрированы в [сервис-провайдерах](/docs/{{version}}/providers), то все следующие примеры демонстрируют использование контейнеров в данном контексте.

> {tip} Если классы не зависят от каких-либо интерфейсов, то нет необходимости связывать их в контейнере. Не нужно объяснять контейнеру, как создавать эти объекты, поскольку он автоматически извлекает такие 
 объекты при помощи рефлексии.

#### Простые связывания

В сервис-провайдере всегда есть доступ к контейнеру через свойство `$this->app`. Зарегистрировать привязку можно методом `bind`, передав имя того класса или интерфейса, который мы хотим зарегистрировать, вместе с функцией-замыкания `Closure`, которая возвращает экземпляр класса:

    $this->app->bind('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

Обратите внимание, что мы получаем сам контейнер в виде аргумента ресолвера. Затем мы можем использовать контейнер, чтобы получать под-зависимости создаваемого объекта.

#### Привязка синглтона

Метод `singleton` привязывает класс или интерфейс к контейнеру, который должен быть создан только один раз, и все последующие обращения к нему будут возвращать этот созданный экземпляр:

    $this->app->singleton('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

#### Привязка экземпляра

Вы можете также привязать существующий экземпляр объекта к контейнеру, используя метод `instance`. Данный экземпляр будет всегда возвращаться при последующих обращениях к контейнеру:

    $api = new HelpSpot\API(new HttpClient);

    $this->app->instance('HelpSpot\API', $api);

#### Связывание примитивов

Иногда у вас может быть класс, который получает некие внедрённые классы, но которому также требуется внедрение примитивных значений, таких как целые числа. Вы можете легко использовать контекстную привязку для внедрения любых значений, которые могут понадобиться вашему классу:

    $this->app->when('App\Http\Controllers\UserController')
              ->needs('$variableName')
              ->give($value);

<a name="binding-interfaces-to-implementations"></a>
### Связывание интерфейса с реализацией

Довольно мощная функция сервис-контейнера — возможность связать интерфейс с реализацией. Например, допустим у нас есть интерфейс `EventPusher` и реализация `RedisEventPusher`. И как только мы написали реализацию `RedisEventPusher` этого интерфейса, мы можем зарегистрировать его в сервис-контейнере следующим образом:

    $this->app->bind(
        'App\Contracts\EventPusher',
        'App\Services\RedisEventPusher'
    );

Мы сообщаем контейнеру, что он должен внедрить `RedisEventPusher`, когда классу потребуется реализация `EventPusher`. Теперь мы можем указать интерфейс `EventPusher` в качестве аргумента метода контроллера в конструкторе, либо в любом другом месте, где сервис-контейнер внедряет зависимости:

    use App\Contracts\EventPusher;

    /**
     * Создание нового экземпляра класса.
     *
     * @param  EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }

<a name="contextual-binding"></a>
### Контекстное связывание

Иногда у вас может быть два класса, которые используют один интерфейс. Но вы хотите внедрить различные реализации в каждый класс. Например, два контроллера могут зависеть от различных реализаций [контракта](/docs/{{version}}/contracts) `Illuminate\Contracts\Filesystem\Filesystem`. Laravel предоставляет простой и гибкий интерфейс для описания такого поведения:

    use Illuminate\Support\Facades\Storage;
    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\VideoController;
    use Illuminate\Contracts\Filesystem\Filesystem;

    $this->app->when(PhotoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('local');
              });

    $this->app->when(VideoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('s3');
              });

<a name="tagging"></a>
### Тегирование

Иногда вам может потребоваться получить все реализации в определенной категории. Например, вы пишете сборщик отчётов, который принимает массив различных реализаций интерфейса `Report`. После регистрации реализаций `Report` вы можете присвоить им тег, используя метод `tag`:

    $this->app->bind('SpeedReport', function () {
        //
    });

    $this->app->bind('MemoryReport', function () {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

Теперь вы можете получить их по тегу методом `tagged`:

    $this->app->bind('ReportAggregator', function ($app) {
        return new ReportAggregator($app->tagged('reports'));
    });

<a name="resolving"></a>
## Применение на практике

<a name="the-make-method"></a>
#### Метод `make`

Вы можете использовать метод `make` для получения экземпляра класса из контейнера. Метод `make` принимает имя класса или интерфейса, который вы хотите получить:

    $api = $this->app->make('HelpSpot\API');

Если вы в месте своего кода, откуда нет доступа к переменной `$app`, то можете использовать глобальный хелпер `resolve`:

    $api = resolve('HelpSpot\API');

В некоторых зависимостях вашего класса, которые нельзя получить через контейнер, можно внедрять их, передавая их в качестве ассоциативного массива в метод `makeWith`:

    $api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);

<a name="automatic-injection"></a>
#### Автоматическое внедрение

И, наконец, самое главное, вы можете просто указать тип зависимости в конструкторе класса, который имеется в контейнере, включая [контроллеры](/docs/{{version}}/controllers), [слушателей событий](/docs/{{version}}/events), [очереди задач](/docs/{{version}}/queues), [посредников](/docs/{{version}}/middleware) и др. Это те способы, с помощью которых получаются большинство объектов из контейнера на практике.

Например, вы можете указать тип репозитория, определённого вашим приложением в конструкторе контроллера. Репозиторий будет автоматически получен и внедрён в класс:

    <?php

    namespace App\Http\Controllers;

    use App\Users\Repository as UserRepository;

    class UserController extends Controller
    {
        /**
         * Экземпляр репозитория пользователя.
         */
        protected $users;

        /**
         * Создание нового экземпляра контроллера.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Показать пользователя с данным ID.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            //
        }
    }

<a name="container-events"></a>
## События контейнера

Контейнер создаёт событие каждый раз, когда из него извлекается объект. Вы можете слушать эти события, используя метод `resolving`:

    $this->app->resolving(function ($object, $app) {
        // Вызывается при извлечении объекта любого типа...
    });

    $this->app->resolving(HelpSpot\API::class, function ($api, $app) {
        // Вызывается при извлечении объекта типа "HelpSpot\API"...
    });

Как видите, объект, получаемый из контейнера, передаётся в функцию обратного вызова, что позволяет вам задать любые дополнительные свойства для объекта перед тем, как отдать его тому, кто его запросил.
