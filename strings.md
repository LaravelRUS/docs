---
git: dc113868736123ab845451034064a460d21ca13e
---

# Строки

<a name="introduction"></a>
## Введение

Laravel включает в себя различные функции для работы с строковыми значениями. Многие из этих функций используются самим фреймворком; однако, вы вольны использовать их в своих собственных приложениях, если считаете их удобными.

<a name="available-methods"></a>
## Available Methods

<a name="strings-method-list"></a>
### Strings

<div class="docs-column-list" markdown="1">

- [\__](#method-__)
- [class_basename](#method-class-basename)
- [e](#method-e)
- [preg_replace_array](#method-preg-replace-array)
- [Str::after](#method-str-after)
- [Str::afterLast](#method-str-after-last)
- [Str::apa](#method-str-apa)
- [Str::ascii](#method-str-ascii)
- [Str::before](#method-str-before)
- [Str::beforeLast](#method-str-before-last)
- [Str::between](#method-str-between)
- [Str::betweenFirst](#method-str-between-first)
- [Str::camel](#method-camel-case)
- [Str::charAt](#method-char-at)
- [Str::contains](#method-str-contains)
- [Str::containsAll](#method-str-contains-all)
- [Str::endsWith](#method-ends-with)
- [Str::excerpt](#method-excerpt)
- [Str::finish](#method-str-finish)
- [Str::headline](#method-str-headline)
- [Str::inlineMarkdown](#method-str-inline-markdown)
- [Str::is](#method-str-is)
- [Str::isAscii](#method-str-is-ascii)
- [Str::isJson](#method-str-is-json)
- [Str::isUlid](#method-str-is-ulid)
- [Str::isUrl](#method-str-is-url)
- [Str::isUuid](#method-str-is-uuid)
- [Str::kebab](#method-kebab-case)
- [Str::lcfirst](#method-str-lcfirst)
- [Str::length](#method-str-length)
- [Str::limit](#method-str-limit)
- [Str::lower](#method-str-lower)
- [Str::markdown](#method-str-markdown)
- [Str::mask](#method-str-mask)
- [Str::orderedUuid](#method-str-ordered-uuid)
- [Str::padBoth](#method-str-padboth)
- [Str::padLeft](#method-str-padleft)
- [Str::padRight](#method-str-padright)
- [Str::password](#method-str-password)
- [Str::plural](#method-str-plural)
- [Str::pluralStudly](#method-str-plural-studly)
- [Str::position](#method-str-position)
- [Str::random](#method-str-random)
- [Str::remove](#method-str-remove)
- [Str::repeat](#method-str-repeat)
- [Str::replace](#method-str-replace)
- [Str::replaceArray](#method-str-replace-array)
- [Str::replaceFirst](#method-str-replace-first)
- [Str::replaceLast](#method-str-replace-last)
- [Str::replaceMatches](#method-str-replace-matches)
- [Str::replaceStart](#method-str-replace-start)
- [Str::replaceEnd](#method-str-replace-end)
- [Str::reverse](#method-str-reverse)
- [Str::singular](#method-str-singular)
- [Str::slug](#method-str-slug)
- [Str::snake](#method-snake-case)
- [Str::squish](#method-str-squish)
- [Str::start](#method-str-start)
- [Str::startsWith](#method-starts-with)
- [Str::studly](#method-studly-case)
- [Str::substr](#method-str-substr)
- [Str::substrCount](#method-str-substrcount)
- [Str::substrReplace](#method-str-substrreplace)
- [Str::swap](#method-str-swap)
- [Str::take](#method-take)
- [Str::title](#method-title-case)
- [Str::toHtmlString](#method-str-to-html-string)
- [Str::ucfirst](#method-str-ucfirst)
- [Str::ucsplit](#method-str-ucsplit)
- [Str::upper](#method-str-upper)
- [Str::ulid](#method-str-ulid)
- [Str::unwrap](#method-str-unwrap)
- [Str::uuid](#method-str-uuid)
- [Str::wordCount](#method-str-word-count)
- [Str::wordWrap](#method-str-word-wrap)
- [Str::words](#method-str-words)
- [Str::wrap](#method-str-wrap)
- [str](#method-str)
- [trans](#method-trans)
- [trans_choice](#method-trans-choice)

</div>

<a name="fluent-strings-method-list"></a>
### Строки Fluent

<div class="docs-column-list" markdown="1">

- [after](#method-fluent-str-after)
- [afterLast](#method-fluent-str-after-last)
- [apa](#method-fluent-str-apa)
- [append](#method-fluent-str-append)
- [ascii](#method-fluent-str-ascii)
- [basename](#method-fluent-str-basename)
- [before](#method-fluent-str-before)
- [beforeLast](#method-fluent-str-before-last)
- [between](#method-fluent-str-between)
- [betweenFirst](#method-fluent-str-between-first)
- [camel](#method-fluent-str-camel)
- [charAt](#method-fluent-str-char-at)
- [classBasename](#method-fluent-str-class-basename)
- [contains](#method-fluent-str-contains)
- [containsAll](#method-fluent-str-contains-all)
- [dirname](#method-fluent-str-dirname)
- [endsWith](#method-fluent-str-ends-with)
- [excerpt](#method-fluent-str-excerpt)
- [exactly](#method-fluent-str-exactly)
- [explode](#method-fluent-str-explode)
- [finish](#method-fluent-str-finish)
- [headline](#method-fluent-str-headline)
- [inlineMarkdown](#method-fluent-str-inline-markdown)
- [is](#method-fluent-str-is)
- [isAscii](#method-fluent-str-is-ascii)
- [isEmpty](#method-fluent-str-is-empty)
- [isNotEmpty](#method-fluent-str-is-not-empty)
- [isJson](#method-fluent-str-is-json)
- [isUlid](#method-fluent-str-is-ulid)
- [isUrl](#method-fluent-str-is-url)
- [isUuid](#method-fluent-str-is-uuid)
- [kebab](#method-fluent-str-kebab)
- [lcfirst](#method-fluent-str-lcfirst)
- [length](#method-fluent-str-length)
- [limit](#method-fluent-str-limit)
- [lower](#method-fluent-str-lower)
- [ltrim](#method-fluent-str-ltrim)
- [markdown](#method-fluent-str-markdown)
- [mask](#method-fluent-str-mask)
- [match](#method-fluent-str-match)
- [matchAll](#method-fluent-str-match-all)
- [isMatch](#method-fluent-str-is-match)
- [newLine](#method-fluent-str-new-line)
- [padBoth](#method-fluent-str-padboth)
- [padLeft](#method-fluent-str-padleft)
- [padRight](#method-fluent-str-padright)
- [pipe](#method-fluent-str-pipe)
- [plural](#method-fluent-str-plural)
- [position](#method-fluent-str-position)
- [prepend](#method-fluent-str-prepend)
- [remove](#method-fluent-str-remove)
- [repeat](#method-fluent-str-repeat)
- [replace](#method-fluent-str-replace)
- [replaceArray](#method-fluent-str-replace-array)
- [replaceFirst](#method-fluent-str-replace-first)
- [replaceLast](#method-fluent-str-replace-last)
- [replaceMatches](#method-fluent-str-replace-matches)
- [replaceStart](#method-fluent-str-replace-start)
- [replaceEnd](#method-fluent-str-replace-end)
- [rtrim](#method-fluent-str-rtrim)
- [scan](#method-fluent-str-scan)
- [singular](#method-fluent-str-singular)
- [slug](#method-fluent-str-slug)
- [snake](#method-fluent-str-snake)
- [split](#method-fluent-str-split)
- [squish](#method-fluent-str-squish)
- [start](#method-fluent-str-start)
- [startsWith](#method-fluent-str-starts-with)
- [stripTags](#method-fluent-str-strip-tags)
- [studly](#method-fluent-str-studly)
- [substr](#method-fluent-str-substr)
- [substrReplace](#method-fluent-str-substrreplace)
- [swap](#method-fluent-str-swap)
- [take](#method-fluent-str-take)
- [tap](#method-fluent-str-tap)
- [test](#method-fluent-str-test)
- [title](#method-fluent-str-title)
- [trim](#method-fluent-str-trim)
- [ucfirst](#method-fluent-str-ucfirst)
- [ucsplit](#method-fluent-str-ucsplit)
- [unwrap](#method-fluent-str-unwrap)
- [upper](#method-fluent-str-upper)
- [when](#method-fluent-str-when)
- [whenContains](#method-fluent-str-when-contains)
- [whenContainsAll](#method-fluent-str-when-contains-all)
- [whenEmpty](#method-fluent-str-when-empty)
- [whenNotEmpty](#method-fluent-str-when-not-empty)
- [whenStartsWith](#method-fluent-str-when-starts-with)
- [whenEndsWith](#method-fluent-str-when-ends-with)
- [whenExactly](#method-fluent-str-when-exactly)
- [whenNotExactly](#method-fluent-str-when-not-exactly)
- [whenIs](#method-fluent-str-when-is)
- [whenIsAscii](#method-fluent-str-when-is-ascii)
- [whenIsUlid](#method-fluent-str-when-is-ulid)
- [whenIsUuid](#method-fluent-str-when-is-uuid)
- [whenTest](#method-fluent-str-when-test)
- [wordCount](#method-fluent-str-word-count)
- [words](#method-fluent-str-words)

</div>

<a name="strings"></a>
## Строки

<a name="method-__"></a>
#### `__()` 

Функция `__` переводит переданную строку перевода или ключ перевода, используя ваши [файлы локализации](/docs/{{version}}/localization):

    echo __('Welcome to our application');

    echo __('messages.welcome');

Если указанная строка перевода или ключ не существует, то функция `__` вернет переданное значение. Итак, используя приведенный выше пример, функция `__` вернет `messages.welcome`, если этот ключ перевода не существует.

<a name="method-class-basename"></a>
#### `class_basename()` 

Функция `class_basename` возвращает имя переданного класса с удаленным пространством имен этого класса:

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` 

Функция `e` запускает PHP-функцию `htmlspecialchars` с параметром `double_encode`, установленным по умолчанию в `true`:

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-preg-replace-array"></a>
#### `preg_replace_array()` 

Функция `preg_replace_array` последовательно заменяет переданный шаблон в строке, используя массив:

    $string = 'The event will take place between :start and :end';

    $replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-after"></a>
#### `Str::after()` 

Метод `Str::after` возвращает все после переданного значения в строке. Если значение не существует в строке, то будет возвращена вся строка:

    use Illuminate\Support\Str;

    $slice = Str::after('This is my name', 'This is');

    // ' my name'

<a name="method-str-after-last"></a>
#### `Str::afterLast()` 

Метод `Str::afterLast` возвращает все после последнего вхождения переданного значения в строке. Если значение не существует в строке, то будет возвращена вся строка:

    use Illuminate\Support\Str;

    $slice = Str::afterLast('App\Http\Controllers\Controller', '\\');

    // 'Controller'

<a name="method-str-apa"></a>
#### `Str::apa()` 

Метод `Str::apa` преобразует заданную строку в `Title Case` в соответствии с [правилами APA](https://apastyle.apa.org/style-grammar-guidelines/capitalization/title-case):
    use Illuminate\Support\Str;

    $title = Str::apa('Creating A Project');

    // 'Creating a Project'

<a name="method-str-ascii"></a>
#### `Str::ascii()` 

Метод `Str::ascii` попытается транслитерировать строку в ASCII значение:

    use Illuminate\Support\Str;

    $slice = Str::ascii('û');

    // 'u'

<a name="method-str-before"></a>
#### `Str::before()` 

Метод `Str :: before` возвращает все до переданного значения в строке:

    use Illuminate\Support\Str;

    $slice = Str::before('This is my name', 'my name');

    // 'This is '

<a name="method-str-before-last"></a>
#### `Str::beforeLast()` 

Метод `Str::beforeLast` возвращает все до последнего вхождения переданного значения в строке:

    use Illuminate\Support\Str;

    $slice = Str::beforeLast('This is my name', 'is');

    // 'This '

<a name="method-str-between"></a>
#### `Str::between()` 

Метод `Str::between` возвращает часть строки между двумя значениями:

    use Illuminate\Support\Str;

    $slice = Str::between('This is my name', 'This', 'name');

    // ' is my '

<a name="method-str-between-first"></a>
#### `Str::betweenFirst()` 

Метод `Str::betweenFirst` возвращает наименьший возможный участок строки между двумя значениями:

    use Illuminate\Support\Str;

    $slice = Str::betweenFirst('[a] bc [d]', '[', ']');

    // 'a'

<a name="method-camel-case"></a>
#### `Str::camel()` 

Метод `Str::camel` преобразует переданную строку в `camelCase`:

    use Illuminate\Support\Str;

    $converted = Str::camel('foo_bar');

    // 'fooBar'

<a name="method-char-at"></a>

#### `Str::charAt()` 

Метод `Str::charAt` возвращает символ по указанному индексу. Если индекс выходит за границы, возвращается значение `false`:

    use Illuminate\Support\Str;

    $character = Str::charAt('This is my name.', 6);

    // 's'

<a name="method-str-contains"></a>
#### `Str::contains()` 

Метод `Str::contains` определяет, содержит ли переданная строка указанное значение (с учетом регистра):

    use Illuminate\Support\Str;

    $contains = Str::contains('This is my name', 'my');

    // true

Вы также можете указать массив значений, чтобы определить, содержит ли переданная строка какое-либо из значений:

    use Illuminate\Support\Str;

    $contains = Str::contains('This is my name', ['my', 'foo']);

    // true

<a name="method-str-contains-all"></a>
#### `Str::containsAll()` 

Метод `Str::containsAll` определяет, содержит ли переданная строка все значения массива:

    use Illuminate\Support\Str;

    $containsAll = Str::containsAll('This is my name', ['my', 'name']);

    // true

<a name="method-ends-with"></a>
#### `Str::endsWith()` 

Метод `Str::endsWith` определяет, заканчивается ли переданная строка указанным значением:

    use Illuminate\Support\Str;

    $result = Str::endsWith('This is my name', 'name');

    // true


Вы также можете указать массив значений, чтобы определить, заканчивается ли переданная строка каким-либо из значений:

    use Illuminate\Support\Str;

    $result = Str::endsWith('This is my name', ['name', 'foo']);

    // true

    $result = Str::endsWith('This is my name', ['this', 'foo']);

    // false

<a name="method-excerpt"></a>
#### `Str::excerpt()` 

Метод `Str::excerpt` извлекает отрывок из заданной строки, соответствующий первому вхождению фразы в эту строку:

    use Illuminate\Support\Str;

    $excerpt = Str::excerpt('This is my name', 'my', [
        'radius' => 3
    ]);

    // '...is my na...'

Опция `radius`, по умолчанию равная `100`, позволяет определить количество символов, которые должны появиться с каждой стороны усеченной строки.

Кроме того, вы можете использовать опцию `omission`, чтобы определить строку, которая будет добавлена перед и после усеченной строки:

    use Illuminate\Support\Str;

    $excerpt = Str::excerpt('This is my name', 'name', [
        'radius' => 3,
        'omission' => '(...) '
    ]);

    // '(...) my name'

<a name="method-str-finish"></a>
#### `Str::finish()` 

Метод `Str::finish` добавляет один экземпляр указанного значения в переданную строку, если она еще не заканчивается этим значением:

    use Illuminate\Support\Str;

    $adjusted = Str::finish('this/string', '/');

    // this/string/

    $adjusted = Str::finish('this/string/', '/');

    // this/string/

<a name="method-str-headline"></a>
#### `Str::headline()` 

Метод `Str::headline` преобразует строки, разделенные регистром, дефисами или подчеркиванием, в строку, разделенную пробелами, с заглавной первой буквой каждого слова:

    use Illuminate\Support\Str;

    $headline = Str::headline('steve_jobs');

    // Steve Jobs

    $headline = Str::headline('EmailNotificationSent');

    // Email Notification Sent

<a name="method-str-inline-markdown"></a>
#### `Str::inlineMarkdown()` 

Метод `Str::inlineMarkdown` преобразует Markdown в стиле GitHub в HTML в одну строку с использованием [CommonMark](https://commonmark.thephpleague.com/). Однако, в отличие от метода `markdown`, он не оборачивает весь сгенерированный HTML в блочный элемент:
    use Illuminate\Support\Str;

    $html = Str::inlineMarkdown('**Laravel**');

    // <strong>Laravel</strong>

<a name="method-str-is"></a>
#### `Str::is()` 

Метод `Str::is` определяет, соответствует ли переданная строка указанному шаблону. Допускается использование метасимвола подстановки `*`:

    use Illuminate\Support\Str;

    $matches = Str::is('foo*', 'foobar');

    // true

    $matches = Str::is('baz*', 'foobar');

    // false

<a name="method-str-is-ascii"></a>
#### `Str::isAscii()` 

Метод `Str::isAscii` определяет, является ли переданная строка 7-битной ASCII:

    use Illuminate\Support\Str;

    $isAscii = Str::isAscii('Taylor');

    // true

    $isAscii = Str::isAscii('ü');

    // false

<a name="method-str-is-json"></a>
#### `Str::isJson()` 

Метод `Str::isJson` определяет, является ли заданная строка допустимым JSON:

    use Illuminate\Support\Str;

    $result = Str::isJson('[1,2,3]');

    // true

    $result = Str::isJson('{"first": "John", "last": "Doe"}');

    // true

    $result = Str::isJson('{first: "John", last: "Doe"}');

    // false

<a name="method-str-is-url"></a>
#### `Str::isUrl()` 

Метод `Str::isUrl` определяет, является ли заданная строка допустимым URL:

    use Illuminate\Support\Str;

    $isUrl = Str::isUrl('http://example.com');

    // true

    $isUrl = Str::isUrl('laravel');

    // false

<a name="method-str-is-ulid"></a>
#### `Str::isUlid()` 

Метод `Str::isUlid` определяет, является ли заданная строка допустимым ULID:

    use Illuminate\Support\Str;

    $isUlid = Str::isUlid('01gd6r360bp37zj17nxb55yv40');

    // true

    $isUlid = Str::isUlid('laravel');

    // false

<a name="method-str-is-uuid"></a>
#### `Str::isUuid()` 

Метод `Str::isUuid` определяет, является ли заданная строка допустимым UUID:

    use Illuminate\Support\Str;

    $isUuid = Str::isUuid('a0a2a2d2-0b87-4a18-83f2-2529882be2de');

    // true

    $isUuid = Str::isUuid('laravel');

    // false

<a name="method-kebab-case"></a>
#### `Str::kebab()` 

Метод `Str::kebab` преобразует переданную строку в `kebab-case`:

    use Illuminate\Support\Str;

    $converted = Str::kebab('fooBar');

    // foo-bar

<a name="method-str-lcfirst"></a>
#### `Str::lcfirst()` 

Метод `Str::lcfirst` возвращает переданную строку с первым символом в нижнем регистре:

    use Illuminate\Support\Str;

    $string = Str::lcfirst('Foo Bar');

    // foo Bar

<a name="method-str-length"></a>
#### `Str::length()` 

Метод `Str::length` возвращает длину переданной строки:

    use Illuminate\Support\Str;

    $length = Str::length('Laravel');

    // 7

<a name="method-str-limit"></a>
#### `Str::limit()` 

Метод `Str::limit` усекает переданную строку до указанной длины:

    use Illuminate\Support\Str;

    $truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20);

    // The quick brown fox...

Вы также можете передать третий строковый аргумент, содержимое которого будет добавлено в конец:

    use Illuminate\Support\Str;

    $truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

    // The quick brown fox (...)

<a name="method-str-lower"></a>
#### `Str::lower()` 

Метод `Str::lower` преобразует переданную строку в нижний регистр:

    use Illuminate\Support\Str;

    $converted = Str::lower('LARAVEL');

    // laravel

<a name="method-str-markdown"></a>
#### `Str::markdown()` 

Метод `Str::markdown` конвертирует текст с разметкой [GitHub flavored Markdown](https://github.github.com/gfm/) в HTML:

    use Illuminate\Support\Str;

    $html = Str::markdown('# Laravel');

    // <h1>Laravel</h1>

    $html = Str::markdown('# Taylor <b>Otwell</b>', [
        'html_input' => 'strip',
    ]);

    // <h1>Taylor Otwell</h1>

<a name="method-str-mask"></a>
#### `Str::mask()` 

Метод `Str::mask` маскирует часть строки повторяющимся символом и может использоваться для обфускации сегментов строк, таких как адреса электронной почты и номера телефонов:

    use Illuminate\Support\Str;

    $string = Str::mask('taylor@example.com', '*', 3);

    // tay***************

При необходимости вы можете указать отрицательное число в качестве третьего аргумента метода `mask`, который даст указание методу начать маскировку на заданном расстоянии от конца строки:

    $string = Str::mask('taylor@example.com', '*', -15, 3);

    // tay***@example.com

<a name="method-str-ordered-uuid"></a>
#### `Str::orderedUuid()` 

Метод `Str::orderedUuid` генерирует UUID с «префиксом временной метки», который может быть эффективно сохранен в индексированном столбце базы данных. Каждый UUID, созданный с помощью этого метода, будет отсортирован после UUID, ранее созданных с помощью этого метода:

    use Illuminate\Support\Str;

    return (string) Str::orderedUuid();

<a name="method-str-padboth"></a>
#### `Str::padBoth()` 

Метод `Str::padBoth` оборачивает функцию `str_pad` PHP, заполняя обе стороны строки другой строкой, пока конечная строка не достигнет желаемой длины:

    use Illuminate\Support\Str;

    $padded = Str::padBoth('James', 10, '_');

    // '__James___'

    $padded = Str::padBoth('James', 10);

    // '  James   '

<a name="method-str-padleft"></a>
#### `Str::padLeft()` 

Метод `Str::padLeft` оборачивает функцию `str_pad` PHP, заполняя левую часть строки другой строкой, пока конечная строка не достигнет желаемой длины:

    use Illuminate\Support\Str;

    $padded = Str::padLeft('James', 10, '-=');

    // '-=-=-James'

    $padded = Str::padLeft('James', 10);

    // '     James'

<a name="method-str-padright"></a>
#### `Str::padRight()` 

Метод `Str::padRight` оборачивает функцию `str_pad` PHP, заполняя правую часть строки другой строкой, пока конечная строка не достигнет желаемой длины:

    use Illuminate\Support\Str;

    $padded = Str::padRight('James', 10, '-');

    // 'James-----'

    $padded = Str::padRight('James', 10);

    // 'James     '

<a name="method-str-password"></a>
#### `Str::password()` 

Метод `Str::password` можно использовать для генерации безопасного, случайного пароля заданной длины. Пароль будет состоять из комбинации букв, цифр, символов и пробелов. По умолчанию пароли имеют длину 32 символа:
    use Illuminate\Support\Str;

    $password = Str::password();

    // 'EbJo2vE-AS:U,$%_gkrV4n,q~1xy/-_4'

    $password = Str::password(12);

    // 'qwuar>#V|i]N'

<a name="method-str-plural"></a>
#### `Str::plural()` 

Метод `Str::plural` преобразует строку единственного числа в ее форму множественного числа. Эта функция поддерживает [любые из языков, поддерживаемых плюрализатором Laravel](/docs/{{version}}/localization#pluralization-language):
    use Illuminate\Support\Str;

    $plural = Str::plural('car');

    // cars

    $plural = Str::plural('child');

    // children

Вы можете передать целое число в качестве второго аргумента метода для получения строки в единственном или множественном числе:

    use Illuminate\Support\Str;

    $plural = Str::plural('child', 2);

    // children

    $singular = Str::plural('child', 1);

    // child

<a name="method-str-plural-studly"></a>
#### `Str::pluralStudly()` 

Метод `Str::pluralStudly` преобразует строку единственного слова, отформатированную в заглавном регистре studly, в форму множественного числа. Эта функция поддерживает [любой из языков, поддерживаемых плюрализатором Laravel](/docs/{{version}}/localization#pluralization-language):
    
    use Illuminate\Support\Str;

    $plural = Str::pluralStudly('VerifiedHuman');

    // VerifiedHumans

    $plural = Str::pluralStudly('UserFeedback');

    // UserFeedback

Вы можете передать целое число в качестве второго аргумента метода для получения строки в единственном или множественном числе:

    use Illuminate\Support\Str;

    $plural = Str::pluralStudly('VerifiedHuman', 2);

    // VerifiedHumans

    $singular = Str::pluralStudly('VerifiedHuman', 1);

    // VerifiedHuman

<a name="method-str-position"></a>
#### `Str::position()` 

Метод `Str::position` возвращает позицию первого вхождения подстроки в строке. Если подстрока не существует в данной строке, возвращается значение `false`:
    use Illuminate\Support\Str;

    $position = Str::position('Hello, World!', 'Hello');

    // 0

    $position = Str::position('Hello, World!', 'W');

    // 7

<a name="method-str-random"></a>
#### `Str::random()` 

Метод `Str::random` генерирует случайную строку указанной длины. Этот метод использует функцию `random_bytes` PHP:

    use Illuminate\Support\Str;

    $random = Str::random(40);

<a name="method-str-remove"></a>
#### `Str::remove()` 

Метод `Str::remove` удаляет указанную подстроку или массив подстрок в строке:

    use Illuminate\Support\Str;

    $string = 'Peter Piper picked a peck of pickled peppers.';

    $removed = Str::remove('e', $string);

    // Ptr Pipr pickd a pck of pickld ppprs.

Вы можете передать `false` в качестве третьего аргумента для игнорирования регистра удаляемых подстрок.

<a name="method-str-repeat"></a>
#### `Str::repeat()` 

Метод `Str::repeat` повторяет заданную строку:

```php
use Illuminate\Support\Str;

$string = 'a';

$repeat = Str::repeat($string, 5);

// aaaaa
```

<a name="method-str-replace"></a>
#### `Str::replace()` 

Метод `Str::replace` заменяет в строке одну подстроку другой:

    use Illuminate\Support\Str;

    $string = 'Laravel 8.x';

    $replaced = Str::replace('8.x', '9.x', $string);

    // Laravel 9.x

Метод `replace` также принимает аргумент `caseSensitive`. По умолчанию метод `replace` чувствителен к регистру:

    Str::replace('Framework', 'Laravel', caseSensitive: false);

<a name="method-str-replace-array"></a>
#### `Str::replaceArray()` 

Метод `Str::replaceArray` последовательно заменяет указанное значение в строке, используя массив:

    use Illuminate\Support\Str;

    $string = 'The event will take place between ? and ?';

    $replaced = Str::replaceArray('?', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-replace-first"></a>
#### `Str::replaceFirst()` 

Метод `Str::replaceFirst` заменяет первое вхождение переданного значения в строке:

    use Illuminate\Support\Str;

    $replaced = Str::replaceFirst('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // a quick brown fox jumps over the lazy dog

<a name="method-str-replace-last"></a>
#### `Str::replaceLast()` 

Метод `Str::replaceLast` заменяет последнее вхождение переданного значения в строке:

    use Illuminate\Support\Str;

    $replaced = Str::replaceLast('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // the quick brown fox jumps over a lazy dog

<a name="method-str-replace-matches"></a>
#### `Str::replaceMatches()` 

Метод `Str::replaceMatches` заменяет все части строки, соответствующие шаблону, заданной строкой замены:

    use Illuminate\Support\Str;

    $replaced = Str::replaceMatches(
        pattern: '/[^A-Za-z0-9]++/',
        replace: '',
        subject: '(+1) 501-555-1000'
    )

    // '15015551000'

Метод `replaceMatches` также принимает замыкание, которое будет вызвано для каждой части строки, соответствующей заданному шаблону, что позволяет вам выполнять логику замены внутри замыкания и возвращать замененное значение:
    
    use Illuminate\Support\Str;

    $replaced = Str::replaceMatches('/\d/', function (array $matches) {
        return '['.$matches[0].']';
    }, '123');

    // '[1][2][3]'

<a name="method-str-replace-start"></a>
#### `Str::replaceStart()` 

Метод `Str::replaceStart` заменяет только первое вхождение заданного значения, если значение появляется в начале строки:

    use Illuminate\Support\Str;

    $replaced = Str::replaceStart('Hello', 'Laravel', 'Hello World');

    // Laravel World

    $replaced = Str::replaceStart('World', 'Laravel', 'Hello World');

    // Hello World

<a name="method-str-replace-end"></a>
#### `Str::replaceEnd()` 

Метод `Str::replaceEnd` заменяет только последнее вхождение заданного значения, если значение появляется в конце строки:

    use Illuminate\Support\Str;

    $replaced = Str::replaceEnd('World', 'Laravel', 'Hello World');

    // Hello Laravel

    $replaced = Str::replaceEnd('Hello', 'Laravel', 'Hello World');

    // Hello World

<a name="method-str-reverse"></a>
#### `Str::reverse()` 

Метод `Str::reverse` переворачивает данную строку:

    use Illuminate\Support\Str;

    $reversed = Str::reverse('Hello World');

    // dlroW olleH

<a name="method-str-singular"></a>
#### `Str::singular()` 

Метод `Str::singular` преобразует строку в ее форму единственного числа. Эта функция поддерживает [любые из языков, поддерживаемых плюрализатором Laravel](/docs/{{version}}/localization#pluralization-language):
    use Illuminate\Support\Str;

    $singular = Str::singular('cars');

    // car

    $singular = Str::singular('children');

    // child

<a name="method-str-slug"></a>
#### `Str::slug()` 

Метод `Str::slug` создает «дружественный фрагмент» URL-адреса из переданной строки:

    use Illuminate\Support\Str;

    $slug = Str::slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-snake-case"></a>
#### `Str::snake()` 

Метод `Str::snake` преобразует переданную строку в `snake_case`:

    use Illuminate\Support\Str;

    $converted = Str::snake('fooBar');

    // foo_bar

    $converted = Str::snake('fooBar', '-');

    // foo-bar

<a name="method-str-squish"></a>
#### `Str::squish()` 

Метод `Str::squish` удаляет все лишние пробелы из строки, включая лишние пробелы между словами:

    use Illuminate\Support\Str;

    $string = Str::squish('    laravel    framework    ');

    // laravel framework

<a name="method-str-start"></a>
#### `Str::start()` 

Метод `Str::start` добавляет один экземпляр указанного значения в переданную строку, если она еще не начинается этим значением:

    use Illuminate\Support\Str;

    $adjusted = Str::start('this/string', '/');

    // /this/string

    $adjusted = Str::start('/this/string', '/');

    // /this/string

<a name="method-starts-with"></a>
#### `Str::startsWith()` 

Метод `Str::startsWith` определяет, начинается ли переданная строка с указанного значения:

    use Illuminate\Support\Str;

    $result = Str::startsWith('This is my name', 'This');

    // true

Если передан массив возможных значений, метод `startsWith` вернет `true`, если строка начинается с любого из заданных значений:

    $result = Str::startsWith('This is my name', ['This', 'That', 'There']);

    // true

<a name="method-studly-case"></a>
#### `Str::studly()` 

Метод `Str::studly` преобразует переданную строку в `StudlyCase`:

    use Illuminate\Support\Str;

    $converted = Str::studly('foo_bar');

    // FooBar

<a name="method-str-substr"></a>
#### `Str::substr()` 

Метод `Str::substr` возвращает часть строки, заданную параметрами «начало» и «длина»:

    use Illuminate\Support\Str;

    $converted = Str::substr('The Laravel Framework', 4, 7);

    // Laravel

<a name="method-str-substrcount"></a>
#### `Str::substrCount()` 

Метод `Str::substrCount` возвращает число вхождений подстроки в строку:

    use Illuminate\Support\Str;

    $count = Str::substrCount('If you like ice cream, you will like snow cones.', 'like');

    // 2

<a name="method-str-substrreplace"></a>
#### `Str::substrReplace()` 

Метод `Str::substrReplace` заменяет текст в части строки, начиная с позиции, указанной третьим аргументом, и заменяет число символов, указанное четвертым аргументом. Передав `0` четвертым аргументом в метод, строка будет вставлена в указанную позицию без замены каких-либо существующих символов в строке:

    use Illuminate\Support\Str;

    $result = Str::substrReplace('1300', ':', 2);
    // 13:

    $result = Str::substrReplace('1300', ':', 2, 0);
    // 13:00

<a name="method-str-swap"></a>
#### `Str::swap()` 

Метод `Str::swap` заменяет несколько значений в заданной строке, используя функцию `strtr` PHP:

    use Illuminate\Support\Str;

    $string = Str::swap([
        'Tacos' => 'Burritos',
        'great' => 'fantastic',
    ], 'Tacos are great!');

    // Burritos are fantastic!

<a name="method-take"></a>
#### `Str::take()` 

Метод `Str::take` возвращает указанное количество символов из начала строки:

    use Illuminate\Support\Str;

    $taken = Str::take('Build something amazing!', 5);

    // Build

<a name="method-title-case"></a>
#### `Str::title()` 

Метод `Str::title` преобразует переданную строку в `Title Case`:

    use Illuminate\Support\Str;

    $converted = Str::title('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-str-to-html-string"></a>
#### `Str::toHtmlString()` 

Метод `Str::toHtmlString` преобразует экземпляр строки в экземпляр `Illuminate\Support\HtmlString`, который может отображаться в шаблонах Blade:

    use Illuminate\Support\Str;

    $htmlString = Str::of('Nuno Maduro')->toHtmlString();

<a name="method-str-ucfirst"></a>
#### `Str::ucfirst()` 

Метод `Str::ucfirst` возвращает переданную строку с первой заглавной буквой:

    use Illuminate\Support\Str;

    $string = Str::ucfirst('foo bar');

    // Foo bar

<a name="method-str-ucsplit"></a>
#### `Str::ucsplit()` 

Метод `Str::ucsplit` разделяет заданную строку на массив по символам в верхнем регистре:

    use Illuminate\Support\Str;

    $segments = Str::ucsplit('FooBar');

    // [0 => 'Foo', 1 => 'Bar']

<a name="method-str-upper"></a>
#### `Str::upper()` 

Метод `Str::upper` преобразует переданную строку в верхний регистр:

    use Illuminate\Support\Str;

    $string = Str::upper('laravel');

    // LARAVEL

<a name="method-str-ulid"></a>
#### `Str::ulid()` 

Метод `Str::ulid` генерирует ULID, который является компактным, уникальным и упорядоченным по времени идентификатором:

    use Illuminate\Support\Str;

    return (string) Str::ulid();
    
    // 01gd6r360bp37zj17nxb55yv40

Если вы хотите получить экземпляр даты `Illuminate\Support\Carbon`, представляющий дату и время создания заданного ULID, вы можете использовать метод `createFromId`, предоставленный интеграцией Carbon в Laravel:
```php
use Illuminate\Support\Carbon;
use Illuminate\Support\Str;

$date = Carbon::createFromId((string) Str::ulid());
```

<a name="method-str-unwrap"></a>
#### `Str::unwrap()` 

Метод `Str::unwrap` удаляет указанные строки из начала и конца заданной строки:

    use Illuminate\Support\Str;

    Str::unwrap('-Laravel-', '-');

    // Laravel

    Str::unwrap('{framework: "Laravel"}', '{', '}');

    // framework: "Laravel"

<a name="method-str-uuid"></a>
#### `Str::uuid()` 

Метод `Str::uuid` генерирует UUID (версия 4):

    use Illuminate\Support\Str;

    return (string) Str::uuid();

<a name="method-str-word-count"></a>
#### `Str::wordCount()` 

Метод `Str::wordCount` возвращает число слов в строке:

```php
use Illuminate\Support\Str;

Str::wordCount('Hello, world!'); // 2
```

<a name="method-str-word-wrap"></a>
#### `Str::wordWrap()` 

Метод `Str::wordWrap` переносит строку по заданному количеству символов:

    use Illuminate\Support\Str;

    $text = "The quick brown fox jumped over the lazy dog."

    Str::wordWrap($text, characters: 20, break: "<br />\n");

    /*
    The quick brown fox<br />
    jumped over the lazy<br />
    dog.
    */

<a name="method-str-words"></a>
#### `Str::words()` 

Метод `Str::words` ограничивает количество слов в строке. Дополнительная строка может быть передана этому методу через его третий аргумент, чтобы указать, какая строка должна быть добавлена в конец усеченной строки:

    use Illuminate\Support\Str;

    return Str::words('Perfectly balanced, as all things should be.', 3, ' >>>');

    // Perfectly balanced, as >>>

<a name="method-str-wrap"></a>
#### `Str::wrap()` 

Метод `Str::wrap` оборачивает заданную строку дополнительной строкой или парой строк:

    use Illuminate\Support\Str;

    Str::wrap('Laravel', '"');

    // "Laravel"

    Str::wrap('is', before: 'This ', after: ' Laravel!');

    // This is Laravel!

<a name="method-str"></a>
#### `str()` 

Функция `str` возвращает новый экземпляр `Illuminate\Support\Stringable` для заданной строки. Эта функция эквивалентна методу `Str::of`:

    $string = str('Taylor')->append(' Otwell');

    // 'Taylor Otwell'

Если функции `str` не передается аргумент, она возвращает экземпляр `Illuminate\Support\Str`:

    $snake = str()->snake('FooBar');

    // 'foo_bar'

<a name="method-trans"></a>
#### `trans()` 

Функция `trans` переводит переданный ключ перевода, используя ваши [файлы локализации](/docs/{{version}}/localization):

    echo trans('messages.welcome');

Если указанный ключ перевода не существует, функция `trans` вернет данный ключ. Итак, используя приведенный выше пример, функция `trans` вернет `messages.welcome`, если ключ перевода не существует.

<a name="method-trans-choice"></a>
#### `trans_choice()` 

Функция `trans_choice` переводит заданный ключ перевода с изменением формы слова:

    echo trans_choice('messages.notifications', $unreadCount);

Если указанный ключ перевода не существует, функция `trans_choice` вернет данный ключ. Итак, используя приведенный выше пример, функция `trans_choice` вернет `messages.notifications`, если ключ перевода не существует.

<a name="fluent-strings"></a>
## Строки Fluent

Строки Fluent обеспечивают более гибкий объектно-ориентированный интерфейс для работы со строковыми значениями, позволяя объединять несколько строковых операций вместе с использованием более удобочитаемого синтаксиса по сравнению с традиционными строковыми операциями.

<a name="method-fluent-str-after"></a>
#### `after` 

Метод `after` возвращает все после переданного значения в строке. Вся строка будет возвращена, если значение не существует в строке:

    use Illuminate\Support\Str;

    $slice = Str::of('This is my name')->after('This is');

    // ' my name'

<a name="method-fluent-str-after-last"></a>
#### `afterLast` 

Метод `afterLast` возвращает все после последнего вхождения переданного значения в строке. Вся строка будет возвращена, если значение не существует в строке:

    use Illuminate\Support\Str;

    $slice = Str::of('App\Http\Controllers\Controller')->afterLast('\\');

    // 'Controller'

<a name="method-fluent-str-apa"></a>
#### `apa` 

Метод `apa` преобразует заданную строку в `Title Case` в соответствии с [правилами APA](https://apastyle.apa.org/style-grammar-guidelines/capitalization/title-case):

    use Illuminate\Support\Str;

    $converted = Str::of('a nice title uses the correct case')->apa();

    // A Nice Title Uses the Correct Case

<a name="method-fluent-str-append"></a>
#### `append` 

Метод `append` добавляет указанные значения в строку:

    use Illuminate\Support\Str;

    $string = Str::of('Taylor')->append(' Otwell');

    // 'Taylor Otwell'

<a name="method-fluent-str-ascii"></a>
#### `ascii` 

Метод `ascii` попытается транслитерировать строку в значение ASCII:

    use Illuminate\Support\Str;

    $string = Str::of('ü')->ascii();

    // 'u'

<a name="method-fluent-str-basename"></a>
#### `basename` 

Метод `basename` вернет завершающий компонент имени переданной строки:

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz')->basename();

    // 'baz'

При необходимости вы можете указать «расширение», которое будет удалено из завершающего компонента:

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz.jpg')->basename('.jpg');

    // 'baz'

<a name="method-fluent-str-before"></a>
#### `before` 

Метод `before` возвращает все до указанного значения в строке:

    use Illuminate\Support\Str;

    $slice = Str::of('This is my name')->before('my name');

    // 'This is '

<a name="method-fluent-str-before-last"></a>
#### `beforeLast` 

Метод `beforeLast` возвращает все до последнего вхождения переданного значения в строку:

    use Illuminate\Support\Str;

    $slice = Str::of('This is my name')->beforeLast('is');

    // 'This '

<a name="method-fluent-str-between"></a>
#### `between` 

Метод `between` возвращает часть строки между двумя значениями:

    use Illuminate\Support\Str;

    $converted = Str::of('This is my name')->between('This', 'name');

    // ' is my '

<a name="method-fluent-str-between-first"></a>
#### `betweenFirst` 

Метод `betweenFirst` возвращает наименьший возможный участок строки между двумя значениями:

    use Illuminate\Support\Str;

    $converted = Str::of('[a] bc [d]')->betweenFirst('[', ']');

    // 'a'

<a name="method-fluent-str-camel"></a>
#### `camel` 

Метод `camel` преобразует переданную строку в` camelCase`:

    use Illuminate\Support\Str;

    $converted = Str::of('foo_bar')->camel();

    // 'fooBar'

<a name="method-fluent-str-char-at"></a>
#### `charAt` 

Метод `charAt` возвращает символ по указанному индексу. Если индекс выходит за границы, возвращается значение `false`:
    use Illuminate\Support\Str;

    $character = Str::of('This is my name.')->charAt(6);

    // 's'

<a name="method-fluent-str-class-basename"></a>
#### `classBasename` 

Метод `classBasename` возвращает имя класса без пространства имен:

    use Illuminate\Support\Str;

    $class = Str::of('Foo\Bar\Baz')->classBasename();

    // 'Baz'

<a name="method-fluent-str-contains"></a>
#### `contains` 

Метод `contains` определяет, содержит ли переданная строка указанное значение (с учетом регистра):

    use Illuminate\Support\Str;

    $contains = Str::of('This is my name')->contains('my');

    // true

Вы также можете указать массив значений, чтобы определить, содержит ли переданная строка какое-либо из этих значений:

    use Illuminate\Support\Str;

    $contains = Str::of('This is my name')->contains(['my', 'foo']);

    // true

<a name="method-fluent-str-contains-all"></a>
#### `containsAll` 

Метод `containsAll` определяет, содержит ли переданная строка все значения массива:

    use Illuminate\Support\Str;

    $containsAll = Str::of('This is my name')->containsAll(['my', 'name']);

    // true

<a name="method-fluent-str-dirname"></a>
#### `dirname` 

Метод `dirname` возвращает родительскую часть директории переданной строки:

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz')->dirname();

    // '/foo/bar'

При желании вы можете указать, сколько уровней каталогов вы хотите вырезать из строки:

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz')->dirname(2);

    // '/foo'

<a name="method-fluent-str-excerpt"></a>
#### `excerpt` 

Метод `excerpt` извлекает отрывок из заданной строки, соответствующий первому вхождению фразы в эту строку:

    use Illuminate\Support\Str;
    
    $excerpt = Str::excerpt('This is my name', 'my', [
        'radius' => 3
    ]);
    
    // '...is my na...'


Опция `radius`, по умолчанию равная `100`, позволяет определить количество символов, которые должны появиться с каждой стороны усеченной строки.

Кроме того, вы можете использовать опцию `omission`, чтобы определить строку, которая будет добавлена перед и после усеченной строки:

    use Illuminate\Support\Str;
    
    $excerpt = Str::excerpt('This is my name', 'name', [
        'radius' => 3,
        'omission' => '(...) '
    ]);
    
    // '(...) my name'

<a name="method-fluent-str-ends-with"></a>
#### `endsWith` 

Метод `endsWith` определяет, заканчивается ли переданная строка указанным значением:

    use Illuminate\Support\Str;

    $result = Str::of('This is my name')->endsWith('name');

    // true

Вы также можете указать массив значений, чтобы определить, заканчивается ли переданная строка каким-либо из указанных значений:

    use Illuminate\Support\Str;

    $result = Str::of('This is my name')->endsWith(['name', 'foo']);

    // true

    $result = Str::of('This is my name')->endsWith(['this', 'foo']);

    // false

<a name="method-fluent-str-exactly"></a>
#### `exactly` 

Метод `exactly` определяет, является ли переданная строка точным совпадением с другой строкой:

    use Illuminate\Support\Str;

    $result = Str::of('Laravel')->exactly('Laravel');

    // true

<a name="method-fluent-str-explode"></a>
#### `explode` 

Метод `explode` разделяет строку по заданному разделителю и возвращает коллекцию, содержащую каждый раздел строки разбиения:

    use Illuminate\Support\Str;

    $collection = Str::of('foo bar baz')->explode(' ');

    // collect(['foo', 'bar', 'baz'])

<a name="method-fluent-str-finish"></a>
#### `finish` 

Метод `finish` добавляет один экземпляр указанного значения в переданную строку, если она еще не заканчивается этим значением:

    use Illuminate\Support\Str;

    $adjusted = Str::of('this/string')->finish('/');

    // this/string/

    $adjusted = Str::of('this/string/')->finish('/');

    // this/string/

<a name="method-fluent-str-headline"></a>
#### `headline` 

Метод `headline` преобразует строки, разделенные регистром, дефисами или подчеркиваниями, в строку с пробелами, где первая буква каждого слова написана заглавной:

    use Illuminate\Support\Str;

    $headline = Str::of('taylor_otwell')->headline();

    // Taylor Otwell

    $headline = Str::of('EmailNotificationSent')->headline();

    // Email Notification Sent

<a name="method-fluent-str-inline-markdown"></a>
#### `inlineMarkdown` 

Метод `inlineMarkdown` преобразует Markdown в стиле GitHub в HTML в одну строку с использованием [CommonMark](https://commonmark.thephpleague.com/). Однако, в отличие от метода `markdown`, он не оборачивает весь сгенерированный HTML в блочный элемент:
    
    use Illuminate\Support\Str;

    $html = Str::of('**Laravel**')->inlineMarkdown();

    // <strong>Laravel</strong>

<a name="method-fluent-str-is"></a>
#### `is` 

Метод `is` определяет, соответствует ли переданная строка указанному шаблону. Допускается использование метасимвола подстановки `*`:

    use Illuminate\Support\Str;

    $matches = Str::of('foobar')->is('foo*');

    // true

    $matches = Str::of('foobar')->is('baz*');

    // false

<a name="method-fluent-str-is-ascii"></a>
#### `isAscii` 

Метод `isAscii` определяет, является ли переданная строка строкой ASCII:

    use Illuminate\Support\Str;

    $result = Str::of('Taylor')->isAscii();

    // true

    $result = Str::of('ü')->isAscii();

    // false

<a name="method-fluent-str-is-empty"></a>
#### `isEmpty` 

Метод `isEmpty` определяет, является ли переданная строка пустой:

    use Illuminate\Support\Str;

    $result = Str::of('  ')->trim()->isEmpty();

    // true

    $result = Str::of('Laravel')->trim()->isEmpty();

    // false

<a name="method-fluent-str-is-not-empty"></a>
#### `isNotEmpty` 

Метод `isNotEmpty` определяет, является ли переданная строка не пустой:

    use Illuminate\Support\Str;

    $result = Str::of('  ')->trim()->isNotEmpty();

    // false

    $result = Str::of('Laravel')->trim()->isNotEmpty();

    // true

<a name="method-fluent-str-is-json"></a>
#### `isJson` 

Метод `isJson` определяет, является ли заданная строка допустимым JSON:

    use Illuminate\Support\Str;

    $result = Str::of('[1,2,3]')->isJson();

    // true

    $result = Str::of('{"first": "John", "last": "Doe"}')->isJson();

    // true

    $result = Str::of('{first: "John", last: "Doe"}')->isJson();

    // false

<a name="method-fluent-str-is-ulid"></a>
#### `isUlid` 

Метод `isUlid` определяет, является ли заданная строка ULID:

    use Illuminate\Support\Str;

    $result = Str::of('01gd6r360bp37zj17nxb55yv40')->isUlid();

    // true

    $result = Str::of('Taylor')->isUlid();

    // false

<a name="method-fluent-str-is-url"></a>
#### `isUrl` 

Метод `isUrl` определяет, является ли заданная строка URL:

    use Illuminate\Support\Str;

    $result = Str::of('http://example.com')->isUrl();

    // true

    $result = Str::of('Taylor')->isUrl();

    // false

<a name="method-fluent-str-is-uuid"></a>
#### `isUuid` 

Метод `isUuid` определяет, является ли заданная строка UUID:

    use Illuminate\Support\Str;

    $result = Str::of('5ace9ab9-e9cf-4ec6-a19d-5881212a452c')->isUuid();

    // true

    $result = Str::of('Taylor')->isUuid();

    // false

<a name="method-fluent-str-kebab"></a>
#### `kebab` 

Метод `kebab` преобразует переданную строку в `kebab-case`:

    use Illuminate\Support\Str;

    $converted = Str::of('fooBar')->kebab();

    // foo-bar

<a name="method-fluent-str-lcfirst"></a>
#### `lcfirst` 

Метод `lcfirst` возвращает заданную строку с первым символом в нижнем регистре:

    use Illuminate\Support\Str;

    $string = Str::of('Foo Bar')->lcfirst();

    // foo Bar


<a name="method-fluent-str-length"></a>
#### `length` 

Метод `length` возвращает длину переданной строки:

    use Illuminate\Support\Str;

    $length = Str::of('Laravel')->length();

    // 7

<a name="method-fluent-str-limit"></a>
#### `limit` 

Метод `limit` усекает переданную строку до указанной длины:

    use Illuminate\Support\Str;

    $truncated = Str::of('The quick brown fox jumps over the lazy dog')->limit(20);

    // The quick brown fox...

Вы также можете передать второй строковый аргумент, содержимое которого будет добавлено в конец:

    use Illuminate\Support\Str;

    $truncated = Str::of('The quick brown fox jumps over the lazy dog')->limit(20, ' (...)');

    // The quick brown fox (...)

<a name="method-fluent-str-lower"></a>
#### `lower` 

Метод `lower` преобразует переданную строку в нижний регистр:

    use Illuminate\Support\Str;

    $result = Str::of('LARAVEL')->lower();

    // 'laravel'

<a name="method-fluent-str-ltrim"></a>
#### `ltrim` 

Метод `ltrim` удаляет символы из начала строки:

    use Illuminate\Support\Str;

    $string = Str::of('  Laravel  ')->ltrim();

    // 'Laravel  '

    $string = Str::of('/Laravel/')->ltrim('/');

    // 'Laravel/'

<a name="method-fluent-str-markdown"></a>
#### `markdown` 

Метод `markdown` преобразует Markdown в стиле GitHub в HTML:

    use Illuminate\Support\Str;

    $html = Str::of('# Laravel')->markdown();

    // <h1>Laravel</h1>

    $html = Str::of('# Taylor <b>Otwell</b>')->markdown([
        'html_input' => 'strip',
    ]);

    // <h1>Taylor Otwell</h1>

<a name="method-fluent-str-mask"></a>
#### `mask` 

Метод `mask` маскирует часть строки повторяющимся символом и может использоваться для обфускации сегментов строк, таких как адреса электронной почты и номера телефонов:

    use Illuminate\Support\Str;

    $string = Str::of('taylor@example.com')->mask('*', 3);

    // tay***************

При необходимости вы указываете отрицательное число в качестве третьего аргумента метода `mask`, который даст указание методу начать маскировку на заданном расстоянии от конца строки:

    $string = Str::of('taylor@example.com')->mask('*', -15, 3);

    // tay***@example.com

    $string = Str::of('taylor@example.com')->mask('*', 4, -4);

    // tayl**********.com

<a name="method-fluent-str-match"></a>
#### `match` 

Метод `match` вернет часть строки, которая соответствует указанному шаблону регулярного выражения:

    use Illuminate\Support\Str;

    $result = Str::of('foo bar')->match('/bar/');

    // 'bar'

    $result = Str::of('foo bar')->match('/foo (.*)/');

    // 'bar'

<a name="method-fluent-str-match-all"></a>
#### `matchAll` 

Метод `matchAll` вернет коллекцию, содержащую части строки, которые соответствуют указанному шаблону регулярного выражения:

    use Illuminate\Support\Str;

    $result = Str::of('bar foo bar')->matchAll('/bar/');

    // collect(['bar', 'bar'])

Если вы укажете группировку в выражении, то Laravel вернет коллекцию совпадений этой группы:

    use Illuminate\Support\Str;

    $result = Str::of('bar fun bar fly')->matchAll('/f(\w*)/');

    // collect(['un', 'ly']);

Если совпадений не найдено, будет возвращена пустая коллекция.

<a name="method-fluent-str-is-match"></a>
#### `isMatch` 

Метод `isMatch` вернет `true`, если строка соответствует заданному регулярному выражению:

    use Illuminate\Support\Str;

    $result = Str::of('foo bar')->isMatch('/foo (.*)/');

    // true

    $result = Str::of('laravel')->isMatch('/foo (.*)/');

    // false

<a name="method-fluent-str-new-line"></a>
#### `newLine` 

Метод `newLine` добавляет символ "конец строки" к строке:

    use Illuminate\Support\Str;

    $padded = Str::of('Laravel')->newLine()->append('Framework');

    // 'Laravel
    //  Framework'

<a name="method-fluent-str-padboth"></a>
#### `padBoth` 

Метод `padBoth` оборачивает функцию `str_pad` PHP, заполняя обе стороны строки другой строкой, пока конечная строка не достигнет желаемой длины:

    use Illuminate\Support\Str;

    $padded = Str::of('James')->padBoth(10, '_');

    // '__James___'

    $padded = Str::of('James')->padBoth(10);

    // '  James   '

<a name="method-fluent-str-padleft"></a>
#### `padLeft` 

Метод `padLeft` оборачивает функцию `str_pad` PHP, заполняя левую часть строки другой строкой, пока конечная строка не достигнет желаемой длины:

    use Illuminate\Support\Str;

    $padded = Str::of('James')->padLeft(10, '-=');

    // '-=-=-James'

    $padded = Str::of('James')->padLeft(10);

    // '     James'

<a name="method-fluent-str-padright"></a>
#### `padRight` 

Метод `padRight` оборачивает функцию `str_pad` PHP, заполняя правую часть строки другой строкой, пока конечная строка не достигнет желаемой длины:

    use Illuminate\Support\Str;

    $padded = Str::of('James')->padRight(10, '-');

    // 'James-----'

    $padded = Str::of('James')->padRight(10);

    // 'James     '

<a name="method-fluent-str-pipe"></a>
#### `pipe` 

Метод `pipe` позволяет вам преобразовать строку, передав ее текущее значение указанной функции обратного вызова:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $hash = Str::of('Laravel')->pipe('md5')->prepend('Checksum: ');

    // 'Checksum: a5c95b86291ea299fcbe64458ed12702'

    $closure = Str::of('foo')->pipe(function (Stringable $str) {
        return 'bar';
    });

    // 'bar'

<a name="method-fluent-str-plural"></a>
#### `plural` 

Метод `plural` преобразует строку в единственном числе во множественное число. Эта функция поддерживает [любые из языков, поддерживаемых плюрализатором Laravel](/docs/{{version}}/localization#pluralization-language):
    use Illuminate\Support\Str;

    $plural = Str::of('car')->plural();

    // cars

    $plural = Str::of('child')->plural();

    // children

Вы можете передать целое число в качестве второго аргумента метода для получения строки в единственном или множественном числе:

    use Illuminate\Support\Str;

    $plural = Str::of('child')->plural(2);

    // children

    $plural = Str::of('child')->plural(1);

    // child

<a name="method-fluent-str-position"></a>
#### `position` 

Метод `position` возвращает позицию первого вхождения подстроки в строку. Если подстрока не существует внутри строки, возвращается значение `false`:

    use Illuminate\Support\Str;

    $position = Str::of('Hello, World!')->position('Hello');

    // 0

    $position = Str::of('Hello, World!')->position('W');

    // 7

<a name="method-fluent-str-prepend"></a>
#### `prepend` 

Метод `prepend` добавляет указанные значения в начало строки:

    use Illuminate\Support\Str;

    $string = Str::of('Framework')->prepend('Laravel ');

    // Laravel Framework

<a name="method-fluent-str-remove"></a>
#### `remove` 

Метод `remove` удаляет указанную подстроку или массив подстрок в строке:

    use Illuminate\Support\Str;

    $string = Str::of('Arkansas is quite beautiful!')->remove('quite');

    // Arkansas is beautiful!

Вы можете передать `false` в качестве второго аргумента для игнорирования регистра удаляемых строк.

<a name="method-fluent-str-repeat"></a>
#### `repeat` 

Метод `repeat` повторяет заданную строку:

```php
use Illuminate\Support\Str;

$repeated = Str::of('a')->repeat(5);

// aaaaa
```

<a name="method-fluent-str-replace"></a>
#### `replace` 

Метод `replace` заменяет указанную строку внутри строки:

    use Illuminate\Support\Str;

    $replaced = Str::of('Laravel 6.x')->replace('6.x', '7.x');

    // Laravel 7.x

Метод `replace` также принимает аргумент `caseSensitive`. По умолчанию метод `replace` чувствителен к регистру:

    $replaced = Str::of('macOS 13.x')->replace(
        'macOS', 'iOS', caseSensitive: false
    );

<a name="method-fluent-str-replace-array"></a>
#### `replaceArray` 

Метод `replaceArray` последовательно заменяет указанное значение в строке, используя массив:

    use Illuminate\Support\Str;

    $string = 'The event will take place between ? and ?';

    $replaced = Str::of($string)->replaceArray('?', ['8:30', '9:00']);

    // The event will take place between 8:30 and 9:00

<a name="method-fluent-str-replace-first"></a>
#### `replaceFirst` 

Метод `replaceFirst` заменяет первое вхождение указанного значения в строке:

    use Illuminate\Support\Str;

    $replaced = Str::of('the quick brown fox jumps over the lazy dog')->replaceFirst('the', 'a');

    // a quick brown fox jumps over the lazy dog

<a name="method-fluent-str-replace-last"></a>
#### `replaceLast` 

Метод `replaceLast` заменяет последнее вхождение указанного значения в строке:

    use Illuminate\Support\Str;

    $replaced = Str::of('the quick brown fox jumps over the lazy dog')->replaceLast('the', 'a');

    // the quick brown fox jumps over a lazy dog

<a name="method-fluent-str-replace-matches"></a>
#### `replaceMatches` 

Метод `replaceMatches` заменяет все части строки, соответствующие указанному шаблону, переданной строки:

    use Illuminate\Support\Str;

    $replaced = Str::of('(+1) 501-555-1000')->replaceMatches('/[^A-Za-z0-9]++/', '')

    // '15015551000'

Метод `replaceMatches` также принимает замыкание, которое будет вызвано для каждой части строки, соответствующей шаблону, что позволяет вам выполнять логику замены в замыкании и возвращать замененное значение:

    use Illuminate\Support\Str;

    $replaced = Str::of('123')->replaceMatches('/\d/', function (array $matches) {
        return '['.$matches[0].']';
    });

    // '[1][2][3]'

<a name="method-fluent-str-replace-start"></a>
#### `replaceStart` 

Метод `replaceStart` заменяет только первое вхождение заданного значения, если значение появляется в начале строки:

    use Illuminate\Support\Str;

    $replaced = Str::of('Hello World')->replaceStart('Hello', 'Laravel');

    // Laravel World

    $replaced = Str::of('Hello World')->replaceStart('World', 'Laravel');

    // Hello World

<a name="method-fluent-str-replace-end"></a>
#### `replaceEnd` 

Метод `replaceEnd` заменяет только последнее вхождение заданного значения, если значение появляется в конце строки:

    use Illuminate\Support\Str;

    $replaced = Str::of('Hello World')->replaceEnd('World', 'Laravel');

    // Hello Laravel

    $replaced = Str::of('Hello World')->replaceEnd('Hello', 'Laravel');

    // Hello World

<a name="method-fluent-str-rtrim"></a>
#### `rtrim` 

Метод `rtrim` удаляет символы из конца строки:

    use Illuminate\Support\Str;

    $string = Str::of('  Laravel  ')->rtrim();

    // '  Laravel'

    $string = Str::of('/Laravel/')->rtrim('/');

    // '/Laravel'

<a name="method-fluent-str-scan"></a>
#### `scan` 

Метод `scan` анализирует входные данные из строки в коллекцию в соответствии с форматом, поддерживаемым [`sscanf` функцией PHP](https://www.php.net/manual/ru/function.sscanf.php):

    use Illuminate\Support\Str;

    $collection = Str::of('filename.jpg')->scan('%[^.].%s');

    // collect(['filename', 'jpg'])

<a name="method-fluent-str-singular"></a>
#### `singular` 

Метод `singular` преобразует строку в ее форму единственного числа. Эта функция поддерживает [любые из языков, поддерживаемых плюрализатором Laravel](/docs/{{version}}/localization#pluralization-language):

    use Illuminate\Support\Str;

    $singular = Str::of('cars')->singular();

    // car

    $singular = Str::of('children')->singular();

    // child

<a name="method-fluent-str-slug"></a>
#### `slug` 

Метод `slug` создает «дружественный фрагмент» URL-адреса из переданной строки:

    use Illuminate\Support\Str;

    $slug = Str::of('Laravel Framework')->slug('-');

    // laravel-framework

<a name="method-fluent-str-snake"></a>
#### `snake` 

Метод `snake` преобразует переданную строку в `snake_case`:

    use Illuminate\Support\Str;

    $converted = Str::of('fooBar')->snake();

    // foo_bar

<a name="method-fluent-str-split"></a>
#### `split` 

Метод `split` разбивает строку на коллекцию с помощью регулярного выражения:

    use Illuminate\Support\Str;

    $segments = Str::of('one, two, three')->split('/[\s,]+/');

    // collect(["one", "two", "three"])

<a name="method-fluent-str-squish"></a>
#### `squish` 

Метод `squish` удаляет все лишние пробелы из строки, включая лишние пробелы между словами:

    use Illuminate\Support\Str;

    $string = Str::of('    laravel    framework    ')->squish();

    // laravel framework

<a name="method-fluent-str-start"></a>
#### `start` 

Метод `start` добавляет один экземпляр указанного значения в переданную строку, если она еще не начинается этим значением:

    use Illuminate\Support\Str;

    $adjusted = Str::of('this/string')->start('/');

    // /this/string

    $adjusted = Str::of('/this/string')->start('/');

    // /this/string

<a name="method-fluent-str-starts-with"></a>
#### `startsWith` 

Метод `startsWith` определяет, начинается ли переданная строка с указанного значения:

    use Illuminate\Support\Str;

    $result = Str::of('This is my name')->startsWith('This');

    // true

<a name="method-fluent-str-strip-tags"></a>
#### `stripTags` 

Метод `stripTags` удаляет все HTML- и PHP-теги из строки:

    use Illuminate\Support\Str;

    $result = Str::of('<a href="https://laravel.com">Taylor <b>Otwell</b></a>')->stripTags();

    // Taylor Otwell

    $result = Str::of('<a href="https://laravel.com">Taylor <b>Otwell</b></a>')->stripTags('<b>');

    // Taylor <b>Otwell</b>

<a name="method-fluent-str-studly"></a>
#### `studly` 

Метод `studly` преобразует переданную строку в `StudlyCase`:

    use Illuminate\Support\Str;

    $converted = Str::of('foo_bar')->studly();

    // FooBar

<a name="method-fluent-str-substr"></a>
#### `substr` 

Метод `substr` возвращает часть строки, заданную параметрами «начало» и «длина»:

    use Illuminate\Support\Str;

    $string = Str::of('Laravel Framework')->substr(8);

    // Framework

    $string = Str::of('Laravel Framework')->substr(8, 5);

    // Frame

<a name="method-fluent-str-substrreplace"></a>
#### `substrReplace` 

Метод `substrReplace` заменяет текст в части строки, начиная с позиции, указанной третьим аргументом, и заменяет число символов, указанное четвертым аргументом. Передав 0 четвертым аргументом в метод, строка будет вставлена в указанную позицию без замены каких-либо существующих символов в строке:

    use Illuminate\Support\Str;

    $string = Str::of('1300')->substrReplace(':', 2);

    // 13:

    $string = Str::of('The Framework')->substrReplace(' Laravel', 3, 0);

    // The Laravel Framework

<a name="method-fluent-str-swap"></a>
#### `swap` 

Метод `swap` заменяет несколько значений в строке с использованием функции `strtr` PHP:

    use Illuminate\Support\Str;

    $string = Str::of('Tacos are great!')
        ->swap([
            'Tacos' => 'Burritos',
            'great' => 'fantastic',
        ]);

    // Burritos are fantastic!

<a name="method-fluent-str-take"></a>
#### `take` 

Метод `take` возвращает указанное количество символов из начала строки:

    use Illuminate\Support\Str;

    $taken = Str::of('Build something amazing!')->take(5);

    // Build

<a name="method-fluent-str-tap"></a>
#### `tap` 

Метод `tap` передает строку заданному замыканию, позволяя вам взаимодействовать с ней, не затрагивая при этом саму строку. Исходная строка возвращается методом `tap` независимо от того, что возвращает замыкание:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('Laravel')
        ->append(' Framework')
        ->tap(function (Stringable $string) {
            dump('String after append: '.$string);
        })
        ->upper();

    // LARAVEL FRAMEWORK

<a name="method-fluent-str-test"></a>
#### `test` 

Метод `test` определяет, соответствует ли строка переданному шаблону регулярного выражения:

    use Illuminate\Support\Str;

    $result = Str::of('Laravel Framework')->test('/Laravel/');

    // true

<a name="method-fluent-str-title"></a>
#### `title` 

Метод `title` преобразует переданную строку в `Title Case`:

    use Illuminate\Support\Str;

    $converted = Str::of('a nice title uses the correct case')->title();

    // A Nice Title Uses The Correct Case

<a name="method-fluent-str-trim"></a>
#### `trim` 

Метод `trim` обрезает переданную строку:

    use Illuminate\Support\Str;

    $string = Str::of('  Laravel  ')->trim();

    // 'Laravel'

    $string = Str::of('/Laravel/')->trim('/');

    // 'Laravel'

<a name="method-fluent-str-ucfirst"></a>
#### `ucfirst` 

Метод `ucfirst` возвращает переданную строку с первой заглавной буквой:

    use Illuminate\Support\Str;

    $string = Str::of('foo bar')->ucfirst();

    // Foo bar

<a name="method-fluent-str-ucsplit"></a>
#### `ucsplit` 

Метод `upper` преобразует переданную строку в верхний регистр:

    use Illuminate\Support\Str;

    $string = Str::of('Foo Bar')->ucsplit();

    // collect(['Foo', 'Bar'])

<a name="method-fluent-str-unwrap"></a>
#### `unwrap` 

Метод `unwrap` удаляет указанные строки из начала и конца заданной строки:

    use Illuminate\Support\Str;

    Str::of('-Laravel-')->unwrap('-');

    // Laravel

    Str::of('{framework: "Laravel"}')->unwrap('{', '}');

    // framework: "Laravel"

<a name="method-fluent-str-upper"></a>
#### `upper` 

Метод `upper` преобразует заданную строку в верхний регистр:

    use Illuminate\Support\Str;

    $adjusted = Str::of('laravel')->upper();

    // LARAVEL

<a name="method-fluent-str-when"></a>
#### `when` 

Метод `when` вызывает указанное замыкание, если переданное условие истинно. Замыкание получит экземпляр Fluent:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('Taylor')
                    ->when(true, function (Stringable $string) {
                        return $string->append(' Otwell');
                    });

    // 'Taylor Otwell'

При необходимости вы можете передать другое замыкание в качестве третьего параметра методу `when`. Это замыкание будет выполнено, если параметр условия оценивается как `false`.

<a name="method-fluent-str-when-contains"></a>
#### `whenContains` 

Метод `whenContains` вызывает данное замыкание, если строка содержит заданное значение. Замыкание получит экземпляр класса `Stringable` в качестве аргумента:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('tony stark')
                ->whenContains('tony', function (Stringable $string) {
                    return $string->title();
                });

    // 'Tony Stark'

При необходимости вы можете передать другое замыкание в качестве третьего параметра метода `when`. Это замыкание будет выполнено, если строка не содержит заданного значения.

Вы также можете передать массив значений, чтобы определить, содержит ли данная строка какие-либо значения в массиве:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('tony stark')
                ->whenContains(['tony', 'hulk'], function (Stringable $string) {
                    return $string->title();
                });

    // Tony Stark

<a name="method-fluent-str-when-contains-all"></a>
#### `whenContainsAll` 

Метод `whenContainsAll` вызывает данное замыкание, если строка содержит все заданные подстроки. Замыкание получит экземпляр класса `Stringable` в качестве аргумента:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('tony stark')
                    ->whenContainsAll(['tony', 'stark'], function (Stringable $string) {
                        return $string->title();
                    });

    // 'Tony Stark'

При необходимости вы можете передать другое замыкание в качестве третьего параметра метода `when`. Это замыкание будет выполнено, если параметр условия оценивается как `false`.

<a name="method-fluent-str-when-empty"></a>
#### `whenEmpty` 

Метод `whenEmpty` вызывает переданное замыкание, если строка пуста. Если замыкание возвращает значение, то это значение будет возвращено методом `whenEmpty`. Если замыкание не возвращает значение, будет возвращен экземпляр Fluent:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('  ')->whenEmpty(function (Stringable $string) {
        return $string->trim()->prepend('Laravel');
    });

    // 'Laravel'

<a name="method-fluent-str-when-not-empty"></a>
#### `whenNotEmpty` 

Метод `whenNotEmpty` вызывает данное замыкание, если строка не пуста. Если замыкание возвращает значение, это значение также будет возвращено методом `whenNotEmpty`. Если замыкание не возвращает значение, будет возвращен экземпляр класса `Stringable`:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('Framework')->whenNotEmpty(function (Stringable $string) {
        return $string->prepend('Laravel ');
    });

    // 'Laravel Framework'

<a name="method-fluent-str-when-starts-with"></a>
#### `whenStartsWith` 

Метод `whenStartsWith` вызывает данное замыкание, если строка начинается с данной подстроки. Замыкание получит свободный экземпляр класса `Stringable` в качестве аргумента:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('disney world')->whenStartsWith('disney', function (Stringable $string) {
        return $string->title();
    });

    // 'Disney World'

<a name="method-fluent-str-when-ends-with"></a>
#### `whenEndsWith` 

Метод `whenEndsWith` вызывает данное замыкание, если строка заканчивается заданной подстрокой. Замыкание получит свободный экземпляр строки:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('disney world')->whenEndsWith('world', function (Stringable $string) {
        return $string->title();
    });

    // 'Disney World'

<a name="method-fluent-str-when-exactly"></a>
#### `whenExactly` 

Метод `whenExactly` вызывает данное замыкание, если строка точно соответствует заданной строке. Закрытие получит свободный экземпляр строки:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('laravel')->whenExactly('laravel', function (Stringable $string) {
        return $string->title();
    });

    // 'Laravel'

<a name="method-fluent-str-when-not-exactly"></a>
#### `whenNotExactly` 

Метод `whenExactly` вызывает данное замыкание, если строка не соответствует заданной строке. Закрытие получит свободный экземпляр строки:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('framework')->whenNotExactly('laravel', function (Stringable $string) {
        return $string->title();
    });

    // 'Framework'

<a name="method-fluent-str-when-is"></a>
#### `whenIs` 

Метод `whenIs` вызывает данное замыкание, если строка соответствует заданному шаблону. Звездочки могут использоваться в качестве подстановочных знаков. Замыкание получит экземпляр класса `Stringable` в качестве аргумента:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('foo/bar')->whenIs('foo/*', function (Stringable $string) {
        return $string->append('/baz');
    });

    // 'foo/bar/baz'

<a name="method-fluent-str-when-is-ascii"></a>
#### `whenIsAscii` 

Метод `whenIsAscii` вызывает данное замыкание, если строка представляет собой 7-битный ASCII. Замыкание получит экземпляр класса `Stringable` в качестве аргумента:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('laravel')->whenIsAscii(function (Stringable $string) {
        return $string->title();
    });

    // 'Laravel'

<a name="method-fluent-str-when-is-ulid"></a>
#### `whenIsUlid` 

Метод `whenIsUlid` вызывает заданное замыкание, если строка является допустимым ULID. Замыкание получит экземпляр класса `Stringable` в качестве аргумента:

    use Illuminate\Support\Str;

    $string = Str::of('01gd6r360bp37zj17nxb55yv40')->whenIsUlid(function (Stringable $string) {
        return $string->substr(0, 8);
    });

    // '01gd6r36'

<a name="method-fluent-str-when-is-uuid"></a>
#### `whenIsUuid` 

Метод `whenIsUuid` вызывает данное замыкание, если строка является допустимым UUID. Замыкание получит экземпляр класса `Stringable` в качестве аргумента:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('a0a2a2d2-0b87-4a18-83f2-2529882be2de')->whenIsUuid(function (Stringable $string) {
        return $string->substr(0, 8);
    });

    // 'a0a2a2d2'

<a name="method-fluent-str-when-test"></a>
#### `whenTest` 

Метод `whenTest` вызывает данное замыкание, если строка соответствует заданному регулярному выражению. Замыкание получит экземпляр класса `Stringable` в качестве аргумента:

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('laravel framework')->whenTest('/laravel/', function (Stringable $string) {
        return $string->title();
    });

    // 'Laravel Framework'

<a name="method-fluent-str-word-count"></a>
#### `wordCount` 

Метод `wordCount` возвращает число слов в строке:

```php
use Illuminate\Support\Str;

Str::of('Hello, world!')->wordCount(); // 2
```

<a name="method-fluent-str-words"></a>
#### `words` 

Метод `words` ограничивает количество слов в строке. Дополнительная строка может быть передана этому методу, чтобы указать, какая строка должна быть добавлена в конец усеченной строки:

    use Illuminate\Support\Str;

    $string = Str::of('Perfectly balanced, as all things should be.')->words(3, ' >>>');

    // Perfectly balanced, as >>>
