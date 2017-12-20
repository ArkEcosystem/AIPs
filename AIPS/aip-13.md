<pre>
  AIP: 13
  Layer: Applications
  Title: URI Scheme
  Author: William Dens <denswilliam1@gmail.com>
  Status: Draft
  Type: Standards Track
  Created: 2017-11-22
  Last Update: 2017-11-22
</pre>

## Abstract

This AIP proposes a URI scheme for making Ark payments. A Uniform Resource Identifier (URI) scheme is a string of characters used to identify a resource.


## Motivation

The purpose of this URI scheme is to allow users to make payments by sharing a URI and thus, reducing errors when sending transactions while making them faster and painless for users. This URI could be a link or a QR code and may be clicked or scanned from any devices.


## Rationale

### Payment identifiers

Current best practices are that a unique address should be used for every transaction. Therefore, a URI scheme should not represent an exchange of personal information, but a one-time payment.

### Accessibility

Should someone from the outside happen to see such a URI, the URI scheme already gives a self-explanatory description. A quick search should then do the rest to help them find the resources needed to make their payment.


## Specifications

### Handling rules

Ark clients must not act on URIs without getting the user's authorization. They should require the user to manually approve each payment individually, though in some cases they may allow the user to automatically make this decision.

### OS integration

Ark clients should register themselves as the handler for the "ark:" URI scheme by default, if no other handler is already registered. If there is already a registered handler, they may prompt the user to change it once when they first run the client. Official Ark desktop client is build upon electron, which [protocol module](https://github.com/electron/electron/blob/master/docs/api/protocol.md) allow a convenient way to register and intercept a custom protocol URI scheme.

### General Format

Ark URIs follow the general format for URIs as set forth in [RFC 3986](https://www.ietf.org/rfc/rfc3986.txt). The path component consists of a Ark address, and the query component provides additional payment options.

Elements of the query component may contain characters outside the valid range. These must first be encoded according to UTF-8, and then each octet of the corresponding UTF-8 sequence must be percent-encoded as described in RFC 3986.

### ABNF grammar

```
 arkuri           = "ark:" arkaddress [ "?" arkparams ]
 arkaddress       = *base58
 arkparams        = arkparam [ "&" arkparams ]
 arkparam         = [ amountparam / vendorfieldparam / otherparam ]
 amountparam      = "amount=" *digit [ "." *digit ]
 labelparam       = "label=" *qchar
 vendorfieldparam = "vendorField=" *qchar
 otherparam       = qchar *qchar [ "=" *qchar ]
```
Here, "qchar" corresponds to valid characters of an [RFC 3986](https://tools.ietf.org/html/rfc3986) URI query component, excluding the "=" and "&" characters, which this AIP takes as separators.

The scheme component ("ark:") is case-insensitive, and implementations must accept any combination of uppercase and lowercase letters. The rest of the URI is case-sensitive, including the query parameter keys.

### Query

#### Keys

- `<arkaddress>` : Ark recipient address encoded in Base58.
- `[?label=<label>]` : Recipient label string.
- `[?amount=<amount>]` : Amount in ARK (Ѧ).
- `[?vendorField=<vendorField>]` : Vendor field string.
- `[?otherparam=<otherparam>]` : Reserved for future extensions.

#### Ark address

The Ark address must be valid and encoded in Base58.

#### Label

Label must be a UTF-8 encoded string and should label `<arkaddress>`. *e.g* `label=John-Doe`

#### Amount

If an amount is provided, it must be specified in decimal ARK (Ѧ). All amounts must contain no commas and use a period (.) as the separating character to separate whole numbers and decimal fractions. *e.g.* `amount=25.00` or `amount=25` is treated as 25 Ark, and `amount=25,000.00` is considered invalid.

Ark clients may display the amount in any format that is not misleading to the user. They should choose a format that is foremost least confusing, and only after that most reasonable given the amount requested. For example, so long as the majority of users work in ARK (Ѧ) units, values should always be displayed in ARK (Ѧ) by default.

#### Vendor field

Vendor field must be a UTF-8 encoded string. *e.g.* `vendorField=Message%20for%20Ark`

#### Other param

Allow URI schemes extend by custom parameters. Parameter's value must be a UTF-8 encoded string and must not conflict with a reserved key (`amount`, `label`, `vendorField`). *e.g.* `fromURL=mydomain.com` or `refId=57458`

## Appendix

### Simpler syntax

This section is non-normative and does not cover all possible syntax.

`ark:<address>[?amount=<amount>][?label=<label>][?vendor=<vendor>][?otherparam=<otherparam>]`

### Examples

Just the address

` ark:AePNZAAtWhLsGFLXtztGLAPnKm98VVC8tJ`

Address with amount

` ark:AePNZAAtWhLsGFLXtztGLAPnKm98VVC8tJ?amount=20.3`

Address with label and amount

` ark:AePNZAAtWhLsGFLXtztGLAPnKm98VVC8tJ?label=John-Doe?amount=20.3`

Address with label, amount and vendor field

` ark:AePNZAAtWhLsGFLXtztGLAPnKm98VVC8tJ?label=John-Doe?amount=20.3?vendorField=Message%20for%20Ark`

Address with custom parameters

` ark:AePNZAAtWhLsGFLXtztGLAPnKm98VVC8tJ?fromURL=ark.io`