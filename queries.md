git 88b1ecd0b36c62f375af33c2f49fdef24cedfc4e

---

# Конструктор запросов

- [Введение](#introduction)
- [Выборка (SELECT)](#selects)
- [Объединения (JOIN)](#joins)
- [Сложные выражения WHERE](#advanced-wheres)
- [Аггрегатные функции](#aggregates)
- [Сырые выражения](#raw-expressions)
- [Вставка (INSERT)](#inserts)
- [Обновление (UPDATE)](#updates)
- [Удаление (DELETE)](#deletes)
- [Слияние (UNION)](#unions)
- [Блокирование (lock) данных](#pessimistic-locking)

<a name="introduction"></a>
## Введение

Query Builder - конструктор запросов - предоставляет удобный, выразительный интерфейс для создания и выполнения запросов к базе данных. Он может использоваться для выполнения большинства типов операций и работает со всеми подерживаемыми СУБД.

> **Примечание:** конструктор запросов Laravel использует средства PDO для защиты вашего приложения от SQL-инъекций. Нет необходимости экранировать строки перед их передачей в запрос.

<a name="selects"></a>
## Выборка (SELECT)

#### Получение всех записей таблицы

	$users = DB::table('users')->get();

	foreach ($users as $user)
	{
		var_dump($user->name);
	}

#### Ограничение результатов выборки

Вы можете отобрать некоторое количество результатов из выборки:

	DB::table('users')->chunk(100, function($users)
	{
		foreach ($users as $user)
		{
			//
		}
	});

Вы можете остановить отбор результатов в чанк, вернув `false` из функции-замыкания.

	DB::table('users')->chunk(100, function($users)
	{
		//

		return false;
	});

#### Получение одной записи

	$user = DB::table('users')->where('name', 'Джон')->first();

	var_dump($user->name);

#### Получение одного поля из записей

	$name = DB::table('users')->where('name', 'Джон')->pluck('name');

#### Получение списка всех значений одного поля

	$roles = DB::table('roles')->lists('title');

Этот метод вернёт массив всех заголовков (title). Вы можете указать произвольный ключ для возвращаемого массива:

	$roles = DB::table('roles')->lists('title', 'name');

#### Указание полей для выборки

	$users = DB::table('users')->select('name', 'email')->get();

	$users = DB::table('users')->distinct()->get();

	$users = DB::table('users')->select('name as user_name')->get();

#### Добавление полей к созданному запросу

	$query = DB::table('users')->select('name');

	$users = $query->addSelect('age')->get();

#### Использование фильтрации WHERE

	$users = DB::table('users')->where('votes', '>', 100)->get();

#### Условия ИЛИ:

	$users = DB::table('users')
	                    ->where('votes', '>', 100)
	                    ->orWhere('name', 'Джон')
	                    ->get();

#### Фильтрация по интервалу значений

	$users = DB::table('users')
	                    ->whereBetween('votes', array(1, 100))->get();

#### Фильтрация по совпадению с массивом значений

	$users = DB::table('users')
	                    ->whereIn('id', array(1, 2, 3))->get();

	$users = DB::table('users')
	                    ->whereNotIn('id', array(1, 2, 3))->get();

#### Поиск неустановленных значений (NULL)

	$users = DB::table('users')
	                    ->whereNull('updated_at')->get();

#### Использование By, Group By и Having

	$users = DB::table('users')
	                    ->orderBy('name', 'desc')
	                    ->groupBy('count')
	                    ->having('count', '>', 100)
	                    ->get();

#### Смещение от начала и лимит числа возвращаемых строк

	$users = DB::table('users')->skip(10)->take(5)->get();

<a name="joins"></a>
## Объединения (JOIN)

Конструктор запросов может быть использован для выборки данных из нескольких таблиц через JOIN. Посмотрите на примеры ниже.

#### Простое объединение

	DB::table('users')
	            ->join('contacts', 'users.id', '=', 'contacts.user_id')
	            ->join('orders', 'users.id', '=', 'orders.user_id')
	            ->select('users.id', 'contacts.phone', 'orders.price');

#### Объединение типа LEFT JOIN

	DB::table('users')
		    ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
		    ->get();

Вы можете указать более сложные условия:

	DB::table('users')
	        ->join('contacts', function($join)
	        {
	        	$join->on('users.id', '=', 'contacts.user_id')->orOn(...);
	        })
	        ->get();

Внутри `join()` можно использовать `where` и `orWhere`:

	DB::table('users')
	        ->join('contacts', function($join)
	        {
	        	$join->on('users.id', '=', 'contacts.user_id')
	        	     ->where('contacts.user_id', '>', 5);
	        })
	        ->get();

<a name="advanced-wheres"></a>
## Сложные выражения WHERE

Иногда вам нужно сделать выборку по более сложным параметрам, таким как "существует ли" или вложенная группировка условий. Конструктор запросов Laravel справится и с такими запросами.

#### Группировка условий

	DB::table('users')
	            ->where('name', '=', 'Джон')
	            ->orWhere(function($query)
	            {
	            	$query->where('votes', '>', 100)
	            	      ->where('title', '<>', 'Админ');
	            })
	            ->get();

Команда выше выполнит такой SQL:

	select * from users where name = 'Джон' or (votes > 100 and title <> 'Админ')

#### Проверка на существование

	DB::table('users')
	            ->whereExists(function($query)
	            {
	            	$query->select(DB::raw(1))
	            	      ->from('orders')
	            	      ->whereRaw('orders.user_id = users.id');
	            })
	            ->get();

Эта команда выше выполнит такой запрос:

	select * from users
	where exists (
		select 1 from orders where orders.user_id = users.id
	)

<a name="aggregates"></a>
## Аггрегатные функции

Конструктор запросов содержит множество аггрегатных методов, таких как `count`, `max`, `min`, `avg` и `sum`.

#### Использование аггрегатных функций

	$users = DB::table('users')->count();

	$price = DB::table('orders')->max('price');

	$price = DB::table('orders')->min('price');

	$price = DB::table('orders')->avg('price');

	$total = DB::table('users')->sum('votes');

<a name="raw-expressions"></a>
## Сырые SQL-выражения

Иногда вам может быть нужно использовать уже готовое SQL-выражение в вашем запросе. Такие выражения вставляются в запрос напрямую в виде строк, поэтому будьте внимательны и не создавайте возможных точек для SQL-инъекций. Для создания сырого выражения используется метод `DB::raw`.

#### Использование SQL-выражения в конструкторе запросов

	$users = DB::table('users')
	                     ->select(DB::raw('count(*) as user_count, status'))
	                     ->where('status', '<>', 1)
	                     ->groupBy('status')
	                     ->get();

<a name="inserts"></a>
## Вставка (INSERT)

#### Вставка записи в таблицу

	DB::table('users')->insert(
		array('email' => 'john@example.com', 'votes' => 0)
	);

#### Вставка записи и получение её нового ID

Если таблица имеет автоинкрементный индекс, то можно использовать метод `insertGetId` для вставки записи и получения её порядкового номера.

	$id = DB::table('users')->insertGetId(
		array('email' => 'john@example.com', 'votes' => 0)
	);

> **Примечание:** при использовании PostgreSQL автоинкрементное поле должно иметь имя "id".

#### Вставка нескольких записей одновременно

	DB::table('users')->insert(array(
		array('email' => 'taylor@example.com', 'votes' => 0),
		array('email' => 'dayle@example.com', 'votes' => 0),
	));

<a name="updates"></a>
## Обновление (UPDATE)

#### Обновление записей в таблице

	DB::table('users')
	            ->where('id', 1)
	            ->update(array('votes' => 1));

#### Увеличение или уменьшение значения поля

	DB::table('users')->increment('votes');

	DB::table('users')->increment('votes', 5);

	DB::table('users')->decrement('votes');

	DB::table('users')->decrement('votes', 5);

Вы также можете указать дополнительные поля для изменения:

	DB::table('users')->increment('votes', 1, array('name' => 'Джон'));	            

<a name="deletes"></a>
## Удаление (DELETE)

#### Удаление записей из таблицы

	DB::table('users')->where('votes', '<', 100)->delete();

#### Удаление всех записей

	DB::table('users')->delete();

#### Очистка таблицы

	DB::table('users')->truncate();

> Очистка таблицы аналогична удалению всех её записей, а также сбросом счётчика автоинкремент-поля - прим. пер.

<a name="unions"></a>
## Слияние (UNION)

Конструктор запросов позволяет создавать слияния двух запросов вместе.

	$first = DB::table('users')->whereNull('first_name');

	$users = DB::table('users')->whereNull('last_name')->union($first)->get();

Также существует метод `unionAll` с аналогичными параметрами.

<a name="pessimistic-locking"></a>
## Блокирование (lock) данных

SELECT с 'shared lock':

	DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

SELECT с 'lock for update':

	DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();

<a name="caching-queries"></a>
## Кэширование запросов

Вы можете легко закэшировать запрос методом `remember`:

#### Кэширование запросов

	$users = DB::table('users')->remember(10)->get();

В этом примере результаты выборки будут сохранены в кэше на 10 минут. В течении этого времени данный запрос не будет отправляться к СУБД - вместо этого результат будет получен из системы кэширования, указанного по умолчанию в вашем файле настроек.

Если система кэширования, которую вы выбрали, поддерживает [тэги](/docs/{{version}}/cache#cache-tags), вы можете их указать в запросе:

	$users = DB::table('users')->cacheTags(array('people', 'authors'))->remember(10)->get();