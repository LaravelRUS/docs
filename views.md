git 7a2a7184633dcbc0ad94eacf93b61b06de604b6e

---

# Шаблоны (views)

- [Основы использования](#basic-usage)
    - [Передача данных в шаблон](#passing-data-to-views)
    - [Данные доступные из всех шаблонов](#sharing-data-with-all-views)
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

После сохранения этого кода в файл `resources/views/greeting.php`, мы можем отобразить его, используя глобальную функцию-хелпер `view`:

	Route::get('/', function() {
		return view('greeting', ['name' => 'James']);
	});	

Как вы можете видеть, первый аргумент хелпера `view` - имя файла шаблона в папке `resources/views`, а второй - данные, которые будут в нем доступны. В данном примере, мы передаем переменную `name`, которая отображается через `echo`.

Конечно, шаблоны могут бы вложены в поддиректории внутри `resources/views`. Для обращения к таким шаблонам, можно использовать точку в качестве разделителя. Например, если ваш шаблон находится в файле `resources/views/admin/profile.php`, то вызов хэлпера будет выглядеть так:

   return view('admin.profile', $data); 

#### Проверка существования шаблона

Если вам требуется проверить существование шаблона, можно использовать метод `exists` после вызова хэлпера `view` без аргументов. Данный метод вернет `true` если представление существует на диске:

    if (view()->exists('emails.customer')) {
        //
    }
    
Когда хэлпер `view` вызывается без аргументов, он возвращает имплементацию контракта `Illuminate\Contracts\View\Factory`, позволяя вам получить доступ к любым методам фабрики.

<a name="view-data"></a>
### Данные шаблона

<a name="passing-data-to-views"></a>
#### Передача данных в шаблон

Как вы видели в предыдущем примере, вы можете просто передать массив с вашими данными в шаблон:

    return view('greetings', ['name' => 'Victoria']);
    
Когда вы передаете данные таким способом, `$data` должна являться массивом пар ключ-значение. Внутри представления вы можете обратиться к любому значению по соответствующему ему ключу, через `<?php echo $key; ?>`. В качестве альтернативы передачи полного массива с данными в хэлпер `view` можно использовать метод `with`, для добавления необходимых данных независимо друг от друга:

    return view('greeting')->with('name', 'Victoria');
    
<a name="sharing-data-with-all-views"></a>
#### Передача данных во все шаблоны

Иногда вам может потребоваться сделать часть данных доступными для всех шаблонов вашего приложения. Это можно реализовать используя метод `share` фабрики шаблонов. Обычно вы будете размещать вызовы `share` внутри метода `boot` сервис-провайдера. Это можно делать как в `AppServiceProvider`, так и сгенерировать для этого отдельный сервис-провайдер:

    <?php

    namespace App\Providers;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            view()->share('key', 'value');
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

<a name="view-composers"></a>
## Композеры

Композеры (view composers) - функции-замыкания или методы класса, которые вызываются, когда шаблон рендерится в строку. Если у вас есть данные, которые вы хотите привязать к шаблону при каждом его рендеринге, то композеры помогут вам выделить такую логику в отдельное место. 

Давайте зарегистрируем свой композер в [сервис-провайдере](/docs/{{version}}/providers). Мы будем использовать функцию-хэлпер `view` для того, чтобы получить доступ к имплементации контракта `Illuminate\Contracts\View\Factory`. Помните, Laravel не содержит папку по умолчанию для композеров и вы вольны организовать их как вам захочется. К примеру, вы можете создать папку `App\Http\ViewComposers`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function boot()
        {
            // Using class based composers...
            view()->composer(
                'profile', 'App\Http\ViewComposers\ProfileComposer'
            );

            // Using Closure based composers...
            view()->composer('dashboard', function ($view) {
                //
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

Помните, если вы создадите отдельный сервис провайдер для ваших композеров, вам потребуется добавить его в массив `providers` в конфигурационном файле `config/app.php`.

Теперь у нас есть зарегистрированный композер - метод `ProfileComposer@compose`, который будет вызываться каждый раз перед тем, как шаблон `profile` будет рендерится в строку. Давайте определим его класс:

    <?php

    namespace App\Http\ViewComposers;

    use Illuminate\View\View;
    use App\Repositories\UserRepository;

    class ProfileComposer
    {
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
            // Dependencies automatically resolved by service container...
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

Перед тем как шаблон отрендерится в строку, будет вызван метод `compose` с экземпляром `Illuminate\View\View`. Вы можете использовать метод `with` для передачи в него данных.

> **Примечание:** Все композеры берутся из [сервис-контейнера](/docs/{{version}}/container), поэтому вы можете указать необходимые зависимости в конструкторе композера - они будут автоматически поданы ему.

#### Назначение композера для нескольких шаблонов

Вы можете назначить композер для нескольких шаблонов сразу, передав массив шаблонов в качестве первого аргумента метода `composer`:

    view()->composer(
        ['profile', 'dashboard'],
        'App\Http\ViewComposers\MyViewComposer'
    );
        
Метод `composer` так же принимает символ `*` в качестве wildcard (паттерна), позволяя вам назначать композер для всех шаблонов сразу:

    view()->composer('*', function ($view) {
        //
    });

### Создатель (view creators)

Создатели шаблонов работают почти так же, как композеры, но вызываются сразу после создания объекта шаблона, а не во время его рендеринга в строку. Для регистрации используйте метод `creator`:

    view()->creator('profile', 'App\Http\ViewCreators\ProfileCreator');
