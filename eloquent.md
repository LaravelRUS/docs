git 4345d54b249a569299aa0de15e4215fcb473e590

---

# Eloquent: введение

- [Введение](#introduction)
- [Определение моделей](#defining-models)
	- [Принятые соглашения](#eloquent-model-conventions)
- [Получение нескольких моделей](#retrieving-multiple-models)
- [Получение нескольких экземпляров моделей](#retrieving-single-models)
	- [Агрегирующие функции](#retrieving-aggregates)
- [Вставка и обновление моделей](#inserting-and-updating-models)
	- [Вставка](#basic-inserts)
	- [Обновление](#basic-updates)
	- [Массовое присваивание атрибутов](#mass-assignment)
- [Удаление моделей](#deleting-models)
	- [Псевдоудаление](#soft-deleting)
	- [Работа с псевдоудалёнными моделями](#querying-soft-deleted-models)
- [События моделей](#events)

<a name="introduction"></a>
## Введение

ORM Eloquent - красивая и простая реализация паттерна ActiveRecord для работы с базой данных. Каждая таблица имеет соответствующий класс-модель, который используется для работы с этой таблицей. Модели позволяют читать данные из таблиц и записывать данные в таблицу.

Прежде чем начать настройте ваше соединение с БД в файле `config/database.php`. [Инструкция по настройке БД](/docs/{{version}}/database#configuration).

<a name="defining-models"></a>
## Определение моделей

Давайте создадим Eloquent-модель. Модели обычно лежат в корне папки `app`, но вы вольны располагать их в любой папке, которая участвует в автозагрузке классов. Модель должна расширять класс `Illuminate\Database\Eloquent\Model`.

Самый простой способ создать модель - воспользоваться [артизан-командой](/docs/{{version}}/artisan) `make:model`:

	php artisan make:model User

Если вы хотите вместе с моделью создать [миграцию](/docs/{{version}}/schema#database-migrations) для соответственного изменения БД, добавьте опцию `--migration` или `-m`:

	php artisan make:model User --migration

	php artisan make:model User -m

<a name="eloquent-model-conventions"></a>
### Принятые соглашения именования

Для удобства разработки в Laravel существуют определённые правила названий таблиц и столбцов, принятые по умолчанию.

Возьмем, например, модель `Flight`, которая оперирует данными в таблице `flights`:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Flight extends Model
	{
	    //
	}


#### Название таблицы

По умолчанию считается, что название таблицы представляет собой множественное число от названия модели, записанное в "snake case", т.е. неведущие большие буквы снабжаются префиксным символом подчеркивания и все слово переводится в прописную (нижнюю) раскладку. Например: SomeModel -> some_models.

Чтобы изменить это, задайте явно название таблицы в свойстве `table`:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Flight extends Model
	{
		/**
		 * The table associated with the model.
		 *
		 * @var string
		 */
		protected $table = 'my_flights';
	}

#### Primary Keys

По умолчанию primary key называется `id`. Вы можете изменить имя в свойстве `$primaryKey`.

#### Timestamps

По умолчанию Eloquent ожидает, что в таблице будут столбцы `created_at` и `updated_at`, куда записывается записывается дата (timestamp) создания и обновления соответственно. Если вам не нужна эта фича и в таблице нет этих столбцов, то установите свойство `$timestamps` в `false`:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Flight extends Model
	{
		/**
		 * Indicates if the model should be timestamped.
		 *
		 * @var bool
		 */
		public $timestamps = false;
	}

Изменить формат даты в этих столбцах вы можете при помощи свойства `$dateFormat`. Оно определяет, в каком формате дата будет хранится в БД и выглядеть в массиве или json данных, полученных из БД.

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Flight extends Model
	{
		/**
		 * The storage format of the model's date columns.
		 *
		 * @var string
		 */
		protected $dateFormat = 'U';
	}


<a name="retrieving-multiple-models"></a>
## Получение нескольких экземпляров моделей

После того, как вы создали модель [и таблицу, ассоциированную с ней](/docs/{{version}}/schema), можно попробовать получить из таблицы через модель какие-нибудь данные. Хороший путь - думать о модели как о продвинутом [конструкторе запросов](/docs/{{version}}/queries), при помощи которого можно писать более короткие и гибкие запросы, связанные с таблицей модели. Например:

	<?php namespace App\Http\Controllers;

	use App\Flight;
	use App\Http\Controllers\Controller;

	class FlightController extends Controller
	{
		/**
		 * Show a list of all available flights.
		 *
		 * @return Response
		 */
		public function index()
		{
			$flights = Flight::all();

			return view('flight.index', ['flights' => $flights]);
		}
	}

#### Получение значений столбцов

Чтобы получить значение столбца, обратитесь к свойству модели с таким же именем. Например, пройдемся по всем полученным строкам таблицы и выведем содержимое столбца `name`:

	foreach ($flights as $flight) {
		echo $flight->name;
	}

#### Корректировка запроса

Метод `all` Eloquent возвращает все строки из таблицы модели. Но так как каждая модель Eloquent содержит методы [конструктора запросов](/docs/{{version}}/queries), мы можем использовать их для того, чтобы задать нужные ограничения в запрос к таблице и получить данные при помощи метода `get()`:

	$flights = App\Flight::where('active', 1)
				   ->orderBy('name', 'desc')
				   ->take(10)
				   ->get();

> **Примечание:** Хорошо изучите [конструктор запросов](/docs/{{version}}/queries), так как все его методы доступны в моделях.

#### Коллекции

Методы `all` и `get`, возвращающие набор данных - возвращают объект `Illuminate\Database\Eloquent\Collection`. Этот класс предоставляет [множество полезных методов](/docs/{{version}}/eloquent-collections) для работы с результатами запросов. Также вы можете работать с коллекцией как с массивом, перебирая её в цикле `foreach`:

	foreach ($flights as $flight) {
		echo $flight->name;
	}

#### Chunking Results

If you need to process thousands of Eloquent records, use the `chunk` command. The `chunk` method will retrieve a "chunk" of Eloquent models, feeding them to a given `Closure` for processing. Using the `chunk` method will conserve memory when working with large result sets:

Если вам надо обработать результат запроса, который содержит тысячи элементов, и не потратить всю память на сервере, используйте метод `chunk`. Он получает "кусочек" моделей и обрабатывает их в функции-замыкании:

	Flight::chunk(200, function ($flights) {
		foreach ($flights as $flight) {
			//
		}
	});

<a name="retrieving-single-models"></a>
## Получение одного экземпляра модели / Аггрегация

Чтобы получить один экземпляр модели из запроса, вспользуйтесь методами `find` и `first`. Они возвращают экземпляр модели, а не коллекцию моделей.

	// Получить экземпляр модели по ключу...
	$flight = App\Flight::find(1);

	// Получить первый экземпляр модели по запросу...
	$flight = App\Flight::where('active', 1)->first();

#### Not Found Exceptions

Иногда вам нужно бросить исключение, если экземпляр модели не найден. Это, как правило, нужно в роутах или контроллерах, когда вы получаете критичные для отображения страницы данные. В этом случае вам помогут методы `findOrFail` и `firstOrFail`. Они бросают исключение `Illuminate\Database\Eloquent\ModelNotFoundException`, если результат не получен:

	$model = App\Flight::findOrFail(1);

	$model = App\Flight::where('legs', '>', 100)->firstOrFail();

Если иключение не поймано, пользователю будет отдана HTTP-ошибка 404, так что нет необходимости обрабатывать исключение для того, чтобы бросать 404 вручную:

	Route::get('/api/flights/{id}', function ($id) {
		return App\Flight::findOrFail($id);
	});

<a name="retrieving-aggregates"></a>
### Агрегирующие функции

Вы также можете использовать в запросе к моделям аггрегирующие функции [конструктора запросов](/docs/{{version}}/queries) - такие, как `count`, `sum`, `max`. Они возвращают числовое значение, а не экземпляр модели.

	$count = App\Flight::where('active', 1)->count();

	$max = App\Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>
## Вставка и обновление моделей

<a name="basic-inserts"></a>
### Вставка

Чтобы вставить строку в таблицу, создайте модель, установите нужные аттрибуты и вызовите метод `save`:

	<?php namespace App\Http\Controllers;

	use App\Flight;
	use Illuminate\Http\Request;
	use App\Http\Controllers\Controller;

	class FlightController extends Controller
	{
		/**
		 * Create a new flight instance.
		 *
		 * @param  Request  $request
		 * @return Response
		 */
		public function store(Request $request)
		{
			// Валидируем запрос...

			$flight = new Flight;

			$flight->name = $request->name;

			$flight->save();
		}
	}

В этом примере мы взяли значение `name` из аттрибутов HTTP-запроса и присвоили его аттрибуту `name` модели. Когда вызывается метод `save`, содержимое модели добавляется в соответствующую таблицу базы данных. Помимо значений модели, автоматически заполняются столбцы `created_at` и `updated_at`, если их поддержка не запрещена.

<a name="basic-updates"></a>
### Обновление

Если модель существует в базе данных (есть строка с тем же primary key, что и в модели) метод `save` обновляет данные в БД, записывая их туда из модели. Таким образом, чтобы одновить модель, вы должны получить её из БД, установить необходимые аттрибуты и сохранить. Столбец `updated_at` обновится автоматически, если поддержка не запрещена в модели.

	$flight = App\Flight::find(1);

	$flight->name = 'New Flight Name';

	$flight->save();

Вы текже можете обновлять несколько моделей за один запрос, воспользовавшись преимуществами [конструктора запросов](/docs/{{version}}/queries). Например, мы хотим пометить, что все активные рейсы в San Diego задерживаются:

	App\Flight::where('active', 1)
			  ->where('destination', 'San Diego')
			  ->update(['delayed' => 1]);

Метод `update` принимает на вход массив вида "название столбца" => "данные для записи".

<a name="mass-assignment"></a>
### Массовое присваивание атрибутов

Для создания и одновременного сохранения модели можно использовать метод `create`. Метод получает на вход ассоциативный массив данных, которые надо присвоить модели и возвращает экземпляр сохранённой модели. Однако, прежде чем использовать этот метод, вы должны определить свойства `fillable` или `guarded` модели, так как модели Eloquent имеют защиту от массового присваивания аттрибутов.

Чем плохо массовое присваивание и почему от него нужно защищаться ? Дело в том, что при POST HTTP-запросе злоумышленник может прислать не только значения из формы, но и добавить несколько аттрибутов от себя. Например, `is_admin`, или другое поле, которое пользователям нельзя изменять напрямую. И если вы передаете данные запроса в метод `create` сразу, без фильтрации (в случае больших форм это может быть удобно), то без защиты от массового присваивания вы получаете потенциальную дыру в безопасности.

Вы можете оставить запрет массового присваивания для модели и разрешить его только для выбранных свойств. Для этого перечислите список этих свойств в свойстве модели `$fillable`:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Flight extends Model
	{
		/**
		 * The attributes that are mass assignable.
		 *
		 * @var array
		 */
		protected $fillable = ['name'];
	}

Здесь в методах массового присвоения мы разрешили использовать только свойство `name`.

	$flight = App\Flight::create(['name' => 'Flight 10']);

Но вы можете поступить и по-другому. Вы можете разрешить массовое присваивание для всех свойств модели, запретив его только для избранных свойств. Для этого перечислите их в свойстве модели `$guarded`:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Flight extends Model
	{
		/**
		 * The attributes that aren't mass assignable.
		 *
		 * @var array
		 */
		protected $guarded = ['price', 'delayed'];
	}

Здесь в методах массового присвоения можно использовать любые свойства, **кроме** `price` и `delayed`.

Естественно, вы можете использовать или `$fillable` или `$guarded` - но не оба сразу.

#### Другие способы создания модели

Есть еще два метода создания экземпляра модели, которые используют массовое присваивание - `firstOrCreate` и `firstOrNew`. `firstOrCreate` обращается к БД, используя в качестве условий поданный в аргументы ассоциативный массив. Если запись по таким условиям не найдена в БД, она там создастся.

Метод `firstOrNew`, как и `firstOrCreate` пытается найти запись в БД по переданным условиям, и если не находит - возвращает экземпляр модели с установленными параметрами. Обратите внимание, что этот метод не сохраняет модель в БД.

	// Retrieve the flight by the attributes, or create it if it doesn't exist...
	$flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

	// Retrieve the flight by the attributes, or instantiate a new instance...
	$flight = App\Flight::firstOrNew(['name' => 'Flight 10']);

<a name="deleting-models"></a>
## Удаление модели

Для того, чтобы удалить модель, вызовите метод `delete()`:

	$flight = App\Flight::find(1);

	$flight->delete();

#### Удаление модели по ключу

В примере выше мы сначала получили модель, а затем удалили её. Но если вам известен primary key модели, необязательно её загружать из БД, можно удалить сразу при помощи метода `destroy`:

	App\Flight::destroy(1);

	App\Flight::destroy([1, 2, 3]);

	App\Flight::destroy(1, 2, 3);

#### Удаление моделей при помощи запроса

Вы также можете удалить все модели, попадающие под определённые условия:

	$deletedRows = App\Flight::where('votes', '>', 100)->delete();

<a name="soft-deleting"></a>
### Псевдоудаление

Модели Eloquent имеют опцию псевдоудаления - при таком удалении данные остаются в базе данных, но не участвуют в дальнейших выборках. В поле `deleted_at` при этом записывается текущее время удаления. Для включения псевдоудалений добавьте в модель трейт `Illuminate\Database\Eloquent\SoftDeletes`:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;
	use Illuminate\Database\Eloquent\SoftDeletes;

	class Flight extends Model
	{
		use SoftDeletes;

		/**
		 * The attributes that should be mutated to dates.
		 *
		 * @var array
		 */
		protected $dates = ['deleted_at'];
	}

И, конечно, в таблице модели должен существовать столбец `deleted_at`. Добавить его в [миграции](/docs/{{version}}/schema) можно так:

	Schema::table('flights', function ($table) {
		$table->softDeletes();
	});

Метод `trashed` возвращает true, если `deleted_at` у модели не равняется null, т.е. модель подвергалась псевдоудалению:

	if ($flight->trashed()) {
		//
	}

<a name="querying-soft-deleted-models"></a>
### Работа с псевдоудалёнными моделями

#### Получение псевдоудалённых моделей

Псевдоудалённые модели автоматически исключаются из любых обычных выборок. Чтобы использовать выборки с псевдоудалёнными моделями используйте метод `withTrashed`:

	$flights = App\Flight::withTrashed()
					->where('account_id', 1)
					->get();

Вы можете использовать его в подгрузке [отношений](/docs/{{version}}/eloquent-relationships) модели:

	$flight->history()->withTrashed()->get();

#### Получение только псевдоудалённых моделей

Метод `onlyTrashed` вернет **только** псевдоудалённые модели:

	$flights = App\Flight::onlyTrashed()
					->where('airline_id', 1)
					->get();

#### Восстановление псевдоудалённых моделей

Иногда вам нужно восстановить псевдоудалённую модель. Для этого используйте метод `restore`:

	$flight->restore();

Его можно использовать на выборке нескольких моделей:

	App\Flight::withTrashed()
			->where('airline_id', 1)
			->restore();

Также можно восстановить все псевдоудалённые модели в [отношениях](/docs/{{version}}/eloquent-relationships):

	$flight->history()->restore();

#### Окончательное удаление 

Иногда вам нужно окончательно удалить псевдоудалённую модель. Метод `forceDelete` физически удаляет запись в базе данных:

	// удаление псевдоудалённого экземпляра
	$flight->forceDelete();

	// удаление всех псевдоудалённых моделей в отношении
	$flight->history()->forceDelete();

<a name="events"></a>
## События моделей

Модели Eloquent в процессе работы инициируют некоторые события (envents), что позволяет вам добавить некоторый свой функционал к работе моделей Eloquent. Подписаться на события можно при помощи следующих методов: `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`.

<a name="basic-usage"></a>
### Основы использования

Когда новая модель сохраняется в базу данных в первый раз, запускаются события `creating` и `created`. Если модель уже есть в базе данных и сохраняется в ней при помощи метода `save`, запускаются события `updating` и `updated`. И в обоих случаях запускаются события `saving` и `saved`.

Если обработчики `creating`, `updating`, `saving` или `deleting` вернут значение `false`, то действие будет отменено.

Давайте попробуем внедриться в процесс создания модели и отменим его, если данные модели не пройдут внутреннюю валидацию (эту валидацию мы будем проводить с методе `isValid` модели). Для этого определим слушателя события `creating` в [сервис-провайдере](/docs/{{version}}/providers). Если слушатель возвращает `false`, то операция `save()` или `update()` будет отменена.

	<?php namespace App\Providers;

	use App\User;
	use Illuminate\Support\ServiceProvider;

	class AppServiceProvider extends ServiceProvider
	{
	    /**
	     * Bootstrap any application services.
	     *
	     * @return void
	     */
		public function boot()
		{
			User::creating(function ($user) {
				if ( ! $user->isValid()) {
					return false;
				}
			});
		}

		/**
		 * Register the service provider.
		 *
		 * @return void
		 */
		public function register()
		{
			//
		}
	}