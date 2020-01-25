git 6f3da9260a7fc73f861074f7a6a1318e186b5c63

---

# Шаблоны Blade

- [Введение](#introduction)
- [Наследование шаблонов](#template-inheritance)
    - [Определение макета](#defining-a-layout)
    - [Наследование макета](#extending-a-layout)
- [Компоненты и слоты](#components-and-slots)
- [Отображение данных](#displaying-data)
    - [Фреймворки Blade и JavaScript](#blade-and-javascript-frameworks)
- [Управляющие конструкции](#control-structures)
    - [Оператор If](#if-statements)
    - [Оператор Switch](#switch-statements)
    - [Циклы](#loops)
    - [Переменная Loop](#the-loop-variable)
    - [Комментарии](#comments)
    - [Выполнение PHP](#php)
- [Формы](#forms)
    - [CSRF-поле](#csrf-field)
    - [Указание метода отправки формы](#method-field)
    - [Отображение ошибок валидации](#validation-errors)    
- [Включение подшаблонов](#including-sub-views)
    - [Отрисовка шаблонов для коллекций](#rendering-views-for-collections)
- [Стека](#stacks)
- [Внедрение сервисов](#service-injection)
- [Собственные директивы](#extending-blade)
    - [Собственные условные выражения](#custom-if-statements)

<a name="introduction"></a>
## Введение

Blade — простой, но мощный шаблонизатор, поставляемый с Laravel. В отличие от других популярных шаблонизаторов для PHP Blade не ограничивает вас в использовании чистого PHP-кода в ваших шаблонах. На самом деле все шаблоны Blade скомпилированы в чистый PHP-код и кешированы, пока в них нет изменений, а значит, Blade практически не нагружает ваше приложение. Файлы шаблонов Blade используют расширение `.blade.php` и обычно хранятся в директории `resources/views`.

<a name="template-inheritance"></a>
## Наследование шаблонов

<a name="defining-a-layout"></a>
### Определение макета

Два основных преимущества использования Blade — _наследование шаблонов_ и _секции_. Для начала давайте рассмотрим простой пример. Во-первых, изучим макет "главной" страницы. Поскольку многие веб-приложения используют один общий макет для разных страниц, удобно определить этот макет как один шаблон Blade:

    <!-- Хранится в resources/views/layouts/app.blade.php -->

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

Как видите, этот файл имеет типичную HTML-разметку. Но обратите внимание на директивы `@section` и `@yield`. Директива `@section`, как следует из её названия, определяет секцию содержимого, а директива `@yield` используется для отображения содержимого заданной секции.

Мы определили макет для нашего приложения, давайте определим дочернюю страницу, которая унаследует макет.

<a name="extending-a-layout"></a>
### Наследование макета

При определении дочернего шаблона используйте Blade-директиву `@extends` для указания макета, который должен быть "унаследован" дочерним шаблоном. Шаблоны, которые наследуют макет Blade, могут внедрять содержимое в секции макета с помощью директив `@section`. Запомните, как видно из приведённого выше примера, содержимое этих секций будет отображено в макете при помощи `@yield`:

    <!-- Хранится в resources/views/child.blade.php -->

    @extends('layouts.app')

    @section('title', 'Page Title')

    @section('sidebar')
        @@parent

        <p>Это дополнение к основной боковой панели.</p>
    @endsection

    @section('content')
        <p>Это содержимое тела страницы.</p>
    @endsection

В этом примере секция `sidebar` использует директиву `@@parent` для дополнения (а не перезаписи) содержимого к боковой панели макета. Директива `@@parent` будет заменена содержимым макета при отрисовке шаблона.

> {tip} В отличие от предыдущего примера (листинг кода `app.blade.php`), тут секция `sidebar` заканчивается директивой `@endsection`, а не `@show`. Директива `@endsection` служит только для определения секции, в то время как `@show` обозначает окончание секции **и немедленную отрисовку** в `yield`

Директива `yield` может принимать дефолтное значение в качестве второго аргумента. Это значение отрисовывается, если секция, которая должна располагаться на этом месте, не определена.

    @yield('content', View::make('view.name'))

Blade-шаблоны могут быть возвращены из роутов при помощи глобального хелпера `view`:

    Route::get('blade', function () {
        return view('child');
    });

<a name="components-and-slots"></a>
## Компоненты и слоты

Компоненты и слоты предоставляют аналогичные преимущества для секций и макетов; однако, некоторые могут счесть ментальную модель компонентов и слотов более простой в понимании. Во-первых, давайте представим повторно используемый компонент "оповещения", который мы хотели бы использовать повторно в нашем приложении:

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        {{ $slot }}
    </div>

Переменная `{{ $slot }}` будет соджержать контент, который мы хотим внедрить в компонент. Теперь чтобы сконструировать этот компонент мы можем использовать Blade-директиву `@component`:

    @component('alert')
        <strong>Ой!</strong> Что-то пошло не так!
    @endcomponent

Можно указать несколько имён файлов шаблонов, Laravel будет использовать тот, который найдётся первым, перебирая их в указанном порядке. Для этого служит директива `componentFirst`:

    @componentfirst(['custom.alert', 'alert'])
        <strong>Whoops!</strong> Something went wrong!
    @endcomponentfirst    

Иногда бывает полезно определить несколько слотов для компонента. Давайте модифицируем наш компонент оповещений, чтобы разрешить внедрение "заголовка". Именованные слоты могут отображаться просто путем "отражения" переменной, которая соответствует их имени:

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        <div class="alert-title">{{ $title }}</div>

        {{ $slot }}
    </div>

Теперь мы можем внедрить контент в именованный слот, используя директиву `@slot`. Любой контент, не входящий в директиву `@slot`, будет передан компоненту в переменной  `$slot`:

    @component('alert')
        @slot('title')
            Forbidden
        @endslot

        You are not allowed to access this resource!
    @endcomponent

#### Передача дополнительных данных компоненту

Иногда вам может потребоваться передать дополнительные данные компоненту. Для этой цели вы можете передать массив данных в качестве второго аргумента директиве `@component`. Все данные будут доступны для шаблона компонента как переменные:

    @component('alert', ['foo' => 'bar'])
        ...
    @endcomponent

#### Алиасы компонентов

Если компонент Blade хранится глубоко в подпапках (например, `resources/views/components/alert.blade.php`), то чтобы не писать каждый раз директиву и полный путь до шаблона (`components.alert`), вы можете определить алиас. Это можно сделать в методе `boot` сервис-провайдера `AppServiceProvider`: 

    use Illuminate\Support\Facades\Blade;

    Blade::component('components.alert', 'alert');

Теперь в шаблонах вы можете писать кратко:

    @alert(['type' => 'danger'])
        Доступ запрещён !
    @endalert

Или вы можете опустить параметры, если компонент не содержит дополнительных слотов:

    @alert
        Доступ запрещён !
    @endalert

<a name="displaying-data"></a>
## Отображение данных

Вы можете отобразить данные, переданные в ваши Blade-шаблоны, обернув переменную в фигурные скобки. Например, для такого роута:

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

Вы можете отобразить содержимое переменной `name` вот так:

    Hello, {{ $name }}.

Конечно, вы не ограничены отображением только содержимого переменных, передаваемых в шаблон. Вы также можете выводить результаты любых PHP-функций. На самом деле, вы можете поместить любой необходимый PHP-код в оператор вывода Blade:

    The current UNIX timestamp is {{ time() }}.

> {note} Blade-оператор `{{ }}` автоматически отправляется через PHP-функцию `htmlspecialchars` для предотвращения XSS-атак.

#### Вывод неэкранированных данных

По умолчанию Blade-оператор `{{ }}` автоматически отправляется через PHP-функцию `htmlspecialchars` для предотвращения XSS-атак. Если вы не хотите экранировать данные, используйте такой синтаксис:

    Hello, {!! $name !!}.

> {note} Будьте очень осторожны и экранируйте переменные, которые содержат ввод от пользователя. Всегда используйте экранирование синтаксисом с двойными скобками, чтобы предотвратить XSS-атаки при отображении предоставленных пользователем данных.

#### Рендеринг JSON

Иногда вам может понадобиться передать в шаблон JSON для инициализации джаваскрипт-переменных. Например:

    <script>
        var app = <?php echo json_encode($array); ?>;
    </script>

Вместо вызова `json_encode` можно использовать директиву `@json`. Аргументы полностью идентичны php-функции `json_encode`: 

    <script>
        var app = @json($array);

        var app = @json($array, JSON_PRETTY_PRINT);
    </script>

> {note} Вы должны использовать `@json` только для передачи существующих переменных, в которых содержится массив. Сложные выражения в аргументе могут вызывать ошибки рендеринга.

При помощи директивы `@json` удобно передавать значения из php-переменных в vue-компоненты или в `data-*` аттрибуты

    <example-component :some-prop='@json($array)'></example-component>

> {note} Для передачи аттрибутов в `@json` используйте обрамление одинарными кавычками.

#### Экранирование вывода

By default, Blade (and the Laravel `e` helper) will double encode HTML entities. If you would like to disable double encoding, call the `Blade::withoutDoubleEncoding` method from the `boot` method of your `AppServiceProvider`:

По умолчанию Blade (и хелпер `e`) экранируют вывод, переводя управляющие HTML-последовательности в безопасный текст. Чтобы запретить это поведение используйте `Blade::withoutDoubleEncoding` в методе `boot` сервис-провайдера `AppServiceProvider`.

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::withoutDoubleEncoding();
        }
    }

<a name="blade-and-javascript-frameworks"></a>
### Фреймворки Blade и JavaScript

Поскольку многие JavaScript-фреймворки тоже используют фигурные скобки для обозначения того, что данное выражение должно быть отображено в браузере, то вы можете использовать символ `@`, чтобы указать механизму отрисовки Blade, что выражение должно остаться нетронутым. Например:

    <h1>Laravel</h1>

    Hello, @{{ name }}.

В этом примере Blade удалит символ `@`, но выражение `{{ name }}` останется нетронутым, что позволит вашему JavaScript-фреймворку отрисовать его вместо Blade.

#### Директива `@verbatim`

Если вы выводите JavaScript-переменные в большой части вашего шаблона, вы можете обернуть HTML директивой `@verbatim` , тогда вам не нужно будет ставить символ `@` перед каждым оператором вывода Blade:

    @verbatim
        <div class="container">
            Hello, {{ name }}.
        </div>
    @endverbatim

<a name="control-structures"></a>
## Управляющие конструкции

В дополнение к наследованию шаблонов и отображению данных Blade предоставляет удобные сокращения для распространенных управляющих конструкций PHP, таких как условные операторы и циклы. Эти сокращения обеспечивают очень чистый и краткий способ работы с управляющими конструкциями PHP и при этом остаются очень похожими на свои PHP-прообразы.

<a name="if-statements"></a>
### Оператор If

Вы можете конструировать оператор `if` при помощи директив `@if`, `@elseif`, `@else` и `@endif`. Эти директивы работают идентично своим PHP-прообразам:

    @if (count($records) === 1)
        I have one record!
    @elseif (count($records) > 1)
        I have multiple records!
    @else
        I don't have any records!
    @endif

Для удобства Blade предоставляет и директиву `@unless`:

    @unless (Auth::check())
        You are not signed in.
    @endunless

В дополнение к обычным директивам, которые мы уже обсуждали, можно использовать и директивы `@isset` и `@empty` в виде удобных сокращений для их соответствующих PHP-функций:

    @isset($records)
        // $records is defined and is not null...
    @endisset

    @empty($records)
        // $records is "empty"...
    @endempty

#### Директивы аутентификации

Директивы `@auth` и `@guest` определяют видимость контента для залогиненного пользователя и незалогиненного.

    @auth
        // The user is authenticated...
    @endauth

    @guest
        // The user is not authenticated...
    @endguest

Вы можете указать [гвард аутентификации](/docs/{{version}}/authentication), который надо проверять при использовании директив: 

    @auth('admin')
        // The user is authenticated...
    @endauth

    @guest('admin')
        // The user is not authenticated...
    @endguest

#### Директивы работы с секциями

Узнать, содержит ли заданная секция какой-то контент (т.е. существует) можно при помощи директивы `@hasSection`

    @hasSection('navigation')
        <div class="pull-right">
            @yield('navigation')
        </div>

        <div class="clearfix"></div>
    @endif

<a name="switch-statements"></a>
### Оператор Switch

Функционал выбора отображения исходя заданных условий (switch) конструируется при помощи директив `@switch`, `@case`, `@break`, `@default` и `@endswitch`:

    @switch($i)
        @case(1)
            First case...
            @break

        @case(2)
            Second case...
            @break

        @default
            Default case...
    @endswitch

<a name="loops"></a>
### Циклы

В дополнение к условным операторам Blade предоставляет простые директивы для работы с конструкциями циклов PHP. Данные директивы тоже идентичны их PHP-прообразам:

    @for ($i = 0; $i < 10; $i++)
        Текущее значение: {{ $i }}
    @endfor

    @foreach ($users as $user)
        <p>Это пользователь {{ $user->id }}</p>
    @endforeach

    @forelse ($users as $user)
        <li>{{ $user->name }}</li>
    @empty
        <p>Нет пользователей</p>
    @endforelse

    @while (true)
        <p>Этот цикл будет длиться вечно.</p>
    @endwhile

> {tip} При работе с циклами вы можете использовать [переменную loop](#the-loop-variable) для получения полезной информации о цикле, например, находитесь ли вы на первой или последней итерации цикла.

При работе с циклами вы также можете закончить цикл или пропустить текущую итерацию:

    @foreach ($users as $user)
        @if ($user->type == 1)
            @continue
        @endif

        <li>{{ $user->name }}</li>

        @if ($user->number == 5)
            @break
        @endif
    @endforeach

Также можно включить условие в строку объявления директивы:

    @foreach ($users as $user)
        @continue($user->type == 1)

        <li>{{ $user->name }}</li>

        @break($user->number == 5)
    @endforeach

<a name="the-loop-variable"></a>
### Переменная Loop

При работе с циклами внутри цикла будет доступна переменная `$loop`. Эта переменная предоставляет доступ к некоторым полезным данным, например, текущий индекс цикла, или находитесь ли вы на первой или последней итерации цикла:

    @foreach ($users as $user)
        @if ($loop->first)
            Это первая итерация.
        @endif

        @if ($loop->last)
            Это последняя итерация.
        @endif

        <p>Это пользователь {{ $user->id }}</p>
    @endforeach

Если вы во вложенном цикле, вы можете обратиться к переменной `$loop` родительского цикла через свойство `parent`:

    @foreach ($users as $user)
        @foreach ($user->posts as $post)
            @if ($loop->parent->first)
                Это первая итерация родительского цикла.
            @endif
        @endforeach
    @endforeach

Переменная `$loop` одержит также множество других полезных свойств:

Свойство  | Описание
------------- | -------------
`$loop->index`  |  Индекс текущей итерации цикла (начинается с 0).
`$loop->iteration`  |  Текущая итерация цикла (начинается с 1).
`$loop->remaining`  |  Число оставшихся итераций цикла.
`$loop->count`  |  Общее число элементов итерируемого массива.
`$loop->first`  |  Первая ли это итерация цикла.
`$loop->last`  |  Последняя ли это итерация цикла.
`$loop->even`  |  Чётная ли это итерация цикла.
`$loop->odd`  |  Нечётная ли это итерация цикла.
`$loop->depth`  |  Уровень вложенности текущего цикла.
`$loop->parent`  |  Переменная loop родительского цикла, для вложенного цикла.

<a name="comments"></a>
### Комментарии

Blade также позволяет вам определить комментарии в ваших шаблонах. Но в отличие от HTML-комментариев, Blade-комментарии не включаются в HTML-код, возвращаемый вашим приложением:

    {{-- Этого комментария не будет в итоговом HTML --}}

<a name="php"></a>
### PHP

В некоторых случаях бывает полезно встроить PHP-код в ваши шаблоны. Вы можете использовать Blade-директиву `@php` для выполнения блока чистого PHP в вашем шаблоне:

    @php
        //
    @endphp

> {tip} Несмотря на то, что в Blade есть эта возможность, её частое использование может быть сигналом того, что у вас слишком много встроенной в шаблон логики.


<a name="forms"></a>
## Формы

<a name="csrf-field"></a>
### CSRF-поле 

В каждой форме для предотвращения CSRF-аттак должно быть поле с [CSRF токеном](https://laravel.com/docs/{{version}}/csrf). Это поле может быть сформировано при помощи директивы `csrf`:

    <form method="POST" action="/profile">
        @csrf

        ...
    </form>

<a name="method-field"></a>
### Указание метода отправки формы

Чтобы указать метод отправки формы `PUT`, `PATCH` и `DELETE` нужно, чтобы в форме было невидимое поле с именем `_method`. Это поле может быть сформировано при помощи директивы `@method`: 

    <form action="/foo/bar" method="POST">
        @method('PUT')

        ...
    </form>

<a name="validation-errors"></a>
### Отображение ошибок валидации

[Ошибки валидации](/docs/{{version}}/validation#quick-displaying-the-validation-errors) для заданного поля можно показать при помощи директивы `@error`. Название поля указывается в аргументе директивы. Текст ошибки будет содержаться в переменной `$message` внутри директивы:

    <!-- /resources/views/post/create.blade.php -->

    <label for="title">Post Title</label>

    <input id="title" type="text" class="@error('title') is-invalid @enderror">

    @error('title')
        <div class="alert alert-danger">{{ $message }}</div>
    @enderror

Вторым аргументом вы можете явно указать [имя MessageBag формы](/docs/{{version}}/validation#named-error-bags), если на странице находятся несколько форм.

    <!-- /resources/views/auth.blade.php -->

    <label for="email">Email address</label>

    <input id="email" type="email" class="@error('email', 'login') is-invalid @enderror">

    @error('email', 'login')
        <div class="alert alert-danger">{{ $message }}</div>
    @enderror

<a name="including-sub-views"></a>
## Включение подшаблонов

> {tip} Включение подшаблонов является повторение функционала компонентов, только в "старом" стиле.

Blade-директива `@include` позволяет вам включать Blade-шаблон в другой шаблон. Все переменные, доступные родительскому шаблону, будут доступны и включаемому шаблону:

    <div>
        @include('shared.errors')

        <form>
            <!-- Содержимое формы -->
        </form>
    </div>

Хотя включаемый шаблон унаследует все данные, доступные родительскому шаблону, вы также можете передать в него массив дополнительных данных:

    @include('view.name', ['some' => 'data'])

Само собой, если вы попробуете сделать `@include` шаблона, которого не существует, то Laravel выдаст ошибку. Если вы хотите включить шаблон, которого может не существовать, вам надо использовать директиву `@includeIf`:

    @includeIf('view.name', ['some' => 'data'])

Если вы хотите включить (`@include`) шаблон в зависимости от логического условия, можно использовать директиву `@includeWhen` (подключение состоится, если $boolean равно true) или директиву `@includeUnless` (подключение состоится, если $boolean равно false):

    @includeWhen($boolean, 'view.name', ['some' => 'data'])

    @includeUnless($boolean, 'view.name', ['some' => 'data'])

Директива `includeFirst` подключает первый найденный шаблон:

    @includeFirst(['custom.admin', 'admin'], ['some' => 'data'])    

> {note} Вам следует избегать использования констант `__DIR__` и `__FILE__` в ваших Blade-шаблонах, поскольку они будут ссылаться на расположение кешированных, скомпилированных шаблонов.

#### Алиасы для подключений

If your Blade includes are stored in a subdirectory, you may wish to alias them for easier access. For example, imagine a Blade include that is stored at `resources/views/includes/input.blade.php` with the following content:

Можно задать алиас для подключения шаблона, если не хотите каждый раз обращаться к нему по длинному пути. Например, у вас есть шаблон `resources/views/includes/input.blade.php` со следующим контентом:

    <input type="{{ $type ?? 'text' }}">

Можете создать алиас `input` из шаблона `includes.input` в методе `boot` сервис-провайдера `AppServiceProvider`:

    use Illuminate\Support\Facades\Blade;

    Blade::include('includes.input', 'input');

Теперь вы можете использовать этот алиас как новую директиву в шаблонах:

    @input(['type' => 'email'])

<a name="rendering-views-for-collections"></a>
### Отрисовка шаблонов для коллекций

Вы можете комбинировать циклы и включения в одной строке при помощи Blade-директивы `@each`:

    @each('view.name', $jobs, 'job')

Первый аргумент — часть шаблона, которую надо отрисовать для каждого элемента массива или коллекции. Второй аргумент — массив или коллекция для перебора, а третий — имя переменной, которое будет назначено для текущей итерации в шаблоне. Например, если вы перебираете массив `jobs`, то скорее всего захотите обращаться к каждому элементу как к переменной `job` внутри вашей части шаблона. Ключ для текущей итерации будет доступен в виде переменной `key` в вашей части шаблона.

Вы также можете передать четвёртый аргумент в директиву `@each`. Этот аргумент определяет шаблон, который будет отрисовано, если данный массив пуст.

    @each('view.name', $jobs, 'job', 'view.empty')

> {note} Переменные из родительского шаблона не передаются при использовании директивы `@each`. Используйте вместо неё `@foreach` или `@include`, если вам нужно такое поведение.

<a name="stacks"></a>
## Стеки

Blade позволяет использовать именованные стеки, которые могут быть отрисованы где-нибудь ещё в другом шаблоне или макете. Это удобно в основном для указания любых JavaScript-библиотек, требуемых для ваших дочерних шаблонов:

    @push('scripts')
        <script src="/example.js"></script>
    @endpush

"Пушить" в стек можно сколько угодно раз. Для отрисовки всего содержимого стека передайте имя стека в директиву `@stack`:

    <head>
        <!-- Head Contents -->

        @stack('scripts')
    </head>

If you would like to prepend content onto the beginning of a stack, you should use the `@prepend` directive:

Если вам нужно добавить контент в начало стека используйте директиву `@prepend`:

@push('scripts')
    Это будет в конце
@endpush

// в другом месте кода

@prepend('scripts')
    А это в начале
@endprepend

<a name="service-injection"></a>
## Внедрение сервисов

Директива `@inject` служит для извлечения сервиса из [сервис-контейнера](/docs/{{version}}/container) Laravel. Первый аргумент, передаваемый в `@inject`, это имя переменной, в которую будет помещён сервис. А второй аргумент — имя класса или интерфейса сервиса, который вы хотите извлечь:

    @inject('metrics', 'App\Services\MetricsService')

    <div>
        Месячный доход: {{ $metrics->monthlyRevenue() }}.
    </div>

<a name="extending-blade"></a>
## Собственные директивы

Blade позволяет вам определять даже свои собственные директивы с помощью метода `directive`. Когда компилятор Blade встречает пользовательскую директиву, он вызывает предоставленный обратный вызов с содержащимся в директиве выражением.

Следующий пример создаёт директиву `@datetime($var)`, которая форматирует данный `$var`, который должен быть экземпляром `DateTime`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {

        /**
         * Регистрация привязок в контейнере.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Выполнение послерегистрационной загрузки сервисов.
         *
         * @return void
         */
        public function boot()
        {
            Blade::directive('datetime', function ($expression) {
                return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
            });
        }

    }

Как видите, мы прицепили метод `format` к тому выражению, которое передаётся в директиву. Поэтому финальный PHP-код, сгенерированный этой директивой, будет таким:

    <?php echo ($var)->format('m/d/Y H:i'); ?>

> {note} После изменения логики директивы Blade вам надо удалить все кешированные шаблоны Blade. Это можно сделать Artisan-командой `view:clear`.


<a name="custom-if-statements"></a>
### Собственные условные выражения

`Blade::if` в методе `boot` сервис-провайдера `AppServiceProvider` позволяет создать свою директиву, которая управляет выводом в зависимости от аргумента, поданного ей на вход.

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }

Once the custom conditional has been defined, we can easily use it on our templates:

Теперь, задав имя новой директивы `env`, мы можем использовать в шаблонах директивы `@env`, `@elseenv` и `@unlessenv`

    @env('local')
        // Текущая среда разработки - local
    @elseenv('testing')
        // Текущая среда разработки - testing 
    @else
        // Текущая среда разработки - не local и не testing
    @endenv

    @unlessenv('production')
        // Текущая среда разработки - не production
    @endenv