git 6bfa0031c86b1c7c2f3b6df16420ec8a1ae85c35

---

# Интерфейс командной строки Artisan

- [Введение](#introduction)
- [Использование](#usage)
- [Вызов команд из приложения](#calling-commands-outside-of-cli)
- [Планировщик заданий](#scheduling-artisan-commands)

<a name="introduction"></a>
## Введение

Artisan - название интерфейса командной строки, входящей в состав Laravel. Он предоставляет полезные команды для использования во время разработки вашего приложения. Работает на основе мощного компонента Symfony Console.

<a name="usage"></a>
## Использование

#### Вывод всех доступных команд

Чтобы вывести все доступные команды Artisan, используйте команду `list`:

	php artisan list

#### Просмотр помощи для команды

Каждая команда также включает и инструкцию, которая отображает и описывает доступные аргументы и опции для команды. Для того, чтобы её вывести, просто добавьте слово `help` перед командой:

	php artisan help migrate

#### Запуск в заданной среде выполнения

Вы также можете указать среду выполнения, в которой будет выполнена команда, при помощи опции `--env`:

	php artisan migrate --env=local

#### Отображение используемой версии Laravel

Вы также можете увидеть версию Laravel вашего приложения используя опцию `--version`:

	php artisan --version

<a name="calling-commands-outside-of-cli"></a>
## Вызов команд из приложения

Иногда может потребоваться выполнить команду Artisan из вашего приложения, например, в обработчике роута или в контроллере.
Для этого используется фасад `Artisan`:

	Route::get('/foo', function()
	{
		$exitCode = Artisan::call('command:name', ['--option' => 'foo']);

		//
	});

Вы даже можете добавить команду в очередь для того, чтобы она выполнялась на фоне [менеджером очереди](/docs/{{version}}/queues):

	Route::get('/foo', function()
	{
		Artisan::queue('command:name', ['--option' => 'foo']);

		//
	});

<a name="scheduling-artisan-commands"></a>
## Планировщик заданий

Раньше разработчикам приходилось добавлять задание в Cron для каждой консольной команды и это была большая головная боль. Давайте сделаем нашу жизнь проще. Планировщик заданий Laravel позволяет просто и гибко составлять расписание запуска ваших команд из самого приложения и для этого потребуется добавить всего одно Cron задание.

Ваш планировщик находится в файле `app/Console/Kernel.php`. В классе `Kernel` вы увидите метод `schedule`, который уже содержит в себе простой пример.
Вы можете добавить сколько угодно заданий, используя объект `Schedule`. Единственное Cron задание, которое нужно добавить на сервер:

	* * * * * php /path/to/artisan schedule:run 1>> /dev/null 2>&1

Это Cron задание вызывает планировщик заданий каждую минуту. Затем, Laravel просматривает задания и запускает необходимые. Проще некуда!

### Несколько примеров

Давайте рассмотрим несколько примеров использования планировщика:

#### Замыкание в качестве задания

	$schedule->call(function()
	{
		// Do some task...

	})->hourly();

#### Консольная команда в качестве задания

	$schedule->exec('composer self-update')->daily();

#### Добавление задания, используя синтаксис Cron

	$schedule->command('foo')->cron('* * * * *');

#### Постоянные задания

	$schedule->command('foo')->everyFiveMinutes();

	$schedule->command('foo')->everyTenMinutes();

	$schedule->command('foo')->everyThirtyMinutes();

#### Ежедневные задания

	$schedule->command('foo')->daily();

#### Ежедневные задания с запуском в определённое время (24-часовой формат времени)

	$schedule->command('foo')->dailyAt('15:00');

#### Задания, выполняемые дважды в день

	$schedule->command('foo')->twiceDaily();

#### Задания на каждый день, кроме выходных

	$schedule->command('foo')->weekdays();

#### Еженедельные задания

	$schedule->command('foo')->weekly();

	// Можно указать время выполнения для каждого дня (0-6)...
	$schedule->command('foo')->weeklyOn(1, '8:00');

#### Ежемесячные задания

	$schedule->command('foo')->monthly();

#### Запуск по дням недели

	$schedule->command('foo')->mondays();
	$schedule->command('foo')->tuesdays();
	$schedule->command('foo')->wednesdays();
	$schedule->command('foo')->thursdays();
	$schedule->command('foo')->fridays();
	$schedule->command('foo')->saturdays();
	$schedule->command('foo')->sundays();	

#### Выполнение задания только в определённой среде выполнения

	$schedule->command('foo')->monthly()->environments('production');

#### Выполнение задания, даже если приложение находится в режиме обслуживания

	$schedule->command('foo')->monthly()->evenInMaintenanceMode();

#### Выполнять, но только если функция-параметр вернула `true`

	$schedule->command('foo')->monthly()->when(function()
	{
		return true;
	});

#### Отправить вывод на email

	$schedule->command('foo')->emailOutputTo('foo@example.com');

#### Записать вывод в файл

	$schedule->command('foo')->sendOutputTo($filePath);

#### Дернуть url по завершении задачи

	$schedule->command('foo')->thenPing($url);