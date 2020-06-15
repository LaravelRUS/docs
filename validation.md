git 125bee02c051167ccc0b7bfac106574fda0868c5

---

# Валидация

- [Введение](#introduction)
- [Быстрый старт](#validation-quickstart)
    - [Определение роутов](#quick-defining-the-routes)
    - [Создание контроллера](#quick-creating-the-controller)
    - [Написание логики валидации](#quick-writing-the-validation-logic)
    - [Вывод ошибок валидации](#quick-displaying-the-validation-errors)
    - [Необязательные поля](#a-note-on-optional-fields)
- [Валидация в классах Form Request](#form-request-validation)
    - [Создание Form Requests](#creating-form-requests)
    - [Авторизация в Form Requests](#authorizing-form-requests)
    - [Модификация сообщений об ошибках](#customizing-the-error-messages)
    - [Модификация аттрибутов валидации](#customizing-the-validation-attributes)
    - [Подготовка входных данных для валидации](#prepare-input-for-validation)
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
    - [Используя объекты Rule](#using-rule-objects)
    - [Используя анонимную функцию](#using-closures)
    - [Используя расширения](#using-extensions)
    - [Неявные расширения](#implicit-extensions)

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
         * Show the form to create a new blog post.
         *
         * @return Response
         */
        public function create()
        {
            return view('post.create');
        }

        /**
         * Store a new blog post.
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

Теперь мы готовы написать в методе `store` логику для валидации новой записи в блоге. Для этого мы воспользуемся методом `validate` который предоставлен объектом `Illuminate\Http\Request`. В случае успешной валидации код будет выполняться в обычном режиме. Однако, если валидация не пройдена, будет выброшено исключение и корректный ответ с ошибкой автоматически отправится пользователю. В случае традиционного HTTP запроса будет сгенерирован ответ с редиректом, тогда как для AJAX запроса отправится JSON ответ.

Для лучшего понимания метода `validate`, вернемся обратно к методу `store`:

    /**
     * Store a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $validatedData = $request->validate([
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        // The blog post is valid...
    }

Как вы можете видеть, мы передаем желаемые правила валидации в метод `validate`. Отметим ещё раз, если валидация не пройдена, корректный ответ автоматически сгенерируется. Если валидация успешна, наш контроллер продолжит выполняться.

В качестве альтернативного  синтаксиса, правила валидации могут быть заданы с помощью массива правил вместо единой строки с разделителем `|`.

    $validatedData = $request->validate([
        'title' => ['required', 'unique:posts', 'max:255'],
        'body' => ['required'],
    ]);

Если вы хотите явно задать [MessageBag](#named-error-bags) в который должно попасть сообщение об ошибке, вы можете использовать метод `validateWithBag`:

    $request->validateWithBag('blog', [
        'title' => ['required', 'unique:posts', 'max:255'],
        'body' => ['required'],
    ])

#### Остановка после первой неудачной проверки

Иногда необходимо остановить проверку правил валидации для атрибута после первой неудачи. Чтобы это сделать, добавьте правило `bail` к атрибуту:

    $request->validate([
        'title' => 'bail|required|unique:posts|max:255',
        'body' => 'required',
    ]);

В этом примере, если для атрибута `title` не выполняется правило `unique`, правило `max` не будет проверено. Порядок валидации правил определяется порядком их назначения.

#### Заметка о вложенных атрибутах

Если данные HTTP запроса содержат "вложенные" параметры, можно указать их, используя синтаксис с точкой:

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);

<a name="quick-displaying-the-validation-errors"></a>
### Отображение ошибок валидации

Что если входящие данные не проходят проверку с учетом правил? Как упоминалось ранее, Laravel автоматически перенаправляет пользователя на предыдущую страницу. Кроме того, все ошибки валидации будут автоматически записаны во [flash-переменные](/docs/{{version}}/session#flash-data).

Опять же, обратите внимание, что мы не должны явно передавать сообщения об ошибках в шаблоне роута `GET`. Это потому, что Laravel будет проверять наличие ошибок в текущем сеансе и автоматически привязывать их к шаблону, если они доступны. Переменная `$errors` является экземпляром `Illuminate\Support\MessageBag`. Для получения дополнительных сведений о работе с этим объектом, [смотрите в документации](#working-with-error-messages).

> {tip} Переменная `$errors` привязывается к странице с помощью посредника `Illuminate\View\Middleware\ShareErrorsFromSession`, который входит в группу посредников `web`. **Когда посредник применен, переменная `$errors` всегда будет доступна в ваших шаблонах**, что позволяет считать переменную `$errors` всегда объявленной и безопасно её использовать.

В нашем примере пользователь будет перенаправлен в метод `create` нашего контроллера если валидация не пройдена, позволяя нам отобразить сообщения об ошибках в шаблоне:

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

#### Директива `@error`

Вы также можете использовать директиву `@error` чтобы быстро проверить наличие сообщений об ошибке валидации для конкретного атрибута. Внутри директивы `@error` вы можете вывести переменную `$message` для отображения сообщения ошибки:

    <!-- /resources/views/post/create.blade.php -->

    <label for="title">Post Title</label>

    <input id="title" type="text" class="@error('title') is-invalid @enderror">

    @error('title')
        <div class="alert alert-danger">{{ $message }}</div>
    @enderror

<a name="a-note-on-optional-fields"></a>
### Необязательные поля

По умолчанию Laravel содержит посредники `TrimStrings` и `ConvertEmptyStringsToNull` в глобальном стеке посредников. Они перечислены в классе `App\Http\Kernel`. По этой причине, зачастую, вам необходимо помечать необязательные поля как `nullable`, если вы не хотите, чтобы валидатор считал значения `null` как некорректные. Например:

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

В этом примере мы указываем что поле `publish_at` может быть `null` или должно содержать дату. Если модификатор `nullable` не добавлен в правила, валидатор будет считать `null` недопустимой датой.

<a name="quick-ajax-requests-and-validation"></a>
#### AJAX Запросы и валидация

В этом примере мы рассмотрели традиционную форму передачи данных. Однако, многие приложения используют AJAX запросы. При использовании метода `validate` для AJAX запроса, Laravel не будет генерировать ответ с редиректом. Вместо этого, Laravel создаст JSON ответ в котором будут содержаться все ошибки валидации. Этот JSON ответ будет отправлен с HTTP кодом 422.

<a name="form-request-validation"></a>
## Валидация в классах Form Request

<a name="creating-form-requests"></a>
### Создание Form Request

Для более сложных сценариев валидаций, будут более удобны `Form Requests`. Form Requests это специальные классы, которые содержат в себе логику проверки. Для создания класса, используйте artisan-команду `make:request`:

    php artisan make:request StoreBlogPost

Сгенерированный класс будет размещен в каталоге `app/Http/Requests`. Если этот каталог не существует, он будет создан. Давайте добавим несколько правил проверки в метод `rules`:

    /**
     * Get the validation rules that apply to the request.
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

> {tip} Вы можете указать любые зависимости, которые вам необходимы, в аргументах метода `rules`. Они будут автоматически подставлены через [сервис-контейнер](/docs/{{version}}/container).

Так как же здесь работают правила валидации? Все, что нужно сделать — это указать класс Form Request в аргументах метода вашего контроллера. Входящий запрос перед вызовом метода контроллера будет валидироваться автоматически, что позволит не загромождать контроллер логикой валидации:

    /**
     * Store the incoming blog post.
     *
     * @param  StoreBlogPost  $request
     * @return Response
     */
    public function store(StoreBlogPost $request)
    {
        // The incoming request is valid...

        // Retrieve the validated input data...
        $validated = $request->validated();
    }

Если валидация не пройдена, будет сгенерирован ответ с редиректом на предыдущую страницу. Ошибки будут записаны в сессию, чтобы они были доступны для отображения. Если запрос был выполнен с помощью AJAX, ответ, содержащий ошибки валидации в формате JSON, будет отправлен пользователю с HTTP кодом 422.

#### Добавление хуков в Form Request

Если вы хотите добавить хук "after" в Form Requests, можно использовать метод `withValidator`. Этот метод получает полностью сформированный валидатор, позволяя вызвать любой из его методов, прежде чем фактически применяются правила:

    /**
     * Configure the validator instance.
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
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        $comment = Comment::find($this->route('comment'));

        return $comment && $this->user()->can('update', $comment);
    }

Так как все Form Request расширяют базовый класс Request, мы можем использовать метод `user`, чтобы получить доступ к текущему пользователю. Так же обратите внимание на вызов метода `route`. Этот метод предоставляет доступ к параметрам URI, определенным в роуте (в приведенном ниже примере это `{comment}`):

    Route::post('comment/{comment}');

Если метод `authorize` возвращает `false`, автоматически генерируется ответ с кодом 403 и метод контроллера не выполняется.

Если же логика авторизации организована в другом месте вашего приложения, просто верните `true` из метода `authorize`:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

> {tip} Вы можете указать любые зависимости, которые вам необходимы, в аргументах метода `authorize`. Они будут автоматически подставлены через [сервис-контейнер](/docs/{{version}}/container).

<a name="customizing-the-error-messages"></a>
### Модификация сообщений об ошибках

Вы можете изменить сообщения об ошибках в рамках текущего класса Form Request, с помощью переопределения метода `messages`. Этот метод должен возвращать массив атрибутов/правил и соответствующие им сообщения об ошибке:

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

<a name="customizing-the-validation-attributes"></a>
### Модификация аттрибутов валидации

Если вы хотите чтобы часть `:attribute` сообщения об ошибке была заменена на определенное имя, вы можете указать желаемые имена переопределив метод `attributes`. Этот метод должен возвращать массив пар аттрибут / имя:

    /**
     * Get custom attributes for validator errors.
     *
     * @return array
     */
    public function attributes()
    {
        return [
            'email' => 'email address',
        ];
    }

<a name="prepare-input-for-validation"></a>
### Подготовка входных данных для валидации

Если вам необходимо подготовить какие-либо данные запроса перед применением валидации, вы можете использовать метод `prepareForValidation`:

    use Illuminate\Support\Str;

    /**
     * Prepare the data for validation.
     *
     * @return void
     */
    protected function prepareForValidation()
    {
        $this->merge([
            'slug' => Str::slug($this->slug),
        ]);
    }

<a name="manually-creating-validators"></a>
## Создание валидаторов вручную

Если вы не хотите использовать метод `validate` запроса, вы можете создать экземпляр валидатора вручную с помощью [фасада](/docs/{{version}}/facades) `Validator`. Метод `make` фасада создает новый экземпляр валидатора:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Validator;

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

Первым аргументом в метод `make` передаются данные подлежащие проверке. Второй аргумент — правила валидации, которые применяются к данным.

После проверки, если валидация не пройдена, вы можете использовать метод `withErrors` чтобы записать сообщения об ошибках в сессию. При использовании этого метода, переменная `$errors` будет автоматически передана в ваши шаблоны после редиректа, позволяя вам легко отобразить сообщения пользователю. Метод `withErrors` принимает экземпляр валидатора и `MessageBag` или просто массив.

<a name="automatic-redirection"></a>
### Автоматическое перенаправление

Если вы хотите создать экземпляр валидатора вручную, но все же использовать преимущество автоматического редиректа которое предлагает метод `validate` запроса, вы можете вызвать метод `validate` у существующего экземпляра валидатора. Если валидация не пройдена, пользователь будет автоматически перенаправлен или, в случае AJAX запроса, JSON ответ будет возвращен:

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

#### Получение первого для поля сообщения об ошибке

Чтобы получить первое сообщения об ошибке для заданного поля используйте метод `first`:

    $errors = $validator->errors();

    echo $errors->first('email');

#### Получение всех сообщений об ошибках для одного поля

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

#### Указание собственных атрибутов в файлах локализации

Если вы хотите чтобы часть `:attribute` сообщения об ошибке была заменена на определенное имя, вы можете указать желаемое имя в массиве `attributes` файла локализации `resources/lang/xx/validation.php`:

    'attributes' => [
        'email' => 'email address',
    ],

#### Указание собственных значений в файлах локализации

Иногда вам требуется заменить часть `:value` в сообщении об ошибке на другое значение. Например, рассмотрим следующее правило, которое указывает, что номер кредитной карты обязателен, если поле `payment_type` имеет значение `cc`:

    $request->validate([
        'credit_card_number' => 'required_if:payment_type,cc'
    ]);

Если это правило нарушено, возникнет ошибка содержащая следующее сообщение:

    The credit card number field is required when payment type is cc.

Вместо отображения `cc` в качесте значения типа оплаты, вы можете задать собственное представление в файле локализации `validation`, определив массив `values`:

    'values' => [
        'payment_type' => [
            'cc' => 'credit card'
        ],
    ],

Теперь, при нарушении правила, сообщение будет иметь следующий вид:

    The credit card number field is required when payment type is credit card.

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
[Bail](#rule-bail)
[Before (Date)](#rule-before)
[Before Or Equal (Date)](#rule-before-or-equal)
[Between](#rule-between)
[Boolean](#rule-boolean)
[Confirmed](#rule-confirmed)
[Date](#rule-date)
[Date Equals](#rule-date-equals)
[Date Format](#rule-date-format)
[Different](#rule-different)
[Digits](#rule-digits)
[Digits Between](#rule-digits-between)
[Dimensions (Image Files)](#rule-dimensions)
[Distinct](#rule-distinct)
[E-Mail](#rule-email)
[Ends With](#rule-ends-with)
[Exclude If](#rule-exclude-if)
[Exclude Unless](#rule-exclude-unless)
[Exists (Database)](#rule-exists)
[File](#rule-file)
[Filled](#rule-filled)
[Greater Than](#rule-gt)
[Greater Than Or Equal](#rule-gte)
[Image (File)](#rule-image)
[In](#rule-in)
[In Array](#rule-in-array)
[Integer](#rule-integer)
[IP Address](#rule-ip)
[JSON](#rule-json)
[Less Than](#rule-lt)
[Less Than Or Equal](#rule-lte)
[Max](#rule-max)
[MIME Types](#rule-mimetypes)
[MIME Type By File Extension](#rule-mimes)
[Min](#rule-min)
[Not In](#rule-not-in)
[Not Regex](#rule-not-regex)
[Nullable](#rule-nullable)
[Numeric](#rule-numeric)
[Password](#rule-password)
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
[Sometimes](#conditionally-adding-rules)
[Starts With](#rule-starts-with)
[String](#rule-string)
[Timezone](#rule-timezone)
[Unique (Database)](#rule-unique)
[URL](#rule-url)
[UUID](#rule-uuid)

</div>

<a name="rule-accepted"></a>
#### accepted

Проверяемое поле должно быть в значении _yes_, _on_, _1_ или _true_. Это полезно для проверки принятия правил и лицензий.

<a name="rule-active-url"></a>
#### active_url

Проверяемое поле должно иметь действительную A или AAAA DNS-запись согласно функции PHP `dns_get_record`. Перед передачей в `dns_get_record`, доменное имя (hostname) предоставленного URL извлекается, используя функцию PHP `parse_url`.

<a name="rule-after"></a>
#### after:_date_

Проверяемое поле должно быть после указанной даты. Даты передаются в функцию PHP `strtotime`:

    'start_date' => 'required|date|after:tomorrow'

Вместо передачи строковой даты для вычисления с помощью `strtotime`, вы можете указать другое поле для сравнения с датой:

    'finish_date' => 'required|date|after:start_date'

<a name="rule-after-or-equal"></a>
#### after\_or\_equal:_date_

Проверяемое поле должно быть после или равно указанной даты. Для получения дополнительной информации смотрите правило [after](#rule-after)

<a name="rule-alpha"></a>
#### alpha

Проверяемое поле должно содержать только буквенные символы.

<a name="rule-alpha-dash"></a>
#### alpha_dash

Проверяемое поле должно содержать только буквенные символы, цифры, знаки подчёркивания `_` и дефисы `-`.

<a name="rule-alpha-num"></a>
#### alpha_num

Проверяемое поле можно содержать только буквенные символы и цифры.

<a name="rule-array"></a>
#### array

Проверяемое поле должно быть PHP-массивом.

<a name="rule-bail"></a>
#### bail

Останавливает проверку правил валидации после первой ошибки.

<a name="rule-before"></a>
#### before:_date_

Проверяемое поле должно иметь значение, предшествующее указанной датой. Даты передаются в функцию PHP `strtotime`. Дополнительно, как и в правиле [`after`](#rule-after), имя другого валидируемого поля может быть указано как значение `date`.

<a name="rule-before-or-equal"></a>
#### before\_or\_equal:_date_

Проверяемое поле должно иметь значение, предшествующее или равное указанной дате. Даты передаются в функцию PHP `strtotime`. Дополнительно, как и в правиле [`after`](#rule-after), имя другого валидируемого поля может быть указано как значение `date`.

<a name="rule-between"></a>
#### between:_min_,_max_

Проверяемое поле должно иметь размер в диапазоне от _min_ до _max_. Размеры строк, чисел и файлов трактуются аналогично правилу [`size`](#rule-size).

<a name="rule-boolean"></a>
#### boolean

Проверяемое поле должно быть логическим (булевым). Разрешенные значения: `true`, `false`, `1`, `0`, `"1"`, и `"0"`.

<a name="rule-confirmed"></a>
#### confirmed

Проверяемое поле должно иметь совпадающее поле `foo_confirmation`. Например, если проверяется поле `password`, совпадающее поле `password_confirmation` должно присутствовать в данных.

<a name="rule-date"></a>
#### date

Проверяемое поле должно быть правильной, не относительной датой в соответствии с PHP функцией `strtotime`.

<a name="rule-date-equals"></a>
#### date_equals:_date_

Проверяемое поле должно быть равно указанной дате. Даты передаются в функцию PHP `strtotime`.

<a name="rule-date-format"></a>
#### date_format:_format_

Проверяемое поле должно соответствовать заданному _формату_. Необходимо **использовать** функцию `date` или `date_format` при проверке поля, но не обе. Это правило поддерживает все форматы доступные PHP классу [DateTime](https://www.php.net/manual/ru/class.datetime.php).

<a name="rule-different"></a>
#### different:_field_

Проверяемое поле должно иметь значение отличное от _field_.

<a name="rule-digits"></a>
#### digits:_value_

Проверяемое поле должно быть _числовым_ и иметь точную длину _value_.

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

Проверяемое поле должно быть _числовым_ и иметь длину между _min_ и _max_.

<a name="rule-dimensions"></a>
#### dimensions

Проверяемый файл должен быть изображением, которое соответствует ограничением, указанным в параметрах правила:

    'avatar' => 'dimensions:min_width=100,min_height=200'

Доступные ограничения: _min\_width_, _max\_width_, _min\_height_, _max\_height_, _width_, _height_, _ratio_.

Ограничение _ratio_ должно быть представлено как ширина, деленная на высоту. Это может быть обыкновенная (`3/2`) или десятичная (`1.5`) дробь:

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

Проверяемое поле должно иметь формат e-mail адреса. Под капотом, это правило использует пакет [`egulias/email-validator`](https://github.com/egulias/EmailValidator) для валидации e-mail адреса. По умолчанию применяется валидатор `RFCValidation`, но вы можете применить другие способы валидации при желании:

    'email' => 'email:rfc,dns'

Пример выше применит валидации `RFCValidation` и `DNSCheckValidation`. Полный список доступных способов валидации:

<div class="content-list" markdown="1">
- `rfc`: `RFCValidation`
- `strict`: `NoRFCWarningsValidation`
- `dns`: `DNSCheckValidation`
- `spoof`: `SpoofCheckValidation`
- `filter`: `FilterEmailValidation`
</div>

Валидатор `filter`, который использует PHP функцию `filter_var` под капотом, предоставляется с Laravel и является поведением по умолчанию для версии Laravel до 5.8. Валидаторы `dns` и `spoof` требуют PHP расширения `intl`.

<a name="rule-ends-with"></a>
#### ends_with:_foo_,_bar_,...

Проверяемое поле должно заканчиваться одним из указанных значений.

<a name="rule-exclude-if"></a>
#### exclude_if:_anotherfield_,_value_

Проверяемое поле будет исключено из данных запроса, которые возвращают методы `validate` и `validated`, если поле _anotherfield_ равно _value_.

<a name="rule-exclude-unless"></a>
#### exclude_unless:_anotherfield_,_value_

Проверяемое поле будет исключено из данных запроса, которые возвращают методы `validate` и `validated`, если поле _anotherfield_ не равно _value_.

<a name="rule-exists"></a>
#### exists:_table_,_column_

Проверяемое поле должно существовать в указанной таблице базы данных.

#### Базовое использование правила Exists

    'state' => 'exists:states'

Если параметр `column` не указан, будет использовано имя поля.

#### Указание пользовательского названия столбца

    'state' => 'exists:states,abbreviation'

Иногда вам может потребоваться указать конкретное соединение с базой данных, которое будет использовано для запроса `exists`. Это можно сделать, добавив имя соединения к названию таблицы используя точку для разделения:

    'email' => 'exists:connection.staff,email'

Вместо непосредственного указания имени таблицы можно указать Eloquent модель, которая будет использована для определения имени таблицы:

    'user_id' => 'exists:App\User,id'

Если вы хотите модифицировать запрос, выполняемый правилом валидации, вы можете использовать класс `Rule` для тонкого определения правила. В этом примере мы также укажем правила валидации в виде массива вместо использования символа `|` для разделения:

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

Проверяемое поле должно быть успешно загруженным файлом.

<a name="rule-filled"></a>
#### filled

Проверяемое поле не должно быть пустым, когда оно присутствует.

<a name="rule-gt"></a>
#### gt:_field_

Проверяемое поле должно быть больше заданного поля _field_. Оба поля должны быть одного типа. Строки, числа, массивы и файлы оцениваются по тем же принципам что и в правиле [`size`](#rule-size).

<a name="rule-gte"></a>
#### gte:_field_

Проверяемое поле должно быть больше или равно заданному полю _field_. Оба поля должны быть одного типа. Строки, числа, массивы и файлы оцениваются по тем же принципам что и в правиле [`size`](#rule-size).

<a name="rule-image"></a>
#### image

Проверяемый файл должен быть изображением (jpeg, png, bmp, gif, svg или webp).

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

Проверяемое поле должно быть включено в заданный список значений. Так как это правило часто требует применять функцию `implode` к массиву, то для удобного построения правила можно использовать метод `Rule::in`:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'zones' => [
            'required',
            Rule::in(['first-zone', 'second-zone']),
        ],
    ]);

<a name="rule-in-array"></a>
#### in_array:_anotherfield_.*

Проверяемое поле должно существовать среди значений поля _anotherfield_.

<a name="rule-integer"></a>
#### integer

Проверяемое поле должно быть целочисленным.

> {note} Данное правило валидации не проверяет что входные данные являются переменной типа integer, а только то, что входные данные являются строкой или числовым типом, содержащим целое число.

<a name="rule-ip"></a>
#### ip

Проверяемое поле должно быть IP адресом.

#### ipv4

Проверяемое поле должно быть IPv4 адресом.

#### ipv6

Проверяемое поле должно быть IPv6 адресом.

<a name="rule-json"></a>
#### json

Проверяемое поле должно быть валидной JSON строкой.

<a name="rule-lt"></a>
#### lt:_field_

Проверяемое поле должно быть меньше заданного поля _field_. Оба поля должны быть одного типа. Строки, числа, массивы и файлы оцениваются по тем же принципам что и в правиле [`size`](#rule-size).

<a name="rule-lte"></a>
#### lte:_field_

Проверяемое поле должно быть меньше или равно заданному полю _field_. Оба поля должны быть одного типа. Строки, числа, массивы и файлы оцениваются по тем же принципам что и в правиле [`size`](#rule-size).

<a name="rule-max"></a>
#### max:_value_

Проверяемое поле должно быть меньше или равно значению _value_. Строки, числа, массивы и файлы оцениваются по тем же принципам что и в правиле [`size`](#rule-size).

<a name="rule-mimetypes"></a>
#### mimetypes:_text/plain_,...

Проверяемый файл должен иметь один из перечисленных MIME-типов:

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'

Для определения MIME-типа загруженного файла, содержимое файла будет прочитано, а фреймворк попытается угадать MIME-тип, который может отличаться от типа, предоставленного пользователем.

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

Проверяемый файл должен иметь MIME-тип, соответствующий одному из перечисленных расширений.

#### Базовое использование MIME-правила

    'photo' => 'mimes:jpeg,bmp,png'

Несмотря на то, что вам нужно лишь указать расширения, это правило на самом деле проверяет MIME-тип файла, читая его содержимое и угадывая его MIME тип.

Полный список типов MIME и их соответствующих расширений можно найти на следующей странице: [https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="rule-min"></a>
#### min:_value_

Проверяемое поле должно иметь минимальное значение _value_. Строки, числа, массивы и файлы оцениваются по тем же принципам что и в правиле [`size`](#rule-size).

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

Проверяемое поле не должно быть включено в заданный список значений. Для удобного построения правила может быть использован метод `Rule::notIn`:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'toppings' => [
            'required',
            Rule::notIn(['sprinkles', 'cherries']),
        ],
    ]);

<a name="rule-not-regex"></a>
#### not_regex:_pattern_

Проверяемое поле не должно соответствовать заданному регулярному выражению.

Внутренне это правило использует функцию PHP `preg_match`. Указанный шаблон должен соблюдать то же форматированние, которое требуется для `preg_match` и, таким образом, включать в себя допустимые разделители. Например: `'email' => 'not_regex:/^.+$/i'`.

**Примечание:** При использовании шаблонов `regex` / `not_regex` может потребоваться указать правила в масиве вместо использования разделителя `|`, особенно если регулярное выражение содержит символ `|`.

<a name="rule-nullable"></a>
#### nullable

Проверяемое поле может быть `null`. Это особенно полезно при валидации примитивов, таких как строки и целые числа, которые могут содержать `null` значения.

<a name="rule-numeric"></a>
#### numeric

Проверяемое поле должно быть числовым.

<a name="rule-password"></a>
#### password

Проверяемое поле должно совпадать с паролем аутентифицированного пользователя. Вы можете указать гвард аутентификации (authentication guard), используя первый параметр правила:

    'password' => 'password:api'

<a name="rule-present"></a>
#### present

Проверяемое поле должно присутствовать во входных данных, но может быть пустым.

<a name="rule-regex"></a>
#### regex:_pattern_

Проверяемое поле должно соответствовать заданному регулярному выражению.

Внутренне это правило использует функцию PHP `preg_match`. Указанный шаблон должен соблюдать то же форматированние, которое требуется для `preg_match` и, таким образом, включать в себя допустимые разделители. Например: `'email' => 'not_regex:/^.+$/i'`.

**Примечание:** При использовании шаблонов `regex` / `not_regex` может потребоваться указать правила в масиве вместо использования разделителя `|`, особенно если регулярное выражение содержит символ `|`.

<a name="rule-required"></a>
#### required

Проверяемое поле должно присутствовать во входных данных и не должно быть пустым. Поле считается "пустым", если одно из следующих условий является верным:

<div class="content-list" markdown="1">

- Значение равно `null`.
- Значение — пустая строка.
- Значения — пустой массив или пустой объект `Countable`.
- Значение — загруженный файл без пути.

</div>

<a name="rule-required-if"></a>
#### required_if:_anotherfield_,_value_,...

Проверяемое поле должно присутствовать и не должно быть пустым, если поле _anotherfield_ равно любому значению _value_.

Если вы хотите создать более сложное услови для правила `required_if`, вы можете использовать метод `Rule::requiredIf`. Этот метод принимает boolean или функцию. При передаче функции, функция должна возвращать `true` или `false`, чтобы указать, является ли проверяемое поле обязательным:

    use Illuminate\Validation\Rule;

    Validator::make($request->all(), [
        'role_id' => Rule::requiredIf($request->user()->is_admin),
    ]);

    Validator::make($request->all(), [
        'role_id' => Rule::requiredIf(function () use ($request) {
            return $request->user()->is_admin;
        }),
    ]);

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,...

Проверяемое поле должно присутствовать и не должно быть пустым, если поле _anotherfield_ не равно любому значению _value_.

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

Проверяемое поле должно присутствовать и не должно быть пустым _только если_ присутствует любое из остальных указанных полей.

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

Проверяемое поле должно присутствовать и не должно быть пустым _только если_ присутствуют все остальные указанные поля.

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

Проверяемое поле должно присутствовать и не должно быть пустым _только тогда_ когда нет ни одного из остальных указанных полей.

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

Проверяемое поле должно присутствовать и не должно быть пустым _только тогда_ когда все остальные указанные поля не присутствуют.

<a name="rule-same"></a>
#### same:_field_

Заданное поле _field_ должно соответствовать проверяемому полю.

<a name="rule-size"></a>
#### size:_value_

Проверяемое поле должно иметь размер, соответствующий заданному значению _value_. Для строковых данных _value_ соответствует количеству символов. Для числовых данных, _value_ соответствует заданному целому значению (атрибут также должен иметь правило `numeric` или `integer`). Для массива _size_ соответствует значению функции `count`. Для файлов _size_ соответствует размеру файла в килобайтах. Рассмотрим некоторые примеры:

    // Проверим, что длина строки ровно 12 символов...
    'title' => 'size:12';

    // Проверим, что предоставленное целое число равно 10...
    'seats' => 'integer|size:10';

    // Проверим, что в массиве ровно 5 элементов...
    'tags' => 'array|size:5';

    // Проверим, что загруженный файл имеет размер ровно 512 килобайт...
    'image' => 'file|size:512';

<a name="rule-starts-with"></a>
#### starts_with:_foo_,_bar_,...

Проверяемое поле должно начинаться с одного из заданных значений.

<a name="rule-string"></a>
#### string

Проверяемое поле должно быть строкой. Если вы хотите разрешить значение `null`, следует добавить этому полю правило `nullable`.

<a name="rule-timezone"></a>
#### timezone

Проверяемое поле должно быть валидным идентификатором часового пояса в соответствии с PHP-функцией `timezone_identifiers_list`.

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

Проверяемое поле не должно существовать в данной таблице базы данных.

**Указание таблицы/названия столбца:**

Вместо непосредственного указания имени таблицы можно указать Eloquent модель, которая будет использована для определения имени таблицы:

    'user_id' => 'exists:App\User,id'

Параметр `column` может быть использован для указания соответствующего столбца БД. Если параметр `column` не указан, будет использовано имя поля.

    'email' => 'unique:users,email_address'

**Выбор подключения к базе данных**

Иногда вам может потребоваться указать конкретное соединение с базой данных, которое будет использовано при валидации. Как видно выше, задание правила `unique:users` будет использовать подключение по умолчанию для запросов к базе данных. Чтобы переопределить это, укажите соединение и название таблицы используя точку как разделитель:

    'email' => 'unique:connection.users,email_address'

**Игнорирование ID при проверке на уникальность:**

Иногда потребуется игнорировать ID при проверке на уникальность. Например, рассмотрим обновление профиля пользователя, который включает в себя имя пользователя, адрес электронной почты и местоположение. Конечно, вы хотите убедиться, что адрес электронной почты является уникальным. Однако, если пользователь изменяет только имя и не изменяет электронную почту, нам не требуется вывод ошибки, но тем не менее возникнет исключение, поскольку пользователь уже является владельцем адреса электронной почты.

Чтобы игнорировать пользователя с определенным ID, мы воспользуемся классом `Rule` для удобного определения правила. В этом примере мы также укажем правила валидации в виде массива вместо использования символа `|` для разделения правил:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::unique('users')->ignore($user->id),
        ],
    ]);

> {note} Вы никогда не должны передавать в метод `ignore` какие-либо данные пришедшие от пользователя. Вместо этого, вы должны передавать только сгенерированный системой уникальный ID, такой как авто-инкриментный ID или UUID из экземпляра модели Eloquent. В противном случае ваше приложение будет уязвимо для SQL-инъекции.

Вместо того, чтобы передавать значение ключа модели методу `ignore`, можно передать экземпляр модели целиком. Laravel автоматически извлечет ключ из модели:

    Rule::unique('users')->ignore($user)

Если ваша таблица использует имя столбца первичного ключа, отличное от `id`, вы можете указать имя столбца при вызове метода `ignore`:

    Rule::unique('users')->ignore($user->id, 'user_id')

По умолчанию правило `unique` проверяет уникальность колонки, соответствующей имени проверяемого атрибута. Однако, Вы можете передать другому аргументу имя колонки в качестве второго аргумента в метод `unique`:

    Rule::unique('users', 'email_address')->ignore($user->id),

**Добавление дополнительных условий Where:**

Вы также можете указать дополнительные ограничения в запросе, используя метод `where`. Например, добавим ограничение, которое проверяет, что значение `account_id` равно `1`:

    'email' => Rule::unique('users')->where(function ($query) {
        return $query->where('account_id', 1);
    })

<a name="rule-url"></a>
#### url

Проверяемое поле должно быть действительным URL-адресом.

<a name="rule-uuid"></a>
#### uuid

Проверяемое поле должно быть действительным универсальным уникальным идентификатором (UUID) по RFC 4122 (version 1, 3, 4 или 5).

<a name="conditionally-adding-rules"></a>
## Добавление правил с условиями

#### Валидация при наличии поля

В некоторых ситуациях, необходимо выполнить проверку поля **только** тогда, когда оно присутствует во входном массиве. Чтобы быстро это сделать, добавьте правило `sometimes` в список правил:

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

В приведенном выше примере поле `email` будет проверено только в том случае, если оно присутствует в массиве `$data`.

> {tip} Если вы пытаетесь проверить поле, которое должно присутствовать всегда, но может быть пустым, обратите внимание на заметку про [необязательные поля](#a-note-on-optional-fields)

#### Сложная проверка с условиями

Иногда может возникнуть желание добавить правила проверки, основанные на более сложной условной логике. Например, вы можете захотеть потребовать заданное поле только в том случае, если другое поле имеет большее значение, чем 100. Или же вам могут понадобиться два поля с заданным значением только в том случае, если другое поле присутствует. Добавление этих правил проверки не должно быть проблемой. Во-первых, создайте экземпляр класса `Validator` с вашими _static rules_, которые никогда не меняются:

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

Предположим, что наше веб-приложение предназначено для коллекционеров игр. Если игровой коллектор регистрируется в нашем приложении и ему принадлежит более 100 игр, мы хотим, чтобы он объяснил, почему ему принадлежит так много игр. Например, возможно, он управляет магазином по перепродаже игр, или, может быть, он просто обожает их собирать. Чтобы добавить это требование, мы можем использовать метод `'sometimes` на экземпляре класса `Validator`.

    $v->sometimes('reason', 'required|max:500', function ($input) {
        return $input->games >= 100;
    });

Первый аргумент, переданный в метод `sometimes` — это имя поля, которое мы проверяем при условии. Второй аргумент — правила, которые мы хотим добавить. Если функция, переданная в качестве третьего аргумента, возвращает `true`, то правила будут добавлены. Этот метод делает построение сложных условных валидаций легким делом. Можно даже добавлять условные валидации сразу для нескольких полей:

    $v->sometimes(['reason', 'cost'], 'required', function ($input) {
        return $input->games >= 100;
    });

> {tip} Параметр `$input` переданный в анонимную функцию, будет экземпляром класса `Illuminate\Support\Fluent` и может быть использован для доступа к входным данным и файлам.

<a name="validating-arrays"></a>
## Валидация массивов

Проверка массива полей из формы не должна вызывать затруднений. Для проверки атрибутов внутри массива можно использовать точечную запись (dot notation). Например, если входящий HTTP-запрос содержит поле `photos[profile]`, вы можете проверить его таким образом:

    $validator = Validator::make($request->all(), [
        'photos.profile' => 'required|image',
    ]);

Вы также можете проверить каждый элемент массива. Например, чтобы проверить, что каждый email в данном массивe уникалeн, можно сделать следующее:

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);

Аналогичным образом, вы можете использовать символ `*` при указании своих сообщений в языковых файлах, что сделает использование одного сообщения для полей массива достаточно простым:

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique e-mail address',
        ]
    ],

<a name="custom-validation-rules"></a>
## Собственные правила валидации

<a name="using-rule-objects"></a>
### Используя объекты Rule

Laravel предоставляет множество полезных правил проверки; однако, вы можете создать их самостоятельно. Одним из способов регистрации собственных правил валидации является использование объектов правил. Чтобы сгенерировать новый объект правила, вы можете использовать команду Artisan `make:rule`. Используем эту команду для генерации правила, которое проверяет, что строка в верхнем регистре. Laravel поместит новое правило в каталог `app/Rules`:

    php artisan make:rule Uppercase

После создания правила мы готовы определить его поведение. Объект правила содержит два метода: `passes` и `message`. Метод `passes` получает значение и имя атрибута и должен возвращать `true` или `false` в зависимости от того, является ли значение атрибута корректным или нет. Метод `message` должен возвращать сообщение об ошибке валидации, которое должно использоваться в случае ошибки валидации:

    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\Rule;

    class Uppercase implements Rule
    {
        /**
         * Determine if the validation rule passes.
         *
         * @param  string  $attribute
         * @param  mixed  $value
         * @return bool
         */
        public function passes($attribute, $value)
        {
            return strtoupper($value) === $value;
        }

        /**
         * Get the validation error message.
         *
         * @return string
         */
        public function message()
        {
            return 'The :attribute must be uppercase.';
        }
    }

Вы можете использовать хелпер `trans` в методе `message`, если хотите вернуть сообщение об ошибке из ваших файлов перевода:

    /**
     * Get the validation error message.
     *
     * @return string
     */
    public function message()
    {
        return trans('validation.uppercase');
    }

После того, как правило определено, вы можете использовать его в валидаторе, передав экземпляр правила вместе с другими правилами валидации:

    use App\Rules\Uppercase;

    $request->validate([
        'name' => ['required', 'string', new Uppercase],
    ]);

<a name="using-closures"></a>
### Используя анонимную функцию

Если вам нужна функциональность правила только в одном месте приложения, вы можете использовать анонимную функцию вместо объекта правила. Функция получает имя атрибута, значение атрибута и функцию обратного вызова `$fail`, которая должна быть вызвана, если проверка не прошла успешно:

    $validator = Validator::make($request->all(), [
        'title' => [
            'required',
            'max:255',
            function ($attribute, $value, $fail) {
                if ($value === 'foo') {
                    $fail($attribute.' is invalid.');
                }
            },
        ],
    ]);

<a name="using-extensions"></a>
### Используя расширения

Еще одним методом создания собственных правил валидации является использование метода `extend` у [фасада](/docs/{{version}}/facades) `Validator`. Используем этот метод внутри [сервис-провайдера](/docs/{{version}}/providers) для регистрации нашего правила валидации:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Validator;

    class AppServiceProvider extends ServiceProvider
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
            Validator::extend('foo', function ($attribute, $value, $parameters, $validator) {
                return $value == 'foo';
            });
        }
    }

Анонимная функция нашего валидатора получает четыре аргумента: имя проверяемого атрибута `$attribute`, значение `$value` атрибута, массив параметров `$parameters`, переданных в правило, и экземпляр класса `Validator`.

Вы также можете передать класс и метод в `extend` вместо функции:

    Validator::extend('foo', 'FooValidator@validate');

#### Определение сообщения об ошибки

Вам также нужно будет определить сообщение об ошибке для вашего правила. Вы можете сделать это либо с помощью встроенного массива сообщений, либо добавив запись в файл локализации. Эти сообщения должны быть размещены на первом уровне массива, а не в массиве `custom`, который предназначен только для сообщений об ошибках, специфичных для конкретных атрибутов:

    "foo" => "Your input was invalid!",

    "accepted" => "The :attribute must be accepted.",

    // The rest of the validation error messages...

При создании правил валидации, иногда может понадобиться определить плейсхолдеры для сообщений об ошибках. Вы можете сделать это, создав валидатор, как описано выше, а затем вызвать метод `replacer` у фасада `Validator`. Это можно сделать в рамках метода `boot` [сервис провайдера](/docs/{{version}}/providers):

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

<a name="implicit-extensions"></a>
### Неявные расширения

По умолчанию, когда проверяемый атрибут отсутствует или содержит пустую строку, обычные правила проверки, включая пользовательские расширения, не выполняются. Например, правило [`unique`](#rule-unique) не будет запущено для пустой строки:

    $rules = ['name' => 'unique:users,name'];

    $input = ['name' => ''];

    Validator::make($input, $rules)->passes(); // true

Для того, чтобы правило выполнялось, даже когда атрибут пуст, необходимо подразумевать, что атрибут является обязательным. Для создания такого "неявного" расширения используйте метод `Validator::extensionImplicit()`:

    Validator::extendImplicit('foo', function ($attribute, $value, $parameters, $validator) {
        return $value == 'foo';
    });

> {note} "Неявное" расширение только _подразумевает_, что атрибут является обязательным. Действительно ли оно недопускает отсутствующее или пустое значение, зависит от вас.

#### Неявные объекты Rule

Если вы хотите, чтобы объект правила выполнялся, когда атрибут пуст, то вы должны реализовать интерфейс `Illuminate\Contracts\Validation\ImplicitRule`. Этот интерфейс служит маркером для валидатора, поэтому он не содержит методов, которые необходимо реализовать.
