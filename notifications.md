---
git: c36b582408cb0eddf291238ce5fb1bee21979402
---

# Уведомления


<a name="introduction"></a>
## Введение

В дополнение к поддержке [отправки электронной почты](/docs/{{version}}/mail), Laravel обеспечивает поддержку отправки уведомлений по различным каналам доставки, включая электронную почту, SMS (через [Vonage](https://www.vonage.com/communications-apis/), бывший Nexmo) и [Slack](https://slack.com). Кроме того, сообществом было создано множество [каналов уведомлений](https://laravel-notification-channels.com/about/#suggesting-a-new-channel) для отправки уведомлений по десяткам различных каналов! Уведомления также могут храниться в базе данных, поэтому они могут быть отображены в вашем веб-интерфейсе.

Как правило, уведомления должны быть короткими информационными сообщениями, которые уведомляют пользователей о том, что произошло в вашем приложении. Например, если вы пишете приложение для выставления счетов, то вы можете отправить своим пользователям уведомление «Счет оплачен» по каналам электронной почты и SMS.

<a name="generating-notifications"></a>
## Генерация уведомлений

В Laravel каждое уведомление представлено единым классом. Чтобы сгенерировать новое уведомление, используйте команду `make:notification` [Artisan](artisan). Эта команда поместит новый класс уведомления в каталог `app/Notifications` вашего приложения. Если этот каталог не существует в вашем приложении, то Laravel предварительно создаст его:

```shell
php artisan make:notification InvoicePaid
```

Каждый класс уведомления содержит метод `via` и переменное количество методов формирования сообщений, таких как `toMail` или `toDatabase`, которые преобразуют уведомление в сообщение, адаптированное для этого конкретного канала.

<a name="sending-notifications"></a>
## Отправка уведомлений

<a name="using-the-notifiable-trait"></a>
### Использование трейта `Notifiable`

Уведомления могут быть отправлены двумя способами: с использованием метода `notify` трейта `Notifiable` или с помощью [фасада](/docs/{{version}}/facades) `Notification`. Трейт `Notifiable` по умолчанию содержится в модели `App\Models\User` вашего приложения:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;
    }

Метод `notify`, предоставляемый этим трейтом, ожидает получить экземпляр уведомления:

    use App\Notifications\InvoicePaid;

    $user->notify(new InvoicePaid($invoice));

> [!NOTE] 
> Помните, что вы можете использовать трейт `Notifiable` в любой из ваших моделей. Вы не ограничены использованием его только в модели `User`.

<a name="using-the-notification-facade"></a>
### Использование фасада `Notification`

Как вариант, вы можете отправлять уведомления через [фасад](/docs/{{version}}/facades) `Notification`. Этот подход полезен при отправке уведомления нескольким уведомляемым объектам, например группе пользователей. Чтобы отправлять уведомления с помощью фасада, передайте все уведомляемые сущности и экземпляр уведомления методу `send`:

    use Illuminate\Support\Facades\Notification;

    Notification::send($users, new InvoicePaid($invoice));

Вы также можете немедленно отправлять уведомления, используя метод `sendNow`. Этот метод немедленно отправит уведомление, даже если оно реализует интерфейс `ShouldQueue`:

    Notification::sendNow($developers, new DeploymentCompleted($deployment));

<a name="specifying-delivery-channels"></a>
### Определение каналов доставки

Каждый класс уведомлений имеет метод `via`, который определяет, по каким каналам будет доставлено уведомление. Уведомления можно отправлять по каналам `mail`, `database`, `broadcast`, `vonage` и `slack`.

> [!NOTE] 
> Если вы хотите использовать другие каналы доставки, такие как Telegram или Pusher, то посетите веб-сайт сообщества [Laravel Notification Channels](http://laravel-notification-channels.com).

Метод `via` получает экземпляр `$notifiable`, представляющий экземпляр класса, которому отправляется уведомление. Вы можете использовать `$notifiable`, чтобы определить, по каким каналам должно доставляться уведомление:

    /**
     * Получить каналы доставки уведомлений.
     *
     * @return array<int, string>
     */
    public function via(object $notifiable): array
    {
        return $notifiable->prefers_sms ? ['vonage'] : ['mail', 'database'];
    }

<a name="queueing-notifications"></a>
### Очереди уведомлений

> [!WARNING]  
> Перед отправкой уведомлений в очередь вы должны настроить и запустить [обработчик очереди](/docs/{{version}}/queues).

Отправка уведомлений может занять время, особенно если каналу необходимо выполнить внешний вызов API для доставки уведомления. Чтобы ускорить время отклика вашего приложения, поместите ваше уведомление в очередь, добавив интерфейс `ShouldQueue` и трейт `Queueable` в ваш класс. Интерфейс и трейт уже импортированы для всех уведомлений, сгенерированных с помощью команды `make:notification`, поэтому вы можете сразу добавить их в свой класс уведомлений:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Notifications\Notification;

    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;

        // ...
    }

После добавления интерфейса `ShouldQueue` к уведомлению вы можете отправить уведомление как обычно. Laravel обнаружит интерфейс `ShouldQueue` в классе и автоматически поставит в очередь доставку уведомления:

    $user->notify(new InvoicePaid($invoice));

При постановке уведомлений в очередь для каждого получателя и комбинации каналов будет создана задача в очереди. Например, если ваше уведомление имеет три получателя и два канала, в очередь будет отправлено шесть задач.

<a name="delaying-notifications"></a>
#### Отложенные уведомления

Если вы хотите отложить доставку уведомления, то вы можете вызвать метод `delay` экземпляра уведомления:

    $delay = now()->addMinutes(10);

    $user->notify((new InvoicePaid($invoice))->delay($delay));

<a name="delaying-notifications-per-channel"></a>
#### Отложенные уведомления для каналов

Вы можете передать массив методу `delay`, чтобы указать величину задержки для определенных каналов:

    $user->notify((new InvoicePaid($invoice))->delay([
        'mail' => now()->addMinutes(5),
        'sms' => now()->addMinutes(10),
    ]));

Как альтернативу, вы можете определить метод `withDelay` непосредственно в классе уведомления. Метод `withDelay` должен возвращать массив с именами каналов и значениями задержек:

```php
/**
 * Определение задержки доставки уведомления.
 *
 * @return array<string, \Illuminate\Support\Carbon>
 */
public function withDelay(object $notifiable): array
{
    return [
        'mail' => now()->addMinutes(5),
        'sms' => now()->addMinutes(10),
        // Задержки для других каналов
    ];
}
```

Этот подход позволяет централизованно управлять задержками для разных каналов в одном месте, что упрощает поддержку и модификацию кода.

<a name="customizing-the-notification-queue-connection"></a>
#### Изменение соединения отложенных уведомлений

По умолчанию, уведомления, помещенные в очередь, будут использовать стандартное соединение очереди вашего приложения. Если вы хотите указать другое соединение, которое должно использоваться для конкретного уведомления, вы можете вызвать метод `onConnection` в конструкторе вашего уведомления:

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    /**
     * Создание нового экземпляра уведомления.
     */
    public function __construct()
    {
        $this->onConnection('redis');
    }
}
```

Или, если вы хотите указать конкретное соединение с очередью, которое должно использоваться для каждого канала уведомлений, поддерживаемого уведомлением, вы можете определить метод `viaConnections` в вашем уведомлении. Этот метод должен возвращать массив пар имя канала / имя соединения с очередью:

    /**
     * Определение, какое соединение должно использоваться для каждого канала уведомлений.
     *
     * @return array<string, string>
     */
    public function viaConnections(): array
    {
        return [
            'mail' => 'redis',
            'database' => 'sync',
        ];
    }


<a name="customizing-notification-channel-queues"></a>
#### Изменение очереди канала уведомлений

Если вы хотите указать конкретную очередь, которая должна использоваться для каждого канала уведомления, поддерживаемого уведомлением, то вы можете определить метод `viaQueues` в своем уведомлении. Этот метод должен возвращать массив пар имя канала / имя очереди:

    /**
     * Определить, какие очереди следует использовать для каждого канала уведомления.
     *
     * @return array<string, string>
     */
    public function viaQueues(): array
    {
        return [
            'mail' => 'mail-queue',
            'slack' => 'slack-queue',
        ];
    }

<a name="queued-notifications-and-database-transactions"></a>
#### Уведомления в очереди и транзакции в базе данных

Когда отложенные уведомления отправляются в транзакциях базы данных, они могут быть обработаны очередью до того, как транзакция базы данных будет зафиксирована. Когда это происходит, любые обновления, внесенные вами в модели или записи базы данных во время транзакции базы данных, могут еще не быть отражены в базе данных. Кроме того, любые модели или записи базы данных, созданные в рамках транзакции, могут не существовать в базе данных.

Если для параметра `after_commit` конфигурации вашего соединения с очередью установлено значение `false`, то вы все равно можете указать, что конкретное отложенное уведомление должно быть отправлено после того, как все открытые транзакции базы данных были зафиксированы, путем вызова метода `afterCommit` при отправке уведомления:

    use App\Notifications\InvoicePaid;

    $user->notify((new InvoicePaid($invoice))->afterCommit());

В качестве альтернативы вы можете вызвать метод`afterCommit` из конструктора вашего уведомления:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Notifications\Notification;

    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;

        /**
         * Create a new notification instance.
         */
        public function __construct()
        {
            $this->afterCommit();
        }
    }

> [!NOTE]  
> Чтобы узнать больше о том, как обойти эти проблемы, просмотрите документацию, касающуюся [заданий в очереди и транзакций базы данных](/docs/{{version}}/queues#jobs-and-database-transactions).

<a name="determining-if-the-queued-notification-should-be-sent"></a>
#### Определение необходимости отправки уведомления в очереди

После того как уведомление из очереди было отправлено для фоновой обработки, оно обычно принимается работником очереди и отправляется предполагаемому получателю.

Однако, если вы хотите окончательно определить, следует ли отправлять уведомление в очереди после его обработки работником очереди, вы можете определить метод `shouldSend` в классе уведомлений. Если этот метод возвращает `false`, уведомление не будет отправлено:

    /**
     * Определите, нужно ли отправлять уведомление.
     */
    public function shouldSend(object $notifiable, string $channel): bool
    {
        return $this->invoice->isPaid();
    }

<a name="on-demand-notifications"></a>
### Уведомления по запросу

По желанию можно отправить уведомление кому-то, кто не сохранен как «пользователь» вашего приложения. Используя метод `route` фасада `Notification`, вы можете указать информацию о маршрутизации специального уведомления перед отправкой уведомления:

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Support\Facades\Notification;

    Notification::route('mail', 'taylor@example.com')
                ->route('vonage', '5555555555')
                ->route('slack', '#slack-channel')
                ->route('broadcast', [new Channel('channel-name')])
                ->notify(new InvoicePaid($invoice));

Если вы хотите указать имя получателя при отправке уведомления по запросу на маршрут `mail`, вы можете предоставить массив, содержащий адрес электронной почты в качестве ключа и имя в качестве значения первого элемента в массиве:

    Notification::route('mail', [
        'barrett@example.com' => 'Barrett Blair',
    ])->notify(new InvoicePaid($invoice));

С помощью метода `routes` вы можете предоставить сразу несколько маршрутов для нескольких каналов уведомлений:

    Notification::routes([
        'mail' => ['barrett@example.com' => 'Barrett Blair'],
        'vonage' => '5555555555',
    ])->notify(new InvoicePaid($invoice));

<a name="mail-notifications"></a>
## Почтовые уведомления

<a name="formatting-mail-messages"></a>
### Формирование почтовых сообщений

Если уведомление поддерживает отправку по электронной почте, то вы должны определить метод `toMail` в классе уведомления. Этот метод получит объект `$notifiable` и должен вернуть экземпляр `Illuminate\Notifications\Messages\MailMessage`.

Класс `MailMessage` содержит несколько простых методов, которые помогут вам создавать транзакционные сообщения электронной почты. Почтовые сообщения могут содержать строки текста, а также «призыв к действию». Давайте посмотрим на пример метода `toMail`:

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): MailMessage
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->greeting('Hello!')
                    ->line('One of your invoices has been paid!')
                    ->action('View Invoice', $url)
                    ->line('Thank you for using our application!');
    }

> [!NOTE] 
> Обратите внимание, что мы используем `$this->invoice->id` в нашем методе `toMail`. Вы можете передать любые данные, которые необходимы вашему уведомлению для генерации сообщения, в конструктор уведомления.

В этом примере мы регистрируем приветствие, строку текста, призыв к действию, а затем еще одну строку текста. Эти методы, предоставляемые объектом `MailMessage`, упрощают и ускоряют формирование небольших транзакционных электронных писем. Затем канал `mail` преобразует компоненты сообщения в красивый, отзывчивый HTML-шаблон сообщения электронной почты с аналогом в виде обычного текста. Вот пример электронного письма, созданного каналом `mail`:

<img src="https://laravel.com/img/docs/notification-example-2.png" width="100%" alt="Notification example">

> [!NOTE] 
> При отправке почтовых уведомлений не забудьте установить параметр `name` в вашем конфигурационном файле `config/app.php`. Это значение будет использоваться в верхнем и нижнем колонтитулах ваших почтовых уведомлений.

#### Сообщения об ошибках

Некоторые уведомления информируют пользователей об ошибках, таких как неудачный платеж по счету. Вы можете указать, что почтовое сообщение относится к ошибке, вызвав метод `error` при составлении вашего сообщения. При использовании метода `error` в почтовом сообщении кнопка действия будет красной, а не черной:

```php
/**
 * Получить почтовое представление уведомления.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
                ->error()
                ->subject('Invoice Payment Failed')
                ->line('...');
}
```

Этот подход помогает ясно передать пользователю, что сообщение содержит информацию об ошибке, повышая визуальную восприимчивость и понимание сообщения.

<a name="other-mail-notification-formatting-options"></a>
#### Другие параметры формирования почтовых уведомлений

Вместо определения «строк» текста в классе уведомления, вы можете использовать метод `view`, чтобы указать собственный шаблон, который следует использовать для отображения почтового уведомления:

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)->view(
            'mail.invoice.paid', ['invoice' => $this->invoice]
        );
    }

Вы можете определить текстовое содержимое для почтового сообщения, указав имя представления в качестве второго элемента массива, передаваемого методу `view`:

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)->view(
            ['mail.invoice.paid', 'mail.invoice.paid-text'],
            ['invoice' => $this->invoice]
        );
    }

Или, если ваше сообщение содержит только простой текст, вы можете использовать метод `text`:

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)->text(
            'mail.invoice.paid-text', ['invoice' => $this->invoice]
        );
    }

<a name="customizing-the-sender"></a>
### Изменение отправителя

По умолчанию адрес отправителя электронного письма определяется в конфигурационном файле `config/mail.php`. Однако вы можете указать адрес отправителя для конкретного уведомления с помощью метода `from`:

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->from('barrett@example.com', 'Barrett Blair')
                    ->line('...');
    }

<a name="customizing-the-recipient"></a>
### Изменение получателя

При отправке уведомлений по каналу `mail` система уведомлений автоматически ищет свойство `email` уведомляемого объекта. Вы можете указать, какой адрес электронной почты будет использоваться для доставки уведомления, определив метод `routeNotificationForMail` объекта уведомления:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Notifications\Notification;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Маршрутизация уведомлений для почтового канала.
         *
         * @return  array<string, string>|string
         */
        public function routeNotificationForMail(Notification $notification): array|string
        {
            // Вернуть только адрес электронной почты ...
            return $this->email_address;

            // Вернуть адрес электронной почты и имя ...
            return [$this->email_address => $this->name];
        }
    }

