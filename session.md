git a0bd8e684a14f4ef9699392b1a6bc847619f71e5

---

# Сессии

- [Введение](#introduction)
- [Использование сессий](#basic-usage)
    - [Одноразовые flash-данные](#flash-data)
- [Свои драйверы сессии](#adding-custom-session-drivers)

<a name="introduction"></a>
## Введение

Протокол HTTP не имеет средств для фиксации своего состояния. Сессии - способ сохранения информации (например, ID залогиненного пользователя) между отдельными HTTP-запросами. Laravel поставляется со множеством различных механизмов сессий, доступных через единое API. Изначально существует поддержка таких систем, как [Memcached](http://memcached.org), [Redis](http://redis.io) и СУБД.
Фактически, сессия - это хранилище данных типа ключ-значение.

### Настройка

Конфиг сессий находится в файле `config/session.php`. Он достаточно хорошо документирован, рекомендуется ознакомиться с ним - многое сразу станет понятно без лишних слов.

По умолчанию Laravel сконфигурирован с драйвером сессий `file`, так как этот способ работает практически везде. Но для повышения производительности вашего приложения рекомендуется использовать драйвера `memcached` или `redis`.

Драйвер сессии определяет, где будут хранится данные сессии между зепросами. Доступные драйверы:

<div class="content-list" markdown="1">
- `file` - сессии хранятся в файлах в папке `storage/framework/sessions`.
- `cookie` - Сессии хранятся в куках в браузере пользователя. Разумеется, в зашифрованном виде.
- `database` - сессии хранятся в базе данных.
- `memcached` / `redis` - сессии хранятся в быстрых хранилищах ключ/значение, работающих в памяти.
- `array` - сессии хранятся в php-массиве и не сохраняются между запросами.
</div>

> **Примечания:** Драйвер `array` обячно используется для [тестирования](/docs/{{version}}/testing) приложений.

### Настройка драйверов сессий

#### Database

При использовании драйвера `database` вам нужно создать таблицу, которая будет содержать данные сессий. Ниже пример такого объявления с помощью конструктора таблиц (`Schema`):

    Schema::create('sessions', function ($table) {
        $table->string('id')->unique();
        $table->integer('user_id')->nullable();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity');
    });


Artisan-команда `session:table` создаст миграцию для создания этой таблицы:

    php artisan session:table

    composer dump-autoload

    php artisan migrate

#### Redis

Для использования Redis вам нужно установить пакет `predis/predis` (~1.0) через Composer. 

### Замечания

Не используйте в сессии ключ `flash` - он зарезервирован за Laravel для хранения одноразовых flash-данных.

Если вы хотите, чтобы данные сессий были зашифрованы, установите в конфиге параметр `encrypt` в `true`.

<a name="basic-usage"></a>
## Использование сессий

#### Получение данных из сессии

К данным сессии можно получить доступ через класс HTTP-запроса, который [хинтится](/docs/{{version}}/container) (type-hint) в аргументах метода контроллера:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function showProfile(Request $request, $id)
        {
            $value = $request->session()->get('key');

            //
        }
    }

При получении значении ключа, хранимого в сессии, мы можем указать вторым аргументом дефолтное значение, в виде строки или результата выполнения функции:

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function() {
        return 'default';
    });

Так можно получить все данные, которые хранятся в сессии:

    $data = $request->session()->all();

Помимо работы чере класс `Request` можно пользоваться функцией-хелпером `session()`:

    Route::get('home', function () {
        // Получение данных по ключу
        $value = session('key');

        // Сохранение данных в сессии
        session(['key' => 'value']);
    });

#### Проверка на существование ключа

Можно проверить, хранится ли в сессии заданный ключ (метод возвратит true, если данные с таким ключом есть):

    if ($request->session()->has('users')) {
        //
    }    

#### Сохранение данных в сессии

    $request->session()->put('key', 'value');

#### Pushing To Array Session Values

The `push` method may be used to push a new value onto a session value that is an array. For example, if the `user.teams` key contains an array of team names, you may push a new value onto the array like so:

Если в сессии под ключом `'user.teams'` хранится массив, следующий код добавит в него элемент со значением `'developers'`:

    $request->session()->push('user.teams', 'developers');

#### Получение значения с удалением в сессии

Метод `pull` возвращает значение по ключу и удаляет этот ключ из сессии:

    $value = $request->session()->pull('key', 'default');

#### Удаление данных из сессии

Можно удалить один ключ:

    $request->session()->forget('key');

Можно удалить все данные из сессии:

    $request->session()->flush();

#### Регенерация Session ID

Для присвоения сессии нового идентификатора используется метод `regenerate`:

    $request->session()->regenerate();

<a name="flash-data"></a>
### Одноразовые flash-данные

Иногда вам нужно сохранить переменную только для следующего запроса, после выполнения которого она должна быть автоматически удалена. Это нужно, например, для передачи ошибок валидации в форму - при сабмите формы выполняется POST-запрос, после которого обязательно должен идти редирект, и для сохранения текста ошибки используют сессии. 
Вы можете сделать это методом `flash()`:

    $request->session()->flash('status', 'Task was successful!');

Если вам понадобилось передать flash-данные дальше, вы можете сохранить их все для следующего запроса:

    $request->session()->reflash();

Или вы можете указать только конкретные ключи, которые надо сохранить:    

    $request->session()->keep(['username', 'email']);

<a name="adding-custom-session-drivers"></a>
## Свои драйверы сессии

Для создания своих драйверов сессий воспользуйтесь методом `extend` [фасада](/docs/{{version}}/facades) `Session`. Вызывать его следует в методе `boot()` одного из ваших [сервис-провайдеров](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Session;
    use App\Extensions\MongoSessionStore;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Session::extend('mongo', function($app) {
                // Return implementation of SessionHandlerInterface...
                return new MongoSessionStore;
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Ваш драйвер сессии должен вернуть класс, имплементирующий `SessionHandlerInterface`. Например, если мы пишем драйвер сессии для MongoDB, этот класс будет выглядеть так: 

    <?php

    namespace App\Extensions;

    class MongoHandler implements SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }

Кратное описение методов:

<div class="content-list" markdown="1">
- The `open` method would typically be used in file based session store systems. Since Laravel ships with a `file` session driver, you will almost never need to put anything in this method. You can leave it as an empty stub. It is simply a fact of poor interface design (which we'll discuss later) that PHP requires us to implement this method.
- The `close` method, like the `open` method, can also usually be disregarded. For most drivers, it is not needed.
- The `read` method should return the string version of the session data associated with the given `$sessionId`. There is no need to do any serialization or other encoding when retrieving or storing session data in your driver, as Laravel will perform the serialization for you.
- The `write` method should write the given `$data` string associated with the `$sessionId` to some persistent storage system, such as MongoDB, Dynamo, etc.
- The `destroy` method should remove the data associated with the `$sessionId` from persistent storage.
- The `gc` method should destroy all session data that is older than the given `$lifetime`, which is a UNIX timestamp. For self-expiring systems like Memcached and Redis, this method may be left empty.
</div>

Once the session driver has been registered, you may use the `mongo` driver in your `config/session.php` configuration file.