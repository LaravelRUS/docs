git d0bfe4ef3fd2dbfd4499180caaec98281fa3d26b

---

# Расширение фреймворка

- [Managers и Factory](#managers-and-factories)
- [Кэш](#cache)
- [Сессии](#session)
- [Аутентификация](#authentication)
- [Расширения посредством IoC](#ioc-based-extension)

<a name="managers-and-factories"></a>
## Managers и Factory

Laravel содержит несколько классов `Manager`, которые управляют созданием компонентов, основанных на драйверах. Эти компоненты включают в себя кэш, сессии, авторизацию и очереди. Класс-управляющий ответственнен за создание конкретной реализации драйвера в зависимости от настроек приложения. Например, класс `CacheManager` может создавать объекты-реализации APC, Memcached, Native и различных других драйверов.

Каждый из этих управляющих классов имеет метод `extend`, который может использоваться для простого добавления новой реализации драйвера. Мы поговорим об этих управляющих классах ниже, с примерами о том, как добавить собственный драйвер в каждый из них.

> **Примечание:** посветите несколько минут изучению различных классов `Manager`, которые поставляются с Laravel, таких как `CacheManager` и `SessionManager`. Знакомство с их кодом поможет вам лучше понять внутреннюю работу Laravel. Все классы-управляющие наследуют базовый класс `Illuminate\Support\Manager`, который реализует общую полезную функциональность для каждого из них.

<a name="cache"></a>
## Cache

Для расширения подсистемы кэширования мы используем метод `extend` класса `CacheManager`, который используется для привязки стороннего драйвера к управляющему классу и является общим для всех таких классов. Например, для регистрации нового драйвера кэша с именем "mongo" нужно будет сделать следующее:

	Cache::extend('mongo', function($app)
	{
		return Cache::repository(new MongoStore);
	});

Первый параметр, передаваемый методу `extend` - имя драйвера. Это имя соответствует значению параметра `driver` файла настроек `config/cache.php`. Второй параметр - функция-замыкание, которая должна вернуть объект типа `Illuminate\Cache\Repository`. Замыкание получит параметр `$app` - объект `Illuminate\Foundation\Application`, IoC-контейнер.

Этот вызов `Cache::extend` можно делать в методе `boot()` сервис-провайдера `App\Providers\AppServiceProvider`, или другого вашего сервис-провайдера.

Для создания стороннего драйвера для кэша мы начнём с реализации интерфейса `Illuminate\Contracts\Cache\Store`. Итак, наша реализация MongoDB будет выглядеть примерно так:

	class MongoStore implements Illuminate\Contracts\Cache\Store {

		public function get($key) {}
		public function put($key, $value, $minutes) {}
		public function increment($key, $value = 1) {}
		public function decrement($key, $value = 1) {}
		public function forever($key, $value) {}
		public function forget($key) {}
		public function flush() {}

	}

Нам только нужно реализовать каждый из этих методов с использованием подключения к MongoDB. Как только мы это сделали, можно закончить регистрацию нового драйвера:

	Cache::extend('mongo', function($app)
	{
		return Cache::repository(new MongoStore);
	});

Если вы задумались о том, куда поместить ваш новый дравер - подумайте о том, чтобы сделать отдельный модуль и распространять его через Packagist. Либо вы можете создать пространство имён `Extensions` вместе с соответствующей папкой в папке `app`. Впрочем, так как Laravel не имеет жёсткой структуры папок, вы можете организовать свои файлы, как вам удобно.

<a name="session"></a>
## Сессии

Расширение системы сессий Laravel собственным драйвером так же просто, как и расширение драйвером кэша. Мы вновь используем метод `extend` для регистрации собственного кода:

	Session::extend('mongo', function($app)
	{
		// Вернуть объект, реализующий SessionHandlerInterface
	});

### Где расширять Session

Наиболее подходящее место - метод `boot` сервис-провайдера  `AppServiceProvider`.

### Написание расширения

Заметьте, что наш драйвер сессии должен реализовывать интерфейс `SessionHandlerInterface`. Он включен в ядро PHP 5.4+. Если вы используете PHP 5.3, то Laravel создаст его для вас, что позволит поддерживать совместимость будущих версий. Этот интерфейс содержит несколько простых методов, которые нам нужно написать. Заглушка драйвера MongoDB выглядит так:

	class MongoHandler implements SessionHandlerInterface {

		public function open($savePath, $sessionName) {}
		public function close() {}
		public function read($sessionId) {}
		public function write($sessionId, $data) {}
		public function destroy($sessionId) {}
		public function gc($lifetime) {}

	}	

Эти методы не так легки в понимании, как методы драйвера кэша (`StoreInterface`), поэтому давайте пробежимся по каждому из них подробнее:

- Метод `open` обычно используется при открытии системы сессий, основанной на файлах. Laravel поставляется с драйвером `native`, который использует стандартное файловое хранилище PHP, вам почти никогда не понадобится добавлять что-либо в этот метод. Вы можете всегда оставить его пустым. Фактически, это просто тяжелое наследие плохого дизайна PHP, из-за которого мы должны написать этот метод (мы обсудим это ниже).
- Метод `close`, аналогично методу `open`, обычно также игнорируется. Для большей части драйверов он не требуется.
- Метод `read` должен вернуть строку - данные сессии, связанные с переданным `$sessionId`. Нет необходимости сериализовать объекты или делать какие-то другие преобразования при чтении или записи данных сессии в вашем драйвере - Laravel делает это автоматически.
- Метод `write` должен связать строку `$data` с данными сессии с переданным идентификатором `$sessionId`, сохранив её в каком-либо постоянном хранилище, таком как MongoDB, Dynamo и др.
- Метод `destroy` должен удалить все данные, связанные с переданным `$sessionId`, из постоянного хранилища.
- Метод `gc` должен удалить все данные, которые старее переданного `$lifetime` (unixtime секунд). Для самоочищающихся систем вроде Memcached и Redis этот метод может быть пустым.

Как только интерфейс `SessionHandlerInterface` реализован, мы готовы к тому, чтобы зарегистрировать новый драйвер в управляющем классе `Session`:

	Session::extend('mongo', function($app)
	{
		return new MongoHandler;
	});

Как только драйвер сессии зарегистрирован мы можем использовать его имя `mongo` в нашем файле настроек `config/session.php`.

> **Подсказка:** если вы написали новый драйвер сессии, поделитесь им на Packagist!

<a name="authentication"></a>
## Аутентификация

Механизм аутентификации может быть расширен тем же способом, что и кэш и сессии. Мы используем метод `extend`, с которым вы уже знакомы:

	Auth::extend('riak', function($app)
	{
		// Вернуть объект, реализующий Illuminate\Contracts\Auth\UserProvider
	});

Реализация `UserProvider` ответственна только за то, чтобы получать нужный объект, реализующий `Illuminate\Contracts\Auth\Authenticatable` из постоянного хранилища, такого как MySQL, Riak и др. Эти два интерфейса позволяют работать механизму авторизации Laravel вне зависимости от того, как хранятся пользовательские данные и какой класс используется для их представления.

Давайте посмотрим на контракт `UserProvider`:

	interface UserProvider {

		public function retrieveById($identifier);
		public function retrieveByToken($identifier, $token);
		public function updateRememberToken(Authenticatable $user, $token);
		public function retrieveByCredentials(array $credentials);
		public function validateCredentials(Authenticatable $user, array $credentials);

	}

Метод `retrieveById` обычно получает числовой ключ, идентифицирующий пользователя - такой, как первичный ключ в MySQL. Метод должен возвращать объект, реализующий  `Authenticatable`, соответствующий переданному ID.


Метод `retrieveByToken` запрашивает пользователя по уникальному `$identifier` и "запомнить меня"-токену, который хранится в столбце `remember_token`. Метод должен возвращать объект, реализующий `Authenticatable`

The `updateRememberToken` method updates the `$user` field `remember_token` with the new `$token`. The new token can be either a fresh token, assigned on successfull "remember me" login attempt, or a null when user is logged out.

Метод `updateRememberToken` обновляет у `$user` поле `remember_token` новым `$token`. 

Метод `retrieveByCredentials` получает массив данных, ктоорые были переданы методу `Auth::attempt` при попытке входа в систему. Этот метод должен запросить своё постоянное хранилище на наличие пользователя с совпадающими данными. Обычно этот метод выполнит SQL-запрос с проверкой на `$credentails['username']`. **Этот метод не должен производить сравнение паролей или выполнять вход.**

Метод `validateCredentials` должен сравнить переданный объект пользователя `$user` с данными для входа `$credentials` для того, чтобы его авторизовать. К примеру, этот метод может сравнивать строку `$user->getAuthPassword()` с результатом вызова `Hash::make` а строке `$credentials['password']`.

Теперь, когда мы узнали о каждом методе интерфейса `UserProviderInterface`давайте посмотрим на интерфейс `Authenticatable`. Как вы помните, поставщик должен вернуть реализацию этого интерфейса из своих методов `retrieveById` и `retrieveByCredentials`.

	interface Authenticatable {

		public function getAuthIdentifier();
		public function getAuthPassword();
		public function getRememberToken();
		public function setRememberToken($value);
		public function getRememberTokenName();

	}

Это простой интерфейс. Метод `getAuthIdentifier` должен просто вернуть "первичный ключ" пользователя. Если используется хранилище MySQL, то это будет автоматическое числовое поле - первичный ключ. Метод `getAuthPassword` должен вернуть хэшированный пароль. Этот интерфейс позволяет системе авторизации работать с любым классом пользователя, вне зависимости от используемой ORM или хранилища данных. Изначально Laravel содержит класс `User` в папке `app` который реализует этот интерфейс, поэтому мы можете обратиться к этому классу, чтобы увидеть пример реализации.

Наконец, как только мы написали класс-реализацию `UserProvider`, у нас готово для регистрации расширения в фасаде `Auth`:

	Auth::extend('riak', function($app)
	{
		return new RiakUserProvider($app['riak.connection']);
	});

Когда вы зарегистрировали драйвер методом `extend` вы можете активировать него в вашем файле настроек `config/auth.php`.

<a name="ioc-based-extension"></a>
## Расширения посредством IoC

Почти каждый сервис-провайдер Laravel получает свои объекты из контейнера IoC. Вы можете увидеть список сервис-провайдеров в вашем приложения в файле `config/app.php`. Вам стоит пробежаться по коду каждого из поставщиков в свободное время - сделав это вы получите намного более чёткое представление о том, какую именно функционалньость каждый из них добавляет к фреймворку, а также какие ключи используются для регистрации различных услуг в контейнере IoC.

Например, `HashServiceProvider` использует ключ `hash` для получения экземпляра `Illuminate\Hashing\BcryptHasher` из контейнера IoC. Вы можете легко расширить и перекрыть этот класс в вашем приложении, перекрыв эту привязку. Например:

	<?php namespace App\Providers;

	class SnappyHashProvider extends \Illuminate\Hashing\HashServiceProvider {

		public function boot()
		{
			$this->app->bindShared('hash', function()
			{
				return new \Snappy\Hashing\ScryptHasher;
			});

			parent::boot();
		}

	}

Заметьте, что этот класс расширяет `HashServiceProvider`, а не базовый класс `ServiceProvider`. Как только вы расширили этот сервис-провайдер, измените `HashServiceProvider` в файле настроек `config/app.php` на имя вашего нового сервис-провайдера.

Это общий подход к расширению любого класса ядра, который привязан к IoC-контейнеру. Фактически каждый класс так или иначе привязан к нему, и с помощью вышеописанного метода может быть перекрыт. Опять же, посмотрев код сервис-провайдеров фреймворка, вы познакомитесь с тем, где различные классы привязываются к IoC-контейнеру и какие ключи для этого используются. Это отличный способ понять глубинную работу Laravel.