<a name="customizing-the-subject"></a>
### Изменение темы сообщения

По умолчанию темой электронного письма является название класса уведомления в регистре «Title Case». Итак, если ваш класс уведомлений называется `InvoicePaid`, то темой электронного письма будет «Invoice Paid». Если вы хотите указать другую тему для сообщения, то вы можете вызвать метод `subject` при создании своего сообщения:

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-mailer"></a>
### Изменение почтового драйвера

По умолчанию уведомление по электронной почте будет отправлено с использованием почтового драйвера по умолчанию, определенной в конфигурационном файле `config/mail.php`. Однако вы можете указать другой почтовый драйвер во время выполнения, вызвав метод `mailer` при создании вашего сообщения:

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->mailer('postmark')
                    ->line('...');
    }

<a name="customizing-the-templates"></a>
### Изменение почтовых шаблонов

Вы можете изменить шаблон из HTML и обычного текста, используемый для почтовых уведомлений, опубликовав необходимые ресурсы уведомления. После выполнения этой команды шаблоны почтовых уведомлений будут расположены в каталоге `resources/views/vendor/notifications`:

```shell
php artisan vendor:publish --tag=laravel-notifications
```

<a name="mail-attachments"></a>
### Почтовые вложения

Чтобы добавить вложения к почтовому уведомлению, используйте метод `attach` при создании сообщения. Метод `attach` принимает абсолютный путь к файлу в качестве своего первого аргумента:

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attach('/path/to/file');
    }

