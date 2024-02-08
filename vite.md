---
git: d655507c36ef5f692f67cd091028758345ab2e53
---

# Сборка ресурсов (Vite)

## Введение

[Vite](https://vitejs.dev) - это современный инструмент сборки фронтенда, который обеспечивает чрезвычайно быстрое окружение разработки и собирает ваш код для продакшена. При создании приложений с использованием Laravel вы обычно используете Vite для сборки файлов CSS и JavaScript вашего приложения в готовые к продакшену ресурсы.

Laravel интегрируется с Vite без проблем, предоставляя официальный плагин и директиву Blade для загрузки ваших ресурсов как для разработки, так и для продакшена.

> [!NOTE]  
> Вы используете Laravel Mix? Vite заменил Laravel Mix в новых установках Laravel. Для документации по Mix посетите веб-сайт [Laravel Mix](https://laravel-mix.com/). Если вы хотите перейти на Vite, ознакомьтесь с нашим [руководством по миграции](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite).

<a name="vite-or-mix"></a>
<a name="vite-or-mix"></a>
#### Выбор между Vite и Laravel Mix

Прежде чем перейти на Vite, новые приложения Laravel использовали [Mix](https://laravel-mix.com/) при сборке ресурсов, который работает на основе [webpack](https://webpack.js.org/). Vite сосредоточен на предоставлении более быстрого и продуктивного опыта при создании мощных JavaScript-приложений. Если вы разрабатываете одностраничное приложение (SPA), включая те, которые разработаны с использованием инструментов, таких как [Inertia](https://inertiajs.com), то Vite будет идеальным выбором.

Vite также хорошо работает с традиционными приложениями с серверным рендерингом с "примесью" JavaScript, включая те, которые используют [Livewire](https://livewire.laravel.com). Однако у него нет некоторых функций, которые поддерживает Laravel Mix, таких как возможность копирования произвольных ресурсов, на которые нет прямых ссылок в вашем приложении JavaScript, в сборку.

<a name="migrating-back-to-mix"></a>
#### Возвращение к Mix

Вы начали новое приложение Laravel, используя нашу структуру Vite, но вам нужно вернуться к Laravel Mix и webpack? Нет проблем. Пожалуйста, обратитесь к нашему [официальному руководству по миграции с Vite на Mix](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-vite-to-laravel-mix).

<a name="installation"></a>
## Установка и настройка

> [!NOTE]  
> В следующей документации рассматривается процесс ручной установки и настройки плагина Laravel Vite. Однако стартовые комплекты Laravel уже включают в себя всю эту структуру и являются самым быстрым способом начать работу с Laravel и Vite.

<a name="installing-node"></a>
### Установка Node

Перед запуском Vite и плагина Laravel убедитесь, что у вас установлены Node.js (версии 16 и выше) и NPM:

```sh
node -v
npm -v
```

Вы можете легко установить последнюю версию Node и NPM, используя простые установщики с [официального сайта Node](https://nodejs.org/en/download/). Или, если вы используете [Laravel Sail](https://laravel.com/docs/{{version}}/sail), вы можете вызвать Node и NPM через Sail:

```sh
./vendor/bin/sail node -v
./vendor/bin/sail npm -v
```

<a name="installing-vite-and-laravel-plugin"></a>
### Установка Vite и плагина Laravel

В новой установке Laravel в корне структуры каталогов вашего приложения вы найдете файл `package.json`. В файле `package.json` уже содержится все необходимое для начала работы с Vite и плагином Laravel. Вы можете установить зависимости фронтенда вашего приложения через NPM:

```sh
npm install
```

<a name="configuring-vite"></a>
### Настройка Vite

Vite настраивается с помощью файла `vite.config.js` в корне вашего проекта. Вы можете настраивать этот файл по своему усмотрению, а также устанавливать любые другие плагины, необходимые для вашего приложения, такие как `@vitejs/plugin-vue` или `@vitejs/plugin-react`.

Плагин Laravel Vite требует указания точек входа для вашего приложения. Это могут быть файлы JavaScript или CSS, включая предварительно обработанные языки, такие как TypeScript, JSX, TSX и Sass.

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css',
            'resources/js/app.js',
        ]),
    ],
});
```

Если вы создаете SPA, включая приложения, построенные с использованием Inertia, то Vite будет лучше всего работать без точек входа CSS:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css', // [tl! remove]
            'resources/js/app.js',
        ]),
    ],
});
```

