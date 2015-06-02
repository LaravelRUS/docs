git c344e6ca5495ae9bc3f1cc62559b4a750a6c2515

---

# HTTP-response (HTTP-ответ)

- [Основы](#basic-responses)
- [Редиректы](#redirects)
- [Особые HTTP-ответы](#special-responses)
- [Макросы](#response-macros)

<a name="basic-responses"></a>
## Основы

HTTP-Response - это ответ фреймворка, который отдается клиенту (обычно это браузер), от которого пришел HTTP-запрос.

Наиболее простой способ создать HTTP-ответ - это возвратить строку в роуте или контроллере.

#### Response в виде возврата строки из роута:

	Route::get('/', function()
	{
		return 'Hello world';
	});

#### Создание своих ответов

Однако чаще в контроллерах вы возвращаете объект `Illuminate\Http\Response` или [шаблон](/docs/{{version}}/views). Возврат объекта `Response` позволяет изменить HTTP-код и заголовки ответа. Этот объект наследуется от `Symfony\Component\HttpFoundation\Response`, который предоставляет разнообразные методы для построения HTTP-ответа:

	use Illuminate\Http\Response;

	return (new Response($content, $status))
	              ->header('Content-Type', $value);

Для удобства вы можете использовать хэлпер `response`:

	return response($content, $status)
	              ->header('Content-Type', $value);

> **Примечание:** Полный список методов `Response` можно увидеть в [документации по API Laravel](http://laravel.com/api/master/Illuminate/Http/Response.html) и [документации по API Symfony](http://api.symfony.com/2.5/Symfony/Component/HttpFoundation/Response.html).

#### Добавление контента в HTTP-ответ

Если вам нужно не просто изменить заголовки, но и вывести какой-то контент, вы можете указать имя шаблона при помощи метода `view()`:

	return response()->view('hello')->header('Content-Type', $type);

#### Добавление куки

	return response($content)->withCookie(cookie('name', 'value'));

<a name="redirects"></a>
## Редиректы

Редирект - это объект класса `Illuminate\Http\RedirectResponse`, фактически это обычный HTTP-ответ без контента с установленным заголовком `Location`.

#### Возвращение редиректа

Есть несколько способов создать объект `RedirectResponse`. Самый простой - воспользоваться хэлпером `redirect`. 

	return redirect('user/login');

#### Возвращение редиректа с flash-данными в сессии

Редирект [с flash-данными в сессии](/docs/{{version}}/session) - типичная задача в случае, когда после POST-запроса надо перейти на страницу с формой и показать ошибки валидации. Записать flash-данные в сессию можно при помощи метода `with()`:

	return redirect('user/login')->with('message', 'Login Failed');

#### Редирект на предыдущий URL

Для перехода назад к форме можно использовать также метод `back()`. Метод withInput() передаст данные, которые пришли от этой формы, для того, чтобы отобразить их в форме и не заставлять пользователя снова вносить их.

	return redirect()->back();

	return redirect()->back()->withInput();

#### Редирект на именованный роут

Если использовать хэлпер `redirect()` без параметров, он вернет объект `Illuminate\Routing\Redirector`, у которого есть несколько интересных методов. При помощи них, например, вы можете сделать редирект на роут по его имени:

	return redirect()->route('login');

#### Редирект на именованный роут с параметрами

Если ваш роут содержит параметры, то передать их вы можете так:

	return redirect()->route('profile', [1]);

Если параметр роута - это ID некой модели, вы можете передать в аргументе экземпляр этой модели, Laravel возьмет оттуда ID сам:

	return redirect()->route('profile', [$user]);

#### Редирект на именованный роут с именованными параметрами

	// Если ваш роут с именем 'profile' имеет урл 'profile/{user}':

	return redirect()->route('profile', ['user' => 1]);

#### Редирект на метод определённого контроллера

Вы можете также сделать редирект на [экшн](/docs/{{version}}/controllers) заданного контроллера:

	return redirect()->action('App\Http\Controllers\HomeController@index');

> **Примечание:** Вам не нужно писать полный неймспейс контроллера, если вы задали его в `URL::setRootControllerNamespace`.

#### Редирект на метод определённого контроллера с параметрами

	return redirect()->action('App\Http\Controllers\UserController@profile', [1]);

#### Редирект на метод определённого контроллера с именованными параметрами

	return redirect()->action('App\Http\Controllers\UserController@profile', ['user' => 1]);

<a name="other-responses"></a>
## Особые HTTP-ответы

Если хэлпер `response()` вызывается без параметров, он возвращает имплементацию [контракта](/docs/{{version}}/contracts) `Illuminate\Contracts\Routing\ResponseFactory`, которая содержит несколько методов для генерации HTTP-ответа. 

#### Отдача JSON

Метод `json` автоматически устанавливает заголовок `Content-Type` в `application/json`

	return response()->json(['name' => 'Steve', 'state' => 'CA']);

#### Отдача JSONP

	return response()->json(['name' => 'Steve', 'state' => 'CA'])
	                 ->setCallback($request->input('callback'));

#### Отдача файла

	return response()->download($pathToFile);

	return response()->download($pathToFile, $name, $headers);

> **Примечание:** Классы Symfony HttpFoundation, которые занимаются функцией отдачи файла, требуют, чтобы имя файла было в ASCII-формате.

<a name="response-macros"></a>
## Макросы

Вы можете оформить свой вариант HTTP-ответа в виде макроса, чтобы использовать его в других роутах или контроллерах в короткой форме. 

HTTP-макросы определяются в методе `boot()` [сервис-провайдера](/docs/{{version}}/providers):

	<?php namespace App\Providers;

	use Response;
	use Illuminate\Support\ServiceProvider;

	class ResponseMacroServiceProvider extends ServiceProvider {

		/**
		 * Perform post-registration booting of services.
		 *
		 * @return void
		 */
		public function boot()
		{
			Response::macro('caps', function($value)
			{
				return Response::make(strtoupper($value));
			});
		}

	}

Используется макрос так:	

	return response()->caps('foo');
