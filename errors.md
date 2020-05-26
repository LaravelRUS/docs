git 861a15a58db44cda432d3c6a357fc5304c9418c0

---

# Обработка ошибок

- [Введение](#introduction)
- [Настройка](#configuration)
- [Обработчик исключений](#the-exception-handler)
    - [Метод Report](#report-method)
    - [Метод Render](#render-method)
    - [Reportable и Renderable исключения](#renderable-exceptions)
- [HTTP-исключения](#http-exceptions)
    - [Пользовательские страницы HTTP-ошибок](#custom-http-error-pages)

<a name="introduction"></a>
## Introduction

В Laravel обработка исключений ведётся классом `App\Exceptions\Handler`. Возникшие исключения логируются и при необходимости показываются пользователю. Давайте рассмотрим работу этого класса поподробнее.

<a name="configuration"></a>
## Настройка

Опция `debug` в `config/app.php` определяет, показывать ли информацию об ошибке пользователю. По умолчанию значение берётся из `APP_DEBUG`, определённого в `.env` файле.

В процессе разработки `APP_DEBUG` удобно установить в `true`. На сервере, в продакшне, это значение логично ставить в `false`, чтобы в сообщении об ошибке случайно не показать чувствительные для безопасности проекта вещи.

<a name="the-exception-handler"></a>
## Обработчик исключений

Все исключения обрабатываются классом `App\Exceptions\Handler`. Этот класс содержит два метода: `report` и `render`. Рассмотрим каждый из них подробнее. 

<a name="report-method"></a>
### Метод Report

Метод `report` используется для логирование исключений или для отправки их во внешний сервис, такой как [Flare](https://flareapp.io), [Bugsnag](https://bugsnag.com) или [Sentry](https://github.com/getsentry/sentry-laravel). По умолчанию метод `report` просто передаёт исключение в базовую реализацию родительского класса, где это происходит запись исключения в лог-файл. Но вы можете регистрировать исключения как пожелаете. 

Например, если вам необходимо сообщать о различных типах исключений разными способами, вы можете использовать оператор сравнения PHP `instanceof`:

    /**
     * Report or log an exception.
     *
     * This is a great spot to send exceptions to Flare, Sentry, Bugsnag, etc.
     *
     * @param  \Exception  $exception
     * @return void
     */
    public function report(Exception $exception)
    {
        if ($exception instanceof CustomException) {
            //
        }

        parent::report($exception);
    }

> {tip} Вместо проверок `instanceof` можно использовать [reportable exceptions](/docs/{{version}}/errors#renderable-exceptions).

#### Глобальный контекст для логирования

Если доступно, Laravel автоматически добавляет id текущего пользователя к каждому сообщению журнала исключений в качестве контекстных данных. Вы можете определить свои собственные глобальные контекстные данные, переопределив метод `context` класса `App\Exceptions\Handler`. Эта информация будет включена в лог каждого исключения:

    /**
     * Get the default context variables for logging.
     *
     * @return array
     */
    protected function context()
    {
        return array_merge(parent::context(), [
            'foo' => 'bar',
        ]);
    }

#### Хелпер `report`

Иногда вам надо зафиксировать наличие исключения, но продолжить обрабатывать запрос. Хелпер `report` поможет это сделать, без рендеринга страницы ошибки:

    public function isValid($value)
    {
        try {
            // Validate the value...
        } catch (Exception $e) {
            report($e);

            return false;
        }
    }

#### Игнорирование исключений по типу

Свойство `$dontReport` класса `App\Exceptions\Handler` содержит массив типов исключений, которые не будут логироваться. Например, в лог-файлы не записываются исключения ошибок 404, а также некоторые другие типы ошибок. При необходимости в этот массив можно добавить и другие типы исключений:

    /**
     * A list of the exception types that should not be reported.
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

Метод `render` отвечает за преобразование заданного исключения в ответ HTTP, который должен быть отправлен обратно в браузер. По умолчанию исключение передается базовому классу, который генерирует ответ. Однако, вы можете проверить тип исключения или вернуть свой собственный пользовательский ответ:

    /**
     * Render an exception into an HTTP response.
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

<a name="renderable-exceptions"></a>
### Reportable и Renderable исключения

Вместо того, чтобы проверять в `App\Exceptions\Handler`, как именно надо среагировать на исключение по его типу, отправить просто репорт или http-ответ, вы можете добавить в свои исключения методы `report` или `render` - и фреймворк будет вызывать их автоматически.

    <?php

    namespace App\Exceptions;

    use Exception;

    class RenderException extends Exception
    {
        /**
         * Report the exception.
         *
         * @return void
         */
        public function report()
        {
            //
        }

        /**
         * Render the exception into an HTTP response.
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function render($request)
        {
            return response(...);
        }
    }

> {tip} Фреймворк также позволяет добавить зависимости в аргументы метода `report` - они будут автоматически подгружены при помощи [сервис-контейнера](/docs/{{version}}/container)

<a name="http-exceptions"></a>
## HTTP-исключения

Некоторые исключения описывают коды HTTP-ошибок от сервера. Например, это может быть ошибка "страница не найдена" (404), "ошибка авторизации" (401) или даже сгенерированная разработчиком ошибка 500. Для возврата такого отклика из любого места в приложении можете использовать хелпер `abort`:

    abort(404);

Хелпер `abort` немедленно создаёт исключение, которое будет отрисовано обработчиком исключений. Или вы можете предоставить такой отклик:

    abort(403, 'Unauthorized action.');    

<a name="custom-http-error-pages"></a>
### Пользовательские страницы HTTP-ошибок

Для того, чтобы сделать свою страницу для показа в случае возникновения ошибки 404 - создайте файл `resources/views/errors/404.blade.php`. Со страницами по ошибкам с другими кодами - действия аналогичны, располагайте blade-файлы с кодом ошибки в названии в папке `resources/views/errors`. Исключение будет доступно в переменной `$exception`:

    <h2>{{ $exception->getMessage() }}</h2>

Чтобы опубликовать в вашем приложении дефолтные страницы ошибок Laravel (например, для того, чтобы их изменить), воспользуйтесь artisan-командой `vendor::publish` с указанным тэгом:

    php artisan vendor:publish --tag=laravel-errors

