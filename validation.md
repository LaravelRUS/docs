git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Валидация

- [Введение](#introduction)
- [Быстрый старт](#validation-quickstart)
    - [Определение роутов](#quick-defining-the-routes)
    - [Создание контроллера](#quick-creating-the-controller)
    - [Написание логики валидации](#quick-writing-the-validation-logic)
    - [Вывод ошибок валидации](#quick-displaying-the-validation-errors)
    - [Дополнительные поля](#a-note-on-optional-fields)
- [Валидация в классах Form Request](#form-request-validation)
    - [Создание Form Requests](#creating-form-requests)
    - [Авторизация в Form Requests](#authorizing-form-requests)
    - [Настройка формата вывода ошибок валидации](#customizing-the-error-format)
    - [Настройка сообщений об ошибках](#customizing-the-error-messages)
- [Создание валидаторов вручную](#manually-creating-validators)
    - [Автоматический редирект](#automatic-redirection)
    - [MessageBag](#named-error-bags)
    - [Хук после валидации](#after-validation-hook)
- [Работа с сообщениями об ошибках](#working-with-error-messages)
    - [Пользовательские сообщения об ошибках](#custom-error-messages)
- [Доступные правила валидации](#available-validation-rules)
- [Добавление правил с условиями](#conditionally-adding-rules)
- [Валидация массивов](#validating-arrays)
- [Собственные правила валидации](#custom-validation-rules)

<a name="introduction"></a>
## Введение

Laravel предоставляет несколько способов для валидации входящих данных. По умолчанию базовый контроллер использует трейт `ValidatesRequests`, который обеспечивает удобный способ валидации HTTP запросов c большим количеством правил валидации.

<a name="validation-quickstart"></a>
## Быстрый старт

Перед тем как узнать обо всех функциях, давайте рассмотрим полный пример валидации формы и вывод сообщений об ошибках для пользователя.

<a name="quick-defining-the-routes"></a>
### Определение роутов

Во первых, представим что мы имеем следующие роуты в файле `routes/web.php`:

    Route::get('post/create', 'PostController@create');

    Route::post('post', 'PostController@store');

Конечно, роут `GET` отображает форму для создания нового поста в блоге, в то время как `POST` будет сохранять новую запись в базе данных.

<a name="quick-creating-the-controller"></a>
### Создание контроллера

Посмотрим на простой контроллер, который обрабатывает эти роуты. Метод `store` пока оставим пустым:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Показать форму для создания новой записи в блоге.
         *
         * @return Response
         */
        public function create()
        {
            return view('post.create');
        }

        /**
         * Хранить новую запись в блоге.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Validate and store the blog post...
        }
    }

<a name="quick-writing-the-validation-logic"></a>
### Написание логики валидации

Теперь мы готовы заполнить метод `store` валидацией при создания нового поста. Если проанализировать базовый контроллер (`App\Http\Controllers\Controller`), вы заметите, что он включает в себя трейт `ValidatesRequests`, который обеспечивает все контроллеры удобным методом `validate`.

Метод `validate` принимает два параметра экземпляр HTTP запроса и правила валидации. Если все правила не нарушены, код будет выполняться далее. Однако, если проверка не пройдена, будет выброшено исключение и сообщение об ошибке автоматически отправится обратно пользователю. По традициям HTTP запроса, ответ будет перенаправлен обратно с заполненными flash-переменными, в то время как на AJAX запрос отправится JSON.

Для лучшего понимания метода `validate`, вернемся обратно к `store`:

    /**
     * Store a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $this->validate($request, [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        // The blog post is valid, store in database...
    }

Как видно, мы передаем экземпляр с данными HTTP запроса и правила валидации в метод `validate`. Помните, что если проверка завершится неудачей, будет автоматически сгенерирован ответ с сообщением об ошибках, иначе ваш контроллер продолжит работать.

#### Остановка после первой неудачной проверки

Иногда нужно остановить выполнение остальных правил после первой неудачной проверки. Для этого используется атрибут `bail`:

    $this->validate($request, [
        'title' => 'bail|unique:posts|max:255',
        'body' => 'required',
    ]);

В этом примере, если для атрибута `title` не выполняется правило `required`, следующие правило `unique` проверяться не будет. Правила выполняются именно в той последовательности, в какой они назначаются.

#### Заметка о вложенных атрибутах

Если данные HTTP запроса содержат "вложенные" параметры, можно указать их, используя синтаксис с точкой:

    $this->validate($request, [
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);

<a name="quick-displaying-the-validation-errors"></a>
### Отображение ошибок валидации

Что, если входящие данные не проходят проверку с учетом правил? Как упоминалось ранее, Laravel автоматически перенаправляет пользователя на предыдущую страницу. Кроме того, все ошибки валидации будут автоматически записаны во [flash-переменные](/docs/{{version}}/session#flash-data).

Опять же, обратите внимание, что мы не должны явно передавать сообщения об ошибках в шаблоне роута `GET`. Это потому, что Laravel будет проверять наличие ошибок в текущем сеансе и автоматически привязывать их к шаблону, если они доступны.  Переменная `$errors` является экземпляром `Illuminate\Support\MessageBag`. Для получения дополнительных сведений о работе с этим объектом, [смотрите в документации](#working-with-error-messages).

> {tip}  Переменная `$errors`привязана к посреднику `Illuminate\View\Middleware\ShareErrorsFromSession`, который входит в группу посредников `web `. При использовании этого посредника, **`$errors` всегда будет доступна в ваших шаблонах**, что позволяет удобно и безопасно ее использовать.

В нашем примере пользователь будет перенаправлен в метод `create` вашего контроллера и можно отобразить сообщения об ошибках в шаблоне:

    <!-- /resources/views/post/create.blade.php -->

    <h1>Create Post</h1>

    @if ($errors->any())
        <div class="alert alert-danger">
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    <!-- Create Post Form -->

<a name="a-note-on-optional-fields"></a>
### Заметка о дополнительных полях

По умолчанию в Laravel включены глобальные посредники `TrimStrings` и `ConvertEmptyStringsToNull`. Они перечислены в свойстве `$middleware` класса `App\Http\Kernel`. Из-за этого нужно часто помечать дополнительные поля как `nullable`, если не нужно, чтобы валидатор считал не действительным значение `null`. Например:

    $this->validate($request, [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

В этом примере мы указываем что поле `publish_at` может быть `null` или должно содержать дату. Если модификатор `nullable` не добавляется в правило, проверяющий элемент будет рассматривать `null` как недопустимую дату.

<a name="quick-customizing-the-flashed-error-format"></a>
#### Настройка формата вывода ошибок валидации

Если вы хотите настроить вывод ошибок валидации, которые будут во flash-переменных после нарушений правил, переопределите метод `formatValidationErrors` в базовом контроллере. Не забудьте подключить класс `Illuminate\Contracts\Validation\Validator`:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Foundation\Bus\DispatchesJobs;
    use Illuminate\Contracts\Validation\Validator;
    use Illuminate\Routing\Controller as BaseController;
    use Illuminate\Foundation\Validation\ValidatesRequests;

    abstract class Controller extends BaseController
    {
        use DispatchesJobs, ValidatesRequests;

        /**
         * {@inheritdoc}
         */
        protected function formatValidationErrors(Validator $validator)
        {
            return $validator->errors()->all();
        }
    }

<a name="quick-ajax-requests-and-validation"></a>
#### AJAX запросы и валидация

В последнем примере мы использовали традиционные формы для отправки данных в наше приложение. Однако многие приложения используют AJAX-запросы. При использовании метода `validate` во время запроса AJAX, Laravel не будет генерировать ответ с перенаправлением. Вместо этого Laravel генерирует ответ с JSON данными, содержащий в себе все ошибки проверки. Этот ответ будет отправлен с кодом состояния HTTP 422.

<a name="form-request-validation"></a>
## Валидация Form Request

<a name="creating-form-requests"></a>
### Создание Form Request

Для более сложных сценариев валидаций, будут более удобны `Form Requests`. Form Requests это специальные классы, которые содержат в себе логику проверки. Для создания класса, используйте artisan-команду `make:request`:

    php artisan make:request StoreBlogPost

Сгенерированный класс будет размещен в каталоге `app/Http/Requests`. Если этот каталог не существует, он будет создан. Давайте добавим несколько правил проверки в метод `rules`:

    /**
     * Получить правила валидации, применимые к запросу.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

Так как же здесь работают правила валидации? Все, что нужно сделать — это указать класс Form Request в аргументах метода вашего контроллера. Входящий запрос перед вызовом метода контроллера будет валидироваться автоматически, что позволит загромождать контроллер логикой валидации:

    /**
     * Store the incoming blog post.
     *
     * @param  StoreBlogPost  $request
     * @return Response
     */
    public function store(StoreBlogPost $request)
    {
        // The incoming request is valid...
    }

Если проверка не пройдена, то при традиционном запросе ошибки будут записываться в сессию и будут доступны в шаблонах, иначе, если запрос был AJAX, HTTP-ответ с кодом 422 будет возвращен пользователю, включая JSON с ошибками валидации.

#### Добавление хуков в Form Request

Если вы хотите добавить хук "after" в Form Requests, можно использовать метод `withValidator`. Этот метод получает полностью сформированный валидатор, позволяя вызвать любой из его методов, прежде чем фактически применяются правила:

    /**
     * Настройка экземпляра валидатора.
     *
     * @param  \Illuminate\Validation\Validator  $validator
     * @return void
     */
    public function withValidator($validator)
    {
        $validator->after(function ($validator) {
            if ($this->somethingElseIsInvalid()) {
                $validator->errors()->add('field', 'Something is wrong with this field!');
            }
        });
    }

<a name="authorizing-form-requests"></a>
### Авторизация Form Request

Класс Form Request содержит в себе метод `authorize`. В этом методе можно проверить, имеет ли аутентифицированный пользователь права на выполнение данного запроса. Например, можно проверить, есть ли у пользователя право для добавления комментариев в блог:

    /**
     * Определить авторизован ли пользователь делать такой запрос.
     *
     * @return bool
     */
    public function authorize()
    {
        $comment = Comment::find($this->route('comment'));

        return $comment && $this->user()->can('update', $comment);
    }

Так как все Form Request расширяют базовый класс Request, мы можем использовать метод `user`, чтобы получить доступ к текущему пользователю.Так же обратите внимание на вызов метода `route`. Этот метод предоставляет доступ к параметрам URI, определенным в  роуте (в приведенном ниже примере это `{comment}`):

    Route::post('comment/{comment}');

Если метод `authorize` возвращает `false`, автоматически генерируется ответ с кодом 403 и метод контроллера не выполняется.

Если же логика авторизации организована в другом месте вашего приложения, просто верните `true` из метода `authorize`:

    /**
     * Определить авторизован ли пользователь делать такой запрос.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

<a name="customizing-the-error-format"></a>
### Настройка формата вывода ошибок

Если вы хотите настроить формат вывода ошибок валидации, которые будут заполнять flash-переменные при неудачном выполнении, переопредилите метод `formatErrors` в вашем базовом request (`App\Http\Requests\Request`). И не забывайте подключить класс `Illuminate\Contracts\Validation\Validator`:

    /**
     * {@inheritdoc}
     */
    protected function formatErrors(Validator $validator)
    {
        return $validator->errors()->all();
    }

<a name="customizing-the-error-messages"></a>
### Настройка сообщений об ошибках

Вы можете кастомизировать сообщения об ошибках, используя в form request метод `messages`. Этот метод должен возвращать массив атрибутов/правил и их соответствующие сообщения об ошибках:

    /**
     * Get the error messages for the defined validation rules.
     *
     * @return array
     */
    public function messages()
    {
        return [
            'title.required' => 'A title is required',
            'body.required'  => 'A message is required',
        ];
    }

<a name="manually-creating-validators"></a>
## Создание валидаторов вручную

Если вы не хотите использовать трейт `ValidatesRequests` и его метод  `validate`, можно создать экземпляр валидатора вручную с помощью [фасада](/docs/{{version}}/facades) `Validator`, используя метод `make`:

    <?php

    namespace App\Http\Controllers;

    use Validator;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Store a new blog post.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $validator = Validator::make($request->all(), [
                'title' => 'required|unique:posts|max:255',
                'body' => 'required',
            ]);

            if ($validator->fails()) {
                return redirect('post/create')
                            ->withErrors($validator)
                            ->withInput();
            }

            // Store the blog post...
        }
    }

Первый аргумент, передаваемый в метод `make`, получает данные для проверки. Вторым аргументом идут правилами проверки, которые должны применяться к данным.

После проверки, если валидация не будет пройдена, вы можете использовать метод `withErrors` для загрузки ошибок во flash-переменные. При использовании этого метода переменная `$errors` будет автоматически передаваться в ваши макеты, после перенаправления, что позволяет легко отображать данные пользователю. Метод `withErrors` принимает экземпляр валидатора и `MessageBag` или простой массив.

<a name="automatic-redirection"></a>
### Автоматическое перенаправление

Если вы хотите создать экземпляр валидации вручную, но все же воспользоваться автоматической переадресацией трейта `ValidatesRequest`, можно вызвать метод `validate` в существующим экземпляре. После того, как  проверка терпит неудачу, пользователь будет автоматически перенаправляться, в случае с AJAX-запросом, как и ранее в ответ отправится JSON:

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validate();

<a name="named-error-bags"></a>
### MessageBag

Если у вас есть несколько форм на одной странице, которые необходимо провалидировать, вам понадобится `MessageBag` — он позволяет получать сообщения об ошибках для определенной формы. Просто передайте имя в качестве второго аргумента `withErrors`:

    return redirect('register')
                ->withErrors($validator, 'login');

You may then access the named `MessageBag` instance from the `$errors` variable:

    {{ $errors->login->first('email') }}

<a name="after-validation-hook"></a>
### Хук после валидации

Валидатор также позволяет вам использовать функции обратного вызова после завершения всех проверок. Это позволяет легко выполнять дальнейшие проверки и даже добавить больше сообщений об ошибках в коллекции сообщений. Чтобы начать работу, используйте метод `after` на экземпляре валидатора:

    $validator = Validator::make(...);

    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });

    if ($validator->fails()) {
        //
    }

<a name="working-with-error-messages"></a>
## Работа с сообщениями об ошибках

После вызова метода `errors` в экземпляре валидатора, вы получаете экземпляр `Illuminate\Support\MessageBag`, который имеет целый ряд удобных методов для работы с сообщениями об ошибках. Переменная `$errors`, которая автоматически становится доступной для всех макетов, также является экземпляром класса `MessageBag`.

#### Извлечение первого для поля сообщения об ошибке

Чтобы получить первое сообщения об ошибке для заданного поля используйте метод `first`:

    $errors = $validator->errors();

    echo $errors->first('email');

#### Извлечение всех сообщений об ошибках для одного поля

Для того, чтобы получить массив всех сообщений об ошибках для одного поля, необходимо использовать метод `get`:

    foreach ($errors->get('email') as $message) {
        //
    }

Если выполняется проверка поля формы с массивом, можно получить все сообщения для каждого из элементов массива с помощью символа `*`:

    foreach ($errors->get('attachments.*') as $message) {
        //
    }

#### Получение всех сообщений об ошибках для всех полей

Чтобы извлечь массив всех сообщений для всех полей, используйте метод `all`:

    foreach ($errors->all() as $message) {
        //
    }

#### Определить наличие сообщения для определенного поля

Метод `has` может определять наличие сообщения об ошибках для данного поля:

    if ($errors->has('email')) {
        //
    }

<a name="custom-error-messages"></a>
### Пользовательские сообщения об ошибках

При необходимости, вы можете использовать свои сообщения об ошибках вместо значений по умолчанию. Существует несколько способов для указания кастомных сообщений. Во-первых, можно передать сообщения в качестве третьего аргумента в метод `Validator::make`:

    $messages = [
        'required' => 'The :attribute field is required.',
    ];

    $validator = Validator::make($input, $rules, $messages);

В этом примере `:attribute`будет заменен на имя проверяемого поля. Вы также можете использовать и другие строки-переменные. Пример:

    $messages = [
        'same'    => 'The :attribute and :other must match.',
        'size'    => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute must be between :min - :max.',
        'in'      => 'The :attribute must be one of the following types: :values',
    ];

#### Указание пользовательского сообщения для заданного атрибута

Иногда есть необходимость указать собственное сообщение для конкретного поля, это можно сделать с помощью синтаксиса с точкой. Просто укажите имя атрибута и текст сообщения:

    $messages = [
        'email.required' => 'We need to know your e-mail address!',
    ];

<a name="localization"></a>
#### Указание собственных сообщений в файлах локализации

Также можно определять сообщения в файле локализации вместо того, чтобы передавать их в валидатор напрямую. Для этого добавьте сообщения в массив `custom` файла локализации `resources/lang/xx/validation.php`.

    'custom' => [
        'email' => [
            'required' => 'We need to know your e-mail address!',
        ],
    ],

#### Указание пользовательских атрибутов в файлах локализации

Если вы хотите, чтобы `:attribute` был заменен на кастомное имя, можно указать в массиве `attributes` файле локализации `resources/lang/xx/validation.php`:

    'attributes' => [
        'email' => 'email address',
    ],

<a name="available-validation-rules"></a>
## Доступные правила валидации

Ниже приведен список всех доступных правил и их функции:

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">

[Accepted](#rule-accepted)
[Active URL](#rule-active-url)
[After (Date)](#rule-after)
[After Or Equal (Date)](#rule-after-or-equal)
[Alpha](#rule-alpha)
[Alpha Dash](#rule-alpha-dash)
[Alpha Numeric](#rule-alpha-num)
[Array](#rule-array)
[Before (Date)](#rule-before)
[Before Or Equal (Date)](#rule-before-or-equal)
[Between](#rule-between)
[Boolean](#rule-boolean)
[Confirmed](#rule-confirmed)
[Date](#rule-date)
[Date Format](#rule-date-format)
[Different](#rule-different)
[Digits](#rule-digits)
[Digits Between](#rule-digits-between)
[Dimensions (Image Files)](#rule-dimensions)
[Distinct](#rule-distinct)
[E-Mail](#rule-email)
[Exists (Database)](#rule-exists)
[File](#rule-file)
[Filled](#rule-filled)
[Image (File)](#rule-image)
[In](#rule-in)
[In Array](#rule-in-array)
[Integer](#rule-integer)
[IP Address](#rule-ip)
[JSON](#rule-json)
[Max](#rule-max)
[MIME Types](#rule-mimetypes)
[MIME Type By File Extension](#rule-mimes)
[Min](#rule-min)
[Nullable](#rule-nullable)
[Not In](#rule-not-in)
[Numeric](#rule-numeric)
[Present](#rule-present)
[Regular Expression](#rule-regex)
[Required](#rule-required)
[Required If](#rule-required-if)
[Required Unless](#rule-required-unless)
[Required With](#rule-required-with)
[Required With All](#rule-required-with-all)
[Required Without](#rule-required-without)
[Required Without All](#rule-required-without-all)
[Same](#rule-same)
[Size](#rule-size)
[String](#rule-string)
[Timezone](#rule-timezone)
[Unique (Database)](#rule-unique)
[URL](#rule-url)

</div>

<a name="rule-accepted"></a>
#### accepted

Поле должно быть в значении `yes`, `on` или `1`. Это полезно для проверки принятия правил и лицензий.

<a name="rule-active-url"></a>
#### active_url

Поле должно иметь действительную A или AAAA DNS-запись согласно функции PHP `dns_get_record`.

<a name="rule-after"></a>
#### after:_date_

Поле проверки должно быть после _date_. Строки приводятся к датам функцией `strtotime`:

    'start_date' => 'required|date|after:tomorrow'

Вместо того чтобы приводить строки к датам, вы можете указать другое поле для сравнения даты:

    'finish_date' => 'required|date|after:start_date'

<a name="rule-after-or-equal"></a>
#### after\_or\_equal:_date_

Поле проверки должно быть после или равно _date_. Для получения дополнительной информации смотрите правило [after](#rule-after)

<a name="rule-alpha"></a>
#### alpha

Поле должно содержать только алфавитные символы.

<a name="rule-alpha-dash"></a>
#### alpha_dash

Поле можно содержать только алфавитные символы, цифры, знаки подчёркивания `_` и дефисы `-`.

<a name="rule-alpha-num"></a>
#### alpha_num

Поле можно содержать только алфавитные символы и цифры.

<a name="rule-array"></a>
#### array

Поле должно быть PHP-массивом.

<a name="rule-before"></a>
#### before:_date_

Поле должно быть датой более ранней, чем заданная дата. Строки приводятся к датам функцией `strtotime`.

<a name="rule-before-or-equal"></a>
#### before\_or\_equal:_date_

Поле должно быть более ранней или равной заданной дате. Строки приводятся к датам функцией `strtotime`.

<a name="rule-between"></a>
#### between:_min_,_max_

Поле должно быть числом в диапазоне от _min_ до _max_. Размеры строк, чисел и файлов трактуются аналогично правилу [`size`](#rule-size).

<a name="rule-boolean"></a>
#### boolean

Поле должно быть логическим (булевым). Разрешенные значения: `true`, `false`, `1`, `0`, `"1"`, и `"0"`.

<a name="rule-confirmed"></a>
#### confirmed

Значение поля должно соответствовать значению поля с этим именем, плюс `foo_confirmation`. Например, если проверяется поле `password`, то на вход должно быть передано совпадающее по значению поле `password_confirmation`.

<a name="rule-date"></a>
#### date

Поле должно быть правильной датой в соответствии с PHP функцией `strtotime`.

<a name="rule-date-format"></a>
#### date_format:_format_

Поле должно соответствовать заданному _формату_. Необходимо **использовать** функцию `date` или `date_format` при проверке поля, но не обе.

<a name="rule-different"></a>
#### different:_field_

Значение проверяемого поля должно отличаться от значения поля _field_.

<a name="rule-digits"></a>
#### digits:_value_

Поле должно быть _числовым_ и иметь точную длину _значения_.

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

Длина значения поля проверки должна быть между _min_ и _max_.

<a name="rule-dimensions"></a>
#### dimensions

Файл изображения должен иметь ограничения согласно параметрам:

    'avatar' => 'dimensions:min_width=100,min_height=200'

Доступные ограничения: _min\_width_, _max\_width_, _min\_height_, _max\_height_, _width_, _height_, _ratio_.

Ограничение _ratio_ должно быть представлено как ширина к высоте. Это может быть обыкновенная (`3/2`) или десятичная (`1.5`) дробь:

    'avatar' => 'dimensions:ratio=3/2'

Поскольку это правило требует несколько аргументов, вы можете использовать метод `Rule::dimensions`:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'avatar' => [
            'required',
            Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
        ],
    ]);

<a name="rule-distinct"></a>
#### distinct

При работе с массивами, поле не должно иметь повторяющихся значений.

    'foo.*.id' => 'distinct'

<a name="rule-email"></a>
#### email

Поле должно быть корректным адресом e-mail.

<a name="rule-exists"></a>
#### exists:_table_,_column_

Поле должно существовать в указанной таблице базы данных.

#### Базовое использование правила Exists

    'state' => 'exists:states'

#### Указание пользовательского названия столбца

    'state' => 'exists:states,abbreviation'

Иногда может потребоваться подключение к базе данных и использование в запросе `exists`, этого можно добиться путем добавления к соединению название таблицы, используя синтаксис с точкой:

    'email' => 'exists:connection.staff,email'

Если бы вы хотите модифицировать запрос, можно использовать класс `Rule`, в данном примере мы будем использовать массив вместо знака `|`:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::exists('staff')->where(function ($query) {
                $query->where('account_id', 1);
            }),
        ],
    ]);

<a name="rule-file"></a>
#### file

Поле должно быть успешно загруженным файлом.

<a name="rule-filled"></a>
#### filled

Поле не должно быть пустым.

<a name="rule-image"></a>
#### image

Загруженный файл должен быть в формате jpeg, png, bmp, gif или svg.

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

Значение поля должно быть одним из перечисленных. Поскольку это правило иногда вынуждает вас использовать функцию `implode`, для этого случая есть метод `Rule::in`:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'zones' => [
            'required',
            Rule::in(['first-zone', 'second-zone']),
        ],
    ]);

<a name="rule-in-array"></a>
#### in_array:_anotherfield_

В массиве должны существовать значения _anotherfield_.

<a name="rule-integer"></a>
#### integer

Поле должно иметь корректное целочисленное значение.

<a name="rule-ip"></a>
#### ip

Поле должно быть корректным IP-адресом.

#### ipv4

Поле должно быть IPv4-адресом.

#### ipv6

Поле должно быть IPv6-адресом.

<a name="rule-json"></a>
#### json

Поле должно быть валидной строкой JSON.

<a name="rule-max"></a>
#### max:_value_

Значение поля должно быть меньше или равно _value_. Размеры строк, чисел и файлов трактуются аналогично правилу [`size`](#rule-size).

<a name="rule-mimetypes"></a>
#### mimetypes:_text/plain_,...

MIME-тип загруженного файла должен быть одним из перечисленных:

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'

Чтобы определить MIME-тип загруженного файла, фреймворк будет читать содержимое и пытаться угадать MIME-тип, который может отличаться от того, что указал пользователь.

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

MIME-тип загруженного файла должен быть одним из перечисленных.

#### Основное использование MIME-правила

    'photo' => 'mimes:jpeg,bmp,png'

Даже если необходимо только указать расширение, это правило проверяет MIME-тип файла, читая содержимое файла и пытаясь угадать его.

Полный список MIME-типов и соответствующие им расширения можно найти в: [https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="rule-min"></a>
#### min:_value_

Значение поля должно быть больше _value_. Размеры строк, чисел и файлов трактуются аналогично правилу [`size`](#rule-size).

<a name="rule-nullable"></a>
#### nullable

Поле может быть равно `null`. Это особенно полезно при проверке примитивов, такие как строки и целые числа, которые могут содержать `null` значения.

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

Поле не должно быть включено в заданный список значений. Метод `Rule::notIn` можно использовать для конструирования этого правила:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'toppings' => [
            'required',
            Rule::notIn(['sprinkles', 'cherries']),
        ],
    ]);

<a name="rule-numeric"></a>
#### numeric

Поле должно иметь корректное числовое или дробное значение.

<a name="rule-present"></a>
#### present

Поле для проверки должно присутствовать во входных данных, но может быть пустым.

<a name="rule-regex"></a>
#### regex:_pattern_

Поле должно соответствовать заданному регулярному выражению.

**Примечание:** Для использования `regex` может быть необходимо определить правила в виде массива вместо использования разделителя, особенно если регулярное выражение содержит символ разделителя.

<a name="rule-required"></a>
#### required

Проверяемое поле должно иметь непустое значение. Поле считается пустым, если одно из следующих значений верно:

<div class="content-list" markdown="1">

- Если значение равно `null`.
- Если значение — пустая строка.
- Если значение является пустым массивом или пустым объектом `Countable`.
- Если значение это загруженный файл без пути.

</div>

<a name="rule-required-if"></a>
#### required_if:_anotherfield_,_value_,...

Поле должно присутствовать и не быть пустым, если _anotherfield_ равно любому _value_.

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,...

Поле должно присутствовать и не быть пустым, за исключением случая, когда _anotherfield_ равно любому _value_.

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

Проверяемое поле должно иметь непустое значение, но только если присутствует хотя бы одно из перечисленных полей (foo, bar и т.д.).

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

Проверяемое поле должно иметь непустое значение, но _только_ если присутствуют все перечисленные поля (foo, bar и т.д.).

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

Проверяемое поле должно иметь непустое значение, но только если не присутствует хотя бы одно из перечисленных полей (foo, bar и т.д.).

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

Проверяемое поле должно иметь непустое значение, но только если не присутствуют все перечисленные поля (foo, bar и т.д.).

<a name="rule-same"></a>
#### same:_field_

Поле должно иметь то же значение, что и поле field.

<a name="rule-size"></a>
#### size:_value_

Поле должно иметь совпадающий с _value_ размер. Для строковых данных _value_ соответствует количество символов, для массива _size_ соответствует количеству элементов массива, для чисел — число, для файлов — размер в килобайтах.

<a name="rule-string"></a>
#### string

Поле должно быть строкой. Если вы хотите, чтобы поле было `null`, следует доолнитель указать это полю правило `nullable`.

<a name="rule-timezone"></a>
#### timezone

Поле должно содержать идентификатор часового пояса (таймзоны), один из перечисленных в php-функции `timezone_identifiers_list`.

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

Значение поля должно быть уникальным в заданной таблице базы данных. Если `column` не указано, то будет использовано имя поля.

**Указание пользовательского названия столбца:**

    'email' => 'unique:users,email_address'

**Пользовательское подключение к БД**

Иногда вам может понадобиться установить собственное соединение с базой данных, как замечено выше, параметр `unique:users`, будет использовать соединение по умолчанию. Чтобы переопределить это, укажите подключение и имя таблицы через синтаксис с точкой:

    'email' => 'unique:connection.users,email_address'

**Игнорирование ID при проверке на уникальность:**

Иногда потребуется игнорировать ID при проверке на уникальность. Например, рассмотрим обновление профиля пользователя, который включает в себя имя пользователя, адрес электронной почты и местоположение. Конечно, вы хотите убедиться, что адрес электронной почты является уникальным. Однако, если пользователь изменяет только имя и не изменяет электронную почту, нам не требуется вывод ошибки, но тем не менее возникнет исключение, поскольку пользователь уже является владельцем адреса электронной почты.

Для того, чтобы игнорировать ID пользователя, мы будем использовать класс `Rule` который позволяет гибко строить наши правила в таком случае. В примере, мы укажем правила в качестве массива вместо `|` символа-разделителя:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::unique('users')->ignore($user->id),
        ],
    ]);

Если таблица использует имя столбца первичного ключа помимо `id`, можно указать имя столбца при вызове метода `ignore`:

    'email' => Rule::unique('users')->ignore($user->id, 'user_id')

**Добавление дополнительных условий Where:**

Вы также можете указать дополнительные условия, используя метод `where`. Например, давайте добавим ограничение, которое проверяет, что `account_id` равно `1`:

    'email' => Rule::unique('users')->where(function ($query) {
        $query->where('account_id', 1);
    })

<a name="rule-url"></a>
#### url

Поле должно быть корректным URL.

<a name="conditionally-adding-rules"></a>
## Добавление правил с условиями

#### Валидация при наличии поля

Иногда вам нужно проверить некое поле **только** тогда, когда оно присутствует во входных данных. Для этого добавьте правило `sometimes`:

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

В примере выше для поля `email` будет запущена валидация только, когда оно присутствует в массиве `$data`.

> {tip} Если вы пытаетесь валидировать поле, которое всегда должно быть в наличии, но может быть пустым, смотрите [эту заметку о необязательных полях](#a-note-on-optional-fields)

#### Сложная составная проверка

Иногда возникает необходимость добавить правила с более сложной логикой проверки. Например, потребовать поле, только если другое поле имеет значение большее, чем 100. Или понадобится два поля, когда другое поле присутствует. Добавление этих правил не должно вызывать затруднения. Во-первых, создайте экземпляр `Validator` с вашими _постоянными правилами_, которые никогда не изменятся:

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

Давайте предположим, что наше веб-приложение для коллекционеров игр. Если коллекционер регистрирует в нашем приложении игру и он владеет больше чем 100 играми в данный момент, мы хотим, чтобы он объяснил, почему он владеет таким количеством игр. Возможно, он управляет магазином игр, или возможно, он просто любит их собирать. Чтобы добавить это требование, мы можем использовать метод `sometimes` в экземпляре валидатора:

    $v->sometimes('reason', 'required|max:500', function ($input) {
        return $input->games >= 100;
    });

Первый аргумент, переданный в метод `sometimes` это имя поля, которое мы условно проверяем. Второй аргумент — правила, которые мы хотим добавить. Если анонимная функция передается как третий аргумент, и возвращает значение `true`, то правила будут добавлены. Этот метод универсален для того, чтобы строить целый комплекс условных проверок. Вы можете даже добавить условные проверки на нескольких полях одновременно:

    $v->sometimes(['reason', 'cost'], 'required', function ($input) {
        return $input->games >= 100;
    });

> {tip} Параметр `$input` переданный в анонимную функцию, будет экземпляром `Illuminate\Support\Fluent` и может быть использован для доступа к вашим полям и файлам.

<a name="validating-arrays"></a>
## Валидация массивов

Проверка массива полей из формы не должна вызывать затруднений. Например, чтобы проверить, что каждая последующая электронная почта является уникальной, вы можете сделать следующее:

    $validator = Validator::make($request->all(), [
        'photos.profile' => 'required|image',
    ]);

Кроме того, вы можете использовать символ `*` в ваших языковых файлах, использовать одно сообщение для проверки массива полей:

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);

Таким же образом, вы можете использовать символ `*` при указании своих сообщений валидации в своих языковых файлах, значительно упрощая использование одного сообщения валидации для полей, основанных на массивах:

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique e-mail address',
        ]
    ],

<a name="custom-validation-rules"></a>
## Собственные правила валидации

Laravel предоставляет разнообразные и полезные правила для валидации. Однако, возможно, вам потребуется определить некоторые из своих собственных. Один из методов регистрации своих правил метод `extend` [`фасада`](/docs/{{version}}/facades) `Validator`. Давайте зарегистрируем этот метод в [`сервис-провайдере`](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Validator;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Предварительная загрузка любых сервисов приложения.
         *
         * @return void
         */
        public function boot()
        {
            Validator::extend('foo', function ($attribute, $value, $parameters, $validator) {
                return $value == 'foo';
            });
        }

        /**
         * Регистрация нового сервис-провайдера.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Анонимная функция получает четыре аргумента: имя проверяемого поля (`$attribute`), значение поля (`$value`), массив дополнительных параметров (`$parameters`) и экземпляр валидатора (`$validator`).

Класс и метод также можно передать методу `extend` вместо анонимной функции:

    Validator::extend('foo', 'FooValidator@validate');

#### Определение сообщения об ошибки

Необходимо будет также определить сообщение об ошибке для вашего правила. Вы можете сделать это, либо передавая его в виде массива строк в валидатор, либо добавив в файл локализации. Это сообщение должно помещаться на первом уровне массива, но не в массиве `custom`, который появляется только для сообщения об ошибке конкретного атрибута:

    "foo" => "Your input was invalid!",

    "accepted" => "The :attribute must be accepted.",

    // The rest of the validation error messages...

При создании своих правил проверки, может потребоваться определить места замены для сообщений об ошибках. Вы можете сделать это, реализовав через метод `replacer` в фасаде `Validator`. Действие необходимо определить внутри метода `boot` [`сервис-провайдера`](/docs/{{version}}/providers):

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Validator::extend(...);

        Validator::replacer('foo', function ($message, $attribute, $rule, $parameters) {
            return str_replace(...);
        });
    }

#### Скрытые расширения

По умолчанию, когда проверяемый атрибут отсутствует или содержит пустое значение, как в правиле [`required`](#rule-required), валидация не выполняется, в том числе и для ваших расширений. Например, [`unique`](#rule-unique) не будет выполнено для значения `null`:

    $rules = ['name' => 'unique'];

    $input = ['name' => null];

    Validator::make($input, $rules)->passes(); // true

Правило должно подразумевать, что атрибут обязателен, даже, если он пуст. Для создания «скрытых» расширений используйте метод `Validator::extendImplicit()`:

    Validator::extendImplicit('foo', function ($attribute, $value, $parameters, $validator) {
        return $value == 'foo';
    });

> {note} "Скрытое" расширение лишь _подразумевает_, что атрибут является обязательным. Будет ли это на самом деле недействительный или пустой атрибут, зависит только от вас.
