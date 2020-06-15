git b01a1c4ccfcd940447ea25457eff41e056628e88

---

# Сервис-провайдеры

- [Введение](#introduction)
- [Использование сервис-провайдеров](#writing-service-providers)
    - [Метод Register](#the-register-method)
    - [Метод Boot](#the-boot-method)
- [Регистрация провайдеров](#registering-providers)
- [Отложенные провайдеры](#deferred-providers)

<a name="introduction"></a>
## Введение

Сервис-провайдеры лежат в основе первоначальной загрузки всех приложений на Laravel. И ваше приложение, и все базовые сервисы Laravel загружаются через сервис-провайдеры.

Но что мы понимаем под "первоначальной загрузкой"? В общих чертах, мы имеем ввиду **регистрацию** таких вещей, как биндингов в IoC-контейнер (фасадов и т.д.), слушателей событий, фильтров роутов и даже самих роутов. Сервис-провайдеры - центральное место для конфигурирования вашего приложения.

Если вы откроете файл `config/app.php`, поставляемый с Laravel, то увидите массив `providers`. В нём перечислены все классы сервис-провайдеров, которые загружаются для вашего приложения. Обратите внимание, что многие из них являются "отложенными" провайдерами, т.е. они не загружаются при каждом запросе, а только при необходимости.

В этом обзоре вы узнаете, как создавать свои собственные сервис-провайдеры и регистрировать их в своём приложении.

<a name="writing-service-providers"></a>
## Использование сервис-провайдеров

Все сервис-провайдеры наследуют класс `Illuminate\Support\ServiceProvider`. В большинстве сервис-провайдеров есть методы `register` и `boot`. В методе `register` вы должны **только привязывать свои классы в [сервис-контейнер](/docs/{{version}}/container)**. Никогда не пытайтесь зарегистрировать слушателей событий, роуты и какие-либо другие возможности в методе`register`.

С помощью Artisan CLI можно создать новый провайдер командой `make:provider`:

    php artisan make:provider RiakServiceProvider

<a name="the-register-method"></a>
### Метод Register

Как уже было сказано, внутри метода `register`, вы должны только привязывать свои классы в [сервис-контейнер](/docs/{{version}}/container). Никогда не пытайтесь зарегистрировать слушателей событий, роуты и какие-либо другие возможности в методе `register`. Иначе вы можете случайно обратиться к сервису, предоставляемому сервис-провайдером, который ещё не был загружен.

Давайте взглянем на простой сервис-провайдер. Из любого метода вашего сервис-провайдера у вас всегда есть доступ к свойству `$app`, которое предоставляет доступ к сервис-контейнеру:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Riak\Connection;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
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

Этот сервис-провайдер только определяет метод `register` и использует его, чтобы определить реализацию `Riak\Connection` в сервис-контейнере. Если вы не понимаете как работает сервис-контейнер, прочитайте [его документацию](/docs/{{version}}/container).

#### Свойства `bindings` и `singletons`

Если ваш сервис провайдер регистрирует много простых связываний, вы можете использовать свойства `bindings` и `singletons` вместо того, чтобы вручную регистрировать каждое связывание. Когда сервис провайдер загружается фреймворком, он автоматически проверяет эти свойства и создает связывания:

    <?php

    namespace App\Providers;

    use App\Contracts\DowntimeNotifier;
    use App\Contracts\ServerProvider;
    use App\Services\DigitalOceanServerProvider;
    use App\Services\PingdomDowntimeNotifier;
    use App\Services\ServerToolsProvider;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * All of the container bindings that should be registered.
         *
         * @var array
         */
        public $bindings = [
            ServerProvider::class => DigitalOceanServerProvider::class,
        ];

        /**
         * All of the container singletons that should be registered.
         *
         * @var array
         */
        public $singletons = [
            DowntimeNotifier::class => PingdomDowntimeNotifier::class,
            ServerToolsProvider::class => ServerToolsProvider::class,
        ];
    }

<a name="the-boot-method"></a>
### Метод Boot

А что, если нам нужно зарегистрировать [вью-композер](/docs/{{version}}/views#view-composers) в нашем сервис-провайдере? Это нужно делать в методе `boot`. **Этот метод вызывают после того, как были зарегистрированы все другие сервис-провайдеры**. И это значит, что у вас есть доступ ко всем другим сервисам, которые были зарегистрированы фреймворком:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Загрузка любых сервисов приложения..
         *
         * @return void
         */
        public function boot()
        {
            view()->composer('view', function () {
                //
            });
        }
    }

#### Внедрение зависимостей метода Boot

Вы можете указать зависимости для метода `boot` вашего сервис-провайдера. [Сервис-контейнер](/docs/{{version}}/container) автоматически внедрит те зависимости, которые вы зададите:

    use Illuminate\Contracts\Routing\ResponseFactory;

    public function boot(ResponseFactory $response)
    {
        $response->macro('caps', function ($value) {
            //
        });
    }

<a name="registering-providers"></a>
## Регистрация провайдеров

Все сервис-провайдеры регистрируются в конфиге `config/app.php`. В этом файле содержится массив `providers`, где можно добавить имена классов ваших сервис-провайдеров. По-умолчанию в нём указан набор базовых сервис-провайдеров Laravel. Эти провайдеры загружают базовые компоненты Laravel, такие как обработчик почты, очередь, кэш и другие.

Чтобы зарегистрировать свой сервис-провайдер, добавьте его в этот массив:

    'providers' => [
        // Другие сервис-провайдеры

        App\Providers\ComposerServiceProvider::class,
    ],

<a name="deferred-providers"></a>
## Отложенные провайдеры

Если ваш провайдер **только** регистрирует привязки в [сервис-контейнере](/docs/{{version}}/container), то можно отложить регистрацию до момента, когда одна из этих привязок будет запрошена из сервис-контейнера. Это позволит не тревожить файловую систему при каждом запросе, что увеличит производительность вашего приложения.

Laravel компилирует и хранит список всех сервисов, предоставляемых отложенными сервис-провайдерами, и их классов. Laravel загрузит нужный сервис-провайдер только когда в процессе работы приложению понадобится один из этих сервисов.

Чтобы отложить загрузку сервис-провайдера, реализуем интерфейс `\Illuminate\Contracts\Support\DeferrableProvider` и определим метод `provides`. Метод `provides` должен возвращать связи сервис-контейнера, регистрируемые в провайдере:

    <?php

    namespace App\Providers;

    use Illuminate\Contracts\Support\DeferrableProvider;
    use Illuminate\Support\ServiceProvider;
    use Riak\Connection;

    class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
    {
        /**
         * Register any application services.
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