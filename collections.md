git 4deba2bfca6636d5cdcede3f2068eff3b59c15ce

---

# Коллекции

- [Введение](#introduction)
- [Основы использования](#basic-usage)

<a name="introduction"></a>
## Введение

Класс `Illuminate\Support\Collection` - гибкая и удобная обёртка для работы с массивами. 

Для примера, взгляните на код ниже. При помощи хэлпера `collect` мы создаём коллекцию из массива:

	$collection = collect(['taylor', 'abigail', null])->map(function($name)
	{
		return strtoupper($name);
	})
	->reject(function($name)
	{
		return is_null($value);
	});

Как вы можете видеть, класс `Collection` позволяет строить цепочки вызовов для операций типа map и reduce над заданным массивом данных.
В основном каждый метод класса `Collection` возвращает новый объект класса `Collection`.

<a name="basic-usage"></a>
## Основы использования

#### Создание коллекции

Чтобы создать коллекцию из массива, воспользуйтесь функцией-хэлпером или методом `make`:

	$collection = collect([1, 2, 3]);

	$collection = Collection::make([1, 2, 3]);

Там, где методы [Eloquent](/docs/{{version}}/eloquent) возвращают не один, а несколько объектов - возвращается коллекция. Однако, коллекции
предоставляют слишком мощный функционал, чтобы оставить его только для Eloquent-моделей, попробуйте применить класс `Collection` у себя в приложении.

#### Посмотрите, что могут коллекции

Вместо того, чтобы приводить здесь огромный скучный список методов коллекции, мы отсылаем вас к более удобной [документации API этого класса](http://laravel.com/api/master/Illuminate/Support/Collection.html).
