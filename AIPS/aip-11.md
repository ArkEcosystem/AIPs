<pre>
  AIP: 11
  Title: Upgrade of Transaction Protocol
  Authors: François-Xavier Thoorens fx.thoorens@ark.io, dafty
  Status: Draft
  Type: Standards Track
  Created: 2017-09-25
  Last Update: 2019-08-31
</pre>

![Ark Improvement Proposals](https://cdn-images-1.medium.com/max/2000/1*vD5i8JJVGjvIAdOxSi-iFA.png)

# Abstract

In order to move forward with the ARK blockchain for future evolution, the transaction protocol needs to be upgraded.
Comment thread: https://github.com/ArkEcosystem/AIPs/issues/11

# Motivation

The current status of protocol has several limitations that prevent from future development of services as envisioned on the roadmap:

- impossibility to have cost efficient deserialisation, preventing from scalability of the network
- `retention-replay` attack: a third party can prevent transaction from hitting blockchain inducing the transaction author to create a new transaction. However the transaction is still valid and could be included in the blockchain whenever in the future
- impossible to upgrade transactions without hard fork / no support for private transactions
- impossible to scale fees following the market price

# Rationale

- Fast serialisation and deserialisation
- Lay groundwork for future upgrade without the need of a fork
- Able to validate custom private transactions
- Prevent “retention-replay” attack scheme

# Specifications

**All numeric values are stored in unsigned little-endian form.**

The preferred signature scheme will be Schnorr. Schnorr signatures are fixed 64 bytes long while ECDSA signatures vary between 70-72 bytes. The latter will still be supported just fine, although discouraged.

## General form (total header size excluding vendorfield: 59 bytes)

| Description        | Size (bytes) | Example                                                              |
| ------------------ | ------------ | :------------------------------------------------------------------- |
| header             | 1            | 0xff                                                                 |
| version            | 1            | 0x02                                                                 |
| network            | 1            | 0x17                                                                 |
| typeGroup          | 4            | 0x01                                                                 |
| type               | 2            | 0x00                                                                 |
| nonce              | 8            | 0x00293fa0                                                           |
| sender public key  | 33           | 0x025f81956d5826bad7d30daed2b5c8c98e72046c1ec8323da336445476183fb7ca |
| fee                | 8            | 0x00000000000e7468                                                   |
| vendorfield length | 1            | 0x0e                                                                 |
| vendorfield        | 0-255        | 0x7468697320697320612074657374                                       |
| asset              | variable     | see details below                                                    |

Version 0x01 is used for the legacy V1 signature format. Starting at 0x02 all transactions will follow
the new signature format and the transaction timestamp will be replaced by a nonce.

## Type & TypeGroup
See [AIP29](https://github.com/ArkEcosystem/AIPs/blob/master/AIPS/aip-29.md#motivation)

## Nonce
A monotonically increasing number derived from the sender wallet.

## Assets

The asset is defined according to the type of the transaction. This may be changed with a version change in the future.

**Type 0 (transfer, 33bytes)**

| Description       | Size (bytes) | Example                                      |
| ----------------- | ------------ | :------------------------------------------- |
| amount            | 8            | 0x8096980000000000                           |
| expiration        | 4            | 0x04000000 (0x00000000 = no expiration)      |
| recipient address | 21           | 0x171dfc69b54c7fe901e91d5a9ab78388645e2427ea |

Expiration is the blockchain height at or after which the transaction cannot be included in a block.
In other words, if the block height is equal or bigger than the transaction expiration, the transaction is invalid.

**Type 1 (second signature registration, 33 bytes)**

| Description                    | Size (bytes) | Example                                                              |
| ------------------------------ | ------------ | :------------------------------------------------------------------- |
| public key of second signature | 33           | 0x025f81956d5826bad7d30daed2b5c8c98e72046c1ec8323da336445476183fb7ca |

**Type 2 (delegate registration, 1 + 3-20 bytes)**

| Description     | Size (bytes) | Example                           |
| --------------- | ------------ | :-------------------------------- |
| length          | 1            | 0x10 (minimum 0x03, maximum 0x14) |
| username (utf8) | 3-20         | 0x6669786372797074                |

**Type 3 (vote, 34 bytes)**

| Description     | Size (bytes) | Example                                                                                                                                    |
| --------------- | ------------ | :----------------------------------------------------------------------------------------------------------------------------------------- |
| vote           | 34            | 0x0103a02b9d5fdd1307c2ee4652ba54d492d1fd11a7d1bb3f3a44c4a05e79f19de933000380728436880a0a11eadf608c4d4e7f793719e044ee5151074a5f2d5d43cb9066 |

A `Vote` consists of:

- Vote flag (0x01 if vote, 0x00 if unvote)
- Public key of the delegate

**Type 4 (multisignature registration, 1 + 33N bytes)**

| Description          | Size (bytes) | Example                                                                                                                                                                                                  |
| -------------------- | ------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| min participants     | 1            | 0x02 (minimum 0x01, maximum 0x10)                                                                                                                                                         
| public keys          | 33\*N        | 0x03a02b9d5fdd1307c2ee4652ba54d492d1fd11a7d1bb3f3a44c4a05e79f19de9330380728436880a0a11eadf608c4d4e7f793719e044ee5151074a5f2d5d43cb906603a02b9d5fdd1307c2ee4652ba54d492d1fd11a7d1bb3f3a44c4a05e79f19de933 |

`Public keys` is the concatenation of the public key of all participants of the multi signature wallet.
Limited to a maximum of 16 participants for now.

Also see [AIP18](https://github.com/ArkEcosystem/AIPs/blob/master/AIPS/aip-18.md)

**Type 5 (IPFS, 0-255 bytes)**

| Description   | Size (bytes) | Example            |
| ------------- | ------------ | :----------------- |
| dag           | 0-255        | 0x6669786372797074 |

**Type 6 (multipayment, 2-1900544 bytes)**

| Description | Size (bytes) | Example                                      |
| ----------- | ------------ | :------------------------------------------- |
| N payments  | 2            | 0x0021                                       |
| amount1     | 8            | 0x8096980000000000                           |
| address1    | 21           | DFJ5Z51F1euNNdRUQJKQVdG4h495LZkc6T |
| ...         | ...          | ...                                          |
| amountN     | 8            | 0x8096980000000000                           |
| addressN    | 21           | DFJ5Z51F1euNNdRUQJKQVdG4h495LZkc6T |

- N > 1 (ideally fees should be higher than type 0 if N < 2)
- Maximum of possible payments per transaction: 2^16 or 65536. Initially capped to 500 for performance
reasons.

**Type 7 (delegate resignation)**

| Description | Size (bytes) | Example |
| ----------- | ------------ | :------ |
| No payload  | 0            | .       |

The sender must be a delegate. Once forged the delegate is irreversibly deactivated:
- no longer possible to vote for this delegate
- no longer treated as an active delegates, regardless of vote balance

**Type 8 (htlc lock 66 bytes)**

| Description       | Size (bytes) | Example                                                            |
| ----------------- | ------------ | :----------------------------------------------------------------- |
| amount            | 8            | 0x8096980000000000                                                 |
| secretHash        | 32           | 0x0f128d401958b1b30ad0d10406f47f9489321017b4614e6cb993fc63913c5454 |
| expirationType    | 1            | 0x01(network epoch timestamp), 0x02 (block height)                 |
| expirationValue   | 4            | 0x80969800                                                         |
| recipient         | 21           | DFJ5Z51F1euNNdRUQJKQVdG4h495LZkc6T                                 |

Also see [AIP102](https://github.com/ArkEcosystem/AIPs/blob/master/AIPS/aip-102.md)

**Type 9 (htlc claim 64 bytes)**

| Description       | Size (bytes) | Example                                                            |
| ----------------- | ------------ | :----------------------------------------------------------------- |
| lockTransactionId | 32           | 0x025f81956d5826bad7d30daed2b5c8c98e72046c1ec8323da336445476183fb7 |
| unlockSecret      | 32           | 0x025f81956d5826bad7d30daed2b5c8c98e72046c1ec8323da336445476183fb7 |

Also see [AIP102](https://github.com/ArkEcosystem/AIPs/blob/master/AIPS/aip-102.md)

**Type 10 (htlc refund 32 bytes)**

| Description       | Size (bytes) | Example                                                            |
| ----------------- | ------------ | :----------------------------------------------------------------- |
| lockTransactionId | 8            | 0x025f81956d5826bad7d30daed2b5c8c98e72046c1ec8323da336445476183fb7 |

Also see [AIP102](https://github.com/ArkEcosystem/AIPs/blob/master/AIPS/aip-102.md)

## Dynamic Fees calculation

> see [AIP-16](https://github.com/ArkEcosystem/AIPs/blob/master/AIPS/aip-16.md) for more detailed information.

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
