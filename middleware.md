git d7de7ce5504e3f2a9c89fe57b8658cf1413ab261

---

# HTTP Middleware (посредники)

- [Введение](#introduction)
- [Создание middleware](#defining-middleware)
- [Регистрация middleware](#registering-middleware)

<a name="introduction"></a>
## Введение

HTTP Middleware (посредники) - это фильтры обработки HTTP-запроса. Так, например, в Laravel включены middlewares для проверки аутентификации пользователя. Если пользователь не залогинен, middleware редиректит его на страницу логина. Если же залогинен - middleware не вмешивается в прохождение запроса, передавая его дальше по цепочке middleware-посредников к собственно приложению.

> **Примечание:** Middlewares похожи на фильтры роутов в Laravel4.

Конечно, проверка авторизации - не единственная задача, которую способны выполнять посредники. Это также добавление особых заголовков, например, CORS http-ответ вашего приложения, или, например, логирование всех http-запросов.

В Laravel есть несколько дефолтных посредников, которые находятся в папке `app/Http/Middleware`. Это посредники для реализации режима обслуживания сайта ("сайт временно не работает, зайдите позже"), проверки авторизации, CSRF-защиты и т.п.

<a name="defining-middleware"></a>
## Создание middleware

Давайте для примера создадим middleware, который будет пропускать только те запросы, у которых параметр `age` будет больше чем 200, а всех остальных редиректить на урл `home`.

Для создания посредника воспользуйтесь командой `make:middleware`:

	php artisan make:middleware OldMiddleware

В папке `app/Http/Middleware` будет создан файл с классом `OldMiddleware`: 

	<?php namespace App\Http\Middleware;

	class OldMiddleware {

		/**
		 * Run the request filter.
		 *
		 * @param  \Illuminate\Http\Request  $request
		 * @param  \Closure  $next
		 * @return mixed
		 */
		public function handle($request, Closure $next)
		{
			if ($request->input('age') < 200)
			{
				return redirect('home');
			}

			return $next($request);
		}

	}

Как видно из примера, чтобы пропустить запрос дальше, нужно возвратить функцию-замыкание, которая приходит в middleware вторым аргументом, с аргументом, который приходит в middleware первым аргументом.

<a name="registering-middleware"></a>
## Регистрация middleware

### Глобально

Если вам нужно, чтобы через ваш middleware проходили все HTTP-запросы, то просто добавьте его в свойство `$middleware` класса `app/Http/Kernel.php` :

	protected $middleware = [
		'Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode',
		'Illuminate\Cookie\Middleware\EncryptCookies',
		'Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse',
		'Illuminate\Session\Middleware\StartSession',
		'Illuminate\View\Middleware\ShareErrorsFromSession',
		'Illuminate\Foundation\Http\Middleware\VerifyCsrfToken',
	];

### Сопоставить с заданными роутами

Добавьте ваш middleware в свойство `routeMiddleware` класса `app/Http/Kernel.php`, назначив ему некоторое имя, например, auth, которое будет ключем массива:

	protected $routeMiddleware = [
		'auth' => 'App\Http\Middleware\Authenticate',
		'auth.basic' => 'Illuminate\Auth\Middleware\AuthenticateWithBasicAuth',
		'guest' => 'App\Http\Middleware\RedirectIfAuthenticated',
	];

Теперь вы можете назначить этот middleware роуту или группе:

	Route::get('admin/profile', ['middleware' => 'auth', function()
	{
		//
	}]);