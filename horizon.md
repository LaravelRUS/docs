---
git: 2cf67bcaacfec590098cefb45af824b74671cfa0
---

# Laravel Horizon

<a name="introduction"></a>
## Вступление

> {tip} Прежде чем углубляться в Laravel Horizon, вам следует ознакомиться с базовыми [службами очередей](/docs/{{version}}/queues) Laravel. Horizon дополняет очередь Laravel дополнительными функциями, которые могут сбивать с толку, если вы еще не знакомы с основными функциями, предлагаемыми Laravel.

[Laravel Horizon](https://github.com/laravel/horizon) предоставляет красивую панель управления и конфигурацию для ваших [очередей Redis](/docs/{{version}}/queues), работающих на Laravel. Horizon позволяет легко отслеживать ключевые показатели системы очередей, такие как пропускная способность, время выполнения и сбои заданий.

При использовании Horizon вся ваша конфигурация обработчика очереди хранится в одном простом файле конфигурации. Определив конфигурацию воркеров (worker) приложения в файле с контролем версий, вы можете легко масштабировать или изменять воркеры очереди приложения при развертывании.

<img src="https://laravel.com/img/docs/horizon-example.png" alt="horizon-example.png" width="100%">

<a name="installation"></a>
## Установка

> {note} Laravel Horizon требует, чтобы вы использовали [Redis](https://redis.io) для управления очередью. Следовательно, вы должны убедиться, что соединение с очередью настроено на `redis` в файле конфигурации приложения `config/queue.php`.

Вы можете установить Horizon в свой проект с помощью диспетчера пакетов Composer:

    composer require laravel/horizon

После установки Horizon опубликуйте его ресурсы с помощью Artisan-команды `horizon:install`:

    php artisan horizon:install

<a name="configuration"></a>
### Настройка

После публикации ресурсов Horizon его основной файл конфигурации будет расположен по адресу `config/horizon.php`. Этот файл конфигурации позволяет вам настроить параметры обработчика очереди для приложения. Каждый вариант конфигурации включает описание своего назначения, поэтому обязательно внимательно изучите этот файл.

> {note} Horizon использует Redis-соединение с именем «horizon». Это имя соединения Redis зарезервировано и не должно назначаться другому Redis-соединению в файле конфигурации `database.php` или в качестве значения параметра` use` в файле конфигурации `horizon.php`.

<a name="environments"></a>
#### Окружение

Основным параметром конфигурации Horizon, с которым вы должны ознакомиться после установки, является параметр конфигурации `environments`. Этот параметр конфигурации представляет собой массив сред, в которых работает ваше приложение, и определяет параметры рабочего процесса для каждой среды. По умолчанию эта запись содержит окружение `production` и `local`. Однако вы можете добавлять дополнительные среды по мере необходимости:

    'environments' => [
        'production' => [
            'supervisor-1' => [
                'maxProcesses' => 10,
                'balanceMaxShift' => 1,
                'balanceCooldown' => 3,
            ],
        ],

        'local' => [
            'supervisor-1' => [
                'maxProcesses' => 3,
            ],
        ],
    ],

Когда вы запускаете Horizon, он будет использовать параметры конфигурации рабочего процесса для среды, в которой работает ваше приложение. Как правило, среда определяется значением `APP_ENV` [переменной среды](/docs/{{version}}/configuration#determining-the-current-environment). Например, стандартная локальная среда Horizon настроена на запуск трех рабочих процессов и автоматическое выравнивание количества рабочих процессов, назначенных каждой очереди. По умолчанию рабочая среда настроена на запуск максимум 10 рабочих процессов и автоматический баланс количества рабочих процессов, назначенных каждой очереди.

> {note} Вы должны убедиться, что раздел `environment` файла конфигурации` horizon` содержит запись для каждой [среды](/docs/{{version}}/configuration#environment-configuration), в которой вы планируете запускать Horizon.

<a name="supervisors"></a>
####  Supervisors (Наблюдатели)

Как вы можете увидеть в файле конфигурации Horizon по умолчанию — каждая среда может содержать один или несколько "supervisors" (наблюдателей). По умолчанию в файле конфигурации этот supervisor определяется как `supervisor-1`; однако вы можете называть своих supervisors как хотите. Каждый supervisor, по сути, отвечает за "наблюдение" за группой рабочих процессов и заботится о балансировке рабочих процессов по очередям.

Вы можете добавить дополнительных "supervisors" в данную среду, если хотите определить новую группу рабочих процессов, которые должны выполняться в этой среде. Вы можете сделать это, если хотите определить другую стратегию балансировки или количество рабочих процессов для данной очереди, используемой вашим приложением.

<a name="default-values"></a>
#### Значения по умолчанию

В файле конфигурации Horizon по умолчанию вы заметите параметр конфигурации `defaults`. Эта опция конфигурации определяет значения по умолчанию для [Supervisors](#supervisors) приложения. Значения конфигурации супервизора по умолчанию будут объединены с конфигурацией супервизора для каждой среды, что позволит вам избежать ненужного повторения при определении супервизоров.

<a name="balancing-strategies"></a>
### Стратегии балансировки

В отличие от стандартной системы очередей Laravel, Horizon позволяет вам выбирать из трех стратегий балансировки рабочих процессов (worker): `simple`, `auto` и `false`. Стратегия `simple`, которая используется в конфигурационном файле по умолчанию, равномерно распределяет входящие задания между рабочими процессами:

    'balance' => 'simple',

Стратегия `auto` регулирует количество рабочих процессов в очереди на основе текущей рабочей нагрузки очереди. Например, если ваша очередь `notifications` имеет 1000 ожидающих заданий, а ваша очередь `render` пуста, Horizon будет выделять больше воркеров в очередь `notifications`, пока она не станет пустой.

При использовании стратегии `auto` вы можете определить параметры конфигурации `minProcesses` и `maxProcesses` для управления минимальным и максимальным количеством рабочих процессов, которые Horizon должен масштабировать в большую или меньшую стороны:

    'environments' => [
        'production' => [
            'supervisor-1' => [
                'connection' => 'redis',
                'queue' => ['default'],
                'balance' => 'auto',
                'minProcesses' => 1,
                'maxProcesses' => 10,
                'balanceMaxShift' => 1,
                'balanceCooldown' => 3,
                'tries' => 3,
            ],
        ],
    ],

Значения конфигурации `balanceMaxShift` и `balanceCooldown` определяют, насколько быстро Horizon будет масштабироваться в соответствии с требованиями рабочих процессов. В приведенном выше примере каждые три секунды будет создаваться или уничтожаться максимум один новый процесс. Вы можете изменять эти значения по мере необходимости в зависимости от потребностей вашего приложения.

Когда для параметра `balance` установлено значение `false`, будет использоваться поведение Laravel по умолчанию, обрабатывающее очереди в том порядке, в котором они перечислены в конфигурации.

<a name="dashboard-authorization"></a>
### Авторизация в информационной панели (dashboard)

Horizon предоставляет информационную панель (dashboard) по URI `/horizon`. По умолчанию вы сможете получить доступ к этой панели инструментов только в локальной среде. Однако в файле `app/Providers/HorizonServiceProvider.php` есть определение [шлюза авторизации](/docs/{{version}}/authorization#gates). Этот шлюз контролирует доступ к Horizon **во внешних средах**. Вы можете настроить этот шлюз по мере необходимости, чтобы ограничить доступ к вашему приложению Horizon:

    /**
     * Регистрация шлюза Horizon.
     *
     * Этот шлюз определяют, кто может получить доступ к Horizon во внешней среде.
     *
     * @return void
     */
    protected function gate()
    {
        Gate::define('viewHorizon', function ($user) {
            return in_array($user->email, [
                'taylor@laravel.com',
            ]);
        });
    }

<a name="alternative-authentication-strategies"></a>
#### Альтернативные стратегии аутентификации

Помните, что Laravel автоматически добавляет к шлюзу аутентифицированного пользователя замыкание (closure). Если ваше приложение обеспечивает безопасность Horizon с помощью другого метода, например ограничения IP-адресов, то пользователям Horizon может и не требоваться "аутентификация". Следовательно, нужно будет изменить написание функции (сигнатуру) выше с `function ($user)` на `function ($user = null)`, чтобы Laravel не требовал аутентификации.

<a name="upgrading-horizon"></a>
## Обновление Horizon

При обновлении до новой версии Horizon важно внимательно изучить [руководство по обновлению](https://github.com/laravel/horizon/blob/master/UPGRADE.md). Кроме того, при обновлении до любой новой версии Horizon вам следует повторно опубликовать ресурсы Horizon:

    php artisan horizon:publish

Чтобы поддерживать ресурсы в актуальном состоянии и избежать проблем в будущих обновлениях, вы можете добавить команду `horizon: publish` к сценариям `post-update-cmd` в файле `composer.json` вашего приложения:

    {
        "scripts": {
            "post-update-cmd": [
                "@php artisan horizon:publish --ansi"
            ]
        }
    }

<a name="running-horizon"></a>
## Запуск Horizon

После того как вы настроили свои супервизоры (supervisors) и рабочие процессы (workers) в файле конфигурации приложения `config/horizon.php`, вы можете запустить Horizon, используя Artisan-команду `horizon`. Эта единственная команда запустит все настроенные рабочие процессы для текущей среды:

    php artisan horizon

Вы можете приостановить процесс Horizon и дать ему указание продолжить обработку заданий, используя Artisan-команды `horizon:pause`и `horizon:continue`:

    php artisan horizon:pause

    php artisan horizon:continue

Вы также можете приостановить и продолжить определенные Horizon [супервизоры](#supervisors), используя Artisan-команды `horizon:pause-supervisor` и `horizon:continue-supervisor`:

    php artisan horizon:pause-supervisor supervisor-1

    php artisan horizon:continue-supervisor supervisor-1

Вы можете проверить текущий статус процесса Horizon, используя Artisan-команду `horizon:status`:

    php artisan horizon:status

Вы можете корректно завершить процесс Horizon, используя Artisan-команду `horizon:terminate`. Все задания, которые в настоящее время обрабатываются, будут завершены, а затем Horizon прекратит работу:

    php artisan horizon:terminate

<a name="deploying-horizon"></a>
### Развертывание Horizon (deploy)

Когда вы будете готовы развернуть Horizon на фактическом сервере приложения, вам следует настроить монитор процессов для отслеживания командой `php artisan horizon` и перезапустить ее, если она неожиданно завершится. Не волнуйтесь, ниже мы обсудим, как установить монитор процессов.

Во время процесса развертывания приложения вы должны дать команду Horizon завершить процесс, чтобы он был перезапущен монитором процессов и получил изменения кода:

    php artisan horizon:terminate

<a name="installing-supervisor"></a>
#### Установка Supervisor

Supervisor - это монитор процессов для операционной системы Linux, который автоматически перезапустит ваш процесс `horizon`, если он перестанет выполняться. Чтобы установить Supervisor в Ubuntu, вы можете использовать следующую команду. Если вы не используете Ubuntu, вы, вероятно, можете установить Supervisor с помощью диспетчера пакетов вашей операционной системы:

    sudo apt-get install supervisor

> {tip} Если настройка Supervisor сама по себе кажется утомительной, рассмотрите возможность использования [Laravel Forge](https://forge.laravel.com), который автоматически установит и настроит Supervisor для ваших проектов Laravel.

<a name="supervisor-configuration"></a>
#### Настройка Supervisor

Файлы конфигурации супервизора обычно хранятся в каталоге вашего сервера `/etc/supervisor/conf.d`. В этом каталоге вы можете создать любое количество файлов конфигурации, которые определяют для супервизора, как следует контролировать процессы. Например, давайте создадим файл `horizon.conf`, который запускает и отслеживает процесс `horizon`:

    [program:horizon]
    process_name=%(program_name)s
    command=php /home/forge/example.com/artisan horizon
    autostart=true
    autorestart=true
    user=forge
    redirect_stderr=true
    stdout_logfile=/home/forge/example.com/horizon.log
    stopwaitsecs=3600

> {note} Вы должны убедиться, что значение `stopwaitsecs` больше, чем количество секунд, потребляемых вашим самым длительным выполняемым заданием. В противном случае Supervisor может удалить ваш процесс задания до того, как оно завершит обработку.

<a name="starting-supervisor"></a>
#### Запуск Supervisor

После создания файла конфигурации вы можете обновить конфигурацию Supervisor и запустить отслеживаемые процессы, используя следующие команды:

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start horizon

> {tip} Для получения дополнительной информации о запуске Supervisor обратитесь к [документации Supervisor](http://supervisord.org/index.html).

<a name="tags"></a>
## Теги

Horizon позволяет назначать “теги” (tags) заданиям, включая почтовые сообщения, широковещательные события, уведомления и прослушиватели событий в очереди. Фактически, Horizon будет интеллектуально и автоматически помечать большинство заданий в зависимости от моделей Eloquent, прикрепленных к заданию. Например, взгляните на следующее задание (job):

    <?php

    namespace App\Jobs;

    use App\Models\Video;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Queue\SerializesModels;

    class RenderVideo implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * The video instance.
         *
         * @var \App\Models\Video
         */
        public $video;

        /**
         * Create a new job instance.
         *
         * @param  \App\Models\Video  $video
         * @return void
         */
        public function __construct(Video $video)
        {
            $this->video = $video;
        }

        /**
         * Execute the job.
         *
         * @return void
         */
        public function handle()
        {
            //
        }
    }

Если это задание поставлено в очередь с экземпляром `App\Models\Video` с атрибутом `id` равным `1`, то оно автоматически получит тег `App\Models\Video:1`. Это потому, что Horizon будет искать в свойствах задания любые модели Eloquent. Если модели Eloquent будут найдены, Horizon разумно пометит задание, используя имя класса модели и первичный ключ:

    use App\Jobs\RenderVideo;
    use App\Models\Video;

    $video = Video::find(1);

    RenderVideo::dispatch($video);

<a name="manually-tagging-jobs"></a>
#### Самостоятельное тегирование заданий

Если вы хотите самостоятельно определить теги для одного из объектов в очереди, вы можете определить в классе метод "tags()":

    class RenderVideo implements ShouldQueue
    {
        /**
         * Получить теги которые назначаются заданию.
         *
         * @return array
         */
        public function tags()
        {
            return ['render', 'video:'.$this->video->id];
        }
    }

<a name="notifications"></a>
## Уведомления

> {note} При настройке Horizon для отправки уведомлений Slack или SMS необходимо ознакомиться с [предварительными условиями для соответствующего канала уведомлений](/docs/{{version}}/notifications).

Если вы хотите получать уведомления, когда одна из ваших очередей имеет длительное время ожидания, вы можете использовать методы `Horizon::routeMailNotificationsTo`, `Horizon::routeSlackNotificationsTo` и `Horizon::routeSmsNotificationsTo`. Вы можете вызвать эти методы из метода `boot` [провайдера](/docs/{{version}}/providers) вашего приложения `App\Providers\HorizonServiceProvider`:

    /**
     * Загрузчик сервисов приложения.
     *
     * @return void
     */
    public function boot()
    {
        parent::boot();

        Horizon::routeSmsNotificationsTo('15556667777');
        Horizon::routeMailNotificationsTo('example@example.com');
        Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
    }

<a name="configuring-notification-wait-time-thresholds"></a>
#### Настройка пороговых значений времени ожидания уведомлений

Вы можете настроить, сколько секунд будет считаться "долгим ожиданием" в файле конфигурации Horizon `config/horizon.php`. Параметр конфигурации `waits` в этом файле позволяет вам настраивать пороги ожидания для каждой комбинации соединения/очереди:

    'waits' => [
        'redis:default' => 60,
        'redis:critical,high' => 90,
    ],

<a name="metrics"></a>
## Метрики

Horizon включает панель показателей, которая предоставляет информацию о времени ожидания и пропускной способности вашего задания и очереди. Чтобы заполнить эту информационную панель, вы должны настроить Artisan-команду Horizon `snapshot` на запуск каждые пять минут через [планировщик (scheduler)](/docs/{{version}}/scheduling) вашего приложения:

    /**
     * Определение расписания для команд.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('horizon:snapshot')->everyFiveMinutes();
    }

<a name="deleting-failed-jobs"></a>
## Удаление невыполненных заданий

Если вы хотите удалить неудавшееся задание, можете использовать команду `horizon:forget`. Команда `horizon:forget` принимает идентификатор или UUID неудачного задания в качестве своего единственного аргумента:

    php artisan horizon:forget 5

<a name="clearing-jobs-from-queues"></a>
## Удаление заданий из очередей

Если вы хотите удалить все задания из очереди приложения по умолчанию, то вы можете сделать это с помощью Artisan-команды `horizon:clear`:

    php artisan horizon:clear

Можно добавить опцию `queue` для удаления заданий из определенной очереди:

    php artisan horizon:clear --queue=emails
