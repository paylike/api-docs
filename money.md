# Money in programming

How to deal with monetary values. [TL;DR](#conclusion).

Monetary values are problematic for (at least) three reasons:

- [Floats](#floats)
- [Currencies](#currencies)
- [I18n (internationalization)](#i18n)
- [System compatibility (API design)](#system-compatibility-api-design)

Follow [these best practices](#best-practices) to get a head start.

That is why monetary values are often processed, stored and inputted into APIs
as minor units.

## Floats

Floats may cause issues because of the way computers represent them internally.
[Read more about that here](https://floating-point-gui.de) and
[here](https://0.30000000000000004.com). For instance:

```js
0.1 + 0.2 === 0.3
// false

1 - 0.9
// 0.09999999999999998

// However
1 + 2 === 3
// true
```

## Currencies

If you are used to USD or GBP you might (incorrectly) assume that

- ~All currencies have a major unit (e.g. dollars) and a minor unit (e.g.
  cents)~
- ~100 minor units are equal to 1 major unit~
- ~This never changes~

None are true. For instance the JPY does not have a minor unit (in most
systems), for the BHD 1 of the major unit is equal to 1000 of the minor unit,
and the HUF is moving from having a minor unit to not having one.

## I18n

Formatting (decimal) numbers is hard. Formatting monetary values is worse.

You have to consider:

- Numbers are formatted differently according to language and region (locale)
- Some currencies have symbols such as "€" for the euro and "¥" for the Japanese
  yen
- Some, like the Danish kroner (DKK), does not have a dedicated symbol, but is
  often shown with "kr."
- Depending on the language (and region) both the symbol and its position may
  vary

## System compatibility (API design)

Systems (and their APIs) may have different practices when it comes to
representing and validating monetary values. Some use a static map of currencies
and exponents, some accept floats, and some allow specifying an exponent. The
accepted precision may also vary. These could also change over time with or
without explicit versioning.

When integrating with other systems, consider their approach for representation
and changes, and if conversion is needed. Assuming synchronization of exponents
between systems may prevent or cause bugs during changes.

For API design, which format your system accepts as valid monetary values,
consider changes (e.g. versioning), explicit or implicit exponents, and
precisions.

## Best practices

### Represent monetary values as minor units

Convert your monetary value into minors at least before doing arithmetic.

If you deal with very large amounts you may hit the upper limit for integers.
Conside something like `BigInt` in JavaScript or similar.

Dealing with minor units and inputting only integers as input, the following
operations can be considered safe (always result in an integer): addition,
subtraction and multiplication.

```js
// integers only

var sourceWallet = 650000
var targetWallet = 12699
var transactionAmount = 20000
var fee = 100

sourceWallet += -transactionAmount - fee
targetWallet += transactionAmount - fee

var feesCollected = fee * 2
```

Should you do computations that will inevitably result in floats (e.g. currency
conversion) you should conclude your computation with a rounding.

Operations with non-integers or division should make you think twice. Take care
not to make a mistake such as the following:

```js
// wrong

var billAmount = 10000
var participantsCount = 3

var amountPerParticipant = Math.round(billAmount / participantsCount)
```

In this case, the waiter will be "off-by-one" because they were only paid
\$99.99. Someone has to pay the last penny as it cannot be split any further. A
solution could be:

```js
// right

var billAmount = 10000
var participantsCount = 3

// Add tips of 20 %
var estimatedTips = billAmount * 0.2

var amountPerParticipant = Math.ceil(
  (billAmount + estimatedTips) / participantsCount
)
var tips = amountPerParticipant * 3 - billAmount
```

The tips are now being used as a buffer and the outcome is guaranteed to result
in whole minor units (integers). `ceil` is used to avoid a situation of negative
tips with very small amounts.

Why not use a decimal type or other built-in language type? You might be able to
do that, at least for your database storage, but options and details vary
greatly between languages so it will require a higher awareness from your team
especially if you have code in more than one language. Using such, however, may
disguise cases such as the waiter example instead of explicitly taking care of
it. In the real world, money is rarely a simple numeric value.

#### Converting to minor units

Before you can represent money in minor units, you need to be able to convert
back and forth. To do so, you need to know how many minor units comprise a major
unit. Most systems (including banking ones) restrict this to be a factor of 10
depending on the currency. This is called the "exponent". The exponent is the
factor of 10 that a single major is larger than a single minor. For instance the
USD has an exponent of 2 meaning you need `10^2 = 100` pennies to have a dollar.
The JPY has an exponent of 0 meaning 1 minor unit is equal to 1 major unit - in
other terms, they are the same.

Here is a table showing some examples of how exponents work:

|                  | Major (float) | Minor (integer) |
| ---------------- | ------------: | --------------: |
| EUR (exponent 2) |        100.99 |           10099 |
| EUR (exponent 2) |           5.5 |             550 |
| JPY (exponent 0) |           100 |             100 |
| BHD (exponent 3) |       100.155 |         1001555 |
| HUF (exponent 2) |        100.50 |           10050 |
| HUF (exponent 0) |        100.50 |             100 |

Once you have decided on the exponent, converting from and to minor units is
trivial:

```js
var toMinor = (major, exponent) => round(major * pow(10, exponent))

var toMajor = (minor, exponent) => minor / pow(10, exponent)

var changeExponent = (minor, source, target) =>
  toMinor(toMajor(minor, source), target)
// or
var changeExponent = (minor, source, target) =>
  round(minor * pow(10, target - source))
```

Notice that there is a potential loss of data in the `toMinor` function if you
accept input that is more precise than what the exponent can accomodate. This is
one reason many financial systems accept input only in whole minor units.

### I18n

Unless you intend to make this your primary project, do not try to localize the
display of a monetary value yourself but use a library.

For JavaScript take a look at
[`Number.prototype.toLocaleString()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/toLocaleString):

```js
var amount = 1500.95

amount.toLocaleString('da-DK', {
  style: 'currency',
  currency: 'DKK',
})
// 1.500,95 kr.

amount.toLocaleString('en-US', {
  style: 'currency',
  currency: 'DKK',
})
// DKK 1,500.95
```

In PHP the [`NumberFormatter`](https://php.net/numberformatter) class is
available.

Most languages have some built-in support for formatting money and many more
libraries exist to help out on this.

If correct formatting is impossible consider a fallback of `CCC #.##` (e.g. "USD
100.00").

### API design

Make sure the exponent is not assumed.

Ask explicitly for the currency close to the amount and represent the amount
either as a float or with an explicit exponent. Make sure to validate amounts
for precision (exponent value or decimal places).

In JSON, use a format like `{currency, exponent, value}` accepting only
integers. Return specific errors if the currency, or precision is not supported.
If the exponent is replaced by a static map, make sure the map is versioned
alongside the API.

## Conclusion

Represent money in minor units (use conversion functions from
[converting to minor units](#converting-to-minor-units)) as integers and use
libraries for fomatting according to currency and locale (convert to major
before passing the amount).
