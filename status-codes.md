# Status codes in responses

We use the following status codes:

- **2xx** Success

	- **200** Ok

		All successful `GET` requests.

	- **201** A new thing was created

		All successful `POST` requests.

	- **204** No content is returned

		All `PUT` and `DELETE` requests.

- **4xx** Client error

	- **400** Bad (invalid) request

		- Missing input data
		- Malformed data
		- Failed validation
		- Inconsistencies

	- **401** Unauthorized

		You need to provide credentials (an app's API key).

	- **403** Forbidden

		You are correctly authenticated but do not have access.

	- **409** Conflict

		Everything you submitted was fine at the time of validation, but
		something changed in the meantime and came into conflict with this
		(e.g. double-capture).

- **5xx** Server error

	If you ever receive one of these, we are deeply sorry. We screwed up
	somewhere and hope you will forgive us some day. Please talk to us about
	this so we can fix it.
