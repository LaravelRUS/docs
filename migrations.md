git 41a03286a174d53e2ca3f13d2a194be245d45032

---

# База данных: Миграции (Database: Migrations)

- [Введение (Introduction)](#introduction)
- [Создание (Generating Migrations)](#generating-migrations)
- [Структура (Migration Structure)](#migration-structure)
- [Запуск (Running Migrations)](#running-migrations)
    - [Откат (Rolling Back Migrations)](#rolling-back-migrations)
- [Написание (Writing Migrations)](#writing-migrations)
    - [Создание таблиц (Creating Tables)](#creating-tables)
    - [Переименование/удаление таблиц  (Renaming / Dropping Tables)](#renaming-and-dropping-tables)
    - [Создание столбцов (Creating Columns)](#creating-columns)
    - [Изменение столбцов (Modifying Columns)](#modifying-columns)
    - [Удаление столбцов (Dropping Columns)](#dropping-columns)
    - [Создание индексов (Creating Indexes)](#creating-indexes)
    - [Удаление индексов (Dropping Indexes)](#dropping-indexes)
    - [Ограничения внешнего ключа (Foreign Key Constraints)](#foreign-key-constraints)

<a name="introduction"></a>
## Введение (Introduction)

Миграции похожи на систему контроля версиий, но только для базы данных (БД). Они позволяют команде разработчиков легко изменять схему БД приложения и делиться этими изменениями. Миграции обычно связаны с построителем схем Laravel (Laravel's schema builder) для облегчения создания схемы БД приложения.

Laravel `Schema` [facade](/docs/{{version}}/facades) предоставляет независимую от БД поддержку создания и управления таблицами. Она предоставляет выразительный API для всех поддерживаемых Laravel БД.

<a name="generating-migrations"></a>
## Создание (Generating Migrations)

Для создания миграции используйте `make:migration` [Artisan command](/docs/{{version}}/artisan):

    php artisan make:migration create_users_table

Миграция будет помещена в папку `database/migrations`. Каждое имя файла миграции содержит метку времени, которая позволяет Laravel определять порядок применения миграций.

Опции `--table` и `--create` служат для задания имени таблицы и указания будет ли миграция создавать новую таблицу. Проще говоря, эти опции служат для предварительного заполнения генерируемого файла миграции:

    php artisan make:migration add_votes_to_users_table --table=users

    php artisan make:migration create_users_table --create=users

Для определения иного пути для сохранения файла миграции служит опция `--path`. Путь должен задаваться относительно корня приложения.

<a name="migration-structure"></a>
## Структура миграции (Migration Structure)

Класс миграции содержит 2 метода: `up` и `down`. `up` нужен для добавления новых таблиц, столбцов или индексов в БД, тогда как `down` просто отменяет операции, выполненные в методе `up`.

Внутри обоих методов вы можете использовать конструктор схем для создания и модификации таблиц. Для изучения всех методов, доступных в конструкторе, обратитесь к документации [check out its documentation](#creating-tables). Давайте взглянем на пример миграции, которая создает таблицу `flights`:

    <?php

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;

    class CreateFlightsTable extends Migration
    {
        /**
         * Run the migrations.
         *
         * @return void
         */
        public function up()
        {
            Schema::create('flights', function (Blueprint $table) {
                $table->increments('id');
                $table->string('name');
                $table->string('airline');
                $table->timestamps();
            });
        }

        /**
         * Reverse the migrations.
         *
         * @return void
         */
        public function down()
        {
            Schema::drop('flights');
        }
    }


<a name="running-migrations"></a>
## Запуск миграций (Running Migrations)

Для запуска всех еще незапускавшихся миграций используйте команду `migrate`. Если вы используете [Homestead virtual machine](/docs/{{version}}/homestead), то вы должны запустить ее на вашей VM:

    php artisan migrate

Если вы получили сообщение об ошибке "class not found" во время запуска, попробуйте выполнить команду `composer dump-autoload` и потом вновь выполнить миграцию.

#### Принудительные миграции на продакшене (Forcing Migrations To Run In Production)

Некоторые операции миграции могут привести к потере данных. В целях защиты от запуска таких команд у вас будет запрашиваться подтверждение перед их выполнением. Для выполнения команд без подтверждения укажите флаг `force`:

    php artisan migrate --force

<a name="rolling-back-migrations"></a>
### Откат миграций (Rolling Back Migrations)

Для отката последней выполненной миграции используйте команду `rollback`. Обратите внимание, что откат отменит операции, выполненные последним запущенным файлом миграции, который может подключать другие файлы миграции:

    php artisan migrate:rollback

Команда `migrate:reset` отменит все миграции:

    php artisan migrate:reset

#### Откат/запуск миграции одной командой (Rollback / Migrate In Single Command)

Команда `migrate:refresh` выполняет откат всех миграций, а затем запускает команду `migrate`. Команда полезна для пересоздания всей БД:

    php artisan migrate:refresh

    php artisan migrate:refresh --seed

<a name="writing-migrations"></a>
## Написание миграций (Writing Migrations)

<a name="creating-tables"></a>
### Создание таблиц (Creating Tables)

Метод `Schema::create` служит для создания таблицы. Принимает 2 аргумента. Первый - имя таблицы, второй - замыкание, которое в качестве аргумента принимает объект `Blueprint`, используемый для определения новой таблицы:

    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
    });

Для определения столбцов таблицы вы можете использовать любые методы конструктора схем [column methods](#creating-columns).

#### Проверка существования таблицы/столбца (Checking For Table / Column Existence)

Проверить существует ли таблица или колонка в таблице можно методами `hasTable` и `hasColumn`:

    if (Schema::hasTable('users')) {
        //
    }

    if (Schema::hasColumn('users', 'email')) {
        //
    }

#### Подключение и Движок Хранилища (Connection & Storage Engine)

Если требуется использовать подключение, отличное от дефолтного, используйте метод `connection`:

    Schema::connection('foo')->create('users', function ($table) {
        $table->increments('id');
    });

Для установки движка таблицы, задайте свойство `engine`:

    Schema::create('users', function ($table) {
        $table->engine = 'InnoDB';

        $table->increments('id');
    });

<a name="renaming-and-dropping-tables"></a>
### Переименование/удаление таблиц (Renaming / Dropping Tables)

Переименование таблицы:

    Schema::rename($from, $to);

Удаление таблицы:

    Schema::drop('users');

    Schema::dropIfExists('users');

<a name="creating-columns"></a>
### Создание столбцов (Creating Columns)

Для изменения существующей таблицы служит метод `Schema::table`. Подобно методу `create` метод `table` принимает два аргумента: имя таблицы и замыкание, которое получает в качестве аргумента экземпляр класса `Blueprint`. Добавление столбца в таблицу:

    Schema::table('users', function ($table) {
        $table->string('email');
    });

#### Доступные типы столбцов (Available Column Types)

Перечень доступных типов в классе конструкторе схемы:

Команда  | Описание
------------- | -------------
`$table->bigIncrements('id');`  |  Incrementing ID (primary key) using a "UNSIGNED BIG INTEGER" equivalent.
`$table->bigInteger('votes');`  |  BIGINT equivalent for the database.
`$table->binary('data');`  |  BLOB equivalent for the database.
`$table->boolean('confirmed');`  |  BOOLEAN equivalent for the database.
`$table->char('name', 4);`  |  CHAR equivalent with a length.
`$table->date('created_at');`  |  DATE equivalent for the database.
`$table->dateTime('created_at');`  |  DATETIME equivalent for the database.
`$table->decimal('amount', 5, 2);`  |  DECIMAL equivalent with a precision and scale.
`$table->double('column', 15, 8);`  |  DOUBLE equivalent with precision, 15 digits in total and 8 after the decimal point.
`$table->enum('choices', ['foo', 'bar']);` | ENUM equivalent for the database.
`$table->float('amount');`  |  FLOAT equivalent for the database.
`$table->increments('id');`  |  Incrementing ID (primary key) using a "UNSIGNED INTEGER" equivalent.
`$table->integer('votes');`  |  INTEGER equivalent for the database.
`$table->json('options');`  |  JSON equivalent for the database.
`$table->jsonb('options');`  |  JSONB equivalent for the database.
`$table->longText('description');`  |  LONGTEXT equivalent for the database.
`$table->mediumInteger('numbers');`  |  MEDIUMINT equivalent for the database.
`$table->mediumText('description');`  |  MEDIUMTEXT equivalent for the database.
`$table->morphs('taggable');`  |  Adds INTEGER `taggable_id` and STRING `taggable_type`.
`$table->nullableTimestamps();`  |  Same as `timestamps()`, except allows NULLs.
`$table->rememberToken();`  |  Adds `remember_token` as VARCHAR(100) NULL.
`$table->smallInteger('votes');`  |  SMALLINT equivalent for the database.
`$table->softDeletes();`  |  Adds `deleted_at` column for soft deletes.
`$table->string('email');`  |  VARCHAR equivalent column.
`$table->string('name', 100);`  |  VARCHAR equivalent with a length.
`$table->text('description');`  |  TEXT equivalent for the database.
`$table->time('sunrise');`  |  TIME equivalent for the database.
`$table->tinyInteger('numbers');`  |  TINYINT equivalent for the database.
`$table->timestamp('added_on');`  |  TIMESTAMP equivalent for the database.
`$table->timestamps();`  |  Adds `created_at` and `updated_at` columns.
`$table->uuid('id');`  |  UUID equivalent for the database.

#### Модификация столбцов (Column Modifiers)

В дополнение к типам, перечисленным выше, доступны модификаторы столбцов, которые можно использовать при добавлении столбца. Добавим столбцу возможность принимать значения NULL:

    Schema::table('users', function ($table) {
        $table->string('email')->nullable();
    });

Ниже приведен список доступных модификаторов. Список не включает индексные модификаторы [index modifiers](#creating-indexes):

Модификатор  | Описание
------------- | -------------
`->first()`  |  Place the column "first" in the table (MySQL Only)
`->after('column')`  |  Place the column "after" another column (MySQL Only)
`->nullable()`  |  Allow NULL values to be inserted into the column
`->default($value)`  |  Specify a "default" value for the column
`->unsigned()`  |  Set `integer` columns to `UNSIGNED`

<a name="changing-columns"></a>
<a name="modifying-columns"></a>
### Изменение столбцов (Modifying Columns)

#### Необходимые условия (Prerequisites)

Перед изменением столбца убедитесь, что в файле `composer.json` есть зависимость `doctrine/dbal`. Библиотека Doctrine DBAL нужна для определения текущего статуса столбца и создания SQL запросов для выполнения заданных изменений столбца.

#### Обновление атрибутов столбца (Updating Column Attributes)

Метод `change` меняет тип столбца либо атрибуты. Например, увеличим размер строкового столбца `name` с 25 до 50:

    Schema::table('users', function ($table) {
        $table->string('name', 50)->change();
    });

Добавить вставку значений NULL:

    Schema::table('users', function ($table) {
        $table->string('name', 50)->nullable()->change();
    });

> **Примечание:** Изменение столбцов с типом `enum` на текущий момент не поддерживается.

<a name="renaming-columns"></a>
#### Переименование столбцов (Renaming Columns)

Переименование столбца:

    Schema::table('users', function ($table) {
        $table->renameColumn('from', 'to');
    });

> **Примечание:** Переименование столбцов типа `enum` на текущий момент не поддерживается.

<a name="dropping-columns"></a>
### Удаление столбцов (Dropping Columns)

Удаление столбца:

    Schema::table('users', function ($table) {
        $table->dropColumn('votes');
    });

Удаление нескольких столбцов:

    Schema::table('users', function ($table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });

> **Примечание:** Перед удалением столбцов из СУБД SQLite убедитесь, что у вас установлена `doctrine/dbal`. Если нет, то пропишите ее в файле `composer.json` и выполните команду `composer update`.

> **Примечание:** Удаление/изменение нескольких столбцов внутри одной миграции для СУБД SQLite не поддерживается.

<a name="creating-indexes"></a>
### Создание индексов (Creating Indexes)

Уникальный индекс:

    $table->string('email')->unique();

Создание индекса для существующего столбца:

    $table->unique('email');

Индекс на основе нескольких столбцов:

    $table->index(['account_id', 'created_at']);

Laravel автоматически генерирует имя индекса, но можно указать его самому во втором параметре метода:

    $table->index('email', 'my_index_name');

#### Доступные типа индексов (Available Index Types)

Команда  | Описание
------------- | -------------
`$table->primary('id');`  |  Add a primary key.
`$table->primary(['first', 'last']);`  |  Add composite keys.
`$table->unique('email');`  |  Add a unique index.
`$table->unique('state', 'my_index_name');`  |  Add a custom index name.
`$table->index('state');`  |  Add a basic index.

<a name="dropping-indexes"></a>
### Удаление индексов (Dropping Indexes)

Для удаление индекса нужно указать имя индекса. По умолчанию, Laravel создает имя для индекса, соединяя имя таблицы, имя столбца и тип индекса. Несколько примеров:

Команда  | Описание
------------- | -------------
`$table->dropPrimary('users_id_primary');`  |  Удалить primary key из таблицы "users".
`$table->dropUnique('users_email_unique');`  |  Удалить уникальный индекс из таблицы "users".
`$table->dropIndex('geo_state_index');`  |  Удалить обычный индекс из таблицы "geo".


Если методу для удаления передать массив столбцов для удаления индекса, то имя индекса будет сформировано автоматически:

    Schema::table('geo', function ($table) {
        $table->dropIndex(['state']); // Удалить индекс 'geo_state_index'
    });

<a name="foreign-key-constraints"></a>
### Ограничения внешнего ключа (Foreign Key Constraints)

Laravel поддерживает создание внешних ключей, которые служат для обеспечения целостности данных на уровне БД. Например, определим столбец `user_id` в таблице `posts`, который сылается на столбец `id` таблицы `users`:

    Schema::table('posts', function ($table) {
        $table->integer('user_id')->unsigned();

        $table->foreign('user_id')->references('id')->on('users');
    });

Задать желаемое действие для свойств "on delete" и "on update" внешнего ключа:

    $table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

При создании имени внешнего ключа используется то же соглашение о присвоении имен, что и для индексов. Laravel будет создавать имя на основе
имени таблицы, столбца и суффикса "_foreign". Удаление  внешнего ключа:

    $table->dropForeign('posts_user_id_foreign');


Удаление при задании столбца в массиве, что формирует имя удаляемого ключа автоматически:

    $table->dropForeign(['user_id']);
