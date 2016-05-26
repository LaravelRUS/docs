git aea1779faa73dd71b969c23a4f7ebe1921227c2a

---

# Basic Task List

- [Введение](#introduction)
- [Установка](#installation)
- [Подготавливаем базу данных](#prepping-the-database)
	- [Миграции](#database-migrations)
	- [Модель Eloquent](#eloquent-models)
- [Маршрутизация](#routing)
	- [Создание маршрутов](#stubbing-the-routes)
	- [Отображение представления](#displaying-a-view)
- [Создаем структуру макета и представления](#building-layouts-and-views)
	- [Определение макет структуры](#defining-the-layout)
	- [Определение унаследованного представления](#defining-the-child-view)
- [Добавляем задачу](#adding-tasks)
	- [Проверка ввода](#validation)
	- [Создаем задачу](#creating-the-task)
	- [Отображаем задачу](#displaying-existing-tasks)
- [Удаляем задачу](#deleting-tasks)
	- [Добавляем кнопку <Удалить>](#adding-the-delete-button)
	- [Удаляем задачу](#deleting-the-task)

<a name="introduction"></a>
## Введение
В данном гайде мы познакомимся с фреймворком, узнаем о миграциях, Eloquent ORM, роутинге, валидации, шаблонах и шаблонизаторе Blade. Это неплохая стартовая точка для тех, кто не знаком с фреймворком и сам PHP знает на среднем уровне. Если вы опытный программист - лучше перейдите к [гайду для продвинутых](/docs/{{version}}/quickstart-intermediate).

В качестве реального приложения, которое мы будем изучать, выступит приложение простейшего todo-листа. Код его [доступен на GitHub](https://github.com/laravel/quickstart-basic).

<a name="installation"></a>
## Установка

#### Установка Laravel

Для установки Laravel вам нужно иметь на компьютере рабочее PHP-окружение, или воспользоваться [виртуальной машиной Homestead](/docs/{{version}}/homestead).
Фреймворк устанавливается в папку `quickstart` следующей командой Composer:

	composer create-project laravel/laravel quickstart --prefer-dist

#### Установка The Quickstart (Необязательно)

Вы можете продолжать читать этот гайд и применять его на фреймворке, а можете скачать исходный код проекта и исследовать его:

	git clone https://github.com/laravel/quickstart-basic quickstart
	cd quickstart
	composer install
	php artisan migrate

Подробнее об установке фреймворка можно прочитать в [соответствующем разделе документации](/docs/{{version}}/installation). 

<a name="prepping-the-database"></a>
## Подготавливаем базу данных

<a name="database-migrations"></a>
### Миграции баз данных

Сперва, давайте используем миграции, для того чтобы определить таблицу, для хранения всех наших задач. Миграции БД Laravel обеспечивают простой способ определения структуры таблиц и модификации с использованием PHP кода. Вместо того чтобы постоянно предупреждать вашу команду, о каких либо изменения в БД и им не приходилось вручную модифицировать их локальные копии, они могут просто запустить миграцию, которую вы отправили в вашу систему управления версиями.

Итак, давайте создадим таблицу БД, которая будет содержать все ваши задачи. [Интерфейс Artisan](/docs/{{version}}/artisan) может быть использован для создания различных классов и сэкономить вам много времени, пока вы разрабатываете свои проекты на Laravel. В нашем случае, давайте используем команду `make:migration` , для создания новой миграции БД для нашей таблицы `tasks`:

	php artisan make:migration create_tasks_table --create=tasks

Миграция будет помещена в папку `database/migrations` вашего проекта. Как вы могли заметить, команда `make:migration` добавляет метку времени, которая позволит фреймворку определить порядок применения миграций. Давайте отредактируем этот файл и добавим дополнительный столбец `string` для названия наших задач:

	<?php

	use Illuminate\Database\Schema\Blueprint;
	use Illuminate\Database\Migrations\Migration;

	class CreateTasksTable extends Migration
	{
	    /**
	     * Run the migrations.
	     *
	     * @return void
	     */
	    public function up()
	    {
	        Schema::create('tasks', function (Blueprint $table) {
	            $table->increments('id');
	            $table->string('name');
	            $table->timestamps();
	        });
	    }

	    /**
	     * Reverse the migrations.
	     *
	     * @return void
	     */
	    public function down()
	    {
	        Schema::drop('tasks');
	    }
	}

Для запуска наших миграций, мы будем использовать команду Artisan `migrate`. Если вы используете Homestead, вы должны выполнить эту команду из вашей виртуальной машины:

	php artisan migrate

Эта команда создаст все наши таблицы БД. Если вы проверите таблицы БД, вы увидите новую таблицу `tasks`, которая содержит столбцы, определенные в нашей миграции. Теперь мы готовы определить модель Eloquent ORM для наших задач!

<a name="eloquent-models"></a>
### Модель Eloquent

[ORM Eloquent](/docs/{{version}}/eloquent) - красивая и простая реализация паттерна ActiveRecord для работы с БД.
Каждая таблица имеет соответствующий класс-модель, который используется для работы с этой таблицей. Модели позволяют читать данные из таблиц и записывать данные в таблицу.

Итак, давайте определим модель `Task`, которая соответствует нашей, только что созданной, таблице `tasks`. Мы можем использовать команду Artisan для создания этой модели. В данном случае, мы будем использовать команду `make:model` :

	php artisan make:model Task

Модель будет помещена в папку `app`. По умолчанию класс модели будет пустой. Мы не должны явно указывать Eloquent какую таблицу должен привязывать к нашей модели, потому что Eloquent будет использовать в качестве названия таблицы имя класса в нижнем регистре и во множественном числе. В нашем случае Eloquent предположит, что модель `Task` хранит свои данные в таблице `tasks`. Вот как будет выглядеть наша модель:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Task extends Model
	{
		//
	}

Мы изучим больше о том как использовать модели Eloquent, когда мы добавим маршруты в наше приложении. Более подробно узнать о Eloquent можно в [документации](/docs/{{version}}/eloquent) 

<a name="routing"></a>
## Машрутизация

<a name="stubbing-the-routes"></a>
### Создание маршрутов

Итак, мы готовы добавить несколько роутов(маршрутов, routes) для нашего приложения. Роуты используются для того, чтобы указать URL-адреса для контроллеров или анонимных функций, которые должны выполняться, когда пользователь получает доступ к данной странице. По умолчанию роуты вашего приложения будут определены в файле `app/Http/routes.php`, который есть во всех новых проектах.

В нашем приложении, мы знаем что нам потребуется по крайней мере 3 роута: роут для отображения списка задач, роут для добавления новой задачи и роут для удаления существующей задачи. Мы обернем все 3 маршрута в `web` middleware (HTTP Middleware (посредники) - это фильтры обработки HTTP-запроса.), поэтому они будут иметь сессии и защиту от CSRF. Итак, давайте опишим все эти маршруты в файле `app/Http/routes.php`:

	<?php

	use App\Task;
	use Illuminate\Http\Request;

	Route::group(['middleware' => 'web'], function () {

		/**
		 * Show Task Dashboard
		 */
		Route::get('/', function () {
			//
		});

		/**
		 * Add New Task
		 */
		Route::post('/task', function (Request $request) {
			//
		});

		/**
		 * Delete Task
		 */
		Route::delete('/task/{task}', function (Task $task) {
			//
		});
	});


<a name="displaying-a-view"></a>
### Отображение представления

Views (представления, отображения, шаблоны) обычно содержат HTML-код вашего приложения и представляют собой удобный способ разделения бизнес-логики и логики отображения информации. 
Давайте заполним маршрут `/`. По этому пути, мы сделаем шаблон HTML формы, которая будет добавлять новые задачи, а также выводить список всех текущих задач.

В Laravel, все HTML шаблоны хранятся в папке `resources/views`, мы будем использовать вспомогательную функцию `view` для возврата одного из шаблонов:

	Route::get('/', function () {
		return view('tasks');
	});

Функции `view` создает экземпляр объекта представления, который соответствует шаблону resources/views/tasks.blade.php`. Теперь нам нужно создать представление.

<a name="building-layouts-and-views"></a>
## Создаем структуру макета и представления

Это приложение имеет только одну страницу, которая содержит форму для добавления новых задач, а также список всех текущих задач. Так будет выглядеть наша страница с использованием основных Bootstrap CSS стилей:

![Application Image](https://laravel.com/assets/img/quickstart/basic-overview.png)

<a name="defining-the-layout"></a>
### Определение структуры макета

Почти все веб-приложения используют одну и туже структуру. Например, наше приложение имеет меню навигации одинаковое на всех страницах(если у нас было бы их больше). Laravel позволяет легко разделяет эти общие черты на каждой странице с помощью шаблонизатора Blade.

Как мы уже говорили ранее, все шаблоны Laravel хранит в `resources/views`. Итак, давайте определим новый макет в `resources/views/layouts/app.blade.php`. Расширение `.blade.php` инструктирует Laravel использовать [Шаблонизатор Blade](/docs/{{version}}/blade) для отображения представления. Blade - простой, но мощный шаблонизатор, входящий в состав Laravel. В отличие от других шаблонизаторов, он не ограничивает вас в использовании конструкций PHP внутри шаблонов. Шаблоны Blade компилируются в PHP-код и кэшируются фреймворком - Blade не вносит дополнительных тормозов в работу фреймворка.

Наш`app.blade.php` шаблон должен выглядеть следующим образом:

    <!-- resources/views/layouts/app.blade.php -->

	<!DOCTYPE html>
	<html lang="en">
		<head>
			<title>Laravel Quickstart - Basic</title>

			<!-- CSS And JavaScript -->
		</head>

		<body>
			<div class="container">
				<nav class="navbar navbar-default">
					<!-- Navbar Contents -->
				</nav>
			</div>

			@yield('content')
		</body>
	</html>

Обратите внимание на `@yield('content')` часть шаблона. Это специальная директива, которая определяет, где будут находится дочерние шаблоны. Они будут расширять базовый шаблон, нашим собственным контентом. Давайте создадим наш собственный шаблон, который будет наследовать этот макет и содержать основной контент.

<a name="defining-the-child-view"></a>
###  Определение унаследованного представления

Нам необходимо определить, как будет выглядеть наша форма, в которой мы будем добавлять новые задачи, а также таблицу со всеми нашими существующими задачами. Давайте создадим наш шаблон в `resources/views/tasks.blade.php`.

Мы пропустим верстку нашего шаблона и сосредоточимся на более важных вещах. Напоминаем, что весь исходных код вы можете скачать по ссылке на [GitHub](https://github.com/laravel/quickstart-basic):

    <!-- resources/views/tasks.blade.php -->

	@extends('layouts.app')

	@section('content')

        <!-- Bootstrap Boilerplate... -->

		<div class="panel-body">
            <!-- Display Validation Errors -->
			@include('common.errors')

			<!-- New Task Form -->
			<form action="{{ url('task') }}" method="POST" class="form-horizontal">
				{!! csrf_field() !!}

                <!-- Task Name -->
				<div class="form-group">
					<label for="task" class="col-sm-3 control-label">Task</label>

					<div class="col-sm-6">
						<input type="text" name="name" id="task-name" class="form-control">
					</div>
				</div>

                <!-- Add Task Button -->
				<div class="form-group">
					<div class="col-sm-offset-3 col-sm-6">
						<button type="submit" class="btn btn-default">
							<i class="fa fa-plus"></i> Add Task
						</button>
					</div>
				</div>
			</form>
		</div>

		<!-- TODO: Current Tasks -->
	@endsection

#### Несколько объяснений

Прежде чем двигаться дальше, давайте рассмотрим этот шаблон. Во первых, директива `@extends` информирует шаблонизатор Blade, что мы используем шаблон определенный в `resources/views/layouts/app.blade.php`. Все содержимое между `@section('content')` и `@endsection` будет вставлено в директиву `@yield('content')` нашего шаблона `app.blade.php`.

Директива `@include('common.errors')` будет загружать шаблон расположенный в `resources/views/common/errors.blade.php`. Пока мы не создавали этот шаблон, но все еще впереди!

Мы определили внешний вид нашего приложения. Помните, что мы возвращаем это представление из нашего маршрута `/` :

	Route::get('/', function () {
		return view('tasks');
	});

Далее, мы рассмотрим как создать маршрут `POST /task`, чтобы мы могли добавить новую задачу в БД.

<a name="adding-tasks"></a>
## Добавляем задачу

<a name="validation"></a>
### Проверка ввода


Теперь, когда мы создали нашу форму, мы должны добавить маршрут для `POST /task`, чтобы проверить правильность ввода и создать новую задачу. Сперва давайте проверим правильность ввода.

Для этой формы, мы создадим поле `name`, которое будет обязательным и которое будет содержать менее `255` символов.
Если проверка вернет отрицательный результат, мы перенаправим пользователя назад на главную страницу `/`, а также добавим введенные данные в [сессии](/docs/{{version}}/session). Использование сессий  позволит сохранить введенные данные, если наша проверка вернет отрицательный результат:

	Route::post('/task', function (Request $request) {
		$validator = Validator::make($request->all(), [
			'name' => 'required|max:255',
		]);

		if ($validator->fails()) {
			return redirect('/')
				->withInput()
				->withErrors($validator);
		}

		// Create The Task...
	});

#### Переменная `$errors` 

Давайте рассмотрим часть кода `->withErrors($validator)`. Если проверка возвращает отрицательный результат, мы передаем переменную `validator` объекту переадресации `redirect` с помощью метода `withErrors`. Этот метод передает сообщения об ошибках в сессию, которые могут быть доступны с помощью переменной `$errors`.

Мы использовали директиву `@include('common.errors')` в нашем шаблоне, чтобы могли проверять на ошибки наши формы.`common.errors` позволяет нам легко выводить ошибки валидации. Давайте создадим этот шаблон:

    <!-- resources/views/common/errors.blade.php -->

    @if (count($errors) > 0)
        <!-- Form Error List -->
        <div class="alert alert-danger">
            <strong>Whoops! Something went wrong!</strong>

            <br><br>

            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif


> **Примечание:** Переменная `$errors` будет доступна для всех наших шаблонов. Если не будет проблем с проверкой, то она попросту будет пустой.

<a name="creating-the-task"></a>
### Создаем задачу

Теперь, когда мы разобрались с проверкой данных, давайте создадим новую задачу, продолжая заполнять наш маршрут. После того, как новая задача будет создана, мы будем возвращать пользователя на нашу главную страницу `/`. Для того чтобы создать новую задачу, мы будем использовать метод `save` после того, как создадим и настроим наш новый метод Eloquent:

	Route::post('/task', function (Request $request) {
		$validator = Validator::make($request->all(), [
			'name' => 'required|max:255',
		]);

		if ($validator->fails()) {
			return redirect('/')
				->withInput()
				->withErrors($validator);
		}

		$task = new Task;
		$task->name = $request->name;
		$task->save();

		return redirect('/');
	});

Отлично! Теперь мы можем добавлять задачи. Теперь, нам нужно вывести список всех существующих задач.

<a name="displaying-existing-tasks"></a>
### Отображаем задачуs

Для начала, нам надо немного изменить наш маршрут `/`, чтобы передать все существующие задачи в наш шаблон. Функция `view` принимает второй аргумент, который является массивом данных. Обращаться к массиву мы будем по ключам:

	Route::get('/', function () {
		$tasks = Task::orderBy('created_at', 'asc')->get();

		return view('tasks', [
			'tasks' => $tasks
		]);
	});

Для вывода наших задач в шаблон `tasks.blade.php`, мы будем использовать цикл. Шаблонизатор Blade, позволяет использовать простую конструкцию `@foreach`, которая будет компилироваться в простой PHP код:

	@extends('layouts.app')

	@section('content')
        <!-- Create Task Form... -->

        <!-- Current Tasks -->
        @if (count($tasks) > 0)
            <div class="panel panel-default">
                <div class="panel-heading">
                    Current Tasks
                </div>

                <div class="panel-body">
                    <table class="table table-striped task-table">

                        <!-- Table Headings -->
                        <thead>
                            <th>Task</th>
                            <th>&nbsp;</th>
                        </thead>

                        <!-- Table Body -->
                        <tbody>
                            @foreach ($tasks as $task)
                                <tr>
                                    <!-- Task Name -->
                                    <td class="table-text">
                                        <div>{{ $task->name }}</div>
                                    </td>

                                    <td>
                                        <!-- TODO: Delete Button -->
                                    </td>
                                </tr>
                            @endforeach
                        </tbody>
                    </table>
                </div>
            </div>
        @endif
	@endsection

Наше приложение почти готово. Нам осталось только добавить возможность удалять наши задачи. Давайте это сделаем!

<a name="deleting-tasks"></a>
## Удаление задач

<a name="adding-the-delete-button"></a>
### Добавляем кнопку <Удалить>

Помните, в нашем шаблоне мы оставляли комментарий <!-- TODO: Delete Button -->, чтоб не забыть добавить кнопку <удалить>? Итак, давайте отредактируем наш шаблон `tasks.blade.php` и добавим кнопку удаления для каждой задачи в нашем списке. Мы создадим небольшую форму для кнопки <удалить>. При нажатии на кнопку, будет отправлен запрос `DELETE /task` на удаление задачи:

    <tr>
        <!-- Task Name -->
        <td class="table-text">
            <div>{{ $task->name }}</div>
        </td>

        <!-- Delete Button -->
        <td>
            <form action="{{ url('task/'.$task->id) }}" method="POST">
                {!! csrf_field() !!}
                {!! method_field('DELETE') !!}

                <button type="submit" class="btn btn-danger">
                    <i class="fa fa-trash"></i> Delete
                </button>
            </form>
        </td>
    </tr>

<a name="a-note-on-method-spoofing"></a>
#### ПОДМЕНА HTTP-МЕТОДА

Обратите внимание, HTML-формы не поддерживают методы HTTP-запроса PUT или DELETE. Для того, чтобы отправить на сервер HTTP-запрос с этими методами, вам нужно добавить в форму скрытый input с именем _method.

Значение этого поля будет восприниматься фреймворком как тип HTTP-запроса. Например:

	<input type="hidden" name="_method" value="DELETE">

<a name="deleting-the-task"></a>
### Удаление задач

Наконец, давайте добавим логику для нашего маршрута, чтоб на самом деле удалить задачу. Мы можем использовать [параметры роутов](/docs/{{version}}/routing#route-model-binding) для автоматического получения Модели `Task`, которая соответствует параметру маршрута `{task}`.

Для удаления записи, мы будем использовать метод `delete`.После того, как запись будет удалена, мы будем перенаправлять пользователя обратно на URL `/`:

	Route::delete('/task/{task}', function (Task $task) {
		$task->delete();

		return redirect('/');
	});
