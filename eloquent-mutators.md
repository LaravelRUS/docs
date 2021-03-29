# Laravel 8 · Eloquent · Мутаторы и типизация

- [Введение](#introduction)
- [Аксессоры и мутаторы](#accessors-and-mutators)
    - [Определение аксессора](#defining-an-accessor)
    - [Определение мутатора](#defining-a-mutator)
- [Приведение атрибутов к типам](#attribute-casting)
    - [Преобразование в массив и JSON](#array-and-json-casting)
    - [Типизация даты](#date-casting)
    - [Типизация во время запроса](#query-time-casting)
- [Пользовательская типизация](#custom-casts)
    - [Типизация объект-значение](#value-object-casting)
    - [Сериализация в массив и JSON](#array-json-serialization)
    - [Входящая типизация](#inbound-casting)
    - [Параметры типизации](#cast-parameters)
    - [Интерфейс `Castable`](#castables)

<a name="introduction"></a>
## Введение

Аксессоры, мутаторы и приведение атрибутов к типам позволяют преобразовывать значения атрибутов Eloquent, когда вы извлекаете экземпляр модели или присваиваете их экземпляру модели. Например, вы можете использовать [шифровальщик Laravel](encryption), чтобы зашифровать значение при его сохранении в базу данных, а затем автоматически расшифровать атрибут при доступе к нему в модели Eloquent. Или вы можете преобразовать строку JSON, которая хранится в вашей базе данных, в массив при доступе к ней через вашу модель Eloquent.

<a name="accessors-and-mutators"></a>
## Аксессоры и мутаторы

<a name="defining-an-accessor"></a>
### Определение аксессора

Аксессор преобразует значение атрибута экземпляра Eloquent при обращении к нему. Чтобы определить метод доступа, создайте метод `get{Attribute}Attribute` в вашей модели, где `{Attribute}` – это имя столбца, к которому вы хотите получить доступ, в «верхнем» регистре.

В этом примере мы определим аксессор для атрибута `first_name`. Аксессор будет автоматически вызван Eloquent при попытке получить значение атрибута `first_name`:

    <?php

    namespace App\Models;

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

Как видите, исходное значение столбца передается аксессору, что позволяет вам манипулировать и возвращать значение. Чтобы получить доступ к значению аксессора, вы можете просто получить доступ к атрибуту `first_name` экземпляра модели:

    use App\Models\User;

    $user = User::find(1);

    $firstName = $user->first_name;

Вы не ограничены взаимодействием с одним атрибутом в вашем аксессоре. Вы также можете использовать аксессор для возврата новых вычисленных значений из существующих атрибутов:

    /**
     * Получить полное имя пользователя.
     *
     * @return string
     */
    public function getFullNameAttribute()
    {
        return "{$this->first_name} {$this->last_name}";
    }

> {tip} Если вы хотите, чтобы эти вычисленные значения были добавлены к представлениям массива / JSON вашей модели, [вам нужно будет добавить их](eloquent-serialization.md#appending-values-to-json).

<a name="defining-a-mutator"></a>
### Определение мутатора

Мутатор преобразует значение атрибута в момент их присвоения экземпляру Eloquent. Чтобы определить мутатор, определите метод `set{Attribute}Attribute` в вашей модели, где `{Attribute}` – это имя столбца, к которому вы хотите получить доступ, в «верхнем» регистре.

Определим мутатор для атрибута `first_name`. Этот мутатор будет автоматически вызываться, когда мы попытаемся присвоить значение атрибута `first_name` модели:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Присвоить имя пользователю.
         *
         * @param  string  $value
         * @return void
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }

Мутатор получит значение, заданное для атрибута, что позволит вам манипулировать этим значением и устанавливать желаемое значение во внутреннем свойстве `$attributes` модели Eloquent. Чтобы использовать наш мутатор, нам нужно только установить атрибут `first_name` для модели Eloquent:

    use App\Models\User;

    $user = User::find(1);

    $user->first_name = 'Sally';

В этом примере метод `setFirstNameAttribute` будет вызываться со значением `Sally`. Затем, мутатор применит к имени функцию `strtolower` и установит полученное значение во внутреннем массиве `$attributes`.

<a name="attribute-casting"></a>
## Приведение атрибутов к типам

Приведение атрибутов обеспечивает функциональность, аналогичную аксессорам и мутаторам, но без необходимости определения каких-либо дополнительных методов вашей модели. Вместо этого свойство `$casts` вашей модели представляет удобный способ преобразования атрибутов в распространенные типы данных.

Свойство `$casts` должно быть массивом, где ключ – это имя преобразуемого атрибута, а значение – это тип, к которому вы хотите привести столбец. Поддерживаемые типы преобразования:

<!-- <div class="content-list" markdown="1"> -->
- `array`
- `boolean`
- `collection`
- `date`
- `datetime`
- `decimal:<digits>`
- `double`
- `encrypted`
- `encrypted:array`
- `encrypted:collection`
- `encrypted:object`
- `float`
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
        //
    }

> {note} Атрибуты, которые имеют значение `null`, не будут преобразованы. Кроме того, вы никогда не должны определять типизацию (или атрибут), имя которого совпадает с именем отношения.

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

Чтобы обновить одно поле JSON-атрибута с помощью краткого синтаксиса, используйте оператор `->` при вызове метода `update`:

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

Точно так же Laravel предлагает типизацию `AsCollection`, которая преобразует ваш атрибут JSON в экземпляр Laravel [Collection](collections):

    use Illuminate\Database\Eloquent\Casts\AsCollection;

    /**
     * Атрибуты, которые должны быть типизированы.
     *
     * @var array
     */
    protected $casts = [
        'options' => AsCollection::class,
    ];

<a name="date-casting"></a>
### Типизация даты

По умолчанию Eloquent преобразует столбцы `created_at` и `updated_at` в экземпляры [Carbon](https://github.com/briannesbitt/Carbon), расширяющего класс DateTime PHP и предоставляющего набор полезных методов. Вы можете типизировать дополнительные атрибуты даты, определив дополнительные преобразования даты в массиве свойств вашей модели `$cast`. Обычно даты следует приводить с использованием типизации `datetime`.

При определении типизации `date` или `datetime` вы также можете указать формат даты. Этот формат будет использоваться, когда [модель сериализуется в массив или JSON](eloquent-serialization):

    /**
     * Атрибуты, которые должны быть типизированы.
     *
     * @var array
     */
    protected $casts = [
        'created_at' => 'datetime:Y-m-d',
    ];

Когда столбец типизирован как дата, вы можете установить его значение в виде временной метки форматов UNIX, строки даты (`Y-m-d`), строки даты-времени или экземпляров `DateTime` / `Carbon`. Значение даты будет правильно преобразовано и сохранено в вашей базе данных:

Вы можете настроить формат сериализации по умолчанию для всех дат вашей модели, переопределив метод `serializeDate` вашей модели. Этот метод не влияет на форматирование дат для их сохранения в базе данных:

    /**
     * Подготовить дату для сериализации массива / JSON.
     *
     * @param  \DateTimeInterface  $date
     * @return string
     */
    protected function serializeDate(DateTimeInterface $date)
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

В Laravel есть множество встроенных полезных преобразователей; однако иногда требуется определить свои собственные. Вы можете добиться этого, определив класс, реализующий интерфейс `CastsAttributes`.

Классы, реализующие этот интерфейс, должны определять методы `get` и `set`. Метод `get` отвечает за преобразование сырого значения из базы данных к типизированному значению, а метод `set` – должен преобразовывать типизированное значение в сырое значение, которое можно сохранить в базе данных. В качестве примера мы повторно реализуем встроенный преобразователь `json` как пользовательский типизатор:

    <?php

    namespace App\Casts;

    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

    class Json implements CastsAttributes
    {
        /**
         * Преобразовать значение к пользовательскому типу.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  mixed  $value
         * @param  array  $attributes
         * @return array
         */
        public function get($model, $key, $value, $attributes)
        {
            return json_decode($value, true);
        }

        /**
         * Подготовить переданное значение к сохранению.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  array  $value
         * @param  array  $attributes
         * @return string
         */
        public function set($model, $key, $value, $attributes)
        {
            return json_encode($value);
        }
    }

После того, как вы определили собственный типизатор, вы можете добавить его к атрибуту модели, используя его имя класса:

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

    use App\Models\Address as AddressModel;
    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
    use InvalidArgumentException;

    class Address implements CastsAttributes
    {
        /**
         * Преобразовать значение к пользовательскому типу.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  mixed  $value
         * @param  array  $attributes
         * @return \App\Models\Address
         */
        public function get($model, $key, $value, $attributes)
        {
            return new AddressModel(
                $attributes['address_line_one'],
                $attributes['address_line_two']
            );
        }

        /**
         * Подготовить переданное значение к сохранению.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  \App\Models\Address  $value
         * @param  array  $attributes
         * @return array
         */
        public function set($model, $key, $value, $attributes)
        {
            if (! $value instanceof AddressModel) {
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

> {tip} Если вы планируете сериализовать свои модели Eloquent, содержащие объекты-значения, в JSON или массивы, вам следует реализовать интерфейсы `Illuminate\Contracts\Support\Arrayable` и `JsonSerializable` для объекта-значения.

<a name="array-json-serialization"></a>
### Сериализация в массив и JSON

Когда модель Eloquent преобразуется в массив или JSON с использованием методов `toArray` и `toJson`, ваши пользовательские типизаторы объекты-значения обычно будут сериализованы, в частности, пока они (типизаторы) реализуют интерфейсы `Illuminate\Contracts\Support\Arrayable` и `JsonSerializable`. Однако при использовании объектов-значений, предоставляемых сторонними библиотеками, у вас может не быть возможности добавить эти интерфейсы к объекту.

Поэтому вы можете указать, что ваш собственный класс типизатора будет отвечать за сериализацию объекта-значения. Для этого ваш собственный класс типизатора должно реализовывать интерфейс `Illuminate\Contracts\Database\Eloquent\SerializesCastableAttributes`. В этом интерфейсе указано, что ваш класс должен содержать метод `serialize`, возвращающий сериализованную форму вашего объекта значения:

    /**
     * Получить сериализованное представление значения.
     *
     * @param  \Illuminate\Database\Eloquent\Model  $model
     * @param  string  $key
     * @param  mixed  $value
     * @param  array  $attributes
     * @return mixed
     */
    public function serialize($model, string $key, $value, array $attributes)
    {
        return (string) $value;
    }

<a name="inbound-casting"></a>
### Входящая типизация

Иногда требуется написать свой типизатор, который только преобразует указанные значения атрибутов модели, и не выполняет никаких операций при обращении к этим атрибутам. Классическим примером только входящей типизации является «хеширование». Пользовательские типизаторы только для входящих значений должны реализовывать интерфейс `CastsInboundAttributes`, требующий определение метода `set`.

    <?php

    namespace App\Casts;

    use Illuminate\Contracts\Database\Eloquent\CastsInboundAttributes;

    class Hash implements CastsInboundAttributes
    {
        /**
         * Алгоритм хеширования.
         *
         * @var string
         */
        protected $algorithm;

        /**
         * Создать новый экземпляр класса типизации.
         *
         * @param  string|null  $algorithm
         * @return void
         */
        public function __construct($algorithm = null)
        {
            $this->algorithm = $algorithm;
        }

        /**
         * Подготовить переданное значение к сохранению.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  array  $value
         * @param  array  $attributes
         * @return string
         */
        public function set($model, $key, $value, $attributes)
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

    use App\Models\Address;

    protected $casts = [
        'address' => Address::class,
    ];

Объекты, реализующие интерфейс `Castable`, должны определять метод `castUsing`, который возвращает имя [пользовательского класса типизатора](#value-object-casting), отвечающего за двустороннее преобразование:

    <?php

    namespace App\Models;

    use Illuminate\Contracts\Database\Eloquent\Castable;
    use App\Casts\Address as AddressCast;

    class Address implements Castable
    {
        /**
         * Получить имя класса типизатора для использования двустороннего преобразования.
         *
         * @param  array  $arguments
         * @return string
         */
        public static function castUsing(array $arguments)
        {
            return AddressCast::class;
        }
    }

При использовании классов `Castable` вы все равно можете указывать аргументы в свойстве `$casts`. Аргументы будут переданы методу `castUsing`:

    use App\Models\Address;

    protected $casts = [
        'address' => Address::class.':argument',
    ];

<a name="anonymous-cast-classes"></a>
#### Интерфейс `Castable` и анонимные классы типизаторов

Комбинируя `castable` и [анонимными классами](https://www.php.net/manual/ru/language.oop5.anonymous.php) PHP, вы можете определить объект-значение и его логику преобразования как единый типизируемый объект. Для этого верните анонимный класс из метода `castUsing` вашего объекта-значения. Анонимный класс должен реализовывать интерфейс `CastsAttributes`:

    <?php

    namespace App\Models;

    use Illuminate\Contracts\Database\Eloquent\Castable;
    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

    class Address implements Castable
    {
        // ...

        /**
         * Получить имя класса типизатора для использования двустороннего преобразования.
         *
         * @param  array  $arguments
         * @return object|string
         */
        public static function castUsing(array $arguments)
        {
            return new class implements CastsAttributes
            {
                public function get($model, $key, $value, $attributes)
                {
                    return new Address(
                        $attributes['address_line_one'],
                        $attributes['address_line_two']
                    );
                }

                public function set($model, $key, $value, $attributes)
                {
                    return [
                        'address_line_one' => $value->lineOne,
                        'address_line_two' => $value->lineTwo,
                    ];
                }
            };
        }
    }