Вместо этого вы должны импортировать свой CSS через JavaScript. Обычно это делается в файле `resources/js/app.js` вашего приложения:

```js
import './bootstrap';
import '../css/app.css'; // [tl! add]
```

Плагин Laravel также поддерживает несколько точек входа и расширенные параметры конфигурации, такие как [точки входа SSR](#ssr).

<a name="working-with-a-secure-development-server"></a>
#### Работа с защищенным сервером разработки

Если ваш локальный веб-сервер разработки обслуживает ваше приложение через HTTPS, у вас могут возникнуть проблемы с подключением к серверу разработки Vite.

Если вы используете [Laravel Herd](https://herd.laravel.com) и зашифровали сайт, или вы используете [Laravel Valet](/docs/{{version}}/valet) и запустили [команду secure](/docs/{{version}}/valet#securing-sites) для вашего приложения, плагин Laravel Vite автоматически обнаружит и использует сгенерированный TLS-сертификат.

Если вы зашифровали сайт с использованием хоста, который не соответствует имени каталога приложения, вы можете вручную указать хост в файле `vite.config.js` вашего приложения:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            detectTls: 'my-app.test', // [tl! add]
        }),
    ],
});
```

Когда вы используете другой веб-сервер, вы должны сгенерировать доверенный сертификат и вручную настроить Vite для использования сгенерированных сертификатов:

```js
// ...
import fs from 'fs'; // [tl! add]

const host = 'my-app.test'; // [tl! add]

export default defineConfig({
    // ...
    server: { // [tl! add]
        host, // [tl! add]
        hmr: { host }, // [tl! add]
        https: { // [tl! add]
            key: fs.readFileSync(`/path/to/${host}.key`), // [tl! add]
            cert: fs.readFileSync(`/path/to/${host}.crt`), // [tl! add]
        }, // [tl! add]
    }, // [tl! add]
});
```

Если вы не можете сгенерировать доверенный сертификат для вашей системы, вы можете установить и настроить плагин `@vitejs/plugin-basic-ssl`. При использовании ненадежных сертификатов вам нужно будет принять предупреждение о сертификате для сервера разработки Vite в вашем браузере, перейдя по ссылке "Local" в консоли при выполнении команды `npm run dev`.

<a name="configuring-hmr-in-sail-on-wsl2"></a>
#### Запуск сервера разработки в Sail на WSL2

При запуске сервера разработки Vite в [Laravel Sail](/docs/{{version}}/sail) на Windows Subsystem for Linux 2 (WSL2), вам следует добавить следующую конфигурацию в ваш файл `vite.config.js`, чтобы обеспечить связь браузера с сервером разработки:

```js
// ...

export default defineConfig({
    // ...
    server: { // [tl! add:start]
        hmr: {
            host: 'localhost',
        },
    }, // [tl! add:end]
});
```

Если изменения в файлах не отображаются в браузере при запущенном сервере разработки, вам также может потребоваться настроить опцию [`server.watch.usePolling`](https://vitejs.dev/config/server-options.html#server-watch) в Vite.
<a name="loading-your-scripts-and-styles"></a>
### Загрузка ваших скриптов и стилей

После настройки точек входа Vite вы можете ссылаться на них с помощью директивы `@vite()` в Blade, которую вы должны добавить в `<head>` корневого шаблона вашего приложения:

```blade
<!doctype html>
<head>
    {{-- ... --}}

    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
```

Если вы импортируете свой CSS через JavaScript, вам нужно включить только точку входа JavaScript:

```blade
<!doctype html>
<head>
    {{-- ... --}}

    @vite('resources/js/app.js')
</head>
```

Директива `@vite` автоматически обнаруживает сервер разработки Vite и внедряет клиент Vite для возможности горячей замены модулей. В режиме сборки директива загрузит ваши скомпилированные и пронумерованные ресурсы, включая любой импортированный CSS.

При необходимости вы также можете указать путь сборки ваших скомпилированных ресурсов при вызове директивы `@vite`.

```blade
<!doctype html>
<head>
    {{-- Given build path is relative to public path. --}}

    @vite('resources/js/app.js', 'vendor/courier/build')
