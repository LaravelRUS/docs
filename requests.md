git f502c5b5edf90b7390a7c010ba1f24b1e440aabb

---

# Запросы и входные данные

- [Текущие входные данные](#basic-input)
- [Cookies](#cookies)
- [Старый ввод](#old-input)
- [Файлы](#files)
- [Информация о запросе](#request-information)

<a name="basic-input"></a>
## Текущие входные данные

Вы можете получить доступ ко всем данным, переданным приложению, используя всего несколько простых методов. Вам не нужно думать о том, какой тип HTTP-запроса был использован (GET, POST и т.д.) - методы работают одинаково для любого из них.

#### Получение переменной

	$name = Input::get('name');

#### Получение переменной или значения по умолчанию, если переменная не была передана

	$name = Input::get('name', 'Sally');

#### Была ли передана переменная?

	if (Input::has('name'))
	{
		//
	}

#### Получение всех переменных запроса:

	$input = Input::all();

#### Получение некоторых переменных:

	// Получить только перечисленные:
	$input = Input::only('username', 'password');

	// Получить все, кроме перечисленных:
	$input = Input::except('credit_card');

> **Примечание:** Некоторые JavaScript-библиотеки, такие как Backbone, могут передавать переменные в виде JSON. Вне зависимости от этого `Input::get` будет работать одинаково.

<a name="cookies"></a>
## Cookies

Все cookie, создаваемые Laravel, шифруются и подписываются специальным кодом - таким образом, если они изменятся на клиенте, то они станут неверными и не будут приниматься фреймворком.

#### Чтение cookie

	$value = Cookie::get('name');

#### Добавление cookie к ответу (response)

	$response = Response::make('Hello World');

	$response->withCookie(Cookie::make('name', 'value', $minutes));

#### Помещение cookie в очередь на выставление

Может возникнуть ситуация, когда вам нужно установить определенную cookie в том месте кода, где response (ответ) еще не создан. В таком случай используйте `Cookie::queue()` - cookie будет автоматически добавлена в response после его окончательного формирования.

	Cookie::queue($name, $value, $minutes);	

#### Создание cookie, которая хранится вечно

	$cookie = Cookie::forever('name', 'value');

<a name="old-input"></a>
## Старые входные данные

Вам может пригодиться сохранение входных данных между двумя запросами. Например, после проверки формы на корректность вы можете заполнить её старыми значениями в случае ошибки.

#### Сохранение всего ввода для следующего запроса

	Input::flash();

#### Сохранение некоторых переменных для следующего запроса

	// Сохранить только перечисленные:
	Input::flashOnly('username', 'email');

	// Сохранить все, кроме перечисленных:
	Input::flashExcept('password');

Обычно требуется сохранить входные данные при переадресации на другую страницу - это делается легко:

	return Redirect::to('form')->withInput();

	return Redirect::to('form')->withInput(Input::except('password'));

> **Примечание:** Вы можете сохранять и другие данные внутри сессии, используя класс [Session](/docs/4.2/session).

#### Получение старых входных данных

	Input::old('username');

<a name="files"></a>
## Файлы

#### Получение объекта загруженного файла

	$file = Input::file('photo');

#### Определение успешной загрузки файла

	if (Input::hasFile('photo'))
	{
		//
	}

Метод `file` возвращает объект класса `Symfony\Component\HttpFoundation\File\UploadedFile`, который в свою очередь расширяет стандартный класс `SplFileInfo`, который предоставляет множество методов для работы с файлами.

#### Проверка загруженного файла на валидность

	if (Input::file('photo')->isValid())
	{
		//
	}

#### Перемещение загруженного файла

	Input::file('photo')->move($destinationPath);

	Input::file('photo')->move($destinationPath, $fileName);

#### Получение пути к загруженному файлу

	$path = Input::file('photo')->getRealPath();

#### Получение имени файла на клиентской системе (до загрузки)

	$name = Input::file('photo')->getClientOriginalName();

#### Получение расширения загруженного файла

	$extension = Input::file('photo')->getClientOriginalExtension();

#### Получение размера загруженного файла

	$size = Input::file('photo')->getSize();

#### Определение MIME -типа загруженного файла

	$mime = Input::file('photo')->getMimeType();

<a name="request-information"></a>
## Информация о запросе (request)

Класс `Request` содержит множество методов для изучения входящего запроса в вашем приложении. Он расширяет класс `Symfony\Component\HttpFoundation\Request`. Ниже - несколько полезных примеров.

#### Получение URI (пути) запроса

	$uri = Request::path();

#### Определение метода запроса (GET, POST и т.п.)

	$method = Request::method();

	if (Request::isMethod('post'))
	{
		//
	}	

#### Соответствует ли запрос маске пути?

	if (Request::is('admin/*'))
	{
		//
	}

#### Получение URL запроса

	$url = Request::url();

#### Извлечение сегмента URI (пути)

	$segment = Request::segment(1);

#### Чтение заголовка запроса

	$value = Request::header('Content-Type');

#### Чтение значения из $_SERVER

	$value = Request::server('PATH_INFO');


#### Используется ли HTTPS ?

	if (Request::secure())
	{
		//
	}

#### Это ajax-запрос ?

	if (Request::ajax())
	{
		//
	}

#### Запрос имеет формат JSON ?

	if (Request::isJson())
	{
		//
	}

#### Ожидается, что ответ будет в формате JSON ?

	if (Request::wantsJson())
	{
		//
	}

#### Определение ожидаемого формата ответа

Метод `Request::format` основывается на данных http-заголовка `Accept`

	if (Request::format() == 'json')
	{
		//
	}