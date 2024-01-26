---
git: 46c2634ef5a4f15427c94a3157b626cf5bd3937f
---

# Eloquent · Мутаторы и типизация

<a name="introduction"></a>
## Введение

Аксессоры, мутаторы и приведение атрибутов к типам позволяют преобразовывать значения атрибутов Eloquent, когда вы извлекаете экземпляр модели или присваиваете их экземпляру модели. Например, вы можете использовать [шифровальщик Laravel](/docs/{{version}}/encryption), чтобы зашифровать значение при его сохранении в базу данных, а затем автоматически расшифровать атрибут при доступе к нему в модели Eloquent. Или вы можете преобразовать строку JSON, которая хранится в вашей базе данных, в массив при доступе к ней через вашу модель Eloquent.

<a name="accessors-and-mutators"></a>
## Аксессоры и мутаторы (Accessors and Mutators)

<a name="defining-an-accessor"></a>
### Определение аксессора (Accessor)

Аксессор преобразует значение атрибута экземпляра Eloquent при обращении к нему. Чтобы определить аксессор, создайте protected метод в вашей модели, представляющий соответствующий атрибут. Имя этого метода должно соответствовать "camel case" представлению фактического атрибута модели / столбца базы данных, когда это применимо.

В этом примере мы определим аксессор для атрибута `first_name`. Этот аксессор будет автоматически вызываться Eloquent при попытке получения значения атрибута first_name. Все методы аксессоров и мутаторов атрибутов должны объявлять тип возвращаемого значения `Illuminate\Database\Eloquent\Casts\Attribute`:


    <?php

    namespace App\Models;
    
    use Illuminate\Database\Eloquent\Casts\Attribute;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Получить имя пользователя.
         */
        protected function firstName(): Attribute
        {
            return Attribute::make(
                get: fn (string $value) => ucfirst($value),
            );
        }
    }


Все аксессоры возвращают экземпляр `Attribute`, который определяет, как будет осуществлен доступ к атрибуту и, при необходимости, его мутация. В данном примере мы определяем только способ доступа к атрибуту. Для этого мы передаем аргумент `get` конструктору класса `Attribute`.

Как видите, исходное значение столбца передается аксессору, что позволяет вам манипулировать и возвращать значение. Чтобы получить доступ к значению аксессора, вы можете просто получить доступ к атрибуту `first_name` экземпляра модели:

    use App\Models\User;

    $user = User::find(1);

    $firstName = $user->first_name;

> [!NOTE]  
> Если вы хотите, чтобы эти вычисленные значения были добавлены к представлениям массива / JSON вашей модели, [вам нужно будет добавить их](/docs/{{version}}/eloquent-serialization#appending-values-to-json).

<a name="building-value-objects-from-multiple-attributes"></a>
#### Создание объектов-значений из нескольких атрибутов

Иногда вашему аксессору может потребоваться преобразовать несколько атрибутов модели в один "объект-значение" (value object). Для этого ваше замыкание `get` может принимать второй аргумент `$attributes`, который будет автоматически передаваться в замыкание и будет содержать массив всех текущих атрибутов модели:

```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

/**
 * Взаимодействуйте с адресом пользователя.
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
    );
}
```

<a name="accessor-caching"></a>
#### Кэширование аксессоров

При возвращении объектов-значений из аксессоров любые изменения, внесенные в объект-значение, автоматически синхронизируются с моделью перед ее сохранением. Это возможно благодаря тому, что Eloquent сохраняет экземпляры, возвращаемые аксессорами, чтобы каждый раз при вызове аксессора возвращать тот же экземпляр:

    use App\Models\User;

    $user = User::find(1);

    $user->address->lineOne = 'Updated Address Line 1 Value';
    $user->address->lineTwo = 'Updated Address Line 2 Value';

    $user->save();


Однако иногда вам может потребоваться включить кэширование для примитивных значений, таких как строки и логические значения, особенно если они требуют больших вычислительных ресурсов. Для этого вы можете вызвать метод `shouldCache` при определении вашего аксессора:


```php
protected function hash(): Attribute
{
    return Attribute::make(
        get: fn (string $value) => bcrypt(gzuncompress($value)),
    )->shouldCache();
}
```

Если вы хотите отключить кэширование объектов для атрибутов, вы можете вызвать метод `withoutObjectCaching` при определении атрибута:

```php
/**
 * Взаимодействуйте с адресом пользователя.
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
    )->withoutObjectCaching();
}
```

<a name="defining-a-mutator"></a>
### Определение мутатора (Mutator)

Мутатор преобразует значение атрибута в момент их присвоения экземпляру Eloquent. Чтобы определить мутатор, вы можете использовать аргумент `set` при определении вашего атрибута.

Определим мутатор для атрибута `first_name`. Этот мутатор будет автоматически вызываться, когда мы попытаемся присвоить значение атрибута `first_name` модели:

        <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Casts\Attribute;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Манипуляции с именем пользователя
         */
        protected function firstName(): Attribute
        {
            return Attribute::make(
                get: fn (string $value) => ucfirst($value),
                set: fn (string $value) => strtolower($value),
            );
        }
    }

