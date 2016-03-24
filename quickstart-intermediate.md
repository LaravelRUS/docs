git e14c3e25bd11f4fa4b5136a5cdcef6b22c9ad89f

---

# TODO  лист для продвинутых

- [Введение](#introduction)
- [Установка](#installation)
- [Подготавливаем базу данных](#prepping-the-database)
    - [Миграции](#database-migrations)
    - [Модель Eloquent](#eloquent-models)
    - [Взаимосвязи в модели Eloquent](#eloquent-relationships)
- [Маршрутизация](#routing)
    - [Отображение представления](#displaying-a-view)
    - [Аутентификация](#authentication-routing)
    - [Котроллер задач](#the-task-controller)
- [Создаем структуру макета и представления](#building-layouts-and-views)
    - [Определение макет структуры](#defining-the-layout)
    - [Определение унаследованного представления](#defining-the-child-view)
- [Добавляем задачу](#adding-tasks)
    - [Проверка ввода](#validation)
    - [Создаем задачу](#creating-the-task)
- [Отображаем существующие задачи](#displaying-existing-tasks)
    - [Внедрение зависимостей](#dependency-injection)
    - [Отображаем задачи](#displaying-the-tasks)
- [Удаляем задачу](#deleting-tasks)
    - [Добавляем кнопку <Удалить>](#adding-the-delete-button)
    - [Группы маршрутов](#route-model-binding)
    - [Авторизация](#authorization)
    - [Удаляем задачу](#deleting-the-task)

<a name="introduction"></a>
## Введение

В данном гайде мы познакомимся с фреймворком, узнаем о миграциях, Eloquent ORM, роутинге,аутентификации, авторизации, внедрении зависимостей, валидации, шаблонах и шаблонизаторе Blade. Это неплохая стартовая точка для тех, кто знаком с фреймворком и PHP на среднем уровне.

В качестве реального приложения, которое мы будем изучать, выступит приложение простейшего todo-листа. В отличии от [начального гайда](/docs/{{version}}/quickstart), тут мы рассмотрим также возможность аутентификации и авторизации. Полный код приложения можете скачать на [GitHub](https://github.com/laravel/quickstart-intermediate).

<a name="installation"></a>
## Установка

#### Установка Laravel

Для установки Laravel вам нужно иметь на компьютере рабочее PHP-окружение, или воспользоваться [виртуальной машиной Homestead](/docs/{{version}}/homestead).
Фреймворк устанавливается в папку `quickstart` следующей командой Composer:

    composer create-project laravel/laravel quickstart --prefer-dist

#### Установка The Quickstart (Необязательно)

Вы можете продолжать читать этот гайд и применять его на фреймворке, а можете скачать исходный код проекта и исследовать его:

    git clone https://github.com/laravel/quickstart-intermediate quickstart
    cd quickstart
    composer install
    php artisan migrate

Подробнее об установке фреймворка можно прочитать в [соответствующем разделе документации](/docs/{{version}}/installation). 

<a name="prepping-the-database"></a>
## Подготавливаем базу данных

<a name="database-migrations"></a>
### Миграции баз данных

Сперва, давайте используем миграции, для того чтобы определить таблицу, для хранения всех наших задач. Миграции БД Laravel обеспечивают просто способ определения структуры таблиц и модификации с использованием PHP кода. Вместо того чтобы постоянно предупреждать вашу команду, о каких либо изменения в БД и им не приходилось вручную модифицировать их локальные копии, они могут просто запустить миграцию, которую вы отправили в вашу систему управления версиями.

#### Таблица `users` 

Для того чтоб пользователи могли создавать свои учетные записи, нам нужно создать таблицу для их хранения. К счастью, базовая таблица `users` в Laravel идет из коробки, поэтому нам не нужно создавать ее в ручную. По умолчанию, данная миграция рпсположена в папке `database/migrations`.

#### Таблица `tasks`

А теперь давайте создадим таблицу, для хранения наших задач. [Интерфейс Artisan](/docs/{{version}}/artisan) может быть использован для создания различных классов и сэкономить вам много времени, пока вы разрабатываете свои проекты на Laravel. В нашем случае, давайте используем команду `make:migration` , для создания новой миграции БД для нашей таблицы `tasks`:

    php artisan make:migration create_tasks_table --create=tasks

Миграция будет помещена в папку `database/migrations` вашего проекта. Как вы могли заметить, команда `make:migration` добавляет метку времени, которая позволит фреймворку определить порядок применения миграций. Давайте отредактируем этот файл и добавим дополнительный столбец `string` для названия наших задач, а также столбец `user_id`, который соединит наши таблицы `tasks` и `users`

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

Эта команда создаст все наши таблицы БД. Если вы проверите таблицы БД, вы увидите новую таблицу `tasks`, которая содержит столбцы, определенные в нашей миграции. Теперь мы готовы определить модель Eloquent ORM для наших задач!

<a name="eloquent-models"></a>
### Модель Eloquent

[ORM Eloquent](/docs/{{version}}/eloquent) - красивая и простая реализация паттерна ActiveRecord для работы с БД.
Каждая таблица имеет соответствующий класс-модель, который используется для работы с этой таблицей. Модели позволяют читать данные из таблиц и записывать данные в таблицу.


#### Модель`User`

Модель`User` также идет за коробоки, и нам не придется создавать ее вручную

#### Модель`Task`

Итак, давайте определим модель `Task`, которая соответствует нашей, только что созданной, таблице `tasks`. Мы можем использовать команду Artisan для создания этой модели. В данном случае, мы будем использовать команду `make:model` :

    php artisan make:model Task
Модель будет помещена в папку `app`. По умолчанию класс модели будет пустой. Мы не должны явно указывать Eloquent какую таблицу должен привязывать к нашей модели, потому что Eloquent будет использовать в качестве названия таблицы имя класса в нижнем регистре и во множественном числе. В нашем случае Eloquent предположит, что модель `Task` хранит свои данные в таблице `tasks`.

Давайте отредактируем нашу модель. Для начала, нам надо защитить от массового присваивания атрибут `name`. Это позволит нам заполнить атрибут `name` когда будем использовать метод `create`:

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
### Взаимосвязи в модели Eloquent

Теперь, когда наши модели определены, нам нужно связать их. Например, у пользователя может быть неограниченное количество задач, но у задачи может быть только один пользователь. Благодаря взаимосвязям мы сможем показать все задачи определенного пользователя слежующим образом:

    $user = App\User::find(1);

    foreach ($user->tasks as $task) {
        echo $task->name;
    }

#### Взаимосвязь между `tasks` и моделью `User`

Давайте определим нашу взаимосвязь между `tasks` и моделью `User`. Взаимосвязи в модели Eloquent определяется методами. Подробнее о взаимосвязях вы можете почитать в [соответствующем разделе документации](/docs/{{version}}/eloquent-relationships). В нашем случае, мы определим функцию `tasks` в модели `User` которая будет вызывать метод `hasMany` :

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

#### Взаимосвязь между `user` и моделью `Tasks`

Давайте определим нашу взаимосвязь между `Task` и моделью `user`. Также мы будем определять наши взаимоотношения как метод в нашей модели. В этом случае, мы будем использовать метод `belongsTo` :

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

С взаимосвязями мы закончили! Дальше мы рассмотрим наши контроллеры.

<a name="routing"></a>
## Машрутизация

В [гайде для начинающих](/docs/{{version}}/quickstart) всю логику обработки запросов мы определяли в одном файле `routes.php`. В этом приложении, мы организуем её с помощью [контроллеров](/docs/{{version}}/controllers). Контроллеры могут группировать связанную с обработкой HTTP-запросов логику в отдельный класс

<a name="displaying-a-view"></a>
### Отображение представления

Мы будем использовать всего один маршрут, логика которого будет обрабатываться только в файле `routes.php`. Маршрут `/` будет просто отображать стартовую страницу "welcome" для незалогиненных пользователей. 

We will have a single route that uses a Closure: our `/` route, which will simply be a landing page for application guests. So, let's fill out our `/` route. From this route, we want to render an HTML template that contains the "welcome" page:

В Laravel, все HTML шаблоны хранятся в папке `resources/views`, мы будем использовать вспомогательную функцию `view` для возврата нашей главной страницы:

    Route::get('/', function () {
        return view('welcome');
    });

Чуть позже мы создадим нашу страницу "welcome". А пока перейдем к аутентификации.


<a name="authentication-routing"></a>
### Аутентификация

Нам все еще нужно дать возможность пользователям завести акаунт и залогиниться в наше приложение. Как правило, создание полной аутентификации и авторизации, это очень трудноёмкая и долгая задача. Однако, в Laravel все уже готово из коробки, благодаря чему эта процедура пройдет для нас безполезненно.

Вы уже можете найти контроллер аутентификации `app/Http/Controllers/Auth/AuthController` в своем проекте.
Этот контроллер уже содержит всю необходимую логику для регистрации и авторизации пользователей.

#### Authentication Routes & Views

И так, что осталось сделать нам? Нам все еще необходимо создать шаблоны входа и регистрации, а также определить роуты, которые будут указывать на наш контроллер аутентификации. Все это мы можем сделать всего одной командой `make:auth`:

    php artisan make:auth --views

> **Примечание:** Ксли вы хотите посмотреть полные примеры этих шаблонов, помните, весь исходный код вы можете посмотреть на [GitHub](https://github.com/laravel/quickstart-intermediate).

Теперь, все что нам нужно сделать, это добавить роуты аутентификации в наш файл маршрутов. Мы сможем сделать это с помощью метода `auth`, который будет следить за всеми роутами , которые нужны для регистраци, входа или сброса пароля:

    // Authentication Routes...
    Route::auth();

Давайте сделаем так, чтоб после того как пользователь зарегистрировался или залогинился, его перенаправляло по роуту /tasks'. Для этого, в контроллере `app/Http/Controllers/Auth/AuthController` свойству `$redirectTo` нужно указать значение '/tasks':

    protected $redirectTo = '/tasks';

Кроме того, необходимо обновить `app/Http/Middleware/RedirectIfAuthenticated.php` и указать правильный роут для переадрессации:

    return redirect('/tasks');

<a name="the-task-controller"></a>
### Контроллер задач

Так как нам придеться сохранять и выводить наши задачи, давайте создадим контроллер задач используя команду Artisan, которая создаст наш новый контроллер в папку `app/Http/Controllers`

    php artisan make:controller TaskController

Теперь, когда наш контроллер создан, давайте свяжим наши маршруты с контроллером в файле `app/Http/routes.php`:

    Route::get('/tasks', 'TaskController@index');
    Route::post('/task', 'TaskController@store');
    Route::delete('/task/{task}', 'TaskController@destroy');

#### Аутентификация всех роутов

В нашем приложении, все маршруты по работе с задачами требуют аутентификации. Другими словами, пользователь должен быть залогинен для создания новой задачи. Нам нужно,чтбы только залогиненные прользователи имели возможность работать с нашими задачами. 
For this application, we want all of our task routes to require an authenticated user. In other words, the user must be "logged into" the application in order to create a task. So, we need to restrict access to our task routes to only authenticated users. В Laravel осуществляется это благодаря [middleware](/docs/{{version}}/middleware).

Для того, чтобы ограничить доступ для незалогиненных пользователей, мы добавим вызов метода `middleware` в конструкторе нашего контроллера. Все доступные пути определены в файле `app/Http/Kernel.php`. В нашем случае, мы добавим проверку авторизации для всех действий на контроллере:

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
## Создаем структуру макета и представления

Это приложение имеет только одну страницу, которая содержит форму для добавления новых задач, а также список всех текущих задач. Так будет выглядеть наша страница с использованием основных Bootstrap стилей:

![Application Image](https://laravel.com/assets/img/quickstart/basic-overview.png)

<a name="defining-the-layout"></a>
### Определение структуры макета

Почти все веб-приложения используют одну и туже структуру. Например, наше приложение имеет меню навигации одинаковое на всех страницах(если у нас было бы их больше). Laravel позволяет легко разделяет эти общие черты на каждой странице с помощью шаблонизатора Blade.

Как мы уже говорили ранее, все шаблоны Laravel хранит в `resources/views`. Итак, давайте определим новый макет в `resources/views/layouts/app.blade.php`. Расширение `.blade.php` инструктирует Laravel использовать [Шаблонизатор Blade](/docs/{{version}}/blade) для отображения представления. Blade - простой, но мощный шаблонизатор, входящий в состав Laravel. В отличие от других шаблонизаторов, он не ограничивает вас в использовании конструкций PHP внутри шаблонов. Шаблоны Blade компилируются в PHP-код и кэшируются фреймворком - Blade не вносит дополнительных тормозов в работу фреймворка.

Наш`app.blade.php` шаблон должен выглядеть следующим образом:

    <!-- resources/views/layouts/app.blade.php ->

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

Обратите внимание на `@yield('content')` часть шаблона. Это специальная директива, которая определяет, где будут находится дочерние шаблоны. Они будут расширять базовый шаблон, нашим собственным контентом. Давайте создадим наш собственный шаблон, который будет наследовать этот макет и содержать основной контент.

<a name="defining-the-child-view"></a>
### Определение унаследованного представления

Нам необходимо определить, как будет выглядеть наша форма, в которой мы будем добавлять новые задачи, а также таблицу со всеми нашими существующими задачами. Давайте создадим наш шаблон в `resources/views/tasks/index.blade.php`, который будет соответствовать нашему методу `index` в контроллере `TaskController`.

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
                {!! csrf_field() !!}

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

Мы определили внешний вид нашего приложения.Теперь давайте вернем этот шаблон из метода `index` нашего контроллера `TaskController`:

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

Далее, мы рассмотрим как создать маршрут `POST /task`, чтобы мы могли добавить новую задачу в БД.

<a name="adding-tasks"></a>
## Добавляем задачу

<a name="validation"></a>
### Проверка ввода

Теперь, когда мы создали нашу форму, нам нужно дополнить наш метод `TaskController@store`, чтобы проверить правильность ввода и создать новую задачу. Сперва давайте проверим правильность ввода.

Для этой формы, мы создадим поле `name`, которое будет обязательным и которое будет содержать менее `255` символов.
Если проверка вернет отрицательный результат, мы перенаправим пользователя назад на главную страницу `/`, а также добавим введенные данные в [сессии](/docs/{{version}}/session). Использование сессий  позволит сохранить введенные данные, если наша проверка вернет отрицательный результат:

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

Если вы проходили [гайд для начинающих](/docs/{{version}}/quickstart), вы должны были заметить, что код проверки выглядит немного по другому. Поскольку мы находимся в контроллере, мы можем использовать  `ValidatesRequests` который включен в базовый контроллер. Метод `validate`принимает массив с правилами проверки.

Если проверка вернет отрицательный результат, пользователь будет автоматически перенаправлен на страницу, с нашей формой, при этом сессии сохранят все введенные данные.

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
### Создаем задачу

Теперь, когда мы разобрались с проверкой данных, давайте создадим новую задачу, продолжая заполнять наш маршрут. После того, как новая задача будет создана, мы будем возвращать пользователя на страницу `/tasks`. Для того чтобы создать новую задачу мы будем использовать всю мощь взаимосвязей моделей Eloquent.

Большенство взаимосвязей Laravel используют метод `create`, который принимает массив атрибутов и автоматически установит значение ключа на соответствующей модели перед сохранением в БД. В нашем случаее, метод  `create` автоматически установит в свойство `user_id` создаваемой задачи ID залогиненного пользователя, к которому мы обращаемся с помощью `$request->user()`:


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
## Отображаем задачу

Для начала, нам надо немного изменить наш метод `TaskController@index`, чтобы передать все существующие задачи в наш шаблон. Функция `view` принимает второй аргумент, который является массивом данных. Обращаться к массиву мы будем по ключам:

    /**
     * Display a list of all of the user's task.
     *
     * @param  Request  $request
     * @return Response
     */
    public function index(Request $request)
    {
        $tasks = Task::where('user_id', $request->user()->id)->get();

        return view('tasks.index', [
            'tasks' => $tasks,
        ]);
    }
Давайте рассмотрим некоторые возможности по внедрению зависимостей, чтобы вставить `TaskRepository` в наш `TaskController`, который мы будем использовать для доступа к нашим данным.

<a name="dependency-injection"></a>
### Внедрение зависимостей

[Service Container](/docs/{{version}}/container) (сервис-контейнер, ранее IoC-контейнер) - это мощное средство для управлением зависимостями классов. В современном мире веб-разработки есть такой модный термин - Dependency Injection, «внедрение зависимостей», он означает внедрение неких классов в создаваемый класс через конструктор или метод-сеттер. Создаваемый класс использует эти классы в своей работе. Сервис-контейнер реализует как раз этот функционал. после прочтения этого гайда, обязательно прочитайте документацию.

#### Создание репозитория

Как мы уже упоминали ранее, мы хотим определить `TaskRepository`, который одержит всю нашу логику доступа к данным модели `Task`. Всю прелесть можно будет оценить, когда приложение начнет расти.

Итак, давайте создадим папку `app/Repositories` и добавим класс `TaskRepository`. Помните, что все папки автоматически загружаются с помощью стандарта PSR-4, поэтому вы можете создавать столько дополнительных каотологов, сколько вам будет угодно:

    <?php

    namespace App\Repositories;

    use App\User;
    use App\Task;

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
            return Task::where('user_id', $user->id)
                        ->orderBy('created_at', 'asc')
                        ->get();
        }
    }

#### Подключаем репозиторий
После того как мы создали наш репозиторий, нам нужно указать его в конструкторе и использовать его в наших маршрутах  `index`. Так как Laravel использует репозитории в своих контроллерах, наши зависимости будут автоматически подключаться в экземпляр контроллера:

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
### Отображаем задачу


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

Помните, в нашем шаблоне мы оставляли комментарий <!-- TODO: Delete Button -->, чтоб не забыть добавить кнопку <удалить>? Итак, давайте отредактируем наш шаблон `tasks.blade.php` и добавим кнопку удаления для каждой задачи в нашем списке. Мы создадим небольшую форму для кнопки <удалить>. При нажатии на кнопку, будет отправлен запрос `DELETE /task`, который будет инициировать метод `TaskController@destroy`:

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
### Группы маршрутов
Мы уже почти готовы, чтобы определить метод `destroy` в нашем контроллере `TaskController`. Но, для начала, давайте пересмотрим наш роут и методы :

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

Так как переменная `{task}` в нашем маршруте соответствует переменной `{task}`, определенной в методе нашего контроллера, [Группы маршрутов](/docs/{{version}}/routing#route-model-binding) Laravel будут автоматически придавать соответствующий экземпляр модели Task.

<a name="authorization"></a>
### Aвторизация

Теперь наш экземпляр `Task`, который подключает  наш метод `destroy`; Тем не менее, мы должны удостоворетиться что авторизованный пользователь является владельцем задачи. Например, злоумышлиник может удалить задачу другого пользователя просто передавая случайный индитификатор задачи для `/tasks/{task}`. Для этого нам нужно проверить, что пользователь действительно является владельцем данной задачи.

#### Создаем правило авторизации

Laravel использует "правила" для организации логики в простые, маленькие классы.  Как правила, каждое правило соответствует определенной модели. Итак, давайте создадим `TaskPolicy` использую команду Artisan, которая разместит наш новый файл в `app/Policies/TaskPolicy.php`:

    php artisan make:policy TaskPolicy

Далее, давайте добавим метод `destroy` к нашему правилу. Этот метод будет получать экземпляры `User` и `Task`.Дальше метод будет проверять совпадают ли  индитификаторы пользователя и задачи. Все правила повзращают `true` или` false`:

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

И, наконец, нам нужно связать нашу модель `Task` с нашим правилом ` Task Policy`. Мы может осуществить это, добавив свойство `$policies` в файле `app/Providers/AuthServiceProvider.php`. Это сообщит Laravel, какое правило следует использовать всякий раз, когда мы пытаемся совершить какое-либо действие с `Task`:

    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        'App\Task' => 'App\Policies\TaskPolicy',
    ];


#### Authorizing The Action

Теперь, кога готова наше правило, давайте используем его в методе `destroy`:

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

Давайте рассмотрем вызов этого метода. Первый аргумент передаваемый методу `authorize` это имя  метода который мы хотим вызвать. Второй аргумент- это модель эеземпляра. Помните, мы указывали Laravel, что наша модель `Task` соотвествует нашему правилу `TaskPolicy`, поэтому фреймворк знает для какой именно задачи вызывать метод `destroy`. Текущий пользователь будет автоматически направлена правилом, поэтому нам не надо будет вручную это указывать.

Если у данного пользователя есть разрешение на удаление записи, наш код будет продолжать работать в обычном режиме. Если же нет(тоесть метод `destroy` вернет `false`), пользователь увидит страницу с ошибкой.

> ** Примечание: ** Также существуют другие способы взаимодействия с авторизацией. Полностью об авторизации вы можете посмотреть в [документации(/docs/{{version}}/authorization).

<a name="deleting-the-task"></a>
### Удаляем задачу

Ну и наконец, давайте закончим, добавив логику для нашего метода `destroy`, чтобы удалить задачу. Мы можем использовать метод Eloquent `delete`, чтобы удалить задачу из БД.  После того, как запись будет удалена, мы будем перенаправлять пользователя обратно в `/ tasks` :

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
