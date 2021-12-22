git 6bc580908dfe05fc9cba1dabdb42868d1bf633a3

---

# Ограничение скорости

- [Введение](#introduction)
    - [Конфигурация кеша](#cache-configuration)
- [Базовое использование](#basic-usage)
    - [Ручное увеличение числа попыток](#manually-incrementing-attempts)
    - [Очистка счетчика попыток](#clearing-attempts)

<a name="introduction"></a>
## Введение

Laravel включает простую в использовании абстракцию ограничения скорости, которая в сочетании с [кешем](/docs/{{version}}/cache) вашего приложения обеспечивает простой способ ограничить любое действие в течение указанного периода времени.

> {tip} Если вас интересует ограничение скорости входящих HTTP-запросов, обратитесь к документации [посредника для ограничения частоты запросов](/docs/{{version}}/routing#rate-limiting).

<a name="cache-configuration"></a>
### Конфигурация кеша

Обычно ограничитель скорости использует кеш вашего приложения по умолчанию, как определено ключом `default` в файле конфигурации `cache` вашего приложения. Однако вы можете указать, какой драйвер кеша должен использовать ограничитель скорости, задав ключ `limiter` в файле конфигурации `cache` вашего приложения:

    'default' => 'memcached',

    'limiter' => 'redis',

<a name="basic-usage"></a>
## Базовое использование

Фасад `Illuminate\Support\Facades\RateLimiter` может использоваться для взаимодействия с ограничителем скорости. Самый простой метод, предлагаемый ограничителем скорости - это метод `attempt` который ограничивает скорость данного обратного вызова на заданное количество секунд.

Метод `attempt` возвращает `false` если для обратного вызова не осталось доступных попыток; в противном случае метод `attempt` вернет результат обратного вызова или `true`. Первым аргументом, принимаемым методом `attempt` является «ключ» ограничителя скорости, который может быть любой строкой по вашему выбору, представляющей действие с ограничением скорости:

    use Illuminate\Support\Facades\RateLimiter;
    
    $executed = RateLimiter::attempt(
        'send-message:'.$user->id,
        $perMinute = 5,
        function() {
            // Send message...
        }
    );
    
    if (! $executed) {
      return 'Too many messages sent!';
    }

<a name="manually-incrementing-attempts"></a>
### Ручное увеличение числа попыток

Если вы хотите вручную взаимодействовать с ограничителем скорости, доступно множество других методов. Например, вы можете вызвать метод `tooManyAttempts`, чтобы определить, не превысил ли заданный ключ ограничителя скорости максимальное количество разрешенных попыток в минуту:

    use Illuminate\Support\Facades\RateLimiter;
    
    if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
        return 'Too many attempts!';
    }

В качестве альтернативы вы можете использовать метод `remaining` для получения количества попыток, оставшихся для данного ключа. Если для данного ключа остались повторные попытки, вы можете вызвать метод `hit`, чтобы увеличить общее количество попыток:

    use Illuminate\Support\Facades\RateLimiter;
    
    if (RateLimiter::remaining('send-message:'.$user->id, $perMinute = 5)) {
        RateLimiter::hit('send-message:'.$user->id);

        // Send message...
    }

<a name="determining-limiter-availability"></a>
#### Определение доступности ограничителя скорости

Когда у ключа больше не осталось попыток, метод `availableIn` возвращает количество секунд, оставшихся до тех пор, пока не будут доступны новые попытки:

    use Illuminate\Support\Facades\RateLimiter;

    if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
        $seconds = RateLimiter::availableIn('send-message:'.$user->id);

        return 'You may try again in '.$seconds.' seconds.';
    }

<a name="clearing-attempts"></a>
### Очистка счетчика попыток

Вы можете сбросить количество попыток для данного ключа ограничителя скорости, используя метод `clear`. Например, вы можете сбросить количество попыток, когда данное сообщение прочитано получателем:

    use App\Models\Message;
    use Illuminate\Support\Facades\RateLimiter;

    /**
     * Отметьте сообщение как прочитанное.
     *
     * @param  \App\Models\Message  $message
     * @return \App\Models\Message
     */
    public function read(Message $message)
    {
        $message->markAsRead();
        
        RateLimiter::clear('send-message:'.$message->user_id);
        
        return $message;
    }
