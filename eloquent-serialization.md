git b5a83c06945d912c81b3b5a723f73e3974aeaf10

---

# Eloquent: Сериализация

- [Введение](#introduction)
- [Сериализация моделей и коллекций](#serializing-models-and-collections)
    - [Сериализация в массивы](#serializing-to-arrays)
    - [Сериализация в JSON](#serializing-to-json)
- [Скрытие атрибутов от JSON](#hiding-attributes-from-json)
- [Добавление значений в JSON](#appending-values-to-json)
- [Сериализация дат](#date-serialization)

<a name="introduction"></a>
## Введение

При создании api вам часто приходится представлять данные моделей в виде json. У Laravel есть инструменты для этого.

<a name="serializing-models-and-collections"></a>
## Сериализация моделей и коллекций

<a name="serializing-to-arrays"></a>
### Сериализация в массивы

Для преобразования модели и её загруженных [отношений](/docs/{{version}}/eloquent-relationships) в массив надо использовать метод `toArray`. Этот метод рекурсивный, поэтому все атрибуты и все отношения (включая отношения отношений) будут конвертированы в массивы:

    $user = App\User::with('roles')->first();

    return $user->toArray();

При помощи `attributesToArray` можно преобразовать в массив только аттрибуты модели, не трогая отношений:

    $user = App\User::first();

    return $user->attributesToArray();    

Вы можете также преобразовывать целые [коллекции](/docs/{{version}}/eloquent-collections) моделей в массивы:

    $users = App\User::all();

    return $users->toArray();

<a name="serializing-to-json"></a>
### Сериализация в JSON

Для преобразования модели в JSON вам надо использовать метод `toJson`. Как и `toArray`, метод `toJson` рекурсивный, поэтому все атрибуты и отношения будут преобразованы в JSON. В качестве аргумента можно указать одну из опций, поддерживаемых [функцией json_encode](https://secure.php.net/manual/en/function.json-encode.php):

    $user = App\User::find(1);

    return $user->toJson();

    return $user->toJson(JSON_PRETTY_PRINT);

В качестве альтернативы, вы можете преобразовать модель или коллекцию в строку, что автоматически вызовет метод `toJson` на модели или коллекции:

    $user = App\User::find(1);

    return (string) $user;

Поскольку модели и коллекции конвертируются в JSON при их преобразовании в строку, вы можете возвращать объекты Eloquent напрямую из ваших роутов или контроллеров:

    Route::get('users', function () {
        return App\User::all();
    });

#### Отношения

Обратите внимание, что имена отношений, которые имеют вид "camelСase", преобразуются в JSON в формат "snake_case"

<a name="hiding-attributes-from-json"></a>
## Скрытие атрибутов от JSON

Иногда вам может быть нужно ограничить список атрибутов, включённых в преобразованный массив или JSON-строку — например, скрыть поле пароля. Для этого добавьте в модель свойство `$hidden`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Атрибуты, которые должны быть невидимы для массива.
         *
         * @var array
         */
        protected $hidden = ['password'];
    }

> {note} При скрытии отношений используйте имя метода отношения, а не имя его динамического свойства.

Вы также можете использовать свойство `visible`, чтобы определить белый список атрибутов, которые должны быть включены в ваш массив модели и преобразованный JSON. Все остальные атрибуты будут скрыты при конвертировании модели в массив или JSON:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Атрибуты, которые должны быть видны в массивах.
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }

#### Временное изменение видимости атрибута

Используйте метод `makeVisible`, чтобы сделать обычно скрытые атрибуты видимыми в данном экземпляре модели. Метод `makeVisible` возвращает экземпляр модели для удобной сцепки методов:

    return $user->makeVisible('attribute')->toArray();

А также, используйте метод `makeHidden`, чтобы сделать обычно видимые атрибуты скрытыми в данном экземпляре модели.

    return $user->makeHidden('attribute')->toArray();

<a name="appending-values-to-json"></a>
## Добавление значений в JSON

Иногда, при конвертировании моделей в массив или JSON, вам может понадобиться добавить атрибуты, для которых нет соответствующих столбцов в вашей БД. Для этого просто определите для него [аксессора](/docs/{{version}}/eloquent-mutators):

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Получить флаг администратора для пользователя.
         *
         * @return bool
         */
        public function getIsAdminAttribute()
        {
            return $this->attributes['admin'] === 'yes';
        }
    }

После создания читателя, добавьте имя атрибута в свойство модели `appends`. Обратите внимание на то, что имена атрибутов указываются в стиле "snake_case", хотя метод определяется в стиле "camelСase":

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Аксессоры, добавленные к форме массива модели..
         *
         * @var array
         */
        protected $appends = ['is_admin'];
    }

Когда атрибут добавлен в список `appends`, он будет включён в оба шаблона — и в массив модели, и в JSON. Атрибуты в массиве `appends` соответствуют настройкам модели `visible` и `hidden`.

#### Добавление аттрибутов во время выполнения

При помощи метода `append` вы можете добавить аттрибут в модель во время выполнения, если он не содержится в `$appends`. Также вы можете переопределить `$appends` в заданной модели при помощи метода `setAppends`:

    return $user->append('is_admin')->toArray();

    return $user->setAppends(['is_admin'])->toArray();

<a name="date-serialization"></a>
## Сериализация дат

#### Формат даты

При использовании [мутаторов аттрибутов](/docs/{{version}}/eloquent-mutators#attribute-casting) вы можете задавать формат вывода дат индивидуально для каждого поля .

    protected $casts = [
        'birthday' => 'date:Y-m-d',
        'joined_at' => 'datetime:Y-m-d H:00',
    ];