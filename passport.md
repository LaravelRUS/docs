git e69b326ad6ab5c42944cd6bfd39ba458d023a1e0

---

# Laravel Passport

- [Введение](#introduction)
    - [Passport или Sanctum?](#passport-or-sanctum)
- [Установка](#installation)
    - [Развертывание Passport](#deploying-passport)
    - [Настройка миграции](#migration-customization)
    - [Обновление Passport](#upgrading-passport)
- [Настройка](#configuration)
    - [Хеширование секретного ключа клиента](#client-secret-hashing)
    - [Срок жизни токена](#token-lifetimes)
    - [Переопределение моделей по умолчанию](#overriding-default-models)
- [Выдача токенов доступа](#issuing-access-tokens)
    - [Управление клиентами](#managing-clients)
    - [Запрос токенов](#requesting-tokens)
    - [Обновление токенов](#refreshing-tokens)
    - [Отзыв токенов](#revoking-tokens)
    - [Удаление токенов](#purging-tokens)
- [Предоставление кода авторизации с помощью PKCE](#code-grant-pkce)
    - [Создание клиента](#creating-a-auth-pkce-grant-client)
    - [Запрос токенов](#requesting-auth-pkce-grant-tokens)
- [Парольные токены](#password-grant-tokens)
    - [Создание токенов](#creating-a-password-grant-client)
    - [Запрос токенов](#requesting-password-grant-tokens)
    - [Запрос токена для всех областей](#requesting-all-scopes)
    - [Настройка пользовательского провайдера](#customizing-the-user-provider)
    - [Настройка поля имени пользователя](#customizing-the-username-field)
    - [Настройка проверки пароля пользователя](#customizing-the-password-validation)
- [Неявные токены](#implicit-grant-tokens)
- [Токены учетных данных](#client-credentials-grant-tokens)
- [Токены персонального доступа](#personal-access-tokens)
    - [Создание токенов](#creating-a-personal-access-client)
    - [Управление токенами](#managing-personal-access-tokens)
- [Защита маршрутов](#protecting-routes)
    - [Через посредников](#via-middleware)
    - [Через передачу токена](#passing-the-access-token)
- [Области токенов](#token-scopes)
    - [Определение областей](#defining-scopes)
    - [Области по-умолчанию](#default-scope)
    - [Назначение областей токенам](#assigning-scopes-to-tokens)
    - [Проверка областей](#checking-scopes)
- [Использование API через JavaScript](#consuming-your-api-with-javascript)
- [События](#events)
- [Тестирование](#testing)

<a name="introduction"></a>
## Введение

[Laravel Passport](https://github.com/laravel/passport) обеспечивает полную реализацию сервера OAuth2 для вашего приложения Laravel за считанные минуты. Passport построен на основе [League OAuth2](https://github.com/thephpleague/oauth2-server), который поддерживается Энди Миллингтоном (Andy Millington) и Саймоном Хэмпом (Simon Hamp).

> {Примечание} В этой документации предполагается, что вы уже знакомы с OAuth2. Если вы ничего не знаете о OAuth2, перед продолжением ознакомьтесь с общей [терминологией](https://oauth2.thephpleague.com/terminology/) и функциями OAuth2.

<a name="passport-or-sanctum"></a>
### Passport или Sanctum?

Прежде чем начать, вы можете определиться, будет ли ваше приложение лучше обслуживаться через Laravel Passport или [Laravel Sanctum](/docs/{{version}}/sanctum). Если вашему приложению необходима поддержка OAuth2, то следует использовать Laravel Passport.

Однако, если вы пытаетесь аутентифицировать одностраничное приложение, мобильное приложение или выдавать токены API, вам следует использовать [Laravel Sanctum](/docs/{{version}}/sanctum). Laravel Sanctum не поддерживает OAuth2; однако он обеспечивает гораздо более простой опыт разработки аутентификации API.

<a name="installation"></a>
## Установка

Для начала установите Passport через менеджер пакетов Composer:

    composer require laravel/passport

[Сервис-провайдер](/docs/{{version}}/providers) Passport регистрирует свой собственный каталог миграции базы данных, поэтому вам следует перенести свою базу данных после установки пакета. При миграции паспорта будут созданы таблицы, необходимые вашему приложению для хранения клиентов OAuth2 и токенов доступа:

    php artisan migrate

Затем вы должны выполнить Artisan-команду `passport:install`. Эта команда создаст ключи шифрования, необходимые для создания токенов безопасного доступа. Кроме того, команда создаст клиентов "personal access" и "password grant", которые будут использоваться для генерации токенов доступа:

    php artisan passport:install

> {Примечание} Если вы хотите использовать UUID в качестве значения первичного ключа модели Passport `Client` вместо автоматически увеличивающихся целых чисел, установите Passport, используя [the `uuids` option](#client-uuids).

После выполнения команды `passport:install` добавьте [трейт](https://www.php.net/manual/ru/language.oop5.traits.php) `Laravel\Passport\HasApiTokens` в свою модель `App\Models\User`. Этот трейт предоставит вашей модели несколько вспомогательных методов, которые позволят вам проверить токен и области аутентифицированного пользователя. Если ваша модель уже использует трейт `Laravel\Sanctum\HasApiTokens`, вы можете его удалить:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Factories\HasFactory;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, HasFactory, Notifiable;
    }

Затем вы должны вызвать метод `Passport::routes` в методе `boot` вашего `App\Providers\AuthServiceProvider`. Этот метод зарегистрирует маршруты, необходимые для выдачи токенов, отзыва токенов, токенов персонального доступа и клиентов:

    <?php

    namespace App\Providers;

    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    use Illuminate\Support\Facades\Gate;
    use Laravel\Passport\Passport;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Сопоставление политик приложения.
         *
         * @var array
         */
        protected $policies = [
            'App\Models\Model' => 'App\Policies\ModelPolicy',
        ];

        /**
         * Регистрация сервисов аутентификации и авторизации.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            if (! $this->app->routesAreCached()) {
                Passport::routes();
            }
        }
    }

Наконец, в файле конфигурации приложения `config/auth.php` вы должны установить для параметра `driver` раздела `api` значение `passport`. Это укажет вашему приложению использовать Passport `TokenGuard` при аутентификации входящих запросов API:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
        ],
    ],

<a name="client-uuids"></a>
#### Клиентские UUID

Вы также можете запустить команду `passport:install` с опцией `--uuids`. Эта опция укажет Passport, что вы хотели бы использовать UUID вместо автоматически увеличивающихся целых чисел в качестве значений первичного ключа модели Passport `Client`. После выполнения команды `passport:install` с параметром `--uuids` вы получите дополнительные инструкции по отключению миграций по умолчанию для Passport:

    php artisan passport:install --uuids

<a name="deploying-passport"></a>
### Развертывание Passport

При первом развертывании Passport на серверах вашего приложения вам, вероятно, потребуется выполнить команду `passport:keys`. Эта команда генерирует ключи шифрования, необходимые Passport для создания токенов доступа. Сгенерированные ключи обычно не хранятся в системе контроля версий:

    php artisan passport:keys

При необходимости вы можете указать путь, откуда должны быть загружены ключи Passport. Для этого вы можете использовать метод `Passport::loadKeysFrom`. Обычно этот метод следует вызывать из метода `boot` класса `App\Providers\AuthServiceProvider` вашего приложения:

    /**
     * Регистрация сервисов аутентификации и авторизации.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::loadKeysFrom(__DIR__.'/../secrets/oauth');
    }

<a name="loading-keys-from-the-environment"></a>
#### Загрузка ключей из окружения

В качестве альтернативы вы можете опубликовать файл конфигурации Passport с помощью Artisan-команды `vendor:publish`:

    php artisan vendor:publish --tag=passport-config

После публикации файла конфигурации вы можете загрузить ключи шифрования вашего приложения, определив их как переменные среды:

```bash
PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
<private key here>
-----END RSA PRIVATE KEY-----"

PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
<public key here>
-----END PUBLIC KEY-----"
```

<a name="migration-customization"></a>
### Настройка миграции

Если вы не собираетесь использовать миграции Passport по умолчанию, вам следует вызвать метод `Passport::ignoreMigrations` в методе `register` вашего класса `App\Providers\AppServiceProvider`. Вы можете экспортировать миграции по умолчанию, используя Artisan-команду `vendor:publish`:

    php artisan vendor:publish --tag=passport-migrations

<a name="upgrading-passport"></a>
### Обновление Passport

При обновлении до новой основной версии Passport важно внимательно изучить [руководство по обновлению](https://github.com/laravel/passport/blob/master/UPGRADE.md).

<a name="configuration"></a>
## Настройка

<a name="client-secret-hashing"></a>
### Хеширование секретного ключа клиента

Если вы хотите, чтобы секретные ключи клиента хешировались при хранении в базе данных, вы должны вызвать метод `Passport::hashClientSecrets` в методе `boot` класса `App\Providers\AuthServiceProvider`:

    use Laravel\Passport\Passport;

    Passport::hashClientSecrets();

После включения все секретные ключи будут отображаться пользователю один раз, только после их создания. Поскольку значение секретного ключа в виде обычного текста никогда не хранится в базе данных, невозможно восстановить значение ключа, если оно утеряно.

<a name="token-lifetimes"></a>
### Срок жизни токена

По умолчанию Passport выдает долговременные токены доступа, срок действия которых истекает через год. Если вы хотите настроить более длительный / более короткий срок жизни токена, вы можете использовать методы `tokensExpireIn`, `refreshTokensExpireIn` и `personalAccessTokensExpireIn`. Эти методы следует вызывать из метода `boot` класса `App\Providers\AuthServiceProvider` вашего приложения:

    /**
     * Регистрация сервисов аутентификации и авторизации.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::tokensExpireIn(now()->addDays(15));
        Passport::refreshTokensExpireIn(now()->addDays(30));
        Passport::personalAccessTokensExpireIn(now()->addMonths(6));
    }

> {Примечание} Поля `expires_at` в таблицах базы данных Passport доступны только для чтения и только для отображения. При выпуске токенов Passport сохраняет информацию об истечении срока действия в подписанных и зашифрованных токенах. Если вам нужно сделать токен недействительным, вы должны [отозвать его](#revoking-tokens).

<a name="overriding-default-models"></a>
### Переопределение моделей по умолчанию

Вы можете свободно расширять модели, используемые внутри Passport, определяя свою собственную модель и расширяя соответствующую модель Passport:

    use Laravel\Passport\Client as PassportClient;

    class Client extends PassportClient
    {
        // ...
    }

После определения модели вы можете указать Passport использовать вашу пользовательскую модель через класс `Laravel\Passport\Passport`. Как правило, вы должны сообщить Passport о ваших пользовательских моделях в методе `boot` класса `App\Providers\AuthServiceProvider` вашего приложения:

    use App\Models\Passport\AuthCode;
    use App\Models\Passport\Client;
    use App\Models\Passport\PersonalAccessClient;
    use App\Models\Passport\Token;

    /**
     * Регистрация сервисов аутентификации и авторизации.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::useTokenModel(Token::class);
        Passport::useClientModel(Client::class);
        Passport::useAuthCodeModel(AuthCode::class);
        Passport::usePersonalAccessClientModel(PersonalAccessClient::class);
    }

<a name="issuing-access-tokens"></a>
## Выдача токенов доступа

Использование OAuth2 через коды авторизации — это то, через что большинство разработчиков знакомится с OAuth2. При использовании кодов авторизации клиентское приложение перенаправит пользователя на ваш сервер, где он либо утвердит, либо отклонит запрос на выдачу токена доступа клиенту.

<a name="managing-clients"></a>
### Управление клиентами

Во-первых, разработчики, которым необходимо взаимодействовать с API вашего приложения, должны будут зарегистрировать свое приложение в вашем, создав «клиента» (client). Обычно это состоит из указания имени своего приложения и URL-адреса, на который ваше приложение может перенаправить после того, как пользователи одобрят свой запрос на авторизацию.

<a name="the-passportclient-command"></a>
#### Команда `passport:client`

Самый простой способ создать клиента — использовать Artisan-команду `passport:client`. Эта команда может использоваться для создания ваших собственных клиентов для тестирования вашей функциональности OAuth2. Когда вы запускаете команду `client`, Passport запросит у вас дополнительную информацию о вашем клиенте и предоставит вам идентификатор клиента и секретный ключ:

    php artisan passport:client

**URL-адреса перенаправления**

Если вы хотите разрешить несколько URL-адресов перенаправления для своего клиента, вы можете указать их, используя список с разделителями-запятыми, когда вам будет предложено ввести URL-адрес командой `passport:client`. Любые URL-адреса, содержащие запятые, должны быть закодированы:

```bash
http://example.com/callback,http://examplefoo.com/callback
```

<a name="clients-json-api"></a>
#### JSON API

Поскольку пользователи вашего приложения не смогут использовать команду `client`, Passport предоставляет JSON API, который вы можете использовать для создания клиентов. Это избавляет вас от необходимости вручную кодировать контроллеры для создания, обновления и удаления клиентов.

Однако вам нужно будет связать JSON API Passport с вашим собственным интерфейсом, чтобы предоставить вашим пользователям панель управления для управления своими клиентами. Ниже мы рассмотрим все конечные точки API для управления клиентами. Для удобства мы будем использовать [Axios](https://github.com/axios/axios), чтобы продемонстрировать выполнение HTTP-запросов к конечным точкам.

JSON API защищен посредниками `web` и `auth`; поэтому его можно вызывать только из вашего собственного приложения. Он не может быть вызван из внешнего источника.

<a name="get-oauthclients"></a>
#### `GET /oauth/clients`

Этот маршрут возвращает всех клиентов для аутентифицированного пользователя. Это в первую очередь полезно для перечисления всех клиентов пользователя, чтобы пользователи могли редактировать или удалять их:

    axios.get('/oauth/clients')
        .then(response => {
            console.log(response.data);
        });

<a name="post-oauthclients"></a>
#### `POST /oauth/clients`

Этот маршрут используется для создания новых клиентов. Для этого требуются два параметра: `name` - имя клиента и `redirect` - URL-адрес перенаправления. URL-адрес `redirect` - это то, куда пользователь будет перенаправлен после утверждения или отклонения запроса на авторизацию.

Когда клиент будет создан, ему будет выдан идентификатор клиента и секретный ключ. Эти значения будут использоваться при запросе токенов доступа из вашего приложения. Маршрут создания клиента вернет новый экземпляр клиента:

    const data = {
        name: 'Client Name',
        redirect: 'http://example.com/callback'
    };

    axios.post('/oauth/clients', data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // List errors on response...
        });

<a name="put-oauthclientsclient-id"></a>
#### `PUT /oauth/clients/{client-id}`

Этот маршрут используется для обновления клиентов. Для этого требуются два параметра: `name` - имя клиента и `redirect` - URL-адрес перенаправления. URL-адрес `redirect` - это то, куда пользователь будет перенаправлен после утверждения или отклонения запроса на авторизацию. Маршрут вернет обновленный экземпляр клиента:

    const data = {
        name: 'New Client Name',
        redirect: 'http://example.com/callback'
    };

    axios.put('/oauth/clients/' + clientId, data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // List errors on response...
        });

<a name="delete-oauthclientsclient-id"></a>
#### `DELETE /oauth/clients/{client-id}`

Этот маршрут используется для удаления клиентов:

    axios.delete('/oauth/clients/' + clientId)
        .then(response => {
            //
        });

<a name="requesting-tokens"></a>
### Запрос токенов

<a name="requesting-tokens-redirecting-for-authorization"></a>
#### Перенаправление для авторизации

После создания клиента разработчики могут использовать свой идентификатор клиента и секретный ключ, чтобы запросить код авторизации и токен доступа из вашего приложения. Во-первых, приложение-потребитель должно сделать запрос перенаправления на маршрут вашего приложения `/oauth/authorize` следующим образом:

    use Illuminate\Http\Request;
    use Illuminate\Support\Str;

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'response_type' => 'code',
            'scope' => '',
            'state' => $state,
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

> {Примечание} Помните, что маршрут `/oauth/authorize` уже определен методом `Passport::routes`. Вам не нужно вручную определять этот маршрут.

<a name="approving-the-request"></a>
#### Подтверждение запроса

При получении запросов на авторизацию Passport автоматически отображает шаблон для пользователя, позволяющий подтвердить или отклонить запрос авторизации. Если они подтвердят запрос, то будут перенаправлены обратно на адрес `redirect_uri`, который был указан приложением-потребителем. Адрес `redirect_uri` должен совпадать с URL-адресом `redirect`, который был указан при создании клиента.

Если вы хотите настроить экран утверждения авторизации, вы можете опубликовать макет Passport с помощью Artisan-команды `vendor: publish`. Опубликованные макеты будут помещены в каталог `resources/views/vendor/passport`:

    php artisan vendor:publish --tag=passport-views

Иногда вам может потребоваться пропустить запрос авторизации, например, при авторизации основного клиента. Вы можете добиться этого, [расширив модель `Client`](#overriding-default-models) и определив метод `skipsAuthorization`. Если `skipsAuthorization` возвращает `true`, клиент будет одобрен, и пользователь будет немедленно перенаправлен обратно в `redirect_uri`:

    <?php

    namespace App\Models\Passport;

    use Laravel\Passport\Client as BaseClient;

    class Client extends BaseClient
    {
        /**
         * Определите, должен ли клиент пропускать запрос авторизации.
         *
         * @return bool
         */
        public function skipsAuthorization()
        {
            return $this->firstParty();
        }
    }

<a name="requesting-tokens-converting-authorization-codes-to-access-tokens"></a>
#### Преобразование кодов авторизации в токены доступа

Если пользователь одобряет запрос авторизации, он будет перенаправлен обратно в приложение-потребитель. Потребитель должен сначала сверить параметр `state` со значением, которое было сохранено до перенаправления. Если параметр `state` совпадает, то потребитель должен отправить вашему приложению запрос `POST`, чтобы запросить токен доступа. Запрос должен включать код авторизации, который был выдан вашим приложением, когда пользователь утвердил запрос авторизации:

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Http;

    Route::get('/callback', function (Request $request) {
        $state = $request->session()->pull('state');

        throw_unless(
            strlen($state) > 0 && $state === $request->state,
            InvalidArgumentException::class
        );

        $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
            'grant_type' => 'authorization_code',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'code' => $request->code,
        ]);

        return $response->json();
    });

Маршрут `/oauth/token` вернет ответ JSON, содержащий атрибуты `access_token`, `refresh_token` и `expires_in`. Атрибут `expires_in` содержит количество секунд до истечения срока действия токена доступа.

> {Примечание} Как и маршрут `/oauth/authorize`, маршрут `/oauth/token` определяется для вас методом `Passport::routes`. Нет необходимости определять этот маршрут вручную.

<a name="tokens-json-api"></a>
#### JSON API

Passport также включает JSON API для управления авторизованными токенами доступа. Вы можете связать это со своим собственным интерфейсом, чтобы предложить своим пользователям панель управления для управления токенами доступа. Для удобства мы будем использовать [Axios](https://github.com/mzabriskie/axios), чтобы продемонстрировать выполнение HTTP-запросов к конечным точкам. JSON API защищен посредниками `web` и `auth`; поэтому его можно вызывать только из вашего собственного приложения.

<a name="get-oauthtokens"></a>
#### `GET /oauth/tokens`

Этот маршрут возвращает все токены доступа, созданные аутентифицированным пользователем. Это в первую очередь полезно для просмотра всех токенов пользователя, чтобы он мог их отозвать:

    axios.get('/oauth/tokens')
        .then(response => {
            console.log(response.data);
        });

<a name="delete-oauthtokenstoken-id"></a>
#### `DELETE /oauth/tokens/{token-id}`

Этот маршрут может использоваться для отзыва токенов доступа и связанных с ними токенов обновления:

    axios.delete('/oauth/tokens/' + tokenId);

<a name="refreshing-tokens"></a>
### Обновление токенов

Если ваше приложение выдает недолговечные токены доступа, пользователям потребуется обновить свои токены доступа с помощью токена обновления, предоставленного им при выдаче токена доступа:

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'refresh_token',
        'refresh_token' => 'the-refresh-token',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => '',
    ]);

    return $response->json();

Этот маршрут `/oauth/token` вернет ответ JSON, содержащий атрибуты `access_token`, `refresh_token` и `expires_in`. Атрибут `expires_in` содержит количество секунд до истечения срока действия токена доступа.

<a name="revoking-tokens"></a>
### Отзыв токенов

Вы можете отозвать токен с помощью метода `revokeAccessToken` в `Laravel\Passport\TokenRepository`. Вы можете отозвать токены обновления токена с помощью метода `revokeRefreshTokensByAccessTokenId` в `Laravel\Passport\RefreshTokenRepository`. Эти классы могут быть разрешены с помощью [сервисного контейнера](/docs/{{version}}/container) Laravel:

    use Laravel\Passport\TokenRepository;
    use Laravel\Passport\RefreshTokenRepository;

    $tokenRepository = app(TokenRepository::class);
    $refreshTokenRepository = app(RefreshTokenRepository::class);

    // Revoke an access token...
    $tokenRepository->revokeAccessToken($tokenId);

    // Revoke all of the token's refresh tokens...
    $refreshTokenRepository->revokeRefreshTokensByAccessTokenId($tokenId);

<a name="purging-tokens"></a>
### Удаление токенов

Когда токены были отозваны или срок их действия истек, вы можете удалить их из базы данных. Команда `passport:purge` Artisan, содержащаяся в Passport, может сделать это за вас:

    # Удалить отозванные и просроченные токены, и коды авторизации ...
    php artisan passport:purge

    # Удалить только отозванные токены и коды авторизации ...
    php artisan passport:purge --revoked

    # Удалить только просроченные токены и коды авторизации ...
    php artisan passport:purge --expired

Вы также можете настроить [запланированное задание](/docs/{{version}}/scheduling) в классе вашего приложения `App\Console\Kernel` для автоматического удаления токенов по расписанию:

    /**
     * Определите расписание приложения.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('passport:purge')->hourly();
    }

<a name="code-grant-pkce"></a>
## Предоставление кода авторизации с помощью PKCE

Предоставление кода авторизации с `Proof Key for Code Exchange` (PKCE) - это безопасный способ аутентификации одностраничных приложений или собственных приложений для доступа к вашему API. Это разрешение следует использовать, когда вы не можете гарантировать, что секретный ключ клиента будет храниться конфиденциально, или, чтобы уменьшить угрозу перехвата кода авторизации злоумышленником. Комбинация `code verifier` и `code challenge` заменяет секретный ключ клиента при замене кода авторизации на токен доступа.

<a name="creating-a-auth-pkce-grant-client"></a>
### Создание клиента

Прежде чем ваше приложение сможет выдавать токены через предоставление кода авторизации с помощью PKCE, вам необходимо создать клиента с поддержкой PKCE. Вы можете сделать это с помощью Artisan-команды `passport:client` с параметром `--public`:

    php artisan passport:client --public

<a name="requesting-auth-pkce-grant-tokens"></a>
### Запрос токенов

<a name="code-verifier-code-challenge"></a>
#### Code Verifier & Code Challenge

Поскольку это разрешение на авторизацию не предоставляет секретный ключ клиента, разработчикам необходимо сгенерировать комбинацию `code verifier` и `code challenge`, чтобы запросить токен.

Средство проверки кода должно представлять собой случайную строку от 43 до 128 символов, содержащую буквы, цифры и символы `"-"`, `"."`, `"_"`, `"~"`, как определено в [спецификации RFC 7636](https://tools.ietf.org/html/rfc7636).

Итоговым результатом должна быть строка в кодировке Base64 с URL-адресом и безопасными для имени файла символами. Завершающие символы '=' должны быть удалены, и не должно быть разрывов строк, пробелов или других дополнительных символов.

    $encoded = base64_encode(hash('sha256', $code_verifier, true));

    $codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');

<a name="code-grant-pkce-redirecting-for-authorization"></a>
#### Перенаправление для авторизации

После создания клиента вы можете использовать идентификатор клиента и сгенерированный `code verifier` и `code challenge`, чтобы запросить код авторизации и токен доступа из вашего приложения. Во-первых, приложение-потребитель должно сделать запрос перенаправления на маршрут вашего приложения `/oauth/authorize`:

    use Illuminate\Http\Request;
    use Illuminate\Support\Str;

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $request->session()->put(
            'code_verifier', $code_verifier = Str::random(128)
        );

        $codeChallenge = strtr(rtrim(
            base64_encode(hash('sha256', $code_verifier, true))
        , '='), '+/', '-_');

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'response_type' => 'code',
            'scope' => '',
            'state' => $state,
            'code_challenge' => $codeChallenge,
            'code_challenge_method' => 'S256',
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

<a name="code-grant-pkce-converting-authorization-codes-to-access-tokens"></a>
#### Преобразование кодов авторизации в токены доступа

Если пользователь одобряет запрос авторизации, он будет перенаправлен обратно в приложение-потребитель. Потребитель должен сверить параметр `state` со значением, которое было сохранено до перенаправления, как в стандартном предоставлении кода авторизации.

Если параметр состояния совпадает, потребитель должен отправить вашему приложению запрос `POST`, чтобы запросить токен доступа. Запрос должен включать код авторизации, который был выдан вашим приложением, когда пользователь утвердил запрос авторизации, вместе с первоначально сгенерированным верификатором кода:

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Http;

    Route::get('/callback', function (Request $request) {
        $state = $request->session()->pull('state');

        $codeVerifier = $request->session()->pull('code_verifier');

        throw_unless(
            strlen($state) > 0 && $state === $request->state,
            InvalidArgumentException::class
        );

        $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
            'grant_type' => 'authorization_code',
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'code_verifier' => $codeVerifier,
            'code' => $request->code,
        ]);

        return $response->json();
    });

<a name="password-grant-tokens"></a>
## Парольные токены

Предоставление пароля OAuth2 позволяет другим сторонним клиентам, таким как мобильное приложение, получать токен доступа, используя адрес электронной почты / имя пользователя и пароль. Это позволяет вам безопасно выдавать токены доступа своим основным клиентам, не требуя от пользователей прохождения всего потока перенаправления кода авторизации OAuth2.

<a name="creating-a-password-grant-client"></a>
### Создание токенов

Прежде чем ваше приложение сможет выдавать токены с помощью предоставления пароля, вам необходимо создать клиент предоставления пароля. Вы можете сделать это с помощью Artisan-команды `passport:client` с параметром `--password`. **Если вы уже выполнили команду `passport:install`, вам не нужно запускать эту команду:**

    php artisan passport:client --password

<a name="requesting-password-grant-tokens"></a>
### Запрос токенов

После того как вы создали клиента для предоставления пароля, вы можете запросить токен доступа, отправив запрос `POST` по маршруту `/oauth/token` с адресом электронной почты и паролем пользователя. Помните, что этот маршрут уже зарегистрирован методом `Passport::routes`, поэтому нет необходимости определять его вручную. Если запрос будет успешным, вы получите от сервера `access_token` и `refresh_token` в ответе JSON:

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'password',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'username' => 'taylor@laravel.com',
        'password' => 'my-password',
        'scope' => '',
    ]);

    return $response->json();

> {Примечание} Помните, токены доступа по умолчанию являются долгоживущими. Однако вы можете [настроить максимальное время жизни токена доступа](#configuration), если это необходимо.

<a name="requesting-all-scopes"></a>
### Запрос токена для всех областей

При использовании доступа по паролю или доступа с учетными данными клиента вы можете авторизовать токен для всех областей, поддерживаемых вашим приложением. Вы можете сделать это, указав `*` в параметре `scope`. При этом метод `can` экземпляра токена всегда будет возвращать `true`. Эта расширенная область может быть назначена только токену, который выпущен с использованием разрешений `password` или `client_credentials`:

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'password',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'username' => 'taylor@laravel.com',
        'password' => 'my-password',
        'scope' => '*',
    ]);

<a name="customizing-the-user-provider"></a>
### Настройка пользовательского провайдера

Если ваше приложение использует более одного [провайдера аутентификации пользователя](/docs/{{version}}/authentication#introduction), вы можете указать, какой провайдер использует клиент предоставления пароля, указав параметр `--provider` при создании клиента через команду `artisan passport:client --password`. Указанное имя провайдера должно соответствовать допустимому провайдеру, определенному в файле конфигурации приложения `config/auth.php`. Затем вы можете [защитить свой маршрут с помощью посредника](#via-middleware), чтобы гарантировать, что авторизованы только пользователи из указанного провайдера.

<a name="customizing-the-username-field"></a>
### Настройка поля имени пользователя

При аутентификации с использованием предоставления пароля Passport будет использовать атрибут `email` вашей аутентифицируемой модели в качестве "username". Однако вы можете настроить это поведение, определив метод `findForPassport` в своей модели:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;

        /**
         * Возвращает экземпляр пользователя для переданного имени.
         *
         * @param  string  $username
         * @return \App\Models\User
         */
        public function findForPassport($username)
        {
            return $this->where('username', $username)->first();
        }
    }

<a name="customizing-the-password-validation"></a>
### Настройка проверки пароля пользователя

При аутентификации с использованием предоставления пароля Passport будет использовать атрибут `password` модели для проверки пароля. Если модель не имеет атрибута `password` или вы хотите настроить логику проверки пароля, вы можете определить метод `validateForPassportPasswordGrant` в своей модели:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Support\Facades\Hash;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;

        /**
         * Проверьте пароль пользователя для предоставления разрешения.
         *
         * @param  string  $password
         * @return bool
         */
        public function validateForPassportPasswordGrant($password)
        {
            return Hash::check($password, $this->password);
        }
    }

<a name="implicit-grant-tokens"></a>
## Неявные токены

Неявное разрешение аналогично предоставлению кода авторизации; однако токен возвращается клиенту без обмена кодом авторизации. Это разрешение чаще всего используется для JavaScript или мобильных приложений, где учетные данные клиента не могут быть надежно сохранены. Чтобы включить разрешение, вызовите метод `enableImplicitGrant` в методе `boot` класса `App\Providers\AuthServiceProvider` вашего приложения:

    /**
     * Регистрация сервисов аутентификации и авторизации.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::enableImplicitGrant();
    }

После включения разрешения разработчики могут использовать свой идентификатор клиента для запроса токена доступа из вашего приложения. Приложение-потребитель должно сделать запрос перенаправления на маршрут вашего приложения `/oauth/authorize` следующим образом:

    use Illuminate\Http\Request;

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'response_type' => 'token',
            'scope' => '',
            'state' => $state,
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

> {Примечание} Помните, что маршрут `/oauth/authorize` уже определен методом `Passport::routes`. Вам не нужно вручную определять этот маршрут.

<a name="client-credentials-grant-tokens"></a>
## Токены учетных данных

Предоставление учетных данных клиента подходит для межмашинной (machine-to-machine) аутентификации. Например, вы можете использовать это разрешение в запланированном задании, которое выполняет задачи обслуживания через API.

Прежде чем ваше приложение сможет выдавать токены с помощью предоставления учетных данных клиента, вам необходимо создать клиента предоставления учетных данных. Вы можете сделать это, используя параметр `--client` в Artisan-команде `passport:client`:

    php artisan passport:client --client

Затем, чтобы использовать этот тип разрешения, вам необходимо добавить посредника `CheckClientCredentials` в свойство `$routeMiddleware` вашего файла `app/Http/Kernel.php`:

    use Laravel\Passport\Http\Middleware\CheckClientCredentials;

    protected $routeMiddleware = [
        'client' => CheckClientCredentials::class,
    ];

Затем назначьте посредника к маршруту:

    Route::get('/orders', function (Request $request) {
        ...
    })->middleware('client');

Чтобы ограничить доступ к маршруту определенными областями, вы можете предоставить разделенный запятыми список требуемых областей при подключении посредника `client` к маршруту:

    Route::get('/orders', function (Request $request) {
        ...
    })->middleware('client:check-status,your-scope');

<a name="retrieving-tokens"></a>
### Получение токенов

Чтобы получить токен с использованием этого типа разрешения, сделайте запрос к конечной точке `oauth/token`:

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'client_credentials',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => 'your-scope',
    ]);

    return $response->json()['access_token'];

<a name="personal-access-tokens"></a>
## Токены персонального доступа

Иногда ваши пользователи могут захотеть выдать себе токены доступа, не проходя типичный поток перенаправления кода авторизации. Разрешение пользователям выдавать себе токены через пользовательский интерфейс вашего приложения может быть полезно для предоставления пользователям возможности экспериментировать с вашим API или может служить более простым подходом к выдаче токенов доступа в целом.

<a name="creating-a-personal-access-client"></a>
### Создание токенов персонального доступа

Прежде чем ваше приложение сможет выдавать токены персонального доступа, вам необходимо создать клиента личного доступа. Вы можете сделать это, выполнив Artisan-команду `passport:client` с параметром `--personal`. Если вы уже выполнили команду `passport:install`, вам не нужно запускать эту команду:

    php artisan passport:client --personal

После создания клиента личного доступа поместите идентификатор клиента и секретное значение в виде обычного текста в файл `.env` вашего приложения:

```bash
PASSPORT_PERSONAL_ACCESS_CLIENT_ID="client-id-value"
PASSPORT_PERSONAL_ACCESS_CLIENT_SECRET="unhashed-client-secret-value"
```

<a name="managing-personal-access-tokens"></a>
### Управление токенами персонального доступа

После того как вы создали клиент персонального доступа, вы можете выдавать токены для данного пользователя, используя метод `createToken` в экземпляре модели `App\Models\User`. Метод `createToken` принимает имя токена в качестве первого аргумента и необязательный массив [области](#token-scopes) в качестве второго аргумента:

    use App\Models\User;

    $user = User::find(1);

    // Создание токена без области действия ...
    $token = $user->createToken('Token Name')->accessToken;

    // Создание токена с областью ...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="personal-access-tokens-json-api"></a>
#### JSON API

Passport также включает JSON API для управления токенами личного доступа. Вы можете связать это со своим собственным интерфейсом, чтобы предложить своим пользователям панель управления для управления токенами личного доступа. Ниже мы рассмотрим все конечные точки API для управления токенами личного доступа. Для удобства мы будем использовать [Axios](https://github.com/mzabriskie/axios), чтобы продемонстрировать выполнение HTTP-запросов к конечным точкам.

JSON API защищен посредниками `web` и `auth`; поэтому его можно вызывать только из вашего собственного приложения. Он не может быть вызван из внешнего источника.

<a name="get-oauthscopes"></a>
#### `GET /oauth/scopes`

Этот маршрут возвращает все [области](#token-scopes), определенные для вашего приложения. Вы можете использовать этот маршрут для перечисления областей, которые пользователь может назначить личному токену доступа:

    axios.get('/oauth/scopes')
        .then(response => {
            console.log(response.data);
        });

<a name="get-oauthpersonal-access-tokens"></a>
#### `GET /oauth/personal-access-tokens`

Этот маршрут возвращает все токены личного доступа, созданные аутентифицированным пользователем. Это в первую очередь полезно для перечисления всех токенов пользователей, чтобы они могли редактировать или отзывать их:

    axios.get('/oauth/personal-access-tokens')
        .then(response => {
            console.log(response.data);
        });

<a name="post-oauthpersonal-access-tokens"></a>
#### `POST /oauth/personal-access-tokens`

Этот маршрут создает новые токены личного доступа. Для этого требуются два параметра: `name` и` scopes`, которые должны быть назначены токену:

    const data = {
        name: 'Token Name',
        scopes: []
    };

    axios.post('/oauth/personal-access-tokens', data)
        .then(response => {
            console.log(response.data.accessToken);
        })
        .catch (response => {
            // список ошибок ответа
        });

<a name="delete-oauthpersonal-access-tokenstoken-id"></a>
#### `DELETE /oauth/personal-access-tokens/{token-id}`

Этот маршрут может использоваться для отзыва токенов личного доступа:

    axios.delete('/oauth/personal-access-tokens/' + tokenId);

<a name="protecting-routes"></a>
## Защита маршрутов

<a name="via-middleware"></a>
### Защита маршрутов через посредников

Паспорт включает в себя [защиту аутентификации](/docs/{{version}}/authentication#adding-custom-guards), которая проверяет токены доступа при входящих запросах. После того как вы настроили защиту `api` для использования драйвера `passport`, вам нужно указать посредника `auth:api` на всех маршрутах, для которых требуется действующий токен доступа:

    Route::get('/user', function () {
        //
    })->middleware('auth:api');

> {note} Если вы используете [токены учетных данных](#client-credentials-grant-tokens), вы должны вместо этого использовать [посредник `client`](#client-credentials-grant-tokens) для защиты ваших маршрутов `auth:api`.

<a name="multiple-authentication-guards"></a>
#### Множественная защита аутентификации

Если ваше приложение аутентифицирует разные типы пользователей, которые, возможно, используют совершенно разные модели Eloquent, вам, вероятно, потребуется определить конфигурацию защиты для каждого типа провайдера пользователей в вашем приложении. Это позволяет защитить запросы, предназначенные для конкретных поставщиков услуг. Например, при следующей конфигурации защиты конфигурационный файл `config/auth.php`:

    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],

    'api-customers' => [
        'driver' => 'passport',
        'provider' => 'customers',
    ],

Следующий маршрут будет использовать защиту `api-customers`, которая использует провайдера пользователей `customers` для аутентификации входящих запросов:

    Route::get('/customer', function () {
        //
    })->middleware('auth:api-customers');

> {Примечание} Для получения дополнительной информации об использовании нескольких поставщиков пользователей с Passport обратитесь к [документации по предоставлению пароля](#customizing-the-user-provider).

<a name="passing-the-access-token"></a>
### Защита маршрутов через передачу токена

При вызове маршрутов, защищенных Passport, пользователи API вашего приложения должны указать свой токен доступа как токен Bearer в заголовке Authorization своего запроса. Например, при использовании HTTP-библиотеки Guzzle:

    use Illuminate\Support\Facades\Http;

    $response = Http::withHeaders([
        'Accept' => 'application/json',
        'Authorization' => 'Bearer '.$accessToken,
    ])->get('https://passport-app.test/api/user');

    return $response->json();

<a name="token-scopes"></a>
## Области токенов

Области позволяют вашим клиентам API запрашивать определенный набор разрешений при запросе авторизации для доступа к учетной записи. Например, если вы создаете приложение для электронной коммерции, не всем потребителям API потребуется возможность размещать заказы. Вместо этого вы можете разрешить потребителям запрашивать авторизацию только для доступа к статусам отгрузки заказа. Другими словами, области позволяют пользователям вашего приложения ограничивать действия, которые стороннее приложение может выполнять от их имени.

<a name="defining-scopes"></a>
### Определение областей

Вы можете определить области своего API, используя метод `Passport::tokensCan` в методе `boot` класса `App\Providers\AuthServiceProvider` вашего приложения. Метод `tokensCan` принимает массив имен и описаний областей видимости. Описание области действия может быть любым, и оно будет отображаться для пользователей на экране утверждения авторизации:

    /**
     * Регистрация сервисов аутентификации и авторизации.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::tokensCan([
            'place-orders' => 'Place orders',
            'check-status' => 'Check order status',
        ]);
    }

<a name="default-scope"></a>
### Области по-умолчанию

Если клиент не запрашивает какие-либо определенные области, вы можете настроить свой сервер Passport для присоединения области (областей) по умолчанию к токену с помощью метода `setDefaultScope`. Как правило, вы должны вызывать этот метод из метода `boot` класса `App\Providers\AuthServiceProvider` вашего приложения:

    use Laravel\Passport\Passport;

    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);

    Passport::setDefaultScope([
        'check-status',
        'place-orders',
    ]);

<a name="assigning-scopes-to-tokens"></a>
### Назначение областей токенам

<a name="when-requesting-authorization-codes"></a>
#### При запросе кодов авторизации

При запросе токена доступа с использованием предоставления кода авторизации потребители должны указать свои желаемые области в качестве параметра строки запроса `scope`. Параметр `scope` должен быть списком областей, разделенных пробелами:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => 'place-orders check-status',
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

<a name="when-issuing-personal-access-tokens"></a>
#### При выдаче токенов личного доступа

Если вы выдаете токены личного доступа с помощью метода `createToken` модели `App\Models\User`, вы можете передать массив желаемых областей в качестве второго аргумента метода:

    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="checking-scopes"></a>
### Проверка областей

Passport включает два посредника, которые могут использоваться для проверки подлинности входящего запроса с помощью токена, и для них предоставлена заданная область. Для начала добавьте следующие посредники в свойство `$routeMiddleware` вашего файла `app/Http/Kernel.php`:

    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,

<a name="check-for-all-scopes"></a>
#### Проверка всех областей

Посреднику областей `scopes` может быть назначен маршрут для проверки того, что токен доступа входящего запроса содержит все перечисленные области:

    Route::get('/orders', function () {
        // Токен содержит обе области - "check-status" и "place-orders" ...
    })->middleware(['auth:api', 'scopes:check-status,place-orders']);

<a name="check-for-any-scopes"></a>
#### Проверка любых областей

Посреднику областей `scopes` может быть назначен маршрут для проверки того, что токен доступа входящего запроса имеет *хотя бы одну* из перечисленных областей:

    Route::get('/orders', function () {
        // Токен содержит одну из областей - "check-status" или "place-orders" ...
    })->middleware(['auth:api', 'scope:check-status,place-orders']);

<a name="checking-scopes-on-a-token-instance"></a>
#### Проверка областей на экземпляре токена

После того как запрос с аутентификацией токена доступа поступил в ваше приложение, вы все равно можете проверить, имеет ли токен заданную область действия, используя метод `tokenCan` в экземпляре `App\Models\User`:

    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            //
        }
    });

<a name="additional-scope-methods"></a>
#### Дополнительные методы области

Метод `scopeIds` вернет массив всех определенных идентификаторов / имен:

    use Laravel\Passport\Passport;

    Passport::scopeIds();

Метод `scopes` вернет массив всех определенных областей как экземпляры `Laravel\Passport\Scope`:

    Passport::scopes();

Метод `scopesFor` вернет массив экземпляров `Laravel\Passport\Scope`, соответствующих указанным идентификаторам / именам:

    Passport::scopesFor(['place-orders', 'check-status']);

Вы можете определить, была ли определена область, используя метод `hasScope`:

    Passport::hasScope('place-orders');

<a name="consuming-your-api-with-javascript"></a>
## Использование API через JavaScript

При создании API может быть чрезвычайно полезно иметь возможность использовать собственный API из приложения JavaScript. Такой подход к разработке API позволяет вашему собственному приложению использовать тот же API, которым вы делитесь со всем миром. Один и тот же API может использоваться вашим веб-приложением, мобильными приложениями, сторонними приложениями и любыми SDK, которые вы можете публиковать в различных менеджерах пакетов.

Как правило, если вы хотите использовать свой API из своего приложения JavaScript, вам необходимо вручную отправить токен доступа в приложение и передать его с каждым запросом в ваше приложение. Однако Passport включает посредник, который может справиться с этим за вас. Все, что вам нужно сделать, это добавить посредник `CreateFreshApiToken` в вашу группу посредников `web` в файле `app/Http/Kernel.php`:

    'web' => [
        // Другие посредники ...
        \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
    ],

> {Примечание} Вы должны убедиться, что посредник `CreateFreshApiToken` является последним в списке ваших посредников указанных ранее.

Это посредник будет прикреплять файл cookie `laravel_token` к вашим исходящим ответам. Этот файл cookie содержит зашифрованный JWT, который Passport будет использовать для аутентификации запросов API от вашего приложения JavaScript. Время жизни JWT равно вашему значению конфигурации session.lifetime. Теперь, поскольку браузер автоматически отправляет cookie со всеми последующими запросами, вы можете делать запросы к API вашего приложения без явной передачи токена доступа:

    axios.get('/api/user')
        .then(response => {
            console.log(response.data);
        });

<a name="customizing-the-cookie-name"></a>
#### Настройка имени Cookie

При необходимости вы можете настроить имя файла cookie `laravel_token`, используя метод `Passport::cookie`. Обычно этот метод следует вызывать из метода `boot` класса `App\Providers\AuthServiceProvider` вашего приложения:

    /**
     * Регистрация сервисов аутентификации и авторизации.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::cookie('custom_name');
    }

<a name="csrf-protection"></a>
#### CSRF защита

При использовании этого метода аутентификации вам необходимо убедиться, что в ваши запросы включен действительный заголовок токена CSRF. В состав шаблонов JavaScript Laravel по умолчанию входит экземпляр Axios, который будет автоматически использовать зашифрованное значение cookie `XSRF-TOKEN` для отправки заголовка `X-XSRF-TOKEN` в запросах.

> {Примечание} Если вы решите отправить заголовок `X-CSRF-TOKEN` вместо `X-XSRF-TOKEN`, вам нужно использовать незашифрованный токен, предоставленный `csrf_token()`.

<a name="events"></a>
## События

Passport вызывает события при выдаче токенов доступа и обновлении токенов. Вы можете использовать эти события для удаления или отмены других токенов доступа в вашей базе данных. При желании вы можете прикрепить слушателей к этим событиям в классе `App\Providers\EventServiceProvider` вашего приложения:

    /**
     * Подписчики событий приложения.
     *
     * @var array
     */
    protected $listen = [
        'Laravel\Passport\Events\AccessTokenCreated' => [
            'App\Listeners\RevokeOldTokens',
        ],

        'Laravel\Passport\Events\RefreshTokenCreated' => [
            'App\Listeners\PruneOldTokens',
        ],
    ];

<a name="testing"></a>
## Тестирование

Метод `actingAs` Passport может использоваться для указания аутентифицированного в данный момент пользователя, а также его областей действия. Первым аргументом, передаваемым методу `actingAs`, является экземпляр пользователя, а вторым - массив областей видимости, которые должны быть предоставлены токену пользователя:

    use App\Models\User;
    use Laravel\Passport\Passport;

    public function test_servers_can_be_created()
    {
        Passport::actingAs(
            User::factory()->create(),
            ['create-servers']
        );

        $response = $this->post('/api/create-server');

        $response->assertStatus(201);
    }

Метод `actingAsClient` Passport может использоваться для указания аутентифицированного в данный момент клиента, а также его областей. Первым аргументом, передаваемым методу `actingAsClient`, является экземпляр клиента, а вторым — массив областей видимости, которые должны быть предоставлены токену клиента:

    use Laravel\Passport\Client;
    use Laravel\Passport\Passport;

    public function test_orders_can_be_retrieved()
    {
        Passport::actingAsClient(
            Client::factory()->create(),
            ['check-status']
        );

        $response = $this->get('/api/orders');

        $response->assertStatus(200);
    }