</head>
```

<a name="inline-assets"></a>
#### Встраивание ресурсов

Иногда может быть необходимо включить сырое содержимое ресурсов, а не ссылаться на версионный URL ресурса. Например, вам может понадобиться включить содержимое ресурса непосредственно на страницу, когда передаете HTML-контент генератору PDF. Вы можете выводить содержимое ресурсов Vite с помощью метода `content`, предоставленного фасадом `Vite`:

```blade
@php
use Illuminate\Support\Facades\Vite;
@endphp

<!doctype html>
<head>
    {{-- ... --}}

    <style>
        {!! Vite::content('resources/css/app.css') !!}
    </style>
    <script>
        {!! Vite::content('resources/js/app.js') !!}
    </script>
</head>
```

<a name="running-vite"></a>
## Запуск Vite

Существует два способа запуска Vite. Вы можете запустить сервер разработки с помощью команды `dev`, что полезно во время локальной разработки. Сервер разработки автоматически обнаруживает изменения в ваших файлах и мгновенно отображает их в любых открытых окнах браузера.

Или выполнить команду `build`, которая версионирует и соберет ресурсы вашего приложения, подготовив их к развертыванию в производственной среде:

```shell
# Run the Vite development server...
npm run dev

# Build and version the assets for production...
npm run build
```

Если вы запускаете сервер разработки в [Sail](/docs/{{version}}/sail) на WSL2, вам могут потребоваться дополнительные [опции конфигурации](#configuring-hmr-in-sail-on-wsl2).

<a name="working-with-scripts"></a>
## Работа с JavaScript

<a name="aliases"></a>
### Псевдонимы

По умолчанию плагин Laravel предоставляет общий псевдоним, чтобы вы могли начать работу сразу и удобно импортировать ресурсы вашего приложения:

```js
{
    '@' => '/resources/js'
}
```

Вы можете переопределить псевдоним `'@'`, добавив собственный в файл конфигурации `vite.config.js`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel(['resources/ts/app.tsx']),
    ],
    resolve: {
        alias: {
            '@': '/resources/ts',
        },
    },
});
```

<a name="vue"></a>
### Vue

Если вы хотите собрать свой фронтенд, используя фреймворк [Vue](https://vuejs.org/), вам также нужно установить плагин `@vitejs/plugin-vue`:

```sh
npm install --save-dev @vitejs/plugin-vue
```

Затем вы можете включить плагин в ваш файл конфигурации `vite.config.js`. При использовании плагина Vue с Laravel вам потребуются несколько дополнительных параметров:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.js']),
        vue({
            template: {
                transformAssetUrls: {
                    // Плагин Vue перепишет URL-адреса ресурсов, когда они 
                    // будут использоваться в однофайловых компонентах, 
                    // чтобы указывать на веб-сервер Laravel. Установка этого 
                    // значения в `null` позволяет вместо этого плагину Laravel 
                    // переписывать URL-адреса ресурсов, чтобы они указывали на сервер Vite.
                    base: null,

                    // Плагин Vue будет анализировать абсолютные URL-адреса и рассматривать 
                    // их как абсолютные пути к файлам на диске. Установка этого значения в 
                    // `false` оставит абсолютные URL-адреса нетронутыми, чтобы они могли 
                    // ссылаться на ресурсы в папке public, как ожидается.
                    includeAbsolute: false,
                },
            },
        }),
    ],
});
```

> [!NOTE]  
> Стартовые наборы Laravel ([starter kits](/docs/{{version}}/starter-kits)) уже включают правильную конфигурацию Laravel, Vue и Vite. Посмотрите [Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia) для самого быстрого способа начать работу с Laravel, Vue и Vite.

<a name="react"></a>
### React

Если вы хотите собрать свой фронтенд, используя фреймворк [React](https://reactjs.org/), вам также необходимо установить плагин `@vitejs/plugin-react`:

```sh
npm install --save-dev @vitejs/plugin-react
```

Затем вы можете включить плагин в ваш файл конфигурации `vite.config.js`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.jsx']),
        react(),
    ],
});
```

