git 043025da5afcab0f811be66a0cb3ceabf86ea1aa

---

# Конструктор таблиц

- [Введение](#introduction)
- [Создание и удаление таблиц](#creating-and-dropping-tables)
- [Добавление полей](#adding-columns)
- [Изменение полей](#changing-columns)
- [Переименование полей](#renaming-columns)
- [Удаление полей](#dropping-columns)
- [Проверка на существование](#checking-existence)
- [Добавление индексов](#adding-indexes)
- [Внешние ключи](#foreign-keys)
- [Удаление индексов](#dropping-indexes)
- [Удаление столбцов дат создания и псевдоудаления](#dropping-timestamps)
- [Storage Engines](#storage-engines)

<a name="introduction"></a>
## Введение

Класс `Schema` представляет собой независимый от типа БД интерфейс манипулирования таблицами. Он хорошо работает со всеми БД, поддерживаемыми Laravel и предоставляет унифицированный API для любой из этих систем.

<a name="creating-and-dropping-tables"></a>
## Создание и удаление таблиц

Для создания новой таблицы используется метод `Schema::create`:

	Schema::create('users', function(Blueprint $table)
	{
		$table->increments('id');
	});

Первый параметр метода `create` - имя таблицы, а второй - функция-замыкание, которое получает объект `Blueprint`, использующийся для определения таблицы.

Чтобы переименовать существующую таблицу используется метод `rename`:

	Schema::rename($from, $to);

Для указания иного использующегося подключения к БД используется метод `Schema::connection`:

	Schema::connection('foo')->create('users', function(Blueprint $table)
	{
		$table->increments('id');
	});

Для удаления таблицы вы можете использовать метод `Schema::drop`:

	Schema::drop('users');

	Schema::dropIfExists('users');

<a name="adding-columns"></a>
## Добавление полей

Для изменения существующей таблицы используется метод `Schema::table`:

	Schema::table('users', function(Blueprint $table)
	{
		$table->string('email');
	});

Конструктор таблиц поддерживает различные типы полей:

Команда  | Описание
------------- | -------------
`$table->bigIncrements('id');`  |  Первичный последовательный (autoincrement) ключ типа BIGINT
`$table->bigInteger('votes');`  |  Поле BIGINT
`$table->binary('data');`  |  Поле BLOB
`$table->boolean('confirmed');`  |  Поле BOOLEAN
`$table->char('name', 4);`  |  Поле CHAR с указанием длины
`$table->date('created_at');`  |  Поле DATE
`$table->dateTime('created_at');`  |  Поле DATETIME
`$table->decimal('amount', 5, 2);`  |  Поле DECIMAL с параметрами "точность" (общее количество значащих десятичных знаков) и "масштаб" (количество десятичных знаков после запятой)
`$table->double('column', 15, 8);`  |  Поле DOUBLE (параметры аналогичны полю DECIMAL)
`$table->enum('choices', array('foo', 'bar'));` | Поле ENUM
`$table->float('amount');`  |  Поле FLOAT
`$table->increments('id');`  |  Первичный последовательный (autoincrement) ключ 
`$table->integer('votes');`  |  Поле INTEGER
`$table->json('options');`  |  Текстовое поле для хранения JSON-данных
`$table->longText('description');`  |  Поле LONGTEXT
`$table->mediumInteger('numbers');`  |  Поле MEDIUMINT
`$table->mediumText('description');`  |  Поле MEDIUMTEXT
`$table->morphs('taggable');`  |  Создается два поля - INTEGER `taggable_id` и STRING `taggable_type`
`$table->nullableTimestamps();`  |  То же, что и `timestamps()`, но разрешены NULL
`$table->smallInteger('votes');`  |  Поле SMALLINT
`$table->tinyInteger('numbers');`  |  Поле TINYINT
`$table->softDeletes();`  |  Столбец **deleted\_at** для реализации псевдоудаления
`$table->string('email');`  |  Поле VARCHAR
`$table->string('name', 100);`  |  Поле VARCHAR с заданной длиной
`$table->text('description');`  |  Поле TEXT
`$table->time('sunrise');`  |  Поле TIME
`$table->timestamp('added_on');`  |  Поле TIMESTAMP
`$table->timestamps();`  |  Столбцы **created\_at** и **updated\_at**
`$table->rememberToken();`  |  Столбец `remember_token` VARCHAR(100) NULL у таблицы users для реализации функции "запомнить меня" при логине 
`->nullable()`  |  данное поле может быть NULL
`->default($value)`  |  установка дефолтного значения поля
`->unsigned()`  |  запрещаются отрицательные значения

#### Вставка поля после существующего (в MySQL)

Если вы используете MySQL, то при изменении таблицы вы можете указать, куда именно добавлять новое поле 

	$table->string('name')->after('email');

<a name="changing-columns"></a>
## Изменение полей

Иногда возникает необходимость изменить существующее поле. К примеру, увеличить длину строкового поля. В этом поможет метод `change`. Давайте увеличим длину поля `name` до 50 символов:

	Schema::table('users', function($table)
	{
		$table->string('name', 50)->change();
	});

Так же можно указать, может ли поле быть NULL:

	Schema::table('users', function($table)
	{
		$table->string('name', 50)->nullable()->change();
	});
	
<a name="renaming-columns"></a>
## Переименование полей

Для переименования поля можно использовать метод `renameColumn`. Переименование возможно только при подключенном пакете `doctrine/dbal` в `composer.json`.

	Schema::table('users', function(Blueprint $table)
	{
		$table->renameColumn('from', 'to');
	});

> **Примечание:** переименование полей типа `enum` не поддерживается.

<a name="dropping-columns"></a>
## Удаление полей

Для удаления поля можно использовать метод `dropColumn`. Удаление возможно только при подключенном пакете `doctrine/dbal` в `composer.json`.

#### Удаление одного поля из таблицы:

	Schema::table('users', function(Blueprint $table)
	{
		$table->dropColumn('votes');
	});

#### Удаление сразу нескольких полей

	Schema::table('users', function(Blueprint $table)
	{
		$table->dropColumn(array('votes', 'avatar', 'location'));
	});

<a name="checking-existence"></a>
## Проверка на существование

#### Проверка существования таблицы

Вы можете легко проверить существование таблицы или поля с помощью методов `hasTable` и `hasColumn`.

	if (Schema::hasTable('users'))
	{
		//
	}

#### Проверка существования поля

	if (Schema::hasColumn('users', 'email'))
	{
		//
	}

<a name="adding-indexes"></a>
## Добавление индексов

Есть два способа добавлять индексы: вместе с определением полей, либо отдельно:

	$table->string('email')->unique();

	$table->unique('email');

Ниже список всех доступных типов индексов.

Команда  | Описание
------------- | -------------
`$table->primary('id');`  |  Добавляет первичный ключ
`$table->primary(array('first', 'last'));`  |  Добавляет составной первичный ключ
`$table->unique('email');`  |  Добавляет уникальный индекс
`$table->index('state');`  |  Добавляет простой индекс

<a name="foreign-keys"></a>
## Внешние ключи

Laravel поддерживает добавление внешних ключей (foreign key constraints) для таблиц:

	$table->integer('user_id')->unsigned();
	$table->foreign('user_id')->references('id')->on('users');

В этом примере мы указываем, что поле `user_id` связано с полем `id` таблицы `users`. Обратите внимание, что поле `user_id` должно существовать перед определением ключа.

Вы также можете задать действия, происходящие при обновлении (on update) и добавлении (on delete) записей. 

	$table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

Для удаления внешнего ключа используется метод `dropForeign`. Схема именования ключей - та же, что и индексов:

	$table->dropForeign('posts_user_id_foreign');

> **Внимание:** при создании внешнего ключа, указывающего на автоинкрементное (автоувеличивающееся) числовое поле, не забудьте сделать указывающее поле (поле внешнего ключа) типа `unsigned`.

<a name="dropping-indexes"></a>
## Удаление индексов

Для удаления индекса вы должны указать его имя. По умолчанию Laravel присваивает каждому индексу осознанное имя. Просто объедините имя таблицы, имена всех его полей и добавьте тип индекса. Вот несколько примеров:

Command  | Description
------------- | -------------
`$table->dropPrimary('users_id_primary');`  |  Удаление первичного ключа из таблицы "users"
`$table->dropUnique('users_email_unique');`  |  Удаление уникального индекса на полях "email" и "password" из таблицы "users"
`$table->dropIndex('geo_state_index');`  |  Удаление простого индекса из таблицы "geo"

<a name="dropping-timestamps"></a>
## Удаление столбцов дат создания и псевдоудаления

Для того, чтобы удалить столбцы **created\_at** , **updated\_at** и **deleted\_at**, которые создаются методами `timestamps` (или `nullableTimestamps`) и `softDeletes`, вы можете использовать следующие методы:

Команда  | Описания
------------- | -------------
`$table->dropTimestamps();`  |  Удаление столбцов **created\_at** и **updated\_at**
`$table->dropSoftDeletes();`  |  Удаление столбца **deleted\_at**

<a name="storage-engines"></a>
## Storage Engines (системы хранения)

Для задания конкретной системы хранения таблицы установите свойство `engine` объекта конструктора:

    Schema::create('users', function(Blueprint $table)
    {
        $table->engine = 'InnoDB';

        $table->string('email');
    });

> **Система хранения** - тип архитектуры таблицы. Некоторые СУБД поддерживают только свой встроенный тип (такие, как SQLite), в то время другие - например, MySQL - позволяют использовать различные системы даже внутри одной БД (наиболее используемыми являются `MyISAM`, `InnoDB` и `MEMORY`).
