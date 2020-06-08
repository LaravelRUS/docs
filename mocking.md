git 5fbabacd2b9ad49a062ea276a5f238df07bb80cb

---

# Мокинг (имитация)

- [Введение](#introduction)
- [Имитирование Bus](#bus-fake)
- [Имитирование Event](#event-fake)
- [Имитирование Mail](#mail-fake)
- [Имитирование Notification](#notification-fake)
- [Имитирование Queue](#queue-fake)
- [Имитирование Storage](#storage-fake)
- [Фасады](#mocking-facades)

<a name="introduction"></a>
## Введение

При тестировании приложений Laravel вам может потребоваться "сымитировать" определенные аспекты ваших приложений, так чтобы они по факту не выполнялись во время теста. Например, при тестировании контроллера, который распределяет событие, вы можете захотеть использовать моки (имитацию) слушателей событий, чтобы они не выполнялись во время этого теста. Это позволит проверить только HTTP-ответ контроллера, не волнуясь о выполнении слушателей событий, так как слушателей событий можно проверить в их собственных тест-кейсах.

Laravel изначально предоставляет хелперов для имитации событий, задач и фасадов. Эти хелперы в основном служат удобным слоем поверх Mockery, чтобы вам не пришлось вручную вызывать сложные методы Mockery. Конечно, для этой же цели все еще можно использовать [Mockery](http://docs.mockery.io/en/latest/) или PHPUnit.

<a name="bus-fake"></a>
## Имитирование Bus

В качестве альтернативы мокингу (имитации), можно использовать метод `fake` фасада `Bus`, чтобы предотвратить выполнение задач. При использовании имитаций, после выполнения кода теста делаются утверждения:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Jobs\ShipOrder;
    use Illuminate\Support\Facades\Bus;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Bus::fake();

            // Perform order shipping...

            Bus::assertDispatched(ShipOrder::class, function ($job) use ($order) {
                return $job->order->id === $order->id;
            });

            // Assert a job was not dispatched...
            Bus::assertNotDispatched(AnotherJob::class);
        }
    }

<a name="event-fake"></a>
## Имитирование Event

В качестве альтернативы мокингу (имитации), можно использовать метод `fake` фасада `Event`, чтобы предотвратить выполнение слушателей. Затем можно утверждать, что события были отправлены и даже проверить данные, которые они получили. При использовании имитаций, после выполнения кода теста делаются утверждения:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Events\OrderShipped;
    use App\Events\OrderFailedToShip;
    use Illuminate\Support\Facades\Event;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        /**
         * Test order shipping.
         */
        public function testOrderShipping()
        {
            Event::fake();

            // Perform order shipping...

            Event::assertDispatched(OrderShipped::class, function ($e) use ($order) {
                return $e->order->id === $order->id;
            });

            Event::assertNotDispatched(OrderFailedToShip::class);
        }
    }

<a name="mail-fake"></a>
## Имитирование Mail

Можно использовать метод `fake` фасада `Mail` для предоствращения отправки почты. Затем можно утверждать, что [mailables](/docs/{{version}}/mail) были отправлены пользователям и даже проверить данные, которые они получили. При использовании имитаций, после выполнения кода теста делаются утверждения:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Mail\OrderShipped;
    use Illuminate\Support\Facades\Mail;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Mail::fake();

            // Perform order shipping...

            Mail::assertSent(OrderShipped::class, function ($mail) use ($order) {
                return $mail->order->id === $order->id;
            });

            // Assert a message was sent to the given users...
            Mail::assertSent(OrderShipped::class, function ($mail) use ($user) {
                return $mail->hasTo($user->email) &&
                       $mail->hasCc('...') &&
                       $mail->hasBcc('...');
            });

            // Assert a mailable was not sent...
            Mail::assertNotSent(AnotherMailable::class);
        }
    }

<a name="notification-fake"></a>
## Имитирование Notification

Можно использовать метод `fake` фасада `Notification` для предоствращения отправки уведомлений. Вы можете утверждать, что [уведомления](/docs/{{version}}/notifications) были отправлены и даже инспектировать данные, которые они получили. При использовании имитаций, после выполнения кода теста делаются утверждения:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Notifications\OrderShipped;
    use Illuminate\Support\Facades\Notification;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Notification::fake();

            // Perform order shipping...

            Notification::assertSentTo(
                $user,
                OrderShipped::class,
                function ($notification, $channels) use ($order) {
                    return $notification->order->id === $order->id;
                }
            );

            // Assert a notification was sent to the given users...
            Notification::assertSentTo(
                [$user], OrderShipped::class
            );

            // Assert a notification was not sent...
            Notification::assertNotSentTo(
                [$user], AnotherNotification::class
            );
        }
    }

<a name="queue-fake"></a>
## Имитирование Queue

В качестве альтернативы мокингу (имитации), можно использовать метод `fake` фасада `Queue` для предоствращения помещения задач в очередь. Вы можете утверждать, что задачи были помещены в очередь и даже инспектировать полученные ими данные. При использовании имитаций, после выполнения кода теста делаются утверждения:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Jobs\ShipOrder;
    use Illuminate\Support\Facades\Queue;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Queue::fake();

            // Perform order shipping...

            Queue::assertPushed(ShipOrder::class, function ($job) use ($order) {
                return $job->order->id === $order->id;
            });

            // Assert a job was pushed to a given queue...
            Queue::assertPushedOn('queue-name', ShipOrder::class);

            // Assert a job was not pushed...
            Queue::assertNotPushed(AnotherJob::class);
        }
    }

