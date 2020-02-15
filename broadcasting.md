git 63a231cc5940dda64e8f0b08e0b9e7be33b6e27c

---

# Широковещание

- [Введение](#introduction)
    - [Настройка](#configuration)
    - [Требования драйвера](#driver-prerequisites)
- [Обзор концепции](#concept-overview)
    - [Использование примера приложения](#using-example-application)
- [Определение широковещательных событий](#defining-broadcast-events)
    - [Имя широковещания](#broadcast-name)
    - [Данные широковещания](#broadcast-data)
    - [Очередь широковещания](#broadcast-queue)
    - [Условия широковещания](#broadcast-conditions)
- [Авторизация на каналах](#authorizing-channels)
    - [Определение роутов авторизации](#defining-authorization-routes)
    - [Определение анонимных функций авторизации](#defining-authorization-callbacks)
    - [Классы каналов](#defining-channel-classes)
- [Широковещание событий](#broadcasting-events)
    - [Только другим](#only-to-others)
- [Получение широковещаний](#receiving-broadcasts)
    - [Установка Laravel Echo](#installing-laravel-echo)
    - [Прослушивание событий](#listening-for-events)
    - [Уход с канала](#leaving-a-channel)
    - [Пространства имён](#namespaces)
- [Каналы присутствия](#presence-channels)
    - [Авторизация каналов присутствия](#authorizing-presence-channels)
    - [Присоединение к каналам присутствия](#joining-presence-channels)
    - [Широковещание каналам присутствия](#broadcasting-to-presence-channels)
- [События клиента](#client-events)
- [Уведомления](#notifications)

<a name="introduction"></a>
## Введение

Во многих современных веб-приложениях WebSockets (веб-сокеты) используются для реализации пользовательских интерфейсов, обновляющихся в режиме реального времени. Когда на сервере обновляются какие-то данные, сообщение обычно отправляется через подключение WebSocket, чтобы потом быть обработанным клиентом. Это обеспечивает более надежную, эффективную альтернативу постоянному опросу вашего приложения на наличие изменений.

Чтобы помочь вам в построении таких типов приложений, Laravel упрощает "широковещание" (бродкастинг) ваших [событий](/docs/{{version}}/events) через подключение WebSocket. Широковещание ваших Laravel-событий позволяет использовать одни и те же названия событий и в серверном коде, и в клиентском JavaScript.

> {tip} Прежде чем погрузиться в широковещание событий, убедитесь, что прочитали всю документацию о [событиях и слушателях](/docs/{{version}}/events) Laravel.

<a name="configuration"></a>
### Настройка

Все настройки широковещания (broadcast) событий вашего приложения хранятся в конфиге `config/broadcasting.php`. Laravel изначально поддерживает несколько драйверов вещания: [Pusher Channels](https://pusher.com/channels), [Redis](/docs/{{version}}/redis) и драйвер `log` для локальной разработки и отладки. Дополнительно включен драйвер `null`, который позволяет полностью отключить широковещание. Для каждого из этих драйверов есть пример настройки в конфиге `config/broadcasting.php`.

#### Сервис-провайдер широковещания

Перед вещанием любых событий сначала нужно зарегистрировать `App\Providers\BroadcastServiceProvider`. В свежем приложении Laravel вам потребуется только откомментировать этого провайдера в массиве `providers` вашего конфига `config/app.php`. Данный провайдер позволит зарегистрировать роуты и анонимные функции авторизации широковещания.

#### CSRF токен

У [Laravel Echo](#installing-laravel-echo) должен быть доступ к CSRF токену текущей сессии. Вы должны задать тег `meta` в HTML-элементе `head` вашего приложения:

    <meta name="csrf-token" content="{{ csrf_token() }}">

<a name="driver-prerequisites"></a>
### Требования драйвера

#### Pusher

Если вы вещаете свои события через [Pusher Channels](https://pusher.com/channels), вам нужно установить Pusher PHP SDK, используя менеджер пакетов Composer:

    composer require pusher/pusher-php-server "~4.0"

Далее нужно настроить учетные данные Pusher Channels в конфиге `config/broadcasting.php`. Пример конфига уже включен в этот файл, что позволит вам быстро указать ваш ключ Pusher Channels, секрет, а также ID приложения. Настройка `pusher` файла `config/broadcasting.php` позволяет указать дополнительные опции `options`, поддерживаемые Pusher, такие как кластер:

    'options' => [
        'cluster' => 'eu',
        'useTLS' => true
    ],

При использовании Pusher Channels и [Laravel Echo](#installing-laravel-echo) следует указать `pusher` в качестве желаемого широковещателя при установке экземпляра Echo в файле `resources/assets/js/bootstrap.js`:

    import Echo from "laravel-echo";

    window.Pusher = require('pusher-js');

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key'
    });

#### Redis

Если вы используете широковещание при помощи Redis, вам нужно установить php-расширение `phpredis` или библиотеку `Predis` при помощи Composer:

    composer require predis/predis

Широковещатель Redis будет вещать сообщения через функцию Redis pub/sub; однако, потребуется связать его с WebSocket-сервером, который может получать сообщения от Redis и вещать их на ваши WebSocket-каналы.

Когда широковещатель Redis публикует событие, оно будет опубликовано на каналах события с ранее указанными именами. Информационным наполнением будет служить кодированная строка JSON, содержащая имя события, `data` и пользователя, который сгенерировал сокет ID события (если применимо).

#### Socket.IO

Если вы собираетесь соединить широковещатель Redis с сервером Socket.IO, потребуется включить клиентскую библиотеку Socket.IO JavaScript в HTML-элемент `head` вашего приложения. После запуска сервер Socket.IO автоматически раскроет клиентскую JavaScript-библиотеку в виде стандартного URL. Например, если ваш сервер Socket.IO находится на том же домене, как и ваше веб-приложение, доступ к клиентской библиотеке можно получить следующим образом:

    <script src="//{{ Request::getHost() }}:6001/socket.io/socket.io.js"></script>

Если вы собираетесь соединить широковещатель Redis с сервером Socket.IO, потребуется установить клиентскую библиотеку Socket.IO: 

    npm install --save socket.io-client

Затем нужно инстанцировать Echo с коннектором `socket.io` и хостом `host`.

    import Echo from "laravel-echo"

    window.io = require('socket.io-client');

    window.Echo = new Echo({
        broadcaster: 'socket.io',
        host: window.location.hostname + ':6001'
    });

Наконец, надо запустить совместимый Socket.IO сервер. Laravel не включает реализацию сервера Socket.IO; однако, управляемый обществом сервер Socket.IO в данный момент поддерживается в GitHub репозитории [tlaverdure/laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server).

#### Требования к очереди

Перед широковещанием событий вам также потребуется настроить и запустить [слушателя очереди](/docs/{{version}}/queues). Все вещание события происходит через задачи в очереди, чтобы не оказать значительного влияния на время ответа вашего приложения.

<a name="concept-overview"></a>
## Обзор концепции

Широковещание событий Laravel позволяет вещать свои серверные события Laravel клиентскому JavaScript приложения, используя подход, основанный на драйверах. Сейчас Laravel поставляется с драйверами [Pusher Channels](https://pusher.com/channels) и Redis. События могут быть легко использованы кодом на стороне клиента благодаря Javascript-пакету [Laravel Echo](#installing-laravel-echo).

События вещаются по "каналам", которые можно указать как общедоступные или приватные. Любой посетитель вашего приложения может подписаться на общедоступный канал без аутентификации или авторизации; однако, чтобы подписаться на приватный канал пользователь должен быть аутентифицирован и авторизован прослушивать этот канал.

<a name="using-example-application"></a>
### Использование примера приложения

Прежде чем погрузиться в каждый компонент вещания событий, давайте рассмотрим общий пример - онлайн-магазин. Мы не будем обсуждать детали настройки [Pusher Channels](https://pusher.com/channels) или [Laravel Echo](#installing-laravel-echo), так как более детальное обсуждение вас ждет в других разделах этой документации.

Давайте предположим, что в нашем приложении есть страница, которая позволяет пользователям просматривать статус отправки своих заказов. Также предположим, что когда приложением обрабатывается обновление статуса отправки, выдается событие `ShippingStatusUpdated`:

    event(new ShippingStatusUpdated($update));

#### Интерфейс `ShouldBroadcast`

Нам бы не хотелось, чтобы пользователю приходилось обновлять страницу, чтобы просмотреть обновление статуса заказа, когда пользователь просматривает один из своих заказов. Вместо этого мы бы хотели вещать обновления приложению по мере их создания. Поэтому нужно обозначить событие `ShippingStatusUpdated` в интерфейсе `ShouldBroadcast`. Это прикажет Laravel вещать событие, когда оно запускается:

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
    use Illuminate\Queue\SerializesModels;

    class ShippingStatusUpdated implements ShouldBroadcast
    {
        /**
         * Information about the shipping status update.
         *
         * @var string
         */
        public $update;
    }

Интерфейс `ShouldBroadcast` требует от нашего события определения метода `broadcastOn`. Этот метод отвечает за возвращение каналов, на которых должно вещаться событие. Пустая заглушка данного метода уже определена в сгенерированных классах событий, поэтому нам нужно заполнить только подробности. Нам только нужно, чтобы создатель заказа имел возможность просматривать обновления статуса, поэтому мы будем вещать это событие на приватном канале, который привязан к заказу:

    /**
     * Get the channels the event should broadcast on.
     *
     * @return \Illuminate\Broadcasting\PrivateChannel
     */
    public function broadcastOn()
    {
        return new PrivateChannel('order.'.$this->update->order_id);
    }

#### Авторизация на канале

Помните, что пользователи должны быть авторизованы, чтобы слушать на приватных каналах. Можно определить наши правила авторизации в файле `routes/channels.php`. В данном примере нам нужно проверить, что пользователь, пытающийся слышать на приватном канале `order.1`, фактически является создателем этого заказа:

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

Метод `channel` принимает два аргумента: имя канала и анонимную функцию, которая возвращает `true` или `false`, указывая авторизован ли пользователь слушать на этом канале.

Все функции авторизации получают аутентифицированного в данный момент пользователя в качестве своего первого аргумента и любые дополнительные параметры маски - в качестве последующих аргументов. В этом примере мы используем заполнитель `{orderId}` для указания того, что "ID"-часть канала - специальный символ.

#### Слушаем широковещание событий

Затем, все что остается - слушать событие в нашем JavaScript-приложении. Для этого используется Laravel Echo. Сначала мы используем метод `private` для подписки на приватный канал. Потом можно использовать метод `listen`, чтобы слушать событие `ShippingStatusUpdated`. По умолчанию все общедоступные свойства события будут включены в вещаемое событие:

    Echo.private(`order.${orderId}`)
        .listen('ShippingStatusUpdated', (e) => {
            console.log(e.update);
        });

<a name="defining-broadcast-events"></a>
## Определение широковещательных событий

Внедрите интерфейс `Illuminate\Contracts\Broadcasting\ShouldBroadcast` в классе события, чтобы собщить Laravel, что это событие нужно вещать. Такой интерфейс уже импортирован во все классы событий, сгенерированные фреймворком, так что его легко можно добавить к любому из своих событий.

Интерфейс `ShouldBroadcast` требует реализовать единственный метод: `broadcastOn`. Метод `broadcastOn` должен возвращать канал или массив каналов, на которых должно вещаться событие. Каналы должны быть экземплярами `Channel`, `PrivateChannel` или `PresenceChannel`. Экземпляры `Channel` представляют общедоступные каналы, на которые может подписаться любой пользователь, в то время как `PrivateChannels` и `PresenceChannels` представляют приватные каналы, для которых требуется [авторизация на канале](#authorizing-channels):

    <?php

    namespace App\Events;

    use App\User;
    use Illuminate\Broadcasting\Channel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
    use Illuminate\Queue\SerializesModels;

    class ServerCreated implements ShouldBroadcast
    {
        use SerializesModels;

        public $user;

        /**
         * Create a new event instance.
         *
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * Get the channels the event should broadcast on.
         *
         * @return Channel|array
         */
        public function broadcastOn()
        {
            return new PrivateChannel('user.'.$this->user->id);
        }
    }

Затем нужно только [запустить событие](/docs/{{version}}/events) как обычно. Как только событие было запущено, [задача в очереди](/docs/{{version}}/queues) будет автоматически вещать событие через указанный вами драйвер широковещания.

<a name="broadcast-name"></a>
### Имя широковещания

По умолчанию Laravel будет вещать событие используя имя класса события. Однако, вы можете настроить имя широковещания, определив метод `broadcastAs`:

    /**
     * The event's broadcast name.
     *
     * @return string
     */
    public function broadcastAs()
    {
        return 'server.created';
    }

Если вы измените имя широковещания, используя метод `broadcastAs`, нужно не забыть зарегистрировать своего слушателя с символом `.` в начале. Это прикажет Echo не добавлять пространство имён приложения к событию:

    .listen('.server.created', function (e) {
        ....
    });


<a name="broadcast-data"></a>
### Данные широковещания

Когда событие является широковещательным, все его свойства `public` автоматически сериализируются и вещаются как данные события, позволяя вам получить доступ к его общедоступным данным из вашего JavaScript-приложения. Так, к примеру, если у вашего события есть единственное общедоступное свойство `$user`, которое содержит модель Eloquent, полезная нагрузка вещания события будет:

    {
        "user": {
            "id": 1,
            "name": "Patrick Stewart"
            ...
        }
    }

Однако, если вам нужен более тщательный контроль над полезной нагрузкой вещания, можно добавить метод `broadcastWith` к своему событию. Этот метод должен возвращать массив данных, которые вы хотите передавать в качестве полезной нагрузки своего вещания:

    /**
     * Get the data to broadcast.
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['id' => $this->user->id];
    }

<a name="broadcast-queue"></a>
### Очередь широковещания

По умолчанию каждое событие помещается в очередь по умолчанию для подключения очереди по умолчанию, указанным в вашем конфиге `queue.php`. Можно настроить очередь, используемую в вещателе, определив свойство `broadcastQueue` на вашем классе события. Это свойство должно указывать имя очереди, которую вы хотите использовать во время широковещания:

    /**
     * The name of the queue on which to place the event.
     *
     * @var string
     */
    public $broadcastQueue = 'your-queue-name';

Если вы хотите вещать свое событие используя очередь `sync` вместо драйвера очереди по умолчанию, можно реализовать интерфейс `ShouldBroadcastNow` вместо `ShouldBroadcast`:

    <?php

    use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

    class ShippingStatusUpdated implements ShouldBroadcastNow
    {
        //
    }

<a name="broadcast-conditions"></a>
### Условия широковещания

Иногда может потребоваться широковещать ваше событие только если выполняются определенные условия. Эти условия можно определить добавив метод `broadcastWhen` к вашему классу события:

    /**
     * Determine if this event should broadcast.
     *
     * @return bool
     */
    public function broadcastWhen()
    {
        return $this->value > 100;
    }

<a name="authorizing-channels"></a>
## Авторизация на каналах

Приватные каналы требуют авторизовывать то, что аутентифицированный в данный момент пользователь может фактически слушать на канале. Это достигается путем совершения HTTP-запроса к вашему приложению Laravel с именем канала и позволяя вашему приложению определять может ли пользователь слушать на этом канале. При использовании [Laravel Echo](#installing-laravel-echo), HTTP-запрос для авторизации подписок на приватные каналы будет сделан автоматически; однако, вам не нужно определять подходящие роуты в ответ на эти запросы.

<a name="defining-authorization-routes"></a>
### Определение роутов авторизации

К счастью, в Laravel просто определить роуты в ответ на запросы авторизации каналов. В `BroadcastServiceProvider`, включенном в ваше Laravel-приложение, вы увидите вызов метода `Broadcast::routes`. Этот метод зарегистрирует роут `/broadcasting/auth` для обработки запросов авторизации:

    Broadcast::routes();

Метод `Broadcast::routes` будет автоматически размещать свои роуты в группе посредников `web`; Тем не менее, можно передавать массив атрибутов роута методу, если вам нужно настроить назначенные атрибуты:

    Broadcast::routes($attributes);

#### Настройка урла авторизации

По умолчанию Echo использует урл `/broadcasting/auth` для авторизации каналов. Вы можете изменить это в параметре `authEndpoint` настройки инстанса Echo:

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key',
        authEndpoint: '/custom/endpoint/auth'
    });

<a name="defining-authorization-callbacks"></a>
### Определение анонимных функций авторизации

Далее нам нужно определить логику, которая будет выполнять авторизацию канала. Это делается в файле `routes/channels.php`, который включен в ваше приложение. В этом файле можно использовать метод `Broadcast::channel` для регистрации анонимных функций авторизации канала:

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

Метод `channel` принимает два аргумента: имя канала и обратную функцию, которая возвращает `true` или `false`, указывая авторизован ли пользователь слушать канал.

Все анонимные функции авторизации получают аутентифицированнного в данный момент пользователя в качестве своего первого аргумента, и любые дополнительные подстановочные параметры в качестве дополнительных аргументов. В этом примере мы используем заполнитель `{orderId}` для указания, что "ID"-часть канала является спецсимволом.

#### Привязка моделей анонимной функции авторизации

Как и HTTP роуты, роуты каналов также могут пользоваться явной и неявной [привязкой моделей роутов](/docs/{{version}}/routing#route-model-binding). Например, вместо получения строки или числового ID заказа можно запросить экземпляр модели фактического заказа `Order`:

    use App\Order;

    Broadcast::channel('order.{order}', function ($user, Order $order) {
        return $user->id === $order->user_id;
    });

#### Аутентификация анонимной функции авторизации

Приватные (private) каналы и каналы присутствия (presence) аутентифицируют пользователя при помощи дефолтного гварда аутентификации, если пользователь не аутентифицирован, функция авторизации канала не вызывается. Вы также можете указать, что для аутентификации нужно использовать заданные гварды, отличающиеся от дефолтного:

    Broadcast::channel('channel', function () {
        // ...
    }, ['guards' => ['web', 'admin']]);

<a name="defining-channel-classes"></a>
### Классы каналов

Если ваше приложение использует большое число каналов, `routes/channels.php` может получиться слишком раздутым. Поэтому вместо анонимных функций авторизации каналов вы можете использовать классы каналов. Класс можно создать командой `make:channel`, класс создастся в папке `App/Broadcasting`:

    php artisan make:channel OrderChannel

Далее надо зарегистрировать класс в `routes/channels.php`:

    use App\Broadcasting\OrderChannel;

    Broadcast::channel('order.{order}', OrderChannel::class);

Логику авторизации нужно разместить в методе `join` созданного класса. Принципы написания те же, что и при использовании анонимнах функций авторизации, включая неявную привязку моделей: 

    <?php

    namespace App\Broadcasting;

    use App\Order;
    use App\User;

    class OrderChannel
    {
        /**
         * Create a new channel instance.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Authenticate the user's access to the channel.
         *
         * @param  \App\User  $user
         * @param  \App\Order  $order
         * @return array|bool
         */
        public function join(User $user, Order $order)
        {
            return $user->id === $order->user_id;
        }
    }

> {tip} Классаы каналов, как и другие классы Laravel, вызываются при помощи [сервис-контейнера](/docs/{{version}}/container), поэтому в них работает автоподстановка классов (type-hint) в аргументах конструктора

<a name="broadcasting-events"></a>
## Широковещание событий

Как только вы определили событие и пометили его интерфейсом `ShouldBroadcast`, нужно только запустить событие, используя функцию `event`. Планировщик события заметит, что событие помечено интерфейсом `ShouldBroadcast` и поместит данное событие в очередь на широковещание:

    event(new ShippingStatusUpdated($update));

<a name="only-to-others"></a>
### Только другим

При создании приложения, которое использует широковещание событий, можно заменить функцию `event` функцией `broadcast`. Как и функция `event`, функция `broadcast` отправляет событие вашим серверным слушателям:

    broadcast(new ShippingStatusUpdated($update));

Однако, функция `broadcast` также предоставляет метод `toOthers`, который позволяет исключить текущего пользователя из получателей широковещания:

    broadcast(new ShippingStatusUpdated($update))->toOthers();

Чтобы лучше понять когда мы можете захотеть использовать метод `toOthers`, давайте представим приложение со списком задач, где пользователь может создавать новую задачу путем ввода имени задачи. Чтобы создать задачу, ваше приложение может сделать запрос к api-эндпоинту (урлу) `/task`, которая вещает результат этой задачи и возвращает JSON-представление новой задачи. Когда ваше JavaScript-приложение получает ответ от эндпоинта, оно может напрямую вставить новую задачу в свой список задач, как указано ниже:

    axios.post('/task', task)
        .then((response) => {
            this.tasks.push(response.data);
        });

Но не забывайте, что факт создания задачи мы так же и вещаем. Если ваше JavaScript-приложение слушает это событие, чтобы добавить задачи в список задач, вы создадите дубли задач в своем списке: одну от эндпоинта и одну от широковещателя. С этим можно разобраться используя метод `toOthers`, чтобы сообщить широковещателю не вещать событие текущему пользователю.

> {note} Ваше событие должно использовать трейт `Illuminate\Broadcasting\InteractsWithSockets` , чтобы в нём был доступен метод `toOthers`.

#### Настройка

При инициализации экземпляра Laravel Echo, подключению назначается сокет ID. Если вы используете [Vue](https://vuejs.org) и [Axios](https://github.com/mzabriskie/axios), сокет ID будет автоматически прикрепляться к каждому исходящему запросу как заголовок `X-Socket-ID`. В таком случае когда вы вызовете метод `toOthers`, Laravel извлечет сокет ID из заголовка и прикажет широковещателю не вещать никаким соединениям с этим сокет ID.

Если вы не используете Vue и Axios, то потребуется вручную настраивать свое JavaScript приложение отправлять заголовок `X-Socket-ID`. Этот сокет ID можно получить, используя метод `Echo.socketId`:

    var socketId = Echo.socketId();

<a name="receiving-broadcasts"></a>
## Получение широковещаний

<a name="installing-laravel-echo"></a>
### Установка Laravel Echo

Laravel Echo - это JavaScript-библиотека, которая делает подписку на каналы и слушание вещания событий Laravel безболезненным. Echo можно установить через менеджер пакетов NPM. В этом примере мы также установим пакет `pusher-js`, так как мы собираемся использовать вещатель Pusher Channels:

    npm install --save laravel-echo pusher-js

После установки Echo вы будете готовы создать свежий экземпляр Echo в JavaScript своего приложения. Отличное место для этого - нижняя часть файла `resources/assets/js/bootstrap.js`, который включен в фреймворк Laravel:

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key'
    });

При создании экземпляра Echo, который использует коннектор `pusher`, также можно указать и `cluster`, а также должно ли подключение шифроваться при помощи TLS:

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key',
        cluster: 'eu',
        forceTLS: true
    });

#### Использование существующего экземпляра приложения

Если у вас в приложении уже используется Pusher Channels или Socket.io клиент, вы можете использовать его, указав его как `client` при конфигурировании:

    const client = require('pusher-js');

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key',
        client: client
    });    

<a name="listening-for-events"></a>
### Слушание событий

Как только вы установили и инстанцировали Echo, вы готовы начать слушать широковещания событий. Сначала используйте метод `channel`, чтобы получить экземпляр канала, затем вызовите метод `listen`, чтобы слушать указанное событие:

    Echo.channel('orders')
        .listen('OrderShipped', (e) => {
            console.log(e.order.name);
        });

Если нужно слушать события на приватном канале - вместе этого используйте метод `private`. Можно продолжить привязывать вызовы к методу `listen`, чтобы слушать несколько событий на одном канале:

    Echo.private('orders')
        .listen(...)
        .listen(...)
        .listen(...);

<a name="leaving-a-channel"></a>
### Уход с канала

Чтобы уйти с канала можно вызвать метод `leaveChannel` на экземпляре Echo:

    Echo.leaveChannel('orders');

Если вы хотите покинуть канал, а также связанные с ним приватные каналы и каналы присутствия, вы можете вызвать метод `leave`:

    Echo.leave('orders');

<a name="namespaces"></a>
### Пространства имён

Вы могли заметить в примерах выше, что мы не указывали полное пространство имён для классов событий. Это из-за того, что Echo будет автоматически предполагать, что события расположены в пространстве имён `App\Events`. Однако, можно настроить корневое пространство имён при инстанцировании Echo передав опцию настройки `namespace`:

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key',
        namespace: 'App.Other.Namespace'
    });

Или же, можно добавить к классам событий префикс `.` при подписке на них с использованием Echo. Это позволит всегда указывать полное имя класса:

    Echo.channel('orders')
        .listen('.Namespace\\Event\\Class', (e) => {
            //
        });

<a name="presence-channels"></a>
## Каналы присутствия (presence)

Каналы присутствия (presence channels) строятся на безопасности приватных каналов, предоставляя дополнительную особенность осведомленности о том, кто подписан на канал. Это упрощает построение мощных, коллаборативных функций приложения, таких как уведомление пользователей о том, когда другой пользователь просматривает ту же страницу.

<a name="authorizing-presence-channels"></a>
### Авторизация каналов присутствия

Все каналы присутствия также являются приватными каналами; таким образом, пользователи должны быть [авторизованы, чтобы получить к ним доступ](#authorizing-channels). Однако, при определении анонимных функций авторизации для каналов присутствия вы не будете возвращать `true`, если пользователь авторизован присоединяться к каналу. Вместо этого вы должны вернуть массив данных о пользователе.

Данные, возвращаемые функцией авторизации, будут доступны слушателям события канала присутствия в вашем JavaScript-приложении. Если пользователь не авторизован присоединяться к каналу присутствия, вы должны вернуть `false` или `null`:

    Broadcast::channel('chat.{roomId}', function ($user, $roomId) {
        if ($user->canJoinRoom($roomId)) {
            return ['id' => $user->id, 'name' => $user->name];
        }
    });

<a name="joining-presence-channels"></a>
### Присоединение к каналам присутствия

Чтобы присоединиться к каналу присутствия можно использовать метод Echo `join`. Этот метод вернет реализацию `PresenceChannel`, которая, вместе с предоставлением метода `listen`, позволит подписаться на события `here`, `joining` и `leaving`.

    Echo.join(`chat.${roomId}`)
        .here((users) => {
            //
        })
        .joining((user) => {
            console.log(user.name);
        })
        .leaving((user) => {
            console.log(user.name);
        });

Анонимная функция `here` будет выполнена незамедлительно после успешного присоединения к каналу, и получит массив, содержащий информацию пользователя для всех других пользователей, в данный момент подписанных на канал. Метод `joining` будет выполнен, когда к каналу присоединится новый пользователь, в то время как метод `leaving` будет выполнен, когда пользователь покинет канал.

<a name="broadcasting-to-presence-channels"></a>
### Широковещание каналам присутствия

Каналы присутствия могут получить события как и общедоступные или приватные каналы. На примере чат-комнаты: мы можем захотеть вещать события `NewMessage` на канал присутствия комнаты. Для этого мы вернем экземпляр `PresenceChannel` из метода `broadcastOn` события:

    /**
     * Get the channels the event should broadcast on.
     *
     * @return Channel|array
     */
    public function broadcastOn()
    {
        return new PresenceChannel('room.'.$this->message->room_id);
    }

Как и общедоступные или приватные события, события канала присутствия можно вещать, используя функцию  `broadcast`. Как и с другими событиями, можно использовать метод `toOthers` для исключения текущего пользователя из получателей вещания:

    broadcast(new NewMessage($message));

    broadcast(new NewMessage($message))->toOthers();

Событие присоединения можно слушать через метод Echo `listen`:

    Echo.join(`chat.${roomId}`)
        .here(...)
        .joining(...)
        .leaving(...)
        .listen('NewMessage', (e) => {
            //
        });

<a name="client-events"></a>
## Клиентские события

> {tip} Если вы используете [Pusher Channels](https://pusher.com/channels), для использования клиентских событий вы должны включить опцию "Client Events" в разделе "App Settings" [дашбоарда на pusher.com](https://dashboard.pusher.com/)

Иногда вам может потребоваться вещать событие другим клиентам подключения, совсем не трогая ваше Laravel-приложение. Это может особенно пригодится для таких вещей, как уведомлений о том, что пользователь "печатает", где одному из пользователей на экране приложения будет отображаться уведомление, что другу пользователь в данный момент печатает ему сообщение. 

Для вещания событий клиента можно использовать метод Echo `whisper`:

    Echo.private('chat')
        .whisper('typing', {
            name: this.user.name
        });

Чтобы слушать события клиентов можно использовать метод `listenForWhisper`:

    Echo.private('chat')
        .listenForWhisper('typing', (e) => {
            console.log(e.name);
        });

<a name="notifications"></a>
## Уведомления

Связывая широковещание события с [уведомлениями](/docs/{{version}}/notifications), ваше JavaScript-приложение может получать новые уведомления по мере их появления без необходимости обновлять страницу. Сначала убедитесь, что ознакомились с документацией об использовании [канала уведомлений вещания](/docs/{{version}}/notifications#broadcast-notifications).

Как только настроено, что уведомление будет использовать канал широковещания, можно слушать события вещания через Echo метод `notification`. Помните, что название канала должно совпадать с именем класса сущности, получающей уведомления:

    Echo.private(`App.User.${userId}`)
        .notification((notification) => {
            console.log(notification.type);
        });

В этом примере все уведомления, отправляемые экземплярам `App\User` через канал `broadcast`, будут получены анонимной функцией. Функция авторизации канала для канала `App.User.{id}` включена в `BroadcastServiceProvider` по умолчанию, который поставляется с фреймворком Laravel.
