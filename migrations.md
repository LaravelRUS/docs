git fb85738c37ef8da3e0b3ed2c6e059b94d3d0dbac

---

# Миграции и начальные данные

- [Введение](#introduction)
- [Создание миграций](#creating-migrations)
- [Применение миграций](#running-migrations)
- [Откат миграций](#rolling-back-migrations)
- [Загрузка начальных данных в БД](#database-seeding)

<a name="introduction"></a>
## Введение

Миграции - это что-то вроде системы контроля версий для вашей базы данных. Они позволяют команде программистов изменять её структуру, в то же время оставаясь в курсе изменений других участников. Миграции обычно идут рука об руку с [конструктором таблиц](/docs/schema) для более простого обращения с архитектурой вашего приложения.

<a name="creating-migrations"></a>
## Создание миграций

Для создания новой миграции вы можете использовать команду `migrate:make` командного интерфейса Artisan ("артизан-команда").

	php artisan migrate:make create_users_table

Миграция будет помещена в папку `app/database/migrations` и будет содержать текущее время, которое позволяет библиотеке определять порядок применения миграций.

> **Примечание:** Старайтесь давать миграциям многословные имена - например, не `comments`, а `create_comments_table` - так вы избежите возможного конфликта названий классов.

При создании миграции вы можете также передать параметр `--path`. Путь должен быть относительным к папке вашей установки Laravel.

	php artisan migrate:make foo --path=app/migrations

Можно также использовать параметры `--table` и `--create` для указания имени таблицы и того факта, что миграция будет создавать новую таблицу, а не изменять существующую.

	php artisan migrate:make create_users_table --table=users --create

<a name="running-migrations"></a>
## Применение миграций

#### Накатывание всех новых неприменённых миграций

	php artisan migrate

#### Накатывание новых миграций, расположенных в указанной папке

	php artisan migrate --path=app/foo/migrations


#### Накатывание новых миграций для пакета

	php artisan migrate --package=vendor/package

> **Внимание:** если при применении миграций вы сталкиваетесь с ошибкой "class not found" ("Класс не найден") - попробуйте выполнить команду `composer dump-autoload`.

<a name="rolling-back-migrations"></a>
## Откат миграций

#### Отмена изменений последней миграции

	php artisan migrate:rollback

#### Отмена изменений всех миграций

	php artisan migrate:reset

#### Откат всех миграций и их повторное применение

	php artisan migrate:refresh

	php artisan migrate:refresh --seed

<a name="database-seeding"></a>
## Загрузка начальных данных в БД

Кроме миграций, описанных выше, Laravel также включает в себя механизм наполнения вашей БД начальными данными (seeding) с помощью специальных классов. Все такие классы хранятся в `app/database/seeds`. Они могут иметь любое имя, но вам, вероятно, следует придерживаться какой-то логики в их именовании - например, `UserTableSeeder`и т.д. По умолчанию для вас уже определён класс `DatabaseSeeder`. Из этого класса вы можете вызывать метод `call` для подключения других классов с данными, что позволит вам контролировать порядок их выполнения.

#### Примерные классы для загрузки начальных данных

	class DatabaseSeeder extends Seeder {

		public function run()
		{
			$this->call('UserTableSeeder');

			$this->command->info('Таблица пользователей заполнена данными!');
		}

	}

	class UserTableSeeder extends Seeder {

		public function run()
		{
			DB::table('users')->delete();

			User::create(array('email' => 'foo@bar.com'));
		}

	}

Для добавления данных в БД используйте артизан-команду `db:seed`:

	php artisan db:seed

По умолчанию команда `db:seed` запускает метод `run()` класса `DatabaseSeeder`. В этом методе вы можете вызывать другие ваши сидеры. Или, вы можете задать название класса, который будет вызван вместо дефолтного:

	php artisan db:seed --class=UserTableSeeder

Вы также можете заполнить БД первоначальными данными командой `migrate:refresh`, которая перед этим откатит и заново применит все ваши миграции:

	php artisan migrate:refresh --seed