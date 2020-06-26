git 5d6ccc15a67dd5d9f902745acb5d9c0f8b05be68

---

# Аутентификация

- [Введение](#introduction)
    - [Требования для базы данных](#introduction-database-considerations)
- [Краткое руководство по аутентификации](#authentication-quickstart)
    - [Роутинг](#included-routing)
    - [Шаблоны](#included-views)
    - [Аутентификация](#included-authenticating)
    - [Получение аутентифицированного пользователя](#retrieving-the-authenticated-user)
    - [Защита роутов](#protecting-routes)
    - [Защита повторным вводом пароля](#password-confirmation)
    - [Троттлинг аутентификации (Ограничение числа неудачных попыток входа)](#login-throttling)
- [Аутентификация пользователей вручную](#authenticating-users)
    - [Запоминание пользователей](#remembering-users)
    - [Другие методы аутентификации](#other-authentication-methods)
- [HTTP-аутентификация](#http-basic-authentication)
    - [HTTP-аутентификация без сохранения состояния](#stateless-http-basic-authentication)
- [Разлогинивание](#logging-out)
    - [Инвалидация сессий на других устройствах](#invalidating-sessions-on-other-devices)
- [Аутентификация через социальные сети](https://github.com/laravel/socialite)
- [Добавление собственных гвардов](#adding-custom-guards)
    - [в виде анонимных функций](#closure-request-guards)
- [Добавление собственных провайдеров пользователей](#adding-custom-user-providers)
    - [Контракт User Provider](#the-user-provider-contract)
    - [Контракт Authenticatable](#the-authenticatable-contract)
- [События](#events)

<a name="introduction"></a>
## Введение

Аутентификация - это процесс регистрации и логина пользователей. Не путайте с авторизацией - проверкой прав уже залогиненного пользователя.

> {tip} **Хотите быстро начать работу?** Установите composer-пакет `laravel/ui` (1.0) и запустите `php artisan ui vue --auth` в свежем приложении Laravel. Запустите миграцию базы данных и зайдите на `http://your-app.test/register`. У вас появилась готовая система аутентификации, которую вы можете менять под себя как захотите!

В Laravel сделать аутентификацию очень просто. Фактически, почти всё сконфигурировано для вас уже изначально. Конфигурационный файл аутентификации расположен в `config/auth.php`, он содержит несколько хорошо описанных опций для тонкой настройки поведения служб аутентификации.

По сути средства аутентификации Laravel состоят из "гвардов" и "провайдеров". Гварды определяют то, как именно аутентифицируются пользователи, для каждого запроса. Например, Laravel поставляется с гвардом `session`, который поддерживает состояние аутентифицированности с помощью хранилища сессий и кук.

Провайдеры определяют то, как именно пользователи извлекаются из вашей базы данных. Laravel поставляется с поддержкой извлечения пользователей с помощью Eloquent и конструктора запросов БД. Но при необходимости вы можете определить дополнительные провайдеры для своего приложения.

Не переживайте, если сейчас это звучит слишком запутанно! Для большинства приложений никогда не потребуется изменять стандартные настройки аутентификации.

<a name="introduction-database-considerations"></a>
### Требования для базы данных

По умолчанию в Laravel есть [модель Eloquent](/docs/{{version}}/eloquent) `App\User` в директории `app`. Эта модель может использоваться с базовым драйвером аутентификации Eloquent. Если ваше приложение не использует Eloquent, вы можете использовать драйвер аутентификации `database`, который использует конструктор запросов Laravel.

При создании схемы базы данных для модели `App\User` создайте столбец для паролей с длиной не менее 60 символов. Хорошим выбором будет длина 255 символов.

Кроме того, перед началом работы удостоверьтесь, что ваша таблица `users` (или эквивалентная) содержит строковый столбец `remember_token` на 100 символов. Этот столбец будет использоваться для хранения ключей сессий «запомнить меня», обрабатываемых вашим приложением.

<a name="authentication-quickstart"></a>
## Краткое руководство по аутентификации

Laravel поставляется с несколькими контроллерами аутентификации, расположенными в пространстве имён `App\Http\Controllers\Auth`. `RegisterController` обрабатывает регистрацию нового пользователя, `LoginController` - его аутентификацию, `ForgotPasswordController` обрабатывает отправку по электронной почте ссылок на сброс пароля, а `ResetPasswordController` содержит логику для сброса паролей. Каждый из этих контроллеров использует типажи для подключения необходимых методов. Для многих приложений вам вообще не придётся изменять эти контроллеры.

<a name="included-routing"></a>
### Роутинг

Laravel обеспечивает быстрый способ создания заготовок всех необходимых для аутентификации роутов и шаблонов при помощи пакета `laravel/ui`:

    composer require laravel/ui "^1.0" --dev

    php artisan ui vue --auth

Эту команду надо использовать на свежем приложении, она установит шаблоны для регистрации и логина, а также роуты, требуемые для системы аутентификации. Также будет сгенерирован `HomeController`, на коорый будет сделан редирект залогиненного пользователя. Но вы можете изменить или даже удалить это контроллер, если это необходимо для вашего приложения.

> {tip} Если вы хотите запретить регистрацию, удалите контроллер `RegisterController` , который также делает эта команда, и измените роуты аутентификации: `Auth::routes(['register' => false]);`.

#### Создание системы аутентификации вместе с созданием приложения

На этапе создания приложения вы можете указать флаг `--auth` и инсталлятор сразу добавит систему аутентификации:

    laravel new blog --auth

<a name="included-views"></a>
### Шаблоны

Как было упомянуто в предыдущем разделе, команда `php artisan ui vue --auth` из пакета `laravel/ui` создаст все необходимые вам шаблоны для аутентификации и поместит их в директорию `resources/views/auth`.

Команда `ui` также создаст директорию `resources/views/layouts`, содержащую основной макет для вашего приложения. Все эти макеты используют CSS-фреймворк Bootstrap, но вы можете изменять их как угодно.

<a name="included-authenticating"></a>
### Аутентификация

Теперь, когда у вас есть роуты и шаблоны для имеющихся контроллеров аутентификации, вы готовы регистрировать и аутентифицировать новых пользователей своего приложения! Вы можете просто перейти по этим роутам в браузере. Контроллеры аутентификации уже содержат логику (благодаря их типажам) для аутентификации существующих пользователей и сохранения новых пользователей в базе данных.

#### Изменение пути

Когда пользователь успешно аутентифицируется, он будет перенаправлен на URI `/home`. Вы можете изменить место для перенаправления после входа, задав константу `HOME` в сервис-провайдере `RouteServiceProvider`:

    public const HOME = '/home';

Если вам нужна более гибкая настройка ответа, возвращаемого при аутентификации пользователя, Laravel предоставляет пустой метод `authenticated(Request $request, $user)`, который, при желании, может быть переопределен:

    /**
     * The user has been authenticated.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  mixed  $user
     * @return mixed
     */
    protected function authenticated(Request $request, $user)
    {
        return response([
            //
        ]);
    }

#### Настройка имени пользователя

По умолчанию для аутентификации Laravel использует поле `email`. Если вы хотите это изменить, то можете определить метод `username` в своем `LoginController`:

    public function username()
    {
        return 'username';
    }

#### Настройка гварда

Вы также можете изменить "гварда", который используется для аутентификации и регистрации пользователей. Для начала задайте метод `guard` в `LoginController`, `RegisterController` и `ResetPasswordController`. Метод должен возвращать экземпляр гварда:

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### Настройка валидации / хранения

Чтобы изменить требуемые поля для формы регистрации нового пользователя, или для изменения способа добавления новых записей в вашу базу данных, вы можете изменить класс `RegisterController`. Этот класс отвечает за проверку ввода и создание новых пользователей в вашем приложении.

Метод `validator` класса `RegisterController` содержит правила проверки ввода данных для новых пользователей приложения.

Метод `create` класса `RegisterController` отвечает за создание новых записей  `App\User` в вашей базе данных с помощью [Eloquent ORM](/docs/{{version}}/eloquent). Вы можете изменить каждый из этих методов, как пожелаете.

<a name="retrieving-the-authenticated-user"></a>
### Получение аутентифицированного пользователя

Вы можете обращаться к аутентифицированному пользователю через фасад `Auth`:

    use Illuminate\Support\Facades\Auth;

    // получить текущего залогиненного юзера
    $user = Auth::user();

    // получить id текущего залогиненного юзера
    $id = Auth::id();

Или, когда пользователь аутентифицирован, вы можете обращаться к нему через экземпляр `Illuminate\Http\Request`. Не забывайте, что указание типов классов приводит к их автоматическому внедрению в методы вашего контроллера:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class ProfileController extends Controller
    {
        /**
         * Обновление профиля пользователя.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // $request->user() возвращает экземпляр аутентифицированного пользователя...
        }
    }

#### Определение, аутентифицирован ли пользователь

Чтобы определить, что пользователь уже вошёл в ваше приложение, вы можете использовать метод `check` фасада `Auth`, который вернёт `true`, если пользователь аутентифицирован:

    use Illuminate\Support\Facades\Auth;

    if (Auth::check()) {
        // Пользователь вошёл в систему...
    }

> {tip} Хотя и возможно определить аутентифицирован ли пользователь, используя метод `check`, обычно вы будете использовать посредников для проверки был ли пользователь аутентифицирован ранее, позволяя этому пользователю получать доступ к определенным роутам / контроллерам. Для получения дополнительной информации смотрите документацию о [защите роутов](/docs/{{version}}/authentication#protecting-routes).

<a name="protecting-routes"></a>
### Защита роутов

Чтобы давать доступ к определённому роуту только аутентифицированным пользователям можно использовать специальный [посредник (middleware)](/docs/{{version}}/middleware). Laravel поставляется с посредником `auth`, который определён в `Illuminate\Auth\Middleware\Authenticate`. Так как этот посредник уже зарегистрирован в фреймворке и всё, что вам надо сделать — это присоединить его к определению роута:

    Route::get('profile', function () {
        // Только аутентифицированные пользователи могут зайти...
    })->middleware('auth');

Вы также можете вызвать метод `middleware` из конструктора контроллера, вместо присоединения его напрямую к определению роута:

    public function __construct()
    {
        $this->middleware('auth');
    }

#### Обработка неаутентифицированных пользователей

Когда посредник `auth` определяет, что пользователь неаутентифицирован, он перенаправляет его на [роут](/docs/{{version}}/routing#named-routes) с именем `login`. Вы можете изменить это поведение в методе `redirectTo` класса `app/Http/Middleware/Authenticate.php`:

    /**
     * Получить путь, куда отправлять гостей с защищённого роута
     *
     * @param  \Illuminate\Http\Request  $request
     * @return string
     */
    protected function redirectTo($request)
    {
        return route('login');
    }    

#### Указание гварда

Во время прикрепления посредника `auth` к роуту, вы можете также указать, какой гвард должен быть использован для выполнения аутентификации. Указанный гвард должен соответствовать одному из ключей в массиве `guards` вашего конфига `auth.php`:

    public function __construct()
    {
        $this->middleware('auth:api');
    }

<a name="password-confirmation"></a>
### Защита повторным вводом пароля

Особо важные места сайта (изменение критичных настроек - например, платёжной информации) можно защитить путём повторного запроса пароля у пользователя.

Для этого защитите роут или группу роутов посредником `password.confirm` . При заходе на этот урл будет произведён редирект на промежуточную страницу с формой ввода пароля.

    Route::get('/settings/security', function () {
        // Users must confirm their password before continuing...
    })->middleware(['auth', 'password.confirm']);

После успешного ввода пароля произойдёт редирект на защищаемый роут. При повтором входе на этот роут пароль не будет спрашиваться в течении трёх часов. Вы можете изменить это время путём редактирования опции `auth.password_timeout` в конфиге.

<a name="login-throttling"></a>
### Троттлинг аутентификации (ограничение числа неудачных попыток входа)

Если вы используете встроенный в Laravel класс `LoginController`, трейт `Illuminate\Foundation\Auth\ThrottlesLogins` уже будет включён в ваш контроллер. По умолчанию пользователь не сможет войти в приложение в течение одной минуты, если он несколько раз указал неправильные данные для входа. Ограничение происходит отдельно для имени пользователя/адреса e-mail и его IP-адреса.

<a name="authenticating-users"></a>
## Аутентификация пользователей вручную

Вы можете написать свои контроллеры аутентификации пользователей, не пользоваться теми, что генерирует фреймворк, если по какой-то причине они вас не устраивают. Для этого вам нужно использовать в своих контроллерах классы аутентификации Laravel. Не волнуйтесь, это просто.

Мы будем работать со службами аутентификации Laravel через [фасад](/docs/{{version}}/facades) `Auth`, поэтому нам надо не забыть импортировать фасад Auth в начале класса. Далее давайте посмотрим на метод `attempt`:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Auth;

    class LoginController extends Controller
    {
        /**
         * Обработка попытки аутентификации.
         *
         * @param  \Illuminate\Http\Request $request
         * @return Response
         */
        public function authenticate(Request $request)
        {
            $credentials = $request->only('email', 'password');

            if (Auth::attempt(credentials)) {
                // Аутентификация успешна...
                return redirect()->intended('dashboard');
            }
        }
    }

Метод `attempt` принимает массив пар ключ/значение в качестве первого аргумента. Значения массива будут использованы для поиска пользователя в таблице базы данных. Так, в приведённом выше примере пользователь будет получен по значению столбца `email`. Если пользователь будет найден, хешированный пароль, сохранённый в базе данных, будет сравниваться с хешированным значением `password` , переданным в метод через массив. Вы не должны хешировать пароль, указанный в качестве значения `password`, поскольку фреймворк автоматически хеширует значение, прежде чем сравнивать его с хешированным паролем в базе данных. Если два хешированных пароля совпадут, то для пользователя будет запущена новая аутентифицированная сессия.

Метод `attempt` вернет `true`, если аутентификация прошла успешно. В противном случае будет возвращён `false`.

Метод `intended` "редиректора" перенаправит пользователя к тому URL, к которому он обращался до того, как был перехвачен фильтром аутентификации. В этот метод можно передать запасной URI, на случай недоступности требуемого пути.

#### Указание дополнительных условий

При необходимости вы можете добавить дополнительные условия к запросу аутентификации, помимо адреса e-mail и пароля. Например, можно проверить отметку "активности" пользователя:

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // Пользователь активен, не приостановлен и существует.
    }

> {note} В этих примерах `email` не является обязательным вариантом, он приведён только для примера. Вы можете использовать какой угодно столбец, соответствующий "username" в вашей базе данных.

#### Обращение к конкретным экземплярам гварда

С помощью метода `guard` фасада `Auth` вы можете указать, какой экземпляр гварда необходимо использовать. Это позволяет управлять аутентификацией для отдельных частей вашего приложения, используя полностью отдельные модели для аутентификации или таблицы пользователей.

Передаваемое в метод `guard` имя гварда должно соответствовать одному из защитников, настроенных в конфиге `auth.php`:

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

#### Завершение сессии

Для завершения сессии пользователя можно использовать метод `logout` фасада `Auth`. Он очистит информацию об аутентификации в сессии пользователя:

    Auth::logout();

<a name="remembering-users"></a>
### Запоминание пользователей

Если вы хотите обеспечить функциональность "запомнить меня" в вашем приложении, вы можете передать логическое значение как второй параметр методу `attempt`, который сохранит пользователя аутентифицированным на неопределённое время, или пока он вручную не выйдет из системы. Ваша таблица `users` должна содержать строковый столбец `remember_token`, который будет использоваться для хранения токенов "запомнить меня".

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // Пользователь запомнен...
    }

> {tip} Если вы используете встроенный в Laravel `LoginController`, логика "запоминания" пользователей уже реализована трейтами, используемыми контроллером.

Если вы "запоминаете" пользователей, то можете использовать метод `viaRemember` , чтобы определить, аутентифицировался ли пользователь, используя cookie "запомнить меня":

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### Другие методы аутентификации

#### Аутентификация экземпляра пользователя

Если вам необходимо "залогинить" в приложение существующий экземпляр пользователя, вызовите метод `login` с экземпляром пользователя. Данный объект должен быть реализацией [контракта](/docs/{{version}}/contracts) `Illuminate\Contracts\Auth\Authenticatable`. Конечно, встроенная в Laravel модель `App\User` реализует этот интерфейс:

    Auth::login($user);

    // Войти и "запомнить" данного пользователя...
    Auth::login($user, true);

Конечно, вы можете указать, какой экземпляр гварда надо использовать:

    Auth::guard('admin')->login($user);

#### Аутентификация пользователя по ID

Для входа пользователя в приложение по его ID, используйте метод `loginUsingId`. Этот метод просто принимает первичный ключ пользователя, которого необходимо аутентифицировать:

    Auth::loginUsingId(1);

    // Войти и "запомнить" данного пользователя...
    Auth::loginUsingId(1, true);

#### Однократная аутентификация пользователя

Вы также можете использовать метод `once` для пользовательского входа в систему для одного запроса. Сеансы и cookies не будут использоваться, что может быть полезно при создании API без сохранения состояний:

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## HTTP-аутентификация

[HTTP-аутентификация](https://en.wikipedia.org/wiki/Basic_access_authentication) — простой и быстрый способ аутентификации пользователей вашего приложения без создания дополнительной страницы входа. Для начала прикрепите [посредника](/docs/{{version}}/middleware) `auth.basic` к своему роуту. Посредник `auth.basic` встроен в Laravel, поэтому вам не надо определять его:

    Route::get('profile', function () {
        // Войти могут только аутентифицированные пользователи...
    })->middleware('auth.basic');

Когда посредник прикреплён к роуту, вы автоматически получите запрос данных для входа при обращении к роуту через браузер. По умолчанию посредник `auth.basic` будет использовать столбец `email` из записи пользователя в качестве "username".

#### Замечание по FastCGI

Если вы используете PHP FastCGI, то простая HTTP-аутентификация изначально может работать неправильно. Надо добавить следующие строки к вашему файлу `.htaccess`:

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### HTTP-аутентификация без сохранения состояния

Вы также можете использовать HTTP-аутентификацию, не задавая пользовательскую cookie для сессии, что особенно полезно для API-аутентификации. Чтобы это сделать, [определите посредника](/docs/{{version}}/middleware), который вызывает метод `onceBasic`. Если этот метод ничего не возвращает, запрос может быть передан дальше в приложение:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Support\Facades\Auth;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * Обработка входящего запроса.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

Затем [зарегистрируйте посредника роута](/docs/{{version}}/middleware#registering-middleware) и прикрепите его к роуту:

    Route::get('api/user', function () {
        // Войти могут только аутентифицированные пользователи...
    })->middleware('auth.basic.once');

<a name="logging-out"></a>
## Завершение сессии

Чтобы разлогинить пользователя используйте метод `logout` фасада `Auth`. Он очистит информацию об аутентификации в сессии пользователя:

    use Illuminate\Support\Facades\Auth;

    Auth::logout();

<a name="invalidating-sessions-on-other-devices"></a>
### Инвалидация сессий на других устройствах

В Laravel есть механизм для завершении сессии на других устройствах (браузерах), оставляя сессию на данном устройстве активной. Например, это требуется когда пользователь изменяет свой пароль.

Чтобы этот функционал работал, в `app/Http/Kernel.php` в секции `web` должен быть добавлен посредник `Illuminate\Session\Middleware\AuthenticateSession`

    'web' => [
        // ...
        \Illuminate\Session\Middleware\AuthenticateSession::class,
        // ...
    ],

Теперь вы можете использовать метод `logoutOtherDevices` фасада `Auth`. В качестве аргумента он принимает текущий пароль пользователя.

    use Illuminate\Support\Facades\Auth;

    Auth::logoutOtherDevices($password);

При вызове метода `logoutOtherDevices` все сессии пользователя кроме текущей будут инвалидированы, что означает, что пользователь будет "разлогинен" во всех гвардах, в которых он был ранее аутентифицирован.

> {note} При использовании посредника `AuthenticateSession` вместе с нестандартным именем маршрута `login`, вы должны переопределить метод `unauthenticated` в обработчике исключений вашего приложения, чтобы корректно перенаправить пользователей на страницу входа в систему.

<a name="adding-custom-guards"></a>
## Добавление собственных гвардов

Вы можете определить собственные гварды аутентификации, используя метод `extend` фасада `Auth`. Вы должны поместить этот вызов `extend` внутри [сервис-провайдера](/docs/{{version}}/providers). Так как в состав Laravel уже входит `AuthServiceProvider`, можно поместить данный код в провайдер:

    <?php

    namespace App\Providers;

    use App\Services\Auth\JwtGuard;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    use Illuminate\Support\Facades\Auth;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Выполнение пост-регистрационной загрузки служб.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::extend('jwt', function ($app, $name, array $config) {
                // Вернуть экземпляр Illuminate\Contracts\Auth\Guard...

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }
    }

Как видите, в этом примере переданная в метод `extend` анонимная функция должна вернуть реализацию `Illuminate\Contracts\Auth\Guard`. Этот интерфейс содержит несколько методов, которые вам надо реализовать, для определения собственного гварда. Когда вы определили своего гварда, вы можете использовать его в настройке `guards` своего конфига `auth.php`:

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="closure-request-guards"></a>
### Гварды в виде анонимных функций

Функционалом аутентификации можно воспользоваться без создания классов гвардов, при помощи анонимной функции в `Auth::viaRequest`.

Добавьте `Auth::viaRequest` в метод `boot` вашего `AuthServiceProvider`. В качестве аргументов `viaRequest` принимает название драйвера аутентификации, который вы определите в функции и собственно саму функцию, которая возвращает экземпляр класса User, залогиненного пользователя, если аутентификация пройдена и `null` если нет. На вход эта функция принимает экземпляр класса HTTP запроса.

Вот пример аутентификации по токену, который передан в запросе:

    use App\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Auth;

    /**
     * Register any application authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Auth::viaRequest('custom-token', function ($request) {
            return User::where('token', $request->token)->first();
        });
    }

Чтобы сообщить приложению, что аутентификацию нужно проводить при помощи драйвера, указанного в `viaRequest`, укажите его имя в конфиге `auth.php` в секции `guards`:

    'guards' => [
        'api' => [
            'driver' => 'custom-token',
        ],
    ],


<a name="adding-custom-user-providers"></a>
## Добавление собственных провайдеров пользователей

Если вы не используете традиционную реляционную базу данных для хранения ваших пользователей, вам необходимо добавить в Laravel свой собственный провайдер аутентификации пользователей. Мы используем метод `provider` фасада `Auth` для определения своего провайдера:

    <?php

    namespace App\Providers;

    use App\Extensions\RiakUserProvider;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    use Illuminate\Support\Facades\Auth;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Выполнение пост-регистрационной загрузки служб.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::provider('riak', function ($app, array $config) {
                // Вернуть экземпляр Illuminate\Contracts\Auth\UserProvider...

                return new RiakUserProvider($app->make('riak.connection'));
            });
        }
    }

После регистрации провайдера методом `provider`, вы можете переключиться на новый провайдер в файле настроек `auth.php`. Сначала определите провайдера `provider`, который использует ваш новый драйвер:

    'providers' => [
        'users' => [
            'driver' => 'riak',
        ],
    ],

Затем вы можете использовать этот провайдер в вашей настройке `guards`:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

<a name="the-user-provider-contract"></a>
### Контракт User Provider

Реализации `Illuminate\Contracts\Auth\UserProvider` отвечают только за извлечение реализаций `Illuminate\Contracts\Auth\Authenticatable` из постоянных систем хранения, таких как MySQL, Riak, и т.п. Эти два интерфейса позволяют механизмам аутентификации Laravel продолжать функционировать независимо от того, как хранятся данные пользователей и какой тип класса использован для их представления.

Давайте взглянем на контракт `Illuminate\Contracts\Auth\UserProvider`:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider 
    {
        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);
    }

Функция `retrieveById` обычно принимает ключ, отображающий пользователя, такой как автоинкрементный ID из базы данных MySQL. Реализация `Authenticatable`, соответствующая этому ID, должна быть получена и возвращена этим методом.

Функция `retrieveByToken` принимает пользователя по его уникальному `$identifier` и ключу `$token`  "запомнить меня", хранящемуся в поле `remember_token`. Как и предыдущий метод, он должен возвращать реализацию `Authenticatable`.

Метод `updateRememberToken` обновляет поле `remember_token` пользователя `$user` значением нового `$token`. 

Метод `retrieveByCredentials` принимает массив авторизационных данных, переданных в метод `Auth::attempt` при попытке входа в приложение. Затем метод должен "запросить" у основного постоянного хранилища того пользователя, который соответствует этим авторизационным данным. Обычно этот метод выполняет запрос с условием "where" для `$credentials['username']`. Затем метод должен вернуть реализацию `Authenticatable`. **Этот метод не должен пытаться проверить пароль или аутентифицировать пользователя.**

Метод `validateCredentials` должен сравнить данного `$user` с `$credentials` для аутентификации пользователя. Например, этот метод может сравнить строку `$user->getAuthPassword()` с `Hash::check` для сравнения значения `$credentials['password']`. Этот метод должен возвращать `true` или `false`, указывая верен ли пароль.

<a name="the-authenticatable-contract"></a>
### Контракт Authenticatable

Теперь, когда мы изучили каждый метод в `UserProvider`, давайте посмотрим на контракт `Authenticatable`. Помните, провайдер должен вернуть реализацию этого интерфейса из методов `retrieveById`, `retrieveByToken`, и `retrieveByCredentials`:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable 
    {
        public function getAuthIdentifierName();
        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();
    }

Этот интерфейс довольно прост. Метод `getAuthIdentifierName` должен возвращать имя поля "первичного ключа» пользователя", а метод `getAuthIdentifier` - "первичный ключ" пользователя. При использовании MySQL это будет автоинкрементный первичный ключ. Метод `getAuthPassword` должен возвращать хешированный пароль пользователя. Этот интерфейс позволяет системе аутентификации работать с классом User, независимо от используемой ORM и уровня абстракции хранилища. По умолчанию Laravel содержит в директории `app` класс `User`, который реализует этот интерфейс. Вы можете подсмотреть в нём пример реализации.

<a name="events"></a>
## События

Laravel генерирует различные [события](/docs/{{version}}/events) в процессе аутентификации. Вы можете прикрепить слушателей к этим событиям в вашем `EventServiceProvider`:

    /**
     * Сопоставления слушателя событий для вашего приложения.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Registered' => [
            'App\Listeners\LogRegisteredUser',
        ],

        'Illuminate\Auth\Events\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],

        'Illuminate\Auth\Events\Authenticated' => [
            'App\Listeners\LogAuthenticated',
        ],

        'Illuminate\Auth\Events\Login' => [
            'App\Listeners\LogSuccessfulLogin',
        ],

        'Illuminate\Auth\Events\Failed' => [
            'App\Listeners\LogFailedLogin',
        ],

        'Illuminate\Auth\Events\Logout' => [
            'App\Listeners\LogSuccessfulLogout',
        ],

        'Illuminate\Auth\Events\Lockout' => [
            'App\Listeners\LogLockout',
        ],
        
        'Illuminate\Auth\Events\PasswordReset' => [
            'App\Listeners\LogPasswordReset',
        ],
    ];
