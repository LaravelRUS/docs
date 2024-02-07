---
git: 83761449ff271ccda95e4ea87eca0f5a772f59df
---

# Процессы

<a name="introduction"></a>
## Введение

Laravel предоставляет выразительное, минималистичное API вокруг [компонента Symfony Process](https://symfony.ru/doc/current/components/process.html), что позволяет вам удобно вызывать внешние процессы из вашего приложения Laravel. Возможности работы с процессами в Laravel сосредоточены на наиболее распространенных сценариях использования, обеспечивая отличный опыт разработчика.

<a name="invoking-processes"></a>
## Вызов процессов

Для вызова процесса вы можете использовать методы `run` и `start` предоставленные фасадом `Process` . Метод `run` вызовет процесс и будет ожидать завершения выполнения, в то время как метод `start` используется для асинхронного выполнения процесса. Оба подхода будут рассмотрены в этой документации. Давайте сначала изучим, как вызвать базовый синхронный процесс и проверить его результат:

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

return $result->output();
```

Конечно, экземпляр  `Illuminate\Contracts\Process\ProcessResult` возвращаемый методом `run` предоставляет разнообразие полезных методов, которые можно использовать для анализа результата выполнения процесса:

```php
$result = Process::run('ls -la');

$result->successful();
$result->failed();
$result->exitCode();
$result->output();
$result->errorOutput();
```

<a name="throwing-exceptions"></a>
#### Обработка исключений

Если у вас есть результат выполнения процесса и вы хотите выбросить экземпляр  `Illuminate\Process\Exceptions\ProcessFailedException`, если код завершения больше нуля (что указывает на ошибку), вы можете использовать методы  `throw` и `throwIf`. Если процесс не завершился ошибкой, будет возвращен экземпляр результата процесса:

```php
$result = Process::run('ls -la')->throw();

$result = Process::run('ls -la')->throwIf($condition);
```

<a name="process-options"></a>
### Параметры прощесса

Конечно, возможно, вам понадобится настроить поведение процесса перед его вызовом. К счастью, Laravel позволяет вам настраивать различные характеристики процесса, такие как рабочий каталог, таймаут и переменные окружения.
<a name="working-directory-path"></a>
#### Путь к рабочему каталогу

Вы можете использовать метод  `path`  для указания рабочего каталога процесса. Если этот метод не вызывается, процесс унаследует рабочий каталог текущего выполняющегося скрипта PHP:

```php
$result = Process::path(__DIR__)->run('ls -la');
```

<a name="input"></a>
#### Ввод

Вы можете предоставить ввод через "стандартный ввод" процесса, используя метод `input`:

```php
$result = Process::input('Hello World')->run('cat');
```

<a name="timeouts"></a>
#### Таймаут

По умолчанию процессы будут выбрасывать экземпляр `Illuminate\Process\Exceptions\ProcessTimedOutException` сли выполняются более 60 секунд. Однако вы можете настроить это поведение с помощью метода `timeout`:

```php
$result = Process::timeout(120)->run('bash import.sh');
```

Или, если вы хотите полностью отключить таймаут процесса, вы можете вызвать метод  `forever`:

```php
$result = Process::forever()->run('bash import.sh');
```

Метод `idleTimeout` можно использовать для указания максимального количества секунд, в течение которых процесс может выполняться, не возвращая никакого вывода:

```php
$result = Process::timeout(60)->idleTimeout(30)->run('bash import.sh');
```

<a name="environment-variables"></a>
#### Переменные среды

Переменные среды могут быть предоставлены процессу с помощью метода  `env`. Вызванный процесс также унаследует все переменные среды, определенные в вашей системе:

```php
$result = Process::forever()
            ->env(['IMPORT_PATH' => __DIR__])
            ->run('bash import.sh');
```

Если вы хотите удалить унаследованную переменную среды из вызванного процесса, вы можете предоставить этой переменной среды значение `false`:

```php
$result = Process::forever()
            ->env(['LOAD_PATH' => false])
            ->run('bash import.sh');
```

<a name="tty-mode"></a>
#### Режим TTY

Метод `tty` можно использовать для включения режима TTY для вашего процесса. Режим TTY соединяет ввод и вывод процесса с вводом и выводом вашей программы, что позволяет вашему процессу открывать редактор, такой как Vim или Nano, как процесс:

```php
Process::forever()->tty()->run('vim');
```

<a name="process-output"></a>
### Вывод процесса

Как уже обсуждалось ранее, вывод процесса может быть получен с использованием методов `output` (stdout) и `errorOutput` (stderr)  в результате выполнения процесса:

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

echo $result->output();
echo $result->errorOutput();
```

Однако вывод также можно собрать в реальном времени, передав замыкание в качестве второго аргумента методу  `run` . Замыкание будет получать два аргумента: "тип" вывода  (`stdout` или `stderr`) и сам вывод в виде строки:

```php
$result = Process::run('ls -la', function (string $type, string $output) {
    echo $output;
});
```

Laravel также предлагает методы  `seeInOutput` and `seeInErrorOutput`, которые предоставляют удобный способ определить, содержится ли заданная строка в выводе процесса:

```php
if (Process::run('ls -la')->seeInOutput('laravel')) {
    // ...
}
```

<a name="disabling-process-output"></a>
#### Отключение вывода процесса

Если ваш процесс записывает большое количество вывода, которое вам не интересно, вы можете сэкономить память, полностью отключив получение вывода. Для этого вызовите метод `quietly`  при создании процесса:

```php
use Illuminate\Support\Facades\Process;

$result = Process::quietly()->run('bash import.sh');
```

<a name="process-pipelines"></a>
### Pipelines

Иногда вам может потребоваться передать вывод одного процесса в качестве ввода для другого процесса. Это часто называется "перенаправлением" (piping) ывода одного процесса в другой. Метод `pipe`, предоставляемый фасадом `Process` упрощает это. Метод`pipe` выполнит связанные процессы синхронно и вернет результат последнего процесса в pipeline:

```php
use Illuminate\Process\Pipe;
use Illuminate\Support\Facades\Process;

$result = Process::pipe(function (Pipe $pipe) {
    $pipe->command('cat example.txt');
    $pipe->command('grep -i "laravel"');
});

if ($result->successful()) {
    // ...
}
```

Если вам не нужно настраивать отдельные процессы, составляющие pipeline, вы можете просто передать массив строк команд методу `pipe`:

```php
$result = Process::pipe([
    'cat example.txt',
    'grep -i "laravel"',
]);
```

Вывод процесса можно собрать в реальном времени, передав замыкание в качестве второго аргумента методу `pipe` амыкание будет принимать два аргумента: "тип" вывода (`stdout` или `stderr`) и сам вывод в виде строки:

```php
$result = Process::pipe(function (Pipe $pipe) {
    $pipe->command('cat example.txt');
    $pipe->command('grep -i "laravel"');
}, function (string $type, string $output) {
    echo $output;
});
```

Laravel также позволяет назначать строковые ключи каждому процессу, содержащемуся в pipeline,  с помощью метода `as`. Этот ключ также будет передан в замыкание вывода, предоставленное методу `pipe`, что позволит вам определить, к какому процессу относится вывод:

```php
$result = Process::pipe(function (Pipe $pipe) {
    $pipe->as('first')->command('cat example.txt');
    $pipe->as('second')->command('grep -i "laravel"');
})->start(function (string $type, string $output, string $key) {
    // ...
});
```

<a name="asynchronous-processes"></a>
## Асинхронные процессы

В то время как метод  `run` вызывает процессы синхронно, метод `start` может быть использован для вызова процесса асинхронно. Это позволяет вашему приложению продолжать выполнение других задач, пока процесс выполняется в фоновом режиме. После вызова процесса вы можете использовать метод `running` для определения, выполняется ли процесс:

```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    // ...
}

$result = $process->wait();
```

Как вы могли заметить, вы можете вызвать метод `wait`, чтобы дождаться завершения выполнения процесса и получить экземпляр результата процесса:

```php
$process = Process::timeout(120)->start('bash import.sh');

// ...

$result = $process->wait();
```

<a name="process-ids-and-signals"></a>
### Идентификаторы процессов и сигналы

Метод `id` может быть использован для получения присвоенного операционной системой идентификатора выполняющегося процесса:

```php
$process = Process::start('bash import.sh');

return $process->id();
```

Вы можете использовать метод `signal` для отправки "сигнала" запущенному процессу. Список предопределенных констант сигналов можно найти в [документации по PHP](https://www.php.net/manual/en/pcntl.constants.php):

```php
$process->signal(SIGUSR2);
```

<a name="asynchronous-process-output"></a>
### Вывод асинхронного процесса

Во время выполнения асинхронного процесса вы можете получить доступ к его текущему выводу с помощью методов `output` и `errorOutput`. Однако, для получения вывода процесса, который произошел после последнего вывода, вы можете использовать методы `latestOutput` и `latestErrorOutput`:
```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    echo $process->latestOutput();
    echo $process->latestErrorOutput();

    sleep(1);
}
```

Как и в случае с методом `run`, для асинхронных процессов вывод также можно собирать в реальном времени, передав замыкание вторым аргументом методу `start`. Замыкание будет получать два аргумента: "тип" вывода (`stdout` или `stderr`) и саму строку вывода:

```php
$process = Process::start('bash import.sh', function (string $type, string $output) {
    echo $output;
});

$result = $process->wait();
```

<a name="concurrent-processes"></a>
## Параллельные процессы

Laravel также делает легким управление пулом одновременных асинхронных процессов, что позволяет легко выполнять множество задач параллельно. Для начала используйте метод `pool`, который принимает замыкание, получающее экземпляр `Illuminate\Process\Pool`.

Внутри этого замыкания вы можете определить процессы, принадлежащие пулу. После запуска пула процессов с помощью метода `start` вы можете получить доступ к [коллекции](/docs/{{version}}/collections) запущенных процессов с помощью метода `running`:

```php
use Illuminate\Process\Pool;
use Illuminate\Support\Facades\Process;

$pool = Process::pool(function (Pool $pool) {
    $pool->path(__DIR__)->command('bash import-1.sh');
    $pool->path(__DIR__)->command('bash import-2.sh');
    $pool->path(__DIR__)->command('bash import-3.sh');
})->start(function (string $type, string $output, int $key) {
    // ...
});

while ($pool->running()->isNotEmpty()) {
    // ...
}

$results = $pool->wait();
```

Как видите, вы можете дождаться завершения выполнения всех процессов в пуле и получить их результаты с помощью метода `wait`. Метод `wait` возвращает объект, доступный в виде массива, который позволяет получить экземпляр результата каждого процесса в пуле по его ключу:

```php
$results = $pool->wait();

echo $results[0]->output();
```

Или, для удобства, можно использовать метод `concurrently` для запуска асинхронного пула процессов и немедленного ожидания их результатов. Это может обеспечить особенно выразительный синтаксис при использовании в сочетании с возможностями деструктуризации массивов в PHP:

```php
[$first, $second, $third] = Process::concurrently(function (Pool $pool) {
    $pool->path(__DIR__)->command('ls -la');
    $pool->path(app_path())->command('ls -la');
    $pool->path(storage_path())->command('ls -la');
});

echo $first->output();
```

<a name="naming-pool-processes"></a>
### Именование процессов пула

Доступ к результатам пула процессов по числовому ключу не очень выразителен; поэтому Laravel позволяет вам назначать строковые ключи каждому процессу в пуле с помощью метода `as`. Этот ключ также будет передан замыканию, предоставленному методу `start`, что позволит вам определить, к какому процессу относится вывод:

```php
$pool = Process::pool(function (Pool $pool) {
    $pool->as('first')->command('bash import-1.sh');
    $pool->as('second')->command('bash import-2.sh');
    $pool->as('third')->command('bash import-3.sh');
})->start(function (string $type, string $output, string $key) {
    // ...
});

$results = $pool->wait();

return $results['first']->output();
```

<a name="pool-process-ids-and-signals"></a>
### Идентификаторы и сигналы процессов пула

Поскольку метод `running` пула процессов предоставляет коллекцию всех вызванных процессов внутри пула, вы легко можете получить доступ к идентификаторам процессов в основном пуле:

```php
$processIds = $pool->running()->each->id();
```

И, для удобства, вы можете вызвать метод `signal` пула процессов, чтобы отправить сигнал каждому процессу внутри пула:

```php
$pool->signal(SIGUSR2);
```

<a name="testing"></a>
## Тестирование

Многие службы Laravel предоставляют функциональность для удобного и выразительного написания тестов, и служба процессов Laravel не является исключением. Метод `fake` фасада `Process` позволяет вам указать Laravel возвращать фиктивные / заглушечные результаты при вызове процессов.

<a name="faking-processes"></a>
### Фиктивные процессы

Для исследования возможности фальсификации процессов в Laravel, представим маршрут, который вызывает процесс:

```php
use Illuminate\Support\Facades\Process;
use Illuminate\Support\Facades\Route;

Route::get('/import', function () {
    Process::run('bash import.sh');

    return 'Import complete!';
});
```

При тестировании этого маршрута мы можем указать Laravel вернуть поддельный успешный результат для каждого вызванного процесса, вызвав метод `fake` на фасаде `Process` без аргументов. Кроме того, мы даже можем [проверить](#available-assertions), что определенный процесс был "запущен":
```php
<?php

namespace Tests\Feature;

use Illuminate\Process\PendingProcess;
use Illuminate\Contracts\Process\ProcessResult;
use Illuminate\Support\Facades\Process;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_process_is_invoked(): void
    {
        Process::fake();

        $response = $this->get('/import');

        // Simple process assertion...
        Process::assertRan('bash import.sh');

        // Or, inspecting the process configuration...
        Process::assertRan(function (PendingProcess $process, ProcessResult $result) {
            return $process->command === 'bash import.sh' &&
                   $process->timeout === 60;
        });
    }
}
```

Как обсуждалось, вызов метода `fake` фасада `Process` указывает Laravel всегда возвращать успешный результат процесса без вывода. Тем не менее, вы легко можете указать вывод и код завершения для поддельных процессов с использованием метода `result` фасада `Process`:

```php
Process::fake([
    '*' => Process::result(
        output: 'Test output',
        errorOutput: 'Test error output',
        exitCode: 1,
    ),
]);
```

<a name="faking-specific-processes"></a>
### Фальсификация определенных процессов

Как вы могли заметить в предыдущем примере, фасад `Process` позволяет вам указывать различные поддельные результаты для каждого процесса, передав массив методу `fake`.

Ключи массива должны представлять шаблоны команд, которые вы хотите подделать, и их  результаты. Символ `*` может быть использован в качестве символа-заменителя. Любые команды процессов, которые не были подделаны, будут действительно вызваны. Вы можете использовать метод `result` фасада `Process` для создания заглушек / фейковых результатов для этих команд:
```php
Process::fake([
    'cat *' => Process::result(
        output: 'Test "cat" output',
    ),
    'ls *' => Process::result(
        output: 'Test "ls" output',
    ),
]);
```

Если вам не нужно настраивать код завершения или вывод ошибок поддельного процесса, вам может быть удобнее указывать результаты фейкового процесса в виде простых строк:
```php
Process::fake([
    'cat *' => 'Test "cat" output',
    'ls *' => 'Test "ls" output',
]);
```

<a name="faking-process-sequences"></a>
### Подделка последовательности процессов

Если код, который вы тестируете, вызывает несколько процессов с одной и той же командой, вы можете назначить различные фейковые результаты каждому вызову процесса. Вы можете сделать это с помощью метода `sequence` фасада `Process`:
```php
Process::fake([
    'ls *' => Process::sequence()
                ->push(Process::result('First invocation'))
                ->push(Process::result('Second invocation')),
]);
```

<a name="faking-asynchronous-process-lifecycles"></a>
### Имитация жизненного цикла асинхронных процессов

До сих пор мы в основном говорили о фейковых процессах, которые вызываются синхронно с использованием метода `run`. Однако, если вы пытаетесь протестировать код, который взаимодействует с асинхронными процессами, вызываемыми с помощью `start`, вам может потребоваться более сложный подход к описанию ваших фейковых процессов.

Например, представим следующий маршрут, который взаимодействует с асинхронным процессом:

```php
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Route;

Route::get('/import', function () {
    $process = Process::start('bash import.sh');

    while ($process->running()) {
        Log::info($process->latestOutput());
        Log::info($process->latestErrorOutput());
    }

    return 'Done';
});
```

To properly fake this process, we need to be able to describe how many times the `running` method should return `true`. In addition, we may want to specify multiple lines of output that should be returned in sequence. To accomplish this, we can use the `Process` facade's `describe` method:

```php
Process::fake([
    'bash import.sh' => Process::describe()
            ->output('First line of standard output')
            ->errorOutput('First line of error output')
            ->output('Second line of standard output')
            ->exitCode(0)
            ->iterations(3),
]);
```

Чтобы корректно подделать этот процесс, нам нужно иметь возможность описать, сколько раз метод `running` должен возвращать `true`. Кроме того, по желанию, мы можем указать несколько строк вывода, которые должны быть возвращены последовательно. Для этого мы можем использовать метод `describe` фасада `Process`:

<a name="available-assertions"></a>
### Доступные утверждения

Как [уже обсуждалось ранее](#faking-processes), Laravel предоставляет несколько утверждений процессов для ваших функциональных тестов. Рассмотрим каждое из этих утверждений.

<a name="assert-process-ran"></a>
#### assertRan

Утверждение, что определенный процесс был вызван:

```php
use Illuminate\Support\Facades\Process;

Process::assertRan('ls -la');
```

Метод `assertRan` также принимает замыкание, которое получит экземпляр процесса и результат процесса, что позволяет вам проверить настроенные опции процесса. Если это замыкание возвращает `true`, утверждение будет "пройдено":

```php
Process::assertRan(fn ($process, $result) =>
    $process->command === 'ls -la' &&
    $process->path === __DIR__ &&
    $process->timeout === 60
);
```

Переменная `$process`, переданная в замыкание `assertRan`, является экземпляром `Illuminate\Process\PendingProcess`, в то время как `$result` - экземпляром `Illuminate\Contracts\Process\ProcessResult`.

<a name="assert-process-didnt-run"></a>
#### assertDidntRun

Утверждение, что определенный процесс не был вызван:

```php
use Illuminate\Support\Facades\Process;

Process::assertDidntRun('ls -la');
```

Как и метод `assertRan`, метод `assertDidntRun` также принимает замыкание, которое получит экземпляр процесса и результат процесса, что позволяет вам проверить настроенные опции процесса. Если это замыкание возвращает `true`, утверждение будет "провалено":

```php
Process::assertDidntRun(fn (PendingProcess $process, ProcessResult $result) =>
    $process->command === 'ls -la'
);
```

<a name="assert-process-ran-times"></a>
#### assertRanTimes

Утверждение, что определенный процесс был вызван определенное количество раз:

```php
use Illuminate\Support\Facades\Process;

Process::assertRanTimes('ls -la', times: 3);
```

Метод `assertRanTimes` также принимает замыкание, которое получит экземпляр процесса и результат процесса, что позволяет вам проверить настроенные опции процесса. Если это замыкание возвращает `true` и процесс был вызван указанное количество раз, утверждение будет "пройдено":
```php
Process::assertRanTimes(function (PendingProcess $process, ProcessResult $result) {
    return $process->command === 'ls -la';
}, times: 3);
```

<a name="preventing-stray-processes"></a>
### Предотвращение случайных процессов

Если вы хотите убедиться, что все вызванные процессы были подделаны в пределах отдельного теста или набора тестов, вы можете вызвать метод `preventStrayProcesses`. После вызова этого метода любые процессы, для которых нет соответствующего поддельного результата, вызовут исключение, а не фактический процесс:

    use Illuminate\Support\Facades\Process;

    Process::preventStrayProcesses();

    Process::fake([
        'ls *' => 'Test output...',
    ]);

    // Fake response is returned...
    Process::run('ls -la');

    // An exception is thrown...
    Process::run('bash import.sh');
