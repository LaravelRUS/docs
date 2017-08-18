git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Фасады

- [Введение](#introduction)
- [Когда использовать фасады](#when-to-use-facades)
    - [Фасады в сравнении с Внедрением зависимостей](#facades-vs-dependency-injection)
    - [Фасады в сравнении с Вспомогательными функциями](#facades-vs-helper-functions)
- [Как работают фасады](#how-facades-work)
- [Справочное описание классов фасадов](#facade-class-reference)

<a name="introduction"></a>
## Введение

Фасады предоставляют "статический" интерфейс к классам, доступным в [сервис-контейнере](/docs/{{version}}/container). Laravel поставляется со множеством фасадов, которые предоставляют доступ практически ко всем функциям Laravel. Фасады Laravel служат "статическими прокси" для основополагающих классов в сервис-контейнере, предоставляя преимущество лаконичного, выразительного синтаксиса, сохраняя при этом большую тестируемость и гибкость по сравнению с обычными статическими методами.

Все фасады Laravel определены в пространстве имен `Illuminate\Support\Facades`. Таким образом, мы можем легко получить доступ к фасаду:

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

В документации Laravel фасады будут использоваться во многих примерах, чтобы продемонстрировать различный функционал фреймфорка.

<a name="when-to-use-facades"></a>
## Когда использовать фасады

У фасадов множество преимуществ. Они обеспечивают лаконичный, запоминающийся синтаксис, который позволяет использовать функции Laravel без необходимости запоминать длинные названия классов, которые нужно вставить или настроить вручную. Кроме того, из-за их уникального использования динамических методов PHP, их легко тестировать.

Однако при использовании фасадов необходимо проявлять определенную осторожность. Основная опасность фасадов - расширение области ответственности класса. Поскольку фасады настолько просты в использовании и не требуют внедрения, легко будет позволить вашим классам продолжать расти и использовать много фасадов в одном классе. При использовании внедрения зависимостей этот потенциал подавляется визуальной обратной связью, которую дает большой конструктор относительно того, что ваш класс становится слишком большим - т.е. однажды вы посмотрите на конструктор с множеством подключённых сторонних классов и подумаете "что-то я тут увлёкся. разобью-ка этот класс на два", а при использовании фасадов накопление сторонних зависимостей в классе проходит гораздо незаметнее. Поэтому при использовании фасадов обращайте особое внимание на размер вашего класса, чтобы его область ответственности оставалась малой, чтобы он делал одну узкоспециализированную задачу.

> {совет} При создании стороннего пакета, который взаимодействует с Laravel, лучше внедрять [контракты Laravel](/docs/{{version}}/contracts) вместо использования фасадов. Так как пакеты создаются вне самого Laravel, у вас не будет доступа к хелперам тестирования Laravel.

<a name="facades-vs-dependency-injection"></a>
### Фасады в сравнении с Внедрением зависимостей

Одним из основных преимуществ внедрения зависимостей является возможность замены реализаций внедряемого класса. Это полезно во время тестирования, так как вы можете внедрить заглушку и утверждать, что на заглушке есть различные методы.

Для настоящих статических методов обычно невозможно сделать заглушку. Однако, поскольку фасады используют динамические методы для вызова метода прокси для объектов, отделенных от сервис-контейнеров, мы фактически можем тестировать фасады так же, как мы бы тестировали экземпляр внедренного класса. Например, учитывая следующий роут:

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

Мы можем написать следующий тест, чтобы проверить, что метод `Cache::get` был вызван с ожидаемым аргументом:

    use Illuminate\Support\Facades\Cache;

    /**
     * Пример базового функционального теста.
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $this->visit('/cache')
             ->see('value');
    }

<a name="facades-vs-helper-functions"></a>
### Фасады в сравнении с Хелперами

Помимо фасадов, Laravel включает множество вспомогательных функций, хелперов, которые могут выполнять общие задачи, такие как формирование шаблонов, запуск событий, постановка задач в очередь или отправка HTTP-ответов. Многие из этих хелперов выполняют ту же функцию, как и соответствующий фасад. Например, этот вызов фасада и этот вызов хелпера эквивалентны:

    return View::make('profile');

    return view('profile');

Нет никакой практической разницы между фасадами и хелперами. При использовании хелперов вы можете тестировать их точно так же, как и соответствующие фасады. Например, учитывая следующий роут:

    Route::get('/cache', function () {
        return cache('key');
    });

С точки зрения внутренней структуры, помощник `cache` собирается вызвать метод `get` в классе, основополагающем для фасада `Cache`. Поэтому, хотя мы используем вспомогательную функцию, мы можем написать следующий тест, чтобы убедиться, что метод был вызван с ожидаемым аргументом:

    use Illuminate\Support\Facades\Cache;

    /**
     * Пример базового функционального теста.
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $this->visit('/cache')
             ->see('value');
    }

<a name="how-facades-work"></a>
## Как работают фасады

В контексте приложения на Laravel, фасад - это класс, который предоставляет доступ к объекту в контейнере. Весь этот механизм реализован в классе `Facade`. Фасады как Laravel, так и ваши собственные, наследуют этот базовый класс `Illuminate\Support\Facades\Facade`.

Класс `Facade` использует магический метод PHP `__callStatic()` для перенаправления вызовов методов с вашего фасада на полученный объект из контейнера. В примере ниже делается обращение к механизму кэширования Laravel. На первый взгляд может показаться, что статический метод `get` принадлежит классу `Cache`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Показать профиль для данного пользователя.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Cache::get('user:'.$id);

            return view('profile', ['user' => $user]);
        }
    }

Обратите внимание, что в верхней части файла мы "импортируем" фасад `Cache`. Этот фасад служит прокси-сервером для доступа к базовой реализации интерфейса `Illuminate\Contracts\Cache\Factory`. Любые вызовы, которые мы осуществляем с использованием фасада, будут переданы в исходный сервис кэша Laravel.

Однако, если вы посмотрите в исходный код класса `Illuminate\Support\Facades\Cache`, то увидите, что он не содержит метода `get`:

    class Cache extends Facade
    {
        /**
         * Получить зарегистрированное имя компонента.
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }
    }

Вместо этого фасад `Cache` расширяется на базе класса `Facade` и определяет метод `getFacadeAccessor()`. Задача этого метода - вернуть строковое имя (ключ) привязки объекта в сервис-контейнере. Когда вы обращаетесь к любому статическому методу фасада `Cache`, Laravel получает объект `cache` из [сервис-контейнера](/docs/{{version}}/container) и вызывает у него требуемый метод (в этом случае - `get`).

<a name="facade-class-reference"></a>
## Справочное описание классов фасадов

Ниже вы найдете каждый фасад и его основной класс. Это полезный инструмент для быстрой обработки документации API для данной корневой директории фасада. Ключ [привязки сервис-контейнера](/docs/{{version}}/container) также включен там, где это применимо.

Фасад  |  Класс  |  Привязка сервис-контейнера
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](https://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)  | `app`
Artisan  |  [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](https://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)  |  `auth`
Blade  |  [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)  |  &nbsp;
Cache  |  [Illuminate\Cache\Repository](https://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)  |  `cache`
Config  |  [Illuminate\Config\Repository](https://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](https://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](https://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](https://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](https://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)  |  &nbsp;
Event  |  [Illuminate\Events\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Gate  |  [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html)  |  &nbsp;
Hash  |  [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Lang  |  [Illuminate\Translation\Translator](https://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](https://laravel.com/api/{{version}}/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](https://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Notification  |  [Illuminate\Notifications\ChannelManager](https://laravel.com/api/{{version}}/Illuminate/Notifications/ChannelManager.html)  |  &nbsp;
Password  |  [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html)  |  `auth.password`
Queue  |  [Illuminate\Queue\QueueManager](https://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance)  |  [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html)  |  `queue`
Queue (Base Class) |  [Illuminate\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)  |  &nbsp;
Redirect  |  [Illuminate\Routing\Redirector](https://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\Database](https://laravel.com/api/{{version}}/Illuminate/Redis/Database.html)  |  `redis`
Request  |  [Illuminate\Http\Request](https://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)  |  &nbsp;
Route  |  [Illuminate\Routing\Router](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Blueprint](https://laravel.com/api/{{version}}/Illuminate/Database/Schema/Blueprint.html)  |  &nbsp;
Session  |  [Illuminate\Session\SessionManager](https://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](https://laravel.com/api/{{version}}/Illuminate/Session/Store.html)  |  &nbsp;
Storage  |  [Illuminate\Contracts\Filesystem\Factory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Factory.html)  |  `filesystem`
URL  |  [Illuminate\Routing\UrlGenerator](https://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](https://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](https://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html) |  &nbsp;
View  |  [Illuminate\View\Factory](https://laravel.com/api/{{version}}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](https://laravel.com/api/{{version}}/Illuminate/View/View.html)  |  &nbsp;