> [!NOTE]  
> Метод `attach`, предоставляемый почтовыми сообщениями уведомлений, также принимает [объекты, прикрепляемые к сообщению](/docs/{{version}}/mail#attachable-objects). Пожалуйста, ознакомьтесь с подробной [документацией об объектах, прикрепляемых к сообщениям](/docs/{{version}}/mail#attachable-objects), чтобы узнать больше.

При прикреплении файлов к сообщению вы также можете указать отображаемое имя и / или MIME-тип, передав массив в качестве второго аргумента методу `attach`:

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attach('/path/to/file', [
                        'as' => 'name.pdf',
                        'mime' => 'application/pdf',
                    ]);
    }

В отличие от прикрепления файлов к почтовым отправлениям, вы не можете прикреплять файл непосредственно с диска файлового хранилища с помощью `attachFromStorage`. Лучше использовать метод `attach` с абсолютным путем к файлу на диске. В качестве альтернативы вы можете вернуть [отправление](/docs/{{version}}/mail#generating-mailables) из метода `toMail`:

    use App\Mail\InvoicePaid as InvoicePaidMailable;

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): Mailable
    {
        return (new InvoicePaidMailable($this->invoice))
                    ->to($notifiable->email)
                    ->attachFromStorage('/path/to/file');
    }

При необходимости к сообщению можно прикрепить несколько файлов, используя метод `attachMany`:

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attachMany([
                        '/path/to/forge.svg',
                        '/path/to/vapor.svg' => [
                            'as' => 'Logo.svg',
                            'mime' => 'image/svg+xml',
                        ],
                    ]);
    }

<a name="raw-data-attachments"></a>
#### Почтовые вложения необработанных данных

Метод `attachData` используется для присоединения необработанной строки в качестве вложения. При вызове метода `attachData` вы должны указать имя файла, которое должно быть присвоено вложению:

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): Mailable
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attachData($this->pdf, 'name.pdf', [
                        'mime' => 'application/pdf',
                    ]);
    }

<a name="adding-tags-metadata"></a>
### Добавление тегов и метаданных

Некоторые сторонние почтовые провайдеры, такие как Mailgun и Postmark, поддерживают "теги" и "метаданные" сообщений, которые могут использоваться для группировки и отслеживания электронных писем, отправленных вашим приложением. Вы можете добавить теги и метаданные к электронному сообщению с помощью методов `tag` и `metadata`:

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->greeting('Comment Upvoted!')
                    ->tag('upvote')
                    ->metadata('comment_id', $this->comment->id);
    }

Если ваше приложение использует драйвер Mailgun, вы можете обратиться к документации Mailgun для получения дополнительной информации о [тегах](https://documentation.mailgun.com/en/latest/user_manual.html#tagging-1) и [метаданных](https://documentation.mailgun.com/en/latest/user_manual.html#attaching-data-to-messages). Аналогично, документацию Postmark можно также проконсультировать для получения информации о их поддержке [тегов](https://postmarkapp.com/blog/tags-support-for-smtp) и [метаданных](https://postmarkapp.com/support/article/1125-custom-metadata-faq).

Если ваше приложение использует Amazon SES для отправки электронных писем, вы должны использовать метод `metadata` для прикрепления [тегов SES](https://docs.aws.amazon.com/ses/latest/APIReference/API_MessageTag.html) к сообщению.

<a name="customizing-the-symfony-message"></a>
### Настройка сообщения Symfony

Метод `withSymfonyMessage` класса `MailMessage` позволяет зарегистрировать функцию обратного вызова, которая будет вызвана с экземпляром сообщения Symfony перед отправкой сообщения. Это дает вам возможность глубоко настроить сообщение перед его доставкой:

    use Symfony\Component\Mime\Email;
    
    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->withSymfonyMessage(function (Email $message) {
                        $message->getHeaders()->addTextHeader(
                            'Custom-Header', 'Header Value'
                        );
                    });
    }

<a name="using-mailables"></a>
### Использование почтовых отправлений

При необходимости вы можете вернуть полный [объект почтового отправления](/docs/{{version}}/mail) из метода `toMail` вашего уведомления. При возврате `Mailable` вместо `MailMessage` вам нужно будет указать получателя сообщения с помощью метода `to` объекта почтового отправления:

    use App\Mail\InvoicePaid as InvoicePaidMailable;
    use Illuminate\Mail\Mailable;

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): Mailable
    {
        return (new InvoicePaidMailable($this->invoice))
                    ->to($notifiable->email);
    }

