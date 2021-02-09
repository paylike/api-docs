# Getting started

Let's create an account, invite a user to, create a transaction and capture
that transaction using only the API.

1. Register your app

	```shell
	curl -i https://api.paylike.io/apps \
		-d name="Pizzkebabhouse"
	```

	Should return something like:

	```json
	{
		"app": {
			"id": "6017ff785b1f7e3d1ec02139",
			"name": "Pizzakebabhouse",
			"key": "93424265-0617-484d-8486-bc7c69b50272"
		}
	}
	```

	Save these details.

2. Create a merchant account

	Using the `key` from above we are now going to create a merchant account:

	```shell
	curl -i https://api.paylike.io/merchants \
		-u :93424265-0617-484d-8486-bc7c69b50272 \
		-d test="yes" \
		-d name="My Pizzakebabhouse" \
		-d email="my-contact-pizzakebabhouse@outlook.com" \
		-d currency="NOK" \
		-d website="https://Pizzakebabhouse.no" \
		-d descriptor="My pizzakebabhouse" \
		-d company[country]="Norge"
	```

	Should return something along the lines of:

	```json
	{
		"merchant": {
			"id": "6017ff785b1f7e3d1ec02139",
			"key": "93424265-0617-484d-8486-bc7c69b50272",
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

	Still using your app for authentication and you new account's ID, you can
	now invite your email to get access via https://app.paylike.io

	```shell
	curl -i https://api.paylike.io/merchants/6017ff785b1f7e3d1ec02139/users \
		-u :93424265-0617-484d-8486-bc7c69b50272 \
		-d email="my-pizzakebabhouse@outlook.com"
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
	curl -i https://api.paylike.io/transactions/6017ff785b1f7e3d1ec02139/captures \
		-u :93424265-0617-484d-8486-bc7c69b50272
		-d amount=100
	```

6. Explore

	See what else you can do in the [README file](README.md).
