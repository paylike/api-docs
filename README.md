# Paylike API

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
	- [Recurring payments (important information)](#recurring-payments-important-information)
- [Generate payment link](#generate-payment-link)

### Getting an API key

An API key can be obtained by creating a merchant and adding an app. If your
app's target audience is third parties, please reach out and we will make your
app's API key hidden.

The service is located at `https://midgard.paylike.io`.

### Authenticating

All requests are performed with a basic authorization header containing the
API key as password:

```shell
curl -u :<api-key> <url>
```

### Data (request body)

Request body data can be send with either `application/x-www-form-urlencoded`
(standard form data) or `application/json` as `Content-Type` header and data
formatted accordingly:

```shell
curl -X POST -u :<api-key> <url> --data key=val&key2=val2
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
curl -u :<api-key> https://midgard.paylike.io/me
```

Will return:

```js
{
	identity: {
		pk: String,		// unique key for referencing
		name: String,	// name of you app, if it has one
	}
}
```

## Merchants

### Create a merchant

Make sure to mark accounts as test when implementing.

```shell
curl -X POST -u :<api-key> https://midgard.paylike.io/merchants <data>
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
		country: String,	// required, the English name of an EU country (e.g. Denmark)
	},
}
```

#### Country

See https://github.com/paylike/countries for a list of supported countries. It
also has ISO 3166-1-alpha-2 codes and primary currency of each country.

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
curl -X POST -u :<api-key> https://midgard.paylike.io/merchants/<pk>/invite <data>
```

Expected data:

```js
{
	email: String,	// required
}
```

### Fetch all merchants

```shell
curl -X POST -u :<api-key> https://midgard.paylike.io/identities/<app-pk>/merchants
```

### Fetch a merchant

```shell
curl -X POST -u :<api-key> https://midgard.paylike.io/merchants/<merchant-pk>
```

Query parameters: (pagination), live

## Transactions

### Create a transaction

When using [payment links](#generate-payment-link) or our [frontend SDK](https://github.com/paylike/sdk)
you do not need to create any transactions.

Creating transactions is only used for recurring payments. In order to create
a transaction, you will first need to [obtain a card key](#save-a-card).

```shell
curl -X POST -u :<api-key> https://midgard.paylike.io/merchants/<merchant-pk>/transactions <data>
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
curl -X POST -u :<api-key> https://midgard.paylike.io/transactions/<transaction-pk>/captures <data>
```

Expected input data:

```js
{
	amount: Number,			// required, amount in minor units (100 = DKK 1,00)
	descriptor: String,		// optional, text on client bank statement
}
```

#### Refund a transaction

```shell
curl -X POST -u :<api-key> https://midgard.paylike.io/transactions/<transaction-pk>/refunds <data>
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
curl -X POST -u :<api-key> https://midgard.paylike.io/transactions/<transaction-pk>/voids <data>
```

Expected input data:

```js
{
	amount: Number,			// required, amount in minor units (100 = DKK 1,00)
}
```

### Fetch all transactions

```shell
curl -X POST -u :<api-key> https://midgard.paylike.io/merchants/<merchant-pk>/transactions
```

Query parameters: (pagination)

### Fetch a transaction

```shell
curl -u :<api-key> https://midgard.paylike.io/transactions/<transaction-pk>
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
curl -X POST -u :<api-key> /merchants/<merchant-pk>/cards <data>
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

### Recurring payments (important information)

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

1. A payment popup is shown or a payment link is generated
2. The user is asked to save their card for future payments
3. *async/server side* [Save the card](#save-a-card) from the transaction key
4. *async/server side* Capture the transaction

	This step should be completed only when your services or your goods are
	dispatched to the customer.

5. (future payment/server side)

	[Create a transaction](#create-a-transaction) based on the card key
	obtained in 3. and capture it, if it fails for whatever reason (expired,
	not supported, insufficient funds, etc.), notify the customer by email or
	other means and restart the process from 1.

You do not need to do clever stuff about expiration if you follow this flow -
cards will fail for whatever reason and be replaced by the customer.

You could enhance the flow by creating subsequent payments a bit earlier to
warn the user if an upcoming payment will fail and the card needs replaced.
Delay the capture for the actual renewal date.

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