<a name="mailables-and-on-demand-notifications"></a>
#### Почтовые отправления и уведомления по запросу

Если вы отправляете [уведомление по запросу](#on-demand-notifications), то экземпляр `$notifiable`, переданный методу `toMail`, будет экземпляром `Illuminate\Notifications\AnonymousNotifiable`, содержащий метод `routeNotificationFor`, который можно использовать для получения адреса электронной почты для отправления уведомления по запросу:

    use App\Mail\InvoicePaid as InvoicePaidMailable;
    use Illuminate\Notifications\AnonymousNotifiable;
    use Illuminate\Mail\Mailable;

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): Mailable
    {
        $address = $notifiable instanceof AnonymousNotifiable
                ? $notifiable->routeNotificationFor('mail')
                : $notifiable->email;

        return (new InvoicePaidMailable($this->invoice))
                    ->to($address);
    }

<a name="previewing-mail-notifications"></a>
### Предварительный просмотр почтовых уведомлений

При разработке шаблона почтового уведомления удобно быстро просмотреть визуализированное почтовое сообщение в браузере, как типичный шаблон Blade. По этой причине Laravel позволяет вам возвращать любое почтовое сообщение непосредственно из замыкания маршрута или контроллера. При возврате `MailMessage`, оно будет обработано и отображено в браузере, что позволит вам быстро просмотреть его дизайн без необходимости отправлять его на реальный адрес электронной почты:

    use App\Models\Invoice;
    use App\Notifications\InvoicePaid;

    Route::get('/notification', function () {
        $invoice = Invoice::find(1);

        return (new InvoicePaid($invoice))
                    ->toMail($invoice->user);
    });

<a name="markdown-mail-notifications"></a>
## Почтовые уведомления с разметкой Markdown

Почтовые уведомления с разметкой Markdown позволяют вам воспользоваться преимуществами предварительно созданных шаблонов почтовых уведомлений. Поскольку сообщения написаны на Markdown, Laravel может отображать красивые, отзывчивые HTML-шаблоны для сообщений, а также автоматически генерировать аналог в виде простого текста.

<a name="generating-the-message"></a>
### Генерация сообщения

Чтобы сгенерировать уведомление с соответствующим шаблоном Markdown, вы можете использовать параметр `--markdown` команды `make:notification` Artisan:

```shell
php artisan make:notification InvoicePaid --markdown=mail.invoice.paid
```

Как и все другие почтовые уведомления, уведомления, использующие шаблоны Markdown, должны определять метод `toMail` в своем классе уведомлений. Однако вместо использования методов `line` и `action` для создания уведомления используйте метод `markdown`, чтобы указать имя шаблона Markdown, который следует использовать. Массив данных, который вы хотите сделать доступным для шаблона, может быть передан в качестве второго аргумента метода:

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): MailMessage
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }

<a name="writing-the-message"></a>
### Написание сообщения

Почтовые уведомления Markdown используют комбинацию компонентов Blade и синтаксиса Markdown, которые позволяют легко создавать почтовые уведомления, используя предварительно созданные компоненты уведомлений Laravel:

```blade
<x-mail::message>
# Invoice Paid

Your invoice has been paid!

<x-mail::button :url="$url">
View Invoice
</x-mail::button>

Thanks,<br>
{{ config('app.name') }}
</x-mail::message>
```

<a name="button-component"></a>
#### Компонент Button

Компонент кнопки отображает ссылку на кнопку по центру. Компонент принимает два аргумента: `url` и необязательный `color`. Поддерживаемые цвета: `primary`, `green`, и `red`. Вы можете добавить к уведомлению столько компонентов кнопки, сколько захотите:

```blade
<x-mail::button :url="$url" color="green">
View Invoice
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

Чтобы настроить тему для отдельного уведомления, вы можете вызвать метод `theme` при создании почтового сообщения уведомления. Метод `theme` принимает имя темы, которая должна использоваться при отправке уведомления:

    /**
     * Получить содержимое почтового уведомления.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->theme('invoice')
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }

<a name="database-notifications"></a>
## Уведомления через канал `database`

<a name="database-prerequisites"></a>
### Предварительная подготовка базы данных

Канал уведомлений `database` хранит информацию уведомления в таблице базы данных. Эта таблица будет содержать такую информацию, как тип уведомления, а также JSON-структуру данных, которая описывает уведомление.

Вы можете запросить таблицу, чтобы отобразить уведомления в пользовательском интерфейсе вашего приложения. Но прежде чем вы сможете это сделать, вам нужно будет создать таблицу базы данных для хранения ваших уведомлений. Вы можете использовать команду `notifications:table` для создания [миграции](/docs/{{version}}/migrations) с необходимой схемой таблицы:

```shell
php artisan notifications:table

php artisan migrate
```

> [!NOTE]  
> Если ваши модели с уведомлениями используют [UUID или ULID в качестве первичных ключей](/docs/{{version}}/eloquent#uuid-and-ulid-keys), вы должны заменить метод `morphs` на [`uuidMorphs`](/docs/{{version}}/migrations#column-method-uuidMorphs) или [`ulidMorphs`](/docs/{{version}}/migrations#column-method-ulidMorphs) в миграции таблицы уведомлений.

<a name="formatting-database-notifications"></a>
### Формирование уведомлений канала `database`

Чтобы уведомление было сохранено в таблице базы данных, вы должны определить метод `toDatabase` или `toArray` в классе уведомления. Каждый из этих методов получает объект `$notifiable` и должен возвращать простой массив PHP. Возвращенный массив будет закодирован как JSON и сохранен в столбце `data` вашей таблицы `notifications`. Давайте посмотрим на пример метода `toArray`:

    /**
     * Получить массив данных уведомления.
     *
     * @return array<string, mixed>
     */
    public function toArray(object $notifiable): array
    {
        return [
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ];
    }

Когда уведомление сохраняется в базе данных вашего приложения, столбец `type` заполняется именем класса уведомления. Однако вы можете настроить это поведение, определив метод `databaseType` в вашем классе уведомления:

```php
/**
 * Получить тип уведомления для базы данных.
 *
 * @return string
 */
