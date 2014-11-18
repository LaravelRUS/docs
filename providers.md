git 6891fd2c068c98022b89a7dbc728e6681a5bfd87

---

# Service Providers

- [Введение](#introduction)
- [Использование провайдеров](#basic-provider-example)
- [Регистрация провайдеров](#registering-providers)
- [Отложенные провайдеры](#deferred-providers)
- [Команда создания провайдера](#generating-service-providers)

<a name="introduction"></a>
## Введение

((Service providers)) (сервис-провайдеры, дословно - "поставщики услуг") занимают центральное место в архитектуре Laravel. Они предназначены для первоначальной загрузки (bootstraping) приложения. Ваше приложение, а также сервисы самого фреймворка загружаются через сервис-провайдеры.

Что конкретно означает термин "первоначальная загрузка" или "bootsraping" ? Главным образом это **регистрация** некоторых вещей - таких как биндинги в IoC-контейнер (фасадов и т.д.), слушателей событий (event listeners), фильтров роутов (route filters) и самих роутов (routes). Сервис-провайдеры - центральное место для конфигурирования вашего приложения. 

Если вы откроете файл `config/app.php`, вы увидите массив `providers`. В нем перечислены все классы сервис-провайдеров, которые загружаются во время старта вашего приложения (конечно, кроме тех, которые  являются "отложенными" (deferred), т.е. загружаются по требованию другого сервис-провайдера).

Можно и нужно создавать свои собственные сервис-провайдеры для загрузки и настройки различных частей своего приложения.

<a name="basic-provider-example"></a>
## Использование провайдеров

Сервис-провайдеры должны расширять (extends) класс `Illuminate\Support\ServiceProvider`. Это абстрактный класс, который требует, чтобы в наследуемом классе был метод `register()`. В методе `register()` вы можете **только** регистрировать свои классы (bindings) в [сервис-контейнере](/docs/master/container), слушателей событий (event listeners), роуты и фильтры роутов там регистрировать **нельзя**.

### Метод register()

Вот так может выглядеть простейший сервис-провайдер:

	<?php namespace App\Providers;

	use Riak\Connection;
	use Illuminate\Support\ServiceProvider;

	class RiakServiceProvider extends ServiceProvider {

		/**
		 * Register bindings in the container.
		 *
		 * @return void
		 */
		public function register()
		{
			$this->app->singleton('Riak\Contracts\Connection', function($app)
			{
				return new Connection($app['config']['riak']);
			});
		}

	}

В `register()` мы регистрируем (bind) как singleton (т.е. класс не будет переинициализироваться после вызова из контейнера) в сервис-контейнере класс работы с базой данных Riak. Если для вас этот код выглядит абракадаброй, не беспокойтесь, работу [сервис-контейнера мы рассмотрим позже](/docs/master/container).

Неймспейс `App\Providers` , в котором находится этот класс сервис-провайдера - дефолтное место для хранения сервис-провайдеров вашего Laravel-приложения, но вы можете располагать свои сервис-провайдеры где угодно внутри вашей PSR-4 папки (если вы не меняли `composer.json`, то это папка `app`).

### Метод boot()

Когда вызвались методы `register()` всех сервис-провайдеров приложения, вызывается метод `boot()` сервис-провайдеров. Там уже можно использовать весь существующий функционал классов фреймворка и вашего приложения - регистрировать слушателей событий, подключать роуты и т.п. . 

	<?php namespace App\Providers;

	use Illuminate\Support\ServiceProvider;
	use Illuminate\Contracts\Events\Dispatcher;

	class EventServiceProvider extends ServiceProvider {

		/**
		 * Perform post-registration booting of services.
		 *
		 * @param  Dispatcher  $events
		 * @return void
		 */
		public function boot(Dispatcher $events)
		{
			$events->listen('SomeEvent', 'SomeEventHandler');
		}

		/**
		 * Register bindings in the container.
		 *
		 * @return void
		 */
		public function register()
		{
			//
		}

	}

Обратите внимание, что сервис-контейнер, вызывая метод `boot()`, сам проинжектит те зависимости, которые вы зададите, в частности, Dispatcher.

<a name="registering-providers"></a>
## Регистрация провайдеров

Все сервис-провайдеры регистрируются в файле `config/app.php` путем добавления в массив `providers`. Все сервис-провайдеры фреймворка находятся там. 

Чтобы зарегистрировать свой сервис-провайдер, добавьте название класса в этот массив:

	'providers' => [
		'App\Providers\EventServiceProvider',

		// другие сервис-провайдеры
	],

<a name="deferred-providers"></a>
## Отложенные провайдеры

Если ваш провайдер **только** регистрирует (bind) классы в [сервис-контейнере](/docs/master/container), то вы можете отложить вызов его метода `register()` до момента, когда эти классы будут затребованы из сервис-контейнера. Это позволит не дергать файловую систему каждый запрос в попытках загрузить файл с нужным классом с диска.

Для того, чтобы сделать сервис-провайдер отложенным, установите свойство `defer` в `true` и определить метод `provides()` , чтобы фреймворк знал, какие классы биндятся (регистрируются в сервис-контейнере, "связываются") в вашем провайдере.

	<?php namespace App\Providers;

	use Riak\Connection;
	use Illuminate\Support\ServiceProvider;

	class RiakServiceProvider extends ServiceProvider {

		/**
		 * Indicates if loading of the provider is deferred.
		 *
		 * @var bool
		 */
		protected $defer = true;

		/**
		 * Register the service provider.
		 *
		 * @return void
		 */
		public function register()
		{
			$this->app->singleton('Riak\Contracts\Connection', function($app)
			{
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
			return ['Riak\Contracts\Connection'];
		}

	}

Когда в процессе работы приложению понадобится класс `Riak\Contracts\Connection`, он вызовет метод `register()` сервис провайдера `RiakServiceProvider`.

Laravel в процессе запуска собирает данные об отложенных сервис-провайдерах и классах, которые ими биндятся - и держит их в файле `storage/meta/services.json`

<a name="generating-service-providers"></a>
## Команда создания провайдера

Сервис-провайдер можно создать artisan-командой `make:provider` :

	php artisan make:provider "App\Providers\RiakServiceProvider"



