git 9b8e3b1346f3a205f6d8865b58036f3ee92449d3

---

# Авторизация

- [Введение](#introduction)
- [Установка правил авторизации](#defining-abilities)
- [Проверка правил авторизации](#checking-abilities)
	- [при помощи фасада Gate](#via-the-gate-facade)
	- [при помощи модели User](#via-the-user-model)
	- [внутри шаблонов Blade](#within-blade-templates)
    - [при валидации запросов](#within-form-requests)
- [Политики](#policies)
	- [Создание политик](#creating-policies)
	- [Написание политик](#writing-policies)
	- [Проверка политик](#checking-policies)
- [Авторизация в контроллерах](#controller-authorization)

<a name="introduction"></a>
## Введение

Вместе [аутентификацией](/docs/{{version}}/authentication) пользователей в Laravel есть средства авторизации, т.е. средства управления доступом к различным ресурсам приложения.

> **Примечание:** Авторизация появилась в Laravel 5.1.11, если вы используете версию ниже - обновитесь, как описано в [мануале](/docs/{{version}}/upgrade).

<a name="defining-abilities"></a>
## Установка правил авторизации

Самый простой способ определить, есть ли права на некоторую операцию у данного пользователя - определить правило авторизации ("ability", "возможность"), используя класс `Illuminate\Auth\Access\Gate`. `AuthServiceProvider`, который идет с Laravel - подходящее для этого место. 

Давайте, например, определим правило `update-post`, коотрое будет определять, имеет ли заданный пользователь возможность редактировать заданный пост. На вход оно будет принимать, очевидно, модели `User` и `Post`:

	<?php

	namespace App\Providers;

	use Illuminate\Contracts\Auth\Access\Gate as GateContract;
	use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

	class AuthServiceProvider extends ServiceProvider
	{
	    /**
	     * Register any application authentication / authorization services.
	     *
	     * @param  \Illuminate\Contracts\Auth\Access\Gate  $gate
	     * @return void
	     */
	    public function boot(GateContract $gate)
	    {
	        parent::registerPolicies($gate);

	        $gate->define('update-post', function ($user, $post) {
	        	return $user->id === $post->user_id;
	        });
	    }
	}

Если операция разрешена (в нашем случае если id пользователя совпадает с user_id поста), то операцию разрешаем, возвращая true.
Если запрещена - правило должно возвращать false.

Заметьте, что мы не проверяем, залогинен ли пользователь, существует ли объект `$user`. Laravel проверяет это автоматически и возвращает false для всех правил, где объект User не определён, включая метод `forUser`.

#### Определение метода в классе

Помимо функции-замыкания, вы можете определить класс и метод для реализации правила:

    $gate->define('update-post', 'PostPolicy@update');

<a name="checking-abilities"></a>
## Проверка правил

<a name="via-the-gate-facade"></a>
### при помощи фасада Gate

Как только правило определено, мы можем проверить его на соотвествие. Во-перых, мы можем воспользоваться методами `check`, `allows` и `denies` [фасада](/docs/{{version}}/facades) `Gate`. Эти методы принимают на вход имя правила и аргументы, которые должны быть переданы правилу для обработки. **Нет необходимости** явно передавать текущего юзера - если среди аргументов есть объект модели User, фреймворк сам подставит туда экземпляр модели залогиненого в данный момент юзера. 

    <?php

    namespace App\Http\Controllers;

    use Gate;
    use App\User;
    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
        	$post = Post::findOrFail($id);

        	if (Gate::denies('update-post', $post)) {
        		abort(403);
        	}

        	// Update Post...
        }
    }

Метод `denies()` возвращает true, если операция запрещена, иначе false.
Метод `allows()` позвращает true, если операция разрешена, иначе false.
Метод `check()` является алиасом метода `allows()`, делает то же самое.

#### Проверка правил для определённых пользователей

Чтобы явно задать пользователя, по отношению к которому вы проверяете правило авторизации, воспользуйтесь методом `forUser()`:

	if (Gate::forUser($user)->allows('update-post', $post)) {
		//
	}

#### Передача нескольких аргументов

Of course, ability callbacks may receive multiple arguments:

Если в правило вам нужно передать не два аргумента (пользователь и некая сущность), а несколько, то в определении правила перечислите аргументы обычным способом:

	Gate::define('delete-comment', function ($user, $post, $comment) {
		//
	});

... а в вызове проверки правила подайте соответствующие объекты в виде массива:

	if (Gate::allows('delete-comment', [$post, $comment])) {
		//
	}

<a name="via-the-user-model"></a>
### при помощи модели User

Другой способ проверить правила - воспользоваться методами `can` и `cannot` класса `User`. Эти методы появляются, если подключить к классу трейт `Authorizable` и они идентичны методам `allows()` и `denies()` класса `Gate`. Вот предыдущий пример, переписанный с использованием этих методов:

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  int  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
        	$post = Post::findOrFail($id);

        	if ($request->user()->cannot('update-post', $post)) {
        		abort(403);
        	}

        	// Update Post...
        }
    }

А вот так выглядит секция проверки с использованием метода `can()`:

	if ($request->user()->can('update-post', $post)) {
		// Update Post...
	}

<a name="within-blade-templates"></a>
### внутри шаблонов Blade

For convenience, Laravel provides the `@can` Blade directive to quickly check if the currently authenticated user has a given ability. For example:

Для проверки правил для текущего залогиненого пользователя в шаблонах Blade можно использовать директивe `@can`:

	<a href="/post/{{ $post->id }}">View Post</a>

	@can('update-post', $post)
		<a href="/post/{{ $post->id }}/edit">Edit Post</a>
	@endcan

You may also combine the `@can` directive with `@else` directive:

	@can('update-post', $post)
		<!-- The Current User Can Update The Post -->
	@else
		<!-- The Current User Can't Update The Post -->
	@endcan

<a name="within-form-requests"></a>
### при валидации запросов

Проверку правил авторизации можно проводить в процессе [валидации запросов](/docs/{{version}}/validation#form-request-validation), в методе `authorize()`. Делать это можно при помощи класса `Gate`:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        $postId = $this->route('post');

        return Gate::allows('update', Post::findOrFail($postId));
    }

<a name="policies"></a>
## Политики

<a name="creating-policies"></a>
### Создание политик

Прописывать правила для каждой сущности в `AuthServiceProvider` может быть утомительно, а в случае большого приложения - еще и слабочитаемо. Для наведения порядка в этой области Laravel предлагает так называемые классы политик (policies). В них правила доступа сгруппированы по каждой сущности.

Например, сделаем класс политики для сущности `Post`. Для создания класса воспользуемся [артизан-командой](/docs/{{version}}/artisan) `make:policy`. Она создаст класс политики в папке `app/Policies`:

	php artisan make:policy PostPolicy

#### Регистрация политик

Как только класс политики создан, его надо зарегистрировать. Для этого добавьте его в свойство `policies` класса `AuthServiceProvider` вместе с сущностью, доступ к которой он призван регулировать:

	<?php

	namespace App\Providers;

	use App\Post;
	use App\Policies\PostPolicy;
	use Illuminate\Contracts\Auth\Access\Gate as GateContract;
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
	}

<a name="writing-policies"></a>
### Написание политик

Теперь, после создания и регистрации класса политик, мы можем приступить к написанию правил авторизации. Давайте опишем правило `update`, которое будет определять имеет залогиненый пользователь права для редактирования поста, или нет.

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

> **Примечание:** Классы политик создаются из [сервис-контейнера](/docs/{{version}}/container) Laravel, что означает, что вы можете указывать в аргументах конструктора нужные вам классы - они будут поданы на вход автоматически.

<a name="checking-policies"></a>
### Проверка политик

Проверка правил классов политик фактически не отличается от проверки правил, заданных в функциях-замыканиях. Вы можете использовать фасад `Gate`, метод `can()` модели User, директиву Blade `@can`. Плюс вы можете использовать специальный хелпер для класса политик. 

#### при помощи фасада Gate

Фасад `Gate` автоматичести распознает тип объекта, который подается в качестве аргумента, и выбирает соответствующий класс политик, если таковой зарегистрирован. Имя правила для проверки, которое подается первым аргументом - это название метода класса политик, который будет вызывать `Gate`.

    <?php

    namespace App\Http\Controllers;

    use Gate;
    use App\User;
    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
        	$post = Post::findOrFail($id);

        	if (Gate::denies('update', $post)) {
        		abort(403);
        	}

        	// Update Post...
        }
    }

#### при помощи модели User

То же самое работает в отношении методов `can()` и `cannot()` модели `User`. Если для объекта, который указывается во втором аргументе, зарегистрирован класс политики, правило для проверки (метод класса политики) берется оттуда.

	if ($user->can('update', $post)) {
		//
	}

	if ($user->cannot('update', $post)) {
		//
	}

#### внутри шаблонов Blade

То же самое работает в отношении директивы Blade `@can`. Если для объекта, который указывается во втором аргументе, зарегистрирован класс политики, правило для проверки (метод класса политики) берется оттуда.

	@can('update', $post)
		<!-- The Current User Can Update The Post -->
	@endcan

#### при помощи хелпера политики

При помощи глобального хелпера `policy()` можно получить инстанс класса политики, связанного с заданным объектом (в нешам случае - с объектом $post) и вызвать метод проверки правила авторизации напрямую:

	if (policy($post)->update($user, $post)) {
		//
	}

<a name="controller-authorization"></a>
## Авторизация в контроллере

Класс `App\Http\Controllers\Controller`, от которого по умолчанию наследуются все контроллеры приложения, имеет трейт `AuthorizesRequests`. Этот трейт предоставляет метод `authorize()`, который может использоваться для проверки правила авторизации и выброса эксепшна `HttpException` в случае неудачи.

Метод `authorize()` используется так же, как `Gate::allows` и `$user->can()`, т.е. проверяет, возвращает ли правило `true`.

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
        	$post = Post::findOrFail($id);

        	$this->authorize('update', $post);

        	// Update Post...
        }
    }

В данном примере если действие разрешается, выполнение кода продолжается дальше. Если же правило вернуло false, то бросается исключение `HttpException`, которое генерирует http-ответ с кодом ошибки 403 ("Not Authorized").

Также в трейте `AuthorizesRequests` есть метод `authorizeForUser()`, который осуществляет проверку правила авторизации не по отношению к залогиненному юзеру, а для произвольно заданного:

	$this->authorizeForUser($user, 'update', $post);

#### Автоматическое определение метода класса политики

Часто название правила авторизации совпадает с названием метода контроллера. Например, в нашем примере они оба имеют имя "update". Для таких случаев в Laravel предусмотрено такое поведение - если в `authorize()` не передается строка названия метода, то берется название метода контроллера, из которого вызывается `authorize()`:

    /**
     * Update the given post.
     *
     * @param  int  $id
     * @return Response
     */
    public function update($id)
    {
    	$post = Post::findOrFail($id);

    	$this->authorize($post);

    	// Update Post...
    }

Полный цикл в данном случае выглядит так: Laravel смотрит, какой класс политики зарегистрирован для объекта типа $post (это в нашем случае ), далее смотрит название текущего метода метода контроллера, и вызывает метод `update` класса `PostPolicy`