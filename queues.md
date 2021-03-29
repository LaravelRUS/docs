# Laravel 8 · Очереди

- [Введение](#introduction)
    - [Соединения и очереди](#connections-vs-queues)
    - [Предварительная подготовка драйверов](#driver-prerequisites)
- [Создание заданий](#creating-jobs)
    - [Генерация класса задания](#generating-job-classes)
    - [Структура класса](#class-structure)
    - [Уникальные задания](#unique-jobs)
- [Посредник задания](#job-middleware)
    - [Ограничение частоты](#rate-limiting)
    - [Предотвращение дублирования задания](#preventing-job-overlaps)
    - [Ограничение частоты генерации исключений](#throttling-exceptions)
- [Отправка заданий](#dispatching-jobs)
    - [Отложенная отправка](#delayed-dispatching)
    - [Синхронная отправка](#synchronous-dispatching)
    - [Задания и транзакции базы данных](#jobs-and-database-transactions)
    - [Цепочка заданий](#job-chaining)
    - [Настройка соединения и очереди](#customizing-the-queue-and-connection)
    - [Указание максимального количества попыток задания / значений тайм-аута](#max-job-attempts-and-timeout)
    - [Обработка ошибок](#error-handling)
- [Пакетная обработка заданий](#job-batching)
    - [Определение пакета заданий](#defining-batchable-jobs)
    - [Отправка пакета заданий](#dispatching-batches)
    - [Добавление заданий в пакет заданий](#adding-jobs-to-batches)
    - [Инспектирование пакета](#inspecting-batches)
    - [Отмена пакетов](#cancelling-batches)
    - [Отказы в пакете заданий](#batch-failures)
    - [Очистка пакетов](#pruning-batches)
- [Анонимные очереди](#queueing-closures)
- [Запуск обработчика очереди](#running-the-queue-worker)
    - [Команда `queue:work`](#the-queue-work-command)
    - [Приоритеты очереди](#queue-priorities)
    - [Обработчики очереди и развертывание](#queue-workers-and-deployment)
    - [Истечение срока и тайм-ауты задания](#job-expirations-and-timeouts)
- [Конфигурация Supervisor](#supervisor-configuration)
- [Разбор неудачных заданий](#dealing-with-failed-jobs)
    - [Очистка после неудачных заданий](#cleaning-up-after-failed-jobs)
    - [Повторная попытка выполнения неудачных заданий](#retrying-failed-jobs)
    - [Игнорирование отсутствующих моделей](#ignoring-missing-models)
    - [События неудачных заданий](#failed-job-events)
- [Удаление заданий из очередей](#clearing-jobs-from-queues)
- [События задания](#job-events)

<a name="introduction"></a>
## Введение

При создании веб-приложения у вас могут быть некоторые задачи, такие как синтаксический анализ и сохранение загруженного файла CSV, выполнение которых во время обычного веб-запроса занимает слишком много времени. К счастью, Laravel позволяет легко создавать задания в очереди, которые могут обрабатываться в фоновом режиме. Перемещая трудоемкие задания в очередь, ваше приложение может отвечать на веб-запросы с повышенной скоростью, улучшая уровень пользования приложением вашими клиентами.

Очереди Laravel предоставляют унифицированный API для различных серверных служб очередей, таких как [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io) или даже реляционная база данных.

Параметры конфигурации очереди Laravel хранятся в файле конфигурации вашего приложения `config/queue.php`. В этом файле вы найдете конфигурации подключения для каждого из драйверов очереди фреймворка: база данных, [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io) и [Beanstalkd](https://beanstalkd.github.io/), а также синхронный драйвер для немедленного выполнения задания (используется во время локальной разработки). Также имеется драйвер очереди `null`, который выбрасывает задания из очереди.

> {tip} Laravel также предлагает Horizon, красивую панель управления и систему конфигурации для ваших очередей с поддержкой Redis. Дополнительную информацию можно найти в полной [документации Horizon](horizon).

<a name="connections-vs-queues"></a>
### Соединения и очереди

Прежде чем приступить к работе с очередями Laravel, важно понять различие между «соединениями» и «очередями». В конфигурационном файле `config/queue.php` есть массив `connections`. Этот параметр определяет подключения к серверным службам очередей, таким как Amazon SQS, Beanstalk или Redis. Однако любое указанное «соединение» очереди может иметь несколько «очередей», которые можно рассматривать как разные стеки или пачки поочередных заданий.

Обратите внимание, что каждый пример конфигурации соединения в файле конфигурации `queue` содержит ключ `queue`. Это очередь по умолчанию, в которую будут отправляться задания при их отправке в определенное соединение. Другими словами, если вы отправляете задание без явного определения очереди, в которую оно должно быть отправлено, задание будет поставлено в очередь, которая определена в ключе `queue` конфигурации соединения:

    use App\Jobs\ProcessPodcast;

    // Это задание отправляется в очередь `default` соединения по умолчанию ...
    ProcessPodcast::dispatch();

    // Это задание отправляется в очередь `emails` соединения по умолчанию ...
    ProcessPodcast::dispatch()->onQueue('emails');

Некоторым приложениям может не понадобиться помещать задания в несколько очередей, вместо этого предпочитая иметь одну простую очередь. Однако отправка заданий в несколько очередей может быть особенно полезна для приложений, определяющих приоритеты или сегментацию процесса обработки заданий, поскольку обработчик очереди Laravel позволяет вам указать, какие очереди он должен обрабатывать по приоритету. Например, если вы помещаете задания в очередь `high`, то вы можете запустить обработчик, который даст им более высокий приоритет обработки:

    php artisan queue:work --queue=high,default

<a name="driver-prerequisites"></a>
### Предварительная подготовка драйверов

<a name="database"></a>
#### База данных

Чтобы использовать драйвер очереди `database`, вам понадобится таблица базы данных для хранения заданий. Чтобы сгенерировать миграцию, которая создает эту таблицу, запустите команду `queue:table` Artisan. После того, как миграция будет создана, вы можете выполнить ее миграцию с помощью команды `migrate`:

    php artisan queue:table

    php artisan migrate

<a name="redis"></a>
#### Redis

Чтобы использовать драйвер очереди `redis`, вы должны настроить соединение с базой данных Redis в файле конфигурации `config/database.php`.

**Кластер Redis**

Если ваше соединение с очередью Redis использует кластер Redis, то имена ваших очередей должны содержать [ключевой хеш-тег](https://redis.io/topics/cluster-spec#keys-hash-tags). Это необходимо для того, чтобы все ключи Redis для указанной очереди были поставлены в один и тот же хеш-слот:

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => '{default}',
        'retry_after' => 90,
    ],

**Блокировка**

При использовании очереди Redis вы можете использовать параметр конфигурации `block_for`, чтобы указать, как долго драйвер должен ждать, пока задание станет доступным, прежде чем выполнить итерацию через рабочий цикл и повторно опросить базу данных Redis.

Настройка этого значения зависит от загрузки очереди и может быть более эффективной, чем постоянный опрос базы данных Redis на предмет новых заданий. Например, вы можете установить значение `5`, чтобы указать, что драйвер должен блокироваться на пять секунд, ожидая, пока задание станет доступным:

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => 'default',
        'retry_after' => 90,
        'block_for' => 5,
    ],

> {note} Установка для `block_for` значения `0` заставит обработчиков очереди блокироваться на неопределенный срок, пока задание не станет доступным. Это также предотвратит обработку таких сигналов, как `SIGTERM`, до тех пор, пока не будет обработано следующее задание.

<a name="other-driver-prerequisites"></a>
#### Дополнительные зависимости драйверов

Для перечисленных драйверов очереди необходимы следующие зависимости. Эти зависимости могут быть установлены через менеджер пакетов Composer:

<!-- <div class="content-list" markdown="1"> -->
- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~4.0`
- Redis: `predis/predis ~1.0` or phpredis PHP extension
<!-- </div> -->

<a name="creating-jobs"></a>
## Создание заданий

<a name="generating-job-classes"></a>
### Генерация класса задания

Чтобы сгенерировать новое задание, используйте команду `make:job` [Artisan](artisan). Эта команда поместит новый класс задания в каталог `app/Jobs` вашего приложения. Если этот каталог не существует в вашем приложении, то Laravel предварительно создаст его:

    php artisan make:job ProcessPodcast

Сгенерированный класс будет реализовывать интерфейс `Illuminate\Contracts\Queue\ShouldQueue`, указывая Laravel, что задание должно быть поставлено в очередь для асинхронного выполнения.

> {tip} Заготовки заданий можно настроить с помощью [публикации заготовок](artisan.md#stub-customization).

<a name="class-structure"></a>
### Структура класса задания

Классы заданий очень простые, обычно они содержат только метод `handle`, который вызывается, когда задание обрабатывается очередью. Для начала рассмотрим пример класса задания. В этом примере мы представим, что управляем службой публикации подкастов и нам необходимо обработать загруженные файлы подкастов перед их публикацией:

    <?php

    namespace App\Jobs;

    use App\Models\Podcast;
    use App\Services\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Queue\SerializesModels;

    class ProcessPodcast implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * Экземпляр подкаста.
         *
         * @var \App\Models\Podcast
         */
        protected $podcast;

        /**
         * Создать новый экземпляр задания.
         *
         * @param  App\Models\Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * Выполнить задание.
         *
         * @param  App\Services\AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // Обработка загруженного подкаста ...
        }
    }

Обратите внимание, что в этом примере мы смогли передать [модель Eloquent](eloquent) непосредственно в конструктор задания. Благодаря трейту `SerializesModels`, который использует задание, модели Eloquent и их загруженные отношения будут корректно сериализованы и десериализованы при обработке задания.

Если ваше задание в очереди принимает модель Eloquent в своем конструкторе, в очередь будет сериализован только идентификатор модели. Когда задание действительно обрабатывается, система очередей автоматически повторно извлекает полный экземпляр модели и его загруженные отношения из базы данных. Такой подход к сериализации модели позволяет отправлять в драйвер очереди гораздо меньший объем данных.

<a name="handle-method-dependency-injection"></a>
#### Внедрение зависимости метода `handle`

Метод `handle` вызывается, когда задание обрабатывается очередью. Обратите внимание, что мы можем объявить тип зависимости в методе `handle` задания. [Контейнер служб](container) Laravel автоматически внедряет эти зависимости.

Если вы хотите получить полный контроль над тем, как контейнер внедряет зависимости в метод `handle`, вы можете использовать метод `bindMethod` контейнера. Метод `bindMethod` принимает замыкание, которое получает задание и контейнер. В замыкании вы можете вызывать метод `handle`, как хотите. Обычно вы должны вызывать этот метод из метода `boot` вашего [поставщик служб](providers) `App\Providers\AppServiceProvider`:

    use App\Jobs\ProcessPodcast;
    use App\Services\AudioProcessor;

    $this->app->bindMethod([ProcessPodcast::class, 'handle'], function ($job, $app) {
        return $job->handle($app->make(AudioProcessor::class));
    });

> {note} Двоичные данные, например, необработанное содержимое изображения, должны быть переданы через функцию `base64_encode` перед передачей заданию. В противном случае задание может неправильно сериализоваться в JSON при отправки в очередь.

<a name="handling-relationships"></a>
#### Обработка Отношений

Поскольку загруженные отношения также сериализуются, сериализованная строка задания иногда может стать довольно большой. Чтобы предотвратить сериализацию отношений, вы можете вызвать метод `withoutRelations` модели при указании значения свойства. Этот метод вернет экземпляр модели без загруженных отношений:

    /**
     * Создать новый экземпляр задания.
     *
     * @param  \App\Models\Podcast  $podcast
     * @return void
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast->withoutRelations();
    }

<a name="unique-jobs"></a>
### Уникальные задания

> {note} Для уникальных заданий требуется драйвер кеша, поддерживающий [блокировки](cache.md#atomic-locks). В настоящее время драйверы кеширования `memcached`, `redis`, `dynamodb`, `database`, `file`, and `array` поддерживают атомарные блокировки. Кроме того, уникальность заданий не учитывается при пакетной обработке.

Иногда требуется убедиться, что только один экземпляр определенного задания находится в очереди в любой момент времени. Вы можете сделать это, реализовав интерфейс `ShouldBeUnique` в своем классе задания. Этот интерфейс не требует от вас определения каких-либо дополнительных методов в вашем классе:

    <?php

    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Contracts\Queue\ShouldBeUnique;

    class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
    {
        ...
    }

В приведенном выше примере задание `UpdateSearchIndex` уникально. Таким образом, задание не будет отправлено, если другой экземпляр задания уже находится в очереди и еще не завершил обработку.

В некоторых случаях вам может потребоваться определить конкретный «ключ», который делает задание уникальным, или вы можете указать тайм-аут, по истечении которого задание больше не считается уникальным. Для этого вы можете определить свойства или методы `uniqueId` и `uniqueFor` в своем классе задания:

    <?php

    use App\Product;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Contracts\Queue\ShouldBeUnique;

    class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
    {
        /**
         * Экземпляр продукта.
         *
         * @var \App\Product
         */
        public $product;

        /**
         * Количество секунд, по истечении которых уникальная блокировка задания будет снята.
         *
         * @var int
         */
        public $uniqueFor = 3600;

        /**
         * Уникальный идентификатор задания.
         *
         * @return string
         */
        public function uniqueId()
        {
            return $this->product->id;
        }
    }

В приведенном выше примере задание `UpdateSearchIndex` уникально по идентификатору продукта. Таким образом, любые новые отправленные задания с тем же идентификатором продукта будут игнорироваться, пока существующее задание не завершит обработку. Кроме того, если существующее задание не будет обработано в течение одного часа, уникальная блокировка будет снята, и в очередь может быть отправлено другое задание с таким же уникальным ключом.

<a name="keeping-jobs-unique-until-processing-begins"></a>
#### Сохранение уникальности задания только до начала обработки

По умолчанию уникальные задания «разблокируются» после того, как задание завершит обработку или потерпит неудачу во всех повторных попытках. Однако, могут возникнуть ситуации, когда вы захотите, чтобы ваше задание было разблокировано непосредственно перед его обработкой. Для этого ваше задание должно реализовать контракт `ShouldBeUniqueUntilProcessing` вместо контракта `ShouldBeUnique`:

    <?php

    use App\Product;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;

    class UpdateSearchIndex implements ShouldQueue, ShouldBeUniqueUntilProcessing
    {
        // ...
    }

<a name="unique-job-locks"></a>
#### Блокировки уникальных заданий

За кулисами, когда отправляется задание `ShouldBeUnique`, Laravel пытается получить [блокировку](cache.md#atomic-locks) с ключом `uniqueId`. Если блокировка не получена, задание не отправляется. Эта блокировка снимается, когда задание завершает обработку или терпит неудачу во всех повторных попытках. По умолчанию Laravel будет использовать драйвер кеша, назначенный по умолчанию, для получения этой блокировки. Однако, если вы хотите использовать другой драйвер для получения блокировки, вы можете определить метод `uniqueVia`, который возвращает драйвер кеша, который следует использовать:

    use Illuminate\Support\Facades\Cache;

    class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
    {
        ...

        /**
         * Получить драйвер кеша для блокировки уникального задания.
         *
         * @return \Illuminate\Contracts\Cache\Repository
         */
        public function uniqueVia()
        {
            return Cache::driver('redis');
        }
    }

> {tip} Если вам нужно ограничить только параллельную обработку задания, используйте вместо этого посредник [`WithoutOverlapping`](#preventing-job-overlaps).

<a name="job-middleware"></a>
## Посредник задания

Посредник задания позволяет обернуть пользовательскую логику вокруг выполнения заданий в очереди, уменьшая шаблонность самих заданий. Например, рассмотрим следующий метод `handle`, который использует функции ограничения частоты, позволяющие обрабатывать только одно задание каждые пять секунд:

    use Illuminate\Support\Facades\Redis;

    /**
     * Выполнить задание.
     *
     * @return void
     */
    public function handle()
    {
        Redis::throttle('key')->block(0)->allow(1)->every(5)->then(function () {
            info('Lock obtained...');

            // Обработка задания ...
        }, function () {
            // Не удалось получить блокировку ...

            return $this->release(5);
        });
    }

Хотя этот код действителен, реализация метода `handle` становится «шумной», так как она загромождена логикой ограничения частоты Redis. Кроме того, эта логика ограничения частоты должна быть продублирована для любых других заданий, для которых мы хотим установить ограничение частоты.

Вместо ограничения частоты в методе `handle` мы могли бы определить посредника задания, который обрабатывает ограничение частоты. В Laravel нет места по умолчанию для посредников заданий, поэтому вы можете разместить их в любом месте вашего приложения. В этом примере мы поместим его в каталог `app/Jobs/Middleware`:

    <?php

    namespace App\Jobs\Middleware;

    use Illuminate\Support\Facades\Redis;

    class RateLimited
    {
        /**
         * Обработать задание в очереди.
         *
         * @param  mixed  $job
         * @param  callable  $next
         * @return mixed
         */
        public function handle($job, $next)
        {
            Redis::throttle('key')
                    ->block(0)->allow(1)->every(5)
                    ->then(function () use ($job, $next) {
                        // Блокировка получена ...

                        $next($job);
                    }, function () use ($job) {
                        // Не удалось получить блокировку ...

                        $job->release(5);
                    });
        }
    }

Как вы можете видеть, как и [посредник маршрута](middleware), посредник задания получает обрабатываемое задание и замыкание, которое должно быть вызвано для продолжения обработки задания.

После создания посредника задания он может быть назначен заданию, вернув их из метода `middleware` задания. Этот метод не существует для заданий, созданных с помощью команды `make:job` Artisan, поэтому вам нужно будет вручную добавить его в свой класс задания:

    use App\Jobs\Middleware\RateLimited;

    /**
     * Получить посредника, через которого должно пройти задание.
     *
     * @return array
     */
    public function middleware()
    {
        return [new RateLimited];
    }

<a name="rate-limiting"></a>
### Ограничение частоты

Хотя мы только что продемонстрировали, как написать собственного посредника, ограничивающего частоту, Laravel на самом деле включает посредника, который вы можете использовать для задания ограничения частоты. Как и [ограничители частоты маршрута](routing.md#defining-rate-limiters), ограничители частоты задания определяются с помощью метода `for` фасада `RateLimiter`.

Например, вы можете разрешить пользователям выполнять резервное копирование своих данных один раз в час, при этом не накладывая таких ограничений на премиум-клиентов. Для этого вы можете определить `RateLimiter` в методе `boot` вашего `AppServiceProvider`:

    use Illuminate\Cache\RateLimiting\Limit;
    use Illuminate\Support\Facades\RateLimiter;

    /**
     * Загрузка любых служб приложения.
     *
     * @return void
     */
    public function boot()
    {
        RateLimiter::for('backups', function ($job) {
            return $job->user->vipCustomer()
                        ? Limit::none()
                        : Limit::perHour(1)->by($job->user->id);
        });
    }

В приведенном выше примере мы определили часовой лимит частоты; однако вы можете легко определить ограничение на основе минут, используя метод `perMinute`. Кроме того, вы можете передать любое значение методу `by` ограничения; однако это значение чаще всего используется для сегментации ограничений частоты с по клиентам:

    return Limit::perMinute(50)->by($job->user->id);

После того, как вы определили ограничение частоты, вы можете назначить ограничитель частоты своему заданию резервного копирования с помощью посредника `Illuminate\Queue\Middleware\RateLimited`. Каждый раз, когда задание превышает ограничение частоты, этот посредник отправляет задание обратно в очередь с соответствующей задержкой в зависимости от продолжительности ограничения частоты.

    use Illuminate\Queue\Middleware\RateLimited;

    /**
     * Получить посредника, через которого должно пройти задание.
     *
     * @return array
     */
    public function middleware()
    {
        return [new RateLimited('backups')];
    }

Возвращение задания с ограниченной частотой обратно в очередь все равно увеличит общее количество «попыток» (`attempts`) задания. Возможно, вы захотите соответствующим образом настроить свойства `tries` и `maxExceptions` в своем классе задания. Или вы можете использовать метод [`retryUntil`](#time-based-attempts), чтобы определить время, по истечению которого попыток выполнения задания больше не будет.

> {tip} Если вы используете Redis, то вы можете использовать посредника `Illuminate\Queue\Middleware\RateLimitedWithRedis`, который лучше настроен для Redis и более эффективен, чем базовый посредник с ограничением частоты.

<a name="preventing-job-overlaps"></a>
### Предотвращение дублирования задания

Laravel включает посредника `Illuminate\Queue\Middleware\WithoutOverlapping`, который позволяет предотвращать перекрытия заданий на основе произвольного ключа. Это может быть полезно, когда задание в очереди изменяет ресурс, который должен изменяться только одним заданием за раз.

Например, представим, что у вас есть задание в очереди, которое обновляет кредитный рейтинг пользователя, и вы хотите предотвратить дублирование задания обновления кредитного рейтинга для одного и того же идентификатора пользователя. Для этого вы можете вернуть посредника `WithoutOverlapping` из метода `middleware` вашего задания:

    use Illuminate\Queue\Middleware\WithoutOverlapping;

    /**
     * Получить посредника, через которого должно пройти задание.
     *
     * @return array
     */
    public function middleware()
    {
        return [new WithoutOverlapping($this->user->id)];
    }

Любые перекрывающиеся задания будут возвращены в очередь. Можно также указать время в секундах, которое должно пройти до повторной попытки возвращенного задания:

    /**
     * Получить посредника, через которого должно пройти задание.
     *
     * @return array
     */
    public function middleware()
    {
        return [(new WithoutOverlapping($this->order->id))->releaseAfter(60)];
    }

Если вы хотите немедленно удалить все перекрывающиеся задания, чтобы они не повторялись, вы можете использовать метод `dоntRelease`:

    /**
     * Получить посредника, через которого должно пройти задание.
     *
     * @return array
     */
    public function middleware()
    {
        return [(new WithoutOverlapping($this->order->id))->dontRelease()];
    }

> {note} Для посредника `WithoutOverlapping` требуется драйвер кеша, который поддерживает [блокировки](cache.md#atomic-locks). В настоящее время драйверы кеша `memcached`, `redis`, `dynamodb`, `database`, `file`, и `array` поддерживают атомарные блокировки.

<a name="throttling-exceptions"></a>
### Ограничение частоты генерации исключений

Laravel содержит посредника `Illuminate\Queue\Middleware\ThrottlesExceptions`, который позволяет вам регулировать вызываемые исключения. Как только задание вызывает переданное количество исключений, все дальнейшие попытки выполнить задание откладываются до истечения заданного интервала времени. Этот посредник особенно полезен для заданий, которые взаимодействуют с нестабильно работающими сторонними службами.

Например, представим себе задание в очереди, которое взаимодействует со сторонним API, который начинает выбрасывать исключения. Чтобы ограничить исключения, вы можете вернуть посредника `ThrottlesExceptions` из метода `middleware` вашего задания. Как правило, этот посредник должен быть связан с заданием, которое реализует [попытки, основанные на времени](#time-based-attempts):

    use Illuminate\Queue\Middleware\ThrottlesExceptions;

    /**
     * Получить посредника, через которого должно пройти задание.
     *
     * @return array
     */
    public function middleware()
    {
        return [new ThrottlesExceptions(10, 5)];
    }

    /**
     * Задать временной предел попыток выполнить задания.
     *
     * @return \DateTime
     */
    public function retryUntil()
    {
        return now()->addMinutes(30);
    }

Первый аргумент конструктора посредника – это количество исключений, которые задание может выбросить перед ограничением, а второй аргумент конструктора – это количество минут, которые должны пройти, прежде чем будет предпринято повторное выполнение задания после его ограничения. В приведенном выше примере кода, если задание выбросит 10 исключений в течение 5 минут, мы подождем 5 минут перед его повторной попыткой выполнения

Когда задание вызывает исключение, но порог исключения еще не достигнут, то задание обычно немедленно повторяется. Однако вы можете указать количество секунд, на которые такое задание должно быть отложено, вызвав метод `backoff` при определении метода `middleware`:

    use Illuminate\Queue\Middleware\ThrottlesExceptions;

    /**
     * Получить посредника, через которого должно пройти задание.
     *
     * @return array
     */
    public function middleware()
    {
        return [(new ThrottlesExceptions(10, 5))->backoff(5)];
    }

Внутренне этот посредник использует систему кеширования Laravel для реализации ограничений частоты, а имя класса задания используется в качестве «ключа» кеша. Вы можете переопределить этот ключ, вызвав метод `by` при определении метода `middleware` вашего задания. Это может быть полезно, если у вас есть несколько заданий, взаимодействующих с одной и той же сторонней службой, и вы хотите, чтобы у них была общая «корзина» ограничений:

    use Illuminate\Queue\Middleware\ThrottlesExceptions;

    /**
     * Получить посредника, через которого должно пройти задание.
     *
     * @return array
     */
    public function middleware()
    {
        return [(new ThrottlesExceptions(10, 10))->by('key')];
    }

> {tip} Если вы используете Redis в качестве драйвера кеша вашего приложения, то вы можете использовать класс `Illuminate\Queue\Middleware\ThrottlesExceptionsWithRedis`. Этот класс более эффективен при управлении ограничениями исключений с помощью Redis.

<a name="dispatching-jobs"></a>
## Отправка заданий

После того, как вы написали свой класс задания, вы можете отправить его, используя метод `dispatch` самого задания. Аргументы, переданные методу `dispatch`, будут переданы конструктору задания:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Jobs\ProcessPodcast;
    use App\Models\Podcast;
    use Illuminate\Http\Request;

    class PodcastController extends Controller
    {
        /**
         * Сохранить новый подкаст.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function store(Request $request)
        {
            $podcast = Podcast::create(...);

            // ...

            ProcessPodcast::dispatch($podcast);
        }
    }

Если требуется условно отправить задание, то можно использовать методы `dispatchIf` и `dispatchUnless`:

    ProcessPodcast::dispatchIf($accountActive, $podcast);

    ProcessPodcast::dispatchUnless($accountSuspended, $podcast);

<a name="delayed-dispatching"></a>
### Отложенная отправка

Если вы хотите указать, что задание не должно быть немедленно доступно для обработчика очереди, вы можете использовать метод `delay` при отправке задания. Например, давайте укажем, что задание не должно быть доступно для обработки в течение 10 минут после его отправки:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Jobs\ProcessPodcast;
    use App\Models\Podcast;
    use Illuminate\Http\Request;

    class PodcastController extends Controller
    {
        /**
         * Сохранить новый подкаст.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function store(Request $request)
        {
            $podcast = Podcast::create(...);

            // ...

            ProcessPodcast::dispatch($podcast)
                        ->delay(now()->addMinutes(10));
        }
    }

> {note} У сервиса очередей Amazon SQS максимальное время задержки составляет 15 минут.

<a name="dispatching-after-the-response-is-sent-to-browser"></a>
#### Отправка задания после отправки ответа в браузер

В качестве альтернативы, метод `dispatchAfterResponse` задерживает отправку задания до тех пор, пока HTTP-ответ не будет отправлен в браузер пользователя. Это по-прежнему позволит пользователю начать использовать приложение, даже если задание в очереди все еще выполняется. Обычно это следует использовать только для заданий, которые занимают около секунды, например, для отправки электронного письма. Поскольку они обрабатываются в рамках текущего HTTP-запроса, отправляемые таким образом задания не требуют запуска обработчика очереди для их обработки:

    use App\Jobs\SendNotification;

    SendNotification::dispatchAfterResponse();

Вы также можете отправить замыкание и связать метод `afterResponse` с помощником `dispatch`, чтобы выполнить замыкание после того, как HTTP-ответ был отправлен в браузер:

    use App\Mail\WelcomeMessage;
    use Illuminate\Support\Facades\Mail;

    dispatch(function () {
        Mail::to('taylor@example.com')->send(new WelcomeMessage);
    })->afterResponse();

<a name="synchronous-dispatching"></a>
### Синхронная отправка

Если вы хотите отправить задание немедленно (синхронно), то вы можете использовать метод `dispatchSync`. При использовании этого метода задание не будет поставлено в очередь и будет выполнено немедленно в рамках текущего процессе:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Jobs\ProcessPodcast;
    use App\Models\Podcast;
    use Illuminate\Http\Request;

    class PodcastController extends Controller
    {
        /**
         * Сохранить новый подкаст.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function store(Request $request)
        {
            $podcast = Podcast::create(...);

            // Создание подкаста ...

            ProcessPodcast::dispatchSync($podcast);
        }
    }

<a name="jobs-and-database-transactions"></a>
### Задания и транзакции базы данных

Несмотря на то, что отправка заданий в рамках транзакций базы данных – это нормально, вам следует позаботиться о том, чтобы ваше задание действительно могло быть успешно выполнено. При отправке задания в рамках транзакции возможно, что задание будет обработано до фиксации транзакции. Когда это происходит, любые обновления, внесенные вами в модели или записи базы данных во время транзакции базы данных, могут еще не быть отражены в базе данных. Кроме того, любые модели или записи базы данных, созданные в рамках транзакции, могут даже не существовать в базе данных.

К счастью, Laravel содержит несколько методов решения этой проблемы. Во-первых, вы можете задать параметр соединения `after_commit` в массиве конфигурации соединения к очереди:

    'redis' => [
        'driver' => 'redis',
        // ...
        'after_commit' => true,
    ],

Когда параметр `after_commit` имеет значение `true`, вы можете отправлять задания в транзакциях базы данных; однако, Laravel будет ждать, пока все открытые транзакции базы данных не будут зафиксированы, прежде чем фактически отправить задание. Конечно, если в настоящее время транзакции базы данных не открыты, задание будет отправлено немедленно.

При откате транзакции из-за исключения, возникшего во время транзакции, отправленные во время этой транзакции задания будут отброшены.

> {tip} Установка параметру конфигурации `after_commit` значения `true` также вызовет отправку всех поставленных в очередь слушателей событий, почтовых отправлений, уведомлений и широковещательных событий после того, как все открытые транзакции базы данных были зафиксированы.

<a name="specifying-commit-dispatch-behavior-inline"></a>
#### Непосредственное указание поведения отправки при фиксации транзакций БД

Если вы не установите для параметра конфигурации соединения очереди `after_commit` значение `true`, то вы все равно можете указать, что конкретное задание должно быть отправлено после того, как все открытые транзакции базы данных были зафиксированы. Для этого вы можете связать метод `afterCommit` с операцией отправки:

    use App\Jobs\ProcessPodcast;

    ProcessPodcast::dispatch($podcast)->afterCommit();

Аналогично, если для параметра конфигурации `after_commit` установлено значение `true`, вы можете указать, что конкретное задание должно быть отправлено немедленно, не дожидаясь фиксации каких-либо открытых транзакций базы данных:

    ProcessPodcast::dispatch($podcast)->beforeCommit();

<a name="job-chaining"></a>
### Цепочка заданий

Цепочка заданий позволяет указать список заданий в очереди, которые должны выполняться последовательно после успешного выполнения основного задания. Если одно задание в последовательности завершается неуспешно, то остальные задания не выполняются. Чтобы выполнить цепочку заданий в очереди, вы можете использовать метод `chain`, фасада `Bus`. Командная шина Laravel – это компонент нижнего уровня, на котором построена диспетчеризация заданий в очереди:

    use App\Jobs\OptimizePodcast;
    use App\Jobs\ProcessPodcast;
    use App\Jobs\ReleasePodcast;
    use Illuminate\Support\Facades\Bus;

    Bus::chain([
        new ProcessPodcast,
        new OptimizePodcast,
        new ReleasePodcast,
    ])->dispatch();

В дополнение к цепочке экземпляров класса задания вы также можете передавать замыкания:

    Bus::chain([
        new ProcessPodcast,
        new OptimizePodcast,
        function () {
            Podcast::update(...);
        },
    ])->dispatch();

> {note} Удаление заданий с помощью метода `$this->delete()` внутри задания не остановить обработку связанных заданий. Цепочка прекратит выполнение только в случае сбоя задания в цепочке.

<a name="chain-connection-queue"></a>
#### Соединения и очередь цепочки заданий

Если вы хотите указать соединение и очередь, которые должны использоваться для связанных заданий, вы можете использовать методы `onConnection` и `onQueue`. Эти методы указывают соединение и имя очереди, которые следует использовать, если заданию явно не назначено другое соединение / очередь:

    Bus::chain([
        new ProcessPodcast,
        new OptimizePodcast,
        new ReleasePodcast,
    ])->onConnection('redis')->onQueue('podcasts')->dispatch();

<a name="chain-failures"></a>
#### Отказы в цепочке заданий

При объединении заданий в цепочку вы можете использовать метод `catch`, чтобы указать замыкание, которое должно вызываться, если задание в цепочке завершается неуспешно. Данный замыкание получит экземпляр `Throwable`, спровоцировавшего провал задания:

    use Illuminate\Support\Facades\Bus;
    use Throwable;

    Bus::chain([
        new ProcessPodcast,
        new OptimizePodcast,
        new ReleasePodcast,
    ])->catch(function (Throwable $e) {
        // Задание в цепочке не выполнено ...
    })->dispatch();

<a name="customizing-the-queue-and-connection"></a>
### Настройка соединения и очереди

<a name="dispatching-to-a-particular-queue"></a>
#### Отправка в определенную очередь

Помещая задания в разные очереди, вы можете «классифицировать» свои задания в оереди и даже определять приоритеты, сколько обработчиков вы назначаете в разные очереди. Имейте в виду, что при этом задания не отправляются в разные «соединения» очередей, как определено в файле конфигурации очереди, а только в определенные очереди в рамках одного соединения. Чтобы указать очередь, используйте метод `onQueue` при отправке задания:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Jobs\ProcessPodcast;
    use App\Models\Podcast;
    use Illuminate\Http\Request;

    class PodcastController extends Controller
    {
        /**
         * Сохранить новый подкаст.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function store(Request $request)
        {
            $podcast = Podcast::create(...);

            // Создание подкаста ...

            ProcessPodcast::dispatch($podcast)->onQueue('processing');
        }
    }

Кроме того, вы можете указать очередь задания, вызвав метод `onQueue` в конструкторе задания:

    <?php

    namespace App\Jobs;

     use Illuminate\Bus\Queueable;
     use Illuminate\Contracts\Queue\ShouldQueue;
     use Illuminate\Foundation\Bus\Dispatchable;
     use Illuminate\Queue\InteractsWithQueue;
     use Illuminate\Queue\SerializesModels;

    class ProcessPodcast implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * Создать новый экземпляр задания.
         *
         * @return void
         */
        public function __construct()
        {
            $this->onQueue('processing');
        }
    }

<a name="dispatching-to-a-particular-connection"></a>
#### Отправка в конкретное соединение

Если ваше приложение взаимодействует с несколькими соединениями очередей, то вы можете указать, на какое соединение отправить задание, используя метод `onConnection`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Jobs\ProcessPodcast;
    use App\Models\Podcast;
    use Illuminate\Http\Request;

    class PodcastController extends Controller
    {
        /**
         * Сохранить новый подкаст.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function store(Request $request)
        {
            $podcast = Podcast::create(...);

            // Создание подкаста ...

            ProcessPodcast::dispatch($podcast)->onConnection('sqs');
        }
    }

Вы можете связать методы `onConnection` и `onQueue` вместе, чтобы указать соединение и очередь для задания:

    ProcessPodcast::dispatch($podcast)
                  ->onConnection('sqs')
                  ->onQueue('processing');

Кроме того, вы можете указать соединение задания, вызвав метод `onConnection` в конструкторе задания:

    <?php

    namespace App\Jobs;

     use Illuminate\Bus\Queueable;
     use Illuminate\Contracts\Queue\ShouldQueue;
     use Illuminate\Foundation\Bus\Dispatchable;
     use Illuminate\Queue\InteractsWithQueue;
     use Illuminate\Queue\SerializesModels;

    class ProcessPodcast implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * Создать новый экземпляр задания.
         *
         * @return void
         */
        public function __construct()
        {
            $this->onConnection('sqs');
        }
    }

<a name="max-job-attempts-and-timeout"></a>
### Указание максимального количества попыток задания / значений тайм-аута

<a name="max-attempts"></a>
#### Максимальное количество попыток

Если в одном из ваших заданий в очереди обнаруживается ошибка, то вы, вероятно, не хотите, чтобы оно продолжало повторять попытки бесконечно. Laravel предлагает различные способы указать, сколько раз и как долго задание может быть повторно выполняться.

Один из подходов к указанию максимального количества попыток выполнения задания – это использование переключателя `--tries` в командной строке Artisan. Это будет применяться ко всем заданиям обработчика, если только обрабатываемое задание не указывает более конкретное количество попыток его выполнения:

    php artisan queue:work --tries=3

Если задание превышает максимальное количество попыток, то оно будет считаться «неудачным». Для получения дополнительной информации об обработке невыполненных заданий обратитесь к [документации по разбору неудачных заданий](#dealing-with-failed-jobs).

Вы можете применить более детальный подход, указав максимальное количество попыток выполнения задания для самого класса задания. Если для задания указано максимальное количество попыток, оно будет иметь приоритет над значением `--tries`, указанным в командной строке:

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * Количество попыток выполнения задания.
         *
         * @var int
         */
        public $tries = 5;
    }

<a name="time-based-attempts"></a>
#### Попытки, основанные на времени

В качестве альтернативы определению количества попыток выполнения задания до того, как оно завершится ошибкой, вы можете определить время, когда прекратить попытки выполнения задания. Это позволяет выполнять задание любое количество раз в течение заданного периода времени. Чтобы определить время, через которое больше не следует пытаться выполнить задание, добавьте метод `retryUntil` в свой класс задания. Этот метод должен возвращать экземпляр `DateTime`:

    /**
     * Задать временной предел попыток выполнить задания.
     *
     * @return \DateTime
     */
    public function retryUntil()
    {
        return now()->addMinutes(10);
    }

> {tip} Вы также можете определить свойство `$tries` или метод `retryUntil` в ваших [слушателях событий](events.md#queued-event-listeners).

<a name="max-exceptions"></a>
#### Максимальное количество исключений

Иногда вы можете указать, что задание может быть выполнено много раз, но должно завершиться ошибкой, если повторные попытки инициированы заданным количеством необработанных исключений (в отличие от отправки напрямую методом `release`). Для этого вы можете определить свойство `maxExceptions` в своем классе задания:

    <?php

    namespace App\Jobs;

    use Illuminate\Support\Facades\Redis;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * Количество попыток выполнения задания.
         *
         * @var int
         */
        public $tries = 25;

        /**
         * Максимальное количество разрешенных необработанных исключений.
         *
         * @var int
         */
        public $maxExceptions = 3;

        /**
         * Выполнить задание.
         *
         * @return void
         */
        public function handle()
        {
            Redis::throttle('key')->allow(10)->every(60)->then(function () {
                // Блокировка получена, обрабатываем подкаст ...
            }, function () {
                // Невозможно получить блокировку ...
                return $this->release(10);
            });
        }
    }

В этом примере задание высвобождается на десять секунд, если приложение не может получить блокировку Redis, и будет продолжать повторяться до 25 раз. Однако задание завершится ошибкой, если оно вызовет три необработанных исключения.

<a name="timeout"></a>
#### Тайм-аут

> {note} Расширение PHP `pcntl` должно быть установлено для указания тайм-аутов задания.

Часто вы приблизительно знаете, сколько времени займет выполнение заданий в очереди. По этой причине Laravel позволяет вам указать значение «тайм-аута». Если задание обрабатывается дольше, чем количество секунд, указанное значением тайм-аута, обработчик завершится с ошибкой. Обычно обработчик перезапускается автоматически [диспетчером, настроенным на вашем сервере](#supervisor-configuration).

Максимальное количество секунд, в течение которых могут выполняться задания, можно указать с помощью переключателя `--timeout` в командной строке Artisan:

    php artisan queue:work --timeout=30

Если задание превышает максимальное количество попыток из-за постоянного тайм-аута, оно будет помечено как «неудачное».

Вы также можете определить максимальное количество секунд, в течение которого задание может выполняться в самом классе задания. Если для задания указан тайм-аут, он будет иметь приоритет над любым тайм-аутом, указанным в командной строке:

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * Количество секунд, в течение которых задание может выполняться до истечения тайм-аута.
         *
         * @var int
         */
        public $timeout = 120;
    }

Иногда процессы блокировки ввода-вывода, такие как сокеты или исходящие HTTP-соединения, могут не учитывать указанный вами тайм-аут. Следовательно, при использовании этих функций вы всегда должны пытаться указать тайм-аут, используя их API. Например, при использовании Guzzle вы всегда должны указывать значение таймаута соединения и запроса.

<a name="error-handling"></a>
### Обработка ошибок

Если во время обработки задания возникает исключение, задание автоматически возвращается в очередь, чтобы его можно было повторить. Задание будет продолжать возвращаться до тех пор, пока оно не будет выполнено максимальное количество раз, разрешенное вашим приложением. Максимальное количество попыток определяется переключателем `--tries`, используемым в команде `queue:work` Artisan. В качестве альтернативы максимальное количество попыток может быть определено в самом классе задания. Более подробную информацию о запуске обработчика очереди [можно найти ниже](#running-the-queue-worker).

<a name="manually-releasing-a-job"></a>
#### Manually Releasing A Job

По желанию можно вручную вернуть задание в очередь, чтобы его можно было повторить позже. Вы можете сделать это, вызвав метод `release`:

    /**
     * Выполнить задание.
     *
     * @return void
     */
    public function handle()
    {
        // ...

        $this->release();
    }

По умолчанию метод `release` помещает задание обратно в очередь для немедленной обработки. Однако, передав целое число методу `release`, вы можете указать очереди не делать задание доступным для обработки, пока не истечет заданное количество секунд:

    $this->release(10)

<a name="manually-failing-a-job"></a>
#### Manually Failing A Job

Иногда требуется вручную пометить задание как «неудачное». Для этого вы можете вызвать метод `fail`:

    /**
     * Выполнить задание.
     *
     * @return void
     */
    public function handle()
    {
        // ...

        $this->fail();
    }

Если вы хотите пометить свою работу как неудавшуюся из-за обнаруженного исключения, то вы можете передать исключение методу `fail`:

    $this->fail($exception);

> {tip} Для получения дополнительной информации об обработке невыполненных заданий обратитесь к [документации по разбору неудачных заданий](#dealing-with-failed-jobs).

<a name="job-batching"></a>
## Пакетная обработка заданий

Функционал пакетной обработки заданий Laravel позволяет вам легко выполнить пакет заданий, по завершению которого дополнительно совершить определенные действия. Перед тем, как начать, вы должны создать миграцию базы данных, чтобы построить таблицу, содержащую метаинформацию о ваших пакетах заданий, такую как процент их завершения. Эта миграция может быть сгенерирована с помощью команды `queue:batches-table` Artisan:

    php artisan queue:batches-table

    php artisan migrate

<a name="defining-batchable-jobs"></a>
### Определение пакета заданий

Чтобы определить задание с возможностью пакетной передачи, вы как обычно должны [создать задание в очереди](#creating-jobs); тем не менее, вы должны добавить к классу задания трейт `Illuminate\Bus\Batchable`. Этот трейт обеспечивает доступ к методу `batch`, который может использоваться для получения текущего пакета, в котором выполняется задание:

    <?php

    namespace App\Jobs;

    use Illuminate\Bus\Batchable;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Queue\SerializesModels;

    class ImportCsv implements ShouldQueue
    {
        use Batchable, Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * Выполнить задание.
         *
         * @return void
         */
        public function handle()
        {
            if ($this->batch()->cancelled()) {
                // Определяем, был ли пакет отменен ...

                return;
            }

            // Импортируем часть CSV-файла ...
        }
    }

<a name="dispatching-batches"></a>
### Отправка пакета заданий

Чтобы отправить пакет заданий, вы должны использовать метод `batch` фасада `Bus`. Конечно, обработка пакета заданий в первую очередь полезна в сочетании с вызовом замыканий по завершению. Итак, вы можете использовать методы `then`, `catch` и `finally` для определения замыканий завершения для пакета. Каждое из этих замыканий получит при вызове экземпляр `Illuminate\Bus\Batch`. В этом примере мы представим, что отправляем в очередь пакет заданий, каждое из которых обрабатывает указанное количество строк из файла CSV:

    use App\Jobs\ImportCsv;
    use Illuminate\Bus\Batch;
    use Illuminate\Support\Facades\Bus;
    use Throwable;

    $batch = Bus::batch([
        new ImportCsv(1, 100),
        new ImportCsv(101, 200),
        new ImportCsv(201, 300),
        new ImportCsv(301, 400),
        new ImportCsv(401, 500),
    ])->then(function (Batch $batch) {
        // Все задания успешно завершены ...
    })->catch(function (Batch $batch, Throwable $e) {
        // Обнаружено первое проваленное задание из пакета ...
    })->finally(function (Batch $batch) {
        // Завершено выполнение пакета ...
    })->dispatch();

    return $batch->id;

Идентификатор пакета, к которому можно получить доступ через свойство `$batch->id`, можно использовать для [запроса к командной шине Laravel](#inspecting-batches) для получения информации о пакете после того, как он был отправлен.

<a name="naming-batches"></a>
#### Именованные пакеты заданий

Некоторые инструменты, такие как Laravel Horizon и Laravel Telescope, могут предоставлять более удобную для пользователя отладочную информацию о пакет, если пакеты имеют имена. Чтобы присвоить пакету произвольное имя, вы можете вызвать метод `name` при определении пакета:

    $batch = Bus::batch([
        // ...
    ])->then(function (Batch $batch) {
        // Все задания успешно завершены ...
    })->name('Import CSV')->dispatch();

<a name="batch-connection-queue"></a>
#### Соединение и очередь пакета

Если вы хотите указать соединение и очередь, которые должны использоваться для пакетных заданий, то вы можете использовать методы `onConnection` и `onQueue`. Все пакетные задания должны выполняться в одном соединении и в одной очереди:

    $batch = Bus::batch([
        // ...
    ])->then(function (Batch $batch) {
        // Все задания успешно завершены ...
    })->onConnection('redis')->onQueue('imports')->dispatch();

<a name="chains-within-batches"></a>
#### Цепочки заданий внутри пакета

Вы можете определить набор [связанных заданий](#job-chaining) в пакете, поместив связанные задания в массив. Например, мы можем выполнить две цепочки заданий параллельно и выполнить замыкание, когда обе цепочки заданий завершат обработку:

    use App\Jobs\ReleasePodcast;
    use App\Jobs\SendPodcastReleaseNotification;
    use Illuminate\Bus\Batch;
    use Illuminate\Support\Facades\Bus;

    Bus::batch([
        [
            new ReleasePodcast(1),
            new SendPodcastReleaseNotification(1),
        ],
        [
            new ReleasePodcast(2),
            new SendPodcastReleaseNotification(2),
        ],
    ])->then(function (Batch $batch) {
        // ...
    })->dispatch();

<a name="adding-jobs-to-batches"></a>
### Добавление заданий в пакет заданий

Иногда может быть полезно добавить дополнительные задания в пакет, непосредственно из задания, уже находящегося в пакете. Этот шаблон может быть полезен, когда вам нужно выполнить пакетную обработку тысяч заданий, выполнение которых может занять слишком много времени во время веб-запроса. Таким образом, вместо этого вы можете отправить начальный пакет заданий «загрузчику», которые дополнят пакет еще большим количеством заданий:

    $batch = Bus::batch([
        new LoadImportBatch,
        new LoadImportBatch,
        new LoadImportBatch,
    ])->then(function (Batch $batch) {
        // Все задания успешно завершены ...
    })->name('Import Contacts')->dispatch();

В этом примере мы будем использовать задание `LoadImportBatch`, чтобы дополнить пакет дополнительными заданиями. Для этого мы можем использовать метод `add` экземпляра пакета, к которому можно получить доступ через метод `batch` задания:

    use App\Jobs\ImportContacts;
    use Illuminate\Support\Collection;

    /**
     * Выполнить задание.
     *
     * @return void
     */
    public function handle()
    {
        if ($this->batch()->cancelled()) {
            return;
        }

        $this->batch()->add(Collection::times(1000, function () {
            return new ImportContacts;
        }));
    }

> {note} Вы можете добавлять задания в пакет только из задания, которое принадлежит к тому же пакету.

<a name="inspecting-batches"></a>
### Инспектирование пакета

Экземпляр `Illuminate\Bus\Batch`, который передается замыканиям по завершению пакета, имеет множество свойств и методов, помогающих взаимодействовать с данным пакетом заданий и его анализа:

    // UUID пакета ...
    $batch->id;

    // Название пакета (если применимо) ...
    $batch->name;

    // Количество заданий, назначенных пакету ...
    $batch->totalJobs;

    // Количество заданий, которые не были обработаны очередью ...
    $batch->pendingJobs;

    // Количество неудачных заданий ...
    $batch->failedJobs;

    // Количество заданий, обработанных на данный момент ...
    $batch->processedJobs();

    // Процент завершения пакетной обработки (0-100) ...
    $batch->progress();

    // Указывает, завершено ли выполнение пакета ...
    $batch->finished();

    // Отменить выполнение пакета ...
    $batch->cancel();

    // Указывает, был ли пакет отменен ...
    $batch->cancelled();

<a name="returning-batches-from-routes"></a>
#### Возврат пакетов заданий из маршрутов

Все экземпляры `Illuminate\Bus\Batch` являются сериализуемыми в формате JSON, что означает, что вы можете возвращать их непосредственно из одного из маршрутов вашего приложения, чтобы получить полезную нагрузку JSON, содержащую информацию о пакете, включая ход его завершения. Это позволяет удобно отображать информацию о ходе выполнения пакета в пользовательском интерфейсе вашего приложения.

Чтобы получить пакет по его идентификатору, вы можете использовать метод `findBatch` фасада `Bus`:

    use Illuminate\Support\Facades\Bus;
    use Illuminate\Support\Facades\Route;

    Route::get('/batch/{batchId}', function (string $batchId) {
        return Bus::findBatch($batchId);
    });

<a name="cancelling-batches"></a>
### Отмена пакетов

Иногда требуется отменить выполнение определенного пакета. Это можно сделать, вызвав метод `cancel` экземпляра `Illuminate\Bus\Batch`:

    /**
     * Выполнить задание.
     *
     * @return void
     */
    public function handle()
    {
        if ($this->user->exceedsImportLimit()) {
            return $this->batch()->cancel();
        }

        if ($this->batch()->cancelled()) {
            return;
        }
    }

Как вы могли заметить в предыдущих примерах, пакетные задания обычно должны проверять, не был ли пакет отменен, в начале их метода `handle`:

    /**
     * Выполнить задание.
     *
     * @return void
     */
    public function handle()
    {
        if ($this->batch()->cancelled()) {
            return;
        }

        // Продолжаем обработку ...
    }

<a name="batch-failures"></a>
### Отказы в пакете заданий

Если задание в пакете завершается неуспешно, то будет вызвано замыкание `catch` (если назначено). Это замыкание вызывается только для первого проваленного задания в пакете.

<a name="allowing-failures"></a>
#### Допущение отказов

Когда задание в пакете завершается неуспешно, Laravel автоматически помечает пакет как «отмененный». При желании вы можете отключить это поведение, чтобы при провале задания пакет не отмечался автоматически как отмененный. Это может быть выполнено путем вызова метода `allowFailures` при отправке пакета:

    $batch = Bus::batch([
        // ...
    ])->then(function (Batch $batch) {
        // Все задания успешно завершены ...
    })->allowFailures()->dispatch();

<a name="retrying-failed-batch-jobs"></a>
#### Повторная попытка выполнения неудачных пакетных заданий

Для удобства Artisan содержит команду `queue:retry-batch`, которая позволяет вам легко повторить все неудачные задания для указанного пакета. Команда `queue:retry-batch` принимает UUID пакета, чьи неудачные задания следует повторить:

```bash
php artisan queue:retry-batch 32dbc76c-4f82-4749-b610-a639fe0099b5
```

<a name="pruning-batches"></a>
### Очистка пакетов

Если не применять очистку, то таблица `job_batches` может очень быстро накапливать записи. Чтобы избежать этого, вы должны [запланировать](scheduling) ежедневный запуск команды `queue:prune-batches` Artisan:

    $schedule->command('queue:prune-batches')->daily();

По умолчанию все готовые пакеты, возраст которых превышает 24 часа, будут удалены. Вы можете использовать параметр `hours` при вызове команды, чтобы определить, как долго хранить пакетные данные. Например, следующая команда удалит все пакеты, завершенные более 48 часов назад:

    $schedule->command('queue:prune-batches --hours=48')->daily();

<a name="queueing-closures"></a>
## Анонимные очереди

Вместо отправки класса задания в очередь вы также можете отправить замыкание. Это отлично подходит для быстрых и простых задач, которые необходимо выполнять вне текущего цикла запроса. При отправке замыканий в очередь содержимое кода замыкания криптографически подписывается, поэтому его нельзя изменить при передаче:

    $podcast = App\Podcast::find(1);

    dispatch(function () use ($podcast) {
        $podcast->publish();
    });

Используя метод `catch`, вы можете предоставить замыкание, которое должно быть выполнено, если анонимная очередь не завершится успешно после исчерпания всех [сконфигурированных попыток повтора](#max-job-attempts-and-timeout) вашей очереди:

    use Throwable;

    dispatch(function () use ($podcast) {
        $podcast->publish();
    })->catch(function (Throwable $e) {
        // Это задание завершилось неудачно ...
    });

<a name="running-the-queue-worker"></a>
## Запуск обработчика очереди

<a name="the-queue-work-command"></a>
### Команда `queue:work`

Laravel включает команду Artisan, которая запускает обработчика очереди и обрабатывает новые задания по мере их помещения в очередь. Вы можете запустить обработчик с помощью команды `queue:work` Artisan. Обратите внимание, что после запуска команды `queue:work` она будет продолжать работать, пока не будет остановлена вручную или пока вы не закроете терминал (консоль):

    php artisan queue:work

> {tip} Чтобы процесс `queue:work` постоянно работал в фоновом режиме, вы должны использовать диспетчер процессов, такой как [Supervisor](#supervisor-configuration), чтобы гарантировать, что обработчик очереди не перестанет работать.

Помните, что обработчики очереди – это долгоживущие процессы, которые хранят состояние загруженного приложения в памяти. В результате они не заметят изменений в вашей кодовой базе после их запуска. Итак, во время процесса развертывания обязательно [перезапустите своих обработчиков очереди](#queue-workers-and-deployment). Кроме того, помните, что любое статическое состояние, созданное или измененное вашим приложением, не будет автоматически пробрасываться между заданиями.

Как вариант, вы можете запустить команду `queue:listen`. При использовании команды `queue:listen` вам не нужно вручную перезапускать обработчик, если вы хотите перезагрузить обновленный код или сбросить состояние приложения; однако эта команда значительно менее эффективна, чем команда `queue:work`:

    php artisan queue:listen

<a name="running-multiple-queue-workers"></a>
#### Запуск нескольких обработчиков очереди

Чтобы назначить несколько обработчиков в очередь и обрабатывать задания одновременно, вы должны просто запустить несколько процессов `queue:work`. Это можно сделать либо локально с помощью нескольких вкладок в вашем терминале, либо в эксплуатационном режиме, используя параметры конфигурации вашего диспетчера процессов. [При использовании Supervisor](#supervisor-configuration) вы можете использовать значение конфигурации `numprocs`.

<a name="specifying-the-connection-queue"></a>
#### Указание соединения и очереди

Вы также можете указать, какое соединение очереди должен использовать обработчик. Имя соединения, переданное команде `work`, должно соответствовать одному из соединений, определенных в конфигурационном файле `config/queue.php`:

    php artisan queue:work redis

Дополнительно можно указать, какие очереди необходимо обрабатывать для указанного соединения. Например, если все ваши электронные письма обрабатываются в очереди `emails` соединения `redis`, то вы можете использовать команду, чтобы запустить обработчик только для этой очереди:

    php artisan queue:work redis --queue=emails

<a name="processing-a-specified-number-of-jobs"></a>
#### Обработка указанного количества заданий

Переключатель `--once` обработчика используется для указания обработать только одно задание из очереди:

    php artisan queue:work --once

Параметр `--max-jobs` обработчика проинструктирует его обработать заданное количество заданий, а затем выйти. Этот параметр может быть полезен в сочетании с [Supervisor](#supervisor-configuration), чтобы ваши рабочие процессы автоматически перезапускались после обработки заданного количества заданий, освобождая любую занятую ими память:

    php artisan queue:work --max-jobs=1000

<a name="processing-all-queued-jobs-then-exiting"></a>
#### Обработка всех заданий в очереди с последующим выходом

Переключатель `--stop-when-empty` обработчика может использоваться, чтобы дать ему указание обработать все задания и затем корректно завершить работу. Этот параметр может быть полезен при обработке очередей Laravel в контейнере Docker, если вы хотите выключить контейнер после того, как очередь пуста:

    php artisan queue:work --stop-when-empty

<a name="processing-jobs-for-a-given-number-of-seconds"></a>
#### Обработка заданий за заданное количество секунд

Параметр `--max-time` обработчика может использоваться, чтобы дать ему указание обрабатывать задания в течение заданного количества секунд, а затем выйти. Этот параметр может быть полезен в сочетании с [Supervisor](#supervisor-configuration), чтобы ваши рабочие процессы автоматически перезапускались после обработки заданий в течение заданного времени, освобождая любую занятую ими память:

    // Обрабатываем задания в течение одного часа, а затем выходим ...
    php artisan queue:work --max-time=3600

<a name="worker-sleep-duration"></a>
#### Продолжительность задержки выполнения обработчика

Когда задания доступны в очереди, обработчик будет продолжать обрабатывать задания без задержки между ними. Однако опция `sleep` определяет, сколько секунд обработчик будет «спать», если нет новых доступных заданий. Во время задержки выполнения обработчик не будет обрабатывать никаких новых заданий – задания будут обработаны после того, как обработчик снова проснется.

    php artisan queue:work --sleep=3

<a name="resource-considerations"></a>
#### Соображения относительно ресурсов

Демоны обработчиков очередей не «перезагружают» фреймворк перед обработкой каждого задания. Следовательно, вы должны освобождать все тяжелые ресурсы после завершения каждого задания. Например, если вы выполняете манипуляции с изображениями с помощью библиотеки GD, вы должны освободить память с помощью `imagedestroy`, когда вы закончите обработку изображения.

<a name="queue-priorities"></a>
### Приоритеты очереди

Иногда вы можете установить приоритетность обработки очередей. Например, в конфигурационном файле `config/queue.php` для очереди по умолчанию вашего соединения `redis` вы можете установить `low`. По желанию можно поместить задание в очередь с «высоким» (`high`) приоритетом, например:

    dispatch((new Job)->onQueue('high'));

Чтобы запустить обработчика, который проверяет, что все задания очереди `high` обработаны, прежде чем переходить к любым заданиям в очереди `low`, передайте разделенный запятыми список имен очередей команде `work`:

    php artisan queue:work --queue=high,low

<a name="queue-workers-and-deployment"></a>
### Обработчики очереди и развертывание

Поскольку обработчики очереди – это долгоживущие процессы, они не заметят изменений в вашем коде без перезапуска. Итак, самый простой способ развернуть приложение с использованием обработчиков очереди – это перезапустить обработчиков во время процесса развертывания. Вы можете корректно перезапустить всех обработчиков, используя команду `queue:restart`:

    php artisan queue:restart

Эта команда проинструктирует всех обработчиков очереди корректно выйти после завершения обработки своего текущего задания, чтобы существующие задания не были потеряны. Поскольку обработчики очереди выйдут при выполнении команды `queue:restart`, вы должны запустить диспетчер процессов, такой как [Supervisor](#supervisor-configuration), для автоматического перезапуска обработчиков очереди.

> {tip} Очередь использует [кеш](cache) для хранения сигналов перезапуска, поэтому перед использованием этой функции необходимо убедиться, что драйвер кеша правильно настроен для приложения.

<a name="job-expirations-and-timeouts"></a>
### Истечение срока и тайм-ауты задания

<a name="job-expiration"></a>
#### Истечение срока задания

В вашем файле конфигурации `config/queue.php` каждое соединение с очередью определяет параметр `retry_after`. Этот параметр указывает, сколько секунд соединение очереди должно ждать перед повторной попыткой выполнения задания, которое обрабатывается. Например, если значение `retry_after` установлено на `90`, задание будет возвращено в очередь, если оно обрабатывалось в течение 90 секунд, но не было высвобождено или удалено. Как правило, вы должны установить значение `retry_after` на максимальное количество секунд, которое может потребоваться вашим заданиям для завершения обработки.

> {note} Единственное соединение очереди, которое не содержит значения `retry_after` – это Amazon SQS. SQS будет повторять выполнение задания в соответствии с [таймаутом видимости по умолчанию](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html), управляемый консолью AWS.

<a name="worker-timeouts"></a>
#### Тайм-ауты обработчиков

Команда `queue:work` Artisan также содержит параметр `--timeout`. Если задание обрабатывается дольше, чем количество секунд, указанное значением тайм-аута, Обработчик, выполняющий задание, завершится с ошибкой. Обычно обработчик перезапускается автоматически [диспетчером, настроенным на вашем сервере](#supervisor-configuration):

```bash
php artisan queue:work --timeout=60
```

Параметр конфигурации `retry_after` и параметр `--timeout` Artisan отличаются, но работают вместе, чтобы гарантировать, что задания не будут потеряны и что задания будут успешно обработаны только один раз.

> {note} Значение `--timeout` всегда должно быть как минимум на несколько секунд короче, чем ваше значение конфигурации `retry_after`. Это гарантирует, что обрабатывающий замороженное задание обработчик, всегда завершает работу перед повторной попыткой выполнения задания. Если параметр `--timeout` выше значения конфигурации `retry_after`, то ваши задания могут быть обработаны дважды.

<a name="supervisor-configuration"></a>
## Конфигурация Supervisor

В эксплуатационном окружении вам нужен способ поддерживать процессы `queue:work` в рабочем состоянии. Процесс `queue:work` может перестать работать по разным причинам, например, из-за превышения тайм-аута обработчика или выполнения команды `queue:restart`.

По этой причине вам необходимо настроить диспетчер процессов, который может определять, когда ваши процессы `queue:work` завершаются, и автоматически перезапускать их. Кроме того, диспетчеры процессов могут позволить вам указать, сколько процессов `queue:work` вы хотите запускать одновременно. Supervisor – это диспетчер процессов, обычно используемый в средах Linux, и мы обсудим, как его настроить в следующей документации.

<a name="installing-supervisor"></a>
#### Установка Supervisor

Supervisor – это диспетчер процессов для операционной системы Linux, который автоматически перезапускает ваши процессы `queue:work` в случае их сбоя. Чтобы установить Supervisor в Ubuntu, вы можете использовать следующую команду:

    sudo apt-get install supervisor

> {tip} Если настройка Supervisor и управление им самостоятельно кажется ошеломляющим, рассмотрите возможность использования [Laravel Forge](https://forge.laravel.com), который автоматически установит и настроит Supervisor для ваших проектов Laravel.

<a name="configuring-supervisor"></a>
#### Настройка Supervisor

Файлы конфигурации Supervisor обычно хранятся в каталоге `/etc/supervisor/conf.d`. В этом каталоге вы можете создать любое количество файлов конфигурации, которые сообщают Supervisor, как следует контролировать ваши процессы. Например, давайте создадим файл `laravel-worker.conf`, который запускает и отслеживает процессы `queue:work`:

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/home/forge/app.com/worker.log
stopwaitsecs=3600
```

В этом примере директива `numprocs` инструктирует Supervisor запустить восемь процессов `queue:work` и отслеживать их все, автоматически перезапуская их в случае сбоя. Вы должны изменить директиву `command` конфигурации, чтобы отразить желаемое соединение с очередью и параметры обработчика.

> {note} Вы должны убедиться, что значение `stopwaitsecs` больше, чем количество секунд, затраченных на выполнение вашего самого продолжительного задания. В противном случае Supervisor может убить задание до того, как оно завершит обработку.

<a name="starting-supervisor"></a>
#### Запуск Supervisor

После создания файла конфигурации вы можете обновить конфигурацию Supervisor и запустить процессы, используя следующие команды:

```bash
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start laravel-worker:*
```

Для получения дополнительной информации о Supervisor обратитесь к [документации Supervisor](http://supervisord.org/index.html).

<a name="dealing-with-failed-jobs"></a>
## Разбор неудачных заданий

Иногда ваши задания в очереди терпят неудачу. Не волнуйтесь, не всегда все идет по плану! Laravel включает удобный способ [указать максимальное количество попыток выполнения задания](#max-job-attempts-and-timeout). После того, как задание превысит это количество попыток, оно будет вставлено в таблицу базы данных `failed_jobs`. Конечно, нам нужно будет создать эту таблицу, если она еще не существует. Чтобы создать миграцию для таблицы `failed_jobs`, вы можете использовать команду `queue:failed-table`:

    php artisan queue:failed-table

    php artisan migrate

При запуске [обработчика очереди](#running-the-queue-worker) вы можете указать максимальное количество попыток выполнения задания, используя переключатель `--tries` команды `queue:work`. Если вы не укажете значение для параметра `--tries`, задания будут выполняться только один раз или столько раз, сколько указано в свойстве класса задания `$tries`:

    php artisan queue:work redis --tries=3

Используя параметр `--backoff`, вы можете указать, сколько секунд Laravel должен ждать перед повторной попыткой выполнения задания, для которого возникло исключение. По умолчанию задание сразу же возвращается в очередь, чтобы его можно было повторить:

    php artisan queue:work redis --tries=3 --backoff=3

Если вы хотите настроить, сколько секунд Laravel должен ждать перед повторной попыткой выполнения каждого из заданий, для которого возникло исключение, вы можете сделать это, определив свойство `$backoff` в своем классе задания:

    /**
     * Количество секунд ожидания перед повторной попыткой выполнения задания.
     *
     * @var int
     */
    public $backoff = 3;

Если вам требуется более сложная логика для определения времени отсрочки выполнения задания, вы можете определить метод `backoff` для своего класса задания:

    /**
    * Рассчитать количество секунд ожидания перед повторной попыткой выполнения задания.
    *
    * @return int
    */
    public function backoff()
    {
        return 3;
    }

Вы можете легко настроить «экспоненциальную» отсрочку, возвращая массив значений отсрочки из метода `backoff`. В этом примере задержка повторной попытки выполнения будет составлять 1 секунду для первой попытки, 5 секунд для второй попытки и 10 секунд для третьей попытки:

    /**
    * Рассчитать количество секунд ожидания перед повторной попыткой выполнения задания.
    *
    * @return array
    */
    public function backoff()
    {
        return [1, 5, 10];
    }

<a name="cleaning-up-after-failed-jobs"></a>
### Очистка после неудачных заданий

В случае сбоя определенного задания вы можете отправить предупреждение своим пользователям или отменить любые действия, которые были частично выполнены заданием. Для этого вы можете определить метод `failed` в своем классе работы. Экземпляр `Throwable`, который привел к сбою задания, будет передан методу `failed`:

    <?php

    namespace App\Jobs;

    use App\Models\Podcast;
    use App\Services\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Queue\SerializesModels;
    use Throwable;

    class ProcessPodcast implements ShouldQueue
    {
        use InteractsWithQueue, Queueable, SerializesModels;

        /**
         * Экземпляр подкаста.
         *
         * @var \App\Podcast
         */
        protected $podcast;

        /**
         * Создать новый экземпляр задания.
         *
         * @param  \App\Models\Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * Выполнить задание.
         *
         * @param  \App\Services\AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // Process uploaded podcast...
        }

        /**
         * Обработать провал задания.
         *
         * @param  \Throwable  $exception
         * @return void
         */
        public function failed(Throwable $exception)
        {
            // Отправляем пользователю уведомление об ошибке и т.д.
        }
    }

<a name="retrying-failed-jobs"></a>
### Повторная попытка выполнения неудачных заданий

Чтобы просмотреть все неудачные задания, которые были вставлены в вашу таблицу базы данных `failed_jobs`, вы можете использовать команду `queue:failed` Artisan:

    php artisan queue:failed

Команда `queue:failed` перечислит идентификатор задания, соединение, очередь, время сбоя и другую информацию о задании. Идентификатор задания может быть использован для повторной попытки выполнить неудачное задание. Например, чтобы повторить неудачное задание с идентификатором `5`, введите следующую команду:

    php artisan queue:retry 5

При необходимости вы можете передать команде несколько идентификаторов или диапазон идентификаторов (при использовании числовых идентификаторов):

    php artisan queue:retry 5 6 7 8 9 10

    php artisan queue:retry --range=5-10

Чтобы повторить все неудачные задания, выполните команду `queue:retry` и передайте `all` вместо идентификаторов:

    php artisan queue:retry all

Если вы хотите удалить неудачные задание, вы можете использовать команду `queue:forget`:

    php artisan queue:forget 5

> {tip} При использовании [Horizon](horizon) вы должны использовать команду `horizon:forget` для удаления неудачного задания вместо команды `queue:forget`.

Чтобы удалить все неудачные задания из таблицы `failed_jobs`, вы можете использовать команду `queue:flush`:

    php artisan queue:flush

<a name="ignoring-missing-models"></a>
### Игнорирование отсутствующих моделей

При внедрении модели Eloquent в задание, модель автоматически сериализуется перед помещением в очередь и повторно извлекается из базы данных при обработке задания. Однако, если модель была удалена в то время, когда задание ожидало обработки, ваше задание может завершиться ошибкой с `ModelNotFoundException`.

Для удобства вы можете выбрать автоматическое удаление заданий с отсутствующими моделями, установив для свойства задания `$deleteWhenMissingModels` значение `true`. Когда для этого свойства установлено значение `true`, Laravel незаметно отбрасывает задание, не вызывая исключения:

    /**
     * Удалить задание, если модели больше не существуют.
     *
     * @var bool
     */
    public $deleteWhenMissingModels = true;

<a name="failed-job-events"></a>
### События неудачных заданий

Если вы хотите зарегистрировать слушатель событий, который будет вызываться при сбое задания, вы можете использовать метод `failing` фасада `Queue`. Например, мы можем передать замыкание этому событию из метода `boot` поставщика `AppServiceProvider`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Queue\Events\JobFailed;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Регистрация любых служб приложения.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Загрузка любых служб приложения.
         *
         * @return void
         */
        public function boot()
        {
            Queue::failing(function (JobFailed $event) {
                // $event->connectionName
                // $event->job
                // $event->exception
            });
        }
    }

<a name="clearing-jobs-from-queues"></a>
## Удаление заданий из очередей

> {tip} При использовании [Horizon](horizon) вы должны использовать команду `horizon:clear` для удаления заданий из очереди вместо команды `queue:clear`.

Если вы хотите удалить все задания, принадлежащие соединению и очереди по умолчанию, вы можете сделать это с помощью команды `queue:clear` Artisan:

    php artisan queue:clear

Вы также можете указать аргумент `connection` и параметр `queue` для удаления заданий из конкретного соединения / очереди:

    php artisan queue:clear redis --queue=emails

> {note} Удаление заданий из очередей доступно только для драйверов очереди SQS, Redis и базы данных. Кроме того, процесс удаления в SQS занимает до 60 секунд, поэтому задания, отправленные в очередь SQS в течение 60 секунд после очистки очереди, также могут быть удалены.

<a name="job-events"></a>
## События задания

Используя методы `before` и `after` [фасада](facades) `Queue`, вы можете указать замыкания, которые будут выполняться до или после обработки задания в очереди. Эти замыкания – прекрасная возможность для дополнительной регистрации или увеличения статистики для панели мониторинга. Как правило, вызов этих методов осуществляется в методе `boot` [поставщика служб](providers). Например, мы можем использовать `AppServiceProvider`, который включен в Laravel:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Queue\Events\JobProcessed;
    use Illuminate\Queue\Events\JobProcessing;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Регистрация любых служб приложения.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Загрузка любых служб приложения.
         *
         * @return void
         */
        public function boot()
        {
            Queue::before(function (JobProcessing $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });

            Queue::after(function (JobProcessed $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });
        }
    }

Используя метод `looping` [фасада](facades) `Queue`, вы можете указать замыкания, которые выполняются до того, как обработчик попытается получить задание из очереди. Например, вы можете зарегистрировать замыкание для отката любых транзакций, оставшихся открытыми из-за ранее неудачного задания:

    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\Facades\Queue;

    Queue::looping(function () {
        while (DB::transactionLevel() > 0) {
            DB::rollBack();
        }
    });
