git e73c40f0dea4db1205c83584d6c5b544b5ff1683

---

# Контракты

- [Введение](#introduction)
    - [Контракты или фасады?](#contracts-vs-facades)
- [Когда использовать контракты](#when-to-use-contracts)
    - [Слабая связность](#loose-coupling)
    - [Упрощение кода](#simplicity)
- [Как использовать контракты](#how-to-use-contracts)
- [Таблица основных контрактов](#contract-reference)

<a name="introduction"></a>
## Введение

Контракты в Laravel — это набор интерфейсов, которые описывают основной функционал, предоставляемый фреймворком. Например, контракт `Illuminate\Contracts\Queue\Queue` определяет методы, необходимые для организации очередей, в то время как контракт `Illuminate\Contracts\Mail\Mailer` определяет методы, необходимые для отправки электронной почты.

Каждый контракт имеет свою реализацию во фреймворке. Например, Laravel предоставляет реализацию `Queue` с различными драйверами и реализацию `Mailer`, использующую [SwiftMailer](https://swiftmailer.symfony.com/).

Все контракты Laravel живут в [своих собственных репозиториях GitHub](https://github.com/illuminate/contracts). Это ссылка на все доступные контракты, а также на один отдельный пакет, который может быть использован разработчиками пакетов.

<a name="contracts-vs-facades"></a>
### Контракты или фасады?

[Фасады](/docs/{{version}}/facades) Laravel и хелперы дают простой способ использования сервисов Laravel без необходимости типизирования и извлечения контрактов из сервис-контейнера. В большинстве случаев у каждого фасада есть эквивалентный контракт.

В отличие от фасадов, которые не требуют того, чтобы вы запрашивали их в конструкторе вашего класса, контракты позволяют вам определить конкретные зависимости для ваших классов. Некоторые разработчики предпочитают именно так явно определять свои зависимости, поэтому предпочитают использовать контракты, а другие разработчики наслаждаются удобством фасадов.

> {tip} Для большинства приложений неважно, что вы выберете — фасады или контракты. Но если вы создаёте пакет, то вам надо использовать контракты, так как в этом случае их проще тестировать.

<a name="when-to-use-contracts"></a>
## Когда использовать контракты

Это обсуждается повсюду, и большинство дискуссий сводятся к тому, что использование контрактов или фасадов — это дело вкуса или предпочтений вашей команды разработчиков. И те, и другие можно использовать для создания надёжных, проверенных Laravel-приложений. Пока вы сохраняете границы ответственности вашего класса узкими, вы сможете заметить всего несколько практических различий между использованием контрактов и фасадов.

Однако, у вас по-прежнему могут остаться некоторые вопросы о контрактах. Например, зачем вообще нужны интерфейсы? Разве использовать их не слишком сложно? Определим причины использования интерфейсов как следующие: это слабая связность и упрощение кода.

<a name="loose-coupling"></a>
### Слабая связность

Для начала рассмотрим код с сильной связностью с реализацией кэша:

    <?php

    namespace App\Orders;

    class Repository
    {
        /**
         * Экземпляр кэша.
         */
        protected $cache;

        /**
         * Создание нового экземпляра репозитория.
         *
         * @param  \SomePackage\Cache\Memcached  $cache
         * @return void
         */
        public function __construct(\SomePackage\Cache\Memcached $cache)
        {
            $this->cache = $cache;
        }

        /**
         * Получение заказа Order по ID.
         *
         * @param  int  $id
         * @return Order
         */
        public function find($id)
        {
            if ($this->cache->has($id))    {
                //
            }
        }
    }

В этом классе код сильно связан с реализацией кэша, потому что мы зависим от конкретного класса Cache данного пакета. Если API этого пакета изменится, наш код должен также измениться.

Аналогично, если мы хотим заменить нашу базовую технологию кэша (Memcached) другой технологией (Redis), нам придётся вносить изменения в наш репозиторий. А наш репозиторий не должен задумываться о том, кто именно предоставляет данные или как он это делает.

**Вместо такого подхода, мы можем улучшить наш код, добавив зависимость от простого интерфейса, который не зависит от поставщика:**

    <?php

    namespace App\Orders;

    use Illuminate\Contracts\Cache\Repository as Cache;

    class Repository
    {
        /**
         * Экземпляр кэша.
         */
        protected $cache;

        /**
         * Создание нового экземпляра репозитория.
         *
         * @param  Cache  $cache
         * @return void
         */
        public function __construct(Cache $cache)
        {
            $this->cache = $cache;
        }
    }

Теперь код не привязан к какому-либо определённому поставщику, и даже не привязан к Laravel. Контракт не содержит никакой конкретной реализации и никаких зависимостей. Вы можете легко написать свою реализацию любого контракта, что позволяет вам заменить реализацию работы с кэшем, не изменяя ни одной строчки вашего кода, работающего с кэшем.

<a name="simplicity"></a>
### Упрощение кода

Когда все сервисы ядра фреймворка аккуратно определены в простых интерфейсах, очень легко определить, что именно делает тот или иной сервис. **Фактически, контракты являются краткой документацией функций Laravel.**

Кроме того, когда в своём приложении вы внедряете в классы зависимости от простых интерфейсов, в вашем коде легче разобраться и его проще поддерживать. Вместо того, чтобы искать методы в большом и сложном классе, вы можете обратиться к простому и понятному интерфейсу.

<a name="how-to-use-contracts"></a>
## Как использовать контракты

Как получить реализацию контракта? Это довольно просто.

Множество типов классов в Laravel регистрируются в [сервис-контейнере](/docs/{{version}}/container), включая контроллеры, слушатели событий, посредники, очереди и даже замыкания. Поэтому, чтобы получить реализацию контракта, вам достаточно указать тип интерфейса в конструкторе необходимого класса.

Например, посмотрите на этот обработчик событий:

    <?php

    namespace App\Listeners;

    use App\User;
    use App\Events\OrderWasPlaced;
    use Illuminate\Contracts\Redis\Database;

    class CacheOrderInformation
    {
        /**
         * Реализация базы данных Redis.
         */
        protected $redis;

        /**
         * Создание нового экземпляра обработчика событий.
         *
         * @param  Database  $redis
         * @return void
         */
        public function __construct(Database $redis)
        {
            $this->redis = $redis;
        }

        /**
         * Обработка события.
         *
         * @param  OrderWasPlaced  $event
         * @return void
         */
        public function handle(OrderWasPlaced $event)
        {
            //
        }
    }

Когда будет получен слушатель события, сервис-контейнер прочитает указание типа в конструкторе класса и внедрит нужное значение. Узнать больше о регистрации в сервис-контейнере можно в [его документации](/docs/{{version}}/container).

<a name="contract-reference"></a>
## Таблица основных контрактов

В этой таблице приведены ссылки на все контракты Laravel, а также эквивалентные им фасады:

Контракт  |  Соответствующий фасад
------------- | -------------
[Illuminate\Contracts\Auth\Access\Authorizable](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Access/Authorizable.php) | &nbsp;
[Illuminate\Contracts\Auth\Access\Gate](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Access/Gate.php) | `Gate`
[Illuminate\Contracts\Auth\Authenticatable](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Authenticatable.php) | &nbsp;
[Illuminate\Contracts\Auth\CanResetPassword](https://github.com/illuminate/contracts/blob/{{version}}/Auth/CanResetPassword.php) | &nbsp;
[Illuminate\Contracts\Auth\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Factory.php) | `Auth`
[Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Guard.php) | `Auth::guard()`
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/{{version}}/Auth/PasswordBroker.php) | `Password::broker()`
[Illuminate\Contracts\Auth\PasswordBrokerFactory](https://github.com/illuminate/contracts/blob/{{version}}/Auth/PasswordBrokerFactory.php) | `Password`
[Illuminate\Contracts\Auth\StatefulGuard](https://github.com/illuminate/contracts/blob/{{version}}/Auth/StatefulGuard.php) | &nbsp;
[Illuminate\Contracts\Auth\SupportsBasicAuth](https://github.com/illuminate/contracts/blob/{{version}}/Auth/SupportsBasicAuth.php) | &nbsp;
[Illuminate\Contracts\Auth\UserProvider](https://github.com/illuminate/contracts/blob/{{version}}/Auth/UserProvider.php) | &nbsp;
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Bus/Dispatcher.php) | `Bus`
[Illuminate\Contracts\Bus\QueueingDispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Bus/QueueingDispatcher.php) | `Bus::dispatchToQueue()`
[Illuminate\Contracts\Broadcasting\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/Factory.php) | `Broadcast`
[Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/Broadcaster.php)  | `Broadcast::connection()`
[Illuminate\Contracts\Broadcasting\ShouldBroadcast](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/ShouldBroadcast.php) | &nbsp;
[Illuminate\Contracts\Broadcasting\ShouldBroadcastNow](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/ShouldBroadcastNow.php) | &nbsp;
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Factory.php) | `Cache`
[Illuminate\Contracts\Cache\Lock](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Lock.php) | &nbsp;
[Illuminate\Contracts\Cache\LockProvider](https://github.com/illuminate/contracts/blob/{{version}}/Cache/LockProvider.php) | &nbsp;
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Repository.php) | `Cache::driver()`
[Illuminate\Contracts\Cache\Store](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Store.php) | &nbsp;
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/{{version}}/Config/Repository.php) | `Config`
[Illuminate\Contracts\Console\Application](https://github.com/illuminate/contracts/blob/{{version}}/Console/Application.php) | &nbsp;
[Illuminate\Contracts\Console\Kernel](https://github.com/illuminate/contracts/blob/{{version}}/Console/Kernel.php) | `Artisan`
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/{{version}}/Container/Container.php) | `App`
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/Factory.php) | `Cookie`
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/QueueingFactory.php) | `Cookie::queue()`
[Illuminate\Contracts\Database\ModelIdentifier](https://github.com/illuminate/contracts/blob/{{version}}/Database/ModelIdentifier.php) | &nbsp;
[Illuminate\Contracts\Debug\ExceptionHandler](https://github.com/illuminate/contracts/blob/{{version}}/Debug/ExceptionHandler.php) | &nbsp;
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/{{version}}/Encryption/Encrypter.php) | `Crypt`
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Events/Dispatcher.php) | `Event`
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Cloud.php) | `Storage::cloud()`
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Factory.php) | `Storage`
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Filesystem.php) | `Storage::disk()`
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/{{version}}/Foundation/Application.php) | `App`
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/{{version}}/Hashing/Hasher.php) | `Hash`
[Illuminate\Contracts\Http\Kernel](https://github.com/illuminate/contracts/blob/{{version}}/Http/Kernel.php) | &nbsp;
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/{{version}}/Logging/Log.php) | `Log`
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/{{version}}/Mail/MailQueue.php) | `Mail::queue()`
[Illuminate\Contracts\Mail\Mailable](https://github.com/illuminate/contracts/blob/{{version}}/Mail/Mailable.php) | &nbsp;
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/{{version}}/Mail/Mailer.php) | `Mail`
[Illuminate\Contracts\Notifications\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Notifications/Dispatcher.php) | `Notification`
[Illuminate\Contracts\Notifications\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Notifications/Factory.php) | `Notification`
[Illuminate\Contracts\Pagination\LengthAwarePaginator](https://github.com/illuminate/contracts/blob/{{version}}/Pagination/LengthAwarePaginator.php) | &nbsp;
[Illuminate\Contracts\Pagination\Paginator](https://github.com/illuminate/contracts/blob/{{version}}/Pagination/Paginator.php) | &nbsp;
[Illuminate\Contracts\Pipeline\Hub](https://github.com/illuminate/contracts/blob/{{version}}/Pipeline/Hub.php) | &nbsp;
[Illuminate\Contracts\Pipeline\Pipeline](https://github.com/illuminate/contracts/blob/{{version}}/Pipeline/Pipeline.php) | &nbsp;
[Illuminate\Contracts\Queue\EntityResolver](https://github.com/illuminate/contracts/blob/{{version}}/Queue/EntityResolver.php) | &nbsp;
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Factory.php) | `Queue`
[Illuminate\Contracts\Queue\Job](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Job.php) | &nbsp;
[Illuminate\Contracts\Queue\Monitor](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Monitor.php) | `Queue`
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Queue.php) | `Queue::connection()`
[Illuminate\Contracts\Queue\QueueableCollection](https://github.com/illuminate/contracts/blob/{{version}}/Queue/QueueableCollection.php) | &nbsp;
[Illuminate\Contracts\Queue\QueueableEntity](https://github.com/illuminate/contracts/blob/{{version}}/Queue/QueueableEntity.php) | &nbsp;
[Illuminate\Contracts\Queue\ShouldQueue](https://github.com/illuminate/contracts/blob/{{version}}/Queue/ShouldQueue.php) | &nbsp;
[Illuminate\Contracts\Redis\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Redis/Factory.php) | `Redis`
[Illuminate\Contracts\Routing\BindingRegistrar](https://github.com/illuminate/contracts/blob/{{version}}/Routing/BindingRegistrar.php) | `Route`
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/{{version}}/Routing/Registrar.php) | `Route`
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/{{version}}/Routing/ResponseFactory.php) | `Response`
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/{{version}}/Routing/UrlGenerator.php) | `URL`
[Illuminate\Contracts\Routing\UrlRoutable](https://github.com/illuminate/contracts/blob/{{version}}/Routing/UrlRoutable.php) | &nbsp;
[Illuminate\Contracts\Session\Session](https://github.com/illuminate/contracts/blob/{{version}}/Session/Session.php) | `Session::driver()`
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Htmlable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Htmlable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\MessageBag](https://github.com/illuminate/contracts/blob/{{version}}/Support/MessageBag.php) | &nbsp;
[Illuminate\Contracts\Support\MessageProvider](https://github.com/illuminate/contracts/blob/{{version}}/Support/MessageProvider.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Support\Responsable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Responsable.php) | &nbsp;
[Illuminate\Contracts\Translation\Loader](https://github.com/illuminate/contracts/blob/{{version}}/Translation/Loader.php) | &nbsp;
[Illuminate\Contracts\Translation\Translator](https://github.com/illuminate/contracts/blob/{{version}}/Translation/Translator.php) | `Lang`
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Factory.php) | `Validator`
[Illuminate\Contracts\Validation\ImplicitRule](https://github.com/illuminate/contracts/blob/{{version}}/Validation/ImplicitRule.php) | &nbsp;
[Illuminate\Contracts\Validation\Rule](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Rule.php) | &nbsp;
[Illuminate\Contracts\Validation\ValidatesWhenResolved](https://github.com/illuminate/contracts/blob/{{version}}/Validation/ValidatesWhenResolved.php) | &nbsp;
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Validator.php) | `Validator::make()`
[Illuminate\Contracts\View\Engine](https://github.com/illuminate/contracts/blob/{{version}}/View/Engine.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/{{version}}/View/Factory.php) | `View`
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/{{version}}/View/View.php) | `View::make()`
