git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Работа с e-mail 

- [Введение](#introduction)
    - [Требования для драйверов](#driver-prerequisites)
- [Генерация Mailables](#generating-mailables)
- [Написание Mailables](#writing-mailables)
    - [Настройка отправителя](#configuring-the-sender)
    - [Настройка шаблона](#configuring-the-view)
    - [Данные шаблона](#view-data)
    - [Вложения](#attachments)
    - [Встроенные вложения](#inline-attachments)
    - [Настройка сообщения SwiftMailer](#customizing-the-swiftmailer-message)
- [Markdown Mailables](#markdown-mailables)
    - [Генерация Markdown Mailables](#generating-markdown-mailables)
    - [Написание Markdown-сообщений](#writing-markdown-messages)
    - [Настройка компонентов](#customizing-the-components)
- [Отправка почты](#sending-mail)
    - [Очереди отправки](#queueing-mail)
- [Почта и локальная разработка](#mail-and-local-development)
- [События](#events)

<a name="introduction"></a>
## Введение

Laravel предоставляет простой API к популярной библиотеке [SwiftMailer](http://swiftmailer.org) с драйверами для SMTP, Mailgun, SparkPost, Amazon SES, PHP-функций `mail` и `sendmail`, поэтому вы можете быстро приступить к рассылке почты с помощью локального или облачного сервиса на ваш выбор.

<a name="driver-prerequisites"></a>
### Требования для драйверов

Основанные на API драйвера, такие как Mailgun и SparkPost, часто гораздо проще и быстрее, чем SMTP-серверы. Вам следует использовать один из таких драйверов, если это возможно. Для работы таких драйверов необходимо, чтобы в вашем приложении была установлена HTTP-библиотека Guzzle, которую можно установить через менеджер пакетов Composer:

    composer require guzzlehttp/guzzle

#### Драйвер Mailgun

Для использования драйвера Mailgun установите Guzzle и задайте для параметра `driver` в конфиге `config/mail.php` значение `mailgun`. Затем проверьте, что в конфиге `config/services.php` содержатся следующие параметры:

    'mailgun' => [
        'domain' => 'your-mailgun-domain',
        'secret' => 'your-mailgun-key',
    ],

#### Драйвер SparkPost

Для использования драйвера SparkPost установите Guzzle и задайте для параметра `driver` в конфиге`config/mail.php` значение `sparkpost`. Затем проверьте, что в файле `config/services.php` содержатся следующие параметры:

    'sparkpost' => [
        'secret' => 'your-sparkpost-key',
    ],

#### Драйвер SES

Чтобы использовать драйвер Amazon SES, установите Amazon AWS SDK для PHP. Вы можете установить эту библиотеку, добавив следующую строку в раздел `require` файла `composer.json` и выполнив команду `composer update`:

    "aws/aws-sdk-php": "~3.0"

Затем задайте для параметра `driver` в конфиге `config/mail.php` значение `ses` и проверьте, что в файле `config/services.php` содержатся следующие параметры:

    'ses' => [
        'key' => 'your-ses-key',
        'secret' => 'your-ses-secret',
        'region' => 'ses-region',  // e.g. us-east-1
    ],

<a name="generating-mailables"></a>
## Генерация Mailables

В Laravel каждый тип email-сообщений, отправляемых вашим приложением, представлен классом "mailable". Эти классы хранятся в директории `app/Mail`. Не волнуйтесь, если не видите этту директорию в своем приложении, так как она будет сгенерирована когда вы создадите первый подобный класс, используя команду`make:mail`:

    php artisan make:mail OrderShipped

<a name="writing-mailables"></a>
## Написание Mailables

Настройка класса mailable выполняется в методе `build`. В рамках этого метода можно вызвать различные методы, такие как `from`, `subject`, `view` и `attach` для настройки самого электронного письма и его доставки.

<a name="configuring-the-sender"></a>
### Настройка отправителя

#### Использование метода `from`

Сначала давайте рассмотрим настройку отправителя электронных писем. Или, другими словами, кто будет указан в поле отправителя - "from". Есть два способа настройки отправителя. Во-первых, можно использовать метод  `from` внутри вашего метода mailable-класса `build`:

    /**
     * Построение сообщения.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->view('emails.orders.shipped');
    }

#### Использование глобального адреса `from`

Однако, если ваше приложение использует один и тот же адрес "from" во всех своих письмах, будет довольно утомительно вызывать метод `from` в каждом генерируемом классе mailable. Вместо этого можно указать глобальный адрес "from" в конфиге `config/mail.php`. Этот адрес будет использоваться, если больше не указан ни один адрес "from" в классе mailable:

    'from' => ['address' => 'example@example.com', 'name' => 'App Name'],

<a name="configuring-the-view"></a>
### Настройка шаблона

В методе `build` mailable-класса можно использовать метод `view`, чтобы указать какой шаблон следует использовать при визуальном представлении содержимого этого email-сообщения. Как как каждый email обычно использует [шаблон Blade](/docs/{{version}}/blade) для представления своего содержимого, в вашем распоряжении будет вся мощь и удобство движка обработки шаблонов Blade при построении HTML вашего электронного сообщения:

    /**
     * Построение сообщения.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped');
    }

> {tip} Возможно, вы захотите создать директорию `resources/views/emails`, чтобы разместить все свои email-шаблоны; тем не менее, вы можете размещать их где угодно внутри своей директории `resources/views`.

#### Email-сообщения без форматирования

Если вы хотите определить версию своего email без форматирования, то можно использовать метод `text`. Как и метод `view`, метод `text` принимает имя шаблона, который будет использоваться для визуального представления содержимого письма. Вы свободно можете определить и HTML-версию и версию без форматирования своего сообщения:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->text('emails.orders.shipped_plain');
    }

<a name="view-data"></a>
### Данные шаблона

#### Через общедоступные свойства

Как правило, вы захотите передать некоторые данные своему шаблону, которые можно использовать при визуальном отображении электронного сообщения в виде HTML. Существует два способа, благодаря которым можно сделать данные доступными для вашего шаблона. Первый: любое общедоступное свойство, определенное в вашем классе mailable, можно автоматически сделать доступным для шаблона. Поэтому, к примеру, вы можете передать данные своему конструктору класса mailable и изменить данные на общедоступные свойства, определенные в классе:

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * The order instance.
         *
         * @var Order
         */
        public $order;

        /**
         * Создать новый экземпляр сообщения.
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * Построить сообщение.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped');
        }
    }

Как только данные были заменены на общедоступные свойства, они станут автоматически доступными для вашего шаблона - вы сможете получать к ним доступ так же, как и к любым другим данным в ваших шаблонах Blade:

    <div>
        Price: {{ $order->price }}
    </div>

#### Через метод `with`:

Если вы хотите изменить формат данных своего e-mail сообщения, прежде чем они будут отправлены шаблону, вы можете вручную передать данные шаблону через метод `with`. Как правило, вы все еще будете передавать данные через контруктор класса mailable; однако, вы должны задать свойства данных `protected` или `private`, чтобыд данные не были автоматически доступны шаблону. Тогда во время вызова метода `with` нужно передать массив данных, которые вы хотите сделать доступным для шаблона:

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * The order instance.
         *
         * @var Order
         */
        protected $order;

        /**
         * Создать новый экземпляр сообщения.
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->with([
                            'orderName' => $this->order->name,
                            'orderPrice' => $this->order->price,
                        ]);
        }
    }

Как только данные были переданы методу `with`, они автоматически будут доступны в вашем шаблоне - вы сможете получить к ним доступ точно так же, как и к любым другим данным в своих шаблонах Blade:

    <div>
        Price: {{ $orderPrice }}
    </div>

<a name="attachments"></a>
### Вложения

Чтобы добавить вложение к электронному сообщению, используйте метод `attach` в методе mailable-класса `build`. В качестве первого аргумента метод `attach` принимает полный путь к файлу:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file');
        }

Во время присоединения файлов к сообщению, вы также можете указать отображаемое имя и / или MIME-тип, передав `array` в качестве второго аргумента методу `attach`:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file', [
                            'as' => 'name.pdf',
                            'mime' => 'application/pdf',
                        ]);
        }

#### Вложения с сырыми данными

Метод `attachData` можно использовать для присоединения сырой строки байтов в качестве вложения. Например, вы можете использовать этот метод, если сгенерировали PDF в памяти и хотите присоединить его к электронному письму без записи на диск. Метод `attachData` принимает байты сырых данных в качестве первого аргумента, имя файла - в качестве второго аргумента, а массив опций - в качестве третьего:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attachData($this->pdf, 'name.pdf', [
                            'mime' => 'application/pdf',
                        ]);
        }

<a name="inline-attachments"></a>
### Встроенные вложения

Обычно добавление встроенных вложений — утомительное занятие, однако Laravel делает его проще, позволяя вам добавлять изображения и получать соответствующие CID. Для вставки встроенного изображения используйте метод `embed` на переменной `$message` в вашем email-шаблоне. Laravel автоматически делает переменную `$message` доступной для всех ваших email-шаблонов, так что вам не нужно волноваться о том, чтобы передать ее вручную:

    <body>
        Вот изображение:

        <img src="{{ $message->embed($pathToFile) }}">
    </body>

> {note} Переменная `$message` недоступна в markdown-сообщениях.

#### Встроенные вложения с сырыми данными

Если у вас уже есть строка с сырыми данными, которую вы хотите встроить в шаблон электронного сообщения, можно использовать метод `embedData` на переменной `$message`:

    <body>
        Вот изображение из сырых данных:

        <img src="{{ $message->embedData($data, $name) }}">
    </body>

<a name="customizing-the-swiftmailer-message"></a>
### Настройка сообщения SwiftMailer

Метод `withSwiftMessage` базового класса `Mailable` позволяет вам зарегистрировать анонимную функцию, которая будет вызываться экземпляром сообщения SwiftMailer перед отправкой сообщения. Это дает вам возможность кастомизировать сообщение перед тем как оно будет доставлено:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            $this->view('emails.orders.shipped');

            $this->withSwiftMessage(function ($message) {
                $message->getHeaders()
                        ->addTextHeader('Custom-Header', 'HeaderValue');
            });
        }

<a name="markdown-mailables"></a>
## Markdown Mailables

Mailable-сообщения в формате Markdown позволяют вам воспользоваться предварительно собранными шаблонами и компонентами почтовых уведомлений в ваших mailables. Так как сообщения написаны в формате Markdown, Laravel способен отображать красивые, отзывчивые HTML-шаблоны сообщений, в то же время генерируя идентичную копию без форматирования (только текст).

<a name="generating-markdown-mailables"></a>
### Генерация Markdown Mailables

Чтобы сгенерировать mailable с соответствующим Markdown можно использовать опцию `--markdown` Artisan-команды `make:mail`:

    php artisan make:mail OrderShipped --markdown=emails.orders.shipped

Тогда при настройке mailable в своем методе `build`, вызовите метод `markdown` вместо метода `view`. Метод `markdown` принимает имя Markdown-шаблона в качестве необязательного массива данных, который затем делает доступным для шаблона:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->markdown('emails.orders.shipped');
    }

<a name="writing-markdown-messages"></a>
### Написание Markdown-сообщений

Markdown mailable-сообщения используют комбинацию Blade-компонентов и синтаксиса Markdown, что позволяет вам довольно просто конструировать почтовые сообщения, используя заранее созданные компоненты Laravel:

    @component('mail::message')
    # Заказ отправлен

    Ваш заказ был отправлен!

    @component('mail::button', ['url' => $url])
    Просмотреть заказ
    @endcomponent

    Спасибо,<br>
    {{ config('app.name') }}
    @endcomponent

> {tip}Не используйте чрезмерное количество отступов во время написания Markdown-сообщений. Markdown парсеры выполнят содержимое с отступом в виде блоков кода.

#### Компонент Button (кнопка)

Компонент кнопки отображает отцентрованную ссылку-кнопку. Этот компонент принимает два аргумента: `url` и необязательный `color`. Поддерживаются цвета: `blue` (синий), `green` (зеленый) и `red` (красный). Вы можете добавить сколько угодно компонентов-кнопок в свое сообщение:

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    Просмотреть заказ
    @endcomponent

#### Компонент Panel (область)

Компонент panel отображает заданный блок текста в области с цветом фона, слегка отличающимся от заднего фона самого сообщения. Это позволяет привлечь внимание к данной области текста:

    @component('mail::panel')
    This is the panel content.
    @endcomponent

#### Компонента Table (таблица)

Компонент table позволяет трансформировать Markdown-таблицу в HTML-таблицу. Этот компонент принимает  Markdown-таблицу в качестве содержимого. Поддерживается выравнивание столбцов с использованием синтаксиса выравнивания в Markdown таблицах по умолчанию:

    @component('mail::table')
    | Laravel       | Table         | Example  |
    | ------------- |:-------------:| --------:|
    | Col 2 is      | Centered      | $10      |
    | Col 3 is      | Right-Aligned | $20      |
    @endcomponent

<a name="customizing-the-components"></a>
### Настройка компонентов

Вы можете экспортировать все Markdown-компоненты почты в собственное приложение, чтобы настроить их. Чтобы экспортировать: используйте Artisan-команду `vendor:publish`, чтобы опубликовать ассет-тэг `laravel-mail`:

    php artisan vendor:publish --tag=laravel-mail

Эта команда опубликует Markdown-компоненты почты в директорию `resources/views/vendor/mail`. Директория `mail` будет содержать директорию `html` и `markdown`, каждая из которых содержит соответсвующие представления каждого доступного компонента. Вы можете свободно настраивать эти компоненты по собственному усмотрению.

#### Настройка CSS

После экспорта компонентов в директории `resources/views/vendor/mail/html/themes` будет содержаться файл `default.css`. Вы можете настроить CSS в этом файле, и ваши стили будут автоматически приведены в соответствие с HTML-представлениями ваших Markdown-сообщений.

> {tip} Если вы хотите собрать полностью новую тему для Markdown-кмпонентов, просто напишите новый CSS-файл в директории `html/themes` и измените параметр `theme` в конфиге `mail`.

<a name="sending-mail"></a>
## Отправка почты

Используйте метод `to` [фасада](/docs/{{version}}/facades) `Mail`, чтобы отправить сообщение. Метод `to` принимает адрес email, экземпляр пользователя или коллекцию пользователей. Если вы передаете объект или коллекцию объектов, почтовая программа будет автоматически использовать их свойства `email` и `name` при настройке получателей электронного сообщения, поэтому убедитесь, что у ваших объектов есть эти атрибуты. Как только вы указали получателей, вы можете передать экземпляр своего класса mailable методу `send`:

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Mail\OrderShipped;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Mail;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * Отправить заданный заказ.
         *
         * @param  Request  $request
         * @param  int  $orderId
         * @return Response
         */
        public function ship(Request $request, $orderId)
        {
            $order = Order::findOrFail($orderId);

            // Отправить заказ...

            Mail::to($request->user())->send(new OrderShipped($order));
        }
    }

Конечно, вы не ограничены исключительно указанием получателей в поле "to" при отправке сообщения. Также можно указать и получателей "to", "cc" и "bcc" - всех в одном связанном вызове метода:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));

<a name="queueing-mail"></a>
### Очереди отправки

#### Помещение сообщения в очередь отправки

Из-за того, что отправка сообщений может сильно повлиять на время обработки запроса, многие разработчики помещают их в очередь на фоновую отправку. Laravel позволяет легко делать это, используя [единое API очередей](/docs/{{version}}/queues). Для помещения сообщения в очередь просто используйте метод `queue` фасада `Mail` после указания получателей сообщения:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));

Этот метод автоматически позаботится о помещении в очередь задачи для фоновой отправки почтового сообщения. Конечно, вам нужно будет [настроить механизм очередей](/docs/{{version}}/queues) перед использованием данной возможности.

#### Задержка отправки сообщения

Вы можете задержать отправку сообщения методом `later`. В качестве своего первого аргумента метод `later` принимает экземпляр `DateTime`, указывая когда следует отправить сообщение:

    $when = Carbon\Carbon::now()->addMinutes(10);

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later($when, new OrderShipped($order));

#### Помещение сообщения в определённую очередь

Так как все mailable-классы генерируемые с использованием команды `make:mail` используют трейт `Illuminate\Bus\Queueable`, вы можете вызвать методы `onQueue` и `onConnection` в любом экземпляре класса mailable, что позволит указать имя очереди и подключение для сообщения:

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);

#### Помещение в очередь по умолчанию

Если у вас есть mailable-классы, которые всегда должны быть в очереди, вы можете реализовать на классе контракт `ShouldQueue`. Теперь даже если вы вызовете метод `send` при отправке почты, mailable все еще будет находится в очереди, так как он реализует контракт:

    use Illuminate\Contracts\Queue\ShouldQueue;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        //
    }

<a name="mail-and-local-development"></a>
## Почта и локальная разработка

При разработке приложения обычно предпочтительно отключить доставку отправляемых сообщений. В Laravel есть несколько способов "отключить" реальную отправку почтовых сообщений

#### Драйвер Log

Вместо отправки ваших электронных сообщений, драйвер `log` будет записывать все email-сообщения в ваши логи для анализа. Это полезно в первую очередь для быстрой, локальной отладки и проверки данных. Подробнее о настройке различных окружений для приложения читайте в [документации по настройке](/docs/{{version}}/configuration#environment-configuration).

#### Универсальный получатель

Другой вариант — задать универсального получателя для всех сообщений от фреймворка. при этом все сообщения, генерируемые вашим приложением, будут отсылаться на заданный адрес, вместо адреса, указанного при отправке сообщения. Это можно сделать с помощью параметра `to` в конфиге `config/mail.php`:

    'to' => [
        'address' => 'example@example.com',
        'name' => 'Example'
    ],

#### Mailtrap

И, наконец, вы можете использовать сервис [Mailtrap](https://mailtrap.io) и драйвер `smtp` для отправки ваших почтовых сообщений на фиктивный почтовый ящик, где вы сможете посмотреть их при помощи настоящего почтового клиента. Преимущество этого вариант в том, что вы можете проверить то, как в итоге будут выглядеть ваши почтовые сообщения, при помощи средства для просмотра сообщений Mailtrap.

<a name="events"></a>
## События

Laravel генерирует событие непосредственно перед отправкой почтовых сообщений. Учтите, это событие возникает при *отправке*, сообщения, а не при помещении его в очередь. Вы можете зарегистрировать слушатель события в своём `EventServiceProvider`:

    /**
     * Отображения слушателя событий для приложения.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Mail\Events\MessageSending' => [
            'App\Listeners\LogSentMessage',
        ],
    ];
