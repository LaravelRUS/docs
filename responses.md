git 1255c35731dd6cc815bd261fb5883f12eabd7816

---

# HTTP-отклики

- [Создание откликов](#creating-responses)
    - [Добавление заголовков в отклики](#attaching-headers-to-responses)
    - [Добавление cookie в отклики](#attaching-cookies-to-responses)
    - [Cookies и шифрование](#cookies-and-encryption)
- [Редиректы](#redirects)
    - [Редиректы на именованные роуты](#redirecting-named-routes)
    - [Редиректы на действие контроллера](#redirecting-controller-actions)
    - [Редиректы с одноразовыми переменными сессии](#redirecting-with-flashed-session-data)
- [Другие типы откликов](#other-response-types)
    - [Отклики шаблона](#view-responses)
    - [Отклики JSON](#json-responses)
    - [Скачивания файла](#file-downloads)
    - [Отклики файла](#file-responses)
- [Макрос отклика](#response-macros)

<a name="creating-responses"></a>
## Создание откликов (response)

#### Строки и массивы

Все роуты и контроллеры должны возвращать отклик для отправки обратно в браузер. Laravel предоставляет несколько разных способов для возврата откликов. Самый основной отклик — простой возврат строки из роута или контроллера. Фреймворк автоматически конвертирует строку в полный HTTP-отклик:

    Route::get('/', function () {
        return 'Hello World';
    });

Помимо строк из роутов и контроллеров можно также возвращать массивы. Фреймворк автоматически конвертирует массив в JSON-отклик:

    Route::get('/', function () {
        return [1, 2, 3];
    });

> {tip} Вы знали, что вы можете также возвращать [коллекции Eloquent](/docs/{{version}}/eloquent-collections) из роутов и контроллеров? Они будут автоматически сконвертированы в JSON. Попробуйте!

#### Объекты откликов

Чаще всего вы будете возвращать из действий роутов не просто строки или массивы. Вместо этого вы будете возвращать полные экземпляры `Illuminate\Http\Response` или [шаблоны](/docs/{{version}}/views).

Возврат полного экземпляра `Response` позволяет вам изменять HTTP-код состояния и заголовки откликов. Экземпляр `Response` наследует класс `Symfony\Component\HttpFoundation\Response`, который предоставляет множество методов для создания HTTP-откликов:

    Route::get('home', function () {
        return response('Hello World', 200)
                      ->header('Content-Type', 'text/plain');
    });

<a name="attaching-headers-to-responses"></a>
#### Добавление заголовков в отклики

Имейте ввиду, что большинство методов сцепляемы, что делает создание откликов более гибким. Например, вы можете использовать метод `header` для добавления нескольких заголовков в отклик перед его отправкой пользователю:

    return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');

Или, вы можете использовать метод `withHeaders` для добавления массива заголовков в отклик:

    return response($content)
                ->withHeaders([
                    'Content-Type' => $type,
                    'X-Header-One' => 'Header Value',
                    'X-Header-Two' => 'Header Value',
                ]);

<a name="attaching-cookies-to-responses"></a>
#### Добавление cookie в отклики

Использование метода `cookie` на экземплярах отклика позволяет вам легко добавить cookie в отклик. Например, вы можете использовать метод `cookie` для создания cookie и гибкого добавления его в экземпляр отклика:

    return response($content)
                    ->header('Content-Type', $type)
                    ->cookie('name', 'value', $minutes);

Метод `cookie` также принимает ещё несколько аргументов, используемых реже. В целом эти аргументы имеют такое же назначение и значение, как аргументы, передаваемые в PHP-метод [setcookie](https://secure.php.net/manual/en/function.setcookie.php):

    ->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)

<a name="cookies-and-encryption"></a>
#### Cookies и шифрование

По умолчанию все генерируемые cookie в Laravel шифруются и подписываются, поэтому они не могут быть прочитаны или изменены клиентом. Если вы хотите отключить шифрование для определённого набора cookie в вашем приложении, вы можете использовать свойство `$except` посредника `App\Http\Middleware\EncryptCookies`, который находится в директории `app/Http/Middleware`:

    /**
     * Названия тех cookie, которые не надо шифровать.
     *
     * @var array
     */
    protected $except = [
        'cookie_name',
    ];

<a name="redirects"></a>
## Редиректы

Отклики для редиректа — это объекты класса  `Illuminate\Http\RedirectResponse`, и они содержат надлежащие заголовки, необходимые для переадресации пользователя на другой URL. Есть несколько способов создания экземпляра `RedirectResponse`. Простейший из них — использовать глобальный вспомогательный хелпер `redirect`:

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

Если вы захотите переадресовать пользователя на предыдущую страницу (например, после отправки формы с некорректными данными), используйте глобальный хелпер `back`. Поскольку для этого используются [сессии](/docs/{{version}}/session), роут, вызывающий метод `back` должен использовать группу посредников `web` или должен применять всех посредников сессий:

    Route::post('user/profile', function () {
        // Validate the request...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
### Редиректы на именованные роуты

Когда вы вызываете хелпер `redirect` без параметров, возвращается экземпляр `Illuminate\Routing\Redirector`, позволяя вам вызывать любой метод объекта `Redirector`. Например, для создания `RedirectResponse` на именованный роут вы можете использовать метод `route`:

    return redirect()->route('login');

Если у вашего роута есть параметры, вы можете передать их в качестве второго аргумента в метод `route`:

    // Для роута с таким URI: profile/{id}

    return redirect()->route('profile', ['id' => 1]);

#### Заполнение параметров через модели Eloquent

Если вы переадресовываете на роут с параметром "ID", который был взят из модели Eloquent, то вы можете просто передать саму модель. ID будет извлечён автоматически:

    // Для роута с таким URI: profile/{id}

    return redirect()->route('profile', [$user]);

Для изменения значения, помещённого в параметр роута, переопределите метод `getRouteKey` в вашей модели Eloquent:

    /**
     * Получить значение ключа роута модели.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
### Редиректы на действие контроллера

Также вы можете создать отклик-редирект на [действия контроллера](/docs/{{version}}/controllers). Для этого передайте контроллер и название действия в метод `action`. Не забудьте, что вам не надо указывать полное пространство имён для контроллера, так как `RouteServiceProvider` в Laravel автоматически задаст пространство имён базового контроллера:

    return redirect()->action('HomeController@index');

Если роут вашего контроллера требует параметров, вы можете передать их вторым аргументом метода `action`:

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );

<a name="redirecting-with-flashed-session-data"></a>
### Редиректы с одноразовыми переменными сессии

Редирект на новый URL и [передача данных в сессию](/docs/{{version}}/session#flash-data) обычно происходят в одно и то же время. Обычно это делается после успешного выполнения действия, когда вы передаёте сообщение об этом в сессию. Для удобства вы можете создать экземпляр `RedirectResponse` и передать данные в сессию в одной гибкой связке методов:

    Route::post('user/profile', function () {
        // Обновление профиля пользователя...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

Когда пользователь переадресован, вы можете вывести одноразовое сообщение из [сессии](/docs/{{version}}/session). Например, с помощью [синтаксиса Blade](/docs/{{version}}/blade):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif

<a name="other-response-types"></a>
## Другие типы откликов

Хелпер `response` можно использовать для создания экземпляров откликов других типов. Когда хелпер `response` вызывается без аргументов, возвращается реализация [контракта](/docs/{{version}}/contracts) `Illuminate\Contracts\Routing\ResponseFactory`. Этот контракт предоставляет несколько полезных методов для создания откликов.

<a name="view-responses"></a>
### Отклики шаблона

Если вам необходим контроль над статусом и заголовком отклика, но при этом необходимо возвращать [шаблон](/docs/{{version}}/views) в качестве содержимого отклика, используйте метод `view`:

    return response()
                ->view('hello', $data, 200)
                ->header('Content-Type', $type);

Конечно, если вам не надо передавать свой код HTTP-статуса и свои заголовки, вы можете просто использовать глобальный хелпер`view`.

<a name="json-responses"></a>
### Отклики JSON

Метод `json` автоматически задаст заголовок `Content-Type` для `application/json`, а также конвертирует данный массив в JSON при помощи PHP-функции `json_encode`:

    return response()->json([
        'name' => 'Abigail',
        'state' => 'CA'
    ]);

Если вы хотите создать JSONP-отклик, используйте метод `json` совместно с методом `withCallback`:

    return response()
                ->json(['name' => 'Abigail', 'state' => 'CA'])
                ->withCallback($request->input('callback'));

<a name="file-downloads"></a>
### Скачивание файлов

Метод `download` используется для создания отклика, получив который, браузер пользователя скачивает файл по указанному пути. Метод `download` принимает вторым аргументом имя файла, именно это имя увидит пользователь при скачивании файла. И наконец, вы можете передать массив HTTP-заголовков третьим аргументом метода:

    return response()->download($pathToFile);

    return response()->download($pathToFile, $name, $headers);
    
    return response()->download($pathToFile)->deleteFileAfterSend(true);

> {note} Класс Symfony HttpFoundation, который управляет скачиванием файлов, требует, чтобы загружаемый файл имел ASCII-имя.

<a name="file-responses"></a>
### Отклик отображения файла

Метод `file` служит для вывода файла (такого как изображение или PDF) прямо в браузере пользователя, вместо запуска его скачивания. Этот метод принимает первым аргументом путь к файлу, а вторым аргументом — массив заголовков:

    return response()->file($pathToFile);

    return response()->file($pathToFile, $headers);

<a name="response-macros"></a>
## Макрос отклика

Если вы хотите определить собственный отклик, который вы смогли бы использовать повторно в различных роутах и контроллерах, то можете использовать метод `macro` на фасаде `Response`. Например, из метода `boot` [сервис-провайдера](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Response;

    class ResponseMacroServiceProvider extends ServiceProvider
    {
        /**
         * Регистрация макроса отклика приложения.
         *
         * @return void
         */
        public function boot()
        {
            Response::macro('caps', function ($value) {
                return Response::make(strtoupper($value));
            });
        }
    }

Функция `macro` принимает имя в качестве первого аргумента и замыкание в качестве второго. Замыкание макроса будет выполнено при вызове имени макроса из реализации `ResponseFactory` или из хелпера `response`:

    return response()->caps('foo');
