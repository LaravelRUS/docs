git 13313dae8d36b15430a74f683c5da59d04b82a32

---

# Шифрование

- [Введение](#introduction)
- [Основы использования](#basic-usage)

<a name="introduction"></a>
## Введение

Laravel предоставляет удобный механизм для использования стойкого шифрования алгоритмом AES на основе PHP-модуля Mcrypt.

<a name="basic-usage"></a>
## Основы использования

#### Шифрование

	$encrypted = Crypt::encrypt('secret');

> **Примечание:** Обязательно укажите строку из случайных символов длиной в 16, 24, или 32 символа в параметре `key` файла `config/app.php`.
В противном случае, зашифрованное значение будет не очень стойким к взлому.

#### Дешифровка

	$decrypted = Crypt::decrypt($encryptedValue);

#### Настройка алгоритма шифрования и режима работы

Вы можете указать [алгоритм шифрования](http://php.net/manual/en/mcrypt.ciphers.php) и [режим работы](http://php.net/manual/en/mcrypt.constants.php):

	Crypt::setMode('cfb');

	Crypt::setCipher($cipher);






# Шифрование

- [Настройка](#configuration)
- [Основы использования](#basic-usage)

<a name="configuration"></a>
## Настройка

Перед использованием шифрования в Laravel, вы должны указать опцию `key` в вашем конфигурационном файле `config/app.php`, которая должна состоять из 32 символов (случайный набор символов). Если это значение не задано или задано неправильно, то все значения, зашифрованные с помощью Laravel будут не безопасными. 

<a name="basic-usage"></a>
## Основы использования

#### Шифрование значений

Вы можете зашифровать какое либо значение используя [фассад](/docs/{{version}}/facades) `Crypt`. Все шифрованные значения будут зашифрованы с использованием OpenSSL и шифра `AES-256-CBC`. Более того, все шифрованные данные помечаются сообщением с кодом аутентификации (MAC) для обнаружения изменений в зашифрованной строке.

You may encrypt a value using the `Crypt` [facade](/docs/{{version}}/facades). All encrypted values are encrypted using OpenSSL and the `AES-256-CBC` cipher. Furthermore, all encrypted values are signed with a message authentication code (MAC) to detect any modifications to the encrypted string.

Например, мы можем использовать метод `encrypt` для шифрования секретных данных и сохранить их в [модели Eloquent](/docs/{{version}}/eloquent):

    <?php

    namespace App\Http\Controllers;

    use Crypt;
    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Store a secret message for the user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function storeSecret(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $user->fill([
                'secret' => Crypt::encrypt($request->secret)
            ])->save();
        }
    }

> **Примечание:** Зашифрованные значения проходят сериализацию `serialize` во время шифрования, что позволяет нам шифровать объекты и массивы. Таким образом не-PHP клиентам, которые получают шифрованные данные, понадобится десериализовать данные.


#### Дешифрование значения. 

Конечно же, вы можете расшифровать значения используя метод `decrypt` фассада `Crypt`. Если значение не может быть правильно расшифровано, из-за невалидного MAC, то будет выброшено исключение `Illuminate\Contracts\Encryption\DecryptException`:

    use Illuminate\Contracts\Encryption\DecryptException;

    try {
        $decrypted = Crypt::decrypt($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
