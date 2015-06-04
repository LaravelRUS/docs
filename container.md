git 4deba2bfca6636d5cdcede3f2068eff3b59c15ce

---

# Service Container

- [Введение](#introduction)
- [Использование](#basic-usage)
- [Связывание интерфейса с реализацией](#binding-interfaces-to-implementations)
- [Контекстное связывание](#contextual-binding)
- [Тэгирование](#tagging)
- [Применение на практике](#practical-applications)
- [События](#container-events)

<a name="introduction"></a>
## Введение

Service Container (сервис-контейнер, ранее IoC-контейнер) - это мощное средство для управлением зависимостями классов.
В современном мире веб-разработки есть такой модный термин - Dependency Injection, «внедрение зависимостей», он означает внедрение неких
классов в создаваемый класс через конструктор или метод-сеттер. Создаваемый класс использует эти классы в своей работе.
Сервис-контейнер реализует как раз этот функционал.

Несколько упрощая, можно сказать так: когда фреймворку нужно создать класс, он применяет не конструкцию `new SomeClass(new SomeService())`,
а `App::make('SomeClass')`, предварительно зарегистрировав функцию, которая создает класс `SomeClass` и все классы, которые `SomeClass`
принимает в качестве аргументов конструктора.

Вот простой пример:

    <?php namespace App\Handlers\Commands;

	use App\Commands\PurchasePodcast;
	use Illuminate\Contracts\Mail\Mailer;

	class PurchasePodcastHandler {

		/**
		 * The mailer implementation.
		 */
		protected $mailer;

		/**
		 * Create a new instance.
		 *
		 * @param  Mailer  $mailer
		 * @return void
		 */
		public function __construct(Mailer $mailer)
		{
			$this->mailer = $mailer;
		}

		/**
		 * Purchase a podcast.
		 *
		 * @param  PurchasePodcastCommand  $command
		 * @return void
		 */
		public function handle(PurchasePodcastCommand $command)
		{
			//
		}

	}

В этом примере нам нужно в обработчике `PurchasePodcast` написать письмо пользователю для подтверждения покупки.
Так как мы хотим соблюдать первый принцип SOLID - [«Принцип разделения ответственности»](http://habrahabr.ru/post/208442/), мы не пишем
в нём код общения с SMTP-сервером и т.п., а встраиваем, внедряем (inject) в него класс отправки мейлов.
Преимущество такого подхода - не изменяя код класса `PurchasePodcast` мы можем легко сменить способ отправки почты, например,
с сервиса MailChimp на Mailjet или другой, а для тестирования можем использовать класс-заглушку.

Сервис-контейнер - очень важная вещь, без него невозможно построить действительно большое приложение Laravel. Также глубокое понимание его
работы необходимо, если вы хотите изменять код ядра Laravel и предлагать новые фичи.

<a name="basic-usage"></a>
## Использование

### Связывание (Binding, регистрация)

Так как практически все биндинги, т.е. соответствие строкового ключа реальному объекту в контейнере, в вашем приложении будут регистрироваться
в методе `register()` [сервис-провайдеров](/docs/{{version}}/provider), все нижеследующие примеры даны для этого контекста.
Если вы хотите использовать контейнер в другом месте своего приложения, вы можете внедрить в свой класс `Illuminate\Contracts\Container\Container`.
Так же для доступа к контейнеру можно использовать фасад `App`. (TODO дополнить примерами)

#### Регистрация обычного класса

Внутри сервис-провайдера экземпляр контейнера находится в `$this->app`.

Зарегистрировать (bind, связать) класс можно двумя путями - при помощи коллбэк-функции или привязки интерфейса к реализации.

Рассмотрим первый способ.
Коллбэк регистрируется в сервис-контейнере под неким строковым ключом (в данном случае `FooBar`) - обычно для этого используют название класса,
который будет возвращаться этим коллбэком:

    $this->app->bind('FooBar', function($app)
    {
        return new FooBar($app['SomethingElse']);
    });

Когда из контейнера будет запрошен объект по ключу `FooBar`, контейнер создаст объект класса FooBar, в конструктор которого в качестве
аргумента добавит объект из контейнера с ключом `SomethingElse`.

#### Регистрация класса-синглтона

Иногда вам нужно, чтобы объект создавался один раз, а все остальные разы, когда вы запрашиваете его, вам возвращался тот же созданный экземпляр.
В этом случае вместо `bind` используйте `singleton`:

    $this->app->singleton('FooBar', function($app)
    {
        return new FooBar($app['SomethingElse']);
    });

#### Добавление существующего экземпляра класса в контейнер

Вы можете добавить в контейнер существующий экземпляр класса:

    $fooBar = new FooBar(new SomethingElse);

    $this->app->instance('FooBar', $fooBar);

### Получение из контейнера

Есть несколько способов получить (resolve) содержимое контейнера. Во-первых, вы можете использовать метод `make()`:

    $fooBar = $this->app->make('FooBar');

Во-вторых, вы можете обратиться к контейнеру как к массиву:

    $fooBar = $this->app['FooBar'];

И, наконец, в-третьих (и в главных) вы можете явно указать тип аргумента в конструкторе класса и фреймворк сам возьмёт его из контейнера
(в примере ниже это `UserRepository`): 

    <?php namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;
    use App\Users\Repository as UserRepository;

    class UserController extends Controller {

        /**
         * The user repository instance.
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Show the user with the given ID.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            //
        }

    }

<a name="binding-interfaces-to-implementations"></a>
## Связывание интерфейса с реализацией

### Инъекции зависимостей

Особенно интересная и мощная возможность сервис-контейнера - связывать интерфейсы с различными их реализациями. Например, наше приложение
использует [Pusher](https://pusher.com/) для отправки и приема push-сообщений. Если мы используем Pusher PHP SDK, мы должны внедрить
экземпляр класса `PusherClient` в наш класс:

    <?php namespace App\Handlers\Commands;

    use App\Commands\CreateOrder;
    use Pusher\Client as PusherClient;

    class CreateOrderHandler {

        /**
         * The Pusher SDK client instance.
         */
        protected $pusher;

        /**
         * Create a new order handler instance.
         *
         * @param  PusherClient  $pusher
         * @return void
         */
        public function __construct(PusherClient $pusher)
        {
            $this->pusher = $pusher;
        }

        /**
         * Execute the given command.
         *
         * @param  CreateOrder  $command
         * @return void
         */
        public function execute(CreateOrder $command)
        {
            //
        }

    }

Все бы ничего, но наш код становится завязанным на конкретный сервис - Pusher. Если в дальнейшем мы заходим его сменить, или просто Pusher
сменит названия методов в своем SDK, мы будем вынуждены менять код в нашем классе `CreateOrderHandler`.

### От класса к интерфейсу

Для того, чтобы «изолировать» класс `CreateOrderHandler` от постоянно меняющегося внешнего мира, определим некий постоянный интерфейс,
с реализациями которого наш класс будет теперь работать. 

    <?php namespace App\Contracts;

    interface EventPusher {

        /**
         * Push a new event to all clients.
         *
         * @param  string  $event
         * @param  array  $data
         * @return void
         */
        public function push($event, array $data);

    } 

Когда мы создадим реализацию (implementation) этого интерфейса, `PusherEventPusher`, мы можем связать её с интерфейсом
в методе `register()` сервис-провайдера:

    $this->app->bind('App\Contracts\EventPusher', 'App\Services\PusherEventPusher');

Здесь мы говорим фреймворку, что когда из контейнера будет запрошен `EventPusher`, вместо него отдавать реализацию этого интерфейса,
`PusherEventPusher`. Теперь мы можем переписать наш конструктор класса `СreateOrderHandler` следующим образом:

    /**
     * Create a new order handler instance.
     *
     * @param  EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }

Теперь, с какой бы реализацией работы реалтаймовых сообщений мы бы ни работали, изменять код в `CreateOrderHandler` нам не потребуется.

<a name="contextual-binding"></a>
## Контекстное связывание

Иногда у вас может быть несколько реализаций одного интерфейса и вы хотите внедрять их каждый в свой класс. Например, когда
делается новый заказ, вам нужно отправлять сообщение в [PubNub](http://www.pubnub.com/) вместо Pusher. Вы можете сделать это следующим образом:

    $this->app->when('App\Handlers\Commands\CreateOrderHandler')
              ->needs('App\Contracts\EventPusher')
              ->give('App\Services\PubNubEventPusher');

<a name="tagging"></a>
## Тэгирование

Иногда вам может потребоваться ресолвить реализации в определенной категории. Например, вы пишете сборщик отчётов, который принимает на вход
массив различных реализаций интерфейса `Report`. Вы можете протэгировать их следующим образом:

    $this->app->bind('SpeedReport', function()
    {
        //
    });

    $this->app->bind('MemoryReport', function()
    {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

Теперь вы можете получить их все сразу по тэгу:

    $this->app->bind('ReportAggregator', function($app)
    {
        return new ReportAggregator($app->tagged('reports'));
    });

<a name="practical-applications"></a>
## Использование на практике

Laravel предлагает несколько возможностей использования сервис-контейнера для повышения гибкости и тестируемости вашего кода.
Один из характерных примеров - реализация Dependency Injection в контроллерах. Laravel регистрирует все контроллеры в сервис-контейнере
и поэтому при получении (resolve) класса контроллера из контейнера, автоматически получаются все зависимости, указанные в аргументах
конструктора и других методов контроллера.

    <?php namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;
    use App\Repositories\OrderRepository;

    class OrdersController extends Controller {

        /**
         * The order repository instance.
         */
        protected $orders;

        /**
         * Create a controller instance.
         *
         * @param  OrderRepository  $orders
         * @return void
         */
        public function __construct(OrderRepository $orders)
        {
            $this->orders = $orders;
        }

        /**
         * Show all of the orders.
         *
         * @return Response
         */
        public function index()
        {
            $all = $this->orders->all();

            return view('orders', ['all' => $all]);
        }

    }

В этом примере `OrderRepository` будет автоматически создан и подан как аргумент конструктору. Во время тестирования вы можете связать
ключ `'OrderRepository'` с классом-заглушкой и абстрагироваться от слоя базы данных, протестировав только функционал самого класса `OrdersController`.

#### Другие примеры

Разумеется, контроллеры не единственные классы, которые фреймворк берет из сервис-контейнера. Вы можете использовать этот же принцип
в обработчиках маршрутов, событий, очередей и т.д. Примеры использования сервис-контейнера приведены в соответствующих
разделах документации.

<a name="container-events"></a>
## События контейнера

### Регистрация события на извлечение объекта из контейнера

Сервис-контейнер запускает событие каждый раз, когда объект извлекается из контейнера. Можно ловить все события, можно только те,
которые привязаны к конкретному ключу.

	$this->app->resolvingAny(function($object, $app)
	{
		//
	});

	$this->app->resolving('FooBar', function($fooBar, $app)
	{
		//
	});

Объект, получаемый из контейнера, передается в функцию-коллбэк.
