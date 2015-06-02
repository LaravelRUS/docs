git 8edcd7b931677ffab8784eee72d88b797cf60cf3

---

# HTTP Middleware (посредники)

- [Введение](#introduction)
- [Создание middleware](#defining-middleware)
- [Регистрация middleware](#registering-middleware)
- [Terminable Middleware](#terminable-middleware)

<a name="introduction"></a>
## Введение

HTTP Middleware (посредники) - это фильтры обработки HTTP-запроса. Так, например, в Laravel включены middlewares для проверки аутентификации пользователя. Если пользователь не залогинен, middleware перенаправляет его на страницу логина. Если же залогинен - middleware не вмешивается в прохождение запроса, передавая его дальше по цепочке middleware-посредников к собственно приложению.

> **Примечание:** Middlewares похожи на фильтры роутов в Laravel 4.

Конечно, проверка авторизации - не единственная задача, которую способны выполнять middlewares. Это также добавление особых заголовков (например, CORS http-ответ вашего приложения) или логирование всех http-запросов.

В Laravel есть несколько дефолтных middleware, которые находятся в папке `app/Http/Middleware`. Это middlewares для реализации режима обслуживания сайта ("сайт временно не работает, зайдите позже"), проверки авторизации, CSRF-защиты и т.п.

<a name="defining-middleware"></a>
## Создание middleware

Давайте для примера создадим middleware, который будет пропускать только те запросы, у которых параметр `age` будет больше чем 200, а всех остальных перенаправлять на `/home`.

Для создания middleware воспользуемся командой `make:middleware`:

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

Чтобы пропустить запрос дальше, нужно вызвать функцию-замыкание `$next` с параметром `$request`.

Лучше всего представлять middlewares как набор уровней, которые HTTP-запрос должен пройти, прежде чем дойдёт до вашего приложения. На каждом уровне запрос может быть проверен по различным критериям и, если нужно, полностью отклонён.

<a name="registering-middleware"></a>
## Регистрация middleware

### Глобально

Если вам нужно, чтобы через ваш middleware проходили все HTTP-запросы, то просто добавьте его в свойство `$middleware` класса `app/Http/Kernel.php`:

	protected $middleware = [
		'Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode',
		'Illuminate\Cookie\Middleware\EncryptCookies',
		'Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse',
		'Illuminate\Session\Middleware\StartSession',
		'Illuminate\View\Middleware\ShareErrorsFromSession',
		'Illuminate\Foundation\Http\Middleware\VerifyCsrfToken',
	];

### Сопоставить с заданными роутами

Добавьте ваш middleware в свойство `routeMiddleware` класса `app/Http/Kernel.php`, назначив ему некоторое имя, например, `auth`, которое будет ключом массива:

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

<a name="terminable-middleware"></a>
## Terminable Middleware

Иногда, middleware может понадобиться проделать некоторую работу после того как HTTP ответы были отправлены в браузер. Например, middleware "session", поставляемый с Laravel, записывает данные сессии в хранилище _после_ отправки ответа браузеру. Для этого вы можете определить middleware как "terminable".

	use Illuminate\Contracts\Routing\TerminableMiddleware;

	class StartSession implements TerminableMiddleware {

		public function handle($request, $next)
		{
			return $next($request);
		}

		public function terminate($request, $response)
		{
			// Храним данные сессии...
		}

	}

Как вы видите, в дополнение к определению метода `handle`, `TerminableMiddleware` определяет и `terminate` метод. Этот метод получает как запрос так и ответ. После того как вы определили terminable middleware, вы должны добавить его в список глобальных посредников в ваше ядро HTTP.
