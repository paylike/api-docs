# Paylike API reference

Clients:

- Node.js: https://github.com/paylike/node-api (official)

Building a client in [insert your favourite language here]? Reach out
(hello@paylike.io), we might consider sponsoring the maintenance.

**Make sure to [subscribe to our mailling list](http://eepurl.com/bCGmg1)
for deprecation notices, API changes and new features**, or you can watch this repository for changes.

Backwards incompatible changes including migration paths will be announced on
the mailing list and here 6 months in advance. This way you will always have
at least half a year to update your system.

- Basics
	- [Getting an API key](#getting-an-api-key)
	- [Authenticating](#authenticating)
	- [Data (request body)](#data-request-body)
	- [Response](#response)
	- [Pagination](#pagination)
	- [Amounts](#amounts)
- [Types](#types)
	- User
	- App
	- Merchant
	- Transaction
	- Card
- [Fetch current app](#fetch-current-app)
- [Merchants](#merchants)
	- [Create a merchant](#create-a-merchant)
		- [Country](#country)
		- [Currency](#currency)
		- [Descriptor](#descriptor)
	- [Invite user to a merchant](#invite-user-to-a-merchant)
	- [Fetch all merchants](#fetch-all-merchants)
	- [Fetch a merchant](#fetch-a-merchant)
- [Transactions](#transactions)
	- [Create a transaction](#create-a-transaction)
		- [Capture a transaction](#capture-a-transaction)
		- [Refund a transaction](#refund-a-transaction)
		- [Void a transaction](#void-a-transaction)
	- [Fetch all transactions](#fetch-all-transactions)
	- [Fetch a transaction](#fetch-a-transaction)
- [Cards](#cards)
	- [Save a card](#save-a-card)
- [Recurring payments](#recurring-payments)
- [Generate payment link](#generate-payment-link)

### Getting an API key

An API key can be obtained by creating a merchant and adding an app. If your
app's target audience is third parties, please reach out and we will make your
app's API key hidden.

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

Nested properties like `company.country` of a merchant can be provided using
form data as `company[country]`.

### Response

All successful calls will return a 2xx status code. 4xx is used for errors in
your end (validation, constraints, etc.) and 5xx is for server errors (please
report those).

Add header `Accept: application/json` (curl: `-H 'Accept: application/json'`)
for forward compatibility although the service will return JSON at the moment.

### Pagination

Pagination is achieved using a `limit` and a `skip` parameter (`limit` is
required where pagination is supported).

### Amounts

All amounts are represented in minor (e.g. "DKK 9.95" is represented as 995).

## Types

- User

	A human being with email and password as credentials.

	- [Invite a user to a merchant](#invite-user-to-a-merchant)

- App

	A machine with an API key as credentials.

	- [Fetch app's own data](#fetch-current-app)

- Merchant

	Has a funding bank account and contains all transactions.

	Can have several users and apps associated. All users and apps have
	complete access to the merchant and to invite and revoke others.

	- [Create a merchant](#create-a-merchant)
	- [Fetch all merchants you can access](#fetch-all-merchants)
	- [Fetch a merchant](#fetch-a-merchant)

- Transaction

	An authorization (reservation) of a given amount and subsequent captures,
	refunds and voids.

	- [Fetch all transactions](#fetch-all-transactions)
	- [Fetch a transaction](#fetch-a-transaction)
	- [Create a transaction (recurring)](#create-a-transaction)

	All transactions have a "trail" property which is a list of actions. You
	can check the type of the action by looking at a property of the same name
	(e.g. `trails[0].capture === true`). Each entry also have an `amount`
	property. The types are:

	- Capture

		The total amount of captures is always less than the transasctions
		amount.

		- [Capture a transaction](#capture-a-transaction)

	- Refund

		The total amount of refunds is always less than the total amount
		captured.

		- [Refund a transaction](#refund-a-transaction)

	- Void

		A complete or partial cancellation of the reserved amount.

		- [Void a transaction](#void-a-transaction)

- Card

	A card is.. a credit card.

	- [Save a card](#save-a-card)
	- [Create a transaction](#create-a-transaction)

## Fetch current app

Get information about the authenticated app, such as the "pk" and "name".

```shell
curl -i https://api.paylike.io/me \
	-u :<api-key>
```

Will return:

```js
{
	identity: {
		pk: String,		// unique key for referencing
		name: String,	// name of your app, if it has one
	}
}
```

## Merchants

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
		pk: String,		// unique key for referencing
		key: String,	// public key used for transactions and links
		...,			// more..
	}
}
```

You probably want to store one or both of "pk" and "key".

The created merchant is automatically associated with the creating entity
(user or app).

### Invite user to a merchant

The user will receive an email if they are not signed up at Paylike, or if
they are not a member of the merchant.

```shell
curl -i https://api.paylike.io/merchants/<pk>/invite \
	-u :<api-key> \
	-d <data>
```

Expected data:

```js
{
	email: String,	// required
}
```

### Fetch all merchants

```shell
curl -i https://api.paylike.io/identities/<app-pk>/merchants \
	-u :<api-key>
```

Query parameters: [pagination](#pagination) (required)

### Fetch a merchant

```shell
curl -i https://api.paylike.io/merchants/<merchant-pk> \
	-u :<api-key>
```

## Transactions

### Create a transaction

When using [payment links](#generate-payment-link) or our [frontend SDK](https://github.com/paylike/sdk)
you do not need to create any transactions.

Creating transactions is only used for [recurring payments](#recurring-payments).

#### From a previous transaction

```shell
curl -i https://api.paylike.io/merchants/<merchant-pk>/transactions \
	-u :<api-key> \
	-d <data>
```

Expected input data:

```js
{
	transactionPk: String,	// required
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
		pk: String,		// unique key for referencing
		...,			// more..
	}
}
```

#### From a saved card

Using a previous transaction is, in most cases, superior to saving a card due
to the extra work involed.

You will first need to [obtain a card key](#save-a-card).

```shell
curl -i https://api.paylike.io/merchants/<merchant-pk>/transactions \
	-u :<api-key> \
	-d <data>
```

Expected input data:

```js
{
	cardPk: String,			// required
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
		pk: String,		// unique key for referencing
		...,			// more..
	}
}
```

#### Capture a transaction

```shell
curl -i https://api.paylike.io/transactions/<transaction-pk>/captures \
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

```shell
curl -i https://api.paylike.io/transactions/<transaction-pk>/refunds \
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

```shell
curl -i https://api.paylike.io/transactions/<transaction-pk>/voids \
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
curl -i https://api.paylike.io/merchants/<merchant-pk>/transactions \
	-u :<api-key>
```

Query parameters: [pagination](#pagination) (required)

### Fetch a transaction

```shell
curl https://api.paylike.io/transactions/<transaction-pk> \
	-u :<api-key>
```

Will return:

```js
{
	transaction: {
		pk: String,
		amount: Number,
		pendingAmount: Number,	// available for capture or void
		capturedAmount: Number,	// captured (available for refund)
		refundedAmount: Number,	// refunded (no further action possible)
		voidedAmount: Number,	// voided (no further action possible)
		card: {
			bin: String,	// first 6 numbers in PAN (card number)
			last4: String,
			expiry: Date,
			scheme: String, // "visa" or "mastercard"
		},
		currency: "USD",
		custom: Object,		// custom data
		trail: Array,		// list of all captures, voids and refunds
	}
}
```

## Cards

### Save a card

When using our [frontend SDK](https://github.com/paylike/sdk) for saving cards
you do not have to do anything further - the card will be in your vault.

The instructions below are for saving a card from an earlier transaction.

```shell
curl -i /merchants/<merchant-pk>/cards \
	-u :<api-key> \
	-d <data>
```

Expected input data:

```js
{
	transactionPk: String,	// required
	notes: String,			// optional
}
```

Will return:

```js
{
	card: {
		pk: String,		// unique key for referencing
		...,			// more..
	}
}
```

Once you have a card key, you should be able to [create new transactions](#create-a-transaction).

## Recurring payments

Things you should be aware of:

- recurring payments are not supported by all card issuing banks
- cards expire
- a card could be reported stolen between payments
- the card may not have sufficient funds
- a host of things will go wrong

Your flow should gracely handle these failures and allow affected users to pay
with another card.

This is the reason you should avoid using the "save card" part of our frontend
SDK for anything else than updating the card of an existing customer - the
transaction is **more likely to be successful with a regular payment**.

Instead present the user with the initial payment and ask your user whether
they want to subscribe to future payments. On the next payment you simply try
creating a transaction - if it fails, ask the user to do the payment manually
and restart the process.

An example flow could look like this:

1. (client) A payment popup is shown or a payment link is generated
2. (client) The user is asked whether to save their card for future payments
3. (server) Save transaction
4. (server/async) Capture the transaction

	This step should be completed only when your services or your goods are
	dispatched to the customer.

5. (server/async) Recurring payment

	[Create a transaction](#create-a-transaction) based on the previous transaction key
	saved in 3. and capture it, if it fails for whatever reason (expired,
	not supported, insufficient funds, etc.), notify the customer by email or
	other means and restart the process from 1.

You do not need to do clever stuff about expiration if you follow this flow -
cards will fail for whatever reason and be replaced by the customer.

You could enhance the flow by creating rrecurring payments a bit earlier to
warn the user if an upcoming payment will fail and needs to be completed
manually. Delay the capture for the actual renewal date.

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
```

All except `key` are optional. If `amount` is included the user is shown the
payment page, if not, it is considered an mPOS case and a pre-screen is shown
for manually setting the amount.

Payment link example: https://pos.paylike.io/?key=ba21f7ac-095c-4941-3196-f6ba24effbaf&currency=DKK&amount=100&reference=order%20232&redirect=http://google.dk

mPOS link example: https://pos.paylike.io/?key=ba21f7ac-095c-4941-3196-f6ba24effbaf&currency=DKK&reference=order%20232
