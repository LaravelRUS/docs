git fb1cb8292f4c85218bb202c94a4b543779725c65

---

# Юнит-тесты

- [Введение](#introduction)
- [Написание и запуск тестов](#defining-and-running-tests)
- [Тестовое окружение](#test-environment)
- [Обращение к URL](#calling-routes-from-tests)
- [Тестирование фасадов](#mocking-facades)
- [Проверки (assertions)](#framework-assertions)
- [Вспомогательные методы](#helper-methods)
- [Ресет IoC-контейнера Laravel](#refreshing-the-application)

<a name="introduction"></a>
## Введение

Laravel построен с учётом того, что современная профессиональная разработка немыслима без юнит-тестирования. Поддержка PHPUnit доступна "из коробки", а файл `phpunit.xml` уже настроен для вашего приложения. 

Папка `tests` уже содержит файл теста для примера. После установки нового приложения Laravel просто выполните команду `phpunit` для запуска процесса тестирования.

<a name="defining-and-running-tests"></a>
## Написание и запуск тестов

Для создания теста просто создайте новый файл в папке `tests`. Класс теста должен наследовать класс `TestCase`. Вы можете объявлять методы тестов как вы обычно объявляете их для PHPUnit.

#### Пример тестового класса

	class FooTest extends TestCase {

		public function testSomethingIsTrue()
		{
			$this->assertTrue(true);
		}

	}

Вы можете запустить все тесты в вашем приложении командой `phpunit` в терминале.

> **Примечание:** если вы определили собственный метод `setUp`, не забудьте вызвать `parent::setUp`.

<a name="test-environment"></a>
## Тестовое окружение

Во время выполнения тестов Laravel автоматически установит текущую среду в `testing`. Кроме этого Laravel подключит настройки тестовой среды для сессии (`session`) и кэширования (`cache`). Оба эти драйвера устанавливаются в `array`, что позволяет данным существовать в памяти, пока работают тесты. Вы можете свободно создать любое другое тестовое окружение по необходимости.

Переменные тестовой среды исполнения (`testing`) можно изменить в файле `phpunit.xml`.

<a name="calling-routes-from-tests"></a>
## Обращение к URL

#### Вызов URL из теста

Вы можете легко вызвать любой ваш URL методом `call`:

	$response = $this->call('GET', 'user/profile');

	$response = $this->call($method, $uri, $parameters, $files, $server, $content);

После этого вы можете обращаться к свойствам объекта `Illuminate\Http\Response`:

	$this->assertEquals('Hello World', $response->getContent());

#### Вызов контроллера из теста

Вы также можете вызвать из теста любой контроллер.

	$response = $this->action('GET', 'HomeController@index');

	$response = $this->action('GET', 'UserController@profile', array('user' => 1));

> **Примечание:** Нет необходимости указывать полный неймспейс в методе `action` - `App\Http\Controllers` можно опустить.

Метод `getContent` вернёт содержимое-строку ответа роута или контроллера. Если был возвращён `View` вы можете получить его через свойство `original`:

	$view = $response->original;

	$this->assertEquals('John', $view['name']);

Для вызова HTTPS-маршрута можно использовать метод `callSecure`:

	$response = $this->callSecure('GET', 'foo/bar');

<a name="mocking-facades"></a>
## Тестирование фасадов

При тестировании вам может потребоваться отловить вызов (mock a call) к одному из статических классов-фасадов Laravel. К примеру, у вас есть такой контроллер:

	public function getIndex()
	{
		Event::fire('foo', ['name' => 'Дейл']);

		return 'All done!';
	}

Вы можете отловить обращение к `Event` с помощью метода `shouldReceive` этого фасада, который вернёт объект [Mockery](https://github.com/padraic/mockery).

#### Мок (mocking) фасада `Event`

	public function testGetIndex()
	{
		Event::shouldReceive('fire')->once()->with(['name' => 'Дейл']);

		$this->call('GET', '/');
	}

> **Примечание:** не делайте этого для объекта `Request`. Вместо этого передайте желаемый ввод методу `call` во время выполнения вашего теста.

<a name="framework-assertions"></a>
## Проверки (assertions)

Laravel предоставляет несколько `assert`-методов, чтобы сделать ваши тесты немного проще.

#### Проверка на успешный запрос

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertResponseOk();
	}

#### Проверка статуса ответа

	$this->assertResponseStatus(403);

#### Проверка переадресации в ответе

	$this->assertRedirectedTo('foo');

	$this->assertRedirectedToRoute('route.name');

	$this->assertRedirectedToAction('Controller@method');

#### Проверка наличия данных в шаблоне

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertViewHas('name');
		$this->assertViewHas('age', $value);
	}

#### Проверка наличия данных в сессии

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertSessionHas('name');
		$this->assertSessionHas('age', $value);
	}

#### Проверка на наличие ошибок в сессии 

    public function testMethod()
    {
        $this->call('GET', '/');

        $this->assertSessionHasErrors();

        // Asserting the session has errors for a given key...
        $this->assertSessionHasErrors('name');

        // Asserting the session has errors for several keys...
        $this->assertSessionHasErrors(array('name', 'age'));
    }

#### Проверка на наличие "старого пользовательского ввода"

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertHasOldInput();
	}

<a name="helper-methods"></a>
## Вспомогательные методы

Класс `TestCase` содержит несколько вспомогательных методов для упрощения тестирования вашего приложения.

#### Установка текущего авторизованного пользователя

Вы можете установить текущего авторизованного пользователя с помощью метода `be`:

	$user = new User(array('name' => 'John'));

	$this->be($user);

Вы можете заполнить вашу БД начальными данными изнутри теста методом `seed`.

**Заполнение БД тестовыми данными

	$this->seed();

	$this->seed($connection);

Больше информации на тему начальных данных доступно в разделе [Миграции и начальные данные](/docs/{{version}}/migrations#database-seeding).

<a name="refreshing-the-application"></a>
## Ресет IoC-контейнера Laravel

Как вы уже знаете, в любой части теста вы можете получить доступ к IoC-контейнеру приложения Laravel при помощи `$this->app`. Объект приложения Laravel обновляется для каждого тестового класса, но не метода. Если вы хотите вручную ресетить объект приложения Laravel в произвольном месте, вы можете это сделать при помощи метода `refreshApplication`. Это сбросит все моки (mock) и другие дополнительные биндинги, которые были сделаны с момента запуска тест-сессии.
