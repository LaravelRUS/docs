git 504364a3588b3f54993a8b8566673270b36cda3b

---

# Хелперы

- [Массивы](#arrays)
- [Пути](#paths)
- [Строки](#strings)
- [URLs](#urls)
- [Прочее](#miscellaneous)

<a name="arrays"></a>
## Массивы

### array_add

Добавить указанную пару ключ/значение в массив, если она там ещё не существует.

	$array = array('foo' => 'bar');

	$array = array_add($array, 'key', 'value');

### array_divide

Вернуть два массива - один с ключами, другой со значениями оригинального массива.

	$array = array('foo' => 'bar');

	list($keys, $values) = array_divide($array);

### array_dot

Сделать многоуровневый массив одноуровневым, объединяя вложенные массивы с помощью точки в именах.

	$array = array('foo' => array('bar' => 'baz'));

	$array = array_dot($array);

	// array('foo.bar' => 'baz');

### array_except

Удалить указанную пару ключ/значение из массива.

	$array = array_except($array, array('ключи', 'для', 'удаления'));

### array_fetch

Вернуть одноуровневый массив с выбранными элементами по переданному пути.

	$array = array(
		array('developer' => array('name' => 'Taylor')),
		array('developer' => array('name' => 'Dayle')),
	);

	$array = array_fetch($array, 'developer.name');

	// array('Taylor', 'Dayle');

### array_first

Вернуть первый элемент массива, прошедший (return true) требуемый тест.

	$array = array(100, 200, 300);

	$value = array_first($array, function($key, $value)
	{
		return $value >= 150;
	});

Третьим параметром можно передать значение по умолчанию:

	$value = array_first($array, $callback, $default);

### array_flatten

Сделать многоуровневый массив плоским.

	$array = array('name' => 'Joe', 'languages' => array('PHP', 'Ruby'));

	$array = array_flatten($array);

	// array('Joe', 'PHP', 'Ruby');

### array_forget

Удалить указанную пару ключ/значение из многоуровневого массива, используя синтаксис имени с точкой.

	$array = array('names' => array('joe' => array('programmer')));

	array_forget($array, 'names.joe');

### array_get

Вернуть значение из многоуровневого массива, используя синтаксис имени с точкой.

	$array = array('names' => array('joe' => array('programmer')));

	$value = array_get($array, 'names.joe');
	
	$value = array_get($array, 'names.john', 'default');

> **Примечание:** Нужен `array_get` для объектов? Используйте `object_get`.

### array_only

Вернуть из массива только указанные пары ключ/значения.

	$array = array('name' => 'Joe', 'age' => 27, 'votes' => 1);

	$array = array_only($array, array('name', 'votes'));

### array_pluck

Извлечь значения из многоуровневого массива, соответствующие переданному ключу.

	$array = array(array('name' => 'Taylor'), array('name' => 'Dayle'));

	$array = array_pluck($array, 'name');

	// array('Taylor', 'Dayle');

### array_pull

Извлечь значения из многоуровневого массива, соответствующие переданному ключу, и удалить их.

	$array = array('name' => 'Taylor', 'age' => 27);

	$name = array_pull($array, 'name');

### array_set

Установить значение в многоуровневом массиве, используя синтаксис имени с точкой.

	$array = array('names' => array('programmer' => 'Joe'));

	array_set($array, 'names.editor', 'Taylor');

### array_sort

Отсортировать массив по результатам вызывов переданной функции-замыкания.

	$array = array(
		array('name' => 'Jill'),
		array('name' => 'Barry'),
	);

	$array = array_values(array_sort($array, function($value)
	{
		return $value['name'];
	}));

### array_where

Отфильтровать массив функцией-замыканием.

	$array = array(100, '200', 300, '400', 500);

	$array = array_where($array, function($key, $value)
	{
		return is_string($value);
	});

	// Array ( [1] => 200 [3] => 400 )

### head

Вернуть первый элемент массива. Полезно при сцеплении методов в PHP 5.3.x.

	$first = head($this->returnsArray('foo'));

### last

Вернуть последний элемент массива. Полезно при сцеплении методов.

	$last = last($this->returnsArray('foo'));

<a name="paths"></a>
## Пути

### app_path

Получить абсолютный путь к папке `app`.

### base_path

Получить абсолютный путь к корневой папке приложения.

### public_path

Получить абсолютный путь к папке `public`.

### storage_path

Получить абсолютный путь к папке `storage`.

<a name="strings"></a>
## Строки

### camel_case

Преобразовать строку к `camelCase`.

	$camel = camel_case('foo_bar');

	// fooBar

### class_basename

Получить имя класса переданного класса без пространства имён.

	$class = class_basename('Foo\Bar\Baz');

	// Baz

### e

Выполнить над строкой `htmlentities` в кодировке UTF-8.

	$entities = e('<html>foo</html>');

### ends_with

Определить, заказчивается ли строка переданной подстрокой.

	$value = ends_with('This is my name', 'name');

### snake_case

Преобразовать строку к `snake_case` (стиль именования Си, с подчёркиваниями вместо пробелов - прим. пер.).

	$snake = snake_case('fooBar');

	// foo_bar


### str_limit

Ограничить строку заданным количеством символов и символами окончания.

	str_limit($value, $limit = 100, $end = '...')

Example:

	$value = str_limit('The PHP framework for web artisans.', 7);

	// The PHP...

### starts_with

Определить, начинается ли строка с переданной подстроки.

	$value = starts_with('This is my name', 'This');

### str_contains

Определить, содержит ли строка переданную подстрокую.

	$value = str_contains('This is my name', 'my');

### str_finish

Добавить одно вхождение подстроки в конец переданной строки и удалить повторы в конце, если они есть.

	$string = str_finish('this/string', '/');

	// this/string/

### str_is

Определить, соответствует ли строка маске. Можно использовать звёздочки (*) как символы подстановки.

	$value = str_is('foo*', 'foobar');

### str_plural

Преобразовать слово-строку во множественное число (только для английского языка).

	$plural = str_plural('car');

### str_random

Создать последовательность случайных символов заданной длины.

	$string = str_random(40);

### str_singular

Преобразовать слово-строку в единственное число (только для английского языка).

	$singular = str_singular('cars');

### str_slug

Создать строку-идентификатор (slug) с учётом ограничений URL.

	str_slug($title, $separator);

Пример:
	
	$title = str_slug('Laravel 5 Framework', '-');
	
	// laravel-5-framework
	
### studly_case

Преобразовать строку в `StudlyCase`.

	$value = studly_case('foo_bar');

	// FooBar

### trans

Перевести переданную языковую строку. Это алиас для `Lang::get`.

	$value = trans('validation.required'):

### trans_choice

Перевести переданную языковую строку с изменениями. Алиас для `Lang::choice`.

	$value = trans_choice('foo.bar', $count);

<a name="urls"></a>
## URLs

### action

Сгенерировать URL для заданного экшна (метода) контроллера.

	$url = action('HomeController@getIndex', $params);

### route

Сгенерировать URL для заданного именованного роута.

	$url = route('routeName', $params);

### asset

Сгенерировать URL ко внешнему ресурсу (изображению и пр.).

	$url = asset('img/photo.jpg');

### link_to

Сгенерировать HTML-ссылку на указанный URL.

	echo link_to('foo/bar', $title, $attributes = array(), $secure = null);

### link_to_asset

Сгенерировать HTML-ссылку на внешний ресурс (изображение и пр.).

	echo link_to_asset('foo/bar.zip', $title, $attributes = array(), $secure = null);

### link_to_route

Сгенерировать HTML-ссылку на заданный именованный маршрут.

	echo link_to_route('route.name', $title, $parameters = array(), $attributes = array());

### link_to_action

Сгенерировать HTML-ссылку на заданное действие контроллера.

	echo link_to_action('HomeController@getIndex', $title, $parameters = array(), $attributes = array());

### secure_asset

Сгенерировать HTML-ссылку на внешний ресурс (изображение и пр.) через HTTPS.

	echo secure_asset('foo/bar.zip', $title, $attributes = array());

### secure_url

Сгенерировать HTML-ссылку на указанный путь через HTTPS.

	echo secure_url('foo/bar', $parameters = array());

### url

Сгенерировать HTML-ссылку на указанный абсолютный путь.

	echo url('foo/bar', $parameters = array(), $secure = null);

<a name="miscellaneous"></a>
## Прочее

### csrf_token

Получить текущее значение CSRF-токена.

	$token = csrf_token();

### dd

Вывести дамп переменной и завершить выполнение скрипта.

	dd($value);

### value

Если переданное значение - функция-замыкание, вызвать её и вернуть результат. В противном случае вернуть само значение.

	$value = value(function() { return 'bar'; });

### with

Вернуть переданный объект. Полезно при сцеплении методов в PHP 5.3.x.

	$value = with(new Foo)->doWork();
