git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Коллекции

- [Введение](#introduction)
    - [Создание коллекций](#creating-collections)
- [Доступные методы](#available-methods)
- [Операции высшего порядка](#higher-order-messages)

<a name="introduction"></a>
## Введение

Класс `Illuminate\Support\Collection` предоставляет гибкую и удобную обёртку для работы с массивами данных. Например, посмотрите на следующий код. Мы будем использовать хелпер `collect`, чтобы создать новый экземпляр коллекции из массива, выполним функцию `strtoupper` для каждого элемента, а затем удалим все пустые элементы:

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    });


Как видите, класс `Collection` позволяет использовать свои методы в связке для гибкого отображения и уменьшения исходного массива. В общем, коллекции "неизменны", то есть каждый метод класса `Collection` возвращает совершенно новый экземпляр `Collection`.

<a name="creating-collections"></a>
### Создание коллекций

Как упоминалось выше, хелпер `collect` возвращает новый экземпляр класса `Illuminate\Support\Collection` для заданного массива. Поэтому создать коллекцию очень просто:

    $collection = collect([1, 2, 3]);

> {tip} Результаты запросов [Eloquent](/docs/{{version}}/eloquent) всегда возвращаются в виде экземпляров класса `Collection`.

<a name="available-methods"></a>
## Доступные методы

В остальной части данной документации, мы будем обсуждать каждый метод, доступный в классе `Collection`. Помните, все эти методы могут использоваться в связке для гибкого управления заданным массивом. Кроме того, почти каждый метод возвращает новый экземпляр класса `Collection`, позволяя вам при необходимости сохранить оригинал коллекции:

<style>
    #collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[collapse](#method-collapse)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsStrict](#method-containsstrict)
[count](#method-count)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffKeys](#method-diffkeys)
[each](#method-each)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectKey](#method-intersectkey)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[map](#method-map)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[median](#method-median)
[merge](#method-merge)
[min](#method-min)
[mode](#method-mode)
[nth](#method-nth)
[only](#method-only)
[partition](#method-partition)
[pipe](#method-pipe)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[reverse](#method-reverse)
[search](#method-search)
[shift](#method-shift)
[shuffle](#method-shuffle)
[slice](#method-slice)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[splice](#method-splice)
[split](#method-split)
[sum](#method-sum)
[take](#method-take)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[values](#method-values)
[when](#method-when)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[zip](#method-zip)

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

<a name="method-all"></a>
#### `all()` {#collection-method .first-collection-method}

Метод `all` возвращает заданный массив, представленный коллекцией:

    collect([1, 2, 3])->all();

    // [1, 2, 3]

<a name="method-average"></a>
#### `average()` {#collection-method}

Псевдоним метода [`avg`](#method-avg).

<a name="method-avg"></a>
#### `avg()` {#collection-method}

Метод `avg` возвращает [среднее значение](https://en.wikipedia.org/wiki/Average) переданного ключа:

    $average = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->avg('foo');

    // 20

    $average = collect([1, 1, 2, 4])->avg();

    // 2

<a name="method-chunk"></a>
#### `chunk()` {#collection-method}

Метод `chunk` разбивает коллекцию на множество мелких коллекций заданного размера:

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->toArray();

    // [[1, 2, 3, 4], [5, 6, 7]]

Этот метод особенно полезен в [шаблонах](/docs/{{version}}/views) при работе с системой сеток, такой как [Bootstrap](https://getbootstrap.com/css/#grid). Представьте, что у вас есть коллекция моделей [Eloquent](/docs/{{version}}/eloquent), которую вы хотите отобразить в сетке:

    @foreach ($products->chunk(3) as $chunk)
        <div class="row">
            @foreach ($chunk as $product)
                <div class="col-xs-4">{{ $product->name }}</div>
            @endforeach
        </div>
    @endforeach

<a name="method-collapse"></a>
#### `collapse()` {#collection-method}

Метод `collapse` сворачивает коллекцию массивов в одну одномерную коллекцию:

    $collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-combine"></a>
#### `combine()` {#collection-method}

Метод `combine` комбинирует ключи коллекции со значениями другого массива или коллекции:

    $collection = collect(['name', 'age']);

    $combined = $collection->combine(['George', 29]);

    $combined->all();

    // ['name' => 'George', 'age' => 29]

<a name="method-concat"></a>
#### `concat()` {#collection-method}
 
Метод `concat` добавляет массив в конец коллекции. Если массив ассоциативный, ключи игнорируются.
 
    $collection = collect(['John Doe']);
 
    $concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);
 
    $concatenated->all();
 
    // ['John Doe', 'Jane Doe', 'Johnny Doe']     

<a name="method-contains"></a>
#### `contains()` {#collection-method}

Метод `contains` определяет, содержит ли коллекция заданное значение:

    $collection = collect(['name' => 'Desk', 'price' => 100]);

    $collection->contains('Desk');

    // true

    $collection->contains('New York');

    // false

Также вы можете передать пару ключ/значение в метод `contains`, определяющий, существует ли заданная пара в коллекции:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->contains('product', 'Bookcase');

    // false

Напоследок, вы можете передать функцию обратного вызова в метод `contains` для выполнения своих собственных условий:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->contains(function ($value, $key) {
        return $value > 5;
    });

    // false

Метод `contains` использует "неточные" сравнения при проверке значений элементов; то есть строка с целым значением будет считаться равной целому числу с тем же значением. Используйте метод [`containsStrict`](#method-containsstrict) для фильтрации с использованием строгих сравнений.

<a name="method-containsstrict"></a>
#### `containsStrict()` {#collection-method}

Этот метод использует ту же сигнатуру, как и метод [`contains`](#method-contains); однако, все значения сравниваются с использованием "строгих" сравнений.

<a name="method-count"></a>
#### `count()` {#collection-method}

Метод `count` возвращает общее количество элементов в коллекции:

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-diff"></a>
#### `diff()` {#collection-method}

Метод `diff` сравнивает одну коллекцию с другой коллекцией или с простым массивом, основываясь на его значениях. Этот метод вернёт те значения исходной коллекции, которых нет в переданной для сравнения коллекции:

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]

<a name="method-diffassoc"></a>
#### `diffAssoc()` {#collection-method}

Метод `diffAssoc` сравнивает коллекцию с другой коллекцией или простым массивом, основываясь на ключах и значениях. Этот метод вернёт те пары ключ/значение исходной коллекции, которых нет в переданной для сравнения коллекции:

    $collection = collect([
        'color' => 'orange',
        'type' => 'fruit',
        'remain' => 6
    ]);
    
    $diff = $collection->diffAssoc([
        'color' => 'yellow',
        'type' => 'fruit',
        'remain' => 3,
        'used' => 6
    ]);
    
    $diff->all();
    
    // ['color' => 'orange', 'remain' => 6]

<a name="method-diffkeys"></a>
#### `diffKeys()` {#collection-method}

Метод `diffKeys` сравнивает одну коллекцию с другой коллекцией или с простым массивом на основе их ключей. Этот метод вернёт те пары ключ/значение из исходной коллекции, которых нет в переданной для сравнения коллекции:

    $collection = collect([
        'one' => 10,
        'two' => 20,
        'three' => 30,
        'four' => 40,
        'five' => 50,
    ]);

    $diff = $collection->diffKeys([
        'two' => 2,
        'four' => 4,
        'six' => 6,
        'eight' => 8,
    ]);

    $diff->all();

    // ['one' => 10, 'three' => 30, 'five' => 50]

<a name="method-each"></a>
#### `each()` {#collection-method}

Метод `each` перебирает элементы в коллекции и передает каждый элемент в функцию обратного вызова:

    $collection = $collection->each(function ($item, $key) {
        //
    });

Верните `false` из функции обратного вызова, чтобы выйти из цикла:

    $collection = $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });

<a name="method-every"></a>
#### `every()` {#collection-method}

Метод `every` можно использовать, чтобы проверить, что все элементы коллекции прошли проверку на истинность:

    collect([1, 2, 3, 4])->every(function ($value, $key) {
        return $value > 2;
    });

    // false

<a name="method-except"></a>
#### `except()` {#collection-method}

Метод `except` возвращает все элементы в коллекции, кроме тех, чьи ключи указаны в передаваемом массиве:

    $collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

    $filtered = $collection->except(['price', 'discount']);

    $filtered->all();

    // ['product_id' => 1]

Метод [only](#method-only) - инверсный методу `except`.

<a name="method-filter"></a>
#### `filter()` {#collection-method}

Метод `filter` фильтрует коллекцию с помощью переданной функции обратного вызова, оставляя только те элементы, которые соответствуют заданному условию:

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->filter(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [3, 4]

Если анонимная функция не указана, будут удалены все элементы коллекции, эквивалентные `false`:

    $collection = collect([1, 2, 3, null, false, '', 0, []]);

    $collection->filter()->all();

    // [1, 2, 3]

Метод [reject](#method-reject) - инверсный методу `filter`.

<a name="method-first"></a>
#### `first()` {#collection-method}

Метод `first` возвращает первый элемент в коллекции, который подходит под заданное условие:

    collect([1, 2, 3, 4])->first(function ($value, $key) {
        return $value > 2;
    });

    // 3

Также вы можете вызвать метод `first` без параметров, чтобы получить первый элемент в коллекции. Если коллекция пуста, то вернётся `null`:

    collect([1, 2, 3, 4])->first();

    // 1

<a name="method-flatmap"></a>
#### `flatMap()` {#collection-method}

Метод `flatMap` проходит по коллекции и передаёт каждое значение в заданную функцию обратного вызова. Эта функция может изменить элемент и вернуть его, формируя таким образом новую коллекцию модифицированных элементов. Затем массив "сплющивается" в одномерный:

    $collection = collect([
        ['name' => 'Sally'],
        ['school' => 'Arkansas'],
        ['age' => 28]
    ]);

    $flattened = $collection->flatMap(function ($values) {
        return array_map('strtoupper', $values);
    });

    $flattened->all();

    // ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];

<a name="method-flatten"></a>
#### `flatten()` {#collection-method}

Метод `flatten` преобразует многомерную коллекцию в одномерную:

    $collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

    $flattened = $collection->flatten();

    $flattened->all();

    // ['taylor', 'php', 'javascript'];

При необходимости вы можете передать в метод аргумент "глубины":

    $collection = collect([
        'Apple' => [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ],
        'Samsung' => [
            ['name' => 'Galaxy S7', 'brand' => 'Samsung']
        ],
    ]);

    $products = $collection->flatten(1);

    $products->values()->all();

    /*
        [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
            ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
        ]
    */

В этом примере: если вызвать `flatten` без указания глубины, то вложенные массивы тоже «расплющатся», и получим `['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`. Глубина задаёт уровень вложенности массивов, ниже которого "расплющивать" не нужно.

<a name="method-flip"></a>
#### `flip()` {#collection-method}

Метод `flip` меняет местами ключи и значения в коллекции:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $flipped = $collection->flip();

    $flipped->all();

    // ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>
#### `forget()` {#collection-method}

Метод `forget` удаляет элемент из коллекции по его ключу:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $collection->forget('name');

    $collection->all();

    // ['framework' => 'laravel']

> {note} В отличие от большинства других методов коллекции, `forget` не возвращает новую модифицированную коллекцию. Он изменяет коллекцию при вызове.

<a name="method-forpage"></a>
#### `forPage()` {#collection-method}

Метод `forPage` возвращает новую коллекцию, содержащую элементы, которые будут присутствовать на странице с заданным номером. Первый аргумент метода — номер страницы, второй аргумент — число элементов для вывода на странице:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunk = $collection->forPage(2, 3);

    $chunk->all();

    // [4, 5, 6]

<a name="method-get"></a>
#### `get()` {#collection-method}

Метод `get` возвращает нужный элемент по заданному ключу. Если ключ не существует, то возвращается `null`:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('name');

    // taylor

Вторым параметром вы можете передать значение по умолчанию:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('foo', 'default-value');

    // default-value

Вы даже можете передать функцию обратного вызова в качестве значения по умолчанию. Результат функции обратного вызова будет возвращён, если указанный ключ не существует:

    $collection->get('email', function () {
        return 'default-value';
    });

    // default-value

<a name="method-groupby"></a>
#### `groupBy()` {#collection-method}

Метод `groupBy` группирует элементы коллекции по заданному ключу:

    $collection = collect([
        ['account_id' => 'account-x10', 'product' => 'Chair'],
        ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ['account_id' => 'account-x11', 'product' => 'Desk'],
    ]);

    $grouped = $collection->groupBy('account_id');

    $grouped->toArray();

    /*
        [
            'account-x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'account-x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

В дополнение к передаваемой строке `key`, вы можете также передать функцию обратного вызова. Она должна возвращать значение, по которому вы хотите группировать:

    $grouped = $collection->groupBy(function ($item, $key) {
        return substr($item['account_id'], -3);
    });

    $grouped->toArray();

    /*
        [
            'x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

<a name="method-has"></a>
#### `has()` {#collection-method}

Метод `has` определяет, существует ли заданный ключ в коллекции:

    $collection = collect(['account_id' => 1, 'product' => 'Desk']);

    $collection->has('product');

    // true

<a name="method-implode"></a>
#### `implode()` {#collection-method}

Метод `implode` соединяет элементы в коллекции. Его параметры зависят от типа элементов в коллекции. Если коллекция содержит массивы или объекты, вы должны передать ключ атрибутов, значения которых вы хотите соединить, и "промежуточную" строку, которую вы хотите поместить между значениями:

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);

    $collection->implode('product', ', ');

    // Desk, Chair

Если коллекция содержит простые строки или числовые значения, просто передайте только "промежуточный" параметр в метод:

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

<a name="method-intersect"></a>
#### `intersect()` {#collection-method}

Метод `intersect` удаляет любые значения из исходной коллекции, которых нет в переданном `array` или коллекции. Результирующая коллекция сохранит ключи оригинальной коллекции:

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

<a name="method-intersectkey"></a>
#### `intersectKey()` {#collection-method}

Метод `intersectKey` удаляет любые ключи из исходной коллекции, которых нет в переданном `array` или коллекции:

    $collection = collect([
        'serial' => 'UX301', 'type' => 'screen', 'year' => 2009
    ]);

    $intersect = $collection->intersectKey([
        'reference' => 'UX404', 'type' => 'tab', 'year' => 2011
    ]);

    $intersect->all();

    // ['type' => 'screen', 'year' => 2009]

<a name="method-isempty"></a>
#### `isEmpty()` {#collection-method}

Метод `isEmpty` возвращает `true`, если коллекция пуста. В противном случае вернётся `false`:

    collect([])->isEmpty();

    // true

<a name="method-isnotempty"></a>
#### `isNotEmpty()` {#collection-method}

Метод `isNotEmpty` возвращает `true`, если коллекция не пуста; в противном случае вернётся `false`:

    collect([])->isNotEmpty();

    // false

<a name="method-keyby"></a>
#### `keyBy()` {#collection-method}

Метод `keyBy` возвращает коллекцию по указанному ключу. Если несколько элементов имеют одинаковый ключ, в результирующей коллекции появится только последний их них:

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'desk'],
        ['product_id' => 'prod-200', 'name' => 'chair'],
    ]);

    $keyed = $collection->keyBy('product_id');

    $keyed->all();

    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

Также вы можете передать в метод функцию обратного вызова, которая должна возвращать значение ключа коллекции для этого метода:

    $keyed = $collection->keyBy(function ($item) {
        return strtoupper($item['product_id']);
    });

    $keyed->all();

    /*
        [
            'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

<a name="method-keys"></a>
#### `keys()` {#collection-method}

Метод `keys` возвращает все ключи коллекции:

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keys = $collection->keys();

    $keys->all();

    // ['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` {#collection-method}

Метод `last` возвращает последний элемент в коллекции, который проходит проверку на истинность:

    collect([1, 2, 3, 4])->last(function ($value, $key) {
        return $value < 3;
    });

    // 2

Также вы можете вызвать метод `last` без параметров, чтобы получить последний элемент в коллекции. Если коллекция пуста, то вернётся `null`:

    collect([1, 2, 3, 4])->last();

    // 4

<a name="method-map"></a>
#### `map()` {#collection-method}

Метод `map` перебирает коллекцию и передаёт каждое значению в функцию обратного вызова. Функция обратного вызова может свободно изменять элемент и возвращать его, формируя тем самым новую коллекцию измененных элементов:

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });

    $multiplied->all();

    // [2, 4, 6, 8, 10]

> {note} Как и большинство других методов коллекции, метод `map` возвращает новый экземпляр коллекции. Он не изменяет коллекцию при вызове. Если вы хотите преобразовать оригинальную коллекцию, используйте метод[`transform`](#method-transform).

<a name="method-mapwithkeys"></a>
#### `mapWithKeys()` {#collection-method}

Метод `mapWithKeys` проходит по элементам коллекции и передаёт каждое значение в функцию обратного вызова, которая должна вернуть ассоциативный массив, содержащий одну пару ключ/значение:

    $collection = collect([
        [
            'name' => 'John',
            'department' => 'Sales',
            'email' => 'john@example.com'
        ],
        [
            'name' => 'Jane',
            'department' => 'Marketing',
            'email' => 'jane@example.com'
        ]
    ]);

    $keyed = $collection->mapWithKeys(function ($item) {
        return [$item['email'] => $item['name']];
    });

    $keyed->all();

    /*
        [
            'john@example.com' => 'John',
            'jane@example.com' => 'Jane',
        ]
    */

<a name="method-max"></a>
#### `max()` {#collection-method}

Метод `max`  возвращает максимальное значение по заданному ключу:

    $max = collect([['foo' => 10], ['foo' => 20]])->max('foo');

    // 20

    $max = collect([1, 2, 3, 4, 5])->max();

    // 5

<a name="method-median"></a>
#### `median()` {#collection-method}

Метод `median` возвращает [медианное значение](https://en.wikipedia.org/wiki/Median) заданного ключа:

    $median = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->median('foo');

    // 15

    $median = collect([1, 1, 2, 4])->median();

    // 1.5

<a name="method-merge"></a>
#### `merge()` {#collection-method}

Метод `merge` добавляет указанный массив в исходную коллекцию. Значения исходной коллекции, имеющие тот же строковый ключ, что и значение в массиве, будут перезаписаны:

    $collection = collect(['product_id' => 1, 'price' => 100]);

    $merged = $collection->merge(['price' => 200, 'discount' => false]);

    $merged->all();

    // ['product_id' => 1, 'price' => 200, 'discount' => false]

Если заданные ключи в массиве числовые, то значения будут добавляться в конец коллекции:

    $collection = collect(['Desk', 'Chair']);

    $merged = $collection->merge(['Bookcase', 'Door']);

    $merged->all();

    // ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-min"></a>
#### `min()` {#collection-method}

Метод `min` возвращает минимальное значение по заданному ключу:

    $min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

    // 10

    $min = collect([1, 2, 3, 4, 5])->min();

    // 1

<a name="method-mode"></a>
#### `mode()` {#collection-method}

Метод `mode` возвращает [значение мода](https://en.wikipedia.org/wiki/Mode_(statistics)) заданного ключа:

    $mode = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->mode('foo');

    // [10]

    $mode = collect([1, 1, 2, 4])->mode();

    // [1]

<a name="method-nth"></a>
#### `nth()` {#collection-method}

Метод `nth` создает новую коллекцию, состоящую из каждого n-ного элемента:

    $collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

    $collection->nth(4);

    // ['a', 'e']

Можно передать смещение в качестве необязательного второго аргумента:

    $collection->nth(4, 1);

    // ['b', 'f']

<a name="method-only"></a>
#### `only()` {#collection-method}

Метод `only` возвращает элементы коллекции с заданными ключами:

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

    $filtered = $collection->only(['product_id', 'name']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

Метод [except](#method-except) - инверсный для метода `only`.

<a name="method-partition"></a>
#### `partition()` {#collection-method}

Метод `partition` можно объединить с функцией PHP `list`, чтобы отделить элементы, которые прошли заданную проверку на истинность от тех элементов, которые её не прошли:

    $collection = collect([1, 2, 3, 4, 5, 6]);

    list($underThree, $aboveThree) = $collection->partition(function ($i) {
        return $i < 3;
    });

<a name="method-pipe"></a>
#### `pipe()` {#collection-method}

Метод `pipe` передает коллекцию в заданную функцию и возвращает результат:

    $collection = collect([1, 2, 3]);

    $piped = $collection->pipe(function ($collection) {
        return $collection->sum();
    });

    // 6

<a name="method-pluck"></a>
#### `pluck()` {#collection-method}

Метод `pluck` извлекает все значения по заданному ключу:

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $plucked = $collection->pluck('name');

    $plucked->all();

    // ['Desk', 'Chair']

Также вы можете указать, с каким ключом вы хотите получить коллекцию:

    $plucked = $collection->pluck('name', 'product_id');

    $plucked->all();

    // ['prod-100' => 'Desk', 'prod-200' => 'Chair']

<a name="method-pop"></a>
#### `pop()` {#collection-method}

Метод `pop` удаляет и возвращает последний элемент из коллекции:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop();

    // 5

    $collection->all();

    // [1, 2, 3, 4]

<a name="method-prepend"></a>
#### `prepend()` {#collection-method}

Метод `prepend` добавляет элемент в начало коллекции:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    // [0, 1, 2, 3, 4, 5]

Вторым аргументом вы можете передать ключ добавляемого элемента:

    $collection = collect(['one' => 1, 'two' => 2]);

    $collection->prepend(0, 'zero');

    $collection->all();

    // ['zero' => 0, 'one' => 1, 'two' => 2]

<a name="method-pull"></a>
#### `pull()` {#collection-method}

Метод `pull` удаляет и возвращает элемент из коллекции по его ключу:

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    // 'Desk'

    $collection->all();

    // ['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` {#collection-method}

Метод `push` добавляет элемент в конец коллекции:

    $collection = collect([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    // [1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` {#collection-method}

Метод `put` устанавливает заданный ключ и значение в коллекцию:

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {#collection-method}

Метод `random` возвращает случайный элемент из коллекции:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->random();

    // 4 - (получен в случайном порядке)

Также вы можете передать целое число в `random`, чтобы указать, сколько случайных элементов необходимо получить. Всегда возвращается коллекция элементов при явной передаче количества элементов, которые вам необходимо получить:

    $random = $collection->random(3);

    $random->all();

    // [2, 4, 5] - (получены в случайном порядке)

Если коллекция содержит меньше элементов, чем запрошено, то выбрасывается исключение `InvalidArgumentException`.    

<a name="method-reduce"></a>
#### `reduce()` {#collection-method}

Метод `reduce` уменьшает коллекцию до одного значения, передавая результат каждой итерации в последующую итерацию:

    $collection = collect([1, 2, 3]);

    $total = $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    });

    // 6

Значение для `$carry` в первой итерации — `null`; однако, вы можете указать его начальное значение во втором параметре метода `reduce`:

    $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    }, 4);

    // 10

<a name="method-reject"></a>
#### `reject()` {#collection-method}

Метод `reject` фильтрует коллекцию, используя заданную функцию обратного вызова. Функция обратного вызова должна возвращать `true` для элементов, которые необходимо удалить из результирующей коллекции:

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [1, 2]

Метод [`filter`](#method-filter) - инверсный для метода `reject`.

<a name="method-reverse"></a>
#### `reverse()` {#collection-method}

Метод `reverse` меняет порядок элементов коллекции:

    $collection = collect([1, 2, 3, 4, 5]);

    $reversed = $collection->reverse();

    $reversed->all();

    // [5, 4, 3, 2, 1]

<a name="method-search"></a>
#### `search()` {#collection-method}

Метод `search` ищет в коллекции заданное значение и возвращает его ключ при успешном поиске. Если элемент не найден, то возвращается `false`.

    $collection = collect([2, 4, 6, 8]);

    $collection->search(4);

    // 1

Поиск проводится с помощью "неточного" сравнения, то есть строка с числовым значением будет считаться равной числу с таким же значением. Чтобы использовать строгое сравнение, передайте `true` вторым параметром метода:

    $collection->search('4', true);

    // false

В качестве альтернативы, вы можете передать свою собственную функцию обратного вызова для поиска первого элемента, для которого выполняется ваше условие:

    $collection->search(function ($item, $key) {
        return $item > 5;
    });

    // 2

<a name="method-shift"></a>
#### `shift()` {#collection-method}

Метод `shift` удаляет и возвращает первый элемент из коллекции:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->shift();

    // 1

    $collection->all();

    // [2, 3, 4, 5]

<a name="method-shuffle"></a>
#### `shuffle()` {#collection-method}

Метод `shuffle` перемешивает элементы в коллекции случайным образом::

    $collection = collect([1, 2, 3, 4, 5]);

    $shuffled = $collection->shuffle();

    $shuffled->all();

    // [3, 2, 5, 1, 4] - (generated randomly)

<a name="method-slice"></a>
#### `slice()` {#collection-method}

Метод `slice` возвращает часть коллекции, начиная с заданного индекса:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    // [5, 6, 7, 8, 9, 10]

Если вы хотите ограничить размер получаемой части коллекции, передайте желаемый размер вторым параметром в метод:

    $slice = $collection->slice(4, 2);

    $slice->all();

    // [5, 6]

Полученная часть коллекции сохранит оригинальные ключи. Если вы не хотите сохранять оригинальные ключи, то можете использовать метод [`values`](#method-values), чтобы переиндексировать их.

<a name="method-sort"></a>
#### `sort()` {#collection-method}

Метод `sort` сортирует коллекцию. Отсортированная коллекция сохраняет оригинальные ключи массива, поэтому в этом примере мы используем метод [`values`](#method-values) для сброса ключей и последовательной нумерации индексов:

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sort();

    $sorted->values()->all();

    // [1, 2, 3, 4, 5]

Если вам необходимо отсортировать коллекцию с дополнительными условиями, вы можете передать функцию обратного вызова в метод `sort` с вашим собственным алгоритмом. См. PHP документацию по [`usort`](https://secure.php.net/manual/en/function.usort.php#refsect1-function.usort-parameters), который вызывается внутри метода `sort` вашей коллекции.

> {tip} Для сортировки коллекции вложенных массивов или объектов, смотрите методы [`sortBy`](#method-sortby) и [`sortByDesc`](#method-sortbydesc) methods.

<a name="method-sortby"></a>
#### `sortBy()` {#collection-method}

Метод `sortBy` сортирует коллекцию по заданному ключу. Отсортированная коллекция сохраняет оригинальные ключи массива, поэтому в этом примере мы используем метод [`values`](#method-values) для сброса ключей и последовательной нумерации индексов:

    $collection = collect([
        ['name' => 'Desk', 'price' => 200],
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
    ]);

    $sorted = $collection->sortBy('price');

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'price' => 100],
            ['name' => 'Bookcase', 'price' => 150],
            ['name' => 'Desk', 'price' => 200],
        ]
    */

Также вы можете передать свою собственную анонимную функцию, чтобы определить как сортировать значения коллекции:

    $collection = collect([
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $sorted = $collection->sortBy(function ($product, $key) {
        return count($product['colors']);
    });

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'colors' => ['Black']],
            ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
            ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
        ]
    */

<a name="method-sortbydesc"></a>
#### `sortByDesc()` {#collection-method}

Этот метод использует такую же сигнатуру, как и метод [`sortBy`](#method-sortby), но будет сортировать коллекцию в обратном порядке.

<a name="method-splice"></a>
#### `splice()` {#collection-method}

Метод `splice` удаляет и возвращает часть элементов, начиная с заданного индекса:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    // [3, 4, 5]

    $collection->all();

    // [1, 2]

Вы можете передать второй параметр в метод для ограничения размера возвращаемой части коллекции:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 4, 5]

Также вы можете передать в метод третий параметр, содержащий новые элементы, чтобы заменить элементы, которые будут удалены из коллекции:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 10, 11, 4, 5]

<a name="method-split"></a>
#### `split()` {#collection-method}

Метод `split` разбивает коллекцию на заданное число групп:

    $collection = collect([1, 2, 3, 4, 5]);

    $groups = $collection->split(3);

    $groups->toArray();

    // [[1, 2], [3, 4], [5]]

<a name="method-sum"></a>
#### `sum()` {#collection-method}

Метод `sum` возвращает сумму всех элементов в коллекции:

    collect([1, 2, 3, 4, 5])->sum();

    // 15

Если коллекция содержит вложенные массивы или объекты, вам нужно передать ключ для определения значений, которые нужно суммировать:

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->sum('pages');

    // 1272

Также вы можете передать свою собственную функцию обратного вызова, чтобы определить, какие значения коллекции суммировать:

    $collection = collect([
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $collection->sum(function ($product) {
        return count($product['colors']);
    });

    // 6

<a name="method-take"></a>
#### `take()` {#collection-method}

Метод `take` возвращает новую коллекцию с заданным числом элементов:

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

Также вы можете передать отрицательное целое число, чтобы получить определенное количество элементов с конца коллекции:

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-tap"></a>
#### `tap()` {#collection-method}

Метод `tap` передает коллекцию заданной анонимной функции, что позволяет "подключиться" к коллекции в определенный момент и сделать что-либо с элементами, не оказывая влияния на саму коллекцию:

    collect([2, 4, 3, 1, 5])
        ->sort()
        ->tap(function ($collection) {
            Log::debug('Values after sorting', $collection->values()->toArray());
        })
        ->shift();

    // 1

<a name="method-times"></a>
#### `times()` {#collection-method}

Статический метод `times` создает новую коллекцию, вызывая функцию заданное количество раз:

    $collection = Collection::times(10, function ($number) {
        return $number * 9;
    });

    $collection->all();

    // [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]

Этот метод может быть полезен при комбинировании с фабрикой классов для создания моделей [Eloquent](/docs/{{version}}/eloquent):

    $categories = Collection::times(3, function ($number) {
        return factory(Category::class)->create(['name' => 'Category #'.$number]);
    });

    $categories->all();

    /*
        [
            ['id' => 1, 'name' => 'Category #1'],
            ['id' => 2, 'name' => 'Category #2'],
            ['id' => 3, 'name' => 'Category #3'],
        ]
    */

<a name="method-toarray"></a>
#### `toArray()` {#collection-method}

Метод `toArray` преобразует коллекцию в простой массив. Если значения коллекции являются моделями [Eloquent](/docs/{{version}}/eloquent), то модели также будут преобразованы в массивы:

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> {note} Метод `toArray` также преобразует все вложенные объекты коллекции в массив. Если вы хотите получить базовый массив, используйте вместо этого метод [`all`](#method-all).

<a name="method-tojson"></a>
#### `toJson()` {#collection-method}

Метод `toJson` преобразует коллекцию в упорядоченную строку JSON:

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    // '{"name":"Desk", "price":200}'

<a name="method-transform"></a>
#### `transform()` {#collection-method}

Метод `transform` перебирает коллекцию и вызывает заданную функцию обратного вызова для каждого элемента коллекции. Элементы коллекции будут заменены на значения, полученные из функции обратного вызова:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->transform(function ($item, $key) {
        return $item * 2;
    });

    $collection->all();

    // [2, 4, 6, 8, 10]

> {note} В отличие от большинства других методов коллекции, transform() изменяет саму коллекцию. Если вместо этого вы хотите создать новую коллекцию, используйте метод [`map`](#method-map).

<a name="method-union"></a>
#### `union()` {#collection-method}

Метод `union` добавляет данный массив в коллекцию. Если массив содержит ключи, которые уже есть в исходной коллекции, то будут оставлены значения исходной коллекции:

    $collection = collect([1 => ['a'], 2 => ['b']]);

    $union = $collection->union([3 => ['c'], 1 => ['b']]);

    $union->all();

    // [1 => ['a'], 2 => ['b'], 3 => ['c']]

<a name="method-unique"></a>
#### `unique()` {#collection-method}

Метод `unique` возвращает все уникальные элементы в коллекции. Полученная коллекция сохраняет оригинальные ключи массива, поэтому в этом примере мы используем метод [`values`](#method-values) для сброса ключей последовательной нумерации индексов:

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    // [1, 2, 3, 4]

Имея дело со вложенными массивами или объектами, вы можете задать ключ, используемый для определения уникальности:

    $collection = collect([
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]);

    $unique = $collection->unique('brand');

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ]
    */

Также вы можете передать свою собственную анонимную функцию, чтобы определять уникальность элементов:

    $unique = $collection->unique(function ($item) {
        return $item['brand'].$item['type'];
    });

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
            ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
        ]
    */

Метод `unique` использует "неточные" сравнения при проверке значений элементов, то есть строка с целым значением будет считаться равной целому числу с тем же значением. Используйте метод [`uniqueStrict`](#method-uniquestrict) для фильтрации с использованием строгих сравнений.

<a name="method-uniquestrict"></a>
#### `uniqueStrict()` {#collection-method}

У этого метода та же сигнатура, как и у метода [`unique`](#method-unique); однако, все значения сравниваются путем "строгих" сравнений.

<a name="method-values"></a>
#### `values()` {#collection-method}

Метод `values` возвращает новую коллекцию со сброшенными ключами и последовательно пронумерованными индексами:

    $collection = collect([
        10 => ['product' => 'Desk', 'price' => 200],
        11 => ['product' => 'Desk', 'price' => 200]
    ]);

    $values = $collection->values();

    $values->all();

    /*
        [
            0 => ['product' => 'Desk', 'price' => 200],
            1 => ['product' => 'Desk', 'price' => 200],
        ]
    */

<a name="method-when"></a>
#### `when()` {#collection-method}

Метод `when` выполнит заданную анонимную функцию, когда первый переданный методу элемент будет равен `true`:

    $collection = collect([1, 2, 3]);

    $collection->when(true, function ($collection) {
        return $collection->push(4);
    });

    $collection->all();

    // [1, 2, 3, 4]

<a name="method-where"></a>
#### `where()` {#collection-method}

Метод `where` фильтрует коллекцию по заданной паре ключ/значение:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->where('price', 100);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

Метод `where` использует "неточные" сравнения при проверке значений элементов, то есть строка с целым значением будет считаться равной целому числу с тем же значением. Используйте метод [`whereStrict`](#method-wherestrict) для фильтрации с использованием строгих сравнений.

<a name="method-wherestrict"></a>
#### `whereStrict()` {#collection-method}

Этот метод имеет такую же сигнатуру, как и метод [`where`](#method-where); однако, все значения сравниваются с использованием строгого сравнения.

<a name="method-wherein"></a>
#### `whereIn()` {#collection-method}

Метод `whereIn` фильтрует коллекцию по заданным ключу/значению, содержащимся в данном массиве:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereIn('price', [150, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Bookcase', 'price' => 150],
            ['product' => 'Desk', 'price' => 200],
        ]
    */

Метод `whereIn` использует "неточные" сравнения при проверке значений элементов, то есть строка с целым значением будет считаться равной целому числу с тем же значением. Используйте метод [`whereInStrict`](#method-whereinstrict) для фильтрации с использованием строгих сравнений.

<a name="method-whereinstrict"></a>
#### `whereInStrict()` {#collection-method}

Этот метод имеет такую же сигнатуру, как и метод [`whereIn`](#method-wherein); однако, все значения сравниваются с использованием строгого сравнения.

<a name="method-wherenotin"></a>
#### `whereNotIn()` {#collection-method}

Метод `whereNotIn` фильтрует коллекцию по заданным ключу/значению, которые не содержатся в данном массиве:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereNotIn('price', [150, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

Метод `whereNotIn` использует "неточные" сравнения при проверке значений элементов, то есть строка с целым значением будет считаться равной целому числу с тем же значением. Используйте метод [`whereNotInStrict`](#method-wherenotinstrict) для фильтрации с использованием строгих сравнений.

<a name="method-wherenotinstrict"></a>
#### `whereNotInStrict()` {#collection-method}

Этот метод имеет такую же сигнатуру, как и метод [`whereNotIn`](#method-wherenotin); однако, все значения сравниваются с использованием строгого сравнения.

<a name="method-zip"></a>
#### `zip()` {#collection-method}

Метод `zip` объединяет все значения заданного массива со значениями исходной коллекции на соответствующем индексе:

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]

<a name="higher-order-messages"></a>
## Операции высшего порядка

Коллекции также поддерживают [операции высшего порядка](https://en.wikipedia.org/wiki/Higher_order_message), которые служат сокращениями для выполнения обычных действий с коллекциями. Методы коллекций, которые предоставляют операции высшего порядка: `average`, `avg`, `contains`, `each`, `every`, `filter`, `first`, `flatMap`, `map`, `partition`, `reject`, `sortBy`, `sortByDesc` и `sum`.

Доступ к каждой операции высшего порядка можно получить как к динамическому свойству экземпляра коллекции. К примеру, давайте используем операцию `each`, чтобы вызывать метод для каждого объекта в коллекции:

    $users = User::where('votes', '>', 500)->get();

    $users->each->markAsVip();

Таким же образом, мы можем использовать операцию `sum`, чтобы получить общее количество "голосов" пользователей коллекции:

    $users = User::where('group', 'Development')->get();

    return $users->sum->votes;
