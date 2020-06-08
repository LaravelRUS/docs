git c406fc00d091db5638115113ae9ada0557584b49

---

# Уведомления

- [Введение](#introduction)
- [Создание уведомлений](#creating-notifications)
- [Отправка уведомлений](#sending-notifications)
    - [Использование трейта Notifiable](#using-the-notifiable-trait)
    - [Использование фасада Notification](#using-the-notification-facade)
    - [Указание каналов доставки](#specifying-delivery-channels)
    - [Формирование очередей уведомлений](#queueing-notifications)
- [Mail-уведомления](#mail-notifications)
    - [Форматирование Mail-сообщений](#formatting-mail-messages)
    - [Настройка получателя](#customizing-the-recipient)
    - [Настройка темы](#customizing-the-subject)
    - [Настройка шаблонов](#customizing-the-templates)
- [Markdown Mail-уведомления](#markdown-mail-notifications)
    - [Генерирование сообщения](#generating-the-message)
    - [Написание сообщения](#writing-the-message)
    - [Настройка компонентов](#customizing-the-components)
- [БД-уведомления](#database-notifications)
    - [Требования](#database-prerequisites)
    - [Форматирование БД-уведомлений](#formatting-database-notifications)
    - [Доступ к уведомлениям](#accessing-the-notifications)
    - [Пометить уведомления как прочитанные](#marking-notifications-as-read)
- [Уведомления вещания](#broadcast-notifications)
    - [Требования](#broadcast-prerequisites)
    - [Форматирование уведомлений вещания](#formatting-broadcast-notifications)
    - [Слушать уведомления](#listening-for-notifications)
- [SMS-уведомления](#sms-notifications)
    - [Требования](#sms-prerequisites)
    - [Форматирование SMS-уведомлений](#formatting-sms-notifications)
    - [Настройка номера "From"](#customizing-the-from-number)
    - [Роутинг SMS-уведомлений](#routing-sms-notifications)
- [Slack-уведомления](#slack-notifications)
    - [Требования](#slack-prerequisites)
    - [Форматирование Slack-уведомлений](#formatting-slack-notifications)
    - [Slack-вложения](#slack-attachments)
    - [Роутинг Slack-уведомлений](#routing-slack-notifications)
- [События уведомлений](#notification-events)
- [Пользовательские каналы](#custom-channels)

<a name="introduction"></a>
## Введение

Дополнительно к поддержке [отправки email-сообщений](/docs/{{version}}/mail), Laravel также поддерживает и отправку уведомлений по различным каналам доставки, включая почту, SMS (через [Nexmo](https://www.nexmo.com/)) и [Slack](https://slack.com). Уведомления также можно хранить в БД, поэтому их можно отображать в вашем интерфейсе.

Как правило, уведомления должны быть короткими, информативными сообщениями, которые уведомляют пользователей о каком-либо событий, произошедшем в вашем приложении. Например, если вы пишете биллинг-приложение, то можете отправлять своим пользователям уведомление об оплаченном счете через каналы email и SMS.

<a name="creating-notifications"></a>
## Создание уведомлений

В Laravel каждое уведомление представлено единым классом (обычно хранится в директории `app/Notifications`). Не волнуйтесь, если не видите эту директорию в своем приложении, т.к. она будет создана, когда вы запустите Artisan-команду `make:notification`:

    php artisan make:notification InvoicePaid

Эта команда поместит свежий класс уведомлений в вашу директорию `app/Notifications`. Каждый класс уведомления содержит метод `via` и переменный номер методов построения сообщений (например, `toMail` или `toDatabase`), которые конвертируют уведомление в сообщение, оптимизированное для конкретного канала.

<a name="sending-notifications"></a>
## Отправка уведомлений

<a name="using-the-notifiable-trait"></a>
### Использование трейта Notifiable

Уведомления можно отправлять двумя способами: используя метод `notify` трейта `Notifiable` или используя [фасад](/docs/{{version}}/facades) `Notification`. Сначала давайте рассмотрим использование трейта:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;
    }

Этот трейт используется моделью `App\User` по умолчанию и содержит один метод, который можно использовать для отправки уведомлений: `notify`. Метод `notify` ожидает получить экземпляр уведомления:

    use App\Notifications\InvoicePaid;

    $user->notify(new InvoicePaid($invoice));

> {tip} Помните, что можете использовать трейт `Illuminate\Notifications\Notifiable` на любой своей модели. Вы не ограничены включением его только в вашу модель `User`.

<a name="using-the-notification-facade"></a>
### Использование фасада Notification

Другой способ отправки уведомлений - через [фасад](/docs/{{version}}/facades) `Notification`. Это полезно, прежде всего, когда вам нужно отправить уведомление нескольким уведомляемым объектам, таким как коллекция пользователей. Чтобы отправлять уведомления с использованием фасада, передайте все уведомляемые объекты и экземпляр уведомления методу `send`:

    Notification::send($users, new InvoicePaid($invoice));

<a name="specifying-delivery-channels"></a>
### Указание каналов доставки

У каждого класса уведомлений есть метод `via`, который определяет по каким каналам будет доставляться это уведомление. Изначально уведомления можно отправлять на каналы `mail`, `database`, `broadcast`, `nexmo` и `slack`.

> {tip} Если вы бы хотели использовать другие каналы доставки, такие как Telegram или Pusher, ознакомьтесь с управляемым сообществом [вебсайтом о каналах уведомлений Laravel](http://laravel-notification-channels.com).

Метод `via` получает экземпляр `$notifiable`, который будет экземпляром класса, которому отправляется уведомление. Можно использовать `$notifiable`, чтобы определить по каким каналам следует доставлять уведомление:

    /**
     * Получить каналы доставки уведомления.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return $notifiable->prefers_sms ? ['nexmo'] : ['mail', 'database'];
    }

<a name="queueing-notifications"></a>
### Формирование очередей уведомлений

> {note} Перед формированием очередей уведомлений вам следует настроить свою очередь и [запустить воркер](/docs/{{version}}/queues).

Отправка уведомлений может занять время, особенно если каналу требуется вызывать внешний API с целью доставки этих уведомлений. Чтобы ускорить время ответа вашего приложения, позвольте своим уведомлениям формировать очереди, добавив интерфейс `ShouldQueue` и трейт `Queueable` к своему классу. Этот интерфейс и трейт уже испортированы для всех уведомлений, сгенерированных с использованием `make:notification`, так что вы можете сразу же добавить их к свой класс уведомлений:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use Illuminate\Notifications\Notification;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;

        // ...
    }

Как только к вашему уведомлению был добавлен интерфейс `ShouldQueue`, вы можете отправлять уведомления как обычно. Laravel определит интерфейс `ShouldQueue` в классе и автоматически поставит доставку уведомления в очередь:

    $user->notify(new InvoicePaid($invoice));

Если вы бы хотели отложить доставку уведомления, то можно привязать метод `delay` к экземпляру вашего уведомления:

    $when = Carbon::now()->addMinutes(10);

    $user->notify((new InvoicePaid($invoice))->delay($when));

<a name="mail-notifications"></a>
## Mail-уведомления

<a name="formatting-mail-messages"></a>
### Форматирование Mail-сообщений

Если уведомление поддерживает только отправку в виде электронного сообщения, вы должны задать класс `toMail` в классе уведомления. Этот метод получит сущность `$notifiable` и должен возвратить экземпляр `Illuminate\Notifications\Messages\MailMessage`. Mail-сообщения могут содержать строки текста, а также "призыв к действию". Давайте взглянем на пример метода `toMail`:

    /**
     * Получить представление уведомления в виде письма.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->greeting('Hello!')
                    ->line('One of your invoices has been paid!')
                    ->action('View Invoice', $url)
                    ->line('Thank you for using our application!');
    }

> {tip} Обратите внимание, что мы используем `$this->invoice->id` в нашем методе `message`. В конструктор уведомления можно передавать любые данные, которые требуются уведомлению для генерирования своего сообщения.

В этом примере мы зарегистрируем приветствие, строку текста, призыв к действию, а затем еще одну строку текста. Эти методы, предоставляемые объектом `MailMessage`, делают форматирование небольших email-сообщений простым и быстрым. Почтовый канал затем будет преобразовывать компоненты сообщения в симпатичный, отзывчивый HTML email-шаблон с копией только с простым текстом. Вот пример электронного сообщения, генерируемого каналом `mail`:

<img src="https://laravel.com/assets/img/notification-example.png" width="551" height="596">

{tip} При отправке уведомлений по почте не забудьте установить значение `name` в вашем конфиге `config/app.php`. Это значение будет использоваться в заголовке и подвале ваших сообщений-уведомлений по почте.

#### Другие опции форматирования уведомления

Вместо рпделения "строк" текста в классе уведомления, можно использовать метод`view`, чтобы указать пользовательский шаблон, который следует использовать для визуализации email-уведомления:

    /**
     * Получить представление уведомления в виде письма.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)->view(
            'emails.name', ['invoice' => $this->invoice]
        );
    }

Дополнительно, можно возвращать [mailable-объект](/docs/{{version}}/mail) из метода `toMail`:

    use App\Mail\InvoicePaid as Mailable;

    /**
     * Получить представление уведомления в виде письма.
     *
     * @param  mixed  $notifiable
     * @return Mailable
     */
    public function toMail($notifiable)
    {
        return (new Mailable($this->invoice))->to($this->user->email);
    }

<a name="error-messages"></a>
#### Сообщения об ошибке

Некоторые уведомления оповещают пользователей об ошибках, например, об ошибке при оплате счета. Можно указать, что это почтовое сообщение об ошибке, вызвав метод `error` при построении вашего сообщения. При использовании метода `error` в Mail-сообщении кнопка призыва к действию будет красной, а не синей:

    /**
     * Получить представление уведомления в виде письма.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Message
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->error()
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-recipient"></a>
### Настройка получателя

При отправке уведомлений через канал `mail`, система уведомлений будет автоматически искать свойство `email` в экземпляре notifiable. Можно настроить какие email-адреса используются для доставки уведомлений, определив в сущности метод `routeNotificationForMail`:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Уведомления роута для mail-канала.
         *
         * @return string
         */
        public function routeNotificationForMail()
        {
            return $this->email_address;
        }
    }

<a name="customizing-the-subject"></a>
### Настройка темы

По умолчанию тема сообщения - название класса уведомления, форматированное в виде капитализации начальных букв всех слов в предложении ("title case"). Таким образом, если класс вашего уведомления носит название  `InvoicePaid`, то тема этого email-сообщения будет `Invoice Paid` (счет оплачен). Если вы хотите указать явно заданную тему собщения, то можно вызвать метод `subject` при построении вашего сообщения:

    /**
     * Получить представление уведомления в виде письма.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-templates"></a>
### Настройка шаблонов

Вы можете изменить HTML-шаблон и шаблон только с текстом, используемые уведомлениями по почте, опубликовав ресурсы пакета уведомлений. После запуска этой команды шаблоны уведомлений будут располагаться в директории `resources/views/vendor/notifications`:

    php artisan vendor:publish --tag=laravel-notifications

<a name="markdown-mail-notifications"></a>
## Markdown Mail-уведомления

Почтовые уведомления в формате markdown позволяют воспользоваться заранее построенными шаблонами почтовых уведомлений, в то же время давая вам свободу в написании более длинных, настроенные по вашему вкусу сообщений. Так как сообщения пишутся в формате markdown, Laravel может отображать красивые, отзывчивые HTML шаблоны для сообщений, в то же время генерируя их копию исключительно в виде текста.

<a name="generating-the-message"></a>
### Генерирование сообщения

Чтобы сгенерировать уведомление с соответствующим Markdown-шаблоном можно использовать опцию `--markdown` Artisan-команды `make:notification`:

    php artisan make:notification InvoicePaid --markdown=mail.invoice.paid

Как и все другие почтовые уведомления, уведомления с Markdown-шаблонами должны определять метод `toMail` в классе уведомления. Однако, вместо использования методов `line` и `action` для конструирования уведомления, используйте метод `markdown` для указания названия Markdown-шаблона, который следует использовать:

    /**
     * Получить представление уведомления в виде письма.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }

<a name="writing-the-message"></a>
### Написание сообщения

Почтовые уведомления в формате markdown используют комбинацию компонентов Blade и синтаксиса Markdown, что позволяет вам запросто конструировать уведомления, пользуясь предварительно созданными компонентами уведомлений Laravel:

    @component('mail::message')
    # Счет оплачен

    Ваш счет был оплачен!

    @component('mail::button', ['url' => $url])
    Просмотреть счет
    @endcomponent

    Спасибо,<br>
    {{ config('app.name') }}
    @endcomponent

#### Компонент Button

Компонент кнопки отображает выравненную по центру ссылку-кнопку. Этот компонент принимает два аргумента: `url` и необязательный `color`. Поддерживаются цвета: синий `blue`, зеленый `green` и красный `red`. К уведомлению можно добавлять сколько угодно компонентов-кнопок:

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    Просмотреть счет
    @endcomponent

#### Компонент Panel

Этот компонент отображает заданный блок текста на области, у которой цвет заднего фона для заданной области текста слегка отличается от фона остальной части уведомления. Это позволяет привлечь внимание к заданному блоку текста:

    @component('mail::panel')
    This is the panel content.
    @endcomponent

#### Компонент Table

Компонент таблиц позволяет вам трансформировать Markdown-таблица в HTML-таблицу. Этот компонент принимает Markdown-таблицу и ее содержимое. Поддерживается выравнивание столбцов благодаря Markdown синтаксису выравнивания таблиц по умолчанию:

    @component('mail::table')
    | Laravel       | Table         | Example  |
    | ------------- |:-------------:| --------:|
    | Col 2 is      | Centered      | $10      |
    | Col 3 is      | Right-Aligned | $20      |
    @endcomponent

<a name="customizing-the-components"></a>
### Настройка компонентов

Вы можете экспортировать все Markdown-компоненты уведомления в собственное приложение с целью настройки. Чтобы экспортировать компоненты используйте Artisan-команду `vendor:publish` для публикации тега ассетов `laravel-mail`:

    php artisan vendor:publish --tag=laravel-mail

Эта команда опубликует почтовые компоненты Markdown в директорию `resources/views/vendor/mail`. В директории `mail` будут содержаться директории `html` и`markdown`, в каждой из которых содержится соответствующее представление каждого доступного компонента. Вы можете свободно изменять эти компоненты по собственному желанию.

#### Настройка CSS

После экспортирования компонентов директория `resources/views/vendor/mail/html/themes` будет содержать файл`default.css`. Вы можете настроить CSS в этом файле и ваши стили будут автоматически приведены в соответствие с HTML-представлениями ваших Markdown-уведомлений.

> {tip} Если вы бы хотели построить полностью новую тему для Markdown-компонентов, просто создайте новый файл CSS в директории `html/themes` и измените параметр `theme` конфига `mail`.

<a name="database-notifications"></a>
## БД-уведомления

<a name="database-prerequisites"></a>
### Требования

Канал уведомлений `database` хранит информацию уведомления в таблице базы данных. Эта таблица будет содержать такую информацию как тип уведомления, а также пользовательские данные JSON, которые описывают  уведомление.

Вы можете запросить, чтобы таблица отображала уведомления в пользовательском интерфейсе вашего приложения. Но, прежде чем вы сможете это сделать, вам потребуется создать таблицу БД, которая будет содержать ваши уведомления. Можно использовать команду `notifications:table` для генерирования миграции с подходящей схемой таблицы:

    php artisan notifications:table

    php artisan migrate

<a name="formatting-database-notifications"></a>
### Форматирование БД-уведомлений

Если уведомление можно хранить в таблице БД, вам следует задать метод `toDatabase` или `toArray` в классе уведомления. Этот метод получит сущность `$notifiable` и должен вернуть простой PHP массив. Возвращенный массив будет кодирован как JSON и будет храниться в столбце `data` вашей таблицы `notifications`. Давайте взглянем на метод-пример `toArray`:

    /**
     * Получить представление уведомления в виде массива.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toArray($notifiable)
    {
        return [
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ];
    }

#### `toDatabase` против `toArray`

Метод `toArray` также используется каналом `broadcast`, чтобы определить какие данные вещать вашему JavaScript-клиенту. Если вы хотите, чтобы у вас было два разных представления массива для каналов `database` и `broadcast`, вам следует задать метод `toDatabase` вместо метода `toArray`.

<a name="accessing-the-notifications"></a>
### Доступ к уведомлениям

Если уведомления хранятся в БД, то вам потребуется удобный способ получить к ним доступ из уведомляемых (notifiable) сущностей. Трейт `Illuminate\Notifications\Notifiable`, который включен в модель Laravel `App\User` по умолчанию, включает Eloquent-отношение `notifications`, которое возвращает уведомления для сущности. В целях выборки уведомлений можно получить доступ к данному методу как и к любому другому Eloquent-отношению. По умолчанию уведомления будут отсортированы по метке `created_at`:

    $user = App\User::find(1);

    foreach ($user->notifications as $notification) {
        echo $notification->type;
    }

Если вы хотите получить только "непрочитанные" уведомления, можно использовать отношение `unreadNotifications`. Опять же, эти уведомления будут отсортированы по метке `created_at`:

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        echo $notification->type;
    }

> {tip} Для доступа к уведомлениям из вашего JavaScript-клиента вам следует задать контроллер уведомлений для своего приложения, который будет возвращать уведомления для уведомляемой сущности, такой как текущий пользователь. Затем вы можете выполнить HTTP-запрос к URI этого контроллера из своего JavaScript-клиента.

<a name="marking-notifications-as-read"></a>
### Пометить уведомления как прочитанные

Как правило, нужно будет помечать уведомления "прочитанными" после того, как пользователь просмотрит эти уведомления. Трейт `Illuminate\Notifications\Notifiable` предоставляет метод `markAsRead`, который обновляет столбец `read_at` в записи уведомления в БД:

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        $notification->markAsRead();
    }

Однако, вмето прохождения в цикле по каждому уведомлению можно использовать метод `markAsRead` напрямую на коллекции уведомлений:

    $user->unreadNotifications->markAsRead();

Вы также можете использовать запрос массового обновления, чтобы пометить все уведомления прочитанными без получения их из базы данных:

    $user = App\User::find(1);

    $user->unreadNotifications()->update(['read_at' => Carbon::now()]);

Конечно, можно удалить (`delete`) уведомления, чтобы полностью убрать их из таблицы:

    $user->notifications()->delete();

<a name="broadcast-notifications"></a>
## Бродкаст-уведомления

<a name="broadcast-prerequisites"></a>
### Требования

Перед вещанием уведомлений вам следует настроить и ознакомиться с сервисами [бродкаста событий](/docs/{{version}}/broadcasting) Laravel. Вещание событий предоставляет способ реагировать на выбрасываемые со стороны сервера события Laravel от вашего JavaScript-клиента.

<a name="formatting-broadcast-notifications"></a>
### Форматирование уведомлений вещания

Уведомления вещания канала `broadcast`, использующие сервисы [вещания событий](/docs/{{version}}/broadcasting) Laravel, позволяют вашему JavaScript-клиенту ловить уведомления в режиме реального времени. Если уведомление поддерживает бродкаст, нужно задать метод `toBroadcast` в классе уведомления. Этот метод получит сущность `$notifiable` и должен вернуть экземпляр `BroadcastMessage`. Возвращаемые данные будут кодированы как JSON и будут отправлены по вебсокет-совдинению вашему JavaScript-клиенту. Рассмотрим пример метода `toBroadcast`:

    use Illuminate\Notifications\Messages\BroadcastMessage;

    /**
     * Получить вещаемое представление уведомления.
     *
     * @param  mixed  $notifiable
     * @return BroadcastMessage
     */
    public function toBroadcast($notifiable)
    {
        return new BroadcastMessage([
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ]);
    }

#### Настройка очереди вещания

Все уведомления вещания становятся в очередь на вещание. Если вам нужно настроить подключение очереди или имя очереди, можно использовать методы `onConnection` и `onQueue` в `BroadcastMessage`:

    return (new BroadcastMessage($data))
                    ->onConnection('sqs')
                    ->onQueue('broadcasts');

> {tip} Дополнительно к указываемым данным, уведомления вещания также будут содержать поле `type`, содержащее имя класса уведомления.

<a name="listening-for-notifications"></a>
### Слушать уведомления

Уведомления будут вещаться на приватном канале, форматированном по конвенции `{notifiable}.{id}`. Поэтому если вы отправляете уведомление экземпляру `App\User` с ID равным `1`, уведомление будет вещаться только на приватном канале `App.User.1`. При использовании [Laravel Echo](/docs/{{version}}/broadcasting) можно легко слушать уведомления на канале, используя метод-хелпер `notification`:

    Echo.private('App.User.' + userId)
        .notification((notification) => {
            console.log(notification.type);
        });

#### Настройка канала уведомлений

Можно задать метод `receivesBroadcastNotificationsOn` в уведомляемой сущности, если вы хотите настроить по каким каналам уведомляемая сущность будет получать свои уведомления:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * The channels the user receives notification broadcasts on.
         *
         * @return string
         */
        public function receivesBroadcastNotificationsOn()
        {
            return 'users.'.$this->id;
        }
    }

<a name="sms-notifications"></a>
## SMS-уведомления

<a name="sms-prerequisites"></a>
### Требования

Отправка SMS-уведомлений в Laravel поддерживается благодаря [Nexmo](https://www.nexmo.com/). Прежде чем вы сможете отправлять уведомления через Nexmo, вам потребуется установить пакет Composer `nexmo/client` и добавить несколько опций настройки в свой конфиг `config/services.php`. Для начала вы можете скопировать пример настройки ниже:

    'nexmo' => [
        'key' => env('NEXMO_KEY'),
        'secret' => env('NEXMO_SECRET'),
        'sms_from' => '15556666666',
    ],

Опция `sms_from` - это номер телефона, с которого будут отправляться ваши SMS-сообщения. Генерировать номер телефона для вашего приложения следует на панели управления Nexmo.

<a name="formatting-sms-notifications"></a>
### Форматирование SMS-уведомлений

Если уведомление поддерживает отправку через SMS, нужно задать метод `toNexmo` в классе уведомления. Данный метод получит сущность `$notifiable` и должен вернуть экземпляр `Illuminate\Notifications\Messages\NexmoMessage`:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content');
    }

#### Юникод-контент

Если ваше SMS-сообщение будет содержать символы в кодировке юникод, нужно вызвать метод `unicode` во время конструирования экземпляра `NexmoMessage`:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your unicode message')
                    ->unicode();
    }

<a name="customizing-the-from-number"></a>
### Настройка номера "From"

Если вам нужно отправить некоторые уведомления с номера, отличающегося от телефонного номера, указанного в файле `config/services.php`, можно использовать метод `from` на экземпляре `NexmoMessage`:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content')
                    ->from('15554443333');
    }

<a name="routing-sms-notifications"></a>
### Роутинг SMS-уведомлений

При отправке уведомлений через канал `nexmo` система уведомлений будет автоматически искать атрибут `phone_number` уведомляемой сущности. В случае если нужно настроить номер телефона, куда доставляется уведомление, задайте в сущности метод `routeNotificationForNexmo`:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Роут-уведомления для канала Nexmo.
         *
         * @return string
         */
        public function routeNotificationForNexmo()
        {
            return $this->phone;
        }
    }

<a name="slack-notifications"></a>
## Slack-уведомления

<a name="slack-prerequisites"></a>
### Требования

Прежде чем начать отправлять уведомления через Slack, вам необходимо установить HTTP библиотеку Guzzle через Composer:

    composer require guzzlehttp/guzzle

Вам также нужно будет настроить интеграцию ["Входящего Веб-хука" ("Incoming Webhook")](https://api.slack.com/incoming-webhooks) для своей команды Slack. Эта интеграция предоставит URL, который можно использовать при [роутинге Slack-уведомлений](#routing-slack-notifications).

<a name="formatting-slack-notifications"></a>
### Форматирование Slack-уведомлений

Если уведомление поддерживает отправку в виде Slack-сообщения, то вам нужно задать метод `toSlack` классу уведомления. Этот метод получит сущность `$notifiable` и должен возвратить экземпляр `Illuminate\Notifications\Messages\SlackMessage`. Slack-сообщения могут содержать как текст, так и "вложение", которое форматирует дополнительный текст или массив полей. Рассмотрим базовый пример `toSlack` пример:

    /**
     * Получить Slack-представление уведомления.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->content('One of your invoices has been paid!');
    }

В этом примере мы просто отправляем одну строку текста Slack, что создаст сообщение, которое выглядит следующим образом:

<img src="https://laravel.com/assets/img/basic-slack-notification.png">

#### Настройка отправителя и получателя

Можно использовать методы `from` и `to` для настройки отправителя и получателя. Метод `from` принимает в качестве идентификатора имя пользователя и эмоджи, в то время как метод `to` принимает канал или имя пользователя:

    /**
     * Получить Slack-представление уведомления.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Ghost', ':ghost:')
                    ->to('#other')
                    ->content('This will be sent to #other');
    }

Также в качестве логотипа можно использовать изображение вместо эмоджи:

    /**
     * Получить Slack-представление уведомления.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Laravel')
                    ->image('https://laravel.com/favicon.png')
                    ->content('This will display the Laravel logo next to the message');
    }

<a name="slack-attachments"></a>
### Slack-вложения

Вы также можете добавлять "вложения" к сообщениям Slack. Вложения обеспечивают более богатые возможности форматирования, чем простые текстовые сообщения. В этом примере мы отправим уведомление об исключении, которое произошло в приложении, включая ссылку для просмотра более подробной информации об исключении:

    /**
     * Получить Slack-представление уведомления.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was not found.');
                    });
    }

Пример выше сгенерирует Slack-сообщение, которое выглядит следующим образом:

<img src="https://laravel.com/assets/img/basic-slack-attachment.png">

Вложения также позволяют указывать массив данных, которые нужно презентовать пользователю. Эти данные будут представлены в виде таблицы для упрощения чтения:

    /**
     * Получить Slack-представление уведомления.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/invoices/'.$this->invoice->id);

        return (new SlackMessage)
                    ->success()
                    ->content('One of your invoices has been paid!')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Invoice 1322', $url)
                                   ->fields([
                                        'Title' => 'Server Expenses',
                                        'Amount' => '$1,234',
                                        'Via' => 'American Express',
                                        'Was Overdue' => ':-1:',
                                    ]);
                    });
    }

Пример выше создаст Slack-сообщение, которое выглядит так:

<img src="https://laravel.com/assets/img/slack-fields-attachment.png">

#### Markdown содержимое вложения

Если некоторые из полей вашего вложения содержат разметку Markdown, можно использовать метод `markdown`, чтобы сообщить Slack анализировать и отображать заданные поля вложения как форматированный Markdown-текст:

    /**
     * Получить Slack-представление уведомления.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was *not found*.')
                                   ->markdown(['title', 'text']);
                    });
    }

<a name="routing-slack-notifications"></a>
### Роутинг Slack-уведомлений

Чтобы направить Slack-уведомления в подходящее местоположение, задайте метод `routeNotificationForSlack` вашей уведомляемой сущности. Это должно вернуть URL веб-хука, которому нужно доставить данное уведомление. URL веб-хуков можно генерировать добавляя сервис "Incoming Webhook" своей команде Slack:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Направление уведомлений для Slack-канала.
         *
         * @return string
         */
        public function routeNotificationForSlack()
        {
            return $this->slack_webhook_url;
        }
    }

<a name="notification-events"></a>
## События уведомлений

Когда уведомление уже отправлено, системой уведомления выбрасывается событие `Illuminate\Notifications\Events\NotificationSent`. Оно содержит сущность "notifiable" и сам экземпляр уведомления. Вы можете зарегистрировать слушателей для этого события в своем `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Notifications\Events\NotificationSent' => [
            'App\Listeners\LogNotification',
        ],
    ];

> {tip} После регистрирования слушателей в вашем `EventServiceProvider`, используйте Artisan-команду `event:generate` для быстрого генерирования классов слушателей.

В слушателе событий можно получить доступ к свойствам события `notifiable`, `notification` и `channel`, чтобы узнать больше о получателе уведомления или о самом уведомлении:

    /**
     * Обработка события.
     *
     * @param  NotificationSent  $event
     * @return void
     */
    public function handle(NotificationSent $event)
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
    }

<a name="custom-channels"></a>
## Пользовательские каналы

Laravel поставляется с несколькими каналами уведомлений, но вы можете захотеть написать свои собственные драйверы для доставки уведомлений по другим каналам. В Laravel это чрезвычайно просто. Для начала определите класс, который содержит метод `send`. Этот метод должен получать два аргумента: `$notifiable` и `$notification`:

    <?php

    namespace App\Channels;

    use Illuminate\Notifications\Notification;

    class VoiceChannel
    {
        /**
         * Отправка заданного уведомления.
         *
         * @param  mixed  $notifiable
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return void
         */
        public function send($notifiable, Notification $notification)
        {
            $message = $notification->toVoice($notifiable);

            // Отправка уведомления экземпляру $notifiable...
        }
    }

Как только был определен класс вашего канала уведомлений, вы можете просто вернуть название класса из метода `via` любого из ваших уведомлений:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use App\Channels\VoiceChannel;
    use App\Channels\Messages\VoiceMessage;
    use Illuminate\Notifications\Notification;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class InvoicePaid extends Notification
    {
        use Queueable;

        /**
         * Получить каналы уведомления.
         *
         * @param  mixed  $notifiable
         * @return array|string
         */
        public function via($notifiable)
        {
            return [VoiceChannel::class];
        }

        /**
         * Получить голосовое представление уведомления.
         *
         * @param  mixed  $notifiable
         * @return VoiceMessage
         */
        public function toVoice($notifiable)
        {
            // ...
        }
    }
