git 452b5680b3a434f6736003dd3e7011d270c08fe0

---

# Сервис-провайдеры

- [Введение](#introduction)
- [Написание сервис-провайдеров](#writing-service-providers)
    - [Метод `register`](#the-register-method)
    - [Метод `boot`](#the-boot-method)
- [Регистрация сервис-провайдеров](#registering-providers)
- [Отложенные сервис-провайдеры](#deferred-providers)

<a name="introduction"></a>
## Введение

Сервис-провайдеры – это центральное место начальной загрузки всех приложений Laravel. Ваше собственное приложение, а также все основные службы и сервисы Laravel загружаются через них.

Но, что мы подразумеваем под «начальной загрузкой»? В общем, мы имеем в виду **регистрацию** элементов, включая регистрацию связываний контейнера служб (service container), слушателей событий (event listener), посредников (middleware) и даже маршрутов (route). Сервис-провайдеры являются центральным местом для конфигурирования приложения.

Если вы откроете файл `config/app.php`, включенный в Laravel, вы увидите массив `'providers'`. Это все классы сервис-провайдеров, которые будут загружены вашим приложением. По умолчанию в этом массиве перечислены основные сервис-провайдеры Laravel. Они загружают основные компоненты Laravel, такие, как подсистема отправки почты, очередь, кеш и другие. Многие из этих провайдеров являются «отложенными», что означает, что они не будут загружаться при каждом запросе, а только тогда, когда предоставляемые ими службы действительно необходимы.

В этой документации вы узнаете, как писать собственные сервис-провайдеры и регистрировать их в приложении Laravel.

> {tip} Если вы хотите узнать больше о том, как Laravel обрабатывает запросы и работает изнутри, ознакомьтесь с нашей документацией по [жизненному циклу запроса](/docs/{{version}}/lifecycle) Laravel.

<a name="writing-service-providers"></a>
## Написание сервис-провайдеров

Все сервис-провайдеры расширяют класс `Illuminate\Support\ServiceProvider`. Большинство сервис-провайдеров содержат метод `register` и `boot`. В рамках метода `register` следует **только связывать (bind) сущности в [контейнере служб](/docs/{{version}}/container)**. Никогда не следует пытаться зарегистрировать каких-либо слушателей событий, маршруты или что-то другое в методе `register`.

Чтобы сгенерировать новый сервис-провайдер, используйте команду `make:provider` [Artisan](artisan):

    php artisan make:provider RiakServiceProvider

<a name="the-register-method"></a>
### Метод `register`

Как упоминалось ранее, в рамках метода `register` следует только связывать сущности в [контейнере служб](/docs/{{version}}/container). Никогда не следует пытаться зарегистрировать слушателей событий, маршруты или что-то другое в методе `register`. В противном случае вы можете случайно воспользоваться подсистемой, чей сервис-провайдер еще не загружен.

Давайте взглянем на рядовой сервис-провайдер приложения. В любом из методов сервис-провайдера у вас всегда есть доступ к свойству `$app`, которое обеспечивает доступ к контейнеру служб:

    <?php

    namespace App\Providers;

    use App\Services\Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Регистрация любых служб приложения.
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

Этот сервис-провайдер определяет только метод `register` и использует этот метод для указания, какая именно реализация `App\Services\Riak\Connection` будет применена в нашем приложении - при помощи контейнера служб. Если вы еще не знакомы с контейнером служб Laravel, ознакомьтесь с [его документацией](/docs/{{version}}/container).

<a name="the-bindings-and-singletons-properties"></a>
#### Свойства `bindings` и `singletons`

Если ваш сервис-провайдер регистрирует много простых связываний, вы можете использовать свойства `bindings` и `singletons` вместо ручной регистрации каждого связывания контейнера. Когда сервис-провайдер загружается фреймворком, он автоматически проверяет эти свойства и регистрирует их связывания:

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
         * Все связывания контейнера, которые должны быть зарегистрированы.
         *
         * @var array
         */
        public $bindings = [
            ServerProvider::class => DigitalOceanServerProvider::class,
        ];

        /**
         * Все синглтоны контейнера, которые должны быть зарегистрированы.
         *
         * @var array
         */
        public $singletons = [
            DowntimeNotifier::class => PingdomDowntimeNotifier::class,
            ServerProvider::class => ServerToolsProvider::class,
        ];
    }

<a name="the-boot-method"></a>
### Метод `boot`

Итак, что, если нам нужно зарегистрировать [компоновщик шаблонов](/docs/{{version}}/views#view-composers) в нашем сервис-провайдере? Это должно быть сделано в рамках метода `boot`. **Этот метод вызывается после регистрации всех остальных сервис-провайдеров**, что означает, что в этом месте у вас уже есть доступ ко всем другим службам, которые были зарегистрированы фреймворком:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Загрузка любых служб приложения.
         *
         * @return void
         */
        public function boot()
        {
            View::composer('view', function () {
                //
            });
        }
    }

<a name="boot-method-dependency-injection"></a>
#### Внедрение зависимости в методе `boot`

Вы можете указывать тип зависимостей в методе `boot` сервис-провайдера. [Контейнер служб](/docs/{{version}}/container) автоматически внедрит любые необходимые зависимости:

    use Illuminate\Contracts\Routing\ResponseFactory;

    /**
     * Загрузка любых служб приложения.
     *
     * @param  \Illuminate\Contracts\Routing\ResponseFactory  $response
     * @return void
     */
    public function boot(ResponseFactory $response)
    {
        $response->macro('serialized', function ($value) {
            //
        });
    }

<a name="registering-providers"></a>
## Регистрация сервис-провайдеров

Все сервис-провайдеры регистрируются в файле конфигурации `config/app.php`. Этот файл содержит массив `providers`, в котором можно перечислить имена классов. По умолчанию в этом массиве перечислены основные сервис-провайдеры Laravel. Эти поставщики загружают основные компоненты Laravel, такие, как почтовая подсистема, очереди, кеш и другие.

Чтобы зарегистрировать сервис-провайдер, добавьте его в массив:

    'providers' => [
        // Другие сервис-провайдеры

        App\Providers\ComposerServiceProvider::class,
    ],

<a name="deferred-providers"></a>
## Отложенные сервис-провайдеры

Если ваш сервис-провайдер регистрирует **только** связывания в [контейнере служб](/docs/{{version}}/container), вы можете отложить его регистрацию до тех пор, пока одно из зарегистрированных связываний не понадобится. Отсрочка загрузки такого сервис-провайдера повысит производительность вашего приложения, так как он не загружается из файловой системы при каждом запросе.

Laravel составляет и сохраняет список всех служб, предоставляемых отложенными сервис-провайдерами, а также имя класса сервис-провайдера. Laravel загрузит сервис-провайдер только при необходимости в одной из этих служб.

Чтобы отложить загрузку сервис-провайдера, реализуйте интерфейс `\Illuminate\Contracts\Support\DeferrableProvider`, описав метод `provides`. Метод `provides` должен вернуть связывания контейнера службы, регистрируемые данным классом:

    <?php

    namespace App\Providers;

    use App\Services\Riak\Connection;
    use Illuminate\Contracts\Support\DeferrableProvider;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
    {
        /**
         * Регистрация любых служб приложения.
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
         * Получить службы, предоставляемые поставщиком.
         *
         * @return array
         */
        public function provides()
        {
            return [Connection::class];
        }
    }
