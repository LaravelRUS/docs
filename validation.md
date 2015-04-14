git fb85738c37ef8da3e0b3ed2c6e059b94d3d0dbac

---

# Валидация

- [Использование валидации](#basic-usage)
- [Работа с сообщениями об ошибках](#working-with-error-messages)
- [Ошибки и шаблоны](#error-messages-and-views)
- [Доступные правила проверки](#available-validation-rules)
- [Условные правила](#conditionally-adding-rules)
- [Собственные сообщения об ошибках](#custom-error-messages)
- [Собственные правила проверки](#custom-validation-rules)

<a name="basic-usage"></a>
## Использование валидации

Laravel поставляется с простой, удобной системой валидации (проверки входных данных на соответствие павилам) и получения сообщений об ошибках - классом `Validation`.

#### Простейший пример валидации

	$validator = Validator::make(
		array('name' => 'Дейл'),
		array('name' => 'required|min:5')
	);

Первый параметр, передаваемый методу `make` - данные для проверки. Второй параметр - правила, которые к ним должны быть применены.

#### Использование массивов для указания правил

Несколько правил могут быть разделены либо символом вертикальной черты (|), либо быть отдельными элементами массива.

	$validator = Validator::make(
		array('name' => 'Дейл'),
		array('name' => array('required', 'min:5'))
	);
	
#### Проверка нескольких полей

    $validator = Validator::make(
        array(
            'name' => 'Дейл',
            'password' => 'плохойпароль',
            'email' => 'email@example.com'
        ),
        array(
            'name' => 'required',
            'password' => 'required|min:8',
            'email' => 'required|email|unique'
        )
    );

Как только был создан экземпляр `Validator`, метод `fails` (или `passes`) может быть использован для проведения проверки.

	if ($validator->fails())
	{
		// Переданные данные не прошли проверку
	}

Если `Validator` нашёл ошибки, вы можете получить его сообщения таким образом:

	$messages = $validator->messages();

Вы также можете получить массив правил, данные которые не прошли проверку, без самих сообщений:

	$failed = $validator->failed();

#### Проверка файлов

Класс `Validator` содержит несколько изначальных правил для проверки файлов, такие как `size`, `mimes` и другие. Для выполнения проверки над файлами просто передайте эти файлы вместе с другими данными.

<a name="working-with-error-messages"></a>
## Работа с сообщениями об ошибках

После вызова метода `messages` объекта `Validator` вы получите объект `MessageBag`, который имеет набор полезных методов для доступа к сообщеням об ошибках.

#### Получение первого сообщения для поля

	echo $messages->first('email');

#### Получение всех сообщений для одного поля

	foreach ($messages->get('email') as $message)
	{
		//
	}

#### Получение всех сообщений для всех полей

	foreach ($messages->all() as $message)
	{
		//
	}

#### Проверка на наличие сообщения для поля

	if ($messages->has('email'))
	{
		//
	}

#### Получение ошибки в заданном формате

	echo $messages->first('email', '<p>:message</p>');

> **Примечание:** по умолчанию сообщения форматируются в вид, который подходит для Twitter Bootstrap.

#### Получение всех сообщений в заданном формате

	foreach ($messages->all('<li>:message</li>') as $message)
	{
		//
	}

<a name="error-messages-and-views"></a>
## Ошибки и шаблоны

Как только вы провели проверку, вам понадобится простой способ, чтобы передать ошибки в шаблон. Laravel позволяет удобно сделать это. Например, у нас есть такие роуты:

	Route::get('register', function()
	{
		return View::make('user.register');
	});

	Route::post('register', function()
	{
		$rules = array(...);

		$validator = Validator::make(Input::all(), $rules);

		if ($validator->fails())
		{
			return Redirect::to('register')->withErrors($validator);
		}
	});

Заметьте, что когда проверки не пройдены, мы передаём объект `Validator` объекту переадресации `Redirect` с помощью метода `withErrors`. Этот метод сохранит сообщения об ошибках в одноразовых flash-переменных сессии, таким образом делая их доступными для следующего запроса.

Однако, заметьте, мы не передаем в `View::make('user.register');` переменные $errors в шаблон. Laravel сам проверяет данные сессии на наличие переменных и автоматически передает их в шаблону, если они доступны. **Таким образом, важно помнить, что переменная `$errors` будет доступна для всех ваших шаблонов всегда, при любом запросе.**. Это позволяет вам считать, что переменная `$errors` всегда определена и может безопасно использоваться. Переменная `$errors` - экземпляр класса `MessageBag`.

Таким образом, после переадресации вы можете прибегнуть к автоматически установленной в шаблоне переменной `$errors`:

	<?php echo $errors->first('email'); ?>

<a name="available-validation-rules"></a>
## Доступные правила проверки

Ниже список всех доступных правил и их функции:

- [Accepted](#rule-accepted)
- [Active URL](#rule-active-url)
- [After (Date)](#rule-after)
- [Alpha](#rule-alpha)
- [Alpha Dash](#rule-alpha-dash)
- [Alpha Numeric](#rule-alpha-num)
- [Array](#rule-array)
- [Before (Date)](#rule-before)
- [Between](#rule-between)
- [Confirmed](#rule-confirmed)
- [Date](#rule-date)
- [Date Format](#rule-date-format)
- [Different](#rule-different)
- [E-Mail](#rule-email)
- [Exists (Database)](#rule-exists)
- [Image (File)](#rule-image)
- [In](#rule-in)
- [Integer](#rule-integer)
- [IP Address](#rule-ip)
- [Max](#rule-max)
- [MIME Types](#rule-mimes)
- [Min](#rule-min)
- [Not In](#rule-not-in)
- [Numeric](#rule-numeric)
- [Regular Expression](#rule-regex)
- [Required](#rule-required)
- [Required If](#rule-required-if)
- [Required With](#rule-required-with)
- [Required With All](#rule-required-with-all)
- [Required Without](#rule-required-without)
- [Required Without All](#rule-required-without-all)
- [Same](#rule-same)
- [Size](#rule-size)
- [Unique (Database)](#rule-unique)
- [URL](#rule-url)

<a name="rule-accepted"></a>
#### accepted

Поле должно быть в значении _yes_, _on_ или _1_. Это полезно для проверки принятия правил и лицензий.

<a name="rule-active-url"></a>
#### active_url

Поле должно быть корректным URL, доступным через функцию [checkdnsrr](http://ru2.php.net/checkdnsrr).

<a name="rule-after"></a>
#### after:_date_

Поле должно быть датой, более поздней, чем _date_. Строки приводятся к датам функцией `strtotime`.

<a name="rule-alpha"></a>
#### alpha

Поле можно содержать только только латинские символы.

<a name="rule-alpha-dash"></a>
#### alpha_dash

Поле можно содержать только латинские символы, цифры, знаки подчёркивания (_) и дефисы (-).

<a name="rule-alpha-num"></a>
#### alpha_num

Поле можно содержать только латинские символы и цифры.

<a name="rule-array"></a>
#### array

Поле должно быть массивом.

<a name="rule-before"></a>
#### before:_date_

Поле должно быть датой, более ранней, чем _date_. Строки приводятся к датам функцией `strtotime`.

<a name="rule-between"></a>
#### between:_min_,_max_

Поле должно быть числом в диапазоне от _min_ до _max_. Строки, числа и файлы трактуются аналогично правилу `size`.

<a name="rule-confirmed"></a>
#### confirmed

Значение поля должно соответствовать значению поля с этим именем, плюс `foo_confirmation`. Например, если проверяется поле `password`, то на вход должно быть передано совпадающее по значению поле `password_confirmation`.

<a name="rule-date"></a>
#### date

Поле должно быть правильной датой в соответствии с функцией `strtotime`.

<a name="rule-date-format"></a>
#### date_format:_format_

Поле должно подходить под формату даты _format_ в соответствии с функцией `date_parse_from_format`.

<a name="rule-different"></a>
#### different:_field_

Значение проверяемого поля должно отличаться от значения поля _field_.

<a name="rule-email"></a>
#### email

Поле должно быть корректным адресом e-mail.

<a name="rule-exists"></a>
#### exists:_table_,_column_

Поле должно существовать в заданной таблице базе данных.

**Простое использование:**

	'state' => 'exists:states'

**Указание имени поля в таблице:**

	'state' => 'exists:states,abbreviation'

Вы также можете указать больше условий, которые будут добавлены к запросу "WHERE":

	'email' => 'exists:staff,email,account_id,1'

<a name="rule-image"></a>
#### image

Загруженный файл должен быть изображением в формате jpeg, png, bmp или gif.

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

Значение поля должно быть одним из перечисленных (_foo_, _bar_ и т.д.).

<a name="rule-integer"></a>
#### integer

Поле должно иметь корректное целочисленное значение.

<a name="rule-ip"></a>
#### ip

Поле должно быть корректным IP-адресом.

<a name="rule-max"></a>
#### max:_value_

Значение поля должно быть меньше или равно _value_. Строки, числа и файлы трактуются аналогично правилу `size`.

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

MIME-тип загруженного файла должен быть одним из перечисленных.

**Простое использование:**

	'photo' => 'mimes:jpeg,bmp,png'

<a name="rule-min"></a>
#### min:_value_

Значение поля должно быть более _value_. Строки, числа и файлы трактуются аналогично правилу `size`.

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

Значение поля **не** должно быть одним из перечисленных (_foo_, _bar_ и т.д.).

<a name="rule-numeric"></a>
#### numeric

Поле должно иметь корректное числовое или дробное значение.

<a name="rule-regex"></a>
#### regex:_pattern_

Поле должно соответствовать заданному регулярному выражению.

**Внимание:** при использовании этого правила может быть нужно перечислять другие правила в виде элементов массива, особенно если выражение содержит символ вертикальной черты (|).

<a name="rule-required"></a>
#### required

Проверяемое поле должно иметь непустое значение.

<a name="rule-required-if"></a>
#### required_if:_field_,_value_

Проверяемое поле должно иметь непустое значение, если другое поле _field_ имеет значение _value_.

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

Проверяемое поле должно иметь непустое значение, но только если присутствует хотя бы одно из перечисленных полей (_foo_, _bar_ и т.д.).

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

Проверяемое поле должно иметь непустое значение, но только если присутствуют все перечисленные поля (_foo_, _bar_ и т.д.).

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

Проверяемое поле должно иметь непустое значение, но только если **не** присутствует хотя бы одно из перечисленных полей (_foo_, _bar_ и т.д.).

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

Проверяемое поле должно иметь непустое значение, но только если **не** присутствуют все перечисленные поля (_foo_, _bar_ и т.д.).

<a name="rule-same"></a>
#### same:_field_

Поле должно иметь то же значение, что и поле _field_.

<a name="rule-size"></a>
#### size:_value_

Поле должно иметь совпадающий с _value_ размер. **Для строк** это обозначает длину, **для чисел** - число, **для файлов** - размер в килобайтах.

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

Значение поля должно быть уникальным в заданной таблице базы данных. Если `column` не указано, то будет использовано имя поля.

#### Простое использование

	'email' => 'unique:users'

#### Указание имени поля в таблице

	'email' => 'unique:users,email_address'

#### Игнорирование определённого ID

	'email' => 'unique:users,email_address,10'

#### Добавление дополнительных условий

Вы также можете указать больше условий, которые будут добавлены к запросу "WHERE"":

	'email' => 'unique:users,email_address,NULL,id,account_id,1'

В правиле выше только строки с `account_id` равном `1` будут включены в проверку.

<a name="rule-url"></a>
#### url

Поле должно быть корректным URL.

> **Примечание:** используется PHP-функция `filter_var`


<a name="conditionally-adding-rules"></a>
## Условные правила

Иногда вам нужно валидировать некое поле **только** тогда, когда оно присутствует во входных данных. Для этого добавьте правило `sometimes`:

	$v = Validator::make($data, array(
		'email' => 'sometimes|required|email',
	));

В примере выше поле для поля `email` будет запущена валидация только когда `$data['email']` существует.

#### Сложные условные правила

Иногда вам может нужно, чтобы поле имело какое-либо значение только если другое поле имеет значеие, скажем, больше 100. Или вы можете требовать наличия двух полей только, когда также указано третье. Это легко достигается условными правилами. Сперва создайте объект `Validator` с набором статичных правил, которые никогда не изменяются:

	$v = Validator::make($data, array(
		'email' => 'required|email',
		'games' => 'required|numeric',
	));

Теперь предположим, что ваше приложения написано для коллекционеров игр. Если регистрируется коллекционер с более, чем 100 играми, то мы хотим их спросить, зачем им такое количество. Например, у них может быть магазин или может им просто нравится их собирать. Итак, для добавления такого условного правила мы используем метод `Validator`.

	$v->sometimes('reason', 'required|max:500', function($input)
	{
		return $input->games >= 100;
	});

Первый параметр этого метода - имя поля, которое мы проверяем. Второй параметр - правило, которое мы хотим добавить, если переданная функция-замыкание (третий параметр) вернёт `true`. Этот метод позволяет легко создавать сложные правила проверки ввода. Вы можете даже добавлять одни и те же условные правила для нескольких полей одновременно:

	$v->sometimes(array('reason', 'cost'), 'required', function($input)
	{
		return $input->games >= 100;
	});

> **Примечание:** Параметр `$input`, передаваемый замыканию - объект `Illuminate\Support\Fluent` и может использоваться для чтения проверяемого ввода и файлов.

<a name="custom-error-messages"></a>
## Собственные сообщения об ошибках

Вы можете передать собственные сообщения об ошибках вместо используемых по умолчанию. Если несколько способов это сделать.

#### Передача своих сообщений в `Validator`

	$messages = array(
		'required' => 'Поле :attribute должно быть заполнено.',
	);

	$validator = Validator::make($input, $rules, $messages);

> **Примечание:** строка `:attribute` будет заменена на имя проверяемого поля. Вы также можете использовать и другие строки-переменные.

#### Использование других переменных-строк

	$messages = array(
		'same'    => 'Значения :attribute и :other должны совпадать.',
		'size'    => 'Поле :attribute должно быть ровно exactly :size.',
		'between' => 'Значение :attribute должно быть от :min и до :max.',
		'in'      => 'Поле :attribute должно иметь одно из следующих значений: :values',
	);

#### Указание собственного сообщения для отдельного поля

Иногда вам может потребоваться указать своё сообщение для отдельного поля.

	$messages = array(
		'email.required' => 'Нам нужно знать ваш e-mail адрес!',
	);

<a name="localization"></a>
#### Указание собственных сообщений в файле локализации

Также можно определять сообщения валидации в файле локализации вместо того, чтобы передавать их в `Validator` напрямую. Для этого добавьте сообщения в массив `custom` файла локализации `app/lang/xx/validation.php`.

	'custom' => array(
		'email' => array(
			'required' => 'Нам нужно знать ваш e-mail адрес!',
		),
	),

<a name="custom-validation-rules"></a>
## Собственные правила проверки

#### Регистрация собственного правила валидации

Laravel изначально содержит множество полезных правил, однако вам может понадобиться создать собственные. Одним из способов зарегистрировать произвольное правило - через метод `Validator::extend`.

	Validator::extend('foo', function($attribute, $value, $parameters)
	{
		return $value == 'foo';
	});

> **Примечание:** имя правила должно быть в `формате_с_подчёркиваниями`.

Переданная функция-замыкание получает три параметра: имя проверяемого поля `$attribute`, значение поля `$value` и массив параметров `$parameters` переданных правилу.

Вместо функции в метод `extend` можно передать ссылку на метод класса:

	Validator::extend('foo', 'FooValidator@validate');

Обратите внимание, что вам также понадобиться определить сообщение об ошибке для нового правила. Вы можете сделать это либо передавая его в виде массива строк в `Validator`, либо вписав в файл локализации.

#### Расширение класса `Validator`

Вместо использования функций-замыканий для расширения набора доступных правил вы можете расширить сам класс `Validator`. Для этого создайте класс, который наследует `Illuminate\Validation\Validator`. Вы можете добавить новые методы проверок, начав их имя с `validate`.

	<?php

	class CustomValidator extends Illuminate\Validation\Validator {

		public function validateFoo($attribute, $value, $parameters)
		{
			return $value == 'значение';
		}

	}

#### Регистрация нового класса `Validator`

Затем вам нужно зарегистрировать это расширение валидации. Сделать это можно, например, в вашем [сервис-провайдере](/docs/4.1/ioc#service-providers) или в ваших [старт-файлах](/docs/4.1/lifecycle#start-files).

	Validator::resolver(function($translator, $data, $rules, $messages)
	{
		return new CustomValidator($translator, $data, $rules, $messages);
	});

Иногда при создании своего класса валидации вам может понадобиться определить собственные строки-переменные (типа ":foo") для замены в сообщениях об ошибках. Это делается путём создания класса, как было описано выше, и добавлением функций с именами вида `replaceXXX`.

	protected function replaceFoo($message, $attribute, $rule, $parameters)
	{
		return str_replace(':foo', $parameters[0], $message);
	}

Если вы хотите добавить свое сообщение без использования `Validator::extend`, вы можете использовать метод `Validator::replacer`:

	Validator::replacer('rule', function($message, $attribute, $rule, $parameters)
	{
		//
	});