<a name="storage-fake"></a>
## Имитирование Storage

Метод `fake` фасада `Storage` позволяет запросто генерировать фейковый диск, который, в комбинации с утилитами генерации файлов класса `UploadedFile`, значительно упрощает тестирование загрузок файлов. Например:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        public function testAvatarUpload()
        {
            Storage::fake('avatars');

            $response = $this->json('POST', '/avatar', [
                'avatar' => UploadedFile::fake()->image('avatar.jpg')
            ]);

            // Assert the file was stored...
            Storage::disk('avatars')->assertExists('avatar.jpg');

            // Assert a file does not exist...
            Storage::disk('avatars')->assertMissing('missing.jpg');
        }
    }

> {tip} По умолчанию метод `fake` удалит все файлы в своей временной директории. Если вам нужно сохранить эти файлы, вместо этого можно использовать метод "persistentFake".

<a name="mocking-facades"></a>
## Фасады

В отличие от традиционных вызовов статических методов, [фасады](/docs/{{version}}/facades) можно сымитировать. Это предоставляет большое преимущество над традиционными статическими методами и предоставляет ту же тестируемость, какая была бы у вас при использовании внедрений зависимостей. Во время тестирования вам часто может понадобиться имитировать вызов к фасаду Laravel в одном из своих контроллеров. Например, рассмотрите следующее действие контроллера:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

Мы можем сымитировать вызов к фасаду `Cache`, используя метод `shouldReceive`, который вернет экземпляр мока [Mockery](https://github.com/padraic/mockery). Так как фасады фактически разрешаются и управляются [сервис контейнером](/docs/{{version}}/container) Laravel, у них куда больше тестируемости, чем у типичного статического класса. Например, давайте сымитируем наш вызов к методу `get` фасада `Cache`:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class UserControllerTest extends TestCase
    {
        public function testGetIndex()
        {
            Cache::shouldReceive('get')
                        ->once()
                        ->with('key')
                        ->andReturn('value');

            $response = $this->get('/users');

            // ...
        }
    }

> {note} Не следует имитировать фасад `Request`. Вместо этого передайте желаемый ввод HTTP методам-хелперам, таким как `get` и `post` во время выполнения своего теста. Схожим образом, вместо имитирования фасада `Config` просто вызовите метод `Config::set` в своих тестах.
