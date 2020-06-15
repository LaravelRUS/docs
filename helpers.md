git 195e65d6a2dd689ae7584478054618ffee94deff

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

### Массивы и объекты

<div class="collection-method-list" markdown="1">

[Arr::add](#method-array-add)
[Arr::collapse](#method-array-collapse)
[Arr::crossJoin](#method-array-crossjoin)
[Arr::divide](#method-array-divide)
[Arr::dot](#method-array-dot)
[Arr::except](#method-array-except)
[Arr::first](#method-array-first)
[Arr::flatten](#method-array-flatten)
[Arr::forget](#method-array-forget)
[Arr::get](#method-array-get)
[Arr::has](#method-array-has)
[Arr::isAssoc](#method-array-isassoc)
[Arr::last](#method-array-last)
[Arr::only](#method-array-only)
[Arr::pluck](#method-array-pluck)
[Arr::prepend](#method-array-prepend)
[Arr::pull](#method-array-pull)
[Arr::random](#method-array-random)
[Arr::query](#method-array-query)
[Arr::set](#method-array-set)
[Arr::shuffle](#method-array-shuffle)
[Arr::sort](#method-array-sort)
[Arr::sortRecursive](#method-array-sort-recursive)
[Arr::where](#method-array-where)
[Arr::wrap](#method-array-wrap)
[data_fill](#method-data-fill)
[data_get](#method-data-get)
[data_set](#method-data-set)
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

[\__](#method-__)
[class_basename](#method-class-basename)
[e](#method-e)
[preg_replace_array](#method-preg-replace-array)
[Str::after](#method-str-after)
[Str::afterLast](#method-str-after-last)
[Str::before](#method-str-before)
[Str::beforeLast](#method-str-before-last)
[Str::camel](#method-camel-case)
[Str::contains](#method-str-contains)
[Str::containsAll](#method-str-contains-all)
[Str::endsWith](#method-ends-with)
[Str::finish](#method-str-finish)
[Str::is](#method-str-is)
[Str::isUuid](#method-str-is-uuid)
[Str::kebab](#method-kebab-case)
[Str::limit](#method-str-limit)
[Str::orderedUuid](#method-str-ordered-uuid)
[Str::plural](#method-str-plural)
[Str::random](#method-str-random)
[Str::replaceArray](#method-str-replace-array)
[Str::replaceFirst](#method-str-replace-first)
[Str::replaceLast](#method-str-replace-last)
[Str::singular](#method-str-singular)
[Str::slug](#method-str-slug)
[Str::snake](#method-snake-case)
[Str::start](#method-str-start)
[Str::startsWith](#method-starts-with)
[Str::studly](#method-studly-case)
[Str::title](#method-title-case)
[Str::ucfirst](#method-str-ucfirst)
[Str::uuid](#method-str-uuid)
[Str::words](#method-str-words)
[trans](#method-trans)
[trans_choice](#method-trans-choice)

</div>

### URL

<div class="collection-method-list" markdown="1">

[action](#method-action)
[asset](#method-asset)
[route](#method-route)
[secure_asset](#method-secure-asset)
[secure_url](#method-secure-url)
[url](#method-url)

</div>

### Прочее

<div class="collection-method-list" markdown="1">

[abort](#method-abort)
[abort_if](#method-abort-if)
[abort_unless](#method-abort-unless)
[app](#method-app)
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[blank](#method-blank)
[broadcast](#method-broadcast)
[cache](#method-cache)
[class_uses_recursive](#method-class-uses-recursive)
[collect](#method-collect)
[config](#method-config)
[cookie](#method-cookie)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[dd](#method-dd)
[decrypt](#method-decrypt)
[dispatch](#method-dispatch)
[dispatch_now](#method-dispatch-now)
[dump](#method-dump)
[encrypt](#method-encrypt)
[env](#method-env)
[event](#method-event)
[factory](#method-factory)
[filled](#method-filled)
[info](#method-info)
[logger](#method-logger)
[method_field](#method-method-field)
[now](#method-now)
[old](#method-old)
[optional](#method-optional)
[policy](#method-policy)
[redirect](#method-redirect)
[report](#method-report)
[request](#method-request)
[rescue](#method-rescue)
[resolve](#method-resolve)
[response](#method-response)
[retry](#method-retry)
[session](#method-session)
[tap](#method-tap)
[throw_if](#method-throw-if)
[throw_unless](#method-throw-unless)
[today](#method-today)
[trait_uses_recursive](#method-trait-uses-recursive)
[transform](#method-transform)
[validator](#method-validator)
[value](#method-value)
[view](#method-view)
[with](#method-with)

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
## Массивы и объекты

<a name="method-array-add"></a>
#### `Arr::add()` {#collection-method .first-collection-method}

Метод `Arr::add()` добавляет заданную пару ключ/значение в массив, если данный ключ еще не существует в массиве или имеет значение `null`.

    use Illuminate\Support\Arr;

    $array = Arr::add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

    $array = Arr::add(['name' => 'Desk', 'price' => null], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-collapse"></a>
#### `Arr::collapse()` {#collection-method}

Метод `Arr::collapse()` объединяет массив массивов в единый массив:
Функция `array_collapse` собирает массив массивов в единый массив:

    use Illuminate\Support\Arr;

    $array = Arr::collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-crossjoin"></a>
#### `Arr::crossJoin()` {#collection-method}

Метод `Arr::crossJoin` перекрёстно объединяет переданные массивы, возвращая декартово произведение всех возможных перестановок:

    use Illuminate\Support\Arr;

    $matrix = Arr::crossJoin([1, 2], ['a', 'b']);

    /*
        [
            [1, 'a'],
            [1, 'b'],
            [2, 'a'],
            [2, 'b'],
        ]
    */

    $matrix = Arr::crossJoin([1, 2], ['a', 'b'], ['I', 'II']);

    /*
        [
            [1, 'a', 'I'],
            [1, 'a', 'II'],
            [1, 'b', 'I'],
            [1, 'b', 'II'],
            [2, 'a', 'I'],
            [2, 'a', 'II'],
            [2, 'b', 'I'],
            [2, 'b', 'II'],
        ]
    */

<a name="method-array-divide"></a>
#### `Arr::divide()` {#collection-method}

Метод `Arr::divide` возвращает два массива, один из которых содержит ключи, а другой — значения переданного массива:

    use Illuminate\Support\Arr;

    [$keys, $values] = Arr::divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `Arr::dot()` {#collection-method}

Метод `Arr::dot` преобразует многомерный массив в одномерный, который использует точку для указания глубины:

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    $flattened = Arr::dot($array);

    // ['products.desk.price' => 100]

<a name="method-array-except"></a>
#### `Arr::except()` {#collection-method}

Метод `Arr::except` удаляет из массива заданные пары ключ/значение:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100];

    $filtered = Arr::except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-first"></a>
#### `Arr::first()` {#collection-method}

Метод `Arr::first` возвращает первый элемент массива, удовлетворяющий требуемому условию:

    use Illuminate\Support\Arr;

    $array = [100, 200, 300];

    $first = Arr::first($array, function ($value, $key) {
        return $value >= 150;
    });

    // 200

Значение по умолчанию может быть передано в качестве третьего аргумента. Это значение будет возвращено, если ни одно из значений не удовлетворяет условию:

    use Illuminate\Support\Arr;

    $first = Arr::first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `Arr::flatten()` {#collection-method}

Метод `Arr::flatten` преобразует многомерный массив в одномерный:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $flattened = Arr::flatten($array);

    // ['Joe', 'PHP', 'Ruby']

<a name="method-array-forget"></a>
#### `Arr::forget()` {#collection-method}

Метод `Arr::forget` удаляет заданную пару ключ/значение из многомерного массива, используя синтаксис с точкой ("dot" notation):

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    Arr::forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `Arr::get()` {#collection-method}

Метод `Arr::get` возвращает значение из многомерного массива, используя синтаксис с точкой ("dot" notation):

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    $price = Arr::get($array, 'products.desk.price');

    // 100

Метод `Arr::get` также принимает значение по умолчанию, которое будет возвращено, если заданный ключ не будет найден:

    use Illuminate\Support\Arr;

    $discount = Arr::get($array, 'products.desk.discount', 0);

    // 0

<a name="method-array-has"></a>
#### `Arr::has()` {#collection-method}

Метод `Arr::has` проверяет существование заданного элемента или элементов в массиве, используя синтаксис с точкой ("dot" notation):

    use Illuminate\Support\Arr;

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = Arr::has($array, 'product.name');

    // true

    $contains = Arr::has($array, ['product.price', 'product.discount']);

    // false

<a name="method-array-isassoc"></a>
#### `Arr::isAssoc()` {#collection-method}

Метод `Arr::isAssoc` возвращает `true`, если данный массив является ассоциативным. Массив считается "ассоциативным", если в нем нет последовательных числовых ключей, начинающихся с нуля:

    use Illuminate\Support\Arr;

    $isAssoc = Arr::isAssoc(['product' => ['name' => 'Desk', 'price' => 100]]);

    // true

    $isAssoc = Arr::isAssoc([1, 2, 3]);

    // false

<a name="method-array-last"></a>
#### `Arr::last()` {#collection-method}

Метод `Arr::last` возвращает последний элемент массива, удовлетворяющий требуемому условию:

    use Illuminate\Support\Arr;

    $array = [100, 200, 300, 110];

    $last = Arr::last($array, function ($value, $key) {
        return $value >= 150;
    });

    // 300

Значение по умолчанию может быть передано в качестве третьего аргумента. Это значение будет возвращено, если ни одно из значений не удовлетворяет условию:

    use Illuminate\Support\Arr;

    $last = Arr::last($array, $callback, $default);

<a name="method-array-only"></a>
#### `Arr::only()` {#collection-method}

Метод `Arr::only` возвращает из заданного массива только указанные пары ключ/значение:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $slice = Arr::only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `Arr::pluck()` {#collection-method}

Метод `Arr::pluck` возвращает из массива все значения для данного ключа:

    use Illuminate\Support\Arr;

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $names = Arr::pluck($array, 'developer.name');

    // ['Taylor', 'Abigail']

Также вы можете указать ключ для полученного списка:

    use Illuminate\Support\Arr;

    $names = Arr::pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail']

<a name="method-array-prepend"></a>
#### `Arr::prepend()` {#collection-method}

Метод `Arr::prepend` вставит элемент в начало массива:

    use Illuminate\Support\Arr;

    $array = ['one', 'two', 'three', 'four'];

    $array = Arr::prepend($array, 'zero');

    // ['zero', 'one', 'two', 'three', 'four']

При необходимости можно указать ключ, который будет использоваться для значения:

    use Illuminate\Support\Arr;

    $array = ['price' => 100];

    $array = Arr::prepend($array, 'Desk', 'name');

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pull"></a>
#### `Arr::pull()` {#collection-method}

Метод `Arr::pull` возвращает и удаляет пару ключ/значение из массива:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100];

    $name = Arr::pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

Значение по умолчанию может быть передано в качестве третьего аргумента методу. Это значение будет возвращено, если ключа не существует:

    use Illuminate\Support\Arr;

    $value = Arr::pull($array, $key, $default);

<a name="method-array-random"></a>
#### `Arr::random()` {#collection-method}

Метод `Arr::random` возвращает случайное значение из массива:

    use Illuminate\Support\Arr;

    $array = [1, 2, 3, 4, 5];

    $random = Arr::random($array);

    // 4 - (полученно случайным образом)

Вы также можете указать количество возвращаемых элементов в качестве необязательного второго аргумента. Обратите внимание, что при указании этого аргумента будет возвращен массив, даже если требуется только один элемент:

    use Illuminate\Support\Arr;

    $items = Arr::random($array, 2);

    // [2, 5] - (полученно случайным образом)

<a name="method-array-query"></a>
#### `Arr::query()` {#collection-method}

Метод `Arr::query` преобразует массив в строку запроса:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Taylor', 'order' => ['column' => 'created_at', 'direction' => 'desc']];

    Arr::query($array);

    // name=Taylor&order[column]=created_at&order[direction]=desc

<a name="method-array-set"></a>
#### `Arr::set()` {#collection-method}

Метод `Arr::set` устанавливает значение в многомерном массиве, используя синтаксис с точкой ("dot" notation):

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    Arr::set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-shuffle"></a>
#### `Arr::shuffle()` {#collection-method}

Метод `Arr::shuffle` случайным образом перемешивает элементы в массиве:

    use Illuminate\Support\Arr;

    $array = Arr::shuffle([1, 2, 3, 4, 5]);

    // [3, 2, 5, 1, 4] - (сформированный случайным образом)

<a name="method-array-sort"></a>
#### `Arr::sort()` {#collection-method}

Метод `Arr::sort` сортирует массив по его значениям:

    use Illuminate\Support\Arr;

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = Arr::sort($array);

    // ['Chair', 'Desk', 'Table']

Также можно отсортировать массив по результату переданной функции:

    use Illuminate\Support\Arr;

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(Arr::sort($array, function ($value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
            ['name' => 'Table'],
        ]
    */

<a name="method-array-sort-recursive"></a>
#### `Arr::sortRecursive()` {#collection-method}

Метод `Arr::sortRecursive` рекурсивно сортирует массив, используя функцию `sort` для числовых подмассивов и `ksort` для ассоциативных подмассивов:

    use Illuminate\Support\Arr;

    $array = [
        ['Roman', 'Taylor', 'Li'],
        ['PHP', 'Ruby', 'JavaScript'],
        ['one' => 1, 'two' => 2, 'three' => 3],
    ];

    $sorted = Arr::sortRecursive($array);

    /*
        [
            ['JavaScript', 'PHP', 'Ruby'],
            ['one' => 1, 'three' => 3, 'two' => 2],
            ['Li', 'Roman', 'Taylor'],
        ]
    */

<a name="method-array-where"></a>
#### `Arr::where()` {#collection-method}

Метод `Arr::where` фильтрует массив с помощью переданной функции:

    use Illuminate\Support\Arr;

    $array = [100, '200', 300, '400', 500];

    $filtered = Arr::where($array, function ($value, $key) {
        return is_string($value);
    });

    // [1 => '200', 3 => '400']

<a name="method-array-wrap"></a>
#### `Arr::wrap()` {#collection-method}

Метод `Arr::wrap` обёртывает заданное значение в массив. Если значение уже является массивом, то оно не будет изменено:

    use Illuminate\Support\Arr;

    $string = 'Laravel';

    $array = Arr::wrap($string);

    // ['Laravel']

Если заданное значение является `null`, то будет возвращен пустой массив:

    use Illuminate\Support\Arr;

    $nothing = null;

    $array = Arr::wrap($nothing);

    // []

<a name="method-data-fill"></a>
#### `data_fill()` {#collection-method}

Функция `data_fill` устанавливает недостающее значение внутри вложенного массива или объекта, используя синтаксис с точкой ("dot" notation):

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_fill($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 100]]]

    data_fill($data, 'products.desk.discount', 10);

    // ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]

Эта функция также принимает звездочки `*` в качестве подстановки (wildcards) и соответственно заполняет массив:

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2'],
        ],
    ];

    data_fill($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 100],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */

<a name="method-data-get"></a>
#### `data_get()` {#collection-method}

Функция `data_get` получает значение из вложенного массива или объекта, используя синтаксис с точкой ("dot" notation):

    $data = ['products' => ['desk' => ['price' => 100]]];

    $price = data_get($data, 'products.desk.price');

    // 100

Функция `data_get` также принимает значение по умолчанию, которое будет возвращено, если указанный ключ не будет найден:

    $discount = data_get($data, 'products.desk.discount', 0);

    // 0

Функция также принимает подстановки (wildcards), используя звездочки `*`, которые могут использоваться вместо любого ключа массива или объекта:

    $data = [
        'product-one' => ['name' => 'Desk 1', 'price' => 100],
        'product-two' => ['name' => 'Desk 2', 'price' => 150],
    ];

    data_get($data, '*.name');

    // ['Desk 1', 'Desk 2'];

<a name="method-data-set"></a>
#### `data_set()` {#collection-method}

Функция `data_set` устанавливает значение внутри многомерного массива или объекта, используя синтаксис с точкой ("dot" notation):

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

Функция также принимает подстановки (wildcards), используя звездочки `*`, и устанавливает значения соответственно:

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 150],
        ],
    ];

    data_set($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 200],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */

По умолчанию существующее значение перезаписывается. Если вы хотите установить значение только в том случае, если оно не существует, вы можете передать в качестве четвертого аргумента `false`:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200, false);

    // ['products' => ['desk' => ['price' => 100]]]

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

Функция `config_path` возвращает полный путь к папке `config`. Вы также можете использовать функцию `config_path` для получения полного пути к файлу в папке конфигурации:

    $path = config_path();

    $path = config_path('app.php');

<a name="method-database-path"></a>
#### `database_path()` {#collection-method}

Функция `database_path` возвращает полный путь к папке `database`. Вы также можете использовать функцию `database_path` для получения полного пути к файлу в папке `database`:

    $path = database_path();

    $path = database_path('factories/UserFactory.php');

<a name="method-mix"></a>
#### `mix()` {#collection-method}

Функция `mix` возвращает путь к [версионированному файлу Mix](/docs/{{version}}/mix):

    $path = mix('css/app.css');

<a name="method-public-path"></a>
#### `public_path()` {#collection-method}

Функция `public_path` возвращает полный путь к папке `public`. Вы также можете использовать функцию `public_path` для получения полного пути к файлу в папке `public`:

    $path = public_path();

    $path = public_path('css/app.css');

<a name="method-resource-path"></a>
#### `resource_path()` {#collection-method}

Функция `resource_path` возвращает полный путь к папке `resources`. Вы также можете использовать функцию `resource_path` для получения полного пути к файлу в папке `resources`:

    $path = resource_path();

    $path = resource_path('sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()` {#collection-method}

Функция `storage_path` возвращает полный путь к папке `storage`. Вы также можете использовать функцию `storage_path` для получения полного пути к файлу в папке `storage`:

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## Строки

<a name="method-__"></a>
#### `__()` {#collection-method}

Функция `__` получает перевод для строки или ключа перевода, используя [файлы локализации](/docs/{{version}}/localization):

    echo __('Welcome to our application');

    echo __('messages.welcome');

Если указанная строка или ключ перевода не существует, функция `__` вернет переданное ей значение. Таким образом, используя приведенный выше пример, функция `__` вернет `messages.welcome`, если указанного ключа не существует.

<a name="method-class-basename"></a>
#### `class_basename()` {#collection-method}

Функция `class_basename` возвращает имя переданного класса без пространства имен:

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` {#collection-method}

Функция `e` выполняет функцию PHP `htmlspecialchars` с опцией `double_encode`, установленной по умолчанию в `true`:

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-preg-replace-array"></a>
#### `preg_replace_array()` {#collection-method}

Функция `preg_replace_array` последовательно заменяет заданный паттерн в строке, используя массив:

    $string = 'The event will take place between :start and :end';

    $replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-after"></a>
#### `Str::after()` {#collection-method}

Метод `Str::after` возвращает часть строки после заданного значения. Вся строка будет возвращена, если она не содержит заданного значения:

    use Illuminate\Support\Str;

    $slice = Str::after('This is my name', 'This is');

    // ' my name'

<a name="method-str-after-last"></a>
#### `Str::afterLast()` {#collection-method}

Метод `Str::afterLast` возвращает часть строки после последнего вхождения заданного значения. Вся строка будет возвращена, если она не содержит заданного значения:

    use Illuminate\Support\Str;

    $slice = Str::afterLast('App\Http\Controllers\Controller', '\\');

    // 'Controller'

<a name="method-str-before"></a>
#### `Str::before()` {#collection-method}

Метод `Str::before` возвращает часть строки до заданного значения:

    use Illuminate\Support\Str;

    $slice = Str::before('This is my name', 'my name');

    // 'This is '

<a name="method-str-before-last"></a>
#### `Str::beforeLast()` {#collection-method}

Метод `Str::beforeLast` возвращает часть строки до последнего вхождения заданного значения:

    use Illuminate\Support\Str;

    $slice = Str::beforeLast('This is my name', 'is');

    // 'This '

<a name="method-camel-case"></a>
#### `Str::camel()` {#collection-method}

Метод `Str::camel` преобразует строку в `camelCase`:

    use Illuminate\Support\Str;

    $converted = Str::camel('foo_bar');

    // fooBar

<a name="method-str-contains"></a>
#### `Str::contains()` {#collection-method}

Метод `Str::contains` определяет, содержит ли строка заданное значение (с учетом регистра):

    use Illuminate\Support\Str;

    $contains = Str::contains('This is my name', 'my');

    // true

Также можно передать массив значений, чтобы определить, содержит ли строка какое-либо из значений:

    use Illuminate\Support\Str;

    $contains = Str::contains('This is my name', ['my', 'foo']);

    // true

<a name="method-str-contains-all"></a>
#### `Str::containsAll()` {#collection-method}

Метод `Str::containsAll` определяет, содержит ли строка все значения заданного массива:

    use Illuminate\Support\Str;

    $containsAll = Str::containsAll('This is my name', ['my', 'name']);

    // true

<a name="method-ends-with"></a>
#### `Str::endsWith()` {#collection-method}

Метод `Str::endsWith` определяет, заканчивается ли строка заданным значением:

    use Illuminate\Support\Str;

    $result = Str::endsWith('This is my name', 'name');

    // true


Также можно передать массив значений, чтобы определить, заканчивается ли строка каким-либо из заданных значений:

    use Illuminate\Support\Str;

    $result = Str::endsWith('This is my name', ['name', 'foo']);

    // true

    $result = Str::endsWith('This is my name', ['this', 'foo']);

    // false

<a name="method-str-finish"></a>
#### `Str::finish()` {#collection-method}

Метод `Str::finish` добавляет одно вхождение подстроки в конец строки, если она уже не заканчивается этим вхождением:

    use Illuminate\Support\Str;

    $adjusted = Str::finish('this/string', '/');

    // this/string/

    $adjusted = Str::finish('this/string/', '/');

    // this/string/

<a name="method-str-is"></a>
#### `Str::is()` {#collection-method}

Метод `Str::is` определяет, соответствует ли строка заданному шаблону. Звездочки `*` могут использоваться как символы подстановки:

    use Illuminate\Support\Str;

    $matches = Str::is('foo*', 'foobar');

    // true

    $matches = Str::is('baz*', 'foobar');

    // false

<a name="method-str-ucfirst"></a>
#### `Str::ucfirst()` {#collection-method}

Метод `Str::ucfirst` возвращает строку с первым символом в верхнем регистре:

    use Illuminate\Support\Str;

    $string = Str::ucfirst('foo bar');

    // Foo bar

<a name="method-str-is-uuid"></a>
#### `Str::isUuid()` {#collection-method}

Метод `Str::isUuid` определяет, является ли строка валидным UUID:

    use Illuminate\Support\Str;

    $isUuid = Str::isUuid('a0a2a2d2-0b87-4a18-83f2-2529882be2de');

    // true

    $isUuid = Str::isUuid('laravel');

    // false

<a name="method-kebab-case"></a>
#### `Str::kebab()` {#collection-method}

Метод `Str::kebab` преобразует строку в `kebab-case`:

    use Illuminate\Support\Str;

    $converted = Str::kebab('fooBar');

    // foo-bar

<a name="method-str-limit"></a>
#### `Str::limit()` {#collection-method}

Метод `Str::limit` обрезает строку до заданной длины:

    use Illuminate\Support\Str;

    $truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20);

    // The quick brown fox...

Вы также можете передать третий аргумент для изменения фрагмента, который будет добавлен к строке:

    use Illuminate\Support\Str;

    $truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

    // The quick brown fox (...)

<a name="method-str-ordered-uuid"></a>
#### `Str::orderedUuid()` {#collection-method}

Метод `Str::orderedUuid` генерирует "timestamp first" UUID, который может быть эффективно сохранен в индексируемой колонке базы данных:

    use Illuminate\Support\Str;

    return (string) Str::orderedUuid();

<a name="method-str-plural"></a>
#### `Str::plural()` {#collection-method}

Метод `Str::plural` преобразует слово-строку во множественное число. В настоящее время эта функция поддерживает только английский язык:

    use Illuminate\Support\Str;

    $plural = Str::plural('car');

    // cars

    $plural = Str::plural('child');

    // children

Вы можете передать целое число в качестве второго аргумента функции, чтобы получить единственное или множественное число:

    use Illuminate\Support\Str;

    $plural = Str::plural('child', 2);

    // children

    $plural = Str::plural('child', 1);

    // child

<a name="method-str-random"></a>
#### `Str::random()` {#collection-method}

Метод `Str::random` создает случайную строку заданной длины. Эта функция использует функцию PHP `random_bytes`:

    use Illuminate\Support\Str;

    $random = Str::random(40);

<a name="method-str-replace-array"></a>
#### `Str::replaceArray()` {#collection-method}

Метод `Str::replaceArray` последовательно заменяет заданное значение в строке массивом:

    use Illuminate\Support\Str;

    $string = 'The event will take place between ? and ?';

    $replaced = Str::replaceArray('?', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-replace-first"></a>
#### `Str::replaceFirst()` {#collection-method}

Метод `Str::replaceFirst` заменяет первое вхождение подстроки в строке:

    use Illuminate\Support\Str;

    $replaced = Str::replaceFirst('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // a quick brown fox jumps over the lazy dog

<a name="method-str-replace-last"></a>
#### `Str::replaceLast()` {#collection-method}

Метод `Str::replaceLast` заменяет последнее вхождение подстроки в строке:

    use Illuminate\Support\Str;

    $replaced = Str::replaceLast('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // the quick brown fox jumps over a lazy dog

<a name="method-str-singular"></a>
#### `Str::singular()` {#collection-method}

Метод `Str::singular` преобразует слово-строку в единственное число. В настоящее время эта функция поддерживает только английский язык:

    use Illuminate\Support\Str;

    $singular = Str::singular('cars');

    // car

    $singular = Str::singular('children');

    // child

<a name="method-str-slug"></a>
#### `Str::slug()` {#collection-method}

Метод `Str::slug` генерирует "slug" подходящий для URL из строки:

    use Illuminate\Support\Str;

    $slug = Str::slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-snake-case"></a>
#### `Str::snake()` {#collection-method}

Метод `Str::snake` преобразует строку в `snake_case`:

    use Illuminate\Support\Str;

    $converted = Str::snake('fooBar');

    // foo_bar

<a name="method-str-start"></a>
#### `Str::start()` {#collection-method}

Метод `Str::start` добавляет подстроку в начало переданной строки, если она еще не начинается с этой подстроки:

    use Illuminate\Support\Str;

    $adjusted = Str::start('this/string', '/');

    // /this/string

    $adjusted = Str::start('/this/string', '/');

    // /this/string

<a name="method-starts-with"></a>
#### `Str::startsWith()` {#collection-method}

Метод `Str::startsWith` определяет, начинается ли строка с заданного значения:

    use Illuminate\Support\Str;

    $result = Str::startsWith('This is my name', 'This');

    // true

<a name="method-studly-case"></a>
#### `Str::studly()` {#collection-method}

Метод `Str::studly` преобразует строку в `StudlyCase`:

    use Illuminate\Support\Str;

    $converted = Str::studly('foo_bar');

    // FooBar

<a name="method-title-case"></a>
#### `Str::title()` {#collection-method}

Метод `Str::title` преобразует строку в `Title Case`:

    use Illuminate\Support\Str;

    $converted = Str::title('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-str-uuid"></a>
#### `Str::uuid()` {#collection-method}

Метод `Str::uuid` генерирует UUID (версия 4):

    use Illuminate\Support\Str;

    return (string) Str::uuid();

<a name="method-str-words"></a>
#### `Str::words()` {#collection-method}

Метод `Str::words` ограничивает количество слов в строке:

    use Illuminate\Support\Str;

    return Str::words('Perfectly balanced, as all things should be.', 3, ' >>>');

    // Perfectly balanced, as >>>

<a name="method-trans"></a>
#### `trans()` {#collection-method}

Функция `trans` переводит переданный ключ перевода с помощью ваших [файлов локализации](/docs/{{version}}/localization):

    echo trans('messages.welcome');

Если указанного ключа перевода не существует, функция `trans` вернет переданный ей ключ. Таким образом, используя вышеприведенный пример, функция `trans` вернет `messages.welcome`, если ключ перевода не существует.

<a name="method-trans-choice"></a>
#### `trans_choice()` {#collection-method}

Функция `trans_choice` переводит переданный ключ перевода с изменениями:

    echo trans_choice('messages.notifications', $unreadCount);

Если указанного ключа перевода не существует, функция `trans_choice` вернет переданный ей ключ. Таким образом, используя вышеприведенный пример, функция `trans_choice` вернет `messages.notifications`, если ключ перевода не существует.

<a name="urls"></a>
## URL

<a name="method-action"></a>
#### `action()` {#collection-method}

Функция `action` генерирует URL для заданного метода контроллера. Вам не нужно передавать полное пространство имён контроллера. Вместо этого передайте имя класса контроллера относительно пространства имен `App\Http\Controllers`:

    $url = action('HomeController@index');

    $url = action([HomeController::class, 'index']);

Если метод принимает параметры маршрута, вы можете передать их вторым аргументом:

    $url = action('UserController@profile', ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` {#collection-method}

Функция `asset` создает URL для ресурса, используя текущую схему запроса (HTTP или HTTPS):

    $url = asset('img/photo.jpg');

Вы можете настроить хост URL адреса, установив параметр `ASSET_URL` в файле `.env`. Это может быть полезно, если вы размещаете свои ресурсы на внешнем сервисе, таком как Amazon S3:

    // ASSET_URL=http://example.com/assets

    $url = asset('img/photo.jpg'); // http://example.com/assets/img/photo.jpg

<a name="method-route"></a>
#### `route()` {#collection-method}

Функция `route` генерирует URL для заданного именованного роута:

    $url = route('routeName');

Если роут принимает параметры, вы можете передать их вторым аргументом:

    $url = route('routeName', ['id' => 1]);

По-умолчанию функция `route` генерирует абсолютный URL-адрес. Если вы хотите сгенерировать относительный URL-адрес, можно передать `false` в качестве третьего аргумента:

    $url = route('routeName', ['id' => 1], false);

<a name="method-secure-asset"></a>
#### `secure_asset()` {#collection-method}

Функция `secure_asset` создает URL для ресурса с использованием HTTPS:

    $url = secure_asset('img/photo.jpg');

<a name="method-secure-url"></a>
#### `secure_url()` {#collection-method}

Функция `secure_url` генерирует полный HTTPS URL по заданному пути:

    $url = secure_url('user/profile');

    $url = secure_url('user/profile', [1]);

<a name="method-url"></a>
#### `url()` {#collection-method}

Функция `url` генерирует полный URL по заданному пути:

    $url = url('user/profile');

    $url = url('user/profile', [1]);

Если путь не указан, вернется экземпляр `Illuminate\Routing\UrlGenerator`:

    $current = url()->current();

    $full = url()->full();

    $previous = url()->previous();

<a name="miscellaneous"></a>
## Прочее

<a name="method-abort"></a>
#### `abort()` {#collection-method}

Функция `abort` выбрасывает [HTTP-исключение](/docs/{{version}}/errors#http-exceptions), которое будет отображено [обработчиком исключений](/docs/{{version}}/errors#the-exception-handler):

    abort(403);

Вы также можете передать текст и заголовки ответа:

    abort(403, 'Unauthorized.', $headers);

<a name="method-abort-if"></a>
#### `abort_if()` {#collection-method}

Функция `abort_if` выбрасывает HTTP-исключение, если заданное логическое выражение равно `true`:

    abort_if(! Auth::user()->isAdmin(), 403);

Как и в методе `abort`, вы можете передать текст ответа в качестве третьего аргумента и массив заголовков четвертым аргументом.

<a name="method-abort-unless"></a>
#### `abort_unless()` {#collection-method}

Функция `abort_unless` выбрасывает HTTP-исключение, если заданное логическое выражение равно `false`:

    abort_unless(Auth::user()->isAdmin(), 403);

Как и в методе `abort`, вы можете передать текст ответа в качестве третьего аргумента и массив заголовков четвертым аргументом.

<a name="method-app"></a>
#### `app()` {#collection-method}

Функция `app` возвращает экземпляр [сервис-контейнера](/docs/{{version}}}/container):

    $container = app();

Вы можете передать имя класса или интерфейса для получения его из контейнера:

    $api = app('HelpSpot\API');

<a name="method-auth"></a>
#### `auth()` {#collection-method}

Функция `auth` возвращает экземпляр [аутентификатора](/docs/{{version}}/authentication). Вы можете использовать ее вместо фасада `Auth` для удобства:

    $user = auth()->user();

При необходимости вы можете указать, к какому гварду вы хотите получить доступ:

    $user = auth('admin')->user();

<a name="method-back"></a>
#### `back()` {#collection-method}

Функция `back()` создает [ответ с редиректом](/docs/{{version}}/responses#redirects) на предыдущую страницу пользователя:

    return back($status = 302, $headers = [], $fallback = false);

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` {#collection-method}

Функция `bcrypt` [хеширует](/docs/{{version}}/hashing) переданное значение с помощью Bcrypt. Вы можете использовать ее вместо фасада `Hash`:

    $password = bcrypt('my-secret-password');

<a name="method-blank"></a>
#### `blank()` {#collection-method}

Функция `blank` определяет является ли данное значение "пустым":

    blank('');
    blank('   ');
    blank(null);
    blank(collect());

    // true

    blank(0);
    blank(true);
    blank(false);

    // false

Функция, обратная `blank` — [`filled`](#method-filled).

<a name="method-broadcast"></a>
#### `broadcast()` {#collection-method}

Функция `broadcast` выполняет [широковещание](/docs/{{version}}/broadcasting) переданного [события](/docs/{{version}}/events) для обработчиков:

    broadcast(new UserRegistered($user));

<a name="method-cache"></a>
#### `cache()` {#collection-method}

Функцию `cache` можно использовать для получения значений из [кэша](/docs/{{version}}/cache). Если в кэше нет заданного ключа, будет возвращено необязательное значение по умолчанию:

    $value = cache('key');

    $value = cache('key', 'default');

Вы можете добавить элементы в кэш, передав массив пар ключ/значение. Также вам надо передать количество секунд или время, в течение которого кэшированные значения будут считаться действительными:

    cache(['key' => 'value'], 300);

    cache(['key' => 'value'], now()->addSeconds(10));

<a name="method-class-uses-recursive"></a>
#### `class_uses_recursive()` {#collection-method}

Функция `class_uses_recursive` возвращает все трейты, используемые классом, включая трейты, используемые его родительскими классами:

    $traits = class_uses_recursive(App\User::class);

<a name="method-collect"></a>
#### `collect()` {#collection-method}

Функция `collect` создает экземпляр [коллекции](/docs/{{version}}/collections) из переданного значения:

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {#collection-method}

Функция `config` получает значение переменной из [конфигурации](/docs/{{version}}/configuration). К значениям конфигурации можно обращаться с помощью "точечного" синтаксиса, в котором указывается имя файла и необходимый параметр. Можно указать значение по умолчанию, которое будет возвращено, если параметра не существует:

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

Вы можете задать переменные конфигурации во время выполнения, передав массив пар ключ/значение:

    config(['app.debug' => true]);

<a name="method-cookie"></a>
#### `cookie()` {#collection-method}

Функция `cookie` создает новый экземпляр [cookie](/docs/{{version}}/requests#cookies):

    $cookie = cookie('name', 'value', $minutes);

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

Если вы не хотите останавливать выполнение скрипта, вместо этого используйте функцию [`dump`](#method-dump):

<a name="method-decrypt"></a>
#### `decrypt()` {#collection-method}

Функция `decrypt` расшифровывает заданное значение, используя [шифрование](/docs/{{version}}/encryption) Laravel:

    $decrypted = decrypt($encrypted_value);

<a name="method-dispatch"></a>
#### `dispatch()` {#collection-method}

Функция `dispatch` помещает [задачу](/docs/{{version}}/queues#creating-jobs) в [очередь задач](/docs/{{version}}/queues) Laravel:

    dispatch(new App\Jobs\SendEmails);

<a name="method-dispatch-now"></a>
#### `dispatch_now()` {#collection-method}

Функция `dispatch_now` запускает [задачу](/docs/{{version}}/queues#creating-jobs) немедленно и возвращает значение её метода `handle`:

    $result = dispatch_now(new App\Jobs\SendEmails);

<a name="method-dump"></a>
#### `dump()` {#collection-method}

Функция `dump` производит дамп переменных:

    dump($value);

    dump($value1, $value2, $value3, ...);

Если вы хотите остановить выполнение после вывода переменных, используйте функцию [`dd`](#method-dd).

<a name="method-encrypt"></a>
#### `encrypt()` {#collection-method}

Функция `encrypt` шифрует заданное значение, используя [шифрование](/docs/{{version}}/encryption) Laravel:

    $encrypted = encrypt($unencrypted_value);

<a name="method-env"></a>
#### `env()` {#collection-method}

Функция `env` получает значение [переменной окружения](/docs/{{version}}/configuration#environment-configuration) или возвращает значение по умолчанию:

    $env = env('APP_ENV');

    // Returns 'production' if APP_ENV is not set...
    $env = env('APP_ENV', 'production');

> {note} Если вы выполняете команду `config:cache` во время вашего процесса деплоя, вы должны быть уверены, что вызываете функцию `env` только в конфигурационных файлах. Как только конфигурация будет закэширована, файл `.env` не будет загружаться и все вызовы функции `env` вернут `null`.

<a name="method-event"></a>
#### `event()` {#collection-method}

Функция `event` отправляет указанное [событие](/docs/{{version}}/events) его слушателям:

    event(new UserRegistered($user));

<a name="method-factory"></a>
#### `factory()` {#collection-method}

Функция `factory` создает построитель фабрики моделей для данного класса, имени и количества. Его можно использовать при [тестировании](/docs/{{version}}/database-testing#writing-factories) или [заполнении БД](/docs/{{version}}/seeding#using-model-factories):

    $user = factory(App\User::class)->make();

<a name="method-filled"></a>
#### `filled()` {#collection-method}

Функция `filled` определяет, не является ли заданное значение "пустым":

    filled(0);
    filled(true);
    filled(false);

    // true

    filled('');
    filled('   ');
    filled(null);
    filled(collect());

    // false

Функция, обратная `filled` — [`blank`](#method-blank).

<a name="method-info"></a>
#### `info()` {#collection-method}

Функция `info` запишет информацию в [лог](/docs/{{version}}/logging):

    info('Некая полезная информация!');

В функцию можно передать массив контекстных данных:

    info('Неудачная попытка входа пользователя.', ['id' => $user->id]);

<a name="method-logger"></a>
#### `logger()` {#collection-method}

Функцию `logger` можно использовать, чтобы записать в [лог](/docs/{{version}}/logging) сообщение уровня `debug`:

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

<a name="method-now"></a>
#### `now()` {#collection-method}

Функция `now` создает экземпляр класса `Illuminate\Support\Carbon` с текущим временем:

    $now = now();

<a name="method-old"></a>
#### `old()` {#collection-method}

Функция `old` [получает](/docs/{{version}}/requests#retrieving-input) значение ["старого" ввода](/docs/{{version}}/requests#old-input), переданного в сессию:

    $value = old('value');

    $value = old('value', 'default');

<a name="method-optional"></a>
#### `optional()` {#collection-method}

Функция `optional` принимает любой аргумент и позволяет получить доступ к свойствам или вызвать методы на этом объекте. Если переданный объект — `null`, то свойства и методы будут возвращать `null` вместо того, чтобы вызывать ошибку:

    return optional($user->address)->street;

    {!! old('name', optional($user)->name) !!}

Функция `optional` также принимает функцию в качестве второго аргумента. Функция будет вызвана если значение переданное в первом аргументе не является `null`:

    return optional(User::find($id), function ($user) {
        return new DummyUser;
    });

<a name="method-policy"></a>
#### `policy()` {#collection-method}

Функция `policy` получает экземпляр [политик](/docs/{{version}}/authorization#creating-policies) для заданного класса:

    $policy = policy(App\User::class);

<a name="method-redirect"></a>
#### `redirect()` {#collection-method}

Функция `redirect` возвращает [HTTP-ответ с редиректом](/docs/{{version}}/responses#redirects), или экземпляр переадресатора, если вызывается без аргументов:

    return redirect($to = null, $status = 302, $headers = [], $secure = null);

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-report"></a>
#### `report()` {#collection-method}

Функция `report` сообщит об исключении, используя метод` report` [обработчика исключения](/docs/{{version}}/errors#the-exception-handler):

    report($e);

<a name="method-request"></a>
#### `request()` {#collection-method}

Функция `request` возвращает экземпляр текущего [запроса](/docs/{{version}}/requests) или получает элемент ввода:

    $request = request();

    $value = request('key', $default);

<a name="method-rescue"></a>
#### `rescue()` {#collection-method}

Функция `rescue` выполняет переданную функцию и перехватывает любые исключения, возникающие во время её выполнения. Все пойманные исключения будет отправлены в метод `report` [обработчика исключений](/docs/{{version}}/errors#the-exception-handler); однако, выполнение продолжиться:

    return rescue(function () {
        return $this->method();
    });

Вы также можете передать второй аргумент в функцию `rescue`. Этот аргумент будет значением «по умолчанию», которое должно быть возвращено, если во время выполнения функции возникнет исключение:

    return rescue(function () {
        return $this->method();
    }, false);

    return rescue(function () {
        return $this->method();
    }, function () {
        return $this->failure();
    });

<a name="method-resolve"></a>
#### `resolve()` {#collection-method}

Функция `resolve` получает экземпляр класса или интерфейса из [сервис-контейнера](/docs/{{version}}/container) по его имени:

    $api = resolve('HelpSpot\API');

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

Функция `session` используется для получения или задания значений [сессии](/docs/{{version}}/session):

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
        'email' => $email,
    ]);

Для добавления метода `tap` в класс, вы можете добавить трейт `Illuminate\Support\Traits\Tappable`. Метод `tap` этого трейта принимает функцию в качестве единственного аргумента. Сам экземпляр объекта будет передан в функцию, а затем возвращен методом `tap`:

    return $user->tap(function ($user) {
        //
    });

<a name="method-throw-if"></a>
#### `throw_if()` {#collection-method}

Функция `throw_if` выбрасывает заданное исключение, если логическое выражение равно `true`:

    throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

    throw_if(
        ! Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );

<a name="method-throw-unless"></a>
#### `throw_unless()` {#collection-method}

Функция `throw_unless` выбрасывает заданное исключение, если логическое выражение равно `false`:

    throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

    throw_unless(
        Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );

<a name="method-today"></a>
#### `today()` {#collection-method}

Функция `today` создает экземпляр класса `Illuminate\Support\Carbon` для текущей даты:

    $today = today();

<a name="method-trait-uses-recursive"></a>
#### `trait_uses_recursive()` {#collection-method}

Функция `trait_uses_recursive` возвращает все трейты, используемые трейтом:

    $traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);

<a name="method-transform"></a>
#### `transform()` {#collection-method}

Функция `transform` выполняет функцию для заданного значения, если значение не является [blank](#method-blank) и возвращает результат функции:

    $callback = function ($value) {
        return $value * 2;
    };

    $result = transform(5, $callback);

    // 10

Значение по умолчанию или функция могут быть переданы в качестве третьего аргумента метода. Это значение будет возвращено, если переданное значение пустое:

    $result = transform(null, $callback, 'The value is blank');

    // The value is blank

<a name="method-validator"></a>
#### `validator()` {#collection-method}

Функция `validator` создает новый экземпляр [валидатора](/docs/{{version}}/validation) с заданными аргументами. Для удобства вы можете использовать его вместо фасада `Validator`:

    $validator = validator($data, $rules, $messages);

<a name="method-value"></a>
#### `value()` {#collection-method}

Функция `value` возвращает переданное ей значение. Однако, если передать функцию, она будет выполнена и её результат будет возвращен:

    $result = value(true);

    // true

    $result = value(function () {
        return false;
    });

    // false

<a name="method-view"></a>
#### `view()` {#collection-method}

Функция `view` получает экземпляр [шаблона](/docs/{{version}}/views):

    return view('auth.login');

<a name="method-with"></a>
#### `with()` {#collection-method}

Функция `with` возвращает переданное ей значение. Если в качестве второго арумента передать функцию, то она будет выполненна и её результат будет возвращен:

    $callback = function ($value) {
        return (is_numeric($value)) ? $value * 2 : 0;
    };

    $result = with(5, $callback);

    // 10

    $result = with(null, $callback);

    // 0

    $result = with(5, null);

    // 5
