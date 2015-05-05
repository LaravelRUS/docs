git 4c3d4d4675101a3fa18154d1c85cf292a2292b46

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
