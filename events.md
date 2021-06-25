git 2f9a4c5873fc6e3108db41e7281804463584abb1

---

# События

- [Введение](#introduction)
- [Регистрация событий и слушателей](#registering-events-and-listeners)
    - [Генерация классов событий и слушателей](#generating-events-and-listeners)
    - [Регистрация событий вручную](#manually-registering-events)
- [Определение событий](#defining-events)
- [Определение слушателей](#defining-listeners)
- [Слушатели события в очереди](#queued-event-listeners)
    - [Доступ к очереди вручную](#manually-accessing-the-queue)
    - [Обработка проваленных задач](#handling-failed-jobs)
- [Размещение событий](#dispatching-events)
- [Подписчики событий](#event-subscribers)
    - [Записывание подписчиков событий](#writing-event-subscribers)
    - [Регистрация подписчиков событий](#registering-event-subscribers)

<a name="introduction"></a>
## Введение

События в Laravel представлены реализацией паттерна Observer, что позволяет вам подписываться и прослушивать события вашего приложения. Как правило, классы событий находятся в директории `app/Events`, а классы обработчиков событий — в `app/Listeners`. Не волнуйтесь, если не видите эти директории в своем приложении, так как они будут созданы для вас по мере того, как вы сгенерируете события и слушателей, используя команды консоли Artisan.

События служат отличным способом отделить различные аспекты вашего приложения, поскольку одно событие может иметь несколько слушателей, которые не зависят друг от друга. Например, вы можете отправлять Slack-уведомление своему пользователю каждый раз, когда отправляется заказ. Вместо соединения своего кода обработки заказов со своим кодом Slack-уведомлений, вы можете просто поднять событие `OrderShipped`, которое может получить слушатель и трансформировать в Slack-уведомление.

<a name="registering-events-and-listeners"></a>
## Регистрация событий и слушателей

Сервис-провайдер `EventServiceProvider` , включённый в ваше Laravel приложение, предоставляет удобное место для регистрации всех слушателей событий. Свойство `listen` содержит массив всех событий (ключей) и их слушателей (значения). Конечно, вы можете добавить столько событий в этот массив, сколько требуется вашему приложению. Например, давайте добавим событие `OrderShipped`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'App\Events\OrderShipped' => [
            'App\Listeners\SendShipmentNotification',
        ],
    ];

<a name="generating-events-and-listeners"></a>
### Генерация событий и слушателей

Конечно, вручную создавать файлы для каждого события и слушателя затруднительно. Вместо этого добавьте слушателей и события в ваш `EventServiceProvider` и используйте команду `event:generate`. Эта команда сгенерирует все события и слушателей, которые перечислены в вашем `EventServiceProvider`. Конечно, уже существующие события и слушатели останутся нетронутыми:

    php artisan event:generate

<a name="manually-registering-events"></a>
### Регистрация событий вручную

Как правило, события должны регистрироваться через массив `$listen` в `EventServiceProvider`; Однако, также вы можете регистрировать события вручную с обработчиком событий, используя метод `boot` вашего `EventServiceProvider`:

    /**
     * Регистрация своих событий в приложении.
     *
     * @return void
     */
    public function boot()
    {
        parent::boot();

        Event::listen('event.name', function ($foo, $bar) {
            //
        });
    }

#### Слушатели событий по маске

Вы даже можете регистрировать слушателей, используя символ `*`, что позволит вам поймать несколько событий для одного слушателя. Такой метод вернёт весь массив данных событий одним параметром:

    Event::listen('event.*', function ($eventName, array $data) {
        //
    });

<a name="defining-events"></a>
## Определение событий

Класс события — это просто контейнер данных, содержащий информацию, которая относится к событию. Например, предположим, что наше сгенерированное событие `OrderShipped` принимает объект [Eloquent ORM](/docs/{{version}}/eloquent):

    <?php

    namespace App\Events;

    use App\Order;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped
    {
        use SerializesModels;

        public $order;

        /**
         * Создать новый экземпляр события.
         *
         * @param  Order  $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }
    }

Как видите, этот класс события не содержит логики. Это просто контейнер для объекта `Order`, который был приобретен. Трейт `SerializesModels`, используемый событием, корректно сериализирует любые Eloquent модели, если объект события будет сериализирован php-функцией.

<a name="defining-listeners"></a>
## Определение слушателей

Теперь давайте взглянем на слушателя для нашего примера события. Слушатели событий принимают экземпляр события в свой метод `handle`. Команда `event:generate` автоматически импортирует класс события и указывает тип события в метод `handle`. В методе `handle` вы можете выполнять любую логику, необходимую для ответа на событие:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;

    class SendShipmentNotification
    {
        /**
         * Создать слушатель события.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Обработка события.
         *
         * @param  OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            // Access the order using $event->order...
        }
    }

> {tip} Ваши слушатели события могут также указывать тип любых зависимостей, которые необходимы для их конструкторов. Все слушатели события доступны через [сервис-контейнер](/docs/{{version}}/container) Laravel, поэтому зависимости будут внедрены автоматически.

#### Остановка распространения события

Иногда, вам необходимо остановить распространение события для других слушателей. Вы можете сделать это, возвратив `false` из метода `handle` вашего слушателя.

<a name="queued-event-listeners"></a>
## Слушатели события в очереди

Добавление слушателей в очередь может быть полезным, если ваш слушатель собирается выполнить медленную задачу, такую как отправка e-mail или совершение HTTP-запроса. Перед началом работы со слушателями в очереди убедитесь, что [настроили свою очередь](/docs/{{version}}/queues) и запустите слушателя очереди на своем сервере или в локальной среде разработки.

Чтобы указать, что слушатель должен быть помещен в очередь, просто добавьте интерфейс `ShouldQueue` к классу слушателя. Слушателям, сгенерированным Artisan командой `event:generate`, уже импортирован этот интерфейс в текущее пространство имен. Так что вы можете сразу использовать его:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        //
    }

Вот и всё! Теперь, когда этого слушателя вызывают для события, он будет автоматически поставлен в очередь диспетчером события, использующим [систему очереди](/docs/{{version}}/queues) Laravel. Если никакие исключения не будут выброшены, когда слушатель выполняется из очереди, то задача в очереди будет автоматически удалена после своего выполнения.

#### Настройка подключения очереди и названия очереди

Если вы бы хотели настроить подключение очереди и название очереди, используемые слушателем события, вы можете задать свойства `$connection` и `$queue` в классе своего слушателя:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        /**
         * The name of the connection the job should be sent to.
         *
         * @var string|null
         */
        public $connection = 'sqs';

        /**
         * The name of the queue the job should be sent to.
         *
         * @var string|null
         */
        public $queue = 'listeners';
    }

<a name="manually-accessing-the-queue"></a>
### Ручной доступ к очереди

Если вам необходимо получить доступ к базовым методам очереди `delete` и `release` вручную, вы можете сделать так. Трейт `Illuminate\Queue\InteractsWithQueue`, который импортирован по умолчанию в сгенерированные слушатели, предоставляет вам доступ к этим методам:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        public function handle(OrderShipped $event)
        {
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="handling-failed-jobs"></a>
### Обработка проваленных задач

Иногда работа ваших слушателей в очереди может завершиться неудачей. Если слушатель в очередь преодолевает максимальное количество попыток, как указано в воркере очереди, будет вызван метод метод `failed` на вашем слушателе. Этот метод получает экземпляр события и исключение, которое стало причиной провала задачи:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        public function handle(OrderShipped $event)
        {
            //
        }

        public function failed(OrderShipped $event, $exception)
        {
            //
        }
    }

<a name="dispatching-events"></a>
## Размещение событий

Чтобы разместить событие можно передать экземпляр события хелперу `event`. Этот хелпер разместит событие всем его зарегистрированным слушателям. Так как хелпер `event` доступен глобально, его можно вызвать из любого места в вашем приложении:

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Events\OrderShipped;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * Отправить заданный заказ.
         *
         * @param  int  $orderId
         * @return Response
         */
        public function ship($orderId)
        {
            $order = Order::findOrFail($orderId);

            // Order shipment logic...

            event(new OrderShipped($order));
        }
    }

> {tip} Во время тестирования может быть полезно утверждать, что определенные события были размещены без фактического вызова их слушателей. [Встроенные хелперы тестирования](/docs/{{version}}/mocking#event-fake) Laravel делают эту задачу чрезвычайно легкой.

<a name="event-subscribers"></a>
## Подписчики событий

<a name="writing-event-subscribers"></a>
### Записывание подписчиков событий

Подписчики событий — это классы, которые могут подписаться на множество событий из самого класса, что позволяет вам определить несколько обработчиков событий в одном классе. Подписчики должны определить метод `subscribe`, в который будет передан экземпляр диспетчера события:

    <?php

    namespace App\Listeners;

    class UserEventSubscriber
    {
        /**
         * Обработка событий входа пользователей в систему.
         */
        public function onUserLogin($event) {}

        /**
         * Обработка событий выхода пользователей из системы.
         */
        public function onUserLogout($event) {}

        /**
         * Register the listeners for the subscriber.
         *
         * @param  Illuminate\Events\Dispatcher  $events
         */
        public function subscribe($events)
        {
            $events->listen(
                'Illuminate\Auth\Events\Login',
                'App\Listeners\UserEventSubscriber@onUserLogin'
            );

            $events->listen(
                'Illuminate\Auth\Events\Logout',
                'App\Listeners\UserEventSubscriber@onUserLogout'
            );
        }

    }

<a name="registering-event-subscribers"></a>
### Регистрация подписчиков событий

Когда подписчик определён, он может быть зарегистрирован в диспетчере события. Вы можете зарегистрировать подписчиков, используя свойство `$subscribe` в `EventServiceProvider`. Например, давайте добавим `UserEventSubscriber` в список:

    <?php

    namespace App\Providers;

    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * The event listener mappings for the application.
         *
         * @var array
         */
        protected $listen = [
            //
        ];

        /**
         * The subscriber classes to register.
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventSubscriber',
        ];
    }
