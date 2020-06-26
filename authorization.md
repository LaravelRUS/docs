git ebea240ff00e6cdb91d3b2abbcb6c3f718677d6e

---

# Авторизация

- [Введение](#introduction)
- [Шлюзы](#gates)
    - [Написание шлюзов](#writing-gates)
    - [Авторизация действий](#authorizing-actions-via-gates)
    - [Развёрнутые ответы от шлюзов](#gate-responses)
    - [Перехват проверок шлюзов](#intercepting-gate-checks)
- [Создание политик](#creating-policies)
    - [Генерация политик](#generating-policies)
    - [Регистрация политик](#registering-policies)
- [Написание политик](#writing-policies)
    - [Методы политик](#policy-methods)
    - [Развёрнутые ответы политик](#policy-responses)
    - [Методы без моделей](#methods-without-models)
    - [Обработка незалогиненных пользователей](#guest-users)
    - [Фильтры политик](#policy-filters)
- [Авторизация действий с помощью политик](#authorizing-actions-using-policies)
    - [Через модель пользователя](#via-the-user-model)
    - [Через посредников](#via-middleware)
    - [Через хелперы контроллера](#via-controller-helpers)
    - [Через шаблоны Blade](#via-blade-templates)
    - [Передача дополнительного контекста](#supplying-additional-context)

<a name="introduction"></a>
## Введение

В дополнение к изначально предоставленным службам [аутентификации](/docs/{{version}}/authentication), Laravel также предоставляет способ авторизовать действия пользователя в отношении некого ресурса - возможность разрешить или запретить некоторое действие. Как и в случае с аутентификацией, подход Laravel к авторизации прост, и есть два основных способа авторизации действий: шлюзы (gates) и политики (policies).

Думайте о шлюзах и политиках, как о маршрутах и контроллерах. Шлюзы обеспечивают простое решение на основе замыканий, в свою очередь политики, как контроллеры, группируют свою логику вокруг конкретной модели или ресурса. Сначала мы разберем шлюзы, а затем рассмотрим политики.

Важно не рассматривать шлюзы и политики как взаимоисключающие вещи. Большинство приложений, скорее всего, содержат смесь шлюзов и политик, и это прекрасно! Шлюзы наиболее применимы к действиям, которые не связаны с какой-либо моделью или ресурсом, например, просмотр панели администратора. В противоположность этому, политика должна быть использована, если вы хотите разрешить действие для конкретной модели или ресурса.

<a name="gates"></a>
## Шлюзы

<a name="writing-gates"></a>
### Написание шлюзов

Шлюзы (гейты, gates) - это анонимные функции, которые определяют, имеет ли пользователь право выполнить данное действие; они обычно определяются в классе `App\Providers\AuthServiceProvider` с помощью фасада `Gate`. Шлюзы всегда получают экземпляр пользователя в качестве первого аргумента. Также они могут принимать дополнительные аргументы, например, соответствующую модель Eloquent:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('edit-settings', function ($user) {
            return $user->isAdmin;
        });

        Gate::define('update-post', function ($user, $post) {
            return $user->id === $post->user_id;
        });
    }

Шлюзы также можно задать, используя строку стиля `Class@method`, как, например, в роутах:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', 'App\Policies\PostPolicy@update');
    }

<a name="authorizing-actions-via-gates"></a>
### Авторизация действий

Чтобы авторизовать действие с помощью шлюзов нужно использовать методы `allows` или `denies`. Обратите внимание, что этим методам не нужно передавать текущего аутентифицированного пользователя. Laravel автоматически подставит текущего пользователя в функцию шлюза:

    if (Gate::allows('edit-settings')) {
        // The current user can edit settings
    }

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

Можно использовать несколько проверок:

    if (Gate::any(['update-post', 'delete-post'], $post)) {
        // The user can update or delete the post
    }

    if (Gate::none(['update-post', 'delete-post'], $post)) {
        // The user cannot update or delete the post
    }

#### Автоматический выброс исключения

Если вы хотите, чтобы при проверки авторизации в случае неудачи автоматически выбрасывалось исключение `Illuminate\Auth\Access\AuthorizationException`, то используйте метод `Gate::authorize`. `AuthorizationException` будет автоматически преобразовано в HTTP-ответ с кодом `403`

    Gate::authorize('update-post', $post);

    // Действие разрешено...

#### Дополнительный контекст

Методы авторизации (`allows`, `denies`, `check`, `any`, `none`, `authorize`, `can`, `cannot`) и директивы авторизации [Blade](#via-blade-templates)  (`@can`, `@cannot`, `@canany`) вторым аргументом могут принимать массивы. Элементы массива передаются в шлюз в качестве аргументов и могут быть использованы в качестве дополнительного контекста для принятия решения об авторизации действия:

    Gate::define('create-post', function ($user, $category, $extraFlag) {
        return $category->group > 3 && $extraFlag === true;
    });

    if (Gate::check('create-post', [$category, $extraFlag])) {
        // The user can create the post...
    }

<a name="gate-responses"></a>
### Развёрнутые ответы от шлюзов

Ранее мы получали от шлюзов ответ в виде двоичных значений - `true` или `false`. Но иногда нам требуется получить более детальный ответ, с сообщением что конкретно пошло не так. Для этого можно вернуть объект `Illuminate\Auth\Access\Response`:

    use Illuminate\Auth\Access\Response;
    use Illuminate\Support\Facades\Gate;

    Gate::define('edit-settings', function ($user) {
        return $user->isAdmin
                    ? Response::allow()
                    : Response::deny('You must be a super administrator.');
    });

`Gate::allows` будет по прежнему возвращать логическое значение, а чтобы получить подробный ответ от шлюза - можно использовать `Gate::inspect`:

    $response = Gate::inspect('edit-settings', $post);

    if ($response->allowed()) {
        // Действие разрешено
    } else {
        echo $response->message();
    }

В этом варианте `Gate::authorize`, бросая исключение `AuthorizationException` будет добавлять заданный текст ошибки авторизации в HTTP-ответ: 

    Gate::authorize('edit-settings', $post);

    // Действие разрешено

<a name="intercepting-gate-checks"></a>
### Перехват проверок шлюзов

Если вам нужно дать некоторому пользователю (непример, суперадмину) права проходить любые шлюзы, воспользуйтесь методом `before`, который автоматически запускается перед любой проверкой авторизации шлюзами:

    Gate::before(function ($user, $ability) {
        if ($user->isSuperAdmin()) {
            return true;
        }
    });

Если `before` возвращает что-то отличное от `null`, это значение рассматривается как результат проверки.

Также существует метод `after`, он вызывается после прохождения всех проверок: 

    Gate::after(function ($user, $ability, $result, $arguments) {
        if ($user->isBanned()) {
            return false; // если пользователь забанен, любые разрешения на действия являются недействительными
        }
    });

Как и в случае `before` , отличное от `null` возвращаемое значение рассматривается как результат проверки авторизации.

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

    use App\Policies\PostPolicy;
    use App\Post;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    use Illuminate\Support\Facades\Gate;

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

#### Автоматическая регистрация политик

Политики способны регистрироваться автоматически, если соблюдаются стандарты Laravel для наименования и расположения моделей и политик. Политики должны находиться в папке `Policies`, которая находится на уровень ниже папки, в которой находятся модели. Например, если модели находятся в папке `app`, ожидается, что политики будут находиться в папке `app/Policies`. Также имена классов политик должны совпадать с именами моделей с добавлением слова `Policy` в конце. Так, для модели `User` ожидается политика `UserPolicy`.

Вы можете задать свою логику авторегистрации при помощи `Gate::guessPolicyNamesUsing` в методе `boot` сервис-провайдера `AuthServiceProvider`:

    use Illuminate\Support\Facades\Gate;

    Gate::guessPolicyNamesUsing(function ($modelClass) {
        // вернуть класс политики для данной модели
    });

> {note} Политики, связанные с моделями в `AuthServiceProvider`, не участвуют ни в одной схеме авторегистрации. 

<a name="writing-policies"></a>
## Написание политик

<a name="policy-methods"></a>
### Методы политик

После того, как политика была зарегистрирована, вы можете добавить методы для всех действий, которые она авторизует. Например, давайте определим метод `update`  нашего класса `PostPolicy`, который определяет может ли данный пользователь `User` обновить данный экземпляр `Post`.

Метод `update` в качестве аргументов получит `User` и экземпляр `Post`, и он должен вернуть `true` или `false`, что указывает имеет ли пользователь право обновлять данный `Post`. В данном примере давайте проверим, что `id` пользователя соответствует `user_id` поста:

    <?php

    namespace App\Policies;

    use App\Post;
    use App\User;

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

> {tip} Если вы использовали опцию `--model` при создании вашей политики из консоли Artisan, она уже будет включать методы для действий `viewAny`, `view`, `create`, `update`, `delete`, `restore`, и `forceDelete`

<a name="policy-responses"></a>
### Развёрнутые ответы политик

Обычно результат работы политик - ответ в виде значений `true` или `false`. Но иногда нам требуется получить более детальный ответ, с сообщением, что конкретно не так. Для этого можно вернуть объект `Illuminate\Auth\Access\Response`:

    use Illuminate\Auth\Access\Response;

    /**
     * Determine if the given post can be updated by the user.
     *
     * @param  \App\User  $user
     * @param  \App\Post  $post
     * @return \Illuminate\Auth\Access\Response
     */
    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id
                    ? Response::allow()
                    : Response::deny('You do not own this post.');
    }

`Gate::allows` будет по прежнему возвращать логическое значение, а чтобы получить подробный ответ от шлюза - можно использовать `Gate::inspect`:

    $response = Gate::inspect('update', $post);

    if ($response->allowed()) {
        // действие разрешено
    } else {
        echo $response->message();
    }

В этом варианте `Gate::authorize`, бросая исключение `AuthorizationException` будет добавлять заданный текст ошибки авторизации в HTTP-ответ:

    Gate::authorize('update', $post);

    // действие разрешено

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

<a name="guest-users"></a>
### Обработка незалогиненных пользователей

По умолчанию все шлюзы и политики возвращают `false` для незалогиненных пользователей, запрещая любые действия. Вы можете обойти это поведение и разрешить проверку для незалогиненных пользователей, если укажете в аргументах, что вместо экзепляра класса `User` в метод проверки может прийти `null`:

    <?php

    namespace App\Policies;

    use App\Post;
    use App\User;

    class PostPolicy
    {
        /**
         * Determine if the given post can be updated by the user.
         *
         * @param  \App\User  $user
         * @param  \App\Post  $post
         * @return bool
         */
        public function update(?User $user, Post $post)
        {
            return optional($user)->id === $post->user_id;
        }
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

Если вы хотите запретить все авторизации пользователя, следует вернуть `false` из метода `before`. Если возвращается `null`, будут вызваны дальнейшие проверки.

> {note} Если в классе политики нет метода, проверяющего заданное действие, метод `before` не будет вызван.

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

    use Illuminate\Http\Request;
    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given blog post.
         *
         * @param  Request  $request
         * @param  Post  $post
         * @return Response
         * @throws \Illuminate\Auth\Access\AuthorizationException
         */
        public function update(Request $request, Post $post)
        {
            $this->authorize('update', $post);

            // The current user can update the blog post...
        }
    }

#### Действия которые не требуют моделей

Как обсуждалось ранее, некоторые действия, например `create`, не могут требовать экземпляр модели. В таких случаях, вы должны передать имя класса в метод `authorize`. Имя класса будет использоваться для определения того, какие политики будут использоваться при авторизации действия:

    /**
     * Create a new blog post.
     *
     * @param  Request  $request
     * @return Response
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function create(Request $request)
    {
        $this->authorize('create', Post::class);

        // The current user can create blog posts...
    }

#### Авторизация в ресурсных котроллерах

Если вы используете [ресурсные контроллеры](/docs/{{version}}/controllers#resource-controllers), вы можете упростить написание авторизации, использовав `authorizeResource` в конструкторе такого контроллера.

`authorizeResource` принимает название класса модели в качестве первого аргумента и имя роута / параметр запроса, который содержит ID экземпляра этой модели в качестве второго аргумента:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Post;
    use Illuminate\Http\Request;

    class PostController extends Controller
    {
        public function __construct()
        {
            $this->authorizeResource(Post::class, 'post');
        }
    }

Методы контроллера и методы класса политики будут сопоставлены следующим образом:

| Метод контроллера | Метод политики |
| --- | --- |
| index | viewAny |
| show | view |
| create | create |
| store | create |
| edit | update |
| update | update |
| destroy | delete |

> {tip} Вы можете создать класс политики для заданной модели следующей командой: `php artisan make:policy PostPolicy --model=Post`.

<a name="via-blade-templates"></a>
### Через шаблоны Blade

При написании шаблонов Blade, вам может понадобиться отобразить часть страницы только если пользователь авторизован выполнить данное действие. Например, вы можете показать форму обновления поста в блоге, только если пользователь действительно может обновить пост. В этом случае, вы можете использовать директивы `@can` и `@cannot`:

    @can('update', $post)
        <!-- Текущий пользователь может редактировать данный пост -->
    @elsecan('create', App\Post::class)
        <!-- Текущий пользователь может создавать посты -->
    @endcan

    @cannot('update', $post)
        <!-- Текущий пользователь не может редактировать данный пост -->
    @elsecannot('create', $App\Post::class)
        <!-- Текущий пользователь не может создавать посты -->
    @endcannot

Эти директивы являются удобными краткими записями заявлений `@if` и `@unless`. Директивы `@can` и `@cannot` можно разложить следующим образом:

    @if (Auth::user()->can('update', $post))
        <!-- Текущий пользователь может редактировать данный пост -->
    @endif

    @unless (Auth::user()->can('update', $post))
        <!-- Текущий пользователь не может редактировать данный пост -->
    @endunless

Директива `@canany` позволяет показать контент, если выполняется хотя бы одна из заданных проверок: 

    @canany(['update', 'view', 'delete'], $post)
        // Текущий пользователь может просметривать, редактировать или удалять пост
    @elsecanany(['create'], \App\Post::class)
        // Текущий пользователь может создавать посты
    @endcanany

#### Действия которые не требуют моделей

Как и большинство других методов авторизации, вы можете передать имя класса в директивы `@can` и `@cannot`, если действие не требует экземпляра модели:

    @can('create', App\Post::class)
        <!-- The Current User Can Create Posts -->
    @endcan

    @cannot('create', App\Post::class)
        <!-- The Current User Can't Create Posts -->
    @endcannot


<a name="supplying-additional-context"></a>
### Передача дополнительного контекста

Во время вызова проверок авторизации `$this->authorize` второй аргумент может также быть массивом. В этом случае первый элемент массива будет считаться моделью, чья политика будет использоваться, а остальные аргументы могут быть использованы для дополнительных данных, требующихся для проверки. Например, для `PostPolicy` может понадобиться не только сам пост, но и номер категории:

    /**
     * Определить, может ли данный пользователь редактировать данный пост
     *
     * @param  \App\User  $user
     * @param  \App\Post  $post
     * @param  int  $category
     * @return bool
     */
    public function update(User $user, Post $post, int $category)
    {
        return $user->id === $post->user_id &&
               $category > 3;
    }

Вызов проверки авторизации будет выглядеть так:

    /**
     * Редактирование поста
     *
     * @param  Request  $request
     * @param  Post  $post
     * @return Response
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function update(Request $request, Post $post)
    {
        $this->authorize('update', [$post, $request->input('category')]);

        // The current user can update the blog post...
    }