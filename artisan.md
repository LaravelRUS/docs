git 4a83b82d8e8a6216ab0623aa4db9dd006bbead6b

---

# Интерфейс командной строки Artisan

- [Введение](#introduction)
- [Использование](#usage)
- [Calling Commands Outside Of CLI](#calling-commands-outside-of-cli)
- [Scheduling Artisan Commands](#scheduling-artisan-commands)

<a name="introduction"></a>
## Введение

Artisan - название интерфейса командной строки, входящей в состав Laravel. Он предоставляет полезные команды для использования во время разработки вашего приложения. Работает на основе мощного компонента Symfony Console.

<a name="usage"></a>
## Использование

#### Вывод всех доступных команд

Чтобы вывести все доступные команды Artisan, используйте команду `list`:

	php artisan list

#### Просмотр помощи для команды

Каждая команда также включает и инструкцию, которая отображает и описывает доступные аргументы и опции для команды. Чтобы её вывести, необходимо добавить слово `help` перед командой:

	php artisan help migrate

#### Использование среды

Вы также можете указать среду, в которой будет выполнена команда при помощи опции `--env`:

	php artisan migrate --env=local

#### Отображение используемой версии Laravel

Вы также можете увидеть версию Laravel вашего приложения используя опцию `--version`:

	php artisan --version

<a name="calling-commands-outside-of-cli"></a>
## Вызов команды из приложения

Иногда может потребоваться выполнить команду из вашего приложения, например, в обработчике роута или в контроллере. Для этого используется фасад `Artisan`:

	Route::get('/foo', function()
	{
		$exitCode = Artisan::call('command:name', ['--option' => 'foo']);

		//
	});

Вы можете добавить команду в очередь, так что она будет выполняться в фоне [менеджером очереди](/docs/master/queues):

	Route::get('/foo', function()
	{
		Artisan::queue('command:name', ['--option' => 'foo']);

		//
	});

<a name="scheduling-artisan-commands"></a>
## Scheduling Artisan Commands

Раньше разработчикам приходилось добавлять задание в Cron для каждой консольной команды и это была большая головная боль. Давайте сделаем нашу жизнь проще.
Планировщик заданий Laravel позволяет гибко и просто составлять расписание запуска ваших команд из самого приложения, и для этго потребуется добавить всего одно задание в Cron.

Ваш планировщик находится в файле `app/Console/Kernel.php`. В классе `Kernel` вы увидите метод `schedule`, который уже содержит в себе простой пример.
Вы можете добавить сколько угодно заданий, используя объект `Schedule`.

Но сначала нужно добавить простое задание в Cron:

	* * * * * php /path/to/artisan schedule:run 1>> /dev/null 2>&1

Это Cron-задание указывает, что нужно вызывать планировщик заданий Laravel каждую минуту, который уже и будет запускать необходимые задания.

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

#### Частое выполнение

	$schedule->command('foo')->everyFiveMinutes();

	$schedule->command('foo')->everyTenMinutes();

	$schedule->command('foo')->everyThirtyMinutes();

#### Ежедневное выполнение

	$schedule->command('foo')->daily();

#### Ежедневное выполнение с запуском в определённое время (24-часовой формат времени)

	$schedule->command('foo')->dailyAt('15:00');

#### Выполннеие два раза в день

	$schedule->command('foo')->twiceDaily();

#### Выполнение каждый день, кроме выходных

	$schedule->command('foo')->weekdays();

#### Еженедельное выполнение

	$schedule->command('foo')->weekly();

	// Можно указать время выполнения для каждого дня (0-6)...
	$schedule->command('foo')->weeklyOn(1, '8:00');

#### Ежемесячное выполнение

	$schedule->command('foo')->monthly();

#### Выполнение только в определённой среде исполнения

	$schedule->command('foo')->monthly()->environments('production');

#### Выполнять, даже если приложение в режиме обслуживания

	$schedule->command('foo')->monthly()->evenInMaintenanceMode();

#### Выполнять, но только если функция-параметр вернула `true`

	$schedule->command('foo')->monthly()->when(function()
	{
		return true;
	});
