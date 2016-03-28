git b1d293216e13a68917ae3ab4ef2b4235101610d0

---

# Eloquent: Сериализация

- [Введение](#introduction)
- [Основы использования](#basic-usage)
- [Скрытие атрибутов при экспорте в JSON](#hiding-attributes-from-json)
- [Добавление атрибутов в JSON](#appending-values-to-json)

<a name="introduction"></a>
## Вступление

При построении JSON API очень часто требуется конвертация моделей и отношений в массивы или JSON. Eloquent обладает как удобным механизмом для подобных конвертаций, так и инструментами для контроля над отдельными атрибутами при сериализации.

<a name="basic-usage"></a>
## Основы использования

#### Преобразование модели в массив

Чтобы преобразовать модель вместе с загруженными [отношениями](/docs/{{version}}/eloquent-relationships) в массив, можно использовать  метод `toArray`. Этот метод рекурсивный, так что все атрибуты и все отношения (включая отношения отношений) так же будут преобразованы в массивы:

    $user = App\User::with('roles')->first();

    return $user->toArray();

Также можно преобразовывать в массивы и [коллекции](/docs/{{version}}/eloquent-collections):

    $users = App\User::all();

    return $users->toArray();

#### Преобразование модели в JSON

Для преобразования модели в JSON можно использовать метод `toJson`. Как и `toArray`, метод `toJson` рекурсивный, так что все атрибуты и все отношения так же будут преобразованы в JSON:

    $user = App\User::find(1);

    return $user->toJson();

Как вариант, можно привести модель или коллекцию к строке, что автоматически вызовет метод `toJson`:

    $user = App\User::find(1);

    return (string) $user;

Т.к. модели и коллекции преобразуются в JSON при приведении к строке, вы можете возвращать объекты Eloquent напрямую из маршрутов и контроллеров:

    Route::get('users', function () {
        return App\User::all();
    });

<a name="hiding-attributes-from-json"></a>
## Скрытие атрибутов при экспорте в JSON

Иногда необходимо ограничить видимость атрибутов (например, пароля) при конвертации в массив или JSON представление. Для реализации этого функционала добавьте атрибут `$hidden` к своей модели:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Перечень полей, которые должны быть скрыты при экспорте.
         *
         * @var array
         */
        protected $hidden = ['password'];
    }

> **Обратите внимание:** когда скрываете отношения, используйте  **название метода**, а не имя динамического атрибута.

В качестве альтернативного варианта, можно использовать атрибут `visible` и определить «белый» список разрешенных полей и отношений при конвертации в JSON:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Перечень полей, которые должны быть доступны при экспорте.
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }

#### Временно открываем спрятанные атрибуты

Если вам необходимо в какой-то ситуации открыть обычно спрятанный атрибут, используйте метод `makeVisible`. Он возвращает инстанс модели для удобства построения цепочки вызовов:

    return $user->makeVisible('attribute')->toArray();

<a name="appending-values-to-json"></a>
## Добавление атрибутов в JSON

Бывают ситуации, когда при экспорте вам может понадобиться добавить атрибуты, соответствующих полей для которых в БД нет. Чтобы сделать это, для начала, определите для него [аксессор](/docs/{{version}}/eloquent-mutators):

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Получить признак админа для пользователя.
         *
         * @return bool
         */
        public function getIsAdminAttribute()
        {
            return $this->attributes['admin'] == 'yes';
        }
    }

После того, как такой метод создан, добавьте имя атрибута в свойство `appends`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Перечень атрибутов для добавления при экспорте через аксессоры.
         *
         * @var array
         */
        protected $appends = ['is_admin'];
    }

После того как атрибут добавлен в массив `appends`, он будет доступен как при конвертации в массив так и JSON. К этим атрибутам также относятся правила, заданные в `visible` и `hidden` массивах.
