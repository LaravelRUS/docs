git 213dac979200a6dbebed73f6b96d6aad1236a08f

---

# Аутентификация API

- [Введение](#introduction)
- [Настройка](#configuration)
    - [Миграции базы данных](#database-preparation)
- [Генерация токенов](#generating-tokens)
    - [Хэширование токенов](#hashing-tokens)
- [Защита роутов](#protecting-routes)
- [Передача токенов в запросах](#passing-tokens-in-requests)

<a name="introduction"></a>
## Введение

По умолчанию Laravel поставляется с простым решением для аутентификации API с помощью случайного токена, назначенного каждому пользователю вашего приложения.  В вашем файле конфигурации `config/auth.php` защита `api` уже определена и использует драйвер `token`.  Этот драйвер отвечает за проверку токена API во входящем запросе и проверку его соответствия назначенному токену пользователя в базе данных.

> **Внимание:** Хотя Laravel поставляется с простым средством проверки подлинности на основе токенов, настроятельно рекомендуется использовать [Laravel Passport](/docs/{{version}}/passport) для надежных производственных приложений, которые предлагают аутентификацию с помощью API.

<a name="configuration"></a>
## Настройка

<a name="database-preparation"></a>
### Миграции базы данных

Перед использованием драйвера `token` вам необходимо [создать миграцию](/docs/{{version}}/migrations), которая добавляет столбец `api_token` в таблицу `users`:

    Schema::table('users', function ($table) {
        $table->string('api_token', 80)->after('password')
                            ->unique()
                            ->nullable()
                            ->default(null);
    });

После того, как миграция будет создана, выполните Artisan-команду `migrate`.

<a name="generating-tokens"></a>
## Генерация токенов

После добавления столбца `api_token` в вашу таблицу `users` вы можете назначить случайные токены API каждому пользователю, который регистрируется в вашем приложении. Вы должны назначить эти токены, когда для пользователя создается модель `User` во время регистрации. При использовании [вспомогательной аутентификации](/docs/{{version}}/authentication#authentication-quickstart), предоставляемой командой Artisan `make:auth`, это можно сделать в методе `create` контроллера `RegisterController`:

    use Illuminate\Support\Str;
    use Illuminate\Support\Facades\Hash;

    /**
     * Создает новый экземпляр пользователя после регистрации.
     *
     * @param  array  $data
     * @return \App\User
     */
    protected function create(array $data)
    {
        return User::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => Hash::make($data['password']),
            'api_token' => Str::random(60),
        ]);
    }

<a name="hashing-tokens"></a>
### Хэширование токенов

В приведенных выше примерах токены API хранятся в вашей базе данных в виде простого текста. Если вы хотите хэшировать свои токены API с помощью хэширования SHA-256, вы можете установить для опции `hash` вашей конфигурации защиты `api` значение `true`. Защита `api` определяется в вашем файле конфигурации `config/auth.php`:

    'api' => [
        'driver' => 'token',
        'provider' => 'users',
        'hash' => true,
    ],

#### Генерация хэшированных токенов

При использовании хешированных токенов API вы не должны генерировать свои токены API во время регистрации пользователя. Вместо этого вам потребуется реализовать собственную страницу управления токенами API в вашем приложении. Эта страница должна позволять пользователям инициализировать и обновлять свой токен API. Когда пользователь делает запрос на инициализацию или обновление своего токена, вы должны сохранить хешированную копию токена в базе данных и одноразово вывести текстовую копию токена клиенту в интерфейсе.

Например, метод контроллера, который инициализирует / обновляет токен для данного пользователя и возвращает токен в виде простого текста в виде ответа JSON, может выглядеть следующим образом:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Str;
    use Illuminate\Http\Request;

    class ApiTokenController extends Controller
    {
        /**
         * Обновляет токен API аутентифицированного пользователя.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function update(Request $request)
        {
            $token = Str::random(60);

            $request->user()->forceFill([
                'api_token' => hash('sha256', $token),
            ])->save();

            return ['token' => $token];
        }
    }

> {tip} Поскольку токены API в приведенном выше примере имеют достаточную энтропию, нецелесообразно создавать "радужные таблицы" для поиска исходного значения хешированного токена. Поэтому медленные методы хеширования, такие как `bcrypt`, не нужны.

<a name="protecting-routes"></a>
## Защита роутов

Laravel включает в себя [гвард проверки подлинности](/docs/{{version}}/authentication#adding-custom-guards), который автоматически проверяет токены API на входящих запросах. Вам нужно только указать промежуточное ПО `auth:api` на любом маршруте, для которого требуется действительный токен доступа:

    use Illuminate\Http\Request;

    Route::middleware('auth:api')->get('/user', function(Request $request) {
        return $request->user();
    });

<a name="passing-tokens-in-requests"></a>
## Передача токенов в запросах

Существует несколько способов передачи токена API в ваше приложение. Мы обсудим каждый из этих подходов при использовании библиотеки Guzzle HTTP, чтобы продемонстрировать их использование. Вы можете выбрать любой из этих подходов в зависимости от потребностей вашего приложения.

#### Строка запроса

Пользователи API вашего приложения могут указать свой токен в виде значения строки запроса `api_token`:

    $response = $client->request('GET', '/api/user?api_token='.$token);

#### Форма запроса

Пользователи API вашего приложения могут включить свой токен API в параметры формы запроса в виде `api_token`:

    $response = $client->request('POST', '/api/user', [
        'headers' => [
            'Accept' => 'application/json',
        ],
        'form_params' => [
            'api_token' => $token,
        ],
    ]);

#### Bearer токен

Пользователи API вашего приложения могут предоставить свой API-токен в качестве `Bearer` токена в заголовке `Authorization` запроса:

    $response = $client->request('POST', '/api/user', [
        'headers' => [
            'Authorization' => 'Bearer '.$token,
            'Accept' => 'application/json',
        ],
    ]);
