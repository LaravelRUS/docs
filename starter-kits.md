---
git: a2f48a373e83b529d67d904865205f22bc78d110
---

# Стартовые комплекты


<a name="introduction"></a>
## Введение

Чтобы дать вам фору при создании нового приложения Laravel, мы рады предложить стартовые комплекты приложения и, в частности, аутентификации. Эти комплекты автоматически дополнят ваше приложение маршрутами, контроллерами и шаблонами, необходимыми для регистрации и аутентификации пользователей вашего приложения.

Вы можете использовать эти стартовые комплекты, но они не требуются. Вы можете создать собственное приложение с нуля, просто установив новую копию Laravel. В любом случае мы знаем, что вы создадите что-то отличное!

<a name="laravel-breeze"></a>
## Laravel Breeze

[Laravel Breeze](https://github.com/laravel/breeze) - это минимальная и простая реализация всех функций [аутентификации](/docs/{{version}}/authentication) Laravel, включая вход в систему, регистрацию, сброс пароля, подтверждение по электронной почте и подтверждение пароля. Кроме того, Breeze включает в себя простую "страницу профиля", где пользователь может обновить свое имя, адрес электронной почты и пароль.

Представление по умолчанию в Laravel Breeze состоит из простых [шаблонов Blade](/docs/{{version}}/blade), стилизованных с использованием [Tailwind CSS](https://tailwindcss.com). Кроме того, Breeze предоставляет опции для создания каркасов на основе [Livewire](https://livewire.laravel.com) или [Inertia](https://inertiajs.com), с выбором использования Vue или React для каркаса на основе Inertia.

![Пример страницы регистрации Breeze](https://laravel.com/img/docs/breeze-register.png)

#### Laravel Bootcamp

Если вы новичок в Laravel, не стесняйтесь присоединиться к [Laravel Bootcamp](https://bootcamp.laravel.com). Laravel Bootcamp научит вас создавать свое первое приложение на Laravel с использованием Breeze. Это отличный способ ознакомиться со всем, что Laravel и Breeze могут предложить.

<a name="laravel-breeze-installation"></a>
### Установка

Сначала вам следует [создать новое приложение Laravel](/docs/{{version}}/installation), настроить вашу базу данных и запустить [миграции базы данных](/docs/{{version}}/migrations). После создания нового приложения Laravel вы можете установить Laravel Breeze с помощью Composer:

```shell
composer require laravel/breeze --dev
```

После установки пакета Laravel Breeze с помощью Composer, вы можете выполнить команду Artisan `breeze:install`. Эта команда публикует представления аутентификации, маршруты, контроллеры и другие ресурсы в вашем приложении. Laravel Breeze публикует всей свой код в вашем приложении, чтобы у вас была полная контроль и видимость над его функциями и реализацией.

Команда `breeze:install` запросит у вас предпочтительный стек фронтенда и фреймворк для тестирования:

```shell
php artisan breeze:install

php artisan migrate
npm install
npm run dev
```

<a name="breeze-and-blade"></a>
### Breeze и Blade

Стандартный стек Breeze - это Blade стек, который использует простые [шаблоны Blade](/docs/{{version}}/blade) для отображения фронтенда вашего приложения. Вы можете установить Blade стек, вызвав команду `breeze:install` без дополнительных аргументов и выбрав Blade фронтенд стек. После установки структуры Breeze вам также следует скомпилировать фронтенд-ресурсы вашего приложения:

```shell
php artisan breeze:install

php artisan migrate
npm install
npm run dev
```

Затем, вы можете перейти в своем веб-браузере по URL-адресам вашего приложения `/login` или `/register`. Все маршруты Breeze определены в файле `routes/auth.php`.

> **Note**
> Чтобы узнать больше о компиляции CSS и JavaScript вашего приложения, ознакомьтесь с [документацией Laravel по Vite](/docs/{{version}}/vite#running-vite).

<a name="breeze-and-livewire"></a>
### Breeze и Livewire

Laravel Breeze также предлагает шаблоны для [Livewire](https://livewire.laravel.com). Livewire - это мощный способ создания динамических, реактивных пользовательских интерфейсов (UI) на фронтенде, используя только PHP.

Livewire отлично подходит для команд, которые в основном используют шаблоны Blade и ищут более простую альтернативу фреймворкам для создания SPA (Single Page Application) на JavaScript, таким как Vue и React.

Чтобы использовать стек Livewire, вы можете выбрать Livewire при выполнении команды Artisan `breeze:install`. После установки шаблонов Breeze вам следует запустить миграции базы данных:

```shell
php artisan breeze:install

php artisan migrate
```

<a name="breeze-and-inertia"></a>
### Breeze & React / Vue

Laravel Breeze также предоставляет средства сборки для React и Vue через фронтенд-реализацию [Inertia](https://inertiajs.com). Inertia позволяет создавать современные одностраничные приложения React и Vue с использованием классической маршрутизации на стороне сервера и контроллеров.

Inertia позволяет вам наслаждаться возможностями React и Vue на стороне фронтенда, объединенными с невероятной производительностью Laravel на стороне бэкенда и быстрой компиляцией [Vite](https://vitejs.dev). Чтобы использовать стек Inertia, вы можете выбрать стек Vue или React при выполнении команды Artisan `breeze:install`.

При выборе стека фронтенда Vue или React, установщик Breeze также предложит вам определить, хотите ли вы [Inertia SSR (серверный рендеринг)](https://inertiajs.com/server-side-rendering) или поддержку TypeScript. После установки шаблонов Breeze вам также следует скомпилировать фронтенд-ресурсы вашего приложения:

```shell
php artisan breeze:install

php artisan migrate
npm install
npm run dev
```

Затем вы можете перейти по адресу `/login` или `/register` вашего приложения в веб-браузере. Все маршруты Breeze определены в файле `routes/auth.php`.


<a name="breeze-and-next"></a>
### Breeze & Next.js / API

Laravel Breeze также может создавать структуру аутентификационного API, готовую к аутентификации современных приложений на JavaScript, таких как [Next](https://nextjs.org), [Nuxt](https://nuxt.com) и другие. Для начала выберите стек API в качестве желаемого стека при выполнении команды Artisan `breeze:install`:

```shell
php artisan breeze:install

php artisan migrate
```

Во время установки Breeze добавит переменную окружения `FRONTEND_URL` в файл `.env` вашего приложения. Этот URL должен быть URL вашего JavaScript-приложения. Обычно это будет `http://localhost:3000` во время локальной разработки. Кроме того, убедитесь, что ваша переменная окружения `APP_URL` установлена в `http://localhost:8000`, который является URL по умолчанию для команды Artisan `serve`.

<a name="next-reference-implementation"></a>
#### Эталонная реализация Next.js 

Наконец, вы готовы связать этот бэкэнд с выбранным вами интерфейсом. Следующая эталонная реализация интерфейса Breeze [доступна на GitHub](https://github.com/laravel/breeze-next). Этот интерфейс поддерживается Laravel и содержит тот же пользовательский интерфейс, что и традиционные стеки Blade и Inertia, предоставляемые Breeze.

<a name="laravel-jetstream"></a>
## Laravel Jetstream

В то время как Laravel Breeze обеспечивает простую и минимальную отправную точку для создания приложения Laravel, Jetstream дополняет эту функциональность более надежными функциями и дополнительными стеками технологий клиентского интерфейса. **Для тех, кто новичок в Laravel, мы рекомендуем изучить основы работы с Laravel Breeze перед тем, как перейти на Laravel Jetstream.**

Jetstream предоставляет красиво оформленный каркас приложения для Laravel и включает в себя функции входа, регистрации, проверки почты, двухфакторной аутентификации, управления сеансами, поддержку API с использованием Laravel Sanctum и опциональное управление командами. Jetstream разработан с использованием [Tailwind CSS](https://tailwindcss.com) и предлагает выбор между фронтенд-структурами, основанными на [Livewire](https://livewire.laravel.com) или [Inertia](https://inertiajs.com).

Полное описание по установке Laravel Jetstream можно найти в [официальной документации Jetstream](https://jetstream.laravel.com).
