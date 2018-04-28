git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# База данных: Конструктор запросов 

- [Введение](#introduction)
- [Получение результатов](#retrieving-results)
    - [Разделение результатов на куски](#chunking-results)
    - [Агрегатные функции](#aggregates)
- [Выборка (select)](#selects)
- [Сырые выражения](#raw-expressions)
- [Объединения (join)](#joins)
- [Слияния (union)](#unions)
- [Условия Where](#where-clauses)
    - [Группировка параметров](#parameter-grouping)
    - [Условия Where Exists](#where-exists-clauses)
    - [JSON условия Where](#json-where-clauses)
- [Упорядочивание, группировка, предел и смещение](#ordering-grouping-limit-and-offset)
- [Условные выражения](#conditional-clauses)
- [Вставка (insert)](#inserts)
- [Обновления (update)](#updates)
    - [Обновление JSON-столбцов](#updating-json-columns)
    - [Increment и Decrement](#increment-and-decrement)
- [Удаления (delete)](#deletes)
- [Пессимистическое блокирование](#pessimistic-locking)

<a name="introduction"></a>
## Введение

Конструктор запросов Laravel предоставляет удобный, выразительный интерфейс для создания и выполнения запросов к базе данных. Он может использоваться для выполнения большинства типов операций и работает со всеми поддерживаемыми СУБД

Конструктор запросов Laravel использует привязку параметров к запросам средствами PDO для защиты вашего приложения от SQL-инъекций. Нет необходимости экранировать строки перед их передачей в запрос

<a name="retrieving-results"></a>
## Получение результатов

#### Получение всех записей таблицы

Используйте метод `table` фасада `DB` для создания запроса. Метод `table` возвращает экземпляр конструктора запросов для данной таблицы, позволяя вам "прицепить" к запросу дополнительные условия и в итоге получить результат методом `get`:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show a list of all of the application's users.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->get();

            return view('user.index', ['users' => $users]);
        }
    }

Метод `get` возвращает объект `Illuminate\Support\Collection` c результатами, в котором каждый результат — это экземпляр PHP-класса `StdClass`. Вы можете получить значение каждого столбца, обращаясь к столбцу как к свойству объекта:

    foreach ($users as $user) {
        echo $user->name;
    }

#### Получение одной строки / столбца из таблицы

Если вам необходимо получить только одну строку из таблицы БД, используйте метод `first`. Этот метод вернёт один объект `StdClass`:

    $user = DB::table('users')->where('name', 'John')->first();

    echo $user->name;

Если вам не нужна вся строка, вы можете извлечь одно значение из записи методом `value`. Этот метод напрямую вернёт значение конкретного столбца:

    $email = DB::table('users')->where('name', 'John')->value('email');

#### Получение списка всех значений одного столбца

Если вы хотите получить массив значений одного столбца, используйте метод `pluck`. В этом примере мы получим коллекцию названий ролей:

    $titles = DB::table('roles')->pluck('title');

    foreach ($titles as $title) {
        echo $title;
    }

Вы можете указать произвольный ключ для возвращаемой коллекции:

    $roles = DB::table('roles')->pluck('title', 'name');

    foreach ($roles as $name => $title) {
        echo $title;
    }

<a name="chunking-results"></a>
### Разделение результатов на куски

Если вам необходимо обработать тысячи записей БД, попробуйте использовать метод `chunk`. Этот метод получает небольшой "кусок" результатов за раз и отправляет его в замыкание для обработки. Этот метод очень полезен для написания [Artisan команд](/docs/{{version}}/artisan) , которые обрабатывают тысячи записей. Например, давайте обработаем всю таблицу `users` "кусками" по 100 записей:

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        foreach ($users as $user) {
            //
        }
    });

Вы можете остановить обработку последующих "кусков" вернув `false` из `Closure`:

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        // Process the records...

        return false;
    });

<a name="aggregates"></a>
### Агрегатные функции

Конструктор запросов содержит множество агрегатных методов, таких как `count`, `max`, `min`, `avg` и `sum`. Вы можете вызывать их после создания своего запроса:

    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');

Разумеется, вы можете комбинировать эти методы с другими условиями:

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

<a name="selects"></a>
## Выборка (select)

#### Указание столбцов для выборки

Само собой, не всегда вам необходимо выбрать все столбцы из таблицы БД. Используя метод `select` вы можете указать необходимые столбцы для запроса:

    $users = DB::table('users')->select('name', 'email as user_email')->get();

Метод `distinct` позволяет вернуть только отличающиеся результаты:

    $users = DB::table('users')->distinct()->get();

Если у вас уже есть экземпляр конструктора запросов и вы хотите добавить столбец к существующему набору для выборки, используйте метод `addSelect`:

    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();

<a name="raw-expressions"></a>
## Сырые выражения

Иногда вам может понадобиться использовать уже готовое SQL-выражение в вашем запросе. Такие выражения вставляются в запрос напрямую в виде строк, поэтому будьте внимательны и не допускайте возможностей для SQL-инъекций! Для создания сырого выражения используйте метод `DB::raw`:

    $users = DB::table('users')
                         ->select(DB::raw('count(*) as user_count, status'))
                         ->where('status', '<>', 1)
                         ->groupBy('status')
                         ->get();

<a name="joins"></a>
## Объединения (join)

#### Inner Join

Конструктор запросов может быть использован для объединения данных из нескольких таблиц через объединение. Для выполнения обычного объединения "inner join", используйте метод `join` на экземпляре конструктора запросов. Первый аргумент метода `join` — имя таблицы, к которой необходимо присоединить другие, а остальные аргументы указывают условия для присоединения столбцов. Как видите, вы можете объединять несколько таблиц одним запросом:

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();

#### Left Join

Для выполнения объединения "left join" вместо "inner join", используйте метод `leftJoin`. Этот метод имеет ту же сигнатуру, что и метод `join`:

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

#### Cross Join

Для выполнения объединения "cross join", используйте метод `crossJoin` с именем таблицы, с которой нужно произвести объединение. Этот метод формирует таблицу перекрестным соединением (декартовым произведением) двух таблиц:

    $users = DB::table('sizes')
                ->crossJoin('colours')
                ->get();

#### Сложные условия объединения

Вы можете указать более сложные условия для объединения. Для начала передайте замыкание вторым аргументом метода `join`. `Closure` будет получать объект `JoinClause`, позволяя вам указать условия для объединения:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();

Если вы хотите использовать стиль "where" для ваших объединений, то можете использовать для этого методы `where` и `orWhere`. Вместо сравнения двух столбцов эти методы будут сравнивать столбец и значение:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();

<a name="unions"></a>
## Слияния (union)

Конструктор запросов позволяет создавать слияния двух запросов вместе. Например, вы можете создать начальный запрос и с помощью метода `union` слить его со вторым запросом:

    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

> {tip} Также существует метод `unionAll` с аналогичными параметрами.

<a name="where-clauses"></a>
## Условия Where

#### Простые условия Where

Для добавления в запрос условий `where` используйте метод `where` на экземпляре конструктора запросов. Самый простой вызов `where` требует три аргумента. Первый — имя столбца. Второй — оператор (любой из поддерживаемых базой данных). Третий — значение для сравнения со столбцом.

Например, вот запрос, проверяющий равенство значения столбца "votes" и 100:

    $users = DB::table('users')->where('votes', '=', 100)->get();

Для удобства, если вам необходимо просто проверить равенство значения столбца и данного значения, вы можете передать значение сразу вторым аргументом метода `where`:

    $users = DB::table('users')->where('votes', 100)->get();

Разумеется, вы можете использовать различные другие операторы при написании условия `where`:

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();

В функцию `where` также можно передать массив условий:

    $users = DB::table('users')->where([
        ['status', '=', '1'],
        ['subscribed', '<>', '1'],
    ])->get();

#### Условия Or

Вы можете сцепить вместе условия where, а также добавить условия `or` к запросу. Метод `orWhere` принимает те же аргументы, что и метод `where`:

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();

#### Дополнительные условия Where

**whereBetween**

Метод `whereBetween` проверяет, что значение столбца находится в указанном интервале:

    $users = DB::table('users')
                        ->whereBetween('votes', [1, 100])->get();

**whereNotBetween**

Метод `whereNotBetween` проверяет, что значение столбца находится вне указанного интервала:

    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();

**whereIn / whereNotIn**

Метод `whereIn` проверяет, что значение столбца содержится в данном массиве:

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();

Метод `whereNotIn` проверяет, что значение столбца **не** содержится в данном массиве:

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();

**whereNull / whereNotNull**

Метод `whereNull` проверяет, что значение столбца равно `NULL`:

    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();

Метод `whereNotNull` проверяет, что значение столбца не равно `NULL`:

    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();

**whereDate / whereMonth / whereDay / whereYear**

Метод `whereDate` служит для сравнения значения столбца с датой:

    $users = DB::table('users')
                    ->whereDate('created_at', '2016-12-31')
                    ->get();

Метод `whereMonth` служит для сравнения значения столбца с месяцем в году:

    $users = DB::table('users')
                    ->whereMonth('created_at', '12')
                    ->get();

Метод `whereDay` служит для сравнения значения столбца с днём месяца:

    $users = DB::table('users')
                    ->whereDay('created_at', '31')
                    ->get();

Метод `whereYear` служит для сравнения значения столбца с указанным годом:

    $users = DB::table('users')
                    ->whereYear('created_at', '2016')
                    ->get();

**whereColumn**

Для проверки на совпадение двух столбцов можно использовать метод `whereColumn`:

    $users = DB::table('users')
                    ->whereColumn('first_name', 'last_name')
                    ->get();

В метод также можно передать оператор сравнения:

    $users = DB::table('users')
                    ->whereColumn('updated_at', '>', 'created_at')
                    ->get();

В метод `whereColumn` также можно передать массив с несколькими условиями. Эти условия будут объединены оператором `and`:

    $users = DB::table('users')
                    ->whereColumn([
                        ['first_name', '=', 'last_name'],
                        ['updated_at', '>', 'created_at']
                    ])->get();

<a name="parameter-grouping"></a>
### Группировка параметров

Иногда вам нужно сделать выборку по более сложным параметрам, таким как "where exists" или вложенная группировка условий. Конструктор запросов Laravel справится и с такими запросами. Для начала посмотрим на пример группировки условий в скобках:

    DB::table('users')
                ->where('name', '=', 'John')
                ->orWhere(function ($query) {
                    $query->where('votes', '>', 100)
                          ->where('title', '<>', 'Admin');
                })
                ->get();

Как видите, передав `Closure` в метод `orWhere`, мы дали конструктору запросов команду, начать группировку условий. Замыкание получит экземпляр конструктора запросов, который вы можете использовать для задания условий, поместив их в скобки. Приведённый пример выполнит такой SQL-запрос:

    select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

<a name="where-exists-clauses"></a>
### Условия Where Exists

Метод `whereExists` позволяет написать SQL-условия `where exists`. Метод `whereExists` принимает в качестве аргумента замыкание, которое получит экземпляр конструктора запросов, позволяя вам определить запрос для помещения в условие "exists":

    DB::table('users')
                ->whereExists(function ($query) {
                    $query->select(DB::raw(1))
                          ->from('orders')
                          ->whereRaw('orders.user_id = users.id');
                })
                ->get();

Этот пример выполнит такой SQL-запрос:

    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )

<a name="json-where-clauses"></a>
### JSON условия Where

Laravel также поддерживает запросы для столбцов типа JSON в тех БД, которые поддерживают тип столбцов JSON. На данный момент это MySQL 5.7 и Postgres. Для запроса JSON столбца используйте оператор `->`:

    $users = DB::table('users')
                    ->where('options->language', 'en')
                    ->get();

    $users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();

<a name="ordering-grouping-limit-and-offset"></a>
## Упорядочивание, группировка, предел и смещение

#### orderBy

Метод `orderBy` позволяет вам отсортировать результат запроса по заданному столбцу. Первый аргумент метода `orderBy` — столбец для сортировки по нему, а второй — задаёт направление сортировки и может быть либо `asc`, либо `desc`:

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

#### latest / oldest

Методы `latest` и `oldest` позволяют легко отсортировать результаты по дате. По умолчанию выполняется сортировка по столбцу `created_at`. Или вы можете передать имя столбца для сортировки по нему:

    $user = DB::table('users')
                    ->latest()
                    ->first();

#### inRandomOrder

Для сортировки результатов запроса в случайном порядке можно использовать метод `inRandomOrder`. Например, вы можете использовать этот метод для выбора случайного пользователя:

    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();

#### groupBy / having / havingRaw

Методы `groupBy` и `having` используются для группировки результатов запроса. Сигнатура метода `having` аналогична методу `where`:

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();

Метод `havingRaw` используется для передачи сырой строки в условие `having`. Например, мы можем найти все филиалы с объёмом продаж выше $2,500:

    $users = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > 2500')
                    ->get();

#### skip / take

Для ограничения числа возвращаемых результатов из запроса или для пропуска заданного числа результатов в запросе используются методы `skip` и `take`:

    $users = DB::table('users')->skip(10)->take(5)->get();

Или вы можете использовать методы `limit` и `offset`:

    $users = DB::table('users')
                    ->offset(10)
                    ->limit(5)
                    ->get();

<a name="conditional-clauses"></a>
## Условные выражения

Иногда необходимо применять условие к запросу, только если выполняется какое-то другое условие. Например, выполнять оператор `where`, только если нужное значение есть во входящем запросе. Это можно сделать с помощью метода `when`:

    $role = $request->input('role');

    $users = DB::table('users')
                    ->when($role, function ($query) use ($role) {
                        return $query->where('role_id', $role);
                    })
                    ->get();


Метод `when` выполняет данное замыкание, только когда первый параметр равен `true`. Если первый параметр равен `false`, то замыкание не будет выполнено.

Вы можете передать ещё одно замыкание третьим параметром метода `when`. Это замыкание будет выполнено, если первый параметр будет иметь значение `false`. Для демонстрации работы этой функции мы используем её для настройки сортировки по умолчанию для запроса:

    $sortBy = null;

    $users = DB::table('users')
                    ->when($sortBy, function ($query) use ($sortBy) {
                        return $query->orderBy($sortBy);
                    }, function ($query) {
                        return $query->orderBy('name');
                    })
                    ->get();


<a name="inserts"></a>
## Вставка (insert)

Конструктор запросов предоставляет метод `insert` для вставки записей в таблицу БД. Метод `insert` принимает массив имён столбцов и значений:

    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );

Вы можете вставить в таблицу сразу несколько записей одним вызовом `insert`, передав ему массив массивов, каждый из которых — строка для вставки в таблицу:

    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);

#### Автоинкрементные ID

Если в таблице есть автоинкрементный ID, используйте метод `insertGetId` для вставки записи и получения её ID:

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

> {note} При использовании метода insertGetId для PostgreSQL автоинкрементное поле должно иметь имя `id`. Если вы хотите получить ID из другого поля таблицы, вы можете передать его имя вторым аргументом.

<a name="updates"></a>
## Обновления (update)

Разумеется, кроме вставки записей в БД конструктор запросов может и изменять существующие строки с помощью метода `update`. Метод `update`, как и метод `insert` , принимает массив столбцов и пар значений, содержащих столбцы для обновления. Вы можете ограничить запрос `update` условиями `where` :

    DB::table('users')
                ->where('id', 1)
                ->update(['votes' => 1]);

<a name="updating-json-columns"></a>
### Обновление JSON-столбцов

При обновлении JSON-столбцов используйте синтаксис `->` для обращения к нужному ключу в JSON-объекте. Эта операция поддерживается только в БД, поддерживающих JSON-столбцы:

    DB::table('users')
                ->where('id', 1)
                ->update(['options->enabled' => true]);

<a name="increment-and-decrement"></a>
### Increment и Decrement

Конструктор запросов предоставляет удобные методы для увеличения и уменьшения значений заданных столбцов. Это просто более выразительный и краткий способ по сравнению с написанием оператора `update` вручную.

Оба метода принимают один обязательный аргумент — столбец для изменения. Второй аргумент может быть передан для указания, на какую величину необходимо изменить значение столбца:

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

Вы также можете указать дополнительные поля для изменения:

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

<a name="deletes"></a>
## Удаления (delete)

Конструктор запросов предоставляет метод `delete` для удаления записей из таблиц. Вы можете ограничить оператор `delete`, добавив условия `where` перед его вызовом

    DB::table('users')->delete();

    DB::table('users')->where('votes', '>', 100)->delete();

Если вы хотите очистить таблицу (усечение), удалив все строки и обнулив счётчик ID, используйте метод `truncate`:

    DB::table('users')->truncate();

<a name="pessimistic-locking"></a>
## Пессимистическое блокирование

В конструкторе запросов есть несколько функций, которые помогают делать "пессимистическое блокирование" для ваших операторов SELECT. Для запуска оператора SELECT с "разделяемой блокировкой" вы можете использовать в запросе метод `sharedLock`. Разделяемая блокировка предотвращает изменение выбранных строк до конца транзакции:

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

Или вы можете использовать метод `lockForUpdate`. Блокировка "для изменения" предотвращает изменение строк и их выбор другими разделяемыми блокировками:

    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
