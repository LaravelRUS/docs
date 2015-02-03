git f421873828fd993cd8e15a24f443d202b76df813

---

# Хэширование

- [Введение](#introduction)
- [Основы использования](#basic-usage)

<a name="introduction"></a>
## Введение

Фасад `Hash` предоставляет механизм для защищённого хэширования паролей алгоритмом Bcrypt.
Если вы используете контроллер `AuthController`, идущий «из коробки», то он уже позаботился о сверке сохранённого
хэша пароля и пароля, введённого пользователем, используя для этого трейт `AuthenticatesAndRegistersUsers` и сервис `Registrar`.


<a name="basic-usage"></a>
## Основы использования

#### Хэширование пароля

	$password = Hash::make('secret');

Так же можно использовать функцию-хелпер `bcrypt`:

	$password = bcrypt('secret');

#### Сверка пароля и хэша

	if (Hash::check('secret', $hashedPassword))
	{
		// Пароль верен...
	}

#### Провера не необходимость рехэша пароля

	if (Hash::needsRehash($hashed))
	{
		$hashed = Hash::make('secret');
	}
