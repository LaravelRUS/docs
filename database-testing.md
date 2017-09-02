git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Тестирование БД

- [Введение](#introduction)
- [Сброс БД после каждого теста](#resetting-the-database-after-each-test)
    - [Использование миграций](#using-migrations)
    - [Использование транзакций](#using-transactions)
- [Написание фабрик](#writing-factories)
    - [Состояния фабрик](#factory-states)
- [Использование фабрик](#using-factories)
    - [Создание моделей](#creating-models)
    - [Сохранение моделей](#persisting-models)
    - [Отношения](#relationships)
- [Доступные утверждения](#available-assertions)

<a name="introduction"></a>
## Введение

Laravel предоставляет множество полезных инструментов, чтобы упростить тестирование приложений, основанных на базах данных. Во-первых, можно использовать хелпер `assertDatabaseHas`, чтобы утверждать, что данные существуют в БД, соответствующей заданному набору критериев. Например, если вам нужно проверить, что в таблице `users` есть `email` со значением `sally@example.com`, то можно сделать следующее:

    public function testDatabase()
    {
        // Make call to application...

        $this->assertDatabaseHas('users', [
            'email' => 'sally@example.com'
        ]);
    }

Также можно использовать хелпер `assertDatabaseMissing`, чтобы утверждать, что данные не существуют в базе данных.

Конечно, метод `assertDatabaseHas` и другие хелперы, похожие на него, используются для удобства. Вы можете любой встроенный метод утверждений PHPUnit для своих тестов.

<a name="resetting-the-database-after-each-test"></a>
## Сброс БД после каждого теста

Часто полезно сбрасывать свою БД после каждого теста, так чтобы данные из предыдущего теста не мешали последующим тестам.

<a name="using-migrations"></a>
### Использование миграций

Один из способов сброса состояния базы данных - откат базы данных после каждого теста и ей миграция перед следующим тестом. В Laravel есть простой трейт `DatabaseMigrations`, который автоматически обработает это за вас. Просто используйте трейт на классе своего теста и все будет обработано за вас:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        use DatabaseMigrations;

        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->get('/');

            // ...
        }
    }

<a name="using-transactions"></a>
### Использование транзакций

Другой подход к сбросу состояния БД - обернуть каждый тест-кейс в транзакцию базы данных. Опять же, в Laravel есть удобный трейт `DatabaseTransactions`, который автоматически сделает всю работу за вас:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        use DatabaseTransactions;

        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->get('/');

            // ...
        }
    }

> {note} По умолчанию этот трейт будет оборачивать в транзакцию только подключение БД по умолчанию. Если ваше приложение использует несколько БД-подключений, нужно задать свойство `$connectionsToTransact` в вашем тестовом классе. Это свойство должно быть массивом имен подключений, в которых следует производить транзакции.

<a name="writing-factories"></a>
## Написание фабрик

Во время тестирования может потребоваться вставить в вашу БД несколько записей перед выполнением теста. Вместо того, чтобы указывать значение каждой колонки вручную во время создания тестовых данных, Laravel позволяет задать набор атрибутов по умолчанию для каждой из ваших [моделей Eloquent](/docs/{{version}}/eloquent), используя фабрики моделей. Для начала взгляните на файл `database/factories/ModelFactory.php` в вашем приложении. Изначально в этом файле содержится только одно определение фабрики:

    $factory->define(App\User::class, function (Faker\Generator $faker) {
        static $password;

        return [
            'name' => $faker->name,
            'email' => $faker->unique()->safeEmail,
            'password' => $password ?: $password = bcrypt('secret'),
            'remember_token' => str_random(10),
        ];
    });

В рамках функции Closure, которая служит определением фабрики, можно возвращать тестовые значения по умолчанию для всех атрибутов модели. Функция Closure получит экземпляр PHP-библиотеки [Faker](https://github.com/fzaninotto/Faker), которая позволяет удобно генерировать различные типы рандомных данных для тестирования.

Конечно, можно добавить свои дополнительные фабрики к файлу `ModelFactory.php`. Вы можете также создать дополнительные файлы фабрик для каждой модели с целью лучшей организации. Например, можно создать файлы `UserFactory.php` и `CommentFactory.php` в директории `database/factories`. Все эти файлы в директории `factories` будут автоматически загружены Laravel.

<a name="factory-states"></a>
### Состояния фабрик

Состояния позволяют определить дискретные модификации, которые можно применить к фабрикам ваших моделей в любой комбинации. Например, у вашей модели `User` может быть состояние `delinquent`, которое изменяет одно из его значений атрибутов по умолчанию. Трансформации состояний можно задать методом `state`:

    $factory->state(App\User::class, 'delinquent', function ($faker) {
        return [
            'account_status' => 'delinquent',
        ];
    });

<a name="using-factories"></a>
## Использование фабрик

<a name="creating-models"></a>
### Создание моделей

Как только определены ваши фабрики, можно использовать глобальную функцию `factory` в ваших тестах или файлах пополнения БД для генерирования экземпляров моделей. Давайте рассмотрим несколько примеров создания моделей. Во-первых, мы будем использовать метод `make` для создания моделей, но не будем сохранять их в базу данных:

    public function testDatabase()
    {
        $user = factory(App\User::class)->make();

        // Use model in tests...
    }

Ещё можно создать коллекцию из множества моделей или создать модели заданного типа:

    // Create three App\User instances...
    $users = factory(App\User::class, 3)->make();

#### Применение состояний

Можно применить любое из ваших [состояний](#factory-states) к моделям. Если нужно применить несколько трансформаций состояний к моделям, вам следует указать название каждого состояния, которое желаете применить:

    $users = factory(App\User::class, 5)->states('delinquent')->make();

    $users = factory(App\User::class, 5)->states('premium', 'delinquent')->make();

#### Переопределение атрибутов

Если вам нужно переопределить некоторые из значений своих моделей по умолчанию, можно передать массив значений методу `make`. Будут заменены только указанные значения, в то время как оставшиеся значения будут без изменений, как указано фабрикой:

    $user = factory(App\User::class)->make([
        'name' => 'Abigail',
    ]);

<a name="persisting-models"></a>
### Сохранение моделей

Метод `create` не только создает экземпляры моделей, но также и сохраняет из в базу данных, используя Eloquent-метод `save`:

    public function testDatabase()
    {
        // Create a single App\User instance...
        $user = factory(App\User::class)->create();

        // Create three App\User instances...
        $users = factory(App\User::class, 3)->create();

        // Use model in tests...
    }

Можно переопределить атрибуты модели, передав массив методу `create`:

    $user = factory(App\User::class)->create([
        'name' => 'Abigail',
    ]);

<a name="relationships"></a>
### Отношения

В данном примере мы прикрепи отношение к некоторым созданным моделям. При использовании метода `create` для создания нескольких моделей, возвращается [экземпляр коллекции](/docs/{{version}}/eloquent-collections) Eloquent, позволяя вам использовать любые из удобных функций, предоставляемых коллекцией, таких как `each`:

    $users = factory(App\User::class, 3)
               ->create()
               ->each(function ($u) {
                    $u->posts()->save(factory(App\Post::class)->make());
                });

#### Отношения и замыкания атрибутов

К моделям также можно прикрепить отношения, используя атрибуты Closure в ваших определениях фабрики. Например, если вы бы хотели создать новый экземпляр `User` при создании `Post`, можно сделать следующее:

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            }
        ];
    });

Эти функции-замыкания также получают оцененный массив атрибутов фабрики, который их определяет:

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            },
            'user_type' => function (array $post) {
                return App\User::find($post['user_id'])->type;
            }
        ];
    });

<a name="available-assertions"></a>
## Доступные утверждения

Laravel предоставляет несколько БД-утверждения для ваших тестов [PHPUnit](https://phpunit.de/):

Метод  | Описание
------------- | -------------
`$this->assertDatabaseHas($table, array $data);`  |  Таблица в БД содержит заданные данные.
`$this->assertDatabaseMissing($table, array $data);`  |  Таблица в БД не содержит заданные данные.
`$this->assertSoftDeleted($table, array $data);`  |  Заданная запись была мягко удалена.
