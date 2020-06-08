git f395270e33bd0af177f2da1f5411a73ccce22de0

---

# Роутинг

- [Простейший роутинг](#basic-routing)
- [Параметры роутов](#route-parameters)
    - [Обязательные параметры](#required-parameters)
    - [Необязательные параметры](#parameters-optional-parameters)
    - [Ограничения регулярными выражениями](#parameters-regular-expression-constraints)
- [Именованные роуты](#named-routes)
- [Группы роутов](#route-groups)
    - [Посредники](#route-group-middleware)
    - [Пространства имён](#route-group-namespaces)
    - [Доменный роутинг](#route-group-sub-domain-routing)
    - [Префиксы роута](#route-group-prefixes)
- [Привязка модели роута](#route-model-binding)
    - [Неявная привязка](#implicit-binding)
    - [Явная привязка](#explicit-binding)
- [Подмена методов](#form-method-spoofing)
- [Получение текущего роута](#accessing-the-current-route)

<a name="basic-routing"></a>
## Простейший роутинг

В Laravel простейшие роуты принимают URI (путь) и функцию-замыкание, предоставляя очень простой и выразительный метод определения роутов:

    Route::get('foo', function () {
        return 'Hello World';
    });

#### Файлы роутов по умолчанию

Все роуты Laravel определены в файлах роутов, которые расположены в директории `routes`. Эти файлы автоматически загружаются фреймворком. В файле `routes/web.php` определены роуты для вашего web-интерфейса. Эти роуты входят в группу посредников `web`, которые обеспечивают такие возможности, как состояние сессии и CSRF-защита. Роуты из файла `routes/api.php` не сохраняют состояние приложения и входят в группу посредников `api`.

Для большинства приложений сначала определяются роуты в файле `routes/web.php`.

#### Доступные методы роутера

Роутер (маршрутизатор) позволяет регистрировать роуты для любого HTTP-запроса:

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

Иногда необходимо зарегистрировать роут, который отвечает на HTTP-запросы нескольких типов. Это можно сделать методом `match`. Или вы можете зарегистрировать роут, отвечающий на HTTP-запросы всех типов, с помощью метода `any`:

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('foo', function () {
        //
    });

#### CSRF-защита

Все HTML-формы, ведущие к роутам `POST`, `PUT` или `DELETE`, которые определены в файле роутов `web` , должны иметь поле CSRF-токена. Иначе запрос будет отклонён. Подробнее о CSRF-защите читайте в [документации CSRF](/docs/{{version}}/csrf):

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

<a name="route-parameters"></a>
## Параметры роутов

<a name="required-parameters"></a>
### Обязательные параметры

Разумеется, иногда вам может понадобиться захватить сегменты URI в вашем роуте. Например, если вам необходимо захватить ID пользователя из URL. Это можно сделать, определив параметры роута:

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

Вы можете определить сколько угодно параметров:

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

Параметры роута всегда заключаются в фигурные скобки `{}` и должны состоять из буквенных символов. Параметры роута не могут содержать символ `-`. Используйте вместо него подчёркивание (`_`). Параметры роута внедряются в анонимные функции / контроллеры роута, основываясь на их порядке - названия аргументов анонимных функций / контроллеров не имеют значения.

<a name="parameters-optional-parameters"></a>
### Необязательные параметры

Иногда необходимо указать параметр роута, но при этом сделать его наличие необязательным. Это можно сделать, поместив знак вопроса `?` после названия параметра. Не забудьте задать значение по умолчанию для соответствующей переменной роута:

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### Ограничения регулярными выражениями

Вы можете ограничить формат параметров вашего роута с помощью метода `where` на экземпляре роута. Метод `where` принимает название параметра и регулярное выражение, определяющее ограничения для параметра:

    Route::get('user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');

    Route::get('user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');

    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

<a name="parameters-global-constraints"></a>
#### Глобальные ограничения

Если вы хотите, чтобы параметр был всегда ограничен заданным регулярным выражением, то можете использовать метод `pattern`. Вам следует определить эти шаблоны в методе `boot` вашего `RouteServiceProvider`:

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');

        parent::boot();
    }

Когда шаблон был определён, он автоматически применится ко всем роутам, использующим этот параметр:

    Route::get('user/{id}', function ($id) {
        // Only executed if {id} is numeric...
    });

<a name="named-routes"></a>
## Именованные роуты

Именованные роуты позволяют удобно генерировать URL-адреса и делать переадресацию на конкретный роут. Вы можете задать имя роута, прицепив метод `name` к определению роута:

    Route::get('user/profile', function () {
        //
    })->name('profile');

Также можно указать имена роутов для действий контроллера:

    Route::get('user/profile', 'UserController@showProfile')->name('profile');

#### Генерирование URL адресов для именованных роутов

Когда вы назначили имя роуту, вы можете использовать это имя для генерирования URL адресов и переадресаций глобальным методом `route`:

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

Если у именованного роута есть параметры, вы можете передать их вторым аргументом метода `route`. Эти параметры будут автоматически вставлены в соответствующие места URL:

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

<a name="route-groups"></a>
## Группы роутов

Группы роутов позволяют использовать общие атрибуты роутов, такие как посредники и пространства имён, для большого числа роутов без необходимости определять эти атрибуты для каждого отдельного роута. Общие атрибуты указываются в виде массива первым аргументом метода `Route::group`.

<a name="route-group-middleware"></a>
### Посредники

Посредники применяются ко всем роутам в группе путём указания списка этих посредников с параметром `middleware` в массиве групповых атрибутов. Посредники выполняются в порядке перечисления в этом массиве:

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // Uses first & second Middleware
        });

        Route::get('user/profile', function () {
            // Uses first & second Middleware
        });
    });

<a name="route-group-namespaces"></a>
### Пространства имён

Другой типичный пример использования групп роутов — назначение одного пространства имён PHP для группы контроллеров, используя параметр `namespace` в массиве группы:

    Route::namespace('Admin')->group(function () {
        // Controllers Within The "App\Http\Controllers\Admin" Namespace
    });

Помните, по умолчанию `RouteServiceProvider` включает ваши файлы роутов в группу пространства имён, позволяя вам регистрировать роуты контроллера без указания полного префикса пространства имён `App\Http\Controllers`. Поэтому нам надо указать лишь ту часть пространства имён, которая следует за базовым пространством имён `App\Http\Controllers`.

<a name="route-group-sub-domain-routing"></a>
### Доменный роутинг

Группы роутов можно использовать для обработки маршрутизации поддоменов. Поддоменам можно назначать параметры роутов также как URI роутов, поэтому вы можете захватить часть поддомена и использовать в своём роуте или контроллере. Поддомен можно указать с помощью ключа `domain` в массиве атрибутов группы:

    Route::domain('{account}.myapp.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### Префиксы роута

Метод `prefix` можно использовать для указания URI-префикса каждого роута в группе. Например, если вы хотите добавить `admin` ко всем URI роутов в группе:

    Route::prefix('admin')->group(function () {
        Route::get('users', function () {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-model-binding"></a>
## Привязка модели роута

При внедрении ID модели в действие роута или контроллера бывает часто необходимо получить модель, соответствующую этому ID. Привязка моделей — удобный способ автоматического внедрения экземпляров модели напрямую в ваши роуты. Например, вместо внедрения ID пользователя вы можете внедрить весь экземпляр модели `User`, который соответствует данному ID.

<a name="implicit-binding"></a>
### Неявная привязка

Laravel автоматически включает модели Eloquent, определённые в действиях роута или контроллера, чьи переменные имеют имена, совпадающие с сегментом роута. Например:

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

Так как переменная `$user` указывается в качестве аргумента Eloquent модели `App\User` и название переменной совпадает с URI-сегментом `{user}`, автоматически внедрит экземпляр модели, который имеет ID, совпадающий с соответствующим значением из URI запроса. Если совпадающий экземпляр модели не найден в базе данных, будет автоматически сгенерирован HTTP-отклик 404.

#### Изменение имени ключа

Если вы хотите, чтобы для получения класса данной модели вместо столбца `id` использовался другой столбец базы данных, вы можете переопределить метод `getRouteKeyName` в своей модели Eloquent:

    /**
     * Get the route key for the model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="explicit-binding"></a>
### Явная привязка

Для регистрации явной привязки используйте метод роута `model` для указания класса для данного параметра. Вам надо определить явные привязки вашей модели в методе `boot` класса `RouteServiceProvider`:

    public function boot()
    {
        parent::boot();

        Route::model('user', App\User::class);
    }

Затем определите роут, содержащий параметр `{user}`:

    Route::get('profile/{user}', function (App\User $user) {
        //
    });

Из-за того, что мы ранее привязали все параметры `{user}` к модели `App\User`, её экземпляр будет внедрён в роут. Таким образом, к примеру, запрос `profile/1` внедрит объект `User` , полученный из БД, который соответствует ID `1`.

Если совпадающий экземпляр модели не найден в базе данных, будет автоматически сгенерирован HTTP-отклик 404.

#### Изменение логики принятия решения

Если захотите использовать собственную логику принятия решения, используйте метод `Route::bind` . Переданное в метод `bind` замыкание получит значение сегмента URI, и должно вернуть экземпляр класса, который вы хотите внедрить в роут:

    public function boot()
    {
        parent::boot();

        Route::bind('user', function ($value) {
            return App\User::where('name', $value)->first();
        });
    }

<a name="form-method-spoofing"></a>
## Подмена методов

HTML-формы не поддерживают действия `PUT`, `PATCH` или `DELETE`. Поэтому при определении роутов `PUT`, `PATCH` или `DELETE`, вызываемых из HTML-формы, вам надо добавить в неё скрытое поле `_method`. Переданное в этом поле значение будет использовано как метод HTTP-запроса:

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

Используйте хелпер `method_field`, чтобы сгенерировать ввод `_method`:

    {{ method_field('PUT') }}

<a name="accessing-the-current-route"></a>
## Получение текущего роута

Используйте методы `current`, `currentRouteName` и `currentRouteAction` фасада `Route` для получения информации о роуте, обрабатывающем входящий запрос:

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

Чтобы изучить все доступные методы, обратитесь к документации по API [класса, лежащего в основе фасада Route](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) и [экземпляра Route](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html).
