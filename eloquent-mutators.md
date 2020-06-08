git 25e40a7d2b72b34ab67f46f16b9a67754601b96d

---

# Eloquent: Мутаторы

- [Введение](#introduction)
- [Аксессоры и мутаторы](#accessors-and-mutators)
    - [Определение аксессора](#defining-an-accessor)
    - [Определение мутатора](#defining-a-mutator)
- [Мутаторы дат](#date-mutators)
- [Мутаторы атрибутов](#attribute-casting)
    - [Преобразование в массив и JSON](#array-and-json-casting)

<a name="introduction"></a>
## Введение

Аксессоры и мутаторы (буквально - читатели и преобразователи) позволяют вам форматировать значения атрибутов Eloquent при их чтении или записи в экземпляры моделей. Например, вы хотите использовать [шифратор Laravel](/docs/{{version}}/encryption), чтобы зашифровать значение, пока оно хранится в базе, и затем автоматически расшифровать атрибут, когда вы обращаетесь к нему в модели Eloquent.

В дополнение к обычным аксессорам и мутаторам Eloquent также автоматически преобразует поля с датами в экземпляры [Carbon](https://github.com/briannesbitt/Carbon) или даже [преобразует текстовые поля в JSON](#attribute-casting).

<a name="accessors-and-mutators"></a>
## Аксессоры и мутаторы

<a name="defining-an-accessor"></a>
### Определение аксессора

Чтобы определить аксессора, создайте метод `getFooAttribute` в вашей модели, где `Foo` — отформатированное в соответствии со стилем "studly" название столбца, к которому вы хотите иметь доступ. В данном примере мы определим аксессора для атрибута `first_name`. Аксессор будет автоматически вызван Eloquent при попытке получить значение атрибута `first_name`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Получить имя пользователя.
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }

Как видите, первоначальное значение столбца передается аксессору, позволяя вам управлять значением и возвращать его. Чтобы получить доступ к значению аксессора, вы можете просто обратиться к атрибуту `first_name` экземпляра модели:

    $user = App\User::find(1);

    $firstName = $user->first_name;

<a name="defining-a-mutator"></a>
### Определение мутатора

Чтобы определить мутатор, определите метод `setFooAttribute` для своей модели, где `Foo`  — отформатированное в соответствии со стилем "studly" название столбца, к которому вы хотите иметь доступ. И снова давайте определим мутатор для атрибута `first_name`. Этот мутатор будет автоматически вызван, когда мы попытаемся установить значение атрибута `first_name` в модели:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Установить имя пользователя.
         *
         * @param  string  $value
         * @return void
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }

Мутатор получает значение, которое устанавливается в атрибуте, позволяя вам управлять значением и изменять его во внутреннем свойстве `$attributes` модели Eloquent. Так, например, если мы пытаемся установить атрибут `first_name` в значение `Sally`:

    $user = App\User::find(1);

    $user->first_name = 'Sally';

В этом примере функция `setFirstNameAttribute` будет вызвана со значением `Sally`. к имени и установит его результирующее значение во внутреннем массиве `$attributes`.

<a name="date-mutators"></a>
## Мутаторы дат

По умолчанию Eloquent преобразует столбцы `created_at` и `updated_at` в экземпляры [Carbon](https://github.com/briannesbitt/Carbon), которые наследуют PHP-класс `DateTime` и предоставляют ряд полезных методов. Вы можете сами настроить, какие поля автоматически будут преобразовываться, и даже полностью отключить их преобразование, изменив свойство `$dates` вашей модели:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Атрибуты, которые должны быть преобразованы в даты.
         *
         * @var array
         */
        protected $dates = [
            'created_at',
            'updated_at',
            'deleted_at'
        ];
    }

Когда столбец является датой, вы можете установить его значение в формат времени UNIX, в строку даты (`Y-m-d`), в строку даты-времени, и конечно в экземпляр `DateTime` / `Carbon`, и значение даты будет автоматически правильно сохранено в вашей базе данных:

    $user = App\User::find(1);

    $user->deleted_at = Carbon::now();

    $user->save();

Как было отмечено выше, полученные атрибуты, которые перечислены в вашем свойстве `$dates`, будут автоматически преобразованы к экземплярам [Carbon](https://github.com/briannesbitt/Carbon), позволяя вам использовать любой из методов Carbon для ваших атрибутов:

    $user = App\User::find(1);

    return $user->deleted_at->getTimestamp();

#### Форматы дат

По умолчанию метки времени отформатированы как `'Y-m-d H:i:s'`. Если вам нужно настроить формат метки времени, установите значение `$dateFormat` в своей модели. Это свойство определяет, как атрибуты даты хранятся в базе данных, а также их формат, когда модель преобразована в массив или JSON:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Формат хранения столбцов с датами модели.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

<a name="attribute-casting"></a>
## Мутаторы атрибутов

Свойство `$casts` в вашей модели предоставляет удобный метод преобразования атрибутов к общим типам данных. Свойство `$casts` должно быть массивом, где ключ — название преобразуемого атрибута, а значение — тип, в который вы хотите преобразовать столбец. Поддерживаемые типы для преобразования: `integer`, `real`, `float`, `double`, `string`, `boolean`, `object`, `array`, `collection`, `date`, `datetime` и `timestamp`.

Например, давайте привяжем атрибут `is_admin`, который сохранен в нашей базе данных как `integer` (`0` или `1`) к `boolean`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Атрибуты, которые должны быть преобразованы к базовым типам.
         *
         * @var array
         */
        protected $casts = [
            'is_admin' => 'boolean',
        ];
    }

Теперь атрибут `is_admin` будет всегда преобразовываться в `boolean`, когда вы обращаетесь к нему, даже если само значение хранится в базе данных как `integer`:

    $user = App\User::find(1);

    if ($user->is_admin) {
        //
    }

<a name="array-and-json-casting"></a>
### Преобразование в массив и JSON

Тип `array` особенно полезен для преобразования при работе со столбцами, которые хранятся в формате JSON. Например, если у вашей базы данных есть тип поля `JSON` или `TEXT`, который содержит сериализированные JSON данные, добавление преобразования в `array` к этому атрибуту автоматически десериализует атрибут в PHP массив, во время доступа к нему из вашей модели Eloquent:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Атрибуты, которые должны быть преобразованы к базовым типам.
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }

После определения преобразования вы можете обратиться к атрибуту `options` , и он будет автоматически десериализован из JSON в PHP массив. Когда вы зададите значение атрибута `options`, данный массив будет автоматически преобразован обратно в JSON для хранения:

    $user = App\User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();
