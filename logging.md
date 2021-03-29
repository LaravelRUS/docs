# Laravel 8 · Логирование

- [Введение](#introduction)
- [Конфигурирование](#configuration)
    - [Доступные драйверы канала](#available-channel-drivers)
    - [Предварительная подготовка канала](#channel-prerequisites)
- [Построение стека журналов](#building-log-stacks)
- [Запись сообщений журнала](#writing-log-messages)
    - [Запись в определенные каналы](#writing-to-specific-channels)
- [Настройка канала Monolog](#monolog-channel-customization)
    - [Настройка Monolog для каналов](#customizing-monolog-for-channels)
    - [Создание обработчика каналов Monolog](#creating-monolog-handler-channels)
    - [Создание каналов через фабрики](#creating-custom-channels-via-factories)

<a name="introduction"></a>
## Введение

Чтобы помочь вам узнать больше о том, что происходит в вашем приложении, Laravel предлагает надежные службы ведения журнала, которые позволяют записывать сообщения в файлы, журнал системных ошибок и даже в Slack, чтобы уведомить всю вашу команду.

Ведение журнала Laravel основано на «каналах». Каждый канал представляет собой определенный способ записи информации журнала. Например, канал `single` записывает файлы журнала в один файл журнала, а канал `slack` отправляет сообщения журнала в Slack. Сообщения журнала могут быть записаны в несколько каналов в зависимости от их серьезности.

Под капотом Laravel использует библиотеку [Monolog](https://github.com/Seldaek/monolog), которая обеспечивает поддержку множества мощных обработчиков журналов. Laravel упрощает настройку этих обработчиков, позволяя вам смешивать и сопоставлять их для настройки обработки журналов вашего приложения.

<a name="configuration"></a>
## Конфигурирование

Все параметры конфигурации для ведения журнала вашего приложения размещены в файле конфигурации `config/logging.php`. Этот файл позволяет вам настраивать каналы журнала вашего приложения, поэтому обязательно просмотрите каждый из доступных каналов и их параметры. Ниже мы рассмотрим несколько распространенных вариантов.

По умолчанию Laravel будет использовать канал `stack` при регистрации сообщений. Канал `stack` используется для объединения нескольких каналов журнала в один канал. Для получения дополнительной информации о построении стеков ознакомьтесь с [документацией ниже](#building-log-stacks).

<a name="configuring-the-channel-name"></a>
#### Настройка имени канала

По умолчанию экземпляр Monolog создается с «именем канала», которое соответствует текущей среде, например, `production` или `local`. Чтобы изменить это значение, добавьте параметр `name` в конфигурацию вашего канала:

    'stack' => [
        'driver' => 'stack',
        'name' => 'channel-name',
        'channels' => ['single', 'slack'],
    ],

<a name="available-channel-drivers"></a>
### Доступные драйверы канала

Каждый канал журнала работает через «драйвер». Драйвер определяет, как и где фактически записывается сообщение журнала. Следующие драйверы канала журнала доступны в каждом приложении Laravel. Запись для большинства этих драйверов уже присутствует в файле конфигурации вашего приложения `config/logging.php`, поэтому обязательно просмотрите этот файл, чтобы ознакомиться с его содержимым:

Имя | Описание
------------- | -------------
`custom` | Драйвер, который вызывает указанную фабрику для создания канала.
`daily` | Драйвер Monolog на основе `RotatingFileHandler` с ежедневной ротацией.
`errorlog` | Драйвер Monolog на основе `ErrorLogHandler`.
`monolog` | Драйвер фабрики Monolog, использующий любой поддерживаемый Monolog обработчик.
`null` | Драйвер, который игнорирует все сообщения.
`papertrail` | Драйвер Monolog на основе `SyslogUdpHandler`.
`single` | Канал на основе одного файла или пути (`StreamHandler`)
`slack` | Драйвер Monolog на основе `SlackWebhookHandler`.
`stack` | Обертка для облегчения создания «многоканальных» каналов.
`syslog` | Драйвер Monolog на основе `SyslogHandler`.

> {tip} Изучите документацию по [расширенной настройке канала](#monolog-channel-customization), чтобы узнать больше о драйверах `monolog` и `custom`.

<a name="channel-prerequisites"></a>
### Предварительная подготовка канала

<a name="configuring-the-single-and-daily-channels"></a>
#### Конфигурирование каналов Single и Daily

Каналы `single` и `daily` имеют три необязательных параметра конфигурации: `bubble`, `permission`, и `locking`.

Имя | Описание | По умолчанию
------------- | ------------- | -------------
`bubble` | Должны ли сообщения переходить в другие каналы после обработки | `true`
`locking` | Попытаться заблокировать файл журнала перед записью в него | `false`
`permission` | Права доступа на файл журнала | `0644`

<a name="configuring-the-papertrail-channel"></a>
#### Конфигурирование канала Papertrail

Для канала `papertrail` требуются параметры конфигурации `host` и `port`. Эти значения можно получить из [Papertrail](https://help.papertrailapp.com/kb/configuration/configuring-centralized-logging-from-php-apps/#send-events-from-php-app).

<a name="configuring-the-slack-channel"></a>
#### Конфигурирование канала Slack

Для канала `slack` требуется параметр конфигурации `url`. Этот URL-адрес должен соответствовать URL-адресу [входящего веб-хука](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks), который вы настроили для своей команды Slack.

По умолчанию Slack будет получать логи только с уровнем `critical` и выше; однако вы можете настроить это в своем файле конфигурации `config/logging.php`, изменив параметр конфигурации `level` в массиве вашего драйвера Slack.

<a name="building-log-stacks"></a>
## Построение стека журналов

Как упоминалось ранее, драйвер `stack` позволяет для удобства объединить несколько каналов в один канал журнала. Чтобы проиллюстрировать, как использовать стеки журналов, давайте рассмотрим пример конфигурации, которую вы можете увидеть в эксплуатационном приложении:

    'channels' => [
        'stack' => [
            'driver' => 'stack',
            'channels' => ['syslog', 'slack'],
        ],

        'syslog' => [
            'driver' => 'syslog',
            'level' => 'debug',
        ],

        'slack' => [
            'driver' => 'slack',
            'url' => env('LOG_SLACK_WEBHOOK_URL'),
            'username' => 'Laravel Log',
            'emoji' => ':boom:',
            'level' => 'critical',
        ],
    ],

Давайте разберем эту конфигурацию. Во-первых, обратите внимание, что наш канал `stack` объединяет два других канала с помощью параметра `channels`: `syslog` и `slack`. Таким образом, при регистрации сообщений оба этих канала будут иметь возможность регистрировать сообщение. Однако, как мы увидим ниже, действительно ли эти каналы регистрируют сообщение, может быть определено серьезностью / «уровнем» сообщения.

<a name="log-levels"></a>
#### Уровни журнала

Обратите внимание на параметр конфигурации `level`, присутствующий в конфигурациях каналов `syslog` и `slack` в приведенном выше примере. Эта опция определяет минимальный «уровень» сообщения, которое должно быть зарегистрировано каналом. Monolog, на котором работают службы ведения журналов Laravel, предлагает все уровни журналов, определенные в спецификации [RFC 5424 specification](https://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info**, и **debug**.

Итак, представьте, что мы регистрируем сообщение, используя метод `debug`:

    Log::debug('An informational message.');

Учитывая нашу конфигурацию, канал `syslog` будет записывать сообщение в системный журнал; однако, поскольку сообщение об ошибке не является уровнем `critical` или выше, то оно не будет отправлено в Slack. Однако, если мы регистрируем сообщение уровня `emergency`, то оно будет отправлено как в системный журнал, так и в Slack, поскольку уровень `emergency` выше нашего минимального порогового значения для обоих каналов:

    Log::emergency('The system is down!');

<a name="writing-log-messages"></a>
## Запись сообщений журнала

Вы можете записывать информацию в журналы с помощью [фасада](facades) `Log`. Как упоминалось ранее, средство ведения журнала обеспечивает восемь уровней ведения журнала, определенных в спецификации [RFC 5424 specification](https://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info**, и **debug**.

    use Illuminate\Support\Facades\Log;

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

Вы можете вызвать любой из этих методов, чтобы записать сообщение для соответствующего уровня. По умолчанию сообщение будет записано в канал журнала по умолчанию, как настроено вашим файлом конфигурации `logging`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;
    use Illuminate\Support\Facades\Log;

    class UserController extends Controller
    {
        /**
         * Показать профиль конкретного пользователя.
         *
         * @param  int  $id
         * @return \Illuminate\Http\Response
         */
        public function show($id)
        {
            Log::info('Showing the user profile for user: '.$id);

            return view('user.profile', [
                'user' => User::findOrFail($id)
            ]);
        }
    }

<a name="contextual-information"></a>
#### Контекстная информация

Также, методам журнала может быть передан массив контекстных данных. Эти контекстные данные будут отформатированы и отображены в сообщении журнала:

    Log::info('User failed to login.', ['id' => $user->id]);

<a name="writing-to-specific-channels"></a>
### Запись в определенные каналы

По желанию можно записать сообщение в канал, отличный от канала по умолчанию вашего приложения. Вы можете использовать метод `channel` фасада `Log` для получения и регистрации любого канала, определенного в вашем файле конфигурации:

    use Illuminate\Support\Facades\Log;

    Log::channel('slack')->info('Something happened!');

Если вы хотите создать стек протоколирования по запросу, состоящий из нескольких каналов, вы можете использовать метод `stack`:

    Log::stack(['single', 'slack'])->info('Something happened!');


<a name="monolog-channel-customization"></a>
## Настройка канала Monolog

<a name="customizing-monolog-for-channels"></a>
### Настройка Monolog для каналов

Иногда требуется полный контроль над настройкой Monolog для существующего канала. Например, бывает необходимо настроить собственную реализацию Monolog `FormatterInterface` для встроенного в Laravel канала `single`.

Для начала определите массив `tap` в конфигурации канала. Массив `tap` должен содержать список классов, которые должны иметь возможность настраивать (или «касаться») экземпляр Monolog после его создания. Не существует обычного места для размещения этих классов, поэтому вы можете создать каталог в своем приложении, чтобы разместить эти классы:

    'single' => [
        'driver' => 'single',
        'tap' => [App\Logging\CustomizeFormatter::class],
        'path' => storage_path('logs/laravel.log'),
        'level' => 'debug',
    ],

После того, как вы настроили опцию `tap` своего канала, вы готовы определить класс, который будет контролировать ваш экземпляр Monolog. Этому классу нужен только один метод: `__invoke`, который получает экземпляр `Illuminate\Log\Logger`. Экземпляр `Illuminate\Log\Logger` передает все вызовы методов базовому экземпляру Monolog:

    <?php

    namespace App\Logging;

    use Monolog\Formatter\LineFormatter;

    class CustomizeFormatter
    {
        /**
         * Настроить переданный экземпляр регистратора.
         *
         * @param  \Illuminate\Log\Logger  $logger
         * @return void
         */
        public function __invoke($logger)
        {
            foreach ($logger->getHandlers() as $handler) {
                $handler->setFormatter(new LineFormatter(
                    '[%datetime%] %channel%.%level_name%: %message% %context% %extra%'
                ));
            }
        }
    }

> {tip} Все ваши классы «касания» извлекаются через [контейнером служб](container), поэтому будут автоматически внедренылюбые объявленные зависимости конструктора.

<a name="creating-monolog-handler-channels"></a>
### Создание обработчика каналов Monolog

В Monolog есть множество [доступных обработчиков](https://github.com/Seldaek/monolog/tree/master/src/Monolog/Handler), а в Laravel из коробки не включены каналы для каждого из них. В некоторых случаях вам может потребоваться создать собственный канал, который является просто экземпляром определенного обработчика Monolog, у которого нет соответствующего драйвера журнала Laravel. Эти каналы могут быть легко созданы с помощью драйвера `monolog`.

При использовании драйвера `monolog` параметр конфигурации `handler` используется для указания того, какой обработчик будет создан. При желании любые параметры конструктора, необходимые обработчику, могут быть указаны с помощью опции конфигурации `with`:

    'logentries' => [
        'driver'  => 'monolog',
        'handler' => Monolog\Handler\SyslogUdpHandler::class,
        'with' => [
            'host' => 'my.logentries.internal.datahubhost.company.com',
            'port' => '10000',
        ],
    ],

<a name="monolog-formatters"></a>
#### Форматеры Monolog

При использовании драйвера `monolog`, Monolog `LineFormatter` будет использоваться как средство форматирования по умолчанию. Однако вы можете настроить тип средства форматирования, передаваемого обработчику, используя параметры конфигурации `formatter` и `formatter_with`:

    'browser' => [
        'driver' => 'monolog',
        'handler' => Monolog\Handler\BrowserConsoleHandler::class,
        'formatter' => Monolog\Formatter\HtmlFormatter::class,
        'formatter_with' => [
            'dateFormat' => 'Y-m-d',
        ],
    ],

Если вы используете обработчик Monolog, который может предоставлять свой собственный модуль форматирования, вы можете установить для параметра конфигурации `formatter` значение `default`:

    'newrelic' => [
        'driver' => 'monolog',
        'handler' => Monolog\Handler\NewRelicHandler::class,
        'formatter' => 'default',
    ],

<a name="creating-custom-channels-via-factories"></a>
### Создание каналов через фабрики

Если вы хотите определить полностью настраиваемый канал, в котором у вас есть полный контроль над созданием и конфигурацией Monolog, вы можете указать тип драйвера `custom` в файле конфигурации `config/logging.php`. Ваша конфигурация должна включать параметр `via`, который содержит имя класса фабрики, которая будет вызываться для создания экземпляра Monolog:

    'channels' => [
        'example-custom-channel' => [
            'driver' => 'custom',
            'via' => App\Logging\CreateCustomLogger::class,
        ],
    ],

После того, как вы настроили канал драйвера `custom`, вы готовы определить класс, который будет создавать ваш экземпляр Monolog. Этому классу нужен только один метод `__invoke`, который должен возвращать экземпляр регистратора Monolog. Метод получит массив конфигурации каналов в качестве единственного аргумента:

    <?php

    namespace App\Logging;

    use Monolog\Logger;

    class CreateCustomLogger
    {
        /**
         * Создать экземпляр собственного регистратора Monolog.
         *
         * @param  array  $config
         * @return \Monolog\Logger
         */
        public function __invoke(array $config)
        {
            return new Logger(...);
        }
    }
