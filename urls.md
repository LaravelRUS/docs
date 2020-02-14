git b1b37c94754d45a6c70acb955b387f305ec8927e

---

# Генерация URL

- [Введение](#introduction)
- [Основы](#the-basics)
    - [Генерация URL-адреса](#generating-basic-urls)
    - [Доступ к текущему URL](#accessing-the-current-url)
- [URL-адреса для именованных маршрутов](#urls-for-named-routes)
    - [Подписанные URL-адреса](#signed-urls)
- [URL-адреса для экшенов контроллеров](#urls-for-controller-actions)
- [Значения по умолчанию](#default-values)

<a name="introduction"></a>
## Введение

Laravel поставляется с несколькими вспомогательными функциями (хелперами), которые помогут в построении URL для вашего приложения. Они в основном полезны когда необходимо сгенерировать ссылки в ваших шаблонах и API ответах или для отправки ответа для перенаправления на другую страницу вашего приложения.

<a name="the-basics"></a>
## Основы

<a name="generating-basic-urls"></a>
### Генерация URL-адреса

Вспомогательная функция `url` может быть использована для построения любых URL для вашего приложения. Для генерации URL адреса будет использован протокол (HTTP or HTTPS) и хост из текущего запроса:

    $post = App\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

<a name="accessing-the-current-url"></a>
### Доступ к текущему URL

При вызове функции `url` без указания пути (первого аргумента) будет возвращен объект `Illuminate\Routing\UrlGenerator`, с помощью которого вы сможете получить доступ к информации о текущем URL:

    // Получение текущего URL без query string...
    echo url()->current();

    // Получение текущего URL включая query string...
    echo url()->full();

    // Получение полного URL для предыдущего запроса..
    echo url()->previous();

Получить доступ к каждому из этих методов можно также через `URL` [facade](/docs/{{version}}/facades):

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>
## URL-адреса для именованных маршрутов

Вспомогательная функция `route` будет полезна для генерация URL для именованных маршрутов. Именованные маршруты позволяют создавать URL-адреса без привязки к фактическому URL-адресу, определенному в маршруте. Поэтому, если изменится URL адрес в определении маршрута, нет необходимости вносить изменения в вызываемой функции `route`. Например, допустим ваше приложение содержит следующий маршрут: 

    Route::get('/post/{post}', function () {
        //
    })->name('post.show');

Для генерации URL адреса для этого маршрута можно использовать функцию `route` таким образом:

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

Вам часто придется генерировать URL-адреса используя первичный ключ модели [Eloquent models](/docs/{{version}}/eloquent). Поэтому можно в качестве аргумента передавать объект Eloquent модели, функция `route` автоматически извлечет первичный ключи из объекта:

    echo route('post.show', ['post' => $post]);

Также в функцию `route` может быть передано несколько аргументов:

    Route::get('/post/{post}/comment/{comment}', function () {
        //
    })->name('comment.show');

    echo route('comment.show', ['post' => 1, 'comment' => 3]);

    // http://example.com/post/1/comment/3

<a name="signed-urls"></a>
### Подписанные URL-адреса

Laravel позволит вам с легкостью создавать "подписанные" URL-адреса для именованных маршрутов. В эти адреса будет добавлен "подпись" хэш в виде query string параметра, который позволит Laravel приложению проверить, что URL адрес не был изменен с момента его генерации. Подписанные URL-адреса особенно полезны для публичных маршрутов, которые необходимо защитить от изменения.

Например, вы можете использовать подписанный URL адрес для генерации ссылок "отписки", который отправляются клиентам по электронной почте. Для создания подписанного URL адреса используйте метод `signedRoute` из `URL` фасада:

    use Illuminate\Support\Facades\URL;

    return URL::signedRoute('unsubscribe', ['user' => 1]);

Если вы хотите сгенерировать временную ссылку, которая должна быть действительно некоторое время, вы можете использовать метод `temporarySignedRoute`:

    use Illuminate\Support\Facades\URL;

    return URL::temporarySignedRoute(
        'unsubscribe', now()->addMinutes(30), ['user' => 1]
    );

#### Валидация запроса с подписью

Для проверки, что входящий запрос имеет валидную подпись, вы должны использовать метод `hasValidSignature` входящего запроса `Request`:

    use Illuminate\Http\Request;

    Route::get('/unsubscribe/{user}', function (Request $request) {
        if (! $request->hasValidSignature()) {
            abort(401);
        }

        // ...
    })->name('unsubscribe');

Альтернативный вариант, вы можете использовать middleware `Illuminate\Routing\Middleware\ValidateSignature`, который можно назначить определенному маршруту. Если его еще нет, вам следует назначить этому middleware ключ в массиве `routeMiddleware` в `Http\Kernel.php`:

    /**
     * The application's route middleware.
     *
     * These middleware may be assigned to groups or used individually.
     *
     * @var array
     */
    protected $routeMiddleware = [
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    ];

Как только middleware был зарегистрирован в `Http\Kernel.php`, вы можете применять его к маршрутам. Если входящий запрос не содержит валидной подписи, middleware автоматически вернет ответ с кодом `403`:

    Route::post('/unsubscribe/{user}', function (Request $request) {
        // ...
    })->name('unsubscribe')->middleware('signed');

<a name="urls-for-controller-actions"></a>
## URL-адреса для экшенов контроллеров

Вспомогательная функция `action` позволяет генерировать URL-адреса для экшеном контроллера. Вам не нужно передавать полный путь до контроллера (Namespace), вместо это вы можете передавать только название контроллера относительно пространства имен `App\Http\Controllers`:

    $url = action('HomeController@index');

Также вы можете ссылаться на экшн с помощью "callable" синтаксиса:

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

Если метод экшена ожидает аргументы функции, вы можете передать их вторых аргументом функции:

    $url = action('UserController@profile', ['id' => 1]);

<a name="default-values"></a>
## Значения по умолчанию

Для некоторых приложений возможно вы захотите определить значения по умолчанию для определенных параметров URL адреса. Например, допустим ваш маршрут имеет параметр `{locale}`:

    Route::get('/{locale}/posts', function () {
        //
    })->name('post.index');

Передавать каждый раз параметр `locale` при вызове функции `route` накладно. Поэтому вы можете использовать метод `URL::defaults` для определения значений по умолчанию, которые всегда будут устанавливаться для текущего запроса. Данный метод вы можете вызывать через [route middleware](/docs/{{version}}/middleware#assigning-middleware-to-routes), чтобы получить доступ к объекту текущего запроса:

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Support\Facades\URL;

    class SetDefaultLocaleForUrls
    {
        public function handle($request, Closure $next)
        {
            URL::defaults(['locale' => $request->user()->locale]);

            return $next($request);
        }
    }

После того, как значение по умолчанию для параметра `locale` было установлено, больше нет необходимости передавать его при генерации URL-адресов с помощью функции `route`.
