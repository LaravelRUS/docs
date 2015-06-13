git 5b9dd1842df9560b4fa022cb9fe26fd909fa9882

---
# Eloquent: Mutators

- [Введение](#introduction)
- [Accessors & Mutators](#accessors-and-mutators)
- [Date Mutators](#date-mutators)
- [Attribute Casting](#attribute-casting)

<a name="introduction"></a>
## Введение

Аксессоры (accessors) и мутаторы (mutators) - это специальные методы моделей Eloquent, которые позволяют вам преобразовывать данные на лету в процессе получения или записи данных в модель. Например, вы хотите шифровать данные в базе данных при помощи [Laravel encrypter](/docs/{{version}}/encryption), и дешифровать их в момент чтения из базы данных. При помощи аксессоров и мутаторов вы можете сделать этот процесс прозрачным для всей остальной логики приложения.

In addition to custom accessors and mutators, Eloquent can also automatically cast date fields to [Carbon](https://github.com/briannesbitt/Carbon) instances or even [cast text fields to JSON](#attribute-casting).

Eloquent подобным образом прозрачно преобразует дату в полях `created_at`, `updated_at`, `deleted_at` в объект [Carbon](https://github.com/briannesbitt/Carbon), а также при каждом [преобразовании полей в JSON](#attribute-casting).


<a name="accessors-and-mutators"></a>
## Accessors & Mutators

#### Определение аксессора

Чтобы определить аксессор на поле `foo` модели, создайте метод `getFooAttribute`. `Foo` - это название поля, переведенное в camel case. Названия полей с подчеркиваниями, как всегда, будут преобразованы в строки с большими буквами перед подчеркиваниями. Аксессрор будет вызываться каждый раз, когда будет производиться чтение заданного поля в модели.
Например, создадим аксессор на поле `first_name`:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * Get the user's first name.
		 *
		 * @param  string  $value
		 * @return string
		 */
		public function getFirstNameAttribute($value)
		{
			return ucfirst($value);
		}
	}

Аксессор получает значение оригинальное поля и возвращает преобразованное. Теперь при штатном доступе к полю, мы получим значение с заменённой первой буквой на большую:

	$user = App\User::find(1);

	$firstName = $user->first_name;

#### Определение мутатора

Чтобы определить мутатор на поле `foo` модели, создайте метод `setFooAttribute`. `Foo` - это название поля, переведенное в camel case. Названия полей с подчеркиваниями, как всегда, будут преобразованы в строки с большими буквами перед подчеркиваниями. Мутатор будет автоматически вызываться каждый раз, когда заданому полю модели будет присваиваться значение. 
Например, создадим мутатор на поле `first_name`:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * Set the user's first name.
		 *
		 * @param  string  $value
		 * @return string
		 */
		public function setFirstNameAttribute($value)
		{
			$this->attributes['first_name'] = strtolower($value);
		}
	}

The mutator will receive the value that is being set on the attribute, allowing you to manipulate the value and set the manipulated value on the Eloquent model's internal `$attributes` property. So, for example, if we attempt to set the `first_name` attribute to `Sally`:

Мутатор принимает значение, которое надо присвоить полю и присваивает его, используя внутреннее свойство модели Eloquent `$attributes`.

Теперь, если мы присвоим first_name имя с большой буквы, мутатор при помощи функции `strtolower()` автоматически преобразует большую букву в маленькую.

	$user = App\User::find(1);

	$user->first_name = 'Sally';

<a name="date-mutators"></a>
## Мутаторы дат

По умолчанию в моделях Eloquent есть внутренние мутаторы и аксессоры на поля `created_at` и `updated_at`, которые преобразуют timestamp, хранящийся в базе данных, в объект [Carbon](https://github.com/briannesbitt/Carbon), который расширяет встроенный в PHP класс `DateTime` и содержит много полезных методов для манипуляции с датой.

Вы можете изменить список полей, к которым применяются эти мутаторы дат (в том числе полностью запретив такое поведение), переопределив у себя в модели свойство `$dates`:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * Аттрибуты, которые должны преобразовываться в Carbon
		 *
		 * @var array
		 */
		protected $dates = ['created_at', 'updated_at', 'disabled_at'];
	}

В поля, определенные в `$dates` вы можете записывать UNIX timestamp (целое число секунд с начала эпохи UNIX), строку даты (`Y-m-d`), строку даты со временем (`Y-m-d H:i:s`) и объект `DateTime` или `Carbon` - автоматические мутаторы все это поймут и при записи в базу данных преобразуют в timestamp-строку.

	$user = App\User::find(1);

	$user->disabled_at = Carbon::now();

	$user->save();

При получении значений из этих полей вы получите объект класса [Carbon](https://github.com/briannesbitt/Carbon). Далее вы можете использовать все методы этого класса.

	$user = App\User::find(1);

	return $user->disabled_at->getTimestamp();

<a name="attribute-casting"></a>
## Приведение типов

Помимо аксессоров,уУ моделей Eloquent есть более простой способ при чтении из базы данных автоматически приводить аттрибуты к определённому типу. Для реализации такого поведения, определите свойство `$casts`. Это должен быть массив, где ключ - название аттрибута, а значение - типа данных. Поддерживаются: `integer`, `real`, `float`, `double`, `string`, `boolean`, `object` и `array`. 

Например, мы хотим, чтобы в аттрибут `is_admin`, который в базе данных хранится в поле tiny integer и принимает значеия 0 или 1, у нас в модели имел логический тип:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * The attributes that should be casted to native types.
		 *
		 * @var array
		 */
		protected $casts = [
			'is_admin' => 'boolean',
		];
	}

Теперь каждый раз, когда мы будем запрашивать в модели этот аттрибут, мы будем получать `true` или `false`.

	$user = App\User::find(1);

	if ($user->is_admin) {
		//
	}

#### Преобразование в массив

Преобразование в массив особенно полезно применять там, где данные в базе данных хранятся в сериализованном JSON. В этом случае в коде можно работать с аттрибутом как с массивом, а при записи/чтении из базы данных он будет автоматически сериализовываться/десериализовываться.

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * The attributes that should be casted to native types.
		 *
		 * @var array
		 */
		protected $casts = [
			'options' => 'array',
		];
	}

Использование:

	$user = App\User::find(1);

	$options = $user->options;

	$options['key'] = 'value';

	$user->options = $options;

	$user->save();
