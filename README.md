# Paylike API reference

Clients:

- Node.js: https://github.com/paylike/node-api (official)
- .NET: https://github.com/paylike/Paylike.NET (maintained by [@andronachev](https://github.com/andronachev))
- PHP: https://github.com/paylike/php-api (maintained by [@ionutcalara](https://github.com/ionutcalara))
- Java: https://github.com/paylike/java-api (maintained by [@jankjr](https://github.com/jankjr))

Building a client in [insert your favourite language here]? Reach out
(hello@paylike.io), we might consider sponsoring the maintenance.

**Make sure to [subscribe to our mailling list](http://eepurl.com/bCGmg1) for
deprecation notices, API changes and new features**, or you can watch this
repository for changes.

Check out our [getting started guide](getting-started.md).

Backwards incompatible changes including migration paths will be announced on
the mailing list and here 6 months in advance providing you plenty of time to
update.

- Basics
	- [Getting an API key](#getting-an-api-key)
	- [Authenticating](#authenticating)
	- [Data (request body)](#data-request-body)
	- [Response](#response)
	- [Pagination](#pagination)
	- [Amounts](#amounts)
- [Apps](#apps)
	- [Create an app](#create-an-app)
	- [Fetch current app](#fetch-current-app)
- [Merchants](#merchants)
	- [Create a merchant](#create-a-merchant)
		- [Country](#country)
		- [Currency](#currency)
		- [Descriptor](#descriptor)
	- [Update a merchant](#update-a-merchant)
	- [Fetch all merchants](#fetch-all-merchants)
	- [Fetch a merchant](#fetch-a-merchant)
	- [Users](#merchants-users)
		- [Invite user to a merchant](#invite-user-to-a-merchant)
		- [Revoke user from a merchant](#revoke-user-from-a-merchant)
		- [Fetch all users on a merchant](#fetch-all-users-on-a-merchant)
	- [Apps](#merchants-apps)
		- [Add app to a merchant](#add-app-to-a-merchant)
		- [Revoke app from a merchant](#revoke-app-from-a-merchant)
		- [Fetch all apps on a merchant](#fetch-all-apps-on-a-merchant)
	- [Lines](#merchants-lines)
		- [Fetch all lines on a merchant](#fetch-all-lines-on-a-merchant)
- [Transactions](#transactions)
	- [Create a transaction](#create-a-transaction)
		- [Capture a transaction](#capture-a-transaction)
		- [Refund a transaction](#refund-a-transaction)
		- [Void a transaction](#void-a-transaction)
	- [Fetch all transactions](#fetch-all-transactions)
	- [Fetch a transaction](#fetch-a-transaction)
- [Cards](#cards)
	- [Save a card](#save-a-card)
	- [Fetch a card](#fetch-a-card)
- [Recurring payments](#recurring-payments)
- [Generate payment link](#generate-payment-link)

### Getting an API key

An API key (secret) can be obtained by creating a merchant and adding an app.
If your app's target audience is third parties, please reach out and we will
make your app's API key hidden.

**The service is located at `https://api.paylike.io`.**

### Authenticating

All requests are performed with a basic authorization header containing the
API key as password:

```shell
curl <url> \
	-u :<api-key>
```

### Data (request body)

Request body data can be send with either `application/x-www-form-urlencoded`
(standard form data) or `application/json` as `Content-Type` header and data
formatted accordingly:

```shell
curl <url> \
	-u :<api-key> \
	-d key="val" \
	-d key2="val2"
```

When using `application/x-www-form-urlencoded` nested properties like
`company.country` of a merchant can be provided using form data as
`company[country]` and booleans should be either "yes" or "no" (e.g. `curl
[..] -d test=no`).

### Response

All successful calls will return a 2xx status code. 4xx is used for errors in
your end (validation, constraints, etc.) and 5xx is for server errors (please
report those). Take a look at [how we use status codes](status-codes.md).

Add header `Accept: application/json` (curl: `-H 'Accept: application/json'`)
for forward compatibility although the service will return JSON at the moment.

### Pagination

Pagination is achieved using `after`, `before` and `limit` parameters (`limit`
is required where pagination is supported).

The idea is that you start with `limit` only, then put the last primary key
(`id`) as `before` in the next request. This way you can have a reliable
experience even when dealing with lists that are increasing in realtime (such
as transactions).

All lists are by default sorted with the newest entry first.

Using `after` will automatically reverse the sort order. Use it for retrieving
newer items.

### Amounts

All amounts are represented in minor (e.g. "DKK 9.95" is represented as 995).

## Apps

A machine with an API key as credentials.

### Create an app

```shell
curl -i https://api.paylike.io/apps \
	-d <data>
```

Expected input data:

```js
{
	name: String,			// optional
}
```

Supply a name if you are developing a third party app. It will then be shown
instead of the app's key (which you should always keep secret).

Will return:

```js
{
	app: {
		id: String,		// unique key for referencing
		name: String,
		key: String,	// secret key used for authentication
	}
}
```

### Fetch current app

Get information about the authenticated app, such as the `id` and `name`.

```shell
curl -i https://api.paylike.io/me \
	-u :<api-key>
```

Will return:

```js
{
	identity: {
		id: String,		// unique ID for referencing
		name: String,	// name of your app, if it has one
	}
}
```

## Merchants

Has a funding bank account and contains all transactions.

Can have several users and apps associated. All users and apps have complete
access to the merchant and to invite and revoke others.

### Create a merchant

Make sure to mark accounts as test when implementing.

```shell
curl -i https://api.paylike.io/merchants \
	-u :<api-key> \
	-d <data>
```

Expected input data:

```js
{
	name: String,			// optional
	currency: String,		// required, three letter ISO
	test: Boolean,			// optional, defaults to false
	email: String,			// required, contact email
	website: String,		// required, website with implementation
	descriptor: String,		// required, text on client bank statements
	company: {
		country: String,	// required, ISO 3166 code (e.g. DK)
		number: String,		// optional, registration number ("CVR" in Denmark)
	},
	bank: {					// optional
		iban: String,		// optional, (format: XX00000000, XX is country code, length varies)
	},
}
```

#### Country

See https://github.com/paylike/countries for a list of supported countries,
their code and official currency.

#### Currency

This is the **funding currency**. Although you can charge customers in any
currency, all transactions will be exchanged to the funding currency upon
capture. A list of all supported currencies is available at
https://github.com/paylike/currencies, notice that only a subset is supported
for funding (marked by `funding: true`).

#### Descriptor

This is the default text that customers will see in their bank upon a charge,
if not overwritten when charging or capturing.

See https://github.com/paylike/descriptor for format and restrictions.

Will return:

```js
{
	merchant: {
		id: String,		// unique ID for referencing
		key: String,	// public key used for transactions and links
		...,			// more..
	}
}
```

You probably want to store one or both of `id` and `key`.

The created merchant is automatically associated with the creating entity
(user or app).

### Update a merchant

```shell
curl -i https://api.paylike.io/merchants/<merchant-id> \
	-X PUT \
	-u :<api-key> \
	-d <data>
```

Expected input data (all optional):

```js
{
	name: String,
	email: String,
	descriptor: String,
}
```

All other data on the merchant is immutable. Create a new merchant or contact
us if you need it changed.

### Fetch all merchants

```shell
curl -i https://api.paylike.io/identities/<app-id>/merchants?limit=<num> \
	-u :<api-key>
```

Query parameters: [pagination](#pagination) (required)

### Fetch a merchant

```shell
curl -i https://api.paylike.io/merchants/<merchant-id> \
	-u :<api-key>
```

### Merchant's users

#### Invite user to a merchant

The user will receive an email if they are not signed up at Paylike, or if
they are not a member of the merchant.

```shell
curl -i https://api.paylike.io/merchants/<merchant-id>/users \
	-u :<api-key> \
	-d <data>
```

Expected data:

```js
{
	email: String,	// required
}
```

#### Revoke user from a merchant

```shell
curl -i https://api.paylike.io/merchants/<merchant-id>/users/<user-id> \
	-X DELETE \
	-u :<api-key>
```

#### Fetch all users on a merchant

```shell
curl -i https://api.paylike.io/merchants/<merchant-id>/users?limit=<num> \
	-u :<api-key>
```

Query parameters: [pagination](#pagination) (required)

### Merchant's apps

#### Add app to a merchant

```shell
curl -i https://api.paylike.io/merchants/<merchant-id>/apps \
	-u :<api-key> \
	-d <data>
```

Expected data:

```js
{
	appId: String,	// required
}
```

#### Revoke app from a merchant

```shell
curl -i https://api.paylike.io/merchants/<merchant-id>/apps/<app-id> \
	-X DELETE \
	-u :<api-key>
```

#### Fetch all apps on a merchant

```shell
curl -i https://api.paylike.io/merchants/<merchant-id>/apps?limit=<num> \
	-u :<api-key>
```

Query parameters: [pagination](#pagination) (required)

### Merchant's lines

This is the history that makes up a merchant's balance. Captures, refunds,
payouts and other fincancial transactions are all represented by a line.

#### Fetch all lines on a merchant

```shell
curl -i https://api.paylike.io/merchants/<merchant-id>/lines?limit=<num> \
	-u :<api-key>
```

Query parameters: [pagination](#pagination) (required)

## Transactions

An authorization (reservation) of a given amount and subsequent captures,
refunds and voids.

All transactions have a `trail` property which is a list of actions. You can
check the type of the action by looking at a property of the same name (e.g.
`transaction.trails[0].capture === true`). Each entry also have an `amount`
property.

### Create a transaction

When using [payment links](#generate-payment-link) or our [frontend SDK](https://github.com/paylike/sdk)
you do not need to create any transactions.

Creating transactions is only used for [recurring payments](#recurring-payments).

#### From a previous transaction

```shell
curl -i https://api.paylike.io/merchants/<merchant-id>/transactions \
	-u :<api-key> \
	-d <data>
```

Expected input data:

```js
{
	transactionId: String,	// required
	descriptor: String,		// optional, will fallback to merchant descriptor
	currency: String,		// required, three letter ISO
	amount: Number,			// required, amount in minor units
	custom:	Object,			// optional, any custom data
}
```

Will return:

```js
{
	transaction: {
		id: String,		// unique ID for referencing
	}
}
```

#### From a saved card

Using a previous transaction is, in most cases, superior to saving a card due
to the extra work involed.

You will first need to [obtain a card ID](#save-a-card).

```shell
curl -i https://api.paylike.io/merchants/<merchant-id>/transactions \
	-u :<api-key> \
	-d <data>
```

Expected input data:

```js
{
	cardId: String,			// required
	descriptor: String,		// optional, will fallback to merchant descriptor
	currency: String,		// required, three letter ISO
	amount: Number,			// required, amount in minor units
	custom:	Object,			// optional, any custom reference or data
}
```

Will return:

```js
{
	transaction: {
		id: String,		// unique ID for referencing
	}
}
```

#### Error handling

In case of a processing error, like insufficient funds, you will see a status
code `400` and the response body will contain one of
[the processing errors](https://github.com/paylike/processing-errors).

#### Capture a transaction

The total amount of captures is always less than the transaction's amount.

```shell
curl -i https://api.paylike.io/transactions/<transaction-id>/captures \
	-u :<api-key> \
	-d <data>
```

Expected input data:

```js
{
	amount: Number,			// required, amount in minor units (100 = DKK 1,00)
	currency: String,		// optional, expected currency (for additional verification)
	descriptor: String,		// optional, text on client bank statement
}
```

The only acceptable value for `currency` is that of the transaction itself.
The attribute is provied to make sure a user did not tamper with the currency
during authorization. This way, if the capture succeeds, you are guaranteed to
have at least the right amount of money.

#### Refund a transaction

The total amount of refunds is always less than the total amount captured.

```shell
curl -i https://api.paylike.io/transactions/<transaction-id>/refunds \
	-u :<api-key> \
	-d <data>
```

Expected input data:

```js
{
	amount: Number,			// required, amount in minor units (100 = DKK 1,00)
	descriptor: String,		// optional, text on client bank statement
}
```

#### Void a transaction

A complete or partial cancellation of the reserved amount.

```shell
curl -i https://api.paylike.io/transactions/<transaction-id>/voids \
	-u :<api-key> \
	-d <data>
```

Expected input data:

```js
{
	amount: Number,			// required, amount in minor units (100 = DKK 1,00)
}
```

### Fetch all transactions

```shell
curl -i https://api.paylike.io/merchants/<merchant-id>/transactions?limit=<num> \
	-u :<api-key>
```

Query parameters: [pagination](#pagination) (required)

### Fetch a transaction

```shell
curl https://api.paylike.io/transactions/<transaction-id> \
	-u :<api-key>
```

Will return:

```js
{
	transaction: {
		id: String,
		merchantId: String,		// ID of the owning merchant account
		test: Boolean,			// whether on a test merchant account
		created: Date,			// Date of transaction

		currency: String,		// currency ISO code
		amount: Number,			// amount in minor units
		descriptor: String,		// text on bank statement

		pendingAmount: Number,	// amount available for capture or void
		capturedAmount: Number,	// amount captured (available for refund)
		refundedAmount: Number,	// amount refunded (no further action possible)
		voidedAmount: Number,	// amount voided (no further action possible)
		disputedAmount: Number,	// amount involed in disputes such as chargebacks

		card: {
			bin: String,		// first 6 numbers in PAN (card number)
			last4: String,
			expiry: Date,
			scheme: String,		// "visa" or "mastercard"
		},

		custom: Object,			// custom data

		tds: String,			// one of "attempt" or "full" if 3-D Secure was applied
		recurring: Boolean,		// whether the transaction was made from the server
		successful: Boolean,
		error: false|Object,	// contains a processing error if unsuccessful

		// list of all captures, voids, refunds and disputes
		trail: [
			{
				// only one of the following will be present
				capture: Boolean,
				refund: Boolean,
				void: Boolean,
				dispute: {
					id: String,

					// only one of the following will be present
					won: Boolean,
					lost: Boolean,
				},

				created: Date,

				amount: Number,		// amount in minor units and transaction currency

				/*
				Amount in the merchant account's currency in minor units that
				the merchant account balance was affected with which in
				practice means the actual profit from the transaction after
				fees and/or currency conversion.
				*/
				balance: Number,

				fee: Object,		// detailed description of fees applied
				descriptor: String,	// text on bank statement

				lineId: String,		// ID of the related accounting line
			},
		],
	}
}
```

## Cards

### Save a card

When using our [frontend SDK](https://github.com/paylike/sdk) for saving cards
you do not have to do anything further - the card will be in your vault.

The instructions below are for saving a card from an earlier transaction.

```shell
curl -i /merchants/<merchant-id>/cards \
	-u :<api-key> \
	-d <data>
```

Expected input data:

```js
{
	transactionId: String,	// required
	notes: String,			// optional
}
```

Will return:

```js
{
	card: {
		id: String,		// unique ID for referencing
	}
}
```

Once you have a card ID, you will be able to [create new transactions](#create-a-transaction).

### Fetch a card

```shell
curl https://api.paylike.io/cards/<card-id> \
	-u :<api-key>
```

Will return:

```js
{
	card: {
		id: String,
		merchantId: String,
		created: Date,
		bin: String,	// first 6 numbers in PAN (card number)
		last4: String,
		expiry: Date,
		scheme: String, // "visa" or "mastercard"
	}
}
```

## Recurring payments

Things you should be aware of:

- cards expire
- a card could be reported stolen between payments
- the card may not have sufficient funds
- banks might temporarily decline cards
- recurring payments may require the user to manually enable it at their bank

Your flow should gracely handle failures and allow users to pay with another
card and as a regular transaction.

Due to some banks not accepting recurring payments (CVC-less) by default a
transaction is **more likely to be successful with a regular payment** thus
you should favor regular transactions and use our "new transaction from
existing" as an optimization - don't save card details upfront if you can
avoid it, that's also good conversion karma.

Present the user with the initial payment (regular transaction with CVC) and
ask your user whether they want to subscribe to future payments. On the next
payment try creating a transaction - if it fails, ask the user to do the
payment manually (with CVC) and restart the process.

An example flow could look like this:

1. (client) A payment popup is shown or a payment link is generated
2. (client) The user is asked whether to save their card for future payments
3. (server) Save transaction
4. (server/async) Capture the transaction

	This step should be completed only when your services or your goods are
	dispatched to the customer.

5. (server/async) Recurring payment

	[Create a transaction](#create-a-transaction) based on the previous
	transaction ID saved in 3. and capture it, if it fails for whatever reason
	(expired, not supported, insufficient funds, etc.), notify the customer by
	email or other means and restart the process from 1.

You do not need to do clever stuff about expiration if you follow this flow -
cards will fail for whatever reason and be replaced by the customer.

You could enhance the flow by creating recurring payments a bit earlier to
warn the user if an upcoming payment will fail and needs to be completed
manually. Delay the capture for the actual renewal date.

If you are using 3D-Secure you should be aware that the protocol only supports
a transaction and not saving a card thus you cannot use 3D-Secure for
recurring payments, but you will have a much stronger case if the first
transaction is a regular 3D-Secure protected one.

## Generate payment link

For mPOS and payment links the format of the link should conform to:

```
https://pos.paylike.io/
?key=<public key>
&currency=<three letter ISO>
&amount=<amount in minor>
&reference=<text shown in dashboard>
&text=<text shown on payment page>
&redirect=<url>
&locale=<locale (e.g. en_US or en)>
```

All except `key` are optional. If `amount` is included the user is shown the
payment page, if not, it is considered an mPOS case and a pre-screen is shown
for manually setting the amount.

For now only Danish (`da`) and English (default) is supported for the locale,
please open an issue to request others.

Payment link example: https://pos.paylike.io/?key=ba21f7ac-095c-4941-3196-f6ba24effbaf&currency=DKK&amount=100&reference=order%20232&redirect=http://google.dk

mPOS link example: https://pos.paylike.io/?key=ba21f7ac-095c-4941-3196-f6ba24effbaf&currency=DKK&reference=order%20232
