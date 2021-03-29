# Laravel 8 · Примечания к релизу

- [Схема версионирования](#versioning-scheme)
    - [Исключения](#exceptions)
- [Политика поддержки](#support-policy)
- [Laravel 8](#laravel-8)

<a name="versioning-scheme"></a>
## Схема версионирования

Laravel и другие его собственные пакеты следуют [Семантическому Версионированию](https://semver.org/lang/ru/). Мажорные релизы фреймворка выпускаются каждый год (~сентябрь), тогда как минорные и патч-релизы могут выпускаться каждую неделю. Минорные и патч-релизы **никогда** не должны содержать критических изменений.

Ссылаясь на фреймворк Laravel или его компоненты из вашего приложения или пакета, вы всегда должны использовать ограничение версии `^8.0`, поскольку мажорные релизы Laravel действительно включают критические изменения. Однако мы всегда стремимся к тому, чтобы вы могли выполнить обновление до новой мажорной версии в течение дня или менее.

<a name="exceptions"></a>
### Исключения

<a name="named-arguments"></a>
#### Именованные аргументы

В настоящее время функциональные возможности [именованных аргументов](https://www.php.net/manual/ru/functions.arguments.php#functions.named-arguments) PHP не подпадают под правила обратной совместимости Laravel. При необходимости мы можем переименовать аргументы функции, чтобы улучшить кодовую базу Laravel. Поэтому использовать именованные аргументы при вызове методов Laravel следует осторожно и с пониманием того, что их имена могут измениться в будущем.

<a name="support-policy"></a>
## Политика поддержки

Для релизов LTS, таких как Laravel 6, исправления ошибок предоставляются в течение 2 лет, а исправления безопасности – в течение 3 лет. Эти релизы предоставляют самый продолжительный период поддержки и обслуживания. Для основных релизов, исправления ошибок предоставляются в течение 18 месяцев, а исправления безопасности – в течение 2 лет. Для всех дополнительных библиотек, включая Lumen, только последний релиз получает исправления ошибок. Помимо этого, ознакомьтесь с версиями баз данных, которые [поддерживает Laravel](/docs/database.md#introduction).

| Версия | Дата релиза | Исправление ошибок до | Исправления безопасности до |
| --- | --- | --- | --- |
| 6 (LTS) | September 3rd, 2019 | September 7th, 2021 | September 6rd, 2022 |
| 7 | March 3rd, 2020 | October 6th, 2020 | March 3rd, 2021 |
| 8 | September 8th, 2020 | April 6th, 2021 | September 8th, 2021 |

<a name="laravel-8"></a>
## Laravel 8

Laravel 8 продолжает улучшения, сделанные в Laravel 7.x, представляя Laravel Jetstream, классы фабрики модели, сжатие миграций, пакетную обработку заданий, улучшенное ограничение частоты запросов, улучшения очереди, динамические компоненты Blade, постраничная навигация с использованием Tailwind, помощники по временному тестированию, улучшения в `artisan serve`, улучшения слушателей событий и множество других исправлений ошибок и улучшений юзабилити.

<a name="laravel-jetstream"></a>
### Laravel Jetstream

_Laravel Jetstream был написан [Taylor Otwell](https://github.com/taylorotwell)_.

[Laravel Jetstream](https://jetstream.laravel.com) это красиво оформленный каркас приложений для Laravel. Jetstream обеспечивает идеальную отправную точку для вашего следующего проекта и включает в себя вход в систему, регистрацию, проверку электронной почты, двухфакторную аутентификацию, управление сессией, поддержку API через Laravel Sanctum и дополнительное командное управление. Laravel Jetstream заменяет и улучшает устаревшую структуру пользовательского интерфейса аутентификации, доступную в предыдущих версиях Laravel.

Jetstream разработан с использованием [Tailwind CSS](https://tailwindcss.com) и предлагает на ваш выбор каркасы [Livewire](https://laravel-livewire.com) или [Inertia](https://inertiajs.com).

<a name="models-directory"></a>
### Каталог моделей

По многочисленным просьбам сообщества каркас приложения Laravel по умолчанию теперь содержит каталог `app/Models`. Надеемся, вам понравится это решение для ваших моделей Eloquent! Все соответствующие команды генератора были обновлены, чтобы предполагать, что модели существуют в каталоге `app/Models`, если он существует. Если каталог не существует, фреймворк предполагает, что ваши модели должны быть помещены в каталог `app`.

<a name="model-factory-classes"></a>
### Классы фабрики модели

_Автор: [Taylor Otwell](https://github.com/taylorotwell)_.

[Фабрики модели](/docs/database-testing.md#defining-model-factories) Eloquent были полностью переписаны как фабрики на основе классов и улучшены для обеспечения "first-class" поддержки отношений. Например, `UserFactory`, включенный в Laravel, написан так:

    <?php

    namespace Database\Factories;

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Factory;
    use Illuminate\Support\Str;

    class UserFactory extends Factory
    {
        /**
         * Название модели соответствующей фабрики.
         *
         * @var string
         */
        protected $model = User::class;

        /**
         * Определить состояние модели по умолчанию.
         *
         * @return array
         */
        public function definition()
        {
            return [
                'name' => $this->faker->name,
                'email' => $this->faker->unique()->safeEmail,
                'email_verified_at' => now(),
                'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // пароль
                'remember_token' => Str::random(10),
            ];
        }
    }

Благодаря новому трейту `HasFactory`, доступному для сгенерированных моделей, фабрика модели может использоваться следующим образом:

    use App\Models\User;

    User::factory()->count(50)->create();

Поскольку фабрики модели теперь являются простыми классами PHP, преобразования состояний могут быть записаны как методы класса. Кроме того, при необходимости вы можете добавить любые другие вспомогательные классы в фабрику модели Eloquent.

Например, ваша модель `User` может находиться в состоянии `suspended`, которое изменяет одно из значений ее атрибутов по умолчанию. Вы можете определить свои преобразования состояния, используя метод `state` базовой фабрики. Вы можете называть свой метод состояния как угодно. В конце концов, это просто типичный PHP-метод:

    /**
     * Указать, что аккаунт пользователя временно приостановлен.
     *
     * @return \Illuminate\Database\Eloquent\Factories\Factory
     */
    public function suspended()
    {
        return $this->state([
            'account_status' => 'suspended',
        ]);
    }

После определения метода преобразования состояния мы можем использовать его так:

    use App\Models\User;

    User::factory()->count(5)->suspended()->create();

Как уже упоминалось, фабрики модели Laravel 8 содержат "first-class" поддержку отношений. Итак, предполагая, что наша модель `User` имеет метод-отношение `posts`, мы можем просто запустить следующий код для создания пользователя с тремя постами:

    $users = User::factory()
                ->hasPosts(3, [
                    'published' => false,
                ])
                ->create();

Чтобы упростить процесс обновления, был выпущен пакет [laravel/legacy-factories](https://github.com/laravel/legacy-factories), обеспечивающий поддержку предыдущей итерации фабрик модели в Laravel 8.x.

Переписанные фабрики Laravel содержат гораздо больше функций, которые, как мы думаем, вам понравятся. Чтобы узнать больше о фабриках моделей, обратитесь к [документации по тестированию баз данных](/docs/database-testing.md#defining-model-factories).

<a name="migration-squashing"></a>
### Сжатие миграций

_Автор: [Taylor Otwell](https://github.com/taylorotwell)_.

По мере создания приложения вы можете со временем накапливать все больше и больше миграций. Это может привести к тому, что каталог миграций станет раздутым из-за потенциально сотен миграций. Если вы используете MySQL или PostgreSQL, теперь вы можете «сжать» свои миграции в один файл SQL. Для начала выполните команду `schema:dump`:

    php artisan schema:dump

    // Выгрузить текущую схему БД и удалить все существующие миграции ...
    php artisan schema:dump --prune

Когда вы выполните эту команду, Laravel запишет файл «схемы» в каталог `database/schema` вашего приложения. Теперь, когда вы попытаетесь перенести свою базу данных, Laravel сначала выполнит SQL-операторы файла схемы, при условии, что никакие другие миграции не выполнялись. После выполнения команд файла схемы, Laravel выполнит все оставшиеся миграции, которые не были включены в дамп схемы БД.

<a name="job-batching"></a>
### Пакетная обработка заданий

_Авторы: [Taylor Otwell](https://github.com/taylorotwell) и [Mohamed Said](https://github.com/themsaid)_.

Функционал пакетной обработки заданий Laravel позволяет вам легко выполнить пакет заданий, по завершению которого дополнительно совершить определенные действия.

Новый метод `batch` фасада `Bus` используется для отправки пакета заданий. Конечно, это в первую очередь полезно в сочетании с замыканиями по завершению. Итак, вы можете использовать методы `then`, `catch`, и `finally` для определения замыканий пакета заданий. Каждый из этих замыканий получит экземпляр `Illuminate\Bus\Batch` при вызове:

    use App\Jobs\ProcessPodcast;
    use App\Podcast;
    use Illuminate\Bus\Batch;
    use Illuminate\Support\Facades\Bus;
    use Throwable;

    $batch = Bus::batch([
        new ProcessPodcast(Podcast::find(1)),
        new ProcessPodcast(Podcast::find(2)),
        new ProcessPodcast(Podcast::find(3)),
        new ProcessPodcast(Podcast::find(4)),
        new ProcessPodcast(Podcast::find(5)),
    ])->then(function (Batch $batch) {
        // Все задания успешно завершены ...
    })->catch(function (Batch $batch, Throwable $e) {
        // Обнаружено первое проваленное задание из пакета ...
    })->finally(function (Batch $batch) {
        // Завершено выполнение пакета ...
    })->dispatch();

    return $batch->id;

Чтобы узнать больше о пакетной обработки заданий, обратитесь к [документации по очередям](/docs/queues.md#job-batching).

<a name="improved-rate-limiting"></a>
### Улучшенное ограничение частоты запросов

_Автор: [Taylor Otwell](https://github.com/taylorotwell)_.

Функционал ограничения частоты запросов в Laravel был расширен за счет большей гибкости и возможностей, при этом сохранена обратная совместимость с API посредника `throttle` предыдущих релизов.

Ограничители определяются с помощью метода `for` фасада `RateLimiter`. Метод `for` принимает имя ограничителя и замыкание, которое возвращает конфигурацию ограничений, применяемых к назначенным маршрутам:

    use Illuminate\Cache\RateLimiting\Limit;
    use Illuminate\Support\Facades\RateLimiter;

    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000);
    });

Поскольку замыкание получает экземпляр входящего HTTP-запроса, вы можете динамически создать ограничение на основе входящего запроса или статуса аутентификации пользователя:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100);
    });

Иногда может потребоваться сегментация ограничений по некоторым произвольным значениям. Например, вы можете разрешить пользователям получать доступ к указанному маршруту 100 раз в минуту на каждый IP-адрес. Для этого можно использовать метод `by` при построении лимита:

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100)->by($request->ip());
    });

Ограничители могут быть закреплены за маршрутами или группами маршрутов с помощью [посредника](/docs/middleware) `throttle`. Посредник `throttle` принимает имя ограничителя, которое вы хотите назначить маршруту:

    Route::middleware(['throttle:uploads'])->group(function () {
        Route::post('/audio', function () {
            //
        });

        Route::post('/video', function () {
            //
        });
    });

Чтобы узнать больше об ограничителях запросов, обратитесь к [документации по маршрутизации](/docs/routing.md#rate-limiting).

<a name="improved-maintenance-mode"></a>
### Улучшенный режим обслуживания

_Автор: [Taylor Otwell](https://github.com/taylorotwell), вдохновленный [Spatie](https://spatie.be)_.

В предыдущих релизах Laravel функционал режима обслуживания `php artisan down` можно было обойти с помощью «разрешенного списка» IP-адресов, имеющим доступ к приложению. Эта функция была удалена в пользу более простого токен-решения.

Находясь в режиме обслуживания, вы можете использовать параметр `secret`, чтобы указать токен обхода режима обслуживания:

    php artisan down --secret="1630542a-246b-4b66-afa1-dd72a4c43515"

После перевода приложения в режим обслуживания вы можете перейти по URL-адресу приложения, соответствующему этому токену, и Laravel выдаст вашему браузеру файл cookie обхода режима обслуживания:

    https://example.com/1630542a-246b-4b66-afa1-dd72a4c43515

При доступе к этому скрытому маршруту вы будете перенаправлены на корневой маршрут приложения. Как только cookie будет отправлен вашему браузеру, вы сможете просматривать приложение в обычном режиме, как если бы оно не находилось в режиме обслуживания.

<a name="pre-rendering-the-maintenance-mode-view"></a>
#### Предварительный рендеринг шаблона режима обслуживания

Если вы используете команду `php artisan down` во время развертывания, ваши пользователи могут иногда сталкиваться с ошибками, если они обращаются к приложению во время обновления ваших зависимостей Composer или других компонентов фреймворка. Это происходит потому, что значительная часть фреймворка Laravel должна загружаться, чтобы определить, находится ли ваше приложение в режиме обслуживания, и отобразить шаблон режима обслуживания с помощью механизма шаблонов.

По этой причине Laravel позволяет предварительно визуализировать шаблон режима обслуживания, который будет возвращен в самом начале цикла запроса. Этот шаблон отображается перед загрузкой любых зависимостей вашего приложения. Вы можете выполнить предварительный рендеринг шаблона по вашему выбору, используя параметр `render` команды `down`:

    php artisan down --render="errors::503"

<a name="closure-dispatch-chain-catch"></a>
### Выполнение замыканий с использованием цепочки `catch`

_Автор: [Mohamed Said](https://github.com/themsaid)_.

Используя новый метод `catch`, теперь можно определить замыкание, которое должно быть выполнено, если замыкание, указанное в очереди не удалось успешно завершить при исчерпании попыток повтора очереди:

    use Throwable;

    dispatch(function () use ($podcast) {
        $podcast->publish();
    })->catch(function (Throwable $e) {
        // Не удалось выполнить это задание ...
    });

<a name="dynamic-blade-components"></a>
### Динамические компоненты Blade

_Автор: [Taylor Otwell](https://github.com/taylorotwell)_.

Иногда может потребоваться отрисовать компонент, но вы не знаете, какой именно компонент это будет до момента выполнения. В этой ситуации вы можете использовать встроенный в Laravel компонент `dynamic-component` для рендеринга компонента, зависящего от значения или переменной, сформированных во время выполнения приложения:

    <x-dynamic-component :component="$componentName" class="mt-4" />

Чтобы узнать больше о компонентах Blade, обратитесь к [документации Blade](/docs/blade.md#components).

<a name="event-listener-improvements"></a>
### Улучшения слушателей событий

_Автор: [Taylor Otwell](https://github.com/taylorotwell)_.

Слушатели событий, основанные на замыкании, теперь регистрируются только путем передачи замыкания методу `Event::listen`. Laravel проверит замыкание, чтобы определить, какой тип события обрабатывает слушатель:

    use App\Events\PodcastProcessed;
    use Illuminate\Support\Facades\Event;

    Event::listen(function (PodcastProcessed $event) {
        //
    });

Кроме того, анонимные слушатели событий теперь могут быть помечены как доступные для очереди с помощью функции `Illuminate\Events\queueable`:

    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;

    Event::listen(queueable(function (PodcastProcessed $event) {
        //
    }));

Как и задания в очереди, вы можете использовать методы `onConnection`, `onQueue`, и `delay` для настройки выполнения слушателя в очереди:

    Event::listen(queueable(function (PodcastProcessed $event) {
        //
    })->onConnection('redis')->onQueue('podcasts')->delay(now()->addSeconds(10)));

Если вы хотите обработать сбои запланированного анонимного слушателя, то предоставьте замыкание методу `catch` при определении `queueable`-слушателя:

    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;
    use Throwable;

    Event::listen(queueable(function (PodcastProcessed $event) {
        //
    })->catch(function (PodcastProcessed $event, Throwable $e) {
        // Не удалось выполнить запланированного слушателя ...
    }));

<a name="time-testing-helpers"></a>
### Помощники по временному тестированию

_Автор: [Taylor Otwell](https://github.com/taylorotwell), вдохновленный Ruby on Rails_.

При тестировании вам может иногда потребоваться изменить время, возвращаемое такими помощниками, как `now` или `Illuminate\Support\Carbon::now()`. Базовый класс тестирования функций Laravel теперь включает помощников, которые позволяют вам управлять текущим временем:

    public function testTimeCanBeManipulated()
    {
        // Путешествие в будущее ...
        $this->travel(5)->milliseconds();
        $this->travel(5)->seconds();
        $this->travel(5)->minutes();
        $this->travel(5)->hours();
        $this->travel(5)->days();
        $this->travel(5)->weeks();
        $this->travel(5)->years();

        // Путешествие в прошлое ...
        $this->travel(-5)->hours();

        // Путешествие в определенное время ...
        $this->travelTo(now()->subHours(6));

        // Вернуться в настоящее время ...
        $this->travelBack();
    }

<a name="artisan-serve-improvements"></a>
### Улучшения Artisan `serve`

_Автор: [Taylor Otwell](https://github.com/taylorotwell)_.

Команда `serve` консоли Artisan была улучшена за счет автоматической перезагрузки при обнаружении изменений переменных окружения в вашем локальном файле `.env`. Раньше команду приходилось останавливать и перезапускать вручную.

<a name="tailwind-pagination-views"></a>
### Постраничная навигация с использованием Tailwind

Пагинатор Laravel был обновлен для использования фреймворка [Tailwind CSS](https://tailwindcss.com) по умолчанию. Tailwind CSS – это настраиваемая низкоуровневая структура CSS, которая дает вам все строительные блоки, необходимые для создания нестандартных дизайнов без каких-либо раздражающих самоуверенных стилей, за которые вам придется бороться. Конечно, также остаются доступными шаблоны Bootstrap 3 и 4.

<a name="routing-namespace-updates"></a>
### Обновления пространства имен маршрутизации

В предыдущих релизах Laravel `RouteServiceProvider` содержал свойство `$namespace`. Значение этого свойства будет автоматически добавлено к определениям маршрута контроллера и вызовам помощника `action` / метода `URL::action`. В Laravel 8.x это свойство по умолчанию имеет значение `null`. Это означает, что Laravel не будет автоматически подставлять префиксы пространства имен. Поэтому в новых приложениях Laravel 8.x определения маршрутов контроллера должны быть определены с использованием стандартного синтаксиса объявлений зависимостей PHP:

    use App\Http\Controllers\UserController;

    Route::get('/users', [UserController::class, 'index']);

Вызов методов, связанных с `action`, должен использовать тот же синтаксис объявлений:

    action([UserController::class, 'index']);

    return Redirect::action([UserController::class, 'index']);

Если вы предпочитаете префиксирование маршрута контроллера в стиле Laravel версии 7.x, вы можете просто добавить свойство `$namespace` в `RouteServiceProvider` вашего приложения.

> {note} Это изменение касается только новых приложений Laravel 8.x. Приложения, обновляющиеся с Laravel 7.x, по-прежнему будут иметь свойство `$namespace` в своем `RouteServiceProvider`.
