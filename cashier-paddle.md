git 6ac13f37adbed3ce6a6532fd790f70bd731b8571

---

# Laravel Cashier (Paddle)

- [Вступление](#introduction)
- [Обновление Cashier](#upgrading-cashier)
- [Установка](#installation)
    - [Paddle Sandbox](#paddle-sandbox)
    - [Миграции](#database-migrations)
- [Конфигурация](#configuration)
    - [Оплачиваемая модель](#billable-model)
    - [API ключи](#api-keys)
    - [Paddle JS](#paddle-js)
    - [Конфигурация валюты](#currency-configuration)
    - [Переопределение моделей по умолчанию](#overriding-default-models)
- [Основные концепции](#core-concepts)
    - [Платежные ссылки](#pay-links)
    - [Онлайн-оформление заказа](#inline-checkout)
    - [Идентификация пользователя](#user-identification)
- [Цены](#prices)
- [Клиенты](#customers)
    - [Параметры по умолчанию](#customer-defaults)
- [Подписки](#subscriptions)
    - [Создание подписок](#creating-subscriptions)
    - [Проверка статуса подписки](#checking-subscription-status)
    - [Единовременные платежи за подписку](#subscription-single-charges)
    - [Обновление платежной информации](#updating-payment-information)
    - [Изменение тарифных планов](#changing-plans)
    - [Количество подписок](#subscription-quantity)
    - [Модификаторы подписки](#subscription-modifiers)
    - [Приостановка подписки](#pausing-subscriptions)
    - [Отмена подписки](#cancelling-subscriptions)
- [Пробные периоды](#subscription-trials)
    - [С указанием способа оплаты](#with-payment-method-up-front)
    - [Без указания способа оплаты](#without-payment-method-up-front)
- [Обработка веб-хуков Paddle](#handling-paddle-webhooks)
    - [Определение веб-хука событий](#defining-webhook-event-handlers)
    - [Проверка подписей веб-хука](#verifying-webhook-signatures)
- [Разовые списания](#single-charges)
    - [Разовое списание](#simple-charge)
    - [Списание за продукты](#charging-products)
    - [Возврат заказов](#refunding-orders)
- [Квитанции](#receipts)
    - [Прошлые и предстоящие платежи](#past-and-upcoming-payments)
- [Обработка несостоявшихся платежей](#handling-failed-payments)
- [Тестирование](#testing)

<a name="introduction"></a>
## Вступление

[Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle) предоставляет выразительный, плавный интерфейс для [Paddle](https://paddle.com) услуги выставления счетов по подписке. Он обрабатывает почти весь стандартный код выставления счетов за подписку, которого вы так боитесь. В дополнение к базовому управлению подпиской, кассир может обрабатывать: купоны, обмен подписки, "количество" подписки, льготные периоды отмены и многое другое.

При работе с Cashier мы рекомендуем вам также ознакомиться с [руководствами пользователя Paddle](https://developer.paddle.com/guides) и [документацией по API](https://developer.paddle.com/api-reference/intro).

<a name="upgrading-cashier"></a>
## Обновление Cashier

При обновлении до новой версии Cashier важно внимательно ознакомиться с [руководством по обновлению](https://github.com/laravel/cashier-paddle/blob/master/UPGRADE.md).

<a name="installation"></a>
## Установка

Сначала установите пакет Cashier для Paddle с помощью менеджера пакетов Composer:

    composer require laravel/cashier-paddle

> {note} Чтобы убедиться, что Cashier правильно обрабатывает все события Paddle, не забудьте [настроить обработку веб-хуков Cashier](#handling-paddle-webhooks).

<a name="paddle-sandbox"></a>
### Paddle Sandbox

Во время локальной и промежуточной разработки вам следует [зарегистрировать учетную запись Paddle Sandbox](https://developer.paddle.com/getting-started/sandbox). Эта учетная запись предоставит вам изолированную среду для тестирования и разработки ваших приложений без внесения фактических платежей. Вы можете использовать предоставляемые Paddle [номера тестовых карточек](https://developer.paddle.com/getting-started/sandbox#test-cards) для моделирования различных сценариев оплаты.

При использовании среды Paddle Sandbox вам следует установить переменной окружения `PADDLE_SANDBOX` значение `true` в файле `.env` вашего приложения:

PADDLE_SANDBOX=true

После того, как вы закончите разработку своего приложения, вы можете [подать заявку на создание учетной записи поставщика Paddle](https://paddle.com).

<a name="database-migrations"></a>
### Миграции

Сервис-провайдер Cashier регистрирует свой собственный каталог миграции базы данных, поэтому не забудьте выполнить миграции вашей базы данных после установки пакета. Миграция Cashier создаст новую таблицу `customers`. Кроме того, будет создана новая таблица `subscriptions` для хранения всех подписок вашего клиента. Наконец, будет создана новая таблица `receipts` для хранения всей информации о квитанциях вашего приложения:

    php artisan migrate

Если вам нужно перезаписать миграции, включенные в Cashier, вы можете опубликовать их с помощью команды Artisan `vendor:publish`:

    php artisan vendor:publish --tag="cashier-migrations"

Если вы хотите полностью запретить выполнение миграций Cashier, вы можете использовать `ignoreMigrations`, предоставленное Cashier. Как правило, этот метод должен вызываться в методе `register` вашего `AppServiceProvider`:

    use Laravel\Paddle\Cashier;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Cashier::ignoreMigrations();
    }

<a name="configuration"></a>
## Конфигурация

<a name="billable-model"></a>
### Оплачиваемая модель

Перед использованием Cashier вы должны добавить трейт `Billable` в определение вашей пользовательской модели. Эта функция предоставляет различные методы, позволяющие выполнять обычные задачи выставления счетов, такие как создание подписок, применение купонов и обновление информации о способе оплаты:

    use Laravel\Paddle\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

Если у вас есть оплачиваемые объекты, которые не являются пользователями, вы также можете добавить трейт к этим классам:

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Paddle\Billable;

    class Team extends Model
    {
        use Billable;
    }

<a name="api-keys"></a>
### API ключи

Далее вы должны настроить свои Paddle ключи в файле `.env` вашего приложения. Вы можете получить свои ключи Paddle API в панели управления Paddle:

    PADDLE_VENDOR_ID=your-paddle-vendor-id
    PADDLE_VENDOR_AUTH_CODE=your-paddle-vendor-auth-code
    PADDLE_PUBLIC_KEY="your-paddle-public-key"
    PADDLE_SANDBOX=true

Переменная окружения `PADDLE_SANDBOX` должна быть установлена в значение `true`, когда вы используете [среду Paddle Sandbox](#paddle-sandbox). Переменной `PADDLE_SANDBOX` следует присвоить значение `false`, если вы развертываете свое приложение в рабочей среде и используете среду реального поставщика Paddle.

<a name="paddle-js"></a>
### Paddle JS

Paddle использует свою собственную библиотеку JavaScript для запуска виджета оформления заказа. Вы можете загрузить библиотеку JavaScript, поместив директиву Blade `@paddleJS` прямо перед закрывающим тегом макета вашего приложения `</head>`:

    <head>
        ...

        @paddleJS
    </head>

<a name="currency-configuration"></a>
### Конфигурация валюты

Валютой по умолчанию Cashier являются доллары США (USD). Вы можете изменить валюту по умолчанию, определив переменную среды `CASHIER_CURRENCY` в файле `.env` вашего приложения:

    CASHIER_CURRENCY=EUR

В дополнение к настройке валюты Cashier, вы также можете указать язык, который будет использоваться при форматировании денежных значений для отображения в счетах-фактурах. Внутренне Cashier использует [класс PHP `NumberFormatter`](https://www.php.net/manual/ru/class.numberformatter.php) для установки языкового стандарта валюты:

    CASHIER_CURRENCY_LOCALE=nl_BE

> {note} Чтобы использовать локали, отличные от `en`, убедитесь, что на вашем сервере установлено и настроено расширение PHP `ext-intl`.

<a name="overriding-default-models"></a>
### Переопределение моделей по умолчанию

Вы можете свободно расширять модели, используемые внутри Cashier, определив свою собственную модель и расширив соответствующую модель Cashier:

    use Laravel\Paddle\Subscription as CashierSubscription;

    class Subscription extends CashierSubscription
    {
        // ...
    }

После определения вашей модели вы можете поручить Cashier использовать вашу пользовательскую модель с помощью класса `Laravel\Paddle\Cashier`. Как правило, вы должны сообщить Cashier о ваших пользовательских моделях в методе `boot` класса `App\Providers\AppServiceProvider` вашего приложения:

    use App\Models\Cashier\Receipt;
    use App\Models\Cashier\Subscription;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Cashier::useReceiptModel(Receipt::class);
        Cashier::useSubscriptionModel(Subscription::class);
    }

<a name="core-concepts"></a>
## Основные концепции

<a name="pay-links"></a>
### Платежные ссылки

В Paddle отсутствует обширный CRUD API для выполнения изменений состояния подписки. Таким образом, большинство взаимодействий с Paddle осуществляется через его [виджет оформления заказа](https://developer.paddle.com/guides/how-tos/checkout/paddle-checkout). Прежде чем мы сможем отобразить виджет оформления заказа, мы должны сгенерировать "ссылку для оплаты" с помощью Cashier. "Ссылка для оплаты" проинформирует виджет оформления заказа об операции выставления счета, которую мы хотим выполнить:

    use App\Models\User;
    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $payLink = $request->user()->newSubscription('default', $premium = 34567)
            ->returnTo(route('home'))
            ->create();

        return view('billing', ['payLink' => $payLink]);
    });

Cashier включает в себя `paddle-button` [компонент Blade](/docs/{{version}}/blade#components). Мы можем передать URL-адрес ссылки на оплату этому компоненту в качестве "параметра". При нажатии на эту кнопку отобразится виджет оформления заказа Paddle:

```html
<x-paddle-button :url="$payLink" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

По умолчанию при этом будет отображаться кнопка со стандартным дизайном Paddle. Вы можете удалить весь стиль Paddle, добавив атрибут `data-theme="none"` к компоненту:

```html
<x-paddle-button :url="$payLink" class="px-8 py-4" data-theme="none">
    Subscribe
</x-paddle-button>
```

Виджет оформления заказа Paddle является асинхронным. Как только пользователь создаст или обновит подписку в виджете, Paddle отправит вашему приложению веб-хуки, чтобы вы могли должным образом обновить состояние подписки в нашей собственной базе данных. Поэтому важно, чтобы вы правильно [настроили веб-хуки](#handling-paddle-webhooks), чтобы учесть изменения состояния от Paddle.

Для получения дополнительной информации о платежных ссылках вы можете ознакомиться с [документацией Paddle API по генерации платежных ссылок](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink).

> {note} После изменения состояния подписки задержка получения соответствующего веб-хука обычно минимальна, но вы должны учесть это в своем приложении, учитывая, что подписка вашего пользователя может быть недоступна сразу после завершения оформления заказа.

<a name="manually-rendering-pay-links"></a>
#### Отображение платежных ссылок вручную

Вы также можете вручную отобразить платежную ссылку без использования встроенных компонентов Blade Laravel. Для начала сгенерируйте URL-адрес платежной ссылки, как показано в предыдущих примерах:

    $payLink = $request->user()->newSubscription('default', $premium = 34567)
        ->returnTo(route('home'))
        ->create();

Затем просто прикрепите URL-адрес платежной ссылки к элементу `a` в вашем HTML:

    <a href="#!" class="ml-4 paddle_button" data-override="{{ $payLink }}">
        Paddle Checkout
    </a>

<a name="payments-requiring-additional-confirmation"></a>
#### Платежи, требующие дополнительного подтверждения

Иногда для подтверждения и обработки платежа требуется дополнительная верификация. Когда это произойдет, Paddle отобразит экран подтверждения оплаты. Экраны подтверждения оплаты, предоставляемые Paddle или Cashier, могут быть адаптированы к платежному потоку конкретного банка или эмитента карты и могут включать дополнительное подтверждение карты, временную небольшую плату, отдельную аутентификацию устройства или другие формы проверки.

<a name="inline-checkout"></a>
### Встроенное оформление заказа

Если вы не хотите использовать виджет оформления заказа Paddle в стиле "overlay", Paddle также предоставляет возможность отображать виджет встроенным. Хотя этот подход не позволяет вам настраивать какие-либо HTML-поля оформления заказа, он позволяет вам встроить виджет в ваше приложение.

Чтобы вам было легко начать работу со встроенным оформлением заказа, Cashier включает в себя функцию Blade-компонент `paddle-checkout`. Чтобы начать, вы должны [сгенерировать платежную ссылку](#pay-links) и передать платежную ссылку атрибуту `override` компонента:

```html
<x-paddle-checkout :override="$payLink" class="w-full" />
```

Чтобы настроить высоту встроенного компонента оформления заказа, вы можете передать атрибут `height` компоненту Blade:

    <x-paddle-checkout :override="$payLink" class="w-full" height="500" />

<a name="inline-checkout-without-pay-links"></a>
#### Встроенное офрмление заказа без ссылок для оплаты

В качестве альтернативы вы можете настроить виджет с помощью пользовательских опций вместо использования платежной ссылки:

    $options = [
        'product' => $productId,
        'title' => 'Product Title',
    ];

    <x-paddle-checkout :options="$options" class="w-full" />

Пожалуйста, ознакомьтесь с руководством Paddle по [оформлению заказа во встроенном режиме](https://developer.paddle.com/guides/how-tos/checkout/inline-checkout), а также их [ссылки на параметры](https://developer.paddle.com/reference/paddle-js/parameters) для получения более подробной информации о доступных опциях встроенного оформления заказа.

> {note} Если вы хотели бы также использовать опцию `passthrough` при указании пользовательских параметров, вам следует указать массив ключей / значений в качестве его значения. Cashier автоматически обработает преобразование массива в JSON-строку. Кроме того, опция сквозного ввода `customer_id` зарезервирована для использования внутренним Cashier.

<a name="manually-rendering-an-inline-checkout"></a>
#### Ручная отрисовка встроенного оформления заказа

Вы также можете вручную отобразить встроенную проверку без использования встроенных Blade-компонентов Laravel. Для начала сгенерируйте URL-адрес платежной ссылки [как показано в предыдущих примерах](#платные ссылки).

Далее вы можете использовать Paddle.js чтобы инициализировать оформление заказа. Чтобы упростить этот пример, мы продемонстрируем это с помощью [Alpine.js](https://github.com/alpinejs/alpine); однако вы можете свободно перевести этот пример в свой собственный стек интерфейса:

```html
<div class="paddle-checkout" x-data="{}" x-init="
    Paddle.Checkout.open({
        override: {{ $payLink }},
        method: 'inline',
        frameTarget: 'paddle-checkout',
        frameInitialHeight: 366,
        frameStyle: 'width: 100%; background-color: transparent; border: none;'
    });
">
</div>
```

<a name="user-identification"></a>
### Идентификация пользователя

В отличие от Stripe, пользователи Paddle уникальны для всего Paddle, а не для каждой учетной записи Paddle. Из-за этого API Paddle в настоящее время не предоставляют способ обновления данных пользователя, таких как его адрес электронной почты. При создании платных ссылок Paddle идентифицирует пользователей с помощью параметра `customer_email`. При создании подписки Paddle попытается сопоставить предоставленный пользователем адрес электронной почты с существующим пользователем Paddle.

В свете такого поведения есть несколько важных вещей, о которых следует помнить при использовании Cashier и Paddle. Во-первых, вы должны знать, что, хотя подписки в Cashier привязаны к одному и тому же пользователю приложения, **они могут быть привязаны к разным пользователям во внутренних системах Paddle**. Во-вторых, каждая подписка имеет свою собственную информацию о подключенном способе оплаты, а также может иметь разные адреса электронной почты во внутренних системах Paddle (в зависимости от того, какой адрес электронной почты был назначен пользователю при создании подписки).

Поэтому при отображении подписок вы всегда должны сообщать пользователю, какой адрес электронной почты или информация о способе оплаты подключены к подписке для каждой подписки. Извлечение этой информации может быть выполнено с помощью следующих методов, предоставляемых моделью `Laravel\Paddle\Subscription`:

    $subscription = $user->subscription('default');

    $subscription->paddleEmail();
    $subscription->paymentMethod();
    $subscription->cardBrand();
    $subscription->cardLastFour();
    $subscription->cardExpirationDate();

В настоящее время нет способа изменить адрес электронной почты пользователя с помощью Paddle API. Когда пользователь хочет обновить свой адрес электронной почты в Paddle, единственный способ для него сделать это - обратиться в службу поддержки клиентов Paddle. При общении с Paddle им необходимо указать значение подписки `paddleEmail`, чтобы помочь Paddle обновить нужного пользователя.

<a name="prices"></a>
## Цены

Paddle позволяет вам настраивать цены для каждой валюты, по сути, позволяя вам настраивать разные цены для разных стран. Приложение Cashier Paddle позволяет вам получить все цены на данный товар, используя метод `productPrices`. Этот метод принимает идентификаторы продуктов, цены на которые вы хотите получить:

    use Laravel\Paddle\Cashier;

    $prices = Cashier::productPrices([123, 456]);

Валюта будет определена на основе IP-адреса запроса; однако вы можете дополнительно указать конкретную страну для получения цен для нее:

    use Laravel\Paddle\Cashier;

    $prices = Cashier::productPrices([123, 456], ['customer_country' => 'BE']);

После получения цен вы можете отображать их так, как пожелаете:

```html
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - {{ $price->price()->gross() }}</li>
    @endforeach
</ul>
```

Вы также можете отобразить цену нетто (без учета налога) и сумму налога отдельно:

```html
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - {{ $price->price()->net() }} (+ {{ $price->price()->tax() }} tax)</li>
    @endforeach
</ul>
```

Если вы извлекли цены на планы подписки, вы можете отобразить их начальную и повторяющуюся цены отдельно:

```html
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - Initial: {{ $price->initialPrice()->gross() }} - Recurring: {{ $price->recurringPrice()->gross() }}</li>
    @endforeach
</ul>
```

Для получения дополнительной информации [ознакомьтесь документацией Paddle API по ценам](https://developer.paddle.com/api-reference/checkout-api/prices/getprices).

<a name="prices-customers"></a>
#### Клиенты

Если пользователь уже является клиентом, и вы хотели бы отобразить цены, применимые к этому клиенту, то можете сделать это, извлекая цены непосредственно из экземпляра объекта клиента:

    use App\Models\User;

    $prices = User::find(1)->productPrices([123, 456]);

Внутри Cashier будет использовать пользовательский [метод `paddleCountry`](#customer-defaults) для получения цен в их валюте. Так, например, пользователь, живущий в Соединенных Штатах, увидит цены в долларах США, в то время как пользователь в Бельгии увидит цены в евро. Если подходящая валюта не найдена, будет использоваться валюта продукта по умолчанию. Вы можете настроить все цены на продукт или тарифный план подписки на панели управления Paddle.

<a name="prices-coupons"></a>
#### Купоны

Вы также можете выбрать отображение цен после скидки по купону. При вызове метода `productPrices` купоны могут передаваться в виде строки, разделенной запятой:

    use Laravel\Paddle\Cashier;

    $prices = Cashier::productPrices([123, 456], [
        'coupons' => 'SUMMERSALE,20PERCENTOFF'
    ]);

Затем отобразите рассчитанные цены, используя метод `price`:

```html
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - {{ $price->price()->gross() }}</li>
    @endforeach
</ul>
```

Вы можете отобразить исходные указанные цены (без скидок по купонам), используя метод `listPrice`:

```html
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - {{ $price->listPrice()->gross() }}</li>
    @endforeach
</ul>
```

> {note} При использовании prices API, Paddle позволяет применять купоны только к продуктам для одноразовой покупки, а не к тарифным планам подписки.

<a name="customers"></a>
## Клиенты

<a name="customer-defaults"></a>
### Параметры по умолчанию

Cashier позволяет вам определять некоторые полезные значения по умолчанию для ваших клиентов при создании платежных ссылок. Установка этих значений по умолчанию позволяет вам предварительно ввести адрес электронной почты, страну и почтовый индекс клиента, чтобы он мог сразу перейти к платежной части виджета оформления заказа. Вы можете установить эти значения по умолчанию, переопределив следующие методы в вашей оплачиваемой модели:

    /**
     * Get the customer's email address to associate with Paddle.
     *
     * @return string|null
     */
    public function paddleEmail()
    {
        return $this->email;
    }

    /**
     * Get the customer's country to associate with Paddle.
     *
     * This needs to be a 2 letter code. See the link below for supported countries.
     *
     * @return string|null
     * @link https://developer.paddle.com/reference/platform-parameters/supported-countries
     */
    public function paddleCountry()
    {
        //
    }

    /**
     * Get the customer's postal code to associate with Paddle.
     *
     * See the link below for countries which require this.
     *
     * @return string|null
     * @link https://developer.paddle.com/reference/platform-parameters/supported-countries#countries-requiring-postcode
     */
    public function paddlePostcode()
    {
        //
    }

Эти значения по умолчанию будут использоваться для каждого действия в Cashier, которое генерирует [платежную ссылку](#платежные ссылки).

<a name="subscriptions"></a>
## Подписки

<a name="creating-subscriptions"></a>
### Создание подписок

Чтобы создать подписку, сначала извлеките экземпляр вашей оплачиваемой модели, которая обычно будет экземпляром `App\Models\User`. Как только вы получите экземпляр модели, вы можете использовать метод `newSubscription` для создания ссылки на оплату подписки модели:

    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $payLink = $user->newSubscription('default', $premium = 12345)
            ->returnTo(route('home'))
            ->create();

        return view('billing', ['payLink' => $payLink]);
    });

Первым аргументом, передаваемым методу `newSubscription`, должно быть внутреннее имя подписки. Если ваше приложение предлагает только одну подписку, вы можете назвать это `default` или `primary`. Это название подписки предназначено только для внутреннего использования приложением и не предназначено для показа пользователям. Кроме того, он не должен содержать пробелов и никогда не должен быть изменен после создания подписки. Вторым аргументом, заданным методу `newSubscription`, является конкретный тарифный план, на который подписывается пользователь. Это значение должно соответствовать идентификатору плана в Paddle. Метод `returnTo` принимает URL-адрес, на который ваш пользователь будет перенаправлен после успешного завершения оформления заказа.

Метод `create` создаст ссылку для оплаты, которую вы можете использовать для создания кнопки оплаты. Кнопка оплаты может быть сгенерирована с помощью `paddle-button` [Blade-компонента](/docs/{{version}}/blade#components), которая входит в комплект Cashier Paddle:

```html
<x-paddle-button :url="$payLink" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

После того, как пользователь завершит проверку, из Paddle будет отправлен веб-хук `subscription_created`. Cashier получит этот веб-хук и настроит подписку для вашего клиента. Чтобы убедиться, что все веб-хуки должным образом принимаются и обрабатываются вашим приложением, убедитесь, что у вас правильно [настроена обработка вэб-хуков](#handling-paddle-webhooks).

<a name="additional-details"></a>
#### Дополнительные сведения

Если вы хотите указать дополнительные сведения о клиенте или подписке, вы можете сделать это, передав их в виде массива пар ключ/значение методу `create`. Чтобы узнать больше о дополнительных полях, поддерживаемых Paddle, ознакомьтесь с документацией Paddles по [созданию платежных ссылок](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink):

    $payLink = $user->newSubscription('default', $monthly = 12345)
        ->returnTo(route('home'))
        ->create([
            'vat_number' => $vatNumber,
        ]);

<a name="subscriptions-coupons"></a>
#### Купоны

Если вы хотите применить купон при создании подписки, вы можете использовать метод `withCoupon`:

    $payLink = $user->newSubscription('default', $monthly = 12345)
        ->returnTo(route('home'))
        ->withCoupon('code')
        ->create();

<a name="metadata"></a>
#### Метаданные

Вы также можете передать массив метаданных, используя метод `withMetadata`:

    $payLink = $user->newSubscription('default', $monthly = 12345)
        ->returnTo(route('home'))
        ->withMetadata(['key' => 'value'])
        ->create();

> {note} При предоставлении метаданных, пожалуйста, избегайте использования `subscription_name` в качестве ключа метаданных. Этот ключ зарезервирован для внутреннего использования Cashier.

<a name="checking-subscription-status"></a>
### Проверка статуса подписки

Как только пользователь зарегистрируется в вашем приложении, вы можете проверить статус его подписки различными удобными способами. Во-первых, метод `subscribed` возвращает `true`, если у пользователя активная подписка, даже если в настоящее время подписка действует в течение пробного периода:

    if ($user->subscribed('default')) {
        //
    }

Метод `subscribed` также является отличным кандидатом для [посредника роута](/docs/{{version}}/middleware), позволяя вам фильтровать доступ к маршрутам и контроллерам на основе статуса подписки пользователя:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class EnsureUserIsSubscribed
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->user() && ! $request->user()->subscribed('default')) {
                // This user is not a paying customer...
                return redirect('billing');
            }

            return $next($request);
        }
    }

Если вы хотите определить, находится ли пользователь все еще в пределах своего пробного периода, вы можете использовать его метод `onTrial`. Этот метод может быть полезен для определения того, следует ли отображать предупреждение пользователю о том, что у него все еще действует пробный период:

    if ($user->subscription('default')->onTrial()) {
        //
    }

Метод `subscribedToPlan` может использоваться для определения того, подписан ли пользователь на данный тарифный план, на основе заданного идентификатора тарифного плана Paddle. В этом примере мы определим, является ли подписка пользователя `default` активной подпиской на ежемесячный план:

    if ($user->subscribedToPlan($monthly = 12345, 'default')) {
        //
    }

Передавая массив методу `subscribedToPlan`, вы можете определить, является ли подписка пользователя `default` активной подпиской на ежемесячный или годовой план:

    if ($user->subscribedToPlan([$monthly = 12345, $yearly = 54321], 'default')) {
        //
    }

Метод `recurring` может быть использован для определения того, подписан ли пользователь в данный момент и не проходит ли пробный период:

    if ($user->subscription('default')->recurring()) {
        //
    }

<a name="cancelled-subscription-status"></a>
#### Статус отмененной подписки

Чтобы определить, был ли пользователь когда-то активным подписчиком, но отменил свою подписку, вы можете использовать метод `cancelled`:

    if ($user->subscription('default')->cancelled()) {
        //
    }

Вы также можете определить, отменил ли пользователь свою подписку, но все еще находится в "льготном периоде" до полного истечения срока действия подписки. Например, если пользователь отменяет подписку 5 марта, срок действия которой первоначально планировался на 10 марта, у пользователя действует "льготный период" до 10 марта. Обратите внимание, что метод `subscribed` все еще возвращает `true` в течение этого времени:

    if ($user->subscription('default')->onGracePeriod()) {
        //
    }

Чтобы определить, отменил ли пользователь свою подписку и больше не находится в пределах "льготного периода", вы можете использовать метод `ended`:

    if ($user->subscription('default')->ended()) {
        //
    }

<a name="past-due-status"></a>
#### Просроченный платежный статус

Если оплата подписки не произведена, она будет помечена как `past_due`. Когда ваша подписка находится в этом состоянии, она не будет активна до тех пор, пока клиент не обновит свою платежную информацию. Вы можете определить, просрочена ли подписка, используя метод `pastDue` в экземпляре подписки:

    if ($user->subscription('default')->pastDue()) {
        //
    }

Если срок действия подписки просрочен, вы должны проинструктировать пользователя [обновить свою платежную информацию](#updating-payment-information). Вы можете настроить порядок обработки просроченных подписок в своих [настройках подписки Paddle](https://vendors.paddle.com/subscription-settings).

Если вы хотите, чтобы подписки по-прежнему считались активными, когда они являются `past_due`, вы можете использовать метод `keepPastDueSubscriptionsActive`, предоставляемый Cashier. Как правило, этот метод должен вызываться в методе `register` вашего `AppServiceProvider`:

    use Laravel\Paddle\Cashier;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Cashier::keepPastDueSubscriptionsActive();
    }

> {note} Когда подписка находится в состоянии `past_due`, она не может быть изменена до тех пор, пока не будет обновлена платежная информация. Следовательно, методы `swap` и `updateQuantity` выдадут исключение, когда подписка находится в состоянии `past_due`.

<a name="subscription-scopes"></a>
#### Диапазоны подписки

Большинство состояний подписки также доступны в виде диапазона запросов, так что вы можете легко запрашивать в своей базе данных подписки, находящиеся в заданном состоянии:

    // Get all active subscriptions...
    $subscriptions = Subscription::query()->active()->get();

    // Get all of the cancelled subscriptions for a user...
    $subscriptions = $user->subscriptions()->cancelled()->get();

Полный список доступных диапазонов доступен ниже:

    Subscription::query()->active();
    Subscription::query()->onTrial();
    Subscription::query()->notOnTrial();
    Subscription::query()->pastDue();
    Subscription::query()->recurring();
    Subscription::query()->ended();
    Subscription::query()->paused();
    Subscription::query()->notPaused();
    Subscription::query()->onPausedGracePeriod();
    Subscription::query()->notOnPausedGracePeriod();
    Subscription::query()->cancelled();
    Subscription::query()->notCancelled();
    Subscription::query()->onGracePeriod();
    Subscription::query()->notOnGracePeriod();

<a name="subscription-single-charges"></a>
### Единовременные платежи за подписку

Единовременная оплата подписки позволяет вам взимать с подписчиков единовременную плату сверх их подписки:

    $response = $user->subscription('default')->charge(12.99, 'Support Add-on');

В отличие от [единовременных платежей](#single-charges), этот метод немедленно взимает плату с сохраненного клиентом способа оплаты подписки. Сумма комиссии всегда должна определяться в валюте подписки.

<a name="updating-payment-information"></a>
### Обновление платежной информации

Paddle всегда сохраняет способ оплаты для каждой подписки. Если вы хотите обновить способ оплаты подписки по умолчанию, вам следует сначала сгенерировать "URL обновления подписки", используя метод `updateURL` в модели подписки:

    use App\Models\User;

    $user = User::find(1);

    $updateUrl = $user->subscription('default')->updateUrl();

Затем вы можете использовать сгенерированный URL-адрес в сочетании с предоставленным Cashier Blade-компонентом `paddle-button`, чтобы позволить пользователю запустить виджет Paddle и обновить свою платежную информацию:

```html
<x-paddle-button :url="$updateUrl" class="px-8 py-4">
    Update Card
</x-paddle-button>
```

Когда пользователь завершит обновление своей информации, Paddle отправит веб-запрос `subscription_updated`, и сведения о подписке будут обновлены в базе данных вашего приложения.

<a name="changing-plans"></a>
### Изменение тарифных планов

После того, как пользователь подписался на ваше приложение, он может иногда захотеть перейти на новый тарифный план подписки. Чтобы обновить план подписки для пользователя, вы должны передать идентификатор плана подписки в метод `swap` подписки:

    use App\Models\User;

    $user = User::find(1);

    $user->subscription('default')->swap($premium = 34567);

Если вы хотите поменять планы и немедленно выставить счет пользователю, не дожидаясь его следующего цикла выставления счетов, вы можете использовать метод `swapAndInvoice`:

    $user = User::find(1);

    $user->subscription('default')->swapAndInvoice($premium = 34567);

> {note} Планы нельзя менять местами, когда активна пробная версия. Для получения дополнительной информации об этом ограничении, пожалуйста, ознакомьтесь с [документацией Paddle](https://developer.paddle.com/api-reference/subscription-api/users/updateuser#usage-notes).

<a name="prorations"></a>
#### Пропорции

По умолчанию Paddle пропорционально распределяет расходы при переключении между тарифными планами. Метод `noProrate` может использоваться для обновления подписок без пропорциональных списаний:

    $user->subscription('default')->noProrate()->swap($premium = 34567);

<a name="subscription-quantity"></a>
### Количество подписок

Иногда на подписку влияет "количество". Например, приложение для управления проектами может взимать плату в размере 10 долларов США в месяц за каждый проект. Чтобы легко увеличивать или уменьшать количество ваших подписок, используйте методы `incrementQuantity` и `decrementQuantity`:

    $user = User::find(1);

    $user->subscription('default')->incrementQuantity();

    // Add five to the subscription's current quantity...
    $user->subscription('default')->incrementQuantity(5);

    $user->subscription('default')->decrementQuantity();

    // Subtract five from the subscription's current quantity...
    $user->subscription('default')->decrementQuantity(5);

В качестве альтернативы вы можете установить определенное количество, используя метод `updateQuantity`:

    $user->subscription('default')->updateQuantity(10);

Метод `noProrate` может быть использован для обновления количества подписок без пропорционального увеличения сборов:

    $user->subscription('default')->noProrate()->updateQuantity(10);

<a name="subscription-modifiers"></a>
### Модификаторы подписки

Модификаторы подписки позволяют вам реализовать [расчет по счетчику](https://developer.paddle.com/guides/how-tos/subscriptions/metered-billing#using-subscription-price-modifiers) или расширяйте подписки с помощью дополнений.

Например, вы можете захотеть предложить дополнительно "Премиум-поддержку" к вашей стандартной подписке. Вы можете создать этот модификатор следующим образом:

    $modifier = $user->subscription('default')->newModifier(12.99)->create();

В приведенном выше примере к подписке будет добавлено дополнение в размере 12,99 долларов США. По умолчанию эта плата будет взиматься повторно с каждого интервала, который вы настроили для подписки. Если вы хотите, вы можете добавить читаемое описание к модификатору, используя его метод `description`:

    $modifier = $user->subscription('default')->newModifier(12.99)
        ->description('Premium Support')
        ->create();

Чтобы проиллюстрировать, как реализовать тарификацию с использованием модификаторов, представьте, что ваше приложение взимает плату за SMS-сообщение, отправленное пользователем. Во-первых, вы должны создать план стоимостью 0 долларов на своей панели управления Paddle. Как только пользователь подпишется на этот тарифный план, вы можете добавить к подписке модификаторы, представляющие каждую отдельную плату:

    $modifier = $user->subscription('default')->newModifier(0.99)
        ->description('New text message')
        ->oneTime()
        ->create();

Как вы можете видеть, мы вызвали метод `oneTime` при создании этого модификатора. Этот метод гарантирует, что плата за модификатор взимается только один раз и не повторяется через каждый платежный интервал.

<a name="retrieving-modifiers"></a>
#### Извлечение модификаторов

Вы можете получить список всех модификаторов для подписки с помощью метода `modifiers`:

    $modifiers = $user->subscription('default')->modifiers();

    foreach ($modifiers as $modifier) {
        $modifier->amount(); // $0.99
        $modifier->description; // New text message.
    }

<a name="deleting-modifiers"></a>
#### Удаление модификаторов

Модификаторы могут быть удалены путем вызова метода `delete` в экземпляре `Laravel\Paddle\Modifier`:

    $modifier->delete();

<a name="pausing-subscriptions"></a>
### Приостановка подписки

Чтобы приостановить подписку, вызовите метод `pause` в подписке пользователя:

    $user->subscription('default')->pause();

Когда подписка приостановлена, Cashier автоматически установит столбец `paused_from` в вашей базе данных. Этот столбец используется, чтобы узнать, когда метод `paused` должен начать возвращать `true`. Например, если клиент приостанавливает подписку 1 марта, но повторение подписки не планировалось до 5 марта, метод `paused` будет продолжать возвращать `false` до 5 марта. Это делается потому, что пользователю обычно разрешается продолжать использовать приложение до окончания платежного цикла.

Вы можете определить, приостановил ли пользователь свою подписку, но все еще находится в "льготном периоде", используя метод `onPausedGracePeriod`:

    if ($user->subscription('default')->onPausedGracePeriod()) {
        //
    }

Чтобы возобновить приостановленную подписку, вы можете вызвать метод `unpause` в подписке пользователя:

    $user->subscription('default')->unpause();

> {note} Подписка не может быть изменена, пока она приостановлена. Если вы хотите перейти на другой тарифный план или обновить количество, вы должны сначала возобновить подписку.

<a name="cancelling-subscriptions"></a>
### Отмена подписки

Чтобы отменить подписку, вызовите метод `cancel` в подписке пользователя:

    $user->subscription('default')->cancel();

Когда подписка будет отменена, Cashier автоматически установит столбец `ends_at` в вашей базе данных. Этот столбец используется, чтобы узнать, когда метод `subscribed` должен начать возвращать `false`. Например, если клиент отменяет подписку 1 марта, но завершение подписки не планировалось до 5 марта, метод `subscribed` будет продолжать возвращать `true` до 5 марта. Это делается потому, что пользователю обычно разрешается продолжать использовать приложение до окончания платежного цикла.

Вы можете определить, отменил ли пользователь свою подписку, но все еще находится в "льготном периоде", используя метод `onGracePeriod`:

    if ($user->subscription('default')->onGracePeriod()) {
        //
    }

Если вы хотите немедленно отменить подписку, вы можете вызвать метод `cancelNow` для подписки пользователя:

    $user->subscription('default')->cancelNow();

> {note} Подписки Paddle не могут быть возобновлены после отмены. Если ваш клиент желает возобновить свою подписку, ему придется подписаться на новую подписку.

<a name="subscription-trials"></a>
## Пробные периоды

<a name="with-payment-method-up-front"></a>
### С указанием способа оплаты

> {note} Предварительно опробовав и собрав информацию о способе оплаты, Paddle предотвращает любые изменения подписки, такие как замена тарифных планов или обновление количества. Если вы хотите разрешить клиенту менять планы во время пробной версии, подписку необходимо отменить и создать заново.

Если вы хотите предложить своим клиентам пробные периоды, предварительно собирая информацию о способе оплаты, вам следует использовать метод `trialDays` при создании ссылок для оплаты подписки:

    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $payLink = $request->user()->newSubscription('default', $monthly = 12345)
                    ->returnTo(route('home'))
                    ->trialDays(10)
                    ->create();

        return view('billing', ['payLink' => $payLink]);
    });

Этот метод установит дату окончания пробного периода в записи подписки в базе данных вашего приложения, а также проинструктирует Paddle не начинать выставление счетов клиенту до истечения этой даты.

> {note} Если подписка клиента не будет отменена до даты окончания пробной версии, с него будет снята плата, как только истечет срок действия пробной версии, поэтому вам следует обязательно уведомить своих пользователей о дате окончания пробной версии.

Вы можете определить, находится ли пользователь в пределах своего пробного периода, используя либо метод `onTrial` экземпляра пользователя, либо метод `onTrial` экземпляра подписки. Два приведенных ниже примера эквивалентны:

    if ($user->onTrial('default')) {
        //
    }

    if ($user->subscription('default')->onTrial()) {
        //
    }

<a name="defining-trial-days-in-paddle-cashier"></a>
#### Определение пробных дней в Paddle / Cashier

Вы можете указать, сколько пробных дней предоставляется вашему плану, на панели управления Paddle или всегда передавать их явно с помощью Cashier. Если вы решите определить пробные дни вашего плана в Paddle, вы должны знать, что новые подписки, включая новые подписки для клиента, у которого была подписка в прошлом, всегда будут получать пробный период, если вы явно не вызовете метод `trialDays(0)`.

<a name="without-payment-method-up-front"></a>
### Без указания способа оплаты

Если вы хотите предлагать пробные периоды без предварительного сбора информации о способе оплаты пользователя, вы можете установить в столбце `trial_ends_at` в записи клиента, прикрепленной к вашему пользователю, желаемую дату окончания пробной версии. Обычно это делается во время регистрации пользователя:

    use App\Models\User;

    $user = User::create([
        // ...
    ]);

    $user->createAsCustomer([
        'trial_ends_at' => now()->addDays(10)
    ]);

Cashier называет этот тип пробной версии "общей пробной версией", поскольку она не привязана ни к одной существующей подписке. Метод `onTrial` в экземпляре `User` вернет `true`, если текущая дата не превышает значения `trial_ends_at`:

    if ($user->onTrial()) {
        // User is within their trial period...
    }

Как только вы будете готовы создать фактическую подписку для пользователя, вы можете использовать метод `newSubscription`, как обычно:

    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $payLink = $user->newSubscription('default', $monthly = 12345)
            ->returnTo(route('home'))
            ->create();

        return view('billing', ['payLink' => $payLink]);
    });

Чтобы получить дату окончания пробной версии пользователя, вы можете использовать метод `trialEndsAt`. Этот метод вернет экземпляр Carbon date, если пользователь находится на пробной версии, или `null`, если это не так. Вы также можете передать необязательный параметр имени подписки, если хотите получить дату окончания пробной версии для конкретной подписки, отличной от подписки по умолчанию:

    if ($user->onTrial()) {
        $trialEndsAt = $user->trialEndsAt('main');
    }

Вы можете использовать метод `onGenericTrial`, если хотите точно знать, что пользователь находится в пределах своего "общего" пробного периода и еще не создал фактическую подписку:

    if ($user->onGenericTrial()) {
        // User is within their "generic" trial period...
    }

> {note} Невозможно продлить или изменить пробный период подписки на Paddle после того, как она была создана.

<a name="handling-paddle-webhooks"></a>
## Обработка веб-хуков Paddle

Paddle может уведомлять ваше приложение о различных событиях с помощью вэб-хуков. По умолчанию маршрут, который указывает на контроллер вэб-хук Cashier, регистрируется сервис-провайдером Cashier. Этот контроллер будет обрабатывать все входящие запросы веб-хуков.

По умолчанию этот контроллер автоматически обрабатывает отмену подписок, в которых слишком много сброшенных платежей ([как определено в настройках вашей подписки Paddle])(https://vendors.paddle.com/subscription-settings)), обновления подписки и изменения способа оплаты; однако, как мы скоро узнаем, вы можете расширить этот контроллер для обработки любого события вэб-хука Paddle, как вам нравится.

Чтобы убедиться, что ваше приложение может обрабатывать вэб-хуки Paddle, обязательно [настройте URL-адрес вэб-хука на панели управления Paddle](https://vendors.paddle.com/alerts-webhooks). По умолчанию контроллер вэб-хуков Cashier отвечает на URL-адрес `/paddle/webhook`. Полный список всех веб-подключений, которые вы должны включить на панели управления Paddle, выглядит следующим образом:

- Подписка создана
- Подписка обновлена
- Подписка отменена
- Платеж прошел успешно
- Оплата подписки прошла успешно

> {note} Убедитесь, что вы защищаете входящие запросы с помощью включенного промежуточного программного обеспечения Cashier [проверка подписи вэб-хука](/docs/{{version}}/cashier-paddle#verifying-webhook-signatures).

<a name="webhooks-csrf-protection"></a>
#### Веб-хуки и защита от CSRF

Поскольку веб-хукам Paddle необходимо обходить Laravel [защиту от CSRF](/docs/{{version}}/csrf), обязательно укажите URI как исключение в вашем промежуточном программном обеспечении `App\Http\Middleware\VerifyCsrfToken` или укажите маршрут вне группы промежуточного программного обеспечения `web`:

    protected $except = [
        'paddle/*',
    ];

<a name="webhooks-local-development"></a>
#### Веб-хуки и локальная разработка

Чтобы Paddle мог отправлять веб-хуки вашему приложению во время локальной разработки, вам необходимо предоставить доступ к вашему приложению через службу общего доступа к сайту, такую как [Ngrok](https://ngrok.com/) или [Expose](https://expose.dev/docs/introduction). Если вы разрабатываете свое приложение локально, используя [Laravel Sail](/docs/{{version}}/sail), вы можете использовать команду Sail [общий доступ к сайту](/docs/{{version}}/sail#sharing-your-site).

<a name="defining-webhook-event-handlers"></a>
### Определение веб-хука событий

Cashier автоматически обрабатывает отмену подписки при сбое начислений и других распространенных ошибках веб-хуков Paddle. Однако, если у вас есть дополнительные события вэб-хуков, которые вы хотели бы обработать, вы можете сделать это, прослушав следующие события, отправляемые Cashier:

- `Laravel\Paddle\Events\WebhookReceived`
- `Laravel\Paddle\Events\WebhookHandled`

Оба события содержат полную полезную нагрузку вэб-хука Paddle. Например, если вы хотите обработать веб-запрос `invoice.payment_succeeded`, вы можете зарегистрировать [прослушиватель](/docs/{{version}}/events#defining-listeners), который будет обрабатывать событие:

    <?php

    namespace App\Listeners;

    use Laravel\Paddle\Events\WebhookReceived;

    class PaddleEventListener
    {
        /**
         * Handle received Paddle webhooks.
         *
         * @param  \Laravel\Paddle\Events\WebhookReceived  $event
         * @return void
         */
        public function handle(WebhookReceived $event)
        {
            if ($event->payload['alert_name'] === 'payment_succeeded') {
                // Handle the incoming event...
            }
        }
    }

Как только ваш прослушиватель определен, вы можете зарегистрировать его в `EventServiceProvider` вашего приложения:

    <?php

    namespace App\Providers;

    use App\Listeners\PaddleEventListener;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
    use Laravel\Paddle\Events\WebhookReceived;

    class EventServiceProvider extends ServiceProvider
    {
        protected $listen = [
            WebhookReceived::class => [
                PaddleEventListener::class,
            ],
        ];
    }

Cashier также выдает события, посвященные типу полученного вэб-хука. В дополнение к полной полезной нагрузке из Paddle, они также содержат соответствующие модели, которые использовались для обработки веб-хука, такие как оплачиваемая модель, подписка или квитанция:

<!-- <div class="content-list" markdown="1"> -->

- `Laravel\Paddle\Events\PaymentSucceeded`
- `Laravel\Paddle\Events\SubscriptionPaymentSucceeded`
- `Laravel\Paddle\Events\SubscriptionCreated`
- `Laravel\Paddle\Events\SubscriptionUpdated`
- `Laravel\Paddle\Events\SubscriptionCancelled`

<!-- </div> -->

Вы также можете переопределить встроенный в роут веб-хука по умолчанию, определив переменную среды `CASHIER_WEBHOOK` в файле `.env` вашего приложения. Это значение должно быть полным URL-адресом вашего роута веб-хука и должно совпадать с URL-адресом, заданным в вашей панели управления Paddle:

```bash
CASHIER_WEBHOOK=https://example.com/my-paddle-webhook-url
```

<a name="verifying-webhook-signatures"></a>
### Проверка подписей веб-хука

Чтобы обезопасить свои веб-хуки, вы можете использовать [подписи веб-хуков Paddle](https://developer.paddle.com/webhook-reference/verifying-webhooks). Для удобства Cashier автоматически включает посредника, которое проверяет правильность входящего запроса веб-хука Paddle.

Чтобы включить проверку вэб-хука, убедитесь, что переменная окружения `PADDLE_PUBLIC_KEY` определена в файле `.env` вашего приложения. Открытый ключ можно получить с панели управления вашей учетной записи Paddle.

<a name="single-charges"></a>
## Разовые списания

<a name="simple-charge"></a>
### Разовое списание

Если вы хотите произвести единовременное списание средств с клиента, вы можете использовать метод `charge` в экземпляре оплачиваемой модели, чтобы сгенерировать ссылку для оплаты платежа. Метод `charge` принимает величину платежа (float) в качестве своего первого аргумента и описание платежа в качестве второго аргумента:

    use Illuminate\Http\Request;

    Route::get('/store', function (Request $request) {
        return view('store', [
            'payLink' => $user->charge(12.99, 'Action Figure')
        ]);
    });

После создания платежной ссылки вы можете использовать предоставленный Cashier Blade-компонент `paddle-button`, позволяющий пользователю активировать виджет Paddle и завершить оплату:

```html
<x-paddle-button :url="$payLink" class="px-8 py-4">
    Buy
</x-paddle-button>
```

Метод `charge` принимает массив в качестве своего третьего аргумента, позволяя вам передавать любые параметры, которые вы пожелаете, для создания базовой платежной ссылки Paddle. Пожалуйста, ознакомьтесь с [документацией по Paddle](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink), чтобы узнать больше о вариантах, доступных вам при создании платежей:

    $payLink = $user->charge(12.99, 'Action Figure', [
        'custom_option' => $value,
    ]);

Платежи производятся в валюте, указанной в опции конфигурации `cashier.currency`. По умолчанию это значение равно долларам США. Вы можете переопределить валюту по умолчанию, определив переменную среды `CASHIER_CURRENCY` в файле `.env` вашего приложения:

```bash
CASHIER_CURRENCY=EUR
```

Вы также можете [переопределить цены в каждой валюте](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink#price-overrides) с использованием системы динамического сопоставления цен Paddle. Для этого передайте массив цен вместо фиксированной суммы:

    $payLink = $user->charge([
        'USD:19.99',
        'EUR:15.99',
    ], 'Action Figure');

<a name="charging-products"></a>
### Списание за продукты

Если вы хотите произвести единовременную оплату за определенный продукт, настроенный в Paddle, вы можете использовать метод `chargeProduct` в экземпляре оплачиваемой модели для создания платежной ссылки:

    use Illuminate\Http\Request;

    Route::get('/store', function (Request $request) {
        return view('store', [
            'payLink' => $request->user()->chargeProduct($productId = 123)
        ]);
    });

Затем вы можете передать платежную ссылку на компонент `paddle-button`, чтобы позволить пользователю инициализировать виджет Paddle:

```html
<x-paddle-button :url="$payLink" class="px-8 py-4">
    Buy
</x-paddle-button>
```

Метод `chargeProduct` принимает массив в качестве своего второго аргумента, позволяя вам передавать любые параметры, которые вы пожелаете, для создания базовой платежной ссылки Paddle. Пожалуйста, ознакомьтесь с [документацией по Paddle](https://developer.paddle.com/api-reference/product-api/pay-links/createpaylink) относительно опций, которые доступны вам при создании платежей:

    $payLink = $user->chargeProduct($productId, [
        'custom_option' => $value,
    ]);

<a name="refunding-orders"></a>
### Возврат заказов

Если вам нужно вернуть заказ Paddle, вы можете воспользоваться методом `refund`. Этот метод принимает идентификатор заказа весла в качестве своего первого аргумента. Вы можете получить квитанции для данной оплачиваемой модели, используя метод `receipts`:

    use App\Models\User;

    $user = User::find(1);

    $receipt = $user->receipts()->first();

    $refundRequestId = $user->refund($receipt->order_id);

При желании вы можете указать конкретную сумму для возврата, а также причину возврата:

    $receipt = $user->receipts()->first();

    $refundRequestId = $user->refund(
        $receipt->order_id, 5.00, 'Unused product time'
    );

> {tip} Вы можете использовать `$refundRequestId` в качестве ссылки для возврата средств при обращении в службу поддержки Paddle.

<a name="receipts"></a>
## Квитанции

Вы можете легко получить массив квитанций оплачиваемой модели с помощью свойства `receipts`:

    use App\Models\User;

    $user = User::find(1);

    $receipts = $user->receipts;

При перечислении квитанций для клиента вы можете использовать методы экземпляра квитанции для отображения соответствующей информации о квитанции. Например, вы можете захотеть отобразить каждую квитанцию в таблице, что позволит пользователю легко загрузить любую из квитанций:

```html
<table>
    @foreach ($receipts as $receipt)
        <tr>
            <td>{{ $receipt->paid_at->toFormattedDateString() }}</td>
            <td>{{ $receipt->amount() }}</td>
            <td><a href="{{ $receipt->receipt_url }}" target="_blank">Download</a></td>
        </tr>
    @endforeach
</table>
```

<a name="past-and-upcoming-payments"></a>
### Прошлые и предстоящие платежи

Вы можете использовать методы `lastPayment` и `nextPayment` для получения и отображения прошлых или предстоящих платежей клиента за повторяющиеся подписки:

    use App\Models\User;

    $user = User::find(1);

    $subscription = $user->subscription('default');

    $lastPayment = $subscription->lastPayment();
    $nextPayment = $subscription->nextPayment();

Оба этих метода вернут экземпляр `Laravel\Paddle\Payment`; однако `nextPayment` вернет `null`, когда цикл выставления счетов закончится (например, когда подписка была отменена):

    Next payment: {{ $nextPayment->amount() }} due on {{ $nextPayment->date()->format('d/m/Y') }}

<a name="handling-failed-payments"></a>
## Обработка несостоявшихся платежей

Оплата подписки не производится по разным причинам, таким как просроченный срок действия карты или недостаточное количество средств на карте. Когда это происходит, мы рекомендуем вам позволить Paddle обрабатывать сбои в оплате за вас. В частности, вы можете [настроить электронные письма с автоматическим выставлением счетов Paddle](https://vendors.paddle.com/subscription-settings) на вашей панели управления Paddle.

В качестве альтернативы вы можете выполнить более точную настройку, перехватите веб-хук [`subscription_payment_failed`](https://developer.paddle.com/webhook-reference/subscription-alerts/subscription-payment-failed) и включите опцию "Не удалось оплатить подписку" в настройках Webhook вашей панели управления Paddle:

    <?php

    namespace App\Http\Controllers;

    use Laravel\Paddle\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * Handle subscription payment failed.
         *
         * @param  array  $payload
         * @return void
         */
        public function handleSubscriptionPaymentFailed($payload)
        {
            // Handle the failed subscription payment...
        }
    }

<a name="testing"></a>
## Тестирование

Во время тестирования вам следует вручную протестировать процесс выставления счетов, чтобы убедиться, что ваша интеграция работает должным образом.

Для автоматизированных тестов, в том числе выполняемых в среде CI, вы можете использовать [HTTP-клиент Laravel](/docs/{{version}}/http-client#testing) для подделки HTTP-вызовов, выполняемых в Paddle. Хотя это не проверяет фактические ответы от Paddle, это предоставляет способ протестировать ваше приложение без фактического вызова API Paddle.
