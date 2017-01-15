git ae321319ee005632fcfe4a5bf8f3607dbb198855

---

# Аутентификация

- [Введение](#introduction)
    - [Замечания по базе данных](#introduction-database-considerations)
- [Аутентификация - быстрый старт](#authentication-quickstart)
    - [Маршрутизация](#included-routing)
    - [Представления](#included-views)
    - [Проверка подлинности](#included-authenticating)
    - [Получение аутентифицированного пользовтателя](#retrieving-the-authenticated-user)
    - [Защита маршрутов](#protecting-routes)
    - [Регулирование входа](#login-throttling)
- [Ручное аутентифицирование](#authenticating-users)
    - [Запоминание пользователей](#remembering-users)
    - [Другие способы аутентификации](#other-authentication-methods)
- [Базовая HTTP Аутентификация](#http-basic-authentication)
    - [HTTP Аутентификация без сохранения состояния](#stateless-http-basic-authentication)
- [Добавление своих Guard'ов](#adding-custom-guards)
- [Добавление своих User Provider'ов](#adding-custom-user-providers)
    - [Контракт User Provider](#the-user-provider-contract)
    - [Контракт Authenticatable](#the-authenticatable-contract)
- [События](#events)

<a name="introduction"></a>
## Введение
> Нужен быстрый старт? Просто запустите `php artisan make:auth`  в свежеустановленном приложении Laravel и перейдите в вашем браузере по адресу http://your-app.dev/register или любому другому URL, который назначен для вашего приложения. Эта команда позаботится о создании каркаса всей системы аутентификации!

Laravel позволяет очень легко реализовать аутентификацию. На самом деле, почти все уже настроено из коробки. Файл конфигурации аутентификации находится в `сonfig/auth.php`, который содержит несколько хорошо задокументированных вариантов тонкой настройки поведения служб аутентификации.

По своей сути, средства аутентификации Laravel, выполнены из «охранников»(Guard) и «поставщиков»(Provider). Guard определяет, как пользователи проходят проверку подлинности при каждом запросе. В качестве примера: Laravel поставляется с Guard'ом `session`, который работает с сессией, используя session storage и cookie.

Provider'ы определяют, каким образом пользователи извлекаются с постоянного хранения. Laravel поставляется с поставщиками пользователей Eloquent и Database Query Builder. Но вы так же можете определить и свой поставщик если это понадобится в приложении.

Не переживайте, если сейчас все это звучит слишком запутано! Многим приложениям вообще не потребуется изменять конфигурацию аутентификации по умолчанию.

<a name="introduction-database-considerations"></a>
### Замечания по базе данных

По умолчанию, Laravel включает в себя `App\User` [модель Eloquent] (/docs/{{version}}/eloquent) находящуюся в файле `User.php` каталога `app`. Эта модель может быть использована с eloquent драйвером аутентификации по умолчанию. Если приложение не использует eloquent, вы можете использовать `database` драйвер аутентификации, который использует Laravel Query Builder.

При построении схемы базы данных для модели `App\User`, убедитесь, что колонка пароля вмещает не менее 60 символов в длину. Строки в 255 символов, задаваемая по умолчанию отлично подойдет.

Кроме того, вы должны убедиться, что таблица `users` (или эквивалент) содержит колонку ` remember_token` являющуюся строкой не менее 100 символов. Этот столбец будет использоваться для хранения токена пользователей, установивших флажок "Запомнить меня", при входе в приложение.


<a name="authentication-quickstart"></a>
## Аутентификация - быстрый старт

Laravel поставляется с несколькими уже созданными контроллерами аутентификации, которые располагаются в пространстве имен `App\Http\Controlers\Auth`. `RegisterController` обрабатывает регистрацию новых пользователей,` LoginController` выполняет аутентификацию, `ForgotPasswordController` обрабатывает отправку по электронной почте ссылок для сброса пароля, а` ResetPasswordController` содержит логику для сброса пароля. Каждый из этих контроллеров использует трейт, чтобы подключить необходимые методы. Для многих приложений, вам вообще не нужно будет изменять эти контроллеры.

<a name="included-routing"></a>
### Маршрутизация

Laravel предоставляет быстрый способ создания всех маршрутов и представлений, которые необходимы для аутентификации с помощью одной простой команды:

```
php artisan make:auth
```

Эта команда должна использоваться на свежеустановленном приложении, и установит шаблоны представлений (view templates) регистрации и входа в систему, а также маршруты для аутентификации. Кроме того будет сгенерирован `HomeController` для обработки запросов к "приборной панели" приложения после входа в систему.

<a name="included-views"></a>
### Представления
Как уже упоминалось в предыдущем разделе,  команда `php artisan make:auth` создаст все представления, необходимые для аутентификации и поместит их в каталог `resource/views/auth`.

Команда `make:auth` также создаст каталог ` resource/views/layouts` , содержащий базовый макет для приложения. Все эти представления используют разметку CSS основанную на Bootstrap, но вы можете настроить их, как вы хочется.


<a name="included-authenticating"></a>
### Аутентификация

Теперь, когда у вас есть маршруты, представления и контроллеры аутентификации, вы готовы к регистрации и аутентификации новых пользователей! Вы можете получить доступ к этому приложению в браузере, так как контроллеры аутентификации уже содержат (через трейты) в себе всю логику аутентификации существующих пользователей и сохранения новых пользователей в базе данных.

#### Настройка путей

Когда пользователь успешно прошел аутентификацию, он будет перенаправлен на URI `/home`. Вы можете настроить адрес редиректа после аутентификации путем определения свойства `redirectTo` в ` LoginController`, `RegisterController` и` ResetPasswordController`:

```php
protected $redirectTo = '/';
```

Если пользователю не удалось пройти аутентификацию, то он будет автоматически перенаправлен обратно на форму входа в систему.


#### Настройка Guard

Вы также можете настроить "Guard", который используется для аутентификации и регистрации пользователей. Чтобы приступить к работе, определите метод `guard` в ` LoginController`, `RegisterController` и` ResetPasswordController`. Этот метод должен возвращать экземпляр Guard:

```php

use Illuminate\Support\Facades\Auth;

protected function guard()
{
    return Auth::guard('guard-name');
}
```

#### Валидация / Настройка хранилища

Чтобы изменить поля формы, которые необходимы при регистрации нового пользователя в приложении, или настроить, каким образом новые пользователи хранятся в базе данных, вы можете изменить класс `RegisterController`. Этот класс отвечает за проверку и создание новых пользователей приложения.

Метод `validator` в `RegisterController` содержит правила проверки новых пользователей приложения. Вы можете изменить этот метод, как вам хочется.

Метод `сreate` отвечает за создание новых `App\User` записей в базе данных с помощью функции [Eloquent ORM] (/docs/{{version}}/eloquent). Вы можете изменить этот метод в соответствии с потребностями структуры вашей БД.


<a name="retrieving-the-authenticated-user"></a>
### Получение аутентифицированного пользователя
Вы можете получить доступ к аутентифицированому пользователю через  фасад `Auth`:
```
use Illuminate\Support\Facades\Auth;
$user = Auth::user();
```
Как вариант, как только пользователь прошел аутентификацию, вы можете получить доступ к аутентифицированому пользователю через экземпляр `Illumnate\Http\Request`. Помните, что типизированные классом аргументы будут автоматически подставляться в методы вашего контроллера:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class ProfileController extends Controller
{
    /**
        * Update the user's profile.
        *
        * @param  Request  $request
        * @return Response
        */
    public function update(Request $request)
    {
        // $request->user() returns an instance of the authenticated user...
    }
}
```

#### Проверка аутентифицирован ли текущий пользователь
Для того, чтобы определить, является ли пользователь аутентифицированным, вы можете использовать метод `check` фасада `Auth`, который вернет `true`, если пользователь прошел проверку подлинности:
```php
use Illuminate\Support\Facades\Auth;

if (Auth::check()) {
    // The user is logged in...
}
```
> Несмотря на то, что можно определить, является ли аутентифицированным с помощью метода `check` , чаще всего вы будете использовать посредник (middleware), чтобы убедиться, что пользователь прошел аутентификацию, прежде чем разрешить пользователю доступ к определенным маршрутам/контроллерам. Чтобы узнать больше, читайте документацию по [защите маршрутов] (/ docs/{{version}}/authentication#protecting-routes).

> *здесь и далее говоря о "middleware" применяется слово "посредник", и оно употребляется как неодушевленное. Т.е - в в.п. отвечает не на вопрос "кого?", а на вопрос "что? Вообще, выбор слова "посредник" - довольно сомнительное решение, но оно было принято гораздо раньше.* - п.п.


<a name="protecting-routes"></a>
### Защита маршрутов
[Посредник маршрутов](/docs/{{version}}/middleware) может быть использован, чтобы позволить получить доступ к данному маршруту только пользователям прошедшим аутентификацию. Laravel поставляется с посредником `auth`, который описан в классе `Illuminate\Auth\Middleware\Authenticate`. Так как этот посредник уже зарегистрирован в HTTP kernel, все, что вам нужно сделать, это привязать посредник к конкретному маршруту:

```php
Route::get('profile', function () {
    // только аутентифицированные пользователи могут войти...
})->middleware('auth');
```
Конечно, если вы используете [контроллеры](/docs/{{version}}/controllers),  вы можете вызвать метод `middleware` прямо из конструктора контроллера вместо привязки его при определении маршрута:

```php
public function __construct()
{
    $this->middleware('auth');
}
```

#### Определение Guard

При привязке посредника `auth` к маршруту, вы можете также указать, какой конкретно Guard должен использоваться для аутентификации пользователя.  Указанный Guard должен соответствовать одному из ключей в массиве  `guards` файла `config/auth.php`:

```
public function __construct()
{
    $this->middleware('auth:api');
}
```

<a name="login-throttling"></a>
### Ограничение попыток входа

Если вы используете встроенный в Laravel класс `LoginController`, то `Illuminate\Foundation\Auth\ThrottlesLogins` трейт уже будет включена в ваш контроллер. По умолчанию, пользователь не сможет войти в систему в течение одной минуты, если они не предоставил правильные учетные данные несколько раз подряд. Применение ограничения является уникальным для сочетания имени пользователя/адреса почты и IP-адреса.

> *это место оказалось довольно сложным для перевода, так как английское слово `throttling` не имеет адекватного русского аналога, в полной мере отражающего смысл в текущем контексте* - п.п.


<a name="authenticating-users"></a>
## Аутентифкация пользователей "вручную"

Конечно, вы не обязаны использовать контроллеры аутентификации встроенные в Laravel. Если вы решили удалить эти контроллеры, вам придется управлять аутентификацией пользователя непосредственно с помощью классов аутентификации Laravel. Не волнуйтесь, это возможно!

Мы будем получать доступ к службе аутентификации Laravel через [фасад](/docs/{{version}}/facades) `Auth`, так что мы должны убедиться, что импортировали `Auth` в верхней части файла. Теперь, давайте попробуем применить метод `attempt`:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Auth;

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
            // Аутентификация прошла...
            return redirect()->intended('dashboard');
        }
    }
}
```

Метод `attempt` принимает массив пар ключ/значение в качестве первого аргумента. Значения в массиве будут использоваться, чтобы найти пользователя в таблице базы данных. Таким образом, в приведенном выше примере, пользователь будет найден по значению столбца `email`. Если пользователь найден, хэш пароля хранящийся в базе данных будет сравниваться с хэшем значения `password`, переданного методу в массиве. Если хэши пароля совпадают для пользователя будет запущена аутентифицрованная сессия.

`Метод attempt` вернет значение `true`, если аутентификация прошла успешно. В противном случае, будет возвращено значение `false`.

метод `intended` указанные на редиректе будет перенаправлять пользователя на URL к которому он пытался получить доступ когда был перехвачен посредником `auth`. Вариант по умолчанию будет применен, если пользователь не был перехвачен, а перешел на страницу входа по прямой ссылке.

#### Указание дополнительных условий

Если вы хотите, вы также можете добавить дополнительные условия для запроса аутентификации. Например, мы можем убедиться, что пользователь помечен как "активный":

```php
if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
    // The user is active, not suspended, and exists.
}
```

#### Доступ к определённым экземплярам Guard

C помощью метода `guard` фасада `Auth`, вы можете указать, какой экземпляр Guard вы хотели бы использовать. Это позволяет управлять аутентификацией для отдельных частей приложения с использованием совершенно разных моделей или таблиц аутентификации пользователя.

Имя охранник передаваемое методу `guard` должно соответствовать одному из Guard'ов, настроенных в файле конфигурации `auth.php`

```php
if (Auth::guard('admin')->attempt($credentials)) {
    //
}
```

#### Выход
Для выхода пользователей из вашего приложения, вы можете использовать метод `logout` фасада `Auth`. Это очистит информацию об аутентификации в сессии пользователя:

```
Auth::logout();
```

<a name="remembering-users"></a>
### Запоминание пользователей

Если вы хотите, предоставить в приложении функционал "запомнить меня", вы можете передать булево значение вторым аргументом в метод `attempt`. Это сохранит состояние аутентифицированного пользователя на неопределенный срок, или пока он ну выйдет из системы вручную. Конечно, ваша таблица `users` должна содержать столбец ` remember_token`, который будет использоваться для хранения токена "запомнить меня".

```
if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
    // Пользователь был "запомнен"...
}
```

> Если вы используете встроенный `LoginController`, правильная логика  запоминания пользователей, уже реализована трейтом, используемым контроллером.

> *На самом деле, cookies Laravel хранится один год. Однако, учитывая тот факт, что очень немногие пользователи могут сохранить cookies так долго, выражение "неопределенный срок" - вполне уместно* - п.п.


<a name="other-authentication-methods"></a>
### Другие способы аутентификации

#### Аутентификация объекта пользователя

Если "залогинить" определенный экземпляр пользователя, вы можете вызвать `login` метод с пользовательским экземпляром. Данный объект должен имплементировать `Illuminate\Contracts\Auth\Authenticatable` [контракт](/docs/{{version}}/contracts). Конечно же, модель `App\User` поставляемая с Laravel,  уже реализует этот интерфейс:

```php
Auth::login($user);

// Залогинить и запомнить предоставленного пользователя...
Auth::login($user, true);
```
Само собой, вы можете указать экземпляр Guard, который вы хотели бы использовать:

```
Auth::guard('admin')->login($user);
```

#### Аутентификация пользователя по идентификатору
Для входа пользователя в приложение его идентификатору, вы можете использовать метод `loginUsingId`. Этот метод просто принимает первичный ключ пользователя, который вы хотите аутентифицировать:

```php
Auth::loginUsingId(1);

// Залогинить и запомнить пользователя найденного по идентификатору...
Auth::loginUsingId(1, true);
```

#### Одноразовая аутентификация

Вы можете использовать `once` метод для входа пользователя в приложение для одного запроса. Ни сессия ни cookies не будут использоваться, что означает этот метод может оказаться полезным при создании API без сохранения состояния (stateless):

```php
if (Auth::once($credentials)) {
    //
}
```

<a name="http-basic-authentication"></a>
## Базовая HTTP Аутентификация

[HTTP Basic Authentication] (http://en.wikipedia.org/wiki/Basic_access_authentication) обеспечивает быстрый способ для аутентификации пользователей вашего приложения без создания специальной страницы входа. Чтобы приступить к работе, установите `auth.basic` [посредник](/docs/{{version}}/middleware) для маршрута. Посредник `auth.basic` входит в поставку фреймворка Laravel, так что вам не нужно определять его:
```php
Route::get('profile', function () {
    // только аутентифицированные пользователи могут войти...
})->middleware('auth.basic');
```
После того, посредник был привязан к маршруту, вам автоматически будет предложено ввести учетные данные при доступе к маршруту из браузера. По умолчанию, посредник `auth.basic` использовать поле `email` записи пользователя, как "пользователя".

#### Замечание по FastCGI
Если вы используете PHP FastCGI, Базовая HTTP Аутентификация не может корректно работать из коробки. Следующие строки необходимо добавить в `.htaccess` файл:

```
    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

<a name="stateless-http-basic-authentication"></a>
### Базовая HTTP Аутентификация без сохранения состояния (stateless)
Вы также можете использовать HTTP Basic Authentication без установки идентификатора пользователя cookies в сессии, это особенно полезно для аутентификации API. Для этого, определите [посредник](/docs/{{version}}/middleware), который вызывает `onceBasic` метод. Если  метод `onceBasic` не вернет ответ, запрос может быть передан дальше в приложение:

```php
<?php

namespace Illuminate\Auth\Middleware;

use Illuminate\Support\Facades\Auth;

class AuthenticateOnceWithBasicAuth
{
    /**
        * Handle an incoming request.
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

```
Теперь, [зарегистрируйте посредник маршрута](/docs/{{version}}/middleware#registering-middleware) и привяжите его к маршруту:

```php
Route::get('api/user', function () {
    // Only authenticated users may enter...
})->middleware('auth.basic.once');
```

<a name="adding-custom-guards"></a>
## Добавление пользовательских  Guard'ов
Вы можете определить свои собственные Guard'ы аутентификации с помощью  метода `extend`  фасада ` Auth`. Вы должны поместить его вызов внутри [сервис-провайдера] (/docs/{{version}}/providers). Так как Laravel уже поставляется с `AuthServiceProvider`, мы можем поместить код в этот провайдер:

```php
<?php

    namespace App\Providers;

    use App\Services\Auth\JwtGuard;
    use Illuminate\Support\Facades\Auth;
    use Illuminate\Support\ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::extend('jwt', function ($app, $name, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\Guard...

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }
    }
```
Как вы можете видеть в приведенном выше примере, замыкание передающееся в `extend` метод должно возвращать реализацию `Illuminate\Contracts\Auth\Guard`. Этот интерфейс содержит несколько методов, которые нужно реализовать для определения пользовательского Guard'а. После того, как ваш собственный Guard был определен, вы можете использовать его в `guards` файла конфигурации `auth.php`:

```
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

<a name="adding-custom-user-providers"></a>
## Добавление пользовательских User Provider'ов

Если вы не используете обычные реляционные базы данных для хранения ваших пользователей, то вам нужно расширить Laravel с вашим собственным поставщиком пользователей для аутентификации. Мы будем использовать метод `provider` фасада `Auth`  для определения своего поставщика данных пользователя:

```php

<?php

namespace App\Providers;

use Illuminate\Support\Facades\Auth;
use App\Extensions\RiakUserProvider;
use Illuminate\Support\ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    /**
        * Register any application authentication / authorization services.
        *
        * @return void
        */
    public function boot()
    {
        $this->registerPolicies();

        Auth::provider('riak', function ($app, array $config) {
            // Return an instance of Illuminate\Contracts\Auth\UserProvider...

            return new RiakUserProvider($app->make('riak.connection'));
        });
    }
}
```

После того, как вы зарегистрировали поставщик с помощью метода `provider`, вы можете переключиться на новый поставщик пользователя в файле конфигурации `auth.php`. Во-первых, определите `provider`, который использует свой новый драйвер:

```
'providers' => [
    'users' => [
        'driver' => 'riak',
    ],
],
```
Теперь, вы можете использовать этот поставщик в вашей конфигурации `guards`:

```
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
],
```


<a name="the-user-provider-contract"></a>
### Контракт User Provider

В `Illuminate\Contracts\Auth\UserProvider` ответственны только за получение `Illuminate\Contracts\Auth\Authenticatable` из постоянной любой постоянной системы хранения данных, таких как MySQL, Riak и др. Эти два интерфейса обеспечивают работу механизмов аутентификации Laravel, независимо от того,как хранятся данные пользователя, или какой класс используется для его представления.

Давайте посмотрим на `Illuminate\Contracts\Auth\UserProvider`:

```php
<?php

namespace Illuminate\Contracts\Auth;

interface UserProvider {

    public function retrieveById($identifier);
    public function retrieveByToken($identifier, $token);
    public function updateRememberToken(Authenticatable $user, $token);
    public function retrieveByCredentials(array $credentials);
    public function validateCredentials(Authenticatable $user, array $credentials);

}
```
Функция `retrieveById` обычно получает ключ, представляющий пользователя, например, автоинкремент ID из базы данных MySQL. Реализация `Authenticatable` соответствующая идентификатору должна быть извлечена и возвращена этим методом.


Функция `retrieveByToken` возвращает пользователя по его уникальному идентификатору `$identifier` и "запомнить меня" токену `$token`, хранящемся в поле` remember_token`. Как и в предыдущем методе необходимо вернуть реализацию `Authenticatable`.

Метод  `updateRememberToken` обновляет поле `remember_token` пользователя `$user` новым `$token`. Новый токен может быть либо свежим токном, назначенным при успешном  входе с флагом "запомнить меня", или `null`, когда пользователь выходит из системы.

Метод `retrieveByCredentials` получает массив учетных данных, передаваемых в метод  `Auth::attempt` при попытке войти в приложение. Метод должен затем "запросить" пользователя, соответствующего этим учетным данным, из системы постоянного хранения. Обычно, этот запускает запрос с условием "where"  `$credentials[ 'username']`. После этого, метод затем вернуть реализацию `Authenticatable`. ** Этот метод не должен пытаться делать какие-либо проверки пароля или аутентификации. **

`Метод validateCredentials` должен сопоставить полученного пользователя `$user` с переданными `$credentials` для аутентификации пользователя. Например, этому методу, вероятно, следует использовать `Hash::check` для того чтобы сравнить значение `$ user->getAuthPassword()` со значением `$credentials['password']`. Этот метод должен возвращать `true` или` false`, указывающий на то действителен ли пароль.

<a name="the-authenticatable-contract"></a>
### Контракт Authenticatable

Теперь, когда мы разобрали каждый из методов `UserProvider`'a, давайте посмотрим на  контракт `Authenticatable`. Как вы помните, провайдер должен возвращать реализации этого интерфейса из методов `retrieveByCredentials` и `retrieveById`:

```php
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
```
Этот интерфейс прост. Метод `getAuthIdentifierName` должен возвращать имя поля "первичного ключа" пользователя,а метод `getAuthIdentifier` должен вернуть сам "первичный ключ" пользователя. Опять же, если говорить о MySQL,- это будет автоинкрементный первичный ключ. Метод `getAuthPassword` должен возвращать зашифрованный пароль пользователя. Этот интерфейс позволяет системе аутентификации работать с любым классом пользователей, независимо от того, какую ORM или какую абстракцию системы хранения вы используете. По умолчанию Laravel включает класс `User` находящийся в каталоге `app` , который реализует этот интерфейс, так что вы можете изучить этот класс как пример реализации.

<a name="events"></a>
## Events

Laravel вызывает множество [событий] (/docs/{{version}}/events) во время процесса аутентификации. Вы можете назначить слушателей этих событий в `EventServiceProvider`:

```
  /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],
        
        'Illuminate\Auth\Events\Authenticated' => [
            'App\Listeners\LogAuthenticated',
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
```
