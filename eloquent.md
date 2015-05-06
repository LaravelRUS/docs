git a8ab80b8ba7a353e16508b1d1f5a567e2634e2ce

---

# Eloquent ORM

- [Введение](#introduction)
- [Использование ORM](#basic-usage)
- [Массовое заполнение](#mass-assignment)
- [Вставка, обновление, удаление](#insert-update-delete)
- [Псевдоудаление](#soft-deleting)
- [Дата и время](#timestamps)
- [Заготовки запросов (query scopes)](#query-scopes)
- [Глобальные заготовки запросов (global query scopes)](#query-scopes)
- [Отношения](#relationships)
- [Динамические свойства](#querying-relations)
- [Активная загрузка](#eager-loading)
- [Вставка связанных моделей](#inserting-related-models)
- [Обновление времени владельца](#touching-parent-timestamps)
- [Работа со связующими таблицами](#working-with-pivot-tables)
- [Коллекции](#collections)
- [Читатели и преобразователи](#accessors-and-mutators)
- [Преобразователи дат](#date-mutators)
- [Приведение атрибутов к определенному типу](#attribute-casting)
- [События моделей](#model-events)
- [Наблюдатели моделей](#model-observers)
- [Генерация урлов](#model-url-generation)
- [Преобразование в массивы и JSON](#converting-to-arrays-or-json)

<a name="introduction"></a>
## Введение

Система объектно-реляционного отображения ORM Eloquent - красивая и простая реализация ActiveRecord в Laravel для работы с базами данных.
Каждая таблица имеет соответствующий класс-модель, который используется для работы с этой таблицей. 

Прежде чем начать настройте ваше соединение с БД в файле `config/database.php`.

<a name="basic-usage"></a>
## Использование ORM

Для начала создадим модель Eloquent. Модели обычно располагаются в папке `app`, но вы можете поместить в любое место, в котором работает
автозагрузчик в соответствии с вашим файлом `composer.json`. Модели Eloquent должны расширять класс `Illuminate\Database\Eloquent\Model`.

#### Создание модели Eloquent

	class User extends Model {}

Так же модель Eloquent можно создать Artisan-командой `make:model`:

	php artisan make:model User

Заметьте, что мы не указали, какую таблицу Eloquent должен привязать к нашей модели. Если это имя не указано явно, то будет использовано
имя класса в нижнем регистре и во множественном числе. В нашем случае Eloquent предположит, что модель
`User` хранит свои данные в таблице `users`. Вы можете указать произвольную таблицу, определив свойство `table` в классе модели:

	class User extends Model {

		protected $table = 'my_users';

	}

> **Примечание:** Eloquent также предполагает, что каждая таблица имеет первичный ключ с именем `id`. Вы можете определить свойство `primaryKey`
для изменения этого имени. Аналогичным образом, вы можете определить свойство `connection` для задания имени подключения к БД, которое
должно использоваться при работе с данной моделью.

Как только модель определена у вас всё готово для того, чтобы можно было выбирать и создавать записи. Обратите внимание, что вам нужно
создать в этой таблице поля `updated_at` и `created_at`. Если вы не хотите, чтобы они были автоматически используемы, установите свойство
`timestamps` класса модели в `false`.

#### Получение всех моделей (записей)

	$users = User::all();

#### Получение записи по первичному ключу

	$user = User::find(1);

	var_dump($user->name);

> **Примечание:** Все методы, доступные в [конструкторе запросов](/docs/{{version}}/queries), также доступны при работе с моделями Eloquent.

#### Получение модели по первичному ключу с возбуждением исключения

Иногда вам нужно возбудить исключение, если определённая модель не была найдена, что позволит вам его отловить в обработчике `App::error`
и вывести страницу 404 («Не найдено»).

	$model = User::findOrFail(1);

	$model = User::where('votes', '>', 100)->firstOrFail();

Для регистрации обработчика ошибки подпишитесь на событие `ModelNotFoundException`:

	use Illuminate\Database\Eloquent\ModelNotFoundException;

	App::error(function(ModelNotFoundException $e)
	{
		return Response::make('Not Found', 404);
	});

#### Построение запросов в моделях Eloquent

	$users = User::where('votes', '>', 100)->take(10)->get();

	foreach ($users as $user)
	{
		var_dump($user->name);
	}

#### Аггрегатные функции в Eloquent

Конечно, вам также доступны аггрегатные функции.

	$count = User::where('votes', '>', 100)->count();

Если у вас не получается создать нужный запрос с помощью методов конструктора, то можно использовать метод `whereRaw`:

	$users = User::whereRaw('age > ? and votes = 100', [25])->get();

#### Обработка результата по частям

Если в вашей eloquent-выборке получилось очень много элементов, то обрабатывая их в цикле foreach вы можете израсходовать всю доступную
оперативную память. Чтобы этого не произошло, используйте метод `chunk` таким образом:

	User::chunk(200, function($users)
	{
		foreach ($users as $user)
		{
			//
		}
	});

Данный код вынимает данные порциями по 200 (первый аргумент) записей. Обработка записи производится в функции-замыкании, которая передается
вторым аргументом.

#### Указание имени соединения с БД

Иногда вам нужно указать, какое подключение должно быть использовано при выполнении запроса Eloquent - просто используйте метод `on`:

	$user = User::on('имя-соединения')->find(1);

Если вы используете [отдельные соединения на чтение и запись](/docs/{{version}}/database#read-write-connections), вы можете принудительно
прочитать данные по соединению, которое осуществляет запись:

	$user = User::onWriteConnection()->find(1);

<a name="mass-assignment"></a>
## Массовое заполнение

При создании новой модели вы передаёте её конструктору массив атрибутов. Эти атрибуты затем присваиваются модели через массовое заполнение.
Это удобно, но в то же время представляет **серьёзную** проблему с безопасностью, когда вы передаёте ввод от клиента в модель
без проверок - в этом случае пользователь может изменить **любое** и **каждое** поле вашей модели. По этой причине по умолчанию Eloquent
защищает вас от массового заполнения.

Для начала определите в классе модели свойство  `fillable` или `guarded`.

Свойство `fillable` указывает, какие поля должны быть доступны при массовом заполнении. Их можно указать на уровне класса или объекта.

#### Указание доступных к заполнению атрибутов

	class User extends Model {

		protected $fillable = ['first_name', 'last_name', 'email'];

	}

В этом примере только три перечисленных поля будут доступны к массовому заполнению.

Противоположность `fillable` - свойство `guarded`, которое содержит список запрещённых к заполнению полей.

#### Указание охраняемых (guarded) атрибутов модели

	class User extends Model {

		protected $guarded = ['id', 'password'];

	}

> **Примечание:** Даже если вы используете `guarded`, по возможности избегайте массового присваивания `Input::get()` модели.

#### Защита всех атрибутов от массового заполнения

В примере выше атрибуты  `id` и `password` **не могут** быть присвоены через массовое заполнение. Все остальные атрибуты - могут.
Вы также можете запретить **все** атрибуты для заполнения специальным значением.

	protected $guarded = ['*'];

<a name="insert-update-delete"></a>
## Вставка, обновление, удаление

Для создания новой записи в БД просто создайте экземпляр модели и вызовите метод `save`.

#### Сохранение новой модели

	$user = new User;

	$user->name = 'Джон';

	$user->save();

> **Примечание:** обычно ваши модели Eloquent содержат автоматически увеличивающиеся (autoincrementing) числовые ключи.
Однако если вы хотите использовать собственные ключи, установите свойство `incrementing` класса модели в значение `false`.

Вы также можете использовать метод `create` для создания и сохранения модели одной строкой. Метод вернёт добавленную модель.
Однако перед этим вам нужно определить либо свойство `fillable`, либо `guarded` в классе модели, так как изначально все модели Eloquent
защищены от массового заполнения.

#### Установка охранных свойств модели

	class User extends Model {

		protected $guarded = ['id', 'account_id'];

	}

#### Создание модели

	$user = User::create(['name' => 'Джон']);

	// Создать нового пользователя в базе данных
	$user = User::create(['name' => 'John']);

	// Получить пользователя с данным именем, а если он отсутствует - создать его в базе данных
	$user = User::firstOrCreate(['name' => 'John']);

	// Получить пользователя с данным именем, а если он отсутствует - создать его объект (для сохранения в БД нужно будет сделать `$user->save()`)
	$user = User::firstOrNew(['name' => 'John']);

#### Обновление полученной модели

Для обновления модели вам нужно получить её, изменить атрибут и вызвать метод `save`:

	$user = User::find(1);

	$user->email = 'john@foo.com';

	$user->save();

#### Сохранение модели и её отношений

Иногда вам может быть нужно сохранить не только модель, но и все её отношения. Для этого используйте метод `push`.

	$user->push();

Вы также можете выполнять обновления в виде запросов к набору моделей:

	$affectedRows = User::where('votes', '>', 100)->update(['status' => 2]);

> **Примечание:** В случае подобного апдейта через конструктор запросов, обработчики событий модели запускаться не будут.

## Удаление существующей модели

Для удаления модели вызовите метод `delete` на её объекте.

	$user = User::find(1);

	$user->delete();

#### Удаление модели по ключу

	User::destroy(1);

	User::destroy([1, 2, 3]);

	User::destroy(1, 2, 3);

Конечно, вы также можете выполнять удаление на наборе моделей:
	
	$affectedRows = User::where('votes', '>', 100)->delete();

#### Обновление времени изменения модели

Если вам нужно просто обновить время изменения записи - используйте метод `touch`:

	$user->touch();

<a name="soft-deleting"></a>
## Псевдоудаление (soft deleting)

Когда вы удаляете модель с включенным `softDelete`, она на самом деле остаётся в базе данных, однако в её поле `deleted_at` записывается
текущее время. Для включения псевдоудалений добавьте в модель трейт `SoftDeletes`:

use Illuminate\Database\Eloquent\SoftDeletes;

	class User extends Model {

		use SoftDeletes;

		protected $dates = ['deleted_at'];

	}

Для добавления поля `deleted_at` к таблице можно в миграции использовать метод `softDeletes`:

	$table->softDeletes();

Теперь когда вы вызовите метод `delete`, поле `deleted_at` будет установлено в значение текущего времени. При запросе моделей, использующих
псевдоудаление, «удалённые» модели не будут включены в результат запроса. 

#### Включение удалённых моделей в результат выборки

Для отображения всех моделей, в том числе удалённых, используйте метод `withTrashed`.

	$users = User::withTrashed()->where('account_id', 1)->get();

Он также работает в отношениях модели:

	$user->posts()->withTrashed()->get();	

Если вы хотите получить **только** удалённые модели, вызовите метод `onlyTrashed`:

	$users = User::onlyTrashed()->where('account_id', 1)->get();

Для восстановления псевдоудалённой модели в активное состояние используется метод `restore`:

	$user->restore();

Вы также можете использовать его в запросе:

	User::withTrashed()->where('account_id', 1)->restore();

Метод `restore` можно использовать и в отношениях:

	$user->posts()->restore();

Если вы хотите полностью удалить модель из БД, используйте метод `forceDelete`:

	$user->forceDelete();

Он также работает с отношениями:

	$user->posts()->forceDelete();

Для того, чтобы узнать, удалена ли модель, можно использовать метод `trashed`:

	if ($user->trashed())
	{
		//
	}

<a name="timestamps"></a>
## Дата и время

По умолчанию Eloquent автоматически поддерживает поля `created_at` и `updated_at`, записывая в них, соответственно, дату и время (timestamp)
создания и обновления строки в базе данных. Просто добавьте эти timestamp-поля к таблице и Eloquent позаботится об остальном.
Если вы не хотите, чтобы он поддерживал их, добавьте свойство `timestamps` равное `false` к классу модели.

#### Отключение автоматических полей времени

	class User extends Model {

		protected $table = 'users';

		public $timestamps = false;

	}

Для настройки форматов времени перекройте метод `getDateFormat`:

#### Использование собственного формата времени

	class User extends Model {

		protected function getDateFormat()
		{
			return 'U';
		}

	}

<a name="query-scopes"></a>
## Заготовки запросов (query scopes)

Заготовки (скоупы) позволяют вам повторно использовать логику запросов в моделях. Для создания скоупа просто начните имя метода со `scope`:

#### Создание заготовки запроса

	class User extends Model {

		public function scopePopular($query)
		{
			return $query->where('votes', '>', 100);
		}
		
		public function scopeWomen($query)
		{
			return $query->whereGender('W');
		}

	}

#### Использование скоупа

	$users = User::popular()->women()->orderBy('created_at')->get();

#### Динамические скоупы

Иногда вам может потребоваться определить заготовку запроса, которая принимает параметры. Для этого просто добавьте эти параметры к методу скоупа:

	class User extends Model {

		public function scopeOfType($query, $type)
		{
			return $query->whereType($type);
		}

	}

А затем передайте их при вызове метода заготовки:

	$users = User::ofType('member')->get();

<a name="global-scopes"></a>
## Глобальные заготовки запросов

Иногда вам нужно определить скоуп, который должен быть доступен во всех моделях. Например, псевдоудаление (soft delete) в Laravel построено
на этой фиче. Глобальные заготовки запросов создаются путем комбинации PHP-трейтов и имплементации `Illuminate\Database\Eloquent\ScopeInterface`.

Для начала, определим трейт. Для примера, рассмотрим реализацию псевдоудаления (soft delete):

	trait SoftDeletes {

		/**
		 * Загрузка трейта soft delete.
		 *
		 * @return void
		 */
		public static function bootSoftDeletes()
		{
			static::addGlobalScope(new SoftDeletingScope);
		}

	}

Если в Eloquent-модель подключается трейт, то при загрузке модели вызывается метод трейта, который называется по принципу `bootNameOfTrait`
(т.е. в случае трейта `SoftDeletes` - `bootSoftDeletes() ` ). Этот метод - подходящее место для регистрации глобальной заготовоки запроса.
Этот скоуп должен быть реализацией `ScopeInterface` и иметь два метода - `apply` и `remove`.

Метод `apply` принимает объект конструктора запросов `Illuminate\Database\Eloquent\Builder` и `Model`, к которой надо применить скоуп и добавляет в него все необходимые запросы `where`.
Метод `remove` тоже принимает объект `Builder` и `Model` и отвечает за действия, обратные тем, которые мы сделали в `apply`. Взгляните на пример:

	/**
	 * Берем только те записи, поле `deleted_at` для которых равно NULL
	 *
	 * @param  \Illuminate\Database\Eloquent\Builder  $builder
	 * @param  \Illuminate\Database\Eloquent\Model  $model
	 * @return void
	 */
	public function apply(Builder $builder, Model $model)
	{
		$builder->whereNull($model->getQualifiedDeletedAtColumn());

		$this->extend($builder);
	}

	/**
	 * Удаление whereNull
	 *
	 * @param  \Illuminate\Database\Eloquent\Builder  $builder
	 * @param  \Illuminate\Database\Eloquent\Model  $model
	 * @return void
	 */
	public function remove(Builder $builder, Model $model)
	{
		$column = $model->getQualifiedDeletedAtColumn();

		$query = $builder->getQuery();

		foreach ((array) $query->wheres as $key => $where)
		{
			// Перебираем wheres в запросе, и когда находим наш - удаляем его из массива wheres по ключу. 
			// Это позволяет включать удаленную модель в отношения во время ленивой загрузки.
			if ($this->isSoftDeleteConstraint($where, $column))
			{
				unset($query->wheres[$key]);

				$query->wheres = array_values($query->wheres);
			}
		}
	}

<a name="relationships"></a>
## Отношения

Ваши таблицы скорее всего как-то связаны с другими таблицами БД. Например, статья в блоге может иметь несколько комментариев, а заказ может
быть с связан с оставившим его пользователем. Eloquent упрощает работу и управление такими отношениями.
Laravel поддерживает несколько типов связей:

- [Один к одному](#one-to-one)
- [Один ко многим](#one-to-many)
- [Многие ко многим](#many-to-many)
- [Has Many Through](#has-many-through)
- [Полиморфические связи](#polymorphic-relations)
- [Полиморфические связи многие ко многим](#many-to-many-polymorphic-relations)

<a name="one-to-one"></a>
### Один к одному

#### Создание связи «один к одному»

Связь вида «один к одному» является очень простой. К примеру, модель `User` может иметь один `Phone`.
Мы можем определить такое отношение в Eloquent.

	class User extends Model {

		public function phone()
		{
			return $this->hasOne('App\Phone');
		}

	}

Первый параметр, передаваемый `hasOne` - имя связанной модели. Как только отношение установлено вы можете полчить к нему доступ
через [динамические](#dynamic-properties) свойства Eloquent:

	$phone = User::find(1)->phone;

Сгенерированный SQL имеет такой вид:

	select * from users where id = 1

	select * from phones where user_id = 1

Заметьте, что Eloquent считает, что внешнее поле (`foreign_key`) в связанной таблице называется по имени модели плюс `_id`.
В данном случае предполагается, что это `user_id`. Если вы хотите перекрыть стандартное имя передайте второй параметр методу `hasOne`.
Если же в модели, для которой вы строите отношение (в данном случае `User`) ключ находится не в столбце `id`, то вы можете указать его
в качесте третьего аргумента:

	return $this->hasOne('App\Phone', 'foreign_key');

	return $this->hasOne('App\Phone', 'foreign_key', 'local_key');

#### Создание обратного отношения

Для создания обратного отношения в модели `Phone` используйте метод `belongsTo` («принадлежит к»):

	class Phone extends Model {

		public function user()
		{
			return $this->belongsTo('App\User');
		}

	}

В примере выше Eloquent будет искать поле `user_id` в таблице `phones`. Если вы хотите назвать этот ключ по-другому, передайте это имя вторым
параметром к методу `belongsTo`:

	class Phone extends Model {

		public function user()
		{
			return $this->belongsTo('App\User', 'local_key');
		}

	}

Вы можете добавить третий параметр, чтобы указать, какой столбец в родительской модели (`User`) сопоставлять столбцу `local_key`: 

	class Phone extends Model {

		public function user()
		{
			return $this->belongsTo('App\User', 'local_key', 'parent_key');
		}

	}

Здесь Laravel вернет объекты модели `User`, у которых значение `parent_key` будет равно значению столбца `local_key` у `Phone`.

<a name="one-to-many"></a>
### Один ко многим

Примером отношения «один ко многим» является статья в блоге, которая имеет несколько («много») комментариев. Вы можем смоделировать это
отношение таким образом:

	class Post extends Model {

		public function comments()
		{
			return $this->hasMany('Comment');
		}

	}

Теперь мы можем получить все комментарии с помощью [динамического свойства](#dynamic-properties):

	$comments = Post::find(1)->comments;

Если вам нужно добавить ограничения на получаемые комментарии, можно вызвать метод `comments` и продолжить добавлять условия:

	$comments = Post::find(1)->comments()->where('title', '=', 'foo')->first();

Как обычно, вы можете передать второй параметр к методу `hasMany` для перекрытия стандартного имени внешнего ключа в `Comment`
(в данном случае по дефолту тут `post_id`) и третий параметр для перекрытия стандатного имени местного ключа ('id' по умолчанию):

	return $this->hasMany('Comment', 'foreign_key');

	return $this->hasMany('Comment', 'foreign_key', 'local_key');

#### Определение обратного отношения

Для определения обратного отношения используйте метод `belongsTo`:

	class Comment extends Model {

		public function post()
		{
			return $this->belongsTo('App\Post');
		}

	}

<a name="many-to-many"></a>
### Многие ко многим

Отношения типа «многие ко многим» - более сложные, чем остальные виды отношений. Примером может служить пользователь, имеющий много ролей,
где роли также относятся ко многим пользователям. Например, один пользователь может иметь роль «Admin».
Нужны три таблицы для этой связи: `users`, `roles` и `role_user`. Название таблицы `role_user` происходит от **упорядоченного по алфавиту**
имён связанных моделей и должна иметь поля `user_id` и `role_id`.

Вы можете определить отношение «многие ко многим» через метод `belongsToMany`:

	class User extends Model {

		public function roles()
		{
			return $this->belongsToMany('App\Role');
		}

	}

Теперь мы можем получить роли через модель `User`:

	$roles = User::find(1)->roles;

Вы можете передать второй параметр к методу `belongsToMany` с указанием имени связующей (pivot) таблицы вместо стандартной:

	return $this->belongsToMany('App\Role', 'user_roles');

Вы также можете перекрыть имена ключей по умолчанию:

	return $this->belongsToMany('App\Role', 'user_roles', 'user_id', 'foo_id');

Конечно, вы можете определить и обратное отношение на модели `Role`:

	class Role extends Model {

		public function users()
		{
			return $this->belongsToMany('App\User');
		}

	}

<a name="has-many-through"></a>
### Множество связей через третью таблицу (Has Many Through)

Отношение «has many through» обеспечивает удобный короткий путь для доступа к удаленным отношениям через промежуточные отношения.
Например, сделаем так, чтобы модель `Country` могла иметь много `Post` **через** модель `User`. Структура таблиц такая:

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

Для того, чтобы получить `$country->posts`, определим вот такое отношение `hasManyThrough`:

	class Country extends Model {

		public function posts()
		{
			return $this->hasManyThrough('App\Post', 'App\User');
		}

	}

Как обычно, если наименование столбцов у вас в таблице отличается от общепринятого, вы можете указать их названия явно:

	class Country extends Model {

		public function posts()
		{
			return $this->hasManyThrough('App\Post', 'App\User', 'country_id', 'user_id');
		}

	}

<a name="polymorphic-relations"></a>
### Полиморфические отношения

Полиморфические отношения позволяют модели быть связанной с более, чем одной моделью. Например, может быть модель `Photo`, содержащая записи,
принадлежащие к моделям `Staff` (сотрудник) и `Order` (заказ). Мы можем создать такое отношение следующим образом:

	class Photo extends Model {

		public function imageable()
		{
			return $this->morphTo();
		}

	}

	class Staff extends Model {

		public function photos()
		{
			return $this->morphMany('Photo', 'imageable');
		}

	}

	class Order extends Model {

		public function photos()
		{
			return $this->morphMany('Photo', 'imageable');
		}

	}

Теперь мы можем получить фотографии и для сотрудника, и для заказа.

#### Чтение полиморфической связи

	$staff = Staff::find(1);

	foreach ($staff->photos as $photo)
	{
		//
	}

Однако истинная «магия» полиморфизма происходит при чтении связи на модели `Photo`:

#### Чтение связи на владельце полиморфического отношения

	$photo = Photo::find(1);

	$imageable = $photo->imageable;

Отношение `imageable` модели `Photo` вернёт либо объект `Staff` либо объект `Order` в зависимости от типа модели, к которой принадлежит фотография.

Чтобы понять, как это работает, давайте изучим структуру БД для полиморфического отношения.

#### Структура таблиц полиморфической связи

	staff
		id - integer
		name - string

	orders
		id - integer
		price - integer

	photos
		id - integer
		path - string
		imageable_id - integer
		imageable_type - string

Главные поля, на которые нужно обратить внимание: `imageable_id` и `imageable_type` в таблице `photos`. Первое содержит ID владельца,
в нашем случае - заказа или сотрудника, а второе - имя класса-модели владельца. Это позволяет ORM определить, какой класс модели должен
быть возвращёт при использовании отношения `imageable`.

<a name="many-to-many-polymorphic-relations"></a>
### Полиморфические отношения «многие ко многим»

Помимо стандартных полиморфических отношений, вы можете определить полиморфические отношения «многие ко многим». Например, у нас есть блог,
в котором могут публиковаться `Post` и `Video`, и каждый из них может иметь набор тэгов `Tag`. 

Cтруктура таблиц:

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

Модели `Post` и `Video` будут иметь отношение `morphToMany`:

	class Post extends Model {

		public function tags()
		{
			return $this->morphToMany('App\Tag', 'taggable');
		}

	}

А модель `Tag` - `morphedByMany` для `Post` и `Video`:

	class Tag extends Model {

		public function posts()
		{
			return $this->morphedByMany('App\Post', 'taggable');
		}

		public function videos()
		{
			return $this->morphedByMany('App\Video', 'taggable');
		}

	}

<a name="querying-relations"></a>
## Запросы к отношениям

#### Проверка связей при выборке

При чтении отношений модели вам может быть нужно ограничить результаты в зависимости от существования связи. Например, вы хотите получить
все статьи в блоге, имеющие хотя бы один комментарий. Для этого можно использовать метод `has`:

	$posts = Post::has('comments')->get();

Вы также можете указать оператор и число:

	$posts = Post::has('comments', '>=', 3)->get();

Метод `has` работает и с вложенными отношениями:

	$posts = Post::has('comments.votes')->get();

Здесь будут выбраны только те посты, которые имеют комментарии, за которые хотя бы кто-то проголосовал.		

Методы `whereHas` и `orWhereHas` позволяют использовать `where` в `has`-запросах:

	$posts = Post::whereHas('comments', function($q)
	{
		$q->where('content', 'like', 'foo%');

	})->get();

Здесь в `$posts` будут только те посты, у которых есть комменты, начинающиеся на 'foo'.

<a name="dynamic-properties"></a>
### Динамические свойства

Eloquent позволяет вам читать отношения через динамические свойства. Eloquent автоматически определит используемую связь и даже вызовет
`get` для связей «один ко многим» и `first` - для связей «один к одному». Например, у нас есть модель `Phone`:

	class Phone extends Model {

		public function user()
		{
			return $this->belongsTo('App\User');
		}

	}

	$phone = Phone::find(1);

Вместо того, чтобы получить e-mail пользователя так:

	echo $phone->user()->first()->email;

...вызов может быть сокращён до такого:

	echo $phone->user->email;

> **Примечание:** Отношения, которые возвращают несколько результатов, возвращают коллекцию, т.е. объект `Illuminate\Database\Eloquent\Collection`

<a name="eager-loading"></a>
## Жадная загрузка (eager loading)

Жадная загрузка (eager loading) призвана устранить проблему N+1. Например, представьте, что у нас есть модель `Book` со связью к модели `Author`.
Отношение определено как:

	class Book extends Model {

		public function author()
		{
			return $this->belongsTo('App\Author');
		}

	}

Теперь, у нас есть такой код:

	foreach (Book::all() as $book)
	{
		echo $book->author->name;
	}

Цикл выполнит один запрос для получения всех книг в таблице, а затем будет выполнять по одному запросу на каждую книгу для получения автора.
Таким образом, если у нас 25 книг, то потребуется 26 запросов.

К счастью, мы можем использовать жадную загрузку для кардинального уменьшения числа запросов. При вызове метода `with` отношения будет
загружены все за один запрос:

	foreach (Book::with('author')->get() as $book)
	{
		echo $book->author->name;
	}

В цикле выше будут выполнены всего два запроса:

	select * from books

	select * from authors where id in (1, 2, 3, 4, 5, ...)

Разумное использование жадной загрузки может кардинально повысить производительность вашего приложения.

Вы можете загрузить несколько отношений одновременно:

	$books = Book::with('author', 'publisher')->get();

Вы даже можете загрузить вложенные отношения:

	$books = Book::with('author.contacts')->get();

В примере выше, связь `author` будет загружена вместе со связью `contacts` модели автора.

### Ограничения жадной загрузки

Иногда вам может быть нужно не только загрузить отношение, но также указать условие для его загрузки:

	$users = User::with(['posts' => function($query)
	{
		$query->where('title', 'like', '%первое%');
	}])->get();

В этом примере мы загружаем сообщения пользователя, но только те, заголовок которых содержит подстроку «первое».

### Ленивая жадная загрузка

Можно подгрузить отношения динамически, т.е. после того как вы получили коллекцию основных объектов. Это может быть полезно тогда, когда
вам неизвестно, понадобятся вам далее отношения или нет.

	$books = Book::all();

	$books->load('author', 'publisher');

Вы также можете дополнить запрос при помощи функции-замыкания:

	$books->load(['author' => function($query)
	{
		$query->orderBy('published_date', 'asc');
	}]);	

<a name="inserting-related-models"></a>
## Вставка связанных моделей

#### Создание связанной модели

Часто вам нужно будет добавить связанную модель. Например, вы можете добавить новый комментарий к посту. Вместо явного указания значения
для поля `post_id` вы можете вставить модель через её родительскую модель - `Post`:

	$comment = new Comment(['message' => 'Новый комментарий.']);

	$post = Post::find(1);

	$comment = $post->comments()->save($comment);

В этом примере поле `post_id` вставленного комментария автоматически получит значение ID поста, к которому он оставлен.

Если вам надо сохранить несколько связанных моделей:

	$comments = [
		new Comment(['message' => 'A new comment.']),
		new Comment([message' => 'Another comment.']),
		new Comment(['message' => 'The latest comment.'])
	];

	$post = Post::find(1);

	$post->comments()->saveMany($comments);

### Связывание моделей Belongs To

При обновлении связей `belongsTo` («принадлежит к») вы можете использовать метод `associate`. Он установит внешний ключ (foregin key)
на связанной модели:

	$account = Account::find(10);

	$user->account()->associate($account);

	$user->save();

### Связывание моделей Many to Many

Вы также можете вставлять связанные модели при работе с отношениями «многие ко многим». Продолжим использовать наши модели `User` и `Role`
в качестве примеров (`User` может иметь несколько `Role`). Вы можем легко привязать новые роли к пользователю методом `attach`.

#### Связывание моделей «многие ко многим»

	$user = User::find(1);

	$user->roles()->attach(1);

Вы также можете передать массив атрибутов, которые должны быть сохранены в связующей (pivot) таблице для этого отношения:

	$user->roles()->attach(1, ['expires' => $expires]);

Конечно, существует противоположность `attach` - метод `detach`:

	$user->roles()->detach(1);

И `attach` и `detach` могут принимать массивы ID:

	$user = User::find(1);

	$user->roles()->detach([1, 2, 3]);

	$user->roles()->attach([1 => ['attribute1' => 'value1'], 2, 3]);	

#### Использование Sync для привязки моделей "многие ко многим"

Вы также можете использовать метод `sync` для привязки связанных моделей. Этот метод принимает массив ID, которые должны быть сохранены
в связующей таблице. Когда операция завершится только переданные ID будут существовать в промежуточной таблице для данной модели.

	$user->roles()->sync([1, 2, 3]);

#### Добавление данных для связующий таблицы при синхронизации

Вы также можете связать другие связующие таблицы с нужными ID:

	$user->roles()->sync([1 => ['expires' => true]]);

Иногда вам может быть нужно создать новую связанную модель и добавить её одной командой. Для этого вы можете использовать метод `save`:

	$role = new Role(['name' => 'Editor']);

	User::find(1)->roles()->save($role);

В этом примере новая модель `Role` будет сохранена и привязана к модели `User`. Вы можете также передать массив атрибутов для помещения
в связующую таблицу:

	User::find(1)->roles()->save($role, ['expires' => $expires]);

<a name="touching-parent-timestamps"></a>
## Обновление данных времени владельца

Когда модель принадлежит к другой посредством `belongsTo` - например, `Comment`, принадлежащий `Post`- иногда нужно обновить время
изменения владельца при обновлении связанной модели. Например, при изменении модели `Comment` вы можете обновлять поле `updated_at`
её модели `Post`. Eloquent делает этот процесс простым - просто добавьте свойство `touches`, содержащее имена всех отношений с моделями-потомками.

	class Comment extends Model {

		protected $touches = ['post'];

		public function post()
		{
			return $this->belongsTo('App\Post');
		}

	}

Теперь при обновлении `Comment` владелец `Post` также обновит своё поле `updated_at`:

	$comment = Comment::find(1);

	$comment->text = 'Изменение этого комментария.';

	$comment->save();

<a name="working-with-pivot-tables"></a>
## Работа со связующими таблицами

Как вы уже узнали, работа отношения многие ко многим требует наличия промежуточной (pivot) таблицы. Например, предположим, что наш
объект `User` имеет множество связанных объектов `Role`. После чтения отношения мы можем прочитать свойство `pivot` на обоих моделях:

	$user = User::find(1);

	foreach ($user->roles as $role)
	{
		echo $role->pivot->created_at;
	}

Заметьте, что каждая модель `Role` автоматически получила атрибут `pivot`. Этот атрибут содержит модель, представляющую промежуточную таблицу
и она может быть использована как любая другая модель Eloquent.

По умолчанию, только ключи будут представлены в объекте `pivot`. Если ваша связующая таблица содержит другие поля вы можете указать их
при создании отношения:

	return $this->belongsToMany('App\Role')->withPivot('foo', 'bar');

Теперь атрибуты `foo` и `bar` будут также доступны на объекте `pivot` модели `Role`.

Если вы хотите автоматически поддерживать поля `created_at` и `updated_at` актуальными, используйте метод `withTimestamps` при создании отношения:

	return $this->belongsToMany('App\Role')->withTimestamps();

Для удаления всех записей в связующей таблице можно использовать метод `detach`:

#### Удаление всех связующих записей

	User::find(1)->roles()->detach();

Заметьте, что эта операция не удаляет записи из таблицы `roles`, а только из связующей таблицы.

#### Обновление записей в связующей таблице

Иногда вам нужно просто обновить связующую таблицу, не присоединяя или отсоединяя ничего. Используйте метод `updateExistingPivot`:

	User::find(1)->roles()->updateExistingPivot($roleId, $attributes);

#### Своя модель Pivot

Laravel позволяет создать свою модель Pivot для работы со связанной таблицей. Сначала создайте модель, которая наследуется (extends)
от `Eloquent`, и назовите её, например, `BaseModel`. Далее все свои модели наследуйте уже от неё. А в `BaseModel` создайте следующую функцию,
которая возвращает объект для работы со связанной таблицей:

	public function newPivot(Model $parent, array $attributes, $table, $exists)
	{
		return new YourCustomPivot($parent, $attributes, $table, $exists);
	}

<a name="collections"></a>
## Коллекции

Все методы Eloquent, возвращающие набор моделей - либо через `get`, либо через отношения - возвращают объект-коллекцию. Этот объект реализует
стандартный интерфейс PHP `IteratorAggregate`, что позволяет ему быть использованным в циклах наподобие массива. Однако этот объект также
имеет набор других полезных методов для работы с результатом запроса.

Например, мы можем выяснить, содержит ли результат запись с определённым первичным ключом методом `contains`.

#### Проверка на существование ключа в коллекции

	$roles = User::find(1)->roles;

	if ($roles->contains(2))
	{
		//
	}

Коллекции также могут быть преобразованы в массив или строку JSON:

	$roles = User::find(1)->roles->toArray();

	$roles = User::find(1)->roles->toJson();

Если коллекция преобразуется в строку результатом будет JSON-выражение:

	$roles = (string) User::find(1)->roles;

Коллекции Eloquent имеют несколько полезных методов для прохода и фильтрации содержащихся в них элементов.

#### Проход и фильтрация элементов коллекции

	$roles = $user->roles->each(function($role)
	{

	});

	$roles = $user->roles->filter(function($role)
	{

	});

#### Применение функции обратного вызова

	$roles = User::find(1)->roles;

	$roles->each(function($role)
	{
		//
	});

#### Сохранение коллекции по значению

	$roles = $roles->sortBy(function($role)
	{
		return $role->created_at;
	});

Иногда вам может быть нужно получить собственный объект Collection со своими методами. Вы можете указать его при определении модели Eloquent,
перекрыв метод `newCollection`.

#### Использование произвольного класса коллекции

	class User extends Model {

		public function newCollection(array $models = [])
		{
			return new CustomCollection($models);
		}

	}

<a name="accessors-and-mutators"></a>
## Читатели (accessors) и преобразователи (mutators)

Eloquent содержит мощный механизм для преобразования атрибутов модели при их чтении и записи. Просто объявите в её классе метод `getFooAttribute`.
Помните, что имя метода должно следовать соглашению camelCase, даже если поля таблицы используют соглашение snake-case, т.е. с подчёркиваниями.

#### Объявление читателя

	class User extends Model {

		public function getFirstNameAttribute($value)
		{
			return ucfirst($value);
		}

	}

В примере выше поле `first_name` теперь имеет читателя (accessor). Заметьте, что оригинальное значение атрибута передаётся методу в виде параметра.

Преобразователи (mutators) объявляются подобным образом.

#### Объявление преобразователя

	class User extends Model {

		public function setFirstNameAttribute($value)
		{
			$this->attributes['first_name'] = strtolower($value);
		}

	}

<a name="date-mutators"></a>
## Преобразователи дат

По умолчанию Eloquent преобразует поля `created_at` и `updated_at` в объекты [Carbon](https://github.com/briannesbitt/Carbon), которые
предоставляют множество полезных методов, расширяя стандартный класс PHP `DateTime`.

Вы можете указать, какие поля будут автоматически преобразованы и даже полностью отключить преобразование перекрыв метод `getDates` класса модели.

	public function getDates()
	{
		return ['created_at'];
	}

Когда поле является датой, вы можете установить его в число-оттиск времени формата Unix (timestamp), строку даты формата(`Y-m-d`),
строку даты-времени и, конечно, экземпляр объекта `DateTime` или `Carbon`.

Чтобы полностью отключить преобразование дат просто верните пустой массив из метода `getDates`.

	public function getDates()
	{
		return [];
	}

<a name="attribute-casting"></a>
## Приведение атрибутов к определенному типу

Бывает, что некоторые атрибуты модели вам надо приводить к определенному типу данных. Можно, конечно, написать к этим полям преобразователи,
а можно поступить проще - установить свойство `casts`, в котором перечислить эти аттрибуты и типы данных, к которым их нужно приводить. Например:

	/**
	 * Теперь атрибут is_admin - логического типа
	 *
	 * @var array
	 */
	protected $casts = [
		'is_admin' => 'boolean',
	];

Несмотря на то, что в БД `is_admin` хранится в целочисленном (integer) поле, в модели он будет хранится в виде логической переменной - `true` или `false`.

Поддерживаемые типы данных: `integer`, `real`, `float`, `double`, `string`, `boolean`, и `array`.

Преобразование `array` может быть особенно полезно для столбцов, в которых хранится сериализованный JSON. Такие столбцы будут автоматически
десериализовываться и JSON будет преобразовываться в массив:

	protected $casts = [
		'options' => 'array',
	];

Теперь мы можем хранить массивы в БД:

	$user = User::find(1);

	// $options - массив, полученный из JSON
	$options = $user->options;

	// options автоматически сериализуется обратно в JSON !
	$user->options = ['foo' => 'bar'];

<a name="model-events"></a>
## События моделей

Модели Eloquent инициируют несколько событий, что позволяет вам добавить к ним свои обработчики с помощью следующих методов:
`creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`.

Когда первый раз сохраняется новая модель возникают события `creating` и `created`. Если модель уже существовала на момент вызова
метода `save`, вызываются события `updating` и `updated`. В обоих случаях также возникнут события `saving` и `saved`.

#### Отмена сохранения модели через события

Если обработчики `creating`, `updating`, `saving` или `deleting`вернут значение `false`, то действие будет отменено.

	User::creating(function($user)
	{
		if ( ! $user->isValid()) return false;
	});

#### Где регистрировать слушателей событий

Регистрировать слушателей событий модели можно в вашем сервис-провайдере `EventServiceProvider`. Например:

	/**
	 * Это место для регистрации слушателей событий (event listeners) вашего приложения
	 *
	 * @param  \Illuminate\Contracts\Events\Dispatcher  $events
	 * @return void
	 */
	public function boot(DispatcherContract $events)
	{
		parent::boot($events);

		User::creating(function($user)
		{
			//
		});
	}

<a name="model-observers"></a>
## Наблюдатели моделей

Для того, чтобы держать всех обработчиков событий моделей вместе вы можете зарегистрировать наблюдателя (observer). Объект-наблюдатель может
содержать методы, соответствующие различным событиям моделей. Например, методы `creating`, `updating` и `saving`, а также любые другие методы,
соответствующие именам событий.

К примеру, класс наблюдателя может выглядеть так:

	class UserObserver {

		public function saving($model)
		{
			//
		}

		public function saved($model)
		{
			//
		}

	}

Вы можете зарегистрировать его используя метод `observe`:

	User::observe(new UserObserver);

<a name="model-url-generation"></a>
## Генерация урлов

Вы можете передать экземпляр модель в хелперах `route` или `action`, в урл будет вставлен соответствующий ключ. Например:

	Route::get('user/{user}', 'UserController@show');

	action('UserController@show', [$user]);

Здесь в {user} будет передано `$user->id`. Вы можете изменить передаваемый в урл ключ следующим методом модели:

    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="converting-to-arrays-or-json"></a>
## Преобразование в массивы и JSON

При создании JSON API вам часть потребуется преобразовывать модели к массивам или выражениям JSON. Eloquent содержит методы для выполнения
этих задач. Для преобразования модели или загруженного отношения к массиву можно использовать метод `toArray`.

#### Преобразование модели к массиву

	$user = User::with('roles')->first();

	return $user->toArray();

Заметьте, что целая коллекция моделей также может быть преобразована к массиву:

	return User::all()->toArray();

Для преобразования модели к JSON, вы можете использовать метод `toJson`:

#### Преобразование модели к JSON

	return User::find(1)->toJson();

Обратите внимание, что если модель преобразуется к строке, результатом также будет JSON - это значит, что вы можете возвращать объекты Eloquent
напрямую из ваших маршрутов!

#### Возврат модели из маршрута

	Route::get('users', function()
	{
		return User::all();
	});

Иногда вам может быть нужно ограничить список атрибутов, включённых в преобразованный массив или JSON-строку - например, скрыть пароли.
Для этого определите в классе модели свойство `hidden`.

#### Скрытие атрибутов при преобразовании в массив или JSON

	class User extends Model {

		protected $hidden = ['password'];

	}

Вы также можете использовать атрибут `visible` для указания разрешённых полей:

	protected $visible = ['first_name', 'last_name'];

<a name="array-appends"></a>
Иногда вам может быть нужно добавить поле, которое не существует в таблице. Для этого просто определите для него читателя:

	public function getIsAdminAttribute()
	{
		return $this->attributes['admin'] == 'yes';
	}

Как только вы создали читателя добавьте его имя к свойству-массиву `appends` класса модели:

	protected $appends = ['is_admin'];

Как только атрибут был добавлен к списку `appends`, он будет включён в массивы и выражения JSON, образованные от этой модели.
Атрибуты в `appends` подчиняются правилам `visible` и `hidden` модели.
