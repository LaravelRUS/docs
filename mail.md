---
git: 46c2634ef5a4f15427c94a3157b626cf5bd3937f
---

# Отправка электронной почты


<a name="introduction"></a>
## Введение

Отправка электронной почты не должна быть сложной.
Laravel предлагает чистый и простой почтовый API на базе популярного компонента [Symfony Mailer](https://symfony.com/doc/6.2/mailer.html).
Laravel и Symfony Mailer обеспечены драйверами для отправки электронной почты через SMTP, Mailgun, Postmark, Amazon SES и `sendmail`, что позволяет быстро начать отправку почты через локальный или облачный сервис по вашему выбору.

<a name="configuration"></a>
### Конфигурирование

Почтовые службы Laravel могут быть настроены через конфигурационный файл `config/mail.php` вашего приложения. Каждая почтовая программа, настроенная в этом файле, может иметь свою собственную уникальную конфигурацию и даже свой собственный уникальный «транспорт», что позволяет вашему приложению использовать различные почтовые службы для отправки определенных сообщений электронной почты. Например, ваше приложение может использовать Postmark для отправки транзакционных писем, а Amazon SES – для массовых рассылок.

В конфигурационном файле `config/mail.php` вы найдете массив `mailers`. Этот массив содержит образец записи конфигурации для каждого из основных почтовых драйверов / транспортов, поддерживаемых Laravel, в то время как значение конфигурации `default` определяет, какая почтовая программа будет использоваться по умолчанию, когда ваше приложение должно отправить сообщение электронной почты.

<a name="driver-prerequisites"></a>
### Требования к драйверу и транспорту

Драйверы на основе API, такие, как Mailgun и Postmark, часто проще в использовании и быстрее, чем отправка почты через SMTP-серверы. По возможности мы рекомендуем использовать один из этих драйверов.

<a name="mailgun-driver"></a>
#### Драйвер Mailgun

Для использования драйвера Mailgun установите пакет Symfony's Mailgun Mailer с помощью Composer:

```shell
composer require symfony/mailgun-mailer symfony/http-client
```

Затем установите опцию `default` в конфигурационном файле `config/mail.php` вашего приложения в значение `mailgun`. После настройки основного почтового отправителя вашего приложения убедитесь, что конфигурационный файл `config/services.php` содержит следующие параметры:

    'mailgun' => [
        'transport' => 'mailgun',
        'domain' => env('MAILGUN_DOMAIN'),
        'secret' => env('MAILGUN_SECRET'),
    ],

Если вы не используете [регион Mailgun](https://documentation.mailgun.com/en/latest/api-intro.html#mailgun-regions) США, то вы можете определить конечную точку своего региона в конфигурации файла `services`:

    'mailgun' => [
        'domain' => env('MAILGUN_DOMAIN'),
        'secret' => env('MAILGUN_SECRET'),
        'endpoint' => env('MAILGUN_ENDPOINT', 'api.eu.mailgun.net'),
    ],

<a name="postmark-driver"></a>
#### Драйвер Postmark

Чтобы использовать драйвер Postmark, установите пакет Symfony's Postmark Mailer через Composer:

```shell
composer require symfony/postmark-mailer symfony/http-client
```

Затем установите опцию `default` в конфигурационном файле `config/mail.php` вашего приложения в значение `postmark`. После настройки основного почтового отправителя вашего приложения, убедитесь, что конфигурационный файл `config/services.php` содержит следующие опции:

    'postmark' => [
        'token' => env('POSTMARK_TOKEN'),
    ],

Если вы хотите указать поток сообщений Postmark, который должен использоваться данной почтовой программой, вы можете добавить параметр конфигурации `message_stream_id` в массив конфигурации почтовой программы. Этот массив конфигурации можно найти в файле конфигурации вашего приложения `config/mail.php`:

    'postmark' => [
        'transport' => 'postmark',
        'message_stream_id' => env('POSTMARK_MESSAGE_STREAM_ID'),
    ],

Таким образом, вы также можете настроить несколько почтовых программ Postmark с разными потоками сообщений.

<a name="ses-driver"></a>
#### Драйвер SES

Чтобы использовать драйвер Amazon SES, сначала необходимо установить Amazon AWS SDK для PHP. Вы можете установить эту библиотеку через менеджер пакетов Composer:

```shell
composer require aws/aws-sdk-php
```

Затем установите для параметра `default` в вашем файле конфигурации `config/mail.php` значение `ses` и убедитесь, что конфигурационный файл `config/services.php` содержит следующие параметры:

    'ses' => [
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    ],

Чтобы использовать AWS [временные учетные данные](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html) через токен сеанса, вы можете добавить ключ `token` в конфигурацию SES вашего приложения:

    'ses' => [
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
        'token' => env('AWS_SESSION_TOKEN'),
    ],

Если вы хотите определить [дополнительные параметры](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-sesv2-2019-09-27.html#sendemail), которые Laravel должен передать методу `SendEmail` AWS SDK при отправке сообщения электронной почты, вы можете определить массив `options` в конфигурации `ses`:

    'ses' => [
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
        'options' => [
            'ConfigurationSetName' => 'MyConfigurationSet',
            'EmailTags' => [
                ['Name' => 'foo', 'Value' => 'bar'],
            ],
        ],
    ],

<a name="mailersend-driver"></a>
#### Драйвер MailerSend

[MailerSend](https://www.mailersend.com/), сервис для отправки транзакционных электронных писем и SMS-сообщений, поддерживает свой собственный драйвер для Laravel, основанный на их API. Пакет, содержащий этот драйвер, можно установить с помощью менеджера пакетов Composer:

```shell
composer require mailersend/laravel-driver
```

После установки пакета добавьте переменную окружения `MAILERSEND_API_KEY` в файл `.env` вашего приложения. Кроме того, переменная окружения `MAIL_MAILER` должна быть определена как `mailersend`:

```shell
MAIL_MAILER=mailersend
MAIL_FROM_ADDRESS=app@yourdomain.com
MAIL_FROM_NAME="Имя приложения"

MAILERSEND_API_KEY=ваш-ключ-api
```

Для получения дополнительной информации о MailerSend, включая инструкции по использованию хостинга шаблонов, обратитесь к [документации по драйверу MailerSend](https://github.com/mailersend/mailersend-laravel-driver#usage).

<a name="failover-configuration"></a>
### Конфигурация аварийного переключения

Иногда внешняя служба, которую вы настроили для отправки почты вашего приложения, может не работать. В этих случаях может быть полезно определить одну или несколько резервных конфигураций доставки почты, которые будут использоваться в случае, если ваш основной драйвер доставки не работает.

Для этого вы должны определить почтовую программу в файле конфигурации вашего приложения `mail`, который использует транспорт`failover`. Массив конфигурации для почтовой программы `failover` вашего приложения должен содержать массив `mailers`, который определяет очередность, в которой почтовые программы должны быть выбраны для доставки:

    'mailers' => [
        'failover' => [
            'transport' => 'failover',
            'mailers' => [
                'postmark',
                'mailgun',
                'sendmail',
            ],
        ],

        // ...
    ],

После того как ваш почтовый агент аварийного переключения был определен, вы должны установить его как почтовую программу по умолчанию, используемую вашим приложением, указав ее имя как значение конфигурационного ключа `default` в файле конфигурации вашего приложения `mail`:

    'default' => env('MAIL_MAILER', 'failover'),

<a name="round-robin-configuration"></a>
### Конфигурация Round Robin

Транспорт `roundrobin` позволяет распределить вашу почтовую нагрузку между несколькими почтовыми клиентами. Чтобы начать, определите почтовый клиент в файле конфигурации `mail` вашего приложения, который использует транспорт `roundrobin`. Массив конфигурации почтового клиента вашего приложения `roundrobin` должен содержать массив `mailers`, который указывает, какие настроенные почтовые клиенты должны быть использованы для доставки:

    'mailers' => [
        'roundrobin' => [
            'transport' => 'roundrobin',
            'mailers' => [
                'ses',
                'postmark',
            ],
        ],

        // ...
    ],

После того как ваш почтовый клиент round robin был определен, вы должны установить этот клиент почты в качестве клиента по умолчанию, используемого вашим приложением, указав его имя в качестве значения ключа `default` в конфигурационном файле `mail` вашего приложения:

    'default' => env('MAIL_MAILER', 'roundrobin'),

Транспорт round robin выбирает случайный почтовый клиент из списка настроенных почтовых клиентов, а затем переключается на следующий доступный почтовый клиент для каждого последующего электронного письма. В отличие от транспорта `failover`, который помогает достичь *[высокой доступности](https://en.wikipedia.org/wiki/High_availability)*, транспорт `roundrobin` обеспечивает *[балансировку нагрузки](https://en.wikipedia.org/wiki/Load_balancing_(computing))*.

<a name="generating-mailables"></a>
## Генерация отправлений

При создании приложений Laravel каждый тип электронной почты, отправляемой вашим приложением, представляется экземпляром класса `Illuminate\Mail\Mailable`. Эти классы хранятся в каталоге `app/Mail`. Не беспокойтесь, если вы не видите этот каталог в своем приложении, поскольку он будет сгенерирован для вас, когда вы создадите свой первый почтовый класс с помощью команды `make:mail` [Artisan](artisan):

```shell
php artisan make:mail OrderShipped
```

<a name="writing-mailables"></a>
## Написание отправлений

После того как вы создали класс для отправки электронной почты, откройте его, чтобы мы могли рассмотреть его содержание. Конфигурация класса для отправки электронной почты выполняется в нескольких методах, включая методы `envelope`, `content` и `attachments`.

Метод `envelope` возвращает объект `Illuminate\Mail\Mailables\Envelope`, который определяет тему сообщения и, иногда, получателей сообщения. Метод `content` возвращает объект `Illuminate\Mail\Mailables\Content`, который определяет [шаблон Blade](/docs/{{version}}/blade), который будет использоваться для генерации содержания сообщения.

<a name="configuring-the-sender"></a>
### Конфигурирование отправителя

<a name="using-the-from-method"></a>
#### Использование метода `Envelope`

Давайте сначала рассмотрим настройку отправителя электронной почты, то есть того, от кого будет отправлено письмо. 
Существует два способа настройки отправителя.
Во-первых, вы можете указать адрес отправителя в методе `envelope` вашего сообщения:

    use Illuminate\Mail\Mailables\Address;
    use Illuminate\Mail\Mailables\Envelope;

    /**
     * Get the message envelope.
     */
    public function envelope(): Envelope
    {
        return new Envelope(
            from: new Address('jeffrey@example.com', 'Jeffrey Way'),
            subject: 'Order Shipped',
        );
    }

Если необходимо, вы также можете указать адрес для ответа указав `replyTo`:

```php
return new Envelope(
    from: new Address('jeffrey@example.com', 'Jeffrey Way'),
    replyTo: [
        new Address('taylor@example.com', 'Taylor Otwell'),
    ],
    subject: 'Заказ отправлен',
);
```

<a name="using-a-global-from-address"></a>
#### Использование глобального адреса `from`

Однако, если ваше приложение использует один и тот же адрес `from` для всех своих электронных писем, вызов метода `from` в каждом создаваемом вами классе рассылки может стать громоздким. Вместо этого вы можете указать глобальный адрес отправителя в файле конфигурации `config/mail.php`. Этот адрес будет использоваться, если в почтовом классе не указан другой адрес в методе `from`:

Однако, если ваше приложение использует один и тот же адрес "from" (От) для всех своих электронных писем, может быть неудобно добавлять его в каждый создаваемый класс для отправки электронной почты.
Вместо этого вы можете указать глобальный адрес "from" (От) в файле конфигурации `config/mail.php`.
Этот адрес будет использоваться, если в классе для отправки электронной почты не указан другой адрес "from":

    'from' => [
        'address' => env('MAIL_FROM_ADDRESS', 'hello@example.com'),
        'name' => env('MAIL_FROM_NAME', 'Example'),
    ],

Кроме того, вы можете определить глобальный адрес `reply_to` в конфигурационном файле `config/mail.php`:

    'reply_to' => ['address' => 'example@example.com', 'name' => 'App Name'],

<a name="configuring-the-view"></a>
### Конфигурирование шаблона

Внутри метода `build` почтового класса вы можете использовать метод `view`, чтобы указать, какой шаблон следует использовать при отображении содержимого электронного письма. Поскольку каждое электронное письмо для визуализации своего содержимого обычно использует [шаблон Blade](/docs/{{version}}/blade), вы получаете всю мощь и удобство механизма шаблонов Blade при создании HTML-кода электронной почты:

В методе `content` класса для отправки электронной почты вы можете определить метод `view`, то есть, какой шаблон должен использоваться при отображении содержимого электронного письма.
Поскольку каждое электронное письмо обычно использует [шаблон Blade](/docs/{{version}}/blade) для отображения своего содержания, у вас есть вся мощь и удобство шаблонизатора Blade при создании HTML-содержимого письма:

    /**
     * Get the message content definition.
     *
     * @return $this
     */
    public function content(): Content
    {
        return new Content(
            view: 'mail.orders.shipped',
        );
    }

> [!NOTE]  
> Вы можете создать каталог `resources/views/emails` для размещения всех ваших шаблонов электронной почты; однако, вы можете размещать их где угодно в каталоге `resources/views`.

<a name="plain-text-emails"></a>
#### Письма с обычным текстом

Если вы хотите указать версию письма для plain-text (обычный текст), вы можете указать его шаблон при определении `Content`. Как и параметр `view`, параметр `text` должен содержать имя шаблона, который будет использоваться для отображения содержимого в текстовом формате. Вы можете определить как версию в HTML, так и версию в plain-text для вашего сообщения:

    /**
     * Get the message content definition.
     */
    public function content(): Content
    {
       return new Content(
            view: 'mail.orders.shipped',
            text: 'mail.orders.shipped-text'
        );
    }

Для большей ясности параметр `html` может быть использован в качестве псевдонима параметра `view`:

```php
return new Content(
    html: 'mail.orders.shipped',
    text: 'mail.orders.shipped-text'
);
```

<a name="view-data"></a>
### Данные шаблона

<a name="via-public-properties"></a>
#### Передача данных шаблону через публичные свойства

Как правило, вам нужно передать в шаблон некоторые данные, которые можно использовать при отображении HTML-кода электронного письма. Есть два способа сделать данные доступными для вашего шаблона. Во-первых, любое публичное свойство, определенное в вашем почтовом классе, будет автоматически доступно для шаблона. Так, например, можно передать данные в конструктор почтового класса и присвоить этим данные публичным свойствам, определенным в классе:

    <?php

    namespace App\Mail;

    use App\Models\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Mail\Mailables\Content;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * Создать экземпляр нового сообщения.
         */
        public function __construct(
            public Order $order,
        ) {}

        /**
         * Получить содержимое сообщения
         */
        public function content(): Content
        {
           return new Content(
                view: 'mail.orders.shipped',
            );
        }
    }

После того как данные были заданы как публичные свойства, они будут автоматически доступны в вашем шаблоне, поэтому вы можете получить к ним доступ так же, как и к любым другим данным в ваших шаблонах Blade:

    <div>
        Price: {{ $order->price }}
    </div>

<a name="via-the-with-parameter"></a>
#### Передача данных шаблону через параметр `with`

Если вы хотите настроить формат данных вашего электронного письма перед их отправкой в шаблон, то вы можете вручную передать свои данные в шаблон с помощью параметра `with`.
Как правило, вы по-прежнему будете передавать данные через конструктор почтового класса; однако, вы должны установить для этих данных свойства `protected` или `private`, чтобы данные не были автоматически доступны для шаблона:


    <?php

    namespace App\Mail;

    use App\Models\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Mail\Mailables\Content;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * Создать экземпляр нового сообщения.
         */
        public function __construct(
            protected Order $order,
        ) {}

        /**
         * Получить содержимое сообщения
         */
        public function content(): Content
        {
            return new Content(
                view: 'mail.orders.shipped',
                with: [
                    'orderName' => $this->order->name,
                    'orderPrice' => $this->order->price,
                ],
            );
        }
    }

После того как данные были переданы методу `with`, они автоматически станут доступны в вашем шаблоне, поэтому вы можете получить к ним доступ так же, как и к любым другим данным в ваших шаблонах Blade:

    <div>
        Price: {{ $orderPrice }}
    </div>

<a name="attachments"></a>
### Вложения

Для добавления вложений к электронному письму, вы добавляете их в массив, который возвращает метод `attachments` сообщения. Сначала вы можете добавить вложение, указав путь к файлу с использованием метода `fromPath` класса `Attachment`:


    use Illuminate\Mail\Mailables\Attachment;
    
    /**
     * Get the attachments for the message.
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [
            Attachment::fromPath('/path/to/file'),
        ];
    }

При прикреплении файлов к сообщению вы также можете указать отображаемое имя и/или MIME-тип, используя методы `as` и `withMime`:

    /**
     * Get the attachments for the message.
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [
            Attachment::fromPath('/path/to/file')
                    ->as('name.pdf')
                    ->withMime('application/pdf'),
        ];
    }

<a name="attaching-files-from-disk"></a>
#### Прикрепление файлов с диска

Если вы сохранили файл на одном из [дисков файлового хранилища](/docs/{{version}}/filesystem), то вы можете прикрепить его к электронному письму с помощью метода `fromStorage`:

    /**
     * Get the attachments for the message.
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [
            Attachment::fromStorage('/path/to/file'),
        ];
    }

Конечно, вы также можете указать имя и MIME-тип вложения:

    /**
     * Get the attachments for the message.
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [
            Attachment::fromStorage('/path/to/file')
                    ->as('name.pdf')
                    ->withMime('application/pdf'),
        ];
    }

Метод `fromStorageDisk` используется, если вам нужно указать диск хранения, отличный от вашего диска по умолчанию:

    /**
     * Get the attachments for the message.
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [
            Attachment::fromStorageDisk('s3', '/path/to/file')
                    ->as('name.pdf')
                    ->withMime('application/pdf'),
        ];
    }

<a name="raw-data-attachments"></a>
#### Вложения необработанных данных

Метод `fromData` используется для присоединения сырой строки байтов в качестве вложения. Например, вы можете использовать этот метод, если вы сгенерировали PDF-файл в памяти и хотите прикрепить его к электронному письму, не записывая его на диск. Метод `fromData` принимает замыкание, которое разрешает сырые байты данных, а также имя, которое следует присвоить вложению:

    /**
     * Get the attachments for the message.
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [
            Attachment::fromData(fn () => $this->pdf, 'Report.pdf')
                    ->withMime('application/pdf'),
        ];
    }

<a name="inline-attachments"></a>
### Встраиваемые вложения

Встраивание изображений в ваши электронные письма, как правило, обременительно; однако Laravel предлагает удобный способ прикреплять изображения к вашим письмам. Чтобы встроить изображение, используйте метод `embed` для переменной `$message` в вашем шаблоне электронной почты. Laravel автоматически делает переменную `$message` доступной для всех ваших шаблонов электронной почты, поэтому вам не нужно беспокоиться о ее передаче вручную:

```html
<body>
    Here is an image:

    <img src="{{ $message->embed($pathToImage) }}">
</body>
```

> [!WARNING]  
> Переменная `$message` недоступна в шаблонах текстовых сообщений, так как в текстовых сообщениях не используются встроенные вложения.

<a name="embedding-raw-data-attachments"></a>
#### Встраиваемые вложения необработанных данных

Если у вас уже есть строка необработанных данных изображения, которую вы хотите встроить в шаблон электронной почты, то вы можете вызвать метод `embedData` для переменной `$message`. При вызове метода `embedData` вам необходимо указать имя файла, которое должно быть присвоено встраиваемому изображению:

```html
<body>
    Here is an image from raw data:

    <img src="{{ $message->embedData($data, 'example-image.jpg') }}">
</body>
```

<a name="attachable-objects"></a>
### Объекты, которые можно прикреплять

В большинстве случаев прикрепление файлов к сообщениям с указанием путей является достаточным, но во многих случаях объекты, которые можно прикреплять, уже представлены классами в вашем приложении.
Например, если ваше приложение прикрепляет фотографию к сообщению, то в вашем приложении может существовать модель `Photo`, которая представляет эту фотографию.
В таком случае было бы удобно просто передать модель `Photo` методу `attach`.
Объекты, которые можно прикреплять, позволяют вам сделать именно это.

Для начала реализуйте интерфейс `Illuminate\Contracts\Mail\Attachable` для объекта, который будет являться прикрепляемым к сообщениям. Этот интерфейс предписывает, что ваш класс должен определить метод `toMailAttachment`, который возвращает экземпляр `Illuminate\Mail\Attachment`:

    <?php

    namespace App\Models;

    use Illuminate\Contracts\Mail\Attachable;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Mail\Attachment;

    class Photo extends Model implements Attachable
    {
        /**
         * Get the attachable representation of the model.
         */
        public function toMailAttachment(): Attachment
        {
            return Attachment::fromPath('/path/to/file');
        }
    }

После того как вы определите свой объект, который можно прикреплять, вы можете вернуть экземпляр этого объекта из метода `attachments`, когда создаете сообщение электронной почты:


    /**
     * Get the attachments for the message.
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [$this->photo];
    }

Конечно, данные вложения могут храниться на удаленном сервисе файлового хранения, таком как Amazon S3. Поэтому Laravel также позволяет создавать экземпляры вложений на основе данных, хранящихся на одном из дисков файловой системы вашего приложения:

```php
// Создание вложения из файла на вашем основном диске...
return Attachment::fromStorage($this->path);

// Создание вложения из файла на конкретном диске...
return Attachment::fromStorageDisk('backblaze', $this->path);
```

Кроме того, вы можете создавать экземпляры вложений на основе данных, хранящихся в памяти. Для этого передайте замыкание методу `fromData`. Замыкание должно возвращать сырые данные, представляющие вложение:

```php
return Attachment::fromData(fn () => $this->content, 'Имя фотографии');
```

Laravel также предоставляет дополнительные методы, которые вы можете использовать для настройки ваших вложений. Например, вы можете использовать методы `as` и `withMime` для настройки имени файла и MIME-типа:

```php
return Attachment::fromPath('/путь/к/файлу')
    ->as('Имя фотографии')
    ->withMime('image/jpeg');
```

<a name="headers"></a>
### Заголовки

Иногда вам может потребоваться прикрепить дополнительные заголовки к исходящему сообщению. Например, вам может потребоваться установить пользовательский `Message-Id` или другие произвольные текстовые заголовки.

Для этого определите метод `headers` в вашем классе для отправки электронной почты. Метод `headers` должен возвращать экземпляр класса `Illuminate\Mail\Mailables\Headers`. Этот класс принимает параметры `messageId`, `references` и `text`. Конечно, вы можете предоставить только те параметры, которые вам нужны для вашего конкретного сообщения:

```php
use Illuminate\Mail\Mailables\Headers;

/**
 * Получить заголовки сообщения.
 */
public function headers(): Headers
{
    return new Headers(
        messageId: 'custom-message-id@example.com',
        references: ['previous-message@example.com'],
        text: [
            'X-Custom-Header' => 'Custom Value',
        ],
    );
}
```

<a name="tags-and-metadata"></a>
Некоторые сторонние провайдеры электронной почты, такие как Mailgun и Postmark, поддерживают "теги" и "метаданные" сообщений, которые можно использовать для группировки и отслеживания отправленных вашим приложением электронных писем. Вы можете добавить теги и метаданные к сообщению электронной почты через ваше определение `Envelope`:

```php
use Illuminate\Mail\Mailables\Envelope;

/**
 * Получите конверт сообщения.
 *
 * @return \Illuminate\Mail\Mailables\Envelope
 */
public function envelope(): Envelope
{
    return new Envelope(
        subject: 'Заказ отправлен',
        tags: ['shipment'],
        metadata: [
            'order_id' => $this->order->id,
        ],
    );
}
```

Если ваше приложение использует драйвер Mailgun, вы можете проконсультироваться с документацией Mailgun для получения дополнительной информации о [тегах](https://documentation.mailgun.com/en/latest/user_manual.html#tagging-1) и [метаданных](https://documentation.mailgun.com/en/latest/user_manual.html#attaching-data-to-messages). Аналогично, вы можете проконсультироваться с документацией Postmark для получения дополнительной информации о поддержке [тегов](https://postmarkapp.com/blog/tags-support-for-smtp) и [метаданных](https://postmarkapp.com/support/article/1125-custom-metadata-faq).

Если ваше приложение использует Amazon SES для отправки электронных писем, вы должны использовать метод `metadata` для добавления [тегов SES](https://docs.aws.amazon.com/ses/latest/APIReference/API_MessageTag.html) к сообщению.

### Настройка Symfony Message


<a name="customizing-the-symfony-message"></a>
### Настройка Symfony Message

Функциональность почты Laravel основана на Symfony Mailer.
Laravel позволяет вам зарегистрировать пользовательские обратные вызовы (замыкания), которые будут вызываться с экземпляром Symfony Message перед отправкой сообщения.
Это дает вам возможность глубоко настраивать сообщение перед его отправкой.
Для этого определите параметр `using` в вашем определении `Envelope`:

```php
use Illuminate\Mail\Mailables\Envelope;
use Symfony\Component\Mime\Email;

/**
 * Получите конверт сообщения.
 */
public function envelope(): Envelope
{
    return new Envelope(
        subject: 'Заказ отправлен',
        using: [
            function (Email $message) {
                // ...
            },
        ]
    );
}
```

<a name="markdown-mailables"></a>
## Отправления с разметкой Markdown

Почтовые сообщения с разметкой Markdown позволяют вам воспользоваться преимуществами предварительно созданных шаблонов и компонентов [почтовых уведомлений](notifications#mail-notifications) в ваших почтовых рассылках. Поскольку сообщения написаны на Markdown, Laravel может отображать красивые, отзывчивые HTML-шаблоны для сообщений, а также автоматически генерировать их аналоги в виде простого текста.

<a name="generating-markdown-mailables"></a>
### Генерация отправлений с разметкой Markdown

Чтобы сгенерировать почтовый класс с соответствующим шаблоном Markdown, вы можете использовать параметр `--markdown` в команде `make:mail` Artisan:

```shell
php artisan make:mail OrderShipped --markdown=mail.orders.shipped
```

Затем, при настройке определения `Content` внутри его метода `content`, используйте параметр `markdown` вместо параметра `view`:
    
    /**
     * Get the message content definition.
     */
    public function content(): Content
       return new Content(
            markdown: 'mail.orders.shipped',
            with: [
                'url' => $this->orderUrl,
            ],
        );
    }

<a name="writing-markdown-messages"></a>
### Написание сообщений с разметкой Markdown

Почтовые сообщения Markdown используют комбинацию компонентов Blade и синтаксиса Markdown, которые позволяют легко создавать почтовые сообщения, используя предварительно созданные компоненты пользовательского интерфейса электронной почты Laravel:

```blade
<x-mail::message>
# Order Shipped
Your order has been shipped!
<x-mail::button :url="$url">
View Order
</x-mail::button>
Thanks,<br>
{{ config('app.name') }}
</x-mail::message>
```

> [!NOTE]  
> Не используйте лишние отступы при написании писем Markdown. По стандартам Markdown парсеры будут отображать контент с отступом в виде блоков кода.

<a name="button-component"></a>
#### Компонент Button

Компонент кнопки отображает ссылку на кнопку по центру. Компонент принимает два аргумента: `url` и необязательный `color`. Поддерживаемые цвета: `primary`, `success`, и `error`. Вы можете добавить к сообщению столько компонентов кнопки, сколько захотите:

```html
<x-mail::button :url="$url" color="success">
View Order
</x-mail::button>
```

<a name="panel-component"></a>
#### Компонент Panel

Компонент панели отображает указанный блок текста на панели, цвет фона которой немного отличается от цвета остальной части сообщения. Это позволяет привлечь внимание к указанному блоку текста:

```blade
<x-mail::panel>
This is the panel content.
</x-mail::panel>
```

<a name="table-component"></a>
#### Компонент Table

Компонент таблицы позволяет преобразовать таблицу Markdown в таблицу HTML. Компонент принимает в качестве содержимого таблицу Markdown. Выравнивание столбцов таблицы поддерживается с использованием синтаксиса выравнивания таблицы Markdown по умолчанию:

```blade
<x-mail::table>
| Laravel       | Table         | Example  |
| ------------- |:-------------:| --------:|
| Col 2 is      | Centered      | $10      |
| Col 3 is      | Right-Aligned | $20      |
</x-mail::table>
```

<a name="customizing-the-components"></a>
### Изменение компонентов

Вы можете экспортировать все почтовые компоненты Markdown в собственное приложение для настройки. Чтобы экспортировать компоненты, используйте команду `vendor:publish` Artisan с параметром `--tag=laravel-mail`:

```shell
php artisan vendor:publish --tag=laravel-mail
```

Эта команда опубликует почтовые компоненты Markdown в каталоге `resources/views/vendor/mail`. Каталог `mail` будет содержать каталог `html` и `text`, каждый из которых содержит соответствующие представления каждого доступного компонента. Вы можете настроить эти компоненты по своему усмотрению.

<a name="customizing-the-css"></a>
#### Редактирование файла CSS

После экспорта компонентов в каталоге `resources/views/vendor/mail/html/themes` будет содержаться файл `default.css`. Вы можете отредактировать CSS в этом файле, и ваши стили будут автоматически преобразованы во встроенные стили CSS в HTML-представлениях ваших почтовых сообщений Markdown.

Если вы хотите создать совершенно новую тему для компонентов Laravel Markdown, вы можете поместить файл CSS в каталог `html/themes`. После присвоения имени и сохранения файла CSS обновите параметр `theme` в файле конфигурации вашего приложения `config/mail.php`, чтобы он соответствовал имени вашей новой темы.

Чтобы настроить тему для отдельного почтового сообщения, вы можете установить в свойстве `$theme` почтового класса имя темы, которое следует использовать при отправке этого почтового сообщения.

<a name="sending-mail"></a>
## Отправка почты

Чтобы отправить сообщение, используйте метод `to` [фасада](/docs/{{version}}/facades) `Mail`. Метод `to` принимает адрес электронной почты, экземпляр пользователя или коллекцию пользователей. Если вы передаете объект или коллекцию объектов, почтовая программа будет автоматически использовать их свойства `email` и `name` при определении получателей электронной почты, поэтому убедитесь, что эти атрибуты доступны для ваших объектов. После того как вы указали своих получателей, вы можете передать экземпляр вашего почтового класса методу `send`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Mail\OrderShipped;
    use App\Models\Order;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Mail;

    class OrderShipmentController extends Controller
    {
        /**
         * Отправить заказ.
         */
        public function store(Request $request): RedirectResponse
        {
            $order = Order::findOrFail($request->order_id);

            // Отправляем заказ ...

            Mail::to($request->user())->send(new OrderShipped($order));

            return redirect('/orders');
        }
    }

Вы не ограничены простым указанием получателей при отправке сообщения. Вы можете указать получателей to, `cc` и `bcc`, связав их соответствующие методы вместе:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));

<a name="looping-over-recipients"></a>
#### Итерация списка получателей

Иногда требуется отправить почтовое сообщение списку получателей, перебирая массив получателей / адресов электронной почты. Однако, поскольку метод `to` добавляет адреса электронной почты к списку получателей почтового сообщения, каждая итерация цикла будет отправлять другое электронное письмо каждому предыдущему получателю. Следовательно, вы всегда должны повторно создавать почтовый экземпляр для каждого получателя:

    foreach (['taylor@example.com', 'dries@example.com'] as $recipient) {
        Mail::to($recipient)->send(new OrderShipped($order));
    }

<a name="sending-mail-via-a-specific-mailer"></a>
#### Указание драйвера при отправке почты

По умолчанию Laravel будет отправлять электронную почту, используя почтовую программу, настроенную как почтовую программу `default` в файле конфигурации вашего приложения `mail`. Однако вы можете использовать метод `mailer` для отправки сообщения с использованием определенной конфигурации почтовой программы:

    Mail::mailer('postmark')
            ->to($request->user())
            ->send(new OrderShipped($order));

<a name="queueing-mail"></a>
### Очередь почты

<a name="queueing-a-mail-message"></a>
#### Постановка сообщения в очередь почты

Поскольку отправка сообщений электронной почты может негативно повлиять на время отклика вашего приложения, многие разработчики ставят сообщения электронной почты в очередь для фоновой отправки. Laravel упрощает это с помощью встроенного [API унифицированной очереди](/docs/{{version}}/queues). Чтобы поставить почтовое сообщение в очередь, используйте метод `queue` фасада `Mail` после указания получателей сообщения:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));

Этот метод автоматически помещает задание в очередь, чтобы сообщение отправлялось в фоновом режиме. Перед использованием этого функционала вам необходимо [настроить очереди](/docs/{{version}}/queues).

<a name="delayed-message-queueing"></a>
#### Очередь отложенных сообщений

Если вы хотите отложить доставку электронного сообщения в очереди, вы можете использовать метод `later`. В качестве первого аргумента метод `later` принимает экземпляр `DateTime`, указывающий, когда сообщение должно быть отправлено:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later(now()->addMinutes(10), new OrderShipped($order));

<a name="pushing-to-specific-queues"></a>
#### Постановка сообщения в конкретную очередь почты

Поскольку все почтовые классы, сгенерированные с помощью команды `make:mail`, используют трейт `Illuminate\Bus\Queueable`, вы можете вызвать методы `onQueue` и `onConnection` для любого экземпляра почтового класса, что позволит вам указать соединение и имя очереди для сообщения:

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);

<a name="queueing-by-default"></a>
#### Очередь почты, используемая по умолчанию

Если у вас есть почтовые классы, которые вы хотите всегда ставить в очередь, то вы можете реализовать контракт `ShouldQueue` для этого класса. Теперь, даже если вы вызовете метод `send` для отправки, почтовый класс все равно будет помещен в очередь, поскольку он содержит контракт:

    use Illuminate\Contracts\Queue\ShouldQueue;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        // ...
    }

<a name="queued-mailables-and-database-transactions"></a>
#### Почтовые сообщения в очереди и транзакции в базе данных

Когда помещенные в очередь почтовые сообщения отправляются в рамках транзакций базы данных, они могут быть обработаны очередью до того, как транзакция базы данных будет зафиксирована. Когда это происходит, любые обновления, внесенные вами в модели или записи базы данных во время транзакции базы данных, могут еще не быть отражены в базе данных. Кроме того, любые модели или записи базы данных, созданные в рамках транзакции, могут не существовать в базе данных. Если ваше почтовое сообщение зависит от этих моделей, при обработке задания, отправляющего почтовое сообщение в очереди, могут возникнуть непредвиденные ошибки.

Если для параметра `after_commit` конфигурации вашего соединения с очередью задано значение `false`, то вы все равно можете указать, что конкретное почтовое сообщение в очереди должно быть отправлено после того, как все открытые транзакции базы данных были зафиксированы, путем вызова метода `afterCommit` при отправке почтового сообщения:

    Mail::to($request->user())->send(
        (new OrderShipped($order))->afterCommit()
    );

В качестве альтернативы вы можете вызвать метод `afterCommit` из конструктора вашего почтового сообщения:

    <?php

    namespace App\Mail;

    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        use Queueable, SerializesModels;

        /**
         * Create a new message instance.
         */
        public function __construct()
        {
            $this->afterCommit();
        }
    }

> [!NOTE] 
> Чтобы больше узнать о том, как обойти эти проблемы, просмотрите документацию, касающуюся [заданий в очереди и транзакций базы данных](/docs/{{version}}/queues#jobs-and-database-transactions).

<a name="rendering-mailables"></a>
## Отображение отправлений

Иногда требуется получить HTML-содержимое почтового сообщения, не отправляя его. Для этого вы можете вызвать метод `render` почтового сообщения. Этот метод вернет проанализированное HTML-содержимое почтового сообщения в виде строки:

    use App\Mail\InvoicePaid;
    use App\Models\Invoice;

    $invoice = Invoice::find(1);

    return (new InvoicePaid($invoice))->render();

<a name="previewing-mailables-in-the-browser"></a>
### Предварительный просмотр отправлений в браузере

При разработке шаблона почтового сообщения удобно быстро просмотреть визуализированное почтовое сообщение в браузере как типичный шаблон Blade. По этой причине Laravel позволяет вам возвращать любое почтовое сообщение непосредственно из замыкания маршрута или контроллера. Когда почтовое сообщение возвратится, оно будет обработано и отображено в браузере, что позволит вам быстро просмотреть его дизайн без необходимости отправлять его на реальный адрес электронной почты:

    Route::get('/mailable', function () {
        $invoice = App\Models\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

<a name="localizing-mailables"></a>
## Локализация отправлений

Laravel позволяет отправлять почтовые сообщения, используя язык, отличный от текущего языка запроса, и даже будет помнить его, если почта находится в очереди.

Для этого фасад `Mail` содержит метод `locale` для установки желаемого языка. Приложение изменит язык при анализе шаблона почтового сообщения, а затем вернется к предыдущему языку, когда анализ будет завершен:

    Mail::to($request->user())->locale('es')->send(
        new OrderShipped($order)
    );

<a name="user-preferred-locales"></a>
### Предпочитаемые пользователем локализации

Иногда приложения хранят предпочтительный язык каждого пользователя. Реализуя контракт `HasLocalePreference` в ваших моделях, вы можете указать Laravel использовать этот сохраненный язык при отправке почты:

    use Illuminate\Contracts\Translation\HasLocalePreference;

    class User extends Model implements HasLocalePreference
    {
        /**
         * Получить предпочитаемую пользователем локализацию.
         */
        public function preferredLocale(): string
        {
            return $this->locale;
        }
    }

После того как вы реализовали интерфейс, Laravel будет автоматически использовать предпочтительный язык при отправке уведомлений и почтовых сообщений модели. Следовательно, при использовании этого интерфейса нет необходимости вызывать метод `locale`:

    Mail::to($request->user())->send(new OrderShipped($order));

<a name="testing-mailables"></a>
## Тестирование

<a name="testing-mailable-content"></a>
### Тестирование содержимого отправлений

Laravel предоставляет разнообразные методы для анализа структуры вашего класса для отправки электронной почты. Кроме того, Laravel предоставляет несколько удобных методов для проверки наличия ожидаемого содержимого в вашем классе для отправки электронной почты. Эти методы включают в себя: `assertSeeInHtml`, `assertDontSeeInHtml`, `assertSeeInOrderInHtml`, `assertSeeInText`, `assertDontSeeInText`, `assertSeeInOrderInText`, `assertHasAttachment`, `assertHasAttachedData`, `assertHasAttachmentFromStorage` и `assertHasAttachmentFromStorageDisk`.

Как и следовало ожидать, утверждения «HTML» утверждают, что HTML-версия вашего почтового сообщения содержит переданную строку, в то время как утверждения «текст» утверждают, что текстовая версия вашего почтового сообщения содержит переданную строку:

    use App\Mail\InvoicePaid;
    use App\Models\User;

    public function test_mailable_content(): void
    {
        $user = User::factory()->create();

        $mailable = new InvoicePaid($user);

        $mailable->assertFrom('jeffrey@example.com');
        $mailable->assertTo('taylor@example.com');
        $mailable->assertHasCc('abigail@example.com');
        $mailable->assertHasBcc('victoria@example.com');
        $mailable->assertHasReplyTo('tyler@example.com');
        $mailable->assertHasSubject('Invoice Paid');
        $mailable->assertHasTag('example-tag');
        $mailable->assertHasMetadata('key', 'value');

        $mailable->assertSeeInHtml($user->email);
        $mailable->assertSeeInHtml('Invoice Paid');
        $mailable->assertSeeInOrderInHtml(['Invoice Paid', 'Thanks']);

        $mailable->assertSeeInText($user->email);
        $mailable->assertSeeInOrderInText(['Invoice Paid', 'Thanks']);

        $mailable->assertHasAttachment('/path/to/file');
        $mailable->assertHasAttachment(Attachment::fromPath('/path/to/file'));
        $mailable->assertHasAttachedData($pdfData, 'name.pdf', ['mime' => 'application/pdf']);
        $mailable->assertHasAttachmentFromStorage('/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
        $mailable->assertHasAttachmentFromStorageDisk('s3', '/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
    }

<a name="testing-mailable-sending"></a>
### Тестирование отправки почтовых сообщений

Мы рекомендуем тестировать содержимое ваших классов для отправки электронной почты отдельно от ваших тестов, которые утверждают, что определенное письмо было "отправлено" определенному пользователю. Обычно содержимое классов для отправки электронной почты не имеет отношения к коду, который вы тестируете, и достаточно просто утверждать, что Laravel был указан для отправки определенного класса для отправки электронной почты.

Вы можете использовать метод `fake` фасада `Mail`, чтобы предотвратить отправку электронных писем. После вызова метода `fake` фасада `Mail`, вы можете утверждать, что было указано отправить классы для отправки электронной почты пользователям и даже проверять данные, которые были получены классами для отправки электронной почты:

```php
<?php

namespace Tests\Feature;

use App\Mail\OrderShipped;
use Illuminate\Support\Facades\Mail;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_orders_can_be_shipped(): void
    {
        Mail::fake();

        // Выполните доставку заказа...

        // Утверждение, что ни одно письмо не было отправлено...
        Mail::assertNothingSent();

        // Утверждение, что было отправлено одно письмо...
        Mail::assertSent(OrderShipped::class);

        // Утверждение, что было отправлено два письма...
        Mail::assertSent(OrderShipped::class, 2);

        // Утверждение, что другое письмо не было отправлено...
        Mail::assertNotSent(AnotherMailable::class);

        // Утверждение, что всего было отправлено 3 письма...
        Mail::assertSentCount(3);
    }
}
```

Если вы отправляете отправку электронной почты в очередь в фоновом режиме, вам следует использовать метод `assertQueued` вместо `assertSent`:

```php
Mail::assertQueued(OrderShipped::class);
Mail::assertNotQueued(OrderShipped::class);
Mail::assertNothingQueued();
Mail::assertQueuedCount(3);
```

Вы можете передать замыкание в методы `assertSent`, `assertNotSent`, `assertQueued` или `assertNotQueued`, чтобы утверждать, что было отправлено письмо, которое соответствует определенному "тесту истинности". Если хотя бы одно письмо было отправлено и прошло указанный тест, то утверждение будет успешным:

```php
Mail::assertSent(function (OrderShipped $mail) use ($order) {
    return $mail->order->id === $order->id;
});
```

При вызове методов проверки фасада `Mail`, принимаемый замыканием экземпляр класса для отправки электронной почты предоставляет полезные методы для анализа класса для отправки электронной почты:

```php
Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) use ($user) {
    return $mail->hasTo($user->email) &&
           $mail->hasCc('...') &&
           $mail->hasBcc('...') &&
           $mail->hasReplyTo('...') &&
           $mail->hasFrom('...') &&
           $mail->hasSubject('...');
});
```

Экземпляр класса для отправки электронной почты также включает в себя несколько полезных методов для анализа вложений в классе для отправки электронной почты:

```php
use Illuminate\Mail\Mailables\Attachment;

Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) {
    return $mail->hasAttachment(
        Attachment::fromPath('/путь/к/файлу')
                ->as('name.pdf')
                ->withMime('application/pdf')
    );
});

Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) {
    return $mail->hasAttachment(
        Attachment::fromStorageDisk('s3', '/путь/к/файлу')
    );
});

Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) use ($pdfData) {
    return $mail->hasAttachment(
        Attachment::fromData(fn () => $pdfData, 'name.pdf')
    );
});
```

Вы, возможно, заметили, что существует два метода для утверждения, что почта не была отправлена: `assertNotSent` и `assertNotQueued`. Иногда вам может потребоваться утверждать, что почта не была отправлена **или** поставлена в очередь. Для этого вы можете использовать методы `assertNothingOutgoing` и `assertNotOutgoing`:

```php
Mail::assertNothingOutgoing();

