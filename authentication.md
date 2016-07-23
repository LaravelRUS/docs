git 55f6089a29a81b948a03d5e3be397a782bba9d2e

---

# Аутентификация

- [Введение](#introduction)
    - [Настройки базы данных](#introduction-database-considerations)
- [Быстрый старт](#authentication-quickstart)
    - [Routing](#included-routing)
    - [Views](#included-views)
    - [Аутентификация](#included-authenticating)
    - [Получение аутентифицированного пользователя](#retrieving-the-authenticated-user)
    - [Ограничение доступа к роутам](#protecting-routes)
    - [Защита от перебора паролей](#authentication-throttling)
- [Ручная аутентификация пользователей](#authenticating-users)
    - [Запоминание пользователей](#remembering-users)
    - [Другие методы аутентификации](#other-authentication-methods)
- [Аутентификация HTTP Basic](#http-basic-authentication)
     - [Stateless HTTP Basic Аутентификация](#stateless-http-basic-authentication)
- [Сброс пароля](#resetting-passwords)
    - [Настройка базы данных](#resetting-database)
    - [Routing](#resetting-routing)
    - [Views](#resetting-views)
    - [Действия после сброса пароля](#after-resetting-passwords)
    - [Настройка сброса пароля](#password-customization)
- [Аутентификация через соцсети](https://github.com/laravel/socialite)
- [Создание своего гарда](#adding-custom-guards)
- [Создание своего провайдера](#adding-custom-user-providers)
- [События](#events)

<a name="introduction"></a>
## Введение

Аутентификация - это процесс сопоставления введенных логина и пароля с данными зарегистрированных пользователей сайта и определение, является ли пользователь сайта одним из них (тогда следует логин пользователя) или нет (тогда пользователь получает сообщение об ошибке). Не путать с [авторизацией](/docs/{{version}}/authorization) - процессом проверки прав на выполнение какого-либо действия. Это разные вещи, но для каждой из них Laravel предоставляет удобные инструменты.

Аутентификация в Laravel делается очень просто. Фактически, почти всё уже готово к использованию «из коробки». Настройки аутентификации находятся в файле `config/auth.php`, который содержит несколько хорошо документированных опций - посмотрите в это файл и многим из вас все сразу станет понятно.

Фактически, подсистема аутентификации Laravel состоит из двух частей:

1. Guards, "гарды", "охранники". Это по сути правила аутентификации пользователя - в каких частях запроса хранить информацию о том, что данный запрос идет от аутентифицированного пользователя. Например, это можно делать в сессии/куках, или в некотором токене, который должен содержаться в каждом запросе. В Laravel это гарды `session` и `token` соответственно.

2. Providers, "провайдеры". Они определяют, как можно получить данные пользователя из базы данных или другого места хранения. В Laravel можно получать пользователя через Eloquent и Query Builder, но вы можете написать свой провайдер, если по каким-то причинам, хотите хранить данные пользователей, например, в файле.

Можно создавать собственные гарды и провайдеры. Это нужно, если у вас, например, несколько таблиц с пользователями, или несколько областей в приложении, куда нужно логиниться отдельно, даже уже аутентифицированным пользователям - например, админка.

Но если вы только изучаете фреймворк - не беспокойтесь, чтобы использовать аутентификацию в Laravel вам не нужно досконально разбираться, как работают гарды и провайдеры. Весь необходимый код уже написан, и схема, которая принята по умолчанию, подойдет практически всем.

<a name="introduction-database-considerations"></a>
### Настройки базы данных

По умолчанию Laravel использует модель `App\User` [Eloquent](/docs/{{version}}/eloquent) в каталоге `app`. Эта модель может использоваться вместе с драйвером аутентификации на базе Eloquent. Если ваше приложение не использует Eloquent, вы можете применить драйвер аутентификации `database`, который использует Query Builder.

При создании схемы базы данных для модели `App\User` убедитесь, что поле пароля имеет длину минимум в 60 символов. Дефолтное значение для поля varchar - 255 - подойдет замечательно.

Также вы должны убедиться, что ваша таблица `users` (или её эквивалент) содержит строковое nullable поле `remember_token` длиной в 100 символов. Это поле используется для хранения токена сессии, если ваше приложение предоставляет функциональность «запомнить меня». Создать такое поле можно с помощью `$table->rememberToken();` в миграции.

<a name="authentication-quickstart"></a>
## Быстрый старт

Laravel оснащён двумя контроллерами аутентификации «из коробки». Они находятся в пространстве имён `App\Http\Controllers\Auth`. `AuthController` обрабатывает регистрацию и аутентификацию пользователей, а `PasswordController` содержит логику для сброса паролей существующих пользователей. Каждый из этих контроллеров использует трейты для включения необходимых методов. В большинстве случаев вам не понадобится редактировать эти контроллеры.

<a name="included-routing"></a>
### Routing

Чтобы сгенерировать роуты и шаблоны, которые нужны для процесса аутентификации (страница регистрации, логина, восстановления пароля и т.п.), вам нужно выполнить всего одну команду:

    php artisan make:auth

Вместе с ними создастся контроллер `HomeController`, куда будет вести редирект после успешного логина пользователя.    

Естественно, вы можете отредактировать эти сгенерированные файлы так, как вам нужно.

<a name="included-views"></a>
### Views

Как было сказано выше, шаблоны страниц регистрации создаются при помощи команды, в папке `resources/views/auth`. Кроме того, будет создан главный шаблон приложения `resources/views/layouts`, в который эти шаблоны будут подключаться. Вы можете строить приложение опираясь на этот главный шаблон, или использовать вместо него свой.

<a name="included-authenticating"></a>
### Аутентификация

Контроллеры аутентификации уже были в приложении, роуты и шаблоны вы только что сгенерировали. Всё, теперь пользователи могут регистрироваться и логиниться в ваше приложение. 

#### Настройка путей

После успешного логина пользователя надо куда-то редиректить. Куда именно - за это отвечает свойство `redirectTo` класса `AuthController`:

    protected $redirectTo = '/home';

Если логин не успешный, то происходит автоматический редирект назад на страницу логина.


Для изменения пути, куда будет перенаправлен пользователь после выхода, требуется определить свойство `redirectAfterLogout` класса `AuthController`:

    protected $redirectAfterLogout = '/login';

Если этого не сделать, пользователь будет перенаправлен на URI `/` .

#### Настройка гардов (guards)

Вы также можете назначить специфичный гард для обработки процесса аутентификации. Для этого создайте свойство `guard` в вашем классе `AuthController`. Значением этого свойства должно быть название одного из гардов, определённых вами в файле `config/auth.php`.

    protected $guard = 'admin';

#### Настройка валидации и сохранения пользователей

Для изменения полей формы, обязательных к заполнению при регистрации пользователей, или для изменения способа создания пользователей в базе данных вы можете править класс `AuthController`. Этот класс отвечает за валидацию и создание новых пользователей вашего приложения.

Метод `validator` класса `AuthController` содержит в себе правила валидации формы регистрации новых пользователей. Вы можете изменять его, как пожелаете.

Метод `create` класса `AuthController` отвечает за создание новых `App\User` записей в вашей базе данных, используя [Eloquent ORM](/docs/{{version}}/eloquent). Вы также можете изменять его под требования вашего приложения.

<a name="retrieving-the-authenticated-user"></a>
### Получение аутентифицированного пользователя

Вы можете получить аутентифицированного пользователя при помощи фасада `Auth`:

    $user = Auth::user();

То же самое вы можете сделать при помощи экземпляра `Illuminate\Http\Request`, который инжектится в аргументы метода контроллера, обрабатывающего, например, POST-запрос:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class ProfileController extends Controller
    {
        /**
         * Update the user's profile.
         *
         * @param  Request  $request
         * @return Response
         */
        public function updateProfile(Request $request)
        {
            if ($request->user()) {
                // $request->user() возвращает экземпляр аутентифицированного пользователя
            }
        }
    }

#### Проверка пользователя на аутентифицированность в приложении

Для проверки текущего пользователя на аутентифицированность в вашем приложении вы можете использовать метод `check` фасада `Auth`, который возвращает `true`, если пользователь аутентифицирован:

    if (Auth::check()) {
        // Пользователь аутентифицирован
    }

Однако, вы также можете использовать посредников (`middleware`) для проверки аутентификации пользователя перед доступом к конкретным роутам / контроллерам.

<a name="protecting-routes"></a>
### Ограничение доступа к роутам

[Route middleware](/docs/{{version}}/middleware) может использоваться для разрешения доступа к роутам только аутентифицированным пользователям. В Laravel уже есть посредник `auth`, который находится в файле `app\Http\Middleware\Authenticate.php`. Всё, что вам нужно — указать его в описании нужного роута:

    // Если роут описан как замыкание...

    Route::get('profile', ['middleware' => 'auth', function() {
        // Доступ разрешён только аутентифицированным пользователям...
    }]);

    // Или как контроллер...

    Route::get('profile', [
        'middleware' => 'auth',
        'uses' => 'ProfileController@show'
    ]);

Конечно, если вы используете [контроллеры](/docs/{{version}}/controllers), вы можете вызвать метод `middleware` внутри них, в конструкторе:

    public function __construct()
    {
        $this->middleware('auth');
    }

### Назначение гарда

Когда вы назначаете middleware `auth` для защиты роута, вы можете явно указать, какой guard из тех, что определены у вас в `config/auth.php` вы хотите использовать в данном случае:

    Route::get('profile', [
        'middleware' => 'auth:api',
        'uses' => 'ProfileController@show'
    ]);

<a name="authentication-throttling"></a>
### Защита от перебора паролей

В классе `AuthController`, который идет с фреймворком, есть трейт `Illuminate\Foundation\Auth\ThrottlesLogins`, который реализует так называемый throttling, то есть ограничение по количеству действий за единицу времени. По умолчанию, после нескольких безуспешных попыток залогиниться, для этого IP закрывается возможность логиниться в приложение в течении минуты. 

    <?php

    namespace App\Http\Controllers\Auth;

    use App\User;
    use Validator;
    use App\Http\Controllers\Controller;
    use Illuminate\Foundation\Auth\ThrottlesLogins;
    use Illuminate\Foundation\Auth\AuthenticatesAndRegistersUsers;

    class AuthController extends Controller
    {
        use AuthenticatesAndRegistersUsers, ThrottlesLogins;

        // Rest of AuthController class...
    }

<a name="authenticating-users"></a>
## Ручная аутентификация пользователей

Конечно же, вы не обязаны использовать контроллеры аутентификации, включённые в Laravel. Если вы удалите эти контроллеры, нужно будет управлять аутентификацией пользователей при помощи классов аутентификации Laravel. Не волнуйтесь, это просто!

Доступ к сервисам аутентификации Laravel осуществляется при помощи [фасада](/docs/{{version}}/facades) `Auth`, поэтому нужно убедиться, что вы импортировали его перед вашим классом. Далее давайте рассмотрим метод `attempt`:

    <?php

    namespace App\Http\Controllers;

    use Auth;
    use Illuminate\Routing\Controller;

    class AuthController extends Controller
    {
        /**
         * Handle an authentication attempt.
         *
         * @return Response
         */
        public function authenticate()
        {
            if (Auth::attempt(['email' => $email, 'password' => $password])) {
                // Аутентификация прошла успешно
                return redirect()->intended('dashboard');
            }
        }
    }

Метод `attempt` принимает массив пар «ключ - значение» первым аргументом. Значения в этом массиве будут использованы для поиска пользователя в базе данных. Таким образом, в этом примере пользователь ищется по полю `email`. Если пользователь найден, хеш пароля из базы данных сравнится с хешем указанного пароля в массиве. Если хеши совпадут, то сессия аутентификации стартует для пользователя.

Метод `attempt` возвратит `true`, если аутентификация прошла успешно. Иначе — `false`.

Метод `intended` редиректора переместит пользователя на URL, на который он хотел попасть до прохождения аутентификации. Запасной URL указывается в параметре и будет использован, если путь, куда хотел перейти пользователь, недоступен.

#### Дополнительные условия прохождения аутентификации

Если хотите, вы также можете добавить дополнительные условия прохождения аутентификации в дополнение к e-mail и паролю. Например, вы можете проверять, активен ли пользователь:

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // Пользователь активен, существует и ввёл верный пароль
    }

> **Обратите внимание:** В этих примерах поле `email` не обязательно должно использоваться для поиска пользователя, оно выбрано для примера. Вы можете использовать любое поле для поиска пользователя, которое является уникальным в таблице пользователей вашей базы данных.

#### Авторизация с использованием заданного гарда

Вы можете явно задать, при помощи какого гарда обслуживать процесс авторизации. Это позволит вам иметь в приложении несколько частей, вход в которые осуществляется по своим правилам. Пользователь может быть залогинен в одну из них, или несколько. Самый простой пример - это админка. Ваш гард `admin` определяет правило, залогинен данный пользователь как админ, или нет - например, установкой специальной куки.

Тогда при логине в админку вы делаете так:

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

#### Выход из приложения

Для осуществления выхода пользователя из приложения можно воспользоваться методом `logout` фасада `Auth`. Это очистит информацию об аутентификации из сессии пользователя:

    Auth::logout();

<a name="remembering-users"></a>
## Запоминание пользователей

Если вы хотите реализовать в вашем приложении функциональность «запомнить меня», можно указать булево значение вторым аргументом методу `attempt`. Это аутентифицирует пользователя на неограниченное время или пока он не выйдет из приложения. Само собой, ваша таблица `users` должна содержать поле `remember_token`, которое будет использоваться для сохранения токена «запомнить меня».

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // Пользователь запомнен
    }

Если вы «запоминаете» пользователей, можете использовать метод `viaRemember` для определения того, аутентифицировался ли пользователь с помощью куки «запомнить меня»:

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### Другие методы аутентификации

#### Аутентификация экземпляра пользователя

Если вам нужно аутентифицировать существующего пользователя в ваше приложение, вы можете вызвать метод `login` на экземпляре этого пользователя. Этот экземпляр должен реализовать [контракт](/docs/{{version}}/contracts) `Illuminate\Contracts\Auth\Authenticatable`. Модель `App\User`, используемая Laravel по умолчанию, уже реализует этот интерфейс:

    Auth::login($user);
    
    // Аутентификация с функцией "запомнить меня"
    Auth::login($user, true);

Вы также можете явно указать гард, при помощи которого будете фиксировать процесс аутентификации.

    Auth::guard('admin')->login($user);

#### Аутентификация по ID пользователя

Для аутентификации пользователя по его ID вы можете использовать метод `loginUsingId`. Этот метод принимает ID пользователя аргументом:

    Auth::loginUsingId(1);
    
    // Аутентификация с функцией "запомнить меня"
    Auth::login(1, true);

#### Аутентификация пользователя на один запрос

Вы также можете использовать метод `once` для аутентификации пользователя в вашем приложении на один запрос. Ни сессии, ни куки не будут созданы, что может быть использовано для создания stateless API. Метод `once` работает по тому же принципу, что и `attempt`:

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## Аутентификация HTTP Basic

[HTTP Basic Аутентификация](http://en.wikipedia.org/wiki/Basic_access_authentication) предоставляет быстрый способ аутентифицировать пользователя в вашем приложении без создания отдельной страницы входа. Сначала добавьте [посредник](/docs/{{version}}/middleware) `auth.basic` в ваш роут. Он включён в Laravel по умолчанию, так что вам не нужно создавать его:

    Route::get('profile', ['middleware' => 'auth.basic', function() {
        // Только аутентифицированные пользователи могут войти
    }]);

Как только посредник прикреплён к роуту, он автоматически будет показывать окно аутентификации в браузере при посещении этого роута. По умолчанию `auth.basic` использует поле `email` для поиска пользователя.

#### Примечание для FastCGI

Если вы используете PHP FastCGI, HTTP Basic аутентификация может не работать из коробки. Вы должны добавить эти строки в ваш файл `.htaccess`:

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### Stateless HTTP Basic Аутентификация

Вы также можете использовать HTTP Basic аутентификацию без установки куки идентификации пользователя в сессии, что может быть полезно при создании API аутентификации. Для этого [создайте посредника](/docs/{{version}}/middleware), который будет вызывать метод `onceBasic`. Если `onceBasic` ничего не возвратит, запрос пойдёт дальше в приложение.

    <?php 

    namespace Illuminate\Auth\Middleware;

    use Auth;
    use Closure;

    class AuthenticateOnceWithBasicAuth implements Middleware
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

Дальше [зарегистрируйте посредника](/docs/{{version}}/middleware#registering-middleware) и привяжите его к роуту:

    Route::get('api/user', ['middleware' => 'auth.basic.once', function() {
        // Только аутентифицированный пользователь может войти
    }]);

<a name="resetting-passwords"></a>
## Сброс пароля

<a name="resetting-database"></a>
### Настройка базы данных

Большинство веб-приложений предоставляют пользователю возможность сбрасывать свой пароль. Для того, чтобы вам не приходилось реализовывать это в каждом приложении, Laravel предоставляет простой способ отправки «напоминателей» паролей и осуществления сброса пароля.

Для начала проверьте, реализует ли ваша модель `App\User` контракт `Illuminate\Contracts\Auth\CanResetPassword`. По умолчанию она это делает и использует трейт `Illuminate\Auth\Passwords\CanResetPassword`, который предоставляет необходимые методы для сброса пароля.

#### Создание миграции таблицы токенов сброса паролей

Дальше вам необходимо создать таблицу, в которой будут храниться токены сброса паролей. Эта миграция по умолчанию уже включена в Laravel в каталог `database/migrations`. Поэтому вам остаётся только мигрировать базу данных:

    php artisan migrate

<a name="resetting-routing"></a>
### Routing

Laravel предоставляет класс `Auth\PasswordController`, в котором содержится вся необходимая логика для сброса паролей. Для создания роутов и шаблонов вам нужно воспользоваться командой:

    php artisan make:auth

<a name="resetting-views"></a>
### Views

Эта же команда создает в папке `resources/views/auth/passwords` шаблоны для страниц восстановления пароля.

<a name="after-resetting-passwords"></a>
### Действия после сброса пароля

После того, как вы определили роуты и шаблоны для сброса паролей, можете проверить их в браузере, пройдя по урлу `/password/reset`. Класс `PasswordController` включён в фреймворк по умолчанию и содержит всю необходимую логику для отправки писем и сброса паролей в базе данных.

После сброса пароля пользователь автоматически аутентифицируется в вашем приложении и редиректится на страницу `/home`. Вы можете изменить этот путь, создав свойство `redirectTo` у `PasswordController`

    protected $redirectTo = '/dashboard';

> **Обратите внимание:** По умолчанию токен сброса пароля существует 1 час. Вы можете изменить это в конфигурационном файле `config/auth.php` с помощью опции `expire`.

<a name="password-customization"></a>
### Настройка сброса пароля

#### Настройка гардов

В `config/auth.php` вы можете определить несколько гардов. Чтобы использовать выбранный гард для процедуры восстановления пароля, добавьте свойство в класс `PasswordController`:

    /**
     * The authentication guard that should be used.
     *
     * @var string
     */
    protected $guard = 'admins';

#### Password Broker Customization

In your `auth.php` configuration file, you may configure multiple password "brokers", which may be used to reset passwords on multiple user tables. You can customize the included `PasswordController` to use the broker of your choice by adding a `$broker` property to the controller:

    /**
     * The password broker that should be used.
     *
     * @var string
     */
    protected $broker = 'admins';

<a name="adding-custom-guards"></a>
## Создание своего гарда

You may define your own authentication guards using the `extend` method on the `Auth` facade. You should place this call to `provider` within a [service provider](/docs/{{version}}/providers):

Своего гарда вы можете создать при помощи метода `extend` фасада `Auth`. Делать это следует в одном из ваших [сервис-провайдеров](/docs/{{version}}/providers), в методе `boot`:

    <?php

    namespace App\Providers;

    use Auth;
    use App\Services\Auth\JwtGuard;
    use Illuminate\Support\ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Auth::extend('jwt', function($app, $name, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\Guard...

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Как вы видите из примера выше, первым аргументом является имя вашего гарда, а вторым - функция, которая должна возвращать экземпляр класса, который имплементирует интерфейс `Illuminate\Contracts\Auth\Guard` со всеми необходимыми для функционирования гарда методами.

Теперь вы можете использовать гард `jwt` в конфигах:

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="adding-custom-user-providers"></a>
## Создание своего провайдера

Если вы не используете традиционные реляционные базы данных для хранения пользователей, вам нужно расширить Laravel собственным провайдером аутентификации. Для этого используйте метод `provider` фасада `Auth`. Это код вы должны поместить в метод `boot` одного из ваших [сервис-провайдеров](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Support\ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Auth::provider('riak', function($app, array $config) {
                // Возвращаем класс интерфейса (контракта) Illuminate\Contracts\Auth\UserProvider...
                return new RiakUserProvider($app['riak.connection']);
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

После добавления провайдера вы можете переключиться на него в вашем файле конфигурации `config/auth.php`. Сначала поставьте его в качестве драйвера для users:

    'providers' => [
        'users' => [
            'driver' => 'riak',
        ],
    ],

Далее мы можем юзать этот провайдер в конфигурации гардов:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

### Контракт User Provider

Реализации контракта `Illuminate\Contracts\Auth\UserProvider` ответственны только за получение реализации контракта `Illuminate\Contracts\Auth\Authenticatable` из хранилища данных, такого, например, как MySQL, Riak и тд. Эти два интерфейса позволяют Laravel использовать механизмы аутентификации независимо от того, как хранятся данные приложения.

Давайте посмотрим на контракт `Illuminate\Contracts\Auth\UserProvider`:

    <?php 

    namespace Illuminate\Contracts\Auth;

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

Метод `retrieveById` служит для получения пользователя из базы данных по его ID. Реализация класса `Authenticatable` должна быть возвращена в ответ методом.

Метод `retrieveByToken` получает пользователя по его ID и токену «запомнить меня», который хранится в поле `remember_token`. Как и в предыдущем методе, должна быть возвращена реализация класса `Authenticatable`.

Метод `updateRememberToken` обновляет токен пользователя. Новый токен может быть как снова созданным при аутентификации пользователя, так и null — при выходе пользователя.

Метод `retrieveByCredentials` получает  массив данных, указанных в методе `Auth::attempt`, при аутентификации в приложении. Этот метод должен искать пользователя по указанным данных в хранилище данных. Обычно он ищет по полю `username`. Он должен возвращать реализацию интерфейса `UserInterface`. **Этот метод не должен проверять пароль пользователя или аутентифицировать его.**

Метод `validateCredentials` должен проверять правильность данных пользователя. Например, этот метод может сравнивать строку `$user->getAuthPassword()` с `Hash::make` из `$credentials['password']`. Этот метод должен проверять данные пользователя и возвращать булево значение.

### Контракт Authenticatable

Теперь, когда мы рассмотрели все методы `UserProvider`, давайте взглянем на контракт `Authenticatable`. Помните, провайдер должен возвращать реализацию этого интерфейса из методов `retrieveById` и `retrieveByCredentials`:

    <?php 

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifierName();
        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

Интерфейс прост. Метод `getAuthIdentifierName` должен возвращать название столбца, где хранится «primary key» пользователя, `getAuthIdentifier` - собственно «primary key» пользователя. В MySQl, например, это уникальный первичный ключ. Метод `getAuthPassword` должен возвращать хеш пароля пользователя. Этот интерфейс позволяет системе аутентификации работать с любым классом User независимо от используемой ORM или базы данных. По умолчанию Laravel включает в себя класс User в папке `app`, который реализует этот интерфейс, поэтому вы можете использовать его, как пример.

<a name="events"></a>
## События

Laravel запускает различные события в процессе процедуры аутентификации. Вы можете встроиться в процесс при помощи слушателей этих событий в вашем `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],

        'Illuminate\Auth\Events\Login' => [
            'App\Listeners\LogSuccessfulLogin',
        ],

        'Illuminate\Auth\Events\Logout' => [
            'App\Listeners\LogSuccessfulLogout',
        ],

        'Illuminate\Auth\Events\Lockout' => [
            'App\Listeners\LogLockout',
        ],
    ];
