git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Базы данных: страничный вывод

- [Введение](#introduction)
- [Основы использования](#basic-usage)
    - [Страничный вывод выборки из БД](#paginating-query-builder-results)
    - [Страничный вывод запросов Eloquent](#paginating-eloquent-results)
    - [Ручное создание экземпляра страничного вывода](#manually-creating-a-paginator)
- [Отображение результатов страничного вывода](#displaying-pagination-results)
    - [Преобразование результатов в JSON](#converting-results-to-json)
- [Настройка шаблона страничного вывода](#customizing-the-pagination-view)
- [Методы экземпляра страничного вывода](#paginator-instance-methods)

<a name="introduction"></a>
## Введение

В некоторых фреймворках страничный вывод может быть большой проблемой. Страничный вывод в Laravel интегрирован с [построителем запросов](/docs/{{version}}/queries) и [Eloquent ORM](/docs/{{version}}/eloquent) и обеспечивает удобный, простой в использовании вывод результатов БД. Генерируемый HTML-код совместим с [CSS-фреймворком Bootstrap](https://getbootstrap.com/).

<a name="basic-usage"></a>
## Основы использования

<a name="paginating-query-builder-results"></a>
### Страничный вывод выборки из БД

Есть несколько способов разделения данных на страницы. Самый простой — используя метод `paginate` в [конструкторе запросов](/docs/{{version}}/queries) или в [запросе Eloquent](/docs/{{version}}/eloquent). Метод `paginate` автоматически позаботится о задании верных LIMIT и OFFSET в sql-запросе на основе текущей просматриваемой пользователем страницы. По умолчанию текущая страница определяется по значению аргумента `page` в HTTP-запросе. Конечно, Laravel и автоматически определяет это значение, и так же автоматически вставляет его в ссылки, генерируемые для страничного вывода.

В этом примере единственный аргумент метода `paginate` — число элементов на одной странице. Давайте укажем, что мы хотим выводить по `15` элементов на страницу:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Вывести всех пользователей приложения.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->paginate(15);

            return view('user.index', ['users' => $users]);
        }
    }

> {note} На данный момент операции страничного вывода, которые используют оператор `groupBy`, не могут эффективно выполняться в Laravel. Если вам необходимо использовать `groupBy` для постраничного набора результатов, рекомендуется делать запрос в БД и создавать экземпляр страничного вывода вручную.

#### "Простой страничный вывод"

Если вам необходимо вывести для страничного шаблона только ссылки "Далее" и "Назад", вы можете использовать метод `simplePaginate` для более эффективных запросов. Для больших наборов данных очень полезно, когда вам не надо отображать номер каждой страницы в вашем шаблоне:

    $users = DB::table('users')->simplePaginate(15);

<a name="paginating-eloquent-results"></a>
### Страничный вывод запросов Eloquent

Можно также делать постраничный вывод запросов [Eloquent](/docs/{{version}}/eloquent). В этом примере мы разобьём на страницы модель `User` по `15` элементов на странице. Как видите, синтаксис практически совпадает со страничным выводом выборки из БД:

    $users = App\User::paginate(15);

Разумеется, вы можете вызвать `paginate` после задания других условий запроса, таких как `where`:

    $users = User::where('votes', '>', 100)->paginate(15);

Также вы можете использовать метод `simplePaginate` моделей Eloquent:

    $users = User::where('votes', '>', 100)->simplePaginate(15);

<a name="manually-creating-a-paginator"></a>
### Создание своего класса страничного вывода

Иногда необходимо создать экземпляр страничного вывода вручную, передав ему массив данных. Это можно сделать создав либо экземпляр `Illuminate\Pagination\Paginator`, либо экземпляр `Illuminate\Pagination\LengthAwarePaginator`, в зависимости от ситуации.

Классу `Paginator` не надо знать общее количество элементов в конечном наборе, и поэтому класс не должен иметь методы для получения индекса последней страницы. `LengthAwarePaginator` принимает почти те же аргументы, как и `Paginator`; однако, ему требуется общее количество элементов в конечном наборе.

Другими словами, `Paginator` соответствует методу `simplePaginate` в конструкторе запросов и Eloquent, а `LengthAwarePaginator` соответствует методу `paginate`.

> {note} При ручном создании класса страничного вывода вы должны вручную "поделить" передаваемый в него массив результатов. Если вы не знаете, как это сделать, используйте PHP-функцию [array_slice](https://secure.php.net/manual/en/function.array-slice.php).

<a name="displaying-pagination-results"></a>
## Отображение результатов страничного вывода

При вызове метода `paginate`, вы получите экземпляр `Illuminate\Pagination\LengthAwarePaginator`. При вызове метода `simplePaginate`, вы получите экземпляр `Illuminate\Pagination\Paginator`. Эти объекты предоставляют несколько методов для вывода конечного набора. В дополнение к этим хелперам экземпляры страничного вывода — итераторы, к ним можно обращаться как к массивам. Итак, когда вы получили результаты, вы можете вывести их и создать ссылки на страницы с помощью [Blade](/docs/{{version}}/blade):

    <div class="container">
        @foreach ($users as $user)
            {{ $user->name }}
        @endforeach
    </div>

    {{ $users->links() }}

Метод `links` выведет ссылки на остальные страницы в конечном наборе. Каждая из этих ссылок уже будет содержать правильную переменную строки запроса `page`. Помните, сгенерированный методом `links` HTML-код совместим с [CSS-фреймворком Bootstrap](https://getbootstrap.com).

#### Настройка URI для вывода ссылок

Метод `withPath` позволяет настроить URI для вывода ссылок. Например, если вы хотите получить ссылки вида `http://example.com/custom/url?page=N`, вам надо передать `custom/url` в метод `withPath`:

    Route::get('users', function () {
        $users = App\User::paginate(15);

        $users->withPath('custom/url');

        //
    });

#### Параметры в ссылках

Вы можете добавить параметры запросов к ссылкам страниц с помощью метода `appends`. Например, чтобы добавить `sort=votes` к каждой страничной ссылке, вам надо вызвать `appends` вот так:

    {{ $users->appends(['sort' => 'votes'])->links() }}

Если вы хотите добавить "хэш-фрагмент" в URL-адреса страничного вывода, вы можете использовать метод `fragment`. Например, чтобы добавить `#foo` to к каждой страничной ссылке, вам надо вызвать метод `fragment`:

    {{ $users->fragment('foo')->links() }}

<a name="converting-results-to-json"></a>
### Преобразование результатов в JSON

Классы страничного вывода Laravel реализуют контракт интерфейса `Illuminate\Contracts\Support\Jsonable` и предоставляют метод `toJson`, поэтому можно очень легко конвертировать ваш страничный вывод в JSON. Вы также можете преобразовать экземпляр страничного вывода в JSON, просто вернув его из роута или метода контроллера:

    Route::get('users', function () {
        return App\User::paginate();
    });

JSON будет включать некоторые "мета-данные", такие как `total`, `current_page`, `last_page` и другие. Данные экземпляра будут доступны через ключ `data` в массиве JSON. Вот пример JSON, созданного при помощи возврата экземпляра страничного вывода из роута:

    {
       "total": 50,
       "per_page": 15,
       "current_page": 1,
       "last_page": 4,
       "next_page_url": "http://laravel.app?page=2",
       "prev_page_url": null,
       "from": 1,
       "to": 15,
       "data":[
            {
                // Объект вывода
            },
            {
                // Объект вывода
            }
       ]
    }

<a name="customizing-the-pagination-view"></a>
## Настройка шаблона страничного вывода

По умолчанию отрисованные шаблона для отображения ссылок страничного вывода совместимы с CSS-фреймворком Bootstrap. Но если вы не используете Bootstrap, вы можете определить свои собственные шаблона для отрисовки этих ссылок. При вызове метода `links` на экземпляре страничного вывода передайте первым аргументом имя шаблона:

    {{ $paginator->links('view.name') }}

    // Passing data to the view...
    {{ $paginator->links('view.name', ['foo' => 'bar']) }}

Но самый простой способ изменить шаблоны страничного вывода — экспортировать их в ваш каталог `resources/views/vendor` с помощью команды `vendor:publish`:

    php artisan vendor:publish --tag=laravel-pagination

Эта команда поместит шалоны в директорию `resources/views/vendor/pagination`. Файл `default.blade.php` в этой директории отвечает за шаблон страничного вывода по-умолчанию. Просто отредактируйте этот файл, чтобы изменить HTML страничного вывода.

<a name="paginator-instance-methods"></a>
## Методы экземпляра страничного вывода

Каждый экземпляр страничного вывода предоставляет дополнительную информацию с помощью этих методов:

- `$results->count()`
- `$results->currentPage()`
- `$results->firstItem()`
- `$results->hasMorePages()`
- `$results->lastItem()`
- `$results->lastPage() (недоступен при использованииsimplePaginate)`
- `$results->nextPageUrl()`
- `$results->perPage()`
- `$results->previousPageUrl()`
- `$results->total() (недоступен при использовании simplePaginate)`
- `$results->url($page)`