Mail::assertNotOutgoing(function (OrderShipped $mail) use ($order) {
    return $mail->order->id === $order->id;
});
```

<a name="mail-and-local-development"></a>
## Почта и локальная разработка

При разработке приложения для отправки электронной почты вы, вероятно, не захотите отправлять электронные письма на реальные адреса электронной почты. Laravel предлагает несколько способов «отключить» фактическую отправку электронных писем во время локальной разработки.

<a name="log-driver"></a>
#### Драйвер Log

Вместо того чтобы отправлять ваши электронные письма, почтовый драйвер `log` будет записывать все сообщения электронной почты в ваши файлы журналов для проверки. Обычно этот драйвер используется только во время локальной разработки. Для получения дополнительной информации о настройке вашего приложения для каждой среды ознакомьтесь с [документацией по конфигурации](/docs/{{version}}/configuration#environment-configuration).

<a name="mailtrap"></a>
#### HELO / Mailtrap / Mailpit

В качестве альтернативы вы можете использовать такую службу, как [HELO](https://usehelo.com) или [Mailtrap](https://mailtrap.io) и драйвер `smtp`, чтобы отправлять сообщения электронной почты в «фиктивный» почтовый ящик, где вы можете просмотреть их в настоящем почтовом клиенте. Этот подход имеет то преимущество, что позволяет вам фактически проверять окончательные электронные письма, непосредственно в почтовых службах.

Если вы используете [Laravel Sail](/docs/{{version}}/sail), то вы можете предварительно просмотреть свои сообщения с помощью [Mailpit](https://github.com/axllent/mailpit). Когда Sail запущен, вы можете получить доступ к интерфейсу Mailpit по адресу: `http://localhost:8025`.

