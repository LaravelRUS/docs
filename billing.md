git 6ac13f37adbed3ce6a6532fd790f70bd731b8571

---

# Laravel Cashier (Stripe)

- [Введение](#introduction)
- [Обновление Cashier](#upgrading-cashier)
- [Установка](#installation)
    - [Миграции](#database-migrations)
- [Конфигурация](#configuration)
    - [Оплачиваемая модель](#billable-model)
    - [API ключи](#api-keys)
    - [Конфигурация валюты](#currency-configuration)
    - [Конфигурация налогов](#tax-configuration)
    - [Логирование](#logging)
    - [Использование пользовательских моделей](#using-custom-models)
- [Клиенты](#customers)
    - [Получение клиентов](#retrieving-customers)
    - [Создание клиентов](#creating-customers)
    - [Обновление клиентов](#updating-customers)
    - [Балансы](#balances)
    - [Идентификаторы налогоплательщиков](#tax-ids)
    - [Синхронизация клиентских данных с помощью Stripe](#syncing-customer-data-with-stripe)
    - [Биллинг портал](#billing-portal)
- [Способы оплаты](#payment-methods)
    - [Добавление способов оплаты](#storing-payment-methods)
    - [Получение способов оплаты](#retrieving-payment-methods)
    - [Определение, если у пользователя есть способ оплаты](#check-for-a-payment-method)
    - [Обновление способа оплаты по умолчанию](#updating-the-default-payment-method)
    - [Добавление способа оплаты](#adding-payment-methods)
    - [Удаление способа оплаты](#deleting-payment-methods)
- [Подписки](#subscriptions)
    - [Создание подписок](#creating-subscriptions)
    - [Проверка статуса подписки](#checking-subscription-status)
    - [Изменение цен](#changing-prices)
    - [Количество подписки](#subscription-quantity)
    - [Многоценовые подписки](#multiprice-subscriptions)
    - [Дозированный расчет](#metered-billing)
    - [Налоги подписки](#subscription-taxes)
    - [Дата привязки подписки](#subscription-anchor-date)
    - [Отмена подписки](#cancelling-subscriptions)
    - [Возобновление подписок](#resuming-subscriptions)
- [Пробные периоды](#subscription-trials)
    - [С указанием способа оплаты](#with-payment-method-up-front)
    - [Без указания способа оплаты](#without-payment-method-up-front)
    - [Продление пробного периода](#extending-trials)
- [Обработка Stripe веб-хуков](#handling-stripe-webhooks)
    - [Определение веб-хука событий](#defining-webhook-event-handlers)
    - [Проверка подписей веб-хука](#verifying-webhook-signatures)
- [Разовые списания](#single-charges)
    - [Разовое списание](#simple-charge)
    - [Списание со счетом](#charge-with-invoice)
    - [Возврат списаний](#refunding-charges)
- [Оформление](#checkout)
    - [Оформление заказа продукта](#product-checkouts)
    - [Оформление одиночного списания](#single-charge-checkouts)
    - [Оформление заказа подписки](#subscription-checkouts)
    - [Сбор идентификаторов налогоплательщиков](#collecting-tax-ids)
- [Счета](#invoices)
    - [Получение счетов](#retrieving-invoices)
    - [Предстоящие счета-фактуры](#upcoming-invoices)
    - [Предварительный просмотр счетов-фактур по подписке](#previewing-subscription-invoices)
    - [Генерация счетов PDF](#generating-invoice-pdfs)
- [Обработка неудачных платежей](#handling-failed-payments)
- [Аутентификация клиентов(SCA)](#strong-customer-authentication)
    - [Платежи, требующие дополнительного подтверждения](#payments-requiring-additional-confirmation)
    - [Уведомления об оплате вне сессии](#off-session-payment-notifications)
- [Stripe SDK](#stripe-sdk)
- [Тестирование](#testing)

<a name="introduction"></a>
## Введение

[Laravel Cashier Stripe](https://github.com/laravel/cashier-stripe) обеспечивает выразительный, плавный интерфейс для [Stripe's](https://stripe.com) услуги выставления счетов по подписке. Он обрабатывает почти весь стандартный код выставления счетов за подписку, который вы боитесь писать. В дополнение к базовому управлению подпиской, Cashier может обрабатывать купоны, менять подписку, "количество" подписки, льготные периоды отмены и даже создавать PDF-файлы счетов-фактур.

<a name="upgrading-cashier"></a>
## Обновление Cashier

При обновлении Cashier до новой версии важно внимательно ознакомиться с [руководством по обновлению](https://github.com/laravel/cashier-stripe/blob/master/UPGRADE.md).

> {note} Чтобы предотвратить внезапные изменения, Cashier использует фиксированную версию Stripe API. Cashier 13 использует Stripe API версии `2020-08-27`. Версия Stripe API будет обновляться в последующих выпусках, чтобы использовать новые функции и улучшения Stripe.

<a name="installation"></a>
## Установка

Сначала установите пакет Cashier для Stripe с помощью менеджера пакетов Composer:

    composer require laravel/cashier

> {note} Чтобы убедиться, что Cashier должным образом обрабатывает все события Stripe, не забудьте [настроить обработку веб-хуков Cashier] (#handling-stripe-webhooks).

<a name="database-migrations"></a>
### Миграции

Сервис-провайдер Cashier регистрирует свой собственный каталог миграции базы данных, поэтому не забудьте выполнить миграции вашей базы данных после установки пакета. Миграции Cashier добавят несколько столбцов в вашу таблицу `users`, а также создаст новую таблицу `subscriptions` для хранения всех подписок вашего клиента:

    php artisan migrate

Если вам нужно перезаписать миграции, которые поставляются с Cashier, вы можете опубликовать их с помощью команды Artisan `vendor:publish`:

    php artisan vendor:publish --tag="cashier-migrations"

Если вы хотите полностью запретить выполнение миграций Cashier, вы можете использовать метод `ignoreMigrations`, предоставляемый Cashier. Как правило, этот метод должен вызываться в методе `register` вашего `AppServiceProvider':

    use Laravel\Cashier\Cashier;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Cashier::ignoreMigrations();
    }

> {note} Stripe рекомендует, чтобы любой столбец, используемый для хранения идентификаторов Stripe, был чувствителен к регистру. Следовательно, вы должны убедиться, что сопоставление для столбца `stripe_id` установлено в `utf8_bin` при использовании MySQL. Более подробную информацию об этом можно найти в разделе [документация Stripe](https://stripe.com/docs/upgrades#what-changes-does-stripe-consider-to-be-backwards-compatible).

<a name="configuration"></a>
## Конфигурация

<a name="billable-model"></a>
### Оплачиваемая модель

Перед использованием Cashier добавьте трейт `Billable` в определение вашей оплачиваемой модели. Как правило, это будет модель `App\Models\User`. Этот трейт предоставляет различные методы, позволяющие выполнять обычные задачи выставления счетов, такие как создание подписок, применение купонов и обновление информации о способе оплаты:

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

Cashier предполагает, что вашей оплачиваемой моделью будет класс `App\Models\User`, который поставляется с Laravel. Если вы хотите изменить это, вы можете указать другую модель с помощью метода `useCustomerModel`. Этот метод обычно следует вызывать в методе `boot` вашего класса `AppServiceProvider`:

    use App\Models\Cashier\User;
    use Laravel\Cashier\Cashier;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Cashier::useCustomerModel(User::class);
    }

> {note} Если вы используете модель, отличную от поставляемой Laravel модели `App\Models\User`, вам нужно будет опубликовать и изменить предоставленные [миграции Cashier] (#installation), чтобы она соответствовала имени таблицы вашей альтернативной модели.

<a name="api-keys"></a>
### API ключи

Далее вам следует настроить ключи Stripe API в файле `.env` вашего приложения. Вы можете получить свои ключи Stripe API с панели управления Stripe:

    STRIPE_KEY=your-stripe-key
    STRIPE_SECRET=your-stripe-secret

<a name="currency-configuration"></a>
### Конфигурация валюты

Валютой Cashier по умолчанию являются доллары США (USD). Вы можете изменить валюту по умолчанию, установив переменную среды `CASHIER_CURRENCY` в файле `.env` вашего приложения:

    CASHIER_CURRENCY=eur

В дополнение к настройке валюты Cashier, вы также можете указать язык, который будет использоваться при форматировании денежных значений для отображения в счетах-фактурах. Внутренне Cashier использует [класс PHP `NumberFormatter`](https://www.php.net/manual/en/class.numberformatter.php) для установки языкового стандарта валюты:

    CASHIER_CURRENCY_LOCALE=nl_BE

> {note} Чтобы использовать локали, отличные от `en`, убедитесь, что на вашем сервере установлено и настроено расширение PHP `ext-intl`.

<a name="tax-configuration"></a>
### Конфигурация налогов

Благодаря [Stripe Tax](https://stripe.com/tax), можно автоматически рассчитать налоги для всех счетов-фактур, сгенерированных Stripe. Вы можете включить автоматический расчет налогов, вызвав метод `calculateTaxes` в методе `boot` класса `App\Providers\AppServiceProvider` вашего приложения:

    use Laravel\Cashier\Cashier;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Cashier::calculateTaxes();
    }

Как только расчет налога будет включен, все новые подписки и любые сгенерированные разовые счета-фактуры будут автоматически рассчитываться по налогу.

Чтобы эта функция работала должным образом, платежные данные вашего клиента, такие как имя, адрес и идентификационный номер налогоплательщика, должны быть синхронизированы с Stripe. Для этого вы можете использовать методы [синхронизации данных клиента] (#syncing-customer-data-with-stripe) и [Идентификатор налогоплательщика](#tax-ids), предлагаемые Cashier.

> {note} К сожалению, на данный момент налог не рассчитывается для [единовременных платежей] (#single-charges) или [выписок с разовой оплатой] (#single-charge-checkouts). Кроме того, Stripe Tax в настоящее время доступен "только по приглашению" в период бета-тестирования. Вы можете запросить доступ к Stripe Tax через [веб-сайт Stripe Tax](https://stripe.com/tax#request-access).

<a name="logging"></a>
### Логирование

Cashier позволяет вам указать канал регистрации, который будет использоваться при регистрации фатальных ошибок Stripe. Вы можете указать канал ведения журнала, определив переменную среды `CASHIER_LOGGER` в файле `.env` вашего приложения:

    CASHIER_LOGGER=stack

Исключения, генерируемые вызовами API для Stripe, будут регистрироваться через канал журнала вашего приложения по умолчанию.

<a name="using-custom-models"></a>
### Использование пользовательских моделей

Вы можете свободно расширять модели, используемые внутри Cashier, определив свою собственную модель и расширив соответствующую модель Cashier:

    use Laravel\Cashier\Subscription as CashierSubscription;

    class Subscription extends CashierSubscription
    {
        // ...
    }

После определения вашей модели вы можете указать Cashier использовать вашу пользовательскую модель с помощью класса `Laravel\Cashier\Cashier`. Как правило, вы должны сообщить кассиру о ваших пользовательских моделях в методе `boot` класса `App\Providers\AppServiceProvider' вашего приложения:

    use App\Models\Cashier\Subscription;
    use App\Models\Cashier\SubscriptionItem;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Cashier::useSubscriptionModel(Subscription::class);
        Cashier::useSubscriptionItemModel(SubscriptionItem::class);
    }

<a name="customers"></a>
## Клиенты

<a name="retrieving-customers"></a>
### Получение клиентов

Вы можете получить клиента по его идентификатору Stripe ID, используя метод `Cashier::findBillable`. Этот метод вернет экземпляр оплачиваемой модели:

    use Laravel\Cashier\Cashier;

    $user = Cashier::findBillable($stripeId);

<a name="creating-customers"></a>
### Создание клиентов

Иногда вы можете захотеть создать клиента Stripe, не начиная подписку. Вы можете выполнить это с помощью метода `createAsStripeCustomer`:

    $stripeCustomer = $user->createAsStripeCustomer();

Как только клиент будет создан в Stripe, вы можете начать подписку позже. Вы можете предоставить необязательный массив `$options` для передачи любых дополнительных [параметров создания клиента, поддерживаемых Stripe API](https://stripe.com/docs/api/customers/create):

    $stripeCustomer = $user->createAsStripeCustomer($options);

Вы можете использовать их метод `asStripeCustomer`, если хотите вернуть объект клиента Stripe для оплачиваемой модели:

    $stripeCustomer = $user->asStripeCustomer();

Метод `createOrGetStripeCustomer` может быть использован, если вы хотите получить объект клиента Stripe для данной оплачиваемой модели, но не уверены, является ли оплачиваемая модель уже клиентом в Stripe. Этот метод создаст нового клиента в Stripe, если таковой еще не существует:

    $stripeCustomer = $user->createOrGetStripeCustomer();

<a name="updating-customers"></a>
### Обновление клиентов

Иногда вы можете захотеть обновить дополнительную информацию непосредственно на клиенте Stripe. Вы можете выполнить это с помощью метода `updateStripeCustomer`. Этот метод принимает массив [параметров обновления клиента, поддерживаемых Stripe API](https://stripe.com/docs/api/customers/update):

    $stripeCustomer = $user->updateStripeCustomer($options);

<a name="balances"></a>
### Балансы

Stripe позволяет вам зачислять или дебетовать "баланс" клиента. Позже этот остаток будет зачислен или списан с новых счетов-фактур. Чтобы проверить общий баланс клиента, вы можете использовать метод `balance`, доступный в вашей оплачиваемой модели. Метод `balance` вернет форматированное строковое представление баланса в валюте клиента:

    $balance = $user->balance();

Чтобы пополнить баланс клиента, вы можете указать отрицательное значение для метода `applyBalance`. При желании вы также можете предоставить описание:

    $user->applyBalance(-500, 'Premium customer top-up.');

Предоставление положительного значения методу "applyBalance" приведет к списанию средств с баланса клиента:

    $user->applyBalance(300, 'Bad usage penalty.');

Метод `applyBalance` создаст для клиента новые проводки по балансу клиента. Вы можете получить эти записи транзакций, используя метод `balanceTransactions`, который может быть полезен для предоставления клиенту журнала зачислений и дебетований для просмотра:

    // Retrieve all transactions...
    $transactions = $user->balanceTransactions();

    foreach ($transactions as $transaction) {
        // Transaction amount...
        $amount = $transaction->amount(); // $2.31

        // Retrieve the related invoice when available...
        $invoice = $transaction->invoice();
    }

<a name="tax-ids"></a>
### Идентификаторы налогоплательщиков

Cashier предлагает простой способ управления идетификаторами налогоплательщиков. Например, метод `taxIds` может быть использован для извлечения всех [идентификаторов налогоплательщиков](https://stripe.com/docs/api/customer_tax_ids/object), которые назначаются клиенту в качестве коллекции:

    $taxIds = $user->taxIds();

Вы также можете получить конкретный налоговый идентификатор клиента по его идентификатору:

    $taxId = $user->findTaxId('txi_belgium');

Вы можете создать новый налоговый идентификатор, указав действительный [тип](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-type) и значение для метода `createTaxId`:

    $taxId = $user->createTaxId('eu_vat', 'BE0123456789');

Метод `createTaxId` немедленно добавит идентификационный номер плательщика НДС в учетную запись клиента. [Проверка идентификаторов плательщика НДС также осуществляется Stripe](https://stripe.com/docs/invoicing/customer/tax-ids#validation); однако это асинхронный процесс. Вы можете получать уведомления об обновлениях проверки, подписавшись на веб-хук событие `customer.tax_id.updated` и проверив [параметр `verification` идентификаторов НДС](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-verification). Для получения дополнительной информации об обработке веб-хуков, пожалуйста, обратитесь к [документации по определению обработчиков веб-хуков] (#handling-stripe-webhooks).

Вы можете удалить налоговый идентификатор, используя метод `deleteTaxId`:

    $user->deleteTaxId('txi_belgium');

<a name="syncing-customer-data-with-stripe"></a>
### Синхронизация клиентских данных с помощью Stripe

Как правило, когда пользователи вашего приложения обновляют свое имя, адрес электронной почты или другую информацию, которая также хранится в Stripe, вы должны сообщить Stripe об обновлениях. Таким образом, ваша копия информации будет синхронизирована с копией вашего приложения.

Чтобы автоматизировать это, вы можете определить прослушиватель событий в вашей оплачиваемой модели, который реагирует на событие модели `updated`. Затем, в вашем прослушивателе событий, вы можете вызвать метод `syncStripeCustomerDetails` для модели:

    use function Illuminate\Events\queueable;

    /**
     * The "booted" method of the model.
     *
     * @return void
     */
    protected static function booted()
    {
        static::updated(queueable(function ($customer) {
            if ($customer->hasStripeId()) {
                $customer->syncStripeCustomerDetails();
            }
        }));
    }

Теперь каждый раз, когда обновляется ваша клиентская модель, ее информация будет синхронизироваться со Stripe. Для удобства Cashier автоматически синхронизирует информацию о вашем клиенте со Stripe при первоначальном создании клиента.

Вы можете настроить столбцы, используемые для синхронизации информации о клиентах со Stripe, переопределив различные методы, предоставляемые Cashier. Например, вы можете переопределить метод `stripeName`, чтобы настроить атрибут, который следует рассматривать как "имя" клиента, когда Cashier синхронизирует информацию о клиенте со Stripe:

    /**
     * Get the customer name that should be synced to Stripe.
     *
     * @return string|null
     */
    public function stripeName()
    {
        return $this->company_name;
    }

Аналогичным образом, вы можете переопределить методы `stripeEmail`, `stripePhone` и `stripeAddress`. Эти методы будут синхронизировать информацию с соответствующими параметрами клиента при [обновлении объекта клиента Stripe](https://stripe.com/docs/api/customers/update). Если вы хотите получить полный контроль над процессом синхронизации информации о клиенте, вы можете переопределить метод `syncStripeCustomerDetails`.

<a name="billing-portal"></a>
### Биллинг портал

Stripe предлагает [простой способ настройки платежного портала](https://stripe.com/docs/billing/subscriptions/customer-portal), чтобы ваш клиент мог управлять своей подпиской, способами оплаты и просматривать историю выставления счетов. Вы можете перенаправить своих пользователей на портал выставления счетов, вызвав метод `redirectToBillingPortal` в модели выставления счетов с контроллера или маршрута:

    use Illuminate\Http\Request;

    Route::get('/billing-portal', function (Request $request) {
        return $request->user()->redirectToBillingPortal();
    });

По умолчанию, когда пользователь завершит управление своей подпиской, он сможет вернуться к маршруту `home` вашего приложения по ссылке на биллинговом портале Stripe. Вы можете предоставить пользовательский URL, на который пользователь должен вернуться, передав URL в качестве аргумента методу `redirectToBillingPortal`:

    use Illuminate\Http\Request;

    Route::get('/billing-portal', function (Request $request) {
        return $request->user()->redirectToBillingPortal(route('billing'));
    });

Если вы хотите сгенерировать URL-адрес портала выставления счетов без создания ответа на перенаправление HTTP, вы можете вызвать метод `billingPortalUrl`:

    $url = $request->user()->billingPortalUrl(route('billing'));

<a name="payment-methods"></a>
## Способы оплаты

<a name="storing-payment-methods"></a>
### Добавление способов оплаты

Чтобы создавать подписки или осуществлять "одноразовые" платежи с помощью Stripe, вам необходимо сохранить способ оплаты и получить его идентификатор из Stripe. Подход, используемый для достижения этой цели, отличается в зависимости от того, планируете ли вы использовать способ оплаты подписки или разовых платежей, поэтому ниже мы рассмотрим оба способа.

<a name="payment-methods-for-subscriptions"></a>
#### Способы оплаты для подписок 

При сохранении информации о кредитной карте клиента для будущего использования по подписке необходимо использовать API Stripe "Setup Intents" для безопасного сбора информации о способе оплаты клиента. "Setup Intent" указывает Stripe на намерение взимать плату с способа оплаты клиента. Трейт `Billable` Cashier включает метод `createSetupIntent`, позволяющий легко создать новый Setup Intent. Вы должны вызвать этот метод из маршрута или контроллера, который отобразит форму, в которой будут собраны данные о способе оплаты вашего клиента:

    return view('update-payment-method', [
        'intent' => $user->createSetupIntent()
    ]);

После того, как вы создали Setup Intent и передали его в представление, вы должны прикрепить его секрет к элементу, который будет собирать информацию о способе оплаты. Например, рассмотрим эту форму "обновить способ оплаты".:

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button" data-secret="{{ $intent->client_secret }}">
    Update Payment Method
</button>
```

Далее, может быть использована библиотека Stripe.js для прикрепления [элемента Stripe](https://stripe.com/docs/stripe-js) в форму и надежно собирать платежные реквизиты клиента:

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

Затем карта может быть верифицирована, и безопасный "идентификатор способа оплаты" может быть получен из Stripe с помощью [метода Stripe `confirmCardSetup`](https://stripe.com/docs/js/setup_intents/confirm_card_setup):

```js
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');
const clientSecret = cardButton.dataset.secret;

cardButton.addEventListener('click', async (e) => {
    const { setupIntent, error } = await stripe.confirmCardSetup(
        clientSecret, {
            payment_method: {
                card: cardElement,
                billing_details: { name: cardHolderName.value }
            }
        }
    );

    if (error) {
        // Display "error.message" to the user...
    } else {
        // The card has been verified successfully...
    }
});
```

После того, как карта была верифицирована Stripe, вы можете передать полученный идентификатор `setupIntent.payment_method` в ваше приложение Laravel, где он может быть прикреплен к клиенту. Способ оплаты может быть либо [добавлен в качестве нового способа оплаты] (#adding-payment-methods), либо [использован для обновления способа оплаты по умолчанию] (#updating-the-default-payment-method). Вы также можете немедленно использовать идентификатор способа оплаты для [создания новой подписки] (#creating-subscriptions).

> {tip} Если вы хотите получить дополнительную информацию о Setup Intents и сборе платежных реквизитов клиентов, пожалуйста, [ознакомьтесь с этим обзором, предоставленным Stripe](https://stripe.com/docs/payments/save-and-reuse#php).

<a name="payment-methods-for-single-charges"></a>
#### Способы оплаты для единовременных платежей

Конечно, при однократном списании средств с платежного метода клиента нам нужно будет использовать идентификатор платежного метода только один раз. Из-за ограничений Stripe вы не можете использовать сохраненный способ оплаты клиента по умолчанию для разовых платежей. Вы должны разрешить клиенту ввести данные о своем способе оплаты, используя библиотеку Stripe.js. Например, рассмотрим следующую форму:

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button">
    Process Payment
</button>
```

После определения такой формы, библиотека Stripe.js может быть использована для прикрепления [элемента Stripe](https://stripe.com/docs/stripe-js) в форму и надежно собирает платежные реквизиты клиента:

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

Затем карта может быть верифицирована, и безопасный "идентификатор способа оплаты" может быть получен из Stripe с помощью [метода Stripe `createPaymentMethod`].(https://stripe.com/docs/stripe-js/reference#stripe-create-payment-method):

```js
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');

cardButton.addEventListener('click', async (e) => {
    const { paymentMethod, error } = await stripe.createPaymentMethod(
        'card', cardElement, {
            billing_details: { name: cardHolderName.value }
        }
    );

    if (error) {
        // Display "error.message" to the user...
    } else {
        // The card has been verified successfully...
    }
});
```

Если верификация карты прошла успешно, вы можете передать `paymentMethod.id` вашему приложению Laravel и обработать [одноразовую оплату](#simple-charge).

<a name="retrieving-payment-methods"></a>
### Получение способов оплаты

Метод `PaymentMethod` в экземпляре оплачиваемой модели возвращает коллекцию экземпляров `Laravel\Cashier\PaymentMethod`:

    $paymentMethods = $user->paymentMethods();

По умолчанию этот метод возвращает способы оплаты типа `card`. Чтобы получить способы оплаты другого типа, вы можете передать `type` в качестве аргумента методу:

    $paymentMethods = $user->paymentMethods('sepa_debit');

Чтобы получить способ оплаты клиента по умолчанию, может быть использован метод `defaultPaymentMethod`.:

    $paymentMethod = $user->defaultPaymentMethod();

Вы можете получить конкретный способ оплаты, который привязан к оплачиваемой модели, используя метод `findPaymentMethod`:

    $paymentMethod = $user->findPaymentMethod($paymentMethodId);

<a name="check-for-a-payment-method"></a>
### Определение, если у пользователя есть способ оплаты

Чтобы определить, привязан ли к учетной записи оплачиваемой модели способ оплаты по умолчанию, вызовите метод `hasDefaultPaymentMethod`:

    if ($user->hasDefaultPaymentMethod()) {
        //
    }

Вы можете использовать метод `hasPaymentMethod`, чтобы определить, привязан ли к учетной записи оплачиваемой модели хотя бы один способ оплаты:

    if ($user->hasPaymentMethod()) {
        //
    }

Этот метод определит, есть ли в оплачиваемой модели способы оплаты типа `card`. Чтобы определить, существует ли для модели способ оплаты другого типа, вы можете передать `type` в качестве аргумента методу:

    if ($user->hasPaymentMethod('sepa_debit')) {
        //
    }

<a name="updating-the-default-payment-method"></a>
### Обновление способа оплаты по умолчанию

Метод "updateDefaultPaymentMethod" может использоваться для обновления информации о способе оплаты клиента по умолчанию. Этот метод принимает идентификатор платежного метода Stripe и назначает новый способ оплаты в качестве способа выставления счетов по умолчанию:

    $user->updateDefaultPaymentMethod($paymentMethod);

Чтобы синхронизировать информацию о вашем способе оплаты по умолчанию с информацией о способе оплаты клиента по умолчанию в Stripe, вы можете использовать метод `updateDefaultPaymentMethodFromStripe`:

    $user->updateDefaultPaymentMethodFromStripe();

> {note} Способ оплаты по умолчанию для клиента можно использовать только для выставления счетов и создания новых подписок. Из-за ограничений, налагаемых Stripe, его нельзя использовать для разовых платежей.

<a name="adding-payment-methods"></a>
### Добавление способа оплаты

Чтобы добавить новый способ оплаты, вы можете вызвать метод `addPaymentMethod` в оплачиваемой модели, передав идентификатор способа оплаты:

    $user->addPaymentMethod($paymentMethod);

> {tip} Чтобы узнать, как получить идентификаторы способов оплаты, пожалуйста, ознакомьтесь с [документацией по хранению способов оплаты] (#storing-payment-methods).

<a name="deleting-payment-methods"></a>
### Удаление способа оплаты

Чтобы удалить способ оплаты, вы можете вызвать метод `delete` в экземпляре `Laravel\Cashier\PaymentMethod`, который вы хотите удалить:

    $paymentMethod->delete();

Метод `deletePaymentMethod` удалит определенный способ оплаты из оплачиваемой модели:

    $user->deletePaymentMethod('pm_visa');

Метод `deletePaymentMethods` удалит всю информацию о способе оплаты для оплачиваемой модели:

    $user->deletePaymentMethods();

По умолчанию этот метод приведет к удалению способов оплаты типа `card`. Чтобы удалить способы оплаты другого типа, вы можете передать `type` в качестве аргумента методу:

    $user->deletePaymentMethods('sepa_debit');

> {note} Если у пользователя активная подписка, ваше приложение не должно позволять ему удалять способ оплаты по умолчанию.

<a name="subscriptions"></a>
## Подписки

Подписки предоставляют возможность настроить периодические платежи для ваших клиентов. Подписки Stripe, управляемые Cashier, обеспечивают поддержку нескольких цен подписки, количества подписок, пробных версий и многого другого.

<a name="creating-subscriptions"></a>
### Создание подписок

Чтобы создать подписку, сначала извлеките экземпляр вашей оплачиваемой модели, которая обычно будет экземпляром `App\Models\User`. После того, как вы извлекли экземпляр модели, вы можете использовать метод `newSubscription` для создания подписки на модель:

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $request->user()->newSubscription(
            'default', 'price_monthly'
        )->create($request->paymentMethodId);

        // ...
    });

Первым аргументом, передаваемым методу `newSubscription`, должно быть внутреннее имя подписки. Если ваше приложение предлагает только одну подписку, вы можете назвать ее `default` или `primary`. Это имя подписки предназначено только для внутреннего использования приложением и не предназначено для показа пользователям. Кроме того, он не должен содержать пробелов и никогда не должен быть изменен после создания подписки. Второй аргумент - это конкретная цена, на которую подписывается пользователь. Это значение должно соответствовать идентификатору цены в Stripe.

Метод `create`, который принимает [идентификатор способа оплаты Stripe] (#storing-payment-methods) или объект Stripe `PaymentMethod`, запустит подписку, а также обновит вашу базу данных идентификатором клиента Stripe для оплачиваемой модели и другой соответствующей платежной информацией.

> {note} Передача идентификатора способа оплаты непосредственно в метод подписки `create` также автоматически добавит его в сохраненные способы оплаты пользователя.

<a name="collecting-recurring-payments-via-invoice-emails"></a>
#### Сбор периодических платежей через выставление счетов по электронной почте

Вместо автоматического сбора периодических платежей клиента вы можете поручить Stripe отправлять клиенту счет по электронной почте каждый раз, когда наступает срок оплаты. Затем клиент может вручную оплатить счет, как только он его получит. Клиенту не нужно заранее указывать способ оплаты при получении периодических платежей по счетам-фактурам:

    $user->newSubscription('default', 'price_monthly')->createAndSendInvoice();

Время, в течение которого клиент должен оплатить свой счет до отмены подписки, определяется настройками вашей подписки и счета-фактуры в [панели управления Stripe](https://dashboard.stripe.com/settings/billing/automatic).

<a name="subscription-quantities"></a>
#### Величины

Если вы хотите установить конкретное [количество](https://stripe.com/docs/billing/subscriptions/quantities) для получения цены при создании подписки вам следует вызвать метод `quantity` в конструкторе подписок перед созданием подписки:

    $user->newSubscription('default', 'price_monthly')
         ->quantity(5)
         ->create($paymentMethod);

<a name="additional-details"></a>
#### Дополнительные сведения

Если вы хотите указать дополнительные параметры [клиенту](https://stripe.com/docs/api/customers/create) или [подписке](https://stripe.com/docs/api/subscriptions/create), поддерживаемые Stripe, вы можете сделать это, передав их в качестве второго и третьего аргументов методу `create`:

    $user->newSubscription('default', 'price_monthly')->create($paymentMethod, [
        'email' => $email,
    ], [
        'metadata' => ['note' => 'Some extra information.'],
    ]);

<a name="coupons"></a>
#### Купоны

Если вы хотите применить купон при создании подписки, вы можете использовать метод `withCoupon`:

    $user->newSubscription('default', 'price_monthly')
         ->withCoupon('code')
         ->create($paymentMethod);

Или, если вы хотите применить [промокод Stripe](https://stripe.com/docs/billing/subscriptions/discounts/codes), вы можете использовать метод `withPromotionCode`. Указанный идентификатор промо-кода должен быть идентификатором Stripe API, присвоенным промо-коду, а не промо-кодом, с которым сталкивается клиент:

    $user->newSubscription('default', 'price_monthly')
         ->withPromotionCode('promo_code')
         ->create($paymentMethod);

<a name="adding-subscriptions"></a>
#### Добавление подписок

Если вы хотите добавить подписку клиенту, у которого уже есть способ оплаты по умолчанию, вы можете вызвать метод `add` в конструкторе подписок:

    use App\Models\User;

    $user = User::find(1);

    $user->newSubscription('default', 'price_monthly')->add();

<a name="creating-subscriptions-from-the-stripe-dashboard"></a>
#### Создание подписок с помощью панели управления Stripe

Вы также можете создавать подписки с помощью самой панели управления Stripe. При этом Cashier синхронизирует вновь добавленные подписки и присваивает им имя `default`. Чтобы настроить имя подписки, которое присваивается подпискам, созданным на панели управления, [расширьте `WebhookController`](#defining-webhook-event-handlers) и перезапишите метод `newSubscriptionName`.

Кроме того, вы можете создать только один тип подписки через панель управления Stripe. Если ваше приложение предлагает несколько подписок с разными именами, через панель мониторинга Stripe можно добавить только один тип подписки.

Наконец, вы всегда должны быть уверены, что добавляете только одну активную подписку на каждый тип подписки, предлагаемый вашим приложением. Если у клиента есть две подписки `default`, Cashier будет использовать только самую последнюю добавленную подписку, даже если обе будут синхронизированы с базой данных вашего приложения.

<a name="checking-subscription-status"></a>
### Проверка статуса подписки

Как только клиент зарегистрируется в вашем приложении, вы можете легко проверить статус его подписки, используя различные удобные методы. Во-первых, метод `subscribed` возвращает `true`, если у клиента активная подписка, даже если в настоящее время срок действия подписки истекает. Метод `subscribed` принимает имя подписки в качестве своего первого аргумента:

    if ($user->subscribed('default')) {
        //
    }

Метод `subscribed` также является отличным кандидатом для [посредника роута] (/docs/{{version}}/middleware), позволяя вам фильтровать доступ к маршрутам и контроллерам на основе статуса подписки пользователя.:

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

Метод `subscribedToProduct` может использоваться для определения того, подписан ли пользователь на данный продукт, на основе идентификатора данного продукта Stripe. В Stripe товары представляют собой наборы цен. В этом примере мы определим, является ли подписка пользователя `default` активной подпиской на "премиум" продукт приложения. Указанный идентификатор продукта Stripe должен соответствовать одному из идентификаторов вашего продукта на панели мониторинга Stripe:

    if ($user->subscribedToProduct('prod_premium', 'default')) {
        //
    }

Передавая массив методу `subscribedToProduct`, вы можете определить, является ли подписка пользователя `default` активной подпиской на "базовый" или "премиум" продукт приложения.:

    if ($user->subscribedToProduct(['prod_basic', 'prod_premium'], 'default')) {
        //
    }

Метод `subscribedToPrice` может использоваться для определения того, соответствует ли подписка клиента заданному идентификатору цены:

    if ($user->subscribedToPrice('price_basic_monthly', 'default')) {
        //
    }

Метод `recurring` может быть использован для определения того, подписан ли пользователь в данный момент и не проходит ли пробный период:

    if ($user->subscription('default')->recurring()) {
        //
    }

> {note} Если у пользователя есть две подписки с одинаковым именем, самая последняя подписка всегда будет возвращена методом `subscription`. Например, у пользователя могут быть две записи подписки с именем `default`; однако одна из подписок может быть старой, срок действия которой истек, в то время как другая является текущей, активной подпиской. Самая последняя подписка всегда будет возвращена, в то время как более старые подписки хранятся в базе данных для просмотра истории.

<a name="cancelled-subscription-status"></a>
#### Статус отмененной подписки

Чтобы определить, был ли пользователь когда-то активным подписчиком, но отменил свою подписку, вы можете использовать метод `canceled`:

    if ($user->subscription('default')->canceled()) {
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

<a name="incomplete-and-past-due-status"></a>
#### Статус незавершенного и просроченного платежа

Если подписка требует повторной оплаты после создания, подписка будет помечена как `incomplete`. Статусы подписок хранятся в столбце `stripe_status` таблицы `subscriptions` базы данных Cashier.

Аналогично, если при замене цен требуется вторичное платежное действие, подписка будет помечена как `past_due`. Если ваша подписка находится в любом из этих состояний, она не будет активна до тех пор, пока клиент не подтвердит свой платеж. Определение того, имеет ли подписка неполную оплату, может быть выполнено с использованием метода `hasIncompletePayment` в оплачиваемой модели или экземпляре подписки:

    if ($user->hasIncompletePayment('default')) {
        //
    }

    if ($user->subscription('default')->hasIncompletePayment()) {
        //
    }

Если подписка оплачена не полностью, вы должны направить пользователя на страницу подтверждения оплаты в Cashier, указав идентификатор `latestPayment`. Вы можете использовать метод `latestPayment`, доступный в экземпляре подписки, для получения этого идентификатора:

```html
<a href="{{ route('cashier.payment', $subscription->latestPayment()->id) }}">
    Please confirm your payment.
</a>
```

Если вы хотите, чтобы подписка по-прежнему считалась активной, когда она находится в состоянии `past_due`, вы можете использовать метод `keepPastDueSubscriptionsActive`, предоставляемый Cashier. Как правило, этот метод должен вызываться в методе `register` вашего `App\Providers\AppServiceProvider`:

    use Laravel\Cashier\Cashier;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Cashier::keepPastDueSubscriptionsActive();
    }

> {note} Когда подписка находится в состоянии `incomplete`, она не может быть изменена до тех пор, пока платеж не будет подтвержден. Следовательно, методы `swap` и `updateQuantity` выдадут исключение, когда подписка находится в состоянии `incomplete`.

<a name="subscription-scopes"></a>
#### Диапазоны подписки

Большинство состояний подписки также доступны в виде диапазона запросов, так что вы можете легко запрашивать в своей базе данных подписки, находящиеся в заданном состоянии:

    // Get all active subscriptions...
    $subscriptions = Subscription::query()->active()->get();

    // Get all of the canceled subscriptions for a user...
    $subscriptions = $user->subscriptions()->canceled()->get();

Полный список доступных диапазонов доступен ниже:

    Subscription::query()->active();
    Subscription::query()->canceled();
    Subscription::query()->ended();
    Subscription::query()->incomplete();
    Subscription::query()->notCanceled();
    Subscription::query()->notOnGracePeriod();
    Subscription::query()->notOnTrial();
    Subscription::query()->onGracePeriod();
    Subscription::query()->onTrial();
    Subscription::query()->pastDue();
    Subscription::query()->recurring();

<a name="changing-prices"></a>
### Изменение цен

После того, как клиент подписался на ваше приложение, он может иногда захотеть перейти на новую цену подписки. Чтобы перевести клиента на новую цену, передайте идентификатор цены Stripe методу `swap`. При замене цен предполагается, что пользователь хотел бы повторно активировать свою подписку, если она была ранее отменена. Указанный идентификатор цены должен соответствовать идентификатору цены Stripe, доступному на панели мониторинга Stripe:

    use App\Models\User;

    $user = App\Models\User::find(1);

    $user->subscription('default')->swap('price_yearly');

Если клиент находится на пробной версии, пробный период будет сохранен. Кроме того, если для подписки существует "количество", это количество также будет поддерживаться.

Если вы хотите поменять цены и отменить любой пробный период, на котором в данный момент находится клиент, вы можете воспользоваться методом `skipTrial`:

    $user->subscription('default')
            ->skipTrial()
            ->swap('price_yearly');

Если вы хотите поменять цены и немедленно выставить счет клиенту, не дожидаясь его следующего цикла выставления счетов, вы можете использовать метод `swapAndInvoice`:

    $user = User::find(1);

    $user->subscription('default')->swapAndInvoice('price_yearly');

<a name="prorations"></a>
#### Пропорции

По умолчанию Stripe пропорционально распределяет сборы при переключении между ценами. Метод `noProrate` может быть использован для обновления цены подписки без пропорционального увеличения сборов:

    $user->subscription('default')->noProrate()->swap('price_yearly');

Для получения дополнительной информации о распределении подписок обратитесь к [документации Stripe](https://stripe.com/docs/billing/subscriptions/prorations).

> {note} Выполнение метода `noProrate` перед методом `swapAndInvoice` не окажет никакого влияния на распределение. Счет-фактура будет выставляться всегда.

<a name="subscription-quantity"></a>
### Количество подписки

Иногда на подписку влияет "количество". Например, приложение для управления проектами может взимать плату в размере 10 долларов США в месяц за каждый проект. Вы можете использовать методы `incrementQuantity` и `decrementQuantity`, чтобы легко увеличивать или уменьшать количество вашей подписки:

    use App\Models\User;

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

Для получения дополнительной информации о количестве подписок обратитесь к [документации Stripe](https://stripe.com/docs/subscriptions/quantities).

<a name="multiprice-subscription-quantities"></a>
#### Количество подписок по разным ценам

Если ваша подписка является [многоценовой подпиской](#multiprice-subscriptions), вам следует передать название цены, количество которой вы хотите увеличить или уменьшить, в качестве второго аргумента методам increment / decrement:

    $user->subscription('default')->incrementQuantity(1, 'price_chat');

<a name="multiprice-subscriptions"></a>
### Многоценовые подписки

[Многоценовые подписки](https://stripe.com/docs/billing/subscriptions/multiple-products) позволяют вам назначать несколько цен для выставления счетов одной подписке. Например, представьте, что вы создаете приложение "helpdesk" для обслуживания клиентов, базовая стоимость подписки на которое составляет 10 долларов в месяц, но которое предлагает дополнительный чат за дополнительные 15 долларов в месяц. Информация о подписке по нескольким ценам хранится в таблице `subscription_items` базы данных Cashier.

Вы можете указать несколько цен для данной подписки, передав массив цен в качестве второго аргумента методу `newSubscription`:

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $request->user()->newSubscription('default', [
            'price_monthly',
            'price_chat',
        ])->create($request->paymentMethodId);

        // ...
    });

В приведенном выше примере к подписке клиента `default` будут привязаны две цены. Обе цены будут взиматься с соответствующих интервалов выставления счетов. При необходимости вы можете использовать метод `quantity`, чтобы указать конкретное количество для каждой цены:

    $user = User::find(1);

    $user->newSubscription('default', ['price_monthly', 'price_chat'])
        ->quantity(5, 'price_chat')
        ->create($paymentMethod);

Если вы хотите добавить другую цену к существующей подписке, вы можете вызвать метод `addPrice` подписки:

    $user = User::find(1);

    $user->subscription('default')->addPrice('price_chat');

В приведенном выше примере будет добавлена новая цена, и клиенту будет выставлен счет за нее в следующем платежном цикле. Если вы хотите немедленно выставить счет клиенту, вы можете воспользоваться методом `addPriceAndInvoice`:

    $user->subscription('default')->addPriceAndInvoice('price_chat');

Если вы хотите добавить цену с определенным количеством, вы можете передать количество в качестве второго аргумента методов `addPrice` или `addPriceAndInvoice`:

    $user = User::find(1);

    $user->subscription('default')->addPrice('price_chat', 5);

Вы можете удалить цены из подписок, используя метод `removePrice`:

    $user->subscription('default')->removePrice('price_chat');

> {note} Вы не имеете права отменять последнюю цену подписки. Вместо этого вам следует просто отменить подписку.

<a name="swapping-prices"></a>
#### Изменение цен

Вы также можете изменить цены, привязанные к мультиценовой подписке. Например, представьте, что у клиента есть подписка `price_basic` с дополнительной ценой `price_chat`, и вы хотите обновить цену клиента с `price_basic` до `price_pro`:

    use App\Models\User;

    $user = User::find(1);

    $user->subscription('default')->swap(['price_pro', 'price_chat']);

При выполнении приведенного выше примера базовый элемент подписки с `price_basic` удаляется, а элемент с `price_chat` сохраняется. Кроме того, создается новый элемент подписки для `price_pro`.

Вы также можете указать параметры элемента подписки, передав массив пар ключ / значение методу `swap`. Например, вам может потребоваться указать стоимость подписки.:

    $user = User::find(1);

    $user->subscription('default')->swap([
        'price_pro' => ['quantity' => 5],
        'price_chat'
    ]);

Если вы хотите поменять одну цену на подписку, вы можете сделать это, используя метод `swap` для самого элемента подписки. Такой подход особенно полезен, если вы хотите сохранить все существующие метаданные о подписках и других ценах:

    $user = User::find(1);

    $user->subscription('default')
            ->findItemOrFail('price_basic')
            ->swap('price_pro');

<a name="proration"></a>
#### Пропорция

По умолчанию Stripe пропорционально распределяет расходы при добавлении или удалении цен из мультиценовой подписки. Если вы хотите произвести корректировку цены без пропорциональности, вам следует привязать метод `noProrate` к вашей ценовой операции:

    $user->subscription('default')->noProrate()->removePrice('price_chat');

<a name="swapping-quantities"></a>
#### Величины

Если вы хотите обновить количество по ценам отдельных подписок, вы можете сделать это с помощью [существующих методов определения количества] (#subscription-quantity), передав название цены в качестве дополнительного аргумента методу:

    $user = User::find(1);

    $user->subscription('default')->incrementQuantity(5, 'price_chat');

    $user->subscription('default')->decrementQuantity(3, 'price_chat');

    $user->subscription('default')->updateQuantity(10, 'price_chat');

> {note} Когда подписка имеет несколько цен, атрибуты `stripe_price` и `quantity` в модели `Subscription` будут равны `null`. Чтобы получить доступ к отдельным атрибутам цены, вы должны использовать связь `items`, доступную в модели `Subscription`.

<a name="subscription-items"></a>
#### Элементы подписки

Когда подписка имеет несколько цен, в таблице `subscription_items` вашей базы данных будет храниться несколько "элементов" подписки. Вы можете получить к ним доступ через связь `items` в подписке:

    use App\Models\User;

    $user = User::find(1);

    $subscriptionItem = $user->subscription('default')->items->first();

    // Retrieve the Stripe price and quantity for a specific item...
    $stripePrice = $subscriptionItem->stripe_price;
    $quantity = $subscriptionItem->quantity;

Вы также можете получить конкретную цену, используя метод `findItemOrFail`:

    $user = User::find(1);

    $subscriptionItem = $user->subscription('default')->findItemOrFail('price_chat');

<a name="metered-billing"></a>
### Дозированный расчет

[Дозированный расчет](https://stripe.com/docs/billing/subscriptions/metered-billing) позволяет взимать плату с клиентов в зависимости от использования ими продукта в течение платежного цикла. Например, вы можете взимать плату с клиентов в зависимости от количества текстовых сообщений или электронных писем, которые они отправляют в месяц.

Чтобы начать использовать дозированное выставление счетов, сначала вам нужно будет создать новый продукт на панели управления Stripe с дозированной ценой. Затем используйте `meteredPrice`, чтобы добавить идентификатор дозированной цены к подписке клиента:

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $request->user()->newSubscription('default')
            ->meteredPrice('price_metered')
            ->create($request->paymentMethodId);

        // ...
    });

Вы также можете запустить дозированную подписку через [Stripe Checkout](#checkout заказ):

    $checkout = Auth::user()
            ->newSubscription('default', [])
            ->meteredPrice('price_metered')
            ->checkout();

    return view('your-checkout-view', [
        'checkout' => $checkout,
    ]);

<a name="reporting-usage"></a>
#### Использование отчетов

По мере того, как ваш клиент будет использовать ваше приложение, вы будете сообщать Stripe об их использовании, чтобы можно было точно выставить им счет. Чтобы увеличить использование дозированной подписки, вы можете использовать метод `reportUsage`:

    $user = User::find(1);

    $user->subscription('default')->reportUsage();

По умолчанию к расчетному периоду добавляется "количество использований", равное 1. В качестве альтернативы, вы можете указать определенную сумму "использования", чтобы добавить ее к использованию клиента за расчетный период:

    $user = User::find(1);

    $user->subscription('default')->reportUsage(15);

Если ваше приложение предлагает несколько цен на одну подписку, вам нужно будет использовать метод `reportUsageFor`, чтобы указать измеренную цену, по которой вы хотите сообщить об использовании:

    $user = User::find(1);

    $user->subscription('default')->reportUsageFor('price_metered', 15);

Иногда вам может потребоваться обновить информацию об использовании, о котором вы сообщали ранее. Чтобы выполнить это, вы можете передать временную метку или экземпляр `DateTimeInterface` в качестве второго параметра `reportUsage`. При этом Stripe обновит данные об использовании, о которых сообщалось в данный момент времени. Вы можете продолжать обновлять предыдущие записи об использовании, поскольку указанные дата и время все еще находятся в пределах текущего расчетного периода:

    $user = User::find(1);

    $user->subscription('default')->reportUsage(5, $timestamp);

<a name="retrieving-usage-records"></a>
#### Извлечение записей об использовании

Чтобы получить информацию о прошлом использовании клиента, вы можете использовать метод `usageRecords` экземпляров подписки:

    $user = User::find(1);

    $usageRecords = $user->subscription('default')->usageRecords();

Если ваше приложение предлагает несколько цен на одну подписку, вы можете использовать метод `usageRecordsFor`, чтобы указать измеренную цену, для которой вы хотите получить записи об использовании:

    $user = User::find(1);

    $usageRecords = $user->subscription('default')->usageRecordsFor('price_metered');

Методы `usageRecords` и `usageRecordsFor` возвращают экземпляр коллекции, содержащий ассоциативный массив записей об использовании. Вы можете выполнить итерацию по этому массиву, чтобы отобразить общее использование клиентом:

    @foreach ($usageRecords as $usageRecord)
        - Period Starting: {{ $usageRecord['period']['start'] }}
        - Period Ending: {{ $usageRecord['period']['end'] }}
        - Total Usage: {{ $usageRecord['total_usage'] }}
    @endforeach

Для получения полной информации обо всех возвращаемых данных об использовании и о том, как использовать разбивку на страницы Stripe на основе курсора, пожалуйста, обратитесь к [официальной документации Stripe API](https://stripe.com/docs/api/usage_records/subscription_item_summary_list).

<a name="subscription-taxes"></a>
### Налоги подписки

> {note} Вместо расчета налоговых ставок вручную, вы можете [автоматически рассчитать налоги с помощью Stripe Tax](#tax-configuration)

Чтобы указать налоговые ставки, которые пользователь платит по подписке, вам следует реализовать метод `taxRates` в вашей оплачиваемой модели и вернуть массив, содержащий идентификаторы налоговых ставок Stripe. Вы можете определить эти налоговые ставки в [вашей информационной панели Stripe](https://dashboard.stripe.com/test/tax-rates):

    /**
     * The tax rates that should apply to the customer's subscriptions.
     *
     * @return array
     */
    public function taxRates()
    {
        return ['txr_id'];
    }

Метод `taxRates` позволяет вам применять налоговую ставку для каждого отдельного клиента, что может быть полезно для базы пользователей, охватывающей несколько стран и налоговых ставок.

Если вы предлагаете подписку по нескольким ценам, вы можете определить разные налоговые ставки для каждой цены, внедрив метод `priceTaxRates` в вашей оплачиваемой модели:

    /**
     * The tax rates that should apply to the customer's subscriptions.
     *
     * @return array
     */
    public function priceTaxRates()
    {
        return [
            'price_monthly' => ['txr_id'],
        ];
    }

> {note} Метод `taxRates` применяется только к оплате подписки. Если вы используете Cashier для осуществления разовых платежей, вам нужно будет вручную указать налоговую ставку на тот момент.

<a name="syncing-tax-rates"></a>
#### Синхронизация налоговых ставок

При изменении жестко закодированных идентификаторов налоговых ставок, возвращаемых методом `taxRates`, налоговые настройки для любых существующих подписок пользователя останутся прежними. Если вы хотите обновить значение налога для существующих подписок новыми значениями `taxRates`, вам следует вызвать метод `syncTaxRates` в экземпляре подписки пользователя:

    $user->subscription('default')->syncTaxRates();

Это также позволит синхронизировать любые налоговые ставки по элементам подписки с несколькими ценами. Если ваше приложение предлагает многозначные подписки, вам следует убедиться, что ваша оплачиваемая модель реализует метод `priceTaxRates` [обсуждался выше](#subscription-taxes).

<a name="tax-exemption"></a>
#### Освобождение от уплаты налогов

Cashier также предлагает методы `isNotTaxExempt`, `isTaxExempt` и `reverseChargeApplies`, чтобы определить, освобожден ли клиент от уплаты налогов. Эти методы будут вызывать Stripe API для определения статуса освобождения клиента от уплаты налогов:

    use App\Models\User;

    $user = User::find(1);

    $user->isTaxExempt();
    $user->isNotTaxExempt();
    $user->reverseChargeApplies();

> {note} Эти методы также доступны для любого объекта `Laravel\Cashier\Invoice`. Однако при вызове объекта `Invoice` методы будут определять статус исключения на момент создания счета-фактуры.

<a name="subscription-anchor-date"></a>
### Дата привязки подписки

По умолчанию привязкой платежного цикла является дата создания подписки или, если используется пробный период, дата окончания пробной версии. Если вы хотите изменить дату привязки счета, вы можете использовать метод `anchorBillingCycleOn`:

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $anchor = Carbon::parse('first day of next month');

        $request->user()->newSubscription('default', 'price_monthly')
                    ->anchorBillingCycleOn($anchor->startOfDay())
                    ->create($request->paymentMethodId);

        // ...
    });

Для получения дополнительной информации об управлении циклами выставления счетов по подписке обратитесь к [документации по циклу выставления счетов Stripe](https://stripe.com/docs/billing/subscriptions/billing-cycle)

<a name="cancelling-subscriptions"></a>
### Отмена подписки

Чтобы отменить подписку, вызовите метод `cancel` в подписке пользователя:

    $user->subscription('default')->cancel();

Когда подписка отменяется, Cashier автоматически установит столбец `ends_at` в вашей таблице `subscriptions` базы данных. Этот столбец используется, чтобы узнать, когда метод `subscribed` должен начать возвращать `false`.

Например, если клиент отменяет подписку 1 марта, но завершение подписки не планировалось до 5 марта, метод `subscribed` будет продолжать возвращать `true` до 5 марта. Это делается потому, что пользователю обычно разрешается продолжать использовать приложение до окончания платежного цикла.

Вы можете определить, отменил ли пользователь свою подписку, но все еще находится в "льготном периоде", используя метод `onGracePeriod`:

    if ($user->subscription('default')->onGracePeriod()) {
        //
    }

Если вы хотите немедленно отменить подписку, вызовите метод `cancelNow` в подписке пользователя:

    $user->subscription('default')->cancelNow();

Если вы хотите немедленно отменить подписку и выставить счет за любое оставшееся неучтенным дозированное использование или новые / ожидающие оплаты элементы счета-фактуры, вызовите метод `cancelNowAndInvoice` для подписки пользователя:

    $user->subscription('default')->cancelNowAndInvoice();

Вы также можете отменить подписку в определенный момент времени:

    $user->subscription('default')->cancelAt(
        now()->addDays(10)
    );

<a name="resuming-subscriptions"></a>
### Возобновление подписок

Если клиент отменил свою подписку, и вы хотите возобновить ее, вы можете вызвать метод `resume` для подписки. Клиент все еще должен находиться в пределах своего "льготного периода", чтобы возобновить подписку:

    $user->subscription('default')->resume();

Если клиент отменяет подписку, а затем возобновляет ее до того, как срок действия подписки полностью истечет, счет клиенту не будет выставлен немедленно. Вместо этого их подписка будет повторно активирована, и им будет выставлен счет в первоначальном платежном цикле.

<a name="subscription-trials"></a>
## Пробные периоды

<a name="with-payment-method-up-front"></a>
### С указанием способа оплаты

Если вы хотите предложить своим клиентам пробные периоды, предварительно собирая информацию о способе оплаты, вам следует использовать метод `trialDays` при создании своих подписок:

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $request->user()->newSubscription('default', 'price_monthly')
                    ->trialDays(10)
                    ->create($request->paymentMethodId);

        // ...
    });

Этот метод установит дату окончания пробного периода в записи подписки в базе данных и проинструктирует Stripe не начинать выставление счетов клиенту до истечения этой даты. При использовании метода `trialDays` Cashier перезапишет любой пробный период по умолчанию, настроенный для цены в Stripe.

> {note} Если подписка клиента не будет отменена до даты окончания пробной версии, с него будет снята плата, как только истечет срок действия пробной версии, поэтому вам следует обязательно уведомить своих пользователей о дате окончания пробной версии.

Метод `trialUntil` позволяет вам предоставить экземпляр `DateTime`, который указывает, когда должен закончиться пробный период:

    use Carbon\Carbon;

    $user->newSubscription('default', 'price_monthly')
                ->trialUntil(Carbon::now()->addDays(10))
                ->create($paymentMethod);

Вы можете определить, находится ли пользователь в пределах своего пробного периода, используя либо метод `onTrial` экземпляра пользователя, либо метод `onTrial` экземпляра подписки. Два приведенных ниже примера эквивалентны:

    if ($user->onTrial('default')) {
        //
    }

    if ($user->subscription('default')->onTrial()) {
        //
    }

Вы можете использовать метод `endTrial`, чтобы немедленно завершить пробную версию подписки:

    $user->subscription('default')->endTrial();

<a name="defining-trial-days-in-stripe-cashier"></a>
#### Определение пробных дней в Stripe / Cashier

Вы можете указать, сколько пробных дней будет действовать ваша цена, на панели управления Stripe или всегда передавать их явно с помощью Cashier. Если вы решите определить пробные дни вашей цены в Stripe, вы должны знать, что новые подписки, включая новые подписки для клиента, у которого была подписка в прошлом, всегда будут получать пробный период, если вы явно не вызовете метод `skipTrial()`.

<a name="without-payment-method-up-front"></a>
### Без указания способа оплаты

Если вы хотите предлагать пробные периоды без предварительного сбора информации о способе оплаты пользователя, вы можете установить в столбце `trial_ends_at` в записи пользователя желаемую дату окончания пробной версии. Обычно это делается во время регистрации пользователя:

    use App\Models\User;

    $user = User::create([
        // ...
        'trial_ends_at' => now()->addDays(10),
    ]);

> {note} Обязательно добавьте [приведение даты](/docs/{{version}}/eloquent-mutators##date-casting) для атрибута `trial_ends_at` в определении класса вашей оплачиваемой модели.

Cashier называет этот тип пробной версии "общей пробной версией", поскольку она не привязана ни к одной существующей подписке. Метод `onTrial` в экземпляре оплачиваемой модели вернет `true`, если текущая дата не превышает значения `trial_ends_at`:

    if ($user->onTrial()) {
        // User is within their trial period...
    }

Как только вы будете готовы создать фактическую подписку для пользователя, вы можете использовать метод `newSubscription`, как обычно:

    $user = User::find(1);

    $user->newSubscription('default', 'price_monthly')->create($paymentMethod);

Чтобы получить дату окончания пробной версии пользователя, вы можете использовать метод `trialEndsAt`. Этот метод вернет экземпляр даты Carbon, если пользователь находится на пробной версии, или `null`, если это не так. Вы также можете передать необязательный параметр имени подписки, если хотите получить дату окончания пробной версии для конкретной подписки, отличной от подписки по умолчанию:

    if ($user->onTrial()) {
        $trialEndsAt = $user->trialEndsAt('main');
    }

Вы также можете использовать метод `onGenericTrial`, если хотите точно знать, что пользователь находится в пределах своего "общего" пробного периода и еще не создал фактическую подписку:

    if ($user->onGenericTrial()) {
        // User is within their "generic" trial period...
    }

<a name="extending-trials"></a>
### Продление пробного периода

Метод `extendTrial` позволяет вам продлить пробный период подписки после того, как подписка была создана. Если срок действия пробной версии уже истек и клиенту уже выставлен счет за подписку, вы все равно можете предложить ему расширенную пробную версию. Время, потраченное в течение пробного периода, будет вычтено из следующего счета клиента:

    use App\Models\User;

    $subscription = User::find(1)->subscription('default');

    // End the trial 7 days from now...
    $subscription->extendTrial(
        now()->addDays(7)
    );

    // Add an additional 5 days to the trial...
    $subscription->extendTrial(
        $subscription->trial_ends_at->addDays(5)
    );

<a name="handling-stripe-webhooks"></a>
## Обработка Stripe веб-хуков

> {tip} Вы можете использовать [интерфейс Stripe CLI](https://stripe.com/docs/stripe-clip), чтобы помочь протестировать веб-хуки во время локальной разработки.

Stripe может уведомлять ваше приложение о различных событиях с помощью веб-хуков. По умолчанию маршрут, который указывает на контроллер веб-хука Cashier, автоматически регистрируется сервис-провайдером Cashier. Этот контроллер будет обрабатывать все входящие запросы веб-хука.

По умолчанию контроллер веб-хук Cashier автоматически обрабатывает отмену подписок, в которых слишком много сброшенных платежей (как определено в настройках Stripe), обновления клиентов, удаления клиентов, обновления подписки и изменения способа оплаты; однако, как мы скоро узнаем, вы можете расширить этот контроллер для обработки любого события вэб-хука Stripe как вам нравится.

Чтобы убедиться, что ваше приложение может обрабатывать веб-хуки Stripe, обязательно настройте URL-адрес веб-хука на панели управления Stripe. По умолчанию контроллер веб-хука Cashier отвечает на URL-адрес `/stripe/webhook`. Полный список всех веб-подключений, которые вы должны включить на панели управления Stripe, выглядит следующим образом:

- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `customer.updated`
- `customer.deleted`
- `invoice.payment_action_required`

Для удобства в Cashier включена команда Artisan `cashier:webhook`. Эта команда создаст веб-хука в Stripe, который прослушивает все события, требуемые Cashier:

    php artisan cashier:webhook

По умолчанию созданный веб-хук будет указывать на URL, определенный переменной окружения `APP_URL` и `cashier.webhook` маршрут, который входит в комплект поставки Cashier. Вы можете указать параметр `--url` при вызове команды, если хотите использовать другой URL:

    php artisan cashier:webhook --url "https://example.com/stripe/webhook"

Созданный веб-хук будет использовать версию Stripe API, с которой совместима ваша версия Cashier. Если вы хотите использовать другую версию Stripe, вы можете указать опцию `--api-version`:

    php artisan cashier:webhook --api-version="2019-12-03"

После создания веб-хук будет немедленно активен. Если вы хотите создать веб-хук, но отключить его до тех пор, пока не будете готовы, вы можете указать опцию `--disabled` при вызове команды:

    php artisan cashier:webhook --disabled

> {note} Убедитесь, что вы защищаете входящие запросы веб-хук Stripe с помощью встроенного промежуточного программного обеспечения Cashier [проверка подписи веб-хука](#verifying-webhook-signatures).

<a name="webhooks-csrf-protection"></a>
#### Веб-хуки и защита от CSRF

Поскольку веб-хуки Stripe необходимо обходить Laravel [защиту от CSRF](/docs/{{version}}/csrf), обязательно укажите URI как исключение в промежуточном программном обеспечении вашего приложения `App\Http\Middleware\VerifyCsrfToken` или укажите маршрут вне группы промежуточного программного обеспечения `web`:

    protected $except = [
        'stripe/*',
    ];

<a name="defining-webhook-event-handlers"></a>
### Определение веб-хука событий

Cashier автоматически обрабатывает отмены подписки в случае несвоевременных платежей и других распространенных событий веб-хука Stripe. Однако, если у вас есть дополнительные события вэб-хука, которые вы хотели бы обработать, вы можете сделать это, прослушав следующие события, отправляемые Cashier:

- `Laravel\Cashier\Events\WebhookReceived`
- `Laravel\Cashier\Events\WebhookHandled`

Оба события содержат полную полезную нагрузку веб-хука Stripe. Например, если вы хотите обработать веб-запрос `invoice.payment_succeeded`, вы можете зарегистрировать [прослушивателя](/docs/{{version}}/events#defining-listeners), который будет обрабатывать событие:

    <?php

    namespace App\Listeners;

    use Laravel\Cashier\Events\WebhookReceived;

    class StripeEventListener
    {
        /**
         * Handle received Stripe webhooks.
         *
         * @param  \Laravel\Cashier\Events\WebhookReceived  $event
         * @return void
         */
        public function handle(WebhookReceived $event)
        {
            if ($event->payload['type'] === 'invoice.payment_succeeded') {
                // Handle the incoming event...
            }
        }
    }

Как только ваш прослушиватель определен, вы можете зарегистрировать его в `EventServiceProvider` вашего приложения:

    <?php

    namespace App\Providers;

    use App\Listeners\StripeEventListener;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
    use Laravel\Cashier\Events\WebhookReceived;

    class EventServiceProvider extends ServiceProvider
    {
        protected $listen = [
            WebhookReceived::class => [
                StripeEventListener::class,
            ],
        ];
    }

<a name="verifying-webhook-signatures"></a>
### Проверка подписей веб-хука

Чтобы обезопасить свои веб-крючки, вы можете использовать [подписи Stripe веб-хука](https://stripe.com/docs/webhooks/signatures). Для удобства Cashier автоматически включает промежуточное программное обеспечение, которое проверяет правильность входящего запроса веб-хука Stripe.

Чтобы включить проверку веб-хука, убедитесь, что переменная окружения `STRIPE_WEBHOOK_SECRET` установлена в файле `.env` вашего приложения. `Secret` веб-хука можно получить с панели управления вашей учетной записи Stripe.

<a name="single-charges"></a>
## Разовые списания

<a name="simple-charge"></a>
### Разовое списание

> {note} Метод `charge` принимает сумму, которую вы хотели бы списать, в наименьшем знаменателе валюты, используемой вашим приложением. Например, при использовании долларов США суммы следует указывать в пенни.

Если вы хотите произвести единовременное списание средств с клиента, вы можете использовать метод `charge` для экземпляра модели, подлежащего оплате. Вам нужно будет [указать идентификатор способа оплаты](#payment-methods-for-single-charges) в качестве второго аргумента метода `charge`:

    use Illuminate\Http\Request;

    Route::post('/purchase', function (Request $request) {
        $stripeCharge = $request->user()->charge(
            100, $request->paymentMethodId
        );

        // ...
    });

Метод `charge` принимает массив в качестве своего третьего аргумента, позволяя вам передавать любые параметры, которые вы пожелаете, для базового процесса создания Stripe charge. Более подробную информацию о вариантах, доступных вам при создании платежей, можно найти в [документации Stripe](https://stripe.com/docs/api/charges/create):

    $user->charge(100, $paymentMethod, [
        'custom_option' => $value,
    ]);

Вы также можете использовать метод `charge` без участия основного клиента или пользователя. Чтобы выполнить это, вызовите метод `charge` в новом экземпляре оплачиваемой модели вашего приложения:

    use App\Models\User;

    $stripeCharge = (new User)->charge(100, $paymentMethod);

Метод `charge` выдаст исключение, если списание завершится неудачей. Если списание пройдет успешно, экземпляр `Laravel\Cashier\Payment` будет возвращен из метода:

    try {
        $payment = $user->charge(100, $paymentMethod);
    } catch (Exception $e) {
        //
    }

<a name="charge-with-invoice"></a>
### Списание со счетом

Иногда вам может потребоваться произвести единовременную оплату и предложить своему клиенту квитанцию в формате PDF. Метод `invoicePrice` позволяет вам сделать именно это. Например, давайте выставим клиенту счет за пять новых рубашек:

    $user->invoicePrice('price_tshirt', 5);

Счет будет немедленно списан с использованием способа оплаты, используемого пользователем по умолчанию. Метод `invoicePrice` также принимает массив в качестве своего третьего аргумента. Этот массив содержит параметры выставления счетов для элемента счета-фактуры. Четвертый аргумент, принимаемый методом, также является массивом, который должен содержать параметры выставления счета для самого счета-фактуры:

    $user->invoicePrice('price_tshirt', 5, [
        'discounts' => [
            ['coupon' => 'SUMMER21SALE']
        ],
    ], [
        'default_tax_rates' => ['txr_id'],
    ]);

В качестве альтернативы, вы можете использовать метод `invoiceFor`, чтобы произвести "единовременную" оплату за счет способа оплаты, используемого клиентом по умолчанию:

    $user->invoiceFor('One Time Fee', 500);

Хотя вам доступен метод `invoiceFor`, рекомендуется использовать метод `invoicePrice` с заранее определенными ценами. Поступая таким образом, вы получите доступ к улучшенной аналитике и данным на панели мониторинга Stripe, касающимся ваших продаж по каждому продукту.

> {note} Методы `invoicePrice` и `invoiceFor` создадут накладную Stripe, которая повторит неудачные попытки выставления счета. Если вы не хотите, чтобы счета-фактуры повторяли неудачные начисления, вам нужно будет закрыть их с помощью Stripe API после первого неудачного начисления.

<a name="refunding-charges"></a>
### Возврат списаниий

Если вам необходимо возместить стоимость Stripe, вы можете воспользоваться методом `refund`. Этот метод принимает Stripe [идентификатор намерения платежа](#payment-methods-for-single-charges) в качестве своего первого аргумента:

    $payment = $user->charge(100, $paymentMethodId);

    $user->refund($payment->id);

<a name="invoices"></a>
## Счета

<a name="retrieving-invoices"></a>
### Получение счетов

Вы можете легко получить массив счетов-фактур оплачиваемой модели, используя метод `invoices`. Метод `invoices` возвращает коллекцию экземпляров `Laravel\Cashier\Invoice`:

    $invoices = $user->invoices();

Если вы хотите включить в результаты отложенные счета-фактуры, вы можете использовать метод `invoicesIncludingPending`:

    $invoices = $user->invoicesIncludingPending();

Вы можете использовать метод `findInvoice` для получения конкретного счета-фактуры по его идентификатору:

    $invoice = $user->findInvoice($invoiceId);

<a name="displaying-invoice-information"></a>
#### Отображение информации о счете-фактуре

При перечислении счетов-фактур для клиента вы можете использовать методы счета-фактуры для отображения соответствующей информации о счете-фактуре. Например, вы можете захотеть отобразить каждый счет-фактуру в таблице, что позволит пользователю легко загрузить любой из них:

    <table>
        @foreach ($invoices as $invoice)
            <tr>
                <td>{{ $invoice->date()->toFormattedDateString() }}</td>
                <td>{{ $invoice->total() }}</td>
                <td><a href="/user/invoice/{{ $invoice->id }}">Download</a></td>
            </tr>
        @endforeach
    </table>

<a name="upcoming-invoices"></a>
### Предстоящие счета-фактуры

Чтобы получить предстоящий счет-фактуру для клиента, вы можете использовать метод `upcomingInvoice`:

    $invoice = $user->upcomingInvoice();

Аналогично, если у клиента несколько подписок, вы также можете получить предстоящий счет-фактуру для конкретной подписки:

    $invoice = $user->subscription('default')->upcomingInvoice();

<a name="previewing-subscription-invoices"></a>
### Предварительный просмотр счетов-фактур по подписке

Используя метод `previewInvoice`, вы можете просмотреть счет-фактуру перед внесением изменений в цену. Это позволит вам определить, как будет выглядеть счет вашего клиента при изменении цены:

    $invoice = $user->subscription('default')->previewInvoice('price_yearly');

Вы можете передать массив цен методу `previewInvoice`, чтобы просмотреть счета-фактуры с несколькими новыми ценами:

    $invoice = $user->subscription('default')->previewInvoice(['price_yearly', 'price_metered']);

<a name="generating-invoice-pdfs"></a>
### Генерация счетов PDF

Находясь внутри маршрута или контроллера, вы можете использовать метод `downloadInvoice` для создания PDF-загрузки данного счета-фактуры. Этот метод автоматически сгенерирует соответствующий HTTP-ответ, необходимый для загрузки счета-фактуры:

    use Illuminate\Http\Request;

    Route::get('/user/invoice/{invoice}', function (Request $request, $invoiceId) {
        return $request->user()->downloadInvoice($invoiceId, [
            'vendor' => 'Your Company',
            'product' => 'Your Product',
        ]);
    });

По умолчанию все данные в счете-фактуре получены из данных клиента и счета-фактуры, хранящихся в Stripe. Однако вы можете настроить некоторые из этих данных, предоставив массив в качестве второго аргумента методу `downloadInvoice`. Этот массив позволяет вам настраивать информацию, такую как сведения о вашей компании и продукте:

    return $request->user()->downloadInvoice($invoiceId, [
        'vendor' => 'Your Company',
        'product' => 'Your Product',
        'street' => 'Main Str. 1',
        'location' => '2000 Antwerp, Belgium',
        'phone' => '+32 499 00 00 00',
        'email' => 'info@example.com',
        'url' => 'https://example.com',
        'vendorVat' => 'BE123456789',
    ], 'my-invoice');

Метод `downloadInvoice` также допускает пользовательское имя файла с помощью своего третьего аргумента. К этому имени файла автоматически будет добавлен суффикс `.pdf`:

    return $request->user()->downloadInvoice($invoiceId, [], 'my-invoice');

<a name="custom-invoice-render"></a>
#### Средство отображения пользовательских счетов-фактур

Cashier также позволяет использовать пользовательский инструмент отображения счетов-фактур. По умолчанию Cashier использует реализацию `DompdfInvoiceRenderer`, которая использует библиотеку PHP [dompdf](https://github.com/dompdf/dompdf) для генерации счетов Cashier. Однако вы можете использовать любой рендерер, который пожелаете, реализовав интерфейс `Laravel\Cashier\Contracts\InvoiceRenderer`. Например, вы можете захотеть отобразить PDF-файл счета-фактуры с помощью вызова API стороннего сервиса отображения PDF-файлов:

    use Illuminate\Support\Facades\Http;
    use Laravel\Cashier\Contracts\InvoiceRenderer;
    use Laravel\Cashier\Invoice;

    class ApiInvoiceRenderer implements InvoiceRenderer
    {
        /**
         * Render the given invoice and return the raw PDF bytes.
         *
         * @param  \Laravel\Cashier\Invoice. $invoice
         * @param  array  $data
         * @param  array  $options
         * @return string
         */
        public function render(Invoice $invoice, array $data = [], array $options = []): string
        {
            $html = $invoice->view($data)->render();

            return Http::get('https://example.com/html-to-pdf', ['html' => $html])->get()->body();
        }
    }

После того, как вы внедрили контракт с обработчиком счетов-фактур, вам следует обновить значение конфигурации `cashier.invoices.renderer` в настройках файл конфигурации `config/cashier.php` вашего приложения. Это значение конфигурации должно быть установлено в качестве имени класса вашей пользовательской реализации средства визуализации.

<a name="checkout"></a>
## Оформление

Cashier Stripe также предоставляет поддержку [Stripe Checkout](https://stripe.com/payments/checkout). Stripe Checkout избавляет от необходимости внедрять пользовательские страницы для приема платежей, предоставляя предварительно созданную размещенную платежную страницу.

Следующая документация содержит информацию о том, как начать использовать Stripe Checkout с помощью Cashier. Чтобы узнать больше о Stripe Checkout, вам также следует рассмотреть возможность ознакомления с [собственной документацией Stripes по оформлению заказа](https://stripe.com/docs/payments/checkout).

<a name="product-checkouts"></a>
### Оформление заказа продукта

Вы можете выполнить оформление заказа для существующего продукта, который был создан в вашей информационной панели Stripe, используя метод `checkout` для оплачиваемой модели. Метод `checkout` инициирует новый сеанс оформления заказа Stripe. По умолчанию от вас требуется ввести идентификатор цены Stripe:

    use Illuminate\Http\Request;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout('price_tshirt');
    });

При необходимости вы также можете указать количество продукта:

    use Illuminate\Http\Request;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout(['price_tshirt' => 15]);
    });

Когда клиент посещает этот маршрут, он будет перенаправлен на страницу оформления заказа Stripe. По умолчанию, когда пользователь успешно завершает или отменяет покупку, он будет перенаправлен на ваш маршрут `home`, но вы можете указать пользовательские URL-адреса обратного вызова, используя опции `success_url` и `cancel_url`:

    use Illuminate\Http\Request;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout(['price_tshirt' => 1], [
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
    });

При определении вашей опции оформления заказа `success_url` вы можете указать Stripe добавить идентификатор сеанса оформления заказа в качестве параметра строки запроса при вызове вашего URL. Для этого добавьте буквальную строку `{CHECKOUT_SESSION_ID}` в строку вашего запроса `success_url`. Stripe заменит этот заполнитель фактическим идентификатором сеанса оформления заказа:

    use Illuminate\Http\Request;
    use Stripe\Checkout\Session;
    use Stripe\Customer;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout(['price_tshirt' => 1], [
            'success_url' => route('checkout-success') . '?session_id={CHECKOUT_SESSION_ID}',
            'cancel_url' => route('checkout-cancel'),
        ]);
    });

    Route::get('/checkout-success', function (Request $request) {
        $checkoutSession = $request->user()->stripe()->checkout->sessions->retrieve($request->get('session_id'));

        return view('checkout.success', ['checkoutSession' => $checkoutSession]);
    })->name('checkout-success');

<a name="checkout-promotion-codes"></a>
#### Промокоды

По умолчанию Stripe Checkout не разрешает [промокоды, которые могут быть использованы пользователем](https://stripe.com/docs/billing/subscriptions/discounts/codes). К счастью, есть простой способ включить их на вашей странице оформления заказа. Для этого вы можете вызвать метод `allowPromotionCodes`:

    use Illuminate\Http\Request;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()
            ->allowPromotionCodes()
            ->checkout('price_tshirt');
    });

<a name="single-charge-checkouts"></a>
### Оформление одиночного списания

Вы также можете выполнить простую оплату за специальный продукт, который не был создан в вашей информационной панели Stripe. Для этого вы можете использовать метод `checkoutCharge` для модели, подлежащей оплате, и передать ей подлежащую оплате сумму, название продукта и необязательное количество. Когда клиент посещает этот маршрут, он будет перенаправлен на страницу оформления заказа Stripe:

    use Illuminate\Http\Request;

    Route::get('/charge-checkout', function (Request $request) {
        return $request->user()->checkoutCharge(1200, 'T-Shirt', 5);
    });

> {note} При использовании метода `checkoutCharge` Stripe всегда будет создавать новый продукт и цену на вашей информационной панели Stripe. Поэтому мы рекомендуем вам предварительно создать товары на панели управления Stripe и вместо этого использовать метод `checkout`.

<a name="subscription-checkouts"></a>
### Оформление заказа подписки

> {примечание} Для использования Stripe Checkout для подписок требуется включить веб-хук `customer.subscription.created` на панели управления Stripe. Этот веб-хук создаст запись подписки в вашей базе данных и сохранит все соответствующие элементы подписки.

Вы также можете использовать Stripe Checkout для инициирования подписки. После определения вашей подписки с помощью методов построения подписки Cashier, вы можете вызвать метод `checkout`. Когда клиент посещает этот маршрут, он будет перенаправлен на страницу оформления заказа Stripe:

    use Illuminate\Http\Request;

    Route::get('/subscription-checkout', function (Request $request) {
        return $request->user()
            ->newSubscription('default', 'price_monthly')
            ->checkout();
    });

Как и в случае с проверкой товара, вы можете настроить URL-адреса для подтверждения и отмены заказа:

    use Illuminate\Http\Request;

    Route::get('/subscription-checkout', function (Request $request) {
        return $request->user()
            ->newSubscription('default', 'price_monthly')
            ->checkout([
                'success_url' => route('your-success-route'),
                'cancel_url' => route('your-cancel-route'),
            ]);
    });

Конечно, вы также можете включить промо-коды для оформления подписки:

    use Illuminate\Http\Request;

    Route::get('/subscription-checkout', function (Request $request) {
        return $request->user()
            ->newSubscription('default', 'price_monthly')
            ->allowPromotionCodes()
            ->checkout();
    });

> {note} К сожалению, Stripe Checkout не поддерживает все параметры выставления счетов при запуске подписки. Использование метода `anchorBillingCycleOn` в конструкторе подписок, настройка пропорционального поведения или настройка режима оплаты не будут иметь никакого эффекта во время сеансов оформления заказа Stripe. Пожалуйста, ознакомьтесь с [документацией Stripe Checkout Session API](https://stripe.com/docs/api/checkout/sessions/create), чтобы просмотреть, какие параметры доступны.

<a name="stripe-checkout-trial-periods"></a>
#### Оформление заказа Stripe и пробные периоды

Конечно, вы можете определить пробный период при создании подписки, которая будет завершена с помощью Stripe Checkout:

    $checkout = Auth::user()->newSubscription('default', 'price_monthly')
        ->trialDays(3)
        ->checkout();

Однако пробный период должен составлять не менее 48 часов, что является минимальным сроком пробной версии, поддерживаемым Stripe Checkout.

<a name="stripe-checkout-subscriptions-and-webhooks"></a>
#### Подписки и веб-хуки

Помните, что Stripe и Cashier обновляют статусы подписки через веб-хуки, поэтому существует вероятность того, что подписка может быть еще не активна, когда клиент вернется в приложение после ввода своей платежной информации. Чтобы справиться с этим сценарием, вы можете захотеть отобразить сообщение, информирующее пользователя о том, что его оплата или подписка ожидаются.

<a name="collecting-tax-ids"></a>
### Сбор идентификаторов налогоплательщиков

Оформление заказа также поддерживает сбор налогового идентификатора клиента. Чтобы включить это в сеансе проверки, вызовите метод `collectTaxIds` при создании сеанса:

    $checkout = $user->collectTaxIds()->checkout('price_tshirt');

При вызове этого метода клиенту будет доступен новый флажок, который позволяет ему указать, совершает ли он покупку от имени компании. Если это так, у них будет возможность указать свой идентификационный номер налогоплательщика.

> {note} Если вы уже настроили [автоматический сбор налогов](#tax-configuration) у сервис-провайдера вашего приложения, то эта функция будет включена автоматически, и нет необходимости вызывать метод `collectTaxIds`.

<a name="handling-failed-payments"></a>
## Обработка неудачных платежей

Иногда платежи за подписку или разовые платежи могут не выполняться. Когда это произойдет, Cashier выдаст исключение `Laravel\Cashier\Exceptions\IncompletePayment`, которое информирует вас о том, что это произошло. После перехвата этого исключения у вас есть два варианта дальнейших действий.

Во-первых, вы могли бы перенаправить своего клиента на специальную страницу подтверждения платежа, которая входит в комплект поставки Cashier. На этой странице уже есть связанный именованный маршрут, зарегистрированный через сервис-провайдера Cashier. Таким образом, вы можете перехватить исключение `IncompletePayment` и перенаправить пользователя на страницу подтверждения платежа:

    use Laravel\Cashier\Exceptions\IncompletePayment;

    try {
        $subscription = $user->newSubscription('default', 'price_monthly')
                                ->create($paymentMethod);
    } catch (IncompletePayment $exception) {
        return redirect()->route(
            'cashier.payment',
            [$exception->payment->id, 'redirect' => route('home')]
        );
    }

На странице подтверждения оплаты клиенту будет предложено повторно ввести данные своей кредитной карты и выполнить любые дополнительные действия, требуемые Stripe, такие как подтверждение "3D Secure". После подтверждения оплаты пользователь будет перенаправлен на URL, указанный указанным выше параметром `redirect`. При перенаправлении к URL-адресу будут добавлены строковые переменные запроса `message` (строка) и `success` (целое число). Страница оплаты в настоящее время поддерживает следующие типы способов оплаты:

<!-- <div class="content-list" markdown="1"> -->

- Credit Cards
- Alipay
- Bancontact
- BECS Direct Debit
- EPS
- Giropay
- iDEAL
- SEPA Direct Debit

<!-- </div> -->

В качестве альтернативы вы могли бы позволить Stripe обработать подтверждение платежа за вас. В этом случае вместо перенаправления на страницу подтверждения оплаты вы можете [настроить автоматические электронные письма Stripe для выставления счетов](https://dashboard.stripe.com/account/billing/automatic) на вашей панели управления Stripe. Однако, если будет обнаружено исключение `IncompletePayment`, вам все равно следует сообщить пользователю, что он получит электронное письмо с дальнейшими инструкциями по подтверждению платежа.

Исключения для оплаты могут быть созданы для следующих методов: `charge`, `invoiceFor` и `invoice` в моделях, использующих трейт `Billable`. При взаимодействии с подписками метод `create` в `SubscriptionBuilder`, а также методы `incrementAndInvoice` и `swapAndInvoice` в моделях `Subscription` и `SubscriptionItem` могут вызывать исключения неполной оплаты.

Определение того, имеет ли существующая подписка неполную оплату, может быть выполнено с помощью метода `hasIncompletePayment` в оплачиваемой модели или экземпляре подписки:

    if ($user->hasIncompletePayment('default')) {
        //
    }

    if ($user->subscription('default')->hasIncompletePayment()) {
        //
    }

Вы можете получить конкретный статус неполного платежа, проверив свойство `payment` в экземпляре исключения:

    use Laravel\Cashier\Exceptions\IncompletePayment;

    try {
        $user->charge(1000, 'pm_card_threeDSecure2Required');
    } catch (IncompletePayment $exception) {
        // Get the payment intent status...
        $exception->payment->status;

        // Check specific conditions...
        if ($exception->payment->requiresPaymentMethod()) {
            // ...
        } elseif ($exception->payment->requiresConfirmation()) {
            // ...
        }
    }

<a name="strong-customer-authentication"></a>
## Аутентификация клиентов(SCA)

Если ваш бизнес или один из ваших клиентов базируется в Европе, вам необходимо будет соблюдать правила строгой аутентификации клиентов ЕС (SCA). Эти правила были введены Европейским союзом в сентябре 2019 года для предотвращения мошенничества с платежами. К счастью, Stripe и Cashier подготовлены для создания приложений, совместимых с SCA.

> {note} Прежде чем приступить к работе, ознакомьтесь с [руководством Stripe по PSD2 и SCA](https://stripe.com/guides/strong-customer-authentication), а также их [документация по новым SCA API](https://stripe.com/docs/strong-customer-authentication).

<a name="payments-requiring-additional-confirmation"></a>
### Платежи, требующие дополнительного подтверждения

Правила SCA часто требуют дополнительной верификации для подтверждения и обработки платежа. Когда это произойдет, Cashier выдаст исключение `Laravel\Cashier\Exceptions\IncompletePayment`, которое информирует вас о необходимости дополнительной проверки. Более подробную информацию о том, как обрабатывать эти исключения, можно найти в документации по [обработке неудачных платежей](#handling-failed-payments).

Экраны подтверждения оплаты, предоставляемые Stripe или кассиром, могут быть адаптированы к платежному потоку конкретного банка или эмитента карты и могут включать дополнительное подтверждение карты, временную небольшую плату, отдельную аутентификацию устройства или другие формы проверки.

<a name="incomplete-and-past-due-state"></a>
#### Неполное и просроченное состояние

Когда платеж нуждается в дополнительном подтверждении, подписка останется в состоянии `incomplete` или `past_due`, как указано в столбце базы данных `stripe_status`. Cashier автоматически активирует подписку клиента, как только будет завершено подтверждение оплаты и Stripe уведомит вашу заявку через веб-хук о ее завершении.

Для получения дополнительной информации о состояниях `incomplete` и `past_due`, пожалуйста, обратитесь к [нашей дополнительной документации по этим состояниям](#incomplete-and-past-due-status).

<a name="off-session-payment-notifications"></a>
### Уведомления об оплате вне сессии

Поскольку правила SCA требуют, чтобы клиенты время от времени проверяли свои платежные реквизиты, даже когда их подписка активна, кассир может отправить клиенту уведомление, когда требуется подтверждение платежа вне сеанса. Например, это может произойти при обновлении подписки. Уведомление Cashier об оплате можно включить, установив переменную среды `CASHIER_PAYMENT_NOTIFICATION` в класс уведомлений. По умолчанию это уведомление отключено. Конечно, Cashier включает класс уведомлений, который вы можете использовать для этой цели, но при желании вы можете предоставить свой собственный класс уведомлений:

    CASHIER_PAYMENT_NOTIFICATION=Laravel\Cashier\Notifications\ConfirmPayment

Чтобы убедиться, что уведомления о подтверждении оплаты вне сеанса доставляются, убедитесь, что [веб-хуки Stripe настроены](#handling-stripe-webhooks) для вашего приложения и веб-хук `invoice.payment_action_required` включен на вашей панели мониторинга Stripe. Кроме того, ваша `оплачиваемая` модель также должна использовать трейт Laravel "Illuminate\Notifications\Notifiable".

> {note} Уведомления будут отправляться даже тогда, когда клиенты вручную производят платеж, требующий дополнительного подтверждения. К сожалению, у Stripe нет возможности узнать, что платеж был произведен вручную или "вне сеанса". Но клиент просто увидит сообщение "Платеж выполнен успешно", если он зайдет на страницу оплаты после того, как уже подтвердил свой платеж. Клиенту не будет разрешено случайно подтвердить один и тот же платеж дважды и понести случайную повторную оплату.

<a name="stripe-sdk"></a>
## Stripe SDK

Многие объекты Cashier's являются оболочками вокруг объектов Stripe SDK. Если вы хотите взаимодействовать с объектами Stripe напрямую, вы можете удобно извлечь их, используя метод `asStripe`:

    $stripeSubscription = $subscription->asStripeSubscription();

    $stripeSubscription->application_fee_percent = 5;

    $stripeSubscription->save();

Вы также можете использовать метод `updateStripeSubscription` для непосредственного обновления подписки Stripe:

    $subscription->updateStripeSubscription(['application_fee_percent' => 5]);

Вы можете вызвать метод `stripe` в классе `Cashier`, если хотите напрямую использовать клиент `Stripe\StripeClient`. Например, вы могли бы использовать этот метод для доступа к экземпляру `StripeClient` и получения списка цен из вашей учетной записи Stripe:

    use Laravel\Cashier\Cashier;

    $prices = Cashier::stripe()->prices->all();

<a name="testing"></a>
## Тестирование

При тестировании приложения, использующего Cashier, вы можете имитировать фактические HTTP-запросы к Stripe API; однако для этого вам потребуется частично повторно реализовать собственное поведение Cashier. Поэтому мы рекомендуем разрешить вашим тестам использовать фактический Stripe API. Хотя это происходит медленнее, это обеспечивает большую уверенность в том, что ваше приложение работает должным образом, и любые медленные тесты могут быть размещены в их собственной группе тестирования PHPUnit.

При тестировании помните, что у самого Cashier уже есть отличный набор тестов, поэтому вам следует сосредоточиться только на тестировании подписки и потока платежей вашего собственного приложения, а не на каждом базовом поведении Cashier.

Чтобы начать, добавьте **тестовую** версию вашего Stripe секрета в свой файл `phpunit.xml`:

    <env name="STRIPE_SECRET" value="sk_test_<your-key>"/>

Теперь, всякий раз, когда вы взаимодействуете с Cashier во время тестирования, он будет отправлять фактические запросы API в вашу среду тестирования Stripe. Для удобства вам следует предварительно заполнить свой тестовый аккаунт Stripe подписками / ценами, которые вы можете использовать во время тестирования.

> {tip} Для тестирования различных сценариев выставления счетов, таких как отказы в оплате по кредитной карте, вы можете использовать широкий спектр [тестовых номеров карт и токенов](https://stripe.com/docs/testing), предоставленный Stripe.
