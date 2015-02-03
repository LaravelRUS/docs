git 103a627268f66b0c4a3002973174670b1ecc19a3

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
