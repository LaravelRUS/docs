git fc9a35231f270630d6692c7d4e91f4a023895029

---

# SSH

- [Настройка](#configuration)
- [Использование ssh](#basic-usage)
- [Задачи](#tasks)
- [SFTP загрузка](#sftp-downloads)
- [SFTP аплоад](#sftp-uploads)
- [Показ логов](#tailing-remote-logs)
- [Envoy](#envoy-task-runner)

<a name="configuration"></a>
## Настройка

В состав Laravel входит библиотека для коннекта к серверам по ssh и исполнения там команд, что позволяет писать artisan-команды для работы на удаленных серверах. Для работы с этой библиотекой Laravel предоставляет фасад `SSH`.

Файл настроек этой библиотеки - `app/config/remote.php`. Массив `connections` содержит список доступных серверов. Доступ к серверу может осуществляться по паролю или ключу.

> **Примечание:** Если вам надо запускать разнообразные задачи на своем сервере, попробуйте [Envoy](#envoy-task-runner).

<a name="basic-usage"></a>
## Использование ssh

#### Запуск команды на дефолтном сервере

	SSH::run(array(
		'cd /var/www',
		'git pull origin master',
	));

#### Запуск команды на указанном сервере

	SSH::into('staging')->run(array(
		'cd /var/www',
		'git pull origin master',
	));

#### Получение ответа 

Вы можете ловить текстовый вывод исполненных команд функцией-замыканием, переданной вторым аргументом:

	SSH::run($commands, function($line)
	{
		echo $line.PHP_EOL;
	});

<a name="tasks"></a>
## Задачи

Вы можете объединять несколько команд в т.н. задачу:

	SSH::into('staging')->define('deploy', array(
		'cd /var/www',
		'git pull origin master',
		'php artisan migrate',
	));

Когда задача задана, вы можете исполнить её:

	SSH::into('staging')->task('deploy', function($line)
	{
		echo $line.PHP_EOL;
	});

<a name="sftp-downloads"></a>
## SFTP загрузка

Класс `SSH` позволяет загрузить файл по SFTP, при помощи методов `get` и `getString`.

	SSH::into('staging')->get($remotePath, $localPath);

	$contents = SSH::into('staging')->getString($remotePath);

<a name="sftp-uploads"></a>
## SFTP аплоад

Таже вы можете закачивать файлы на удаленный сервер по SFTP:

	SSH::into('staging')->put($localFile, $remotePath);

	SSH::into('staging')->putString($remotePath, 'Foo');

<a name="tailing-remote-logs"></a>
## Показ логов

Laravel имеет удобное средство для просмотра последних изменений лог-файлов на удаленных серверах - т.н. tailing, когда на экран выводится только то, что добавилось в конец файла. Просто укажите после artisan-команды tail имя удаленного сервера.

	php artisan tail staging

	php artisan tail staging --path=/path/to/log.file

<a name="envoy-task-runner"></a>
## Envoy 

- [Установка](#envoy-installation)
- [Запуск задач](#envoy-running-tasks)
- [Запуск на нескольких серверах](#envoy-multiple-servers)
- [Паралельное выполнение](#envoy-parallel-execution)
- [Макросы](#envoy-task-macros)
- [Уведомления](#envoy-notifications)
- [Обновление Envoy](#envoy-updating-envoy)

Envoy - это инструмент для запуска задач на удаленных серверах. Он предоставляет простой синтаксис для записи операций деплоя, запуска artisan-команд и т.п., который базируется на синтаксисе [Blade](/docs/{{version}}/templates#blade-templating).

> **Примечание:** Envoy требует PHP 5.4 и выше, запускается на Mac или Linux.

<a name="envoy-installation"></a>
### Установка

1. Установите Envoy глобально в системе:

	composer global require "laravel/envoy=~1.0"

Проверьте, чтобы `~/.composer/vendor/bin` был у вас в PATH. Для проверки наберите `envoy` в терминале - должен вывестись краткий хелп. 

2. Создайте `Envoy.blade.php` в корне вашего проекта. Например, задача ожет быть такой:

	@servers(['web' => '192.168.1.1'])

	@task('foo', ['on' => 'web'])
		ls -la
	@endtask

Директива `@servers` задает список доступных серверов. Внутри `@task` располагаются bash-команды. На каком сервере запускать задачу говорит параметр 'on'.

Команда `init` создает заготовку envoy-файла:

	envoy init user@192.168.1.1

<a name="envoy-running-tasks"></a>
### Запуск задач

Для запуска задач служит команда `run`

	envoy run foo

Вы можете передать переменную в вашу задачу:

	envoy run deploy --branch=master

В самой задаче она доступна как переменная в blade-шаблоне, в фигурных скобках:

	@servers(['web' => '192.168.1.1'])

	@task('deploy', ['on' => 'web'])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

#### Инициализация

Вы можете использовать директиву `@setup` для объявления переменных. Поддерживается синтаксис PHP.

	@setup
		$now = new DateTime();

		$environment = isset($env) ? $env : "testing";
	@endsetup

Вы также можете использовать директиву `@include` чтобы подключить любой php-файл.

	@include('vendor/autoload.php');

<a name="envoy-multiple-servers"></a>
### Запуск на нескольких серверах

Вы можете запустить выполнение задачи на нескольких серверах, задав их имена в массиве 'on':

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2']])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

По умолчанию задачи выполняются последовательно. Пока не закончится выполнение задачи на одном сервере, выполнение на втором не начнется.

<a name="envoy-parallel-execution"></a>
### Параллельное выполнение

Для одновременного запуска задач, установите параметр `'parallel'` в `true`.

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

<a name="envoy-task-macros"></a>
### Макросы

Макросы позволяют объединять несколько `@task`:

	@servers(['web' => '192.168.1.1'])

	@macro('deploy')
		foo
		bar
	@endmacro

	@task('foo')
		echo "HELLO"
	@endtask

	@task('bar')
		echo "WORLD"
	@endtask

Полученный макрос `deploy` может быть вызван как обычная команда:

	envoy run deploy

<a name="envoy-notifications"></a>
### Уведомления

После выполнения задачи envoy может послать уведомление об этом. В данный момент поддерживается только чат для разработчиков [HipChat](https://www.hipchat.com). 

Для отсылки нотификации в чат групповой разработки [HipChat](https://www.hipchat.com) служит директива `@hipchat`.

	@servers(['web' => '192.168.1.1'])

	@task('foo', ['on' => 'web'])
		ls -la
	@endtask

	@after
		@hipchat('token', 'room', 'Envoy')
	@endafter

Чтобы изменить сообщение, вы можете задействовать переменные, объявленные в директиве `@setup`, или взятые из подключенного `@include` php-файла:

	@after
		@hipchat('token', 'room', 'Envoy', "$task ran on [$environment]")
	@endafter

#### Slack

Для отсылки уведомления в [Slack](https://slack.com) служит директива `@slack`:

	@after
		@slack('team', 'token', 'channel')
	@endafter

<a name="envoy-updating-envoy"></a>
### Обновление Envoy

Для обновления Envoy используйте команду `self-update`

	envoy self-update

Если вы установили envoy в `/usr/local/bin`, используйте `sudo`

	sudo envoy self-update





