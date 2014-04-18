git 61f0ebf9994fa1b4702c21e675f02381cf8d1435

---

# Контроллеры

- [Простейшие контроллеры](#basic-controllers)
- [Фильтры контроллеров](#controller-filters)
- [RESTful-контроллеры](#restful-controllers)
- [Контроллеры ресурсов](#resource-controllers)
- [Обработка неопределённых методов](#handling-missing-methods)

<a name="basic-controllers"></a>
## Простейшие контроллеры

Вместо того, чтобы писать код в роутах, вы можете организовать его, используя класс Controller. Контроллеры могут группировать связанную логику в отдельные классы, а кроме того использовать дополнительные возможности Laravel, такие как автоматическое [внедрение зависимостей](/docs/ioc).

Контроллеры обычно хранятся в папке `app/controllers`, а этот путь по умолчанию зарегистрирован в настройке `classmap` вашего файла `composer.json`. Но вообще-то контроллеры могут находиться в любой папке или подпапке проекта - лишь бы была настроена их загрузка в Composer.

Вот пример простейшего класса контроллера:

	class UserController extends BaseController {

		/**
		 * Отобразить профиль соответствующего пользователя.
		 */
		public function showProfile($id)
		{
			$user = User::find($id);

			return View::make('user.profile', array('user' => $user));
		}

	}

Все контроллеры должны наследовать класс `BaseController`. Этот класс также может хранится в папке `app/controllers` и в него можно поместить общую логику для других контроллеров. The `BaseController` расширяет стандартный класс Laravel, `Controller` class. Теперь, определив контроллер, мы можем зарегистрировать роут для его действия (action):

	Route::get('user/{id}', 'UserController@showProfile');

Если вы решили организовать ваши контроллеры в пространстве имён, просто используйте полное имя класса при определении маршрута:

	Route::get('foo', 'Namespace\FooController@method');

> **Примечание:** Так как Laravel использует Composer для автоматической загрузки php-классов, контроллеры могут располагаться в любом месте фоеймворка, лишь бы Composer знал, как их загрузить. Не обязательно держать контролеры в папке `app/controllers`. Роутинг к контроллерам полностью отвязан от особенностей расположения файлов фреймворка.

Вы также можете присвоить имя этому роуту:

	Route::get('foo', array('uses' => 'FooController@method',
											'as' => 'name'));

Вы можете получить URL к экшну (действию, методу контроллера) методом `URL::action`:

	$url = URL::action('FooController@method');

Получить имя экшна, которое выполняется в данном запросе, можно методом `currentRouteAction`:

	$action = Route::currentRouteAction();

<a name="controller-filters"></a>
## Фильтры контроллеров

[Фильтры](/docs/routing#route-filters) могут указываться для роутов в контроллеры аналогично тому, как они указываются для обычных роутов:

	Route::get('profile', array('before' => 'auth',
				'uses' => 'UserController@showProfile'));

Однако вы можете указывать их и внутри самого контроллера:

	class UserController extends BaseController {

		/**
		 * Создать экземпляр класса UserController.
		 */
		public function __construct()
		{
			$this->beforeFilter('auth');

			$this->beforeFilter('csrf', array('on' => 'post'));

			$this->afterFilter('log', array('only' =>
								array('fooAction', 'barAction')));
		}

	}

Можно устанавливать фильтры в виде функции-замыкания:

	class UserController extends BaseController {

		/**
		 * Создать экземпляр класса UserController.
		 */
		public function __construct()
		{
			$this->beforeFilter(function()
			{
				//
			});
		}

	}

Можно выносить фильтр в метод контроллера:

	class UserController extends BaseController {

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
	    protected function filterRequests($route, $request)
	    {
	        //
	    }

	}


<a name="restful-controllers"></a>
## RESTful-контроллеры

Laravel позволяет вам легко создавать единый маршрут для обработки всех действий контроллера используя простую схему именования REST. Для начала зарегистрируйте маршрут методом `Route::controller`:

#### Регистрация RESTful-контроллера:

	Route::controller('users', 'UserController');

Метод `controller` принимает два аргумента. Первый - корневой URI (путь), который обрабатывает данный контроллер, а второй - имя класса самого контроллера.После регистрации просто добавьте методы в этот класс с префиксом в виде типа HTTP-запроса (HTTP verb), который они обрабатывают.

	class UserController extends BaseController {

		public function getIndex()
		{
			//
		}

		public function postProfile()
		{
			//
		}

	}

В этом случае фреймворк будет обрабатывать обращение (GET-запрос) к /users и POST-запрос на /users/profile. На остальные запросы будет отдаваться 404.
Метод `index` обрабатывает корневой URI контроллера.

Если имя экшна вашего контроллера состоит из нескольких слов вы можете обратиться к нему по URI, используя синтаксис с дефисами (-). Например, следующий экшн в нашем классе `UserController` будет доступен по адресу `users/admin-profile`:

	public function getAdminProfile() {}

#### Имена роутов

Имена роутов для RESTful-контроллеров можно задавать следующим образом:

	Route::controller('manager', 'ManagerController', array(
	    'getIndex' => 'manager.index',
	    'getLogout' => 'manager.logout'
	));

<a name="resource-controllers"></a>
## Контроллеры ресурсов

Контроллеры ресурсов упрощают построение RESTful-контроллеров, работающих с ресурсами. Например, вы можете создать контроллер, обрабатывающий фотографии, хранимые вашим приложением. Вы можете быстро создать такой контроллер с помощью команды `controller:make` интерфейса (Artisan) и метода `Route::resource`.

Для создания контроллера выполните следующую консольную команду:

	php artisan controller:make PhotoController

Теперь мы можем зарегистрировать его как контроллер ресурса:

	Route::resource('photo', 'PhotoController');

Этот единственный вызов создаёт множество маршрутов для обработки различный RESTful-действий на ресурсе `photo`. Сам сгенерированный контроллер уже имеет методы-заглушки для каждого из этих маршрутов с комментариями, которые напоминают вам о том, какие типы запросов они обрабатывают.

#### Запросы, обрабатываемые контроллером ресурсов:

Тип       | Путь                  | Действие     | Имя маршрута
----------|-----------------------|--------------|---------------------
GET       | /resource             | index        | resource.index
GET       | /resource/create      | create       | resource.create
POST      | /resource             | store        | resource.store
GET       | /resource/{id}        | show         | resource.show
GET       | /resource/{id}/edit   | edit         | resource.edit
PUT/PATCH | /resource/{id}        | update       | resource.update
DELETE    | /resource/{id}        | destroy      | resource.destroy

Иногда вам может быть нужно обрабатывать только часть всех возможных действий:

	php artisan controller:make PhotoController --only=index,show

	php artisan controller:make PhotoController --except=index

Вы можете указать этот набор и при регистрации маршрута:

	Route::resource('photo', 'PhotoController',
					array('only' => array('index', 'show')));
					
	// либо:

  	Route::resource('photo', 'PhotoController',
                  array('except' => array('create', 'store', 'update', 'destroy')));

По умолчанию имена у роутов совпадают с названиями экшнов контроллеров ресурсов. Однако, вы можете изменить это:

	Route::resource('photo', 'PhotoController',
	                array('names' => array('create' => 'photo.build')));


<a name="handling-missing-methods"></a>
## Обработка неопределённых методов

Можно определить "catch-all" метод, который будет вызываться для обработки запроса, когда в контроллере нет соответствующего метода. Он должен называться `missingMethod` и принимать массив параметров запроса в виде единственного своего аргумента.

#### Определение Catch-All метода

	public function missingMethod($parameters = array())
	{
		//
	}
