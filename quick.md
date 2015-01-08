git 8f9202241c9fff4d46b54ce2e48f6899ad7bb93a

---

# Быстрый старт

- [Установка](#installation)
- [Маршрутизация](#routing)
- [Создаём шаблон](#creating-a-view)
- [Создаём миграцию](#creating-a-migration)
- [Eloquent ORM](#eloquent-orm)
- [Отображаем данные](#displaying-data)

<a name="installation"></a>
## Установка

#### При помощи установщика Laravel

Поставьте при помощи composer установщик Laravel, лучше глобально:

	composer global require "laravel/installer=~1.1"

Проверьте, чтобы `~/.composer/vendor/bin` (`C:\Users\username\AppData\Roaming\Composer\vendor\bin` в случае Windows) был добавлен в PATH.

После установки, выполните в нужной папке кв командной строке команду `laravel new`. Например, `laravel new blog` создаст в подпапке `blog` приложение laravel, со всеми установленными зависимостями.


#### При помощи Сomposer

Laravel использует Composer для установки пакетов-зависимостей. Если он еще у вас не стоит, [установите его](http://getcomposer.org/doc/00-intro.md).

Чтобы установить Laravel вам нужно выполнить эту команду в командной строке:

	composer create-project laravel/laravel your-project-name --prefer-dist

Либо можно скачать [архив хранилища](https://github.com/laravel/laravel/archive/master.zip) с GitHub. Дальше, [установите Composer](http://getcomposer.org), запустите `composer install` в корневой папке вашего проекта. Она загрузит и установит зависимости фреймворка.

### Разрешение на запись

После установки вам надо открыть для записи папку `app/storage` со всеми подпапками. См. секцию [Установка](/docs/installation).

### Запуск веб-сервера

Обычно для запуска приложения Laravel нужно, чтобы в системе стоял веб-сервер - Apache или Nginx. Но если вы работаете с PHP версии 5.4 или выше , вы можете запустить встроенный в PHP веб-сервер следующей командой:

	php artisan serve

<a name="directories"></a>
### Структура каталогов

После установки изучите структуру папок. Папка `app` одержит подпапки, такие как `views`, `controllers` и `models`. Большая часть кода вашего приложения будет находиться где-то внутри них. Вы также можете посмотреть на содержимое `app/config` и на настройки, которые доступны для изменения.

<a name="local-development-environment"></a>
## Разработка на локальной машине

Для разработки необходимо установить на локальную машину массу дополнительного софта - веб-сервер, php нужной версии и с нужными расширениями, mysql, и т.д. и т.п. Это обычно довольно хлопотное занятие. Но есть путь проще - [Laravel Homestead](/docs/homestead). Это виртуальная машина, которая содержит Ubuntu 14.04 с предустановленным необходимым софтом, а именно:

- Nginx
- PHP 5.6
- MySQL
- Redis
- Memcached
- Beanstalk

<a name="routing"></a>
## Маршрутизация

Для начала давайте создадим наш первый маршрут (роут, route). В Laravel самый простой маршрут - функция-замыкание (анонимная функция, Closure). Откройте файл `app/routes.php` и добавьте этот код в его конец:

	Route::get('users', function()
	{
		return 'Users!';
	});

Теперь если вы перейдёте в браузере на адрес `/users` то должны увидеть текст `Users!`. Отлично! Вы только что создали свой первый маршрут.

Маршруты также могут быть привязаны к классу контроллера. Например:

	Route::get('users', 'UserController@getIndex');

Этот маршрут сообщает Laravel, что запросы к `/users` должны вызывать метод `getIndex` класса `UserController`. Для дополнительной информации см. [раздел о контроллерах](/docs/controllers).

<a name="creating-a-view"></a>
## Создаём шаблон

Давайте теперь создадим вид или вьюху (view, шаблон), чтобы показывать информацию о наших пользователях. Вьюхи находятся в `app/views` и содержат HTML-код вашего приложения. Мы создадим два новых вида в этой папке: `layout.blade.php` и `users.blade.php`. Начнём с `layout.blade.php`:

	<html>
		<body>
			<h1>Laravel Quickstart</h1>

			@yield('content')
		</body>
	</html>

Теперь создадим `users.blade.php`:

	@extends('layout')

	@section('content')
		Users!
	@stop

Кое-что из этого кода, возможно, выглядит для вас весьма странно. Это из-за того, что мы используем шаблонизатор Laravel - Blade. Blade очень быстр благодаря тому, что это всего лишь набор регулярных выражений, которые преобразуют ваши вьюхи в чистый код на PHP. Blade позволяет наследовать вьюхи, а также добавляет "синтаксический сахар" к таким частоиспользуемым PHP-конструкциям, как `if` и `for`. Для инфромации см. [раздел о Blade](/docs/templates).

Теперь, когда у нас есть вьюхи, давайте используем их в нашем маршруте `/users`. Вместо возврата простой строки `Users!` мы вернём экземпляр шаблона:

	Route::get('users', function()
	{
		return View::make('users');
	});

Замечательно! Вы создали шаблон маршрута, который наследует разметку страницы (шаблон layout). А теперь перейдём к работе с базой данных.

<a name="creating-a-migration"></a>
## Создаём миграцию

Для создания таблицы для хранения наших данных мы используем систему миграций Laravel. Миграции позволяют вам определять изменения в БД, используя выразительный синтаксис, а затем легко делиться ими с остальными членами вашей команды.

Для начала настроим соединение с БД. Все соединения настраиваются в файле `app/config/database.php`. По умолчанию Laravel использует MySQL, и для работы вам нужно указать здесь параметры подключения. Если хотите, можете установить параметр `driver` в значение `sqlite` и Laravel будет использовать базу данных SQLite, которая находится в папке `app/database`.


Для создания миграции мы воспользуемся утилитой командной строки [Artisan](/docs/artisan). Выполните следующую команду в корневой папке вашего проекта:

	php artisan migrate:make create_users_table

Теперь найдите созданный файл миграции в папке `app/database/migrations`. Он содержит класс с двумя методами: `up` и `down`. В методе `up` вам нужно произвести требуемые изменения в таблицах, а в методе `down` вам нужно их откатить.

Давайте создадим такую миграцию:

	public function up()
	{
		Schema::create('users', function($table)
		{
			$table->increments('id');
			$table->string('email')->unique();
			$table->string('name');
			$table->timestamps();
		});
	}

	public function down()
	{
		Schema::drop('users');
	}

Теперь мы можем применить миграцию через командную строку, используя команду `migrate`. Просто выполните её в корне проекта:

	php artisan migrate

Если вам нужно откатить миграцию - выполните команду `migrate:rollback`. 

Теперь, когда у нас есть таблица, начнём загружать данные.

<a name="eloquent-orm"></a>
## Eloquent ORM

Laravel поставляется с замечательной ORM (механизм связывания записей БД с объектами PHP - прим. пер.) - Eloquent. Если вы программировали в библиотеке Ruby on Rails, то она покажется вам знакомой, так как Eloquent следует принципам ActiveRecord при взаимодействии с базами данных.

Для начала создадим модель. В Eloquent модель используется для запросов к соответствующей таблице в БД, а также представляет отдельную запись (record) внутри неё. Не беспокойтесь, скоро это станет куда понятнее! Модели обычно хранятся в папке `app/models`. Создадим файл `User.php` с таким кодом:

	class User extends Eloquent {}

Заметьте, что нам не нужно указывать, какую таблицу нужно использовать. Eloquent следует множеству соглашений, одно из которых - то, что имя таблицы соответствует множественному числу имени класса её модели (user -> users - прим. пер.). Это очень удобно!

Добавьте новые записи в таблицу `users`, используя ваш любимый инструмент для работы с БД, и мы посмотрим, как Eloquent позволяет их получить, а затем передать в шаблон.

Итак, изменим наш маршрут `/users`:

	Route::get('users', function()
	{
		$users = User::all();

		return View::make('users')->with('users', $users);
	});

Посмотрим, что здесь происходит. Сперва мы получаем все записи в таблице 'users' через метод 'all' модели 'User'. Дальше мы передаём эти записи шаблону через его метод 'with'. Этот метод принимает имя переменной и её значение и таким образом делает данные доступными внутри своего кода.

Отлично. Теперь мы готовы к тому, чтобы показать пользователей в нашем шаблоне!

<a name="displaying-data"></a>
## Отображаем данные 

Теперь, когда мы сделали переменную 'users' доступной для нашего шаблона мы можем отобразить её таким образом:

	@extends('layout')

	@section('content')
		@foreach($users as $user)
			<p>{{ $user->name }}</p>
		@endforeach
	@stop

> **Примечание:** Код выше открыт для XSS-атак. Blade, как и простой код на PHP, не экранирует вывод, поэтому вам нужно следить, чтобы выводимые строки содержали экранированый HTML. 

Вам может быть интересно, куда подевались вызовы `echo`. Blade позволяет вам выводить строки, обрамляя их двойными фигурными скобками ({{ ... }}). Теперь вы можете перейти в браузере к своему маршруту и увидеть имена всех имеющихся пользователей.

Это только начало. В этом руководстве вы ознакомились с самыми основами Laravel, но у него есть ещё очень много интересных вещей, которые вам стоит узнать. Продолжайте читать документацию и глубже узнавать возможности, предоставляемые [Eloquent](/docs/eloquent) и [Blade](/docs/templates). А может вам больше интересны [очереди](/docs/queues) и [юнит-тесты](/docs/testing). Или же вам хочется размять мускулы с [контейнером IoC](/docs/ioc). Выбор за вами!

<a name="deploying-your-application"></a>
## Деплой (загрузка на хостинг)

One of Laravel's goals is to make PHP application development enjoyable from download to deploy, and [Laravel Forge](https://forge.laravel.com) provides a simple way to deploy your Laravel applications onto blazing fast servers. Forge can configure and provision servers on DigitalOcean, Linode, Rackspace, and Amazon EC2. Like Homestead, all of the latest goodies are included: Nginx, PHP 5.6, MySQL, Postgres, Redis, Memcached, and more. Forge "Quick Deploy" can even deploy your code for you each time you push changes out to GitHub or Bitbucket!

On top of that, Forge can help you configure queue workers, SSL, Cron jobs, sub-domains, and more. For more information, visit the [Forge website](https://forge.laravel.com).