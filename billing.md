git c183ef46073344b1dc63c68db38e4a8d17176a31

---

# Laravel Cashier

- [Введение](#introduction)
- [Подписки](#subscriptions)
	- [Создание подписок](#creating-subscriptions)
	- [Проверка статуса подписки](#checking-subscription-status)
	- [Смена планов подписок](#changing-plans)
	- [Цена подписки](#subscription-quantity)
	- [Налог на подписку](#subscription-taxes)
	- [Отмена подписки](#cancelling-subscriptions)
	- [Приостановка подписки](#resuming-subscriptions)
- [Обработка Stripe Webhooks](#handling-stripe-webhooks)
	- [Ошибка при оплате подписки](#handling-failed-subscriptions)
	- [Другие Webhooks](#handling-other-webhooks)
- [Одиночные оплаты](#single-charges)
- [Инвойсы](#invoices)
	- [Создание PDF инвойса](#generating-invoice-pdfs)

<a name="introduction"></a>
## Введение

Laravel Cashier предоставляет простой и выразительный интерфейс для работы с платными подписками при помощи [Stripe](https://stripe.com). Он предоставляет готовые методы для работы со Stripe, которые вы, возможно, писали раньше вручную. В дополнение к базовому управлению подписками, Cashier поддерживает купоны, смену подписок, цену подписок, отмену подписок на определённый период и даже создание PDF отчётов по инвойсам.

<a name="configuration"></a>
### Настройка

#### Composer

Для начала добавьте пакет Cashier в ваш `composer.json` и выполните команду `composer update`:

	"laravel/cashier": "~5.0" (For Stripe SDK ~2.0, and Stripe APIs on 2015-02-18 version and later)
	"laravel/cashier": "~4.0" (For Stripe APIs on 2015-02-18 version and later)
	"laravel/cashier": "~3.0" (For Stripe APIs up to and including 2015-02-16 version)

#### Сервис-провайдер

Дальше зарегистрируйте [сервис-провайдер](/docs/{{version}}/providers) `Laravel\Cashier\CashierServiceProvider` в вашем файле конфигурации `app`.

#### Миграции

Перед использованием Cashier вам необходимо добавить несколько колонок в вашу базу данных. Для этого вы можете воспользоваться командой Artisan `cashier:table`, которая создаст необходимые миграции. Например, для добавления необходимых колонок в таблицу `users` команда будет выглядеть так: `php artisan cashier:table users`.

После того, как миграция создана, просто запустите команду `migrate`.

#### Настройка модели

Дальше добавьте трейт `Billable` и соответствующие мутаторы даты в вашу модель пользователя:

	use Laravel\Cashier\Billable;
	use Laravel\Cashier\Contracts\Billable as BillableContract;

	class User extends Model implements BillableContract {

		use Billable;

		protected $dates = ['trial_ends_at', 'subscription_ends_at'];

	}

Добавление колонок в свойство `$dates` укажет Eloquent о необходимости возвращать экземпляр Carbon / DateTime вместо обычных строк.

#### Ключ Stripe

И, наконец, добавьте ваш ключ Stripe в конфигурационный файл `services.php`:

	'stripe' => [
		'model'  => 'User',
		'secret' => env('STRIPE_API_SECRET'),
	],

<a name="subscriptions"></a>
## Подписки

<a name="creating-subscriptions"></a>
### Создание подписок

Для создания подписки сначала необходимо получить экземпляр вашей модели `App\User`.  После того, как вы его получили, можете использовать метод `subscription` для управления подписками пользователя:

	$user = User::find(1);

	$user->subscription('monthly')->create($creditCardToken);

Метод `create` автоматически создаст подписку Stripe и обновит вашу базу данных платёжной информацией пользователя. Если у вас настроен пробный период подписок в Stripe, то эта информация также попадёт в базу данных.

Если необходимо реализовать пробный период подписок, но вы хотите управлять им в вашем приложении, а не через Stripe, вы должны вручную выставить дату окончания пробной подписки:

	$user->trial_ends_at = Carbon::now()->addDays(14);

	$user->save();

#### Дополнительная информация о пользователе

Если вы хотите указать дополнительную информацию о пользователе при создании подписки, это можно сделать при помощи второго аргумента метода `create`:

	$user->subscription('monthly')->create($creditCardToken, [
		'email' => $email, 'description' => 'Our First Customer'
	]);

Чтобы узнать больше о дополнительных полях, поддерживаемых Stripe, обратитесь к [документации](https://stripe.com/docs/api#create_customer).

#### Купоны

Если вы хотите применить купон при создании подписки, можете использовать метод `withCoupon`:

	$user->subscription('monthly')
	     ->withCoupon('code')
	     ->create($creditCardToken);

<a name="checking-subscription-status"></a>
### Проверка статуса подписки

После того, как пользователь оформил подписку на ваше приложение, вы легко можете проверить статус подписки различными способами. Метод `subscribed` возвратит `true`, если подписка пользователя активна или в пробном периоде:

	if ($user->subscribed()) {
		//
	}

Этот метод является хорошим кандидатом на [посредника](/docs/{{version}}/middleware) роутов, так как вы можете ограничивать доступ к конкретным роутам и контроллерам в зависимости от статуса подписки:

	public function handle($request, Closure $next)
	{
		if ($request->user() && ! $request->user()->subscribed()) {
			// Пользователь не оплатил подписку
			return redirect('billing');
		}

		return $next($request);
	}

Если вы хотите определить, использует ли пользователь пробный период, воспользуйтесь методом `onTrial`. Он полезен для отображения предупреждений пользователю о том, что у него пробный период подписки:

	if ($user->onTrial()) {
		//
	}

Метод `onPlan` позволяет определить, подписан ли пользователь на определённый план подписки Stripe:

	if ($user->onPlan('monthly')) {
		//
	}

#### Определение отменённых подписок

Для того, чтобы проверить, была ли у пользователя активная подписка, которую он отменил, вы можете воспользоваться методом `cancelled`:

	if ($user->cancelled()) {
		//
	}

Вы также можете определить, отменил ли пользователь свою подписку до конца срока её окончания. Например, пользователь отменил подписку 5-го марта, но она активна до 10-го марта. Тем не менее, подписка всё ещё будет считаться активной при проверке методом `subscribed`. Определить отменённую подписку до окончания срока её действия можно при помощи метода `onGracePeriod`:

	if ($user->onGracePeriod()) {
		//
	}

Метод `everSubscribed` позволяет выяснить, имел ли пользователь когда-либо активную подписку:

	if ($user->everSubscribed()) {
		//
	}

<a name="changing-plans"></a>
### Смена планов подписок

После того, как пользователь оформил подписку в вашем приложении, он может захотеть сменить план подписки. Для этого используйте метод `swap`. Например, вы можете легко сменить подписку пользователя на `premium`:

	$user = App\User::find(1);

	$user->subscription('premium')->swap();

If the user is on trial, the trial period will be maintained. Also, if a "quantity" exists for the subscription, that quantity will also be maintained. When swapping plans, you may also use the `prorate` method to indicate that the charges should be pro-rated. In addition, you may use the `swapAndInvoice` method to immediately invoice the user for the plan change:

	$user->subscription('premium')
				->prorate()
				->swapAndInvoice();

<a name="subscription-quantity"></a>
### Subscription Quantity

Sometimes subscriptions are affected by "quantity". For example, your application might charge $10 per month **per user** on an account. To easily increment or decrement your subscription quantity, use the `increment` and `decrement` methods:

	$user = User::find(1);

	$user->subscription()->increment();

	// Add five to the subscription's current quantity...
	$user->subscription()->increment(5);

	$user->subscription()->decrement();

	// Subtract five to the subscription's current quantity...
	$user->subscription()->decrement(5);

For more information on subscription quantities, consult the [Stripe documentation](https://stripe.com/docs/guides/subscriptions#setting-quantities).

<a name="subscription-taxes"></a>
### Налоги на подписку

С Cashier очень просто ввести налог на подписку при помощи значения `tax_percent` Stripe. Для указания налога нужно воспользоваться методом  `getTaxPercent` вашей модели пользователя и возвратить в нём число от 0 до 100 с максимум двумя знаками после запятой:

	public function getTaxPercent() {
		return 20;
	}

This enables you to apply a tax rate on a model-by-model basis, which may be helpful for a user base that spans multiple countries.

<a name="cancelling-subscriptions"></a>
### Отмена подписки

Для отмены подписки вызовите метод `cancel` у подписки пользователя:

	$user->subscription()->cancel();

Когда подписка отменяется, Cashier автоматически устанавливает поле `subscription_ends_at` в вашей базе данных. Это поле используется методом `subscribed` для возвращения `false`. Например, если пользователь отменил подписку 1-го марта, но она активна до 5-го марта, метод `subscribed` всё ещё будет возвращать `true` до 5-го марта.

Отменил ли пользователь подписку досрочно, вы можете узнать при помощи метода `onGracePeriod`:

	if ($user->onGracePeriod()) {
		//
	}

<a name="resuming-subscriptions"></a>
### Восстановление подписки

Если пользователь отменил свою подписку и хочет восстановить её, используйте метод `resume`:

	$user->subscription('monthly')->resume($creditCardToken);

Если пользователь отменил подписку до того, как она на самом деле истекла, метод снимет оплату не сразу, а после окончания подписки.

<a name="handling-stripe-webhooks"></a>
## Обработка Stripe Webhooks

<a name="handling-failed-subscriptions"></a>
### Ошибка при оплате подписки

Что, если кредитка пользователя стала неактивной? Не беспокойтесь — Cashier включает в себя контроллер хуков, который автоматически отключит подписку за вас. Просто укажите путь к контроллеру в роутах: 

	Route::post('stripe/webhook', 'Laravel\Cashier\WebhookController@handleWebhook');

Это всё! Ошибки оплаты будут обработаны контроллером. Он отменит подписку, когда Stripe определит, что оплата невозможна (обычно это происходит после 3-х попыток). Не забудьте: вам нужно настроить URI для хуков в вашей панели Stripe.

Хуки Stripe должны обходить стороной [CSRF проверку](/docs/{{version}}/routing#csrf-protection) Laravel, поэтому не забудьте добавить URI в исключения посредника `VerifyCsrfToken`:

	protected $except = [
		'stripe/*',
	];

<a name="handling-other-webhooks"></a>
### Другие Webhooks

If you have additional Stripe webhook events you would like to handle, simply extend the Webhook controller. Your method names should correspond to Cashier's expected convention, specifically, methods should be prefixed with `handle` and the "camel case" name of the Stripe webhook you wish to handle. For example, if you wish to handle the `invoice.payment_succeeded` webhook, you should add a `handleInvoicePaymentSucceeded` method to the controller.

	<?php

	namespace App\Http\Controller;

	use Laravel\Cashier\WebhookController as BaseController;

	class WebhookController extends BaseController
	{
		/**
		 * Handle a stripe webhook.
		 *
		 * @param  array  $payload
		 * @return Response
		 */
		public function handleInvoicePaymentSucceeded($payload)
		{
			// Handle The Event
		}
	}

<a name="single-charges"></a>
## Одиночные оплаты

Если вы хотите снять деньги за подписку только один раз, можете воспользоваться методом `charge`. Он принимает аргументом сумму, которую вы хотите снять, в самом низком номинале валюты по умолчанию в вашем приложении. Например, если вы хотите снять 100 центов или 1 $, можно сделать это так:

	$user->charge(100);

Метод `charge` принимает вторым аргументом массив, позволяющий добавлять дополнительные опции:

	$user->charge(100, [
		'source' => $token,
		'receipt_email' => $user->email,
	]);

Метод `charge` возвратит `false`, если оплата завершилась неудачей. Обычно это означает, что оплата была отклонена:

	if ( ! $user->charge(100)) {
		// Оплата отклонена
	}

Если оплата прошла успешно, метод возвратит полный ответ Stripe.

<a name="invoices"></a>
## Инвойсы

Вы можете легко получить все инвойсы пользователя, используя метод `invoices`:

	$invoices = $user->invoices();

При отображении инвойсов пользователю вы также можете использовать вспомогательные методы для получения дополнительной информации. Например, вы можете отобразить список инвойсов таблицей с возможностью скачивания каждого из них:

	<table>
		@foreach ($invoices as $invoice)
			<tr>
				<td>{{ $invoice->dateString() }}</td>
				<td>{{ $invoice->dollars() }}</td>
				<td><a href="/user/invoice/{{ $invoice->id }}">Download</a></td>
			</tr>
		@endforeach
	</table>

<a name="generating-invoice-pdfs"></a>
#### Создание PDF инвойса

Из роута или контроллера вызовите метод `downloadInvoice` для создания PDF и «отдачи» его пользователю. Этот метод автоматически создаёт соответствующий HTTP ответ с заголовками для начала закачки в браузере:

	Route::get('user/invoice/{invoice}', function ($invoiceId) {
		return Auth::user()->downloadInvoice($invoiceId, [
			'vendor'  => 'Your Company',
			'product' => 'Your Product',
		]);
	});
