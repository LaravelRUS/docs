git 63a231cc5940dda64e8f0b08e0b9e7be33b6e27c

---

# Аутентификация API

- [Введение](#introduction)
- [Настройка](#configuration)
    - [Подготовка базы данных](#database-preparation)
- [Создание токенов](#generating-tokens)
    - [Хэширование токенов](#hashing-tokens)
- [Применение в роутах](#protecting-routes)
- [Способы передачи токена в запросе](#passing-tokens-in-requests)

<a name="introduction"></a>
## Введение

В Laravel встроен простой способ для аутентификации API - при помощи случайного строкового токена, который присваивается каждому залогиненному пользователю. Этот токен сообщается клиентскому приложению при логине, сохраняется в базе данных на сервере и проверяется при каждом запросе. В `config/auth.php` гвард `api` уже настроен на работу с этим типом авторизации - в качестве драйвера там по умолчанию указан `token`.

> **Примечание** Если вам нужна аутентификация API, рекомендуем также рассмотреть [Laravel Passport](/docs/{{version}}/passport)

<a name="configuration"></a>
## Настройка

<a name="database-preparation"></a>
### Подготовка базы данных

Для того, чтобы сохранять токен в базе данных, создайте [миграцию](/docs/{{version}}/migrations) для создания столбца `api_token` в таблице `users`: 

    Schema::table('users', function ($table) {
        $table->string('api_token', 80)->after('password')
                            ->unique()
                            ->nullable()
                            ->default(null);
    });

Примените миграцию при помощи команды `php artisan migrate`

> {tip} Если название столбца `api_token` вам не подходит, задайте своё, параллельно указав его в `storage_key` в конфигурационном файле `config/auth.php`

<a name="generating-tokens"></a>
## Создание токенов

В момент регистрации пользователя вам нужно предусмотреть создание токена в виде случайной строки и запись его в поле `api_token`. Если вы используете [генераторы файлов аутентификации](/docs/{{version}}/authentication#authentication-quickstart) помощи пакета `laravel/ui`, вы должны изменить метод `create` контроллера `RegisterController`:

    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Str;

    /**
     * Create a new user instance after a valid registration.
     *
     * @param  array  $data
     * @return \App\User
     */
    protected function create(array $data)
    {
        return User::forceCreate([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => Hash::make($data['password']),
            'api_token' => Str::random(80),
        ]);
    }

<a name="hashing-tokens"></a>
### Хэширование токенов

Если вы не хотите хранить токены в открытом виде (как в примере выше), вы можете указать, что в базе хранится их хэш (SHA-256). Для этого установите параметр `hash` у гварда `api` в `config/auth.php`:

    'api' => [
        'driver' => 'token',
        'provider' => 'users',
        'hash' => true,
    ],

#### Генерация хэша для токенов

При использовании хешированных токенов вы не должны генерировать их при регистрации пользователя. Вместо этого вам потребуется реализовать собственную страницу управления токенами в своем приложении. Эта страница должна позволять пользователям инициализировать и обновлять свой токен. Когда пользователь делает запрос на инициализацию или обновление своего токена, вы должны сохранить хешированную копию токена в базе данных и возвратить текстовую копию токена клиенту. 

Вот пример метода обновления токена:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Str;

    class ApiTokenController extends Controller
    {
        /**
         * Update the authenticated user's API token.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function update(Request $request)
        {
            $token = Str::random(80);

            $request->user()->forceFill([
                'api_token' => hash('sha256', $token),
            ])->save();

            return ['token' => $token];
        }
    }

> {tip} Поскольку токены API в виде случайных строк имеют достаточную энтропию, нецелесообразно использовать «радужные таблицы» для подбора исходного значения хешированного токена. Следовательно, методы медленного хеширования, такие как `bcrypt`, тут не нужны, достаточно `hash`.

<a name="protecting-routes"></a>
## Применение в роутах

В Laravel есть [гвард аутентификации](/docs/{{version}}/authentication#adding-custom-guards) API, который автоматически проверяет токены. Чтобы включить его на заданных роутах используйте посредника (middleware) `auth:api`:

    use Illuminate\Http\Request;

    Route::middleware('auth:api')->get('/user', function (Request $request) {
        return $request->user();
    });

<a name="passing-tokens-in-requests"></a>
## Способы передачи токена в запросе

Существует несколько способов как передать токен в запросе к серверу, чтобы гвард аутентификации его смог проверить. Примеры ниже даны с использованием библиотеки Guzzle HTTP.

#### В урле запроса

Вы можете поставить `api_token` в качестве GET-параметра:

    $response = $client->request('GET', '/api/user?api_token='.$token);

#### В теле запроса

Вы можете добавить `api_token` одним из полей в POST запросе:

    $response = $client->request('POST', '/api/user', [
        'headers' => [
            'Accept' => 'application/json',
        ],
        'form_params' => [
            'api_token' => $token,
        ],
    ]);

#### Bearer токен

Вы можете добавить `api_token` в заголовки запроса:

    $response = $client->request('POST', '/api/user', [
        'headers' => [
            'Authorization' => 'Bearer '.$token,
            'Accept' => 'application/json',
        ],
    ]);