public function databaseType(object $notifiable): string
{
    return 'invoice-paid';
}
```

Этот метод позволяет вам устанавливать пользовательский тип уведомления, который будет сохранен в столбце `type` в таблице уведомлений базы данных. Это может быть полезно для более удобного отслеживания и фильтрации уведомлений по типу.

<a name="todatabase-vs-toarray"></a>
#### Методы `toDatabase` и `toArray`

Метод `toArray` также используется каналом `broadcast`, чтобы определить, какие данные транслировать в JavaScript-приложение на клиентской стороне. Если вы хотите иметь два разных массива данных для каналов `database` и `broadcast`, то вы должны определить метод `toDatabase` вместо метода `toArray`.

<a name="accessing-the-notifications"></a>
### Доступ к уведомлениям

После сохранения уведомления в базу данных, вам понадобится удобный способ доступа к нему из уведомляемых объектов. Трейт `Illuminate\Notifications\Notifiable`, который по умолчанию расположен в модели `App\Models\User` Laravel, содержит [отношение](/docs/{{version}}/eloquent-relationships) `notifications` Eloquent, возвращающее уведомления для объекта. Вы можете обратиться к этому методу, как и к любому другому отношению Eloquent, чтобы получить уведомления. По умолчанию уведомления будут упорядочены по столбцу `created_at` временной метки, причем самые последние уведомления будут помещены в начало коллекции:

    $user = App\Models\User::find(1);

    foreach ($user->notifications as $notification) {
        echo $notification->type;
    }

Для получения только «непрочитанных» уведомлений, используйте отношение `unreadNotifications`. Опять же, эти уведомления будут упорядочены по столбцу `created_at` временной метки с самыми последними уведомлениями в начале коллекции:

    $user = App\Models\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        echo $notification->type;
    }

> [!NOTE] 
> Чтобы получить доступ к уведомлениям в JavaScript-приложении на клиентской стороне, вы должны определить контроллер уведомлений для своего приложения, который возвращает уведомления для уведомляемого объекта, такого как текущий пользователь. Затем вы можете сделать HTTP-запрос к URL-адресу этого контроллера из своего JavaScript-приложения на клиентской стороне.

<a name="marking-notifications-as-read"></a>
### Отметка прочитанных уведомлений

По желанию можно пометить уведомление как «прочитанное», когда пользователь его просматривает. Трейт `Illuminate\Notifications\Notifiable` содержит метод `markAsRead`, который обновляет столбец `read_at` записи уведомления в базе данных:

    $user = App\Models\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        $notification->markAsRead();
    }

Однако вместо того, чтобы перебирать каждое уведомление, вы можете использовать метод `markAsRead` непосредственно для коллекции уведомлений:

    $user->unreadNotifications->markAsRead();

Вы также можете выполнить запрос массового обновления, чтобы пометить все уведомления как прочитанные, не извлекая их из базы данных:

    $user = App\Models\User::find(1);

    $user->unreadNotifications()->update(['read_at' => now()]);

Вы можете полностью удалить уведомления из таблицы, используя метод `delete`:

    $user->notifications()->delete();

<a name="broadcast-notifications"></a>
## Трансляция уведомлений

<a name="broadcast-prerequisites"></a>
### Предварительная подготовка трансляции

Перед трансляцией уведомлений вы должны настроить и ознакомиться со службами [трансляции событий](/docs/{{version}}/broadcasting) Laravel. Трансляция событий – способ реагирования на серверные события Laravel из своего JavaScript-приложения на клиентской стороне.

<a name="formatting-broadcast-notifications"></a>
### Формирование транслируемых уведомлений

Канал `broadcast` транслирует уведомления с использованием служб [трансляции событий](/docs/{{version}}/broadcasting) Laravel, что позволяет вашему JavaScript-приложению на клиентской стороне улавливать уведомления в режиме реального времени. Если уведомление поддерживает трансляцию, то вы должны определить метод `toBroadcast` в классе уведомления. Этот метод получит объект `$notifiable` и должен вернуть экземпляр` BroadcastMessage`. Если метод `toBroadcast` не существует, то метод `toArray` будет использоваться для сбора данных, которые следует транслировать. Возвращенные данные будут закодированы как JSON и переданы вашему JavaScript-приложению на клиентской стороне. Давайте посмотрим на пример метода `toBroadcast`:

    use Illuminate\Notifications\Messages\BroadcastMessage;

    /**
     * Получить содержимое транслируемого уведомления.
     */
    public function toBroadcast(object $notifiable): BroadcastMessage
    {
        return new BroadcastMessage([
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ]);
    }

<a name="broadcast-queue-configuration"></a>
#### Конфигурирование очереди трансляции

Все транслируемые уведомления ставятся в очередь для трансляции. Если вы хотите изменить соединение очереди или имя очереди, которое используется для постановки в очередь трансляции, то вы можете использовать методы `onConnection` и `onQueue` экземпляра `BroadcastMessage`:

    return (new BroadcastMessage($data))
                    ->onConnection('sqs')
                    ->onQueue('broadcasts');

<a name="customizing-the-notification-type"></a>
#### Изменение типа транслируемого уведомления

В дополнение к указанным вами данным все транслируемые уведомления также имеют поле `type`, содержащее полное имя класса уведомления. Если вы хотите изменить `type` уведомления, то вы можете определить метод `broadcastType` в классе уведомления:

    /**
     * Получить тип транслируемого уведомления.
     */
    public function broadcastType(): string
    {
        return 'broadcast.message';
    }

<a name="listening-for-notifications"></a>
### Прослушивание транслируемых уведомлений

Уведомления будут транслироваться по частному каналу, в формате с использованием соглашения `{notifiable}.{id}`. Итак, если вы отправляете уведомление экземпляру `App\Models\User` с идентификатором `1`, то уведомление будет транслироваться по частному каналу `App.Models.User.1`. При использовании [Laravel Echo](/docs/{{version}}/broadcasting#client-side-installation) вы можете легко прослушивать уведомления канала, используя метод `notification`:

    Echo.private('App.Models.User.' + userId)
        .notification((notification) => {
            console.log(notification.type);
        });

<a name="customizing-the-notification-channel"></a>
#### Изменение канала транслируемого уведомления

Если вы хотите изменить канал, на котором транслируются уведомления объекта, то вы можете определить метод `receivesBroadcastNotificationsOn` объекта уведомления:

    <?php

    namespace App\Models;

    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Каналы, по которым пользователь получает рассылку уведомлений.
         */
        public function receivesBroadcastNotificationsOn(): string
        {
            return 'users.'.$this->id;
        }
    }

<a name="sms-notifications"></a>
## Уведомления через SMS

<a name="sms-prerequisites"></a>
### Предварительная подготовка канала SMS

Отправка SMS-уведомлений в Laravel обеспечивается [Vonage](https://www.vonage.com/) (бывший Nexmo).
Прежде чем вы сможете отправлять уведомления через Vonage, вам необходимо установить пакеты `laravel/vonage-notification-channel` и `guzzlehttp/guzzle`:

    composer require laravel/vonage-notification-channel guzzlehttp/guzzle

Пакет включает в себя [файл конфигурации](https://github.com/laravel/vonage-notification-channel/blob/3.x/config/vonage.php). Однако вам не обязательно экспортировать этот файл конфигурации в ваше собственное приложение. Вы можете просто использовать переменные окружения `VONAGE_KEY` и `VONAGE_SECRET` для определения ваших публичного и секретного ключей Vonage.

После определения ваших ключей, вы должны установить переменную окружения `VONAGE_SMS_FROM`, которая определяет номер телефона, с которого по умолчанию будут отправляться ваши SMS-сообщения. Вы можете сгенерировать этот номер телефона в панели управления Vonage:

```
VONAGE_SMS_FROM=15556666666
```

<a name="formatting-sms-notifications"></a>
### Формирование уведомлений через SMS

Если уведомление поддерживает отправку в виде SMS, то вы должны определить метод `toVonage` в классе уведомлений. Этот метод получит объект `$notifiable` и должен вернуть экземпляр `Illuminate\Notifications\Messages\NexmoMessage`:

    /**
     * Получить SMS-представление уведомления.
     */
    public function toVonage(object $notifiable): VonageMessage
    {
        return (new VonageMessage)
                    ->content('Your SMS message content');
    }

<a name="unicode-content"></a>
#### Содержимое Unicode

Если ваше SMS-сообщение будет содержать символы Unicode, то вы должны вызвать метод `unicode` при создании экземпляра `VonageMessage`:

    use Illuminate\Notifications\Messages\VonageMessage;

    /**
     * Получить SMS-представление уведомления.
     */
    public function toVonage(object $notifiable): VonageMessage
    {
        return (new VonageMessage)
                    ->content('Your unicode message')
                    ->unicode();
    }

<a name="customizing-the-from-number"></a>
### Изменение номера отправителя

Если вы хотите отправить уведомление с номера телефона, который отличается от номера телефона, указанного в вашем файле `config/services.php`, то вы можете вызвать метод `from` экземпляра `VonageMessage`:

    /**
     * Получить Vonage / SMS-представление уведомления.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toVonage(object $notifiable): VonageMessage
    {
        return (new VonageMessage)
                    ->content('Your SMS message content')
                    ->from('15554443333');
    }

<a name="adding-a-client-reference"></a>
### Добавление ссылки на клиента

Если вы хотите отслеживать затраты на пользователя, команду или клиента, вы можете добавить в уведомление "ссылку на клиента". Vonage позволит вам создавать отчеты, используя эту ссылку клиента, чтобы вы могли лучше понять использование SMS конкретным клиентом. Ссылка на клиента может быть любой строкой до 40 символов:

    /**
     * Получить Vonage / SMS-представление уведомления.
     */
    public function toVonage(object $notifiable): VonageMessage
    {
        return (new VonageMessage)
                    ->clientReference((string) $notifiable->id)
                    ->content('Your SMS message content');
    }

<a name="routing-sms-notifications"></a>
### Маршрутизация SMS-уведомлений

Для отправки уведомления с использованием Vonage на необходимый номер телефона, определите метод `routeNotificationForVonage` вашего уведомляемого объекта:

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Notifications\Notification;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Маршрутизация уведомлений для канала Vonage.
         */
        public function routeNotificationForVonage(Notification $notification): string
        {
            return $this->phone_number;
        }
    }

<a name="slack-notifications"></a>
## Уведомления через Slack

<a name="slack-prerequisites"></a>
### Предварительная подготовка канала Slack

Прежде чем вы сможете отправлять уведомления через Slack, вы должны установить канал уведомлений Slack через Composer:

```shell
composer require laravel/slack-notification-channel
```

Дополнительно, вы должны создать [Slack приложение](https://api.slack.com/apps?new_app=1) для вашего рабочего пространства Slack.

Если вам нужно отправлять уведомления только в то же рабочее пространство Slack, в котором создано приложение, убедитесь, что ваше приложение имеет разрешения `chat:write`, `chat:write.public` и `chat:write.customize`. Эти разрешения можно добавить на вкладке "OAuth & Permissions" в управлении приложениями Slack.

Далее, скопируйте "Bot User OAuth Token" вашего приложения и поместите его в массив конфигурации `slack` в файле конфигурации `services.php` вашего приложения. Этот токен можно найти на вкладке "OAuth & Permissions" в Slack:

```php
'slack' => [
    'notifications' => [
        'bot_user_oauth_token' => env('SLACK_BOT_USER_OAUTH_TOKEN'),
        'channel' => env('SLACK_BOT_USER_DEFAULT_CHANNEL'),
    ],
],
```

<a name="slack-app-distribution"></a>
#### Распространение приложения Slack

Если ваше приложение будет отправлять уведомления во внешние рабочие пространства Slack, которые принадлежат пользователям вашего приложения, вам нужно будет "распространить" ваше приложение через Slack. Управление распространением приложения можно осуществить на вкладке "Manage Distribution" вашего приложения в Slack. После того, как ваше приложение будет распространено, вы можете использовать [Socialite](/docs/{{version}}/socialite), чтобы [получать токены Slack Bot](/docs/{{version}}/socialite#slack-bot-scopes) от имени пользователей вашего приложения.

<a name="formatting-slack-notifications"></a>
### Формирование уведомления через Slack

Если уведомление поддерживает отправку в виде сообщения Slack, вы должны определить метод `toSlack` в классе уведомления. Этот метод получает сущность `$notifiable` и должен возвращать экземпляр `Illuminate\Notifications\Slack\SlackMessage`. Вы можете создавать богатые уведомления, используя [API Block Kit Slack](https://api.slack.com/block-kit). Следующий пример может быть предпросмотрен в [Block Kit Builder Slack](https://app.slack.com/block-kit-builder/T01KWS6K23Z#%7B%22blocks%22:%5B%7B%22type%22:%22header%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Invoice%20Paid%22%7D%7D,%7B%22type%22:%22context%22,%22elements%22:%5B%7B%22type%22:%22plain_text%22,%22text%22:%22Customer%20%231234%22%7D%5D%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22An%20invoice%20has%20been%20paid.%22%7D,%22fields%22:%5B%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Invoice%20No:*%5Cn1000%22%7D,%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Invoice%20Recipient:*%5Cntaylor@laravel.com%22%7D%5D%7D,%7B%22type%22:%22divider%22%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Congratulations!%22%7D%7D%5D%7D):

    use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
    use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
    use Illuminate\Notifications\Slack\BlockKit\Composites\ConfirmObject;
    use Illuminate\Notifications\Slack\SlackMessage;

    /**
     * Получить представление Slack-уведомления.
     */
    public function toSlack(object $notifiable): SlackMessage
    {
        return (new SlackMessage)
                ->text('One of your invoices has been paid!')
                ->headerBlock('Invoice Paid')
                ->contextBlock(function (ContextBlock $block) {
                    $block->text('Customer #1234');
                })
                ->sectionBlock(function (SectionBlock $block) {
                    $block->text('An invoice has been paid.');
                    $block->field("*Invoice No:*\n1000")->markdown();
                    $block->field("*Invoice Recipient:*\ntaylor@laravel.com")->markdown();
                })
                ->dividerBlock()
                ->sectionBlock(function (SectionBlock $block) {
                    $block->text('Congratulations!');
                });
    }

<a name="slack-interactivity"></a>
### Взаимодействие в Slack

Система уведомлений Block Kit Slack предлагает мощные функции для [обработки взаимодействия пользователя](https://api.slack.com/interactivity/handling). Чтобы использовать эти функции, ваше приложение Slack должно иметь включенную функцию "Interactivity" и настроенный "Request URL", который указывает на URL, обслуживаемый вашим приложением. Эти настройки можно управлять на вкладке "Interactivity & Shortcuts" в управлении приложениями Slack.

В следующем примере, который использует метод `actionsBlock`, Slack отправит `POST` запрос на ваш "Request URL" с полезной нагрузкой, содержащей пользователя Slack, который нажал на кнопку, идентификатор нажатой кнопки и дополнительную информацию. Ваше приложение может затем определить, какое действие следует предпринять на основе полученной полезной нагрузки. Также вы должны [проверить подлинность запроса](https://api.slack.com/authentication/verifying-requests-from-slack), чтобы убедиться, что он был сделан Slack:

    use Illuminate\Notifications\Slack\BlockKit\Blocks\ActionsBlock;
    use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
    use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
    use Illuminate\Notifications\Slack\SlackMessage;

    /**
     * Получить представление Slack-уведомления.
     */
    public function toSlack(object $notifiable): SlackMessage
    {
        return (new SlackMessage)
                ->text('One of your invoices has been paid!')
                ->headerBlock('Invoice Paid')
                ->contextBlock(function (ContextBlock $block) {
                    $block->text('Customer #1234');
                })
                ->sectionBlock(function (SectionBlock $block) {
                    $block->text('An invoice has been paid.');
                })
                ->actionsBlock(function (ActionsBlock $block) {
                     // ID defaults to "button_acknowledge_invoice"...
                    $block->button('Acknowledge Invoice')->primary();

                    // Manually configure the ID...
                    $block->button('Deny')->danger()->id('deny_invoice');
                });
    }

<a name="slack-confirmation-modals"></a>
#### Модальные окна подтверждения

Если вы хотите, чтобы пользователи подтверждали действие перед его выполнением, вы можете использовать метод `confirm` при определении вашей кнопки. Метод `confirm` принимает сообщение и замыкание, которое получает экземпляр `ConfirmObject`:


    use Illuminate\Notifications\Slack\BlockKit\Blocks\ActionsBlock;
    use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
    use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
    use Illuminate\Notifications\Slack\BlockKit\Composites\ConfirmObject;
    use Illuminate\Notifications\Slack\SlackMessage;

    /**
     * Получить представление Slack-уведомления.
     */
    public function toSlack(object $notifiable): SlackMessage
    {
        return (new SlackMessage)
                ->text('One of your invoices has been paid!')
                ->headerBlock('Invoice Paid')
                ->contextBlock(function (ContextBlock $block) {
                    $block->text('Customer #1234');
                })
                ->sectionBlock(function (SectionBlock $block) {
                    $block->text('An invoice has been paid.');
                })
                ->actionsBlock(function (ActionsBlock $block) {
                    $block->button('Acknowledge Invoice')
                        ->primary()
                        ->confirm(
                            'Acknowledge the payment and send a thank you email?',
                            function (ConfirmObject $dialog) {
                                $dialog->confirm('Yes');
                                $dialog->deny('No');
                            }
                        );
                });
    }

<a name="inspecting-slack-blocks"></a>
#### Просмотр cтруктуры блоков Slack

Если вы хотите быстро проверить структуру блоков, которые вы создали, вы можете использовать метод `dd` в экземпляре `SlackMessage`. Метод `dd` сгенерирует и выведет URL-адрес для [Block Kit Builder Slack](https://app.slack.com/block-kit-builder/), который отображает предварительный просмотр полезной нагрузки и уведомления в вашем браузере. Вы можете передать `true` методу `dd`, чтобы вывести сырую полезную нагрузку:

```php
return (new SlackMessage)
        ->text('Один из ваших счетов оплачен!')
        ->headerBlock('Счет Оплачен')
        ->dd(); // Это вызовет просмотр блоков в Block Kit Builder
