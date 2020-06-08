git 1a9c864883f6ce316c5573e3a50f71d216e0eeb2

---

# Тесты браузера (Laravel Dusk)

- [Введение](#introduction)
- [Установка](#installation)
    - [Использование других браузеров](#using-other-browsers)
    - [Опции ChromeDriver](#chromedriver-options)
- [Начало работы](#getting-started)
    - [Генерация тестов](#generating-tests)
    - [Выполнение тестов](#running-tests)
    - [Обработка среды](#environment-handling)
    - [Создание браузеров](#creating-browsers)
    - [Аутентификация](#authentication)
- [Взаимодействие с элементами](#interacting-with-elements)
    - [Нажатие ссылок](#clicking-links)
    - [Текст, значения и атрибуты](#text-values-and-attributes)
    - [Использование форм](#using-forms)
    - [Прикрепление файлов](#attaching-files)
    - [Использование клавиатуры](#using-the-keyboard)
    - [Использование мыши](#using-the-mouse)
    - [Обзор данных селекторов](#scoping-selectors)
    - [Ожидание элементов](#waiting-for-elements)
- [Доступные утверждения](#available-assertions)
- [Страницы](#pages)
    - [Генерация страниц](#generating-pages)
    - [Настройка страниц](#configuring-pages)
    - [Переход на страницы](#navigating-to-pages)
    - [Сокращённые селекторы](#shorthand-selectors)
    - [Методы страниц](#page-methods)
- [Продолжительная интеграция](#continuous-integration)
    - [Travis CI](#running-tests-on-travis-ci)
    - [CircleCI](#running-tests-on-circle-ci)

<a name="introduction"></a>
## Введение

Laravel Dusk предоставляет выразительный, простой в использовании API для автоматизации браузера и тестирования. По умолчанию Dusk не требует установки JDK или Selenium. Вместо этого Dusk использует отдельную установку [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home). Однако, вы можете свободно использовать любой другой совместимый с Selenium драйвер по собственному желанию.

<a name="installation"></a>
## Установка

Для начала вам нужно добавить Composer-зависимость `laravel/dusk` к своему проекту:

    composer require --dev laravel/dusk:"^1.0"

После установки Dusk нужно зарегистрировать сервис-провайдер `Laravel\Dusk\DuskServiceProvider`. Вам нужно зарегистрировать провайдер в методе `register` вашего `AppServiceProvider`, чтобы ограничить среды, в которых доступен Dusk, т.к. это открывает возможность входить как другие пользователи:

    use Laravel\Dusk\DuskServiceProvider;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        if ($this->app->environment('local', 'testing')) {
            $this->app->register(DuskServiceProvider::class);
        }
    }

Затем выполните Artisan-команду `dusk:install`:

    php artisan dusk:install

Директория `Browser` будет создана в вашей директории `tests` и будет содержать пример теста. Затем нужно установить переменную среды `APP_URL` в своем файле `.env`. Это значение должно совпадать с URL, который вы используете для доступа к своему приложению в браузере.

Чтобы запустить свои тесты используйте Artisan-команду `dusk`. Команда `dusk` принимает любой аргумент, который также принимается и командой `phpunit`:

    php artisan dusk

<a name="using-other-browsers"></a>
### Использование других браузеров

По умолчанию Dusk использует Google Chrome и отдельную установку [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home) для выполнения ваших тестов. Однако, вы все еще можете запустить собственный Selenium-сервер и выполнить свои тесты в абсолютно любом браузере.

Для начала откройте свой файл `tests/DuskTestCase.php`, который представляет собой базовый тест-кейс Dusk для вашего приложения. В этом файле можно убрать вызов метода `startChromeDriver`. Это остановит Dusk от автоматического запуска ChromeDriver:

    /**
     * Prepare for Dusk test execution.
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

Затем вы можете просто изменить метод `driver`, чтобы он подключался к URL и порту по вашему выбору. Дополнительно можно изменить желаемые возможности, "desired capabilities", которые следует передавать WebDriver:

    /**
     * Create the RemoteWebDriver instance.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
        );
    }

<a name="chromedriver-options"></a>
### Опции ChromeDriver

Для изменения сессии ChromeDriver можно изменить метод `driver` класса `DuskTestCase`:

    use Facebook\WebDriver\Chrome\ChromeOptions;

    /**
     * Create the RemoteWebDriver instance.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        $options = (new ChromeOptions)->addArguments(['--headless']);

        return RemoteWebDriver::create(
            'http://localhost:9515', DesiredCapabilities::chrome()->setCapability(
                ChromeOptions::CAPABILITY, $options
            )
        );
    }

<a name="getting-started"></a>
## Начало работы

<a name="generating-tests"></a>
### Генерация тестов

Чтобы сгенерировать Dusk-тест используйте Artisan-команду `dusk:make`. Сгенерированный тест будет размещён в директории `tests/Browser`:

    php artisan dusk:make LoginTest

<a name="running-tests"></a>
### Выполнение тестов

Используйте Artisan-команду `dusk` для запуска своих тестов браузера:

    php artisan dusk

Команда `dusk` принимает любой аргумент, который обычно принимается тест-раннером PHPUnit, позволяя выполнять тесты только для заданной [группы](https://phpunit.de/manual/current/en/appendixes.annotations.html#appendixes.annotations.group), и т.д.:

    php artisan dusk --group=foo

#### Запуск ChromeDriver вручную

По умолчанию Dusk будет автоматически пытаться запустить ChromeDriver. Если это не работает для вашей конкретной системы, можно вручную запустить ChromeDriver перед выполнением команды `dusk`. Если вы выбрали запуск ChromeDriver вручную, нужно закомментировать следующую строку в файле `tests/DuskTestCase.php`:

    /**
     * Prepare for Dusk test execution.
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

Дополнительно, если вы запустите ChromeDriver на порте, отличном от 9515, нужно изменить метод `driver` того же класса:

    /**
     * Create the RemoteWebDriver instance.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:9515', DesiredCapabilities::chrome()
        );
    }

<a name="environment-handling"></a>
### Обработка среды

Чтобы заставить Dusk использовать собственный файл среды при выполнении тестов создайте файл `.env.dusk.{environment}` в корневой директории своего проекта. Например, если вы будете инициировать команду `dusk` из своей среды `local`, вам следует создать файл `.env.dusk.local`.

При выполнении тестов Dusk будет выполнять резервное копирование файла `.env` и переименует вашу Dusk-среду в `.env`. После завершения тестов ваш файл `.env` будет восстановлен.

<a name="creating-browsers"></a>
### Создание браузеров

Для начала давайте напишем тест, который проверяет, что мы можем войти в наше приложение. После генерирования теста мы можем изменить его, чтобы он переходил на страницу входу, вводил некоторые данные для входа, а затем нажимал кнопку "Login". Для создания экземпляра браузера вызовите метод `browse`:

    <?php

    namespace Tests\Browser;

    use App\User;
    use Tests\DuskTestCase;
    use Laravel\Dusk\Chrome;
    use Illuminate\Foundation\Testing\DatabaseMigrations;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;

        /**
         * A basic browser test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $user = factory(User::class)->create([
                'email' => 'taylor@laravel.com',
            ]);

            $this->browse(function ($browser) use ($user) {
                $browser->visit('/login')
                        ->type('email', $user->email)
                        ->type('password', 'secret')
                        ->press('Login')
                        ->assertPathIs('/home');
            });
        }
    }

Как видно в примере выше, метод `browse` принимает анонимную функцию. Экземпляр браузера будет автоматически передаваться Dusk этой функции и она служит основным объектом, используемым для взаимодействия и создания утверждений касательно вашего приложения.

> {tip} Этот тест можно использовать для проверки экрана входа, генерируемого Artisan-командой `make:auth`.

#### Создание нескольких браузеров

Иногда для правильного выполнения тестирования вам может потребоваться несколько браузеров. Например, несколько браузеров может понадобится для проверки экрана чата, который взаимодействует с веб-сокетами. Чтобы создать несколько браузеров просто "попросите" использовать более одного браузера в сигнатуре анонимной функции, передаваемой методу `browse`:

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home')
              ->waitForText('Message');

        $second->loginAs(User::find(2))
               ->visit('/home')
               ->waitForText('Message')
               ->type('message', 'Hey Taylor')
               ->press('Send');

        $first->waitForText('Hey Taylor')
              ->assertSee('Jeffrey Way');
    });

<a name="authentication"></a>
### Аутентификация

Вы часто будете тестировать страницы, которые требуют аутентификации. Можно использовать метод Dusk `loginAs`, чтобы избежать взаимодействия с экраном входа во время каждого теста. Метод `loginAs` принимает ID пользователя или экземпляр модели пользователя:

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home');
    });

> {note} После использования метода `loginAs` сессия пользователя будет поддерживаться для всех тестов в файле.

<a name="interacting-with-elements"></a>
## Взаимодействие с элементами

<a name="clicking-links"></a>
### Нажатие ссылок

Можно использовать метод `clickLink` на экземпляре браузера, чтобы нажать ссылку. Метод `clickLink` будет нажимать ссылку, которая предоставляется отображаемым текстом:

    $browser->clickLink($linkText);

> {note} Этот метод взаимодействует с jQuery. Если на странице не доступен jQuery, Dusk автоматически внедрит его на страницу, чтобы он был доступен на всем протяжении тестирования.

<a name="text-values-and-attributes"></a>
### Текст, значения и атрибуты

#### Получение и установка значений

Dusk предоставляет несколько методов для взаимодействия с текущим отображаемым текстом, значениями и атрибутами элементов на странице. Например, чтобы получить "value" элемента, который совпадает с данным селектором, используйте метод `value`:

    // Retrieve the value...
    $value = $browser->value('selector');

    // Set the value...
    $browser->value('selector', 'value');

#### Получение текста

Метод `text` можно использовать для получения отображаемого текста элемента, который совпадает с заданным селектором:

    $text = $browser->text('selector');

#### Получение атрибутов

Наконец, метод `attribute` можно использовать для получения атрибута элемента, который совпадает с заданным селектором:

    $attribute = $browser->attribute('selector', 'value');

<a name="using-forms"></a>
### Использование форм

#### Вводимые значения

Dusk предоставляет множество методов взаимодействия с формами и элементами ввода. Сначала давайте взглянем на пример ввода текста в поле ввода:

    $browser->type('email', 'taylor@laravel.com');

Обратите внимание, что хотя метод и принимает его при необходимости, нам необязательно передавать CSS-селектор в метод `type`. Если CSS-селектор не был предоставлен, Dusk будет искать заданный атрибут `name`. Наконец, Dusk попытается найти `textarea` с заданным атрибутом `name`.

Вы можете "очистить" значение ввода, используя метод `clear`:

    $browser->clear('email');

#### Выпадающие списки

Для выбора значения из выпадающего списка можно использовать метод `select`. Как и методу `type`, методу `select` не требуется полный CSS-селектор. При передаче значения методу `select` вам следует передавать значение исходного значения вместо отображаемого текста:

    $browser->select('size', 'Large');

Вы можете выбрать случайную опцию, опустив второй параметр:

    $browser->select('size');

#### Чекбоксы

Чтобы "пометить" поле-чекбокс можно использовать метод `check`. Как и любых других методах, связанных с вводом, полный CSS-селектор не требуется. Если нельзя найти точное совпадение с селектором, Dusk будет искать чекбокс с совпадающим атрибутом `name`:

    $browser->check('terms');

    $browser->uncheck('terms');

#### Радиокнопки

Для выбора варианта радиокнопки можно использовать метод `radio`. Как и любых других методах, связанных с вводом, полный CSS-селектор не требуется. Если нельзя найти точное совпадение с селектором, Dusk будет искать радиокнопку с совпадающими атрибутами `name` и `value`:

    $browser->radio('version', 'php7');

<a name="attaching-files"></a>
### Прикрепление файлов

Методом `attach` можно пользоваться для прикрепления файла к элементу ввода `file`. Как и любых других методах, связанных с вводом, полный CSS-селектор не требуется. Если нельзя найти точное совпадение с селектором, Dusk будет искать ввод файла с совпадающим атрибутом `name`:

    $browser->attach('photo', __DIR__.'/photos/me.png');

<a name="using-the-keyboard"></a>
### Использование клавиатуры

Метод `keys` позволяет предоставлять более сложные последовательности ввода заданному элементу, чем обычно дозволяется методом `type`. Например, можно удерживать клавиши модификаторы, вводя значения. В этом примере будет удерживаться клавиша `shift` во время ввода `taylor` в элемент, совпадающий с заданным селектором. После того как было напечатано `taylor`, мы напечатаем `otwell` без каких-либо клавиш-модификаторов:

    $browser->keys('selector', ['{shift}', 'taylor'], 'otwell');

Первичному CSS-селектору, который содержит ваше приложение, можно даже отправлять "горячие клавиши":

    $browser->keys('.app', ['{command}', 'j']);

> {tip} Все клавиши-модификаторы оборачиваются в символы `{}` и совпадают с константами, определенным в классе `Facebook\WebDriver\WebDriverKeys`, который можно [найти на GitHub](https://github.com/php-webdriver/php-webdriver/blob/master/lib/WebDriverKeys.php).

<a name="using-the-mouse"></a>
### Использование мыши

#### Нажатие на элементы

Метод `click` используется для "кликов" по элементам, совпадающим с заданным селектором:

    $browser->click('.selector');

#### Наведение курсора мыши

Метод `mouseover` используется в случае, когда вам нужно переместить курсор на элемент, совпадающий с заданным селектором:

    $browser->mouseover('.selector');

#### Перетаскивание

Метод `drag` используется для перетаскивания элемента, совпадающего с заданным селектором, к другому элементу:

    $browser->drag('.from-selector', '.to-selector');

Либо можно перетащить элемент в одном направлении:

    $browser->dragLeft('.selector', 10);
    $browser->dragRight('.selector', 10);
    $browser->dragUp('.selector', 10);
    $browser->dragDown('.selector', 10);

<a name="scoping-selectors"></a>
### Обзор данных селекторов

Иногда вам может потребоваться выполнить несколько операций во время обзора данных всех операций в рамках данного селектора. Например, вам нужно удостовериться, что какой-то текст выходит только в таблице и затем нажать на кнопку внутри той таблицы. Для достижения поставленной цели можно воспользоваться методом `with`. Все операции, выполняемые в рамках анонимной функции, предоставляемой методу `with`, будут охвачены в оригинальный селектор:

    $browser->with('.table', function ($table) {
        $table->assertSee('Hello World')
              ->clickLink('Delete');
    });

<a name="waiting-for-elements"></a>
### Ожидание элементов

При тестировании приложений, которые широко используют JavaScript, часто нужно "подождать" чтобы стали доступны определенные элементы или данные, прежде чем продолжить выполнение теста. В Dusk это очень просто. Используя множество методов можно подождать, чтобы элементы стали видимы на странице или даже пока заданное JavaScript-выражение не выдаст `true`.

#### Ожидание

Если нужно приостановить тест на заданное количество миллисекунд, используйте метод `pause`:

    $browser->pause(1000);

#### Ожидание селекторов

Метод `waitFor` можно использовать для приостановки выполнения теста до того момента, пока на странице не будет отображен элемент, совпадающий с заданным CSS-селектором. По умолчанию это приостановит тест максимум на пять секунд прежде чем выбросить исключение. При необходимости в качестве второго аргумента методу можно передать свой порог времени:

    // Wait a maximum of five seconds for the selector...
    $browser->waitFor('.selector');

    // Wait a maximum of one second for the selector...
    $browser->waitFor('.selector', 1);

Вам также может потребоваться подождать, пока заданный селектор не пропадёт со страницы:

    $browser->waitUntilMissing('.selector');

    $browser->waitUntilMissing('.selector', 1);

#### Обзор селекторов когда они доступны
Время от времени может потребоваться подождать определенный селектор, а затем взаимодействовать с элементом, совпадающим с заданным селектором. Например, нужно подождать пока не станет доступным модальное окно, а затем нажать кнопку "OK" в этом окне. В таком случае пригодится метод `whenAvailable`. Все операции с элементами, выполняемые в рамках заданной функции, будут переданы для обзора оригинальному селектору:

    $browser->whenAvailable('.modal', function ($modal) {
        $modal->assertSee('Hello World')
              ->press('OK');
    });

#### Ожидание текста

Метод `waitForText` используется для ожидания появления заданного текста на странице:

    // Wait a maximum of five seconds for the text...
    $browser->waitForText('Hello World');

    // Wait a maximum of one second for the text...
    $browser->waitForText('Hello World', 1);

#### Ожидание ссылок

Метод `waitForLink` используется для ожидания появления заданного текста-ссылки на странице:

    // Wait a maximum of five seconds for the link...
    $browser->waitForLink('Create');

    // Wait a maximum of one second for the link...
    $browser->waitForLink('Create', 1);

#### Ожидание на местоположении страницы

При утверждении пути, такого как `$browser->assertPathIs('/home')`, утверждение может быть неудачным, если `window.location.pathname` обновляется асинхронно. Можно использовать метод `waitForLocation`, чтобы подождать пока местоположение не будет равно указанному значению:

    $browser->waitForLocation('/secret');

#### Ожидание перезагрузки страницы

Если нужно сделать утверждение после перезагрузки страницы, то используйте метод `waitForReload`:

    $browser->click('.some-action')
            ->waitForReload()
            ->assertSee('something');

#### Ожидание JavaScript-выражений

Иногда требуется приостановить выполнение теста до тех пор, пока заданное JavaScript-выражение не будет равным `true`. Здесь поможет метод `waitUntil`. При передаче выражения этому методу вам не нужно включать ключевое слово `return` или точку с запятой в конце:

    // Wait a maximum of five seconds for the expression to be true...
    $browser->waitUntil('App.dataLoaded');

    $browser->waitUntil('App.data.servers.length > 0');

    // Wait a maximum of one second for the expression to be true...
    $browser->waitUntil('App.data.servers.length > 0', 1);

#### Ожидание с анонимной функцией

Многие из "wait"-методов в Dusk основываются на методе `waitUsing`. Этот метод можно использовать и напрямую, чтобы подождать пока заданная анонимная функция не вернет `true`. Метод `waitUsing` принимает максимальное количество секунд ожидания, интервал, при котором следует оценить функцию, анонимную функцию, а также необязательное сообщение о неудачном выполнении:

    $browser->waitUsing(10, 1, function () use ($something) {
        return $something->isReady();
    }, "Something wasn't ready in time.");

<a name="available-assertions"></a>
## Доступные утверждения

Dusk предоставляет множество утверждений, которые можно сделать касательно вашего приложения. Все они задокументированы в таблице ниже:

Утверждение  | Описание
------------- | -------------
`$browser->assertTitle($title)`  |  Заголовок страницы совпадает с заданным текстом.
`$browser->assertTitleContains($title)`  |  В заголовке страницы содержится заданный текст.
`$browser->assertPathBeginsWith($path)`  |  Текущий путь URL начинается с заданного пути.
`$browser->assertPathIs('/home')`  |  Текущий пути совпадает с заданным путём.
`$browser->assertPathIsNot('/home')`  |  Текущий путь не совпадает с заданным путём.
`$browser->assertRouteIs($name, $parameters)`  |  Текущий URL совпадает с заданным URL именованного роута.
`$browser->assertQueryStringHas($name, $value)`  |  Заданный параметр строки запроса присутствует и имеет заданное значение.
`$browser->assertQueryStringMissing($name)`  |  Заданный параметр строки запроса отсутствует.
`$browser->assertHasQueryStringParameter($name)`  |  Заданный параметр строки запроса присутствует.
`$browser->assertHasCookie($name)`  |  Заданный cookie присутствует.
`$browser->assertCookieValue($name, $value)`  |  В cookie есть заданное значение.
`$browser->assertPlainCookieValue($name, $value)`  |  В незашифрованном cookie есть заданное значение.
`$browser->assertSee($text)`  |  Заданный текст присутствует на странице.
`$browser->assertDontSee($text)`  |  Заданный текст не присутствует на странице.
`$browser->assertSeeIn($selector, $text)`  |  Заданный текст присутствует в селекторе.
`$browser->assertDontSeeIn($selector, $text)`  |  Заданный текст не присутствует в селекторе.
`$browser->assertSourceHas($code)`  |  Заданный исходный код присутствует на странице.
`$browser->assertSourceMissing($code)`  |  Заданный исходный код не присутствует на странице.
`$browser->assertSeeLink($linkText)`  |  Заданная ссылка присутствует на странице.
`$browser->assertSeeLink($link)`  |  Определить видима ли заданная ссылка.
`$browser->assertInputValue($field, $value)`  |  Заданное поле ввода имеет заданное значение.
`$browser->assertInputValueIsNot($field, $value)`  |  Заданное поле ввода не имеет заданное значение.
`$browser->assertChecked($field)`  |  Заданный чекбокс помечен.
`$browser->assertNotChecked($field)`  |  Заданный чекбокс не помечен.
`$browser->assertRadioSelected($field, $value)`  |  Заданное радиополе помечено.
`$browser->assertRadioNotSelected($field, $value)` |  Заданное радиополе не помечено.
`$browser->assertSelected($field, $value)`  |  В заданном выпадающем списке выбрано заданное значение.
`$browser->assertNotSelected($field, $value)`  |  В заданном выпадающем списке не выбрано заданное значение.
`$browser->assertSelectHasOptions($field, $values)`  |  Заданный массив значений доступен для выбора.
`$browser->assertSelectMissingOptions($field, $values)`  |  Заданный массив значений не доступен для выбора.
`$browser->assertSelectHasOption($field, $value)`  |  Заданное значение доступно для выбора в заданном поле.
`$browser->assertValue($selector, $value)`  |  У элемента, совпадающего с заданным селектором, есть заданное значение.
`$browser->assertVisible($selector)`  |  Элемент, совпадающий с заданным селектором, видим.
`$browser->assertMissing($selector)`  |  Элемент, совпадающий с заданным селектором, невидим.
`$browser->assertDialogOpened($message)`  |  Был открыт JavaScript-диалог с заданным сообщением.

<a name="pages"></a>
## Страницы

Иногда в тестах требуется последовательно выполнить несколько сложных действий. Это может сделать ваши тесты куда более сложными для прочтения и понимания. Страницы позволяют определять действия, которые затем можно будет выполнить на заданной странице, используя один метод. Страницы также позволяют задать быстрый доступ к обычным селекторам для вашего приложения или одной страницы.

<a name="generating-pages"></a>
### Генерация страниц

Чтобы сгенерировать объект страницы используйте Artisan-команду `dusk:page`. Все объекты страницы будут помещены в директорию `tests/Browser/Pages`:

    php artisan dusk:page Login

<a name="configuring-pages"></a>
### Настройка страниц

По умолчанию у страниц есть три метода: `url`, `assert` и `elements`. Сейчас мы обсудим методы `url` и `assert`. Метод `elements` мы [обсудим более детально ниже](#shorthand-selectors).

#### Метод `url`

Метод `url` должен возвращать путь URL, который представляет страницу. Dusk будет использовать этот URL при навигации на страницу в браузере:

    /**
     * Get the URL for the page.
     *
     * @return string
     */
    public function url()
    {
        return '/login';
    }

#### Метод `assert`

Метод `assert` может делать утверждения, необходимые для проверки того, что браузер действительно находится на заданной странице. Нет необходимости завершать этот метод; однако, при необходимости вы можете сделать эти утверждения. Они будут запущены автоматически при переходе на страницу:

    /**
     * Assert that the browser is on the page.
     *
     * @return void
     */
    public function assert(Browser $browser)
    {
        $browser->assertPathIs($this->url());
    }

<a name="navigating-to-pages"></a>
### Переход на страницы

Как только была настроена страница, вы можете перейти на нее, используя метод `visit`:

    use Tests\Browser\Pages\Login;

    $browser->visit(new Login);

Иногда вы уже можете находиться на заданной странице и надо "загрузить" селекторы страницы и методы в контекст текущего теста. Это используется при нажатии кнопки и редиректе на заданную страницу без явного перехода на нее. В данной ситуации используйте метод `on` для загрузки страницы:

    use Tests\Browser\Pages\CreatePlaylist;

    $browser->visit('/dashboard')
            ->clickLink('Create Playlist')
            ->on(new CreatePlaylist)
            ->assertSee('@create');

<a name="shorthand-selectors"></a>
### Сокращённые селекторы

Метод `elements` страниц позволяет задать быстрые, простые в запоминании ярлыки для любого CSS-селектора на вашей странице. Например, давайте определим ярлык для поля ввода "email" страницы входа приложения:

    /**
     * Get the element shortcuts for the page.
     *
     * @return array
     */
    public function elements()
    {
        return [
            '@email' => 'input[name=email]',
        ];
    }

Теперь можно использовать этот сокращённый селектор везде, где вы бы использовали полный CSS-селектор:

    $browser->type('@email', 'taylor@laravel.com');

#### Глобальные сокращённые селекторы

После установки Dusk в директорию `tests/Browser/Pages`  будет помещен базовый класс `Page`. Этот класс содержит метод `siteElements`, который можно использовать для определения глобальных сокращённых селекторов, которые должны быть доступны на каждой странице вашего приложения:

    /**
     * Get the global element shortcuts for the site.
     *
     * @return array
     */
    public static function siteElements()
    {
        return [
            '@element' => '#selector',
        ];
    }

<a name="page-methods"></a>
### Методы страниц

В дополнение к методам по умолчанию, заданным на страницах, можно задать и дополнительные методы, которые можно использовать в тестах. Например, давайте представим, что мы создаем приложение для управления музыкой. Обычным действием для одной страницы приложения может быть создание плейлиста. Вместо того, чтобы переписывать логику для создания плейлиста в каждом тесте, можно задать метод `createPlaylist` на классе страницы:

    <?php

    namespace Tests\Browser\Pages;

    use Laravel\Dusk\Browser;

    class Dashboard extends Page
    {
        // Other page methods...

        /**
         * Create a new playlist.
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  string  $name
         * @return void
         */
        public function createPlaylist(Browser $browser, $name)
        {
            $browser->type('name', $name)
                    ->check('share')
                    ->press('Create Playlist');
        }
    }

После того как был задан этот метод, вы можете использовать его в любом тесте, который использует страницу. Экземпляр браузера будет автоматически передаваться методу страницы:

    use Tests\Browser\Pages\Dashboard;

    $browser->visit(new Dashboard)
            ->createPlaylist('My Playlist')
            ->assertSee('My Playlist');

<a name="continuous-integration"></a>
## Продолжительная интеграция

<a name="running-tests-on-travis-ci"></a>
### Travis CI

Для запуска ваших Dusk-тестов на Travis CI нам потребуется использовать "sudo-enabled" среду Ubuntu 14.04 (Trusty). Так как Travis CI - не графическая среда, нужно предпринять несколько дополнительных шагов, чтобы запустить браузер Chrome. А также мы будем использовать команду `php artisan serve`, чтобы запустить встроенный веб-сервер PHP:

    sudo: required
    dist: trusty

    before_script:
        - export DISPLAY=:99.0
        - sh -e /etc/init.d/xvfb start
        - ./vendor/laravel/dusk/bin/chromedriver-linux &
        - cp .env.testing .env
        - php artisan serve > /dev/null 2>&1 &

    script:
        - php artisan dusk

<a name="running-tests-on-circle-ci"></a>
### CircleCI

#### CircleCI 1.0

Если вы используете CircleCI 1.0 для запуска своих Dusk-тестов, можно использовать этот конфиг в роли отправной точки. Как и с TravisCI, мы будем использовать команду `php artisan serve`, чтобы запустить встроенный веб-сервер PHP:

    test:
        pre:
            - "./vendor/laravel/dusk/bin/chromedriver-linux":
                background: true
            - cp .env.testing .env
            - "php artisan serve":
                background: true

        override:
            - php artisan dusk

 #### CircleCI 2.0

Если вы используете CircleCI 2.0 для запуска своих Dusk-тестов, можно добавить эти шаги к вашему билду:

     version: 2
     jobs:
         build:
             steps:
                  - run:
                      name: Start Chrome Driver
                      command: ./vendor/laravel/dusk/bin/chromedriver-linux
                      background: true
                 - run:
                     name: Run Laravel Server
                     command: php artisan serve
                     background: true
                 - run:
                     name: Run Laravel Dusk Tests
                     command: php artisan dusk
