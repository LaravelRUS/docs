---
git: 46c2634ef5a4f15427c94a3157b626cf5bd3937f
---

# Тестирование · Имитация


<a name="introduction"></a>
## Введение

При тестировании приложений Laravel бывает необходимо «сымитировать» определенные аспекты вашего приложения, чтобы они фактически не выполнялись во время текущего теста. Например, при тестировании контроллера, который инициирует событие, вы можете смоделировать слушателей событий, чтобы они фактически не выполнялись во время теста. Это позволяет вам тестировать только HTTP-ответ контроллера, не беспокоясь о запуске слушателей событий, поскольку слушатели событий могут быть протестированы в их собственном тестовом классе.

Laravel предлагает полезные методы для имитации событий, заданий и других фасадов из коробки. Эти помощники в первую очередь обеспечивают удобную обертку над Mockery, поэтому вам не нужно вручную выполнять сложные вызовы методов Mockery.

<a name="mocking-objects"></a>
## Подставные объекты

При имитации объекта, который будет внедрен в ваше приложение через [контейнер служб](/docs/{{version}}/container) Laravel, вам нужно будет привязать ваш подставной экземпляр к контейнеру с помощью `instance`. Это даст указание контейнеру использовать ваш подставной экземпляр объекта вместо создания самого объекта:

    use App\Service;
    use Mockery;
    use Mockery\MockInterface;

    public function test_something_can_be_mocked(): void
    {
        $this->instance(
            Service::class,
            Mockery::mock(Service::class, function (MockInterface $mock) {
                $mock->shouldReceive('process')->once();
            })
        );
    }

Чтобы сделать это более удобным, вы можете использовать метод `mock`, который обеспечен базовым классом тестов Laravel. Например, следующий пример эквивалентен приведенному выше примеру:

    use App\Service;
    use Mockery\MockInterface;

    $mock = $this->mock(Service::class, function (MockInterface $mock) {
        $mock->shouldReceive('process')->once();
    });

Вы можете использовать метод `partialMock`, когда вам нужно только имитировать несколько методов объекта. Методы, которые не являются сымитированными, при вызове будут выполняться в обычном режиме:

    use App\Service;
    use Mockery\MockInterface;

    $mock = $this->partialMock(Service::class, function (MockInterface $mock) {
        $mock->shouldReceive('process')->once();
    });

Точно так же, если вы хотите [шпионить](http://docs.mockery.io/en/latest/reference/spies.html) за объектом, базовый класс тестов Laravel содержит метод `spy` в качестве удобной обертки для метода `Mockery::spy`. Шпионы похожи на подставные объекты; однако, шпионы записывают любое взаимодействие между шпионом и тестируемым кодом, позволяя вам делать утверждения после выполнения кода:

    use App\Service;

    $spy = $this->spy(Service::class);

    // ...

    $spy->shouldHaveReceived('process');

<a name="mocking-facades"></a>
## Имитация фасадов

В отличие от традиционных вызовов статических методов, [фасады](/docs/{{version}}/facades), включая [фасады в реальном времени](/docs/{{version}}/facades#real-time-facades) можно имитировать. Это дает большое преимущество перед традиционными статическими методами и дает вам такую же возможность тестирования, как если бы вы использовали традиционное внедрение зависимостей. При тестировании вы часто можете имитировать вызов фасада Laravel, происходящий в одном из ваших контроллеров. Например, рассмотрим следующее действие контроллера:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Получить список всех пользователей приложения.
         */
        public function index(): array
        {
            $value = Cache::get('key');

            return [
                // ...
            ];
        }
    }

Мы можем имитировать вызов фасада `Cache`, используя метод `shouldReceive`, который вернет экземпляр [Mockery](https://github.com/padraic/mockery). Поскольку фасады фактически извлекаются и управляются [контейнером служб](/docs/{{version}}/container) Laravel, то они имеют гораздо большую тестируемость, чем типичный статический класс. Например, давайте сымитируем наш вызов метода `get` фасада `Cache`:

    <?php

    namespace Tests\Feature;

    use Illuminate\Support\Facades\Cache;
    use Tests\TestCase;

    class UserControllerTest extends TestCase
    {
        public function test_get_index(): void
        {
            Cache::shouldReceive('get')
                        ->once()
                        ->with('key')
                        ->andReturn('value');

            $response = $this->get('/users');

            // ...
        }
    }

> [!WARNING]
> Вы не должны имитировать фасад `Request`. Вместо этого передайте требуемые данные в [методы тестирования HTTP](http-tests), такие как `get` и `post`, при запуске вашего теста. Аналогично, вместо имитации фасада `Config`, вызовите метод `Config::set` в ваших тестах.

<a name="facade-spies"></a>
### Шпионы фасадов

Если вы хотите [шпионить](http://docs.mockery.io/en/latest/reference/spies.html) за фасадом, то вы можете вызвать метод `spy` на соответствующем фасаде. Шпионы похожи на подставные объекты; однако, шпионы записывают любое взаимодействие между шпионом и тестируемым кодом, позволяя вам делать утверждения после выполнения кода:

    use Illuminate\Support\Facades\Cache;

    public function test_values_are_be_stored_in_cache(): void
    {
        Cache::spy();

        $response = $this->get('/');

        $response->assertStatus(200);

        Cache::shouldHaveReceived('put')->once()->with('name', 'Taylor', 10);
    }

<a name="interacting-with-time"></a>
## Взаимодействие со временем

При тестировании вам может иногда потребоваться изменить время, возвращаемое такими помощниками, как `now` или `Illuminate\Support\Carbon::now()`. К счастью, базовый класс тестирования функций Laravel включает помощников, которые позволяют вам управлять текущим временем:

    use Illuminate\Support\Carbon;

    public function test_time_can_be_manipulated(): void
    {
        // Путешествие в будущее ...
        $this->travel(5)->milliseconds();
        $this->travel(5)->seconds();
        $this->travel(5)->minutes();
        $this->travel(5)->hours();
        $this->travel(5)->days();
        $this->travel(5)->weeks();
        $this->travel(5)->years();

        // Путешествие в прошлое ...
        $this->travel(-5)->hours();

        // Путешествие в определенное время ...
        $this->travelTo(now()->subHours(6));

        // Вернуться в настоящее время ...
        $this->travelBack();
    }

Вы также можете предоставить замыкание для различных методов путешествия во времени. Замыкание будет вызвано с замороженным временем в указанное время. После выполнения замыкания время возобновится как обычно:

    $this->travel(5)->days(function () {
        // Test something five days into the future...
    });

    $this->travelTo(now()->subDays(10), function () {
        // Test something during a given moment...
    });

Метод `freezeTime` может быть использован для замораживания текущего времени. Аналогично, метод `freezeSecond` заморозит текущее время, но в начале текущей секунды:

    use Illuminate\Support\Carbon;

    // Freeze time and resume normal time after executing closure...
    $this->freezeTime(function (Carbon $time) {
        // ...
    });

    // Freeze time at the current second and resume normal time after executing closure...
    $this->freezeSecond(function (Carbon $time) {
        // ...
    })

Как и ожидалось, все обсуждаемые выше методы в основном полезны для тестирования поведения приложения, зависящего от времени, такого как блокировка неактивных сообщений на форуме:

    use App\Models\Thread;

    public function test_forum_threads_lock_after_one_week_of_inactivity()
    {
        $thread = Thread::factory()->create();
        
        $this->travel(1)->week();
        
        $this->assertTrue($thread->isLockedByInactivity());
    }
