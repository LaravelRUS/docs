git d0ca8271e6081cd9729709701e52104ff89206b3

---

# Ответ (response) и шаблоны (views)

- [Ответ: основы](#basic-responses)
- [Редиректы](#redirects)
- [Вьюхи](#views)
- [Композеры](#view-composers)
- [Особые ответы](#special-responses)
- [Макросы ответов](#response-macros)

<a name="basic-responses"></a>
## Ответ: основы

#### Ответ в виде возврата строки из роута:

	Route::get('/', function()
	{
		return 'Hello world';
	});

#### Создание собственного ответа

Объект `Response` наследует класс `Symfony\Component\HttpFoundation\Response` который предоставляет набор методов для построения HTTP-ответа.

	$response = Response::make($contents, $statusCode);

	$response->header('Content-Type', $value);

	return $response;

#### Добавление cookie к ответу

	$cookie = Cookie::make('name', 'value');

	return Response::make($content)->withCookie($cookie);

<a name="redirects"></a>
## Редиректы

#### Простой редирект

	return Redirect::to('user/login');

#### Переадресация с одноразовыми переменными сессии:
	
	return Redirect::to('user/login')->with('message', 'Войти не удалось');

> **Примечание:** Метод `with` сохраняет данные в сессии, поэтому вы можете прочитать их, используя обычный метод `Session::get`.

#### Переадресация на именованный роут

	return Redirect::route('login');

#### Переадресация на именованный роут с параметрами

	return Redirect::route('profile', array(1));

#### Переадресация на именованный роут с именованными параметрами

	return Redirect::route('profile', array('user' => 1));

#### Переадресация на метод контроллера

	return Redirect::action('HomeController@index');

#### Переадресация на метод контроллера с параметрами

	return Redirect::action('UserController@profile', array(1));

#### Переадресация на метод контроллера с именованными параметрами

	return Redirect::action('UserController@profile', array('user' => 1));

<a name="views"></a>
## Вьюхи

Вьюхи (views, шаблон) обычно содержат HTML-код вашего приложения и представляют собой удобный способ разделения бизнес-логики и логики отображения информации. Вьюхи хранятся в папке `app/views`.

Простая вьюха может иметь такой вид:

	<!-- app/views/greeting.php -->

	<html>
		<body>
			<h1>Привет, <?php echo $name; ?></h1>
		</body>
	</html>

Из этой вьюхи можно сформировать ответ в браузер, например, следующим образом:

	Route::get('/', function()
	{
		return View::make('greeting', array('name' => 'Тейлор'));
	});

Второй параметр, переданный в `View::make` - массив данных, которые будут доступны внутри вьюхи.

#### Передача переменных в вьюху

	// припомощи метода with()
	$view = View::make('greeting')->with('name', 'Стив');

	// при помощи "магического" метода
	$view = View::make('greeting')->withName('Стив');

В примере выше переменная `$name` будет доступна в шаблоне и будет иметь значение `Стив`.

Можно передать массив данных в виде второго параметра для метода `make`:

	$view = View::make('greetings', $data);

Вы также можете установить глобальную переменную, которая будет видна во всех шаблонах:

	View::share('name', 'Стив');

#### Передача вложенного шаблона в шаблон

Иногда вам может быть нужно передать вьюху внутрь другой вьюхи. Например, передаем `app/views/child/view.php` внутрь `app/views/greeting.php`:

	$view = View::make('greeting')->nest('child', 'child.view');

	// с передачей переменных
	$view = View::make('greeting')->nest('child', 'child.view', $data);

Затем вложенная вьюха может быть отображёна в `app/views/greeting.php`:

	<html>
		<body>
			<h1>Привет!</h1>
			<?php echo $child; ?>
		</body>
	</html>

> **Примечание** Для работы с вложенными вьюхами смотрите также `@include` шаблонизатора [Blade](/docs/template)	

<a name="view-composers"></a>
## Композеры

Композеры (view composers) - функции-замыкания или методы класса, которые вызываются, когда вьюха рендерится в строку. Если у вас есть данные, которые вы хотите привязать к вьюхе при каждом её формировании, то композеры помогут вам выделить такую логику в отдельное место. Можно сказать, композеры - это модели вьюх.

#### Регистрация композера

	View::composer('profile', function($view)
	{
		$view->with('count', User::count());
	});

Теперь при каждом отображении шаблона `profile` к нему будет привязана переменная `count`.

Вы можете привязать композер сразу к нескольким вьюхам:

    View::composer(array('profile','dashboard'), function($view)
    {
        $view->with('count', User::count());
    });

Если вам больше нравится использовать классы (что позволит вам регистрировать несколько композеров в [IoC Container](/docs/ioc) и ресолвить нужный в сервис-провайдере), то вы можете сделать так:

	View::composer('profile', 'ProfileComposer');

Класс композера должен иметь следующий вид:

	class ProfileComposer {

		public function compose($view)
		{
			$view->with('count', User::count());
		}

	}

#### Регистрация нескольких композеров

Вы можете использовать метод `composers` чтобы зарегистрировать несколько композеров одновременно:

	View::composers(array(
	    'AdminComposer' => array('admin.index', 'admin.profile'),
	    'UserComposer' => 'user',
	));	

> **Примечание** Обратите внимание, что нет строгого правила, где должны храниться классы-композеры. Вы можете поместить их в любое место, где их сможет найти автозагрузчик в соответствии с директивами в вашем файле `composer.json`.

### Криейтор (view creators)

Криейторы вьюх работают почти так же, как композеры , но вызываются сразу после создания объекта вьюхи, а не во время её отображения в строку. Для регистрации используйте метод `creator`:

	View::creator('profile', function($view)
	{
		$view->with('count', User::count());
	});

<a name="special-responses"></a>
## Особые ответы (response)

#### Создание JSON-ответа

	return Response::json(array('name' => 'Стив', 'state' => 'CA'));

#### Создание JSONP-ответа

	return Response::json(array('name' => 'Стив', 'state' => 'CA'))->setCallback(Input::get('callback'));

#### Создание ответа передачи файла

	return Response::download($pathToFile);

	return Response::download($pathToFile, $name, $headers);

> **Примечание** Symfony HttpFoundation, при помощи которого реализована отдача файла, требует, чтобы имя файла состояло из ASCII-символов.

<a name="response-macros"></a>
## Макросы ответов

Если вы хотите использовать особый ответ в нескольких роутах или контроллерах, то вы можете определить свой макрос ответа:

	Response::macro('caps', function($value)
	{
	    return Response::make(strtoupper($value));
	});

Использование:

	return Response::caps('foo');

Определения макросов ответов должны располагаться в одном из ваших [старт-файлов](/docs/lifecycle#start-files).

