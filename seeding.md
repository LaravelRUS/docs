git fb1593603bebc170f83a5ef3358a4c31465d3f1c

---

# База данных: наполнение данными (seeding)

- [Введение](#introduction)
- [Создание наполнителей](#writing-seeders)
    - [Использование фабрики](#using-model-factories)
    - [Вызов дополнительных наполнителей](#calling-additional-seeders)
- [Запуск наполнителей](#running-seeders)

<a name="introduction"></a>
## Введение

Laravel имеет простой метод заполнения базы данных тестовыми данными, используя классы-наполнители(seed classes). Эти классы хранятся в `database/seeds`. Можно использовать любое имя для названия класса-наполнителя, но разумным будет применять имена, подобные `UsersTableSeeder`. По умолчанию класс `DatabaseSeeder` уже создан в папке наполнителей. В этом классе вы можете использовать метод `call` для запуска других наполнителей, что позволяет вам контролировать порядок наполнения.

<a name="writing-seeders"></a>
## Создание наполнителей

Для создания наполнителя можно использовать команду `make:seeder` [Artisan command](/docs/{{version}}/artisan). Команда создает наполнитель в папке `database/seeds`:

    php artisan make:seeder UsersTableSeeder

По умолчанию класс-наполнитель содержит только один метод: `run`. Метод вызывается, когда запускается команда `db:seed` [Artisan command](/docs/{{version}}/artisan). В методе `run` вы можете вставлять данные в БД любым удобным способом. Вы можете использовать [query builder](/docs/{{version}}/queries) для ручной вставки или использовать [Eloquent model factories](/docs/{{version}}/testing#model-factories).

Для примера давайте изменим создаваемый по умолчанию класс `DatabaseSeeder`. Добавим выражение вставки в метод `run`:

    <?php

    use Illuminate\Database\Seeder;
    use Illuminate\Database\Eloquent\Model;

    class DatabaseSeeder extends Seeder
    {
        /**
         * Run the database seeds.
         *
         * @return void
         */
        public function run()
        {
            DB::table('users')->insert([
                'name' => str_random(10),
                'email' => str_random(10).'@gmail.com',
                'password' => bcrypt('secret'),
            ]);
        }
    }

<a name="using-model-factories"></a>
### Использование фабрики

Несомненно, ручное определение атрибутов для каждой модели-наполнителя утомительно. Вместо этого вы можете использовать [model factories](/docs/{{version}}/testing#model-factories) для создания
большого числа записей в БД. Первым делом изучите, как нужно определять фабрики [model factory documentation](/docs/{{version}}/testing#model-factories). После определения вы можете использовать функцию-помощник `factory` для вставки записей в БД.

Например, создадим 50 пользователей и добавим связь для каждого пользователя:

    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        factory(App\User::class, 50)->create()->each(function($u) {
            $u->posts()->save(factory(App\Post::class)->make());
        });
    }

<a name="calling-additional-seeders"></a>
### Вызов дополнительных наполнителей

В классе `DatabaseSeeder` можно использовать метод `call` для запуска дополнительных классов-наполнителей. Метод `call` позволить делать наполнение БД посредством множества файлов, не создавая один большой класс-наполнитель. Просто передай методу желаемый класс-наполнитель для его запуска:

    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        Model::unguard();

        $this->call(UsersTableSeeder::class);
        $this->call(PostsTableSeeder::class);
        $this->call(CommentsTableSeeder::class);
        
        Model::reguard();
    }

<a name="running-seeders"></a>
## Запуск наполнителей

После написания классов-наполнителей используйте команду Artisan команду `db:seed` для заполнения БД. По умолчанию она выполнит класс `DatabaseSeeder`, который может быть использован для вызова других  классов-наполнителей. Однако, можно использовать опцию `--class` для указания другого класса-наполнителя:

    php artisan db:seed

    php artisan db:seed --class=UsersTableSeeder

Также можно заполнить БД посредством команды `migrate:refresh`, которая сделает откат изменений и применит все миграции заново. Эта команда полезна для полного перестроения БД:

    php artisan migrate:refresh --seed
