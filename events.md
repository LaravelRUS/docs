git 4deba2bfca6636d5cdcede3f2068eff3b59c15ce

---

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

The `EventServiceProvider` included with your Laravel application provides a convenient place to register all event handlers. The `listen` property contains an array of all events (keys) and their handlers (values). Of course, you may add as many events to this array as your application requires. For example, let's add our `PodcastWasPurchased` event:

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

To generate a handler for an event, use the `handler:event` Artisan CLI command:

Для генерации исполнителя (handler) события используйте artisan-команду `handler:event`:

	php artisan handler:event EmailPurchaseConfirmation --event=PodcastWasPurchased

Of course, manually running the `make:event` and `handler:event` commands each time you need a handler or event is cumbersome. Instead, simply add handlers and events to your `EventServiceProvider` and use the `event:generate` command. This command will generate any events or handlers that are listed in your `EventServiceProvider`:

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

Sometimes, you may wish to stop the propagation of an event to other listeners. You may do so using by returning `false` from your handler:

Иногда вам нужно остановить распространение события, чтобы до других обработчиков, подписанных на это событие, оно не дошло. Для этого надо вернуть `false` в обработчике события:

	Event::listen('App\Events\PodcastWasPurchased', function($event)
	{
		// Обработка события...

		return false;
	});

<a name="queued-event-handlers"></a>
## Обработка события в фоне

Хотите запустить обработчик события в фоне при помощи [queue](/docs/5.0/queues) ? Это делается очень легко. При создании класса обработчика события добавьте флаг `--queued`:

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

Вы также можете использовать [сервис-контейнер](/docs/5.0/ioc) для того, чтобы получить объект своего подписчика на события:

	Event::subscribe('UserEventHandler');
