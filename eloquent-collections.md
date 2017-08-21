git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

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

    $users = App\User::where('active', 1)->get();

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

Все коллекции Eloquent наследуют объект базовой [коллекции Laravel](/docs/{{version}}/collections); поэтому они наследуют все мощные методы, предоставляемые базовым классом коллекции:

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
[average](/docs/{{version}}/collections#method-average)
[avg](/docs/{{version}}/collections#method-avg)
[chunk](/docs/{{version}}/collections#method-chunk)
[collapse](/docs/{{version}}/collections#method-collapse)
[combine](/docs/{{version}}/collections#method-combine)
[contains](/docs/{{version}}/collections#method-contains)
[containsStrict](/docs/{{version}}/collections#method-containsstrict)
[count](/docs/{{version}}/collections#method-count)
[diff](/docs/{{version}}/collections#method-diff)
[diffKeys](/docs/{{version}}/collections#method-diffkeys)
[each](/docs/{{version}}/collections#method-each)
[every](/docs/{{version}}/collections#method-every)
[except](/docs/{{version}}/collections#method-except)
[filter](/docs/{{version}}/collections#method-filter)
[first](/docs/{{version}}/collections#method-first)
[flatMap](/docs/{{version}}/collections#method-flatmap)
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
[isNotEmpty](/docs/{{version}}/collections#method-isnotempty)
[keyBy](/docs/{{version}}/collections#method-keyby)
[keys](/docs/{{version}}/collections#method-keys)
[last](/docs/{{version}}/collections#method-last)
[map](/docs/{{version}}/collections#method-map)
[mapWithKeys](/docs/{{version}}/collections#method-mapwithkeys)
[max](/docs/{{version}}/collections#method-max)
[median](/docs/{{version}}/collections#method-median)
[merge](/docs/{{version}}/collections#method-merge)
[min](/docs/{{version}}/collections#method-min)
[mode](/docs/{{version}}/collections#method-mode)
[nth](/docs/{{version}}/collections#method-nth)
[only](/docs/{{version}}/collections#method-only)
[partition](/docs/{{version}}/collections#method-partition)
[pipe](/docs/{{version}}/collections#method-pipe)
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
[split](/docs/{{version}}/collections#method-split)
[sum](/docs/{{version}}/collections#method-sum)
[take](/docs/{{version}}/collections#method-take)
[tap](/docs/{{version}}/collections#method-tap)
[toArray](/docs/{{version}}/collections#method-toarray)
[toJson](/docs/{{version}}/collections#method-tojson)
[transform](/docs/{{version}}/collections#method-transform)
[union](/docs/{{version}}/collections#method-union)
[unique](/docs/{{version}}/collections#method-unique)
[uniqueStrict](/docs/{{version}}/collections#method-uniquestrict)
[values](/docs/{{version}}/collections#method-values)
[when](/docs/{{version}}/collections#method-when)
[where](/docs/{{version}}/collections#method-where)
[whereStrict](/docs/{{version}}/collections#method-wherestrict)
[whereIn](/docs/{{version}}/collections#method-wherein)
[whereInStrict](/docs/{{version}}/collections#method-whereinstrict)
[whereNotIn](/docs/{{version}}/collections#method-wherenotin)
[whereNotInStrict](/docs/{{version}}/collections#method-wherenotinstrict)
[zip](/docs/{{version}}/collections#method-zip)

</div>

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
