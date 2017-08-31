git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Контроллеры

- [Введение](#introduction)
- [Простейшие контроллеры](#basic-controllers)
    - [Определение контроллеров](#defining-controllers)
    - [Контроллеры и пространства имён](#controllers-and-namespaces)
    - [Контроллеры одного действия](#single-action-controllers)
- [Посредник контроллера](#controller-middleware)
- [Контроллеры ресурсов](#resource-controllers)
    - [Частичные роуты ресурсов](#restful-partial-resource-routes)
    - [Именование роутов ресурсов](#restful-naming-resource-routes)
    - [Именование параметров роутов ресурса](#restful-naming-resource-route-parameters)
    - [Локализация URI ресурсов](#restful-localizing-resource-uris)
    - [Добавление дополнительных роутов в контроллеры ресурсов](#restful-supplementing-resource-controllers)
- [Внедрение зависимостей и контроллеры](#dependency-injection-and-controllers)
- [Кэширование роутов](#route-caching)

<a name="introduction"></a>
## Введение

Вместо того, чтобы определять всю логику обработки запросов в виде замыканий в файлах роутов, вы можете организовать её с помощью классов контроллеров. Контроллеры могут группировать связанную с обработкой HTTP-запросов логику в отдельный класс. Контроллеры хранятся в директории `app/Http/Controllers`.

<a name="basic-controllers"></a>
## Простейшие контроллеры

<a name="defining-controllers"></a>
### Определение контроллеров

Ниже приведён пример простейшего класса контроллера. Обратите внимание, контроллер наследует базовый класс контроллера, встроенный в Laravel. Базовый класс предоставляет несколько удобных методов, таких как метод `middleware`, используемый для назначения посредников на действия контроллера:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Показать профиль данного пользователя.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

Вы можете задать роут для этого контроллера следующим образом:

    Route::get('user/{id}', 'UserController@show');

Теперь при соответствии запроса указанному URI роута будет выполняться метод `show` класса `UserController`. Конечно, параметры роута также будут переданы в метод.

> {tip} Контроллерам не **обязательно** наследовать базовый класс. Но тогда у вас не будет таких удобных возможностей, как методы `middleware`, `validate` и `dispatch`.

<a name="controllers-and-namespaces"></a>
### Контроллеры и пространства имён

Важно помнить, что при определении роута контроллера нам не надо указывать полное пространство имён контроллера. Так как `RouteServiceProvider` загружает файлы вашего роута в группу роута, которая содержит пространство имён, мы указали только ту часть имени класса, которая следует за частью `App\Http\Controllers` пространства имён.

Если вы решите разместить свои контроллеры в поддиректориях `App\Http\Controllers`, то просто используйте конкретное имя класса относительно корня пространства имён `App\Http\Controllers`. Тогда, если полный путь к вашему классу будет `App\Http\Controllers\Photos\AdminController`, то вам надо зарегистрировать роуты к контроллеру следующим образом:

    Route::get('foo', 'Photos\AdminController@method');

<a name="single-action-controllers"></a>
### Контроллеры одного действия

Для определения контроллера, обрабатывающего всего одно действие, поместите в контроллер единственный метод `__invoke`:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class ShowProfile extends Controller
    {
        /**
         * Показать профиль данного пользователя.
         *
         * @param  int  $id
         * @return Response
         */
        public function __invoke($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

При регистрации роутов для контроллеров одного действия вам не надо указывать метод:

    Route::get('user/{id}', 'ShowProfile');

<a name="controller-middleware"></a>
## Посредник контроллера

[Посредников](/docs/{{version}}/middleware) можно назначить роутам контроллера в файлах роутов:

    Route::get('profile', 'UserController@show')->middleware('auth');

Но удобнее указать посредника в конструкторе вашего контроллера. Используя метод `middleware` в конструкторе контроллера, вы легко можете назначить посредника для действия контроллера. Вы можете даже ограничить использование посредника, назначив его только для определённых методов класса контроллера:

    class UserController extends Controller
    {
        /**
         * Создание нового экземпляра контроллера.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');

            $this->middleware('log')->only('index');

            $this->middleware('subscribed')->except('store');
        }
    }

Контроллеры также позволяют регистрировать посредников с помощью замыканий. Это удобный способ определения посредника для одного контроллера, не требующий определения целого класса посредника:

    $this->middleware(function ($request, $next) {
        // ...

        return $next($request);
    });

> {tip} Вы можете назначить посредника на определённый набор действий контроллера, но если возникает такая необходимость, возможно ваш контроллер стал слишком велик. Вместо этого вы можете разбить контроллер на несколько меньших контроллеров.

<a name="resource-controllers"></a>
## Контроллеры ресурсов

Маршрутизация ресурсов Laravel назначает обычные CRUD-роуты на контроллеры одной строчкой кода. Например, вы можете создать контроллер, обрабатывающий все HTTP-запросы к фотографиям, хранимым вашим приложением. Вы можете быстро создать такой контроллер с помощью Artisan-команды `make:controller`:

    php artisan make:controller PhotoController --resource

Эта команда сгенерирует контроллер `app/Http/Controllers/PhotoController.php`. Данный контроллер будет содержать метод для каждой доступной операции с ресурсами.

Теперь мы можем зарегистрировать роут контроллера ресурса:

    Route::resource('photos', 'PhotoController');

Один этот вызов создаёт множество роутов для обработки различных действий для ресурса. Сгенерированный контроллер уже имеет методы-заглушки для каждого из этих действий с комментариями о том, какие URI и типы запросов они обрабатывают.

#### Действия, обрабатываемые контроллером ресурсов

Операция      | URI                  | Действие       | Название роута
----------|-----------------------|--------------|---------------------
GET       | `/photos`              | index        | photos.index
GET       | `/photos/create`       | create       | photos.create
POST      | `/photos`              | store        | photos.store
GET       | `/photos/{photo}`      | show         | photos.show
GET       | `/photos/{photo}/edit` | edit         | photos.edit
PUT/PATCH | `/photos/{photo}`      | update       | photos.update
DELETE    | `/photos/{photo}`      | destroy      | photos.destroy

#### Указание модели ресурса

Если вы используете связывание моделей роутов и хотели бы, чтобы методы контроллера ресурсов указывали в качестве аргумента экземпляр модели, можно использовать опцию `--model` при генерировании контроллера:

    php artisan make:controller PhotoController --resource --model=Photo

#### Подмена методов формы

Поскольку HTML-формы не могут выполнять запросы `PUT`, `PATCH` или `DELETE`, вам надо добавить скрытое поле `_method` для подмены этих HTTP-запросов. Хелпер `method_field` создаст это поле для вас:

    {{ method_field('PUT') }}

<a name="restful-partial-resource-routes"></a>
### Частичные роуты ресурсов

При объявлении роута вы можете указать подмножество всех возможных действий, которые должен обрабатывать контроллер вместо полного набора стандартных действий:

    Route::resource('photo', 'PhotoController', ['only' => [
        'index', 'show'
    ]]);

    Route::resource('photo', 'PhotoController', ['except' => [
        'create', 'store', 'update', 'destroy'
    ]]);

<a name="restful-naming-resource-routes"></a>
### Именование роутов ресурса

По умолчанию все действия контроллера ресурсов имеют имена роутов, но вы можете переопределить эти имена, передав массив `names` вместе с остальными параметрами:

    Route::resource('photo', 'PhotoController', ['names' => [
        'create' => 'photo.build'
    ]]);

<a name="restful-naming-resource-route-parameters"></a>
### Именование параметров роута ресурса

По умолчанию `Route::resource` создаст параметры для ваших роутов ресурсов на основе имени ресурса в единственном числе. Это легко можно изменить для каждого ресурса, передав `parameters` в массив опций. Массив `parameters` должен быть ассоциативным массивом имён ресурсов и имён параметров:

    Route::resource('user', 'AdminUserController', ['parameters' => [
        'user' => 'admin_user'
    ]]);

 Этот пример генерирует следующие URI для роута ресурса `show`:

    /user/{admin_user}

<a name="restful-localizing-resource-uris"></a>
### Локализация URI ресурсов

По умолчанию `Route::resource` будет создавать URI ресурсов, используя английские глаголы. Если вам нужно локализовать глаголы действий `create` и `edit`, вы можете использовать метод `Route::resourceVerbs`. Данную задачу можно выполнить в методе `boot` вашего `AppServiceProvider`:

    use Illuminate\Support\Facades\Route;

    /**
     * Первоначальная загрузка любых сервисов приложения.
     *
     * @return void
     */
    public function boot()
    {
        Route::resourceVerbs([
            'create' => 'crear',
            'edit' => 'editar',
        ]);
    }

Как только глаголы были настроены, регистрация роута ресурса, такая как `Route::resource('fotos', 'PhotoController')`, воспроизведет следующие URI:

    /fotos/crear

    /fotos/{foto}/editar

<a name="restful-supplementing-resource-controllers"></a>
### Добавление дополнительных роутов в контроллеры ресурсов

Если вам надо добавить дополнительные роуты в контроллер ресурсов, не входящие в набор роутов ресурсов по умолчанию, их надо определить до вызова `Route::resource`; в ином случае, определенные методом `resource` роуты могут нечаянно презвойти по важности ваши дополнительные роуты:

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

> {tip} Старайтесь, чтобы контроллеры были узкоспециализированными. Если вам постоянно требуются методы вне стандартного набора действий с ресурсами, попробуйте разделить контроллер на два небольших контроллера.

<a name="dependency-injection-and-controllers"></a>
## Внедрение зависимостей и контроллеры

#### Внедрение в конструктор

[Сервис-контейнер](/docs/{{version}}/container) Laravel используется для работы всех контроллеров Laravel. В результате вы можете указывать типы любых зависимостей, которые могут потребоваться вашему контроллеру в его конструкторе. Заявленные зависимости будут автоматически получены и внедрены в экземпляр контроллера:

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

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
    }

Разумеется, вы можете также указать в качестве аргумента тип любого [Laravel-контракта](/docs/{{version}}/contracts). Если контейнер может с ним работать, значит вы можете указывать его тип. В некоторых случаях внедрение зависимостей в контроллер обеспечивает лучшую тестируемость приложения.

#### Внедрение в метод

Кроме внедрения в конструктор, вы также можете указывать типы зависимостей в методах вашего контроллера. Распространённый пример внедрения в метод — внедрение экземпляра `Illuminate\Http\Request` в один из методов контроллера:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Хранить нового пользователя.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->name;

            //
        }
    }

Если метод вашего контроллера также ожидает данные из параметра роута, просто перечислите аргументы роута после остальных зависимостей. Например, если ваш роут определён так:

    Route::put('user/{id}', 'UserController@update');

Вы по-прежнему можете указать тип `Illuminate\Http\Request` в качестве аргумента и обращаться к параметру `id`, определив метод контроллера как указано ниже:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Обновить данного пользователя.
         *
         * @param  Request  $request
         * @param  string  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

<a name="route-caching"></a>
## Кэширование роутов

> {note} Роуты на основе замыканий нельзя кэшировать. Чтобы использовать кэширование роутов, необходимо перевести все роуты замыканий на классы контроллера.

Если ваше приложение единолично использует роуты контроллера, то вы можете воспользоваться преимуществом кэширования роутов в Laravel. Использование кэша роутов радикально уменьшит время, требуемое для регистрации всех роутов вашего приложения. В некоторых случаях регистрация ваших роутов может стать быстрее в 100 раз. Для создания кэша роутов просто выполните Artisan-команду `route:cache`:

    php artisan route:cache

После выполнения этой команды ваши кэшированные роуты будут загружаться при каждом запросе. Помните, после добавления новых роутов, вам необходимо заново сгенерировать свежий кэш роутов. Поэтому нужно выполнить команду `route:cache` уже при развёртывании вашего проекта.

Для очистки кэша роутов используйте команду `route:clear`:

    php artisan route:clear
