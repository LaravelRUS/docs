git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Очереди

- [Введение](#introduction)
    - [Подключения против Очередей](#connections-vs-queues)
    - [Требования для драйверов](#driver-prerequisites)
- [Создание задач](#creating-jobs)
    - [Генерирование классов задач](#generating-job-classes)
    - [Структура класса](#class-structure)
- [Добавление задач в очередь](#dispatching-jobs)
    - [Отложенные задачи](#delayed-dispatching)
    - [Настройка очереди и подключения](#customizing-the-queue-and-connection)
    - [Указание макс. попыток задач / значений таймаута](#max-job-attempts-and-timeout)
    - [Обработка ошибок](#error-handling)
- [Выполнение воркера очереди](#running-the-queue-worker)
    - [Приоритеты очереди](#queue-priorities)
    - [Воркеры очереди и развертывание](#queue-workers-and-deployment)
    - [Истечение срока задачи и Таймауты](#job-expirations-and-timeouts)
- [Настройка Supervisor](#supervisor-configuration)
- [Проваленные задачи](#dealing-with-failed-jobs)
    - [Очистка после проваленных задач](#cleaning-up-after-failed-jobs)
    - [События проваленных задач](#failed-job-events)
    - [Повторный запуск проваленных задач](#retrying-failed-jobs)
- [События задач](#job-events)

<a name="introduction"></a>
## Введение

Компонент Laravel Queue предоставляет единое API для различных сервисов очередей. Очереди позволяют вам отложить выполнение времязатратной задачи, такой как отправка e-mail, на более позднее время, таким образом на порядок ускоряя обработку запросов в вашем приложении.

Настройки очередей хранятся в файле `config/queue.php`. В нём вы найдёте настройки для каждого драйвера очереди, которые поставляются вместе с фреймворком: база данных, [Beanstalkd](https://kr.github.io/beanstalkd/), [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](http://redis.io),  а также синхронный драйвер (для локального использования). Драйвер очереди `null` просто отменяет задачи очереди, поэтому они никогда не выполнятся.

<a name="connections-vs-queues"></a>
### Подключения против Очередей

Прежде чем начать работать с очередями Laravel, важно понимать различия между "подключениями" и "очередями". В вашем конфиге `config/queue.php` есть опция настройки `connections`. Она определяет определенное полдключение к бэкенд-сервису, такому как Amazon SQS, Beanstalk или Redis. Однако, у любого заданного подключения очереди есть несколько "очередей", о которых можно думать как о различных стеках или пачках задач, стоящих в в очереди.

Обратите внимание, что каждый пример настройки подключения в конфиге `queue` содержит атрибут `queue`. Это очередь, в которую по умолчанию буду попадать задачи, когда они отправляются на заданное подключение. Другими словами, если вы отправите задачу без четкого указания того, в какую очередь её следует послать, эта задача будет помещена в ту очередь, которая определена атрибутом `queue` настройки подключения:

    // Эта задача отправляется в очередь по умолчанию...
    dispatch(new Job);

    // Эта задача отправляется в очередь "emails"...
    dispatch((new Job)->onQueue('emails'));

Некоторым приложениям может даже не требоваться помещать задачи в несколько очередей; вместо этого им будет предпочтительно иметь одну простую очередь. Однако, помещать задачи в несколько очередей может быть особенно полезно для приложений, которые хотят приоретизировать или сегментировать то, как обрабатываются задачи, так как воркер очередей Laravel позволяет указывать приоритетность очередей. Например, если вы поместите задачу в очередь `high`, вы можете запустить воркера, который присвоит ей более высокий приоритет обработки:

    php artisan queue:work --queue=high,default

<a name="driver-prerequisites"></a>
### Требования для драйверов

#### База данных

Для использования драйвера очереди `database` вам понадобится таблица в БД для хранения задач. Чтобы генерировать миграцию для создания этой таблицы, выполните Artisan-команду `queue:table`. Когда миграция создана, вы можете мигрировать свою базу данных командой `migrate`:

    php artisan queue:table

    php artisan migrate

#### Redis

Для использования драйвера очереди `redis` вам потребуется настроить подключение к БД Redis в конфиге `config/database.php`.

Если ваше Redis-подключение использует Redis Cluster, название вашей очередт должно содержать [ключевой хэштег (key hash tag)](https://redis.io/topics/cluster-spec#keys-hash-tags). Это обязательно, чтобы убедиться, что все ключи Redis для заданной очереди помещены в один и тот же хэш-слот:

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => '{default}',
        'retry_after' => 90,
    ],

#### Требования других драйверов

Упомянутым выше драйверам нужны следующие зависимости:

<div class="content-list" markdown="1">
- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~3.0`
- Redis: `predis/predis ~1.0`
</div>

<a name="creating-jobs"></a>
## Создание задач

<a name="generating-job-classes"></a>
### Генерирование классов задач

По умолчанию все помещаемые в очередь задачи вашего приложения хранятся в директории `app/Jobs`. Вы можете сгенерировать новую задачу для очереди с помощью Artisan-команды `make:job`. Новую задачу в очереди можно сгенерировать используя Artisan CLI:

    php artisan make:job SendReminderEmail

Сгенерированный класс будет реализацией интерфейса `Illuminate\Contracts\Queue\ShouldQueue` - так Laravel поймёт, что задачу надо поместить в очередь, а не выполнить немедленно.

<a name="class-structure"></a>
### Структура класса

Классы задач очень просты, обычно они содержат только метод `handle`, который вызывается при обработке задачи в очереди. Для начала давайте посмотрим на пример класса задачи. В этом примере мы представим, что управляем сервисом публикации подкастов и нам нужно обработать загруженные файлы подкастов прежде, чем они будут опубликованы:

    <?php

    namespace App\Jobs;

    use App\Podcast;
    use App\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;

    class ProcessPodcast implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        protected $podcast;

        /**
         * Создать новый экземпляр задачи.
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * Выполнить задачу.
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // Process uploaded podcast...
        }
    }

Обратите внимание, в этом примере мы можем передать [модель Eloquent](/docs/{{version}}/eloquent) напрямую в конструктор задачи. Благодаря используемому в задаче трейту `SerializesModels`, модели Eloquent будут изящно сериализованы и десериализованы при выполнении задачи. Если ваша задача принимает модель Eloquent в своём конструкторе, в очередь будет сериализован только идентификатор модели. А когда очередь начнёт обработку задачи, система очередей автоматически запросит полный экземпляр модели из БД. Это всё полностью прозрачно для вашего приложения и помогает избежать проблем, связанных с сериализацией полных экземпляров моделей Eloquent.

Метод `handle` вызывается при обработке задачи очередью. Обратите внимание, что мы можем указывать зависимости в методе `handle` задачи. [Сервис-контейнер](/docs/{{version}}/container) Laravel автоматически внедрит эти зависимости.

> {note} Двоичные данные, такие как raw-содержимое изображений, нужно передавать через функцию `base64_encode` перед передачей в задачу в очереди. Иначе задача может некорректно сериализироваться в JSON во время помещения в очередь.

<a name="dispatching-jobs"></a>
## Добавление задач в очередь

Как только вы написали класс своей очереди, его можно направить используя хелпер `dispatch`. Единственный аргумент, который следует передавать хелперу `dispatch` - экземпляр задачи:

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Хранить новый подкаст.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            dispatch(new ProcessPodcast($podcast));
        }
    }

> {tip} Хелпер `dispatch` предоставляет удобство короткой, глобально доступной функции, в то же время являясь очень простым в тестировании. Ознакомьтесь с [документацией о тестировании](/docs/{{version}}/testing) Laravel, чтобы узнать об этом побольше.

<a name="delayed-dispatching"></a>
### Отложенные задачи

Если вам нужно задержать выполнение задачи в очереди, то можно использовать метод `delay` на экземпляре своей задачи. Метод `delay` предоставляется трейтом `Illuminate\Bus\Queueable`, который по умолчанию включен на всех генерируемых классах задач. Например, давайте укажем, что задаче не следует быть доступной для обработки до истечения 10 минут после ее отправки:

    <?php

    namespace App\Http\Controllers;

    use Carbon\Carbon;
    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Хранить новый подкаст.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            $job = (new ProcessPodcast($podcast))
                        ->delay(Carbon::now()->addMinutes(10));

            dispatch($job);
        }
    }

> {note} У сервиса задач Amazon SQS максимальное время загрузки составляет 15 минут.

<a name="customizing-the-queue-and-connection"></a>
### Настройка очереди и подключения

#### Задание очереди для задачи

Помещая задачи в разные очереди, вы можете разделять их по категориям, а также задавать приоритеты по количеству обработчиков разных очередей. Это не касается различных «подключений» очередей, определённых в файле настроек очереди, а только конкретных очередей в рамках одного подключения. Чтобы указать очередь используйте метод `onQueue` на экземпляре задачи:

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Хранить новый подкаст.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            $job = (new ProcessPodcast($podcast))->onQueue('processing');

            dispatch($job);
        }
    }

#### Указание подключения к очереди для задачи

Если вы работаете с несколькими подключениями к очередям, то можете указать, в какое из них надо поместить задачу. Для этого служит метод `onConnection` на экземпляре задачи:

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Хранить новый подкаст.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            $job = (new ProcessPodcast($podcast))->onConnection('sqs');

            dispatch($job);
        }
    }

Само собой, вы можете сцепить методы `onConnection` и `onQueue`, чтобы указать подключение и очередь для задачи:

    $job = (new ProcessPodcast($podcast))
                    ->onConnection('sqs')
                    ->onQueue('processing');

<a name="max-job-attempts-and-timeout"></a>
### Указание макс. попыток задач / значений таймаута

#### Максимальное число попыток

Один из подходов к определению максимального количества попыток задания может быть выполнен с помощью оператора выбора `--tries` в командной строке Artisan:

    php artisan queue:work --tries=3

Однако, можно использовать более детализированный подход, указав максимальное число попыток в самом классе задачи. Если для задачи указано максимальное число попыток, это будет иметь приоритет над значением, указанным в командной строке:

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * Количество раз, которое можно попробовать выполнить задачу.
         *
         * @var int
         */
        public $tries = 5;
    }

#### Таймаут

> {note} Функция `timeout` fоптимизирована для PHP 7.1+ и PHP-расширения `pcntl`.

Аналогично, максимальное количество секунд, которые может быть запущена задача, можно указать посредством параметра `--timeout` в командной строке Artisan:

    php artisan queue:work --timeout=30

Вы также можете самостоятельно задать максимальное количество секунд, на протяжении которых может выполняться задача. Если для задачи указан таймаут, это будет иметь приоритет над таймаутом, указанным в командной строке:

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * Количество секунд, во время которых может выполняться задача до таймаута.
         *
         * @var int
         */
        public $timeout = 120;
    }

<a name="error-handling"></a>
### Обработка ошибок

Если во время обработки задачи выбрасывается исключение, задача будет автоматически помещена обратно в очередь, чтобы ее можно было бы попытаться выполнить снова. Задача будет продолжать попытки выполнения, пока не достигнет установленного в вашем приложении максимального количества попыток. Максимальное количество попыток определено в параметре `--tries`, используемом в Artisan-команде `queue:work`. В качестве альтернативы, максимальное количество попыток можно задать в самом классе задачи. Больше информации о воркере очереди [можно найти ниже](#running-the-queue-worker).

<a name="running-the-queue-worker"></a>
## Выполнение воркера очереди

В Laravel включен воркер очереди, который будет обрабатывать новые задачи по мере их помещения в очередь. Воркер можно запустить при помощи Artisan-команды `queue:work`. Обратите внимание, что как только была запущена команда `queue:work`, она продолжит выполняться до тех пор, пока ее не остановить вручную, или пока вы не закроете терминал:

    php artisan queue:work

> {tip} Чтобы процесс `queue:work` постоянно работал в фоне, нужно использовать инструмент мониторинга процессов, такой как [Supervisor](#supervisor-configuration), чтобы убедиться, что ворке очереди не перестал работать.

Помните, что воркеры очереди - длительные процессы и хранят в памяти состояние загруженного приложения. В результате, они не заметят изменений в вашей базе кода после своего запуска. Поэтому во время процесса развертывания убедитесь, что [перезапустили воркеров очереди](#queue-workers-and-deployment).

#### Обработка одной задачи

Опцию `--once` можно использовать, чтобы указать воркеру обработать только одну задачу из очереди:

    php artisan queue:work --once

#### Указание подключения и очереди

Можно указать какое подключение к очереди должен использовать воркер. Название подключения передается команде `work` - оно должен соответствовать одному из подключений, заданных в вашем конфиге `config/queue.php`:

    php artisan queue:work redis

Можно настроить ваш воркер очереди и далее просто обработыв определенные очереди для заданного подключения. Например, если все ваши email-сообщения обрабатываются в очереди `emails` вашего подключения очереди `redis`, вы можете использовать следующую команду, чтобы запустить воркер, который обрабатыват только эту очередь:

    php artisan queue:work redis --queue=emails

#### Рекомендации по ресурсам

Демоны-воркеры очереди не "перезагружают" фреймворк перед обработкой каждой задачи. Таким образом, вам нужно освободить ресурсы после выполнения каждой задачи. К примеру, если вы совершаете манипуляции с изображениями при помощи библиотеки GD, вам нужно освободить память при помощи `imagedestroy` после того как вы закончили.

<a name="queue-priorities"></a>
### Приоритеты очереди

Иногда вам может потребоваться приоретизировать то, как обрабатываются ваши очереди. Например, в`config/queue.php` можно задать `queue`по умолчанию для вашего подключения `redis` равным `low`. Однако, время от времени вы можете захотеть присвоить очереди высокий приоритет следующим образом:

    dispatch((new Job)->onQueue('high'));

Для запуска вворкера, который проверяет, что все `high`-задачи очереди обрабатываются до перехода к любым `low`-задачам в этой же очереди, передайте разделенный запятой список названий очередей команде `work`:

    php artisan queue:work --queue=high,low

<a name="queue-workers-and-deployment"></a>
### Воркеры очереди и развертывание

Так как воркеры очереди - длительные процессы и хранят в памяти состояние загруженного приложения. В результате, они не заметят изменений в вашей базе кода после своего запуска. Поэтому самый простой способ развернуть приложения используя воркеры очереди - перезагрузить воркеров во время процесса развертывания. Это можно сделать командой `queue:restart`:

    php artisan queue:restart

Данная команда поручит всем воркерам корректно завершить работу после того как они завершат обработку своей текущей задачи, чтобы не потерять ни одну из существующих задач. Так как воркеры прекратят свою работу после выполнения команды `queue:restart`, у вас должен быть запущен менеджер процессов, такой как [Supervisor](#supervisor-configuration). Он поможет автоматически перезагрузить воркеров очереди.

> {tip} Очередь использует [кэш](/docs/{{version}}/cache) для хранения сигналов перезапуска, поэтому следует убедиться в правильной настройке драйвера кэша для вашего приложения прежде, чем использовать этот функционал.

<a name="job-expirations-and-timeouts"></a>
### Истечение срока задачи и Таймауты

#### Истечение срока задачи

В вашем конфиге `config/queue.php` каждое подключение очереди определяет опцию `retry_after`. Данная опция указывает сколько секунд должно ждать подключение очереди перед повторным выполнением задачи, которая сейчас обрабатывается. Например, если значение `retry_after` равно `90`, задача будет выпущена обратно в очередь, если она обрабатывалась на протяжении 90 секунд без удаления. Обычно вам нужно будет устанавливать значение `retry_after` равным максимальному количеству секунд, на протяжении которых ваща задача должна успеть выполнить, что полагается.

> {note} Есдинственное подключение очереди, у которого нет значения `retry_after` - Amazon SQS. SQS заново попробует выполнить задачу основываясь на [Таймауте видимости по умолчанию (Default Visibility Timeout)](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html), которым можно управлять из консоли AWS.

#### Таймауты воркера

Artisan-команда `queue:work` раскрывает опцию `--timeout`. Опция `--timeout` указывает как долго мастер-процессу очереди Laravel нужно ждать перед завершением дочернего воркера очереди, который обрабатывает задачу. Иногда дочерний процесс очереди может быть "заморожен" по различным причинам, таким как внешняя HTTP-функция, которая не отвечает. Опция `--timeout` убирает замороженные процессы, которые превзошли этот указанный лимит времени:

    php artisan queue:work --timeout=60

Опция настройки `retry_after` и CLI-опция `--timeout` различаются, но работают вместе, чтобы убедиться в том, что задачи не потеряны и что задачи успешно обрабатываются только один раз.

> {note} Значение `--timeout` всегда должно быть равным как минимум на несколько секунд короче, чем ваше значение `retry_after`. Таким образом, воркер, обрабатывающий определенную задачу, всегда уничтожается перед повторной попыткой выполнить задачу. Если ваша опция `--timeout` дольше, чем значение `retry_after`, ваши задачи обрабатываться дважды.

#### Продолжительность сна воркера

Когда задачи доступны в очереди воркер будет продолжать обрабатывать задачи без задержки между ними. Однако, опция `sleep` определяет как долго будет "спать" воркер, если нет новых доступных задач. Во время сна воркер не будет обрабатывать никакие новые задачи - все они будут обработаны после повторного пробуждения воркера.

    php artisan queue:work --sleep=3

<a name="supervisor-configuration"></a>
## Настройка Supervisor

#### Установка Supervisor

Supervisor — монитор процессов для ОС Linux, он автоматически перезапустит ваш процесс `queue:work`, если он остановится. Для установки Supervisor в Ubuntu можно использовать такую команду:

    sudo apt-get install supervisor

> {tip} Если самостоятельно настроить Supervisor вам сложно, попробуйте использовать [Laravel Forge](https://forge.laravel.com), который автоматически установит и настроит Supervisor для ваших Laravel-проектов.

#### Настройка Supervisor

Файлы настроек Supervisor обычно находятся в директории `/etc/supervisor/conf.d`. Там вы можете создать любое количество файлов с настройками, по которым Supervisor поймёт, как отслеживать ваши процессы. Например, давайте создадим файл `laravel-worker.conf`, который запускает и наблюдает за процессом `queue:work`:

    [program:laravel-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3
    autostart=true
    autorestart=true
    user=forge
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/worker.log

В этом примере директива `numprocs` указывает, что Supervisor должен запустить 8 процессов `queue:work` и наблюдать за ними, автоматически перезапуская их при их остановках. Само собой, вам надо изменить часть `queue:work sqs` директивы `command` в соответствии с вашим драйвером очереди.

#### Запуск Supervisor

После создания файла настроек вы можете обновить конфигурацию Supervisor и запустить процесс при помощи следующих команд:

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start laravel-worker:*

Подробнее о настройке и использовании Supervisor читайте в [документации Supervisor](http://supervisord.org/index.html).

<a name="dealing-with-failed-jobs"></a>
## Проваленные задачи

Не всегда всё идёт по плану, иногда ваши задачи в очереди будут заканчиваться ошибкой. Не волнуйтесь, такое с каждым случается! В Laravel есть удобный способ указать максимальное количество попыток выполнения задачи. После превышения этого количества попыток задача будет добавлена в таблицу `failed_jobs`. Для создании миграции таблицы `failed_jobs` можно использовать команду `queue:failed-table`:

    php artisan queue:failed-table

    php artisan migrate

Затем,, во время запуска вашего [воркера очереди](#running-the-queue-worker), вы должны указать максимальное количество попыток выполнить задачу, используя оператор выбора `--tries` команды `queue:work`. Если вы не укажете значение для настройки `--tries`, попытки выполнения задачи будут неограничены:

    php artisan queue:work redis --tries=3

<a name="cleaning-up-after-failed-jobs"></a>
### Очистка после проваленных задач

Вы можете задать метод `failed` напрямую в своем классе задач, что позволит вам выполнить очистку специально после этой задачи в случае провала. Это отличное место для отправки уведомления вашим пользователям или для отката любых действий, выполненных задачей. То исключение `Exception`, которое способствовало провалу задачи, будет передано методу `failed`:

    <?php

    namespace App\Jobs;

    use Exception;
    use App\Podcast;
    use App\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class ProcessPodcast implements ShouldQueue
    {
        use InteractsWithQueue, Queueable, SerializesModels;

        protected $podcast;

        /**
         * Создать новый экземпляр задачи.
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * Выполнить задачу.
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // Process uploaded podcast...
        }

        /**
         * Неудачная обработка задачи.
         *
         * @param  Exception  $exception
         * @return void
         */
        public function failed(Exception $exception)
        {
            // Send user notification of failure, etc...
        }
    }

<a name="failed-job-events"></a>
### События проваленных задач

Если вы хотите зарегистрировать событие, которое будет вызываться при ошибке выполнения задачи, можете использовать метод `Queue::failing`. Это событие — отличная возможность оповестить вашу команду через e-mail или [HipChat](https://www.hipchat.com). Например, мы можем прикрепить анонимную функцию к данному событию из `AppServiceProvider`, который включён в Laravel:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Queue\Events\JobFailed;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Начальная загрузка всех сервисов приложения.
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

        /**
         * Регистрация сервис-провайдера.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="retrying-failed-jobs"></a>
### Повторный запуск проваленных задач

Чтобы просмотреть все проваленные задачи, которые были помещены в вашу таблицу `failed_jobs`, можно использовать Artisan-команду `queue:failed`:

    php artisan queue:failed

Эта команда выведет список задач с их ID, подключением, очередью и временем ошибки. ID задачи можно использовать для повторной попытки её выполнения. Например, для повторной попытки выполнения задачи с ID= 5 надо выполнить такую команду:

    php artisan queue:retry 5

Чтобы повторить все проваленные задачи, используйте `queue:retry` с указанием `all` в качестве ID:

    php artisan queue:retry all

Если вы хотите удалить проваленную задачу, используйте команду `queue:forget`:

    php artisan queue:forget 5

Для удаления всех проваленных задач используйте команду `queue:flush`:

    php artisan queue:flush

<a name="job-events"></a>
## События задач

Используя методы `before` и `after` [фасада](/docs/{{version}}/facades) `Queue`, вы можете указать анонимные функции, которые нужно выполнить перед или после выполнения задачи в очереди. Эти анонимные функции - отличная возможность выполнить дополнительное логгирование или накопительную статистику для панели управления. Как правило, эти методы следует вызывать из [сервис-провайдера](/docs/{{version}}/providers). Например, мы можем использовать `AppServiceProvider`, который входит в состав Laravel:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Queue\Events\JobProcessed;
    use Illuminate\Queue\Events\JobProcessing;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Начальная загрузка всех сервисов приложения.
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

        /**
         * Регистрация сервис-провайдера.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Используя метод `looping` на [фасаде](/docs/{{version}}/facades) `Queue`, вы можете указать анонимные функции, которые будут выполнены до того как воркер попробует получить задачу из очереди. Например, вы можете зарегистрировать функцию Closure для отката любых транзакций, которые остались открытыми в результате предыдущей проваленной задачи:

    Queue::looping(function () {
        while (DB::transactionLevel() > 0) {
            DB::rollBack();
        }
    });
