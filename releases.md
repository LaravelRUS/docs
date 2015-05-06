git 015017cc1957c677f20ce1d5c8d9cd1e71135d41

---

# Описание версий фреймворка

- [Laravel 5.0](#laravel-5.0)

<a name="laravel-5.0"></a>
## Laravel 5.0

В Laravel 5.0 изменена дефолтная структура приложения. Теперь все приложение входит в стандарт автоматической загрузки PSR-4 целиком и является более подходящей основой для построения надежного приложения. Рассмотрим основные изменения:

### Новая структура папок

Папки `app/models` больше нет. Теперь все ваши классы живут в папке `app`, и по умолчанию находятся в неймспейсе `App`. Название неймспейса хранится в файле `config/namespaces.php` и его можно изменить во всех ваших классах сразу артизан-командой `php artisan app:name`.

Контроллеры, middlewares (посредники, обработчики HTTP-запросов) и requests (новый вид классов в Laravel 5.0) сгруппрованы в папке `app/Http` как классы, относящиеся к HTTP-слою вашего приложения. Вместо файла фильтров роутов теперь во фреймворке используются middlewares, которые находятся каждый в своём файле.

Новая папка `app/Providers` является заменой папки `app/start` в предыдущих версиях Laravel 4.x. В этой папке находятся сервис-провайдеры, которые осуществляют инициализацию приложения - регистрацию классов-обработчиков ошибок, настройку логирования, загрузку файла роутов (маршрутов) и т.п. И, конечно, ваши сервис-провайдеры тоже могут находиться там.

Файлы локализаций (lang) и файлы шаблонов (views) теперь находятся в папке `resources`.

### Контракты

Все основные компоненты Laravel реализуют интерфейсы, размещенные в репозитории `illuminate/contracts`. У этого репозитория нет внешних зависимостей, это скелет фреймворка. Этот удобный корневой набор интерфейсов, который вы можете использовать в DI (dependency injection) своих классов, может служить альтернативой фасадам.

[Документация по контрактам](/docs/{{version}}/contracts).

### Кэширование роутов

Если ваше приложение использует много роутов, то для ускорения их обработки вы можете использовать artisan-команду `route:cache`. Эту команду можно применять на рабочем (продакшн) сервере после развёртывания (деплоя) приложения.

### Middleware

В Laravel 5 появились так называемые middlewares, посредники, которые выполняют роль, которая раньше возлагалась на фильтры роутов. Фильтры роутов продолжают поддерживаться, но все встроенные фильтры HTTP-запросов, как то CSRF-фильтрация, проверка аутентификации, переехали в middlewares. Свои обработчики HTTP-запроса тоже лучше писать в виде middlewares.

[Документация по middleware](/docs/{{version}}/middleware).

### DI (dependency injection) в методах контроллеров

Основной способ передачи классов для использования в вашем контроллере - указать их (type-hint) в аргументах конструктора вашего контроллера. Так как Laravel создает контроллеры и другие классы фреймворка при помощи [сервис-контейнера](/docs/{{version}}/container), он автоматически создает ожидаемые в аргументах конструктора классы и автоматически же подставляет их в вызов контроллера.

Теперь все вышеописанное работает не только для конструктора контроллера, но и для всех его методов.

	public function createPost(Request $request, PostRepository $posts)
	{
		//
	}

### Аутентификация из коробки

Laravel 5 содержит все необходимые миграции, контроллеры, модели и шаблоны для организации регистрации, аутентификации и смены пароля пользователя. Шаблоны находятся в `resources/views/auth`, валидация - `App\Services\Auth\Registrar`, контроллеры - в папке `app\Http\Controllers\Auth`. Теперь не нужно для каждого проекта писать код аутентификации, или копировать его из проекта в проект.

### События-объекты

Теперь вы можете определить событите (event) как объект:

	class PodcastWasPurchased {

		public $podcast;

		public function __construct(Podcast $podcast)
		{
			$this->podcast = $podcast;
		}

	}

Запуск события осуществляется как обычно, только теперь вместо строки-имени события можно использовать экземпляр события-объекта:

	Event::fire(new PodcastWasPurchased($podcast));

Конечно, ваш обработчик события в таком случае должен принимать объект вместо произвольной переменной `$data`:

	class ReportPodcastPurchase {

		public function handle(PodcastWasPurchased $event)
		{
			//
		}

	}

[Документация по событиям](/docs/{{version}}/events).

### Командная шина

В дополнение к задачам (job), помещаемым в очередь, которые вы использовали в Laravel 4, Laravel 5 предлагает концепцию команд, запускаемых через так называемую командную шину (command bus). Команды находятся в папке `app/Commands` и их тоже можно помещать в очередь (а можно и выполнять в текущем запросе). Вот пример команды:

	class PurchasePodcast extends Command implements SelfHandling, ShouldBeQueued {

		use SerializesModels;

		protected $user, $podcast;

		/**
		 * Создание объекта команды.
		 *
		 * @return void
		 */
		public function __construct(User $user, Podcast $podcast)
		{
			$this->user = $user;
			$this->podcast = $podcast;
		}

		/**
		 * Выполнение команды.
		 *
		 * @return void
		 */
		public function handle()
		{
			// Handle the logic to purchase the podcast...

			event(new PodcastWasPurchased($this->user, $this->podcast));
		}

	}


Базовый контроллер содержит трейт `DispatchesCommands` для выполнения команд:

	$this->dispatch(new PurchasePodcastCommand($user, $podcast));

Команды - прекрасное средство для разбивки функционала вашего приложения на изолированные части и разгрузки ваших контроллеров. 

[Документация по командной шине](/docs/{{version}}/bus)

### Реализация очереди в БД

Появился новый драйвер очереди - `database`. Теперь, если вы испытываете трудности с установкой дополнительного софта (Redis или Beanstalk) на ваш сервер, вы можете использовать таблицу MySQL для реализации очереди при помощи этого драйвера.

### Встроенный шедулер (перидический запуск команд)

In the past, developers have generated a Cron entry for each console command they wished to schedule. However, this is a headache. Your console schedule is no longer in source control, and you must SSH into your server to add the Cron entries. Let's make our lives easier. The Laravel command scheduler allows you to fluently and expressively define your command schedule within Laravel itself, and only a single Cron entry is needed on your server.

Раньше нам приходилось каждый скрипт, который должен был запускаться с определённой частотой или в определённое время, руками заносить в Cron, соединяясь с сервером по SSH, а при переезде на другой сервер - не забывать копировать его руками. С Laravel 5 жизнь стала проще. Теперь вы можете настраивать периодический запуск внутри вашего кода и хранить в системе контроля версий.

Вы только посмотрите, как это красиво:

	$schedule->command('artisan:command')->dailyAt('15:00');

[Документация по периодическому запуску команд](/docs/{{version}}/artisan#scheduling-artisan-commands).

### Tinker / Psysh

Команда `php artisan tinker` в Laravel 5 использует [Psysh](https://github.com/bobthecow/psysh) от Justin Hileman. Если вам нравился Boris в Laravel 4, вы полюбите и Psysh. Он лучше и работает под Windows! 

### DotEnv

В Laravel 5 больше нет папок для конфигов, специфичных для определённой среды выполнения. Все конфиги теперь хранятся в одной папке, а значения, специфичные среды выполнения хранятся в файле `.env`. Кроме того, название среды выполнения тоже задается в файле `.env` ! Больше никаких привязок к имени машины и флага `--env` в artisan-командах.
Laravel 5 использует библиотеку [DotEnv](https://github.com/vlucas/phpdotenv) от Vance Lucas. 

[Документация по настройке среды приложения](/docs/{{version}}/configuration#environment-configuration).

### Laravel Elixir

Laravel Elixir от Jeffrey Way - это инструмент для сборки css и js вашего приложения. Если вы слышали о Grunt или Gulp, но использование их казалось вам слишком сложным - попробуйте Elixir. Elixir представляет собой удобную надстройку над Gulp. С помощью него вы легко сможете компилировать Less, Sass или CoffeeScript. Он даже может автоматически запускать тесты за вас !

[Документация по Laravel Elixir](/docs/{{version}}/elixir).

### Laravel Socialite

Laravel Socialite - пакет, совместимый с Laravel 5.0+ для аутентификации на сайте через OAuth-провайдеры. Поддерживаются Facebook, Twitter, Google и GitHub:

	public function redirectForAuth()
	{
		return Socialize::with('twitter')->redirect();
	}

	public function getUserFromProvider()
	{
		$user = Socialize::with('twitter')->user();
	}

Больше нет нужды подбирать работающие библиотеки. Все просто работает! 

[Документация по аутентификации через соцсети](/docs/{{version}}/authentication#social-authentication).

### Облачная файловая система

Laravel 5 содержит [Flysystem](https://github.com/thephpleague/flysystem), пакет абстракции для работы с файловой системой, который поддерживает Amazon S3 и Rackspace Cloud Storage. Теперь вы можете одной строчкой в конфиге переключиться с хранения файлов на локальном диске на хранения файлов в облаке, так как функции работы с файлами не изменятся:

	Storage::put('file.txt', 'contents');

[Документация по Flysystem](/docs/{{version}}/filesystem).

### Form Requests

В Laravel 5.0 появились так называемые form requests. Это объект, который вместе с DI в методах контроллера, предлагает новый встроенный во фреймворк метод проверки и валидации пользовательского ввода. 

Например, реквест-класс формы регистрации:

	<?php namespace App\Http\Requests;

	class RegisterRequest extends FormRequest {

		public function rules()
		{
			return [
				'email' => 'required|email|unique:users',
				'password' => 'required|confirmed|min:8',
			];
		}

		public function authorize()
		{
			return true;
		}

	}

Далее вы вот так используете его в методе контроллера регистрации:

	public function register(RegisterRequest $request)
	{
		var_dump($request->input());
	}

Когда сервис-контейнер видит, что в метод контроллера подключается класс типа `FormRequest`, запускается **автоматическая валидация** пользовательского ввода по правилам, заявленным в подключаемом классе. Если валидация не проходит, произойдет автоматический редирект с передачей ошибок валидации через сессии. **Валидация форм еще никогда не была такой простой**. Узнать больше про реквест-классы можно в соответствующей главе [документации](/docs/{{version}}/validation#form-requests).

### Валидация в контроллерах

Если создавать файл form request на каждый запрос слишком накладно для вас, вы можете валидировать запрос прямо в контроллере. В Laravel 5 это стало еще проще. Теперь в базовом контроллере есть трейт `ValidatesRequests`, который предоставляет метод `validate`:

	public function createPost(Request $request)
	{
		$this->validate($request, [
			'title' => 'required|max:255',
			'body' => 'required',
		]);
	}

Вам не нужно контролировать результат валидации. Если валидация не удалась, фреймворк сам сделает редирект на предыдущую страницу, добавив в сессию сообщения об ошибках валидации и старый пользовательский ввод для отображения в форме. Или, если это был AJAX-запрос, вернёт соответствующий JSON.

[Документация по валидации в контроллерах](/docs/{{version}}/validation#controller-validation).

### Новые генераторы

У фреймворка появились новые команды генерации классов, моделей и т.п. Смотрите `php artisan list` чтобы узнать подробности.

### Кэш конфигов

You may now cache all of your configuration in a single file using the `config:cache` command.

Artisan-команда `config:cache` сливает конфиги в один файл для уменьшения количества операций чтения с диска. Рекомендуется использовать эту команду на рабочих (продакшн) серверах после развертывания (деплоя) приложения.

### VarDumper от Symfony

Хэлпер `dd` теперь использует прекрасный Symfony VarDumper, который раскрашивает дамп и позволяет схлопывать массивы. Посмотрите, как он работает:

	dd([1, 2, 3]);
