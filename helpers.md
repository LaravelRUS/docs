---
git: 46c2634ef5a4f15427c94a3157b626cf5bd3937f
---

# Глобальные помощники (helpers)

<a name="introduction"></a>
## Введение

Laravel содержит множество глобальных «вспомогательных» функций. Многие из этих функций используются самим фреймворком; однако, вы можете использовать их в своих собственных приложениях, если сочтете удобными.

<a name="available-methods"></a>
## Доступные методы


<a name="arrays-and-objects-method-list"></a>
### Массивы и объекты

<div class="docs-column-list" markdown="1">

- [Arr::accessible](#method-array-accessible)
- [Arr::add](#method-array-add)
- [Arr::collapse](#method-array-collapse)
- [Arr::crossJoin](#method-array-crossjoin)
- [Arr::divide](#method-array-divide)
- [Arr::dot](#method-array-dot)
- [Arr::except](#method-array-except)
- [Arr::exists](#method-array-exists)
- [Arr::first](#method-array-first)
- [Arr::flatten](#method-array-flatten)
- [Arr::forget](#method-array-forget)
- [Arr::get](#method-array-get)
- [Arr::has](#method-array-has)
- [Arr::hasAny](#method-array-hasany)
- [Arr::isAssoc](#method-array-isassoc)
- [Arr::isList](#method-array-islist)
- [Arr::join](#method-array-join)
- [Arr::keyBy](#method-array-keyby)
- [Arr::last](#method-array-last)
- [Arr::map](#method-array-map)
- [Arr::mapWithKeys](#method-array-map-with-keys)
- [Arr::only](#method-array-only)
- [Arr::pluck](#method-array-pluck)
- [Arr::prepend](#method-array-prepend)
- [Arr::prependKeysWith](#method-array-prependkeyswith)
- [Arr::pull](#method-array-pull)
- [Arr::query](#method-array-query)
- [Arr::random](#method-array-random)
- [Arr::set](#method-array-set)
- [Arr::shuffle](#method-array-shuffle)
- [Arr::sort](#method-array-sort)
- [Arr::sortDesc](#method-array-sort-desc)
- [Arr::sortRecursive](#method-array-sort-recursive)
- [Arr::sortRecursiveDesc](#method-array-sort-recursive-desc)
- [Arr::toCssClasses](#method-array-to-css-classes)
- [Arr::toCssStyles](#method-array-to-css-styles)
- [Arr::undot](#method-array-undot)
- [Arr::where](#method-array-where)
- [Arr::whereNotNull](#method-array-where-not-null)
- [Arr::wrap](#method-array-wrap)
- [data_fill](#method-data-fill)
- [data_get](#method-data-get)
- [data_set](#method-data-set)
- [data_forget](#method-data-forget)
- [head](#method-head)
- [last](#method-last)
</div>

<a name="numbers-method-list"></a>
### Numbers

<div class="docs-column-list" markdown="1">

[Number::abbreviate](#method-number-abbreviate)
[Number::format](#method-number-format)
[Number::percentage](#method-number-percentage)
[Number::currency](#method-number-currency)
[Number::fileSize](#method-number-file-size)
[Number::forHumans](#method-number-for-humans)

</div>

<a name="paths-method-list"></a>
### Пути

<div class="docs-column-list" markdown="1">

- [app_path](#method-app-path)
- [base_path](#method-base-path)
- [config_path](#method-config-path)
- [database_path](#method-database-path)
- [lang_path](#method-lang-path)
- [mix](#method-mix)
- [public_path](#method-public-path)
- [resource_path](#method-resource-path)
- [storage_path](#method-storage-path)

</div>

<a name="urls-method-list"></a>
### URL-адреса

<div class="docs-column-list" markdown="1">

- [action](#method-action)
- [asset](#method-asset)
- [route](#method-route)
- [secure_asset](#method-secure-asset)
- [secure_url](#method-secure-url)
- [to_route](#method-to-route)
- [url](#method-url)

</div>
<a name="miscellaneous-method-list"></a>
### Разное

<div class="docs-column-list" markdown="1">

- [abort](#method-abort)
- [abort_if](#method-abort-if)
- [abort_unless](#method-abort-unless)
- [app](#method-app)
- [auth](#method-auth)
- [back](#method-back)
- [bcrypt](#method-bcrypt)
- [blank](#method-blank)
- [broadcast](#method-broadcast)
- [cache](#method-cache)
- [class_uses_recursive](#method-class-uses-recursive)
- [collect](#method-collect)
- [config](#method-config)
- [cookie](#method-cookie)
- [csrf_field](#method-csrf-field)
- [csrf_token](#method-csrf-token)
- [decrypt](#method-decrypt)
- [dd](#method-dd)
- [dispatch](#method-dispatch)
- [dispatch_sync](#method-dispatch-sync)
- [dump](#method-dump)
- [encrypt](#method-encrypt)
- [env](#method-env)
- [event](#method-event)
- [fake](#method-fake)
- [filled](#method-filled)
- [info](#method-info)
- [logger](#method-logger)
- [method_field](#method-method-field)
- [now](#method-now)
- [old](#method-old)
- [optional](#method-optional)
- [policy](#method-policy)
- [redirect](#method-redirect)
- [report](#method-report)
- [report_if](#method-report-if)
  [report_unless](#method-report-unless)
- [request](#method-request)
- [rescue](#method-rescue)
- [resolve](#method-resolve)
- [response](#method-response)
- [retry](#method-retry)
- [session](#method-session)
- [tap](#method-tap)
- [throw_if](#method-throw-if)
- [throw_unless](#method-throw-unless)
- [today](#method-today)
- [trait_uses_recursive](#method-trait-uses-recursive)
- [transform](#method-transform)
- [validator](#method-validator)
- [value](#method-value)
- [view](#method-view)
- [with](#method-with)

</div>

<a name="arrays"></a>
## Массивы и объекты

<a name="method-array-accessible"></a>
#### `Arr::accessible()`

Метод `Arr::accessible` определяет, доступно ли переданное значение массиву:

    use Illuminate\Support\Arr;
    use Illuminate\Support\Collection;

    $isAccessible = Arr::accessible(['a' => 1, 'b' => 2]);

    // true

    $isAccessible = Arr::accessible(new Collection);

    // true

    $isAccessible = Arr::accessible('abc');

    // false

    $isAccessible = Arr::accessible(new stdClass);

    // false

<a name="method-array-add"></a>
#### `Arr::add()`

Метод `Arr::add` добавляет переданную пару ключ / значение в массив, если указанный ключ еще не существует в массиве или установлен как `null`:

    use Illuminate\Support\Arr;

    $array = Arr::add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

    $array = Arr::add(['name' => 'Desk', 'price' => null], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]


<a name="method-array-collapse"></a>
#### `Arr::collapse()`

Метод `Arr::collapse` сворачивает массив массивов в один массив:

    use Illuminate\Support\Arr;

    $array = Arr::collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-crossjoin"></a>
#### `Arr::crossJoin()`

Метод `Arr::crossJoin` перекрестно соединяет указанные массивы, возвращая декартово произведение со всеми возможными перестановками:

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
#### `Arr::divide()`

Метод `Arr::divide` возвращает два массива: один содержит ключи, а другой – значения переданного массива:

    use Illuminate\Support\Arr;

    [$keys, $values] = Arr::divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `Arr::dot()`

Метод `Arr::dot` объединяет многомерный массив в одноуровневый, использующий «точечную нотацию» для обозначения глубины:

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    $flattened = Arr::dot($array);

    // ['products.desk.price' => 100]

<a name="method-array-except"></a>
#### `Arr::except()`

Метод `Arr::except` удаляет переданные пары ключ / значение из массива:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100];

    $filtered = Arr::except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-exists"></a>
#### `Arr::exists()`

Метод `Arr::exists` проверяет, существует ли переданный ключ в указанном массиве:

    use Illuminate\Support\Arr;

    $array = ['name' => 'John Doe', 'age' => 17];

    $exists = Arr::exists($array, 'name');

    // true

    $exists = Arr::exists($array, 'salary');

    // false

<a name="method-array-first"></a>
#### `Arr::first()`

Метод `Arr::first` возвращает первый элемент массива, прошедший тест переданного замыкания на истинность:

    use Illuminate\Support\Arr;

    $array = [100, 200, 300];

    $first = Arr::first($array, function (int $value, int $key) {
        return $value >= 150;
    });

    // 200

Значение по умолчанию может быть передано в качестве третьего аргумента методу. Это значение будет возвращено, если ни одно из значений не пройдет проверку на истинность:

    use Illuminate\Support\Arr;

    $first = Arr::first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `Arr::flatten()`

Метод `Arr::flatten` объединяет многомерный массив в одноуровневый:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $flattened = Arr::flatten($array);

    // ['Joe', 'PHP', 'Ruby']

<a name="method-array-forget"></a>
#### `Arr::forget()`

Метод `Arr::forget` удаляет переданную пару ключ / значение из глубоко вложенного массива, используя «точечную нотацию»:

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    Arr::forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `Arr::get()`

Метод `Arr::get` извлекает значение из глубоко вложенного массива, используя «точечную нотацию»:

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    $price = Arr::get($array, 'products.desk.price');

    // 100

Метод `Arr::get` также принимает значение по умолчанию, которое будет возвращено, если указанный ключ отсутствует в массиве:

    use Illuminate\Support\Arr;

    $discount = Arr::get($array, 'products.desk.discount', 0);

    // 0

<a name="method-array-has"></a>
#### `Arr::has()`

Метод `Arr::has` проверяет, существует ли переданный элемент или элементы в массиве, используя «точечную нотацию»:

    use Illuminate\Support\Arr;

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = Arr::has($array, 'product.name');

    // true

    $contains = Arr::has($array, ['product.price', 'product.discount']);

    // false

<a name="method-array-hasany"></a>
#### `Arr::hasAny()`

Метод `Arr::hasAny` проверяет, существует ли какой-либо элемент в переданном наборе в массиве, используя «точечную нотацию»:

    use Illuminate\Support\Arr;

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = Arr::hasAny($array, 'product.name');

    // true

    $contains = Arr::hasAny($array, ['product.name', 'product.discount']);

    // true

    $contains = Arr::hasAny($array, ['category', 'product.discount']);

    // false

<a name="method-array-isassoc"></a>
#### `Arr::isAssoc()`

Метод `Arr::isAssoc` возвращает `true`, если переданный массив является ассоциативным. Массив считается ассоциативным, если в нем нет последовательных цифровых ключей, начинающихся с нуля:

    use Illuminate\Support\Arr;

    $isAssoc = Arr::isAssoc(['product' => ['name' => 'Desk', 'price' => 100]]);

    // true

    $isAssoc = Arr::isAssoc([1, 2, 3]);

    // false

<a name="method-array-islist"></a>
#### `Arr::isList()`

Метод `Arr::isList` возвращает true, если ключи заданного массива представляют собой последовательные целые числа, начиная с нуля:


    use Illuminate\Support\Arr;

    $isList = Arr::isList(['foo', 'bar', 'baz']);

    // true

    $isList = Arr::isList(['product' => ['name' => 'Desk', 'price' => 100]]);

    // false

<a name="method-array-join"></a>
#### `Arr::join()`

Метод `Arr::join` объединяет элементы массива в строку. Используя второй аргумента этого метода вы также можете указать строку для соединения последнего элемента массива:

    use Illuminate\Support\Arr;

    $array = ['Tailwind', 'Alpine', 'Laravel', 'Livewire'];

    $joined = Arr::join($array, ', ');

    // Tailwind, Alpine, Laravel, Livewire

    $joined = Arr::join($array, ', ', ' and ');

    // Tailwind, Alpine, Laravel and Livewire

<a name="method-array-keyby"></a>
#### `Arr::keyBy()`

Метод `Arr::keyBy` присваивает ключи элементам базового массива на основе указанного ключа.  Если у нескольких элементов один и тот же ключ, в новом массиве появится только последний:

    use Illuminate\Support\Arr;

    $array = [
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ];

    $keyed = Arr::keyBy($array, 'product_id');

    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

<a name="method-array-last"></a>
#### `Arr::last()`

Метод `Arr::last` возвращает последний элемент массива, прошедший тест переданного замыкания на истинность:

    use Illuminate\Support\Arr;

    $array = [100, 200, 300, 110];

    $last = Arr::last($array, function (int $value, int $key) {
        return $value >= 150;
    });

    // 300

Значение по умолчанию может быть передано в качестве третьего аргумента методу. Это значение будет возвращено, если ни одно из значений не пройдет проверку на истинность:

    use Illuminate\Support\Arr;

    $last = Arr::last($array, $callback, $default);

<a name="method-array-map"></a>
#### `Arr::map()`

Метод `Arr::map` проходит по массиву и передает каждое значение и ключ указанной функции обратного вызова. Значение массива заменяется значением, возвращаемым обратным вызовом:

    use Illuminate\Support\Arr;

    $array = ['first' => 'james', 'last' => 'kirk'];

    $mapped = Arr::map($array, function (string $value, string $key) {
        return ucfirst($value);
    });

    // ['first' => 'James', 'last' => 'Kirk']

<a name="method-array-map-with-keys"></a>
#### `Arr::mapWithKeys()


Метод `Arr::mapWithKeys` проходит по массиву и передает каждое значение указанной функции обратного вызова, которая должна возвращать ассоциативный массив, содержащий одну пару ключ / значение:

use Illuminate\Support\Arr;

    $array = [
        [
            'name' => 'John',
            'department' => 'Sales',
            'email' => 'john@example.com',
        ],
        [
            'name' => 'Jane',
            'department' => 'Marketing',
            'email' => 'jane@example.com',
        ]
    ];

    $mapped = Arr::mapWithKeys($array, function (array $item, int $key) {
        return [$item['email'] => $item['name']];
    });

    /*
        [
            'john@example.com' => 'John',
            'jane@example.com' => 'Jane',
        ]
    */

<a name="method-array-only"></a>
#### `Arr::only()`

Метод `Arr::only` возвращает только указанные пары ключ / значение из переданного массива:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $slice = Arr::only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `Arr::pluck()`

Метод `Arr::pluck` извлекает все значения для указанного ключа из массива:

    use Illuminate\Support\Arr;

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $names = Arr::pluck($array, 'developer.name');

    // ['Taylor', 'Abigail']

Вы также можете задать ключ результирующего списка:

    use Illuminate\Support\Arr;

    $names = Arr::pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail']

<a name="method-array-prepend"></a>
#### `Arr::prepend()`

Метод `Arr::prepend` помещает элемент в начало массива:

    use Illuminate\Support\Arr;

    $array = ['one', 'two', 'three', 'four'];

    $array = Arr::prepend($array, 'zero');

    // ['zero', 'one', 'two', 'three', 'four']

При необходимости вы можете указать ключ, который следует использовать для значения:

    use Illuminate\Support\Arr;

    $array = ['price' => 100];

    $array = Arr::prepend($array, 'Desk', 'name');

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-prependkeyswith"></a>
#### `Arr::prependKeysWith()`


Метод `Arr::prependKeysWith` добавляет указанный префикс ко всем именам ключей ассоциативного массива:

    use Illuminate\Support\Arr;

    $array = [
        'name' => 'Desk',
        'price' => 100,
    ];

    $keyed = Arr::prependKeysWith($array, 'product.');

    /*
        [
            'product.name' => 'Desk',
            'product.price' => 100,
        ]
    */

<a name="method-array-pull"></a>
#### `Arr::pull()`

Метод `Arr::pull` возвращает и удаляет пару ключ / значение из массива:

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100];

    $name = Arr::pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

Значение по умолчанию может быть передано в качестве третьего аргумента методу. Это значение будет возвращено, если ключ не существует:

    use Illuminate\Support\Arr;

    $value = Arr::pull($array, $key, $default);

<a name="method-array-query"></a>
#### `Arr::query()`

Метод `Arr::query` преобразует массив в строку запроса:

    use Illuminate\Support\Arr;

    $array = [
        'name' => 'Taylor',
        'order' => [
            'column' => 'created_at',
            'direction' => 'desc'
        ]
    ];

    Arr::query($array);

    // name=Taylor&order[column]=created_at&order[direction]=desc

<a name="method-array-random"></a>
#### `Arr::random()`

Метод `Arr::random` возвращает случайное значение из массива:

    use Illuminate\Support\Arr;

    $array = [1, 2, 3, 4, 5];

    $random = Arr::random($array);

    // 4 - (retrieved randomly)

Вы также можете указать количество элементов для возврата в качестве необязательного второго аргумента. Обратите внимание, что при указании этого аргумента, будет возвращен массив, даже если требуется только один элемент:

    use Illuminate\Support\Arr;

    $items = Arr::random($array, 2);

    // [2, 5] - (retrieved randomly)

<a name="method-array-set"></a>
#### `Arr::set()`

Метод `Arr::set` устанавливает значение с помощью «точечной нотации» во вложенном массиве:

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    Arr::set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-shuffle"></a>
#### `Arr::shuffle()`

Метод `Arr::shuffle` случайным образом перемешивает элементы в массиве:

    use Illuminate\Support\Arr;

    $array = Arr::shuffle([1, 2, 3, 4, 5]);

    // [3, 2, 5, 1, 4] - (generated randomly)

<a name="method-array-sort"></a>
#### `Arr::sort()`

Метод `Arr::sort` сортирует массив по его значениям:

    use Illuminate\Support\Arr;

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = Arr::sort($array);

    // ['Chair', 'Desk', 'Table']

Вы также можете отсортировать массив по результатам переданного замыкания:

    use Illuminate\Support\Arr;

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(Arr::sort($array, function (array $value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
            ['name' => 'Table'],
        ]
    */

<a name="method-array-sort-desc"></a>
#### `Arr::sortDesc()`

Метод `Arr::sortDesc` сортирует массив по убыванию значений:

    use Illuminate\Support\Arr;

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = Arr::sortDesc($array);

    // ['Table', 'Desk', 'Chair']

Вы также можете отсортировать массив по результатам переданного замыкания:

    use Illuminate\Support\Arr;

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(Arr::sortDesc($array, function (array $value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Table'],
            ['name' => 'Desk'],
            ['name' => 'Chair'],
        ]
    */

<a name="method-array-sort-recursive"></a>
#### `Arr::sortRecursive()`

Метод `Arr::sortRecursive` рекурсивно сортирует массив с помощью метода `sort` для числовых подмассивов и `ksort` для ассоциативных подмассивов:

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

Если вы хотите, чтобы результаты были отсортированы по убыванию, вы можете использовать метод` Arr::sortRecursiveDesc`.

    $sorted = Arr::sortRecursiveDesc($array);

<a name="method-array-to-css-classes"></a>
#### `Arr::toCssClasses()` 

Метод `Arr::toCssClasses` составляет строку классов CSS исходя из заданных условий. Метод принимает массив классов, где ключ массива содержит класс или классы, которые вы хотите добавить, а значение является булевым выражением. Если элемент массива не имеет строкового ключа, он всегда будет включен в список отрисованных классов:

    use Illuminate\Support\Arr;

    $isActive = false;
    $hasError = true;

    $array = ['p-4', 'font-bold' => $isActive, 'bg-red' => $hasError];

    $classes = Arr::toCssClasses($array);

    /*
        'p-4 bg-red'
    */

<a name="method-array-to-css-styles"></a>
#### `Arr::toCssStyles()`

Метод `Arr::toCssStyles` условно компилирует строку стилей CSS. Метод принимает массив классов, где ключ массива содержит класс или классы, которые вы хотите добавить, а значение - логическое выражение. Если элемент массива имеет числовой ключ, он всегда будет включен в список отображаемых классов:

```php
$hasColor = true;

$array = ['background-color: blue', 'color: blue' => $hasColor];

$classes = Arr::toCssStyles($array);

/*
    'background-color: blue; color: blue;'
*/
```

При помощи этого метода осуществляется [объединение css-классов в Blade](/docs/{{version}}/blade#conditionally-merge-classes), а также [в директиве](/docs/{{version}}/blade#conditional-classes) `@class`. 

<a name="method-array-undot"></a>
#### `Arr::undot()`

Метод `Arr::undot` расширяет одномерный массив, использующий "точечную нотацию", в многомерный массив:

    use Illuminate\Support\Arr;

    $array = [
        'user.name' => 'Kevin Malone',
        'user.occupation' => 'Accountant',
    ];

    $array = Arr::undot($array);

    // ['user' => ['name' => 'Kevin Malone', 'occupation' => 'Accountant']]

<a name="method-array-where"></a>
#### `Arr::where()`

Метод `Arr::where` фильтрует массив, используя переданное замыкание:

    use Illuminate\Support\Arr;

    $array = [100, '200', 300, '400', 500];

    $filtered = Arr::where($array, function (string|int $value, int $key) {
        return is_string($value);
    });

    // [1 => '200', 3 => '400']

<a name="method-array-where-not-null"></a>
#### `Arr::whereNotNull()`

Метод `Arr::whereNotNull`удаляет все значения `null` из данного массива:

    use Illuminate\Support\Arr;

    $array = [0, null];

    $filtered = Arr::whereNotNull($array);

    // [0 => 0]

<a name="method-array-wrap"></a>
#### `Arr::wrap()`

Метод `Arr::wrap` оборачивает переданное значение в массив. Если переданное значение уже является массивом, то оно будет возвращено без изменений:

    use Illuminate\Support\Arr;

    $string = 'Laravel';

    $array = Arr::wrap($string);

    // ['Laravel']

Если переданное значение равно `null`, то будет возвращен пустой массив:

    use Illuminate\Support\Arr;

    $array = Arr::wrap(null);

    // []

<a name="method-data-fill"></a>
#### `data_fill()`

Функция `data_fill` устанавливает отсутствующее значение с помощью «точечной нотации» во вложенном массиве или объекте:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_fill($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 100]]]

    data_fill($data, 'products.desk.discount', 10);

    // ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]

Допускается использование метасимвола подстановки `*`:

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
#### `data_get()`

Функция `data_get` возвращает значение с помощью «точечной нотации» из вложенного массива или объекта:

    $data = ['products' => ['desk' => ['price' => 100]]];

    $price = data_get($data, 'products.desk.price');

    // 100

Функция `data_get` также принимает значение по умолчанию, которое будет возвращено, если указанный ключ не найден:

    $discount = data_get($data, 'products.desk.discount', 0);

    // 0

Допускается использование метасимвола подстановки `*`, предназначенный для любого ключа массива или объекта:

    $data = [
        'product-one' => ['name' => 'Desk 1', 'price' => 100],
        'product-two' => ['name' => 'Desk 2', 'price' => 150],
    ];

    data_get($data, '*.name');

    // ['Desk 1', 'Desk 2'];

<a name="method-data-set"></a>
#### `data_set()`

Функция `data_set` устанавливает значение с помощью «точечной нотации» во вложенном массиве или объекте:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

Допускается использование метасимвола подстановки `*`:

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

По умолчанию все существующие значения перезаписываются. Если вы хотите, чтобы значение было установлено только в том случае, если оно не существует, вы можете передать `false` в качестве четвертого аргумента:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200, overwrite: false);

    // ['products' => ['desk' => ['price' => 100]]]

<a name="method-data-forget"></a>
#### `data_forget()`

Функция `data_forget` удаляет значение внутри вложенного массива или объекта, используя "точечную" нотацию:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_forget($data, 'products.desk.price');

    // ['products' => ['desk' => []]]

Эта функция также принимает маски с использованием звездочек и удаляет соответствующие значения из цели:

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 150],
        ],
    ];

    data_forget($data, 'products.*.price');

    /*
        [
            'products' => [
                ['name' => 'Desk 1'],
                ['name' => 'Desk 2'],
            ],
        ]
    */

<a name="method-head"></a>
#### `head()`

Функция `head` возвращает первый элемент переданного массива:

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()`

Функция `last` возвращает последний элемент переданного массива:

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="numbers"></a>
## Числа 

<a name="method-number-abbreviate"></a>
#### `Number::abbreviate()`

Метод `Number::abbreviate` возвращает числовое значение в удобочитаемом формате с сокращением для единиц измерения:

    use Illuminate\Support\Number;

    $number = Number::abbreviate(1000);

    // 1K

    $number = Number::abbreviate(489939);

    // 490K

    $number = Number::abbreviate(1230000, precision: 2);

    // 1.23M

<a name="method-number-format"></a>
#### `Number::format()`

Метод `Number::format` форматирует предоставленное число в строку с учетом локализации:

use Illuminate\Support\Number;

    $number = Number::format(100000);

    // 100,000

    $number = Number::format(100000, precision: 2);

    // 100,000.00

    $number = Number::format(100000.123, maxPrecision: 2);

    // 100,000.12

    $number = Number::format(100000, locale: 'de');

    // 100.000

<a name="method-number-percentage"></a>
#### `Number::percentage()`

Метод `Number::percentage` возвращает процентное представление указанного значения в виде строки:

    use Illuminate\Support\Number;

    $percentage = Number::percentage(10);

    // 10%

    $percentage = Number::percentage(10, precision: 2);

    // 10.00%

    $percentage = Number::percentage(10.123, maxPrecision: 2);

    // 10.12%

    $percentage = Number::percentage(10, precision: 2, locale: 'de');

    // 10,00%

<a name="method-number-currency"></a>
#### `Number::currency()`

Метод `Number::currency` возвращает представление указанного значения в валюте в виде строки:

    use Illuminate\Support\Number;

    $currency = Number::currency(1000);

    // $1,000

    $currency = Number::currency(1000, in: 'EUR');

    // €1,000

    $currency = Number::currency(1000, in: 'EUR', locale: 'de');

    // 1.000 €

<a name="method-number-file-size"></a>
#### `Number::fileSize()`

Метод `Number::fileSize` для указанного значения в байтах возвращает представление размера файла в виде строки:

    use Illuminate\Support\Number;

    $size = Number::fileSize(1024);

    // 1 KB

    $size = Number::fileSize(1024 * 1024);

    // 1 MB

    $size = Number::fileSize(1024, precision: 2);

    // 1.00 KB

<a name="method-number-for-humans"></a>
#### `Number::forHumans()`

Метод Number::forHumans возвращает числовое значение в удобочитаемом формате:

    use Illuminate\Support\Number;

    $number = Number::forHumans(1000);

    // 1 thousand

    $number = Number::forHumans(489939);

    // 490 thousand

    $number = Number::forHumans(1230000, precision: 2);

    // 1.23 million

<a name="paths"></a>
## Пути

<a name="method-app-path"></a>
#### `app_path()`

Функция `app_path` возвращает полный путь к каталогу вашего приложения `app`. Вы также можете использовать функцию `app_path` для создания полного пути к файлу относительно каталога приложения:

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()`

Функция `base_path` возвращает полный путь к корневому каталогу вашего приложения. Вы также можете использовать функцию `base_path` для генерации полного пути к заданному файлу относительно корневого каталога проекта:

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()`

Функция `config_path` возвращает полный путь к каталогу `config` вашего приложения. Вы также можете использовать функцию `config_path` для создания полного пути к заданному файлу в каталоге конфигурации приложения:

    $path = config_path();

    $path = config_path('app.php');

<a name="method-database-path"></a>
#### `database_path()`

Функция `database_path` возвращает полный путь к каталогу `database` вашего приложения. Вы также можете использовать функцию `database_path` для генерации полного пути к заданному файлу в каталоге базы данных:

    $path = database_path();

    $path = database_path('factories/UserFactory.php');

<a name="method-lang-path"></a>
#### `lang_path()`

Функция `lang_path` возвращает полный путь к каталогу `lang` вашего приложения. Вы также можете использовать функцию `lang_path` для генерации полного пути к указанному файлу внутри этого каталога:

    $path = lang_path();

    $path = lang_path('en/messages.php');

> [!NOTE]
> По умолчанию в структуре приложения Laravel отсутствует каталог `lang`. Если вы хотите настроить языковые файлы Laravel, вы можете опубликовать их с помощью команды Artisan `lang:publish`.

<a name="method-mix"></a>
#### `mix()`

Функция `mix` возвращает путь к [версионированному файлу Mix](/docs/{{version}}/mix#versioning-and-cache-busting):

    $path = mix('css/app.css');

<a name="method-public-path"></a>
#### `public_path()`

Функция `public_path` возвращает полный путь к каталогу `public` вашего приложения. Вы также можете использовать функцию `public_path` для генерации полного пути к заданному файлу в публичном каталоге:

    $path = public_path();

    $path = public_path('css/app.css');

<a name="method-resource-path"></a>
#### `resource_path()`

Функция `resource_path` возвращает полный путь к каталогу `resources` вашего приложения. Вы также можете использовать функцию `resource_path`, чтобы сгенерировать полный путь к заданному файлу в каталоге исходников:

    $path = resource_path();

    $path = resource_path('sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()`

Функция `storage_path` возвращает полный путь к каталогу `storage` вашего приложения. Вы также можете использовать функцию `storage_path` для генерации полного пути к заданному файлу в каталоге хранилища:

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="urls"></a>
## URL-адреса

<a name="method-action"></a>
#### `action()`

Функция `action` генерирует URL-адрес для переданного действия контроллера:

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

Если метод принимает параметры маршрута, вы можете передать их как второй аргумент методу:

    $url = action([UserController::class, 'profile'], ['id' => 1]);

<a name="method-asset"></a>
#### `asset()`

Функция `asset` генерирует URL для исходника (прим. перев.: директория `resources`), используя текущую схему запроса (HTTP или HTTPS):

    $url = asset('img/photo.jpg');

Вы можете настроить хост URL исходников, установив переменную `ASSET_URL` в вашем файле `.env`. Это может быть полезно, если вы размещаете свои исходники на внешнем сервисе, таком как Amazon S3 или другой CDN:

    // ASSET_URL=http://example.com/assets

    $url = asset('img/photo.jpg'); // http://example.com/assets/img/photo.jpg

<a name="method-route"></a>
#### `route()`

Функция `route` генерирует URL для переданного [именованного маршрута](/docs/{{version}}/routing#named-routes):

    $url = route('route.name');

Если маршрут принимает параметры, вы можете передать их в качестве второго аргумента методу:

    $url = route('route.name', ['id' => 1]);

По умолчанию функция `route` генерирует абсолютный URL. Если вы хотите создать относительный URL, вы можете передать `false` в качестве третьего аргумента:

    $url = route('route.name', ['id' => 1], false);

<a name="method-secure-asset"></a>
#### `secure_asset()`

Функция `secure_asset` генерирует URL для исходника, используя HTTPS:

    $url = secure_asset('img/photo.jpg');

<a name="method-secure-url"></a>
#### `secure_url()`

Функция `secure_url` генерирует полный URL-адрес для указанного пути, используя HTTPS. Дополнительные сегменты URL могут быть переданы во втором аргументе функции:

    $url = secure_url('user/profile');

    $url = secure_url('user/profile', [1]);

<a name="method-to-route"></a>
#### `to_route()`

Функция `to_route` генерирует [HTTP-ответ перенаправления](/docs/{{version}}/responses#redirects) для заданного [именованного маршрута](/docs/{{version}}/routing#named-routes) :

    return to_route('users.show', ['user' => 1]);

return to_route('users.show', ['user' => 1], 302, ['X-Framework' => 'Laravel']);

При необходимости вы можете передать методу `to_route` код состояния HTTP, который должен быть присвоен перенаправлению, а также любые дополнительные заголовки ответа в качестве третьего и четвёртого аргументов:

<a name="method-url"></a>
#### `url()`

Функция `url` генерирует полный URL-адрес для указанного пути:

    $url = url('user/profile');

    $url = url('user/profile', [1]);

Если путь не указан, будет возвращен экземпляр `Illuminate\Routing\UrlGenerator`:

    $current = url()->current();

    $full = url()->full();

    $previous = url()->previous();

<a name="miscellaneous"></a>
## Разное

<a name="method-abort"></a>
#### `abort()`

Функция `abort` генерирует [HTTP-исключение](/docs/{{version}}/errors#http-exceptions), которое будет обработано [обработчиком исключения](/docs/{{version}}/errors#the-exception-handler):

    abort(403);

Вы также можете указать текст ответа исключения и пользовательские заголовки ответа, которые должны быть отправлены в браузер:

    abort(403, 'Unauthorized.', $headers);

<a name="method-abort-if"></a>
#### `abort_if()`

Функция `abort_if` генерирует исключение HTTP, если переданное логическое выражение имеет значение `true`:

    abort_if(! Auth::user()->isAdmin(), 403);

Подобно методу `abort`, вы также можете указать текст ответа исключения третьим аргументом и массив пользовательских заголовков ответа в качестве четвертого аргумента.

<a name="method-abort-unless"></a>
#### `abort_unless()`

Функция `abort_unless` генерирует исключение HTTP, если переданное логическое выражение оценивается как `false`:

    abort_unless(Auth::user()->isAdmin(), 403);

Подобно методу `abort`, вы также можете указать текст ответа исключения третьим аргументом и массив пользовательских заголовков ответа в качестве четвертого аргумента.

<a name="method-app"></a>
#### `app()`

Функция `app` возвращает экземпляр [контейнера служб](/docs/{{version}}/container):

    $container = app();

Вы можете передать имя класса или интерфейса для извлечения его из контейнера:

    $api = app('HelpSpot\API');

<a name="method-auth"></a>
#### `auth()`

Функция `auth` возвращает экземпляр [аутентификатора](authentication). Вы можете использовать его вместо фасада `Auth` для удобства:

    $user = auth()->user();

При необходимости вы можете указать, к какому экземпляру охранника вы хотите получить доступ:

    $user = auth('admin')->user();

<a name="method-back"></a>
#### `back()`

Функция `back` генерирует [HTTP-ответ перенаправления](responses#redirects) в предыдущее расположение пользователя:

    return back($status = 302, $headers = [], $fallback = '/');

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()`

Функция `bcrypt` [хеширует](/docs/{{version}}/hashing) переданное значение, используя Bcrypt. Вы можете использовать его как альтернативу фасаду `Hash`:

    $password = bcrypt('my-secret-password');

<a name="method-blank"></a>
#### `blank()`

Функция `blank` проверяет, является ли переданное значение «пустым»:

    blank('');
    blank('   ');
    blank(null);
    blank(collect());

    // true

    blank(0);
    blank(true);
    blank(false);

    // false

Обратной функции `blank` является функция [`filled`](#method-filled).

<a name="method-broadcast"></a>
#### `broadcast()`

Функция `broadcast` [транслирует](/docs/{{version}}/broadcasting) переданное [событие](/docs/{{version}}/events) своим слушателям:

    broadcast(new UserRegistered($user));

    broadcast(new UserRegistered($user))->toOthers();

<a name="method-cache"></a>
#### `cache()`

Функция `cache` используется для получения значений из [кеша](/docs/{{version}}/cache). Если переданный ключ не существует в кеше, будет возвращено необязательное значение по умолчанию:

    $value = cache('key');

    $value = cache('key', 'default');

Вы можете добавлять элементы в кеш, передавая массив пар ключ / значение в функцию. Вы также должны передать количество секунд или продолжительность актуальности кешированного значения:

    cache(['key' => 'value'], 300);

    cache(['key' => 'value'], now()->addSeconds(10));

<a name="method-class-uses-recursive"></a>
#### `class_uses_recursive()`

Функция `class_uses_recursive` возвращает все трейты, используемые классом, включая трейты, используемые всеми его родительскими классами:

    $traits = class_uses_recursive(App\Models\User::class);

<a name="method-collect"></a>
#### `collect()`

Функция `collect` создает экземпляр [коллекции](/docs/{{version}}/collections) переданного значения:

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()`

Функция `config` получает значение переменной [конфигурации](/docs/{{version}}/configuration). Доступ к значениям конфигурации можно получить с помощью «точечной нотации», включающую имя файла и параметр, к которому вы хотите получить доступ. Значение по умолчанию может быть указано и возвращается, если опция конфигурации не существует:

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

Вы можете установить переменные конфигурации на время выполнения скрипта, передав массив пар ключ / значение. Однако обратите внимание, что эта функция влияет только на значение конфигурации для текущего запроса и не обновляет фактические значения конфигурации:

    config(['app.debug' => true]);

<a name="method-cookie"></a>
#### `cookie()`

Функция `cookie` создает новый экземпляр [Cookie](/docs/{{version}}/requests#cookies):

    $cookie = cookie('name', 'value', $minutes);

<a name="method-csrf-field"></a>
#### `csrf_field()`

Функция `csrf_field` генерирует HTML «скрытого» поля ввода, содержащее значение токена CSRF. Например, используя [синтаксис Blade](/docs/{{version}}/blade):

    {{ csrf_field() }}

<a name="method-csrf-token"></a>
#### `csrf_token()`

Функция `csrf_token` возвращает значение текущего токена CSRF:

    $token = csrf_token();

<a name="method-decrypt"></a>
#### `decrypt()`

Функция `decrypt` [расшифровывает](/docs/{{version}}/encryption) предоставленное значение. Вы можете использовать эту функцию в качестве альтернативы фасаду `Crypt`.

$password = decrypt($value);

<a name="method-dd"></a>
#### `dd()`

Функция `dd` выводит переданные переменные и завершает выполнение скрипта:

    dd($value);

    dd($value1, $value2, $value3, ...);

Если вы не хотите останавливать выполнение вашего скрипта, используйте вместо этого функцию [`dump`](#method-dump).

<a name="method-dispatch"></a>
#### `dispatch()`

Функция `dispatch` помещает переданное [задание](/docs/{{version}}/queues#creating-jobs) в [очередь заданий](/docs/{{version}}/queues) Laravel:

    dispatch(new App\Jobs\SendEmails);

<a name="method-dispatch-sync"></a>
#### `dispatch_sync()`

Функция `dispatch_sync` помещает предоставленную задачу в очередь  [синхронно](/docs/{{version}}/queues#synchronous-dispatching) для немедленной обработки:

    dispatch_sync(new App\Jobs\SendEmails);

<a name="method-dump"></a>
#### `dump()`

Функция `dump` выводит переданные переменные:

    dump($value);

    dump($value1, $value2, $value3, ...);

Если вы хотите прекратить выполнение скрипта после вывода переменных, используйте вместо этого функцию [`dd`](#method-dd).

<a name="method-encrypt"></a>
#### `encrypt()`

Функция `encrypt` [шифрует](/docs/{{version}}/encryption) предоставленное значение. Вы можете использовать эту функцию в качестве альтернативы фасаду `Crypt`.

    $secret = encrypt('my-secret-value');

<a name="method-env"></a>
#### `env()`

Функция `env` возвращает значение [переменной окружения](/docs/{{version}}/configuration#environment-configuration) или значение по умолчанию:

    $env = env('APP_ENV');

    $env = env('APP_ENV', 'production');


> [!WARNING]  
> Если вы выполнили команду `config:cache` во время процесса развертывания, вы должны быть уверены, что вызываете функцию `env` только из файлов конфигурации. Как только конфигурации будут кешированы, файл `.env` не будет загружаться, и все вызовы функции `env` будут возвращать `null`.

<a name="method-event"></a>
#### `event()`

Функция `event` отправляет переданное [событие](/docs/{{version}}/events) своим слушателям:

    event(new UserRegistered($user));

<a name="method-fake"></a>
#### `fake()`

Функция `fake` получает экземпляр [Faker](https://github.com/FakerPHP/Faker) из контейнера, что может быть полезно при создании фиктивных данных в фабриках моделей, наполнении базы данных, тестировании и создании макетов представлений:

```blade
@for($i = 0; $i < 10; $i++)
    <dl>
        <dt>Name</dt>
        <dd>{{ fake()->name() }}</dd>
        <dt>Email</dt>
        <dd>{{ fake()->unique()->safeEmail() }}</dd>
    </dl>
@endfor
```

По умолчанию функция `fake` будет использовать опцию `app.faker_locale` из файла конфигурации `config/app.php`. Однако вы также можете указать локализацию, передав ее в функцию `fake`. Для каждой локализации будет создан свой собственный экземпляр:

    fake('nl_NL')->name()

<a name="method-filled"></a>
#### `filled()`

Функция `filled` проверяет, является ли переданное значение не «пустым»:

    filled(0);
    filled(true);
    filled(false);

    // true

    filled('');
    filled('   ');
    filled(null);
    filled(collect());

    // false

Обратной функции `filled` является функция [`blank`](#method-blank).

<a name="method-info"></a>
#### `info()`

Функция `info` запишет информацию в [журнал](/docs/{{version}}/logging):

    info('Some helpful information!');

Также функции может быть передан массив контекстных данных:

    info('User login attempt failed.', ['id' => $user->id]);

<a name="method-logger"></a>
#### `logger()`

Функцию `logger` можно использовать для записи сообщения уровня `debug` в [журнал](/docs/{{version}}/logging):

    logger('Debug message');

Также функции может быть передан массив контекстных данных:

    logger('User has logged in.', ['id' => $user->id]);

Если функции не передано значение, то будет возвращен экземпляр [регистратора](/docs/{{version}}/errors#logging):

    logger()->error('You are not allowed here.');

<a name="method-method-field"></a>
#### `method_field()`

Функция `method_field` генерирует HTML «скрытого» поле ввода, содержащее поддельное значение HTTP-метода формы. Например, используя [синтаксис Blade](/docs/{{version}}/blade):

    <form method="POST">
        {{ method_field('DELETE') }}
    </form>

<a name="method-now"></a>
#### `now()`

Функция `now` создает новый экземпляр `Illuminate\Support\Carbon` для текущего времени:

    $now = now();

<a name="method-old"></a>
#### `old()`

Функция `old` [возвращает](/docs/{{version}}/requests#retrieving-input) значение [прежнего ввода](/docs/{{version}}/requests#old-input), краткосрочно сохраненное в сессии:

    $value = old('value');

    $value = old('value', 'default');

Поскольку значение по умолчанию, предоставляемое вторым аргументом функции `old`, часто является атрибутом модели Eloquent, Laravel позволяет вам просто передать всю модель Eloquent в качестве второго аргумента функции `old`. При этом Laravel предполагает, что первый аргумент, предоставленный функции `old`, - это имя атрибута Eloquent, которое следует считать значением по умолчанию:

    {{ old('name', $user->name) }}

    // Is equivalent to...

    {{ old('name', $user) }}

<a name="method-optional"></a>
#### `optional()`

Функция `optional` принимает любой аргумент и позволяет вам получать доступ к свойствам или вызывать методы этого объекта. Если переданный объект имеет значение `null`, свойства и методы будут возвращать также `null` вместо вызова ошибки:

    return optional($user->address)->street;

    {!! old('name', optional($user)->name) !!}

Функция `optional` также принимает замыкание в качестве второго аргумента. Замыкание будет вызвано, если значение, указанное в качестве первого аргумента, не равно `null`:

    return optional(User::find($id), function (User $user) {
        return $user->name;
    });

<a name="method-policy"></a>
#### `policy()`

Функция `policy` извлекает экземпляр [политики](authorization#creating-policies) для переданного класса:

    $policy = policy(App\Models\User::class);

<a name="method-redirect"></a>
#### `redirect()`

Функция `redirect` возвращает [HTTP-ответ перенаправления](responses#redirects) или возвращает экземпляр перенаправителя, если вызывается без аргументов:

    return redirect($to = null, $status = 302, $headers = [], $https = null);

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-report"></a>
#### `report()`

Функция `report` сообщит об исключении, используя ваш [обработчик исключений](/docs/{{version}}/errors#the-exception-handler):

    report($e);

Функция `report` также принимает строку в качестве аргумента. Когда в функцию передается строка, она создает исключение с переданной строкой в качестве сообщения:

    report('Something went wrong.');

<a name="method-report-if"></a>
#### `report_if()`

Функция `report_if` будет сообщать об исключении с использованием вашего [обработчика исключений](/docs/{{version}}/errors#the-exception-handler), если заданное условие является `true`:

    report_if($shouldReport, $e);

    report_if($shouldReport, 'Something went wrong.');

<a name="method-report-unless"></a>
#### `report_unless()`

Функция `report_unless` будет сообщать об исключении с использованием вашего [обработчика исключений](/docs/{{version}}/errors#the-exception-handler), если заданное условие является `false`:

    report_unless($reportingDisabled, $e);

    report_unless($reportingDisabled, 'Something went wrong.');

<a name="method-request"></a>
#### `request()`

Функция `request` возвращает экземпляр текущего [запроса](/docs/{{version}}/requests) или получает значение поля ввода из текущего запроса:

    $request = request();

    $value = request('key', $default);

<a name="method-rescue"></a>
#### `rescue()`

Функция `rescue` выполняет переданное замыкание и перехватывает любые исключения, возникающие во время его выполнения. Все перехваченные исключения будут отправлены вашему [обработчику исключений](/docs/{{version}}/errors#the-exception-handler); однако, обработка запроса будет продолжена:

    return rescue(function () {
        return $this->method();
    });

Вы также можете передать второй аргумент функции `rescue`. Этот аргумент будет значением «по умолчанию», которое должно быть возвращено, если во время выполнения замыкание возникнет исключение:

    return rescue(function () {
        return $this->method();
    }, false);

    return rescue(function () {
        return $this->method();
    }, function () {
        return $this->failure();
    });

Функции `rescue`  может быть предоставлен аргумент `report`, чтобы определить, следует ли сообщать об исключении чрез функцию `report`:


    return rescue(function () {
        return $this->method();
    }, report: function (Throwable $throwable) {
        return $throwable instanceof InvalidArgumentException;
    });

<a name="method-resolve"></a>
#### `resolve()`

Функция `resolve` извлекает экземпляр связанного с переданным классом или интерфейсом, используя [контейнер служб](/docs/{{version}}/container):

    $api = resolve('HelpSpot\API');

<a name="method-response"></a>
#### `response()`

Функция `response` создает экземпляр [ответа](responses) или получает экземпляр фабрики ответов:

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-retry"></a>
#### `retry()`

Функция `retry` пытается выполнить переданную функцию, пока не будет достигнут указанный лимит попыток. Если функция не выбросит исключение, то будет возвращено её значение. Если функция выбросит исключение, то будет автоматически повторена. Если максимальное количество попыток превышено, будет выброшено исключение

    return retry(5, function () {
        // Attempt 5 times while resting 100ms between attempts...
    }, 100);

Если вы хотите вручную вычислить количество миллисекунд, которое должно пройти между попытками, вы можете передать функцию в качестве третьего аргумента функции `retry`:

    use Exception;

    return retry(5, function () {
        // ...
    }, function (int $attempt, Exception $exception) {
        return $attempt * 100;
    });

Для удобства вы можете передать функции `retry` в качестве первого аргумента массив. Этот массив будет использоваться для определения интервала в миллисекундах между последующими попытками:

    return retry([100, 200], function () {
        // Sleep for 100ms on first retry, 200ms on second retry...
    });

Чтобы повторить попытку только при определенных условиях, вы можете передать функцию, определяющее это условие, в качестве четвертого аргумента функции `retry`:

    use Exception;

    return retry(5, function () {
        // ...
    }, 100, function ($exception) {
        return $exception instanceof RetryException;
    });

<a name="method-session"></a>
#### `session()`

Функция `session` используется для получения или задания значений [сессии](/docs/{{version}}/session):

    $value = session('key');

Вы можете установить значения, передав массив пар ключ / значение в функцию:

    session(['chairs' => 7, 'instruments' => 3]);

Если в функцию не передано значение, то будет возвращен экземпляр хранилища сессий:

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-tap"></a>
#### `tap()`

Функция `tap` принимает два аргумента: произвольное значение и замыкание. Значение будет передано в замыкание, а затем возвращено функцией `tap`. Возвращаемое значение замыкания не имеет значения:

    $user = tap(User::first(), function (User $user) {
        $user->name = 'taylor';

        $user->save();
    });

Если замыкание не передано функции `tap`, то вы можете вызвать любой метод с указанным значением. Возвращаемое значение вызываемого метода всегда будет изначально указанное, независимо от того, что метод фактически возвращает в своем определении. Например, метод Eloquent `update` обычно возвращает целочисленное значение. Однако, мы можем заставить метод возвращать саму модель, увязав вызов метода `update` с помощью функции `tap`:

    $user = tap($user)->update([
        'name' => $name,
        'email' => $email,
    ]);

Чтобы добавить к своему классу метод `tap`, используйте трейт `Illuminate\Support\Traits\Tappable` в вашем классе. Метод `tap` этого трейта принимает замыкание в качестве единственного аргумента. Сам экземпляр объекта будет передан замыканию, а затем будет возвращен методом `tap`:

    return $user->tap(function (User $user) {
        //
    });

<a name="method-throw-if"></a>
#### `throw_if()`

Функция `throw_if` выбрасывает переданное исключение, если указанное логическое выражение оценивается как `true`:

    throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

    throw_if(
        ! Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page.'
    );

<a name="method-throw-unless"></a>
#### `throw_unless()`

Функция `throw_unless` выбрасывает переданное исключение, если указанное логическое выражение оценивается как `false`:

    throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

    throw_unless(
        Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page.'
    );

<a name="method-today"></a>
#### `today()`

Функция `today` создает новый экземпляр `Illuminate\Support\Carbon` для текущей даты:

    $today = today();

<a name="method-trait-uses-recursive"></a>
#### `trait_uses_recursive()`

Функция `trait_uses_recursive` возвращает все трейты, используемые трейтом:

    $traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);

<a name="method-transform"></a>
#### `transform()`

Функция `transform` выполняет замыкание для переданного значения, если значение не [пустое](#method-blank), и возвращает результат замыкания:

    $callback = function (int $value) {
        return $value * 2;
    };

    $result = transform(5, $callback);

    // 10

В качестве третьего параметра могут быть указанны значение по умолчанию или замыкание. Это значение будет возвращено, если переданное значение пустое:

    $result = transform(null, $callback, 'The value is blank');

    // The value is blank

<a name="method-validator"></a>
#### `validator()`

Функция `validator` создает новый экземпляр [валидатора](/docs/{{version}}/validation) с указанными аргументами. Вы можете использовать его для удобства вместо фасада `Validator`:

    $validator = validator($data, $rules, $messages);

<a name="method-value"></a>
#### `value()`

Функция `value` возвращает переданное значение. Однако, если вы передадите замыкание в функцию, то замыкание будет выполнено, и будет возвращен его результат:

    $result = value(true);

    // true

    $result = value(function () {
        return false;
    });

    // false

Функции `value`  могут быть переданы дополнительные аргументы. Если первый аргумент является замыканием, то дополнительные параметры будут переданы в замыкание в качестве аргументов, в противном случае они будут проигнорированы:

    $result = value(function (string $name) {
        return $name;
    }, 'Taylor');

    // 'Taylor'

<a name="method-view"></a>
#### `view()`

Функция `view` возвращает экземпляр [представления](/docs/{{version}}/views):

    return view('auth.login');

<a name="method-with"></a>
#### `with()`

Функция `with` возвращает переданное значение. Если вы передадите замыкание в функцию в качестве второго аргумента, то замыкание будет выполнено и будет возвращен результат его выполнения:

    $callback = function (mixed $value) {
        return is_numeric($value) ? $value * 2 : 0;
    };

    $result = with(5, $callback);

    // 10

    $result = with(null, $callback);

    // 0

    $result = with(5, null);

    // 5

<a name="other-utilities"></a>
## Другие утилиты

<a name="benchmarking"></a>
### Benchmark

Иногда вам может потребоваться быстро оценить производительность определенных частей вашего приложения. В таких случаях вы можете воспользоваться классом `Benchmark` для измерения времени выполнения переданных обратных вызовов в миллисекундах:


    <?php

    use App\Models\User;
    use Illuminate\Support\Benchmark;

    Benchmark::dd(fn () => User::find(1)); // 0.1 ms

    Benchmark::dd([
        'Scenario 1' => fn () => User::count(), // 0.5 ms
        'Scenario 2' => fn () => User::all()->count(), // 20.0 ms
    ]);


По умолчанию переданные обратные вызовы будут выполнены один раз (одна итерация), и их длительность будет отображена в браузере / консоли.

Чтобы выполнить обратный вызов более одного раза, вы можете указать количество итераций вторым аргументом метода. При выполнении обратного вызова более одного раза класс `Benchmark` вернет среднее количество миллисекунд, затраченных на выполнение обратного вызова за все итерации:

    Benchmark::dd(fn () => User::count(), iterations: 10); // 0.5 ms

Иногда вам может потребоваться измерить время выполнения обратного вызова, сохраняя при этом значение, возвращаемое обратным вызовом. Метод `value` вернет кортеж, содержащий значение, возвращаемое обратным вызовом, и количество миллисекунд, затраченных на выполнение обратного вызова:

    [$count, $duration] = Benchmark::value(fn () => User::count());

<a name="dates"></a>
### Даты

Laravel включает в себя [Carbon](https://carbon.nesbot.com/docs/), мощную библиотеку для манипулирования датой и временем. Чтобы создать новый экземпляр `Carbon`, вы можете вызвать функцию `now`. Эта функция доступна глобально в вашем приложении Laravel:

```php
$now = now();
```

Или же вы можете создать новый экземпляр `Carbon`, используя класс `Illuminate\Support\Carbon`:

```php
use Illuminate\Support\Carbon;

$now = Carbon::now();
```

Подробное описание `Carbon` и его функций можно найти в [официальной документации Carbon](https://carbon.nesbot.com/docs/).

<a name="lottery"></a>
### Лотерея

Класс лотереи Laravel может использоваться для выполнения обратных вызовов на основе заданных шансов. Это может быть особенно полезно, когда вы хотите выполнить код только для определенного процента ваших входящих запросов:

    use Illuminate\Support\Lottery;

    Lottery::odds(1, 20)
        ->winner(fn () => $user->won())
        ->loser(fn () => $user->lost())
        ->choose();

Вы можете комбинировать класс лотереи Laravel с другими функциями Laravel. Например, вы можете захотеть сообщать обработчику исключений только о небольшом проценте медленных запросов. А поскольку класс лотереи является вызываемым, мы можем передать экземпляр класса в любой метод, который принимает вызываемые объекты:

    use Carbon\CarbonInterval;
    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\Lottery;

    DB::whenQueryingForLongerThan(
        CarbonInterval::seconds(2),
        Lottery::odds(1, 100)->winner(fn () => report('Querying > 2 seconds.')),
    );

<a name="testing-lotteries"></a>
#### Тестирование лотерей

Laravel предоставляет несколько простых методов, которые позволяют легко тестировать вызовы лотереи в вашем приложении:

    // Лотерея всегда вииграшная...
    Lottery::alwaysWin();

    // Лотерея всегда проиграшная...
    Lottery::alwaysLose();

    // Выигрыш, проигрыш, затем вернуться к нормальному поведению...
    Lottery::fix([true, false]);

    // Вернуться к нормальному поведению...
    Lottery::determineResultsNormally();

<a name="pipeline"></a>
### Pipeline

Фасад `Pipeline` в Laravel предоставляет удобный способ "прокидывания" ввода через серию вызовов классов, замыканий или вызываемых объектов, предоставляя каждому классу возможность проверить или изменить входные данные и вызвать следующий элемент в цепочке вызовов пайплайна:

```php
use Closure;
use App\Models\User;
use Illuminate\Support\Facades\Pipeline;

$user = Pipeline::send($user)
            ->through([
                function (User $user, Closure $next) {
                    // ...

                    return $next($user);
                },
                function (User $user, Closure $next) {
                    // ...

                    return $next($user);
                },
            ])
            ->then(fn (User $user) => $user);
```


Как видите, каждый вызываемый класс или замыкание указанное в pipeline получает входные данные и замыкание `$next`. Вызов замыкания `$next` приведет к вызову следующего вызываемого объекта в пайплайне. Как вы могли заметить, это очень похоже на [middleware](/docs/{{version}}/middleware).

Когда последний вызываемый объект в пайплайне вызывает `$next`, будет выполнен объект, предоставленный методу `then`. Обычно этот вызываемый объект просто возвращает предоставленные входные данные.

Как было описано ранее, вы не ограничены предоставлением только замыканий в свой пайплайн. Вы также можете использовать вызываемые классы. Если предоставлено имя класса, экземпляр класса будет создан с использованием [контейнера служб Laravel](/docs/{{version}}/container), что позволяет внедрять зависимости в вызываемый класс:

```php
$user = Pipeline::send($user)
            ->through([
                GenerateProfilePhoto::class,
                ActivateSubscription::class,
                SendWelcomeEmail::class,
            ])
            ->then(fn (User $user) => $user);
```

<a name="sleep"></a>
### Sleep

Класс `Sleep` в Laravel представляет собой легковесную обертку вокруг нативных функций PHP `sleep` и `usleep`, предоставляя большую тестируемость и удобный API для работы с временем:


    use Illuminate\Support\Sleep;

    $waiting = true;

    while ($waiting) {
        Sleep::for(1)->second();

        $waiting = /* ... */;
    }

Класс `Sleep` предоставляет разнообразные методы, позволяющие вам работать с различными единицами времени:

    //Приостановите выполнение на 90 секунд...
    Sleep::for(1.5)->minutes();

    // Приостановите выполнение на 2 секунды...
    Sleep::for(2)->seconds();

    // Pause execution for 500 milliseconds...
    Sleep::for(500)->milliseconds();

    // Приостановите выполнение на 500 миллисекунд...
    Sleep::for(5000)->microseconds();

    // Приостановить выполнение до заданного времени...
    Sleep::until(now()->addMinute());

    // Псевдоним функции PHP "sleep"...
    Sleep::sleep(2);

    // Псевдоним функции PHP  "usleep"
    Sleep::usleep(5000);

Чтобы легко объединять единицы времени, вы можете использовать метод `and`:

    Sleep::for(1)->second()->and(10)->milliseconds();

<a name="testing-sleep"></a>
#### Тестирование Sleep

При тестировании кода, использующего класс `Sleep` или функции PHP `sleep` , выполнение вашего теста будет приостановлено. Как можно ожидать, это делает ваш пакет тестов значительно медленнее. Например, представьте, что вы тестируете следующий код:


    $waiting = /* ... */;

    $seconds = 1;

    while ($waiting) {
        Sleep::for($seconds++)->seconds();

        $waiting = /* ... */;
    }


Обычно тестирование этого кода займет как минимум одну секунду. К счастью, класс `Sleep` позволяет нам "подделывать" задержку, чтобы наш тестовый набор оставался быстрым:

    public function test_it_waits_until_ready()
    {
        Sleep::fake();

        // ...
    }


При подделке класса `Sleep` реальная задержка выполнения обходится, что приводит к более быстрому тестированию.

Как только класс `Sleep` был подделан, можно делать утверждения относительно ожидаемых "пауз". Для иллюстрации давайте представим, что мы тестируем код, который приостанавливает выполнение три раза, при этом каждая задержка увеличивается на одну секунду. Используя метод `assertSequence`, мы можем проверить, что наш код "спал" нужное количество времени, сохраняя при этом скорость выполнения теста:

    public function test_it_checks_if_ready_four_times()
    {
        Sleep::fake();

        // ...

        Sleep::assertSequence([
            Sleep::for(1)->second(),
            Sleep::for(2)->seconds(),
            Sleep::for(3)->seconds(),
        ]);
    }

Конечно же, класс Sleep предоставляет и другие утверждения, которые вы можете использовать при тестировании:


    use Carbon\CarbonInterval as Duration;
    use Illuminate\Support\Sleep;

    // Утверждение, что sliip вызывали 3 раза...
    Sleep::assertSleptTimes(3);

    // Утверждение, что продолжительность сна... 
    Sleep::assertSlept(function (Duration $duration): bool {
        return /* ... */;
    }, times: 1);

    // Утверждение, что класс Sleep никогда не вызывался...
    Sleep::assertNeverSlept();

    // Утверждение, что, даже если был вызван Sleep, пауза в выполнении не наступила...
    Sleep::assertInsomniac();

Иногда бывает полезно выполнять действие при каждом имитированном ожидании в коде вашего приложения. Для этого вы можете предоставить обратный вызов методу `whenFakingSleep`. В следующем примере мы используем помощники Laravel по [манипулированию временем](/docs/{{version}}/mocking#interacting-with-time), чтобы мгновенно продвинуть время на продолжительность каждого ожидания:


```php
use Carbon\CarbonInterval as Duration;

$this->freezeTime();

Sleep::fake();

Sleep::whenFakingSleep(function (Duration $duration) {
    // Progress time when faking sleep...
    $this->travel($duration->totalMilliseconds)->milliseconds();
});
```

Класс `Sleep` используется внутри Laravel при приостановке выполнения. Например, помощник [retry](#method-retry) использует класс `Sleep` при задержке, что обеспечивает лучшую тестируемость при использовании данного помощника.
