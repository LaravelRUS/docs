git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# HTTP тесты

- [Введение](#introduction)
- [Сессия / Аутентификация](#session-and-authentication)
- [Тестирование JSON API](#testing-json-apis)
- [Тестирование загрузки файлов](#testing-file-uploads)
- [Доступные утверждения](#available-assertions)

<a name="introduction"></a>
## Введение

Laravel предоставляет довольно функциональный API для составления HTTP-запросов к вашему приложению и анализа выходных данных. Например, давайте рассмотрим тест ниже:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function testBasicTest()
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

Метод `get` делает запрос `GET` в приложение, в то время как метод `assertStatus` утверждает, что возвращенный ответ должен иметь заданный HTTP код статуса. В дополнение к этому простому утверждению в Laravel также содержится множество утверждений для инспектирования заголовков ответов, их содержимого, структуры JSON и др.

<a name="session-and-authentication"></a>
## Сессия / Аутентификация

В Laravel есть несколько хелперов для работы с сессией во время HTTP-тестирования. Во-первых, можно задать данные сессии заданному массиву используя метод `withSession`. Это полезно для загрузки сессии с данными до отправки запроса вашему приложению:

    <?php

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $response = $this->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }

Конечно, одно из общепринятых использований сессии - поддержание состояния аутентифицированного пользователя. Хелпер `actingAs` предоставляет простой способ аутентифицировать заданного пользователя как текущего пользователя. Например, можно использовать [фабрику моделей](/docs/{{version}}/database-testing#writing-factories), чтобы сгенерировать и аутентифицировать пользователя:

    <?php

    use App\User;

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $user = factory(User::class)->create();

            $response = $this->actingAs($user)
                             ->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }

Также можно указать какого гварда следует использовать для аутентификации заданного пользователя, передавая имя гварда в качестве второго аргумента методу `actingAs`:

    $this->actingAs($user, 'api')

<a name="testing-json-apis"></a>
## Тестирование JSON API

В Laravel есть несколько хелперов для тестирования JSON API и его ответов. Например, методы `json`, `get`, `post`, `put`, `patch` и `delete` можно использовать, чтобы послать запросы с разными HTTP глаголами. Вы также можете легко передать этим методам данные и заголовки. Для начала напишем тест, чтобы послать запрос `POST` пользователю `/user` и затем выдать утверждение, что были возвращены ожидаемые данные:

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(200)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

> {tip} Метод `assertJson` конвертирует ответ в массив и использует `PHPUnit::assertArraySubset` для проверки, что заданный массив существует в JSON-ответе, возвращенном приложением. Итак, если у JSON-ответа есть другие свойства, этот тест все еще будет пройден, так как присутствует заданный фрагмент.

<a name="verifying-exact-match"></a>
### Проверка точного соответствия

Если нам нужно проверить, что заданный массив - **точное** соответствие JSON, возвращенного приложением, следует использовать метод `assertExactJson`:

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(200)
                ->assertExactJson([
                    'created' => true,
                ]);
        }
    }

<a name="testing-file-uploads"></a>
## Тестирование загрузки файлов

Класс `Illuminate\Http\UploadedFile` предоставляет метод `fake`, который можно использовать для генерирования фиктивных файлов или изображений для тестирования. Вместе с методом `fake` васада `Storage` это значительно упрощает тестирование загрузки файлов. Например, можно скомбинировать эти две функции, чтобы быстро протестировать форму загрузки аватара:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        public function testAvatarUpload()
        {
            Storage::fake('avatars');

            $response = $this->json('POST', '/avatar', [
                'avatar' => UploadedFile::fake()->image('avatar.jpg')
            ]);

            // Assert the file was stored...
            Storage::disk('avatars')->assertExists('avatar.jpg');

            // Assert a file does not exist...
            Storage::disk('avatars')->assertMissing('missing.jpg');
        }
    }

#### Настройка фиктивных файлов

При создании файлов методом `fake` можно указать ширину, высоту и размер изображения, чтобы лучше проверить свои правила валидации:

    UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);

В дополнение к созданию изображений можно создавать файлы любого другого типа используя метод `create`:

    UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);

<a name="available-assertions"></a>
## Доступные утверждения

Laravel предоставляет множество методов утверждений для ваших [PHPUnit](https://phpunit.de/)-тестов. Доступ к этим утверждениям можно получить в ответе, который был возвращен тестовыми методами `json`, `get`, `post`, `put` и `delete`:

Метод  | Описание
------------- | -------------
`$response->assertStatus($code);`  |  Ответ содержит заданный код.
`$response->assertRedirect($uri);`  |  Ответ - это редирект на заданный URI.
`$response->assertHeader($headerName, $value = null);`  |  В ответе присутствует заданные заголовок.
`$response->assertCookie($cookieName, $value = null);`  |  В ответе содержится заданный cookie.
`$response->assertPlainCookie($cookieName, $value = null);`  |  В ответе содержится заданный cookie (незашифрованный).
`$response->assertSessionHas($key, $value = null);`  |  В сессии содержится заданный фрагмент данных.
`$response->assertSessionHasErrors(array $keys);`  |  В сессии содержится ошибка для заданного поля.
`$response->assertSessionMissing($key);`  |  В сессии не содержится заданный ключ.
`$response->assertJson(array $data);`  |  В ответе содержатся заданные данные JSON.
`$response->assertJsonFragment(array $data);`  |  В ответе содержится заданный фрагмент JSON.
`$response->assertJsonMissing(array $data);`  |  В ответе не содержится заданный фрагмент JSON.
`$response->assertExactJson(array $data);`  |  В ответе содержится точное совпадение заданных JSON-данных.
`$response->assertJsonStructure(array $structure);`  |  У ответа есть заданная JSON-структура.
`$response->assertViewIs($value);`  |  Роутом был возвращен заданный шаблон.
`$response->assertViewHas($key, $value = null);`  |  Фрагмент данных был передан шалону ответа.
