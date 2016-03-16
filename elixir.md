git 3c1ff42e25c1ac12261636510a1cdb9c75ef0854

---

# Laravel Elixir

- [Вступление](#introduction)
- [Установка и настройка](#installation)
- [Запуск Elixir](#running-elixir)
- [Работа со стилями](#working-with-stylesheets)
    - [Less](#less)
    - [Sass](#sass)
    - [Простой CSS](#plain-css)
    - [Source Maps](#css-source-maps)
- [Работа со скриптами](#working-with-scripts)
    - [CoffeeScript](#coffeescript)
    - [Browserify](#browserify)
    - [Babel](#babel)
    - [Scripts](#javascript)
- [Копирование файлов и папок](#copying-files-and-directories)
- [Добавление версии / Очистка кеша](#versioning-and-cache-busting)
- [BrowserSync](#browser-sync)
- [Вызов существующих Gulp Tasks](#calling-existing-gulp-tasks)
- [Пишем расширения Elixir](#writing-elixir-extensions)

<a name="introduction"></a>
## Вступление

 
Laravel Elixir предоставляет удобный и гибкий API для исполнения базовых [Gulp](http://gulpjs.com)-задач в вашем приложении. 
Он поддерживает несколько популярных CSS и JavaScript препроцессоров и инструментов тестирования. Декларации css и js выстраиваются в виде цепочки, что позволяет удобно пользоваться функционалом Elixir. Например:

```javascript
elixir(function(mix) {
    mix.sass('app.scss')
       .coffee('app.coffee');
});
```
Если даже вы не поняли сразу как начать работу со сборщиком Gulp и вашими медиа-файлами (assets), вы влюбитесь в Laravel Elixir. Однако вы не обязаны использовать его на этапе разработки вашего приложения. Вы сами можете выбрать инструменты для работы с вашими медиа-файлами (assets), ну и конечно вы можете не использовать их вовсе.  

<a name="installation"></a>
## Установка и настройка

### Установка Node

Перед вызовом Elixir, вы обязаны убедиться в том что Node.js установлен на вашем компьютере.

Before triggering Elixir, you must first ensure that Node.js is installed on your machine.

    node -v


По умолчанию, все что вам нужно это Laravel Homestead; впрочем, если вы не используете Vagrant, тогда вы сможете легко установить Node.js пройдя по ссылке [ссылка для скачивания Node.js](http://nodejs.org/download/).

### Gulp

Далее, вам потребуется [Gulp](http://gulpjs.com) как глобальный NPM пакет:

    npm install --global gulp

Если вы используете систему контроля версий, вы возможно захотите использовать `npm shrinkwrap` для блокировки ваших зависимостей NPM:

     npm shrinkwrap

Выполнив эту команду хотя бы один раз, вы можете свободно добавлять [npm-shrinkwrap.json](https://docs.npmjs.com/cli/shrinkwrap) в контроль версий.

### Laravel Elixir

Остался всего один шаг для установки Elixir! После свежей установки Laravel, вы обнаружите файл `package.json` в корневом каталоге. Взгляните на ваш файл composer.json, он описывает зависимости Node вместо PHP. Вы можете установить описанные зависимости используя команду:


    npm install

Если вы ведете разработку на ОС Windows или вы запускаете вашу VM (виртуальную машину) на ОС Windows, возможно вам понадобится запустить команду `npm install` с ключом `--no-bin-links`, чтобы команда запустилась:

    npm install --no-bin-links

<a name="running-elixir"></a>
## Запуск Elixir

Elixir использует последнюю версию [Gulp](http://gulpjs.com), поэтому для запуска задач Elixir'a вам всего лишь понадобится запустить команду `gulp` в вашей консоли (терминале). Добавление к команде ключа `--production` запустит Elixir с минифицированием ваших CSS стилей и Javascript скриптов:

    // Запуск всех задач...
    gulp

    // Запуск всех задач и минификация CSS и JavaScript...
    gulp --production

#### Наблюдение за изменением ваших медиа файлов и скриптов (assets)

Так как не очень удобно все время запускать команду `gulp` при изменении ваших файлов, вы можете запустить команду `gulp watch`. Эта команда будет будет продолжать работать в вашей консоли и наблюдать за любыми изменениями ваших файлов (assets). Когда произойдут изменения, новые файлы будут автоматически скомпилированы Elixir'ом:

    gulp watch

<a name="working-with-stylesheets"></a>
## Работа со стилями

Файл `gulpfile.js` в корневой папке вашего проекта содержит все задачи Elixir'a, которые могут быть связаны между собой, что позволяет настроить процесс сборки и компиляции ваших файлов (assets) так как вам надо.

<a name="less"></a>
### Less

Для компиляции [Less](http://lesscss.org/) в CSS, вы можете использовать метод `less`. Метод `less` ожидает что ваши Less файлы находятся в папке `resources/assets/less`. По умолчанию задача скомпилирует файлы, указанные в примере, в CSS и сохранит в папку `public/css/app.css`:

```javascript
elixir(function(mix) {
    mix.less('app.less');
});
```
Так же вы можете объединить несколько Less файлов в один CSS файл. И снова скомплированный CSS файл будет сохранен в папке `public/css/app.css`:

```javascript
elixir(function(mix) {
    mix.less([
        'app.less',
        'controllers.less'
    ]);
});
```
Если вам потребуется изменить путь скомпилированного CSS файла, вы можете указать в методе `less` второй аргумент с нужным путем:

```javascript
elixir(function(mix) {
    mix.less('app.less', 'public/stylesheets');
});

// Укажем собственное название скомпилированного файла...
elixir(function(mix) {
    mix.less('app.less', 'public/stylesheets/style.css');
});
```

<a name="sass"></a>
### Sass

Метод `sass` позволяет вам скомпилировать [Sass](http://sass-lang.com/) в CSS. Предполагается что ваши Sass файлы лежат в `resources/assets/sass`, к примеру вы можете использовать метод `sass` так:

```javascript
elixir(function(mix) {
    mix.sass('app.scss');
});
```

И снова, так же как при использовании метода `less`, Вы можете скомпилировать несколько Sass файлов в один CSS файл, и даже изменить папку для скомпилированного CSS:

```javascript
elixir(function(mix) {
    mix.sass([
        'app.scss',
        'controllers.scss'
    ], 'public/assets/css');
});
```

<a name="plain-css"></a>
### Обычный CSS

Если вы просто хотите объединить некиие обычные CSS стили в один файл, вы можете использовать метод `styles`. Пути указанные в этом методе относительны к папке `resources/assets/css` и объединенный CSS файл будет сохранен, как `public/css/all.css`:

```javascript
elixir(function(mix) {
    mix.styles([
        'normalize.css',
        'main.css'
    ]);
});
```

Конечно же, вы так же можете задать альтернативную папку для сохранения в методе `styles`, указав второй аргумент:

```javascript
elixir(function(mix) {
    mix.styles([
        'normalize.css',
        'main.css'
    ], 'public/assets/css');
});
```

<a name="css-source-maps"></a>
### Source Maps (Карта ресурсов)

Source maps (Карты ресурсов) включены и работают из коробки. Для каждого скомпилированного файла вы найдете схожий `*.css.map` файл в папке скомпилированного файла. Эти карты позволяют вам преобразовать скомпилированные Less или Sass файлы для отладки в браузере.

Если вам не нужны source maps (карты ресурсов) для ваших CSS файлов, вы можете выключить их используя опцию:

```javascript
elixir.config.sourcemaps = false;

elixir(function(mix) {
    mix.sass('app.scss');
});
```

<a name="working-with-scripts"></a>
## Работа со скриптами


Elixir также предлагает несколько функций для работы с вашими JavaScript файлами, такими как компиляцию в ECMAScript 6, CoffeeScript, Browserify, минификацию, и простую конкатенацию в обычный JavaScript файлы.

<a name="coffeescript"></a>
### CoffeeScript

Метод `coffee` может быть использован для компиляции [CoffeeScript](http://coffeescript.org/) в обычный Javascript. Этот метод принимает строку или массив CoffeeScript файлов, которые относительны к папке `resources/assets/coffee` и генерирует единственный файл `app.js` в папке `public/js`:


```javascript
elixir(function(mix) {
    mix.coffee(['app.coffee', 'controllers.coffee']);
});
```

<a name="browserify"></a>
### Browserify

Вы можете смело использовать у себя в javascript-коде конструкции из ES6 и JSX - при помощи метода `browserify` Elixir скомпилирует их в стандартный javascript, который понимают все браузеры. 

Эта задача предполагает что ваши скрипты расположены в папке `resources/assets/js`. Она сохранит скомпилированные файлы в папку `public/js/main.js`. Если вы хотите переопределить путь или название для скомпилированного файла, вы можете указать это в качестве второго аргумента:


```javascript
elixir(function(mix) {
    mix.browserify('main.js');
});

// Укажем свое имя для скомпилированного файла...
elixir(function(mix) {
    mix.browserify('main.js', 'public/javascripts/main.js');
});
```
Browserify поддерживает Partialify и Babelify трансформеры, и вы можете легко выбирать и устанавливать их так как захотите:

    npm install aliasify --save-dev

```javascript
elixir.config.js.browserify.transformers.push({
    name: 'aliasify',
    options: {}
});

elixir(function(mix) {
    mix.browserify('main.js');
});
```

<a name="babel"></a>
### Babel


Метод `babel` используется для компиляции [ECMAScript 6 and 7](https://babeljs.io/docs/learn-es2015/) и [JSX](https://facebook.github.io/react/docs/jsx-in-depth.html) в обычный JavaScript. Этот метод принимает массив файлов, расположенных относительно папки `resources/assets/js` и генерирует единственный файл `all.js` в папке `public/js`:

```javascript
elixir(function(mix) {
    mix.babel([
        'order.js',
        'product.js',
        'react-component.jsx'
    ]);
});
```

Для указания нестадартной папки для сохранения вы как обычно можете указать свой путь в качестве второго аргумента. Принцип работы этого скрипта такая же как у метода `mix.scripts()`, за исключенем компиляции Babel.


<a name="javascript"></a>
### Скрипты

Если у вас есть несколько JavaScript файлов и вы бы хотели объединить их в один файл, вы можете воспользоваться методом `scripts`.

Этот метод ожидает что все скрипты расположены в папке `resources/assets/js`, он по умолчанию сохранит объединенный файл JavaScript в папку `public/js/all.js`:


```javascript
elixir(function(mix) {
    mix.scripts([
        'jquery.js',
        'app.js'
    ]);
});
```

If you need to combine multiple sets of scripts into different files, you may make multiple calls to the `scripts` method. The second argument given to the method determines the resulting file name for each concatenation:

```javascript
elixir(function(mix) {
    mix.scripts(['app.js', 'controllers.js'], 'public/js/app.js')
       .scripts(['forum.js', 'threads.js'], 'public/js/forum.js');
});
```

If you need to combine all of the scripts in a given directory, you may use the `scriptsIn` method. The resulting JavaScript will be placed in `public/js/all.js`:

```javascript
elixir(function(mix) {
    mix.scriptsIn('public/js/some/directory');
});
```

<a name="copying-files-and-directories"></a>
## Copying Files & Directories

The `copy` method may be used to copy files and directories to new locations. All operations are relative to the project's root directory:

```javascript
elixir(function(mix) {
	mix.copy('vendor/foo/bar.css', 'public/css/bar.css');
});

elixir(function(mix) {
	mix.copy('vendor/package/views', 'resources/views');
});
```

<a name="versioning-and-cache-busting"></a>
## Versioning / Cache Busting

Many developers suffix their compiled assets with a timestamp or unique token to force browsers to load the fresh assets instead of serving stale copies of the code. Elixir can handle this for you using the `version` method.

The `version` method accepts a file name relative to the `public` directory, and will append a unique hash to the filename, allowing for cache-busting. For example, the generated file name will look something like: `all-16d570a7.css`:

```javascript
elixir(function(mix) {
    mix.version('css/all.css');
});
```

After generating the versioned file, you may use Laravel's global `elixir` PHP helper function within your [views](/docs/{{version}}/views) to load the appropriately hashed asset. The `elixir` function will automatically determine the name of the hashed file:

    <link rel="stylesheet" href="{{ elixir('css/all.css') }}">

#### Versioning Multiple Files

You may pass an array to the `version` method to version multiple files:

```javascript
elixir(function(mix) {
    mix.version(['css/all.css', 'js/app.js']);
});
```

Once the files have been versioned, you may use the `elixir` helper function to generate links to the proper hashed files. Remember, you only need to pass the name of the un-hashed file to the `elixir` helper function. The helper will use the un-hashed name to determine the current hashed version of the file:

    <link rel="stylesheet" href="{{ elixir('css/all.css') }}">

    <script src="{{ elixir('js/app.js') }}"></script>

<a name="browser-sync"></a>
## BrowserSync

BrowserSync automatically refreshes your web browser after you make changes to your front-end resources. You can use the `browserSync` method to instruct Elixir to start a BrowserSync server when you run the `gulp watch` command:

```javascript
elixir(function(mix) {
    mix.browserSync();
});
```

Once you run `gulp watch`, access your web application using port 3000 to enable browser syncing: `http://homestead.app:3000`. If you're using a domain other than `homestead.app` for local development, you may pass an array of [options](http://www.browsersync.io/docs/options/) as the first argument to the `browserSync` method:

```javascript
elixir(function(mix) {
    mix.browserSync({
    	proxy: 'project.app'
    });
});
```

<a name="calling-existing-gulp-tasks"></a>
## Calling Existing Gulp Tasks

If you need to call an existing Gulp task from Elixir, you may use the `task` method. As an example, imagine that you have a Gulp task that simply speaks a bit of text when called:

```javascript
gulp.task('speak', function() {
    var message = 'Tea...Earl Grey...Hot';

    gulp.src('').pipe(shell('say ' + message));
});
```

If you wish to call this task from Elixir, use the `mix.task` method and pass the name of the task as the only argument to the method:

```javascript
elixir(function(mix) {
    mix.task('speak');
});
```

#### Custom Watchers

If you need to register a watcher to run your custom task each time some files are modified, pass a regular expression as the second argument to the `task` method:

```javascript
elixir(function(mix) {
    mix.task('speak', 'app/**/*.php');
});
```

<a name="writing-elixir-extensions"></a>
## Writing Elixir Extensions

If you need more flexibility than Elixir's `task` method can provide, you may create custom Elixir extensions. Elixir extensions allow you to pass arguments to your custom tasks. For example, you could write an extension like so:

```javascript
// File: elixir-extensions.js

var gulp = require('gulp');
var shell = require('gulp-shell');
var Elixir = require('laravel-elixir');

var Task = Elixir.Task;

Elixir.extend('speak', function(message) {

    new Task('speak', function() {
        return gulp.src('').pipe(shell('say ' + message));
    });

});

// mix.speak('Hello World');
```

That's it! Notice that your Gulp-specific logic should be placed within the function passed as the second argument to the `Task` constructor. You may either place this at the top of your Gulpfile, or instead extract it to a custom tasks file. For example, if you place your extensions in `elixir-extensions.js`, you may require the file from your main `Gulpfile` like so:

```javascript
// File: Gulpfile.js

var elixir = require('laravel-elixir');

require('./elixir-extensions')

elixir(function(mix) {
    mix.speak('Tea, Earl Grey, Hot');
});
```

#### Custom Watchers

If you would like your custom task to be re-triggered while running `gulp watch`, you may register a watcher:

```javascript
new Task('speak', function() {
    return gulp.src('').pipe(shell('say ' + message));
})
.watch('./app/**');
```