Вы должны убедиться, что любые файлы, содержащие JSX, имеют расширение `.jsx` или `.tsx`, помня о необходимости обновления вашей точки входа, если это требуется, как показано [выше](#configuring-vite).

Вам также потребуется включить дополнительную директиву `@viteReactRefresh` вместе с вашей текущей директивой `@vite` в Blade.

```blade
@viteReactRefresh
@vite('resources/js/app.jsx')
```

Директива `@viteReactRefresh` должна быть вызвана перед директивой `@vite`.

> [!NOTE]  
> Стартовые наборы Laravel ([starter kits](/docs/{{version}}/starter-kits)) уже включают правильную конфигурацию Laravel, React и Vite. Ознакомьтесь с [Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia) - самым быстрым способом начать работу с Laravel, React и Vite.

<a name="inertia"></a>
### Inertia

Плагин Laravel Vite предоставляет удобную функцию `resolvePageComponent`, которая поможет вам определить ваши компоненты страниц Inertia. Ниже приведен пример использования помощника с Vue 3; однако, вы также можете использовать эту функцию в других фреймворках, таких как React:

```js
import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';

createInertiaApp({
  resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
  setup({ el, App, props, plugin }) {
    return createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
});
```

> [!NOTE]  
> Стартовые наборы Laravel ([starter kits](/docs/{{version}}/starter-kits)) уже включают правильную конфигурацию Laravel, Inertia и Vite. Ознакомьтесь с [Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia) чтобы быстро начать работу с Laravel, Inertia и Vite.
<a name="url-processing"></a>
### Обработка URL

При использовании Vite и ссылок на ресурсы в HTML, CSS или JS вашего приложения, следует учитывать несколько моментов. Во-первых, если вы ссылаетесь на ресурсы с абсолютным путем, Vite не включит ресурс в сборку; поэтому убедитесь, что ресурс доступен в вашей публичной директории.

При использовании относительных путей к ресурсам помните, что пути относительны файлу, в котором они используются. Любые ресурсы, ссылки на которые осуществляются через относительный путь, будут переписаны, версионированны и собраны Vite.

Рассмотрим следующую структуру проекта:

```nothing
public/
  taylor.png
resources/
  js/
    Pages/
      Welcome.vue
  images/
    abigail.png
```

Следующий пример демонстрирует, как Vite будет обрабатывать относительные и абсолютные URL-адреса:

```html
<!-- Этот ресурс не обрабатывается Vite и не будет включен в сборку. -->
<img src="/taylor.png">

<!-- Этот ресурс будет переписан, пронумерован и собран Vite. -->
<img src="../../images/abigail.png">
```

<a name="working-with-stylesheets"></a>
## Working With Stylesheets

Вы можете узнать больше о поддержке CSS в Vite в [документации Vite](https://vitejs.dev/guide/features.html#css). Если вы используете плагины PostCSS, такие как [Tailwind](https://tailwindcss.com), вы можете создать файл `postcss.config.js` в корне вашего проекта, и Vite автоматически его применит.
```js
export default {
    plugins: {
        tailwindcss: {},
        autoprefixer: {},
    },
};
```

> [!NOTE]  
> Стартовые наборы Laravel ([starter kits](/docs/{{version}}/starter-kits)) уже включают правильную конфигурацию Tailwind, PostCSS и Vite. Или, если вы хотите использовать Tailwind и Laravel без использования одного из наших стартовых наборов, ознакомьтесь с [руководством по установке Tailwind для Laravel](https://tailwindcss.com/docs/guides/laravel).

<a name="working-with-blade-and-routes"></a>
## Работа с Blade и маршрутами

<a name="blade-processing-static-assets"></a>
### Обработка статических ресурсов с Vite

При ссылке на ресурсы в вашем JavaScript или CSS, Vite автоматически обрабатывает и версионирует их. Кроме того, при построении приложений на основе Blade, Vite также может обрабатывать и версионировать статические ресурсы, на которые вы ссылаетесь исключительно в шаблонах Blade.

Однако для этого необходимо осведомить Vite о ваших ресурсах, импортировав статические ресурсы в точку входа вашего приложения. Например, если вы хотите обработать и версионировать все изображения, хранящиеся в `resources/images`, и все шрифты, хранящиеся в `resources/fonts`, вам следует добавить следующее в точку входа вашего приложения `resources/js/app.js`:

```js
import.meta.glob([
  '../images/**',
  '../fonts/**',
]);
```

Теперь эти ресурсы будут обрабатываться Vite при запуске `npm run build`. Затем вы можете ссылаться на эти ресурсы в шаблонах Blade, используя метод `Vite::asset`, который вернет пронумерованный URL для указанного ресурса:

```blade
<img src="{{ Vite::asset('resources/images/logo.png') }}">
```

<a name="blade-refreshing-on-save"></a>
### Обновление при сохранении

Когда ваше приложение построено с использованием традиционного рендеринга на стороне сервера с помощью Blade, Vite может улучшить ваш рабочий процесс разработки, автоматически обновляя браузер при внесении изменений в файлы представлений вашего приложения. Чтобы начать, просто укажите параметр `refresh` как `true`.
```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: true,
        }),
    ],
});
```

Когда параметр `refresh` установлен в `true`, сохранение файлов в следующих каталогах вызовет полное обновление страницы в браузере при выполнении команды `npm run dev`:

- `app/View/Components/**`
- `lang/**`
- `resources/lang/**`
- `resources/views/**`
- `routes/**`

Отслеживание каталога `routes/**` полезно, если вы используете [Ziggy](https://github.com/tighten/ziggy) для создания ссылок на маршруты во фронтенде вашего приложения.

Если эти стандартные пути не соответствуют вашим потребностям, вы можете указать собственный список путей для отслеживания:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: ['resources/views/**'],
        }),
    ],
});
```

В основе плагина Laravel Vite используется пакет [`vite-plugin-full-reload`](https://github.com/ElMassimo/vite-plugin-full-reload), который предлагает некоторые дополнительные параметры конфигурации для настройки поведения этой функции. Если вам нужен такой уровень настройки, вы можете предоставить определение `config`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: [{
                paths: ['path/to/watch/**'],
                config: { delay: 300 }
            }],
        }),
    ],
});
```

