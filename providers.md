git 8d41537f3863b477f86ffe17a06410c1f05ff78b

---

# Сервис Провайдеры (Service Providers)

- [Введение (Introduction)](#introduction)
- [Написание Сервис Провайдеров (Writing Service Providers)](#writing-service-providers)
    - [Метод Register (The Register Method)](#the-register-method)
    - [Метод Boot (The Boot Method)](#the-boot-method)
- [Регистрация Провайдеров (Registering Providers)](#registering-providers)
- [Отложенные Провайдеры (Deferred Providers)](#deferred-providers)

<a name="introduction"></a>
## Введение (Introduction)

Сервис провайдеры занимают центральное место в загрузке Laravel приложения. Ваше собственное приложение, также как и службы ядра Laravel загружаются посредством сервис провайдеров.

Но, что именно, означает начальная загрузка ("bootstrapped")? По большей части, мы подразумеваем под этим регистрацию (**registering**) таких вещей, как регистрация привязок сервис контейнера (service container bindings), слушателей событий (event listeners), посредников (middleware) и даже маршрутов. Сервис провайдеры центральное место для конфигурации вашего приложения.

Перечень подключаемых сервис-провайдеров определен в массиве `providers` в файле `config/app.php`. Многие из них будут загружены только тогда, когда они действительно потребуются - это называется отложенной (deferred) загрузкой.

В этом обзоре мы изучим, как написать собственные сервис провайдеры и зарегистрировать их в приложении.

<a name="writing-service-providers"></a>
## Написание Сервис Провайдеров (Writing Service Providers)

Сервис провайдеры наследуют класс `Illuminate\Support\ServiceProvider`. Этот абстрактный класс требует определения хотя бы одного метода в провайдере: `register`. В этом методе вы должны **только сделать привязку к [сервис контейнеру](/docs/{{version}}/container)**. Вы не должны пытаться регистрировать в нем слушателей событий, маршруты и иную функциональность.

Генерация нового провайдера Artisan командой:

    php artisan make:provider RiakServiceProvider

<a name="the-register-method"></a>
### Метод Register (The Register Method)

Как сказано выше вы не должны использовать этот метод для регистрации чего то иного, кроме как привязки к сервис контейнеру. Иначе вы можете получить ситуацию, когда требуемый сервис еще еще не был загружен. Пример простого сервис провайдера:

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection(config('riak'));
            });
        }
    }

Приведенный пример провайдера содержит метод `register`, который определяет реализацию `Riak\Connection`. Для понимания как работает сервис контейнер обратитесь к [документации](/docs/{{version}}/container).

<a name="the-boot-method"></a>
### Метод Boot (The Boot Method)

Для регистрации view composer в сервис провайдере вы должны использовать метод `boot`. **Этот метод вызывается после регистрации всех сервис провайдеров**, а значит, мы будем иметь доступ ко всем сервисам, зарегистрированным во фреймворке:

    <?php

    namespace App\Providers;

    use Illuminate\Contracts\Events\Dispatcher as DispatcherContract;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        // Other Service Provider Properties...

        /**
         * Register any other events for your application.
         *
         * @param  \Illuminate\Contracts\Events\Dispatcher  $events
         * @return void
         */
        public function boot(DispatcherContract $events)
        {
            parent::boot($events);

            view()->composer('view', function () {
                //
            });
        }
    }

#### Метод Boot с Внедрением Зависимости (Boot Method Dependency Injection)

Вы можете определить зависимости с контролем типов для метода `boot`. [Сервис контейнер] (/docs/{{version}}/container) автоматически ее подключит:

    use Illuminate\Contracts\Routing\ResponseFactory;

    public function boot(ResponseFactory $factory)
    {
        $factory->macro('caps', function ($value) {
            //
        });
    }

<a name="registering-providers"></a>
## Регистрация Провайдеров (Registering Providers)

Сервис провайдеры регистрируются в файле `config/app.php` в массиве `providers`. По умолчанию в нем определены провайдеры для загрузки компонентов ядра, такие как mailer, queue, cache и другие.

Для регистрации провайдера просто добавьте его в массив:

    'providers' => [
        // Other Service Providers

        App\Providers\AppServiceProvider::class,
    ],

<a name="deferred-providers"></a>
## Отложенные Провайдеры (Deferred Providers)

Если провайдер **только** регистрирует привязки к [сервис-контейнеру](/docs/{{version}}/container), то вы можете отложить его регистрацию до момента, когда они потребуются. Откладывание загрузки подобных провайдеров улучшит быстродействие, поскольку они не будут загружаться с диска на каждый запрос.

Для откладывания загрузки задайте свойству `defer` значение `true` и определите метод `provides`, который будет возвращать привязки к сервис контейнеру, регистрируемые провайдером:

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Indicates if loading of the provider is deferred.
         *
         * @var bool
         */
        protected $defer = true;

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection($app['config']['riak']);
            });
        }

        /**
         * Get the services provided by the provider.
         *
         * @return array
         */
        public function provides()
        {
            return [Connection::class];
        }

    }

Laravel компилирует и сохраняет список сервисов с отложенной загрузкой вместе с именем класса сервис провайдера. Только когда будет попытка разрешить такой сервис, Laravel его загрузит.
