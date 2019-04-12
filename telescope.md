git 95eeb84aa8767fbda1d75408b46059db474fa260

---

# Laravel Telescope

- [Введение](#introduction)
- [Установка](#installation)
    - [Настройка](#configuration)
    - [Очистка данных](#data-pruning)
    - [Настройка миграции](#migration-customization)
- [Авторизация в рабочей панели ](#dashboard-authorization)
- [Фильтрация](#filtering)
    - [Записи](#filtering-entries)
    - [Пакеты](#filtering-batches)
- [Доступные наблюдатели](#available-watchers)
    - [Наблюдатель кэша](#cache-watcher)
    - [Наблюдатель команд](#command-watcher)
    - [Наблюдатель дампов](#dump-watcher)
    - [Наблюдатель событий](#event-watcher)
    - [Наблюдатель исключений](#exception-watcher)
    - [Наблюдатель шлюза](#gate-watcher)
    - [Наблюдатель заданий](#job-watcher)
    - [Наблюдатель журналов](#log-watcher)
    - [Наблюдатель электронной почты](#mail-watcher)
    - [Наблюдатель модели](#model-watcher)
    - [Наблюдатель уведомлений](#notification-watcher)
    - [Наблюдатель SQL запросов](#query-watcher)
    - [Наблюдатель Redis](#redis-watcher)
    - [Наблюдатель запросов](#request-watcher)
    - [Наблюдатель расписания](#schedule-watcher)

<a name="introduction"></a>
## Введение

Laravel Telescope - элегантный помощник для отладки вашего приложения Laravel. Telescope обеспечивает понимание запросов, поступающих в ваше приложение, исключений, записей журнала, запросов к базе данных, заданий в очереди, почты, уведомлений, операций кэширования, запланированных задач, дампов переменных и многого другого. Телескоп - прекрасный компаньон для вашей локальной среды разработки Laravel.

<p align="center">
<img src="https://res.cloudinary.com/dtfbvvkyp/image/upload/v1539110860/Screen_Shot_2018-10-09_at_1.47.23_PM.png" width="600" height="347">
</p>

<a name="installation"></a>
## Установка

Вы можете использовать Composer для установки Telescope в ваш проект Laravel:

    composer require laravel/telescope

После установки Telescope опубликуйте его ресурсы с помощью Artisan-команды `telescope:install`. После установки Telescope вы также должны выполнить команду `migrate`:

    php artisan telescope:install

    php artisan migrate

#### Обновление Telescope

При обновлении Telescope вы должны повторно опубликовать ресурсы:

    php artisan telescope:publish

### Установка только в определенных средах

Если вы планируете использовать Telescope, только в локальной разработке, вы можете его установить, используя флаг `--dev`:

    composer require laravel/telescope --dev

После запуска `telescope:install` вы должны удалить регистрацию сервис-провайдера `TelescopeServiceProvider` из вашего файла конфигурации `app`. Вместо этого вручную зарегистрируйте сервис-провайдер в методе `register` вашего `AppServiceProvider`:

    use Laravel\Telescope\TelescopeServiceProvider;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        if ($this->app->isLocal()) {
            $this->app->register(TelescopeServiceProvider::class);
        }
    }

<a name="migration-customization"></a>
### Настройка миграции

Если вы не собираетесь использовать миграции Telescope по умолчанию, вам следует вызвать метод `Telescope::ignoreMigrations` в методе `register` вашего `AppServiceProvider`. Вы можете экспортировать миграции по умолчанию, используя команду `php artisan vendor:publish --tag=telescope-migrations`.

<a name="configuration"></a>
### Настройка

После публикации ресурсов Telescope его основной файл конфигурации будет находиться по адресу `config/telescope.php`. Этот файл конфигурации позволяет вам настроить параметры наблюдателя, каждый параметр конфигурации содержит описание его назначения, поэтому обязательно внимательно изучите этот файл.

При желании вы можете полностью отключить сбор данных телескопа, используя опцию конфигурации `enabled`:

    'enabled' => env('TELESCOPE_ENABLED', true),

<a name="data-pruning"></a>
### Очистка данных

Без очистки данных таблица `telescope_entries` может очень быстро разрастись записями. Чтобы избежать этого, вы должны запланировать ежедневную Artisan-команду `telescope:prune`:

    $schedule->command('telescope:prune')->daily();

По умолчанию все записи старше 24 часов будут удалены. Вы можете использовать параметр `hours` при вызове команды, чтобы определить, как долго хранить данные Telescope. Например, следующая команда удалит все записи, созданные более 48 часов назад:

    $schedule->command('telescope:prune --hours=48')->daily();

<a name="dashboard-authorization"></a>
## Авторизация в рабочей панели 

Telescope предоставляет рабочую панель по адресу `/telescope`. По умолчанию вы сможете получить доступ к этой панели только в `local` среде. В вашем файле `app/Providers/TelescopeServiceProvider.php` есть метод `gate`. Этот шлюз авторизации контролирует доступ к телескопу в **нелокальных** средах. Вы можете изменять этот шлюз по мере необходимости, чтобы ограничить доступ к установке вашего Telescope:

    /**
     * Register the Telescope gate.
     *
     * This gate determines who can access Telescope in non-local environments.
     *
     * @return void
     */
    protected function gate()
    {
        Gate::define('viewTelescope', function ($user) {
            return in_array($user->email, [
                'taylor@laravel.com',
            ]);
        });
    }

<a name="filtering"></a>
## Фильтрация

<a name="filtering-entries"></a>
### Записи

Вы можете фильтровать данные, записанные Telescope, с помощью функции обратного вызова `filter`, которая зарегистрирована в вашем `TelescopeServiceProvider`. По умолчанию эта функция обратного вызова записывает все данные в `local` среде и исключения, задания с ошибкой, запланированные задачи и данные с отслеживаемыми тегами во всех других средах:

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->hideSensitiveRequestDetails();

        Telescope::filter(function (IncomingEntry $entry) {
            if ($this->app->isLocal()) {
                return true;
            }

            return $entry->isReportableException() ||
                $entry->isFailedJob() ||
                $entry->isScheduledTask() ||
                $entry->hasMonitoredTag();
        });
    }

<a name="filtering-batches"></a>
### Пакеты

В то время как функция обратного вызова `filter` фильтрует данные для отдельных записей, вы можете использовать метод `filterBatch` для регистрации функции обратного вызова, которая фильтрует все данные для данного запроса или консольной команды. Если функция обратного вызова возвращает `true`, все записи записываются Telescope:

    use Illuminate\Support\Collection;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->hideSensitiveRequestDetails();

        Telescope::filterBatch(function (Collection $entries) {
            if ($this->app->isLocal()) {
                return true;
            }

            return $entries->contains(function ($entry) {
                return $entry->isReportableException() ||
                    $entry->isFailedJob() ||
                    $entry->isScheduledTask() ||
                    $entry->hasMonitoredTag();
                });
        });
    }

<a name="available-watchers"></a>
## Доступные наблюдатели

Наблюдатели Telescope собирают данные приложения при выполнении запроса или консольной команды. Вы можете настроить список наблюдателей, которые вы хотели бы включить в конфигурационном файле `config/telescope.php`:

    'watchers' => [
        Watchers\CacheWatcher::class => true,
        Watchers\CommandWatcher::class => true,
        ...
    ],

Некоторые наблюдатели также предоставляют дополнительные параметры настройки:

    'watchers' => [
        Watchers\QueryWatcher::class => [
            'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
            'slow' => 100,
        ],
        ...
    ],

<a name="cache-watcher"></a>
### Наблюдатель кэша

Наблюдатель кэша записывает данные, когда ключ кэша удаляется, пропускается, обновляется и забывается.

<a name="command-watcher"></a>
### Наблюдатель команд

Наблюдатель команд записывает аргументы, параметры, код выхода и выходные данные при каждом выполнении Artisan-команды. Если вы хотите исключить определенные команды из записи наблюдателем, вы можете указать команду в опции `ignore` в вашем файле `config/telescope.php`:

    'watchers' => [
        Watchers\CommandWatcher::class => [
            'enabled' => env('TELESCOPE_COMMAND_WATCHER', true),
            'ignore' => ['key:generate'],
        ],
        ...
    ],

<a name="dump-watcher"></a>
### Наблюдатель дампов

Наблюдатель дампов записывает и отображает ваши переменные дампы в Telescope. При использовании Laravel переменные могут быть сброшены с помощью глобальной функции `dump`. Вкладка наблюдателя дампов должна быть открыта в браузере, чтобы запись происходила, иначе наблюдатели будут игнорировать дампы.

<a name="event-watcher"></a>
### Наблюдатель событий

Наблюдатель событий записывает полезную нагрузку, слушателей и широковещательные данные для любых событий, отправляемых вашим приложением. Внутренние события фреймворка Laravel игнорируются наблюдателем событий.

<a name="exception-watcher"></a>
### Наблюдатель исключений

Наблюдатель исключений записывает данные и трассировку стека для любых регистрируемых исключений, которые выдает ваше приложение.

<a name="gate-watcher"></a>
### Наблюдатель шлюза

Наблюдатель шлюза записывает данные и результаты проверок шлюза и политики вашим приложением. Если вы хотите исключить определенные способности от записи наблюдателем, вы можете указать их в параметре `ignore_abilities` в вашем файле `config/telescope.php`:

    'watchers' => [
        Watchers\GateWatcher::class => [
            'enabled' => env('TELESCOPE_GATE_WATCHER', true),
            'ignore_abilities' => ['viewNova'],
        ],
        ...
    ],

<a name="job-watcher"></a>
### Наблюдатель заданий

Наблюдатель заданий записывает данные и состояние всех заданий, отправленных вашим приложением.

<a name="log-watcher"></a>
### Наблюдатель журналов

Наблюдатель журналов записывает данные для любых журналов, написанных вашим приложением.

<a name="mail-watcher"></a>
### Наблюдатель электронной почты

Наблюдатель электронной почты позволяет просматривать предварительный просмотр электронных писем в браузере вместе с соответствующими данными. Вы также можете скачать письмо в виде файла `.eml`.

<a name="model-watcher"></a>
### Наблюдатель модели

Наблюдатель модели записывает изменения модели всякий раз, когда отправляется событие Eloquent `created` `updated` `restored` или `deleted` Вы можете указать, какие события модели должны быть записаны с помощью опции `events` наблюдателя:

    'watchers' => [
        Watchers\ModelWatcher::class => [
            'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
            'events' => ['eloquent.created*', 'eloquent.updated*'],
        ],
        ...
    ],

<a name="notification-watcher"></a>
### Наблюдатель уведомлений

Наблюдатель уведомлений записывает все уведомления, отправленные вашим приложением. Если уведомление вызывает сообщение электронной почты, и у вас включен наблюдатель за электронной почтой, сообщение также будет доступно для предварительного просмотра на экране наблюдателя электронной почты.

<a name="query-watcher"></a>
### Наблюдатель SQL запросов

Наблюдатель SQL запросов записывает необработанный SQL, привязки и время выполнения для всех запросов, которые выполняются вашим приложением. Наблюдатель также помечает любые запросы медленнее 100 мс как `slow` Вы можете настроить порог медленного запроса, используя параметр `slow` наблюдателя:

    'watchers' => [
        Watchers\QueryWatcher::class => [
            'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
            'slow' => 50,
        ],
        ...
    ],

<a name="redis-watcher"></a>
### Наблюдатель Redis

> {note} События Redis должны быть включены, чтобы наблюдатель Redis работал. Вы можете включить события Redis, вызвав `Redis::enableEvents()` в методе `boot` вашего файла `app/Providers/AppServiceProvider.php`.

Наблюдатель Redis записывает все команды Redis, выполненные вашим приложением. Если вы используете Redis для кэширования, команды кэширования также будут записываться наблюдателем Redis.

<a name="request-watcher"></a>
### Наблюдатель запросов

Наблюдатель запросов записывает данные запроса, заголовков, сеанса и ответа, связанные с любыми запросами, обрабатываемыми приложением. Вы можете ограничить данные своего ответа с помощью опции `size_limit` (в КБ):

    'watchers' => [
        Watchers\RequestWatcher::class => [
            'enabled' => env('TELESCOPE_REQUEST_WATCHER', true),
            'size_limit' => env('TELESCOPE_RESPONSE_SIZE_LIMIT', 64),
        ],
        ...
    ],

<a name="schedule-watcher"></a>
### Наблюдатель расписания

Наблюдатель расписания записывает команды и выходные данные любых запланированных задач, выполняемых вашим приложением.
