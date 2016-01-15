# Getting started

Let's create an account, invite a user to, create a transaction and capture
that transaction using only the API.

1. Register your app

	```shell
	curl -i https://api.paylike.io/apps \
		-d name="My new app"
	```

	Should return something like:

	```json
	{
	"app": {
		"id": "56983e109967856f4ea4aa71",
		"name": "My new app",
		"key": "416766b7-6d64-446c-a918-6df8e6e7cbf4"
	}
	```

	Save these details.

2. Create a merchant account

	Using the `key` from above we are now going to create a merchant account:

	```shell
	curl -i https://api.paylike.io/merchants \
		-u :416766b7-6d64-446c-a918-6df8e6e7cbf4 \
		-d test="yes" \
		-d name="My webshop" \
		-d email="my-contact-email@example.com" \
		-d currency="USD" \
		-d website="https://example.com" \
		-d descriptor="My webshop" \
		-d company[country]="DK"
	```

	Should return something along the lines of:

	```json
	{
		"merchant": {
			"id": "569843df9967856f4ea4aa77",
			"key": "f4f516ab-f253-4f76-a924-67b6486d42e4",
			...
		}
	}
	```

	The `id` value is used for referencing the merchant account later.

	The `key` attribute is the public key used for creating new transactions.

	Follow the examples at https://github.com/paylike/sdk to set up the
	frontend for payments and create a test sale using the card number `4100
	0000 0000 0000`.

3. Invite your email to the account (optional)

	Still using your app for authentication and you new account's id, you can
	now invite your email to get access via https://app.paylike.io

	```shell
	curl -i https://api.paylike.io/merchants/569843df9967856f4ea4aa77/users \
		-u :416766b7-6d64-446c-a918-6df8e6e7cbf4 \
		-d email="my-email@example.com"
	```

4. Fetch all transactions

	If you have set up a frontend and created some test sales, we can pull the
	list of those:

	```shell
	curl -i https://api.paylike.io/merchants/569843df9967856f4ea4aa77/transactions?limit=10 \
		-u :416766b7-6d64-446c-a918-6df8e6e7cbf4
	```

	This will give you all sorts of information about the transaction, but
	make a note of `id` and `amount`.

5. Capture a transaction

	Finally, let's capture the transaction in its whole:

	```shell
	curl -i https://api.paylike.io/transactions/569843df9967856f4ea4bb59/captures \
		-u :416766b7-6d64-446c-a918-6df8e6e7cbf4
		-d amount=100
	```

6. Explore

	See what else you can do in the [README file](README.md).