<a name="blade-aliases"></a>
### Псевдонимы

Часто в JavaScript-приложениях создают [псевдонимы](#aliases) для часто используемых каталогов. Однако, вы также можете создавать псевдонимы для использования в Blade, используя метод `macro` класса `Illuminate\Support\Facades\Vite`. Обычно "макросы" определяются в методе `boot` [сервис-провайдера](/docs/{{version}}/providers):

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Vite::macro('image', fn (string $asset) => $this->asset("resources/images/{$asset}"));
    }

После определения макроса его можно вызвать в ваших шаблонах. Например, мы можем использовать определенный выше макрос `image`, чтобы ссылаться на ресурс, расположенный по пути `resources/images/logo.png`:

```blade
<img src="{{ Vite::image('logo.png') }}" alt="Laravel Logo">
```

<a name="custom-base-urls"></a>
## Пользовательские базовые URL

Если ваши скомпилированные ресурсы Vite развернуты на домене, отличном от вашего приложения, например, через CDN, вы должны указать переменную окружения `ASSET_URL` в файле `.env` вашего приложения:

```env
ASSET_URL=https://cdn.example.com
```

После настройки URL ресурса в начало всех переписанных URL-адреса ваших ресурсов будет добавлено указанное значение:

```nothing
https://cdn.example.com/build/assets/app.9dce8d17.js
```

Помните, что [абсолютные URL-адреса не переписываются Vite](#url-processing), поэтому они не будут изменены.

<a name="environment-variables"></a>
## Переменные среды

Вы можете внедрить переменные среды в ваш JavaScript, добавив им префикс `VITE_` в файле `.env` вашего приложения:

```env
VITE_SENTRY_DSN_PUBLIC=http://example.com
```

Вы можете получить доступ к внедренным переменным среды через объект `import.meta.env`:

```js
import.meta.env.VITE_SENTRY_DSN_PUBLIC
```

<a name="disabling-vite-in-tests"></a>
## Отключение Vite в тестах

Интеграция Vite в Laravel будет пытаться разрешить ваши ресурсы во время выполнения ваших тестов, что требует запуска сервера разработки Vite или сборки ваших ресурсов.

Если вы предпочитаете использовать имитацию Vite во время тестирования, вы можете вызвать метод `withoutVite`, который доступен для всех тестов, расширяющих класс `TestCase` Laravel:

```php
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_without_vite_example(): void
    {
        $this->withoutVite();

        // ...
    }
}
```

Если вы хотите отключить Vite для всех тестов, вы можете вызвать метод `withoutVite` из метода `setUp` вашего базового класса `TestCase`:

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    use CreatesApplication;

    protected function setUp(): void// [tl! add:start]
    {
        parent::setUp();

        $this->withoutVite();
    }// [tl! add:end]
}
```

<a name="ssr"></a>
## Рендеринг на стороне сервера (SSR)

Плагин Laravel Vite облегчает настройку рендеринга на стороне сервера с использованием Vite. Чтобы начать, создайте точку входа SSR в `resources/js/ssr.js` и укажите ее в плагине Laravel, передав конфигурационную опцию:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            ssr: 'resources/js/ssr.js',
        }),
    ],
});
```

