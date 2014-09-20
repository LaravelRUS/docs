git a06af424f657c4fc4ec53e188aeaff5c9b64b154

---

# Контроллеры

- [Простейшие контроллеры](#basic-controllers)
- [Фильтры контроллеров](#controller-filters)
- [Единые контроллеры](#implicit-controllers)
- [RESTful ресурс контроллеры](#restful-resource-controllers)
- [Обработка неопределённых методов](#handling-missing-methods)

<a name="basic-controllers"></a>
## Простейшие контроллеры

Вместо того, чтобы писать логику маршрутизации в файле `routes.php`, вы можете организовать это, используя классы Controller. Контроллеры могут группировать связанную логику в отдельные классы, а кроме того использовать дополнительные возможности Laravel, такие как автоматическое [внедрение зависимостей](/docs/ioc).

Контроллеры обычно хранятся в папке `app/Http/Controllers`. Однако, контроллеры могут находиться в любой папке или подпапке. Декларация маршрутов не зависит от местонахождения класса контроллера на диске. Пока Composer знает, откуда загружать класс контроллера, он может находиться там, где вам захочется.

Вот пример простейшего класса контроллера:

	namespace App\Http\Controllers;

    	use View, App\User;
    	use Illuminate\Routing\Controller;

    	class UserController extends Controller {

    		/**
    		 * Show the profile for the given user.
    		 */
    		public function showProfile($id)
    		{
    			$user = User::find($id);

    			return View::make('user.profile', ['user' => $user]);
    		}

    	}

Все контроллеры должны наследовать класс `Illuminate\Routing\Controller`. Теперь, мы можем обратиться к действию (action) этого контроллера:

	Route::get('user/{id}', 'UserController@showProfile');

Очень важно отметить, что нам не пришлось указывать всё пространство имён контроллера, только часть названия класса, которая идёт после `App\Http\Controllers` - "корневого" пространства имён. Благорадя вызову вспомогательного метода `namespaced` в вашем классе `App\Providers\RouteServiceProvider`, это "корневое" пространство имён будет автоматически добавляться ко всем маршрутам контроллера, которые вы зарегистрируете.

Если вы решите наследовать или организовать ваши контроллеры используя пространства имён глубже в папке `App\Http\Controllers`, просто используйте название класса относительно корневого пространства имён `App\Http\Controllers`. Таким образом, если ваш полный класс `App\Http\Controllers\Photos\AdminController`, регистрация маршрута будет выглядеть таким образом:

	Route::get('foo', 'Photos\AdminController@method');

> **Примечание:** Так как мы используем [Composer](http://getcomposer.org) для автоматической загрузки наших PHP классов, контроллеры могут располагаться в любом месте нашей файловой системы, лишь бы Composer знал, как их загрузить. Папка, содержащая контроллеры, не навязывает вам никакой структуры построения. Маршруты к контроллерам полностью зависят от особенностей вашей файловой системы.

Вы также можете присвоить имя этому маршруту:

	Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

Для создания ссылки к действию контроллера, можно воспользоваться методом `URL::action` или методом `action` вспомогательного класса:

	$url = URL::action('FooController@method');

	$url = action('FooController@method');

**Ещё раз**, необходимо указать только часть названия класса, которая следует за `App\Http\Controllers` - "корневым" пространством имён. Для создания ссылки на метод контроллера, используя полное имя класса, без помощи автоматического добавления корневого пространства имён, можно использовать косую черту (`\`):

	$url = action('\Namespace\FooController@method');

Получить имя действия, которое выполняется в данном запросе, можно методом `currentRouteAction`:

	$action = Route::currentRouteAction();

<a name="controller-filters"></a>
## Фильтры контроллеров

[Фильтры](/docs/routing#route-filters) могут указываться для маршрутов контроллера аналогично "обычным" маршрутам:

		Route::get('profile', ['before' => 'auth', 'uses' => 'UserController@showProfile']);


Однако вы можете указывать их и внутри самого контроллера:

	class UserController extends Controller {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->beforeFilter('auth', array('except' => 'getLogin'));

			$this->beforeFilter('csrf', array('on' => 'post'));

			$this->afterFilter('log', array('only' =>
								array('fooAction', 'barAction')));
		}

	}

Можно устанавливать фильтры в виде функции-замыкания:

	class UserController extends Controller {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->beforeFilter(function()
			{
				//
			});
		}

	}

Если понадобится использовать метод в контроллере как фильтр, используйте `@` для задания фильтр:

	class UserController extends Controller {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->beforeFilter('@filterRequests');
		}

		/**
		 * Filter the incoming requests.
		 */
		public function filterRequests($route, $request)
		{
			//
		}

	}


<a name="implicit-controllers"></a>
## Единые контроллеры

Laravel позволяет вам легко создавать единый маршрут для обработки всех действий контроллера. Для начала, зарегистрируйте маршрут методом `Route::controller`:

	Route::controller('users', 'UserController');

Метод `controller` принимает два аргумента. Первый - корневой URI (путь), который обрабатывает данный контроллер, в то время как второй - имя класса самого контроллера. Далее, просто добавьте методы в этот контроллер с префиксом в виде типа HTTP-запроса (HTTP verb), который они обрабатывают.

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

Если имя действия вашего контроллера состоит из нескольких слов вы можете обратиться к нему по URI, используя синтаксис с дефисами (-). Например, данное действие в нашем классе `UserController` будет доступен по адресу `users/admin-profile`:

	public function getAdminProfile() {}

<a name="restful-resource-controllers"></a>
## RESTful ресурс контроллеры

Ресурс контроллеры упрощают построение RESTful контроллеров, работающих с ресурсами. Например, вы можете создать контроллер, обрабатывающий фотографии, хранимые вашим приложением. Вы можете быстро создать такой контроллер с помощью команды `controller:make` интерфейса (Artisan CLI) и метода `Route::resource`.

Для создания контроллера выполните следующую консольную команду:

	php artisan controller:make PhotoController

Теперь мы можем зарегистрировать его как ресурс контроллер:

	Route::resource('photo', 'PhotoController');

Эта единственная декларация маршрута создаёт множество маршрутов для обработки различных RESTful действий в ресурсе `photo`. Сгенерированный контроллер уже имеет методы-заглушки для каждого из этих маршрутов с комментариями, которые напоминают вам о том, какие URI и типы запросов они обрабатывают.

#### Действия, обрабатываемые ресурс контроллером:

Verb      | Path                        | Action       | Route Name
----------|-----------------------------|--------------|---------------------
GET       | /resource                   | index        | resource.index
GET       | /resource/create            | create       | resource.create
POST      | /resource                   | store        | resource.store
GET       | /resource/{resource}        | show         | resource.show
GET       | /resource/{resource}/edit   | edit         | resource.edit
PUT/PATCH | /resource/{resource}        | update       | resource.update
DELETE    | /resource/{resource}        | destroy      | resource.destroy

Иногда вам понадобится обрабатывать только часть всех возможных действий:

	Route::resource('photo', 'PhotoController',
					['only' => ['index', 'show']]);

	Route::resource('photo', 'PhotoController',
    					['except' => ['create', 'store', 'update', 'destroy']]);

По умолчанию, все действия ресурс контроллеров имеют имя маршрута. Однако, вы можете изменить эти имена, используя массив `names` в опциях:

	Route::resource('photo', 'PhotoController',
					['names' => ['create' => 'photo.build']]);

#### Обработка наследуемых ресурс контроллеров

Для того чтобы "наследовать" контроллеры ресурсов, используйте синтаксис с разделением точкой (`.`) при регистрации маршрутов:

	Route::resource('photos.comments', 'PhotoCommentController');

Этот маршрут зарегистрирует "унаследованный" контроллер ресурсов, который может принимать URL такие как: `photos/{photoResource}/comments/{commentResource}`.

	class PhotoCommentController extends Controller {

		public function show($photoId, $commentId)
		{
			//
		}

	}

#### Добавление дополнительных маршрутов к ресурс контроллерам

Если вдруг необходимо добавить дополнительные маршруты к уже существующим маршрутам ресурс контроллера, необходимо зарегистрировать эти маршруты перед вызовом метода `Route::resource`:

	Route::get('photos/popular');
	Route::resource('photos', 'PhotoController');

<a name="handling-missing-methods"></a>
## Обработка несуществующих методов

При использовании метода `Route::controller`, можно определить "catch-all" метод, который будет вызываться, когда в контроллере нет соответствующего метода. Он должен называться `missingMethod` и принимать массив параметров запроса в виде единственного своего аргумента.

#### Определение Catch-All метода

	public function missingMethod($parameters = array())
	{
		//
	}

Если вы используете ресурс контроллеры, необходимо задать магический метод `__call` в контроллере для обработки несуществующих методов.