git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Авторизация

- [Введение](#introduction)
- [Шлюзы](#gates)
    - [Написание шлюзов](#writing-gates)
    - [Авторизация действий](#authorizing-actions-via-gates)
- [Создание политик](#creating-policies)
    - [Генерация политик](#generating-policies)
    - [Регистрация политик](#registering-policies)
- [Написание политик](#writing-policies)
    - [Методы политик](#policy-methods)
    - [Методы без моделей](#methods-without-models)
    - [Фильтры политик](#policy-filters)
- [Авторизация действий с помощью политик](#authorizing-actions-using-policies)
    - [Через модель пользователя](#via-the-user-model)
    - [Через посредников](#via-middleware)
    - [Через хелперы контроллера](#via-controller-helpers)
    - [Через шаблоны Blade](#via-blade-templates)

<a name="introduction"></a>
## Введение

В дополнение к изначально предоставленным службам [аутентификации](/docs/{{version}}/authentication), Laravel также предоставляет простой способ авторизовать действия пользователя в отношении данного ресурса. Как и в случае с аутентификацией, подход Laravel к авторизации прост, и есть два основных способа авторизации действий: шлюзы и политики.

Думайте о шлюзах и политиках, как о маршрутах и контроллерах. Шлюзы обеспечивают простое решение на основе замыканий, в свою очередь политики, как контроллеры, группируют свою логику вокруг конкретной модели или ресурса. Сначала мы разберем шлюзы, а затем рассмотрим политики.

Важно не рассматривать шлюзы и политики как взаимоисключающие вещи. Большинство приложений, скорее всего, содержат смесь шлюзов и политик, и это прекрасно! Шлюзы наиболее применимы к действиям, которые не связаны с какой-либо моделью или ресурсом, например, просмотр панели администратора. В противоположность этому, политика должна быть использована, если вы хотите разрешить действие для конкретной модели или ресурса.

<a name="gates"></a>
## Шлюзы

<a name="writing-gates"></a>
### Написание шлюзов

Шлюзы (гейты, gates) - это функции-замыкания, которые определяют, имеет ли пользователь право выполнить данное действие; они обычно определяются в классе `App\Providers\AuthServiceProvider` с помощью фасада `Gate`. Шлюзы всегда получают экземпляр пользователя в качестве первого аргумента. Также они могут принимать дополнительные аргументы, например, соответствующую модель Eloquent:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', function ($user, $post) {
            return $user->id == $post->user_id;
        });
    }

Шлюзы также можно задать, используя строку анонимной функции стиля `Class@method`, как контроллеры:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', 'PostPolicy@update');
    }

#### Шлюзы ресурсов

Можно определить несколько возможностей шлюзов одновременно, используя метод `resource`:

    Gate::resource('posts', 'PostPolicy');

Это идентично ручному определению следующих определений шлюзов:

    Gate::define('posts.view', 'PostPolicy@view');
    Gate::define('posts.create', 'PostPolicy@create');
    Gate::define('posts.update', 'PostPolicy@update');
    Gate::define('posts.delete', 'PostPolicy@delete');

По умолчанию будут определены следующие возможности: `view`, `create`, `update` и `delete`. Можно перезадать возможности по умолчанию, передав массив в качестве третьего аргумента методу `resource`. Ключ массива определяет название возможности, в то время как значение определяет название метода:

    Gate::resource('posts', 'PostPolicy', [
        'photo' => 'updatePhoto',
        'image' => 'updateImage',
    ]);

<a name="authorizing-actions-via-gates"></a>
### Авторизация действий

Чтобы авторизовать действие с помощью шлюзов нужно использовать методы `allows` или `denies`. Обратите внимание, что этим методам не нужно передавать текущего аутентифицированного пользователя. Laravel автоматически подставит текущего пользователя в замыкание шлюза:

    if (Gate::allows('update-post', $post)) {
        // The current user can update the post...
    }

    if (Gate::denies('update-post', $post)) {
        // The current user can't update the post...
    }

Если вы хотите определить, авторизован ли конкретный пользователь для выполнения действия, можно использовать метод `forUser` фасада `Gate`:

    if (Gate::forUser($user)->allows('update-post', $post)) {
        // The user can update the post...
    }

    if (Gate::forUser($user)->denies('update-post', $post)) {
        // The user can't update the post...
    }

<a name="creating-policies"></a>
## Создание политик

<a name="generating-policies"></a>
### Генерация политик

Политики являются классами, организующими логику авторизации вокруг конкретной модели или ресурса. Например, если ваше приложение является блогом, у вас может быть модель `Post` и соответствующая политика `PostPolicy` для авторизации действия пользователя, таких как создание или обновление постов.

Вы можете создать политику, используя [artisan команду](/docs/{{version}}/artisan) `make:policy`. Сформированная политика будет помещена в директорию `app/Policies`. Если этой директории не существует в приложении, Laravel создаст её:

    php artisan make:policy PostPolicy

Команда `make:policy` создаст пустой класс политики. Если вы хотите создать класс c базовыми "CRUD" методами уже включенными в политику, можно указать опцию `--model` при выполнении команды:

    php artisan make:policy PostPolicy --model=Post

> {tip} Все политики создаются через [сервис контейнер](/docs/{{version}}/container) Laravel, позволяя указывать в качестве аргументов любые необходимые зависимости в конструкторе политики, чтобы они внедрялись автоматически.

<a name="registering-policies"></a>
### Регистрация политик

После создания политики её необходимо зарегистрировать. `AuthServiceProvider`, входящий в состав свежеустановленного приложения Laravel, содержит свойство `policies`, которое сопоставляет ваши Eloquent модели соответствующим политикам. Регистрация политики будет указывать Laravel какую политику использовать при авторизации действия для данной моделиl:

    <?php

    namespace App\Providers;

    use App\Post;
    use App\Policies\PostPolicy;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * The policy mappings for the application.
         *
         * @var array
         */
        protected $policies = [
            Post::class => PostPolicy::class,
        ];

        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            //
        }
    }

<a name="writing-policies"></a>
## Написание политик

<a name="policy-methods"></a>
### Методы политик

После того, как политика была зарегистрирована, вы можете добавить методы для всех действий, которые она авторизует. Например, давайте определим метод `update`  нашего класса `PostPolicy`, который определяет может ли данный пользователь `User` обновить данный экземпляр `Post`.

Метод `update` в качестве аргументов получит `User` и экземпляр `Post`, и он должен вернуть `true` или `false`, что указывает имеет ли пользователь право обновлять данный `Post`. В данном примере давайте проверим, что `id` пользователя соответствует `user_id` поста:

    <?php

    namespace App\Policies;

    use App\User;
    use App\Post;

    class PostPolicy
    {
        /**
         * Determine if the given post can be updated by the user.
         *
         * @param  \App\User  $user
         * @param  \App\Post  $post
         * @return bool
         */
        public function update(User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
    }

Можно продолжать определять дополнительные методы политики по необходимости для различных действий, которые она авторизовывает. Например, вы можете определить методы `view` или `delete` для авторизации различных действий над моделью `Post`, но помните, что можно называть методы своих политик как вам того захочется.

> {tip} Если вы использовали опцию `--model` при создании вашей политики из консоли Artisan, она уже будет включать методы для действий `view`, `create`, `update` и `delete`.

<a name="methods-without-models"></a>
### Методы без моделей

Некоторые методы политики получают только текущего аутентифицированного пользователя, но не экземпляр модели, действие над которой они авторизуют. Такая ситуация чаще всего встречается при авторизации действий `create`. Например, если вы создаете блог, вы можете проверить, имеет ли пользователь право создавать какие либо посты.

При определении метода политик, которые не получат экземпляр модели, например метода `create`, следует определить метод таким образом, что он будет ожидать на вход только аутентифицированного пользователя::

    /**
     * Determine if the given user can create posts.
     *
     * @param  \App\User  $user
     * @return bool
     */
    public function create(User $user)
    {
        //
    }

<a name="policy-filters"></a>
### Фильтры политик

Для определенных пользователей может потребоваться разрешить все действия в рамках данной политики. Для достижения этой цели определите в политике метод `before`. Метод `before` будет выполняться перед любыми другими методами политики - это даст вам возможность авторизовать действия до того как будет выполнен конкретный метод политики. Эта функция наиболее часто используется для авторизации выполнения каких-либо действий администратором приложения:

    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }

Если вы хотите запретить все авторизации пользователя, следует вернуть `false` из метода `before`. Если возвращается `null`, авторизация "проскочит" через метод политики.

<a name="authorizing-actions-using-policies"></a>
## Авторизация действий с помощью политик

<a name="via-the-user-model"></a>
### Через модель пользователя

Модель `User`, которая поставляется с Laravel, включает в себя два полезных метода для авторизации действий: `can` и `cant`. Метод `can` принимает действие, которое вы хотите разрешить, и соответствующую модель. Например, давайте определим имеет ли пользователь право обновлять данную модель `Post`:

    if ($user->can('update', $post)) {
        //
    }

Если для данной модели [зарегистрирована политика](#registering-policies), метод `can` будет автоматически вызывать соответствующую политику и вернет булевое (логическое) значение. Если для данной модели политика не зарегистрирована метод `can` попытается вызвать замыкание шлюза, соответствующее данному названию действия.

#### Действия которые не требуют моделей

Помните, некоторые действия, такие как `create`, не могут требовать экземпляр модели. В таких ситуациях, вы можете передать имя класса в метод `can`. Имя класса будет использоваться для определения того, какие политики использовать при авторизации действия:

    use App\Post;

    if ($user->can('create', Post::class)) {
        // Executes the "create" method on the relevant policy...
    }

<a name="via-middleware"></a>
### Через посредников

Laravel включает в себя посредников, которые могут авторизовать действия до того, как входящий запрос достигнет маршрутов или контроллеров. По умолчанию, в классе `App\Http\Kernel` посреднику `Illuminate\Auth\Middleware\Authorize` назначен ключ `can`. Давайте рассмотрим пример использования посредника `can` для проверки того, что он может обновлять пост в блоге:

    use App\Post;

    Route::put('/post/{post}', function (Post $post) {
        // The current user may update the post...
    })->middleware('can:update,post');

В этом примере мы передаем в посредник `can` два аргумента. Первый - это имя действия, которое мы хотим разрешить, а второй - имя параметра маршрута, который мы хотим передать методу политики. В этом случае, так как мы используем [неявную привязку модели](/docs/{{version}}/routing#implicit-binding), модель `Post` будет передана методу политики. Если пользователь не имеет права выполнить данное действие, посредником будет сгенерирован HTTP-ответ с кодом `403`.

#### Действия которые не требуют моделей

Опять же, некоторые действия, такие как `create`, не могут требовать экземпляр модели. В таких ситуациях, вы можете передать в посредник имя класса. Имя класса будет использоваться для определения того, какие политики будут использоваться при авторизации действия:

    Route::post('/post', function () {
        // The current user may create posts...
    })->middleware('can:create,App\Post');

<a name="via-controller-helpers"></a>
### Через хелперы контроллера

В дополнение к полезным методам, предусмотренным в модели `User`, Laravel предоставляет еще один полезный метод `authorize` в любом из контроллеров, которые наследуют базовый класс `App\Http\Controllers\Controller`. Как и метод `can`, этот метод принимает имя действия вы хотите разрешить и соответствующую модель. Если действие не разрешено, то метод `authorize` выбросит исключение `Illuminate\Auth\Access\AuthorizationException`, которое обработчик исключений Laravel по умолчанию преобразует в ответ HTTP с кодом статуса `403`:

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given blog post.
         *
         * @param  Request  $request
         * @param  Post  $post
         * @return Response
         */
        public function update(Request $request, Post $post)
        {
            $this->authorize('update', $post);

            // The current user can update the blog post...
        }
    }

#### Действия которые не требуют моделей

Как обсуждалось ранее, некоторые действия, например `create`, не могут требовать экземпляр модели. В таких случаях, вы можете передать имя класса в метод `authorize`. Имя класса будет использоваться для определения того, какие политики будут использоваться при авторизации действия:

    /**
     * Create a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function create(Request $request)
    {
        $this->authorize('create', Post::class);

        // The current user can create blog posts...
    }

<a name="via-blade-templates"></a>
### Через шаблоны Blade

При написании шаблонов Blade, вам может понадобиться отобразить часть страницы только если пользователь авторизован выполнить данное действие. Например, вы можете показать форму обновления поста в блоге, только если пользователь действительно может обновить пост. В этом случае, вы можете использовать директивы `@can` и `@cannot`:

    @can('update', $post)
        <!-- The Current User Can Update The Post -->
    @elsecan('create', $post)
        <!-- The Current User Can Create New Post -->
    @endcan

    @cannot('update', $post)
        <!-- The Current User Can't Update The Post -->
    @elsecannot('create', $post)
        <!-- The Current User Can't Create New Post -->
    @endcannot

Эти директивы являются удобными краткими записями заявлений `@if` и `@unless`. Директивы `@can` и `@cannot` можно разложить следующим образом:

    @if (Auth::user()->can('update', $post))
        <!-- The Current User Can Update The Post -->
    @endif

    @unless (Auth::user()->can('update', $post))
        <!-- The Current User Can't Update The Post -->
    @endunless

#### Действия которые не требуют моделей

Как и большинство других методов авторизации, вы можете передать имя класса в директивы `@can` и `@cannot`, если действие не требует экземпляра модели:

    @can('create', App\Post::class)
        <!-- The Current User Can Create Posts -->
    @endcan

    @cannot('create', App\Post::class)
        <!-- The Current User Can't Create Posts -->
    @endcannot
