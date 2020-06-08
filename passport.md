git 79c77938dcb4bb3611b281d9e074975f28ce5a1b

---

# API аутентификации (Passport)

- [Введение](#introduction)
- [Установка](#installation)
    - [Быстрое знакомство с фронтендом](#frontend-quickstart)
    - [Развёртывание Passport](#deploying-passport)
- [Настройка](#configuration)
    - [Срок действия токенов](#token-lifetimes)
- [Выдача токенов доступа](#issuing-access-tokens)
    - [Управление клиентами](#managing-clients)
    - [Запрос токенов](#requesting-tokens)
    - [Обновление токенов](#refreshing-tokens)
- [Токены предоставления пароля](#password-grant-tokens)
    - [Создание клиента предоставления пароля](#creating-a-password-grant-client)
    - [Запрос токенов](#requesting-password-grant-tokens)
    - [Запрос всех прав](#requesting-all-scopes)
- [Неявное предоставление токенов](#implicit-grant-tokens)
- [Токены предоставления учётных данных клиента](#client-credentials-grant-tokens)
- [Персональные токены доступа](#personal-access-tokens)
    - [Создание клиента персонального доступа](#creating-a-personal-access-client)
    - [Управление токенами персонального доступа](#managing-personal-access-tokens)
- [Защита роутов](#protecting-routes)
    - [С помощью посредников](#via-middleware)
    - [Передача токена доступа](#passing-the-access-token)
- [Права токена](#token-scopes)
    - [Определение прав](#defining-scopes)
    - [Назначение прав токенам](#assigning-scopes-to-tokens)
    - [Проверка прав](#checking-scopes)
- [Использование вашего API с помощью JavaScript](#consuming-your-api-with-javascript)
- [События](#events)
- [Тестирование](#testing)

<a name="introduction"></a>
## Введение

В Laravel можно легко настроить аутентификацию через обычные формы входа, но что насчёт API? API обычно использует токены для аутентификации пользователей и не сохраняет состояние сессии между запросами. В Laravel реализована простая API аутентификация с помощью Laravel Passport, который предоставляет полную реализацию сервера OAuth2 для вашего приложения. Passport создан на основе [сервера League OAuth2](https://github.com/thephpleague/oauth2-server), созданного Алексом Билби.

> {note} Предполагается, что вы уже знакомы с OAuth2. Если вы ничего не знаете о OAuth2, рекомендуем ознакомиться с основной терминологией и возможностями OAuth2 самостоятельно перед прочтением этой статьи.

<a name="installation"></a>
## Установка

Сначала установите Passport через менеджер пакетов Composer:

    composer require laravel/passport=~4.0

Затем зарегистрируйте сервис-провайдер Passport в массиве `providers` в конфиге `config/app.php`:

    Laravel\Passport\PassportServiceProvider::class,

Сервис-провайдер Passport регистрирует свой собственный каталог миграций БД в фреймворке, поэтому вам надо мигрировать вашу БД после регистрации провайдера. Миграции Passport создадут таблицы, необходимые вашему приложению для хранения клиентов и токенов доступа:

    php artisan migrate

> {note} Если вы не собираетесь использовать миграции Passport по-умолчанию, вас следует вызвать метод `Passport::ignoreMigrations` в методе `register` своего `AppServiceProvider`. Можно экспортировать миграции по-цмолчанию, используя `php artisan vendor:publish --tag=passport-migrations`.

Затем выполните команду `passport:install`. Она создаст ключи шифрования, необходимые для генерирования надёжных токенов доступа. Кроме того команда создаст клиентов "personal access" (персональный доступ) и "password grant" (предоставление пароля), которые будут использоваться для генерирования токенов доступа:

    php artisan passport:install

После выполнения этой команды добавьте трейт `Laravel\Passport\HasApiTokens` в свою модель `App\User`. Этот трейт предоставит вашей модели несколько вспомогательных методов, которые позволят вам проверять токен и права аутентифицированного пользователя:

    <?php

    namespace App;

    use Laravel\Passport\HasApiTokens;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;
    }

Затем вызовите метод `Passport::routes` из метода `boot` своего `AuthServiceProvider`. Этот метод зарегистрирует роуты, необходимые для выдачи токенов доступа и отмены действия токенов доступа, клиентов и персональных токенов доступа:

    <?php

    namespace App\Providers;

    use Laravel\Passport\Passport;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Сопоставления политик для приложения.
         *
         * @var array
         */
        protected $policies = [
            'App\Model' => 'App\Policies\ModelPolicy',
        ];

        /**
         * Регистрация всех сервисов аутентификации / авторизации.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Passport::routes();
        }
    }

И, наконец, в конфиге `config/auth.php` задайте значение `passport` параметру `driver` защитника аутентификации `api`. После этого ваше приложение будет использовать `TokenGuard` Passport при аутентификации входящих API-запросов:

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

<a name="frontend-quickstart"></a>
### Быстрое знакомство с фронтендом

> {note} Чтобы использовать Vue-компоненты Passport, вам необходимо использовать JavaScript-фреймворк [Vue](https://vuejs.org). Эти компоненты также используют CSS-фреймворк Bootstrap. Но даже если вы не используете эти фреймворки, компоненты будут хорошим образцом для вашей собственной реализации фронтенда.

Passport поставляется с JSON API, который можно использовать, чтобы разрешить вашим пользователям создавать клиентов и персональные токены доступа. Но написание кода фронтенда для взаимодействия с этими API может занять много времени. Поэтому Passport также содержит встроенные компоненты [Vue](https://vuejs.org), которые вы можете использовать, как пример или отправную точку для своей собственной реализации.

Для публикации Vue-компонентов Passport используйте Artisan-команду `vendor:publish`:

    php artisan vendor:publish --tag=passport-components

Опубликованные компоненты будут помещены в каталог `resources/assets/js/components`. После публикации компонентов их необходимо зарегистрировать в файле `resources/assets/js/app.js`:

    Vue.component(
        'passport-clients',
        require('./components/passport/Clients.vue')
    );

    Vue.component(
        'passport-authorized-clients',
        require('./components/passport/AuthorizedClients.vue')
    );

    Vue.component(
        'passport-personal-access-tokens',
        require('./components/passport/PersonalAccessTokens.vue')
    );

После регистрации компонентов не забудьте выполнить `npm run dev` для перекомпилирования ваших ресурсов. После этого вы можете поместить компоненты в один из шаблонов вашего приложения, чтобы начать создавать клиентов и персональные токены доступа:

    <passport-clients></passport-clients>
    <passport-authorized-clients></passport-authorized-clients>
    <passport-personal-access-tokens></passport-personal-access-tokens>

<a name="deploying-passport"></a>
### Развёртывание Passport

При первом развёртывании Passport на своем рабочем сервере вам, скорее всего, потребуется запустить команду `passport:keys`. Эта команда генерирует ключи шифрования, необходимые для получения доступа к токену. Сгенерированные ключи обычно не хранятся в системе контроля версий:

    php artisan passport:keys

<a name="configuration"></a>
## Настройка

<a name="token-lifetimes"></a>
### Срок действия токенов

По-умолчанию Passport выдаёт длительные токены доступа, которые вообще не надо обновлять. Если вы хотите настроить более короткое время действия токенов, используйте методы `tokensExpireIn` и `refreshTokensExpireIn`. Эти методы надо вызывать из метода `boot` вашего `AuthServiceProvider`:

    use Carbon\Carbon;

    /**
     * Регистрация всех сервисов аутентификации / авторизации.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::tokensExpireIn(Carbon::now()->addDays(15));

        Passport::refreshTokensExpireIn(Carbon::now()->addDays(30));
    }

<a name="issuing-access-tokens"></a>
## Выдача токенов доступа

Большинство разработчиков знакомы с OAuth2 по работе с кодами авторизации. При использовании кодов авторизации клиентское приложение перенаправляет пользователя на ваш сервер, где разрешается или запрещается его запрос на получение токена на доступ к клиенту.

<a name="managing-clients"></a>
### Управление клиентами

Во-первых, разработчикам приложений, которым необходимо взаимодействие с API вашего приложения, необходимо зарегистрировать свои приложения в вашем, создав «клиента». Обычно это сводится к предоставлению названия их приложения и URL, на который ваше приложение сможет перенаправлять пользователей, когда они запрашивают авторизацию.

#### Команда `passport:client`

Простейший способ создать клиента — Artisan-команда `passport:client`. Её можно использовать для создания своих клиентов для тестирования возможностей OAuth2. Когда вы запустите команду `client`, Passport запросит у вас информацию о вашем клиенте и предоставит вам ID и секретную строку клиента:

    php artisan passport:client

#### JSON API

Поскольку у ваших пользователей не будет возможности использовать команду `client`, Passport предоставляет JSON API, который вы можете использовать для создания клиентов. Это избавляет вас от необходимости вручную писать контроллеры для создания, изменения и удаления клиентов.

Но вам надо будет дополнить Passport JSON API своим собственным фронтендом, чтобы предоставить пользователям панель для управления их клиентами. Далее мы рассмотрим все урлы API для управления клиентами. Для удобства мы будем использовать [Axios](https://github.com/mzabriskie/axios) для демонстрации создания HTTP-запросов к серверу.

> {tip} Если вы не хотите самостоятельно реализовывать весь фронтенд управления клиентами, то можете использовать [быстрое знакомство с фронтендом](#frontend-quickstart), чтобы получить полностью функциональный фронтенд за считанные минуты.

#### `GET /oauth/clients`

Этот роут возвращает всех клиентов для аутентифицированного пользователя. Это особенно полезно для просмотра всех клиентов пользователя, чтобы редактировать или удалять их:

    axios.get('/oauth/clients')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/clients`

Этот роут используется для создания новых клиентов. Ему необходимы следующие данные: название клиента (`name`) и URL переадресации (`redirect`). Этот URL будет использован для переадресации пользователя после подтверждения или отклонения запроса авторизации.

После создания клиента ему будут выданы ID клиента и секретную строку клиента. Эти значения будут использоваться при запросе токенов доступа у вашего приложения. Роут создания клиента вернёт новый экземпляр клиента:

    const data = {
        name: 'Client Name',
        redirect: 'http://example.com/callback'
    };

    axios.post('/oauth/clients', data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // Список ошибок в отклике...
        });

#### `PUT /oauth/clients/{client-id}`

Этот роут используется для изменения клиентов. Ему необходимы следующие данные: название клиента (`name`) и URL переадресации (`redirect`). URL переадресации (`redirect`) будет использован для переадресации пользователя после подтверждения или отклонения запроса авторизации. Роут вернёт обновлённый экземпляр клиента:

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

#### `DELETE /oauth/clients/{client-id}`

Этот роут используется для удаления клиентов:

    axios.delete('/oauth/clients/' + clientId)
        .then(response => {
            //
        });

<a name="requesting-tokens"></a>
### Запрос токенов

#### Переадресация для авторизации

После создания клиента разработчики могут использовать ID и секретную строку своего клиента для запроса кода авторизации и токенов доступа у вашего приложения. Во-первых, запрашивающее приложение должно сделать запрос переадресации роуту `/oauth/authorize` таким образом:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => '',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} Помните, роут `/oauth/authorize` уже определён методом `Passport::routes`. Вам не надо вручную определять этот роут.

####  Подтверждение запроса 

При получении запросов авторизации Passport автоматически выведет пользователю шаблон, позволяя им подтвердить или отклонить запрос авторизации. Если они подтверждают запрос, то автоматически переадресуются обратно на `redirect_uri` , указанный запрашивающим приложением. Этот `redirect_uri` должен совпадать с URL `redirect`, указанным при создании клиента.

Если вы хотите изменить окно подтверждения авторизации, вы можете опубликовать шаблоны Passport Artisan-командой `vendor:publish`. Опубликованные шаблоны будут размещены в каталоге `resources/views/vendor/passport`:

    php artisan vendor:publish --tag=passport-views

#### Конвертирование кодов авторизации в токены доступа

Если пользователь подтвердит запрос авторизации, то будет переадресован обратно в запрашивающее приложение. Затем оно должно сделать запрос `POST` в ваше приложение, чтобы запросить токен доступа. Запрос должен содержать код авторизации, который был выдан вашим приложением, когда пользователь подтвердил запрос авторизации. В этом примере мы используем HTTP-библиотеку Guzzle для создания `POST`-запроса:

    Route::get('/callback', function (Request $request) {
        $http = new GuzzleHttp\Client;

        $response = $http->post('http://your-app.com/oauth/token', [
            'form_params' => [
                'grant_type' => 'authorization_code',
                'client_id' => 'client-id',
                'client_secret' => 'client-secret',
                'redirect_uri' => 'http://example.com/callback',
                'code' => $request->code,
            ],
        ]);

        return json_decode((string) $response->getBody(), true);
    });

Роут `/oauth/token` вернёт JSON, содержащий атрибуты `access_token`, `refresh_token` и`expires_in`. Атрибут `expires_in` содержит число секунд до истечения действия токена доступа.

> {tip} Как и роут `/oauth/authorize`, роут `/oauth/token` также определен методом `Passport::routes`. Вам не надо вручную определять этот роут.

<a name="refreshing-tokens"></a>
### Обновление токенов

Если ваше приложение выдаёт краткосрочные токены доступа, пользователям будет необходимо обновлять свои токены доступа с помощью токена обновления, который предоставляется при выдаче токена доступа. В этом примере мы используем HTTP-библиотеку Guzzle для обновления токена:

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'refresh_token',
            'refresh_token' => 'the-refresh-token',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => '',
        ],
    ]);

    return json_decode((string) $response->getBody(), true);

Роут `/oauth/token` вернёт JSON-отклик, содержащий атрибуты `access_token`, `refresh_token` и `expires_in`. Атрибут `expires_in` содержит число секунд до истечения действия токена доступа.

<a name="password-grant-tokens"></a>
## Токены предоставления пароля

Предоставление пароля OAuth2 позволяет вашим собственным клиентам, таким как мобильное приложение, получить токен доступа с помощью e-mail/логина и пароля. Это позволяет вам безопасно выдавать токены доступа своим собственным клиентам, не требуя от пользователя проходить через весь процесс авторизационной переадресации OAuth2.

<a name="creating-a-password-grant-client"></a>
### Создание клиента предоставления пароля

Перед тем как ваше приложение сможет выдавать токены через предоставление пароля, вам надо создать клиент предоставления пароля. Это можно сделать командой `passport:client` с ключом `--password`. Если вы уже выполняли команду `passport:install`, то вам не надо выполнять эту команду:

    php artisan passport:client --password

<a name="requesting-password-grant-tokens"></a>
### Запрос токенов

После создания клиента предоставления пароля вы можете запросить токен, сделав `POST`-запрос роуту `/oauth/token` с адресом email и паролем пользователя. Запомните, этот роут уже зарегистрирован методом `Passport::routes`, поэтому вам не надо определять его вручную. Если запрос успешен, вы получите `access_token` и `refresh_token` в JSON-отклике от сервера:

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '',
        ],
    ]);

    return json_decode((string) $response->getBody(), true);

> {tip} Запомните, по умолчанию токены долговременны. Но вы можете [настроить максимальное время своих токенов доступа](#configuration) при необходимости.

<a name="requesting-all-scopes"></a>
### Запрос всех прав

При использовании предоставления пароля вам может понадобиться авторизовать токен для всех прав, поддерживаемых в вашем приложении. Это можно сделать, запросив право `*`. Если вы запросили право `*`, метод `can` на экземпляре токена будет всегда возвращать `true`. Это право может быть выдано только токену, выданному с помощью предоставления `password`:

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '*',
        ],
    ]);

<a name="implicit-grant-tokens"></a>
## Неявное предоставление токенов

Неявное предоставление похоже на предоставление авторизационного кода, но токен возвращается клиенту без обмена авторизационным кодом. Это предоставление наиболее часто используется для JavaScript или мобильных приложений, когда клиентские учётные данные не могут надёжно храниться. Для включения предоставления вызовите метод `enableImplicitGrant` в `AuthServiceProvider`:

    /**
     * Регистрация всех сервисов аутентификации / авторизации.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::enableImplicitGrant();
    }

После включения предоставления разработчики смогут использовать ID своего клиента для запроса токенов доступа у вашего приложения. Запрашивающее приложение должно сделать запрос переадресации по роуту `/oauth/authorize` следующим образом:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'token',
            'scope' => '',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} Запомните, роут `/oauth/authorize` уже определён методом `Passport::routes`. Вам не надо определять его вручную.

<a name="client-credentials-grant-tokens"></a>
## Токены предоставления учётных данных клиента

Предоставление учётных данных клиента подходит для аутентификации машина-машина. Например, вы можете использовать это предоставление в запланированной задаче, которая выполняет обслуживающие действия через API. Чтобы использовать этот метод сначала потребуется добавить нового посредника своему `$routeMiddleware` в `app/Http/Kernel.php`:

    use Laravel\Passport\Http\Middleware\CheckClientCredentials;

    protected $routeMiddleware = [
        'client' => CheckClientCredentials::class,
    ];

Затем, привяжите этого посредника к роуту:

    Route::get('/user', function(Request $request) {
        ...
    })->middleware('client');

Для получения токена сделайте запрос к урлу `oauth/token`:

    $guzzle = new GuzzleHttp\Client;

    $response = $guzzle->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'client_credentials',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => 'your-scope',
        ],
    ]);

    echo json_decode((string) $response->getBody(), true);

<a name="personal-access-tokens"></a>
## Персональные токены доступа

Иногда пользователям необходимо выдавать токены доступа самим себе, не проходя через обычный процесс авторизационной переадресации. Позволяя пользователям выдавать токены самим себе через UI вашего приложения, вы даёте им возможность экспериментировать с вашим API или упрощаете весь подход к выдаче токенов.

> {note} Персональные токены доступа всегда долговременны. Их срок действия не изменяется методами `tokensExpireIn` и `refreshTokensExpireIn`.

<a name="creating-a-personal-access-client"></a>
### Создание клиента персонального доступа

Перед тем как ваше приложение сможет выдавать токены персонального доступа, вам надо создать клиент персонального доступа. Это можно сделать командой `passport:client` с ключом `--personal`. Если вы уже выполняли команду `passport:install`, вам не надо выполнять эту команду:

    php artisan passport:client --personal

<a name="managing-personal-access-tokens"></a>
### Управление токенами персонального доступа

После создания клиента персонального доступа вы можете выдавать токены конкретному пользователю с помощью метода `createToken` на экземпляре модели `User`. Метод `createToken` принимает первым аргументом название токена, а вторым аргументом — необязательный массив [прав токена](#token-scopes):

    $user = App\User::find(1);

    // Создание токена без указания прав...
    $token = $user->createToken('Token Name')->accessToken;

    // Создание токена с правами...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

#### JSON API

В Passport также есть JSON API для управления персональными токенами доступа. Вы можете дополнить его своим фронтендом, предоставив пользователям панель для управления токенами персонального доступа. Далее мы рассмотрим все конечные точки API для управления токенами персонального доступа. Для удобства мы используем [Axios](https://github.com/mzabriskie/axios) для демонстрации создания HTTP-запросов к конечным точкам.

> {tip} Если вы не хотите самостоятельно реализовывать весь фронтенд управления персональными токенами доступа, вы можете использовать [быстрое знакомство с фронтендом](#frontend-quickstart) и получить полностью функциональный фронтенд за считанные минуты.

#### `GET /oauth/scopes`

Этот роут возвращает все [права](#token-scopes), определённые для вашего приложения. Вы можете использовать этот роут для получения списка прав, которые пользователь может назначить персональному токену доступа:

    axios.get('/oauth/scopes')
        .then(response => {
            console.log(response.data);
        });

#### `GET /oauth/personal-access-tokens`

Этот роут возвращает все персональные токены доступа, которые создал пользователь. Это особенно полезно для получения списка всек токенов пользователя, чтобы он мог отредактировать или удалить их:

    axios.get('/oauth/personal-access-tokens')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/personal-access-tokens`

Этот роут создаёт новые персональные токены доступа. Ему необходимы следующие данные: название токена (`name`) и права (`scopes`), которые будут назначены токену:

    const data = {
        name: 'Token Name',
        scopes: []
    };

    axios.post('/oauth/personal-access-tokens', data)
        .then(response => {
            console.log(response.data.accessToken);
        })
        .catch (response => {
            // Список ошибок в отклике...
        });

#### `DELETE /oauth/personal-access-tokens/{token-id}`

Этот роут удаляет персональный токен доступа:

    axios.delete('/oauth/personal-access-tokens/' + tokenId);

<a name="protecting-routes"></a>
## Защита роутов

<a name="via-middleware"></a>
### С помощью посредников

Passport содержит [защитника аутентификации](/docs/{{version}}/authentication#adding-custom-guards), который будет проверять токены доступа во входящих запросах. После настройки защитника `api` на использование драйвера `passport` вам остаётся только указать посредника `auth:api` для всех роутов, которые требуют корректный токен доступа:

    Route::get('/user', function () {
        //
    })->middleware('auth:api');

<a name="passing-the-access-token"></a>
### Передача токена доступа

При вызове роутов, защищённых с помощью Passport, пользователи API вашего приложения должны указать свой токен доступа как токен `Bearer` в заголовке `Authorization` своего запроса. Например, при использовании HTTP-библиотеки Guzzle:

    $response = $client->request('GET', '/api/user', [
        'headers' => [
            'Accept' => 'application/json',
            'Authorization' => 'Bearer '.$accessToken,
        ],
    ]);

<a name="token-scopes"></a>
## Права токена


<a name="defining-scopes"></a>
### Определение прав

Права позволяют клиентам вашего API запрашивать определённый набор разрешений при запросе авторизации на доступ к аккаунту. Например, если вы создаёте приложение для продаж, не всем пользователям API будет нужна возможность размещать заказы. Вместо этого вы можете позволить им только запрашивать авторизацию на доступ к статусам доставки. Другими словами, права позволяют пользователям вашего приложения ограничить действия, которые может выполнять стороннее приложение от их имени.

Вы можете определить права своего API методом `Passport::tokensCan` в методе `boot` вашего `AuthServiceProvider`. Метод `tokensCan` принимает массив названий и описаний прав. Описание права может быть каким угодно и будет выводиться пользователям на экране подтверждения авторизации:

    use Laravel\Passport\Passport;

    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);

<a name="assigning-scopes-to-tokens"></a>
### Назначение прав токенам

#### При запросе кодов авторизации

При запросе токена доступа с помощью предоставления кода авторизации, запрашивающая сторона должна указать необходимые ей права в виде строкового параметра запроса `scope`. Параметр `scope` должен быть списком прав, разделённых пробелами:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => 'place-orders check-status',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

#### При выдаче персональных токенов доступа

Если вы выдаёте персональные токены доступа с помощью метода `createToken` модели `User`, вы можете передать массив необходимых вам прав в качестве второго аргумента:

    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="checking-scopes"></a>
### Проверка прав

Passport содержит два посредника, которые можно использовать для проверки того, что входящий запрос аутентифицирован токеном, которому выданы нужные права. Для начала добавьте следующих посредников в свойство `$routeMiddleware` в файле `app/Http/Kernel.php`:

    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,

#### Проверка всех прав

Посредник `scopes` можно назначить на роут для проверки того, что токен доступа входящего запроса имеет *все* перечисленные права:

    Route::get('/orders', function () {
        // Токен доступа имеет оба права: "check-status" и "place-orders"...
    })->middleware('scopes:check-status,place-orders');

#### Проверка на любое право

Посредник `scope` можно назначить на роут для проверки того, что токен доступа входящего запроса имеет *хотя бы одного* из перечисленных прав:

    Route::get('/orders', function () {
        // Токен доступа имеет либо право "check-status", либо "place-orders"...
    })->middleware('scope:check-status,place-orders');

#### Проверка прав экземпляра токена

Когда авторизованный токеном доступа запрос поступил в ваше приложение, у вас всё ещё есть возможность проверить, есть ли у токена определённое право, методом `tokenCan` на аутентифицированном экземпляре `User`:

    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            //
        }
    });

<a name="consuming-your-api-with-javascript"></a>
## Использование вашего API с помощью JavaScript

При создании API бывает чрезвычайно полезно иметь возможность использовать ваш собственный API из вашего JavaScript-приложения. Такой подход к разработке API позволяет вашему приложению использовать тот же API, который вы предоставляете всем остальным. Один и тот же API может быть использован вашим веб-приложением, мобильными приложениями, сторонними приложениями и любыми SDK, которые вы можете опубликовать в различных менеджерах пакетов.

Обычно, если вы хотите использовать свой API из вашего JavaScript-приложения, вам необходимо вручную послать токен доступа в приложение и передавать его с каждым запросом в ваше приложение. Однако, Passport содержит посредника, который может обработать это для вас. Вам надо только добавить посредника `CreateFreshApiToken` в вашу группу посредников `web`:

    'web' => [
        // Другие посредники...
        \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
    ],

Этот посредник Passport будет прикреплять куки `laravel_token` к ваших исходящим откликам. Этот куки содержит зашифрованный JWT, который Passport будет использовать для аутентификации API-запросов из вашего JavaScript-приложения. Теперь вы можете делать запросы в API вашего приложения, не передавая токен доступа в явном виде:

    axios.get('/api/user')
        .then(response => {
            console.log(response.data);
        });

При использовании этого метода аутентификации Axios будет автоматически отправлять заголовок `X-CSRF-TOKEN`. Дополнительно, создание JavaScript модулей Laravel по-умолчанию указывает Axios отправлять заголовок `X-Requested-With`:

    window.axios.defaults.headers.common = {
        'X-Requested-With': 'XMLHttpRequest',
    };

> {note} Если вы используете другой JavaScript-фреймворк, вам надо убедиться, что он настроен на отправку заголовков `X-CSRF-TOKEN` и `X-Requested-With` с каждым исходящим запросом.


<a name="events"></a>
## События

Passport создаёт события при выдаче токенов доступа и обновлении токенов. Вы можете использовать эти события для удаления или отмены других токенов доступа в вашей БД. Вы можете прикрепить слушателей к этим событиям в `EventServiceProvider` вашего приложения:

    /**
     * Привязки слушателя событий для приложения.
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

Метод Passport `actingAs` можено использовать для указания пользователя, авторизованного в данный момент, в также его прав. Первый аргумент, передаваемый методу `actingAs` - экземпляр пользователя, а второй - массив прав, который следует предоставлять токену пользователя:

    public function testServerCreation()
    {
        Passport::actingAs(
            factory(User::class)->create(),
            ['create-servers']
        );

        $response = $this->post('/api/create-server');

        $response->assertStatus(200);
    }