Чтобы не забыть перестроить точку входа SSR, мы рекомендуем изменить скрипт "build" в файле `package.json` вашего приложения для создания сборки SSR:

```json
"scripts": {
     "dev": "vite",
     "build": "vite build" // [tl! remove]
     "build": "vite build && vite build --ssr" // [tl! add]
}
```

Затем, чтобы собрать и запустить сервер SSR, вы можете выполнить следующие команды:

```sh
npm run build
node bootstrap/ssr/ssr.js
```

Если вы используете [SSR с Inertia](https://inertiajs.com/server-side-rendering), вы можете вместо этого использовать команду Artisan `inertia:start-ssr` для запуска сервера SSR:

```sh
php artisan inertia:start-ssr
```

> [!NOTE]  
> Стартовые наборы Laravel ([starter kits](/docs/{{version}}/starter-kits)) уже включают правильную конфигурацию Laravel, SSR Inertia и Vite. Ознакомьтесь с [Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia) для самого быстрого способа начать работу с Laravel, SSR Inertia и Vite.

<a name="script-and-style-attributes"></a>
## Атрибуты тегов Style b Script

<a name="content-security-policy-csp-nonce"></a>
### Content Security Policy (CSP) Nonce

Если вы хотите включить атрибут [`nonce`](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce) в ваших тегах script и style как часть  [ политики безопасности контента (Content Security Policy)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP),  вы можете сгенерировать или указать nonce, используя метод  `useCspNonce` внутри собственного [middleware](/docs/{{version}}/middleware):

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Vite;
use Symfony\Component\HttpFoundation\Response;

class AddContentSecurityPolicyHeaders
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        Vite::useCspNonce();

        return $next($request)->withHeaders([
            'Content-Security-Policy' => "script-src 'nonce-".Vite::cspNonce()."'",
        ]);
    }
}
```

После вызова метода `useCspNonce`, Laravel автоматически включит атрибуты `nonce` во все сгенерированные теги script и style.

Если вам нужно указать nonce в другом месте, включая [директиву `@route` Ziggy](https://github.com/tighten/ziggy#using-routes-with-a-content-security-policy), входящую в стартовые наборы Laravel, вы можете сделать это, используя метод `cspNonce`:

```blade
@routes(nonce: Vite::cspNonce())
```

Если у вас уже есть nonce, который вы хотели бы использовать, вы можете передать его методу `useCspNonce`:

```php
Vite::useCspNonce($nonce);
```

<a name="subresource-integrity-sri"></a>
### Subresource Integrity (SRI) (Wелостность подресурсов)

Если ваш манифест Vite включает хеши `integrity` для ваших ресурсов, Laravel автоматически добавит атрибут `integrity` ко всем тегам script и style, которые он генерирует, чтобы обеспечить [целостность подресурсов](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity). По умолчанию Vite не включает хеш `integrity` в свой манифест, но вы можете включить его, установив плагин [`vite-plugin-manifest-sri`](https://www.npmjs.com/package/vite-plugin-manifest-sri) из NPM:

```shell
npm install --save-dev vite-plugin-manifest-sri
```

Затем вы можете включить этот плагин в вашем файле `vite.config.js`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import manifestSRI from 'vite-plugin-manifest-sri';// [tl! add]

export default defineConfig({
    plugins: [
        laravel({
            // ...
        }),
        manifestSRI(),// [tl! add]
    ],
});
```

При необходимости вы также можете настроить ключ манифеста, где будет находиться хеш целостности:

