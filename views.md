git 4deba2bfca6636d5cdcede3f2068eff3b59c15ce

---

# Шаблоны (views)

- [Основы использования](#views)
- [Композеры](#view-composers)

<a name="basic-usage"></a>
## Основы использования

Views (представления, отображения, шаблоны) обычно содержат HTML-код вашего приложения и представляют собой удобный способ разделения бизнес-логики и логики отображения информации. Шаблоны хранятся в папке `resources/views`.

Простой шаблон выглядит примерно так:

	<!-- resources/views/greeting.php -->

	<html>
		<body>
			<h1>Hello, <?php echo $name; ?></h1>
		</body>
	</html>

Из данного шаблона можно сформировать ответ в браузер примерно так:

	Route::get('/', function()
	{
		return view('greeting', ['name' => 'James']);
	});	

Как вы можете видеть, первый аргумент хелпера `view` - имя файла шаблона в папке `resources/views`, а второй - данные, которые будут переданы в отображение.

Конечно, вы можете делать поддиректории в `resources/views`. Например, если ваш шаблон находится в файле `resources/views/admin/profile.php`, то вызов хэлпера будет выглядеть так:

	return view('admin.profile', $data); 

Или так: 

	return view('admin/profile', $data);	

#### Передача данных в шаблон

Следующие примеры делают доступным в шаблоне переменную `$name` и присваивают ей значение 'Victoria':

	// используя with()
	$view = view('greeting')->with('name', 'Victoria');

	// используя "магический" метод 
	$view = view('greeting')->withName('Victoria');	

	// при помощи второго параметра хэлпера
	$view = view('greetings', ['name' => 'Victoria']);

#### Передача данных во все шаблоны

Иногда вам нужно передать данные во все шаблоны (например, это может быть залогиненый пользователь). Вы можете сделать это несколькими путями: при помощи хэлпера view(), используя `Illuminate\Contracts\View\Factory` [contract](/docs/{{version}}/contracts) или при помощи общего шаблона (wildcard) композера.

При помощи хэлпера это можно сделать вот так:

	view()->share('data', [1, 2, 3]);

Или, если вы хотите использовать фасад, как в Laravel 4.x:
 
	View::share('data', [1, 2, 3]);

Этот код вы можете положить в метод `boot()` сервис-провайдера - или общего сервис-провайдера приложения `AppServiceProvider` или своего собственного.

> **Примечание:** Когда хэлпер `view()` вызывается без аргументов, он возвращает имплементацию контракта `Illuminate\Contracts\View\Factory`

#### Определение наличия шаблона

Чтобы определить, есть ли заданный файл шаблона на диске, используйте метод `exist()`

	if (view()->exists('emails.customer'))
	{
		//
	}

#### Получение шаблона по полному пути 

Вы можете взять файл шаблона по полному пути до него в файловой системе:

	return view()->file($pathToFile, $data);

<a name="view-composers"></a>
## Композеры

Композеры (view composers) - функции-замыкания или методы класса, которые вызываются, когда шаблон рендерится в строку. Если у вас есть данные, которые вы хотите привязать к шаблону при каждом его рендеринге, то композеры помогут вам выделить такую логику в отдельное место. 

### Определение композера

Регистрировать композеры можно внутри [сервис-провайдера](/docs/{{version}}/providers). Мы будем использовать фасад `View` для того, чтобы получить доступ к имплементации контракта `Illuminate\Contracts\View\Factory`:

	<?php namespace App\Providers;

	use View;
	use Illuminate\Support\ServiceProvider;

	class ComposerServiceProvider extends ServiceProvider {

		/**
		 * Register bindings in the container.
		 *
		 * @return void
		 */
		public function boot()
		{
			// Если композер реализуется при помощи класса:
			View::composer('profile', 'App\Http\ViewComposers\ProfileComposer');

			// Если композер реализуется в функции-замыкании:
			View::composer('dashboard', function()
			{

			});
		}
		
		/**
	     	* Register
	     	*
	     	* @return void
	     	*/
	    	public function register()
	    	{
	        	//
	    	}

	}

> **Примечание:** В Laravel нет папки, в которой должны находиться классы композеров, вы можете создать её сами там, где вам будет удобно. Например, это может быть `App\Http\Composers`.

Допустим, мы хотим, чтобы композер `profile` был реализован в классе (см. код выше). Метод `ProfileComposer@compose` будет вызываться каждый раз перед тем, как шаблон `profile` будет рендерится в строку.
Давайте определим класс композера:


	<?php namespace App\Http\Composers;

	use Illuminate\Contracts\View\View;
	use Illuminate\Users\Repository as UserRepository;

	class ProfileComposer {

		/**
		 * The user repository implementation.
		 *
		 * @var UserRepository
		 */
		protected $users;

		/**
		 * Create a new profile composer.
		 *
		 * @param  UserRepository  $users
		 * @return void
		 */
		public function __construct(UserRepository $users)
		{
			// Зависимости разрешаются автоматически службой контейнера...
			$this->users = $users;
		}

		/**
		 * Bind data to the view.
		 *
		 * @param  View  $view
		 * @return void
		 */
		public function compose(View $view)
		{
			$view->with('count', $this->users->count());
		}

	}

Метод `compose` должен получать в качестве аргумента инстанс `Illuminate\Contracts\View\View`. Для передачи переменных в шаблон используйте метод `with()`.

> **Примечание:** Все композеры берутся из [сервис-контейнера](/docs/{{version}}/container), поэтому вы можете указать необходимые зависимости в конструкторе композера - они будут автоматически поданы ему.

#### Wildcards имен шаблонов композеров

В названии шаблонов можно использовать wildcards (паттерны). Например, вот так можно назначить композер для всех шаблонов:

	View::composer('*', function()
	{
		//
	});

#### Назначение композера для нескольких шаблонов

Вместо одного имени шаблона вы можете использовать массив имен:

	View::composer(['profile', 'dashboard'], 'App\Http\ViewComposers\MyViewComposer');

#### Регистрация нескольких композеров

Вы можете использовать метод `composers` чтобы зарегистрировать несколько композеров одновременно:	

	View::composers([
		'App\Http\ViewComposers\AdminComposer' => ['admin.index', 'admin.profile'],
		'App\Http\ViewComposers\UserComposer' => 'user',
		'App\Http\ViewComposers\ProductComposer' => 'product'
	]);

### Создатель (view creators)

Создатели шаблонов работают почти так же, как композеры, но вызываются сразу после создания объекта шаблона, а не во время его рендеринга в строку. Для регистрации используйте метод `creator`:

	View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');	
