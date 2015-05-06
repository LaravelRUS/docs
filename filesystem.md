git 19c5df4f69e82bf8d6a3863e38d694dcaed4fc5f

---

# Работа с файлами / хранение в облаке

- [Введение](#introduction)
- [Настройка](#configuration)
- [Основы использования](#basic-usage)

<a name="introduction"></a>
## Введение

Laravel предоставляет чудесную абстракцию для работы с файловой системой - [Flysystem](https://github.com/thephpleague/flysystem) от Frank de Jonge.
У Flysystem есть драйвера для работы с Amazon S3, Rackspace Cloud Storage, и, конечно, с локальной файловой системой.
Теперь невероятно просто переключиться с хранения файлов на сервере на хранение файлов на S3!

<a name="configuration"></a>
## Настройка

Настройки Flysystem находятся в файле `config/filesystems.php`. Внутри него вы можете настроить несколько `'disks'`.
Каждый диск представляет свой тип хранения - локальная файловая система или облачные хранилища.
В файле настроек уже есть примеры конфигурации дисков для каждого из поддерживаемых хранилищ. 

Перед использованием S3 или Rackspace вы должны установить при помощи Composer соответствующие пакеты:

- Amazon S3: `"league/flysystem-aws-s3-v2": "~1.0"`
- Rackspace: `"league/flysystem-rackspace": "~1.0"`

Вы можете сконфигурировать несколько дисков, с одним и тем же драйвером.

Обратите внимание, что когда вы используете драйвер `'local'`, все пути в командах будут считаться от пути, заданного в параметре `'root'`.
По умолчанию это папка `storage/app`. Например, этот код создаст файл `storage/app/file.txt`:

	Storage::disk('local')->put('file.txt', 'Contents');

<a name="basic-usage"></a>
## Основы использования

Для взаимодействия с вашими дисками вы можете использовать фасад `Storage` или внедрить в конструктор класса объект, реализующий
`Illuminate\Contracts\Filesystem\Factory`, используя [сервис-контейнер](/docs/{{version}}/container).

#### Подключение диска

	$disk = Storage::disk('s3');

	$disk = Storage::disk('local');

#### Определение, существует ли файл

	$exists = Storage::disk('s3')->exists('file.jpg');

#### Выполнение метода на дефолтном диске

	if (Storage::exists('file.jpg'))
	{
		//
	}

#### Чтение файла

	$contents = Storage::get('file.jpg');

#### Запись в файл

	Storage::put('file.jpg', $contents);

#### Добавление контента в начало файла

	Storage::prepend('file.log', 'Prepended Text');

#### Добавление контента в конец файла

	Storage::append('file.log', 'Appended Text');

#### Удвление файла

	Storage::delete('file.jpg');

	Storage::delete(['file1.jpg', 'file2.jpg']);

#### Копировать файл

	Storage::copy('old/file1.jpg', 'new/file1.jpg');

#### Переместить файл

	Storage::move('old/file1.jpg', 'new/file1.jpg');

#### Получить размер файла

	$size = Storage::size('file1.jpg');

#### Получить время последней модификации файла (UNIX)

	$time = Storage::lastModified('file1.jpg');

#### Получить все файлы в директории

	$files = Storage::files($directory);

	// И из всех поддиректорий - рекурсивно...
	$files = Storage::allFiles($directory);

#### Получить все поддиректори

	$directories = Storage::directories($directory);

	// рекурсивно...
	$directories = Storage::allDirectories($directory);

#### Создать директорию

	Storage::makeDirectory($directory);

#### Удалить директорию

	Storage::deleteDirectory($directory);
