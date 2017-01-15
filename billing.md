git 24551706b299d0ba1953a50d2703c73c60022399

---

# Laravel Cashier

- [Введение](#introduction)
- [Настройка Stripe](#stripe-configuration)
- [Настройка Braintree](#braintree-configuration)
- [Подписки](#subscriptions)
	- [Создание подписок](#creating-subscriptions)
	- [Проверка статуса подписки](#checking-subscription-status)
	- [Смена планов подписок](#changing-plans)
	- [Цена подписки](#subscription-quantity)
	- [Налог на подписку](#subscription-taxes)
	- [Отмена подписки](#cancelling-subscriptions)
	- [Восстановление подписки](#resuming-subscriptions)
- [Триал период](#subscription-trials)
    - [С вводом данных карты](#with-credit-card-up-front)
    - [Без ввода данных карты](#without-credit-card-up-front)
- [Обработка Stripe Webhooks](#handling-stripe-webhooks)
	- [Ошибка при оплате подписки](#handling-failed-subscriptions)
	- [Другие Webhooks](#handling-other-webhooks)
- [Обработка Braintree Webhooks](#handling-braintree-webhooks)
    - [Ошибка при оплате подписки](#handling-braintree-failed-subscriptions)
    - [Другие Webhooks](#handling-braintree-other-webhooks)
- [Одиночные оплаты](#single-charges)
- [Инвойсы](#invoices)
	- [Создание PDF инвойса](#generating-invoice-pdfs)

<a name="introduction"></a>
## Введение

Laravel Cashier предоставляет простой и выразительный интерфейс для работы с платными подписками при помощи сервисов [Stripe](https://stripe.com) и [Braintree](https://braintreepayments.com). Он предоставляет готовые методы для работы, которые вы, возможно, писали раньше вручную. В дополнение к базовому управлению подписками, Cashier поддерживает купоны, смену подписок, цену подписок, отмену подписок на определённый период и даже создание PDF отчётов по инвойсам.

<a name="stripe-configuration"></a>
### Настройка Stripe

#### Composer

Для начала добавьте пакет Cashier в ваш `composer.json` и выполните команду `composer update`:

	"laravel/cashier": "~6.0"

#### Сервис-провайдер

Дальше зарегистрируйте [сервис-провайдер](/docs/{{version}}/providers) `Laravel\Cashier\CashierServiceProvider` в вашем файле конфигурации `app`.

#### Миграции

Перед использованием Cashier вам также необходимо подготовить базу данных. Требуется добавить несколько колонок в таблицу `users` и создать новую `subscriptions`, которая будет хранить информацию о подписках:

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

После того, как миграция создана, просто запустите Artisan команду `migrate`.

#### Настройка модели

Дальше добавьте трейт `Billable` в вашу модель пользователя:

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### Ключ Stripe

Далее, добавьте ваш ключ Stripe в конфигурационный файл `services.php`:

    'stripe' => [
        'model'  => App\User::class,
        'secret' => env('STRIPE_SECRET'),
    ],

<a name="braintree-configuration"></a>
##  Настройка Braintree

#### Предостережение о Braintree

Для большинства операций реализации функций Cashier для сервисов Stripe и Braintree одинаковы. Оба сервиса обеспечивают функционал платных подписок через пластиковые карты, но Braintree так же поддерживает платежи через PayPal. Однако, Braintree не поддерживает некоторые функции, которые есть в Stripe. Вы должны учитывать следующие пункты при выборе между Stripe и Braintree:

- Braintree поддерживает PayPal, а Stripe - нет.
- Braintree не поддерживает `increment` и `decrement` методы для подписок. Это ограничение Braintree, а не Cashier.
- Braintree не поддерживает скидки в процентах. Это ограничение Braintree, а не Cashier.

#### Composer

Для начала необходимо добавить пакет Braintree для Cashier в ваш файл `composer.json` и запустить команду `composer update`:

    "laravel/cashier-braintree": "~1.0"

#### Сервис-провайдер

Дальше зарегистрируйте [сервис-провайдер](/docs/{{version}}/providers) `Laravel\Cashier\CashierServiceProvider` в вашем файле конфигурации `app`.

#### Plan Credit Coupon

Before using Cashier with Braintree, you will need to define a `plan-credit` discount in your Braintree control panel. This discount will be used to properly prorate subscriptions that change from yearly to monthly billing, or from monthly to yearly billing. The discount amount configured in the Braintree control panel can be any value you wish, as Cashier will simply override the defined amount with our own custom amount each time we apply the coupon.

#### Миграции

Перед использованием Cashier вам также необходимо подготовить базу данных. Требуется добавить несколько колонок в таблицу `users` и создать новую `subscriptions`, которая будет хранить информацию о подписках:

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

После того, как миграция создана, просто запустите Artisan команду `migrate`.

#### Настройка модели

Далее добавьте трейт `Billable` в вашу модель пользователя:

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### Сервис-провайдер

Так же вам требуется настроить следующие опции в файле конфигурации `services.php`:

    'braintree' => [
        'model'  => App\User::class,
        'environment' => env('BRAINTREE_ENV'),
        'merchant_id' => env('BRAINTREE_MERCHANT_ID'),
        'public_key' => env('BRAINTREE_PUBLIC_KEY'),
        'private_key' => env('BRAINTREE_PRIVATE_KEY'),
    ],

Далее требуется добавить следующие вызовы Braintree SDK в метод `boot` сервис-провайдера `AppServiceProvider`:

    \Braintree_Configuration::environment(env('BRAINTREE_ENV'));
    \Braintree_Configuration::merchantId(env('BRAINTREE_MERCHANT_ID'));
    \Braintree_Configuration::publicKey(env('BRAINTREE_PUBLIC_KEY'));
    \Braintree_Configuration::privateKey(env('BRAINTREE_PRIVATE_KEY'));

<a name="subscriptions"></a>
## Подписки

<a name="creating-subscriptions"></a>
### Создание подписок

Для создания подписки сначала необходимо получить экземпляр вашей модели `App\User`.  После того, как вы его получили, можете использовать метод `newSubscription` для создания модели подписки:

	$user = User::find(1);

	$user->newSubscription('main', 'monthly')->create($creditCardToken);

Первый аргумент переданный в метод `newSubscription` должен быть именем подписки. Если ваше приложение предлагает только одну подписку, вы можете назвать её `main` или `primary`. Второй аргумент - специфичный для Stripe / Braintree план, на который идет подписка. Это значение должно совпадать с идентификатором плана на Stripe или Braintree. 

Метод `create` автоматически создаст подписку и обновит вашу базу данных платёжной информацией пользователя. Если у вас настроен пробный период подписок в Stripe, то эта информация также попадёт в базу данных.

#### Дополнительная информация о пользователе

Если вы хотите указать дополнительную информацию о пользователе при создании подписки, это можно сделать при помощи второго аргумента метода `create`:

	$user->newSubscription('main', 'monthly')->create($creditCardToken, [
        'email' => $email,
    ]);

Чтобы узнать больше о дополнительных полях обратитесь к документации [Stripe](https://stripe.com/docs/api#create_customer) или [Braintree](https://developers.braintreepayments.com/reference/request/customer/create/php).

#### Купоны

Если вы хотите применить купон при создании подписки, можете использовать метод `withCoupon`:

	$user->newSubscription('main', 'monthly')
	     ->withCoupon('code')
	     ->create($creditCardToken);

<a name="checking-subscription-status"></a>
### Проверка статуса подписки

После того, как пользователь оформил подписку на ваше приложение, вы легко можете проверить статус подписки различными способами. Метод `subscribed` возвратит `true`, если подписка пользователя активна или в пробном периоде:

	if ($user->subscribed('main')) {
		//
	}

Этот метод является хорошим кандидатом на [посредника](/docs/{{version}}/middleware) роутов, так как вы можете ограничивать доступ к конкретным роутам и контроллерам в зависимости от статуса подписки:

	public function handle($request, Closure $next)
    {
        if ($request->user() && ! $request->user()->subscribed('main')) {
            // This user is not a paying customer...
            return redirect('billing');
        }

        return $next($request);
    }

Если вы хотите определить, использует ли пользователь пробный период, воспользуйтесь методом `onTrial`. Он полезен для отображения предупреждений пользователю о том, что у него пробный период подписки:

	if ($user->subscription('main')->onTrial()) {
        //
    }

Метод `subscribedToPlan` позволяет определить, подписан ли пользователь на определённый план подписки Stripe / Braintree. В данном примере мы определяем, что `main` подписка пользователя использует план `monthly`:

    if ($user->subscribedToPlan('monthly', 'main')) {
        //
    }

#### Определение отменённых подписок

Для того, чтобы проверить, была ли у пользователя активная подписка, которую он отменил, вы можете воспользоваться методом `cancelled`:

    if ($user->subscription('main')->cancelled()) {
        //
    }

Вы также можете определить, отменил ли пользователь свою подписку до конца срока её окончания. Например, пользователь отменил подписку 5-го марта, но она активна до 10-го марта. Тем не менее, подписка всё ещё будет считаться активной при проверке методом `subscribed`. Определить отменённую подписку до окончания срока её действия можно при помощи метода `onGracePeriod`:

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }

<a name="changing-plans"></a>
### Смена планов подписок

После того, как пользователь оформил подписку в вашем приложении, он может захотеть сменить план подписки. Для этого используйте метод `swap`. Например, вы можете легко сменить подписку пользователя на `premium`:

    $user = App\User::find(1);
    
    $user->subscription('main')->swap('provider-plan-id');

If the user is on trial, the trial period will be maintained. Also, if a "quantity" exists for the subscription, that quantity will also be maintained. When swapping plans, you may also use the `prorate` method to indicate that the charges should be pro-rated. In addition, you may use the `swapAndInvoice` method to immediately invoice the user for the plan change:

	$user->subscription('main')->swap('provider-plan-id');

Если требуется сменить план и пропустить триал период, можно воспользоваться методом `skipTrial`:

    $user->subscription('main')
            ->skipTrial()
            ->swap('provider-plan-id');

<a name="subscription-quantity"></a>
### Количество подписки

> **Note:** Количество подписки поддерживается только Stripe для Cashier. Braintree не имеет аналога "количества".

Иногда подписка измеряется "количеством". Например, ваше приложение может стоить по 10$ в месяц **за каждого пользователя** аккаунта. Для удобного увеличения или уменьшения количества подписки используйте методы `increment` и `decrement`:

    $user = User::find(1);

    $user->subscription('main')->incrementQuantity();

    // Add five to the subscription's current quantity...
    $user->subscription('main')->incrementQuantity(5);

    $user->subscription('main')->decrementQuantity();

    // Subtract five to the subscription's current quantity...
    $user->subscription('main')->decrementQuantity(5);

В качестве альтернативы вы можете указать точное значение количество, используя метод `updateQuantity`:

    $user->subscription('main')->updateQuantity(10);
    
Для получения подробной информации о количестве подписки обратитесь к [документации Stripe](https://stripe.com/docs/guides/subscriptions#setting-quantities).

<a name="subscription-taxes"></a>
### Налоги на подписку

С Cashier очень просто отправлять значение налога `tax_percent` в Stripe / Braintree. Для этого реализуйте метод `taxPercentage` в вашей модели пользователя, который должен возвращать число от 0 до 100 с максимум двумя знаками после запятой:

	public function taxPercentage() {
        return 20;
    }

This enables you to apply a tax rate on a model-by-model basis, which may be helpful for a user base that spans multiple countries.

<a name="cancelling-subscriptions"></a>
### Отмена подписки

Для отмены подписки вызовите метод `cancel` у подписки пользователя:

	$user->subscription('main')->cancel();

Когда подписка отменяется, Cashier автоматически устанавливает поле `ends_at` в вашей базе данных. Это поле используется методом `subscribed` для возвращения `false`. Например, если пользователь отменил подписку 1-го марта, но она активна до 5-го марта, метод `subscribed` всё ещё будет возвращать `true` до 5-го марта.

Для проверки того, что пользователь отменил свою подписку и находится на "grace period" используйте метод `onGracePeriod`:

	if ($user->subscription('main')->onGracePeriod()) {
        //
    }

<a name="resuming-subscriptions"></a>
### Восстановление подписки

Если пользователь отменил свою подписку и хочет восстановить её, используйте метод `resume`. Пользователь **должен** всё еще быть на grace period что бы возобновить подписку:

	$user->subscription('main')->resume();

Если пользователь сначала отменил подписку, а потом возобновил её, до окончания grace period, оплата не будет снята сразу. Вместо этого, его подписка будет просто реактивирована и оплата будет производиться по графику, который применялся до отмены подписки.

<a name="subscription-trials"></a>
## Триал период

<a name="with-credit-card-up-front"></a>
### С вводом данных карты

Если вы хотите предоставить пользователям триал период после ввода данных пластиковой карты, вы можете использовать метод `trialDays` при создании подписки:

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')
                ->trialDays(10)
                ->create($creditCardToken);

Этот метод создает подписку с триал периодом, который записывается в базу и отправляется в Stripe / Braintree, информируя их о дате, до которой не должна сниматься оплата.

> **Примечание:** С карты пользователя будут сняты деньги, если он не отменит подписку до окончания триала, поэтому, вы должны уведомлять его о дате окончания бесплатного периода.

Вы можете определить, что пользователь находится на триал периоде используя метод `onTrial` модели пользователя или подписки. Ниже приведены два примера:

    if ($user->onTrial('main')) {
        //
    }

    if ($user->subscription('main')->onTrial()) {
        //
    }

<a name="without-credit-card-up-front"></a>
### Без ввода данных карты

Если вы хотите предоставить пользователям триал период без ввода данных пластиковой карты, вы можете просто установить свойство `trial_ends_at` модели пользователя в дату окончания его окончания. Примерно так будет выглядеть создание пользователя:

    $user = User::create([
        // Populate other user properties...
        'trial_ends_at' => Carbon::now()->addDays(10),
    ]);

Такой тип триала внутри Cashier называется "общий триал" так как он не присоединяется ни к какой подписке. Метод `onTrial` модели `User` вернет `true` если текущая дата меньше установленной `trial_ends_at`:

    if ($user->onTrial()) {
        // User is within their trial period...
    }

Вы так же можете использовать метод `onGenericTrial` если требуется проверить находится ли пользователь на "общем" триал периоде и не имеет действительной подписки:

    if ($user->onGenericTrial()) {
        // User is within their "generic" trial period...
    }

Когда вы готовы создать настоящую подписку для пользователя, используйте метод `newSubscription`:

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')->create($creditCardToken);

<a name="handling-stripe-webhooks"></a>
## Обработка Stripe Webhooks

<a name="handling-failed-subscriptions"></a>
### Ошибка при оплате подписки

Что, если кредитка пользователя стала неактивной? Не беспокойтесь — Cashier включает в себя контроллер хуков, который автоматически отключит подписку за вас. Просто укажите путь к контроллеру в роутах: 

	Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

Это всё! Ошибки оплаты будут обработаны контроллером. Он отменит подписку, когда Stripe определит, что оплата невозможна (обычно это происходит после 3-х попыток). Не забудьте: вам нужно настроить URI для хуков в вашей панели Stripe.

Хуки Stripe должны обходить стороной [CSRF проверку](/docs/{{version}}/routing#csrf-protection) Laravel, поэтому не забудьте добавить URI в исключения посредника `VerifyCsrfToken` или разместите роут вне действия посредника `web`:

    protected $except = [
        'stripe/*',
    ];

<a name="handling-other-webhooks"></a>
### Другие Webhooks

Если вам требуется обрабатывать другие вебхуки Stripe просто расширьте контроллер `WebhookController`. Имя метода должно соответствовать правилам наименования, применяемым в Cashier, в частности, оно должно начинаться с `handle` и соответствовать имени вебхука Stripe записанному в стиле "camel case". Например, если требуется обработать вебхук `invoice.payment_succeeded`, метод контроллера должен иметь имя `handleInvoicePaymentSucceeded`.

    <?php

    namespace App\Http\Controllers;

    use Laravel\Cashier\Http\Controllers\WebhookController as BaseController;

    class WebhookController extends BaseController
    {
        /**
         * Handle a Stripe webhook.
         *
         * @param  array  $payload
         * @return Response
         */
        public function handleInvoicePaymentSucceeded($payload)
        {
            // Handle The Event
        }
    }
    
<a name="handling-braintree-webhooks"></a>
## Обработка Braintree Webhooks

<a name="handling-braintree-failed-subscriptions"></a>
### Ошибка при оплате подписки

Что, если кредитка пользователя стала неактивной? Не беспокойтесь — Cashier включает в себя контроллер хуков, который автоматически отключит подписку за вас. Просто укажите путь к контроллеру в роутах:

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

Это всё! Ошибки оплаты будут обработаны контроллером. Он отменит подписку, когда Braintree определит, что оплата невозможна (обычно это происходит после 3-х попыток). Не забудьте: вам нужно настроить URI для хуков в вашей панели Braintree.

Хуки Braintree должны обходить стороной [CSRF проверку](/docs/{{version}}/routing#csrf-protection) Laravel, поэтому не забудьте добавить URI в исключения посредника `VerifyCsrfToken` или разместите роут вне действия посредника `web`:

    protected $except = [
        'braintree/*',
    ];

<a name="handling-braintree-other-webhooks"></a>
### Другие Webhooks

Если вам требуется обрабатывать другие вебхуки Braintree просто расширьте контроллер `WebhookController`. Имя метода должно соответствовать правилам наименования, применяемым в Braintree, в частности, оно должно начинаться с `handle` и соответствовать имени вебхука Braintree записанному в стиле "camel case". Например, если требуется обработать вебхук `dispute_opened`, метод контроллера должен иметь имя `handleDisputeOpened`.

    <?php

    namespace App\Http\Controllers;

    use Braintree\WebhookNotification;
    use Laravel\Cashier\Http\Controllers\WebhookController as BaseController;

    class WebhookController extends BaseController
    {
        /**
         * Handle a Braintree webhook.
         *
         * @param  WebhookNotification  $webhook
         * @return Response
         */
        public function handleDisputeOpened(WebhookNotification $notification)
        {
            // Handle The Event
        }
    }

<a name="single-charges"></a>
## Одиночные оплаты

### Простая оплата

> **Примечание:** Когда вы используете Stripe, метод `charge` принимает сумму, которую вы хотите снять в **наименьшем номинале используемой валюты**. Однако, при использовании Braintree вы должны передавать в метод `charge` сумму в долларах:

Если вы хотите снять деньги за подписку только один раз, можете воспользоваться методом `charge`.

	$user->charge(100);

Метод `charge` принимает вторым аргументом массив, позволяющий добавлять дополнительные опции:

	// Stripe принимает сумму в центах...
    $user->charge(100);

    // Braintree принимает сумму в долларах...
    $user->charge(1);

Метод `charge` принимает массив в качестве второго аргумента, позволяя вам передавать параметры в Stripe / Braintree:

    $user->charge(100, [
        'custom_option' => $value,
    ]);

Метод `charge` выбросит исключение, если оплата не прошла. Если оплата прошла успешно, метод вернет полный ответ от Stripe / Braintree:

    try {
        $response = $user->charge(100);
    } catch (Exception $e) {
        //
    }

### Оплата с выставлением счета

Иногда может потребоваться произвести одноразовую оплату, но с выставление счета (инвойса), для которого вы можете предоставить возможность пользователю получить PDF. Метод `invoiceFor` позволит сделать это. Например, давайте выставим счет на 5.00$ за "One Time Fee":

    // Stripe принимает сумму в центах...
    $user->invoiceFor('One Time Fee', 500);

    // Braintree принимает сумму в долларах...
    $user->invoiceFor('One Time Fee', 5);

Счет будет оплачен автоматически с карты пользователя. Метод `invoiceFor` так же принимает массив аргументов, позволяя вам передавать параметры в Stripe / Braintree:

    $user->invoiceFor('One Time Fee', 500, [
        'custom-option' => $value,
    ]);

> **Примечание:** Метод `invoiceFor` создаст инвойс в Stripe который будет повторять попытку оплаты, в случае неудачи. Если вы не хотите повторять попытки оплаты, вам потребуется отменить счет через Stripe API после первой неудачной попытки.

<a name="invoices"></a>
## Инвойсы

Вы можете легко получить все инвойсы пользователя, используя метод `invoices`:

    $invoices = $user->invoices();

При отображении инвойсов пользователю вы также можете использовать вспомогательные методы для получения дополнительной информации. Например, вы можете отобразить список инвойсов таблицей с возможностью скачивания каждого из них:

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
#### Создание PDF инвойса


Для генерации PDF для инвойсов вам потребуется установить библиотеку `dompdf`:

    composer require dompdf/dompdf

Из роута или контроллера вызовите метод `downloadInvoice` для создания PDF и «отдачи» его пользователю. Этот метод автоматически создаёт соответствующий HTTP ответ с заголовками для начала скачивания в браузере:

    Route::get('user/invoice/{invoice}', function ($invoiceId) {
        return Auth::user()->downloadInvoice($invoiceId, [
            'vendor'  => 'Your Company',
            'product' => 'Your Product',
        ]);
    });
