git 4c0f790943685aeb4aef0a47b05817877cd943ca

---

# База данных: основы

- [Введение](#introduction)
- [SQL запросы](#running-queries)
    - [Постобработчик запросов](#listening-for-query-events)
- [Транзакции](#database-transactions)
- [Использование нескольких подключений к БД](#accessing-connections)

<a name="introduction"></a>
## Введение

Laravel делает работу с БД чрезвычайно простой благодаря возможности работы на трех уровнях: на чистом SQL, через конструктор запросов [fluent query builder](/docs/{{version}}/queries), через объектные модели [Eloquent ORM](/docs/{{version}}/eloquent). Laravel поддерживает четыре СУБД:

- MySQL
- Postgres
- SQLite
- SQL Server

<a name="configuration"></a>
### Настройка

Файл настройки БД находится в `config/database.php`. В нем можно опеределить все подключения к БД, задать подключение по умолчанию. Также в файле есть примеры для всех СУБД. Конфигурация по умолчанию готова для работы с виртуальной машиной [Laravel Homestead](/docs/{{version}}/homestead), которая удобна для разработки на локальной машине. Конечно, вы можете менять конфигурацию в файле под ваши нужды.

#### Настройка SQL Server
#### SQL Server Configuration

Laravel поддерживает SQL Server "из коробки", однако, в файл настройки требуется добавить параметры подключения:

    'sqlsrv' => [
        'driver' => 'sqlsrv',
        'host' => env('DB_HOST', 'localhost'),
        'database' => env('DB_DATABASE', 'forge'),
        'username' => env('DB_USERNAME', 'forge'),
        'password' => env('DB_PASSWORD', ''),
        'charset' => 'utf8',
        'prefix' => '',
    ],

<a name="read-write-connections"></a>
#### Подключения для чтения/записи

Если необходимо настроить раздельные подключения для чтения (SELECT) и изменения данных (INSERT, UPDATE, and DELETE),
то Laravel позваоляет это сделать на одном дыхании. Соответствующее подключение будет автоматически использоваться при работе с БД любым из способов: чистый SQL, конструктор запросов, объектные модели (Eloquent ORM)

Пример настройки раздельных подключений для чтения/записи:

    'mysql' => [
        'read' => [
            'host' => '192.168.1.1',
        ],
        'write' => [
            'host' => '196.168.1.2'
        ],
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset'   => 'utf8',
        'collation' => 'utf8_unicode_ci',
        'prefix'    => '',
    ],

Обратите внимание, что в массив настройки были добавлены два ключа: `read` and `write`. Каждый из них является массивом, в котором содержится единственный ключ: `host`. Остальные опции подключений являются общими и заданы непосредственно в массиве `mysql`.

Таким образом, нам нужно будет добавлять параметры в массивы `read` и `write`, только, если мы хотим переписать значения этих параметров в основном массиве. Так в приведенном примере хост `192.168.1.1` будет использоваться для чтения, а `192.168.1.2` для записи. Учетные данные, префикс, кодировка и все другие опции в массиве `mysql` будут общими для обоих подключений.

<a name="running-queries"></a>
## Запросы на чистом SQL

После настройки подключения к БД вы можете делать запросы, используя фасад `DB`. Фасад предоставляет методы для каждого типа запроса: `select`, `update`, `insert`, `delete`, и `statement`.

#### SQL запросы

Для простого запроса мы можем использовать метод `select` фасада `DB`:

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
            $users = DB::select('select * from users where active = ?', [1]);

            return view('user.index', ['users' => $users]);
        }
    }

Первый аргумент метода `select` - это строка, содержащая запрос на чистом SQL, второй аргумент - массив со значениями,
подставляемыми в запрос. Обычно эти значения используется в выражении `where`. Такая привязка параметров защищает от SQL инъекций.

Метод `select` всегда возвращает массив. Каждое значение в массиве будет объектом PHP `StdClass`, предоставляющим доступ к значениям результата запроса:

    foreach ($users as $user) {
        echo $user->name;
    }

#### Именованные параметры запроса

Вместо использования `?`, указывающего на привязку параметра к запросу, вы можете использовать имя:

    $results = DB::select('select * from users where id = :id', ['id' => 1]);

#### Вставка

Для вставки записей в БД используется метод `insert` фасада `DB`. Использование метода подобно методу `select`, где первый аргумент - запрос на SQL, второй - параметры:

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);

#### Обновление

Метод `update` используется для обновления записей в БД. Возвращает количество обновленных строк:

    $affected = DB::update('update users set votes = 100 where name = ?', ['John']);

#### Удаление

Метод `delete` используется для удаления записей из таблицы. Возвращает количество удаленных строк:

    $deleted = DB::delete('delete from users');

#### Иные запросы к БД

Используйте метод `statement` фасада `DB`:

    DB::statement('drop table users');

<a name="listening-for-query-events"></a>
### Постобработчик запросов

Если вы хотите получать каждый SQL запрос, выполненый приложением, используйте метод `listen`. Этот метод полезен для
логирования и отладки запросов. Регистрация постобработчика в [service provider](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use DB;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            DB::listen(function($query) {
                // $query->sql
                // $query->bindings
                // $query->time
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="database-transactions"></a>
## Транзакции

Для использования транзакций предназначен метод `transaction` фасада `DB`. Если будет брошено исключение в функции-замыкании, то транзакция будет отменена. Если транзакция успешна, то она будет автоматически завершена(committed). Вам не требуется вручную делать откат или завершение (committing) при использовании этого метода:

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    });

#### Ручные транзакции

Если требуется начать транзакцию вручную и иметь полный контроль над откатами и завершениями используйте метод `beginTransaction` фасада `DB`:

    DB::beginTransaction();

Откат транзакции:

    DB::rollBack();

Завершение транзакции:

    DB::commit();

> **Примечание:** Используйте перечисленные методы для ручного управления транзакциями при работе с 
[query builder](/docs/{{version}}/queries) и [Eloquent ORM](/docs/{{version}}/eloquent).

<a name="accessing-connections"></a>
## Использолвание нескольких подключений к БД

При использовании нескольких подключений доступ к каждому из них можно получить через метод `connection`. Методу необходимо передать имя подключения, которое должно соответствовать одному из имен в файле настройки БД `config/database.php`:

    $users = DB::connection('foo')->select(...);

Вы также можете получить низкоуровневый объект PDO для текущего подключения:

    $pdo = DB::connection()->getPdo();
