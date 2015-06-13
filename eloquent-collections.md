git 0bd359d512a89c343f4c07dd8b392af106aa1b66

---
# Eloquent: коллекции

- [Введение](#introduction)
- [Доступные методы](#available-methods)
- [Кастомные коллекции](#custom-collections)

<a name="introduction"></a>
## Введение

Все методы Eloquent, которые возвращают не одну, а несколько моделей (включая запросы к отношениям), на самом деле возвращают не массив с этими моделями, а объект `Illuminate\Database\Eloquent\Collection`. Этот класс является расширением класса [стандартных коллекций Laravel](/docs/{{version}}/collections). 

Так как объект коллекций итерируемый, вы можете работать с ним в циклах как будто это обычный php-массив: 

	$users = App\User::where('active', 1)->get();

	foreach ($users as $user) {
		echo $user->name;
	}

However, collections are much more powerful than arrays and expose a variety of map / reduce operations using an intuitive interface. For example, let's remove all inactive models and gather the first name for each remaining user:

Однако коллекции, в отличие от массивов, имеют массу дополнительных очень удобных фич. Например, давайте отфильтруем из результата запроса все неактивные модели и у оставшихся моделей соберем имена пользователей:

	$users = App\User::where('active', 1)->get();

	$names = $users->reject(function ($user) {
		return $user->active === false;
	})
	->map(function ($user) {
		return $user->name;
	});

<a name="available-methods"></a>
## Дступные методы

### Базовые методы коллекций

Так как коллекции Eloquent являются расширением [основного класса коллекций Laravel](/docs/{{version}}/collections)? они наследуют все его методы:

<style>
	#collection-method-list > p {
		column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
		column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
	}

	#collection-method-list a {
		display: block;
	}
</style>

<div id="collection-method-list" markdown="1">
[all](/docs/{{version}}/collections#method-all)
[chunk](/docs/{{version}}/collections#method-chunk)
[collapse](/docs/{{version}}/collections#method-collapse)
[contains](/docs/{{version}}/collections#method-contains)
[count](/docs/{{version}}/collections#method-count)
[diff](/docs/{{version}}/collections#method-diff)
[each](/docs/{{version}}/collections#method-each)
[filter](/docs/{{version}}/collections#method-filter)
[first](/docs/{{version}}/collections#method-first)
[flatten](/docs/{{version}}/collections#method-flatten)
[flip](/docs/{{version}}/collections#method-flip)
[forget](/docs/{{version}}/collections#method-forget)
[forPage](/docs/{{version}}/collections#method-forpage)
[get](/docs/{{version}}/collections#method-get)
[groupBy](/docs/{{version}}/collections#method-groupby)
[has](/docs/{{version}}/collections#method-has)
[implode](/docs/{{version}}/collections#method-implode)
[intersect](/docs/{{version}}/collections#method-intersect)
[isEmpty](/docs/{{version}}/collections#method-isempty)
[keyBy](/docs/{{version}}/collections#method-keyby)
[keys](/docs/{{version}}/collections#method-keys)
[last](/docs/{{version}}/collections#method-last)
[map](/docs/{{version}}/collections#method-map)
[merge](/docs/{{version}}/collections#method-merge)
[pluck](/docs/{{version}}/collections#method-pluck)
[pop](/docs/{{version}}/collections#method-pop)
[prepend](/docs/{{version}}/collections#method-prepend)
[pull](/docs/{{version}}/collections#method-pull)
[push](/docs/{{version}}/collections#method-push)
[put](/docs/{{version}}/collections#method-put)
[random](/docs/{{version}}/collections#method-random)
[reduce](/docs/{{version}}/collections#method-reduce)
[reject](/docs/{{version}}/collections#method-reject)
[reverse](/docs/{{version}}/collections#method-reverse)
[search](/docs/{{version}}/collections#method-search)
[shift](/docs/{{version}}/collections#method-shift)
[shuffle](/docs/{{version}}/collections#method-shuffle)
[slice](/docs/{{version}}/collections#method-slice)
[sort](/docs/{{version}}/collections#method-sort)
[sortBy](/docs/{{version}}/collections#method-sortby)
[sortByDesc](/docs/{{version}}/collections#method-sortbydesc)
[splice](/docs/{{version}}/collections#method-splice)
[sum](/docs/{{version}}/collections#method-sum)
[take](/docs/{{version}}/collections#method-take)
[toArray](/docs/{{version}}/collections#method-toarray)
[toJson](/docs/{{version}}/collections#method-tojson)
[transform](/docs/{{version}}/collections#method-transform)
[unique](/docs/{{version}}/collections#method-unique)
[values](/docs/{{version}}/collections#method-values)
[where](/docs/{{version}}/collections#method-where)
[whereLoose](/docs/{{version}}/collections#method-whereloose)
[zip](/docs/{{version}}/collections#method-zip)
</div>

<a name="custom-collections"></a>
## Кастомные коллекции

Если вам нужно расширить класс коллекций Eloquent, добавив туда свои методы, переопределите метод `newCollection` в нужной модели:

	<?php namespace App;

	use App\CustomCollection;
	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * Create a new Eloquent Collection instance.
		 *
		 * @param  array  $models
		 * @return \Illuminate\Database\Eloquent\Collection
		 */
		public function newCollection(array $models = [])
		{
			return new CustomCollection($models);
		}
	}

Если вам нужно переопределить класс коллекций для всех моделей, переопределите его в базовой модели, и все свои модели наследуйте от этой базовой модели. 