git 07431a1c1064c05eeb7e54f6536e28e6604a9a25

---

# Инструкция по обновлению

- [Обновление до 5.0 с 4.2](#upgrade-5.0)
- [Обновление до 4.2 с 4.1](#upgrade-4.2)
- [Обновление до 4.1.29 с <= 4.1.x](#upgrade-4.1.29)
- [Обновление до 4.1.26 с <= 4.1.25](#upgrade-4.1.26)
- [Обновление до 4.1 с 4.0](#upgrade-4.1)

<a name="upgrade-5.0"></a>
## Обновление до 5.0 с 4.2

### Быстрое обновление с использованием LegacyServiceProvider

Laravel 5.0 представляет новую надежную структуру папок. Но если вы хотите обновить приложение до Laravel 5.0 сохраняя структуру Laravel 4.2, вы можете использовать `Illuminate\Foundation\Providers\LegacyStructureServiceProvider`. Чтобы выполнить обновление до Laravel 5.0 с помощью этого провайдера, вы должны сделать следующее:

**1.** Обновите зависимость в `composer.json`  от `laravel/framework` до версии `5.0.*`.

**2.** Запустите `composer update --no-scripts`.

**3.** Добавьте `Illuminate\Foundation\Providers\LegacyStructureServiceProvider` в массив `providers` в файле `app/config/app.php`.

**4.** Удалите `Illuminate\Session\CommandsServiceProvider`, `Illuminate\Routing\ControllerServiceProvider` и `Illuminate\Workbench\WorkbenchServiceProvider` из массива `providers` в файле `app/config/app.php`.

**5.** Добавьте слудующие пути в конец файла `bootstrap/paths.php`:

	'commands' => __DIR__.'/../app/commands',
	'config' => __DIR__.'/../app/config',
	'controllers' => __DIR__.'/../app/controllers',
	'database' => __DIR__.'/../app/database',
	'filters' => __DIR__.'/../app/filters',
	'lang' => __DIR__.'/../app/lang',
	'providers' => __DIR__.'/../app/providers',
	'requests' => __DIR__.'/../app/requests',

После того, как эти изменения были сделаны, вы можете запустить ваше приложение. Тем не менее необходимо продолжить процедуру обновления.

### Compile Configuration File

Файл конфигурации `app/config/compile.php` теперь имеет следующий формат:

	<?php

	return [

		'files' => [
			//
		],

		'providers' => [
			//
		],

	];

Новая опция `providers` позволяет перечислить сервис-провайдеры, которые возвращают массивы файлов от их метода `compiles`.

### Beanstalk Queuing

Laravel 5.0 теперь требуется  `"pda/pheanstalk": "~3.0"` вместо `"pda/pheanstalk": "~2.1"` как это было в Laravel 4.2.

<a name="upgrade-4.2"></a>
## Обновление до 4.2 с 4.1

### PHP 5.4+

Для Laravel 4.2 необходима версия PHP 5.4.0 или более новая.

### Encryption Defaults

Добавьте новую опцию `cipher` в файле конфигурации `app/config/app.php`. Значение этой опции должно быть `MCRYPT_RIJNDAEL_256`.

	'cipher' => MCRYPT_RIJNDAEL_256

Этот параметр может быть использован для управления шифрованием, используемый по умолчанию в Laravel.

> **Примечание:** в Laravel 4.2 шифрованием по умолчанию является `MCRYPT_RIJNDAEL_128` (AES), Который считается самым безопасным. Изменение шифрования обратно на `MCRYPT_RIJNDAEL_256` является необходимым для расшифровки cookies/values, которые были использованы в Laravel <= 4.1

### Soft Deleting Models Now Use Traits

If you are using soft deleting models, the `softDeletes` property has been removed. Теперь вы должны использовать `SoftDeletingTrait`, например:

	use Illuminate\Database\Eloquent\SoftDeletingTrait;

	class User extends Eloquent {
		use SoftDeletingTrait;
	}

Вы также должны вручную добавить `deleted_at` в `dates`:

	class User extends Eloquent {
		use SoftDeletingTrait;

		protected $dates = ['deleted_at'];
	}

API для всех мягких операций удаления остается таким же.

> **Примечание:**  `SoftDeletingTrait` не могут быть применены на базовой модели. Он должен быть использован на модели класса.

### View / Pagination Environment Renamed

If you are directly referencing the `Illuminate\View\Environment` class or `Illuminate\Pagination\Environment` class, update your code to reference `Illuminate\View\Factory` and `Illuminate\Pagination\Factory` instead. These two classes have been renamed to better reflect their function.

### Additional Parameter On Pagination Presenter

If you are extending the `Illuminate\Pagination\Presenter` class, the abstract method `getPageLinkWrapper` signature has changed to add the `rel` argument:

	abstract public function getPageLinkWrapper($url, $page, $rel = null);

### Iron.Io Queue Encryption

If you are using the Iron.io queue driver, you will need to add a new `encrypt` option to your queue configuration file:

    'encrypt' => true

<a name="upgrade-4.1.29"></a>
## Upgrading To 4.1.29 From <= 4.1.x

Laravel 4.1.29 improves the column quoting for all database drivers. This protects your application from some mass assignment vulnerabilities when **not** using the `fillable` property on models. If you are using the `fillable` property on your models to protect against mass assignment, your application is not vulnerable. However, if you are using `guarded` and are passing a user controlled array into an "update" or "save" type function, you should upgrade to `4.1.29` immediately as your application may be at risk of mass assignment.

To upgrade to Laravel 4.1.29, simply `composer update`. No breaking changes are introduced in this release.

<a name="upgrade-4.1.26"></a>
## Upgrading To 4.1.26 From <= 4.1.25

Laravel 4.1.26 introduces security improvements for "remember me" cookies. Before this update, if a remember cookie was hijacked by another malicious user, the cookie would remain valid for a long period of time, even after the true owner of the account reset their password, logged out, etc.

This change requires the addition of a new `remember_token` column to your `users` (or equivalent) database table. After this change, a fresh token will be assigned to the user each time they login to your application. The token will also be refreshed when the user logs out of the application. The implications of this change are: if a "remember me" cookie is hijacked, simply logging out of the application will invalidate the cookie.

### Upgrade Path

First, add a new, nullable `remember_token` of VARCHAR(100), TEXT, or equivalent to your `users` table.

Next, if you are using the Eloquent authentication driver, update your `User` class with the following three methods:

	public function getRememberToken()
	{
		return $this->remember_token;
	}

	public function setRememberToken($value)
	{
		$this->remember_token = $value;
	}

	public function getRememberTokenName()
	{
		return 'remember_token';
	}

> **Note:** All existing "remember me" sessions will be invalidated by this change, so all users will be forced to re-authenticate with your application.

### Package Maintainers

Two new methods were added to the `Illuminate\Auth\UserProviderInterface` interface. Sample implementations may be found in the default drivers:

	public function retrieveByToken($identifier, $token);

	public function updateRememberToken(UserInterface $user, $token);

The `Illuminate\Auth\UserInterface` also received the three new methods described in the "Upgrade Path".

<a name="upgrade-4.1"></a>
## Upgrading To 4.1 From 4.0

### Upgrading Your Composer Dependency

To upgrade your application to Laravel 4.1, change your `laravel/framework` version to `4.1.*` in your `composer.json` file.

### Replacing Files

Replace your `public/index.php` file with [this fresh copy from the repository](https://github.com/laravel/laravel/blob/master/public/index.php).

Replace your `artisan` file with [this fresh copy from the repository](https://github.com/laravel/laravel/blob/master/artisan).

### Adding Configuration Files & Options

Update your `aliases` and `providers` arrays in your `app/config/app.php` configuration file. The updated values for these arrays can be found [in this file](https://github.com/laravel/laravel/blob/master/app/config/app.php). Be sure to add your custom and package service providers / aliases back to the arrays.

Add the new `app/config/remote.php` file [from the repository](https://github.com/laravel/laravel/blob/master/app/config/remote.php).

Add the new `expire_on_close` configuration option to your `app/config/session.php` file. The default value should be `false`.

Add the new `failed` configuration section to your `app/config/queue.php` file. Here are the default values for the section:

	'failed' => array(
		'database' => 'mysql', 'table' => 'failed_jobs',
	),

**(Optional)** Update the `pagination` configuration option in your `app/config/view.php` file to `pagination::slider-3`.

### Controller Updates

If `app/controllers/BaseController.php` has a `use` statement at the top, change `use Illuminate\Routing\Controllers\Controller;` to `use Illuminate\Routing\Controller;`.

### Password Reminders Updates

Password reminders have been overhauled for greater flexibility. You may examine the new stub controller by running the `php artisan auth:reminders-controller` Artisan command. You may also browse the [updated documentation](/docs/security#password-reminders-and-reset) and update your application accordingly.

Update your `app/lang/en/reminders.php` language file to match [this updated file](https://github.com/laravel/laravel/blob/master/app/lang/en/reminders.php).

### Environment Detection Updates

For security reasons, URL domains may no longer be used to detect your application environment. These values are easily spoofable and allow attackers to modify the environment for a request. You should convert your environment detection to use machine host names (`hostname` command on Mac, Linux, and Windows).

### Simpler Log Files

Laravel now generates a single log file: `app/storage/logs/laravel.log`. However, you may still configure this behavior in your `app/start/global.php` file.

### Removing Redirect Trailing Slash

In your `bootstrap/start.php` file, remove the call to `$app->redirectIfTrailingSlash()`. This method is no longer needed as this functionality is now handled by the `.htaccess` file included with the framework.

Next, replace your Apache `.htaccess` file with [this new one](https://github.com/laravel/laravel/blob/master/public/.htaccess) that handles trailing slashes.

### Current Route Access

The current route is now accessed via `Route::current()` instead of `Route::getCurrentRoute()`.

### Composer Update

Once you have completed the changes above, you can run the `composer update` function to update your core application files! If you receive class load errors, try running the `update` command with the `--no-scripts` option enabled like so: `composer update --no-scripts`.

### Wildcard Event Listeners

The wildcard event listeners no longer append the event to your handler functions parameters. If you require finding the event that was fired you should use `Event::firing()`.