```

Этот метод особенно полезен во время разработки, так как позволяет быстро и удобно проверить внешний вид и структуру ваших уведомлений в Slack, не отправляя их на самом деле.

<a name="routing-slack-notifications"></a>
### Маршрутизация уведомлений в Slack

Чтобы направлять уведомления Slack в соответствующую команду Slack и канал, определите метод `routeNotificationForSlack` в вашей модели, уведомляемой событиями. Этот метод может возвращать одно из трех значений:

- `null` - что означает использование маршрутизации к каналу, настроенному в самом уведомлении. Вы можете использовать метод `to` при создании вашего `SlackMessage` для настройки канала в уведомлении.
- Строку, указывающую Slack канал, куда следует отправить уведомление, например, `#support-channel`.
- Экземпляр `SlackRoute`, который позволяет вам указать OAuth токен и имя канала, например, `SlackRoute::make($this->slack_channel, $this->slack_token)`. Этот метод следует использовать для отправки уведомлений во внешние рабочие пространства.

Например, возвращение `#support-channel` из метода `routeNotificationForSlack` отправит уведомление в канал `#support-channel` рабочего пространства, связанного с OAuth токеном Bot User, расположенным в файле конфигурации `services.php` вашего приложения:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Notifications\Notification;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Маршрутизация уведомлений для канала Slack.
     */
    public function routeNotificationForSlack(Notification $notification): string
    {
        return '#support-channel';
    }
}
```

<a name="notifying-external-slack-workspaces"></a>
### Уведомление во внешние рабочие пространства Slack

> [!NOTE]
> Прежде чем отправлять уведомления во внешние рабочие пространства Slack, ваше приложение Slack должно быть [распространено](#slack-app-distribution).

Конечно, часто вам захочется отправлять уведомления в рабочие пространства Slack, которые принадлежат пользователям вашего приложения. Для этого сначала вам потребуется получить OAuth-токен Slack для пользователя. К счастью, [Laravel Socialite](/docs/{{version}}/socialite) включает драйвер Slack, который позволит вам легко аутентифицировать пользователей вашего приложения в Slack и [получать токен бота](/docs/{{version}}/socialite#slack-bot-scopes).

После получения токена бота и его сохранения в базе данных вашего приложения, вы можете использовать метод `SlackRoute::make` для направления уведомления в рабочее пространство пользователя. Кроме того, ваше приложение, скорее всего, должно предоставить пользователю возможность указать, в какой канал следует отправлять уведомления:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Slack\SlackRoute;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Маршрутизация уведомлений для канала Slack.
     *
     * @param  \Illuminate\Notifications\Notification  $notification
     * @return string
     */
    public function routeNotificationForSlack(Notification $notification): mixed
    {
        return SlackRoute::make($this->slack_channel, $this->slack_token);
    }
}
```