```php
use Illuminate\Support\Facades\Vite;

Vite::useIntegrityKey('custom-integrity-key');
```

Если вы хотите полностью отключить эту автообнаружение, вы можете передать `false` методу `useIntegrityKey`:

```php
Vite::useIntegrityKey(false);
```

<a name="arbitrary-attributes"></a>
### Произвольные атрибуты

Если вам нужно добавить дополнительные атрибуты к вашим тегам script и style, такие как атрибут [`data-turbo-track`](https://turbo.hotwired.dev/handbook/drive#reloading-when-assets-change), вы можете указать их с помощью методов `useScriptTagAttributes` и `useStyleTagAttributes`. Обычно эти методы вызываются из [сервис-провайдера](/docs/{{version}}/providers):

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes([
    'data-turbo-track' => 'reload', // Specify a value for the attribute...
    'async' => true, // Specify an attribute without a value...
    'integrity' => false, // Exclude an attribute that would otherwise be included...
]);

Vite::useStyleTagAttributes([
    'data-turbo-track' => 'reload',
]);
```

Если вам нужно использовать условие для добавления атрибутов, вы можете передать обратный вызов, который будет получать исходный путь ресурса, его URL, его фрагмент манифеста и весь манифест:

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $src === 'resources/js/app.js' ? 'reload' : false,
]);

Vite::useStyleTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $chunk && $chunk['isEntry'] ? 'reload' : false,
]);
```

> [!WARNING]  
> Аргументы `$chunk` и `$manifest` будут равны `null`, если сервер разработки Vite запущен.

<a name="advanced-customization"></a>
## Расширенная настройка

По умолчанию плагин Vite Laravel использует соглашения, которые должны подходить для большинства приложений; однако иногда вам может потребоваться настроить поведение Vite. Для активации дополнительных параметров настройки мы предлагаем следующие методы и параметры, которые могут использоваться вместо директивы Blade `@vite`:

```blade
<!doctype html>
<head>
    {{-- ... --}}

    {{
        Vite::useHotFile(storage_path('vite.hot')) // Customize the "hot" file...
            ->useBuildDirectory('bundle') // Customize the build directory...
            ->useManifestFilename('assets.json') // Customize the manifest filename...
            ->withEntryPoints(['resources/js/app.js']) // Specify the entry points...
            ->createAssetPathsUsing(function (string $path, ?bool $secure) { // Customize the backend path generation for built assets...
                return "https://cdn.example.com/{$path}";
            })
    }}
</head>
```

Затем в файле `vite.config.js` вы должны указать ту же конфигурацию:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            hotFile: 'storage/vite.hot', // Customize the "hot" file...
            buildDirectory: 'bundle', // Customize the build directory...
            input: ['resources/js/app.js'], // Specify the entry points...
        }),
    ],
    build: {
      manifest: 'assets.json', // Customize the manifest filename...
    },
});
```

<a name="correcting-dev-server-urls"></a>
### Коррекция URL-адресов сервера разработки

Некоторые плагины в экосистеме Vite предполагают, что URL-адреса, начинающиеся с косой черты, всегда будут указывать на сервер разработки Vite. Однако из-за характера интеграции с Laravel это не так.

Например, плагин `vite-imagetools` выводит URL-адреса, подобные следующим, пока Vite обслуживает ваши ресурсы:

```html
<img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520">
```

Плагин `vite-imagetools` ожидает, что выходной URL будет перехвачен Vite, и затем плагин сможет обрабатывать все URL-адреса, которые начинаются с `/@imagetools`. Если вы используете плагины, ожидающие такое поведение, вам придется вручную скорректировать URL-адреса. Вы можете сделать это в вашем файле `vite.config.js`, используя опцию `transformOnServe`.

В этом конкретном примере мы добавим префикс URL сервера разработки ко всем вхождениям `/@imagetools` в сгенерированном коде:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import { imagetools } from 'vite-imagetools';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            transformOnServe: (code, devServerUrl) => code.replaceAll('/@imagetools', devServerUrl+'/@imagetools'),
        }),
        imagetools(),
    ],
});
```

Теперь, когда Vite обслуживает ресурсы, он будет выводить URL-адреса, указывающие на сервер разработки Vite:

```html
- <img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! remove] -->
+ <img src="http://[::1]:5173/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! add] -->
```