Замыкание мутатора получит значение, заданное для атрибута, что позволит вам манипулировать этим значением и возвращать измененное значение. Чтобы использовать наш мутатор, нам нужно только установить атрибут `first_name` для модели Eloquent:

    use App\Models\User;

    $user = User::find(1);

    $user->first_name = 'Sally';

В этом примере замыкание `set` будет вызываться со значением `Sally`. Затем, мутатор применит к имени функцию `strtolower` и установит полученное значение во внутреннем массиве `$attributes`.

<a name="mutating-multiple-attributes"></a>
#### Мутация нескольких атрибутов

Иногда вашему мутатору может потребоваться установить несколько атрибутов в основной модели. Для этого вы можете вернуть массив из замыкания `set`. Каждый ключ в массиве должен соответствовать базовому атрибуту / столбцу базы данных, связанному с моделью:


```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

/**
 * Манипуляции с адресом пользователя.
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
        set: fn (Address $value) => [
            'address_line_one' => $value->lineOne,
            'address_line_two' => $value->lineTwo,
        ],
    );
}
```
<a name="attribute-casting"></a>
## Приведение атрибутов к типам

Приведение атрибутов обеспечивает функциональность, аналогичную аксессорам и мутаторам, но без необходимости определения каких-либо дополнительных методов вашей модели. Вместо этого свойство `$casts` вашей модели представляет удобный способ преобразования атрибутов в распространенные типы данных.

Свойство `$casts` должно быть массивом, где ключ – это имя преобразуемого атрибута, а значение – это тип, к которому вы хотите привести столбец. Поддерживаемые типы преобразования:

<!-- <div class="content-list" markdown="1"> -->

- `array`
- `AsStringable::class`
- `boolean`
- `collection`
- `date`
- `datetime`
- `immutable_date`
- `immutable_datetime`
- `decimal:<precision>`
- `double`
- `encrypted`
- `encrypted:array`
- `encrypted:collection`
- `encrypted:object`
- `float`
- `hashed`
- `integer`
- `object`
- `real`
- `string`
- `timestamp`

<!-- </div> -->

Чтобы продемонстрировать преобразование атрибутов, давайте преобразуем атрибут `is_admin`, который хранится в нашей базе данных в виде целого числа (`0` или `1`), в логическое значение:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Атрибуты, которые должны быть типизированы.
         *
         * @var array
         */
        protected $casts = [
            'is_admin' => 'boolean',
        ];
    }

После определения типизации, атрибут `is_admin` всегда будет преобразован в логическое значение при доступе к нему, даже если базовое значение хранится в базе данных как целое число:

    $user = App\Models\User::find(1);

    if ($user->is_admin) {
        // ...
    }

Если вам нужно добавить новое временное приведение во время выполнения, вы можете использовать метод `mergeCasts`. Эти определения приведения будут добавлены к любому из уже определенных для модели приведения:

    $user->mergeCasts([
        'is_admin' => 'integer',
        'options' => 'object',
    ]);

