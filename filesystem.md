git fb8a56a0452bd036aef24fd6b96f59dc92a4cf35

---

# Работа с файлами / хранение в облаке

- [Введение](#introduction)
- [Настройка](#configuration)
- [Основы использования](#basic-usage)

<a name="introduction"></a>
## Введение

Laravel предоставляет чудесную абстракцию для работы с файловой системой - [Flysystem](https://github.com/thephpleague/flysystem) от Frank de Jonge. У Flysystem есть драйвера для работы с Amazon S3, Rackspace Cloud Storage, и, конечно, с локальной файловой системой. Теперь невероятно просто переключиться с хранения файлов на сервере на хранение файлов на S3 !

<a name="configuration"></a>
## Настройка

Конфиг Flysystem - `config/filesystems.php`. Внутри него вы можете настроить несколько `"disks"`. Каждый диск представляет свой тип хранения - локальная файловая система или облачные хранилища. В конфиге уже есть примеры настроек дисков для каждого из поддерживаемых хранилищ. 

Перед использованием S3 или Rackspace вы должны установить при помощи Composer соответствующие пакеты:

- Amazon S3: `"league/flysystem-aws-s3-v2": "~1.0"`
- Rackspace: `"league/flysystem-rackspace": "~1.0"`

Вы можете сконфигурить несколько дисков, с одним и тем же драйвером.

Обратите внимание, что когда вы используете драйвер "local", все пути в командах будут считаться от пути, заданного в параметре `'root'` конфига. По умолчанию это папка `storage/app`. Например, этот код создаст файл `storage/app/file.txt`:

	Storage::disk('local')->put('file.txt', 'Contents');

<a name="basic-usage"></a>
## Основы использования

Для взаимодействия с вашими дисками вы можете использовать фасад `Storage` или внедрить в конструктор класса объект `Illuminate\Contracts\Filesystem\Factory`.

#### Подключение диска

	$disk = Storage::disk('s3');

	$disk = Storage::disk('local');

#### Выполнение метода на дефолтном диске

	$exists = Storage::disk('s3')->exists('file.jpg');

#### Определение, существует ли файл

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