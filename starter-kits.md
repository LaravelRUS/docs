git 2cf67bcaacfec590098cefb45af824b74671cfa0

---

# Стартовые комплекты

- [Введение](#introduction)
- [Laravel Breeze](#laravel-breeze)
    - [Установка](#laravel-breeze-installation)
    - [Breeze & Inertia](#breeze-and-inertia)
    - [Breeze & Next.js / API](#breeze-and-next)
- [Laravel Jetstream](#laravel-jetstream)

<a name="introduction"></a>
## Введение

Чтобы дать вам фору при создании нового приложения Laravel, мы рады предложить стартовые комплекты приложения и, в частности, аутентификации. Эти комплекты автоматически дополнят ваше приложение маршрутами, контроллерами и шаблонами, необходимыми для регистрации и аутентификации пользователей вашего приложения.

Вы можете использовать эти стартовые комплекты, но они не требуются. Вы можете создать собственное приложение с нуля, просто установив новую копию Laravel. В любом случае мы знаем, что вы создадите что-то отличное!

<a name="laravel-breeze"></a>
## Laravel Breeze

[**Laravel Breeze**](https://github.com/laravel/breeze) – это минимальная и простая реализация всего [функционала аутентификации](authentication) Laravel, включая вход в систему, регистрацию, сброс пароля, подтверждение адреса электронной почты и пароля. Слой «View» комплекта Laravel Breeze по умолчанию состоит из простых [шаблонов Blade](/docs/{{version}}/blade), стилизованных с помощью [Tailwind CSS](https://tailwindcss.com).

Breeze обеспечивает прекрасную отправную точку для создания нового приложения Laravel, а также является отличным выбором для проектов, которые планируют вывести свои шаблоны Blade на новый уровень с помощью [Laravel Livewire](https://laravel-livewire.com)

<a name="laravel-breeze-installation"></a>
### Установка

Сначала вы должны [создать новое приложение Laravel](/docs/{{version}}/installation), настроить свою базу данных и запустить [миграции базы данных](/docs/{{version}}/migrations):

```bash
curl -s https://laravel.build/example-app | bash

cd example-app

php artisan migrate
```

Создав новое приложение Laravel, вы можете установить Laravel Breeze с помощью Composer:

```bash
composer require laravel/breeze --dev
```

После того как Composer установит пакет Laravel Breeze, вы можете запустить команду `breeze:install` Artisan. Эта команда опубликует для вашего приложения шаблоны, маршруты, контроллеры и другие ресурсы аутентификации. Laravel Breeze опубликует весь свой код в вашем приложении, чтобы у вас был полный контроль, а также обзор всего функционала и его реализации. После установки Breeze вы также должны скомпилировать свои исходники, чтобы был доступен файл стилей вашего приложения:

```nothing
php artisan breeze:install

npm install

npm run dev
php artisan migrate
```

Затем, вы можете перейти в своем веб-браузере по URL-адресам вашего приложения `/login` или `/register`. Все маршруты Breeze определены в файле `routes/auth.php`.

> {tip} Чтобы узнать больше о компиляции CSS и JavaScript вашего приложения, ознакомьтесь с [документацией Laravel Mix](/docs/{{version}}/mix#running-mix).

<a name="breeze-and-inertia"></a>
### Breeze и Inertia

Laravel Breeze также предлагает реализацию внешнего интерфейса [Inertia.js](https://inertiajs.com) на базе Vue или React. Чтобы использовать стек Inertia, укажите `vue` или` react` в качестве желаемого стека при выполнении Artisan-команды `breeze:install`:

```nothing
php artisan breeze:install vue

// Or...

php artisan breeze:install react

npm install
npm run dev
php artisan migrate
```


<a name="breeze-and-next"></a>
### Breeze & Next.js / API

Laravel Breeze также может формировать API аутентификации, готовый для аутентификации современных приложений JavaScript, таких, как те, которые работают на [Next](https://nextjs.org), [Nuxt](https://nuxtjs.org) и других. Для начала укажите стек `api` в качестве желаемого стека при выполнении Artisan-команды `breeze:install`:

```nothing
php artisan breeze:install api

php artisan migrate
```

Во время установки Breeze добавит переменную среды `FRONTEND_URL` в файл `.env` вашего приложения. Этот URL-адрес должен быть URL-адресом вашего приложения JavaScript. Обычно во время локальной разработки это будет `http://localhost:3000`.

<a name="next-reference-implementation"></a>
#### Эталонная реализация Next.js 

Наконец, вы готовы связать этот бэкэнд с выбранным вами интерфейсом. Следующая эталонная реализация интерфейса Breeze [доступна на GitHub](https://github.com/laravel/breeze-next). Этот интерфейс поддерживается Laravel и содержит тот же пользовательский интерфейс, что и традиционные стеки Blade и Inertia, предоставляемые Breeze.

<a name="laravel-jetstream"></a>
## Laravel Jetstream

В то время как Laravel Breeze обеспечивает простую и минимальную отправную точку для создания приложения Laravel, Jetstream дополняет эту функциональность более надежными функциями и дополнительными стеками технологий клиентского интерфейса. **Для тех, кто новичок в Laravel, мы рекомендуем изучить основы работы с Laravel Breeze перед тем, как перейти на Laravel Jetstream.**

Jetstream предлагает красиво оформленный каркас приложений для Laravel и включает в себя вход в систему, регистрацию, подтверждение адреса электронной почты, двухфакторную аутентификацию, управление сессиями, поддержку API через Laravel Sanctum, и дополнительно, управление командой. Jetstream разработан с использованием [Tailwind CSS](https://tailwindcss.com) и предлагает на ваш выбор каркас клиентского интерфейса под управлением [Livewire](https://laravel-livewire.com) либо [Inertia.js](https://inertiajs.com).

Полное описание по установке Laravel Jetstream можно найти в [официальной документации Jetstream](https://jetstream.laravel.com/2.x/introduction.html).
