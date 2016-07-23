git 75651ca9298f282475c849265173f9180055fc19

---

# Хэширование

- [Введение](#introduction)
- [Основы использования](#basic-usage)

<a name="introduction"></a>
## Введение

[Фасад](/docs/{{version}}/facades) `Hash` предоставляет механизм для защищённого хэширования паролей алгоритмом Bcrypt. Если вы используете контроллер `AuthController`, идущий «из коробки», то он уже применяет Bcrypt для регистрации и аутентификации. 

> **Примечание:** Bcrypt является отличным выбором для хэширования паролей так как его "рабочий фактор" является настраиваемым, что означает, что время, требуемое для генерации хэша, может быть увеличено вместе с ростом вычислительных мощностей.

<a name="basic-usage"></a>
## Основы использования

Вы можете захэшировать пароль вызвав метод `make` фасада `Hash`:

    <?php

    namespace App\Http\Controllers;

    use Hash;
    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Update the password for the user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function updatePassword(Request $request, $id)
        {
            $user = User::findOrFail($id);

            // Validate the new password length...

            $user->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

Так же можно использовать функцию-хелпер `bcrypt`:

    bcrypt('plain-text');

#### Сверка пароля и хэша

Метод `check` позволяет проверить соответствие переданной в него исходной строки и хэша. Однако, если вы используете `AuthController` [идущий в Laravel](/docs/{{version}}/authentication), вам скорее всего не потребуется делать это вручную, так как данный контроллер автоматически выполняет вызов этого метода:

    if (Hash::check('plain-text', $hashedPassword)) {
        // Пароль верен...
    }

#### Проверка на необходимость рехэша пароля

Метод `needsRehash` позволяет вам определить изменился ли используемый рабочий фактор с момента формирования данного хэша:

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
