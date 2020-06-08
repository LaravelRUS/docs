git e3eacb710e8fc72bff96ee2ed0158af37131e107

---

# Хеширование

- [Введение](#introduction)
- [Основы использования](#basic-usage)

<a name="introduction"></a>
## Введение

[Фасад](/docs/{{version}}/facades) `Hash` в Laravel обеспечивает надёжное Bcrypt-хеширование для хранения паролей пользователей. Если вы используете встроенные классы `LoginController` и `RegisterController`, включённые в ваше Laravel-приложение, они будут автоматически использовать Bcrypt для регистрации и авторизации.

> {tip} отличный выбор для хеширования паролей, потому что сложность его вычисления настраивается. Это значит, что можно увеличить время генерации хеша, если вы располагаете мощным железом.

<a name="basic-usage"></a>
## Основы использования

Вы можете хешировать пароль, вызвав метод `make` фасада `Hash`:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use App\Http\Controllers\Controller;

    class UpdatePasswordController extends Controller
    {
        /**
         * Обновление пароля пользователя.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // Проверка длины нового пароля...

            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

#### Сверка пароля с хешем

Метод `check` позволяет сверить текстовую строку и имеющийся хеш. Но если вы используете [встроенный в Laravel](/docs/{{version}}/authentication)`LoginController`, то вам не нужно использовать этот метод, так как этот контроллер автоматически вызывает этот метод:

    if (Hash::check('plain-text', $hashedPassword)) {
        // Пароли совпадают...
    }

#### Проверка необходимости повторного хеширования пароля

Функция `needsRehash` позволяет определить, изменилась ли используемая сложность хеширования с момента создания хеша для пароля:

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
