# Paylike API

## Basics

### Getting an API key

An API key can be obtained by creating a merchant and adding an app. If your
app's target audience is third parties, please reach out and we will make your
app's API key hidden.

### Authenticating

All requests are performed with a basic authorization header containing the
API key as password:

```shell
curl -u :<api-key> <url>
```

### Sending data

Request body data can be send with either `application/x-www-form-urlencoded`
(standard form data) or `application/json` as `Content-Type` header and data
formatted accordingly:

```shell
curl -X POST -u :<api-key> <url> --data key=val&key2=val2
```

Add header `Accept: application/json` (curl: `-H 'Accept: application/json'`)
for forward compatibility although the service will return JSON at the moment.

The service is located at `https://midgard.paylike.io`.

### Amounts

All amounts are represented in minor (e.g. "DKK 9.95" is represented as 995).

## Types

- User

	A human being with email and password as credentials.

	- [Invite a user to a merchant](#invite-user)

- App

	A machine with an API key as credentials.

- Merchant

	Has a funding bank account and contains all transactions.

	Can have several users and apps associated. All users and apps have
	complete access to the merchant and to invite and revoke others.

	- [Create a merchant](#create-merchant)
	- [Fetch all merchants you can access](#fetch-all-merchants)
	- [Fetch a single merchant](#fetch-a-single-merchant)

- Transaction

	An authorization (reservation) of a given amount and subsequent captures,
	refunds and voids.

	All transactions have a "trail" property which is a list of actions. You
	can check the type of the action by looking at a property of the same name
	(e.g. `trails[0].capture === true`). Each entry also have an `amount`
	property. The types are:

	- Capture

		The total amount of captures is always less than the transasctions
		amount.

	- Refund

		The total amount of refunds is always less than the total amount
		captured.

	- Void

		A complete or partial cancellation of the reserved amount.

## Create merchant

Make sure to mark accounts as test when implementing.

```shell
curl -X POST :<api-key> https://midgard.paylike.io/merchants <data>
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

Nested properties like `company.country` can be provided using form data as
`company[country]` if not using JSON format.

The created merchant is automatically associated with the creating entity
(user or app).

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

## Invite user

The user will receive an email if they are not signed up at Paylike, or if
they are not a member of the merchant.

```shell
curl -X POST :<api-key> https://midgard.paylike.io/merchants/<pk>/invite <data>
```

Expected data:

```js
{
	email: String,	// required
}
```

## Capture transaction

```shell
curl -X POST :<api-key> https://midgard.paylike.io/transactions/<transaction-pk>/captures <data>
```

Expected input data:

```js
{
	amount: String,			// required, amount in minor units (100 = DKK 1,00)
	descriptor: String,		// optional, text on client bank statement
}
```

Will return 2xx status code on success.

## Fetch all merchants

```shell
curl -X POST :<api-key> https://midgard.paylike.io/identities/<app-pk>/merchants
```

## Fetch a single merchant

```shell
curl -X POST :<api-key> https://midgard.paylike.io/merchants/<merchant-pk>
```

Query parameters: skip (pagination), limit (pagination), live

## Fetch all transactions

```shell
curl -X POST :<api-key> https://midgard.paylike.io/merchants/<merchant-pk>/transactions
```

Query parameters: skip (pagination), limit (pagination)

## Fetch a single transaction

```shell
curl -X GET :<api-key> https://midgard.paylike.io/transactions/<transaction-pk>
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
			bin: String,
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

All amounts are in minor (e.g. "1000" could be DKK 10,00)

## Generate link

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
