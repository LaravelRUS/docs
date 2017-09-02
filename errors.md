git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Ошибки и логирование

- [Введение](#introduction)
- [Настройка](#configuration)
    - [Детализация ошибок](#error-detail)
    - [Хранилище логов](#log-storage)
    - [Коды серьёзности логов](#log-severity-levels)
    - [Пользовательская настройка Monolog](#custom-monolog-configuration)
- [Обработчик исключений](#the-exception-handler)
    - [Метод Report](#report-method)
    - [Метод Render](#render-method)
- [HTTP-исключения](#http-exceptions)
    - [Пользовательские страницы HTTP-ошибок](#custom-http-error-pages)
- [Логгирование](#logging)

<a name="introduction"></a>
## Введение

Когда вы начинаете новый Laravel проект, обработка ошибок и исключений уже настроена для вас. Все происходящие в вашем приложении исключения записываются в журнал и отображаются пользователю в классе `App\Exceptions\Handler`. В этой документации мы подробно рассмотрим этот класс.

Для логгирования Laravel использует библиотеку [Monolog](https://github.com/Seldaek/monolog), которая обеспечивает поддержку различных мощных обработчиков логов. В Laravel настроены несколько из них, благодаря чему вы можете выбрать между единым файлом журнала, ротируемыми файлами журналов и записью информации в системный журнал.

<a name="configuration"></a>
## Настройка

<a name="error-detail"></a>
### Детализация ошибок

Параметр `debug` в файле настроек `config/app.php` определяет, сколько информации об ошибке показывать пользователю. По умолчанию этот параметр установлен в соответствии со значением переменной среды `APP_DEBUG`, которая хранится в файле `.env`.

Для локальной разработки вам следует установить переменную среды `APP_DEBUG` в значение `true`. В продакшн-среде эта переменная всегда должна иметь значение `false`. Если значение равно `true`, на продакшн-сервере, вы рискуете раскрыть важные значения настроек вашим конечным пользователям.

<a name="log-storage"></a>
### Хранилище логов

Изначально Laravel поддерживает запись журналов в единые файлы `single`, в отдельные файлы для каждого дня `daily`, в `syslog` и `errorlog`. Для использования определённого механизма хранения вам надо изменить параметр `log` в файле `config/app.php`. Например, если вы хотите использовать ежедневные файлы логов вместо единого файла, вам надо установить значение `log` равное `daily` в файле настроек `app`:

    'log' => 'daily'

#### Максимальное число ежедневных файлов логов

При использовании режима `daily` Laravel по умолчанию хранит логи только за последние пять дней. Если вы хотите изменить число хранимых файлов, добавьте в файл `app` значение для параметра `log_max_files`:

    'log_max_files' => 30

<a name="log-severity-levels"></a>
### Коды серьёзности логов

При использовании Monolog сообщения в журнале могут иметь разные уровни серьёзности. По умолчанию Laravel сохраняет в журнал события всех уровней. Но на продакшн-сервере вы можете задать минимальный уровень серьёзности, который необходимо заносить в журнал, добавив параметр `log_level` в конфиг `app.php`.

После задания этого параметра Laravel будет записывать события всех уровней начиная с указанного и выше. Например, при `log_level` равном `error` будут записываться события **error**, **critical**, **alert** и **emergency**:

    'log_level' => env('APP_LOG_LEVEL', 'error'),

> {tip} В Monolog используются следующие уровни серьёзности — от меньшего к большему: `debug`, `info`, `notice`, `warning`, `error`, `critical`, `alert`, `emergency`.

<a name="custom-monolog-configuration"></a>
### Пользовательская настройка Monolog

Если вы хотите иметь полный контроль над конфигурацией Monolog для вашего приложения, вы можете использовать метод приложения `configureMonologUsing`. Вызов этого метода необходимо поместить в файл `bootstrap/app.php` прямо перед тем, как в нём возвращается переменная `$app`:

    $app->configureMonologUsing(function ($monolog) {
        $monolog->pushHandler(...);
    });

    return $app;

<a name="the-exception-handler"></a>
## Обработчик исключений

<a name="report-method"></a>
### Метод Report

Все исключения обрабатываются классом `App\Exceptions\Handler`. Этот класс содержит два метода: `report` и `render`. Рассмотрим каждый из них подробнее. Метод `report` используется для занесения исключений в журнал или для отправки их во внешний сервис, такой как [Bugsnag](https://bugsnag.com) или [Sentry](https://github.com/getsentry/sentry-laravel). По умолчанию метод `report` просто передаёт исключение в базовую реализацию родительского класса, где это исключение зафиксировано. Но вы можете регистрировать исключения как пожелаете.

Например, если вам необходимо сообщать о различных типах исключений разными способами, вы можете использовать оператор сравнения PHP `instanceof`:

    /**
     * Сообщить или зарегистрировать исключение.
     *
     * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
     *
     * @param  \Exception  $exception
     * @return void
     */
    public function report(Exception $exception)
    {
        if ($exception instanceof CustomException) {
            //
        }

        return parent::report($exception);
    }

#### Игнорирование исключений заданного типа

Свойство обработчика исключений `$dontReport` содержит массив с типами исключений, которые не будут заноситься в журнал. Например, исключения, возникающие при ошибке 404, а также при некоторых других типах ошибок, не записываются в журналы. При необходимости вы можете включить другие типы исключений в этот массив:

    /**
     * Список типов исключений, о которых не надо сообщать.
     *
     * @var array
     */
    protected $dontReport = [
        \Illuminate\Auth\AuthenticationException::class,
        \Illuminate\Auth\Access\AuthorizationException::class,
        \Symfony\Component\HttpKernel\Exception\HttpException::class,
        \Illuminate\Database\Eloquent\ModelNotFoundException::class,
        \Illuminate\Validation\ValidationException::class,
    ];

<a name="render-method"></a>
### Метод Render

Метод `render`отвечает за конвертацию исключения в HTTP-отклик, который должен быть возвращён браузеру. По умолчанию исключение передаётся в базовый класс, который генерирует для вас отклик. Но вы можете проверить тип исключения или вернуть ваш собственный отклик:

    /**
     * Отображение HTTP-оклика для исключения.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof CustomException) {
            return response()->view('errors.custom', [], 500);
        }

        return parent::render($request, $exception);
    }

<a name="http-exceptions"></a>
## HTTP-исключения

Некоторые исключения описывают коды HTTP-ошибок от сервера. Например, это может быть ошибка "страница не найдена" (404), "ошибка авторизации" (401) или даже сгенерированная разработчиком ошибка 500. Для возврата такого отклика из любого места в приложении можете использовать хелпер `abort`:

    abort(404);

Хелпер `abort` немедленно создаёт исключение, которое будет отрисовано обработчиком исключений. Или вы можете предоставить такой отклик:

    abort(403, 'Unauthorized action.');

<a name="custom-http-error-pages"></a>
### Пользовательские страницы HTTP-ошибок

В Laravel можно легко возвращать свои собственные страницы для различных кодов HTTP-ошибок. Например, для выдачи собственной страницы для ошибки 404 создайте файл `resources/views/errors/404.blade.php`. Этот файл будет использован для всех ошибок 404, генерируемых вашим приложением. Представления в этой папке должны иметь имена, соответствующие кодам ошибок. Экземпляр `HttpException`, созданный функцией `abort`, будет передан в представление как переменная `$exception`:

    <h2>{{ $exception->getMessage() }}</h2>

<a name="logging"></a>
## Логгирование

Laravel обеспечивает простой простой уровень абстракции над мощной библиотекой [Monolog](https://github.com/seldaek/monolog). По умолчанию Laravel настроен на создание файла журнала в директории `storage/logs`. Вы можете записывать информацию в журнал при помощи [фасада](/docs/{{version}}/facades) `Log`:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Support\Facades\Log;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Показать профиль данного пользователя.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            Log::info('Showing user profile for user: '.$id);

            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

Регистратор событий предоставляет восемь уровней логгирования, как описано в [RFC 5424](https://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** и **debug**.

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

#### Контекстная информация

Также в методы логгирования можно передать массив контекстных данных:

    Log::info('User failed to login.', ['id' => $user->id]);

#### Обращение к расположенному ниже экземпляру Monolog

В Monolog доступно множество дополнительных обработчиков для журналов. При необходимости вы можете получить доступ к расположенному ниже экземпляру Monolog, используемому в Laravel:

    $monolog = Log::getMonolog();