<a name="using-a-global-to-address"></a>
#### Использование глобального адреса `to`

Наконец, вы можете указать глобальный адрес «кому», вызвав метод `alwaysTo`, предлагаемый фасадом Mail. Как правило, этот метод следует вызывать из метода `boot` одного из сервис-провайдеров вашего приложения:

    use Illuminate\Support\Facades\Mail;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        if ($this->app->environment('local')) {
            Mail::alwaysTo('taylor@example.com');
        }
    }

<a name="events"></a>
## События

Laravel запускает два события в процессе отправки почтовых сообщений. Событие `MessageSending` запускается перед отправкой сообщения, а событие `MessageSent` запускается после того, как сообщение было отправлено. Помните, что эти события запускаются, когда почта *отправляется*, а не когда она ставится в очередь. Вы можете зарегистрировать слушатели для этого события в вашем поставщике `App\Providers\EventServiceProvider`:

    use App\Listeners\LogSendingMessage;
    use App\Listeners\LogSentMessage;
    use Illuminate\Mail\Events\MessageSending;
    use Illuminate\Mail\Events\MessageSent;

    /**
     * Карта слушателей событий приложения.
     *
     * @var array
     */
    protected $listen = [
        MessageSending::class => [
            LogSendingMessage::class,
        ],

        MessageSent::class => [
            LogSentMessage::class,
        ],
    ];