> [!Warning]
> Атрибуты, которые имеют значение `null`, не будут преобразованы. Кроме того, вы никогда не должны определять типизацию (или атрибут), имя которого совпадает с именем отношения.

<a name="stringable-casting"></a>
#### Преобразование в строку

Вы можете использовать класс приведения `Illuminate\Database\Eloquent\Casts\AsStringable` для приведения атрибута модели к объекту [строки Fluent `Illuminate\Support\Stringable`](/docs/{{version}}/strings#fluent-strings-method-list):

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Casts\AsStringable;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Атрибуты, которые должны быть типизированы.
         *
         * @var array
         */
        protected $casts = [
            'directory' => AsStringable::class,
        ];
    }

<a name="array-and-json-casting"></a>
### Преобразование в массив и JSON

Преобразование в `array` особенно полезно при работе со столбцами, которые хранятся как сериализованный JSON. Например, если ваша база данных имеет поле типа `JSON` или `TEXT`, содержащее сериализованный JSON, то добавленная типизация `array` этому атрибуту автоматически десериализует атрибут модели Eloquent в массив PHP при обращении к нему:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Атрибуты, которые должны быть типизированы.
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }

Как только типизация определена, вы можете получить доступ к атрибуту `options`, и он будет автоматически десериализован из JSON в массив PHP. Когда вы устанавливаете значение атрибута `options`, данный массив будет автоматически сериализован обратно в JSON для сохранения:

    use App\Models\User;

    $user = User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();

