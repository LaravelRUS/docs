git 4ada7c705933b9bf39f51bd8e1f9c71d9fa323b0

---

# HTTP-контроллеры

- [Введение](#introduction)
- [Простейшие контроллеры](#basic-controllers)
- [Использование посредников с контроллерами](#controller-middleware)
- [RESTful ресурс-контроллеры](#restful-resource-controllers)
    - [Ограничение набора действий для ресурса](#restful-partial-resource-routes)
    - [Задание имен для действий с ресурсом](#restful-naming-resource-routes)
    - [Задание имен для параметров](#restful-naming-resource-route-parameters)
    - [Расширение набора действий с ресурсом](#restful-supplementing-resource-controllers)
- [Внедрение зависимостей в контроллерах](#dependency-injection-and-controllers)
- [Кэширование маршрутов](#route-caching)

<a name="introduction"></a>
## Введение

Вместо того, чтобы писать логику обработки запросов в файле `routes.php`, вы можете организовать её, используя классы Controller, которые позволяют группировать связанные обработчики запросов в отдельные классы. Контроллеры хранятся в папке `app/Http/Controllers`.

<a name="basic-controllers"></a>
## Простейшие контроллеры

Все Laravel контроллеры должны расширять базовый класс контроллера, присутствующий в Laravel по умолчанию. Пример простейшего класса контроллера:

    <?php

    namespace App\Http\Controllers;

    use App\User;
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
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

Привязка метода контроллера к маршруту выглядит так:

    Route::get('user/{id}', 'UserController@showProfile');

Теперь при совпадении запроса со строкой 'user/{id}' будет вызван метод showProfile класса UserController. В метод будет передан параметр {id}.

#### Контроллеры и пространства имён

Очень важно отметить, что нам не пришлось указывать всё пространство имён контроллера при опеределении марщрута. Мы определили только ту пространства имен класса, которая идёт после корня  `App\Http\Controllers`. Так происходит, поскольку, по умолчанию, класс RouteServiceProvider загружает все маршруты из файла `routes.php` в группу с этим «корневым» пространством имён.

Если вы решите наследовать или организовать ваши контроллеры используя пространства имён глубже в папке `App\Http\Controllers`, просто используйте название класса относительно корневого пространства имён `App\Http\Controllers`. Таким образом, если ваш полный класс `App\Http\Controllers\Photos\AdminController`, регистрация маршрута будет выглядеть так:

    Route::get('foo', 'Photos\AdminController@method');

#### Именованные роуты с контроллерами

Как и в случае с обработчиками-замыканиями, вы можете присвоить имя этому роуту:

	Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

Вы можете использовать функцию-помощник `route` для создания URl к маршруту
    $url = route('name');

<a name="controller-middleware"></a>
## Использование посредников с контроллерами

[Посредники](/docs/{{version}}/middleware) могут быть привязаны к маршрутам следующим образом:

	Route::get('profile', [
		'middleware' => 'auth',
		'uses' => 'UserController@showProfile'
	]);

Но более удобно назначать посредника в конструкторе контроллера. Используя метод middleware в конструкторе, вы можете легко назначать посредника для контроллера. Вы даже можете использовать посредника только для определенных методов контроллера:

    class UserController extends Controller
    {
        /**
         * Instantiate a new UserController instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');

            $this->middleware('log', ['only' => [
                'fooAction',
                'barAction',
            ]]);
            $this->middleware('subscribed', ['except' => [
                'fooAction',
                'barAction',
            ]]);
        }
    }

<a name="restful-resource-controllers"></a>
## RESTful ресурс-контроллеры

Ресурс-контроллеры упрощают построение RESTful контроллеров, работающих с ресурсами. Например, вы можете создать контроллер, обрабатывающий фотографии, хранимые вашим приложением. Вы можете быстро создать такой контроллер, используя Artisan-команду `make:controller`.

Для создания контроллера выполните следующую консольную команду:

   php artisan make:controller PhotoController --resource

Команда создаст файл контроллера app/Http/Controllers/PhotoController.php. Контроллер будет содержать методы для всех доступных действий с ресурсами.

Далее вы можете создать маршрут-ресурс для этого контроллера:

Route::resource('photo', 'PhotoController');

Это единственное определение маршрута на самом деле описывает несколько маршрутов для обработки различных RESTful действий для ресурса `photo`. Сгенерированный контроллер уже имеет методы-заглушки для каждого из этих маршрутов с комментариями, которые напоминают о том, какие URI и типы запросов они обрабатывают.

#### Действия, обрабатываемые ресурс-контроллером:

Глагол    | путь                  | Действие     | Имя маршрута
----------|-----------------------|--------------|---------------------
GET       | `/photo`              | index        | photo.index
GET       | `/photo/create`       | create       | photo.create
POST      | `/photo`              | store        | photo.store
GET       | `/photo/{photo}`      | show         | photo.show
GET       | `/photo/{photo}/edit` | edit         | photo.edit
PUT/PATCH | `/photo/{photo}`      | update       | photo.update
DELETE    | `/photo/{photo}`      | destroy      | photo.destroy

<a name="restful-partial-resource-routes"></a>
#### Ограничение набора действий для ресурса

При задании ресурс-роута вы можете определить только часть из всех возможных действий:

    Route::resource('photo', 'PhotoController', ['only' => [
        'index', 'show'
    ]]);

    Route::resource('photo', 'PhotoController', ['except' => [
        'create', 'store', 'update', 'destroy'
    ]]);

<a name="restful-naming-resource-routes"></a>
#### Naming Resource Routes

По умолчанию, все действия ресурс контроллеров имеют имя в формате «ресурс.действие». Однако, вы можете изменить эти имена, используя массив `names` в опциях:

    Route::resource('photo', 'PhotoController', ['names' => [
        'create' => 'photo.build'
    ]]);

<a name="restful-naming-resource-route-parameters"></a>
#### Задание имен для действий с ресурсом

По умолчанию статический метод `Route::resource` будет создавать параметры маршрута для вашего ресурса, используя имя ресурса. Вы можете изменить это, оперделив в массиве опций ключ `parameters`. Ключ должен быть массивом, в котором задана пара 'имя ресурса'=>'имя параметра':

    Route::resource('user', 'AdminUserController', ['parameters' => [
        'user' => 'admin_user'
    ]]);

 Пример выше задает следующий URIs для действия `show`:

    /user/{admin_user}

Вместо задания массива для ключа 'parameters' вы можете присвоить ему значение `singular`, что укажет Laravel использовать имена параметров по умолчанию, но в единственном числе ("singularize"):

    Route::resource('users.photos', 'PhotoController', [
        'parameters' => 'singular'
    ]);

    // /users/{user}/photos/{photo}

Альтернативно, вы можете установить глобально: значение `singular`, имена для параметров ресурсов

    Route::singularResourceParameters();

    Route::resourceParameters([
        'user' => 'person', 'photo' => 'image'
    ]);

При настройке параметров ресурса важно помнить приоритет назначения имен:
1. Параметры явно определенные в `Route::resource`.
2. Глобальный параметр установленный посредством `Route::resourceParameters`.
3. Параметр `singular`, заданный через ключ `parameters` в `Route::resource` или посредством `Route::singularResourceParameters`.
4. Поведение по умолчанию.

<a name="restful-supplementing-resource-controllers"></a>
#### Расширение набора действий с ресурсом

Если необходимо определить дополнительные маршруты для ресурс-контроллера помимио маршрутов по умолчанию, вы должны определять их
перед вызовом  `Route::resource`; иначе, маршруты, определенные через метод `resource` могут иметь приоритет над вашими дополнительными маршрутами:

	Route::get('photos/popular', 'PhotoController@method');
	
	Route::resource('photos', 'PhotoController');

<a name="dependency-injection-and-controllers"></a>
## Внедрение зависимостей в контроллерах

#### Внедрение зависимостей в конструкторе
Laravel [сервис-контейнер](/docs/{{version}}/container) используется для поиска и получения всех контроллеров.
За счёт этого вы можете указать любые зависимости в качестве аргументов конструктора вашего контроллера. Зависимости будут автоматически разрешены и внедрены в экземпляр контроллера:

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
		/**
		 * The user repository instance.
		 */
		protected $users;

		/**
		 * Create a new controller instance.
		 *
		 * @param  UserRepository  $users
		 * @return void
		 */
		public function __construct(UserRepository $users)
		{
			$this->users = $users;
		}

	}
	
Конечно, вы можете использовать в качестве зависимости любой контракт [Laravel contract](/docs/{{version}}/contracts). Если контейнер может его разрешить, вы можете его использовать в контроле типов.

#### Внедрение зависимостей в методах контроллеров
Точно так же можно указывать зависимости для любого метода контролера. Например, давайте укажем Illuminate\Http\Request как зависимость для метода `store`:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Store a new user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->input('name');

            //
        }
    }
	
Если метод контроллера так же ожидает какие-либо данные из маршрута, то можно просто указать их после других зависимостей.
Например, если маршрут задан так:

    Route::put('user/{id}', 'UserController@update');

Вы может использовать зависимость от `Illuminate\Http\Request` и иметь доступ к параметру `id`, определив их в методе:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Update the specified user.
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
## Кэширование маршрутов

> **Примечание:** Кэширование не работает с маршрутами на основе замыканий. Для таких маршрутов для использования кэширования требуется заменить замыкания на контроллеры.

Если ваше приложении использует маршруты исключительно на основе контроллеров, вы должны получить примущество кеширования. Использование кэширования значительно уменьшает время для регистрации всех маршрутов приложения. В некоторых случаях получается 100-кратное ускорение регистрации. Для создания кэша маршрутов нужно только выполнить в консоли Artisan команду `route:cache`:and:

	php artisan route:cache

Теперь сгенерированный кэш-файл маршрутов будет использоваться вместо файла `app/Http/routes.php`. Помните, что при добавлении новых или изменении существующих маршрутов необходимо пересоздавать кэш, поэтому лучше использовать кэширование маршрутов только при развёртывании приложения.

Для удаления кэша маршрутов без генерации нового используется Artisan-команда `route:clear`:

	php artisan route:clear

