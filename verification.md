git 4db6e2f41b8aa262f35e3ef949e9dbf506d3c2d5

---

# Подтверждение Email

- [Введение](#introduction)
- [Database Considerations](#verification-database)
- [Роутинг](#verification-routing)
    - [Защита роутов](#protecting-routes)
- [Шаблоны](#verification-views)
- [После подтверждения Email](#after-verifying-emails)

<a name="introduction"></a>
## Введение

Многие веб-приложения требуют, чтобы пользователи подтверждали свои адреса электронной почты перед использованием приложения. Вместо того, чтобы заставлять вас повторно внедрять это в каждом приложении, Laravel предоставляет удобные методы для отправки и проверки адреса электронной почты пользователя.

### Подготовка модели

Для начала убедитесь, что ваша модель `App\User` реализует контракт `Illuminate\Contracts\Auth\MustVerifyEmail`:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Contracts\Auth\MustVerifyEmail;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable implements MustVerifyEmail
    {
        use Notifiable;

        // ...
    }

<a name="verification-database"></a>
## Настройки базы данных

#### Столбец подтверждения Email

Затем ваша таблица `user` должна содержать столбец `email_verified_at` для хранения даты и времени, когда адрес электронной почты был проверен. По умолчанию миграция таблиц пользователей, включенная в инфраструктуру Laravel, уже включает этот столбец. Итак, все, что вам нужно сделать, это запустить миграцию базы данных:

    php artisan migrate

<a name="verification-routing"></a>
## Роутинг

Laravel включает класс `Auth\VerificationController`, который содержит необходимую логику для отправки проверочных ссылок и проверки электронной почты. Чтобы зарегистрировать необходимые роуты для этого контроллера, передайте опцию `verify` методу `Auth::rout`:

    Auth::routes(['verify' => true]);

<a name="protecting-routes"></a>
### Защита роутов

[Посредников роута](/docs/{{version}}/middleware) можно использовать, чтобы разрешить только проверенным пользователям доступ к данному роуту. Laravel поставляется с `verified` посредником, который определен в `Illuminate\Auth\Middleware\EnsureEmailIsVerified`. Поскольку это посредник уже зарегистрировано в ядре HTTP вашего приложения, все, что вам нужно сделать, это подключить посредника к определению роута:

    Route::get('profile', function () {
        // Только для пользователей с подтвержденным адресом электронной почты...
    })->middleware('verified');

<a name="verification-views"></a>
## Шаблоны

Laravel сгенерирует все необходимые шаблоны проверки электронной почты при выполнении команды `make:auth`. Этот шаблон размещается в `resources/views/auth/verify.blade.php`. Вы можете настроить этот шаблон так, как вам нужно.

<a name="after-verifying-emails"></a>
## После подтверждения Email

После проверки адреса электронной почты пользователь будет автоматически перенаправлен в `/home`. Вы можете настроить местоположение перенаправления после проверки, определив метод или свойство `redirectTo` в `VerificationController`:

    protected $redirectTo = '/dashboard';