<a name="custom-transports"></a>
## Пользовательские транспорты

Laravel включает в себя разнообразные транспорты для отправки электронной почты; однако, возможно, вам захочется написать собственные для доставки электронной почты через другие службы, которые Laravel не поддерживает "из коробки". Для начала определите класс, который расширяет класс `Symfony\Component\Mailer\Transport\AbstractTransport`. Затем реализуйте методы `doSend` и `__toString()` в вашем транспорте:

```php
use MailchimpTransactional\ApiClient;
use Symfony\Component\Mailer\SentMessage;
use Symfony\Component\Mailer\Transport\AbstractTransport;
use Symfony\Component\Mime\Address;
use Symfony\Component\Mime\MessageConverter;

class MailchimpTransport extends AbstractTransport
{
    /**
     * Создайте новый экземпляр транспорта Mailchimp.
     */
    public function __construct(
        protected ApiClient $client,
    ) {
        parent::__construct();
    }

    /**
     * {@inheritDoc}
     */
    protected function doSend(SentMessage $message): void
    {
        $email = MessageConverter::toEmail($message->getOriginalMessage());

        $this->client->messages->send(['message' => [
            'from_email' => $email->getFrom(),
            'to' => collect($email->getTo())->map(function (Address $email) {
                return ['email' => $email->getAddress(), 'type' => 'to'];
            })->all(),
            'subject' => $email->getSubject(),
            'text' => $email->getTextBody(),
        ]]);
    }

    /**
     * Получите строковое представление транспорта.
     */
    public function __toString(): string
    {
        return 'mailchimp';
    }
}
```

