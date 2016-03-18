git 494c1c8efe455d80d8b4c49c5b2bf5bf3832ca77

---

# База данных: Конструктор Запросов

- [Введение](#introduction)
- [Получение результатов](#retrieving-results)
    - [Групповые функции (aggregates)](#aggregates)
- [Выборка данных (Selects)](#selects)
- [Объединение таблиц (Joins)](#joins)
- [Объединение результата запросов (Unions)](#unions)
- [Условия отбора (Where Clauses)](#where-clauses)
    - [Расширенные условия отбора (Advanced Where Clauses)](#advanced-where-clauses)
    - [Условия отбора для полей типа JSON (JSON Where Clauses)](#json-where-clauses)
- [Сортировка, Группировка, Лимит, Отступ (Ordering, Grouping, Limit, & Offset)](#ordering-grouping-limit-and-offset)
- [Вставка (Inserts)](#inserts)
- [Обновление (Updates)](#updates)
- [Удаление (Deletes)](#deletes)
- [Пессимистичная блокировка (Pessimistic Locking)](#pessimistic-locking)

<a name="introduction"></a>
## Введение

Конструктор запросов Query Builder предоставляет удобный, выразительный интерфейс для создания и выполнения запросов к базе данных. Он может использоваться для выполнения большинства типов операций и работает со всеми поддерживаемыми СУБД.

> **Примечание:** конструктор запросов Laravel использует средства PDO для защиты вашего приложения от SQL-инъекций. Нет необходимости экранировать строки перед их передачей в запрос.

<a name="retrieving-results"></a>
## Получение результатов

#### Получение всех строк из таблицы

Для выполнения запроса используйте метод `table` фасада `DB`. Метод возвращает объект конструктора запросов для заданной таблицы, который позволяет строить цепочки для добавления условий выборки и в итоге получить результат. В приведенном примере мы просто получаем все строки из таблицы:

    <?php

    namespace App\Http\Controllers;

    use DB;
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

Подобно [запросам на чистом SQL (raw queries)](/docs/{{version}}/database) метод `get` возвращает массив результатов, где каждый результат - это объект PHP `StdClass`. Вы можете получить значения каждого поля через свойства объекта:

    foreach ($users as $user) {
        echo $user->name;
    }

#### Получение одной строки/колонки из таблицы

Если вы хотите получить одну строку из таблицы, то используйте метод `first`. Метод вернет один объект типа `StdClass`:

    $user = DB::table('users')->where('name', 'John')->first();

    echo $user->name;

Если из этой одной строки требуется только одна колонка, то используйте метод `value`. Метод вернет значение этого поля:

    $email = DB::table('users')->where('name', 'John')->value('email');

#### Разбиение результатов на части (Chunking Results From A Table)

Если требуется работать с тысячами строк, то обратите внимание на метод `chunk`. Метод получает данные маленькими "частями" ("chunk") и обрабатывает их в функции-замыкании. Метод очень полезен для записи [Artisan commands](/docs/{{version}}/artisan) тысяч строк. Например, давайте работать со всей таблицей `users` частями по 100 записей:

    DB::table('users')->chunk(100, function($users) {
        foreach ($users as $user) {
            //
        }
    });

Остановить отбор результатов можно, возвратив `false` из функции-замыкания:

    DB::table('users')->chunk(100, function($users) {
        // Process the records...

        return false;
    });

#### Получение значений столбца

Если нужно получить массив значений одного столбца, то используйте метод `pluck`. Например, получим из таблицы `roles` массив значений столбца `title`:

    $titles = DB::table('roles')->pluck('title');

    foreach ($titles as $title) {
        echo $title;
    }

Также можно задать столбец, значения которого будут ключами для возвращаемого массива:

    $roles = DB::table('roles')->pluck('title', 'name');

    foreach ($roles as $name => $title) {
        echo $title;
    }

<a name="aggregates"></a>
### Групповые функции (aggregates)

Конструктор предоставляет разнообразные групповые методы, такие как `count`, `max`, `min`, `avg`, и `sum`. Вы можете вызвать любой из методов после построения запроса:

    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');

Групповые методы можно комбинировать с другими выражениями:

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

<a name="selects"></a>
## Выборка данных (Selects)

#### Уточнение результата выборки (Specifying A Select Clause)

Конечно, вам не всегда требуется выбирать все столбцы из таблицы. Используя метод `select`, вы можете уточнить выборку:

    $users = DB::table('users')->select('name', 'email as user_email')->get();

Метод `distinct` указывает запросу возвратить только уникальные значения:

    $users = DB::table('users')->distinct()->get();

Если уже есть экземпляр конструктора и нужно добавить колонку к существующему выражению выборки, то можно использовать метод `addSelect`:

    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();

#### Использование SQL (Raw Expressions)

Иногда может потребоваться использовать SQL в запросе. Такие выражения будут внедрены в запрос как строки, при этом будьте уверены - они не создадут возможности для SQL инъекций. Для этой цели существует метод `DB::raw`:

    $users = DB::table('users')
                         ->select(DB::raw('count(*) as user_count, status'))
                         ->where('status', '<>', 1)
                         ->groupBy('status')
                         ->get();

<a name="joins"></a>
## Объединение таблиц (Joins)

#### Внутреннее объединение (Inner Join Statement)

Конструктор запросов умеет делать объединение таблиц. Для выполнения простого SQL "inner join" используйте метод `join`. Первый аргумент метода должен быть именем присоединяемой таблицы, остальные задают столбцы для объединения. Как видно из примера можно одновременно объединять несколько таблиц:

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();

#### Левосторонее объединение (Left Join Statement)

Для левостороннего объединения предназначен метод `leftJoin`. Его параметры аналогичны методу `join`:

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

#### Расширенные объединения (Advanced Join Statements)

Вы также можете определять более продвинутые условия объединения. Для начала в качестве второго аргумента задайте функцию-замыкание. Замыкание получит объект `JoinClause`, который позволяет определять условия объединения:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();

Если вы хотите использовать выражение `where`, то для вас доступны методы `where` и `orWhere`. Вместо сравнения двух столбцов эти методы будут сравнивать столбец с указанным значением:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();

<a name="unions"></a>
## Объединение результата запросов (Unions)

Конструктор предоставляет быстрый способ объединения двух запросов. Например, можно создать первый запрос и затем использовать метод `union` для объединения со вторым запросом:

    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

Также доступен метод `unionAll` с такой же сигнатурой как у `union`.

<a name="where-clauses"></a>
## Условия отбора (Where Clauses)

#### Простые условия (Simple Where Clauses)

Для использования конструкции `where` в запросе используйте метод `where`. Обычный вызов метода требует три аргумента. Первый - имя столбца. Второй - любой поддерживаемый СУБД оператор. Третий - значение для сравнения. Например, запрос, проверяющий, что значение столбца 'votes' равно 100:

    $users = DB::table('users')->where('votes', '=', 100)->get();

Для убодства, если требуется простое сравнение на равенство, можно опустить второй аргумент:

    $users = DB::table('users')->where('votes', 100)->get();

Конечно, вы можете использовать другие операторы:

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();

В методе даже можно задавать массив условий:

    $users = DB::table('users')->where([
        ['status','1'],
        ['subscribed','<>','1'],
    ])->get();

#### Выражения ИЛИ (Or Statements)

Вы можете присоединять условия 'where', также как и добавлять условие ИЛИ (`or`) в запрос. Метод `orWhere` принимает аргументы как и метод `where`:

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();

#### Дополнительные условия отбора (Additional Where Clauses)

**whereBetween**

Метод проверяет, что значение поля находится между двух указанных значений

    $users = DB::table('users')
                        ->whereBetween('votes', [1, 100])->get();

**whereNotBetween**

Метод противоположен методу `whereBetween`:

    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();

**whereIn / whereNotIn**

Значение должно находиться в указанном массиве:

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();

Значение не должно находиться в массиве:

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();

**whereNull / whereNotNull**

Значение поля должно быть `NULL`:

    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();

Значение не должно быть `NULL`:

    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();

<a name="advanced-where-clauses"></a>
## Расширенные условия отбора (Advanced Where Clauses)

#### Группировка (Parameter Grouping)

Иногда требуется создать более продвинутые условия отбора, подобно "where exists" или вложенной группировке (nested parameter groupings). Конструктор запросов Laravel может это делать. Для начала взгляните на пример группировки условий внутри круглых скобок:

    DB::table('users')
                ->where('name', '=', 'John')
                ->orWhere(function ($query) {
                    $query->where('votes', '>', 100)
                          ->where('title', '<>', 'Admin');
                })
                ->get();

Как видите замыкание в методе `orWhere` указывает конструктору начать группу условий. Замыкание получит объект запроса, который вы можете использовать для установки ограничений внутри круглых скобок. Пример выше создаст следующий SQL запрос:

    select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

#### Выражения Exists (Exists Statements)

Метод `whereExists` позволяет писать в запрос SQL выражение `where exist`. Этот метод принимает замыкание в качестве аргумента, которому будет передан объект запроса, позволяющий определить запрос для выражения "exists":

    DB::table('users')
                ->whereExists(function ($query) {
                    $query->select(DB::raw(1))
                          ->from('orders')
                          ->whereRaw('orders.user_id = users.id');
                })
                ->get();

Пример выше сформирует следующий запрос SQL:

    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )

<a name="json-where-clauses"></a>
## Условия отбора для полей типа JSON (JSON Where Clauses)

Laravel поддерживает запросы к полям с типом JSON для СУБД с поддержкой этого типа. На текущий момент это MySQL 5.7 and Postgres. Для запроса к такому полю используйте оператор `->`:

    $users = DB::table('users')
                    ->where('options->language', 'en')
                    ->get();

    $users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();

<a name="ordering-grouping-limit-and-offset"></a>
## Сортировка, Группировка, Лимит, Отступ (Ordering, Grouping, Limit, & Offset)

#### Сортировка (orderBy)

Метод `orderBy` позволяет сортировать результаты запроса по указанной колонке. Первый аргумент метода - колонка, по которой выполняется сортировка. Второй - направление сортировки `asc` or `desc`: 

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

#### Группировка (groupBy / having / havingRaw)

Методы `groupBy` и `having` служат для группировки результатов запроса. Сигнатура метода `having` аналогична `where`:

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();

Метод `havingRaw` нужен для установки значения выражения `having` в виде SQL строки. Например, найдем все отделы, где продажи превысили $2500:

    $users = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > 2500')
                    ->get();

#### Отступ/лимит (skip / take)

Для ограничение числа возвращаемых значений запросом либо для пропуска первых результатов в запросе (`OFFSET`) используйте методы `skip` and `take`:

    $users = DB::table('users')->skip(10)->take(5)->get();

<a name="inserts"></a>
## Вставка (Inserts)

Конструктор также обеспечивает метод `insert` для вставки записей в таблицу. Метод принимает массив пар колонка=>значение:

    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );

Для вставки более одной строки используется массив массивов:

    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);

#### Получение id вставленного столбца (Auto-Incrementing IDs)

Если таблица имеет автоинкрементируемый столбец, то для получения значения этого столбца для вставленной записи используется метод `insertGetId`:

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

> **Примечание:** Когда используется PostgreSQL метод ожидает, что такой столбей называется `id`. Если требуется получать значение из другого "sequence", то его имя нужно указывать во втором аргументе метода `insertGetId`

<a name="updates"></a>
## Обновление (Updates)

Конструктор обеспечивает поддержку обновления записей через метод `update`. Метод аналогично методу вставки принимает массив пар столбец=>значение. Для ограничения обновляемых строк используйте метод `where`:

    DB::table('users')
                ->where('id', 1)
                ->update(['votes' => 1]);

#### Инкремент/Декремент (Increment / Decrement)

Методы `increment` / `decrement` позволяют увеличивать/уменьшать значение стобца. Методы являются сокращениями для метода `update`. Оба метода принимают как минимум один аргумент: имя столбца. Второй аргумент необязателен и служит для указания значения, на которое требуется изменить значение столбца.

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

Также возможно определить дополнительные столбцы для обновления во время выполнения операции:

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

<a name="deletes"></a>
## Удаление (Deletes)

Удаление всех строк из таблицы:

    DB::table('users')->delete();

Удаление строк, соответствующих условию:

    DB::table('users')->where('votes', '<', 100)->delete();

Удаление всех строк и сброс автоинкремента (очистка таблицы):

    DB::table('users')->truncate();

<a name="pessimistic-locking"></a>
## Пессимистичная блокировка (Pessimistic Locking)

Для использования блокировки "shared lock" предназначен метод `sharedLock`. Метод предохраняет выбранные строки от изменения до завершения вашей транзакции:

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

Другой вариант использовать метод `lockForUpdate`. Блокировка "for update" предохраняет выбранные строки от изменения и от установки блокировки "shared lock":

    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
