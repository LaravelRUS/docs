git 1b5d0e0e44128f0cb6210b4f481a603874dd852a

---

# Подтверждение Email

- [Введение](#introduction)
- [Подготовка базы данных](#verification-database)
- [Роутинг](#verification-routing)
    - [Защита роутов](#protecting-routes)
- [Шаблоны](#verification-views)
- [После подтверждения Email](#after-verifying-emails)
- [События](#events)

<a name="introduction"></a>
## Введение

Многие веб-приложения требуют, чтобы пользователи подтверждали свои адреса электронной почты перед использованием приложения. Вместо того, чтобы заставлять вас повторно внедрять это в каждом приложении, Laravel предоставляет удобные методы для отправки и проверки адреса электронной почты пользователя.

### Подготовка модели

Для начала убедитесь, что ваша модель `App\User` реализует контракт `Illuminate\Contracts\Auth\MustVerifyEmail`:

    <?php

    namespace App;

    use Illuminate\Contracts\Auth\MustVerifyEmail;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable implements MustVerifyEmail
    {
        use Notifiable;

        // ...
    }

<a name="verification-database"></a>
## Настройки базы данных

#### Столбец подтверждения Email

Ваша таблица `users` должна содержать столбец `email_verified_at` для хранения даты и времени, когда адрес электронной почты был проверен. По умолчанию миграция таблиц пользователей, включенная в инфраструктуру Laravel, уже включает этот столбец. Итак, все, что вам нужно сделать, это запустить миграцию базы данных:

    php artisan migrate

<a name="verification-routing"></a>
## Роутинг

Laravel включает класс `Auth\VerificationController`, который содержит необходимую логику для отправки ссылок подтверждения email и проверки электронной почты. Чтобы зарегистрировать необходимые роуты для этого контроллера, передайте опцию `verify` методу `Auth::routes`

    Auth::routes(['verify' => true]);

<a name="protecting-routes"></a>
### Защита роутов

[Посредников роута](/docs/{{version}}/middleware) (`Route middleware`) можно использовать, чтобы разрешить доступ только проверенным пользователям к данному роуту. Laravel поставляется с посредником `verified`, который определен в `Illuminate\Auth\Middleware\EnsureEmailIsVerified`. Поскольку это посредник уже зарегистрирован в `app/Http/Kernel.php` вашего приложения, все, что вам нужно сделать, это подключить посредника к определению роута:

    Route::get('profile', function () {
        // Только для пользователей с подтвержденным адресом электронной почты...
    })->middleware('verified');

<a name="verification-views"></a>
## Шаблоны

Для того чтобы сгенерировать все необходимые шаблоны для подтверждения электронной почты, Вы можете использовать пакет `laravel/ui` для Composer:

    composer require laravel/ui --dev
    
    php artisan ui vue --auth
    
Шаблон подтверждения email адреса находится в `resources/views/auth/verify.blade.php`. Вы можете изменять его на свое усмотрение.

<a name="after-verifying-emails"></a>
## Действие после подтверждения Email

После проверки адреса электронной почты пользователь будет автоматически перенаправлен на URL `/home`. Вы можете настроить местоположение перенаправления после проверки, определив метод или свойство `redirectTo` в `VerificationController`:

    protected $redirectTo = '/dashboard';
    
<a name="events"></a>
## События

Laravel отправляет [события](/docs/{{version}}/events) во время процесса подтверждения email. Вы можете объявить слушателей этих событий в Вашем `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Verified' => [
            'App\Listeners\LogVerifiedUser',
        ],
    ];