<a name="localizing-notifications"></a>
## Локализация уведомлений

Laravel позволяет отправлять уведомления, используя язык, отличный от текущего языка запроса, и даже будет помнить его, если уведомление находится в очереди.

Для этого класс `Illuminate\Notifications\Notification` содержит метод `locale` для установки желаемого языка. Приложение изменит язык при анализе уведомления, а затем вернется к предыдущему языку, когда анализ будет завершен:

    $user->notify((new InvoicePaid($invoice))->locale('es'));

Локализация нескольких уведомляемых записей также доступна через фасад `Notification`:

    Notification::locale('es')->send(
        $users, new InvoicePaid($invoice)
    );

<a name="user-preferred-locales"></a>
### Предпочитаемые пользователем локализации

Иногда приложения хранят предпочтительный язык каждого пользователя. Реализуя контракт `HasLocalePreference` в вашей уведомляемой модели, вы можете указать Laravel использовать этот сохраненный язык при отправке уведомления:

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

    $user->notify(new InvoicePaid($invoice));

<a name="testing"></a>
## Тестирование

Вы можете использовать метод `fake` фасада `Notification`, чтобы предотвратить отправку уведомлений. Как правило, отправка уведомлений не имеет отношения к коду, который вы фактически тестируете. Вероятно, будет достаточно просто утверждать, что Laravel получил инструкцию отправить определенное уведомление.

