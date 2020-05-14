git ec84b6789bae6f3233f584bdbea7c1d9aff9dc85

---

# Тестирование БД

- [Введение](#introduction)
- [Создание фабрик](#generating-factories)
- [Сброс БД после каждого теста](#resetting-the-database-after-each-test)
- [Написание фабрик](#writing-factories)
    - [Расширение фабрик](#extending-factories)
    - [Состояния фабрик](#factory-states)
    - [Колбэки фабрик](#factory-callbacks)
- [Использование фабрик](#using-factories)
    - [Создание моделей](#creating-models)
    - [Сохранение моделей](#persisting-models)
    - [Отношения](#relationships)
- [Использование сидов](#using-seeds)    
- [Доступные методы PHPUnit](#available-assertions)

<a name="introduction"></a>
## Введение

Laravel предоставляет ряд инструментов, чтобы упростить тестирование приложений, использующих базу данных. Во-первых, можно использовать хелпер `assertDatabaseHas`, чтобы проверять, что данные существуют в БД. Например, если вам нужно проверить, что в таблице `users` есть `email` со значением `sally@example.com`, то можно сделать следующее:

    public function testDatabase()
    {
        // Make call to application...

        $this->assertDatabaseHas('users', [
            'email' => 'sally@example.com'
        ]);
    }

Также можно использовать хелпер `assertDatabaseMissing`, чтобы утверждать, что данные не существуют в базе данных.

Метод `assertDatabaseHas` и другие хелперы, похожие на него, используются для удобства. Вы можете использовать любой встроенный метод PHPUnit для своих тестов.

<a name="generating-factories"></a>
## Создание фабрик

Для того, чтобы сгенерировать файл фабрики, используйте [Artisan-команду](/docs/{{version}}/artisan) `make:factory` :

    php artisan make:factory PostFactory

Файл будет создан в директории `database/factories`.

При помощи опции `--model` можно указать название модели, данные которой будут взяты для генерации файла фабрики.

    php artisan make:factory PostFactory --model=Post

<a name="resetting-the-database-after-each-test"></a>
## Сброс БД после каждого теста

Часто полезно сбрасывать свою БД после каждого теста, так чтобы данные из предыдущего теста не мешали последующим тестам. Для этого наболее оптимально использовать трейт `RefreshDatabase`. Укажите его и перед каждым тестом миграции будут перенакатываться:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        use RefreshDatabase;

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

<a name="writing-factories"></a>
## Написание фабрик

Во время тестирования может потребоваться вставить в вашу БД несколько записей перед выполнением теста. Вместо того, чтобы указывать значение каждой колонки вручную во время создания тестовых данных, Laravel позволяет задать набор атрибутов по умолчанию для каждой из ваших [моделей Eloquent](/docs/{{version}}/eloquent), используя фабрики моделей. Для начала взгляните на файл `database/factories/UserFactory.php` в вашем приложении. Изначально в этом файле содержится только одно определение фабрики:
    
    use Faker\Generator as Faker;
    use Illuminate\Support\Str;

    $factory->define(App\User::class, function (Faker $faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->unique()->safeEmail,
            'email_verified_at' => now(),
            'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
            'remember_token' => Str::random(10),
        ];
    });

В рамках анонимной функции, которая служит определением фабрики, можно возвращать тестовые значения по умолчанию для всех атрибутов модели. Функция получает экземпляр PHP-библиотеки [Faker](https://github.com/fzaninotto/Faker), которая позволяет удобно генерировать различные типы рандомных данных для тестирования.

Вы можете также создать дополнительные файлы фабрик для каждой модели с целью лучшей организации. Например, можно создать файлы `UserFactory.php` и `CommentFactory.php` в директории `database/factories`. Все эти файлы в директории `factories` будут автоматически загружены Laravel.

> {tip} Язык локализации для Faker устанавливается в опции `faker_locale` в файле конфигурации `config/app.php`.

<a name="extending-factories"></a>
### Расширение фабрик

Если вы создали модель на базе другой модели, расширив (extends) её, вы, возможно, захотите расширить ее фабрику, чтобы использовать атрибуты фабрики дочерней модели. Для этого вы можете вызвать метод `raw` для получения исходного массива атрибутов с заданной фабрики:

    $factory->define(App\Admin::class, function (Faker\Generator $faker) {
        return factory(App\User::class)->raw([
            // ...
        ]);
    });

<a name="factory-states"></a>
### Состояния фабрик

Состояния позволяют определить дискретные модификации, которые можно применить к фабрикам ваших моделей в любой комбинации. Например, у вашей модели `User` может быть состояние `delinquent`, которое изменяет одно из его значений атрибутов по умолчанию. Трансформации состояний можно задать методом `state`. В простейшем случае, в качестве аргумента можно передать массив новых значений модели:

    $factory->state(App\User::class, 'delinquent', [
        'account_status' => 'delinquent',
    ]);

Можно передать анонимную функцию, если нужно генерировать новые значения при помощи `Faker`:

    $factory->state(App\User::class, 'address', function ($faker) {
        return [
            'address' => $faker->address,
        ];
    });

<a name="factory-callbacks"></a>
### Коллбэки фабрик

Используя методы `afterMaking` и `afterCreating` можно задать функции, которые автоматически вызовутся после `make()` фабрики (т.е. создания объекта модели в памяти) и `create()` (т.е. создания объекта модели и сохранения его в БД)

    $factory->afterMaking(App\User::class, function ($user, $faker) {
        // ...
    });

    $factory->afterCreating(App\User::class, function ($user, $faker) {
        $user->accounts()->save(factory(App\Account::class)->make());
    });

Можно определять коллбэки и для [состояний фабрики](#factory-states):

    $factory->afterMakingState(App\User::class, 'delinquent', function ($user, $faker) {
        // ...
    });

    $factory->afterCreatingState(App\User::class, 'delinquent', function ($user, $faker) {
        // ...
    });

<a name="using-factories"></a>
## Использование фабрик

<a name="creating-models"></a>
### Создание моделей

Как только определены ваши фабрики, можно использовать глобальную функцию `factory` в ваших тестах или сидах БД для генерирования экземпляров моделей. Давайте рассмотрим несколько примеров создания моделей. Во-первых, мы будем использовать метод `make`, который создаёт модель, но не соханяет её в БД:

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

> {tip} [Защита от массового присваивания в модели](/docs/{{version}}/eloquent#mass-assignment) во время работы фабрики автоматически отключается.    

<a name="persisting-models"></a>
### Сохранение моделей

Метод `create` не только создает экземпляры моделей, но также и сохраняет их в базу данных, используя Eloquent-метод `save`:

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
               ->each(function ($user) {
                    $user->posts()->save(factory(App\Post::class)->make());
                });

Этот код можно переписать с использованием метода `createMany`:

    $user->posts()->createMany(
        factory(App\Post::class, 3)->make()->toArray()
    );                

#### Отношения и формирование аттрибутов

К моделям также можно прикрепить отношения, используя анонимные функции в атрибутах ваших определениях фабрики. Например, если вы бы хотели создать новый экземпляр `User` при создании `Post`, можно сделать следующее:

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => factory(App\User::class),
        ];
    });

Можно использовать уже ранее определённые в фабрике аттрибуты, передавая их в качестве аргумента в анонимную функцию для создания следующего аттрибута:

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => factory(App\User::class),
            'user_type' => function (array $post) {
                return App\User::find($post['user_id'])->type;
            },
        ];
    });

<a name="using-seeds"></a>
## Использование сидов

Если вы используете [сидирование БД](/docs/{{version}}/seeding) для занесения в БД первоначальных данных для тестирования, вы можете использовать в тестах метод `seed`. По умолчанию выполняются все сиды, но вы можете указать конкретный в его аргументе:

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use OrderStatusesTableSeeder;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * Test creating a new order.
         *
         * @return void
         */
        public function testCreatingANewOrder()
        {
            // Run the DatabaseSeeder...
            $this->seed();

            // Run a single seeder...
            $this->seed(OrderStatusesTableSeeder::class);

            // ...
        }
    }


<a name="available-assertions"></a>
## Доступные методы PHPUnit

Laravel предоставляет следующие методы-утверждения для ваших фич-тестов [PHPUnit](https://phpunit.de/):

Метод  | Описание
------------- | -------------
`$this->assertDatabaseHas($table, array $data);`  |  Таблица в БД содержит заданные данные.
`$this->assertDatabaseMissing($table, array $data);`  |  Таблица в БД не содержит заданные данные.
`$this->assertDeleted($table, array $data);`  |  Запись в таблице удалена.
`$this->assertSoftDeleted($table, array $data);`  |  Заданная запись была мягко удалена.

Для удобства в методы `assertDeleted` и `assertSoftDeleted` можно передать модель, по существованию первичного ключа в БД тест сделает вывод, удалена модель или нет:

    public function testDatabase()
    {
        $user = factory(App\User::class)->create();

        // Make call to application...

        $this->assertDeleted($user);
    }