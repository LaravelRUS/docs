git 22951bd4bcc7a559cb3d991095ad8c7a087ca010

---

# Laravel Cashier

- [Введение](#introduction)
- [Настройка](#configuration)
    - [Stripe](#stripe-configuration)
    - [Braintree](#braintree-configuration)
    - [Настройка валюты](#currency-configuration)
- [Подписки](#subscriptions)
    - [Создание подписок](#creating-subscriptions)
    - [Проверка статуса подписки](#checking-subscription-status)
    - [Смена планов](#changing-plans)
    - [Количество подписки](#subscription-quantity)
    - [Налог на подписку](#subscription-taxes)
    - [Отмена подписки](#cancelling-subscriptions)
    - [Возобновление подписки](#resuming-subscriptions)
    - [Обновление банковских карт](#updating-credit-cards)
- [Пробные подписки](#subscription-trials)
    - [С запросом банковской карты](#with-credit-card-up-front)
    - [Без запроса банковской карты](#without-credit-card-up-front)
- [Обработка веб-хуков Stripe](#handling-stripe-webhooks)
    - [Определение хэндлеров веб-хук событий](#defining-webhook-event-handlers)
    - [Неудавшиеся подписки](#handling-failed-subscriptions)
- [Обработка веб-хуков Braintree](#handling-braintree-webhooks)
    - [Определение хэндлеров веб-хук событий](#defining-braintree-webhook-event-handlers)
    - [Неудавшиеся подписки](#handling-braintree-failed-subscriptions)
- [Одиночные платежи](#single-charges)
- [Счета](#invoices)
    - [Генерация счетов в PDF](#generating-invoice-pdfs)

<a name="introduction"></a>
## Введение

> {tip} На данный момент [Stripe](https://stripe.com) и [Braintree](https://www.braintreepayments.com) не работают в России.

Laravel Cashier обеспечивает выразительный и гибкий интерфейс для сервисов биллинговых подписок [Stripe](https://stripe.com) и [Braintree](https://www.braintreepayments.com). Он сам создаст практически весь шаблонный код биллинговых подписок, который вы боитесь писать. В дополнение к основному управлению подписками Cashier может работать с купонами, заменой подписок, "количеством" подписок, отменой льготного периода, и даже генерировать PDF-файлы счетов.

> {note} Если вы используете только "одноразовые" списания и не предлагаете подписки, вам не следует использовать Cashier. Вместо этого пользуйтесь SDK Stripe и Braintree напрямую.

<a name="configuration"></a>
## Настройка

<a name="stripe-configuration"></a>
### Stripe

#### Composer

Сначала добавьте пакет Cashier для Stripe к своим зависимостям:

    composer require "laravel/cashier":"~7.0"

#### Сервис-провайдер

Затем зарегистрируйте [сервис-провайдер](/docs/{{version}}/providers) `Laravel\Cashier\CashierServiceProvider` в своем конфиге `config/app.php`.

#### Миграции базы данных

Перед тем, как начать использовать Cashier, надо [подготовить БД](/docs/{{version}}/migrations). Надо добавить несколько столбцов в таблицу `users` и создать новую таблицу `subscriptions` для хранения всех подписок пользователей:

    Schema::table('users', function ($table) {
        $table->string('stripe_id')->nullable();
        $table->string('card_brand')->nullable();
        $table->string('card_last_four')->nullable();
        $table->timestamp('trial_ends_at')->nullable();
    });

    Schema::create('subscriptions', function ($table) {
        $table->increments('id');
        $table->integer('user_id');
        $table->string('name');
        $table->string('stripe_id');
        $table->string('stripe_plan');
        $table->integer('quantity');
        $table->timestamp('trial_ends_at')->nullable();
        $table->timestamp('ends_at')->nullable();
        $table->timestamps();
    });

После создания миграции просто выполните Artisan-команду `migrate`.

#### Модель Billable

Затем добавьте трейт `Billable` в определение вашей модели. Этот трейт предоставляет различные методы, позволяющие вам выполнять стандартные биллинг-задачи, такие как создание подписок, применение купонов, а также обновление информации о банковской карте:

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### API ключи

Далее надо настроить ключ Stripe в файле настроек `services.php`. Ключи Stripe API можно получить из панели управления Stripe:

    'stripe' => [
        'model'  => App\User::class,
        'key' => env('STRIPE_KEY'),
        'secret' => env('STRIPE_SECRET'),
    ],

<a name="braintree-configuration"></a>
### Braintree

#### Информация о Braintree

Для многих операций реализация функций Cashier в Stripe и Braintree одинакова. Оба сервиса предоставляют возможность оплаты подписок банковскими картами, но Braintree также поддерживает оплату через PayPal. Однако, в Braintree нет некоторых возможностей, имеющихся в Stripe. При выборе между ними учитывайте следующее:

<div class="content-list" markdown="1">
- Braintree поддерживает PayPal, а Stripe - нет.
- Braintree не поддерживает методы `increment` и `decrement` для подписок. Это ограничение Braintree, не Cashier.
- Braintree не поддерживает скидки в процентах. Это ограничение Braintree, не Cashier.
</div>

#### Composer

Сначала добавьте пакет Cashier для Braintree к своим зависимостям:

    composer require "laravel/cashier-braintree":"~2.0"

#### Сервис-провайдер

Затем зарегистрируйте [сервис-провайдер](/docs/{{version}}/providers) `Laravel\Cashier\CashierServiceProvider` в конфиге `config/app.php`:

    Laravel\Cashier\CashierServiceProvider::class

#### Купон на скидку

Перед использованием Cashier с Braintree вам надо определить скидку `plan-credit` в панели настроек Braintree. Эта скидка будет использоваться для пропорционального пересчёта подписок, которые переходят с годовой на ежемесячную оплату, или наоборот с ежемесячной на годовую.

Настраиваемый в панели настроек Braintree размер скидки может быть любым, на ваше усмотрение, а Cashier будет просто изменять размер по умолчанию на заданный при каждом применении купона. Этот купон нужен, т.к. Braintree нативно не поддерживает пропорциональное распределение подписок.

#### DМиграции базы данных

Перед использование Cashier нам потребуется [подготовить базу данных](/docs/{{version}}/migrations). Надо добавить несколько столбцов в вашу таблицу `users` и создать новую таблицу `subscriptions` для хранения всех подписок пользователей:

    Schema::table('users', function ($table) {
        $table->string('braintree_id')->nullable();
        $table->string('paypal_email')->nullable();
        $table->string('card_brand')->nullable();
        $table->string('card_last_four')->nullable();
        $table->timestamp('trial_ends_at')->nullable();
    });

    Schema::create('subscriptions', function ($table) {
        $table->increments('id');
        $table->integer('user_id');
        $table->string('name');
        $table->string('braintree_id');
        $table->string('braintree_plan');
        $table->integer('quantity');
        $table->timestamp('trial_ends_at')->nullable();
        $table->timestamp('ends_at')->nullable();
        $table->timestamps();
    });

После создания миграций просто выполните Artisan-команду `migrate`.

#### Модель Billable

Далее добавьте трейт `Billable` в определение вашей модели:

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### API ключи

Далее надо настроить следующие параметры в файле настроек `services.php`:

    'braintree' => [
        'model'  => App\User::class,
        'environment' => env('BRAINTREE_ENV'),
        'merchant_id' => env('BRAINTREE_MERCHANT_ID'),
        'public_key' => env('BRAINTREE_PUBLIC_KEY'),
        'private_key' => env('BRAINTREE_PRIVATE_KEY'),
    ],

Затем надо добавить следующие вызовы Braintree SDK в метод `boot` вашего сервис-провайдера `AppServiceProvider`:

    \Braintree_Configuration::environment(config('services.braintree.environment'));
    \Braintree_Configuration::merchantId(config('services.braintree.merchant_id'));
    \Braintree_Configuration::publicKey(config('services.braintree.public_key'));
    \Braintree_Configuration::privateKey(config('services.braintree.private_key'));

<a name="currency-configuration"></a>
### Настрока валюты

Валюта Cashier по умолчанию - доллары США (USD). Можно изменить валюту по умолчанию, вызвав метод `Cashier::useCurrency` из метода `boot` одного из ваших сервис-провайдеров. Метод `useCurrency` принимает два строчных параметра: валюту и символ валюты:

    use Laravel\Cashier\Cashier;

    Cashier::useCurrency('eur', '€');

<a name="subscriptions"></a>
## Подписки

<a name="creating-subscriptions"></a>
### Создание подписок

Для создания подписки сначала получите экземпляр оплачиваемой модели, который обычно является экземпляром `App\User`. Когда вы получили модель, вы можете использовать метод `newSubscription` для управления подписками модели:

    $user = User::find(1);

    $user->newSubscription('main', 'premium')->create($stripeToken);

Первый аргумент метода `newSubscription` — название подписки. Если в вашем приложении используется только одна подписка, то вы можете назвать её `main` или `primary`.  Второй аргумент — конкретный план Stripe/Braintree, на который подписывается пользователь. Это значение должно соответствовать идентификатору плана в Stripe или Braintree.

Метод `create` автоматически создаст подписку, а также внесёт в вашу базу данных ID заказчика и другую связанную информацию по оплате.

#### Дополнительная информация о пользователе

Если вы хотите указать дополнительную информацию о пользователе, передайте её вторым аргументом методу `create`:

    $user->newSubscription('main', 'monthly')->create($stripeToken, [
        'email' => $email,
    ]);

Подробнее о дополнительных полях, поддерживаемых Stripe и Braintree, читайте в [документации по созданию клиента](https://stripe.com/docs/api#create_customer) Stripe или в соответствующей [документации Braintree](https://developers.braintreepayments.com/reference/request/customer/create/php).

#### Купоны

Если надо применить купон при создании подписки, можно использовать метод `withCoupon`:

    $user->newSubscription('main', 'monthly')
         ->withCoupon('code')
         ->create($stripeToken);

<a name="checking-subscription-status"></a>
### Проверка статуса подписки

Когда пользователь подписан на ваше приложение, вы можете легко проверить статус его подписки при помощи различных удобных методов. Во-первых, метод `subscribed` возвращает `true`, если подписка пользователя активна, даже если в данный момент она на пробном периоде:

    if ($user->subscribed('main')) {
        //
    }

Метод `subscribed` также создаёт отличный вариант для [посредника роута](/docs/{{version}}/middleware), позволяя вам фильтровать доступ к роутам и контроллерам на основе статусов подписок:

    public function handle($request, Closure $next)
    {
        if ($request->user() && ! $request->user()->subscribed('main')) {
            // Этот пользователь не оплатил подписку...
            return redirect('billing');
        }

        return $next($request);
    }

Вы также можете определить, идёт ли до сих пор пробный период у пользователя, с помощью метода `onTrial`. Этот метод полезен для предупреждения пользователя о том, что он на пробном периоде:

    if ($user->subscription('main')->onTrial()) {
        //
    }

Метод `subscribedToPlan` используется для определения, подписан ли пользователь на данный тариф, на основе его Stripe/Braintree ID. В этом примере мы определим, подписана ли подписка `main` пользователя на план `monthly`:

    if ($user->subscribedToPlan('monthly', 'main')) {
        //
    }

#### Статус отменённой подписки

Чтобы определить, был ли пользователь ранее активным подписчиком, но позже отменил подписку, используйте метод `cancelled`:

    if ($user->subscription('main')->cancelled()) {
        //
    }

Вы можете также определить, отменил ли пользователь подписку, но находится все ещё на «льготном периоде», пока подписка полностью не истекла. Например, если пользователь отменяет подписку 5 марта, которая по плану закончится 10 марта, пользователь будет на «льготном периоде» до 10 марта. Обратите внимание на то, что метод `subscribed` в это время всё ещё возвращает `true`:

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }

<a name="changing-plans"></a>
### Смена планов

Когда пользователь подписан на ваше приложение, он может захотеть сменить свой тарифный план. Чтобы переключить пользователя на новую подписку, используйте метод `swap`:

    $user = App\User::find(1);

    $user->subscription('main')->swap('provider-plan-id');

Если пользователь был на пробном периоде, то пробный период продолжится. Кроме того, если у подписки есть "количество", то оно тоже применится.

Если вы хотите сменить план, но пропустить пробный период нового плана, используйте метод `skipTrial`:

    $user->subscription('main')
            ->skipTrial()
            ->swap('provider-plan-id');

<a name="subscription-quantity"></a>
### Количество подписки

> {note} Количество подписки поддерживается только версией Cashier для Stripe. В Braintree нет эквивалента "количества" Stripe.

Иногда подписки зависят от "количества". Например, ваше приложение стоит $10 в месяц с **одного пользователя** нв учётной записи. Чтобы легко увеличить или уменьшить количество вашей подписки, используйте методы `incrementQuantity` и `decrementQuantity`:

    $user = User::find(1);

    $user->subscription('main')->incrementQuantity();

    // Добавить 5 к текущему количеству подписок...
    $user->subscription('main')->incrementQuantity(5);

    $user->subscription('main')->decrementQuantity();

    // Вычесть 5 из текущего количества подписок...
    $user->subscription('main')->decrementQuantity(5);

Или вы можете задать конкретное количество с помощью метода `updateQuantity`:

    $user->subscription('main')->updateQuantity(10);

Более подробная информация о количествах подписок есть в [документации Stripe](https://stripe.com/docs/subscriptions/quantities).

<a name="subscription-taxes"></a>
### Налог на подписку

Чтобы указать процент налога, который пользователь платит за подписку, реализуйте метод `taxPercentage` в своей модели, и верните числовое значение от 0 до 100 с не более, чем двумя знаками после запятой.

    public function taxPercentage() {
        return 20;
    }

Метод `taxPercentage` позволяет вам использовать разные налоговые ставки по-модельно, что будет полезно при наличии пользователей из разных стран.

> {note} Метод `taxPercentage` применяется только к налогу на подписку. Если вы используете только "одноразовые" списания и не предлагаете подписки, вам нужно будет указывать налоговую ставку вручную.

<a name="cancelling-subscriptions"></a>
### Отмена подписки

Для отмены подписки просто вызовите метод `cancel` на подписке пользователя:

    $user->subscription('main')->cancel();

При отмене подписки Cashier автоматически задаст столбец `ends_at` в вашей базе данных. Этот столбец используется, чтобы знать, когда метод `subscribed` должен начать возвращать `false`. Например, если клиент отменит подписку 1 марта, но срок подписки по плану до 5 марта, то метод `subscribed` будет продолжать возвращать `true` до 5 марта.

Вы можете определить то, что пользователь отменил подписку, но находится на "льготном периоде", при помощи метода `onGracePeriod`:

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }

Для незамедлительной отмены подписки нужно вызывать метод `cancelNow` на подписке пользователя:

    $user->subscription('main')->cancelNow();

<a name="resuming-subscriptions"></a>
### Возобновление подписки

Если подписка была отменена пользователем, и вам надо её возобновить, используйте метод `resume`. Пользователь **должен** быть по-прежнему на льготном периоде, чтобы возобновить подписку:

    $user->subscription('main')->resume();

Если пользователь отменит подписку и затем возобновит её до того, как она полностью истекла, тогда не произойдет моментального расчёта оплаты. Его подписка будет просто повторно активирована, и расчёт оплаты будет происходить по изначальному циклу биллинга.

<a name="updating-credit-cards"></a>
### Обновление банковской карты

Метод `updateCard` можно использовать для обновления информации о банковской карте пользователя. Этот метод принимает токен Stripe и будет назначать новую банковскую карту в качестве источника оплаты по умолчанию:

    $user->updateCard($stripeToken);

<a name="subscription-trials"></a>
## Пробные подписки

<a name="with-credit-card-up-front"></a>
### С запросом банковской карты

Если вы хотите предлагать клиентам пробные периоды, но при этом сразу собирать данные о способе оплаты, используйте метод `trialDays` при создании подписок:

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')
                ->trialDays(10)
                ->create($stripeToken);

Этот метод задаст дату окончания пробного периода в записи БД о подписке, а также проинструктирует Stripe/Braintree о том, что не нужно начинать считать оплату для клиента до окончания этого периода.

> {note} Если подписка клиента не будет отменена до окончания пробного периода, ему будет выставлен счёт, как только истечёт этот период, поэтому вы должны уведомлять своих пользователей о дате окончания их пробного периода.

Вы можете определить, идёт ли до сих пор пробный период у пользователя, с помощью метода `onTrial` экземпляра пользователя или с помощью метода `onTrial` экземпляра подписки. Эти два примера выполняют одинаковую задачу:

    if ($user->onTrial('main')) {
        //
    }

    if ($user->subscription('main')->onTrial()) {
        //
    }

<a name="without-credit-card-up-front"></a>
### Без запроса банковской карты

Если вы хотите предлагать клиентам пробные периоды, не собирая данные о способе оплаты, вы можете просто задать значение столбца `trial_ends_at` в записи пользователя равное дате окончания пробного периода. Например, это обычно делается во время регистрации пользователя:

    $user = User::create([
        // Заполнение других свойств пользователя...
        'trial_ends_at' => Carbon::now()->addDays(10),
    ]);

> {note}  Не забудьте добавить [мутатор дат](/docs/{{version}}/eloquent-mutators#date-mutators) для `trial_ends_at` к определению своей модели.

Cashier относится к пробным периодам такого типа, как к "общим пробным периодам", поскольку они не прикреплены к какой-либо из существующих подписок. Метод `onTrial` на экземпляре `User` вернёт `true`, если текущая дата не превышает значение `trial_ends_at`:

    if ($user->onTrial()) {
        // Пользователь на пробном периоде...
    }

Если вы хотите узнать, что пользователь именно на "общем" пробном периоде и ещё не создал настоящую подписку, используйте метод `onGenericTrial`:

    if ($user->onGenericTrial()) {
        // Пользователь на "общем" пробном периоде...
    }

Когда вы готовы создать для пользователя настоящую подписку, используйте метод `newSubscription` как обычно:

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')->create($stripeToken);

<a name="handling-stripe-webhooks"></a>
## Обработка веб-хуков Stripe

И Stripe, и Braintree могут уведомлять ваше приложение о разных событиях через веб-хуки. Для обработки веб-хуков Stripe: задайте роут, который указывает на контроллер веб-хуков Cashier. Этот контроллер будет обрабатывать все входящие запросы веб-хуков и посылать их на соответствующий метод контроллера:

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

> {note} Как только вы зарегистрировали свой роут, не забудьте настроить URL веб-хука в настройках панели управления Stripe.

По умолчанию этот контроллер будет автоматически обрабатывать отмены подписок, у которых слишком много неудачных списаний (как задано в ваших настройках Stripe); однако, как нам скоро станет известно, вы можете наследовать этот контроллер, чтобы он обрабатывал любое веб-хук событие.

#### Веб-хуки и CSRF защита

Поскольку веб-хуки Stripe должны идти в обход [CSRF-защиты](/docs/{{version}}/csrf) Laravel, не забудьте включить URI в список исключений вашего посредника `VerifyCsrfToken` или разместить роут вне группы посредников `web`:

    protected $except = [
        'stripe/*',
    ];

<a name="defining-webhook-event-handlers"></a>
### Определение хэндлеров веб-хук событий

Cashier автоматически обрабатывает отмену подписки в результате неудачных списаний, но если есть еще какие-то дополнительные веб-хук события Stripe, которые вы бы хотели обрабатывать, просто наследуйте контроллер Webhook. Названия ваших методов должны соответствовать конвенции ожиданий Cashier, в особенности, у методов должен быть префикс `handle` и название в "camel case" у веб-хуков Stripe, которые вы хотите обрабатывать. Например, если вы хотите обрабатывать веб-хук `invoice.payment_succeeded`, вы должна добавить метод `handleInvoicePaymentSucceeded` к контроллеру:

    <?php

    namespace App\Http\Controllers;

    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * Обработка веб-хука Stripe.
         *
         * @param  array  $payload
         * @return Response
         */
        public function handleInvoicePaymentSucceeded($payload)
        {
            // Handle The Event
        }
    }

<a name="handling-failed-subscriptions"></a>
### Неудавшиеся подписки

Что если срок действия банковской карты клиента истёк? Никаких проблем — Cashier включает в себя контроллер Webhook, который может легко отменить подписку клиента. Просто укажите путь к контроллеру:

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

Вот и всё! Неудавшиеся платежи будут перехвачены и обработаны контроллером. Контроллер отменит подписку клиента, если Stripe определит, что подписка не удалась (обычно после трёх неудавшихся платежей).

<a name="handling-braintree-webhooks"></a>
## Обработка веб-хуков Braintree

И Stripe, и Braintree могут уведомлять ваше приложение о разных событиях через веб-хуки. Для обработки веб-хуков Stripe: задайте роут, который указывает на контроллер веб-хуков Cashier. Этот контроллер будет обрабатывать все входящие запросы веб-хуков и посылать их на соответствующий метод контроллера:

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

> {note} Как только вы зарегистрировали свой роут, не забудьте настроить URL веб-хука в настройках панели управления Braintree.

По умолчанию этот контроллер будет автоматически обрабатывать отмены подписок, у которых слишком много неудачных списаний (как задано в ваших настройках Stripe); однако, как нам скоро станет известно, вы можете наследовать этот контроллер, чтобы он обрабатывал любое веб-хук событие

#### Веб-хуки и CSRF защита

Поскольку веб-хуки Braintree должны идти в обход [CSRF-защиты](/docs/{{version}}/csrf) Laravel, не забудьте включить URI в список исключений вашего посредника `VerifyCsrfToken` или разместить роут вне группы посредников `web`:

    protected $except = [
        'braintree/*',
    ];

<a name="defining-braintree-webhook-event-handlers"></a>
### Определение хэндлеров веб-хук событий

Cashier автоматически обрабатывает отмену подписки в результате неудачных списаний, но если есть еще какие-то дополнительные веб-хук события Braintree, которые вы бы хотели обрабатывать, просто наследуйте контроллер Webhook. Названия ваших методов должны соответствовать конвенции ожиданий Cashier, в особенности, у методов должен быть префикс `handle` и название в "camel case" у веб-хуков Braintree, которые вы хотите обрабатывать. Например, если вы хотите обрабатывать веб-хук `dispute_opened`, вы должна добавить метод `handleDisputeOpened` к контроллеру:

    <?php

    namespace App\Http\Controllers;

    use Braintree\WebhookNotification;
    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * Обработка веб-хука Braintree.
         *
         * @param  WebhookNotification  $webhook
         * @return Response
         */
        public function handleDisputeOpened(WebhookNotification $notification)
        {
            // Handle The Event
        }
    }

<a name="handling-braintree-failed-subscriptions"></a>
### Неудавшиеся подписки

Что если срок действия банковской карты клиента истёк? Никаких проблем — Cashier включает в себя контроллер Webhook, который может легко отменить подписку клиента. Просто укажите путь к контроллеру:

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

Вот и всё! Неудавшиеся платежи будут перехвачены и обработаны контроллером. Контроллер отменит подписку клиента, если Braintree определит, что подписка не удалась (обычно после трёх неудавшихся платежей). Не забудьте: вам надо настроить URI веб-хука на панели настроек вашего Braintree.

<a name="single-charges"></a>
## Одиночные платежи

### Простой платёж

> {note} При использовании Stripe метод `charge` принимает сумму, которую необходимо оплатить, с **наименьшим знаменателем используемой в вашем приложении валюты**. А при использовании Braintree вы должны передавать в метод `charge` полную сумму в долларах:

Если вы хотите сделать "одноразовый" платёж вместо использования банковской карты подписанного пользователя, используйте метод `charge` для экземпляра оплачиваемой модели.

    // Stripe принимает сумму в центах...
    $user->charge(100);

    // Braintree принимает сумму в долларах...
    $user->charge(1);

Метод `charge` принимает вторым аргументом массив, позволяя вам передавать любые необходимые параметры для создания основного Stripe / Braintree-платежа. См. документацию по Stripe или Braintree касательно опций, доступных вам при создании платежей:

    $user->charge(100, [
        'custom_option' => $value,
    ]);

Метод `charge` выбросит исключение при ошибке платёжа. Если платёж пройдёт успешно, метод вернёт полный ответ Stripe / Braintree:

    try {
        $response = $user->charge(100);
    } catch (Exception $e) {
        //
    }

### Платёж со счётом

Иногда бывает необходимо сделать одноразовый платёж и сгенерировать счёт-фактуру для него, чтобы вы могли предоставить клиенту PDF-квитанцию. Именно для этого служит метод `invoiceFor`. Например, давайте выставим клиенту "единоразовый" счёт $5.00:

    // Stripe принимает сумму в центах...
    $user->invoiceFor('One Time Fee', 500);

    // Braintree принимает сумму в долларах...
    $user->invoiceFor('One Time Fee', 5);

Счёт будет немедленно оплачен банковской картой пользователя. Метод `invoiceFor` также принимает третьим аргументом массив, позволяя вам передавать любые параметры для создания платежа Stripe / Braintree:

    $user->invoiceFor('One Time Fee', 500, [
        'custom-option' => $value,
    ]);

> {note} Метод `invoiceFor` создаст счёт Stripe, который будет повторять проваленные попытки оплаты. Если вы не хотите повторять проваленные платежи, вам необходимо закрывать их с помощью Stripe API после первого неудачного платежа.

<a name="invoices"></a>
## Счета

Вы можете легко получить массив счетов пользователя, используя метод `invoices`:

    $invoices = $user->invoices();

    // Включить в результаты ожидающие счета...
    $invoices = $user->invoicesIncludingPending();

При просмотре счетов клиента вы можете использовать эти вспомогательные методы, чтобы вывести на экран соответствующую информацию о счёте. Например, вам может понадобиться просмотреть каждый счёт в таблице, позволив пользователю легко скачать любой из них:

    <table>
        @foreach ($invoices as $invoice)
            <tr>
                <td>{{ $invoice->date()->toFormattedDateString() }}</td>
                <td>{{ $invoice->total() }}</td>
                <td><a href="/user/invoice/{{ $invoice->id }}">Download</a></td>
            </tr>
        @endforeach
    </table>

<a name="generating-invoice-pdfs"></a>
### Генерация счетов в PDF

Используйте метод `downloadInvoice` в роуте или контроллере, чтобы cгенерировать PDF-файл счёта. Этот метод автоматически сгенерирует нужный HTTP-отклик чтобы отправить загрузку в браузер:

    use Illuminate\Http\Request;

    Route::get('user/invoice/{invoice}', function (Request $request, $invoiceId) {
        return $request->user()->downloadInvoice($invoiceId, [
            'vendor'  => 'Your Company',
            'product' => 'Your Product',
        ]);
    });
