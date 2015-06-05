git f3ce27ecc71d9329f1eaede45989ee80d4cabcae

---

# Шаблонизатор Blade

- [Введение](#introduction)
- [Наследование шаблонов](#template-inheritance)
	- [Определение лейаута](#defining-a-layout)
	- [Расширение лейаута](#extending-a-layout)
- [Отображение данных](#displaying-data)
- [Условия и циклы](#control-structures)
- [Внедрение классов](#service-injection)
- [Расширение Blade](#extending-blade)

<a name="introduction"></a>
## Шаблоны Blade

Blade - простой, но мощный шаблонизатор, входящий в состав Laravel. В отличие от других шаблонизаторов, он не ограничивает вас в использовании конструкций PHP внутри шаблонов. Шаблоны Blade компилируются в PHP-код и кэшируются фреймворком - Blade не вносит дополнительных тормозов в работу фреймворка.

Файлы шаблоны Blade оканчиваются на `.blade.php` и обычно находятся в папке `resources/views`.

<a name="template-inheritance"></a>
## Наследование шаблонов

<a name="defining-a-layout"></a>
### Определение лейаута

Two of the primary benefits of using Blade are _template inheritance_ and _sections_. To get started, let's take a look at a simple example. First, we will examine a "master" page layout. Since most web applications maintain the same general layout across various pages, it's convenient to define this layout as a single Blade view:

Два основных преимущества Blade - это _наследование шаблонов_ и _секции_. Чтобы было понятнее, давайте рассмотрим простой пример. Обычно, все веб-приложения имеют базовый шаблон - он же лейаут, макет. В нем происходит подключение css и js, задается базовая верстка и в определенных местах подключаются такие части как хедер (шапка), футер, сайдбар и т.п. Вот он в виде шаблона Blade:

	<!-- Файл resources/views/layouts/master.blade.php -->

	<html>
		<head>
			<title>App Name - @yield('title')</title>
		</head>
		<body>
			@section('sidebar')
				This is the master sidebar.
			@show

			<div class="container">
				@yield('content')
			</div>
		</body>
	</html>

Как вы можете видеть, это обычный HTML с директивами, расставленными в определённых местах. Директива `@section` определяет некоторую секцию контента. Директива `@yield` используется для отображения в заданном месте контента секции с заданным именем.

Хорошо, лейаут у нас есть, давайте теперь посмотрим, что должна представлять из себя дочерняя страница.

<a name="extending-a-layout"></a>
### Расширение лейаута

В контроллерах или роутах мы вызываем именно дочерние страницы (вьюхи), а они уже собирают "снизу вверх" (от себя к лейауту) HTML страницы.

Чтобы показать, какой именно из лейаутов (их у нас в приложении может быть несколько) мы будем использовать, мы должны использовать директиву `@extends`:

	<!-- Файл resources/views/layouts/child.blade.php -->

	@extends('layouts.master')

	@section('title', 'Page Title')

	@section('sidebar')
		@@parent

		<p>This is appended to the master sidebar.</p>
	@endsection

	@section('content')
		<p>This is my body content.</p>
	@endsection

В дочерней странице мы задаем секции, которые будем использовать в лейауте. Обратите внимание, что секция `sidebar` использует директиву `@@parent`, что позволяет не перезаписать секцию `sidebar`, определённую в лейауте, а добавить контент к ней. 

И, как было указано выше, мы обращаемся к дочерней странице при помощи стандартного хелпера `view()`:

	Route::get('blade', function () {
		return view('child');
	});


<a name="displaying-data"></a>
## Отображение данных

Для вывода переменной в шаблоне Blade нужно обернуть её в конструкцию `{{ }}`:

Передача переменной в шаблон:

	Route::get('greeting', function () {
		return view('welcome', ['name' => 'Samantha']);
	});

Отображение переменной:

	Hello, {{ $name }}.

Внутри фигурных скобок вы можете использовать любую PHP-конструкцию, в том числе и вызов функции:

	The current UNIX timestamp is {{ time() }}.

> **Примечание:** Конструкция `{{ }}` автоматически применяет к выводу PHP-функцию `htmlentities` для предотвращения XSS-атак.

#### Blade & javascript-фреймворки

Многие javascript-фреймворки используют фигурные скобки в своих целях. Для того, чтобы запретить Blade обрабатывать некоторые конструкции с фигурными скобками, поставьте перед ними символ `@`:

	<h1>Laravel</h1>

	Hello, @{{ name }}.

Здесь символ `@` будет удален шаблонизатором при обработке этого файла, а строка `{{ name }}` останется нетронутой и перейдет в HTML, чтобы быть обработанной вашим javascript-фреймворком.

#### Вывод данных с проверкой на их существование

Иногда вам нужно вывести переменную, которая, возможно, не определена в шаблоне. Чтобы не получить эксепшн "Переменная не определена", обычно вы делаете следующее:

	{{ isset($name) ? $name : 'Default' }}

Но вместо тернарного оператора вы можете писать так:

	{{ $name or 'Default' }}

Если переменная $name не определена, будет выведена строка `Default`.

#### Вывод неэкранированного контента

By default, Blade `{{ }}` statements are automatically send through PHP's `htmlentities` function to prevent XSS attacks. If you do not want your data to be escaped, you may use the following syntax:

По умолчанию конструкция `{{ }}` прменяет к содержимому PHP-функцию `htmlentities`, заменяя исполняемые html-тэги типа `<script>` и т.п. на неисполняемые строки. Чтобы запретить это поведение и вывести текст без экранирования, используйте конструкцию `{!! !!}`:

	Hello, {!! $name !!}.

> **Примечание:** Будьте очень внимательны и осторожны, когда выводите таким образом контент. Для контента, который могли редактировать пользователи по возможности всегда используйте `{{ }}`.

<a name="control-structures"></a>
## Условия и циклы

Для управления выводом контента вы можете воспользоваться PHP-конструкциями типа `if()` и `foreach()`, а можете воспользоваться директивами Blade, которые в некоторых случаях могут быть удобнее:

#### Условия

Вы можете реализовать условия при помощи директив `@if`, `@elseif`, `@else` и `@endif`:

	@if (count($records) === 1)
		I have one record!
	@elseif (count($records) > 1)
		I have multiple records!
	@else
		I don't have any records!
	@endif

Кроме того, у Blade есть еще директива `@unless`:

	@unless (Auth::check())
		You are not signed in.
	@endunless

#### Циклы

Директивы циклов также очень похожи на PHPшные:

	@for ($i = 0; $i < 10; $i++)
		The current value is {{ $i }}
	@endfor

	@foreach ($users as $user)
		<p>This is user {{ $user->id }}</p>
	@endforeach

	@forelse ($users as $user)
		<li>{{ $user->name }}</li>
	@empty
		<p>No users</p>
	@endforelse

	@while (true)
		<p>I'm looping forever.</p>
	@endwhile

#### Включение страниц

Директива `@include` позволит вам добавить на страницу контент другой страницы без использования секций и `@yield`. Обратите внимание, что при `@include` вам не нужно передавать переменные явным образом - все определённые в данном месте переменные будут доступны в подключаемой вьюхе ! Волшебно, не правда ли ? 

	<div>
		@include('shared.errors')

		<form>
			<!-- Form Contents -->
		</form>
	</div>

Но, если хотите - например, если в подключаемом шаблоне другой формат именования переменных - вы можете передать переменные туда явным образом:

	@include('view.name', ['some_data' => $someData])

#### Комментарии

Комментарии в Blade задаются следующим образом:

	{{-- This comment will not be present in the rendered HTML --}}

Казалось бы, зачем здесь-то юзать шаблонизатор, если можно просто написать `<!-- -->` ? Но фишка в том, в отличие от HTML-комментариев, Blade-комментарии никогда не попадут в результирующий HTML.

<a name="service-injection"></a>
## Внедрение классов

Вы можете вставить в шаблон любой класс, который определён в [сервис-контейнере](/docs/{{version}}/container) фреймворка. Для этого используется директива `@inject`, первый аргумент которой - название переменной, в которую будет помещён экземпляр класса, а второй - название внедряемого класса или интерфейса: 

	@inject('metrics', 'App\Services\MetricsService')

	<div>
		Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
	</div>

<a name="extending-blade"></a>
## Расширение Blade

Blade even allows you to define your own custom directives. You can use the `directive` method to register a directive. When the Blade compiler encounters the directive, it calls the provided callback with its parameter.

Blade позволяет создавать свои директивы. Для этого нужно использовать `Blade::directivr()` в методе boot() сервис-провайдера. В аргументах - название директивы и функция-замыкание, которая возвращает конструкцию для вставки в шаблон.

Например, создадим директиву `@datetime($var)`, которая будет форматировать дату (объест Carbon) определённым, нужным нам образом.

	<?php namespace App\Providers;

	use Blade;
	use Illuminate\Support\ServiceProvider;

	class AppServiceProvider extends ServiceProvider
	{
		/**
		 * Perform post-registration booting of services.
		 *
		 * @return void
		 */
		public function boot()
		{
			Blade::directive('datetime', function($expression) {
				return "<?php echo with{$expression}->format('m/d/Y H:i'); ?>";
			});
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

Как вы можете видеть, здесь используется хелпер Laravel `with()`. Он принимает объект в аргументах и предоставляет возможность строить цепочки из результатов.

В шаблон будет вставлена следующая конструкция:

	<?php echo with($var)->format('m/d/Y H:i'); ?>
