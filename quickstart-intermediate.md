git 363af42789f8de1eb4df1384cebe78fe5e428f86

---

# Руководство для продвинутых

- [Введение](#introduction)
- [Установка](#installation)
- [Подготовка базы данных](#prepping-the-database)
    - [Миграции](#database-migrations)
    - [Модель Eloquent](#eloquent-models)
    - [Отношения Eloquent](#eloquent-relationships)
- [Маршрутизация](#routing)
    - [Отображение представления](#displaying-a-view)
    - [Аутентификация](#authentication-routing)
    - [Контроллер Task](#the-task-controller)
- [Создание структуры макета и представления](#building-layouts-and-views)
    - [Определение макета структуры](#defining-the-layout)
    - [Определение унаследованного представления](#defining-the-child-view)
- [Добавление задачи](#adding-tasks)
    - [Проверка ввода](#validation)
    - [Создание задачи](#creating-the-task)
- [Отображение задачи](#displaying-existing-tasks)
    - [Внедрение зависимостей](#dependency-injection)
    - [Отображение задачи](#displaying-the-tasks)
- [Удаление задачи](#deleting-tasks)
    - [Добавление кнопки <Удалить>](#adding-the-delete-button)
    - [Привязка маршрута к модели](#route-model-binding)
    - [Авторизация](#authorization)
    - [Удаление задачи](#deleting-the-task)

<a name="introduction"></a>
## Введение

В данном руководстве мы познакомимся с фреймворком, узнаем о миграциях, Eloquent ORM, роутинге, аутентификации, авторизации, внедрении зависимостей, валидации, шаблонах и шаблонизаторе Blade. Это неплохая стартовая точка для тех, кто уже знаком с основами Laravel или PHP-фреймворками вообще.

В качестве реального приложения, которое мы будем изучать, выступит приложение простейшего todo-листа. В отличие от базового руководства, в данном пользователи смогут создавать учетные записи и аутентифицироваться в приложении. Код его [доступен на GitHub](https://github.com/laravel/quickstart-intermediate).

<a name="installation"></a>
## Установка

#### Установка Laravel

Для установки Laravel вам нужно иметь на компьютере рабочее PHP-окружение, или воспользоваться [виртуальной машиной Homestead](/docs/{{version}}/homestead).
Фреймворк устанавливается в папку `quickstart` следующей командой Composer:

    composer create-project laravel/laravel quickstart --prefer-dist

#### Установка The Quickstart (Необязательно)

Вы можете продолжать читать это руководство и применять его на фреймворке, а можете скачать исходный код проекта и исследовать его:

    git clone https://github.com/laravel/quickstart-intermediate quickstart
    cd quickstart
    composer install
    php artisan migrate

Подробнее об установке фреймворка можно прочитать в [соответствующем разделе документации](/docs/{{version}}/installation).

<a name="prepping-the-database"></a>
## Подготовка базы данных

<a name="database-migrations"></a>
### Миграции баз данных

Сперва, давайте используем миграции, для того чтобы определить таблицу, предназначенную для хранения всех наших задач. Миграции БД Laravel обеспечивают простой способ определения структуры таблиц и модификации с использованием PHP кода. Вместо того чтобы постоянно предупреждать вашу команду, о каких либо изменения в БД и им не приходилось вручную модифицировать их локальные копии, они могут просто запустить миграцию, которую вы отправили в вашу систему управления версиями.

#### Таблица `users`

Так как мы собираемся позволить пользователям создавать учетные записи в приложении, нам необходима таблица для хранения всех пользователей. К счастью, Laravel уже поставляется с миграцией для создания базовой таблицы `users`, таким образом вручную генерировать ее не нужно. Миграция по умолчанию для таблицы `users` находится в директории `database/migrations`.

#### Таблица `tasks`

Итак, давайте создадим таблицу БД, которая будет содержать все ваши задачи. [Интерфейс Artisan](/docs/{{version}}/artisan) может быть использован для создания различных классов и сэкономить вам много времени, пока вы разрабатываете свои проекты на Laravel. В нашем случае, давайте используем команду `make:migration` , для создания новой миграции БД для нашей таблицы `tasks`:

    php artisan make:migration create_tasks_table --create=tasks

Миграция будет помещена в папку `database/migrations` вашего приложения. Как вы могли заметить, команда `make:migration` уже добавила автоинкриментный идентификатор и временные метки к файлу миграции. Давайте отредактируем этот файл и добавим дополнительный столбец `string` для названия наших задач, а так же столбец `user_id` чтобы связать таблицы `tasks` и `users`:

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
                $table->integer('user_id')->index();
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

Эта команда создаст все наши таблицы БД. Если вы проверите таблицы БД, вы увидите новые таблицы `tasks` и `users` содержащие все определенные нашей миграцией столбцы. Теперь мы готовы определить модели Eloquent ORM!

<a name="eloquent-models"></a>
### Модели Eloquent

[ORM Eloquent](/docs/{{version}}/eloquent) - красивая и простая реализация паттерна ActiveRecord для работы с БД.
Каждая таблица имеет соответствующий класс-модель, который используется для работы с этой таблицей. Модели позволяют читать данные из таблиц и записывать данные в таблицу.

#### Модель `User`

В первую очередь нам нужна модель, соответствующая таблице `users` базы данных. Однако, если вы посмотрите в директорию `app` вашего приложения, вы увидите, что Laravel уже поставляется с готовой моделью `User`, так что нам не нужно генерировать ее вручную.

#### Модель `Task`

Итак, давайте определим модель `Task`, которая соответствует нашей, только что созданной, таблице `tasks`. Мы можем использовать команду Artisan для создания этой модели. В данном случае, мы будем использовать команду `make:model` :

    php artisan make:model Task

Модель будет помещена в папку `app`. По умолчанию класс модели будет пустой. Мы не должны явно указывать Eloquent какую таблицу привязывать к нашей модели, потому что Eloquent будет использовать в качестве названия таблицы имя класса в нижнем регистре и во множественном числе. В нашем случае Eloquent предположит, что модель `Task` хранит свои данные в таблице `tasks`.

Давайте немного дополним нашу модель. Во-первых декларируем, что атрибут `name` модели должен быть массово присваиваемым. Это позволит нам заполнить атрибут `name` используя метод Eloquent `create`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Task extends Model
    {
        /**
         * The attributes that are mass assignable.
         *
         * @var array
         */
        protected $fillable = ['name'];
    }

Мы изучим больше о том как использовать модели Eloquent, когда мы добавим маршруты в наше приложении. Более подробно узнать о Eloquent можно в [документации](/docs/{{version}}/eloquent)

<a name="eloquent-relationships"></a>
### Отношения Eloquent

Теперь, когда наши модели определены, нам нужно связать их. Например, наш `User` может иметь много объектов `Task`, в то время как `Task` относится к единственному объекту `User`. Определение отношений позволит нам свободно использовать связи в нашем коде, к примеру так:

    $user = App\User::find(1);

    foreach ($user->tasks as $task) {
        echo $task->name;
    }

#### Отношение `tasks`

Сначала декларируем, что `tasks` относится к модели `User`. Отношения Eloquent определяются с помощью методов в модели. Eloquent поддерживает несколько различных типов отношений, по этому не забудьте заглянуть в [документацию по отношениям Eloquent](/docs/{{version}}/eloquent-relationships). В нашем случае, мы определим метод `tasks` в модели `User`, который вызывает предоставляемый Eloquent метод `hasMany`:

    <?php

    namespace App;

    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        // Other Eloquent Properties...

        /**
         * Get all of the tasks for the user.
         */
        public function tasks()
        {
            return $this->hasMany(Task::class);
        }
    }

#### Отношение `user`

Далее, определим что `user` относится к `Task` модели. Опять же, с помощью методов модели. Используем метод `belongsTo`, предоставляемый Eloquent для декларации отношения:

    <?php

    namespace App;

    use App\User;
    use Illuminate\Database\Eloquent\Model;

    class Task extends Model
    {
        /**
         * The attributes that are mass assignable.
         *
         * @var array
         */
        protected $fillable = ['name'];

        /**
         * Get the user that owns the task.
         */
        public function user()
        {
            return $this->belongsTo(User::class);
        }
    }

Замечательно! Теперь, когда отношения моделей определены, мы можем приступить к созданию контроллеров!

<a name="routing"></a>
## Маршрутизация

В [базовом руководстве](/docs/{{version}}/quickstart) для создания списка задач, мы определяли всю логику используя функции-замыкания в файле `routes.php`. В данном приложении мы будем использовать [контроллеры](/docs/{{version}}/controllers) для организации роутов (маршрутов). Контроллеры позволят разбить логику обработки HTTP-запросов на несколько файлов для лучшей организации кода.

<a name="displaying-a-view"></a>
### Отображение представления

У нас будет единственный роут использующий функцию-замыкание, это роут `/` какой послужит точкой входа для гостей приложения. Давайте заполним наш корневой роут `/`. По этому маршруту мы хотим отображать шаблон HTML, содержащий страницу "добро пожаловать":

В Laravel, все HTML шаблоны хранятся в папке `resources/views`, мы будем использовать вспомогательную функцию `view` для возврата одного из шаблонов:

    Route::get('/', function () {
        return view('welcome');
    });

Конечно, нам нужно создать и сам шаблон. Мы создадим его понемногу!

<a name="authentication-routing"></a>
### Аутентификация

Помните, что мы также должны предоставить пользователям возможность создавать учетные записи и осуществлять вход в наше приложение. Как правило, слой аутентификации в веб-приложении является трудоемкой задачей. Однако, так как это распространенная потребность в разработке, Laravel старается сделать эту процедуру максимально безболезненной.

Во-первых, обратите внимание что уже существует контроллер `app/Http/Controllers/Auth/AuthController`, включенный в приложение Laravel. Этот контроллер использует специальный трейт `AuthenticatesAndRegistersUsers`, который содержит всю необходимую логику для создания и аутентификации пользователей.

#### Роуты и представления для аутентификации

Что необходимо сделать? Нам все еще нужны представления для регистрации и входа, а так же требуется определить маршруты для контроллера аутентификации. Мы можем сделать все это одной Artisan-командой `make:auth`:

    php artisan make:auth

> **Примечание:** Если вы хотите посмотреть и использовать законченные примеры этих представлений, [они доступны на GitHub](https://github.com/laravel/quickstart-intermediate).

Теперь все что нам нужно - это добавить маршруты для аутентификации в файл роутов. Мы можем это сделать используя метод `auth` в фасаде `Route`, который зарегистрирует все маршруты нужные для регистрации, входа и сброса пароля:

    // Authentication Routes...
    Route::auth();

После того как `auth` роуты зарегистрированы, убедитесь что переменная `$redirectTo` контроллера `app/Http/Controllers/Auth/AuthController` установлена в '/tasks':

    protected $redirectTo = '/tasks';

Кроме того, необходимо обновить файл `app/Http/Middleware/RedirectIfAuthenticated.php`, выставив нужный нам путь для перенаправления:

    return redirect('/tasks');

<a name="the-task-controller"></a>
### Контроллер Task

Поскольку нам нужно получать и сохранять задачи, создадим контроллер `TaskController` используя Artisan интерфейс, который поместит новый контроллер в директорию `app/Http/Controllers`:

    php artisan make:controller TaskController

Теперь, когда контроллер создан, добавим некоторые роуты в файл `app/Http/routes.php` чтобы указать на созданный нами контроллер:

    Route::get('/tasks', 'TaskController@index');
    Route::post('/task', 'TaskController@store');
    Route::delete('/task/{task}', 'TaskController@destroy');

#### Аутентификация для всех роутов Tasks

В нашем приложении мы хотим, чтобы все роуты, касающиеся задач, требовали аутентификации пользователя. Другими словами пользователь должен быть залогинен, чтобы иметь возможность создавать задачи. Итак, нам нужно ограничить доступ ко всем маршрутам задач только для авторизованных пользователей. Laravel реализует это с помощью [middleware](/docs/{{version}}/middleware).

Чтобы требовать аутентификации пользователя для всех экшенов в контроллере, мы можем вызвать метод `middleware` из конструктора контроллера. Все доступные middleware (посредники) для маршрутов определены в `app/Http/Kernel.php`. В нашем случае, мы можем задать `auth` middleware для всех действий контроллера:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Requests;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class TaskController extends Controller
    {
        /**
         * Create a new controller instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');
        }
    }

<a name="building-layouts-and-views"></a>
## Создание структуры макета и представления

Основная часть этого приложения имеет только один вид (шаблон, представление), который содержит форму для добавления новых задач а также список всех текущих задач. Чтобы вы поняли как это выглядит, вот скриншот законченного приложения, дополенный CSS-стилями Twitter Bootstrap:

![Application Image](https://laravel.com/assets/img/quickstart/basic-overview.png)

<a name="defining-the-layout"></a>
### Определение макета структуры

Почти все веб-приложения используют одну и ту же структуру. Например, наше приложение имеет меню навигации одинаковое на всех страницах(если у нас было бы их больше). Laravel позволяет легко разделять эти общие черты на каждой странице с помощью шаблонизатора Blade.

Как мы уже говорили ранее, все шаблоны Laravel хранит в `resources/views`. Итак, давайте определим новый макет в `resources/views/layouts/app.blade.php`. Расширение `.blade.php` инструктирует Laravel использовать [Шаблонизатор Blade](/docs/{{version}}/blade) для отображения представления. Blade - простой, но мощный шаблонизатор, входящий в состав Laravel. В отличие от других шаблонизаторов, он не ограничивает вас в использовании конструкций PHP внутри шаблонов. Шаблоны Blade компилируются в PHP-код и кэшируются фреймворком - Blade не вносит дополнительных тормозов в работу фреймворка.

Наш`app.blade.php` шаблон должен выглядеть следующим образом:

    <!-- resources/views/layouts/app.blade.php -->

    <!DOCTYPE html>
    <html lang="en">
        <head>
            <title>Laravel Quickstart - Intermediate</title>

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

Обратите внимание на `@yield('content')` часть шаблона. Это специальная директива, которая определяет, где будут находиться дочерние шаблоны. Они будут расширять базовый шаблон, нашим собственным контентом. Давайте создадим наш собственный шаблон, который будет наследовать этот макет и содержать основной контент.

<a name="defining-the-child-view"></a>
### Определение унаследованного представления

Отлично, наш базовый шаблон создан. Нам необходимо определить, как будет выглядеть наша форма, в которой мы будем добавлять новые задачи, а также таблицу со всеми нашими существующими задачами. Создадим шаблон в `resources/views/tasks/index.blade.php`, соответствующий методу `index` в нашем контроллере `TaskController`.

Мы пропустим верстку нашего шаблона и сосредоточимся на более важных вещах. Напоминаем, что весь исходных код вы можете скачать по ссылке на [GitHub](https://github.com/laravel/quickstart-intermediate):

    <!-- resources/views/tasks/index.blade.php -->

    @extends('layouts.app')

    @section('content')

        <!-- Bootstrap Boilerplate... -->

        <div class="panel-body">
            <!-- Display Validation Errors -->
            @include('common.errors')

            <!-- New Task Form -->
            <form action="{{ url('task') }}" method="POST" class="form-horizontal">
                {{ csrf_field() }}

                <!-- Task Name -->
                <div class="form-group">
                    <label for="task-name" class="col-sm-3 control-label">Task</label>

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

Мы определили внешний вид нашего приложения. Теперь вернем это представление из метода `index` контроллера `TaskController`:

    /**
     * Display a list of all of the user's task.
     *
     * @param  Request  $request
     * @return Response
     */
    public function index(Request $request)
    {
        return view('tasks.index');
    }

Далее, мы рассмотрим как дополнить метод контроллера, соответствующий маршруту `POST /task`, чтобы мы могли добавить новую задачу в БД.

<a name="adding-tasks"></a>
## Добавление задачи

<a name="validation"></a>
### Проверка ввода

Теперь, когда мы создали нашу форму, мы должны дополнить наш метод `TaskController@store`, чтобы проверить правильность ввода и создать новую задачу. Сперва давайте проверим правильность ввода.

Для этой формы, мы создадим поле `name`, которое будет обязательным и которое будет содержать не более `255` символов. Если проверка вернет отрицательный результат, мы перенаправим пользователя назад на `/tasks` URL, а также добавим введенные данные в [сессии](/docs/{{version}}/session):

    /**
     * Create a new task.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $this->validate($request, [
            'name' => 'required|max:255',
        ]);

        // Create The Task...
    }

Если вы изучали [базовое руководство](/docs/{{version}}/quickstart), вы заметите что код проверки ввода выглядит немного по-другому! Поскольку мы находимся в контроллере, мы можем использовать возможность трейта `ValidatesRequests` который включен в базовый контроллер Laravel. Этот трейт предоставляет простой метод `validate`, который принимает запрос и массив правил проверки.

Нам даже не пришлось определять вручную случай когда проверка ввода возвращает отрицательный результат, или вручную поизводить редирект. Если проверка вернет отрицательный результат для заданных правил, пользователь будет автоматически перенаправлен туда, откуда он пришел и ошибки проверки будут автоматически помещены (для возможности flash) в сессию. Неплохо!

#### Переменная `$errors`

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
### Создание задачи

Теперь, когда мы разобрались с проверкой данных, давайте создадим новую задачу, продолжая заполнять наш маршрут. После того, как новая задача будет создана, мы будем возвращать пользователя обратно на URL `/tasks`. Для создания задачи, мы будем использовать мощь отношений Eloquent.

Большинство отношений в Laravel создаются за счет метода `create`, который принимает массив атрибутов и автоматически
установит значение внешнего ключа на соответствующей модели перед сохранением в базе данных. В нашем случае, метод `create` автоматически установит свойство `user_id` данной задачи в значение ID текущего пользователя прошедшего аутентификацию, к которому мы обращаемся с помощью конструкции `$request->user()`:

    /**
     * Create a new task.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $this->validate($request, [
            'name' => 'required|max:255',
        ]);

        $request->user()->tasks()->create([
            'name' => $request->name,
        ]);

        return redirect('/tasks');
    }

Отлично! Теперь мы можем добавлять задачи. Теперь, нам нужно вывести список всех существующих задач.

<a name="displaying-existing-tasks"></a>
## Отображение задачи

Для начала, нам надо немного изменить наш метод `TaskController@index` чтобы передать все существующие задачи в наш шаблон. Функция `view` принимает второй аргумент, который является массивом данных. Обращаться к массиву мы будем по ключам:

    /**
     * Display a list of all of the user's task.
     *
     * @param  Request  $request
     * @return Response
     */
    public function index(Request $request)
    {
        $tasks = $request->user()->tasks()->get();

        return view('tasks.index', [
            'tasks' => $tasks,
        ]);
    }

Давайте рассмотрим теперь некоторые возможности внедрения зависимостей Laravel чтобы внедрить `TaskRepository` в наш `TaskController`, который мы будем использовать для доступа ко всем нашим данным.

<a name="dependency-injection"></a>
### Внедрение зависимостей

Laravel [сервис-контейнер](/docs/{{version}}/container) является одной из наиболее мощных возможностей фреймворка в целом. После чтения данного руководства, неплохо будет прочесть всю документацию по сервис-контейнеру.

#### Создание репозитория

Как упоминалось ранее, мы хотим создать `TaskRepository` который содержит всю нашу логику доступа к данным для модели `Task`. Это будет особенно полезно, если приложение растет и вам нужно сделать общими запросы Eloquent для всего приложения в целом.

Создадим директорию `app/Repositories` и добавим туда класс `TaskRepository`. Помните, все содержимое директории `app` (включая субдиректории) в Laravel загружается автоматически, используя стандарт автозагрузки PSR-4, таким образом вы можете создать некоторые дополнительные субдиректории если это необходимо:

    <?php

    namespace App\Repositories;

    use App\User;

    class TaskRepository
    {
        /**
         * Get all of the tasks for a given user.
         *
         * @param  User  $user
         * @return Collection
         */
        public function forUser(User $user)
        {
            return $user->tasks()
                        ->orderBy('created_at', 'asc')
                        ->get();
        }
    }

#### Внедрение репозитория

После того как наш репозиторий определен, мы можем просто указать его в качестве типа входного аргумента в конструкторе нашего `TaskController` и использовать его в методе `index` контроллера. Так как Laravel использует сервис-контейнер для всех контроллеров, наши зависимости будут автоматически внедрены в экземпляр контроллера:

    <?php

    namespace App\Http\Controllers;

    use App\Task;
    use App\Http\Requests;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    use App\Repositories\TaskRepository;

    class TaskController extends Controller
    {
        /**
         * The task repository instance.
         *
         * @var TaskRepository
         */
        protected $tasks;

        /**
         * Create a new controller instance.
         *
         * @param  TaskRepository  $tasks
         * @return void
         */
        public function __construct(TaskRepository $tasks)
        {
            $this->middleware('auth');

            $this->tasks = $tasks;
        }

        /**
         * Display a list of all of the user's task.
         *
         * @param  Request  $request
         * @return Response
         */
        public function index(Request $request)
        {
            return view('tasks.index', [
                'tasks' => $this->tasks->forUser($request->user()),
            ]);
        }
    }

<a name="displaying-the-tasks"></a>
### Отображение задачи

После того как данные переданы, мы можем осуществить их перебор в шаблоне `tasks/index.blade.php` и показать в виде таблицы. Конструкция `@foreach` шаблонизатора Blade позволяет писать краткие циклы, какие в итоге будут преобразованы в обычный PHP код:

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
## Удаление задачи

<a name="adding-the-delete-button"></a>
### Добавление кнопки <Удалить>

Помните, в нашем шаблоне мы оставляли комментарий <!-- TODO: Delete Button -->, чтоб не забыть добавить кнопку <удалить>? Итак, давайте отредактируем наш шаблон `tasks/index.blade.php` и добавим кнопку удаления для каждой задачи в нашем списке. Мы создадим небольшую форму для кнопки <удалить>. При нажатии на кнопку, будет отправлен запрос `DELETE /task` на какой приложение отреагирует запуском метода `TaskController@destroy`:

    <tr>
        <!-- Task Name -->
        <td class="table-text">
            <div>{{ $task->name }}</div>
        </td>

        <!-- Delete Button -->
        <td>
            <form action="{{ url('task/'.$task->id) }}" method="POST">
                {{ csrf_field() }}
                {{ method_field('DELETE') }}

                <button type="submit" id="delete-task-{{ $task->id }}" class="btn btn-danger">
                    <i class="fa fa-btn fa-trash"></i>Delete
                </button>
            </form>
        </td>
    </tr>

<a name="a-note-on-method-spoofing"></a>
#### ПОДМЕНА HTTP-МЕТОДА

Обратите внимание, HTML-формы не поддерживают методы HTTP-запроса PUT или DELETE. Для того, чтобы отправить на сервер HTTP-запрос с этими методами, вам нужно добавить в форму скрытый input с именем _method.

Значение этого поля будет восприниматься фреймворком как тип HTTP-запроса. Например:

    <input type="hidden" name="_method" value="DELETE">

<a name="route-model-binding"></a>
### Привязка маршрута к модели

Теперь мы почти готовы определить метод `destroy` в контроллере `TaskController`. Но сначала взглянем еще раз на определение роута и метод контроллера, соответствующий этому роуту:

    Route::delete('/task/{task}', 'TaskController@destroy');

    /**
     * Destroy the given task.
     *
     * @param  Request  $request
     * @param  Task  $task
     * @return Response
     */
    public function destroy(Request $request, Task $task)
    {
        //
    }

Так как переменная `{task}` в нашем роуте соответствует переменной `$task` в методе контроллера, механизм Laravel [привязка моделей к роутам](/docs/{{version}}/routing#route-model-binding) автоматически введет соответствующий экземпляр модели Task.

<a name="authorization"></a>
### Авторизация

Теперь экземпляр модели `Task` внедряется в наш `destroy` метод; тем не менее, у нас нет никакой гарантии того, что аутентифицированный пользователь действительно является "владельцем" данной задачи. Например, злоумышленник может придумать вредоносный запрос для удаления задач другого пользователя, передавая случайный ID задачи в строку адреса (маршрут) `/tasks/{task}`. Таким образом, нам нужно использовать возможности авторизации Laravel чтобы убедиться, что аутентифицированный пользователь действительно является
владельцем экземпляра модели `Task`, внедренного в маршрут.

#### Создание Policy (политики)

Laravel использует "политики" для организации логики авторизации в простые, маленькие классы. Как правило, каждая политика соответствует модели. Итак, создадим `TaskPolicy` используя интерфейс Artisan, который разместит сгенерированный файл в `app/Policies/TaskPolicy.php`:

    php artisan make:policy TaskPolicy

Далее, добавим метод `destroy` к политике. Этот метод получит экземпляры как `User` модели, так и модели `Task`. Метод должен просто проверить, совпадает ли ID пользователя с `user_id` полем задачи. Фактически, все методы политики обязаны возвращать `true` или `false`:

    <?php

    namespace App\Policies;

    use App\User;
    use App\Task;
    use Illuminate\Auth\Access\HandlesAuthorization;

    class TaskPolicy
    {
        use HandlesAuthorization;

        /**
         * Determine if the given user can delete the given task.
         *
         * @param  User  $user
         * @param  Task  $task
         * @return bool
         */
        public function destroy(User $user, Task $task)
        {
            return $user->id === $task->user_id;
        }
    }

Наконец, ассоциируем нашу модель `Task` с политикой `TaskPolicy`. Мы можем это сделать, добавив строку кода в файл `app/Providers/AuthServiceProvider.php`, задавая свойство `$policies`. Это сообщает Laravel о том, какая политика должна использоваться каждый раз, когда мы пытаемся выполнить действие с экземпляром модели `Task`:

    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        'App\Task' => 'App\Policies\TaskPolicy',
    ];


#### Авторизация экшенов (методов, действий)

Теперь, когда политика написана, используем ее в методе `destroy`. Все контроллеры Laravel могут вызывать метод `authorize`, какой предоставлен трейтом `AuthorizesRequest`:

    /**
     * Destroy the given task.
     *
     * @param  Request  $request
     * @param  Task  $task
     * @return Response
     */
    public function destroy(Request $request, Task $task)
    {
        $this->authorize('destroy', $task);

        // Delete The Task...
    }

Давайте мельком посмотрим на вызов метода. Первый аргумент передаваемый методу `authorize` является именем метода политики, какой мы хотим вызвать. Второй аргумент - экземпляр модели к какой политика применяется. Помните, мы недавно сказали Laravel что наша модель `Task` model связана с политикой `TaskPolicy`, таким образом фреймворк знает, какая политика нужна для метода `destroy`. Текущий пользователь будет автоматически передан в метод политики, по этому нам не нужно вручную передавать его здесь.

Если действие контроллера разрешено, наш код будет продолжать выполнение в обычном режиме. Тем не менее, если оно не разрешено (то есть метод `destroy` политики вернул `false`), будет возбуждено исключение 403 и страница с ошибкой будет показана пользователю.

> **Примечание:** Есть несколько других способов взаимодействия с сервисами авторизации, какие предоставляет Laravel. Обязательно ознакомьтесь с [документацией по авторизации](/docs/{{version}}/authorization).

<a name="deleting-the-task"></a>
### Удаление задачи

Наконец, завершим добавление логики в метод `destroy` для удаления задачи. Мы можем использовать метод Eloquent `delete` для удаления данного экземпляра модели из базы данных. После удаления задачи, мы перенаправим пользователя обратно на `/tasks` URL:

    /**
     * Destroy the given task.
     *
     * @param  Request  $request
     * @param  Task  $task
     * @return Response
     */
    public function destroy(Request $request, Task $task)
    {
        $this->authorize('destroy', $task);

        $task->delete();

        return redirect('/tasks');
    }
