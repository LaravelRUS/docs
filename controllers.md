git 77b555a10b132a40fe1f78ae658674cc26b8c95a

---

# HTTP-контроллеры

- [Введение](#introduction)
- [Простейшие контроллеры](#basic-controllers)
- [Использование посредников с контроллерами](#controller-middleware)
- [Единые контроллеры](#implicit-controllers)
- [Фильтры контроллеров](#controller-filters)
- [RESTful ресурс-контроллеры](#restful-resource-controllers)
- [Внедрение зависимостей в контроллерах](#dependency-injection-and-controllers)
- [Кэширование маршрутов](#route-caching)

<a name="introduction"></a>
## Введение

Вместо того, чтобы писать логику обработки запросов в файле `routes.php`, вы можете организовать её, используя классы Controller, которые позволяют группировать связанные обработчики запросов в отдельные классы.

Контроллеры обычно хранятся в папке `app/Http/Controllers`.

<a name="basic-controllers"></a>
## Простейшие контроллеры

Вот пример простейшего класса контроллера:

	<?php namespace App\Http\Controllers;

	use App\Http\Controllers\Controller;

	class UserController extends Controller {

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

Привязка «действия» (action) к определенному маршруту происходит так:

	Route::get('user/{id}', 'UserController@showProfile');

> **Примечание:** Все контроллеры должны наследоваться от базового класса контроллера.

#### Контроллеры и пространства имён

Очень важно отметить, что нам не пришлось указывать всё пространство имён контроллера, только часть названия класса, которая идёт после `App\Http\Controllers` - «корневого» пространства имён.

По умолчанию, класс `RouteServiceProvider` загружает все маршруты из файла `routes.php` в группу с «корневым» пространством имён.

Если вы решите наследовать или организовать ваши контроллеры используя пространства имён глубже в папке `App\Http\Controllers`, просто используйте название класса относительно корневого пространства имён `App\Http\Controllers`. Таким образом, если ваш полный класс `App\Http\Controllers\Photos\AdminController`, регистрация маршрута будет выглядеть так:

	Route::get('foo', 'Photos\AdminController@method');

#### Именованные роуты с контроллерами

Как и в случае с обработчиками-замыканиями, вы можете присвоить имя этому роуту:

	Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

#### Ссылки на «действия» контроллера

Если вы хотите создавать ссылки на «действие» контроллера, используя относительные имена классов, то сначала нужно зарегистрировать «корневое» пространство имён с контроллерами:
    
    URL::setRootControllerNamespace('App\Http\Controllers');

Для непосредственного создания ссылки на «действие» контроллера, воспользуйтесь функцией-помощником `action`:

	$url = action('FooController@method');

Получить имя «действия», которое выполняется в данном запросе, можно методом `currentRouteAction`:

	$action = Route::currentRouteAction();

<a name="controller-middleware"></a>
## Использование посредников с контроллерами

[Посредники](/docs/{{version}}/middleware) могут быть привязаны к маршрутам следующим образом:

	Route::get('profile', [
		'middleware' => 'auth',
		'uses' => 'UserController@showProfile'
	]);
	
Так же, вы можете указывать их и в конструкторе самого контроллера:

	class UserController extends Controller {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->middleware('auth');
			$this->middleware('log', ['only' => ['fooAction', 'barAction']]);
			$this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
        }
            
    }

<a name="implicit-controllers"></a>
## Единые контроллеры

Laravel позволяет вам легко создавать один маршрут для обработки всех действий контроллера. Для начала, зарегистрируйте маршрут методом `Route::controller`:

	Route::controller('users', 'UserController');

Метод `controller` принимает два аргумента. Первый - корневой URI (путь), который обрабатывает данный контроллер, в то время как второй - имя класса самого контроллера. Далее просто добавьте методы в этот контроллер с префиксом в виде типа HTTP-запроса (HTTP verb), который они обрабатывают.

	class UserController extends Controller {

		public function getIndex()
		{
			//
		}

		public function postProfile()
		{
			//
		}

		public function anyLogin()
		{
			//
		}

	}

Методы `index` относятся к корневому URI (пути) контроллера, который, в нашем случае `users`.

Если имя действия вашего контроллера состоит из нескольких слов вы можете обратиться к нему по URI, используя синтаксис с дефисами (-).
Например, данное действие в нашем классе `UserController` будет доступно по адресу `users/admin-profile`:

	public function getAdminProfile() {}

#### Назначение имён 

Если вы хотите задать имена для роутов, регистрируемых при помощи `Route::controller` , вы можете сделать это так:

	Route::controller('users', 'UserController', [
		'anyLogin' => 'user.login',
	]);	
	
Где `anyLogin` - метод класса `UserController`.

<a name="restful-resource-controllers"></a>
## RESTful ресурс-контроллеры

Ресурс-контроллеры упрощают построение RESTful контроллеров, работающих с ресурсами. Например, вы можете создать контроллер, обрабатывающий фотографии, хранимые вашим приложением. Вы можете быстро создать такой контроллер, используя Artisan-команду `make:controller`.

Для создания контроллера выполните следующую консольную команду:

	php artisan make:controller PhotoController

Теперь мы можем зарегистрировать его как ресурс-контроллер:

	Route::resource('photo', 'PhotoController');

Это единственное определение маршрута на самом деле описывает несколько маршрутов для обработки различных RESTful действий для ресурса `photo`.
Сгенерированный контроллер уже имеет методы-заглушки для каждого из этих маршрутов с комментариями, которые напоминают о том, какие URI и типы запросов они обрабатывают.

#### Действия, обрабатываемые ресурс-контроллером:

Verb      | Путь                        | Действие     | Имя маршрута
----------|-----------------------------|--------------|---------------------
GET       | /photo                      | index        | resource.index
GET       | /photo/create               | create       | photo.create
POST      | /photo                      | store        | photo.store
GET       | /photo/{photo}              | show         | photo.show
GET       | /photo/{photo}/edit         | edit         | photo.edit
PUT/PATCH | /photo/{photo}              | update       | photo.update
DELETE    | /photo/{photo}              | destroy      | photo.destroy

#### Настройка маршрутов в ресурс-контроллерах

Иногда вам понадобится обрабатывать только часть всех возможных действий:

	Route::resource('photo', 'PhotoController',
					['only' => ['index', 'show']]);

	Route::resource('photo', 'PhotoController',
					['except' => ['create', 'store', 'update', 'destroy']]);

По умолчанию, все действия ресурс контроллеров имеют имя в формате «ресурс.действие». Однако, вы можете изменить эти имена, используя массив `names` в опциях:

	Route::resource('photo', 'PhotoController',
					['names' => ['create' => 'photo.build']]);

#### Обработка наследуемых ресурс-контроллеров

Для того чтобы "наследовать" контроллеры ресурсов, используйте синтаксис с разделением точкой (`.`) при регистрации маршрутов:

	Route::resource('photos.comments', 'PhotoCommentController');

Этот маршрут зарегистрирует "унаследованный" контроллер ресурсов, который может принимать URL такие как:
`photos/{photos}/comments/{comments}`.

	class PhotoCommentController extends Controller {

		/**
		 * Show the specified photo comment.
		 *
		 * @param  int  $photoId
		 * @param  int  $commentId
		 * @return Response
		 */
		public function show($photoId, $commentId)
		{
			//
		}

	}

#### Добавление дополнительных маршрутов к ресурс контроллерам

Если вдруг необходимо добавить дополнительные маршруты к уже существующим маршрутам ресурс-контроллера, необходимо зарегистрировать эти маршруты
**перед** вызовом метода `Route::resource`:

	Route::get('photos/popular', 'PhotoController@method');
	
	Route::resource('photos', 'PhotoController');

<a name="dependency-injection-and-controllers"></a>
## Внедрение зависимостей в контроллерах

#### Внедрение зависимостей в конструкторе

[Сервис-контейнер](/docs/{{version}}/container) используется для поиска и получения всех контроллеров.
За счёт этого вы можете указать любые зависимости в качестве аргументов конструктора вашего контроллера,
в том числе и любой [контракт](/docs/{{version}}/contracts):

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Repositories\UserRepository;

	class UserController extends Controller {

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

#### Внедрение зависимостей в методах контроллеров

Точно так же можно указывать зависимости для любого метода контролера.
Например, давайте укажем `Request` как зависимость метода `store`:

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller {

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
	
Если метод контроллера так же ожидает какие-либо данные из маршрута, то можно просто указать их **после** всех зависимостей:

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller {

		/**
		 * Store a new user.
		 *
		 * @param  Request  $request
		 * @param  int  $id
		 * @return Response
		 */
		public function update(Request $request, $id)
		{
			//
		}

	}

> **Примечание:** Внедрение зависимостей в методы контроллера полностью дружат с механизмом [связанных моделей](/docs/{{version}}/routing#route-model-binding).
Сервис-контейнер определит, какие из аргументов должны быть связаны с моделями, а какие должны быть внедрены.

<a name="route-caching"></a>
## Кэширование маршрутов

Если ваше приложение использует только контроллеры для обработки маршрутов, вы можете использовать кэширование маршрутов, которое
кардинально уменьшает время, требуемое на разбор и регистрацию всех маршрутов внутри приложения.

В некоторых случаях прирост скорости может достигнуть 100 раз! Для включения кэширования просто выполните Artisan-команду `route:cache`:

	php artisan route:cache

Теперь сгенерированный кэш-файл маршрутов будет использоваться вместо файла `app/Http/routes.php`. Помните, что при добавлении новых или изменении существующих маршрутов необходимо пересоздавать кэш, поэтому лучше использовать кэширование маршрутов только при развёртывании приложения.

Для удаления кэша маршрутов без генерации нового используется Artisan-команда `route:clear`:

	php artisan route:clear

