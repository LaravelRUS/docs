git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Сборка фронтенда (Laravel Mix)

- [Введение](#introduction)
- [Установка и настройка](#installation)
- [Запуск Mix](#running-mix)
- [Работа с таблицами стилей](#working-with-stylesheets)
    - [Less](#less)
    - [Sass](#sass)
    - [Stylus](#stylus)
    - [PostCSS](#postcss)
    - [Простой CSS](#plain-css)
    - [Обработка URL](#url-processing)
    - [Source Maps](#css-source-maps)
- [Работа с JavaScript](#working-with-scripts)
    - [Извлечение библиотек поставщика](#vendor-extraction)
    - [React](#react)
    - [Vanilla JS](#vanilla-js)
    - [Пользовательская настройка Webpack](#custom-webpack-configuration)
- [Копирование файлов и директорий](#copying-files-and-directories)
- [Версии файлов / очистка кэша](#versioning-and-cache-busting)
- [Перезагрузка Browsersync](#browsersync-reloading)
- [Переменные среды](#environment-variables)
- [Уведомления](#notifications)

<a name="introduction"></a>
## Введение

[Laravel Mix](https://github.com/JeffreyWay/laravel-mix) - это чистый и гибкий API для определения инструкций сборки Webpack для вашего Laravel-приложения с использованием нескольких основных препроцессоров CSS и JavaScript. С помощью сцепки методов вы можете гибко определить свой конвейер сборки. Например:

    mix.js('resources/assets/js/app.js', 'public/js')
       .sass('resources/assets/sass/app.scss', 'public/css');

Если вы не знали, с какой стороны подойти к Webpack и вообще к сборке фронтенда, то вам точно понравится Laravel Mix. Но вам необязательно использовать именно его при разработке своего приложения. Вы можете использовать любой другой инструмент для сборки, или вообще не использовать его.

<a name="installation"></a>
## Установка и настройка

#### Установка Node

Перед запуском Mix необходимо убедиться, что на вашей машине установлены Node.js и NPM.

    node -v
    npm -v

По умолчанию Laravel Homestead включает в себя всё необходимое. Однако, если вы не используете Vagrant, то вы можете легко установить последнюю версию Node и NPM, используя простые графические установщики с [их страницы загрузки](https://nodejs.org/en/download/).

#### Laravel Mix

Послений оставшийся шаг - установить Laravel Mix. В свежей установке Laravel вы найдете файл `package.json` в корне вашей структуры директорий. Файл `package.json` по умолчанию включает все необходимое, чтобы начать. Подумайте об этом как о вашем файле `composer.json`, кроме того, что он определяет Node-зависимости вместо PHP. Вы можете установить зависимости, запустив:

    npm install

Если вы занимаетесь разработкой на системе Windows или ваша VM работает на Windows-хосте, вам может потребоваться выполнить команду `npm install` с ключом `--no-bin-links`:

    npm install --no-bin-links

<a name="running-mix"></a>
## Запуск Mix

Mix - это слой настройки поверх [Webpack](https://webpack.js.org), поэтому для запуска задач Mix вам нужно только выполнить один из NPM-скриптов, который включен в файл Laravel `package.json` по умолчанию:

    // Запустить все задачи Mix...
    npm run dev

    // Запустить все задачи Mix и минифицировать вывод...
    npm run production

#### Отслеживание изменений ассетов

Команда `npm run watch` продолжит выполняться в терминале и будет следить за всеми изменениями ваших ресурсов. Когда что-либо изменится, автоматически скомпилируются новые файлы:

    npm run watch

Вы можете обнаружить, что в определенных средах Webpack не обновляется, когда меняются ваши файлы. Если 
это как раз то, с чем вы столкнулись, попробуйте использовать команлу `watch-poll`:

    npm run watch-poll

<a name="working-with-stylesheets"></a>
## Работа с таблицами стилей

Файл `webpack.mix.js` - ваша точка входа для компиляции всех ассетов. Считайте его легкой оболочкой для настройки поверх Webpack. Задачи Mix можно связать вместе, чтобы конкретно указать как должны компилироваться ваши ассеты.

<a name="less"></a>
### Less

Метод `less` можно использовать для компилирования [Less](http://lesscss.org/) в CSS. Давайте скомпилируем наш первичный файл `app.less` в `public/css/app.css`.

    mix.less('resources/assets/less/app.less', 'public/css');

Множественные вызовы метода `less` можно использовать для компилирования нескольких файлов:

    mix.less('resources/assets/less/app.less', 'public/css')
       .less('resources/assets/less/admin.less', 'public/css');

Если вы хотите изменить имя файла скомпилированного CSS, вы можете передать полный путь в качестве второго аргумента методу `less`:

    mix.less('resources/assets/less/app.less', 'public/stylesheets/styles.css');

Если нужно переопределить [лежащие в основе plug-in опции Less](https://github.com/webpack-contrib/less-loader#options), то можно передать объект в качестве третьего аргумента `mix.less()`:

    mix.less('resources/assets/less/app.less', 'public/css', {
        strictMath: true
    });

<a name="sass"></a>
### Sass

Метод `sass` позволяет компилировать [Sass](http://sass-lang.com/) в CSS. Вы можете использовать метод следующим образом:

    mix.sass('resources/assets/sass/app.scss', 'public/css');

Как и в случае с методом `less`, вы можете компилировать несколько файлов Sass в их соответствующие CSS-файлы и даже настроить директорию вывода итогового CSS:

    mix.sass('resources/assets/sass/app.sass', 'public/css')
       .sass('resources/assets/sass/admin.sass', 'public/css/admin');

Дополнительные [plug-in опции Node-Sass](https://github.com/sass/node-sass#options) можно указать в качестве третьего аргумента:

    mix.sass('resources/assets/sass/app.sass', 'public/css', {
        precision: 5
    });

<a name="stylus"></a>
### Stylus

Схоже с Less и Sass, метод `stylus` позволяет компилировать [Stylus](http://stylus-lang.com/) в CSS:

    mix.stylus('resources/assets/stylus/app.styl', 'public/css');

Вы также можете установить дополнительные плагины Stylus, такие как [Rupture](https://github.com/jescalan/rupture). Сначала установите требуемый плагин через NPM (`npm install rupture`) и затем затребуйте его в своем вызове `mix.stylus()`:

    mix.stylus('resources/assets/stylus/app.styl', 'public/css', {
        use: [
            require('rupture')()
        ]
    });

<a name="postcss"></a>
### PostCSS

[PostCSS](http://postcss.org/) - мощный инструмент для трансформации CSS, который включен в Laravel Mix. По умолчанию Mix пользуется популярнм плагином [Autoprefixer](https://github.com/postcss/autoprefixer) для автоматического применения всех необходимых вендор-префиксов CSS3. Вы свободно можете добавлять любые дополнительные плагины, которые подходят для вашего приложения. Сначала нужно установить желаемые плагины через NPM и затем обратиться к ним в вашем файле `webpack.mix.js`:

    mix.sass('resources/assets/sass/app.scss', 'public/css')
       .options({
            postCss: [
                require('postcss-css-variables')()
            ]
       });

<a name="plain-css"></a>
### Простой CSS

Если вам просто хотелось бы сконцентрировать некоторые простые таблицы стилей CSS в единый файл, то можно воспользоваться методом `styles`.

    mix.styles([
        'public/css/vendor/normalize.css',
        'public/css/vendor/videojs.css'
    ], 'public/css/all.css');

<a name="url-processing"></a>
### Обработка URL

Так как Laravel Mix строится поверх Webpack, крайне важно понять некоторые концепты Webpack. Для компиляции CSS Webpack будет перезаписывать и оптимизировать любой вызов `url()` в рамках ваших таблицей стилей. Хотя это и может звучать странно, это невероятно мощный функционал. Представьте, что мы хотим скомпилировать Sass, который включает относительный URL, в изображение:

    .example {
        background: url('../images/example.png');
    }

> {note} Абсолютные пути для любого заданного `url()` будут исключены из перезаписывания URL. Например, `url('/images/thing.png')` или `url('http://example.com/images/thing.png')` не будут изменены.

По умолчанию Laravel Mix и Webpack найдут `example.png`, скопируют их в вашу папку `public/images`, а затем перезапишут `url()` в вашей сгенерированной таблице стилей. Таким образом, ваш скомпилированный CSS будет:

    .example {
      background: url(/images/example.png?d41d8cd98f00b204e9800998ecf8427e);
    }

Хотя эта функция и полезна, возможно ваша существующая папка уже настроена так, как вам нравится. Если это именно ваш случай, можно отключить перезаписывание `url()`:

    mix.sass('resources/assets/app/app.scss', 'public/css')
       .options({
          processCssUrls: false
       });

После добавления вышеуказанного в ваш файл `webpack.mix.js`, Mix больше не будет сопоставлять любой `url()` или копировать ассеты в вашу общую директорию. Другими словами, скомпилированный CSS будет выглядеть так же, как вы его и напечатали изначально:

    .example {
        background: url("../images/thing.png");
    }

<a name="css-source-maps"></a>
### Source Maps (файлы с информацией, которая требуется при отладке)

Хотя они и отключены по умолчанию, source maps (файлы с информацией, которая требуется при отладке) можно активировать, вызвав метод `mix.sourceMaps()` в вашем файле `webpack.mix.js`. Хотя это и связано с ценой компиляции/производительности, это обеспечит дополнительную отладочную информацию инструментам разработчика вашего браузера при использовании скомпилированных ассетов.

    mix.js('resources/assets/js/app.js', 'public/js')
       .sourceMaps();

<a name="working-with-scripts"></a>
## Работа с JavaScript

Mix предоставляет несколько функций для работы с JavaScript-файлами, например: компилирование ECMAScript 2015, бандлинг модулей, минификация и простая конкатенация простых JavaScript-файлов. Даже лучше: это все работает незаметно для пользователя, не требуя ни грамма пользовательской настройки:

    mix.js('resources/assets/js/app.js', 'public/js');

Используя эту единственную строку кода теперь вы можете воспользоваться следующими плюсами:

<div class="content-list" markdown="1">
- Синтаксис ES2015.
- Модули
- Компиляция файлов `.vue`.
- Минификация для продакшна.
</div>

<a name="vendor-extraction"></a>
### Извлечение библиотек поставщика

Одним из потенциальных недостатков бандлинга всего JavaScript приложения с вашими библиотеками поставщиков является то, что это затрудняет долгосрочное кэширование. Например, одно обновление вашего кода приложения заставит браузер повторно загружать все ваши библиотеки поставщиков, даже если они не изменились.

Если вы планируете часто обновлять JavaScript своего приложения, то вам следует рассмотреть вариант извлечение всех своих внешних библиотек в отдельный файл. Таким образом, изменение кода вашего приложения не повлияет на кеширование вашего большого файла `vendor.js`. Метод `extract` в Mix делает эту задачу чрезвычайно простой:

    mix.js('resources/assets/js/app.js', 'public/js')
       .extract(['vue'])

Метод `extract` принимает массив всех библиотек или модулей, которые вы бы хотели извлечь в файл `vendor.js`. Используя вышеуказанный сниппет в качестве примера, Mix сгенерирует следующие файлы:

<div class="content-list" markdown="1">
- `public/js/manifest.js`: *Webpack manifest runtime*
- `public/js/vendor.js`: *Ваши библиотеки поставщика*
- `public/js/app.js`: *Код вашего приложения*
</div>

Убедитесь, что загрузили эти файлы в соответствующем порядке, чтобы избежать ошибок JavaScript:

    <script src="/js/manifest.js"></script>
    <script src="/js/vendor.js"></script>
    <script src="/js/app.js"></script>

<a name="react"></a>
### React

Mix может автоматически установить Babel-плагины, необходимые для поддержки React. Для начала замените  `mix.js()` на `mix.react()`:

    mix.react('resources/assets/js/app.jsx', 'public/js');

Mix в фоновом режиме скачает и включит подходящие Babel-плагины `babel-preset-react`.

<a name="vanilla-js"></a>
### Vanilla JS

Схоже с комбинирование таблиц стилей с `mix.styles()`, вы также можете скомбинировать и минифицировать любое количество файлов JavaScript при помощи метода `scripts()`:

    mix.scripts([
        'public/js/admin.js',
        'public/js/dashboard.js'
    ], 'public/js/all.js');

Эта опция особенно полезна для прежних версий проектов, где вам не требовалась компиляция Webpack для вашего JavaScript.

> {tip} Вариация `mix.scripts()` - `mix.babel()`. Сигнатура этого метода идентична `scripts`; однако, конкатенированный файл получит компиляцию Babel, которая переводит любой код ES2015 в vanilla JavaScript, который поймут все браузеры.

<a name="custom-webpack-configuration"></a>
### Пользовательская настройка Webpack

Laravel Mix обращается к преднастроенному файлу `webpack.config.js`. Время от времени вам может потребоваться вручную изменить этот файл. У вас, возможно, есть специальный лоадер или плагин, который нужно указывать, или может вы предпочитаете использовать Stylus вместо Sass. В таких случаях у вас будет два выбора:

#### Слияние пользовательской настройки

Mix предоставляет полезный метод `webpackConfig`, который позволит выполнить слияение любых коротких Webpack-переопределений. Это крайне привлекательный выбор, так как от вас не требуется копировать и поддерживать собственную копию файла `webpack.config.js`. Метод `webpackConfig` принимает объект, который должен содержать любую [специальную настройку Webpack](https://webpack.js.org/configuration/), которую вы желаете применить.

    mix.webpackConfig({
        resolve: {
            modules: [
                path.resolve(__dirname, 'vendor/laravel/spark/resources/assets/js')
            ]
        }
    });

#### Пользовательские файлы настроек

Скопируйте файл `node_modules/laravel-mix/setup/webpack.config.js` в корневую директорию вашего проекта, если вы бы хотели полностью изменить свою настройку Webpack. Затем укажите все ссылки `--config` в своем файле `package.json` на этот скопированный конфиг. Если вы решите воспользоваться этим подходом, любые будущие upstream-обновления `webpack.config.js` вашего Mix следует вручную склеивать с вашим измененным файлом.

<a name="copying-files-and-directories"></a>
## Копирование файлов и директорий

Метод `copy` используется для копирования файлов и папок в новое место. Это может пригодиться, когда нужно переместить определенный ассет из директории `node_modules` в вашу папку `public`.

    mix.copy('node_modules/foo/bar.css', 'public/css/bar.css');

При копировании директории метод `copy` сделает структуру директории более плоской. Если нужно поддерживать оригинальную структуру директории, вместо этого следует использовать метод `copyDirectory`:

    mix.copyDirectory('assets/img', 'public/img');

<a name="versioning-and-cache-busting"></a>
## Версии файлов /очистка кэша

Многие разработчики добавляют в имена ресурсов время создания или уникальный токен, чтобы браузер загружал свежие ресурсы вместо обработки устаревшего кода. В Mix для этого служит метод `version`.

Метод `version` автоматически добавит уникальный хеш к именам всех скомпилированных файлов, что способствует более удобной очистке кэша:

    mix.js('resources/assets/js/app.js', 'public/js')
       .version();

Сгенерировав версию файла, вы можете использовать глобальную функцию Laravel PHP `mix` в ваших [шаблонах](/docs/{{version}}/views) для загрузки соответствующих хешированных ресурсов. Функция `mix` автоматически определит имя хешированного файла:

    <link rel="stylesheet" href="{{ mix('/css/app.css') }}">

Так как во время разработки обычно нет необходимости в версионированных файлах, вы можете указать, чтобы процесс версионировання запускался только во время `npm run production`:

    mix.js('resources/assets/js/app.js', 'public/js');

    if (mix.inProduction()) {
        mix.version();
    }

<a name="browsersync-reloading"></a>
## Перезагрузка Browsersync

[BrowserSync](https://browsersync.io/) автоматически производит обновление в браузере при изменениях в ваших ресурсах. Вы можете использовать метод `mix.browserSync()`:

    mix.browserSync('my-domain.dev');

    // Or...

    // https://browsersync.io/docs/options
    mix.browserSync({
        proxy: 'my-domain.dev'
    });

Этому методу можно передавать либо строку (прокси), либо объект (настройки BrowserSync). Затем запустите дев-сервер Webpack, используя команду `npm run watch`. Теперь, когда вы изменяете скрипт или файл PHP, смотрите как браузер сразу же обновляет страницу, чтобы отразить внесенные изменения.

<a name="environment-variables"></a>
## Переменные среды

Вы можете внедрите переменные среды в Mix, укахав префикс `MIX_` ключу в вашем файле `.env`:

    MIX_SENTRY_DSN_PUBLIC=http://example.com

После того как переменная была задана в вашем файле `.env`, вы можете получить доступ через объект  `process.env`. Если значение меняет во время выполнения задачи `watch`, вам потребуется перезапустить задачу:

    process.env.MIX_SENTRY_DSN_PUBLIC

<a name="notifications"></a>
## Уведомления

Когда доступно, Mix будет автоматически отображать уведомления ОС для каждого бандла. Это даст вам мгновенную обратную связь о том, была ли компиляция успешной или нет. Однако, иногда может потребоваться отключить эти уведомления. Одним из таких примеров может быть запуск Mix на вашем продакшн-сервере. Уведомления можно отключить через метод `disableNotifications`.

    mix.disableNotifications();