После того как вы определите свой собственный транспорт, вы можете зарегистрировать его с помощью метода `extend`, предоставляемого фасадом `Mail`. Обычно это следует делать в методе `boot` служб-поставщиков вашего приложения, который находится в службе `AppServiceProvider`. Вам передается аргумент `$config`, который содержит массив конфигурации, определенной для отправителя почты в конфигурационном файле `config/mail.php` вашего приложения:

```php
use App\Mail\MailchimpTransport;
use Illuminate\Support\Facades\Mail;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Mail::extend('mailchimp', function (array $config = []) {
        return new MailchimpTransport(/* ... */);
    });
}
```

После того как ваш собственный транспорт был определен и зарегистрирован, вы можете создать определение отправителя почты в конфигурационном файле вашего приложения `config/mail.php`, которое будет использовать новый транспорт:

```php
'mailchimp' => [
    'transport' => 'mailchimp',
    // ...
],
```

### Дополнительные транспорты Symfony

Laravel включает поддержку некоторых существующих транспортов для отправки почты, поддерживаемых Symfony, таких как Mailgun и Postmark. Однако, возможно, вам захочется расширить Laravel для поддержки дополнительных транспортов, поддерживаемых Symfony. Для этого вам нужно установить необходимый транспорт Symfony с помощью Composer и зарегистрировать его в Laravel. Например, вы можете установить и зарегистрировать Symfony mailer "Brevo" (ранее "Sendinblue"):

```bash
composer require symfony/brevo-mailer symfony/http-client
```

После установки пакета Brevo mailer вы можете добавить запись с вашими учетными данными API Brevo в конфигурационный файл `services` вашего приложения:

```php
'brevo' => [
    'key' => 'ваш-ключ-api',
],
```

Затем вы можете использовать метод `extend` фасада `Mail` для регистрации транспорта в Laravel. Обычно это следует делать в методе `boot` служб-поставщиков:

```php
use Illuminate\Support\Facades\Mail;
use Symfony\Component\Mailer\Bridge\Brevo\Transport\BrevoTransportFactory;
use Symfony\Component\Mailer\Transport\Dsn;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Mail::extend('brevo', function () {
        return (new BrevoTransportFactory)->create(
            new Dsn(
                'brevo+api',
                'default',
                config('services.brevo.key')
            )
        );
    });
}
```

После регистрации вашего транспорта вы можете создать определение отправителя почты в конфигурационном файле вашего приложения `config/mail.php`, которое будет использовать новый транспорт:

```php
'brevo' => [
    'transport' => 'brevo',
    // ...
],
```
