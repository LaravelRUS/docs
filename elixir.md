git 04a07feb482ea8c735658367f986e6c305027ec0

---

# Laravel Elixir

- [Введение](#introduction)
- [Установка и настройка](#installation)
- [Использование](#usage)
- [Gulp](#gulp)
- [Расширения](#extensions)

<a name="introduction"></a>
## Введение

Laravel Elixir предназначен для сборки файлов вашего фронтэнда в css- и js-файлы, а также для выполнения различных задач, построенных на слежении за изменениями в файлах вашего проекта - например, запуска тестов. Laravel Elixir представляет собой гибкий и простой API для [Gulp](http://gulpjs.com).

<a name="installation"></a>
## Установка и настройка

### Установка node.js

Сначала убедитесь, стоит ли у вас node.js:

    node -v

По умолчанию node.js есть в Laravel Homestead. Если вы не используете его, вы можете [установить её вручную](http://nodejs.org/download/). 

### Gulp

Далее, вам нужно глобально установить [Gulp](http://gulpjs.com).

    npm install --global gulp

### Laravel Elixir

Осталось установить собственно Elixir. В корне папки фреймворка вы можете видеть файл `package.json`. Он похож на `composer.json`, только предназначен не для Composer, а для пакетного менеджера node.js , который называется `npm`. Вы можете установить зависимости, заданные в этом файле одной командой:

    npm install

<a name="usage"></a>
## Использование

Команды elixir записываются в файле `gulpfile.js`.

#### Компиляция Less

```javascript
elixir(function(mix) {
    mix.less("app.less");
});
```
В данном примере подразумевается, что ваши less-файлы находятся в `resources/assets/less`.

#### Компиляция Sass

```javascript
elixir(function(mix) {
    mix.sass("app.sass");
});
```

Подразумевается, что ваши sass-файлы находятся в `resources/assets/sass`.

#### Компиляция CoffeeScript

```javascript
elixir(function(mix) {
    mix.coffee();
});
```

Подразумевается, что ваши CoffeeScript-файлы находятся в `resources/assets/coffee`.

#### Компиляция всех Less и CoffeeScript файлов

```javascript
elixir(function(mix) {
    mix.less()
       .coffee();
});
```

#### Запуск PHPUnit тестов

```javascript
elixir(function(mix) {
    mix.phpUnit();
});
```

#### Запуск PHPSpec тестов

```javascript
elixir(function(mix) {
    mix.phpSpec();
});
```

#### Объединение Stylesheets

```javascript
elixir(function(mix) {
    mix.styles([
        "normalize.css",
        "main.css"
    ]);
});
```

Пути задаются относительно директории `resources/css`.

#### Объединение Stylesheets и сохранение в определённое место

```javascript
elixir(function(mix) {
    mix.styles([
        "normalize.css",
        "main.css"
    ], 'public/build/css/everything.css');
});
```

#### Объединение Stylesheets в заданном каталоге

```javascript
elixir(function(mix) {
    mix.styles([
        "normalize.css",
        "main.css"
    ], 'public/build/css/everything.css', 'public/css');
});
```

Третий аргумент в `styles` и `scripts` определяет папку, относительно которой будут искаться заданные файлы.

#### Объединение всех css в папке

```javascript
elixir(function(mix) {
    mix.stylesIn("public/css");
});
```

#### Объединение javascript

```javascript
elixir(function(mix) {
    mix.scripts([
        "jquery.js",
        "app.js"
    ]);
});
```

Как и в случае с css, пути задаются относительно папки `resources/js`

#### Объединение всех js в папке

```javascript
elixir(function(mix) {
    mix.scriptsIn("public/js/some/directory");
});
```

#### Объединение нескольких наборов js

```javascript
elixir(function(mix) {
    mix.scripts(['jquery.js', 'main.js'], 'public/js/main.js')
       .scripts(['forum.js', 'threads.js'], 'public/js/forum.js');
});
```

#### Добавление версии файла

```javascript
elixir(function(mix) {
    mix.version("css/all.css");
});
```

Файл будет сохранён с уникальным именем (к имени файла добавляется хэш содержимого и получается что-то вроде `all-16d570a7.css`), чтобы исключить кэширование его на клиенте. 

Внутри ваших шаблонов вы можете использовать хэлпер `elixir()` для указания урла для такого файла:

```html
<link rel="stylesheet" href="{{ elixir("css/all.css") }}">
```

Этот хэлпер считает хэш указанного файла и добавляет его в урл. Все происходит автоматически !

#### Копировать файл в новое место

```javascript
elixir(function(mix) {
    mix.copy('vendor/foo/bar.css', 'public/css/bar.css');
});
```

#### Копировать каталог в новое место

```javascript
elixir(function(mix) {
    mix.copy('vendor/package/views', 'resources/views');
});
```

#### Соединение методов

Вы можете образовывать цепочки из методов:

```javascript
elixir(function(mix) {
    mix.less("app.less")
       .coffee()
       .phpUnit()
       .version("css/bootstrap.css");
});
```

<a name="gulp"></a>
## Gulp

Для выполнения зарегистрированных команд нужно запустить в командной строке `gulp`

#### Выполнение всех зарегистрированных команд

    gulp

#### Запуск команд при изменении файлов

    gulp watch

#### Запуск тестов при изменении классов

    gulp tdd

> **Note:** По умолчанию подразумевается, что конамды выполняются в development-окружении и собираемые скрипты не минифицируются. Чтобы добавить минификацию запустите `gulp --production`.

<a name="extensions"></a>
## Расширение

Вы можете создавать свои gulp-задачи и добавлять в elixir. 
Например, сделаем шуточную задачу, которая выводит в терминал некое сообщение:

```javascript
 var gulp = require("gulp");
 var shell = require("gulp-shell");
 var elixir = require("laravel-elixir");

 elixir.extend("message", function(message) {

     gulp.task("say", function() {
         gulp.src("").pipe(shell("say " + message));
     });

     return this.queueTask("say");

 });
```

Первый аргумент в `extend` - имя задачи, которое мы будет далее использовать в нашем gulpfile.js , а второй - функция-замыкание с собственно кодом задачи, написанной для gulp.

Вы также можете мониторить изменения в заданных файлах:

```javascript
this.registerWatcher("message", "**/*.php");
```

Когда файл с путём, удовлетворяющем регекспу `**/*.php`, будет изменён - запустится задача `message`.

That's it! You may either place this at the top of your Gulpfile, or instead extract it to a custom tasks file. If you choose the latter approach, simply require it into your Gulpfile, like so:

Вы можете разместить этот код в верхней части вашего `gulpfile.js` , или в сыойм файле, и подключить его в `gulpfile.js` следующей конструкцией:

```javascript
require("./custom-tasks")
```

Дальше вы можете добавлять свою команду в микс:

```javascript
elixir(function(mix) {
    mix.message("Tea, Earl Grey, Hot");
});
```

Теперь, как только gulp исполнит какую-либо задачу, в терминал выведется строка "Tea, Earl Grey, Hot".