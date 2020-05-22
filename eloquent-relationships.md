git f438cac0547e19b83c10872ac7aa1df2afe2be39    

---

# Eloquent: Отношения

- [Введение](#introduction)
- [Определение отношений](#defining-relationships)
    - [Один к одному](#one-to-one)
    - [Один ко многим](#one-to-many)
    - [Один ко многим (Обратное отношение)](#one-to-many-inverse)
    - [Многие ко многим](#many-to-many)
    - [Определение пользовательских моделей промежуточных таблиц](#defining-custom-intermediate-table-models)
    - [Один к одному через](#has-one-through)
    - [Один ко многим через](#has-many-through)
- [Полиморфные отношения](#polymorphic-relationships)
    - [Один к одному](#one-to-one-polymorphic-relations)
    - [Один ко многим](#one-to-many-polymorphic-relations)
    - [Многие ко многим](#many-to-many-polymorphic-relations)
    - [Свои полиморфные отношения](#custom-polymorphic-types)
- [Запросы к отношениям](#querying-relations)
    - [Отношения: методы или свойства](#relationship-methods-vs-dynamic-properties)
    - [Проверка существования связей при выборке](#querying-relationship-existence)
    - [Выборка по отсутствию отношения](#querying-relationship-absence)
    - [Запросы в полиморфных отншениях](#querying-polymorphic-relationships)
    - [Подсчёт моделей в отношении](#counting-related-models)
- [Жадная загрузка](#eager-loading)
    - [Условия для жадных загрузок](#constraining-eager-loads)
    - [Отложенная жадная загрузка](#lazy-eager-loading)
- [Вставка и изменение связанных моделей](#inserting-and-updating-related-models)
    - [Метод `save`](#the-save-method)
    - [Метод `create`](#the-create-method)
    - [Отношения "Принадлежит к"](#updating-belongs-to-relationships)
    - [Отношения многие-ко-многим](#updating-many-to-many-relationships)
- [Установка меток времени родителям](#touching-parent-timestamps)

<a name="introduction"></a>
## Введение

Ваши таблицы скорее всего как-то связаны с другими таблицами БД. Например, статья в блоге может иметь много комментариев, а заказ может быть связан с оставившим его пользователем. Eloquent упрощает работу и управление такими отношениями. Laravel поддерживает следующие типы связей:

- [Один к одному](#one-to-one)
- [Один ко многим](#one-to-many)
- [Многие ко многим](#many-to-many)
- [К одному через](#has-one-through)
- [Ко многим через](#has-many-through)
- [Полиморфные один к одному](#one-to-one-polymorphic-relations)
- [Полиморфные один к многим](#one-to-many-polymorphic-relations)
- [Полиморфные многие ко многим](#many-to-many-polymorphic-relations)

<a name="defining-relationships"></a>
## Определение отношений

Eloquent отношения определены как функции в ваших классах модели Eloquent. С отношениями можно работать используя мощный [конструктор запросов](/docs/{{version}}/queries), который обеспечивает сцепку методов и возможность добавлять запросы. Например, мы можем прицепить дополнительный запрос к отношениям `posts` пользователя:

    $user->posts()->where('active', 1)->get();

Но прежде чем погрузиться в использование отношений, давайте узнаем, чем они отличаются друг от друга.

> {note} Названия отношений в модели не должны совпадать с названиями её свойств (атрибутов).

<a name="one-to-one"></a>
### Один к одному

Связь вида «один к одному» является очень простой. К примеру, модель `User` может иметь один `Phone`. Чтобы определить такое отношение, мы помещаем метод `phone` в модель `User`. Метод `phone` должен вызвать метод `hasOne` и вернуть его результат:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Получить запись с номером телефона пользователя.
         */
        public function phone()
        {
            return $this->hasOne('App\Phone');
        }
    }

Первый параметр, передаваемый `hasOne` — имя связанной модели. Как только отношение установлено, вы можете получить к нему доступ через динамические свойства Eloquent. Динамические свойства позволяют вам получить доступ к функциям отношений, если бы они были свойствами модели:

    $phone = User::find(1)->phone;

Eloquent определяет внешний ключ отношения по имени модели. В данном случае предполагается, что модель `Phone` автоматически будет иметь внешний ключ `user_id`. Если вы хотите это переопределить, то можно передать второй аргумент методу `hasOne`:

    return $this->hasOne('App\Phone', 'foreign_key');

Также Eloquent подразумевает, что внешний ключ должен иметь значение, привязанное к родительскому столбцу  `id` (или другому `$primaryKey`). Другими словами, Eloquent будет искать значение столбца `id` пользователя в столбце `user_id` записи `Phone`. Если вы хотите, чтобы в отношении использовалось значение, отличающееся от `id`, можно передать третий аргумент методу `hasOne`, указав свой ключ пользователя:

    return $this->hasOne('App\Phone', 'foreign_key', 'local_key');

#### Создание обратного отношения

Итак, у нас есть доступ к модели `Phone` из нашего `User`. Теперь давайте определим отношение для модели `Phone`, которое будет иметь доступ к `User`, владеющего этим телефоном. Для создания обратного отношения в `hasOne` используйте метод `belongsTo`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Phone extends Model
    {
        /**
         * Получить пользователя, владеющего данным телефоном.
         */
        public function user()
        {
            return $this->belongsTo('App\User');
        }
    }

В примере выше Eloquent будет стараться искать связь между `user_id` в модели `Phone` и `id` в модели `User`. По умолчанию Eloquent определяет имя внешнего ключа по имени метода отношения, добавляя суффикс `_id`. Однако, если имя внешнего ключа модели `Phone` не `user_id`, можно передать это имя вторым параметром в метод `belongsTo`:

    /**
     * Получить пользователя, владеющего данным телефоном.
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key');
    }

Если ваша родительская модель не использует `id` в качестве первичного ключа, или вам бы хотелось присоединить дочернюю модель к другому столбцу, вы можете передать третий параметр в метод `belongsTo`, который определяет имя связанного столбца в родительской таблице:

    /**
     * Получить пользователя, владеющего данным телефоном.
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key', 'other_key');
    }

<a name="default-models"></a>
#### Модели по умолчанию

Отношение `belongsTo` позволяет вам определить модель по умолчанию, которая будет возвращена в случае, если заданное отношение равно `null`. Этот шаблон проектирования часто называют [Null Object](https://en.wikipedia.org/wiki/Null_Object_pattern) и он может помочь убрать проверку условий в вашем коде. В следующем примере отношение `user` вернёт пустую модель `App\User`, если к публикации не был присоединен ни один `user`:

    /**
     * Получить автора публикации.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault();
    }

Чтобы заполнить модель по умолчанию атрибутами, вы можете передать массив или функцию замыкания методу `withDefault`:

    /**
     * Получить автора публикации.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault([
            'name' => 'Guest Author',
        ]);
    }

    /**
     * Получить автора публикации.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault(function ($user) {
            $user->name = 'Guest Author';
        });
    }

<a name="one-to-many"></a>
### Один ко многим

Отношение "один ко многим" используется для определения отношений, где одна модель владеет некоторым количеством других моделей. Примером отношения "один ко многим" является статья в блоге, которая имеет "много" комментариев. Как и другие отношения Eloquent вы можете смоделировать это отношение таким образом:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * Получить комментарии для публикации в блоге.
         */
        public function comments()
        {
            return $this->hasMany('App\Comment');
        }
    }

Помните, что Eloquent автоматически определяет столбец внешнего ключа в модели `Comment`. По соглашению, Eloquent возьмёт «snake_case» названия модели, в которой находится это свойство, и добавит к нему `_id`. Таким образом, для данного примера, Eloquent предполагает, что внешним ключом для модели `Comment` будет `post_id`.

После определения отношения мы можем получить доступ к коллекции комментариев, обратившись к свойству `comments`. Помните, что поскольку Eloquent поддерживает динамические свойства, мы можем обращаться к функциям отношений, как если бы они были определены свойством модели:

    $comments = App\Post::find(1)->comments;

    foreach ($comments as $comment) {
        //
    }

Вы можете добавлять дополнительные условия к тем комментариям, которые получены вызовом метода `comments`:

    $comment = App\Post::find(1)->comments()->where('title', 'foo')->first();

Как и для метода `hasOne`, вы можете указать внешний и локальный ключи, передав дополнительные параметры в метод `hasMany`:

    return $this->hasMany('App\Comment', 'foreign_key');

    return $this->hasMany('App\Comment', 'foreign_key', 'local_key');

<a name="one-to-many-inverse"></a>
### Один ко многим (обратное отношение)

После получения доступа ко всем комментариям статьи давай определим отношение, которое позволит комментарию получить доступ к его статье. Чтобы определить обратное отношение `hasMany`, давайте определим функцию отношения на дочерней модели, которая вызывает метод `belongsTo`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * Получить публикацию, которой принадлежит этот комментарий.
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

После определения отношений мы можем получить модель `Post` для  `Comment`, обратившись к динамическому свойству `post`:

    $comment = App\Comment::find(1);

    echo $comment->post->title;

В примере выше Eloquent пробует связать `post_id` из модели `Comment` с `id` модели `Post`. По умолчанию Eloquent определяет внешний ключ по имени метода отношения, прибавляя через `_` название первичного ключа (например, `_id`). Однако, если внешний ключ для модели `Comment` не `post_id`, вы можете передать своё имя вторым параметром в метод `belongsTo`:

    /**
     * Получить публикацию, которой принадлежит этот комментарий.
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key');
    }

Если ваша родительская модель не использует `id` в качестве первичного ключа, или вам бы хотелось присоединить дочернюю модель к другому столбцу, вы можете передать третий параметр в метод `belongsTo`, который определяет имя связанного столбца в родительской таблице:

    /**
     * Получить публикацию, которой принадлежит этот комментарий.
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key', 'other_key');
    }

<a name="many-to-many"></a>
### Многие ко многим

Отношения типа "многие ко многим" сложнее отношений `hasOne` и `hasMany`. Примером может служить пользователь, имеющий много ролей, где роли также относятся ко многим пользователям. Например, несколько пользователей могут иметь роль "Admin". 

#### Структура таблиц

Нам нужны три таблицы для этой связи: `users`, `roles` и `role_user`. Имя таблицы `role_user` получается из упорядоченных по алфавиту имён связанных моделей, она должна иметь поля `user_id` и `role_id`.

    users
        id - integer
        name - string

    roles
        id - integer
        name - string

    role_user
        user_id - integer
        role_id - integer

#### Структура модели

Для отношения «многие ко многим», нужно написать метод, возвращающий результат метода `belongsToMany`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Роли, принадлежащие пользователю.
         */
        public function roles()
        {
            return $this->belongsToMany('App\Role');
        }
    }

Теперь мы можем получить роли пользователя через динамическое свойство `roles`:

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        //
    }

Как и для других типов отношений, вы можете вызвать метод `roles`, продолжив конструировать запрос для отношения:

    $roles = App\User::find(1)->roles()->orderBy('name')->get();

Как уже упоминалось ранее, чтобы определить имя для таблицы присоединения отношений, Eloquent соединит два названия взаимосвязанных моделей в алфавитном порядке. Тем не менее, вы можете переопределить имя, передав второй параметр методу `belongsToMany`:

    return $this->belongsToMany('App\Role', 'role_user');

В дополнение к заданию имени соединительной таблицы, вы можете также задать имена столбцов ключей в таблице, передав дополнительные параметры методу `belongsToMany`. Третий аргумент — это имя внешнего ключа модели, на которой вы определяете отношения, в то время как четвертый аргумент — это внешний ключ модели, с которой вы собираетесь связаться:

    return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');

#### Определение обратного отношения

Чтобы определить обратное отношение «многие-ко-многим», просто поместите другой вызов `belongsToMany` на вашу модель. Чтобы продолжить пример с ролями пользователя, давайте определим метод `users`  для модели `Role`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * Пользователи, принадлежащие этой роли.
         */
        public function users()
        {
            return $this->belongsToMany('App\User');
        }
    }

Как вы можете видеть, соотношение определяется точно так же, как и его для `User`, за исключением ссылки `App\User`. Так как мы повторно используем метод `belongsToMany`, все обычные таблицы и параметры настройки ключей доступны при определении обратного отношения многих-ко-многим.

#### Получение промежуточных столбцов таблицы

Как вы помните, работа с отношением «многие-ко-многим» требует наличия промежуточной таблицы. Eloquent предоставляет некоторые очень полезные способы взаимодействия с такой таблицей. Например, давайте предположим, что наш объект `User` имеет много связанных с ним объектов `Role`. После получения доступа к этому отношению мы можем получить доступ к промежуточной таблице с помощью атрибута `pivot` моделей:

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }

Обратите внимание на то, что каждой полученной модели `Role` автоматически присваивается атрибут `pivot`. Этот атрибут содержит модель, представляющую промежуточную таблицу, и может быть использован, как и любая другая модель Eloquent.

По умолчанию, только ключи модели будут представлять `pivot`-объект. Если ваша сводная таблица содержит дополнительные атрибуты, вам необходимо указать их при определении отношения:

    return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');

Если вы хотите, чтобы ваша сводная таблица автоматически поддерживала временные метки `created_at` и `updated_at`, используйте метод `withTimestamps` при определении отношений:

    return $this->belongsToMany('App\Role')->withTimestamps();

#### Настройка имёни сводной таблицы

Вы можете переименовать `pivot` в любое удобное вам имя. Например, если у вас пользователи могут подписываться на подкасты, то будет нагляднее, если в коде сводная таблица будет фигурировать под именем `subscription`. Задать произвольное имя можно при помощи метода `as`:

    return $this->belongsToMany('App\Podcast')
                    ->as('subscription')
                    ->withTimestamps();

Теперь вы можете обращаться к сводной таблице по имени `subscription`:

    $users = User::with('podcasts')->get();

    foreach ($users->flatMap->podcasts as $podcast) {
        echo $podcast->subscription->created_at;
    }


#### Фильтрация отношений через столбцы промежуточной таблицы

Вы также можете отфильтровать результаты, возвращённые методом `belongsToMany`, с помощью методов `wherePivot`, `wherePivotIn` и `wherePivotNotIn` при определении отношения:

    return $this->belongsToMany('App\Role')->wherePivot('approved', 1);

    return $this->belongsToMany('App\Role')->wherePivotIn('priority', [1, 2]);

    return $this->belongsToMany('App\Role')->wherePivotNotIn('priority', [1, 2]);

<a name="defining-custom-intermediate-table-models"></a>
### Определение пользовательских моделей промежуточных таблиц

Если вы бы хотели определить пользовательскую модель для представления промежуточной таблицы (pivot) вашего отношения, то можно вызвать метод `using` при определении отношения. Все пользовательские модели, используемые для представления промежуточных таблиц отношений, должны наследовать класс `Illuminate\Database\Eloquent\Relations\Pivot` или `Illuminate\Database\Eloquent\Relations\MorphPivot` , если это полиморфное отношение многие-ко-многим. Например, мы можем определить `Role`, в которой используется пользовательская сводная модель `UserRole`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * Пользователи, принадлежащие этой роли.
         */
        public function users()
        {
            return $this->belongsToMany('App\User')->using('App\RoleUser');
        }
    }

При определении модели `RoleUser`, мы будем наследовать класс `Pivot`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Relations\Pivot;

    class RoleUser extends Pivot
    {
        //
    }

Вы можете комбинировать `using` и `withPivot` для получения столбцов из промежуточной таблицы. Например, так можно получить `created_by` и `updated_by` из `RoleUser`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\User')
                            ->using('App\RoleUser')
                            ->withPivot([
                                'created_by',
                                'updated_by',
                            ]);
        }
    }

> **Note:** Pivot-модели не могут использовать трейт `SoftDeletes`.

#### Пользовательские промежуточные модели и автоинкремент

Если вы определили отношение многие-ко-многим, которое использует пользовательскую промежуточную модель, и эта модель имеет авто-инкрементирующийся основной ключ, вы должны убедиться, что ваш пользовательский класс pivot-модели содержит свойство `incrementing`, которое установлено в `true`.

    /**
     * Indicates if the IDs are auto-incrementing.
     *
     * @var bool
     */
    public $incrementing = true;

<a name="has-one-through"></a>
### Один к одному через

Отношения "Один к одному через" связывают модели через одну промежуточную связь.

Например, если у каждого поставщика есть один пользователь, и каждый пользователь связан с одной записью истории пользователя, то модель поставщика может получить доступ к истории пользователя через пользователя. Структура БД:

    users
        id - integer
        supplier_id - integer

    suppliers
        id - integer

    history
        id - integer
        user_id - integer

Так как таблица `history` не содержит `supplier_id`, для получения истории пользователя из модели поставщика нужно использовать отношение `hasOneThrough` через таблицу `users`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Supplier extends Model
    {
        /**
         * Get the user's history.
         */
        public function userHistory()
        {
            return $this->hasOneThrough('App\History', 'App\User');
        }
    }

Первый аргумент, переданный в `hasOneThrough` - имя модели, которую мы получаем, второй - имя промежуточной модели, через которую мы узнаём id первой модели.

Вы также можете задать явно названия ключей, при помощи которых осуществляются связи:

    class Supplier extends Model
    {
        /**
         * Get the user's history.
         */
        public function userHistory()
        {
            return $this->hasOneThrough(
                'App\History',
                'App\User',
                'supplier_id', // Foreign key on users table...
                'user_id', // Foreign key on history table...
                'id', // Local key on suppliers table...
                'id' // Local key on users table...
            );
        }
    }


<a name="has-many-through"></a>
### Ко многим через

Связь «ко многим через» обеспечивает удобный короткий путь для доступа к удалённым отношениям через промежуточные. Например, модель `Country` может иметь много моделей `Post` через промежуточную модель `User`. В данном примере вы можете просто собрать все публикации для заданной страны. Таблицы для этих отношений будут выглядеть так:

    countries
        id - integer
        name - string

    users
        id - integer
        country_id - integer
        name - string

    posts
        id - integer
        user_id - integer
        title - string

Несмотря на то, что таблица `posts` не содержит столбца `country_id`, отношение `hasManyThrough` предоставляет доступ к публикациям страны через `$country->posts`. Для выполнения этого запроса Eloquent ищет `country_id` в промежуточной таблице `users`. После нахождения совпадающих ID пользователей они используются в запросе к таблице `posts`.

Теперь, когда мы рассмотрели структуру таблицы для отношений, давайте определим отношения для модели `Country`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Country extends Model
    {
        /**
         * Получить все публикации для страны.
         */
        public function posts()
        {
            return $this->hasManyThrough('App\Post', 'App\User');
        }
    }

Первый параметр, переданный в метод `hasManyThrough`, является именем конечной модели, которую мы получаем, а второй параметр — это имя промежуточной модели.

Вы также можете задать явно названия ключей, при помощи которых осуществляются связи:

    class Country extends Model
    {
        public function posts()
        {
            return $this->hasManyThrough(
                'App\Post',
                'App\User',
                'country_id', // Foreign key on users table...
                'user_id', // Foreign key on posts table...
                'id', // Local key on countries table...
                'id' // Local key on users table...
            );
        }
    }

<a name="polymorphic-relationships"></a>
## Полиформные отношения

Полиморфные отношения позволяют модели быть связанной с более чем одной моделью, используя одну и ту же ассоциацию.

<a name="one-to-one-polymorphic-relations"></a>
### Один к одному (полиморфное отношение)

#### Структура таблиц

Полиморфные отношения позволяют модели быть связанной с более чем одной моделью. Например, у нас используются изображения в качестве шапки для постов и профайлов пользователей. Чтобы не делать две отдельные таблицы images vs можем сделать одну с полиморфной связью к постам и пользователям. Структура таблиц будет такой:

    posts
        id - integer
        name - string

    users
        id - integer
        name - string

    images
        id - integer
        url - string
        imageable_id - integer
        imageable_type - string

Обратите внимание на столбцы `imageable_id` и `imageable_type` в таблице `images`. `imageable_id` содержит id поста или пользователя, а `imageable_type` содержит обозначение, какой именно это id - поста или пользователя. 

Согласно стандартам фреймворка отношение в данном случае будет называться `imageable`, чтобы эти названия столбцов автоматически рассматривались в качестве связующих.

#### Структура модели

Модели будут выглядеть так:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Image extends Model
    {
        /**
         * Get the owning imageable model.
         */
        public function imageable()
        {
            return $this->morphTo();
        }
    }

    class Post extends Model
    {
        /**
         * Get the post's image.
         */
        public function image()
        {
            return $this->morphOne('App\Image', 'imageable');
        }
    }

    class User extends Model
    {
        /**
         * Get the user's image.
         */
        public function image()
        {
            return $this->morphOne('App\Image', 'imageable');
        }
    }

#### Получение отношения

Получить изображение, привязанное к посту можно следующим обычным способом:

    $post = App\Post::find(1);

    $image = $post->image;


Также вы можете использовать имя полиморфного отношения, заданного в модели:

    $image = App\Image::find(1);

    $imageable = $image->imageable;

Это отношение вернёт экземпляр модели `Post` или `User` - в зависимости от того, к цему эта картинка привязана.

<a name="one-to-many-polymorphic-relations"></a>
### Один ко многим (полиморфное отношение)

#### Структура таблиц

Полиморфное отношение "один ко многим" аналогично простому отношению "один ко многим". Однако целевая модель может принадлежать к нескольким типам моделей по одной ассоциации. Например, представьте, что пользователи вашего приложения могут комментировать и посты и видео. Используя полиморфные связи, вы можете использовать одну таблицу `comments` для обоих этих сценариев. Структура таблиц будет следующая:

    posts
        id - integer
        title - string
        body - text

    videos
        id - integer
        title - string
        url - string

    comments
        id - integer
        body - text
        commentable_id - integer
        commentable_type - string

Столбец `commentable_id` содержит id поста или видео, а `commentable_type` — имя класса модели, т.е. поста или видео. 

#### Структура модели

Теперь давайте рассмотрим, какие определения для модели нам нужны, чтобы построить её отношения:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * Get the owning commentable model.
         */
        public function commentable()
        {
            return $this->morphTo();
        }
    }

    class Post extends Model
    {
        /**
         * Получить все комментарии публикации.
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

    class Video extends Model
    {
        /**
         * Получить все комментарии видео.
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

#### Получение отношений

После определения моделей и таблиц вы можете получить доступ к отношениям через модели. Например, чтобы получить все комментарии статьи, используйте динамическое свойство `comments`:

    $post = App\Post::find(1);

    foreach ($post->comments as $comment) {
        //
    }

Как и в случае Один-к-одному, вы можете также получить владельца полиморфного отношения от полиморфной модели получив доступ к имени метода, который вызывает `morphTo`. В нашем случае это метод `commentable` для модели `Comment`. Так мы получим доступ к этому методу как к динамическому свойству:

    $comment = App\Comment::find(1);

    $commentable = $comment->commentable;

Отношение `commentable` модели `Comment` вернёт либо объект `Post`, либо объект `Video`, в зависимости от того, к чему оставлен данный комментарий.

<a name="many-to-many-polymorphic-relations"></a>
### Многие ко многим (полиморфное отношение)

#### Структура таблиц

Вы можете также задать полиморфные связи многие ко многим. Например, модели блогов `Post` и `Video` могут разделять полиморфную связь с моделью `Tag`. Используя полиморфное отношение "многие-ко-многим", вы имеете единственный список уникальных тегов, которые совместно используются в постах в блоге и видео. Структура таблиц:

    posts
        id - integer
        name - string

    videos
        id - integer
        name - string

    tags
        id - integer
        name - string

    taggables
        tag_id - integer
        taggable_id - integer
        taggable_type - string

#### Структура модели

Обе модели `Post` и `Video` будут иметь связь `morphToMany` в базовом классе Eloquent через метод `tags`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * Получить все теги публикации.
         */
        public function tags()
        {
            return $this->morphToMany('App\Tag', 'taggable');
        }
    }

#### Определение обратного отношения

Теперь для модели `Tag` вы можете определить метод для каждой из моделей отношения. Для нашего примера мы определим метод `posts` и метод `videos`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Tag extends Model
    {
        /**
         * Получить все публикации, связанные с тегом.
         */
        public function posts()
        {
            return $this->morphedByMany('App\Post', 'taggable');
        }

        /**
         * Получить все видео, которым присвоен этот тег.
         */
        public function videos()
        {
            return $this->morphedByMany('App\Video', 'taggable');
        }
    }

#### Получение отношения

Как только ваша таблица и модели определены, вы можете получить доступ к отношениям через свои модели. Например, чтобы получить доступ ко всем тегам для сообщения, вы можете просто использовать динамическое свойство `tags`:

    $post = App\Post::find(1);

    foreach ($post->tags as $tag) {
        //
    }

Вы также можете получить владельца полиморфного отношения от полиморфной модели, получив доступ к имени метода, который выполняет вызов `morphedByMany`. В нашем случае, это метод `posts` или `videos` для модели `Tag`. Таким образом вы получите доступ к этим методам как к динамическим свойствам:

    $tag = App\Tag::find(1);

    foreach ($tag->videos as $video) {
        //
    }

#### Пользовательские полиморфные типы

По умолчанию Laravel будет использовать полностью определённое имя класса для хранения типа связанной модели. Например, учитывая пример выше, где `Comment` может принадлежать `Post` или `Video`, значение по умолчанию для `commentable_type` было бы или `App\Post`, или `App\Video` соответственно. Однако вы можете захотеть отделить свою базу данных от внутренней структуры вашего приложения. В этом случае вы можете определить отношение "morph map", чтобы дать команду Eloquent использовать заданное имя для каждой модели вместо имени класса:

    use Illuminate\Database\Eloquent\Relations\Relation;

    Relation::morphMap([
        'posts' => 'App\Post',
        'videos' => 'App\Video',
    ]);

Вы можете зарегистрировать `morphMap` в функции `boot` в своём `AppServiceProvider` или создать отдельный сервис-провайдер.

<a name="querying-relations"></a>
## Запросы к отношениям

Вызывая отношения в виде методов, а не свойств (`posts()` , а не `posts`), мы не делаем запросов в БД, а получаем объект [конструктора запросов](/docs/{{version}}/queries), к которому можем применять дополнительные методы для уточнения запроса.

Допустим, у вас модель `User` имеет множество `Post`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Получить все публикации пользователя.
         */
        public function posts()
        {
            return $this->hasMany('App\Post');
        }
    }

Вы можете запросить отношение `posts`, добавить дополнительные условия к запросу (выбрать только посты с `active` равным `1`) и получить данные из БД:

    $user = App\User::find(1);

    $user->posts()->where('active', 1)->get();

Вы можете использовать на отношениях любой из методов [конструктора запросов](/docs/{{version}}/queries).

#### Применение ИЛИ в отношениях

Отдельно нужно сказать про `orWhere`. Будьте осторожны с ним и помните, что условия группируются по ИЛИ на том же уровне, на котором вызываются отношения. 

    $user->posts()
            ->where('active', 1)
            ->orWhere('votes', '>=', 100)
            ->get();

Этот запрос даст следующий SQL, что может быть не тем, что вы ожидаете получить:

    // select * from posts
    // where user_id = ? and active = 1 or votes >= 100

В большинстве случаев для использования ИЛИ вместе с И вас следует объединять запросы в [группы](/docs/{{version}}/queries#parameter-grouping)

    use Illuminate\Database\Eloquent\Builder;

    $user->posts()
            ->where(function (Builder $query) {
                return $query->where('active', 1)
                             ->orWhere('votes', '>=', 100);
            })
            ->get();

Данная конструкция интерпретируется в следующий SQL-запрос:

    // select * from posts
    // where user_id = ? and (active = 1 or votes >= 100)

<a name="relationship-methods-vs-dynamic-properties"></a>
### Отношения: методы или свойства

Если вы обращаетесь к отношению как к методу (`->posts()`) , то вы получаете объект [конструктора запросов](/docs/{{version}}/queries), к которому можем применять дополнительные методы для уточнения запроса.

Если вы обращаетесь к отношению как к динамическому свойству (`->posts`), то вы делаете (если это требуется) запрос к БД и получаете собствено данные, модель или коллекцию моделей.

    $user = App\User::find(1);

    foreach ($user->posts as $post) {
        //
    }

Динамические свойства поддерживают "отложенную (ленивую) загрузку". Это означает, что данные загружаются из БД только в момент обращения к отношениям. Иногда применяют [жадную загрузку](#eager-loading), чтобы предварительно загрузить отношения, для которых известно, что они точно понадобятся. Жадная загрузка обеспечивает значительное сокращение SQL-запросов, если вы берёте не одну модель, а несколько.

<a name="querying-relationship-existence"></a>
### Проверка существования связей при выборке

При чтении отношений модели вам может быть нужно ограничить результаты в зависимости от существования отношения. Например, вы хотите получить все публикации в блоге, имеющие хотя бы один комментарий. Для этого можно использовать метод `has`:

    // Получить все публикации в блоге, имеющие хотя бы один комментарий...
    $posts = App\Post::has('comments')->get();

Вы также можете указать оператора и число, чтобы еще больше настроить запрос:

    // Получить все публикации в блоге, имеющие три или более комментариев...
    $posts = App\Post::has('comments', '>=', 3)->get();

Можно конструировать вложенные операторы `has` с записи через точку. Например, вы можете получить все публикации, которые имеют хотя бы один комментарий и голос:

    // Получить все публикации, которые имеют хотя бы один комментарий и голос...
    $posts = App\Post::has('comments.votes')->get();

Если вам нужно ещё больше возможностей, вы можете использовать методы `whereHas` и `orWhereHas`, чтобы поместить условия "where" в ваши запросы `has`. Эти методы позволяют вам добавить свои ограничения в отношения, такие как проверку содержимого комментария:

    use Illuminate\Database\Eloquent\Builder;

    // Получить посты, где хотя бы один комментарий начинается со слова foo
    $posts = App\Post::whereHas('comments', function (Builder $query) {
        $query->where('content', 'like', 'foo%');
    })->get();

    // Получить посты, где минимум 10 комментариев начинаются со слова foo
    $posts = App\Post::whereHas('comments', function (Builder $query) {
        $query->where('content', 'like', 'foo%');
    }, '>=', 10)->get();


<a name="querying-relationship-absence"></a>
### Выборка по отсутствию отношения

При получении записей модели бывает необходимо ограничить результаты выборки на основе отсутствия отношения. Например, если вы хотите получить все статьи, у которых **нет** комментариев. Для этого передайте имя отношения в метод `doesntHave`:

    $posts = App\Post::doesntHave('comments')->get();

Для ещё большего уточнения используйте методы `whereDoesntHave` и `orWhereDoesntHave`, чтобы добавить условия "where" в ваши запросы `doesntHave`. Этот метод позволяет добавить дополнительные ограничения к отношению, например, проверку содержимого комментария:

    use Illuminate\Database\Eloquent\Builder;

    $posts = App\Post::whereDoesntHave('comments', function (Builder $query) {
        $query->where('content', 'like', 'foo%');
    })->get();

Для запроса к вложенным отношениям можно использовать dot-нотацию. Например, запросим все посты с коментариями незабаненных авторов:

    use Illuminate\Database\Eloquent\Builder;

    $posts = App\Post::whereDoesntHave('comments.author', function (Builder $query) {
        $query->where('banned', 0);
    })->get();

<a name="querying-polymorphic-relationships"></a>
### Выборка по полиморфным отношениям

Чтобы проверить, существуют ли результаты по полиморфному отношению, используйте `whereHasMorph`:

    use Illuminate\Database\Eloquent\Builder;

    // Получить комментарии постов или видео, которые наинаются с foo 
    $comments = App\Comment::whereHasMorph(
        'commentable',
        ['App\Post', 'App\Video'],
        function (Builder $query) {
            $query->where('title', 'like', 'foo%');
        }
    )->get();

    // Получить комментарии постов, начинающиеся с foo
    $comments = App\Comment::whereDoesntHaveMorph(
        'commentable',
        'App\Post',
        function (Builder $query) {
            $query->where('title', 'like', 'foo%');
        }
    )->get();

Вы можете использовать параметр `$type` для добавления различных ограничений в зависимости от соответствующей модели:

    use Illuminate\Database\Eloquent\Builder;

    $comments = App\Comment::whereHasMorph(
        'commentable',
        ['App\Post', 'App\Video'],
        function (Builder $query, $type) {
            $query->where('title', 'like', 'foo%');

            if ($type === 'App\Post') {
                $query->orWhere('content', 'like', 'foo%');
            }
        }
    )->get();

Вместо того, чтобы передавать массив возможных полиморфных моделей, Вы можете использовать `*` и позволить Ларавелу получить все возможные полиморфные типы из базы данных. Для выполнения этой операции Ларавел выполнит дополнительный запрос:

    use Illuminate\Database\Eloquent\Builder;

    $comments = App\Comment::whereHasMorph('commentable', '*', function (Builder $query) {
        $query->where('title', 'like', 'foo%');
    })->get();

<a name="counting-related-models"></a>
### Подсчёт моделей в отношении

Если вы хотите посчитать число результатов отношения, не загружая их целиком, используйте метод `withCount`, который добавит свойство `{relation}_count` в вашу результирующую модель. Например:

    $posts = App\Post::withCount('comments')->get();

    foreach ($posts as $post) {
        echo $post->comments_count;
    }

Вы можете использовать массив для подсчёта сразу нескольких отношений, а также добавить условия к запросам:

    use Illuminate\Database\Eloquent\Builder;

    $posts = App\Post::withCount(['votes', 'comments' => function (Builder $query) {
        $query->where('content', 'like', 'foo%');
    }])->get();

    echo $posts[0]->votes_count;
    echo $posts[0]->comments_count;

Вы также можете создавать псевдоним для результата подсчета отношений, разрешая использовать несколько подсчетов на одном и том же отношении:

    $posts = Post::withCount([
        'comments',
        'comments AS pending_comments' => function ($query) {
            $query->where('approved', false);
        }
    ])->get();

    echo $posts[0]->comments_count;
    echo $posts[0]->pending_comments_count;

Если вы используете в одном запросе `withCount` и `select`, убедитесь, что `withCount` идёт после `select`:

    $posts = App\Post::select(['title', 'body'])->withCount('comments')->get();

    echo $posts[0]->title;
    echo $posts[0]->body;
    echo $posts[0]->comments_count;

Кроме того, используя метод `loadCount`, вы можете вычислить такой счетчик сразу же после получения родительской модели:

    $book = App\Book::first();

    $book->loadCount('genres');

Если вам нужно установить дополнительные условия при подсчёте с помощью `loadCount`, вы можете передать массив с ключами в виде отношений и значениями в виде анонимных функций:

    $book->loadCount(['reviews' => function ($query) {
        $query->where('rating', 5);
    }])


<a name="eager-loading"></a>
## Жадная загрузка

При доступе к Eloquent отношениям как к свойствам отношения "лениво" (lazy) загружаются. Это означает, что данные отношения фактически не загружены, пока вы не обратитесь к свойству - тогда они берутся из БД. Однако Eloquent может и "жадно" (eager) загружать отношения в тот момент, когда вы запрашиваете выборку по родительским моделям. Жадная загрузка решает проблему N+1. 

Для иллюстрации проблемы, рассмотрим модель `Book`, которая связана с `Author`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Book extends Model
    {
        /**
         * Получить автора, который написал книгу.
         */
        public function author()
        {
            return $this->belongsTo('App\Author');
        }
    }

Теперь давайте получим все книги и их авторов:

    $books = App\Book::all();

    foreach ($books as $book) {
        echo $book->author->name;
    }

Этот цикл выполнит 1 запрос, чтобы получить все книги, затем выполнится по одному запросу для каждой книги, чтобы получить автора. Так, если бы у нас было 25 книг, этот код выполнил бы 26 запросов: 1 для исходной книги и 25 дополнительных запросов, чтобы получить автора каждой книги.

К счастью, мы можем использовать жадную загрузку, чтобы уменьшить эту работу всего до 2 запросов. При запросах вы можете определить, какие отношения должны быть жадно загружены с использованием метода `with`:

    $books = App\Book::with('author')->get();

    foreach ($books as $book) {
        echo $book->author->name;
    }

Для данной операции будут выполнены только два запроса:

    select * from books

    select * from authors where id in (1, 2, 3, 4, 5, ...)

#### Жадная загрузка нескольких отношений

В `with` можно указать массив с именами отношений для жадной загрузки:

    $books = App\Book::with(['author', 'publisher'])->get();

#### Вложенная жадная загрузка

Для вложенных отношений жадной загрузки вы можете использовать dot-нотацию, с использованием точки в качестве разделителя родитель - потомок. Например, давайте жадно загружать всех авторов книг и все личные контакты автора в одном Eloquent операторе:

    $books = App\Book::with('author.contacts')->get();

#### Вложенная жадная загрузка для полиморфных отношений

Если Вы хотите загрузить отношения `morphTo`, а также вложенные отношения на различных объектах, которые могут быть возвращены этими отношениями, вы можете использовать метод `with` в сочетании с методом `morphTo` отношений `morphWith`. Чтобы проиллюстрировать этот метод, рассмотрим следующую модель:

    <?php

    use Illuminate\Database\Eloquent\Model;

    class ActivityFeed extends Model
    {
        /**
         * Get the parent of the activity feed record.
         */
        public function parentable()
        {
            return $this->morphTo();
        }
    }

В данном примере, предположим, что модели `Event`, `Photo` и `Post` могут создавать модели `ActivityFeed`. Дополнительно, предположим, что модели `Event` принадлежат к модели `Calendar`, `Photo` модели ассоциируются с моделями `Tag`, а `Post` модели принадлежат к `Author` модели.

Используя эти определения моделей и связи, мы можем получить экземпляры моделей `ActivityFeed` и загрузить все `parentable` модели и их соответствующие вложенные связи:

    use Illuminate\Database\Eloquent\Relations\MorphTo;

    $activities = ActivityFeed::query()
        ->with(['parentable' => function (MorphTo $morphTo) {
            $morphTo->morphWith([
                Event::class => ['calendar'],
                Photo::class => ['tags'],
                Post::class => ['author'],
            ]);
        }])->get();

#### Жадная загрузка указанных столбцов в модели отношения

Вам не всегда может понадобиться каждый столбец из отношений, которые вы получаете. По этой причине, Eloquent дает возможность указать, какие столбцы из отношений вы хотите получить:

    $books = App\Book::with('author:id,name')->get();

> {note} При использовании этой функции, вы всегда должны включать `id` и внешние ключи (например, `user_id`) в список столбцов, которые вы хотите получить.

#### Жадная загрузка по умолчанию

Для того, чтобы модель всегда подгружала указанные отношения, добавьте их в protected свойство `with`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Book extends Model
    {
        /**
         * The relationships that should always be loaded.
         *
         * @var array
         */
        protected $with = ['author'];

        /**
         * Get the author that wrote the book.
         */
        public function author()
        {
            return $this->belongsTo('App\Author');
        }
    }

В случае, когда вам понадобится отключить дефолтную жадную загрузку, используйте метод `without`:

    $books = App\Book::without('author')->get();

<a name="constraining-eager-loads"></a>
### Условия для жадных загрузок

Иногда вам может понадобиться указать дополнительные условия для запроса на загрузку. Например:

    $users = App\User::with(['posts' => function ($query) {
        $query->where('title', 'like', '%first%');
    }])->get();

В данном примере Eloquent будет жадно загружать только те статьи, в которых столбец `title` содержит слово `first`. 

Также вы можете вызвать другие методы [конструктора запросов](/docs/{{version}}/queries) для дополнительной настройки жадной загрузки:

    $users = App\User::with(['posts' => function ($query) {
        $query->orderBy('created_at', 'desc');
    }])->get();

> {note} Методы `limit` и `take` в данном случае не могут быть использованы.    

<a name="lazy-eager-loading"></a>
### Отложенная жадная загрузка

Иногда вам может понадобиться загрузить отношения после того, как родительская модель уже получена. Сделать это можно при помощи `load`. Например, если вам надо позже решить, загружать ли все отношения:

    $books = App\Book::all();

    if ($someCondition) {
        $books->load('author', 'publisher');
    }

Если вам нужно установить дополнительные условия, вместо имени отношения вы можете передать массив с функцией, где укажите нужные условия:

    $author->load(['books' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }]);

Для того, чтобы загрузить отношения, если они ещё не загружены, используйте `loadMissing`:

    public function format(Book $book)
    {
        $book->loadMissing('author');

        return [
            'name' => $book->name,
            'author' => $book->author->name,
        ];
    }

#### Отложенная жадная загрузка для полиморфных отношений

Если вы хотите загрузить отношения `morphTo`, а также вложенные отношения по различным сущностям, которые могут быть возвращены этими отношениями, вы можете использовать метод `loadMorph`.

Этот метод принимает имя отношения `morphTo` в качестве первого аргумента, а массив моделей/пар отношений в качестве второго аргумента. Для иллюстрации этого метода рассмотрим следующую модель:

    <?php

    use Illuminate\Database\Eloquent\Model;

    class ActivityFeed extends Model
    {
        /**
         * Get the parent of the activity feed record.
         */
        public function parentable()
        {
            return $this->morphTo();
        }
    }

В данном примере, предположим, что модели `Event`, `Photo` и `Post` могут создавать модели `ActivityFeed`. Дополнительно, предположим, что модели `Event` принадлежат к модели `Calendar`, `Photo` модели ассоциируются с моделями `Tag`, а `Post` модели принадлежат к `Author` модели.

Используя эти определения моделей и связи, мы можем получить экземпляры моделей `ActivityFeed` и загрузить все `parentable` модели и их соответствующие вложенные отношения:

    $activities = ActivityFeed::with('parentable')
        ->get()
        ->loadMorph('parentable', [
            Event::class => ['calendar'],
            Photo::class => ['tags'],
            Post::class => ['author'],
        ]);

<a name="inserting-and-updating-related-models"></a>
## Вставка и изменение связанных моделей

<a name="the-save-method"></a>
### Метод Save

Если вам понадобится добавить новый `Comment` для модели `Post`, вместо того, чтобы вручную устанавливать атрибут `post_id` у `Comment`, вы можете добавить `Comment` непосредственно из метода `save` отношения:

    $comment = new App\Comment(['message' => 'A new comment.']);

    $post = App\Post::find(1);

    $post->comments()->save($comment);

Заметьте, что мы не обращались к отношению `comments` как к динамическому свойству. Вместо этого мы вызвали метод `comments` , чтобы получить экземпляр отношения. Метод `save` автоматически добавит надлежащее значение `post_id` в новую модель `Comment`.

Если вам нужно добавить сразу несколько моделей, используйте метод `saveMany`:

    $post = App\Post::find(1);

    $post->comments()->saveMany([
        new App\Comment(['message' => 'A new comment.']),
        new App\Comment(['message' => 'Another comment.']),
    ]);

<a name="the-push-method"></a>
#### Рекурсивное сохранение моделей и отношений

Если Вы хотите сохранить вашу модель и все связанные с ней отношения, вы можете использовать метод `push`:

    $post = App\Post::find(1);

    $post->comments[0]->message = 'Message';
    $post->comments[0]->author->name = 'Author Name';

    $post->push();


<a name="the-create-method"></a>
### Метод Create

В дополнение к методам `save` и `saveMany` вы можете также использовать метод `create`, который принимает массив атрибутов, создает модель и вставляет её в базу данных. Различие между `save` и `create` состоит в том, что `save` принимает экземпляр Eloquent модели, в то время как `create` принимает простой массив:

    $post = App\Post::find(1);

    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);

> {tip} Перед использованием метода `create` освежите в памяти документацию по [массовому присваиванию](/docs/{{version}}/eloquent#mass-assignment) атрибутов.

Вы можете использовать метод `createMany` для создания и добавления нескольких моделей:

    $post = App\Post::find(1);

    $post->comments()->createMany([
        [
            'message' => 'A new comment.',
        ],
        [
            'message' => 'Another new comment.',
        ],
    ]);

Также для создания и обновления моделей в отношениях можно использовать методы `findOrNew`, `firstOrNew`, `firstOrCreate` и `updateOrCreate` ([см. документацию по этим методам](https://laravel.com/docs/{{version}}/eloquent#other-creation-methods)).

<a name="updating-belongs-to-relationships"></a>
### Отношения "Принадлежит к"

При обновлении отношения `belongsTo` вы можете использовать метод `associate`. Этот метод установит внешний ключ на дочерней модели для осуществления этой связи. Например, здесь фактически произойдёт `$account->user_id = $user->id`

    $account = App\Account::find(10);

    $user->account()->associate($account);

    $user->save();

При удалении отношения `belongsTo` вы можете использовать метод `dissociate`. Этот метод сбросит внешний ключ в `null`:

    $user->account()->dissociate();

    $user->save();

<a name="default-models"></a>
#### Default Models

Отношения `belongsTo`, `hasOne`, `hasOneThrough` и `morphOne` позволяют определить модель по умолчанию, которая будет возвращена, если данное отношение возвратило `null`. Это поведение часто называют [Null Object pattern] (https://en.wikipedia.org/wiki/Null_Object_pattern). В следующем примере отношение `user` вернет пустую модель `App\User`, если к сообщению не прикреплено ни одного `user`:

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault();
    }

Чтобы присвоить дефолтной модели определённые аттрибуты, их можно передать функцию в `withDefault` - в виде массива или в виде функции.

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault([
            'name' => 'Guest Author',
        ]);
    }

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault(function ($user, $post) {
            $user->name = 'Guest Author';
        });
    }


<a name="updating-many-to-many-relationships"></a>
### Отношения многие-ко-многим

#### Присоединение / Отсоединение

Давайте предположим, что у пользователя может быть много ролей, и у роли может быть много пользователей. Чтобы присоединить роль к пользователю вставкой записи в промежуточную связующую таблицу, которая присоединяется к моделям, используйте метод `attach`:

    $user = App\User::find(1);

    $user->roles()->attach($roleId);

При присоединении отношения к модели вы можете также передать массив дополнительных данных, которые будут вставлены в промежуточную таблицу:

    $user->roles()->attach($roleId, ['expires' => $expires]);

Иногда бывает необходимо удалить роль у пользователя. Чтобы удалить элемент отношения многие-к-многим, используйте метод `detach`. Метод `detach` удалит соответствующую запись связи из промежуточной таблицы, однако обе модели останутся в базе данных:

    // Отсоединить одну роль от пользователя...
    $user->roles()->detach($roleId);

    // Отсоединить все роли от пользователя...
    $user->roles()->detach();

Для удобства `attach` и `detach` также принимают массивы id:

    $user = App\User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([
        1 => ['expires' => $expires],
        2 => ['expires' => $expires],
    ]);

#### Синхронизация ассоциаций

Вы можете также использовать метод `sync`, чтобы добавить ассоциации многие-ко-многим и одновременно удалить некоторые из существующих. Метод `sync` принимает массив ID, чтобы поместить его в промежуточную связующую таблицу. Все ID, которые не находятся в данном массиве, будут удалены из промежуточной таблицы. После того как эта работа завершена, только ID из данного массива будут существовать в промежуточной таблице:

    $user->roles()->sync([1, 2, 3]);

Также вы можете передать дополнительные значения промежуточной таблицы с массивом ID:

    $user->roles()->sync([1 => ['expires' => true], 2, 3]);

Если вы не хотите отделять существующие ID, используйте метод `syncWithoutDetaching`:

    $user->roles()->syncWithoutDetaching([1, 2, 3]);

#### Переключение ассоциаций

Отношение многие-ко-многим также предоставляет метод `toggle`, который "переключает" состояние присоединений с заданными ID. Если данный ID сейчас присоединён, то он будет отсоединён. И наоборот, если сейчас он отсоединён, то будет присоединён:

    $user->roles()->toggle([1, 2, 3]);

#### Сохранение дополнительных данных в связующей таблице

При работе с отношением многие-ко-многим метод `save` принимает вторым аргументом массив дополнительных атрибутов промежуточной таблицы:

    App\User::find(1)->roles()->save($role, ['expires' => $expires]);

#### Изменение записи в связующей таблице

Для изменения существующей строки в связующей таблице используйте метод `updateExistingPivot`. Этот метод принимает внешний ключ связующей записи и массив атрибутов для изменения:

    $user = App\User::find(1);

    $user->roles()->updateExistingPivot($roleId, $attributes);

<a name="touching-parent-timestamps"></a>
## Установка меток времени родителям

Когда модель имеет связь `belongsTo` или `belongsToMany` с другими моделями, например, `Comment`, которая принадлежит `Post`, иногда полезно обновить метку времени родителя, когда дочерняя модель обновлена. Для этого добавьте свойство `$touches`, содержащее имена отношений к дочерней модели:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * Все отношения, которым нужно обновить временные метки.
         *
         * @var array
         */
        protected $touches = ['post'];

        /**
         * Получить публикацию, к которой привязан комментарий.
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

Теперь, когда вы обновляете `Comment`, у владеющего `Post` тоже обновится столбец `updated_at`, позволяя, например, узнать, когда необходимо аннулировать кэш модели `Post`:

    $comment = App\Comment::find(1);

    $comment->text = 'Edit to this comment!';

    $comment->save();
