git 32930537ed8a3591e8b5bc91b61d4acfc9999023

---

# Хранение файлов

- [Введение](#introduction)
- [Настройка](#configuration)
    - [Общедоступный диск](#the-public-disk)
    - [Драйвер Local](#the-local-driver)
    - [Требования к драйверам](#driver-prerequisites)
    - [Кэширование](#caching)
- [Получение экземпляров дисков](#obtaining-disk-instances)
- [Загрузка (download) файлов](#downloading-files)
- [Чтение файлов](#retrieving-files)
    - [URL файла](#file-urls)
    - [Метаданные файла](#file-metadata)
- [Хранение файлов](#storing-files)
    - [Загрузка файлов](#file-uploads)
    - [Видимость файлов](#file-visibility)
- [Удаление файлов](#deleting-files)
- [Директории](#directories)
- [Пользовательские файловые системы](#custom-filesystems)

<a name="introduction"></a>
## Введение

Laravel предоставляет мощную абстракцию для работы с файловой системой благодаря великолепному PHP-пакету [Flysystem](https://github.com/thephpleague/flysystem) от Франка де Жонге. Laravel Flysystem содержит простые в использовании драйвера для работы с локальными файловыми системами и Amazon S3. Более того, можно переключаться между этими вариантами хранения файлов, поскольку у всех одинаковый API.

<a name="configuration"></a>
## Настройка

Настройки файловой системы находятся в конфигурационном файле `config/filesystems.php`. В нём вы можете настроить все свои "диски". Каждый диск представляет определенный драйвер и место хранения. В конфигурационном файле имеются примеры для каждого поддерживаемого драйвера. Поэтому вы можете просто немного изменить конфигурацию под ваши нужды.

Вы можете сконфигурировать столько дисков, сколько вам будет угодно, и даже можете иметь несколько дисков, которые используют один драйвер.

<a name="the-public-disk"></a>
### Общедоступный диск

Диск `public` предназначен для общего доступа к файлам. По умолчанию диск `public` использует драйвер `local` и хранит файлы в `storage/app/public`. Чтобы сделать их доступными через веб, вам надо создать символьную ссылку из `public/storage` на `storage/app/public`. При этом ваши общедоступные файлы будут храниться в одной директории, которую легко можно использовать в разных развёртываниях при использовании систем обновления на лету, таких как [Envoyer](https://envoyer.io).

Для создания символьной ссылки используйте Artisan-команду `storage:link`:

    php artisan storage:link

Когда файл сохранён и создана символьная ссылка, вы можете создать URL к файлу с помощью хелпера `asset`:

    echo asset('storage/file.txt');

<a name="the-local-driver"></a>
### Драйвер Local

При использовании драйвера `local` все файловые операции выполняются относительно директории `root` , определенной в вашем конфигурационном файле. По умолчанию это директория `storage/app`. Поэтому следующий метод сохранит файл в `storage/app/file.txt`:

    Storage::disk('local')->put('file.txt', 'Contents');

#### Разрешения

[Видимость файлов ](#file-visibility) типа `public` по умолчанию подразумевает права `0755` на директории и `0644` на файлы. Вы можете изменить это в конфигурационном файле:

    'local' => [
        'driver' => 'local',
        'root' => storage_path('app'),
        'permissions' => [
            'file' => [
                'public' => 0664,
                'private' => 0600,
            ],
            'dir' => [
                'public' => 0775,
                'private' => 0700,
            ],
        ],
    ],

<a name="driver-prerequisites"></a>
### Требования к драйверам

#### Пакеты Composer

Перед использованием SFTP или S3 вы должны установить соответствующие пакеты при помощи Composer:

- SFTP: `league/flysystem-sftp ~1.0`
- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`

И очень нелишним будет поставить пакет для кэширования запросов к удалённым файловым системам:

- CachedAdapter: `league/flysystem-cached-adapter ~1.0`

#### Настройка драйвера S3

Настройки драйвера S3 находятся в файле `config/filesystems.php`. Там есть пример массива настроек для драйвера S3. Вы можете отредактировать этот массив в соответствии с вашими настройками и учётными данными для S3. Для удобства эти переменные окружения соответствуют соглашениям об именовании, используемым в AWS CLI.

#### Настройка драйвера FTP

Если вы хотите использовать подключение по FTP, вы можете использовать в качестве примера приведенные ниже настройки:

    'ftp' => [
        'driver'   => 'ftp',
        'host'     => 'ftp.example.com',
        'username' => 'your-username',
        'password' => 'your-password',

        // Optional FTP Settings...
        // 'port'     => 21,
        // 'root'     => '',
        // 'passive'  => true,
        // 'ssl'      => true,
        // 'timeout'  => 30,
    ],

#### Настройка драйвера SFTP

Если вы хотите использовать подключение по SFTP, вы можете использовать в качестве примера приведенные ниже настройки:

    'sftp' => [
        'driver' => 'sftp',
        'host' => 'example.com',
        'username' => 'your-username',
        'password' => 'your-password',

        // Settings for SSH key based authentication...
        // 'privateKey' => '/path/to/privateKey',
        // 'password' => 'encryption-password',

        // Optional SFTP Settings...
        // 'port' => 22,
        // 'root' => '',
        // 'timeout' => 30,
    ],

<a name="caching"></a>
### Кэширование 

Чтобы разрешить кэширование, создайте секцию `cache`. Внутри укажите название диска, который будет использоваться для хранения кэша, время в секундах, сколько хранить кэш и префикс для файлов кэша:

    's3' => [
        'driver' => 's3',

        // Other Disk Options...

        'cache' => [
            'store' => 'memcached',
            'expire' => 600,
            'prefix' => 'cache-prefix',
        ],
    ],

<a name="obtaining-disk-instances"></a>
## Получение экземпляров дисков

Для взаимодействия с любым из ваших сконфигурированных дисков можно использовать фасад `Storage`. Например, вы можете использовать метод этого фасада `put` , чтобы сохранить аватар на диск по умолчанию. Если вы вызовите метод фасада `Storage` без предварительного вызова метода `disk` , то вызов метода будет автоматически передан диску по умолчанию:

    use Illuminate\Support\Facades\Storage;

    Storage::put('avatars/1', $fileContents);

При использовании нескольких дисков вы можете обращаться к нужному диску с помощью метода `disk` фасада  `Storage`:

    Storage::disk('s3')->put('avatars/1', $fileContents);

<a name="retrieving-files"></a>
## Чтение файлов

Методом `get` можно получать содержимое файла. Он возвращает сырую строку содержимого файла. Не забывайте, все пути файлов необходимо указывать относительно настроенного для диска "рута":

    $contents = Storage::get('file.jpg');

Методом `exists` можно определить существование файла на диске:

    $exists = Storage::disk('s3')->exists('file.jpg');

Методом `missing` можно определить, что файл на диске отсутствует:

    $missing = Storage::disk('s3')->missing('file.jpg');

<a name="downloading-files"></a>
### Загрузка (download) файлов

Для генерации http-ответа, который заставляет браузер пользователя загрузить файл, можно использовать метод `download`. Метод принимает путь до файла, имя файла, которое будет видно пользователю, и массив http-заголовков:

    return Storage::download('file.jpg');

    return Storage::download('file.jpg', $name, $headers);


<a name="file-urls"></a>
### URL файлов

При использовании драйверов `local` или `s3` вы можете использовать метод `url` для получения URL для файла. При использовании драйвера `local` в начало пути к файлу будет просто подставлено `/storage` , и будет возвращён относительный URL. При использовании драйвера `s3` будет возвращён полный удалённый URL:

    use Illuminate\Support\Facades\Storage;

    $url = Storage::url('file1.jpg');

> {note} При использовании драйвера `local` все файлы, которые должны быть общедоступны, необходимо помещать в директорию `storage/app/public`. Кроме того, вам надо [создать символьную ссылку](#the-public-disk) в `public/storage`, которая указывает на директорию `storage/app/public`.

#### Временные URL

При использовании драйвера `s3` вы можете создать временный URL, файл по которому будет доступен заданное время:

    $url = Storage::temporaryUrl(
        'file.jpg', now()->addMinutes(5)
    );

Если для `s3` вам нужно определить [дополнительные параметры запроса](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectGET.html#RESTObjectGET-requests), вы можете передать их третьим аргументом:

    $url = Storage::temporaryUrl(
        'file.jpg',
        now()->addMinutes(5),
        ['ResponseContentType' => 'application/octet-stream']
    );

#### Натройка локального URL хоста

Если вы бы хотели заранее задать хост для файлов, хранящихся на диске, используя драйвер `local`, вы можете добавить опцию `url` к массиву настройки диска:

    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],

<a name="file-metadata"></a>
### Метаданные файла

Помимо чтения и записи файлов Laravel может предоставить информацию о самих файлах. Например, для получения размера файла в байтах служит метод `size`:

    use Illuminate\Support\Facades\Storage;

    $size = Storage::size('file.jpg');

Для получения времени последней модификации файла (отметка времени UNIX) служит метод `lastModified`:

    $time = Storage::lastModified('file1.jpg');

<a name="storing-files"></a>
## Хранение файлов
Метод `put` используется для записи содержимого на диск. Также вы можете передать PHP `resource` методу `put`, чтобы использовать низкоуровневую поддержку потоков Flysystem. Помните, что все пути файлов должны быть указаны относительно "корневого" (root) места, настроенного для диска:

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

#### Автоматическая работа с потоками

Если вы хотите, чтобы Laravel автоматически использовал потоки для записи файла в хранилище, используйте методы `putFile` или `putFileAs`. Эти методы принимают объект `Illuminate\Http\File` или `Illuminate\Http\UploadedFile`, и автоматически используют потоки для размещения фала в необходимом месте:

    use Illuminate\Http\File;
    use Illuminate\Support\Facades\Storage;

    // Automatically generate a unique ID for file name...
    Storage::putFile('photos', new File('/path/to/photo'));

    // Manually specify a file name...
    Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');

У метода `putFile` есть несколько важных нюансов. Заметьте, мы указали только название каталога без имени файла. По умолчанию метод `putFile` генерирует UUID в качестве имени файла. Расширение файла будет определено путем изучения MIME-типа файла. Метод вернёт путь к файлу, поэтому вы можете сохранить в БД весь путь, включая сгенерированное имя.

Методы `putFile` и `putFileAs` принимают также аргумент «видимости» сохраняемого файла. Это полезно в основном при хранении файлов в облачном хранилище, таком как S3, когда необходим общий доступ к файлам:

    Storage::putFile('photos', new File('/path/to/photo'), 'public');

#### Добавление контента в начало / конец файла

Для вставки контента в начало или конец файла служат методы `prepend` и `append`:

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

#### Копирование и перемещение файлов

Метод `copy` используется для копирования существующего файла в новое расположение на диске, а метод `move` — для переименования или перемещения существующего файла в новое расположение:

    Storage::copy('old/file.jpg', 'new/file.jpg');

    Storage::move('old/file.jpg', 'new/file.jpg');

<a name="file-uploads"></a>
### Загрузка файлов

Загрузка файлов в веб-приложениях — это чаще всего загрузка пользовательских файлов, таких как аватар, фотографии и документы. В Laravel очень просто сохранять загружаемые файлы методом `store` на экземпляре загружаемого файла. Вызовите метод `store`, указав путь для сохранения загружаемого файла:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class UserAvatarController extends Controller
    {
        /**
         * Обновление аватара пользователя.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            $path = $request->file('avatar')->store('avatars');

            return $path;
        }
    }

В этом примере есть несколько важных моментов. Заметьте, мы указали только название каталога без имени файла. По умолчанию метод `store` генерирует UUID в качестве имени файла. Расширение файла будет определено путем изучения MIME-типа файла. Метод вернёт путь к файлу, поэтому вы можете сохранить в БД весь путь, включая сгенерированное имя.

Также вы можете вызвать метод `putFile` фасада `Storage` для выполнения этой же операции над файлом, как показано в примере:

    $path = Storage::putFile('avatars', $request->file('avatar'));

#### Указание имени файла

Если вы не хотите, чтобы файлу автоматически было назначено имя, можете использовать метод `storeAs`, который принимает в виде аргументов путь, имя файла, и (необязательно) диск:

    $path = $request->file('avatar')->storeAs(
        'avatars', $request->user()->id
    );

Вы также можете использовать метод `putFileAs` фасада `Storage`, который выполняет такую же операцию:

    $path = Storage::putFileAs(
        'avatars', $request->file('avatar'), $request->user()->id
    );

> {note} Непечатаемые и недействительные символы юникода будут автоматически удалены из путей к файлам. Поэтому, возможно, вы захотите "санитизировать" пути к файлам перед тем, как передавать их в методы хранения файлов Laravel. Пути к файлам нормализуются методом `League\Flysystem\Util::normalizePath`.    

#### Указание диска

По умолчанию этот метод использует диск по умолчанию. Если необходимо указать другой диск, передайте имя диска в качестве второго аргумента в метод `store`:

    $path = $request->file('avatar')->store(
        'avatars/'.$request->user()->id, 's3'
    );

#### Другая информация о файле

Чтобы получить оригинальное имя загруженного файла воспользуйтесь методом `getClientOriginalName`:

    $name = $request->file('avatar')->getClientOriginalName();

Оригинальное расширение доступно при помощи метода `extension`:

    $extension = $request->file('avatar')->extension();

<a name="file-visibility"></a>
### Видимость файлов

В интеграции Flysystem в Laravel "видимость" — это абстракция разрешений на файлы для использования на нескольких платформах. Файлы могут быть обозначены как `public` или `private`. Если файл отмечен как `public`, значит он должен быть доступен остальным. Например, при использовании драйвера S3 вы можете получить URL для `public`-файлов.

Вы можете задать видимость при размещении файла методом `put`:

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents, 'public');

Если файл уже был сохранён, то получить и задать его видимость можно методами `getVisibility` и `setVisibility`:

    $visibility = Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public');

<a name="deleting-files"></a>
## Удаление файлов

Метод `delete` принимает имя одного файла или массив файлов для удаления с диска:

    use Illuminate\Support\Facades\Storage;

    Storage::delete('file.jpg');

    Storage::delete(['file1.jpg', 'file2.jpg']);

Вы можете явно указать диск, на котором удаляете файл:

    use Illuminate\Support\Facades\Storage;

    Storage::disk('s3')->delete('folder_path/file_name.jpg');    

<a name="directories"></a>
## Директории

#### Получение всех файлов из директории

Метод `files` возвращает массив всех файлов из указанной директории. Если вы хотите получить массив всех файлов директории и её поддиректорий, используйте метод `allFiles`:

    use Illuminate\Support\Facades\Storage;

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

#### Получение всех поддиректорий

Метод `directories` возвращает массив всех директорий из указанной директории. Вдобавок, вы можете использовать метод `allDirectories` для получения списка всех директорий в данной директории и во всех её поддиректориях:

    $directories = Storage::directories($directory);

    // Recursive...
    $directories = Storage::allDirectories($directory);

#### Создание директории

Метод `makeDirectory` создаёт указанную директорию, включая необходимые поддиректории:

    Storage::makeDirectory($directory);

#### Удаление директории

И, наконец, метод `deleteDirectory` удаляет директорию и все её файлы с диска:

    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## Пользовательские файловые системы

Laravel Flysystem предоставляет драйверы для нескольких "drivers" из коробки. Однако, Flysystem не ограничен ими и содержит в себе адаптеры для многих других систем хранения. Вы можете создать свой драйвер, если хотите использовать один из этих дополнительных адаптеров в вашем приложении Laravel.

Чтобы настроить польховательскую файловую систему, вам потребуется адаптер Flysystem. Давайте добавим поддерживаемый сообществом Dropbox-адаптер к нашему проекту:

    composer require spatie/flysystem-dropbox

Затем вы должны создать [сервис-провайдер](/docs/{{version}}/providers), такой как `DropboxServiceProvider`. Для определения своего драйвера вы можете использовать метод `extend` фасада `Storage` в методе `boot` провайдера:

<?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use League\Flysystem\Filesystem;
    use Spatie\Dropbox\Client as DropboxClient;
    use Spatie\FlysystemDropbox\DropboxAdapter;
    use Storage;

    class DropboxServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Storage::extend('dropbox', function ($app, $config) {
                $client = new DropboxClient(
                    $config['authorization_token']
                );

                return new Filesystem(new DropboxAdapter($client));
            });
        }
    }

Первый аргумент метода `extend` — имя драйвера, второй — замыкание, которое получает переменные `$app` и `$config`. Замыкание должно возвратить экземпляр `League\Flysystem\Filesystem`. Переменная `$config` содержит значения, определенные в `config/filesystems.php` для указанного диска.

Затем зарегистрируйте свой сервис-провайдер в `config/app.php`:

    'providers' => [
        // ...
        App\Providers\DropboxServiceProvider::class,
    ];

Когда вы создали сервис-провайдер и зарегистрировали его, вы можете использовать драйвер `dropbox` в своём файле с настройками `config/filesystems.php`.
