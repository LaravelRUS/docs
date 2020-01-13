git 131b0d33e10ac5516abb203f5744eb75f17697c7

---

# Хелперы

- [Введение](#introduction)
- [Доступные методы](#available-methods)

<a name="introduction"></a>
## Введение

Laravel включает множество глобальных вспомогательных PHP-функций ("хелперов"). Большинство таких функций используются самим фреймворком; однако, вы можете использовать их в собственных приложениях, если сочтете их удобными.

<a name="available-methods"></a>
## Доступные методы

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

### Массивы

<div class="collection-method-list" markdown="1">

[array_add](#method-array-add)
[array_collapse](#method-array-collapse)
[array_divide](#method-array-divide)
[array_dot](#method-array-dot)
[array_except](#method-array-except)
[array_first](#method-array-first)
[array_flatten](#method-array-flatten)
[array_forget](#method-array-forget)
[array_get](#method-array-get)
[array_has](#method-array-has)
[array_last](#method-array-last)
[array_only](#method-array-only)
[array_pluck](#method-array-pluck)
[array_prepend](#method-array-prepend)
[array_pull](#method-array-pull)
[array_set](#method-array-set)
[array_sort](#method-array-sort)
[array_sort_recursive](#method-array-sort-recursive)
[array_where](#method-array-where)
[array_wrap](#method-array-wrap)
[head](#method-head)
[last](#method-last)
</div>

### Пути

<div class="collection-method-list" markdown="1">

[app_path](#method-app-path)
[base_path](#method-base-path)
[config_path](#method-config-path)
[database_path](#method-database-path)
[mix](#method-mix)
[public_path](#method-public-path)
[resource_path](#method-resource-path)
[storage_path](#method-storage-path)

</div>

### Строки

<div class="collection-method-list" markdown="1">

[camel_case](#method-camel-case)
[class_basename](#method-class-basename)
[e](#method-e)
[ends_with](#method-ends-with)
[kebab_case](#method-kebab-case)
[snake_case](#method-snake-case)
[str_limit](#method-str-limit)
[starts_with](#method-starts-with)
[str_after](#method-str-after)
[str_before](#method-str-before)
[str_contains](#method-str-contains)
[str_finish](#method-str-finish)
[str_is](#method-str-is)
[str_plural](#method-str-plural)
[str_random](#method-str-random)
[str_singular](#method-str-singular)
[str_slug](#method-str-slug)
[studly_case](#method-studly-case)
[title_case](#method-title-case)
[trans](#method-trans)
[trans_choice](#method-trans-choice)

</div>

### URL

<div class="collection-method-list" markdown="1">

[action](#method-action)
[asset](#method-asset)
[secure_asset](#method-secure-asset)
[route](#method-route)
[secure_url](#method-secure-url)
[url](#method-url)

</div>

### Прочее

<div class="collection-method-list" markdown="1">

[abort](#method-abort)
[abort_if](#method-abort-if)
[abort_unless](#method-abort-unless)
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[cache](#method-cache)
[collect](#method-collect)
[config](#method-config)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[dd](#method-dd)
[dispatch](#method-dispatch)
[env](#method-env)
[event](#method-event)
[factory](#method-factory)
[info](#method-info)
[logger](#method-logger)
[method_field](#method-method-field)
[old](#method-old)
[redirect](#method-redirect)
[report](#method-report)
[request](#method-request)
[response](#method-response)
[retry](#method-retry)
[session](#method-session)
[tap](#method-tap)
[value](#method-value)
[view](#method-view)

</div>

<a name="method-listing"></a>
## Список методов

<style>
    #collection-method code {
        font-size: 14px;
    }

    #collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="arrays"></a>
## Массивы

<a name="method-array-add"></a>
#### `array_add()` {#collection-method .first-collection-method}

Функция `array_add` добавляет указанную пару ключ/значение в массив, если она там еще не существует:

    $array = array_add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-collapse"></a>
#### `array_collapse()` {#collection-method}

Функция `array_collapse` собирает массив массивов в единый массив:

    $array = array_collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-divide"></a>
#### `array_divide()` {#collection-method}

Функция `array_divide` возвращает два массива — один с ключами, другой со значениями оригинального массива:

    list($keys, $values) = array_divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `array_dot()` {#collection-method}

Функция `array_dot` делает многоуровневый массив одноуровневым, объединяя вложенные массивы с помощью точки в именах:

    $array = array_dot(['foo' => ['bar' => 'baz']]);

    // ['foo.bar' => 'baz'];

<a name="method-array-except"></a>
#### `array_except()` {#collection-method}

Функция `array_except` удаляет указанную пару ключ/значение из массива:

    $array = ['name' => 'Desk', 'price' => 100];

    $array = array_except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-first"></a>
#### `array_first()` {#collection-method}

Функция `array_first` возвращает первый элемент массива, удовлетворяющий требуемому условию:

    $array = [100, 200, 300];

    $value = array_first($array, function ($value, $key) {
        return $value >= 150;
    });

    // 200

Третьим параметром можно передать значение по умолчанию на случай, если ни одно значение не пройдет условие:

    $value = array_first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `array_flatten()` {#collection-method}

Функция `array_flatten` сделает многоуровневый массив плоским.

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $array = array_flatten($array);

    // ['Joe', 'PHP', 'Ruby'];

<a name="method-array-forget"></a>
#### `array_forget()` {#collection-method}

Функция `array_forget` удалит указанную пару ключ/значение из многоуровневого массива, используя синтаксис имени с точкой:

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `array_get()` {#collection-method}

Функция `array_get` вернет значение из многоуровневого массива, используя синтаксис имени с точкой:

    $array = ['products' => ['desk' => ['price' => 100]]];

    $value = array_get($array, 'products.desk');

    // ['price' => 100]

Также третьим аргументом функции `array_get` можно передать значение по умолчанию на случай, если указанный ключ не будет найден:

    $value = array_get($array, 'names.john', 'default');

<a name="method-array-has"></a>
#### `array_has()` {#collection-method}

Функция `array_has` проверяет существование данного элемента или элементов в массиве, используя синтаксис имени с точкой:

    $array = ['product' => ['name' => 'desk', 'price' => 100]];

    $hasItem = array_has($array, 'product.name');

    // true

    $hasItems = array_has($array, ['product.price', 'product.discount']);

    // false

<a name="method-array-last"></a>
#### `array_last()` {#collection-method}

Функция `array_last` возвращает последний элемент массива, удовлетворяющий требуемому условию:

    $array = [100, 200, 300, 110];

    $value = array_last($array, function ($value, $key) {
        return $value >= 150;
    });

    // 300

<a name="method-array-only"></a>
#### `array_only()` {#collection-method}

Функция `array_only` вернет из массива только указанные пары ключ/значения:

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $array = array_only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `array_pluck()` {#collection-method}

Функция `array_pluck` извлекает значения из многоуровневого массива, соответствующие переданному ключу:

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $array = array_pluck($array, 'developer.name');

    // ['Taylor', 'Abigail'];

Также вы можете указать ключ для полученного списка:

    $array = array_pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail'];

<a name="method-array-prepend"></a>
#### `array_prepend()` {#collection-method}

Функция `array_prepend` поместит элемент в начало массива:

    $array = ['one', 'two', 'three', 'four'];

    $array = array_prepend($array, 'zero');

    // $array: ['zero', 'one', 'two', 'three', 'four']

<a name="method-array-pull"></a>
#### `array_pull()` {#collection-method}

Функция `array_pull` извлечет значения из многоуровневого массива, соответствующие переданному ключу, и удалит их:

    $array = ['name' => 'Desk', 'price' => 100];

    $name = array_pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

<a name="method-array-set"></a>
#### `array_set()` {#collection-method}

Функция `array_set` установит значение в многоуровневом массиве, используя синтаксис имени с точкой:

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-sort"></a>
#### `array_sort()` {#collection-method}

Функция `array_sort` отсортирует массив по результатам вызовов переданной функции-замыкания:

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Chair'],
    ];

    $array = array_values(array_sort($array, function ($value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
        ]
    */

<a name="method-array-sort-recursive"></a>
#### `array_sort_recursive()` {#collection-method}

Функция `array_sort_recursive` рекурсивно сортирует массив с помощью функции `sort`:

    $array = [
        [
            'Roman',
            'Taylor',
            'Li',
        ],
        [
            'PHP',
            'Ruby',
            'JavaScript',
        ],
    ];

    $array = array_sort_recursive($array);

    /*
        [
            [
                'Li',
                'Roman',
                'Taylor',
            ],
            [
                'JavaScript',
                'PHP',
                'Ruby',
            ]
        ];
    */

<a name="method-array-where"></a>
#### `array_where()` {#collection-method}

Функция `array_where` фильтрует массив с помощью переданной функции-замыкания:

    $array = [100, '200', 300, '400', 500];

    $array = array_where($array, function ($value, $key) {
        return is_string($value);
    });

    // [1 => 200, 3 => 400]

<a name="method-array-wrap"></a>
#### `array_wrap()` {#collection-method}

Функция `array_wrap` поместит заданное значение в массив. Если это значение уже в массиве, то функция никак его не изменит:

    $string = 'Laravel';

    $array = array_wrap($string);

    // [0 => 'Laravel']

<a name="method-head"></a>
#### `head()` {#collection-method}

Функция `head` вернет первый элемент массива:

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()` {#collection-method}

Функция `last` вернет последний элемент массива:

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="paths"></a>
## Пути

<a name="method-app-path"></a>
#### `app_path()` {#collection-method}

Функция `app_path` возвращает полный путь к директории `app`. Также можно использовать функцию `app_path` для получения полного пути к указанному файлу относительно каталога приложения:

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {#collection-method}

Функция `base_path` возвращает полный путь к корневой папке приложения. Также можно использовать функцию `base_path` для получения полного пути к указанному файлу относительно корня проекта:

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {#collection-method}

Функция `config_path` возвращает полный путь к папке настройки приложения:

    $path = config_path();

<a name="method-database-path"></a>
#### `database_path()` {#collection-method}

Функция `database_path` возвращает полный путь к папке базы данных приложения:

    $path = database_path();

<a name="method-mix"></a>
#### `mix()` {#collection-method}

Функция `mix` получает путь к [версионированному файлу Mix](/docs/{{version}}/mix):

    mix($file);

<a name="method-public-path"></a>
#### `public_path()` {#collection-method}

Функция `public_path` возвращает полный путь к папке `public`:

    $path = public_path();

<a name="method-resource-path"></a>
#### `resource_path()` {#collection-method}

Функция `resource_path` возвращает полный путь к папке `resources`. Также можно использовать функцию `resource_path` для получения полного пути к указанному файлу относительно каталога с ресурсами:

    $path = resource_path();

    $path = resource_path('assets/sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()` {#collection-method}

Функция `storage_path` возвращает полный путь к папке `storage`. Также можно использовать функцию `storage_path` для получения полного пути к указанному файлу относительно каталога хранилища:

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## Строки

<a name="method-camel-case"></a>
#### `camel_case()` {#collection-method}

Функция `camel_case` преобразует строку в `camelCase`:

    $camel = camel_case('foo_bar');

    // fooBar

<a name="method-class-basename"></a>
#### `class_basename()` {#collection-method}

`class_basename` возвращает имя переданного класса без пространства имен:

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` {#collection-method}

Функция `e` выполняет функцию PHP `htmlspecialchars` с опцией `double_encode`, равной `false`:

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-ends-with"></a>
#### `ends_with()` {#collection-method}

Функция `ends_with` определяет заканчивается ли строка переданной подстрокой:

    $value = ends_with('This is my name', 'name');

    // true

<a name="method-kebab-case"></a>
#### `kebab_case()` {#collection-method}

Функция `kebab_case` преобразует строку в `kebab-case`:

    $value = kebab_case('fooBar');

    // foo-bar


<a name="method-snake-case"></a>
#### `snake_case()` {#collection-method}

Функция `snake_case` преобразует строку в `snake_case`:

    $snake = snake_case('fooBar');

    // foo_bar

<a name="method-str-limit"></a>
#### `str_limit()` {#collection-method}

Функция `str_limit` ограничивает число сиволов в строке. Функция принимает строку первым аргументом, а вторым — максимальное число символов:

    $value = str_limit('The PHP framework for web artisans.', 7);

    // The PHP...

<a name="method-starts-with"></a>
#### `starts_with()` {#collection-method}

Функция `starts_with` определяет начинается ли строка с переданной подстроки:

    $value = starts_with('This is my name', 'This');

    // true

<a name="method-str-after"></a>
#### `str_after()` {#collection-method}

Функция `str_after` возвращает все, что содержится в строке после переданной подстроки:

    $value = str_after('This is: a test', 'This is:');

    // ' a test'

<a name="method-str-before"></a>
#### `str_before()` {#collection-method}

Функция `str_before` возвращает все, что содержится в строке до переданной подстроки:

    $value = str_before('Test :it before', ':it before');

    // 'Test '

<a name="method-str-contains"></a>
#### `str_contains()` {#collection-method}

Функция `str_contains` определяет содержит ли строка переданную подстроку:

    $value = str_contains('This is my name', 'my');

    // true

Также вы можете передать массив значений, чтобы определить, содержит ли строка любое из них:

    $value = str_contains('This is my name', ['my', 'foo']);

    // true

<a name="method-str-finish"></a>
#### `str_finish()` {#collection-method}

Функция `str_finish` добавляет одно вхождение подстроки в конец переданной строки, если она уже не заканчивается этим вхождением:

    $string = str_finish('this/string', '/');
    $string2 = str_finish('this/string/', '/');

    // this/string/

<a name="method-str-is"></a>
#### `str_is()` {#collection-method}

Функция `str_is` определяет соответствует ли строка маске. Можно использовать звёздочки (*) как символы подстановки:

    $value = str_is('foo*', 'foobar');

    // true

    $value = str_is('baz*', 'foobar');

    // false

<a name="method-str-plural"></a>
#### `str_plural()` {#collection-method}

Функция `str_plural` преобразовывает слово-строку во множественное число. На данные момент функция поддерживает только английский язык:

    $plural = str_plural('car');

    // cars

    $plural = str_plural('child');

    // children

Вы можете указать число вторым аргументом функции для получения единственного или множественного числа строки:

    $plural = str_plural('child', 2);

    // children

    $plural = str_plural('child', 1);

    // child

<a name="method-str-random"></a>
#### `str_random()` {#collection-method}

Функция `str_random` создает последовательность случайных символов заданной длины. Эта функция использует PHP-функцию `random_bytes`:

    $string = str_random(40);

<a name="method-str-singular"></a>
#### `str_singular()` {#collection-method}

Функция `str_singular` преобразует слово-строку в единственное число. Функция поддерживает только английский язык:

    $singular = str_singular('cars');

    // car

<a name="method-str-slug"></a>
#### `str_slug()` {#collection-method}

Функция `str_slug` генерирует подходящую для URL "заготовку" из переданной строки:

    $title = str_slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-studly-case"></a>
#### `studly_case()` {#collection-method}

Функция `studly_case` преобразует строку в `StudlyCase`:

    $value = studly_case('foo_bar');

    // FooBar

<a name="method-title-case"></a>
#### `title_case()` {#collection-method}

The `title_case` преобразует строку в `Title Case`:

    $title = title_case('хороший заголовок пишется в правильном регистре');

    // Хороший Заголовок Пишется В Правильном Регистре

<a name="method-trans"></a>
#### `trans()` {#collection-method}

Функция `trans` переводит переданную языковую строку с помощью ваших [файлов локализации](/docs/{{version}}/localization):

    echo trans('validation.required'):

<a name="method-trans-choice"></a>
#### `trans_choice()` {#collection-method}

Функция `trans_choice` переводит переданную языковую строку с изменениями:

    $value = trans_choice('foo.bar', $count);

<a name="urls"></a>
## URL

<a name="method-action"></a>
#### `action()` {#collection-method}

Функция `action` генерирует URL для заданного действия контроллера. Вам не надо передавать полное пространство имён в контроллер. Вместо этого передайте имя класса контроллера в пространстве имён `App\Http\Controllers`:

    $url = action('HomeController@getIndex');

Если метод принимает параметры маршрута, вы можете передать их вторым аргументом:

    $url = action('UserController@profile', ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` {#collection-method}

Генерирует URL к ресурсу (изображению и пр.) на основе текущей схемы запроса (HTTP или HTTPS):

    $url = asset('img/photo.jpg');

<a name="method-secure-asset"></a>
#### `secure_asset()` {#collection-method}

Генерирует URL для ресурса с использованием HTTPS:

    echo secure_asset('foo/bar.zip', $title, $attributes = []);

<a name="method-route"></a>
#### `route()` {#collection-method}

Функция `route` генерирует URL для заданного именованного роута:

    $url = route('routeName');

Если роут принимает параметры, вы можете передать их вторым аргументом:

    $url = route('routeName', ['id' => 1]);

По-умолчанию функция `route` генерирует абсолютный URL-адрес. Если вы хотите сгенерировать относительный URL-адрес, можно передать `false` в качестве третьего параметра:

    $url = route('routeName', ['id' => 1], false);

<a name="method-secure-url"></a>
#### `secure_url()` {#collection-method}

Функция `secure_url` генерирует полный HTTPS URL по заданному пути:

    echo secure_url('user/profile');

    echo secure_url('user/profile', [1]);

<a name="method-url"></a>
#### `url()` {#collection-method}

Функция `url` генерирует полный URL по заданному пути:

    echo url('user/profile');

    echo url('user/profile', [1]);

Если путь не указан, вернется экземпляр `Illuminate\Routing\UrlGenerator`:

    echo url()->current();
    echo url()->full();
    echo url()->previous();

<a name="miscellaneous"></a>
## Прочее

<a name="method-abort"></a>
#### `abort()` {#collection-method}

Функция `abort` выбрасывает HTTP-исключение, которое будет отображено обработчиком исключений:

    abort(401);

Вы можете передать текст для вывода при ответе с этим исключением:

    abort(401, 'Unauthorized.');

<a name="method-abort-if"></a>
#### `abort_if()` {#collection-method}

Функция `abort_if` выбрасывает HTTP-исключение, если заданное логическое выражение равно `true`:

    abort_if(! Auth::user()->isAdmin(), 403);

<a name="method-abort-unless"></a>
#### `abort_unless()` {#collection-method}

Функция `abort_unless` выбрасывает HTTP-исключение, если заданное логическое выражение равно `false`:

    abort_unless(Auth::user()->isAdmin(), 403);

<a name="method-auth"></a>
#### `auth()` {#collection-method}

Функция `auth` возвращает экземпляр аутентификатора. Вы можете использовать ее вместо фасада `Auth` для удобства:

    $user = auth()->user();

<a name="method-back"></a>
#### `back()` {#collection-method}

Функция `back()` создает отклик-переадресацию на предыдущую страницу пользователя:

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` {#collection-method}

Функция `bcrypt` хеширует переданное значение с помощью Bcrypt. Вы можете использовать ее вместо фасада `Hash`:

    $password = bcrypt('my-secret-password');

<a name="method-cache"></a>
#### `cache()` {#collection-method}

Функцию `cache` можно использовать для получения значений из кэша. Если в кэше нет заданного ключа, будет возвращено необязательное значение по умолчанию:

    $value = cache('key');

    $value = cache('key', 'default');

Вы можете добавить элементы в кэш, передав массив пар ключ/значение. Также вам надо передать количество минут или время, в течение которого кэшированные значения будут считаться действительными:

    cache(['key' => 'value'], 5);

    cache(['key' => 'value'], Carbon::now()->addSeconds(10));

<a name="method-collect"></a>
#### `collect()` {#collection-method}

Функция `collect` создает экземпляр [коллекции](/docs/{{version}}/collections) из переданного массива:

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {#collection-method}

Функция `config` получает значение переменной из конфигурации. К значениям конфигурации можно обращаться с помощью "точечного" синтаксиса, в котором указывается имя файла и необходимый параметр. Можно указать значение по умолчанию, которое будет возвращено, если параметра не существует:

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

Хелпер `config` можно использовать для задания переменных конфигурации во время выполнения, передав массив пар ключ/значение:

    config(['app.debug' => true]);

<a name="method-csrf-field"></a>
#### `csrf_field()` {#collection-method}

Функция `csrf_field` создаёт скрытое поле ввода HTML `hidden`, содержащее значение CSRF-последовательности. Например, используя [синтаксис Blade](/docs/{{version}}/blade):

    {{ csrf_field() }}

<a name="method-csrf-token"></a>
#### `csrf_token()` {#collection-method}

Функция `csrf_token` позволяет получить текущее значение CSRF-последовательности:

    $token = csrf_token();

<a name="method-dd"></a>
#### `dd()` {#collection-method}

Функция `dd` выводит дамп переменных и завершает выполнение скрипта:

    dd($value);

    dd($value1, $value2, $value3, ...);

Если вы не хотите останавливать выполнение скрипта, вместо этого используйте функцию `dump`:

    dump($value);

<a name="method-dispatch"></a>
#### `dispatch()` {#collection-method}

Функция `dispatch` помещает новую задачу в [список задач](/docs/{{version}}/queues) Laravel:

    dispatch(new App\Jobs\SendEmails);

<a name="method-env"></a>
#### `env()` {#collection-method}

Функция `env` позволяет получить значение переменной среды или вернуть значение по умолчанию:

    $env = env('APP_ENV');

    // Return a default value if the variable doesn't exist...
    $env = env('APP_ENV', 'production');

<a name="method-event"></a>
#### `event()` {#collection-method}

Функция `event` отправляет указанное [событие](/docs/{{version}}/events) его слушателям:

    event(new UserRegistered($user));

<a name="method-factory"></a>
#### `factory()` {#collection-method}

Функция `factory` создает построитель фабрики моделей для данного класса, имени и количества. Его можно использовать при [тестировании](/docs/{{version}}/database-testing#writing-factories) или [заполнении БД](/docs/{{version}}/seeding#using-model-factories):

    $user = factory(App\User::class)->make();

<a name="method-info"></a>
#### `info()` {#collection-method}

Функция `info` будет записывать информацию в журнал:

    info('Некая полезная информация!');

В функцию можно передать массив контекстных данных:

    info('Неудачная попытка входа пользователя.', ['id' => $user->id]);

<a name="method-logger"></a>
#### `logger()` {#collection-method}

Функцию `logger` можно использовать, чтобы записать в журнал сообщение уровня `debug`:

    logger('Отладочное сообщение');

В функцию можно передать массив контекстных данных:

    logger('Вход пользователя.', ['id' => $user->id]);

Если в функцию не переданы значения, будет возвращен экземпляр [логгера](/docs/{{version}}/errors#logging):

    logger()->error('Вам сюда нельзя.');

<a name="method-method-field"></a>
#### `method_field()` {#collection-method}

Функция `method_field` создаёт скрытое поле ввода HTML `hidden`, содержащее подмененное значение HTTP-типа формы. Например, используя [синтаксис Blade](/docs/{{version}}/blade):

    <form method="POST">
        {{ method_field('DELETE') }}
    </form>

<a name="method-old"></a>
#### `old()` {#collection-method}

Функция `old` [получает](/docs/{{version}}/requests#retrieving-input) значение "старого" ввода, переданного в сессию:

    $value = old('value');

    $value = old('value', 'default');

<a name="method-redirect"></a>
#### `redirect()` {#collection-method}

Функция `redirect` возвращает HTTP-отклик переадресации, или экземпляр переадресатора, если вызывается без аргументов:

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-report"></a>
#### `report()` {#collection-method}

Функция `report` сообщит об исключении, используя метод` report` обработчика исключения:

    report($e);

<a name="method-request"></a>
#### `request()` {#collection-method}

Функция `request` возвращает экземпляр текущего [запроса](/docs/{{version}}/requests) или получает элемент ввода:

    $request = request();

    $value = request('key', $default = null)

<a name="method-response"></a>
#### `response()` {#collection-method}

Функция `response` создает экземпляр [ответа](/docs/{{version}}/responses) или получает экземпляр фабрики ответов:

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-retry"></a>
#### `retry()` {#collection-method}

Функция `retry` пытается выполнить заданную функцию до тех пор, пока не будет достигнут заданный максимальный порог попыток. Если функция не бросает исключение, возвращается возвращаемое ею значение. Если функция бросает исключение, она будет автоматически выполнена еще раз. Если превышено максимальное количество попыток, будет брошено исключение:

    return retry(5, function () {
        // максимум 5 попыток на выполнение с паузой 100мс между попытками
    }, 100);

<a name="method-session"></a>
#### `session()` {#collection-method}

Функция `session` используется для получения или задания значений сессии:

    $value = session('key');

Вы можете задать значения, передав массив пар ключ/значение в функцию:

    session(['chairs' => 7, 'instruments' => 3]);

Если в функцию не было передано значение, то она вернет значения сессии:

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-tap"></a>
#### `tap()` {#collection-method}

Функция `tap` принимает два аргумента: некоторую переменную `$value` и функцию. Эта переменная передается в функцию-аргумент в виде аргумента. Эта же переменная возвращается как результат выполнения хелпера. Возвращаемое значение функции-аргумента игнорируется:

    $user = tap(User::first(), function ($user) {
        $user->name = 'taylor';

        $user->save();
    });

Если функция-аргумент не передается в хелпер, вы можете вызвать любой метод данного `$value`. Возвращаемое значение будет всегда `$value`, несмотря на то, что возвращает этот метод на самом деле. Например, метод Eloquent `update()` возвращает целое число. Но мы при помощи хелпера `tap` можем заставить возвращать его собственно модель:

    $user = tap($user)->update([
        'name' => $name,
        'email' => $email
    ]);

<a name="method-value"></a>
#### `value()` {#collection-method}

Поведение функции `value` просто вернет данное функции значение. Однако, если передать `Closure` функции, будет выполнено `Closure` и возвращен результат:

    $value = value(function () {
        return 'bar';
    });

<a name="method-view"></a>
#### `view()` {#collection-method}

Функция `view` получает экземпляр [шаблона](/docs/{{version}}/views):

    return view('auth.login');