Чтобы обновить одно поле JSON-атрибута с помощью краткого синтаксиса, вы можете [разрешить масссовое назначение](/docs/{{version}}/eloquent#mass-assignment-json-columns) и использовать оператор `->` при вызове метода `update`:

    $user = User::find(1);

    $user->update(['options->key' => 'value']);

<a name="array-object-and-collection-casting"></a>
#### Типизация ArrayObject и Collection

Хотя типизации стандартного `array` достаточно для многих приложений, но у него есть некоторые недостатки. Поскольку типизация `array` возвращает примитивный тип, невозможно напрямую изменить смещение массива. Например, следующий код вызовет ошибку PHP:

    $user = User::find(1);

    $user->options['key'] = $value;

Чтобы решить эту проблему, Laravel предлагает типизацию `AsArrayObject`, которая преобразует ваш атрибут JSON в класс [ArrayObject](https://www.php.net/manual/ru/class.arrayobject.php). Эта функция реализована с использованием реализации [пользовательской типизации](#custom-casts) Laravel, которая позволяет Laravel интеллектуально кешировать и преобразовывать измененный объект таким образом, что отдельные смещения могли быть изменены без ошибок PHP. Чтобы использовать типизацию `AsArrayObject`, просто назначьте его атрибуту:

    use Illuminate\Database\Eloquent\Casts\AsArrayObject;

    /**
     * Атрибуты, которые должны быть типизированы.
     *
     * @var array
     */
    protected $casts = [
        'options' => AsArrayObject::class,
    ];

Точно так же Laravel предлагает типизацию `AsCollection`, которая преобразует ваш атрибут JSON в экземпляр Laravel [Collection](/docs/{{version}}/collections):

    use Illuminate\Database\Eloquent\Casts\AsCollection;

    /**
     * Атрибуты, которые должны быть типизированы.
     *
     * @var array
     */
    protected $casts = [
        'options' => AsCollection::class,
    ];

Если вы хотите, чтобы приведение типа `AsCollection` создавало экземпляр пользовательского класса коллекции вместо базового класса коллекции Laravel, вы можете указать имя класса коллекции в качестве аргумента приведения типа:


    use App\Collections\OptionCollection;
    use Illuminate\Database\Eloquent\Casts\AsCollection;

    /**
     * Атрибуты, которые должны быть типизированы.
     *
     * @var array
     */
    protected $casts = [
        'options' => AsCollection::class.':'.OptionCollection::class,
    ];

<a name="date-casting"></a>
### Типизация даты

По умолчанию Eloquent преобразует столбцы `created_at` и `updated_at` в экземпляры [Carbon](https://github.com/briannesbitt/Carbon), расширяющего класс DateTime PHP и предоставляющего набор полезных методов. Вы можете типизировать дополнительные атрибуты даты, определив дополнительные преобразования даты в массиве свойств вашей модели `$cast`. Обычно даты следует приводить с использованием типизации `datetime` или `immutable_datetime`.

При определении типизации `date` или `datetime` вы также можете указать формат даты. Этот формат будет использоваться, когда [модель сериализуется в массив или JSON](/docs/{{version}}/eloquent-serialization):

    /**
     * Атрибуты, которые должны быть типизированы.
     *
     * @var array
     */
    protected $casts = [
        'created_at' => 'datetime:Y-m-d',
    ];

Когда столбец типизирован как дата, вы можете установить соответствующее значение атрибута модели в виде временной метки форматов UNIX, строки даты (`Y-m-d`), строки даты-времени или экземпляров `DateTime` / `Carbon`. Значение даты будет правильно преобразовано и сохранено в вашей базе данных.

Вы можете настроить формат сериализации по умолчанию для всех дат вашей модели, переопределив метод `serializeDate` вашей модели. Этот метод не влияет на форматирование дат для их сохранения в базе данных:

    /**
     * Подготовить дату для сериализации массива / JSON.
     */
    protected function serializeDate(DateTimeInterface $date): string
    {
        return $date->format('Y-m-d');
    }

Чтобы указать формат, который следует использовать при фактическом сохранении дат модели в вашей базе данных, вы должны определить свойство `$dateFormat` вашей модели:

    /**
     * Формат хранения столбцов даты модели.
     *
     * @var string
     */
    protected $dateFormat = 'U';

<a name="date-casting-and-timezones"></a>
#### Приведение даты, сериализация и часовые пояса

По умолчанию приведение `date` и `datetime` будут сериализовывать даты в строку даты UTC ISO-8601 (`YYYY-MM-DDTHH:MM:SS.uuuuuuZ`), независимо от часового пояса, указанного в конфигурации `timezone` вашего приложения. Вам настоятельно рекомендуется всегда использовать этот формат сериализации, а также хранить даты вашего приложения в часовом поясе UTC, не изменяя параметр конфигурации вашего приложения `timezone` от значения по умолчанию `UTC`. Последовательное использование часового пояса UTC во всем приложении обеспечит максимальный уровень взаимодействия с другими библиотеками обработки даты, написанными на PHP и JavaScript.

Если к приведению `date` или `datetime` применяется настраиваемый формат, такой как `datetime:Y-m-d H:i:s`, внутренний часовой пояс экземпляра Carbon будет использоваться во время сериализации даты. Обычно это часовой пояс, указанный в параметре конфигурации вашего приложения `timezone`.

<a name="enum-casting"></a>
### Типизация "Enum"

Eloquent также позволяет вам преобразовывать значения ваших атрибутов в [перечисления](https://www.php.net/manual/en/language.enumerations.backed.php) PHP. Для этого вы можете указать атрибут и перечисление, которое вы хотите преобразовать, в массиве свойств вашей модели`$casts`:

    use App\Enums\ServerStatus;

    /**
     * Атрибуты, которые должны быть типизированы.
     *
     * @var array
     */
    protected $casts = [
        'status' => ServerStatus::class,
    ];

После того как вы определили приведение в своей модели, указанный атрибут будет автоматически преобразован в перечисление и из него, когда вы взаимодействуете с атрибутом:

    if ($server->status == ServerStatus::Provisioned) {
        $server->status = ServerStatus::Provisioned;

        $server->save();
    }

<a name="casting-arrays-of-enums"></a>
#### Типизация массивов перечислений
Иногда вам может потребоваться, чтобы ваша модель сохраняла массив значений перечисления в одном столбце. Для этого вы можете воспользоваться приведением `AsEnumArrayObject` или `AsEnumCollection`, предоставленными Laravel:

use App\Enums\ServerStatus;
use Illuminate\Database\Eloquent\Casts\AsEnumCollection;

    /**
     * Атрибуты, которые должны быть типизированы.
     *
     * @var array
     */
    protected $casts = [
        'statuses' => AsEnumCollection::class.':'.ServerStatus::class,
    ];

<a name="encrypted-casting"></a>
### Типизация "Encrypted"

Приведение `encrypted` зашифрует значение атрибута модели, используя встроенные в Laravel функции [шифрования](/docs/{{version}}/encryption). Кроме того, преобразование `encrypted:array`, `encrypted:collection`, `encrypted:object`, `AsEncryptedArrayObject` и `AsEncryptedCollection` работает так же, как и их незашифрованные копии; однако, как и следовало ожидать, базовое значение зашифровано при хранении в вашей базе данных.

Поскольку окончательная длина зашифрованного текста непредсказуема и больше, чем его копия в виде обычного текста, убедитесь, что связанный столбец базы данных имеет тип `TEXT` или больше. Кроме того, поскольку значения зашифрованы в базе данных, вы не сможете запрашивать или искать зашифрованные значения атрибутов.

<a name="key-rotation"></a>
#### Ротация ключей

Как вы, возможно, знаете, для шифрования строк Laravel использует значение `key`, указанное в конфигурационном файле `app` вашего приложения.  Обычно это значение соответствует значению переменной окружения `APP_KEY`. Если вам необходимо изменить ключ шифрования вашего приложения, вам придется вручную повторно зашифровать ваши зашифрованные атрибуты с использованием нового ключа.
<a name="query-time-casting"></a>
### Типизация во время запроса

Иногда может потребоваться применить типизацию при выполнении запроса, например, при выборе сырого значения из таблицы. Например, рассмотрим следующий запрос:

    use App\Models\Post;
    use App\Models\User;

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->get();

Атрибут `last_posted_at` результатов этого запроса будет простой строкой. Было бы замечательно, если бы мы могли применить типизацию `datetime` этого атрибута при выполнении запроса. К счастью, мы можем добиться этого с помощью метода `withCasts`:

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->withCasts([
        'last_posted_at' => 'datetime'
    ])->get();

<a name="custom-casts"></a>
## Пользовательская типизация

В Laravel есть множество встроенных полезных преобразователей; однако иногда требуется определить свои собственные. Для создания типа приведения выполните команду Artisan `make:cast`. Новый класс приведения будет размещен в вашем каталоге `app/Casts`:

Все пользовательские классы приведения должны реализовывать интерфейс `CastsAttributes`. Классы, реализующие этот интерфейс, должны определять методы `get` и `set`. Метод `get` отвечает за преобразование "сырого" значения из базы данных к типизированному значению, а метод `set` – должен преобразовывать типизированное значение в "сырое" значение, которое можно сохранить в базе данных. В качестве примера мы повторно реализуем встроенный преобразователь `json` как пользовательский типизатор:

    <?php

    namespace App\Casts;

    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

    class Json implements CastsAttributes
    {
        /**
         * Привести значение к пользовательскому типу.
         *
         * @param  array<string, mixed>  $attributes
         * @return array<string, mixed>
         */
        public function get(Model $model, string $key, mixed $value, array $attributes): array
        {
            return json_decode($value, true);
        }

        /**
         * Подготовить переданное значение к сохранению.
         *
         * @param array<string, mixed> $attributes
         */
        public function set(Model $model, string $key, mixed $value, array $attributes): string
        {
            return json_encode($value);
        }
    }

После того как вы определили собственный типизатор, вы можете добавить его к атрибуту модели, используя его имя класса:

    <?php

    namespace App\Models;

    use App\Casts\Json;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Атрибуты, которые должны быть типизированы.
         *
         * @var array
         */
        protected $casts = [
            'options' => Json::class,
        ];
    }

<a name="value-object-casting"></a>
### Типизация объект-значение

Вы не ограничены приведением значений к примитивным типам. Вы также можете преобразовать значения к объектам. Определение пользовательских типизаторов, которые преобразуют значения в объекты, очень похоже на приведение к примитивным типам; однако метод `set` должен возвращать массив пар ключ / значение, который будет использоваться для установки сырых значений, сохраняемых в модели.

В качестве примера мы определим собственный класс типизатора, который преобразует несколько значений модели в один объект-значение `Address`. Предположим, что значение `Address` имеет два общедоступных свойства: `lineOne` и `lineTwo`:

    <?php

    namespace App\Casts;

    use App\ValueObjects\Address as AddressValueObject;
    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
    use Illuminate\Database\Eloquent\Model;
    use InvalidArgumentException;

    class Address implements CastsAttributes
    {
        /**
         * Преобразовать значение к пользовательскому типу.
         *
         * @param array<string, mixed> $attributes
         */
        public function get(Model $model, string $key, mixed $value, array $attributes): AddressValueObject
        {
            return new AddressValueObject(
                $attributes['address_line_one'],
                $attributes['address_line_two']
            );
        }

        /**
         * Подготовить переданное значение к сохранению.
         *
         * @param  array<string, mixed>  $attributes
         * @return array<string, string>
         */
        public function set(Model $model, string $key, mixed $value, array $attributes): array
        {
            if (! $value instanceof AddressValueObject) {
                throw new InvalidArgumentException('The given value is not an Address instance.');
            }

            return [
                'address_line_one' => $value->lineOne,
                'address_line_two' => $value->lineTwo,
            ];
        }
    }

При приведении к объектам-значениям любые изменения, внесенные в объект-значения, будут автоматически синхронизированы с моделью до ее сохранения:

    use App\Models\User;

    $user = User::find(1);

    $user->address->lineOne = 'Updated Address Value';

    $user->save();
> [!NOTE] 
> Если вы планируете сериализовать свои модели Eloquent, содержащие объекты-значения, в JSON или массивы, вам следует реализовать интерфейсы `Illuminate\Contracts\Support\Arrayable` и `JsonSerializable` для объекта-значения.

<a name="value-object-caching"></a>
#### Кеширование объектов-значений

Когда атрибуты, которые приведены к объектам значений, вычислены, Eloquent кэширует их. Таким образом, при повторном доступе к атрибуту будет возвращен тот же самый экземпляр объекта.

Если вы хотите отключить поведение кэширования объектов в вашем пользовательском классе приведения, объявите public свойство `withoutObjectCaching` в вашем пользовательском классе приведения:

```php
class Address implements CastsAttributes
{
    public bool $withoutObjectCaching = true;

    // ...
}
```

<a name="array-json-serialization"></a>
### Сериализация в массив и JSON

Когда модель Eloquent преобразуется в массив или JSON с использованием методов `toArray` и `toJson`, ваши пользовательские типизаторы объекты-значения обычно будут сериализованы, в частности, пока они (типизаторы) реализуют интерфейсы `Illuminate\Contracts\Support\Arrayable` и `JsonSerializable`. Однако при использовании объектов-значений, предоставляемых сторонними библиотеками, у вас может не быть возможности добавить эти интерфейсы к объекту.

Поэтому вы можете указать, что ваш собственный класс типизатора будет отвечать за сериализацию объекта-значения. Для этого ваш собственный класс типизатора должен реализовывать интерфейс `Illuminate\Contracts\Database\Eloquent\SerializesCastableAttributes`. В этом интерфейсе указано, что ваш класс должен содержать метод `serialize`, возвращающий сериализованную форму вашего объекта значения:

    /**
     * Получить сериализованное представление значения.
     *
     * @param  array<string, mixed>  $attributes
     */
    public function serialize(Model $model, string $key, mixed $value, array $attributes): string
    {
        return (string) $value;
    }

<a name="inbound-casting"></a>
### Входящая типизация

Иногда требуется написать свой типизатор, который только преобразует указанные значения атрибутов модели, и не выполняет никаких операций при обращении к этим атрибутам. 

Пользовательские типизаторы только для входящих значений должны реализовывать интерфейс `CastsInboundAttributes`, требующий определение метода `set`. Вызовите команду Artisan `make:cast` с опцией `--inbound`, чтобы сгенерировать класс приведения только для входящих значений:

```shell
php artisan make:cast Hash --inbound
```

Классическим примером только входящей типизации является «хеширование». Например, мы можем определить типизатор, которое хеширует входящие значения с использованием указанного алгоритма:

    <?php

    namespace App\Casts;

    use Illuminate\Contracts\Database\Eloquent\CastsInboundAttributes;
    use Illuminate\Database\Eloquent\Model;

    class Hash implements CastsInboundAttributes
    {
        

        /**
         * Создать новый экземпляр класса типизации.
         */
        public function __construct(
            protected string|null $algorithm = null,
        ) {}

        /**
         * Подготовить переданное значение к сохранению.
         *
         * @param array<string, mixed> $attributes
         */
        public function set(Model $model, string $key, mixed $value, array $attributes): string
        {
            return is_null($this->algorithm)
                        ? bcrypt($value)
                        : hash($this->algorithm, $value);
        }
    }

<a name="cast-parameters"></a>
### Параметры типизации

При добавлении пользовательского типизатора к модели, параметры типизатора задаются отделением их от имени класса с помощью символа `:` и разделением нескольких параметров запятыми. Параметры будут переданы в конструктор класса типизатора:

    /**
     * Атрибуты, которые должны быть типизированы.
     *
     * @var array
     */
    protected $casts = [
        'secret' => Hash::class.':sha256',
    ];

<a name="castables"></a>
### Интерфейс `Castable`

Вы можете разрешить объектам-значениям вашего приложения определять свои собственные классы типизаторы. Вместо указания пользовательской типизации в модели, вы можете альтернативно указать класс, который реализует интерфейс `Illuminate\Contracts\Database\Eloquent\Castable`:

    use App\ValueObjects\Address;

    protected $casts = [
        'address' => Address::class,
    ];

Объекты, реализующие интерфейс `Castable`, должны определять метод `castUsing`, который возвращает имя [пользовательского класса типизатора](#value-object-casting), отвечающего за двустороннее преобразование:

    <?php

    namespace App\ValueObjects;

    use Illuminate\Contracts\Database\Eloquent\Castable;
    use App\Casts\Address as AddressCast;

    class Address implements Castable
    {
        /**
         * Получить имя класса типизатора для использования двустороннего преобразования.
         *
         * @param array<string, mixed> $arguments
         */
        public static function castUsing(array $arguments): string
        {
            return AddressCast::class;
        }
    }

При использовании классов `Castable` вы все равно можете указывать аргументы в свойстве `$casts`. Аргументы будут переданы методу `castUsing`:

    use App\ValueObjects\Address;

    protected $casts = [
        'address' => Address::class.':argument',
    ];

<a name="anonymous-cast-classes"></a>
#### Интерфейс `Castable` и анонимные классы типизаторов

Комбинируя `castable` и [анонимными классами](https://www.php.net/manual/ru/language.oop5.anonymous.php) PHP, вы можете определить объект-значение и его логику преобразования как единый типизируемый объект. Для этого верните анонимный класс из метода `castUsing` вашего объекта-значения. Анонимный класс должен реализовывать интерфейс `CastsAttributes`:

    <?php

    namespace App\ValueObjects;

    use Illuminate\Contracts\Database\Eloquent\Castable;
    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

    class Address implements Castable
    {
        // ...

        /**
         * Получить имя класса типизатора для использования двустороннего преобразования.
         *
         * @param array<string, mixed> $arguments
         */
        public static function castUsing(array $arguments): CastsAttributes
        {
            return new class implements CastsAttributes
            {
                public function get(Model $model, string $key, mixed $value, array $attributes): Address
                {
                    return new Address(
                        $attributes['address_line_one'],
                        $attributes['address_line_two']
                    );
                }

                public function set(Model $model, string $key, mixed $value, array $attributes): array
                {
                    return [
                        'address_line_one' => $value->lineOne,
                        'address_line_two' => $value->lineTwo,
                    ];
                }
            };
        }
    }
