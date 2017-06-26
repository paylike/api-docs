# Gateway API reference

If you are building a web client you should probably use our [Web
SDK](https://github.com/paylike/sdk) instead.

The gateway handles only two types of requests; creating a payment and
tokenizing a card, both involving sensitive card details.

If possible, always issue requests to the gateway directly from the client you
are building to minimize the exposure of sensitive data and thereby reduce
your work to comply with the PCI DSS.

## Create a payment

```sh
$ curl 'https://gateway.paylike.io/transactions' \
	-d "key=<public-key>" \
	-d "currency=EUR" \
	-d "amount=2000" \
	-d "card[number]=4100000000000000" \
	-d "card[expiry][month]=08" \
	-d "card[expiry][year]=2018" \
	-d "card[code]=123"
```

If the request is successful (status code in the `200-299` range) returns:

```js
{
	transaction: {
		id: String
	}
}
```

In the event of a [processing error](https://github.com/paylike/processing-errors):

```js
{
	code: Number,
	message: String,
	client: Boolean,
	merchant: Boolean,
	response: {
		transaction: {
			id: String
		}
	}
}
```

## Tokenize a card for later use:

```sh
$ curl 'https://gateway.paylike.io/cards' \
	-d "key=<public-key>" \
	-d "number=4100000000000000" \
	-d "expiry[month]=08" \
	-d "expiry[year]=2018" \
	-d "code=123"
```

If the request is successful (status code in the `200-299` range) returns:

```js
{
	card: {
		id: String
	}
}
```

In the event of a [processing error](https://github.com/paylike/processing-errors):

```js
{
	code: Number,
	message: String,
	client: Boolean,
	merchant: Boolean,
	response: {
		card: {
			id: String
		}
	}
}
```

## Validation

Before sending of data to the gateway you should check the data validity. If
the data is not valid the gateway will respond with a code `400` and a status
message (HTTP status header) giving a more specific reason.

Please make sure the data conforms to the following:

- `currency` must be the 3 letter code of a [supported currency](https://github.com/paylike/currencies)
- `amount` must be an integer above zero
- `card[number]` must be a string of 12-19 digits
- `card[expiry][month]` must be an integer of 1-12
- `card[expiry][year]` must be a four digit year
- `card[code]` must be three digits string
