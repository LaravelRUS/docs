git 328b76a3bd519338dc09ab170764334b623239ea

---

# Eloquent: Коллекции

- [Введение](#introduction)
- [Доступные методы](#available-methods)
- [Пользовательские коллекции](#custom-collections)

<a name="introduction"></a>
## Введение

Все наборы результатов, возвращаемые Eloquent, являются экземплярами объекта `Illuminate\Database\Eloquent\Collection` , в том числе результаты, получаемые с помощью метода `get` или доступные через отношения. Объект коллекции Eloquent наследует [базовую коллекцию](/docs/{{version}}/collections) Laravel, поэтому он наследует десятки методов, используемых для гибкой работы с базовым набором моделей Eloquent.

Конечно же, все коллекции также служат в качестве итераторов, позволяя вам перебирать их в цикле, как будто они простые PHP-массивы:

    $users = App\User::where('active', 1)->get();

    foreach ($users as $user) {
        echo $user->name;
    }

Тем не менее, коллекции гораздо мощнее, чем массивы и предоставляют различные варианты операций map/reduce, которые могут быть сцеплены с использованием интуитивно понятного интерфейса. Например, давайте удалим все неактивные модели и возвратим имена для каждого оставшегося пользователя:

    $users = App\User::all();

    $names = $users->reject(function ($user) {
        return $user->active === false;
    })
    ->map(function ($user) {
        return $user->name;
    });

> {note} Хотя большинство методов для работы с коллекциями Eloquent возвращают новый экземпляр коллекции Eloquent, методы `pluck`, `keys`, `zip`, `collapse`, `flatten` и `flip` возвращают экземпляр [базовой коллекции](/docs/{{version}}/collections). Более того, если операция `map` вернёт коллекцию, в которой нет моделей Eloquent, она будет автоматически приведена к базовой коллекции.

<a name="available-methods"></a>
## Доступные методы

### Базовая коллекция

Все коллекции Eloquent наследуют класс [коллекций Laravel](/docs/{{version}}/collections); поэтому они наследуют все методы, предоставляемые базовым классом коллекций.

Дополнительно класс `Illuminate\Database\Eloquent\Collection` содержит несколько методов, специфичных для работы именно с моделями коллекций. В основном они возвращают `Illuminate\Database\Eloquent\Collection`, но некоторые возвращают и базовые коллекции, `Illuminate\Support\Collection`.

<style>
    #collection-method-list > p {
        column-count: 1; -moz-column-count: 1; -webkit-column-count: 1;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">

[contains](#method-contains)
[diff](#method-diff)
[except](#method-except)
[find](#method-find)
[fresh](#method-fresh)
[intersect](#method-intersect)
[load](#method-load)
[loadMissing](#method-loadMissing)
[modelKeys](#method-modelKeys)
[makeVisible](#method-makeVisible)
[makeHidden](#method-makeHidden)
[only](#method-only)
[unique](#method-unique)

</div>

<a name="method-contains"></a>
#### `contains($key, $operator = null, $value = null)`

Метод `contains` позволяет понять, есть ли в коллекции заданная модель. Если аргумент числовой, он рассматривается как id модели.

    $users->contains(1);
    
    $users->contains(User::find(1));

<a name="method-diff"></a>
#### `diff($items)`

Метод `diff` возвращает все модели, которых нет в переданной коллекции.

    use App\User;

    $users = $users->diff(User::whereIn('id', [1, 2, 3])->get());

<a name="method-except"></a>
#### `except($keys)`

Метод `except` возвращает все модели, кроме тех, чьи id перечислены в аргументе.

    $users = $users->except([1, 2, 3]);

<a name="method-find"></a>
#### `find($key)` {#collection-method .first-collection-method}

Метод `find` находит модели по переданным id. Если вместо id передаётся экземпляр модели, поиск идёт про его id. Если передаётся массив из id, `find` вернёт все эти модели, будет искать по моделям при помощи `whereIn()`:

    $users = User::all();

    $user = $users->find(1);

<a name="method-fresh"></a>
#### `fresh($with = [])`

Метод `fresh` обновляет все модели в коллекции, запрашивая данные из БД. В аргументах можно задать отношение, которое автоматически подгрузится в каждую модель.

    $users = $users->fresh();

    $users = $users->fresh('comments');

<a name="method-intersect"></a>
#### `intersect($items)`

Метод `intersect` возвращает модели которые также присутствуют в коллекции, поданной в аргументе.

    use App\User;

    $users = $users->intersect(User::whereIn('id', [1, 2, 3])->get());

<a name="method-load"></a>
#### `load($relations)`

Метод `load` загружает заданные отношения во все модели.

    $users->load('comments', 'posts');

    $users->load('comments.author');

<a name="method-loadMissing"></a>
#### `loadMissing($relations)`

Метод `loadMissing` загружает отношения во все модели, если они ещё не загружены.

    $users->loadMissing('comments', 'posts');

    $users->loadMissing('comments.author');

<a name="method-modelKeys"></a>
#### `modelKeys()`

Метод `modelKeys` возвращает массив с id моделей из коллекции:

    $users->modelKeys();

    // [1, 2, 3, 4, 5]
    
<a name="method-makeVisible"></a>
#### `makeVisible($attributes)`

Метод `makeVisible` делает поля, обозначенные в моделях как "hidden" видимыми.

    $users = $users->makeVisible(['address', 'phone_number']);

<a name="method-makeHidden"></a>
#### `makeHidden($attributes)`

Метод `makeHidden` добавляет указанные поля моделей в группу "hidden" - эти поля перестанут выводиться в json и т.п.

    $users = $users->makeHidden(['address', 'phone_number']);

<a name="method-only"></a>
#### `only($keys)`

Метод `only` возвращает модели, которые содержат указанные id:

    $users = $users->only([1, 2, 3]);

<a name="method-unique"></a>
#### `unique($key = null, $strict = false)`

Метод `unique` возвращает уникальные модели из коллекции. Уникальность определяется по названию класса модели и id.

    $users = $users->unique();

<a name="custom-collections"></a>
## Пользовательские коллекции

Если вам нужно использовать пользовательский объект `Collection` со своими собственными методами наследования, вы можете переопределить метод `newCollection` в своей модели:

    <?php

    namespace App;

    use App\CustomCollection;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Создание экземпляра новой Eloquent коллекции.
         *
         * @param  array  $models
         * @return \Illuminate\Database\Eloquent\Collection
         */
        public function newCollection(array $models = [])
        {
            return new CustomCollection($models);
        }
    }

После определения метода `newCollection` вы получите экземпляр пользовательской коллекции при любом обращении к экземпляру `Collection` этой модели. Если вы хотите использовать собственную коллекцию для каждой модели в вашем приложении, вы должны переопределить метод `newCollection` в базовом классе модели, наследуемой всеми вашими моделями.
