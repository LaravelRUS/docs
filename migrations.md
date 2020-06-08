git 18796f22fdbb32a6432e1bce92e36ceb9afb42c8

---

# База данных: Миграции

- [Введение](#introduction)
- [Создание миграций](#generating-migrations)
- [Структура миграций](#migration-structure)
- [Выполнение миграций](#running-migrations)
    - [Откат миграций](#rolling-back-migrations)
- [Таблицы](#tables)
    - [Создание таблиц](#creating-tables)
    - [Переименование / удаление таблиц](#renaming-and-dropping-tables)
- [Столбцы](#columns)
    - [Создание столбцов](#creating-columns)
    - [Модификаторы столбцов](#column-modifiers)
    - [Изменение столбцов](#modifying-columns)
    - [Удаление столбцов](#dropping-columns)
- [Индексы](#indexes)
    - [Создание индексов](#creating-indexes)
    - [Удаление индексов](#dropping-indexes)
    - [Ограничения внешнего ключа](#foreign-key-constraints)

<a name="introduction"></a>
## Введение

Миграции — что-то вроде системы контроля версий для вашей базы данных. Они позволяют вашей команде изменять структуру БД, в то же время оставаясь в курсе изменений других участников. Миграции обычно идут рука об руку с построителем структур для более простого обращения с архитектурой вашей базы данных. Если вы когда-нибудь просили коллегу вручную добавить столбец в его локальную БД, значит вы сталкивались с проблемой, которую решают миграции БД.

[Фасад](/docs/{{version}}/facades) Laravel `Schema` обеспечивает поддержку создания и изменения таблиц в независимости от используемой СУБД из числа тех, что поддерживаются в Laravel.

<a name="generating-migrations"></a>
## Создание миграций

Для создания новой миграции используйте [Artisan-команду](/docs/{{version}}/artisan) `make:migration` :

    php artisan make:migration create_users_table

Миграция будет помещена в директорию `database/migrations`. Название файла каждой миграции будет содержать метку времени, которая позволяет фреймворку определять порядок применения миграций.

Можно также использовать параметры `--table` и `--create` для указания имени таблицы и того факта, что миграция будет создавать новую таблицу. Эти параметры просто заранее создают указанную таблицу в создаваемом файле-заглушке миграции:

    php artisan make:migration create_users_table --create=users

    php artisan make:migration add_votes_to_users_table --table=users

Если вы хотите указать свой путь для сохранения создаваемых миграций, используйте параметр `--path` при запуске команды `make:migration`. Этот путь должен быть указан относительно базового пути вашего приложения.

<a name="migration-structure"></a>
## Структура миграций

Класс миграций содержит два метода: `up` и `down`. Метод `up` используется для добавления новых таблиц, столбцов или индексов в вашу БД, а метод `down` просто отменяет операции, выполненные методом `up`.

В обоих методах вы можете использовать построитель структур Laravel для удобного создания и изменения таблиц. О всех доступных методах построителя структур `Schema` [читайте в его доументации](#creating-tables). Например, эта миграция создаёт таблицу `flights`:

    <?php

    use Illuminate\Support\Facades\Schema;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;

    class CreateFlightsTable extends Migration
    {
        /**
         * Выполнение миграций.
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
         * Отмена миграций.
         *
         * @return void
         */
        public function down()
        {
            Schema::drop('flights');
        }
    }


<a name="running-migrations"></a>
## Выполнение миграций

Для запуска всех необходимых вам миграций используйте Artisan-команду `migrate`:

    php artisan migrate

> {note} Если вы используете [виртуальную машину Homestead](/docs/{{version}}/homestead), вам надо выполнить эту команду на своей ВМ.

#### Принудительные миграции в продакшне

Некоторые операции миграций разрушительны, значит они могут привести к потере ваших данных. Для предотвращения случайного запуска этих команд на вашей боевой БД перед их выполнением запрашивается подтверждение. Для принудительного запуска команд без подтверждения используйте ключ `--force`:

    php artisan migrate --force

<a name="rolling-back-migrations"></a>
### Откат миграций

Для отмены изменений, сделанных последней миграцией, используйте команду `rollback`. Эта команда отменит результат последней "партии" миграций, которая может включать несколько файлов миграций:

    php artisan migrate:rollback

Вы можете сделать откат определённого числа миграций, указав параметр `step` для команды `rollback`. Например, эта команда откатит последние пять миграций:

    php artisan migrate:rollback --step=5

Команда `migrate:reset` откатит изменения всех миграций вашего приложения:

    php artisan migrate:reset

#### Откат всех миграций и их повторное применение одной командой

Команда `migrate:refresh` отменит изменения всех ваших миграций, а затем выполнит команду `migrate`. Эта команда эффективно создаёт заново всю вашу БД:

    php artisan migrate:refresh

    // Обновить БД и запустить заполнение БД начальными данными...
    php artisan migrate:refresh --seed

Вы можете откатить и повторно применить определённое число миграций, указав параметр `step` для команды `refresh`. Например, эта команда откатит и повторно применит последние пять миграций:

    php artisan migrate:refresh --step=5

<a name="tables"></a>
## Таблицы

<a name="creating-tables"></a>
### Создание таблиц

Для создания новой таблицы БД используйте метод `create` фасада `Schema`. Метод `create` принимает два аргумента. Первый — имя таблицы, второй — функция `Closure`, которую получает объект `Blueprint`, который можно использовать для определения новой таблицы:

    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
    });

Само собой, при создании таблицы вы можете использовать любые [методы для работы со столбцами](#creating-columns) построителя структур.

#### Проверка существования таблицы / столбца

Вы можете легко проверить существование таблицы или столбца при помощи методов `hasTable` и `hasColumn`:

    if (Schema::hasTable('users')) {
        //
    }

    if (Schema::hasColumn('users', 'email')) {
        //
    }

#### Подключение и подсистема хранения данных

Если вы хотите выполнить операции над структурой через подключение к БД, которое не является вашим основным подключением, используйте метод `connection`:

    Schema::connection('foo')->create('users', function (Blueprint $table) {
        $table->increments('id');
    });

Используйте свойство `engine` построителя структур, чтобы задать подсистему хранения данных для таблицы:

    Schema::create('users', function (Blueprint $table) {
        $table->engine = 'InnoDB';

        $table->increments('id');
    });

<a name="renaming-and-dropping-tables"></a>
### Переименование / удаление таблиц

Для переименования существующей таблицы используйте метод `rename`:

    Schema::rename($from, $to);

Для удаления существующей таблицы используйте методы `drop` или `dropIfExists`:

    Schema::drop('users');

    Schema::dropIfExists('users');

#### Переименование таблиц с внешними ключами

Перед переименованием таблицы вы должны проверить, что для всех ограничений внешних ключей таблицы есть явные имена в файлах вашей миграции, чтобы избежать автоматического назначения имён на основе принятого соглашения. Иначе имя ограничения внешнего ключа будет ссылаться на имя старой таблицы.

<a name="columns"></a>
## Столбцы

<a name="creating-columns"></a>
### Создание столбцов

Для изменения существующей таблицы мы будем использовать метод `table` фасада `Schema`. Как и метод `create`, метод `table` принимает два аргумента: имя таблицы и замыкание, которое получает экземпляр `Blueprint`, который можно использовать для добавления столбцов в таблицу:

    Schema::table('users', function (Blueprint $table) {
        $table->string('email');
    });

#### Доступные типы столбцов

Разумеется, построитель структур содержит различные типы столбцов, которые вы можете указывать при построении ваших таблиц:

Команда  | Описание
------------- | -------------
`$table->bigIncrements('id');`  |  Инкрементный ID (первичный ключ), использующий эквивалент "UNSIGNED BIG INTEGER".
`$table->bigInteger('votes');`  |  Эквивалент BIGINT для базы данных.
`$table->binary('data');`  |  Эквивалент BLOB для базы данных.
`$table->boolean('confirmed');`  |  Эквивалент BOOLEAN для базы данных.
`$table->char('name', 4);`  |  Эквивалент CHAR для базы данных.
`$table->date('created_at');`  |  Эквивалент DATE для базы данных.
`$table->dateTime('created_at');`  |  Эквивалент DATETIME для базы данных.
`$table->dateTimeTz('created_at');`  |  Эквивалент DATETIME (с часовым поясом) для базы данных.
`$table->decimal('amount', 5, 2);`  |  Эквивалент DECIMAL с точностью и масштабом.
`$table->double('column', 15, 8);`  |  Эквивалент DOUBLE с точностью, всего 15 цифр, после запятой 8 цифр.
`$table->enum('choices', ['foo', 'bar']);` | Эквивалент ENUM для базы данных.
`$table->float('amount', 8, 2);`  |  Эквивалент FLOAT для базы данных, всего 8 знаков, из них 2 после запятой.
`$table->increments('id');`  |  Инкрементный ID (первичный ключ), использующий эквивалент "UNSIGNED INTEGER".
`$table->integer('votes');`  |  Эквивалент INTEGER для базы данных.
`$table->ipAddress('visitor');`  |  Эквивалент IP-адреса для базы данных.
`$table->json('options');`  |  Эквивалент JSON для базы данных.
`$table->jsonb('options');`  |  Эквивалент JSONB для базы данных.
`$table->longText('description');`  |  Эквивалент LONGTEXT для базы данных.
`$table->macAddress('device');`  |  Эквивалент MAC-адреса для базы данных.
`$table->mediumIncrements('id');`  |  Инкрементный ID (первичный ключ), использующий эквивалент "UNSIGNED MEDIUM INTEGER".
`$table->mediumInteger('numbers');`  |  Эквивалент MEDIUMINT для базы данных.
`$table->mediumText('description');`  |  Эквивалент MEDIUMTEXT для базы данных.
`$table->morphs('taggable');`  |  Добавление столбца `taggable_id` INTEGER и `taggable_type` STRING.
`$table->nullableMorphs('taggable');`  |  Аналогично `morphs()`, но разрешено значение NULL.
`$table->nullableTimestamps();`  |  Аналогично `timestamps()`, но разрешено значение NULL.
`$table->rememberToken();`  |  Добавление столбца `remember_token` как VARCHAR(100) NULL.
`$table->smallIncrements('id');`  |  Инкрементный ID (первичный ключ), использующий эквивалент "UNSIGNED SMALL INTEGER".
`$table->smallInteger('votes');`  |  Эквивалент SMALLINT для базы данных.
`$table->softDeletes();`  |     Добавление столбца `deleted_at` для мягкого удаления с разрешенным значением NULL.
`$table->string('email');`  |  Эквивалент VARCHAR.
`$table->string('name', 100);`  |  Эквивалент VARCHAR с длиной.
`$table->text('description');`  |  Эквивалент TEXT для базы данных.
`$table->time('sunrise');`  |  Эквивалент TIME для базы данных.
`$table->timeTz('sunrise');`  |  Эквивалент TIME (с часовым поясом) для базы данных.
`$table->tinyInteger('numbers');`  |  Эквивалент TINYINT для базы данных.
`$table->timestamp('added_on');`  |  Эквивалент TIMESTAMP для базы данных.
`$table->timestampTz('added_on');`  |  Эквивалент TIMESTAMP (с часовым поясом) для базы данных.
`$table->timestamps();`  |  Добавление столбцов `created_at` и `updated_at` с разрешенным значением NULL.
`$table->timestampsTz();`  |  Добавление столбцов `created_at` и `updated_at` (с часовым поясом), для которых разрешено значение NULL.
`$table->unsignedBigInteger('votes');`  |  Эквивалент Unsigned BIGINT для базы данных.
`$table->unsignedInteger('votes');`  |  Эквивалент Unsigned INT для базы данных.
`$table->unsignedMediumInteger('votes');`  |  Эквивалент Unsigned MEDIUMINT для базы данных.
`$table->unsignedSmallInteger('votes');`  |  Эквивалент Unsigned SMALLINT для базы данных.
`$table->unsignedTinyInteger('votes');`  |  Эквивалент Unsigned TINYINT для базы данных.
`$table->uuid('id');`  |  Эквивалент UUID для базы данных.

<a name="column-modifiers"></a>
### Модификаторы столбцов

Вдобавок к перечисленным типам столбцов существует несколько "модификаторов" столбцов, которые вы можете использовать при добавлении столбцов в таблицу. Например, чтобы сделать столбец "обнуляемым", используйте метод `nullable`:

    Schema::table('users', function (Blueprint $table) {
        $table->string('email')->nullable();
    });

Ниже перечислены все доступные модификаторы столбцов. В этом списке отсутствуют [модицикаторы индексов](#creating-indexes):

Модификатор  | Описание
------------- | -------------
`->after('column')`  |  Помещает столбец "после" указанного столбца (только MySQL)
`->comment('my comment')`  |  Добавляет комментарий в столбец
`->default($value)`  |  Указывает значение "по умолчанию" для столбца
`->first()`  |  Помещает столбец "первым" в таблице (только MySQL)
`->nullable()`  |  Разрешает вставлять значения NULL в столбец
`->storedAs($expression)`  |  Создать генерируемый столбец типа stored (только MySQL)
`->unsigned()`  |  Делает столбцы `integer` беззнаковыми `UNSIGNED`
`->virtualAs($expression)`  |  Создать генерируемый столбец типа virtual (только MySQL)

<a name="changing-columns"></a>
<a name="modifying-columns"></a>
### Изменение столбцов

#### Требования

Перед изменением столбцов добавьте зависимость `doctrine/dbal` в свой файл `composer.json`. Библиотека Doctrine DBAL используется для определения текущего состояния столбца и создания SQL-запросов, необходимых для выполнения указанных преобразований столбца:

    composer require doctrine/dbal

#### Изменение атрибутов столбца

Метод `change` позволяет изменить тип существующего столбца или изменить его атрибуты. Например, если вы захотите увеличить размер строкового столбца `name` с 25 до 50:

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->change();
    });

Также мы можем изменить столбец, чтобы он могу иметь значения NULL:

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->nullable()->change();
    });

> {note} Столбы следующих типов нельзя "изменить": char, double, enum, mediumInteger, timestamp, tinyInteger, ipAddress, json, jsonb, macAddress, mediumIncrements, morphs, nullableMorphs, nullableTimestamps, softDeletes, timeTz, timestampTz, timestamps, timestampsTz, unsignedMediumInteger, unsignedTinyInteger, uuid.

<a name="renaming-columns"></a>
#### Переименование столбцов

Для переименования столбца используйте метод `renameColumn` на построителе структур. Перед переименованием столбца добавьте зависимость `doctrine/dbal` в свой файл `composer.json`:

    Schema::table('users', function (Blueprint $table) {
        $table->renameColumn('from', 'to');
    });

> {note} Пока не поддерживается переименование любых столбцов в таблице, содержащей столбцы типов `enum`.

<a name="dropping-columns"></a>
### Удаление столбцов

Для удаления столбца используйте метод `dropColumn` на построителе структур. Перед удалением столбцов из базы данных SQLite вам необходимо добавить зависимость `doctrine/dbal` в ваш файл `composer.json` и выполнить команду `composer update` для установки библиотеки:

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('votes');
    });

Вы можете удалить несколько столбцов таблицы, передав массив их имён в метод `dropColumn`:

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });

> {note} Удаление и изменение нескольких столбцов одной миграцией не поддерживается для базы данных SQLite.

<a name="indexes"></a>
## Индексы

<a name="creating-indexes"></a>
### Создание индексов

Построитель структур поддерживает несколько типов индексов. Сначала давайте посмотрим на пример, в котором задаётся, что значения столбца должны быть уникальными. Для создания индекса мы можем просто сцепить метод `unique` с определением столбца:

    $table->string('email')->unique();

Другой вариант — создать индекс после определения столбца. Например:

    $table->unique('email');

Вы можете даже передать массив столбцов в метод индексирования для создания сложного индекса:

    $table->index(['account_id', 'created_at']);

Laravel автоматически генерирует подходящее имя индекса, но вы можете передать своё значение вторым аргументом метода:

    $table->index('email', 'my_index_name');

#### Доступные типы индексов

Команда  | Описание
------------- | -------------
`$table->primary('id');`  |  Добавление первичного ключа.
`$table->primary(['first', 'last']);`  |  Добавление составных ключей.
`$table->unique('email');`  |  Добавление уникального индекса.
`$table->unique('state', 'my_index_name');`  |  Добавление пользовательского имени индекса.
`$table->unique(['first', 'last']);`  |  Добавление составного уникального индекса.
`$table->index('state');`  |  Добавление базового индекса.

#### Длина индексов и MySQL / MariaDB

По умолчанию Laravel использует набор символов `utf8mb4`, который включает поддаржку хранения "эмоджи" в базе данных. Если вы работаете на версии MySQL старее, чем релиз 5.7.7 или версии MariaDB старее, чем релиз 10.2.2, вам может потребоваться вручную настроить длину строки по умолчанию, которая генерируется миграциями. Таким образом, MySQL сможет генерировать для них индексы. Это можно настроить вызвав метод `Schema::defaultStringLength` в вашем `AppServiceProvider`:

    use Illuminate\Support\Facades\Schema;

    /**
     * Начальная загрузка любых сервисов приложения.
     *
     * @return void
     */
    public function boot()
    {
        Schema::defaultStringLength(191);
    }

Или же, вы можете включить опцию `innodb_large_prefix` для своей базы данных. См. документацию своей базы данных для получения инструкций о том, как правильно включить данную опцию.

<a name="dropping-indexes"></a>
### Удаление индексов

Для удаления индекса необходимо указать его имя. По умолчанию Laravel автоматически назначает имена индексам. Просто соедините имя таблицы, имя столбца-индекса и тип индекса. Ниже приведено несколько примеров:

Команда  | Описание
------------- | -------------
`$table->dropPrimary('users_id_primary');`  |  Удаление первичного ключа из таблицы "users".
`$table->dropUnique('users_email_unique');`  |  Удаление уникального индекса из таблицы "users".
`$table->dropIndex('geo_state_index');`  |  Удаление базового индекса из таблицы "geo".

Если вы передадите массив столбцов в метод для удаления индексов, будет сгенерировано стандартное имя индекса на основе имени таблицы, столбца и типа ключа:

    Schema::table('geo', function (Blueprint $table) {
        $table->dropIndex(['state']); // Drops index 'geo_state_index'
    });

<a name="foreign-key-constraints"></a>
### Ограничения внешнего ключа

Laravel также поддерживает создание ограничений для внешнего ключа, которые используются для обеспечения ссылочной целостности на уровне базы данных. Например, давайте определим столбец `user_id` в таблице `posts`, который ссылается на столбец `id` в таблице `users`:

    Schema::table('posts', function (Blueprint $table) {
        $table->integer('user_id')->unsigned();

        $table->foreign('user_id')->references('id')->on('users');
    });

Вы также можете указать требуемое действие для свойств ограничений "on delete" и "on update":

    $table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

Для удаления внешнего ключа используйте метод `dropForeign`. Ограничения внешнего ключа используют те же принципы именования, что и индексы. Итак, мы соединим имя таблицы и столбцов из ограничения, а затем добавим суффикс "_foreign":

    $table->dropForeign('posts_user_id_foreign');

Либо вы можете передать значение массива, при этом для удаления будет автоматически использовано стандартное имя ограничения:

    $table->dropForeign(['user_id']);

Вы можете включить или выключить ограничения внешнего ключа в своих миграциях с помощью следующих методов:

    Schema::enableForeignKeyConstraints();

    Schema::disableForeignKeyConstraints();