После вызова метода `fake` фасада `Notification`, вы можете проверить, было ли передано инструкции отправить уведомления пользователям, и даже проверить данные, полученные уведомлениями:

```php
<?php

namespace Tests\Feature;

use App\Notifications\OrderShipped;
use Illuminate\Support\Facades\Notification;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_orders_can_be_shipped(): void
    {
        Notification::fake();

        // Выполняем доставку заказа...

        // Утверждаем, что уведомления не были отправлены...
        Notification::assertNothingSent();

        // Утверждаем, что уведомление было отправлено указанным пользователям...
        Notification::assertSentTo(
            [$user], OrderShipped::class
        );

        // Утверждаем, что уведомление не было отправлено...
        Notification::assertNotSentTo(
            [$user], AnotherNotification::class
        );

        // Утверждаем, что было отправлено заданное количество уведомлений...
        Notification::assertCount(3);
    }
}
```

Вы можете передать замыкание в методы `assertSentTo` или `assertNotSentTo`, чтобы проверить, что было отправлено уведомление, которое проходит заданный "тест истины". Если хотя бы одно уведомление было отправлено и прошло заданный тест, то утверждение будет успешным:

```php
Notification::assertSentTo(
    $user,
    function (OrderShipped $notification, array $channels) use ($order) {
        return $notification->order->id === $order->id;
    }
);
```

<a name="on-demand-notifications"></a>
#### Уведомления по требованию

Если код, который вы тестируете, отправляет [уведомления по требованию](#on-demand-notifications), вы можете проверить, что уведомление по требованию было отправлено с помощью метода `assertSentOnDemand`:

```php
Notification::assertSentOnDemand(OrderShipped::class);
```

Передав замыкание вторым аргументом метода `assertSentOnDemand`, вы можете определить, отправлено ли уведомление по требованию на правильный "маршрут":

```php
Notification::assertSentOnDemand(
    OrderShipped::class,
    function (OrderShipped $notification, array $channels, object $notifiable) use ($user) {
        return $notifiable->routes['mail'] === $user->email;
    }
);
```

<a name="notification-events"></a>
## События уведомления

<a name="notification-sending-event"></a>
#### Событие отправки уведомления

При отправке уведомлений, система уведомлений запускает [событие](/docs/{{version}}/events) `Illuminate\Notifications\Events\NotificationSending`. Событие содержит уведомляемую сущность и сам экземпляр уведомления. Вы можете зарегистрировать слушателей этого события в поставщике `EventServiceProvider`:

    use App\Listeners\CheckNotificationStatus;
    use Illuminate\Notifications\Events\NotificationSending;

    /**
     * Сопоставление прослушивателя событий для приложения.
     *
     * @var array
     */
    protected $listen = [
        NotificationSending::class => [
            CheckNotificationStatus::class,
        ],
    ];

Уведомление не будет отправлено, если слушатель событий для события `NotificationSending` вернет `false` из своего метода `handle`:

    use Illuminate\Notifications\Events\NotificationSending;

    /**
     * Обработка события.
     */
    public function handle(NotificationSending $event): bool
    {
        return false;
    }

В слушателе событий вы можете получить доступ к свойствам `notifiable`, `notification` и `channel` события, чтобы узнать больше о получателе уведомления или самом уведомлении:

    /**
     * Обработка события.
     */
    public function handle(NotificationSending $event): void
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
    }

<a name="notification-sent-event"></a>
#### Событие после отправки уведомления

Когда уведомление отправлено, система уведомлений запускает [событие](/docs/{{version}}/events) `Illuminate\Notifications\Events\NotificationSent`. Событие содержит уведомляемую сущность и сам экземпляр уведомления. Вы можете зарегистрировать слушателей этого события в поставщике `EventServiceProvider`:

    use App\Listeners\LogNotification;
    use Illuminate\Notifications\Events\NotificationSent;

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        NotificationSent::class => [
            LogNotification::class,
        ],
    ];

> [!NOTE]  
> После регистрации слушателей в вашем `EventServiceProvider` используйте команду `event:generate` Artisan, чтобы быстро сгенерировать классы слушателей.

В слушателе события вы можете получить доступ к свойствам `notifiable`, `notification`, `channel` и `response` события, чтобы узнать больше о получателе уведомления или самом уведомлении:

    /**
     * Обработать переданное событие.
     */
    public function handle(NotificationSent $event): void
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
        // $event->response
    }

<a name="custom-channels"></a>
## Пользовательские каналы уведомлений

Laravel предлагает несколько каналов уведомлений, но вы можете написать свои собственные драйверы для доставки уведомлений по другим каналам. С Laravel это сделать просто. Для начала определите класс, содержащий метод `send`. Этот метод должен получать два аргумента: `$notifiable` и `$notification`.

В методе `send` вы можете вызывать методы уведомления, чтобы получить объект сообщения, понятный вашему каналу, а затем отправить уведомление необходимому экземпляру `$notifiable`:

    <?php

    namespace App\Notifications;

    use Illuminate\Notifications\Notification;

    class VoiceChannel
    {
        /**
         * Отправить переданное уведомление.
         */
        public function send(object $notifiable, Notification $notification): void
        {
            $message = $notification->toVoice($notifiable);

            // Отправка уведомления экземпляру `$notifiable` ...
        }
    }

Как только ваш класс канала уведомления был определен, вы можете вернуть имя класса из метода `via` любого из ваших уведомлений. В этом примере метод вашего уведомления `toVoice` может возвращать любой объект для формирования голосовых сообщений. Например, вы можете определить свой собственный класс `VoiceMessage` для формирования таких сообщений:

    <?php

    namespace App\Notifications;

    use App\Notifications\Messages\VoiceMessage;
    use App\Notifications\VoiceChannel;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Notifications\Notification;

    class InvoicePaid extends Notification
    {
        use Queueable;

        /**
         * Получить каналы доставки уведомлений.
         */
        public function via(object $notifiable): string
        {
            return VoiceChannel::class;
        }

        /**
         * Получить содержимое голосового сообщения.
         */
        public function toVoice(object $notifiable): VoiceMessage
        {
            // ...
        }
    }
