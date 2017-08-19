git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Шифрование

- [Введение](#introduction)
- [Настройка](#configuration)
- [Использование шифратора](#using-the-encrypter)

<a name="introduction"></a>
## Введение

Шифратор Laravel использует OpenSSL для шифрования по алгоритмам AES-256 и AES-128. Настоятельно призываем вас использовать встроенные в Laravel возможности шифрования и не пытаться применять свои "самодельные" алгоритмы шифрования. Все шифрованные значения подписаны кодом аутентификации сообщения (MAC) для предотвращения любых изменений в зашифрованной строке.

<a name="configuration"></a>
## Настройка

Перед использованием шифрования Laravel обязательно задайте ключ `key` в конфиге `config/app.php`. Для этого вам надо использовать команду `php artisan key:generate`, которая использует надёжный генератор случайных чисел для создания вашего ключа. Без этого ключа все зашифрованные Laravel значения не будут безопасными.

<a name="using-the-encrypter"></a>
## Использование шифратора

#### Шифрование значения

Вы можете зашифровать значение с помощью хелпера `encrypt`. Все значения шифруются с помощью OpenSSL и шифра `AES-256-CBC`. Более того, все шифрованные значения подписаны кодом аутентификации сообщения (MAC) для обнаружения любых изменений в зашифрованной строке:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Сохранение секретного сообщения для пользователя.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function storeSecret(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $user->fill([
                'secret' => encrypt($request->secret)
            ])->save();
        }
    }

#### Шифрование без сериализации

При шифровании значения подвергаются "сериализации", что позволяет шифровать объекты и массивы. Поэтому при получении шифрованных значений не-PHP клиентам необходимо будет "десериализовать" данные. Если вы хотите зашифровать и расшифровать данные без сериализации, то можете использовать методы `encryptString` и `decryptString` фасада `Crypt`:

    use Illuminate\Support\Facades\Crypt;

    $encrypted = Crypt::encryptString('Hello world.');

    $decrypted = Crypt::decryptString($encrypted);

#### Расшифровка значения

Вы можете расшифровать значение при помощи хелпера `decrypt`. Если значение не может быть корректно расшифровано, например, при неверном MAC, будет выброшено исключение `Illuminate\Contracts\Encryption\DecryptException`:

    use Illuminate\Contracts\Encryption\DecryptException;

    try {
        $decrypted = decrypt($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
