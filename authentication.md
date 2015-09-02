git b16092e910df2cd465d61711ed06d8bf58e93e76

---

# Аутентификация

- [Введение](#introduction)
- [Быстрый старт](#authentication-quickstart)
    - [Routing](#included-routing)
    - [Views](#included-views)
    - [Аутентификация](#included-authenticating)
    - [Получение аутентифицированного пользователя](#retrieving-the-authenticated-user)
    - [Ограничение доступа к роутам](#protecting-routes)
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
- [Аутентификация через социальные сети](#social-authentication)
- [Добавление драйвера аутентификации](#adding-custom-authentication-drivers)

<a name="introduction"></a>
## Введение

Laravel позволяет сделать аутентификацию очень простой. Фактически, почти всё уже готово к использованию «из коробки». Настройки аутентификации находятся в файле `config/auth.php`, который содержит несколько хорошо документированных опций для настройки механизма аутентификации.

### Настройки базы данных

По умолчанию Laravel использует модель `App\User` [Eloquent](/docs/{{version}}/eloquent) в каталоге `app`. Эта модель может использоваться вместе с драйвером аутентификации на базе Eloquent. Если ваше приложение не использует Eloquent, вы можете применить драйвер аутентификации `database`, который использует Laravel Query Builder.

При создании схемы базы данных для модели `App\User` убедитесь, что поле пароля имеет минимальную длину в 60 символов.

Также вы должны убедиться, что ваша таблица `users` (или её эквивалент) содержит строковое nullable поле `remember_token` длиной в 100 символов. Это поле используется для хранения токена сессии, если ваше приложение предоставляет функциональность «запомнить меня». Создать такое поле можно с помощью `$table->rememberToken();` в миграции.

<a name="authentication-quickstart"></a>
## Быстрый старт

Laravel оснащён двумя контроллерами аутентификации «из коробки». Они находятся в пространстве имён `App\Http\Controllers\Auth`. `AuthController` обрабатывает регистрацию и аутентификацию пользователей, а `PasswordController` содержит логику для сброса паролей существующих пользователей. Каждый из этих контроллеров использует трейты для включения необходимых методов. Для многих приложений вам не понадобится редактировать эти контроллеры.

<a name="included-routing"></a>
### Routing

По умолчанию никакие [роуты](/docs/{{version}}/routing) не ведут к контроллерам аутентификации. Вы можете самостоятельно добавить их в вашем файле `app/Http/routes.php`:

    // Authentication routes...
    Route::get('auth/login', 'Auth\AuthController@getLogin');
    Route::post('auth/login', 'Auth\AuthController@postLogin');
    Route::get('auth/logout', 'Auth\AuthController@getLogout');

    // Registration routes...
    Route::get('auth/register', 'Auth\AuthController@getRegister');
    Route::post('auth/register', 'Auth\AuthController@postRegister');

<a name="included-views"></a>
### Views

Хотя контроллеры аутентификации включены в фреймворк, вам всё равно придётся создать [шаблоны](/docs/{{version}}/views), которые эти контроллеры будут отображать. Они должны находиться в каталоге `resources/views/auth`. Вы можете изменять их, как пожелаете. Шаблон страницы входа должен быть расположен в `resources/views/auth/login.blade.php`, а шаблон регистрации — в `resources/views/auth/register.blade.php`.

#### Пример формы аутентификации

    <!-- resources/views/auth/login.blade.php -->

    <form method="POST" action="/auth/login">
        {!! csrf_field() !!}

        <div>
            Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            Password
            <input type="password" name="password" id="password">
        </div>

        <div>
            <input type="checkbox" name="remember"> Remember Me
        </div>

        <div>
            <button type="submit">Login</button>
        </div>
    </form>

#### Пример формы регистрации

    <!-- resources/views/auth/register.blade.php -->

    <form method="POST" action="/auth/register">
        {!! csrf_field() !!}

        <div class="col-md-6">
            Name
            <input type="text" name="name" value="{{ old('name') }}">
        </div>

        <div>
            Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            Password
            <input type="password" name="password">
        </div>

        <div class="col-md-6">
            Confirm Password
            <input type="password" name="password_confirmation">
        </div>

        <div>
            <button type="submit">Register</button>
        </div>
    </form>

<a name="included-authenticating"></a>
### Аутентификация

Теперь, когда у вас есть роуты и шаблоны, вы можете регистрировать и аутентифицировать пользователей вашего приложения. Контроллеры аутентификации уже содержат логику (через трейты) для аутентификации существующих пользователей и создания новых в вашей базе данных.

При успешной аутентфикации пользователь попадает на страницу `/home`, для которой вы должны создать роут. Вы можете изменить путь редиректа после аутентификации при помощи свойства `redirectTo` у `AuthController`:

    protected $redirectTo = '/dashboard';

#### Дополнительная настройка

Для изменения полей формы, обязательных к заполнению при регистрации пользователей, или для изменения способа создания пользователей в базе данных вы можете править класс `AuthController`. Этот класс отвечает за валидацию и создание новых пользователей вашего приложения.

Метод `validator` класса `AuthController` содержит в себе правила валидации формы регистрации новых пользователей. Вы можете изменять его, как пожелаете.

Метод `create` класса `AuthController`  отвечает за создание новых `App\User` записей в вашей базе данных, используя [Eloquent ORM](/docs/{{version}}/eloquent). Вы также можете изменять его под требования вашего приложения.

<a name="retrieving-the-authenticated-user"></a>
### Получение аутентифицированного пользователя

Вы можете получить аутентифицированного пользователя при помощи фасада `Auth`:

    $user = Auth::user();

То же самое вы можете сделать при помощи экземпляра `Illuminate\Http\Request`:

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

Если хотите, вы также можете добавить дополнительные условия прохождения аутентификации в дополнение к E-mail и паролю. Например, вы можете проверять, активен ли пользователь:

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // Пользователь активен, существует и ввёл верный пароль
    }

Для осуществления выхода пользователя из приложения можно воспользоваться методом `logout` фасада `Auth`. Это очистит информацию об аутентификации из сессии пользователя:

    Auth::logout();

> **Обратите внимание:** В этих примерах поле `email` не обязательно должно использоваться для поиска пользователя, оно выбрано для примера. Вы можете использовать любое поле для поиска пользователя, которое является уникальным в таблице пользователей вашей базы данных.

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

#### Аутентификация по ID пользователя

Для аутентификации пользователя по его ID вы можете использовать метод `loginUsingId`. Этот метод принимает ID пользователя аргументом:

    Auth::loginUsingId(1);

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

    <?php namespace Illuminate\Auth\Middleware;

    use Auth;
    use Closure;
    use Illuminate\Contracts\Routing\Middleware;

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

Laravel предоставляет класс `Auth\PasswordController`, в котором содержится вся необходимая логика для сброса паролей. Тем не менее, вам нужно будет создать роуты к этому контроллеру:

    // Роуты запроса ссылки для сброса пароля
    Route::get('password/email', 'Auth\PasswordController@getEmail');
    Route::post('password/email', 'Auth\PasswordController@postEmail');

    // Роуты сброса пароля
    Route::get('password/reset/{token}', 'Auth\PasswordController@getReset');
    Route::post('password/reset', 'Auth\PasswordController@postReset');

<a name="resetting-views"></a>
### Views

В дополнение к определению роутов к `PasswordController` вы должны создать шаблоны, которые будет отображать ваш контроллер. Не беспокойтесь, ниже предоставлены примеры таких шаблонов. Вы вольны стилизировать их под свои нужды.

#### Пример формы запроса ссылки сброса пароля

Вам нужно создать HTML шаблон формы запроса сброса пароля. Этот шаблон должен находиться в `resources/views/auth/password.blade.php`. Форма содержит единственное поле E-mail адреса, позволяющее запросить ссылку сброса пароля:

    <!-- resources/views/auth/password.blade.php -->

    <form method="POST" action="/password/email">
        {!! csrf_field() !!}

        <div>
        	Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            <button type="submit">
                Send Password Reset Link
            </button>
        </div>
    </form>

Когда пользователь отправит запрос на сброс пароля, он получит E-mail со ссылкой на роут, который ведёт к  методу `getReset` (обычно это `/password/reset`) контроллера `PasswordController`. Вам нужно будет создать шаблон письма в `resources/views/emails/password.blade.php`. Этот шаблон получает переменную `$token`, содержащую токен сброса пароля для пользователя, который запросил его. Пример такого шаблона: 

    <!-- resources/views/emails/password.blade.php -->

    Click here to reset your password: {{ url('password/reset/'.$token) }}

#### Пример формы сброса пароля

Когда пользователь кликает на ссылку в письме сброса пароля, он попадает на страницу с формой сброса пароля. Шаблон этой страницы должен находиться в `resources/views/auth/reset.blade.php`.

Пример формы сброса пароля:

    <!-- resources/views/auth/reset.blade.php -->

    <form method="POST" action="/password/reset">
        {!! csrf_field() !!}
        <input type="hidden" name="token" value="{{ $token }}">

        <div>
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            <input type="password" name="password">
        </div>

        <div>
            <input type="password" name="password_confirmation">
        </div>

        <div>
            <button type="submit">
                Reset Password
            </button>
        </div>
    </form>

<a name="after-resetting-passwords"></a>
### Действия после сброса пароля

После того, как вы определили роуты и шаблоны для сброса паролей, можете проверить их в браузере. Класс `PasswordController` включён в фреймворк по умолчанию и содержит всю необходимую логику для отправки писем и сброса паролей в базе данных.

После сброса пароля пользователь автоматически аутентифицируется в вашем приложении и редиректится на страницу `/home`. Вы можете изменить этот путь, создав свойство `redirectTo` у `PasswordController`

    protected $redirectTo = '/dashboard';

> **Обратите внимание:** По умолчанию токен сброса пароля существует 1 час. Вы можете изменить это в конфигурационном файле `config/auth.php` с помощью опции `reminder.expire`.

<a name="social-authentication"></a>
## Аутентификация через социальные сети

В дополнение к аутентификации, основанной на формах, Laravel предоставляет простой и удобный способ аутентификации на основе OAuth  при помощи [Laravel Socialite](https://github.com/laravel/socialite). Socialite на данный момент поддерживает аутентификацию при помощи Facebook, Twitter, Google, GitHub и Bitbucket.

Для установки Socialite добавьте его как зависимость в `composer.json`:

    composer require laravel/socialite

### Настройка

После установки Socialite зарегистрируйте сервис-провайдер `Laravel\Socialite\SocialiteServiceProvider` в конфигурационном файле `config/app.php`:

    'providers' => [
        // Другие сервис-провайдеры

        Laravel\Socialite\SocialiteServiceProvider::class,
    ],

Также добавьте фасад `Socialite` в массив `aliases` конфигурации:

    'Socialite' => Laravel\Socialite\Facades\Socialite::class,

Далее вам необходимо указать параметры для того сервиса OAuth, который вы используете. Это можно сделать в файле `config/services.php`. Пока доступно четыре сервиса: `facebook`, `twitter`, `google` и `github`. Пример:

    'github' => [
        'client_id' => 'your-github-app-id',
        'client_secret' => 'your-github-app-secret',
        'redirect' => 'http://your-callback-url',
    ],

### Базовое использование

Теперь вы готовы к аутентификации пользователей! Вам нужно создать два роута: один для редиректа пользователя к провайдеру OAuth, и ещё один для получения callback от провайдера после аутентификации. Доступ к Socialite можно получить при помощи [фасада](/docs/{{version}}/facades) `Socialite`:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;

    class AuthController extends Controller
    {
        /**
         * Redirect the user to the GitHub authentication page.
         *
         * @return Response
         */
        public function redirectToProvider()
        {
            return Socialite::driver('github')->redirect();
        }

        /**
         * Obtain the user information from GitHub.
         *
         * @return Response
         */
        public function handleProviderCallback()
        {
            $user = Socialite::driver('github')->user();

            // $user->token;
        }
    }

Метод `redirect` заботится об отправке пользователя к OAuth провайдеру, а метод `user` читает приходящий ответ и получает информацию о пользователе от провайдера. Перед редиректом пользователя вы также можете указать «scopes» (права доступа) запроса при помощи метода `scope`. Этот метод перезапишет существующие scopes:

    return Socialite::driver('github')
                ->scopes(['scope1', 'scope2'])->redirect();

#### Получение деталей о пользователе

После получения экземпляра пользователя вы можете получить дополнительные данные о нём:

    $user = Socialite::driver('github')->user();

    // OAuth Two Providers
    $token = $user->token;

    // OAuth One Providers
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;

    // All Providers
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();

<a name="adding-custom-authentication-drivers"></a>
## Добавление драйвера аутентификации

Если вы не используете традиционные реляционные базы данных для хранения пользователей, вам нужно расширить Laravel собственным драйвером аутентификации. Для этого используйте метод `extend` фасада `Auth` для определения своего драйвера. Вы должны поместить `extend` в [сервис-провайдере](/docs/{{version}}/providers):

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
            Auth::extend('riak', function($app) {
                // Return an instance of Illuminate\Contracts\Auth\UserProvider...
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

После добавления драйвера с помощью метода `extend` вы можете переключиться на него в вашем файле конфигурации `config/auth.php`.

### Контракт User Provider

Реализации контракта `Illuminate\Contracts\Auth\UserProvider` ответственны только за получение реализации контракта `Illuminate\Contracts\Auth\Authenticatable` из хранилища данных, такого, например, как MySQL, Riak и тд. Эти два интерфейса позволяют Laravel использовать механизмы аутентификации независимо от того, как хранятся данные приложения.

Давайте посмотрим на контракт `Illuminate\Contracts\Auth\UserProvider`:

    <?php namespace Illuminate\Contracts\Auth;

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

Теперь, когда мы рассмотрели все методы `UserProvider`, давайте взглянем на  `Authenticatable`. Помните, провайдер должен возвращать реализацию этого интерфейса из методов `retrieveById` и `retrieveByCredentials`:

    <?php namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

Интерфейс прост. Метод `getAuthIdentifier` должен возвращать «primary key» пользователя. В MySQl, например, это уникальный первичный ключ. Метод `getAuthPassword` должен возвращать хеш пароля пользователя. Этот интерфейс позволяет системе аутентификации работать с любым классом User независимо от используемой ORM или базы данных. По умолчанию Laravel включает в себя класс User в папке `app`, который реализует этот интерфейс, поэтому вы можете использовать его, как пример.
