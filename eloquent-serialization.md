git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Eloquent: Сериализация

- [Введение](#introduction)
- [Сериализация моделей и коллекций](#serializing-models-and-collections)
    - [Сериализация в массивы](#serializing-to-arrays)
    - [Сериализация в JSON](#serializing-to-json)
- [Скрытие атрибутов от JSON](#hiding-attributes-from-json)
- [Добавление значений в JSON](#appending-values-to-json)

<a name="introduction"></a>
## Введение

При создании JSON API вам часто потребуется преобразовывать модели и отношения к массивам или формату JSON. Eloquent содержит методы для выполнения этих преобразований и управляет атрибутами, включенными в вашу сериализацию.

<a name="serializing-models-and-collections"></a>
## Сериализация моделей и коллекций

<a name="serializing-to-arrays"></a>
### Сериализация в массивы

Для преобразования модели и её загруженных [отношений](/docs/{{version}}/eloquent-relationships) в массив надо использовать метод `toArray`. Этот метод рекурсивный, поэтому все атрибуты и все отношения (включая отношения отношений) будут конвертированы в массивы:

    $user = App\User::with('roles')->first();

    return $user->toArray();

Вы можете также преобразовывать целые [коллекции](/docs/{{version}}/eloquent-collections) моделей в массивы:

    $users = App\User::all();

    return $users->toArray();

<a name="serializing-to-json"></a>
### Сериализация в JSON

Для преобразования модели в JSON вам надо использовать метод `toJson`. Как и `toArray`, метод `toJson` рекурсивный, поэтому все атрибуты и отношения будут преобразованы в JSON:

    $user = App\User::find(1);

    return $user->toJson();

В качестве альтернативы, вы можете преобразовать модель или коллекцию в строку, что автоматически вызовет метод `toJson` на модели или коллекции:

    $user = App\User::find(1);

    return (string) $user;

Поскольку модели и коллекции конвертируются в JSON при их преобразовании в строку, вы можете возвращать объекты Eloquent напрямую из ваших роутов или контроллеров:

    Route::get('users', function () {
        return App\User::all();
    });

<a name="hiding-attributes-from-json"></a>
## Скрытие атрибутов от JSON

Иногда вам может быть нужно ограничить список атрибутов, включённых в преобразованный массив или JSON-строку — например, скрыть пароли. Для этого добавьте в модель свойство `$hidden`:

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
            return $this->attributes['admin'] == 'yes';
        }
    }

После создания читателя, добавьте имя атрибута в свойство модели `appends`. Обратите внимание на то, что имена атрибутов обычно указываются в стиле "snake case", хотя читатель определяется в стиле "camel case":

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
