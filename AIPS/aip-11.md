<pre>
  AIP: 11
  Title: Upgrade of Transaction Protocol
  Authors: François-Xavier Thoorens <fx@ark.io>, dafty
  Status: Draft
  Type: Standards Track
  Created: 2017-09-25
  Last Update: 2017-10-23
</pre>
![Ark Improvement Proposals](https://cdn-images-1.medium.com/max/2000/1*vD5i8JJVGjvIAdOxSi-iFA.png)

Abstract
========
In order to move forward with the ark blockchain for future evolution, the transaction protocol needs to be upgraded.
Comment thread: https://github.com/ArkEcosystem/AIPs/issues/11

Motivation
==========
The current status of protocol has several limitations that prevent from future development of services as envisioned on the roadmap:
- impossibility to have cost efficient deserialisation, preventing from scalability of the network
- `retention-replay` attack: a third part can prevent transaction from hitting blockchain inducing the transaction author to create a new transaction. However the transaction is still valid and could be included in the blockchain whenever in the future
- impossible to upgrade transactions without hard fork / no support for private transactions
- impossible to scale fees following the market price


Rationale
=========
- Fast serialisation and deserialisation
- Compatible with former version of protocol (0xff head flag)
- Lay groundwork for future upgrade without the need of a fork
- Able to validate custom private transactions
- Prevent “retention-replay” attack scheme

Specifications
==============

**All numbers are stored as unsigned Least Endian forms.**

General form (total header size excluding vendorfield: 47 bytes)
----------------------------------------------------------------

| Description       | Size (bytes)  | Example                                                              |
| -------------     | ------------- | :-------                                                             |
| head              | 1             | 0xff                                                                 |
| version           | 1             | 0x02                                                                 |
| network           | 1             | 0x17                                                                 |
| type              | 1             | 0x00                                                                 |
| nonce/timestamp   | 4             | 0x0x0293fa00                                                         |
| sender public key | 33            | 0x025f81956d5826bad7d30daed2b5c8c98e72046c1ec8323da336445476183fb7ca |
| fee               | 8             | 0x00000000000e7468                                                   |
| vendorfield length| 1             | 0x0e                                                                 |
| vendorfield       | 0-255         | 0x7468697320697320612074657374                                       |
| payload           | variable      | see details below                                                    |

In application, nonce is equivalent as formerly known timestamp and can be used
Version 0x01 will be notifying former use of protocol to serialise the tx before computing signatures.

Payloads
--------
The payload is defined according to the type of the transaction. This may be changed with a version change in the future.


**Type 0 (transfer, 33bytes)**

| Description       | Size (bytes)  | Example                                                              |
| -------------     | ------------- | :-------                                                             |
| amount            | 8             | 0x8096980000000000                                                   |
| expiration        | 4             | 0x04000000 (0x00000000 = no expiration)                              |
| recipient address | 21            | 0x171dfc69b54c7fe901e91d5a9ab78388645e2427ea                         |

Expiration is the timestamp after which it cannot be included in a block.
In other words, if the block timestamp is strictly superior to the transaction expiration timestamp, the block is invalid.


**Type 1 (second signature registration, 33 bytes)**

| Description                    | Size (bytes)  | Example                                                              |
| -------------                  | ------------- | :-------                                                             |
| public key of second signature | 33            | 0x025f81956d5826bad7d30daed2b5c8c98e72046c1ec8323da336445476183fb7ca |


**Type 2 (delegate registration, 1-256 bytes)**

| Description       | Size (bytes)  | Example                                                              |
| -------------     | ------------- | :-------                                                             |
| length            | 1             | 0x80 (minimum 0x03)                                                  |
| username (utf8)   | 3-255         | 0x6669786372797074                                                   |


**Type 3 (vote, 1 + 34N bytes)**

| Description       | Size (bytes)  | Example                                                              |
| -------------     | ------------- | :--------                                                             |
| number of votes   | 1             | 0x02                                                                 |
| votes             | 34*N          | 0x0103a02b9d5fdd1307c2ee4652ba54d492d1fd11a7d1bb3f3a44c4a05e79f19de933000380728436880a0a11eadf608c4d4e7f793719e044ee5151074a5f2d5d43cb9066 |

`Votes` is a concatenation of votes of the form
- Vote flag (0x01 if vote, 0x00 if unvote)
- Public key of the delegate
In the case of ark only there is only one possible vote, so the payload is maximum 69 bytes.


**Type 4 (multisignature registration, 3 + 33N bytes)**

| Description         | Size (bytes)  | Example                                                              |
| -------------       | ------------- | :-------                                                             |
| minimum             | 1             | 0x02                                                                 |
| number of signatures| 1             | 0x03                                                                 |
| lifetime (hours)    | 1             | 0x1a                                                                 |
| Public keys         | 33*N          | 0x03a02b9d5fdd1307c2ee4652ba54d492d1fd11a7d1bb3f3a44c4a05e79f19de9330380728436880a0a11eadf608c4d4e7f793719e044ee5151074a5f2d5d43cb906603a02b9d5fdd1307c2ee4652ba54d492d1fd11a7d1bb3f3a44c4a05e79f19de933|


`Public keys` are the concatenation of public keys of the people signing on this account.


**Type 5 (IPFS, 1-256 bytes)**

| Description         | Size (bytes)  | Example                                                              |
| -------------       | ------------- | :-------                                                             |
| Length of dag       | 1             | 0xea                                                                 |
| dag                 | 0-255         | 0x6669786372797074                                                   |


**Type 6 (timelock transfer 34 bytes)**

| Description         | Size (bytes)  | Example                                                              |
| -------------       | ------------- | :-------                                                             |
| amount              | 8             | 0x8096980000000000                                                   |
| timelock type       | 1             | 0x00 (timestamp), 0x01 (blockheight)                                                                 |
| timelock            | 4             | 0x0087e5a8                                                           |
| recipient address   | 21            | 0x171dfc69b54c7fe901e91d5a9ab78388645e2427ea                         |

`Timelock type` defines different types for `timelock` value field (current supported types are `0:for timestamp type` timelock and `1:for blockheight type` timelock value). If timelock type=0, timelock value shall be specified with timestamp, and if type=1, timelock value shall be specified with blockheight.

- TimeLockType=0, timelock value=timestamp: transaction will be forged when timestamp is passed
- TimeLockType=1, timelock value=blockHeight: transaction will be forged when blockchain reaches height specified at timelock value for blockHeight


**Type 7 (multipayment, 2-65546 bytes)**

| Description         | Size (bytes)  | Example                                                              |
| -------------       | ------------- | :-------                                                             |
| N outputs           | 2             | 0x0021                                                               |
| amount1             | 8             | 0x8096980000000000                                                   |
| address1            | 21            | 0x171dfc69b54c7fe901e91d5a9ab78388645e2427ea                         |
| ...                 | ...           | ...                                                                  |
| amountN             | 8             | 0x8096980000000000                                                   |
| addressN            | 21            | 0x171dfc69b54c7fe901e91d5a9ab78388645e2427ea                         |

- N > 1 (ideally fees should be higher than type 0 if N < 2)
- Max 2259 possible outputs. first versions of network will cap this max


**Type 8 (delegate resignation)**

| Description         | Size (bytes)  | Example                                                              |
| -------------       | ------------- | :-------                                                             |
| No payload          | 0             | .                                                                    |

basically this will make:
- impossible to vote for this delegate anymore
- impossible to be selected as active delegate

Fees calculation
----------------

A new process will be used where the fee is related to:
- Type of the transaction
- Size of the serialised transaction

The calculation formula is `Fee = (T+S) * C`
- T: minimum offset byte depending on transaction type, defined by the network (byte)
- S: size of the serialised transaction (byte)
- C: constant (Ark/byte) defined by the delegate including the transaction in his forged block

For instance, for transfer we could have T = 0 byte, C = 0.0001 Ark/byte. For a classic transfer transaction with empty VendorField S = 80 bytes, hence the fee is 0.008 Ark.

For a vote we could have T = 100 bytes, C=0.0001 Ark/byte, S = 82 bytes, fee = 0.0182 Ark

T is here to account for extra processing power to process special transaction whose transfer value is null, and thus reducing economic interest to spam the network.
