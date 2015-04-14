git fb85738c37ef8da3e0b3ed2c6e059b94d3d0dbac

---

# Роуты (маршрутизация)

- [Простейшая маршрутизация](#basic-routing)
- [Параметры роутов](#route-parameters)
- [Фильтры роутов](#route-filters)
- [Именованные роуты](#named-routes)
- [Группы роутов](#route-groups)
- [Доменная маршрутизация](#sub-domain-routing)
- [Префикс пути](#route-prefixing)
- [Привязка моделей](#route-model-binding)
- [Ошибки 404](#throwing-404-errors)
- [Маршрутизация в контроллер](#routing-to-controllers)

<a name="basic-routing"></a>
## Простейшая маршрутизация

Большинство роутов (маршруты, routes) вашего приложения будут определены в файле `app/routes.php`. В Laravel простейший роут состоит из URI (урла, пути) и функции-замыкания (она же коллбек).

#### Простейший GET-роут:

	Route::get('/', function()
	{
		return 'Hello World';
	});

#### Простейший POST-роут:

	Route::post('foo/bar', function()
	{
		return 'Hello World';
	});

#### Регистрация роута для нескольких методов

	Route::match(array('GET', 'POST'), '/', function()
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

	$url = URL::to('foo');

Здесь 'foo' - это URI.

<a name="route-parameters"></a>
## Параметры роутов

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
	->where(array('id' => '[0-9]+', 'name' => '[a-z]+'))

#### Определение глобальных паттернов

Вы можете определить соответствие параметра пути регулярному выражению глобально для всех роутов. Например, если у вас `{id}` в урлах это всегда числовое значение:

	Route::pattern('id', '[0-9]+');

	Route::get('user/{id}', function($id)
	{
	    // Only called if {id} is numeric.
	});

#### Доступ к значению параметров роута

Если вам нужно получить значение параметра роута не в нем, а, где-то еще, например, в фильтре или контроллере, то вы можете использовать `Route::input()`:

	Route::filter('foo', function()
	{
	    if (Route::input('id') == 1)
	    {
	        //
	    }
	});

<a name="route-filters"></a>
## Фильтры роутов

Фильтры - удобный механизм ограничения доступа к определённому роуту, что полезно при создании областей сайта только для авторизованных пользователей. В Laravel изначально включено несколько фильтров, в том числе `auth`, `auth.basic`, `guest` and `csrf`. Они определены в файле `app/filters.php`.

#### Регистрация фильтра маршрутов:

	Route::filter('old', function()
	{
		if (Input::get('age') < 200)
		{
			return Redirect::to('home');
		}
	});

Если фильтр возвращает значение, оно используется как ответ фреймворка (response) на сам запрос (request) и обработчик маршрута не будет вызван, и все `after`-фильтры тоже будут пропущены.

#### Привязка фильтра к роуту:

	Route::get('user', array('before' => 'old', function()
	{
		return 'You are over 200 years old!';
	}));

#### Сочетания фильтра и привязки роута к контроллеру

	Route::get('user', array('before' => 'old', 'uses' => 'UserController@showProfile'));	

#### Привязка нескольких фильтров к роуту

	Route::get('user', array('before' => 'auth|old', function()
	{
		return 'You are authenticated and over 200 years old!';
	}));

#### Привязка нескольких фильтров к роуту при помощи массива

	Route::get('user', array('before' => array('auth', 'old'), function()
	{
		return 'You are authenticated and over 200 years old!';
	}));	

#### Передача параметров для фильтра

	Route::filter('age', function($route, $request, $value)
	{
		//
	});

	Route::get('user', array('before' => 'age:200', function()
	{
		return 'Hello World';
	}));

Фильтры типа `after` (выполняющиеся после запроса, если он не был отменён фильтром `before` - прим. пер.) получают `$response` как свой третий аргумент:

	Route::filter('log', function($route, $request, $response)
	{
		//
	});

#### Фильтры по шаблону

Вы можете также указать, что фильтр применяется ко всем роутам, URI (путь) которых соответствует шаблону.

	Route::filter('admin', function()
	{
		//
	});

	Route::when('admin/*', 'admin');

В примере выше фильтр `admin` будет применён ко всем маршрутам, адрес которых начинается с `admin/`. Звёздочка (*) используется как символ подстановки и соответствует любому набору символов, в том числе пустой строке.

Вы также можете привязывать фильтры, зависящие от типа HTTP-запроса:

	Route::when('admin/*', 'admin', array('post'));

#### Классы фильтров

Для продвинутой фильтрации вы можете использовать классы вместо функций-замыканий. Так как такие фильтры будут создаваться с помощью [IoC-контейнера](/docs/4.1/ioc), то вы можете использовать внедрение зависимостей (Dependency Injection) для лучшего тестирования.

#### Определение класса для фильтра:

	Route::filter('foo', 'FooFilter');

По умолчанию будет вызван метод `filter` класса `FooFilter`:	

	class FooFilter {

		public function filter()
		{
			// Логика фильтра...
		}

	}

Вы можете изменить это дефолтное поведение и указать метод явно:

	Route::filter('foo', 'FooFilter@foo');

<a name="named-routes"></a>
## Именованные роуты

Присваивая имена роутам вы можете сделать обращение к ним (при генерации URL во вьюхах (views) или переадресациях) более удобным. Вы можете задать имя роуту таким образом:

	Route::get('user/profile', array('as' => 'profile', function()
	{
		//
	}));

Также можно указать контроллер и его действие:

	Route::get('user/profile', array('as' => 'profile', 'uses' => 'UserController@showProfile'));

Теперь вы можете использовать имя маршрута при генерации URL или переадресации:

	$url = URL::route('profile');

	$redirect = Redirect::route('profile');

Получить имя текущего выполняемого маршрута можно методом `currentRouteName`:

	$name = Route::currentRouteName();

<a name="route-groups"></a>
## Группы роутов

Иногда вам может быть нужно применить фильтры к набору маршрутов. Вместо того, чтобы указывать их для каждого маршрута в отдельности вы можете сгруппировать маршруты:

	Route::group(array('before' => 'auth'), function()
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

Внутри группы вы можете указать параметр `namespace` , чтобы не прописывать неймспейсы к каждому контроллеру:

	Route::group(array('namespace' => 'Admin'), function()
	{
		//
	});

<a name="sub-domain-routing"></a>
## Поддоменные роуты

Роуты Laravel способны работать и с поддоменами по их маске и передавать в ваш обработчик параметры из шаблона.

#### Регистрация роута по поддомену:

	Route::group(array('domain' => '{account}.myapp.com'), function()
	{

		Route::get('user/{id}', function($account, $id)
		{
			//
		});

	});
<a name="route-prefixing"></a>
## Префикс пути

Группа роутов может быть зарегистрирована с одним префиксом в URL без его явного указания, с помощью ключа `prefix` в параметрах группы.

	Route::group(array('prefix' => 'admin'), function()
	{

		Route::get('user', function()
		{
			//
		});

	});

<a name="route-model-binding"></a>
## Привязка моделей к роутам

Привязка моделей - удобный способ передачи экземпляров моделей в ваш роут. Например, вместо передачи ID пользователя вы можете передать модель User, которая соответствует данному ID, целиком. Для начала используйте метод `Route::model` для указания модели, которая должна быть использована вместо данного параметра.

#### Привязка параметра к модели

	Route::model('user', 'User');

Затем зарегистрируйте маршрут, который принимает параметр `{user}`:

	Route::get('profile/{user}', function(User $user)
	{
		//
	});

Из-за того, что мы ранее привязали параметр `{user}` к модели `User`, то её экземпляр будет передан в маршрут. Таким образом, к примеру, запрос `profile/1` передаст объект `User` , который соответствует ID 1 (полученному из БД - прим. пер.).

> **Внимание:** если переданный ID не соответствует строке в БД, будет брошено исключение (Exception) 404.

Если вы хотите задать свой собственный обработчик для события "не найдено", вы можете передать функцию-замыкание в метод `model`:

	Route::model('user', 'User', function()
	{
		throw new NotFoundHttpException;
	});

Иногда вам может быть нужно использовать собственный метод для получения модели перед её передачей в маршрут. В этом случае просто используйте метод `Route::bind`:

	Route::bind('user', function($value, $route)
	{
		return User::where('name', $value)->first();
	});

<a name="throwing-404-errors"></a>
## Ошибки 404

Есть два способа вызвать исключение 404 (Not Found) из маршрута. Первый - методом `App::abort`:

	App::abort(404);

Второй - бросив исключение класса или потомка класса `Symfony\Component\HttpKernel\Exception\NotFoundHttpException`.

Больше информации о том, как обрабатывать исключения 404 и отправлять собственный ответ на такой запрос содержится в разделе [об ошибках](/docs/4.1/errors#handling-404-errors).

<a name="routing-to-controllers"></a>
## Роуты в контроллер

Laravel позволяет вам регистрировать маршруты не только в виде функции-замыкания, но и классов-контроллеров и даже создавать [контроллеры ресурсов](/docs/4.1/controllers#resource-controllers).

Больше информации содержится в разделе [о контроллерах](/docs/4.1/controllers).
