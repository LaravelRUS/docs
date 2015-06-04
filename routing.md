git 0e29f42d953c6ad1f0c06968a9b043f808f3ed87

---

# HTTP-маршрутизация

- [Основы](#basic-routing)
- [Защита от CSRF](#csrf-protection)
- [Подмена HTTP-метода](#method-spoofing)
- [Именованные роуты](#named-routes)
- [Группы роутов](#route-groups)
- [Доменная маршрутизация](#sub-domain-routing)
- [Префикс пути](#route-prefixing)
- [Привязка моделей](#route-model-binding)
- [Ошибки 404](#throwing-404-errors)
- [Маршрутизация в контроллер](#routing-to-controllers)

<a name="basic-routing"></a>
## Основы

Большинство роутов (маршруты, routes) вашего приложения будут определены в файле `app/Http/routes.php`, который загружается сервис-провайдером `App\Providers\RouteServiceProvider`. В Laravel простейший роут состоит из URI (урла, пути) и функции-замыкания.

#### Простейший GET-роут:

	Route::get('/', function()
	{
		return 'Hello World';
	});

#### Простейшие роуты различных типов HTTP-запросов:

	Route::post('foo/bar', function()
	{
		return 'Hello World';
	});

	Route::put('foo/bar', function()
	{
		//
	});

	Route::delete('foo/bar', function()
	{
		//
	});

#### Регистрация роута для нескольких типов HTTP-запросов

	Route::match(['get', 'post'], '/', function()
	{
		return 'Hello World';
	});

#### Регистрация роута для любого типа HTTP-запроса:

	Route::any('foo', function()
	{
		return 'Hello World';
	});

#### Регистрация роута, всегда работающего через HTTPS:

	Route::get('foo', array('https', function()
	{
		return 'Must be over HTTPS';
	}));

Вам часто может понадобиться сгенерировать URL к какому-либо роуту - для этого используется метод `URL::to`:

	$url = url('foo');

Здесь 'foo' - это URI.

<a name="csrf-protection"></a>
## Защита от CSRF

Laravel provides an easy method of protecting your application from [cross-site request forgeries](http://en.wikipedia.org/wiki/Cross-site_request_forgery). Cross-site request forgeries are a type of malicious exploit whereby unauthorized commands are performed on behalf of the authenticated user.

Laravel предоставляет встроенное средство защиты от [межсайтовых подделок запросов](https://ru.wikipedia.org/wiki/%D0%9C%D0%B5%D0%B6%D1%81%D0%B0%D0%B9%D1%82%D0%BE%D0%B2%D0%B0%D1%8F_%D0%BF%D0%BE%D0%B4%D0%B4%D0%B5%D0%BB%D0%BA%D0%B0_%D0%B7%D0%B0%D0%BF%D1%80%D0%BE%D1%81%D0%B0) (cross-site request forgeries). Фреймворк автоматически генерит так называемый CSRF-токен для каждой активной сессии. Этот токен можно проверять при обработке запроса - действительно ли запрос послан с вашего сайта, а не с сайта-злоумышленника. Как правило проверяют только POST, PUT и DELETE запросы, так как они могут изменить состояние приложения (внести изменения в БД).

#### Вставка CSRF-токена в форму

    <input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">

То же самое, но с использованием шаблонизатора [Blade](/docs/{{version}}/templates):

	<input type="hidden" name="_token" value="{{ csrf_token() }}">

You do not need to manually verify the CSRF token on POST, PUT, or DELETE requests. The `VerifyCsrfToken` [HTTP middleware](/docs/{{version}}/middleware) will verify token in the request input matches the token stored in the session.    

Вам не нужно проверять вручную соответствие CSRF-токена сессионному в POST, PUT и DELETE запросах, middleware (посредник) `VerifyCsrfToken` делает это автоматически. Вдобавок к полю `_token`, проверяется еще и HTTP-заголовок `X-CSRF-TOKEN`, который часто используется в Javascript-фреймворках.

<a name="method-spoofing"></a>
## Подмена HTTP-метода

HTML-формы не поддерживают методы HTTP-запроса `PUT` или `DELETE`. Для того, чтобы отправить на сервер HTTP-запрос с этими методами, вам нужно добавить в форму скрытый input с именем `_method`. 

Значение этого поля будет восприниматься фреймворком как тип HTTP-запроса. Например:

	<form action="/foo/bar" method="POST">
		<input type="hidden" name="_method" value="PUT">
    	<input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">
    </form>

<a name="route-parameters"></a>
## Параметры роутов

В роутах можно указывать параметры, которые можно получить в виде аргументов функции-замыкания:

	Route::get('user/{id}', function($id)
	{
		return 'User '.$id;
	});

#### Необязательные параметры роута:

	Route::get('user/{name?}', function($name = null)
	{
		return $name;
	});

#### Необязательные параметры со значением по умолчанию:

	Route::get('user/{name?}', function($name = 'John')
	{
		return $name;
	});

#### Роуты с соответствием пути регулярному выражению:

	Route::get('user/{name}', function($name)
	{
		//
	})
	->where('name', '[A-Za-z]+');

	Route::get('user/{id}', function($id)
	{
		//
	})
	->where('id', '[0-9]+');

Конечно, при необходимости вы можете передать массив ограничений (constraints):

	Route::get('user/{id}/{name}', function($id, $name)
	{
		//
	})
	->where(['id' => '[0-9]+', 'name' => '[a-z]+'])

#### Определение глобальных паттернов

Вы можете определить соответствие параметра пути регулярному выражению глобально для всех роутов. Паттерны задаются в методе `before` сервис-провайдера `RouteServiceProvider`. Например, если у вас `{id}` в урлах это всегда числовое значение:

	$router->pattern('id', '[0-9]+');

После того как паттерн определён, он применяется ко все роутам.	

	Route::get('user/{id}', function($id)
	{
	    // Здесь $id будет только числом
	});

#### Доступ к значению параметров роута

Если вам нужно получить значение параметра вне роута, вы можете использовать метод `input()`:

    if ($route->input('id') == 1)
    {
        //
    }

Вы можете обратиться к текущему роуту через объект `Illuminate\Http\Request`. Получить этот объект вы можете при помощи фасада `Request::` или подав его (type-hint) в аргументы конструктора нужного класса или в аргументы функции-замыкания:

	use Illuminate\Http\Request;

	Route::get('user/{id}', function(Request $request, $id)
	{
		if ($request->route('id'))
		{
			//
		}
	});

<a name="named-routes"></a>
## Именованные роуты

Присваивая имена роутам вы можете сделать обращение к ним (при генерации URL в шаблонах (views) или редиректах) более удобным. Вы можете задать имя роуту таким образом:

	Route::get('user/profile', ['as' => 'profile', function()
	{
		//
	}]);

Также можно указать контроллер и его метод:

	Route::get('user/profile', [
        'as' => 'profile', 'uses' => 'UserController@showProfile'
	]);

Теперь вы можете использовать имя маршрута при генерации URL или переадресации:

	$url = route('profile');

	$redirect = redirect()->route('profile');

Получить имя текущего выполняемого роута можно методом `currentRouteName`:

	$name = Route::currentRouteName();

<a name="route-groups"></a>
## Группы роутов

Иногда вам может быть нужно применить фильтры к набору маршрутов. Вместо того, чтобы указывать их для каждого маршрута в отдельности вы можете сгруппировать маршруты:

	Route::group(['middleware' => 'auth'], function()
	{
		Route::get('/', function()
		{
			// К этому маршруту будет привязан фильтр auth.
		});

		Route::get('user/profile', function()
		{
			// К этому маршруту также будет привязан фильтр auth.
		});
	});

Чтобы не писать полный неймспейс к каждому контроллеру, вы можете использовать параметр `namespace` в группе:

	Route::group(['namespace' => 'Admin'], function()
	{
		//
	});

> **Примечание:** По умолчанию `RouteServiceProvider` включает все роуты внутрь неймспейса, определенного в его свойстве `$namespace`.

### Поддоменные роуты

Роуты Laravel способны работать и с поддоменами:

#### Регистрация роута по поддомену:

	Route::group(['domain' => '{account}.myapp.com'], function()
	{

		Route::get('user/{id}', function($account, $id)
		{
			//
		});

	});

### Префикс пути

Группа роутов может быть зарегистрирована с одним префиксом без его явного указания с помощью ключа `prefix` в параметрах группы.

#### Добавление префикса к сгруппированным маршрутам:

	Route::group(['prefix' => 'admin'], function()
	{

		Route::get('user', function()
		{
			//
		});

	});

<a name="route-model-binding"></a>
## Привязка моделей к роутам

Привязка моделей - удобный способ передачи экземпляров классов в ваш роут. Например, вместо передачи ID пользователя вы можете передать модель User, которая соответствует данному ID, целиком. Для начала используйте метод `route` в `RouteServiceProvider::boot` для указания класса, который должен быть внедрён.

#### Привязка параметра к модели

	public function boot(Router $router)
	{
		parent::boot($router);

		$router->model('user', 'App\User');
	}

Затем зарегистрируйте роут, который принимает параметр `{user}`:

	Route::get('profile/{user}', function(App\User $user)
	{
		//
	});

Из-за того, что мы ранее привязали параметр `{user}` к модели `App\User`, то её экземпляр будет передан в маршрут. Таким образом, к примеру, запрос `profile/1` передаст объект `User`, который соответствует ID 1, полученному из БД.

> **Внимание:** если переданный ID не соответствует строке в БД, будет брошено исключение (Exception) 404.

Если вы хотите задать свой собственный обработчик для события "не найдено", вы можете передать функцию-замыкание в метод `model`:

	Route::model('user', 'User', function()
	{
		throw new NotFoundHttpException;
	});

Иногда вам может быть нужно использовать собственный метод для получения модели перед её передачей в маршрут. В этом случае просто используйте метод `Route::bind`. Функция-замыкание принимает в качестве аргумента параметр из урла и должна возвратить экземпляр класса, который должен быть встроен в роут.

	Route::bind('user', function($value)
	{
		return User::where('name', $value)->first();
	});

<a name="throwing-404-errors"></a>
## Ошибки 404

Есть два способа вызвать исключение 404 (Not Found) из маршрута. Первый - при помощи хэлпера `abort`:

	abort(404);

Этот хэлпер бросает исключение `Symfony\Component\HttpFoundation\Exception\HttpException` с кодом ответа 404.

Второй - вы можете сами бросить исключение `Symfony\Component\HttpKernel\Exception\NotFoundHttpException`.

Смотрите также секцию [errors](/docs/{{version}}/errors#http-exceptions) документации.
