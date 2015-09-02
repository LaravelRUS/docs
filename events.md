git 4c3d4d4675101a3fa18154d1c85cf292a2292b46

---
# События (events)

- [Введение](#introduction)
- [Registering Events / Listeners](#registering-events-and-listeners)
- [Defining Events](#defining-events)
- [Defining Listeners](#defining-listeners)
	- [Queued Event Listeners](#queued-event-listeners)
- [Firing Events](#firing-events)
- [Broadcasting Events](#broadcasting-events)
	- [Configuration](#broadcast-configuration)
	- [Marking Events For Broadcast](#marking-events-for-broadcast)
	- [Broadcast Data](#broadcast-data)
	- [Consuming Event Broadcasts](#consuming-event-broadcasts)
- [Event Subscribers](#event-subscribers)

<a name="introduction"></a>
## Введение

Laravel's events provides a simple observer implementation, allowing you to subscribe and listen for events in your application. Event classes are typically stored in the `app/Events` directory, while their listeners are stored in `app/Listeners`.

Events в Laravel - это простая реализация паттерна Observer. Вкратце - вы можете генерить события и подписываться на события по их имени. Классы событий по умолчанию находятся в папке `app/Events`, а классы обработчиков событий (listeners) - в app/Listeners.

<a name="registering-events-and-listeners"></a>
## Регистрация Events / Listeners

Сервис-провайдер `EventServiceProvider` - удобное место для регистрации классов слушателей событий. В массиве `$listen` перечисляются названия события (в ключе массива) и название класса, который его обрабатывает.

	/**
	 * The event listener mappings for the application.
	 *
	 * @var array
	 */
	protected $listen = [
		'App\Events\PodcastWasPurchased' => [
			'App\Listeners\EmailPurchaseConfirmation',
		],
	];

### Генерация Event / Listener Classes

Есть быстрый автоматический путь для создания классов событий и слушателей событий. Внесите изменения в `$listen` в `app/Providers/EventServiceProvider` запустите команду `event:generate`:

	php artisan event:generate

Эта команда проходит по массиву и создает файлы классов, если таковые не найдены. Существующие классы она не трогает.

<a name="defining-events"></a>
## Defining Events

An event class is simply a data container which holds the information related to the event. For example, let's assume our generated `PodcastWasPurchased` event receives a [Eloquent ORM](/docs/{{version}}/eloquent) object:

"Класс события" - это просто объект, в котором находятся данные, которые нужны для исполнения события. Вот, например, класс события `PodcastWasPurchased`, которй принимает на вход объект
[Eloquent ORM](/docs/{{version}}/eloquent):

	<?php namespace App\Events;

	use App\Podcast;
	use App\Events\Event;
	use Illuminate\Queue\SerializesModels;

	class PodcastWasPurchased extends Event
	{
	    use SerializesModels;

	    public $podcast;

	    /**
	     * Create a new event instance.
	     *
	     * @param  Podcast  $podcast
	     * @return void
	     */
	    public function __construct(Podcast $podcast)
	    {
	        $this->podcast = $podcast;
	    }
	}

As you can see, this event class contains no special logic. It is simply a container for the `Podcast` object that was purchased. The `SerializesModels` trait used by the event will gracefully serialize any Eloquent models if the event object is serialized using PHP's `serialize` function.

Как вы видите, в классе отсутствует функцонал. Это просто контейнер, содержащий данные, а именно объект `Podcast`. 

<a name="defining-listeners"></a>
## Defining Listeners

Next, let's take a look at the listener for our example event. Event listeners receive the event instance in their `handle` method. The `event:generate` command will automatically import the proper event class and type-hint the event on the `handle` method. Within the `handle` method, you may perform any logic necessary to respond to the event.

	<?php namespace App\Listeners;

	use App\Events\PodcastWasPurchased;
	use Illuminate\Queue\InteractsWithQueue;
	use Illuminate\Contracts\Queue\ShouldQueue;

	class EmailPurchaseConfirmation
	{
	    /**
	     * Create the event listener.
	     *
	     * @return void
	     */
	    public function __construct()
	    {
	        //
	    }

	    /**
	     * Handle the event.
	     *
	     * @param  PodcastWasPurchased  $event
	     * @return void
	     */
	    public function handle(PodcastWasPurchased $event)
	    {
	        // Access the podcast using $event->podcast...
	    }
	}

Your event listeners may also type-hint any dependencies they need on their constructors. All event listeners are resolved via the Laravel [service container](/docs/{{version}}/container), so dependencies will be injected automatically:

	use Illuminate\Contracts\Mail\Mailer;

	public function __construct(Mailer $mailer)
	{
		$this->mailer = $mailer;
	}

#### Stopping The Propagation Of An Event

Sometimes, you may wish to stop the propagation of an event to other listeners. You may do so using by returning `false` from your listener's `handle` method.

<a name="queued-event-listeners"></a>
### Queued Event Listeners

Need to [queue](/docs/{{version}}/queues) an event listener? It couldn't be any easier. Simply add the `ShouldQueue` interface to the listener class. Listeners generated by the `event:generate` Artisan command already have this interface imported into the current namespace, so you can use it immediately:

	<?php namespace App\Listeners;

	use App\Events\PodcastWasPurchased;
	use Illuminate\Queue\InteractsWithQueue;
	use Illuminate\Contracts\Queue\ShouldQueue;

	class EmailPurchaseConfirmation implements ShouldQueue
	{
		//
	}

That's it! Now, when this listener is called for an event, it will be queued automatically by the event dispatcher using Laravel's [queue system](/docs/{{version}}/queues). If no exceptions are thrown when the listener is executed by the queue, the queued job will automatically be deleted after it has processed.

#### Manually Accessing The Queue

If you need to access the underlying queue job's `delete` and `release` methods manually, you may do so. The `Illuminate\Queue\InteractsWithQueue` trait, which is imported by default on generated listeners, gives you access to these methods:

	<?php namespace App\Listeners;

	use App\Events\PodcastWasPurchased;
	use Illuminate\Queue\InteractsWithQueue;
	use Illuminate\Contracts\Queue\ShouldQueue;

	class EmailPurchaseConfirmation implements ShouldQueue
	{
		use InteractsWithQueue;

		public function handle(PodcastWasPurchased $event)
		{
			if (true) {
				$this->release(30);
			}
		}
	}

<a name="firing-events"></a>
## Firing Events

To fire an event, you may use the `Event` [facade](/docs/{{version}}/facades), passing an instance of the event to the `fire` method. The `fire` method will dispatch the event to all of its registered listeners:

	<?php namespace App\Http\Controllers;

	use Event;
	use App\Podcast;
	use App\Events\PodcastWasPurchased;
	use App\Http\Controllers\Controller;

	class UserController extends Controller
	{
		/**
		 * Show the profile for the given user.
		 *
		 * @param  int  $userId
		 * @param  int  $podcastId
		 * @return Response
		 */
		public function purchasePodcast($userId, $podcastId)
		{
			$podcast = Podcast::findOrFail($podcastId);

			// Purchase podcast logic...

			Event::fire(new PodcastWasPurchased($podcast));
		}
	}

Alternatively, you may use the global `event` helper function to fire events:

	event(new PodcastWasPurchased($podcast));

<a name="broadcasting-events"></a>
## Broadcasting Events

In many modern web applications, web sockets are used to implement real-time, live-updating user interfaces. When some data is updated on the server, a message is typically sent over a websocket connection to be handled by the client.

To assist you in building these types of applications, Laravel makes it easy to "broadcast" your events over a websocket connection. Broadcasting your Laravel events allows you to share the same event names between your server-side code and your client-side JavaScript framework.

<a name="broadcast-configuration"></a>
### Configuration

All of the event broadcasting configuration options are stored in the `config/broadcasting.php` configuration file. Laravel supports several broadcast drivers out of the box: [Pusher](https://pusher.com), [Redis](/docs/{{version}}/redis), and a `log` driver for local development and debugging. A configuration example is included for each of these drivers.

#### Broadcast Prerequisites

The following dependencies are needed for event broadcasting:

- Pusher: `pusher/pusher-php-server ~2.0`
- Redis: `predis/predis ~1.0`

#### Queue Prerequisites

Before broadcasting events, you will also need to configure and run a [queue listener](/docs/{{version}}/queues). All event broadcasting is done via queued jobs so that the response time of your application is not seriously affected.

<a name="marking-events-for-broadcast"></a>
### Marking Events For Broadcast

To inform Laravel that a given event should be broadcast, implement the `Illuminate\Contracts\Broadcasting\ShouldBroadcast` interface on the event class. The `ShouldBroadcast` interface requires you to implement a single method: `broadcastOn`. The `broadcastOn` method should return an array of "channel" names that the event should be broadcast on:

	<?php namespace App\Events;

	use App\User;
	use App\Events\Event;
	use Illuminate\Queue\SerializesModels;
	use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

	class ServerCreated extends Event implements ShouldBroadcast
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
	     * Get the channels the event should be broadcast on.
	     *
	     * @return array
	     */
	    public function broadcastOn()
	    {
	        return ['user.'.$this->user->id];
	    }
	}

Then, you only need to [fire the event](#firing-events) as you normally would. Once the event has been fired, a [queued job](/docs/{{version}}/queues) will automatically broadcast the event over your specified broadcast driver.

<a name="broadcast-data"></a>
### Broadcast Data

When an event is broadcast, all of its `public` properties are automatically serialized and broadcast as the event's payload, allowing you to access any of its public data from your JavaScript application. So, for example, if your event has a single public `$user` property that contains an Eloquent model, the broadcast payload would be:

	{
		"user": {
			"id": 1,
			"name": "Jonathan Banks"
			...
		}
	}

However, if you wish to have even more fine-grained control over your broadcast payload, you may add a `broadcastWith` method to your event. This method should return the array of data that you wish to broadcast with the event:

    /**
     * Get the data to broadcast.
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['user' => $this->user->id];
    }

<a name="consuming-event-broadcasts"></a>
### Consuming Event Broadcasts

#### Pusher

You may conveniently consume events broadcast using the [Pusher](https://pusher.com) driver using Pusher's JavaScript SDK. For example, let's consume the `App\Events\ServerCreated` event from our previous examples:

	this.pusher = new Pusher('pusher-key');

	this.pusherChannel = this.pusher.subscribe('user.' + USER_ID);

	this.pusherChannel.bind('App\\Events\\ServerCreated', function(message) {
		console.log(message.user);
	});

#### Redis

If you are using the Redis broadcaster, you will need to write your own Redis pub/sub consumer to receive the messages and broadcast them using the websocket technology of your choice. For example, you may choose to use the popular [Socket.io](http://socket.io) library which is written in Node.

Using the `socket.io` and `ioredis` Node libraries, you can quickly write an event broadcaster to publish all events that are broadcast by your Laravel application:

	var app = require('http').createServer(handler);
	var io = require('socket.io')(app);

	var Redis = require('ioredis');
	var redis = new Redis();

	app.listen(6001, function() {
		console.log('Server is running!');
	});

	function handler(req, res) {
		res.writeHead(200);
		res.end('');
	}

	io.on('connection', function(socket) {
		//
	});

	redis.psubscribe('*', function(err, count) {
		//
	});

	redis.on('pmessage', function(subscribed, channel, message) {
		message = JSON.parse(message);
		io.emit(channel + ':' + message.event, message.data);
	});

<a name="event-subscribers"></a>
## Event Subscribers

Event subscribers are classes that may subscribe to multiple events from within the class itself, allowing you to define several event handlers within a single class. Subscribers should define a `subscribe` method, which will be passed an event dispatcher instance:

	<?php namespace App\Listeners;

	class UserEventListener {

		/**
		 * Handle user login events.
		 */
		public function onUserLogin($event) {}

		/**
		 * Handle user logout events.
		 */
		public function onUserLogout($event) {}

		/**
		 * Register the listeners for the subscriber.
		 *
		 * @param  Illuminate\Events\Dispatcher  $events
		 * @return array
		 */
		public function subscribe($events)
		{
			$events->listen(
				'App\Events\UserLoggedIn',
				'App\Listeners\UserEventListener@onUserLogin'
			);

			$events->listen(
				'App\Events\UserLoggedOut',
				'App\Listeners\UserEventListener@onUserLogout'
			);
		}

	}

#### Registering An Event Subscriber

Once the subscriber has been defined, it may be registered with the event dispatcher. You may register subscribers using the `$subscribe` property on the `EventServiceProvider`. For example, let's add the `UserEventListener`.

    <?php namespace App\Providers;

    use Illuminate\Contracts\Events\Dispatcher as DispatcherContract;
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
            'App\Listeners\UserEventListener',
        ];
    }

# События (events)

- [Основы использования](#basic-usage)
- [Обработка событий в фоне](#queued-event-handlers)
- [Классы-подписчики](#event-subscribers)

<a name="basic-usage"></a>
## Основы использования

Events в Laravel - это простая реализация паттерна Observer, которая позволяет вам подписываться на так называемые события, которые генерируются в некотором месте вашего приложения и обрабатываются как правило в этом же самом запросе. Классы событий по умолчанию находятся в папке `app/Events`, а классы обработчиков событий - в `app/Handlers/Events`.

Создать класс события можно artisan-командой `make:event`:

	php artisan make:event PodcastWasPurchased

#### Подписка на события


Сервис-провайдер `EventServiceProvider` - это удобное место для регистрации классов слушателей событий. В массиве `listen` перечисляются названия события (в ключе массива) и название класса, который его обрабатывает. Например, для нашего `PodcastWasPurchased`:

	/**
	 * The event handler mappings for the application.
	 *
	 * @var array
	 */
	protected $listen = [
		'App\Events\PodcastWasPurchased' => [
			'App\Handlers\Events\EmailPurchaseConfirmation@handle',
		],
	];


Для генерации исполнителя (handler) события используйте artisan-команду `handler:event`:

	php artisan handler:event EmailPurchaseConfirmation --event=PodcastWasPurchased


Но вообще, использование двух команд, `make:event` и `handler:event` каждый раз, когда вам нужно создать обработчик события - это неудобно. Вместо этого можно заполнить массив `listen` в `EventServiceProvider` и выполнить artisan-команду `event:generate`. Эта команда анализирует `listen` и создает все необходимые классы событий и обработчиков событий:

	php artisan event:generate

#### Запуск события

Событие может быть запущено (fire) при помощи фасада `Event`:

	$response = Event::fire(new PodcastWasPurchased($podcast));

Метод `fire` возвращает массив ответов (responses).

Вместо фасада можно использовать хэлпер:

	event(new PodcastWasPurchased($podcast));

#### Слушатели в функциях-замыканиях

Чтобы упростить себе работу и не создавать два класса, вы можете создать слушателя в виде функции-замыкания. Сделать это можно в методе `boot` сервис-провайдера `EventServiceProvider`:

	Event::listen('App\Events\PodcastWasPurchased', function($event)
	{
		// Обработка события...
	});

#### Остановка распространения события

Иногда вам нужно остановить распространение события, чтобы до других обработчиков, подписанных на это событие, оно не дошло. Для этого надо вернуть `false` в обработчике события:

	Event::listen('App\Events\PodcastWasPurchased', function($event)
	{
		// Обработка события...

		return false;
	});

<a name="queued-event-handlers"></a>
## Обработка события в фоне

Хотите запустить обработчик события в фоне при помощи [queue](/docs/{{version}}/queues) ? Это делается очень легко. При создании класса обработчика события добавьте флаг `--queued`:

	php artisan handler:event SendPurchaseConfirmation --event=PodcastWasPurchased --queued

Или вручную поставьте реализацию (implements) интерфейса `Illuminate\Contracts\Queue\ShouldBeQueued`. Это всё ! Теперь при возникновении события фреймворк поместит класс обработки этого события в очередь, откуда он будет запущен диспетчером очереди.

Если во время выполнения метода `handle` не будет брошено ни одного непойманного исключения, то обработчик события по завершении сам удалится из очереди. Но если вы хотите иметь доступ к методам задачи в очереди `delete` and `release` (вернуть задачу в очередь), то добавьте в класс обработчика события трейт `Illuminate\Queue\InteractsWithQueue`.

	public function handle(PodcastWasPurchased $event)
	{
		if (true)
		{
			$this->release(30);
		}
	}

<a name="event-subscribers"></a>
## Подписчики 

#### Defining An Event Subscriber

Подписчики на события (Event Subscribers) - классы, которые могут быть подписаны на несколько событий и содержать сразу несколько обработчиков событий. Такой класс должен иметь метод `subscribe`, который принимает в аргументах инстанс диспетчера событий:

	class UserEventHandler {

		/**
		 * Handle user login events.
		 */
		public function onUserLogin($event)
		{
			//
		}

		/**
		 * Handle user logout events.
		 */
		public function onUserLogout($event)
		{
			//
		}

		/**
		 * Register the listeners for the subscriber.
		 *
		 * @param  Illuminate\Events\Dispatcher  $events
		 * @return array
		 */
		public function subscribe($events)
		{
			$events->listen('App\Events\UserLoggedIn', 'UserEventHandler@onUserLogin');

			$events->listen('App\Events\UserLoggedOut', 'UserEventHandler@onUserLogout');
		}

	}

#### Регистрация Event Subscriber

После того как класс определён, его можно зарегистрировать следующим образом:

	$subscriber = new UserEventHandler;

	Event::subscribe($subscriber);

Вы также можете использовать [сервис-контейнер](/docs/{{version}}/container) для того, чтобы получить объект своего подписчика на события:

	Event::subscribe('UserEventHandler');
