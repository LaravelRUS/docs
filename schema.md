git 619224347ee7e68c800811a46d9ad6e2c6b40db6

---

# Конструктор таблиц

- [Введение](#introduction)
- [Создание и удаление таблиц](#creating-and-dropping-tables)
- [Добавление полей](#adding-columns)
- [Переименование полей](#renaming-columns)
- [Удаление полей](#dropping-columns)
- [Проверка на существование](#checking-existence)
- [Добавление индексов](#adding-indexes)
- [Внешние ключи](#foreign-keys)
- [Удаление индексов](#dropping-indexes)
- [Storage Engines](#storage-engines)

<a name="introduction"></a>
## Введение

Класс `Schema` представляет собой независимый от БД интерфейс манипулирования таблицами. Он хорошо работает со всеми СУБД, поддерживаемыми Laravel и предоставляет унифицированный API для любой из этих систем.

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

Для изменения существующей таблицы мы будем использовать метод `Schema::table`:

	Schema::table('users', function(Blueprint $table)
	{
		$table->string('email');
	});

Конструктор таблиц поддерживает различные типы полей:

Команда  | Описание
------------- | -------------
`$table->increments('id');`  |  Первичный последовательный ключ (autoincrement).
`$table->bigIncrements('id');`  |  Первичный последовательный ключ типа BIGINT.
`$table->string('email');`  |  Поле VARCHAR
`$table->string('name', 100);`  |  Поле VARCHAR с указанной длиной
`$table->integer('votes');`  |  Поле INTEGER
`$table->bigInteger('votes');`  |  Поле BIGINT
`$table->smallInteger('votes');`  |  Поле SMALLINT
`$table->float('amount');`  |  Поле FLOAT
`$table->decimal('amount', 5, 2);`  |  Поле DECIMAL с указанной размерностью и точностью
`$table->boolean('confirmed');`  |  Поле BOOLEAN
`$table->date('created_at');`  |  Поле DATE
`$table->dateTime('created_at');`  |  Поле DATETIME
`$table->time('sunrise');`  |  Поле TIME
`$table->timestamp('added_on');`  |  Поле TIMESTAMP
`$table->timestamps();`  |  Добавляет поля **created\_at** и **updated\_at**
`$table->softDeletes();`  |  Добавляет поле **deleted\_at** для мягкого удаления
`$table->text('description');`  |  Поле TEXT
`$table->binary('data');`  |  Поле BLOB
`$table->enum('choices', array('foo', 'bar'));` | Поле ENUM
`->nullable()`  |  Указывает, что поле может быть NULL
`->default($value)`  |  Указывает значение по умолчанию для поля
`->unsigned()`  |  Обозначает беззнаковое число UNSIGNED

#### Вставка поля после существующего (в MySQL)

$table->string('name')->after('email');

	$table->string('name')->after('email');

<a name="renaming-columns"></a>
## Переименование полей

Для переименования поля можно использовать метод `renameColumn`. Переименование возможно только при подключенном пакете `doctrine/dbal` в `composer.json`.

	Schema::table('users', function(Blueprint $table)
	{
		$table->renameColumn('from', 'to');
	});

> **Внимание:** переименование полей типа `enum` не поддерживается.

<a name="dropping-columns"></a>
## Удаление полей

#### Удаление одного поля из таблицы:

	Schema::table('users', function(Blueprint $table)
	{
		$table->dropColumn('votes');
	});

#### Удаление сразу нескольких полей

	Schema::table('users', function(Blueprint $table)
	{
		$table->dropColumn('votes', 'avatar', 'location');
	});

<a name="checking-existence"></a>
## Проверка на существование

#### Проверка существования таблицы

Вы можете легко проверить существование таблицы или поля с помощью методов  `hasTable` и `hasColumn`.

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

Есть два способа добавлять индексы: можно определять их во время определения полей, либо добавлять отдельно:

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

Laravel поддерживает добавление внешних ключей (foreign key constraints) для ваших таблиц:

	$table->foreign('user_id')->references('id')->on('users');

В этом примере мы указываем, что поле `user_id` связано с полем `id` таблицы `users`.

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

<a name="storage-engines"></a>
## Storage Engines

Для задания конкретной системы хранения таблицы установите свойство `engine` объекта конструктора:

    Schema::create('users', function(Blueprint $table)
    {
        $table->engine = 'InnoDB';

        $table->string('email');
    });

> **Система хранения** - тип архитектуры таблицы. Некоторые СУБД поддерживают только свой встроенный тип (такие, как SQLite), в то время другие - например, MySQL - позволяют использовать различные системы даже внутри одной БД (наиболее используемыми являются `MyISAM`, `InnoDB` и `MEMORY`). Правда, использование таблиц различных архитектур в одном запросе заметно снижает его производительность - прим. пер.