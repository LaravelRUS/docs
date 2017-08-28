git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Посредники

- [Введение](#introduction)
- [Создание посредника](#defining-middleware)
- [Регистрация посредника](#registering-middleware)
    - [Глобальный посредник](#global-middleware)
    - [Назначение посредника роутам](#assigning-middleware-to-routes)
    - [Группы посредников](#middleware-groups)
- [Параметры посредника](#middleware-parameters)
- [Посредник terminable](#terminable-middleware)

<a name="introduction"></a>
## Введение

Посредники предоставляют удобный механизм для фильтрации HTTP-запросов вашего приложения. Например, в Laravel есть посредник для проверки аутентификации пользователя. Если пользователь не аутентифицирован, посредник перенаправит его на страницу входа в систему. Если же пользователь аутентифицирован, посредник позволит запросу пройти далее в приложение.

Конечно, посредники нужны не только для авторизации. CORS-посредник может пригодиться для добавления особых заголовков ко всем ответам в вашем приложении. А посредник логов может зарегистрировать все входящие запросы.

В Laravel есть несколько стандартных посредников, включая посредники для аутентификации и CSRF-защиты. Все они расположены в директории `app/Http/Middleware`.

<a name="defining-middleware"></a>
## Создание посредника

Чтобы создать посредника, используйте команду Artisan `make:middleware`:

    php artisan make:middleware CheckAge

Эта команда поместит новый класс `CheckAge` в вашу директорию `app/Http/Middleware`. В этом посреднике мы будем пропускать только те запросы, в которых age будет больше 200, а во всех остальных случаях будем перенаправлять пользователей на `home` URI.

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckAge
    {
        /**
         * Обработка входящего запроса.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->age <= 200) {
                return redirect('home');
            }

            return $next($request);
        }

    }

Как видите, если переданный age меньше или равен 200, то посредник вернёт клиенту переадресацию, иначе, запрос будет передан далее в приложение. Чтобы передать запрос дальше в приложение (позволяя посреднику "передать" его), просто вызовите функцию `$next` с параметром `$request`.

Проще всего представить посредника как набор "уровней", которые должен пройти HTTP-запрос, прежде чем он дойдёт до вашего приложения. Каждый уровень может проверить запрос и даже вовсе отклонить его.

### Выполнение посредника "до" или "после" запроса

Момент, в который сработает посредник — до или после запроса, зависит от него самого. Например, этот посредник выполнит некоторую задачу **прежде**, чем запрос будет обработан приложением:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class BeforeMiddleware
    {
        public function handle($request, Closure $next)
        {
            // Выполнить действие

            return $next($request);
        }
    }

Однако, этот посредник выполнит задачу **после** того, как запрос будет обработан приложением:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AfterMiddleware
    {
        public function handle($request, Closure $next)
        {
            $response = $next($request);

            // Выполнить действие

            return $response;
        }
    }

<a name="registering-middleware"></a>
## Регистрация посредника

<a name="global-middleware"></a>
### Глобальный посредник

Если вы хотите, чтобы посредник запускался для каждого HTTP-запроса в вашем приложении, добавьте этот посредник в свойство `$middleware` вашего класса `app/Http/Kernel.php`.

<a name="assigning-middleware-to-routes"></a>
### Назначение посредника роутам

Если вы хотите назначить посредника для конкретных маршрутов, то сначала вам надо добавить ключ посредника в файл `app/Http/Kernel.php`. По умолчанию свойство `$routeMiddleware` этого класса содержит записи посредников Laravel. Чтобы добавить ваш собственный посредник, просто добавьте его к этому списку и присвойте ему ключ на свой выбор. Например:

    // В классе App\Http\Kernel...

    protected $routeMiddleware = [
        'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    ];

Когда посредник определён в HTTP-ядре, вы можете использовать метод `middleware` для назначения посредника роуту:

    Route::get('admin/profile', function () {
        //
    })->middleware('auth');

Также можно назначить несколько посредников роуту:

    Route::get('/', function () {
        //
    })->middleware('first', 'second');

При назначении посредника вы можете указать полное имя класса:

    use App\Http\Middleware\CheckAge;

    Route::get('admin/profile', function () {
        //
    })->middleware(CheckAge::class);

<a name="middleware-groups"></a>
### Группы посредников

Иногда бывает полезно объединить несколько посредников под одним ключом, чтобы проще назначать их на маршруты. Это можно сделать при помощи свойства `$middlewareGroups` вашего HTTP-ядра.

Изначально в Laravel есть группы посредников `web` и `api`, которые содержат те посредники, которые часто применяются к вашим роутам веб-UI и API:

    /**
     * Группы посредников роутов приложения.
     *
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],

        'api' => [
            'throttle:60,1',
            'auth:api',
        ],
    ];

Группы посредников могут быть назначены роутам и действия контроллера с помощью того же синтаксиса, что и для одного посредника. Группы посредников просто делают проще единое назначение нескольких посредников на роут:

    Route::get('/', function () {
        //
    })->middleware('web');

    Route::group(['middleware' => ['web']], function () {
        //
    });

> {tip} Изначально группа посредников `web` автоматически применяется к вашему файлу `routes/web.php` сервис-провайдером `RouteServiceProvider`.

<a name="middleware-parameters"></a>
## Параметры посредника

В посредник можно передавать дополнительные параметры. Например, если в вашем приложении необходима проверка того, есть ли у аутентифицированного пользователя определённая "роль" для выполнения данного действия, вы можете создать посредника `CheckRole`, который принимает название роли в качестве дополнительного аргумента.

Дополнительные параметры посредника будут передаваться в посредник после аргумента `$next`:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckRole
    {
        /**
         * Обработка входящего запроса.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string  $role
         * @return mixed
         */
        public function handle($request, Closure $next, $role)
        {
            if (! $request->user()->hasRole($role)) {
                // Redirect...
            }

            return $next($request);
        }

    }

Параметры посредника можно указать при определении маршрута, отделив название посредника от параметров двоеточием `:`. Несколько параметров разделяются запятыми:

    Route::put('post/{id}', function ($id) {
        //
    })->middleware('role:editor');

<a name="terminable-middleware"></a>
## Посредник terminable

Иногда посредник должен выполнить некоторые действия уже после отправки HTTP-ответа браузеру. Например, посредник "session", поставляемый с Laravel, записывает данные сессии в хранилище после отправки ответа в браузер. Если вы определите метод `terminate` в посреднике, то он будет автоматически вызываться после отправки ответа в браузер.

    <?php

    namespace Illuminate\Session\Middleware;

    use Closure;

    class StartSession
    {
        public function handle($request, Closure $next)
        {
            return $next($request);
        }

        public function terminate($request, $response)
        {
            // Store the session data...
        }
    }

Метод `terminate` должен получать и запрос и ответ. Определив terminable-посредника, вы должны добавить его в список посредников роута или глобальных посредников в файл `app/Http/Kernel.php`.

При вызове метода `terminate` в посреднике, Laravel получит свежий экземпляр посредника из [сервис-контейнера](/docs/{{version}}/container). Если вы хотите использовать тот же самый экземпляр посредника при вызовах методов `handle` и `terminate`, зарегистрируйте посредника в контейнере при помощи метода `singleton`.
