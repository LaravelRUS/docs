git 9683536c234cb7838450f02bb9c6ca98d4c7e4c7

---

# Коллекции

- [Введение](#introduction)
- [Создание коллекции](#creating-collections)
- [Доступные методы](#available-methods)

<a name="introduction"></a>
## Введение

Класс `Illuminate\Support\Collection` обеспечивает гибкую и удобную обёртку для работы с массивами данных. Например, проверьте следующий код. Мы будем использовать функцию-хэлпер `collect`, для того чтобы создать новый экземпляр класса коллекции из массива, затем выполните функцию `strtoupper` для каждого элемента, а после удалите все пустые элементы:

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    });

Как вы можете видеть, класс `Collection` позволяет строить цепочки вызовов для операций типа map и reduce над заданным массивом данных. В общем, каждый метод `Collection` возвращает совершенно новый экземпляр класса `Collection`.

<a name="creating-collections"></a>
## Создание коллекции

Как было упомянуто выше, при помощи функции-хэлпера `collect` мы создаём новый `Illuminate\Support\Collection` экземпляр класса для данного массива. Таким образом, создать коллекцию становиться очень просто:

    $collection = collect([1, 2, 3]);

По умолчанию, коллекции [Eloquent](/docs/{{version}}/eloquent) модели всегда возвращаются как экземпляры класса `Collection`; Тем не менее, не стесняйтесь использовать класс `Collection` везде, где это удобно для вашего приложения.

<a name="available-methods"></a>
## Доступные методы

В оставшейся части этой документации, мы обсудим каждый метод, доступный в классе `Collection`. Помните, что все эти методы могут быть доступны и свободно манипулировать с основным массивом. Кроме того, почти каждый метод возвращает новый экземпляр `Collection`, что позволяет сохранить первоначальную копию коллекции, когда это необходимо.

Вы можете выбрать любой метод из этой таблицы, чтобы увидеть пример его использования:

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
[avg](#method-avg)
[chunk](#method-chunk)
[collapse](#method-collapse)
[combine](#method-combine)
[contains](#method-contains)
[count](#method-count)
[diff](#method-diff)
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
[isEmpty](#method-isempty)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[map](#method-map)
[max](#method-max)
[merge](#method-merge)
[min](#method-min)
[only](#method-only)
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
[sum](#method-sum)
[take](#method-take)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[union](#method-union)
[unique](#method-unique)
[values](#method-values)
[where](#method-where)
[whereLoose](#method-whereloose)
[whereIn](#method-wherein)
[whereInLoose](#method-whereinloose)
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

Метод `all` просто возвращает базовый массив, представленный коллекции:

    collect([1, 2, 3])->all();

    // [1, 2, 3]

<a name="method-avg"></a>
#### `avg()` {#collection-method}

 Метод `avg` возвращает среднее значение всех элементов в коллекции:

    collect([1, 2, 3, 4, 5])->avg();

    // 3

Если коллекция содержит вложенные массивы или объекты, то вы должны передать ключ использующийся для определения того, какое среднее значение необходимо вычислить:

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->avg('pages');

    // 636

<a name="method-chunk"></a>
#### `chunk()` {#collection-method}

Метод `chunk` разбивает коллекцию на множество мелких коллекций определенного размера:

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->toArray();

    // [[1, 2, 3, 4], [5, 6, 7]]

Этот метод особенно полезен в [views](/docs/{{version}}/views) при работе с системой сетки, такие как [Bootstrap](http://getbootstrap.com/css/#grid). Представьте, что у вас есть коллекция [Eloquent](/docs/{{version}}/eloquent) модели, которую вы хотите отобразить в сетке:

    @foreach ($products->chunk(3) as $chunk)
        <div class="row">
            @foreach ($chunk as $product)
                <div class="col-xs-4">{{ $product->name }}</div>
            @endforeach
        </div>
    @endforeach

<a name="method-collapse"></a>
#### `collapse()` {#collection-method}

Метод `collapse` сворачивает коллекцию массивов в плоскую коллекцию:

    $collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-combine"></a>
#### `combine()` {#collection-method}

Метод `combine` комбинирует ключи из одной коллекции со значениями из другой коллекции или массива:

    $collection = collect(['name', 'age']);

    $combined = $collection->combine(['George', 29]);

    $combined->all();

    // ['name' => 'George', 'age' => 29]

<a name="method-contains"></a>
#### `contains()` {#collection-method}

Метод `contains` определяет, содержит ли коллекция указанное значение:

    $collection = collect(['name' => 'Desk', 'price' => 100]);

    $collection->contains('Desk');

    // true

    $collection->contains('New York');

    // false

Вы также можете передать пару ключ / значение в метод `contains`, который будет определять, существует ли данная пара в коллекции:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->contains('product', 'Bookcase');

    // false

В заключение, вы можете также передать функцию обратного вызова в метод `contains`, чтобы исполнить нужное условие:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->contains(function ($key, $value) {
        return $value > 5;
    });

    // false

<a name="method-count"></a>
#### `count()` {#collection-method}

Метод `count` возвращает общее количество элементов в коллекции:

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-diff"></a>
#### `diff()` {#collection-method}

Метод `diff` сравнивает расхождения коллекции с другой коллекции или с простым PHP `array`:

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]
    
<a name="method-diffkeys"></a>
#### `diffKeys()` {#collection-method}

Метод `diffKeys` сравнивает коллекцию с другой коллекцией или простым PHP `array` по ключам:

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

В методе `each` происходит перебор элементов в коллекции и каждый элемент проходит функцию обратного вызова:

    $collection = $collection->each(function ($item, $key) {
        //
    });

Возвращение `false` из функции обратного вызова, нужно для того чтобы выйти из цикла:

    $collection = $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });

<a name="method-every"></a>
#### `every()` {#collection-method}

Метод `every` создает новую коллекцию, состоящую из каждого n-го элемента:

    $collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

    $collection->every(4);

    // ['a', 'e']

Вы можете указать с какого элемента нужно начать смещение, в качестве второго аргумента:

    $collection->every(4, 1);

    // ['b', 'f']

<a name="method-except"></a>
#### `except()` {#collection-method}

Метод `except` возвращает все элементы коллекции кроме указанных ключей в массиве:

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

    $filtered = $collection->except(['price', 'discount']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

Для обратного порядка `except` смотри метод [only](#method-only).

<a name="method-filter"></a>
#### `filter()` {#collection-method}

Метод `filter` фильтрует коллекцию с помощью полученной функции обратного вызова, сохраняя только те элементы, которые проходят условие:

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->filter(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [3, 4]

Для обратного порядка `filter` смотри метод [reject](#method-reject).

<a name="method-first"></a>
#### `first()` {#collection-method}

 Метод `first` возвращает первый элемент в коллекции, который проходит условие:

    collect([1, 2, 3, 4])->first(function ($key, $value) {
        return $value > 2;
    });

    // 3

Вы также можете вызвать метод `first` без аргументов, для того чтобы получить первый элемент в коллекции. Если коллекция пуста, то возвращается `null`:

    collect([1, 2, 3, 4])->first();

    // 1
    
<a name="method-flatmap"></a>
#### `flatMap()` {#collection-method}

Метод `flatMap` итерируется по коллекции и передает каждое значение в функцию обратного вызова. Эта функция может модифицировать значение и вернуть его для формирования новой коллекции, которая будет одномерной:

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

Метод `flatten` преобразует многомерную коллекцию в одномерный и при этом теряются все ключи:

    $collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

    $flattened = $collection->flatten();

    $flattened->all();

    // ['taylor', 'php', 'javascript'];
    
Если требуется, вы можете передать аргумент "глубины":

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

<a name="method-flip"></a>
#### `flip()` {#collection-method}

Метод `flip` меняет в коллекции ключи и значения местами:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $flipped = $collection->flip();

    $flipped->all();

    // ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>
#### `forget()` {#collection-method}

Метод `forget` удаляет элемент из коллекции по ключу:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $collection->forget('name');

    $collection->all();

    // ['framework' => 'laravel']

> ** Примечание: ** В отличие от большинства других методов коллекции, `forget` не возвращает новую модифицированную коллекцию; он изменяет коллекцию при вызове.

<a name="method-forpage"></a>
#### `forPage()` {#collection-method}

 Метод `forPage` возвращает новую коллекцию, содержащую элементы, которые будут присутствовать на указанном номере страницы:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunk = $collection->forPage(2, 3);

    $chunk->all();

    // [4, 5, 6]

Метод соответственно принимает обязательных 2 параметра - номер страницы и количество элементов для отображения на странице.

<a name="method-get"></a>
#### `get()` {#collection-method}

 Метод `get` возвращает нужный элемент по указанному ключу. Если ключ не существует, то возвращается `null`:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('name');

    // taylor

Вы можете при желании передать значение по умолчанию в качестве второго аргумента:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('foo', 'default-value');

    // default-value

Вы можете даже передать функцию обратного вызова в качестве значения по умолчанию. В результат будет возвращена функция обратного вызова, если указанный ключ не существует:

    $collection->get('email', function () {
        return 'default-value';
    });

    // default-value

<a name="method-groupby"></a>
#### `groupBy()` {#collection-method}

В методе `groupBy` будут сгруппированы элементы коллекции с помощью указанного ключа:

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

В дополнение к передаваемой строке `key`, вы можете также передать функцию обратного вызова. Обратный вызов должен возвращать значение, по которому вы хотите сгруппировать:

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

Метод `has` определяет, существует ли данный ключ в коллекции:

    $collection = collect(['account_id' => 1, 'product' => 'Desk']);

    $collection->has('email');

    // false

<a name="method-implode"></a>
#### `implode()` {#collection-method}

Метод `implode` соединяет элементы в коллекции. Ее аргументы зависят от типа элементов в коллекции.

Если коллекция содержит массивы или объекты, вы должны передать ключ атрибутов, который вы хотите соединить и "glue" строку, которую вы хотите поместить между значениями:

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);

    $collection->implode('product', ', ');

    // Desk, Chair

Если коллекция содержит простые строки или числовые значения, просто передайте "glue" в качестве единственного аргумента метода:

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

<a name="method-intersect"></a>
#### `intersect()` {#collection-method}

Метод `Intersect` удаляет любые значения, которые не присутствуют в указанном `array` или коллекции:

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

Как вы можете видеть, в результате чего, коллекция сохранит оригинальные ключи коллекции.

<a name="method-isempty"></a>
#### `isEmpty()` {#collection-method}

Метод `isEmpty` возвращает `true`, если коллекция пуста; в противном случае, возвращает `false`:

    collect([])->isEmpty();

    // true

<a name="method-keyby"></a>
#### `keyBy()` {#collection-method}

Получить ключи для коллекции с помощью указанного ключа:

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

Если несколько элементов имеют одинаковый ключ, то только последняя появится в новой коллекции.

Вы также можете передать свою собственную функцию обратного вызова, которая должна возвращать значение ключа в коллекции:

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

 Метод `keys` возвращает все ключи в коллекции:

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keys = $collection->keys();

    $keys->all();

    // ['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` {#collection-method}

Метод `last` возвращает последний элемент в коллекции, который проходит условие:

    collect([1, 2, 3, 4])->last(function ($key, $value) {
        return $value < 3;
    });

    // 2

Вы также можете вызвать метод `last` без аргументов, чтобы получить последний элемент в коллекции. Если коллекция пуста, то возвращается `null`:

    collect([1, 2, 3, 4])->last();

    // 4

<a name="method-map"></a>
#### `map()` {#collection-method}

В методе `map` перебирается коллекция и передается функция обратного вызова для каждого указанного значения. Функция обратного вызова может свободно изменять элемент и возвращать его, формируя таким образом новую коллекцию измененных элементов:

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });

    $multiplied->all();

    // [2, 4, 6, 8, 10]

> ** Примечание: ** Как и большинство других методов коллекции, `map` возвращает новый экземпляр коллекции; она не изменяет коллекцию при вызове. Если вы хотите, чтобы преобразовалась оригинальная коллекция, используйте метод [`transform`](#method-transform).

<a name="method-max"></a>
#### `max()` {#collection-method}

 Метод `max` возвращает максимальное значение указанного ключа:

    $max = collect([['foo' => 10], ['foo' => 20]])->max('foo');

    // 20

    $max = collect([1, 2, 3, 4, 5])->max();

    // 5

<a name="method-merge"></a>
#### `merge()` {#collection-method}

Метод `merge` объединяет указанный массив с коллекцией. Любой ключ строки в массиве, при совпадении соответствующему ключу строки в коллекции, будет перезаписывать значение в коллекции:

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $merged = $collection->merge(['price' => 100, 'discount' => false]);

    $merged->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]

Если указанные ключи в массиве числовые, то значения будут добавляться в конец коллекции:

    $collection = collect(['Desk', 'Chair']);

    $merged = $collection->merge(['Bookcase', 'Door']);

    $merged->all();

    // ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-min"></a>
#### `min()` {#collection-method}

 Метод `min` возвращает минимальное значение указанного ключа:

    $min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

    // 10

    $min = collect([1, 2, 3, 4, 5])->min();

    // 1

<a name="method-only"></a>
#### `only()` {#collection-method}

 Метод `only` возвращает элементы коллекции, с указанными ключами:

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

    $filtered = $collection->only(['product_id', 'name']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

Для обратного порядка `only` см метод [except](#method-except).

<a name="method-pluck"></a>
#### `pluck()` {#collection-method}

Метод `Pluck` извлекает все значения коллекции для указанного ключа:

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $plucked = $collection->pluck('name');

    $plucked->all();

    // ['Desk', 'Chair']

Вы также можете указать, с каким ключом вы хотите получить коллекцию:

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

При желании вы можете передать второй аргумент, чтобы установить ключ указывая имя пункта:

    $collection = collect(['one' => 1, 'two', => 2]);

    $collection->prepend(0, 'zero');

    $collection->all();

    // ['zero' => 0, 'one' => 1, 'two', => 2]

<a name="method-pull"></a>
#### `pull()` {#collection-method}

 Метод `pull` удаляет и возвращает элемент из коллекции по ключу:

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

Метод `put` устанавливает указанный ключ и значение в коллекцию:

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {#collection-method}

Метод  `random` возвращает случайный элемент из коллекции:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->random();

    // 4 - (retrieved randomly)

Вы можете при желании передать целое число `random`. Если это число больше, чем '1', то возвращается коллекция с 'n' кол-вом элементов:

    $random = $collection->random(3);

    $random->all();

    // [2, 4, 5] - (retrieved randomly)

<a name="method-reduce"></a>
#### `reduce()` {#collection-method}

Метод `reduce` уменьшает коллекцию до одного значения, передавая результат каждой итерации в последующую итерацию:

    $collection = collect([1, 2, 3]);

    $total = $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    });

    // 6

Значение для `$carry` на первой итерации по умолчанию `null`; Тем не менее, вы можете указать его начальное значение, передав второй аргумент в `reduce`:

    $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    }, 4);

    // 10

<a name="method-reject"></a>
#### `reject()` {#collection-method}

Метод `reject` фильтрует коллекцию, используя указанную функцию обратного вызова. Функция обратного вызова должна возвращать `true` для элементов, которые необходимо удалить из полученной коллекции:

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [1, 2]

Для обратного порядка метода `reject` смотри метод [filter](#method-filter).

<a name="method-reverse"></a>
#### `reverse()` {#collection-method}

Метод `reverse` меняет порядок элементов коллекции:

    $collection = collect([1, 2, 3, 4, 5]);

    $reversed = $collection->reverse();

    $reversed->all();

    // [5, 4, 3, 2, 1]

<a name="method-search"></a>
#### `search()` {#collection-method}

Метод `search` ищет в коллекции указанное значение и если найдёт, то вернёт его ключ. Если элемент не найден, то возвращается `false`.

    $collection = collect([2, 4, 6, 8]);

    $collection->search(4);

    // 1

Поиск осуществляется с помощью "loose" сравнения. Для того, чтобы использовать строгое сравнение, передайте `true` в качестве второго аргумента метода:

    $collection->search('4', true);

    // false

В качестве альтернативы, вы можете передать свою собственную функцию обратного вызова, для поиска первого элемента, который пройдёт условие:

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

Метод `shuffle` случайным образом перемешивает элементы в коллекции:

    $collection = collect([1, 2, 3, 4, 5]);

    $shuffled = $collection->shuffle();

    $shuffled->all();

    // [3, 2, 5, 1, 4] // (generated randomly)

<a name="method-slice"></a>
#### `slice()` {#collection-method}

 Метод `slice` возвращает кусочек коллекции, начиная с указанного индекса:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    // [5, 6, 7, 8, 9, 10]

Если вы хотите ограничить размер полученных значений, передайте нужный размер в качестве второго аргумента метода:

    $slice = $collection->slice(4, 2);

    $slice->all();

    // [5, 6]

Полученные значения в коллекции будут иметь новые, числовые индексированные ключи. Если вы хотите сохранить оригинальные ключи, поставьте `true` в качестве третьего аргумента метода.

<a name="method-sort"></a>
#### `sort()` {#collection-method}

Метод `sort` сортирует коллекцию:

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sort();

    $sorted->values()->all();

    // [1, 2, 3, 4, 5]

Сортируется коллекция, сохраняя оригинальные ключи массива. В этом примере мы использовали метод [`values`](#method-values) , чтобы сбросить ключи и последовательно пронумеровать индексы.

Для сортировки коллекции вложенных массивов или объектов, смотри методы [sortBy](#method-sortby) и [sortByDesc](#method-sortbydesc).

Если вам необходимо отсортировать более продвинуто, вы можете передать функцию обратного вызова в метод `sort` с вашим собственным алгоритмом. Обратитесь к документации по PHP на [`usort`](http://php.net/manual/en/function.usort.php#refsect1-function.usort-parameters), который вызывается в коллекции, внутри метода `sort`.

<a name="method-sortby"></a>
#### `sortBy()` {#collection-method}

Метод `sortBy` сортирует коллекцию с помощью указанного ключа:

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

Сортируется коллекция, сохраняя оригинальные ключи массива. В этом примере мы использовали метод [`values`](#method-values), чтобы сбросить ключи и последовательно пронумеровать индексы.

Вы также можете передать свою собственную функцию обратного вызова, чтобы определить, как сортировать значения коллекции:

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

Этот метод имеет ту же сигнатуру, что и  [`sortBy`](#method-sortby), но будет сортировать коллекцию в обратном порядке.

<a name="method-splice"></a>
#### `splice()` {#collection-method}

Метод `splice` удаляет и возвращает часть элементов, начиная с указанного индекса:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    // [3, 4, 5]

    $collection->all();

    // [1, 2]

Вы можете передать второй аргумент, чтобы ограничить размер получаемых значений:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 4, 5]

Кроме того, вы можете передать третий аргумент, содержащий новые элементы, чтобы заменить элементы, которые будут удалены из коллекции:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 10, 11, 4, 5]

<a name="method-sum"></a>
#### `sum()` {#collection-method}

 Метод `sum` возвращает сумму всех элементов в коллекции:

    collect([1, 2, 3, 4, 5])->sum();

    // 15

Если коллекция содержит вложенные массивы или объекты, то вы должны передать ключ для определения того, какие значения суммировать:

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->sum('pages');

    // 1272

Кроме того, вы можете передать свою собственную функцию обратного вызова, чтобы определить, какие значения коллекции суммировать:

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

 Метод `take` возвращает новую коллекцию с указанным числом элементов:

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

Вы также можете передать отрицательное число, для того, чтобы получить определенное количество элементов из конца коллекции:

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-toarray"></a>
#### `toArray()` {#collection-method}

 Метод `toArray` преобразует коллекцию в обычный PHP `array`. Если значения коллекции являются [Eloquent](/docs/{{version}}/eloquent) моделями, то модели также будут преобразованы в массивы:

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> **Note:** `toArray` also converts all of its nested objects to an array. If you want to get the underlying array as is, use the [`all`](#method-all) method instead.

> ** Примечание: ** `toArray` также конвертирует все вложенные объекты в массив. Если вы хотите получить основной массив, то используйте метод [`all`](#method-all) вместо этого.

<a name="method-tojson"></a>
#### `toJson()` {#collection-method}

Метод `toJson` преобразует коллекцию в формате JSON:

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    // '{"name":"Desk","price":200}'

<a name="method-transform"></a>
#### `transform()` {#collection-method}

В методе `transform` перебирается коллекция и вызывается для  каждого элемента коллекции, указанная функция обратного вызова. Элементы коллекции будут заменены значениями, получаемые от функции обратного вызова:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->transform(function ($item, $key) {
        return $item * 2;
    });

    $collection->all();

    // [2, 4, 6, 8, 10]

> ** Примечание: ** В отличие от большинства других методов коллекции, `transform` изменяет саму коллекцию. Если вы хотите создать новую коллекцию вместо этого использовать метод [`map`](#method-map).


<a name="method-union"></a>
#### `union()` {#collection-method}

Метод `union` добавляет переданный в него массив в коллекцию. Если в ключ элемента из переданного массива совпадает с элементом, который уже находится в коллекции, значение заменено не будет:

    $collection = collect([1 => ['a'], 2 => ['b']]);

    $union = $collection->union([3 => ['c'], 1 => ['b']]);

    $union->all();

    // [1 => ['a'], 2 => ['b'], [3 => ['c']]
    
<a name="method-unique"></a>
#### `unique()` {#collection-method}

 Метод `unique` возвращает все уникальные элементы в коллекции:

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    // [1, 2, 3, 4]

Полученная коллекция сохраняет оригинальные ключи массива. В этом примере мы использовали метод [`values`](#method-values), чтобы сбросить ключи и последовательно пронумеровать индексы.

При работе с вложенными массивами или объектами, вы можете указать ключ, используемый для определения уникальности:

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

Вы также можете передать свою собственную функцию обратного вызова, для того, чтобы определить уникальность элементов:

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

<a name="method-values"></a>
#### `values()` {#collection-method}

The `values` method returns a new collection with the keys reset to consecutive integers:

Метод `values` возвращает новую коллекцию со сбросанными ключами и с последовательно нумерованными числами:

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
<a name="method-where"></a>
#### `where()` {#collection-method}

Метод `where` фильтрует коллекцию с помощью указанной пары - ключ / значение:

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

Метод `where` использует строгое сравнение при проверке значений элементов. Используйте метод [`whereLoose`](#method-whereloose) для фильтрации с использованием "loose" сравнения.

<a name="method-whereloose"></a>
#### `whereLoose()` {#collection-method}

Этот метод имеет ту же сигнатуру, что и метод [`where`](#method-where); Тем не менее, все значения сравниваются с использованием "loose" сравнения.

<a name="method-wherein"></a>
#### `whereIn()` {#collection-method}

Метод `whereIn` фильтрует коллекцию с помощью указанного ключа / значения, содержащегося в данном массиве.

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

Метод `whereIn` использует строгое сравнение при проверке значений элементов. Используйте метод [`whereInLoose`](#method-whereinloose), в котором для фильтрации используется "loose" сравнения.

<a name="method-whereinloose"></a>
#### `whereInLoose()` {#collection-method}

Этот метод имеет ту же сигнатуру, что и метод [`whereIn`](#method-wherein); Тем не менее, все значения сравниваются с использованием "loose" сравнения.

<a name="method-zip"></a>
#### `zip()` {#collection-method}

 Метод `zip` объединяет воедино значения указанного массива со значениями коллекции на соответствующем индексе:

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]
