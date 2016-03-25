git ce0ff9bad2c550dd52d8a9219fbd499dd3a6794e

---
# Envoy Task Runner

- [Введение](#introduction)
- [Написание задач](#writing-tasks)
	- [Переменные задач](#task-variables)
	- [Множественные серверы](#envoy-multiple-servers)
	- [Макросы задач](#envoy-task-macros)
- [Запуск задач](#envoy-running-tasks)
- [Уведомления](#envoy-notifications)
	- [HipChat](#hipchat)
	- [Slack](#slack)

<a name="introduction"></a>
## Введение

[Laravel Envoy](https://github.com/laravel/envoy) предоставляет четкий, минимальный синтаксис для определения типичных задач, которые Вы запускаете на Ваших удаленных серверах. Используя синтаксис в стиле Blade, Вы можете легко настроить задачи для деплоя, команды Artisan, и многое другое. На данный момент, Envoy поддерживает только операционные системы Mac и Linux.

<a name="envoy-installation"></a>
### Установка

Первым делом, установите Envoy используя `global`-команду Composer:

	composer global require "laravel/envoy=~1.0"

Убедитесь, что директория `~/.composer/vendor/bin` находится в PATH чтобы ОС имела возможность найти исполняемый файл `envoy`, когда Вы напишете команду `envoy` в своем терминале.

#### Обновление Envoy

Вы также можете использовать Composer, чтобы обновлять Envoy до последней версии:

	composer global update

<a name="writing-tasks"></a>
## Написание задач

Все Ваши задачи Envoy должны быть определены внутри файла `Envoy.blade.php` в корне проекта. Вот пример конфигурации:

	@servers(['web' => '192.168.1.1'])

	@task('foo', ['on' => 'web'])
		ls -la
	@endtask

Как Вы можете видеть, массив `@servers` определен в верхней части файла, позволяя Вам указывать сервера в опции `on` описания задачи. Внутри декларации `@task` Вы можете задать Bash-код, который будет выполнен на удаленном сервере при исполнении задачи.

#### Предзагрузка

Иногда Вам может потребоваться выполнить некоторый PHP-код перед исполнением задач Envoy. Вы можете использовать директиву ```@setup```, чтобы объявить переменные и выполнить PHP-код, внутри файла Envoy:

	@setup
		$now = new DateTime();

		$environment = isset($env) ? $env : "testing";
	@endsetup

Вы можете также использовать ```@include```, для подключения внешних PHP-файлов:

	@include('vendor/autoload.php');

#### Подтверждение задач

Если Вы хотели бы видеть запрос подтверждения перед выполнением определенной задачи на Ваших серверах, Вы можете добавить директиву `confirm` в декларацию задачи:

	@task('deploy', ['on' => 'web', 'confirm' => true])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

<a name="task-variables"></a>
### Переменные задач

При необходимости Вы можете передавать переменные в файл Envoy, используя параметры командной строки для кастомизации своих задач:

	envoy run deploy --branch=master

Вы можете использовать опции в Ваших задачах посредством "echo"-синтаксиса Blade:

	@servers(['web' => '192.168.1.1'])

	@task('deploy', ['on' => 'web'])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

<a name="envoy-multiple-servers"></a>
### Множественные серверы

Вы можете без труда запустить задачу на нескольких серверах. Для начала, добавьте дополнительные серверы в декларацию `@servers`. Каждому серверу должно быть назначено уникальное имя. После определения дополнительных серверов, просто укажите список серверов в массиве `on` внутри декларации задачи:

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2']])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

По умолчанию задачи исполняются последовательно. То есть, задача завершится на первом сервере до начала ее выполнения на втором.

#### Параллельное выполнение
Для параллельного запуска задачи на нескольких серверах, добавьте опцию `parallel` в декларацию задачи:

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

<a name="envoy-task-macros"></a>
### Макросы задач

Макросы позволяют Вам выполнять несколько задач по очереди, используя одну команду. К примеру, макрос `deploy` может запускать задачи `git` и `composer`:

	@servers(['web' => '192.168.1.1'])

	@macro('deploy')
		git
		composer
	@endmacro

	@task('git')
		git pull origin master
	@endtask

	@task('composer')
		composer install
	@endtask

Как только макрос определен, Вы можете запустить его, используя одну простую команду:

	envoy run deploy

<a name="envoy-running-tasks"></a>
## Запуск задач

Чтобы запустить задачу из файла `Envoy.blade.php`, Выполните команду Envoy `run` с указанием имени команды или макроса, который Вы хотите запустить. Envoy выполнит задачу и отобразит вывод с серверов по ходу выполнения:

	envoy run task

<a name="envoy-notifications"></a>
<a name="envoy-hipchat-notifications"></a>
## Уведомления

<a name="hipchat"></a>
### HipChat

После запуска задачи, Вы можете отправить уведомление в чат-комнату своей команды на HipChat используя директиву под названием `@hipchat`. Она принимает API токен, имя комнаты и имя пользователя, которое будет использоваться в качестве имени отправителя:

	@servers(['web' => '192.168.1.1'])

	@task('foo', ['on' => 'web'])
		ls -la
	@endtask

	@after
		@hipchat('token', 'room', 'Envoy')
	@endafter

По желанию, Вы можете также передать произвольное сообщение для отправки в комнату HipChat. При составлении сообщения Вам будут доступны любые переменные, определенные в файле Envoy.

	@after
		@hipchat('token', 'room', 'Envoy', "Команда {$task} была запущена в окружении {$env}.")
	@endafter

<a name="slack"></a>
### Slack

Вдобавок к HipChat, Envoy также поддерживает отправку уведомлений в [Slack](https://slack.com). Директива `@slack` принимает URL хука, имя канала и сообщение для отправки на канал:

	@after
		@slack('hook', 'channel', 'message')
	@endafter

Вы можете получить URL вебхука при создании интеграции `Incoming WebHooks` на сайте Slack. Аргумент `hook` должен представлять из себя URL, полученную от этой интеграции. Например:

	https://hooks.slack.com/services/ZZZZZZZZZ/YYYYYYYYY/XXXXXXXXXXXXXXX

В качестве аргумента `channel` Вы можете передать:

- Для отправки сообщения на канал: `#channel`
- Для отправки уведомления пользователю: `@user`
 
