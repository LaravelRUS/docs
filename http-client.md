---
git: d6033dc56d10e954a4dad1c8a0ccddff4fa7c194
---

# HTTP-клиент

<a name="introduction"></a>
## Введение

Laravel предлагает минимальный и выразительный API для HTTP-клиента [Guzzle](http://docs.guzzlephp.org/en/stable/), позволяющий быстро выполнять исходящие запросы для взаимодействия с другими веб-приложениями. Обертка вокруг Guzzle ориентирована на наиболее распространенные варианты использования и дает прекрасные возможности для разработчиков.

Вы должны убедиться, что пакет Guzzle включен в зависимости вашего приложения. По умолчанию Laravel автоматически включает эту зависимость. Однако, если вы ранее удалили пакет, то вы можете установить его снова через Composer:

```shell
composer require guzzlehttp/guzzle
```

<a name="making-requests"></a>
## Выполнение запросов

Для отправки запросов вы можете использовать методы `head`, `get`, `post`, `put`, `patch` и `delete` фасада `Http`. Сначала давайте рассмотрим, как сделать основной запрос `GET`:

    use Illuminate\Support\Facades\Http;

    $response = Http::get('http://example.com');

Метод `get` возвращает экземпляр `Illuminate\Http\Client\Response`, содержащий методы, которые можно использовать для получения информации об ответе:

    $response->body() : string;
    $response->json($key = null, $default = null) : array|mixed;
    $response->object() : object;
    $response->collect($key = null) : Illuminate\Support\Collection;
    $response->status() : int;
    $response->ok() : bool;
    $response->successful() : bool;
    $response->redirect(): bool;
    $response->failed() : bool;
    $response->serverError() : bool;
    $response->clientError() : bool;
    $response->header($header) : string;
    $response->headers() : array;

Объект `Illuminate\Http\Client\Response` также реализует интерфейс `ArrayAccess` PHP, позволяющий напрямую получать доступ к данным ответа JSON:

    return Http::get('http://example.com/users/1')['name'];

В дополнение к методам ответа, перечисленным выше, для определения того, имеет ли ответ заданный код состояния, можно использовать следующие методы:

    $response->ok() : bool;                  // 200 OK
    $response->created() : bool;             // 201 Created
    $response->accepted() : bool;            // 202 Accepted
    $response->noContent() : bool;           // 204 No Content
    $response->movedPermanently() : bool;    // 301 Moved Permanently
    $response->found() : bool;               // 302 Found
    $response->badRequest() : bool;          // 400 Bad Request
    $response->unauthorized() : bool;        // 401 Unauthorized
    $response->paymentRequired() : bool;     // 402 Payment Required
    $response->forbidden() : bool;           // 403 Forbidden
    $response->notFound() : bool;            // 404 Not Found
    $response->requestTimeout() : bool;      // 408 Request Timeout
    $response->conflict() : bool;            // 409 Conflict
    $response->unprocessableEntity() : bool; // 422 Unprocessable Entity
    $response->tooManyRequests() : bool;     // 429 Too Many Requests
    $response->serverError() : bool;         // 500 Internal Server Error

<a name="uri-templates"></a>
#### Шаблоны URI

HTTP-клиент также позволяет вам формировать URL-запросы с использованием [спецификации шаблонов URI](https://www.rfc-editor.org/rfc/rfc6570). Для определения параметров URL, которые могут быть расширены в вашем шаблоне URI, вы можете использовать метод `withUrlParameters`:

```php
Http::withUrlParameters([
    'endpoint' => 'https://laravel.com',
    'page' => 'docs',
    'version' => '9.x',
    'topic' => 'validation',
])->get('{+endpoint}/{page}/{version}/{topic}');
```

<a name="dumping-requests"></a>
#### Вывод информации о запросах

Если вы хотите получить информацию о сформированном экземпляре исходящего запроса перед его отправкой и прекратить выполнение скрипта, вы можете добавить метод `dd` в начало определения вашего запроса:

    return Http::dd()->get('http://example.com');

<a name="request-data"></a>
### Данные запроса

При выполнении запросов `POST`, `PUT` и `PATCH` обычно отправляются дополнительные данные, поэтому эти методы принимают массив данных в качестве второго аргумента. По умолчанию данные будут отправляться с использованием типа содержимого `application/json`:

    use Illuminate\Support\Facades\Http;

    $response = Http::post('http://example.com/users', [
        'name' => 'Steve',
        'role' => 'Network Administrator',
    ]);

<a name="get-request-query-parameters"></a>
#### Параметры GET-запроса

При выполнении запросов `GET` вы можете либо напрямую добавить строку запроса к URL, либо передать массив пар ключ / значение в качестве второго аргумента метода `get`:

    $response = Http::get('http://example.com/users', [
        'name' => 'Taylor',
        'page' => 1,
    ]);

В качестве альтернативы можно использовать метод `withQueryParameters`:

    Http::retry(3, 100)->withQueryParameters([
        'name' => 'Taylor',
        'page' => 1,
    ])->get('http://example.com/users')

<a name="sending-form-url-encoded-requests"></a>
#### Отправка запросов с передачей данных в URL-кодированной строке

Если вы хотите отправлять данные с использованием типа содержимого `application/x-www-form-urlencoded`, то вы должны вызвать метод `asForm` перед выполнением запроса:

    $response = Http::asForm()->post('http://example.com/users', [
        'name' => 'Sara',
        'role' => 'Privacy Consultant',
    ]);

<a name="sending-a-raw-request-body"></a>
#### Отправка необработанного тела запроса

Вы можете использовать метод `withBody`, если хотите передать необработанное тело запроса при его выполнении. Тип контента может быть указан вторым аргументом метода:

    $response = Http::withBody(
        base64_encode($photo), 'image/jpeg'
    )->post('http://example.com/photo');

<a name="multi-part-requests"></a>
#### Составные запросы

Если вы хотите отправлять файлы в запросах, состоящих из нескольких частей, необходимо вызвать метод `attach` перед выполнением запроса. Этот метод принимает имя файла и его содержимое. При желании вы можете указать третий аргумент, который будет считаться именем файла, в то время как четвертый аргумент может быть использован для предоставления заголовков, связанных с файлом:

    $response = Http::attach(
        'attachment', file_get_contents('photo.jpg'), 'photo.jpg', ['Content-Type' => 'image/jpeg']
    )->post('http://example.com/attachments');

Вы также можете передать потоковый ресурс вместо передачи необработанного содержимого файла:

    $photo = fopen('photo.jpg', 'r');

    $response = Http::attach(
        'attachment', $photo, 'photo.jpg'
    )->post('http://example.com/attachments');

<a name="headers"></a>
### Заголовки

Заголовки могут быть добавлены к запросам с помощью метода `withHeaders`. Метод `withHeaders` принимает массив пар ключ / значение:

    $response = Http::withHeaders([
        'X-First' => 'foo',
        'X-Second' => 'bar'
    ])->post('http://example.com/users', [
        'name' => 'Taylor',
    ]);

Вы можете использовать метод `accept`, чтобы указать тип контента, который ваше приложение ожидает в ответ на ваш запрос:

    $response = Http::accept('application/json')->get('http://example.com/users');

Для удобства вы можете использовать метод `acceptJson`, чтобы быстро указать, что ваше приложение ожидает тип содержимого `application/json` в ответ на ваш запрос:

    $response = Http::acceptJson()->get('http://example.com/users');

Метод `withHeaders` объединяет новые заголовки с существующими заголовками запроса. При необходимости вы можете полностью заменить все заголовки, используя метод `replaceHeaders`:

```php
$response = Http::withHeaders([
    'X-Original' => 'foo',
])->replaceHeaders([
    'X-Replacement' => 'bar',
])->post('http://example.com/users', [
    'name' => 'Taylor',
]);
```

<a name="authentication"></a>
### Аутентификация

Вы можете указать данные **basic** и **digest** аутентификации, используя методы `withBasicAuth` и `withDigestAuth`, соответственно:

    // Basic HTTP-аутентификация ...
    $response = Http::withBasicAuth('taylor@laravel.com', 'secret')->post(/* ... */);

    // Digest HTTP-аутентификациия ...
    $response = Http::withDigestAuth('taylor@laravel.com', 'secret')->post(/* ... */);

<a name="bearer-tokens"></a>
#### Токены Bearer

Если вы хотите добавить токен в заголовок `Authorization` запроса, то используйте метод `withToken`:

    $response = Http::withToken('token')->post(/* ... */);

<a name="timeout"></a>
### Время ожидания

Метод `timeout` используется для указания максимального количества секунд ожидания ответа. По умолчанию время ожидания HTTP-клиента истекает через 30 секунд::

    $response = Http::timeout(3)->post(/* ... */);;

Если указанный тайм-аут превышен, то будет выброшено исключение `Illuminate\Http\Client\ConnectionException`.

Вы можете указать максимальное количество секунд ожидания при попытке подключения к серверу, используя метод `connectTimeout`:

    $response = Http::connectTimeout(3)->get(/* ... */);

<a name="retries"></a>
### Повторные попытки

Если вы хотите, чтобы HTTP-клиент автоматически повторял запрос при возникновении ошибки клиента или сервера, то используйте метод `retry`. Метод `retry` принимает максимальное количество попыток выполнения запроса и количество миллисекунд, которые Laravel должен ждать между попытками:

    $response = Http::retry(3, 100)->post(/* ... */);

При необходимости вы можете передать третий аргумент методу `retry`. Третий аргумент должен быть callable-функцией, которая определяет, следует ли на самом деле попытаться повторить попытку. Например, вы можете захотеть повторить запрос только в том случае, если начальный запрос обнаруживает исключение `ConnectionException`:

    use Exception;
    use Illuminate\Http\Client\PendingRequest;

    $response = Http::retry(3, 100, function (Exception $exception, PendingRequest $request) {
        return $exception instanceof ConnectionException;
    })->post(/* ... */);

Если попытка запроса завершится неудачей, вы можете захотеть внести изменение в запрос перед новой попыткой. Это можно сделать, изменив аргумент запроса, предоставленный вашему вызываемому объекту метода `retry`. Например, вы можете попробовать запрос с новым токеном авторизации, если первая попытка завершилась ошибкой аутентификации:

    use Exception;
    use Illuminate\Http\Client\PendingRequest;
    use Illuminate\Http\Client\RequestException;

    $response = Http::withToken($this->getToken())->retry(2, 0, function (Exception $exception, PendingRequest $request) {
        if (! $exception instanceof RequestException || $exception->response->status() !== 401) {
            return false;
        }

        $request->withToken($this->getNewToken());

        return true;
    })->post(/* ... */);

Если все запросы окажутся неуспешными, то будет выброшено исключение `Illuminate\Http\Client\RequestException`. Если вы хотите отключить это поведение, вы можете предоставить аргумент `throw` со значением `false`. При отключении, после всех попыток повтора, будет возвращен последний полученный клиентом ответ:

    $response = Http::retry(3, 100, throw: false)->post(/* ... */);


> [!WARNING]
> Если все запросы завершаются неудачей из-за проблем с подключением, исключение `Illuminate\Http\Client\ConnectionException` все равно будет сгенерировано, даже если аргумент `throw` установлен в `false`.

<a name="error-handling"></a>
### Обработка ошибок

В отличие от поведения Guzzle по умолчанию, обертка HTTP-клиента Laravel не генерирует исключений при возникновении ошибок клиента или сервера (ответы `400` и `500`, соответственно). Вы можете определить, была ли возвращена одна из этих ошибок, используя методы `successful`, `clientError`, или `serverError`:

    // Определить, имеет ли ответ код состояния >= 200 and < 300...
    $response->successful();

    // Определить, имеет ли ответ код состояния >= 400...
    $response->failed();

    // Определить, имеет ли ответ код состояния 400 ...
    $response->clientError();

    // Определить, имеет ли ответ код состояния 500 ...
    $response->serverError();

    // Немедленно выполнить данную функцию обратного вызова, если произошла ошибка клиента или сервера...
    $response->onError(callable $callback);

<a name="throwing-exceptions"></a>
#### Выброс исключений

Если у вас есть экземпляр ответа и вы хотите выбросить исключение `Illuminate\Http\Client\RequestException`, если код состояния ответа указывает на ошибку клиента или сервера, используйте методы `throw` или `throwIf`:

    use Illuminate\Http\Client\Response;

    $response = Http::post(/* ... */);

    // Выбросить исключение, если произошла ошибка клиента или сервера ...
    $response->throw();

    // Выбросить исключение, если произошла ошибка и данное условие истинно...
    $response->throwIf($condition);

    // Выбросить исключение, если произошла ошибка и данное замыкание принимает значение true...
    $response->throwIf(fn (Response $response) => true);

    // Выбросить исключение, если произошла ошибка и заданное условие равно false...
    $response->throwUnless($condition);

    // Выбросить исключение, если произошла ошибка и данное замыкание принимает значение false...
    $response->throwUnless(fn (Response $response) => false);

    // Выбросить исключение, если ответ имеет определенный код состояния...
    $response->throwIfStatus(403);

    // Выбросить исключение, если только ответ не содержит определенного кода состояния...
    $response->throwUnlessStatus(200);

    return $response['user']['id'];

Экземпляр `Illuminate\Http\Client\RequestException` имеет свойство `$response`, которое позволит вам проверить возвращенный ответ.

Метод `throw` возвращает экземпляр ответа, если ошибки не произошло, что позволяет вам использовать цепочку вызовов после метода `throw`:

    return Http::post(/* ... */)->throw()->json();

Если вы хотите выполнить некоторую дополнительную логику до того, как будет сгенерировано исключение, вы можете передать замыкание методу `throw`. Исключение будет сгенерировано автоматически после вызова замыкания, поэтому вам не нужно повторно генерировать исключение изнутри замыкания:

    use Illuminate\Http\Client\Response;
    use Illuminate\Http\Client\RequestException;

    return Http::post(/* ... */)->throw(function (Response $response, RequestException $e) {
        // ...
    })->json();

<a name="guzzle-middleware"></a>
### Guzzle Middleware

Поскольку HTTP-клиент Laravel работает на основе Guzzle, вы можете воспользоваться [Guzzle Middleware](https://docs.guzzlephp.org/en/stable/handlers-and-middleware.html) для изменения исходящего запроса или анализа входящего ответа. Для изменения исходящего запроса зарегистрируйте middleware Guzzle с помощью метода `withRequestMiddleware`:

    use Illuminate\Support\Facades\Http;
    use Psr\Http\Message\RequestInterface;

    $response = Http::withRequestMiddleware(
        function (RequestInterface $request) {
            return $request->withHeader('X-Example', 'Value');
        }
    )->get('http://example.com');

Точно так же вы можете осмотреть входящий HTTP-ответ, зарегистрировав middleware с помощью метода `withResponseMiddleware`:

    use Illuminate\Support\Facades\Http;
    use Psr\Http\Message\ResponseInterface;

    $response = Http::withResponseMiddleware(
        function (ResponseInterface $response) {
            $header = $response->getHeader('X-Example');

            // ...

            return $response;
        }
    )->get('http://example.com');

<a name="global-middleware"></a>
#### Глобальное Middleware

Иногда вы можете захотеть зарегистрировать middleware, которое применяется ко всем исходящим запросам и входящим ответам. Для этого вы можете использовать методы `globalRequestMiddleware` и `globalResponseMiddleware`. Обычно эти методы следует вызывать в методе `boot` файла `AppServiceProvider` вашего приложения:

```php
use Illuminate\Support\Facades\Http;

Http::globalRequestMiddleware(fn ($request) => $request->withHeader(
    'User-Agent', 'Example Application/1.0'
));

Http::globalResponseMiddleware(fn ($response) => $response->withHeader(
    'X-Finished-At', now()->toDateTimeString()
));
```

<a name="guzzle-options"></a>
### Параметры Guzzle

Вы можете указать дополнительные [параметры запроса для Guzzle](http://docs.guzzlephp.org/en/stable/request-options.html), используя метод `withOptions`. Метод `withOptions` принимает массив пар ключ / значение:

    $response = Http::withOptions([
        'debug' => true,
    ])->get('http://example.com/users');

<a name="concurrent-requests"></a>
## Параллельные запросы

Иногда вы можете захотеть выполнить несколько HTTP-запросов одновременно. Другими словами, вы хотите, чтобы несколько запросов отправлялись одновременно вместо того, чтобы отправлять их последовательно. Это может привести к значительному повышению производительности при взаимодействии с медленными HTTP API.

Вы можете сделать это с помощью метода `pool`. Метод `pool` принимает функцию с аргументом `Illuminate\Http\Client\Pool`, при помощи которого вы можете добавлять запросы в пул запросов для отправки:

    use Illuminate\Http\Client\Pool;
    use Illuminate\Support\Facades\Http;

    $responses = Http::pool(fn (Pool $pool) => [
        $pool->get('http://localhost/first'),
        $pool->get('http://localhost/second'),
        $pool->get('http://localhost/third'),
    ]);

    return $responses[0]->ok() &&
           $responses[1]->ok() &&
           $responses[2]->ok();

Как видите, к каждому экземпляру ответа можно получить доступ в том порядке, в котором он был добавлен в пул. При желании вы можете назвать запросы с помощью метода `as`, что позволит вам получить доступ к соответствующим ответам по имени:

    use Illuminate\Http\Client\Pool;
    use Illuminate\Support\Facades\Http;

    $responses = Http::pool(fn (Pool $pool) => [
        $pool->as('first')->get('http://localhost/first'),
        $pool->as('second')->get('http://localhost/second'),
        $pool->as('third')->get('http://localhost/third'),
    ]);

    return $responses['first']->ok();

<a name="customizing-concurrent-requests"></a>
#### Customizing Concurrent Requests

Метод `pool` не может быть объединен с другими методами HTTP-клиента, такими как `withHeaders` или `middleware`. Если вы хотите применить пользовательские заголовки или middleware к пулу запросов, вы должны настроить эти параметры для каждого запроса в пуле:

```php
use Illuminate\Http\Client\Pool;
use Illuminate\Support\Facades\Http;

$headers = [
    'X-Example' => 'example',
];

$responses = Http::pool(fn (Pool $pool) => [
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
]);
```

<a name="macros"></a>
## Макросы

HTTP-клиент Laravel позволяет вам определять «макросы», которые могут служить плавным, выразительным механизмом для настройки общих путей запросов и заголовков при взаимодействии со службами в вашем приложении. Для начала вы можете определить макрос в методе `boot` класса `App\Providers\AppServiceProvider` вашего приложения:

```php
use Illuminate\Support\Facades\Http;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Http::macro('github', function () {
        return Http::withHeaders([
            'X-Example' => 'example',
        ])->baseUrl('https://github.com');
    });
}
```

После того, как ваш макрос настроен, вы можете вызвать его из любого места вашего приложения, чтобы создать ожидающий запрос с указанной конфигурацией:

```php
$response = Http::github()->get('/');
```

<a name="testing"></a>
## Тестирование

Многие службы Laravel содержат функционал, помогающий вам легко и выразительно писать тесты, и HTTP-клиент Laravel не является исключением. Метод `fake` фасада `Http` позволяет указать HTTP-клиенту возвращать заглушенные / фиктивные ответы при выполнении запросов.

<a name="faking-responses"></a>
### Фиктивные ответы

Например, чтобы дать указание HTTP-клиенту возвращать пустые ответы с кодом состояния `200` на каждый запрос, вы можете вызвать метод `fake` без аргументов:

    use Illuminate\Support\Facades\Http;

    Http::fake();

    $response = Http::post(/* ... */);

<a name="faking-specific-urls"></a>
#### Фальсификация конкретных URL

В качестве альтернативы вы можете передать массив методу `fake`. Ключи массива должны представлять шаблоны URL, которые вы хотите подделать, и связанные с ними ответы. Допускается использование метасимвола подстановки `*`. Любые запросы к URL-адресам, которые не были сфальсифицированы, будут выполнены фактически. Вы можете использовать метод `response` фасада `Http` для создания заглушек / фиктивных ответов для этих адресов:

    Http::fake([
        // Заглушка JSON ответа для адресов GitHub ...
        'github.com/*' => Http::response(['foo' => 'bar'], 200, $headers),

        // Заглушка строкового ответа для адресов Google ...
        'google.com/*' => Http::response('Hello World', 200, $headers),
    ]);

Если вы хотите указать шаблон резервного URL-адреса, который будет заглушать все не сопоставленные URL-адреса, то используйте символ `*`:

    Http::fake([
        // Заглушка JSON ответа для адресов GitHub ...
        'github.com/*' => Http::response(['foo' => 'bar'], 200, ['Headers']),

        // Заглушка строкового ответа для всех остальных адресов ...
        '*' => Http::response('Hello World', 200, ['Headers']),
    ]);

<a name="faking-response-sequences"></a>
#### Фальсификация серии ответов

По желанию можно указать, что один URL должен возвращать серию фиктивных ответов в определенном порядке. Вы можете сделать это, используя метод `Http::sequence` для составления ответов:

    Http::fake([
        // Заглушка серии ответов для адресов GitHub ...
        'github.com/*' => Http::sequence()
                                ->push('Hello World', 200)
                                ->push(['foo' => 'bar'], 200)
                                ->pushStatus(404),
    ]);

Когда все ответы в этой последовательности будут использованы, любые дальнейшие запросы приведут к выбросу исключения. Если вы хотите указать ответ по умолчанию, который должен возвращаться, когда последовательность пуста, то используйте метод `whenEmpty`:

    Http::fake([
        // Заглушка серии ответов для адресов GitHub ...
        'github.com/*' => Http::sequence()
                                ->push('Hello World', 200)
                                ->push(['foo' => 'bar'], 200)
                                ->whenEmpty(Http::response()),
    ]);

Если вы хотите подделать серию ответов без указания конкретного шаблона URL, который следует подделать, то используйте метод `Http::fakeSequence`:

    Http::fakeSequence()
            ->push('Hello World', 200)
            ->whenEmpty(Http::response());

<a name="fake-callback"></a>
#### Анонимные фальсификаторы

Если вам требуется более сложная логика для определения того, какие ответы возвращать для определенных адресов, то вы можете передать замыкание методу `fake`. Это замыкание получит экземпляр `Illuminate\Http\Client\Request` и должно вернуть экземпляр ответа. В замыкании вы можете выполнить любую логику, необходимую для определения типа ответа, который нужно вернуть:

    use Illuminate\Http\Client\Request;

    Http::fake(function (Request $request) {
        return Http::response('Hello World', 200);
    });

<a name="preventing-stray-requests"></a>
### Предотвращение случайных запросов

Если вы хотите удостовериться, что все запросы, отправленные через HTTP-клиент в рамках вашего отдельного теста или всего тестового набора, были поддельными, вы можете вызвать метод `preventStrayRequests`. После вызова этого метода любые запросы, которые не имеют соответствующего поддельного ответа, вызовут исключение, вместо того чтобы делать фактический HTTP-запрос:

    use Illuminate\Support\Facades\Http;

    Http::preventStrayRequests();

    Http::fake([
        'github.com/*' => Http::response('ok'),
    ]);

    // An "ok" response is returned...
    Http::get('https://github.com/laravel/framework');

    // An exception is thrown...
    Http::get('https://laravel.com');

<a name="inspecting-requests"></a>
### Инспектирование запросов

При фальсификации ответов вы можете иногда захотеть проверить запросы, которые получает клиент, чтобы убедиться, что ваше приложение отправляет правильные данные или заголовки. Вы можете сделать это, вызвав метод `Http::assertSent` после вызова `Http::fake`.

Метод `assertSent` принимает замыкание, которому будет передан экземпляр `Illuminate\Http\Client\Request` и, которое должно возвращать значение логического типа, указывающее, соответствует ли запрос вашим ожиданиям. Для успешного прохождения теста должен быть отправлен хотя бы один запрос, соответствующий указанным ожиданиям:

    use Illuminate\Http\Client\Request;
    use Illuminate\Support\Facades\Http;

    Http::fake();

    Http::withHeaders([
        'X-First' => 'foo',
    ])->post('http://example.com/users', [
        'name' => 'Taylor',
        'role' => 'Developer',
    ]);

    Http::assertSent(function (Request $request) {
        return $request->hasHeader('X-First', 'foo') &&
               $request->url() == 'http://example.com/users' &&
               $request['name'] == 'Taylor' &&
               $request['role'] == 'Developer';
    });

При необходимости вы можете утверждать, что конкретный запрос не был отправлен с помощью метода `assertNotSent`:

    use Illuminate\Http\Client\Request;
    use Illuminate\Support\Facades\Http;

    Http::fake();

    Http::post('http://example.com/users', [
        'name' => 'Taylor',
        'role' => 'Developer',
    ]);

    Http::assertNotSent(function (Request $request) {
        return $request->url() === 'http://example.com/posts';
    });

Вы можете использовать метод `assertSentCount`, чтобы определить, сколько запросов было отправлено во время теста:

    Http::fake();

    Http::assertSentCount(5);

Или используйте метод `assertNothingSent`, чтобы утверждать, что во время теста не было отправлено никаких запросов:

    Http::fake();

    Http::assertNothingSent();

<a name="recording-requests-and-responses"></a>
#### Запись Запросов/Ответов

Вы можете использовать метод `recorded` для сбора всех запросов и соответствующих ответов. Метод `recorded` возвращает коллекцию массивов, которая содержит экземпляры `Illuminate\Http\Client\Request` и `Illuminate\Http\Client\Response`:

```php
Http::fake([
    'https://laravel.com' => Http::response(status: 500),
    'https://nova.laravel.com/' => Http::response(),
]);

Http::get('https://laravel.com');
Http::get('https://nova.laravel.com/');

$recorded = Http::recorded();

[$request, $response] = $recorded[0];
```

Кроме того, метод `recorded` принимает замыкание, которое получит экземпляры `Illuminate\Http\Client\Request` и `Illuminate\Http\Client\Response` и может быть использовано для фильтрации пар запрос/ответ в соответствии с вашими ожиданиями:

```php
use Illuminate\Http\Client\Request;
use Illuminate\Http\Client\Response;

Http::fake([
    'https://laravel.com' => Http::response(status: 500),
    'https://nova.laravel.com/' => Http::response(),
]);

Http::get('https://laravel.com');
Http::get('https://nova.laravel.com/');

$recorded = Http::recorded(function (Request $request, Response $response) {
    return $request->url() !== 'https://laravel.com' &&
           $response->successful();
});
```

<a name="events"></a>
## События

Laravel запускает три события в процессе отправки HTTP-запросов. Событие `RequestSending` запускается до отправки запроса, а событие `ResponseReceived` запускается после получения ответа на данный запрос. Событие `ConnectionFailed` запускается, если на данный запрос ответ не получен.

События `RequestSending` и `ConnectionFailed` содержат общедоступное свойство `$request`, которое вы можете использовать для проверки экземпляра `Illuminate\Http\Client\Request`. Аналогично, событие `ResponseReceived` содержит свойство `$request`, а также свойство `$response`, которое может использоваться для проверки экземпляра `Illuminate\Http\Client\Response`. Вы можете зарегистрировать прослушиватели событий для этого события в вашем сервис-провайдере `App\Providers\EventServiceProvider`:

    /**
     * Сопоставления прослушивателей событий для приложения.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Http\Client\Events\RequestSending' => [
            'App\Listeners\LogRequestSending',
        ],
        'Illuminate\Http\Client\Events\ResponseReceived' => [
            'App\Listeners\LogResponseReceived',
        ],
        'Illuminate\Http\Client\Events\ConnectionFailed' => [
            'App\Listeners\LogConnectionFailed',
        ],
    ];
