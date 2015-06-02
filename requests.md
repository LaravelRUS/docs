git 4deba2bfca6636d5cdcede3f2068eff3b59c15ce

---

# HTTP Requests (HTTP-запросы)

- [Получение объекта HTTP-запроса](#obtaining-a-request-instance)
- [Входных данных](#retrieving-input)
- [Предыдущие входные данные](#old-input)
- [Куки](#cookies)
- [Файлы](#files)
- [Other Request Information](#other-request-information)

<a name="obtaining-a-request-instance"></a>
## Получение объекта HTTP-запроса

### При помощи фасада

Фасад `Request` дает доступ к объекту HTTP-запроса:

	$name = Request::input('name');

Не забудьте использовать конструкцию `use Request;` в начале файла класса.

### При помощи DI (dependency injection)

Можно получить объект HTTP-запроса при помощи DI (dependency injection, внедрение зависимости). Способ заключается в том, что в аргументы конструктора контроллера помещается (type-hint) объект, который нам нужен, и Laravel, когда создает контроллер, создает этот объект (см. [сервис-контейнер](/docs/{{version}}/container)) и подает на вход конструктору контроллера:

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller {

		/**
		 * Сохранение данных пользователя.
		 *
		 * @param  Request  $request
		 * @return Response
		 */
		public function store(Request $request)
		{
			$name = $request->input('name');

			//
		}

	}

Если ваш метод контроллера ожидает параметр из роута, укажите его после зависимостей:

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller {

		/**
		 * Store a new user.
		 *
		 * @param  Request  $request
		 * @param  int  $id
		 * @return Response
		 */
		public function update(Request $request, $id)
		{
			//
		}

	}

<a name="retrieving-input"></a>
## Входные данные

#### Получение входных данных

Объект `Illuminate\Http\Request` предоставляет доступ к входным данным, например, к переменным POST или PUT, полученным из формы. Вам не нужно указывать явно метод запроса, есть универсальный метод:

	$name = Request::input('name');

#### Получение переменной с дефолтным значением

	$name = Request::input('name', 'Sally');

#### Определение, содержится ли переменная в запросе

	if (Request::has('name'))
	{
		//
	}

#### Получить все переменные запроса

	$input = Request::all();

#### Получить избранные переменные

	$input = Request::only('username', 'password'); // только эти

	$input = Request::except('credit_card'); // все, кроме этой

C массивами можно работать через нотацию с точкой:

	$input = Request::input('products.0.name');

<a name="old-input"></a>
## Предыдущие входные данные

Часто нужно сохранять входные данные для следующего запроса. Например, это нужно для того, чтобы после ошибки пользователя в форме не заставлять его вводить всё заново, а заполнить правильные поля самим.

#### Сохранение запроса во flash-переменных сессии

Метод `flash` сохранит текущие входные данные в [сессии](/docs/{{version}}/session), так, что они будут доступны в следующем запросе.

	Request::flash();

#### Сохранение избранных переменных запроса

	Request::flashOnly('username', 'email');

	Request::flashExcept('password');

#### Редирект 

Since you often will want to flash input in association with a redirect to the previous page, you may easily chain input flashing onto a redirect.

Чаще всего нам нужно сохранить данные и сделать редирект на урл с формой. В Laravel есть способ записать это просто и коротко:

	return redirect('form')->withInput();

	return redirect('form')->withInput(Request::except('password'));

#### Получение предыдущих данных

Чтобы получить сохранённые в сессии данные запроса, используйте метод `old`:

	$username = Request::old('username');

Для использования в шаблонах можно использовать этот простой хэлпер:

	{{ old('username') }}

<a name="cookies"></a>
## Куки

Все куки, которые пишет Laravel, зашифрованы - это значит, что на клиенте они не могут быть изменены. Изменённую на клиенте куку фреймворк просто не сможет расшифровать и прочесть.

#### Получение значения куки

	$value = Request::cookie('name');

#### Добавить новую куку к запросу

Хэлпер `cookie` создает объект `Symfony\Component\HttpFoundation\Cookie`. Полученный класс может быть добавлен к HTTP-ответу (response) методом `withCookie`:

	$response = new Illuminate\Http\Response('Hello World');

	$response->withCookie(cookie('name', 'value', $minutes));

#### Создание вечной куки

"На самом деле нет". Время жизни "вечной" куки - 5 лет.

	$response->withCookie(cookie()->forever('name', 'value'));

<a name="files"></a>
## Файлы

#### Получение загруженного файла

	$file = Request::file('photo');

#### Определение, загружался ли файл в запросе

	if (Request::hasFile('photo'))
	{
		//
	}

Метод `file` возвращает экземпляр класса `Symfony\Component\HttpFoundation\File\UploadedFile`, который расширяет стандартный PHP-класс `SplFileInfo` и содержит все его методы.

#### Определение валидности загруженного файла

	if (Request::file('photo')->isValid())
	{
		//
	}

#### Перемещение загруженного файла

	Request::file('photo')->move($destinationPath);

	Request::file('photo')->move($destinationPath, $fileName);

### Другие методы работы с файлами

Полный список методов класса `Symfony\Component\HttpFoundation\File\UploadedFile` смотрите [справке API](http://api.symfony.com/2.5/Symfony/Component/HttpFoundation/File/UploadedFile.html).

<a name="other-request-information"></a>
## Другая информация о запросе

The `Request` class provides many methods for examining the HTTP request for your application and extends the `Symfony\Component\HttpFoundation\Request` class. Here are some of the highlights.

Класс `Request` предоставляет множество методов позволяющих работать с HTTP-запросами в вашем приложении, является расширением класса `Symfony\Component\HttpFoundation\Request`. Вот некоторые из основых методов:

#### URI запроса

	$uri = Request::path();

#### Метод запроса

	$method = Request::method();

	if (Request::isMethod('post'))
	{
		//
	}

#### Проверка на соответствие урла шаблону

	if (Request::is('admin/*'))
	{
		//
	}

#### URL запроса

	$url = Request::url();
