<pre>
  AIP: 26
  Title: URI Scheme Improvements
  Authors: vmunich
  Status: Draft / Open Discussion
  Type: Standards Track
  Created: 2018-12-21
  Last Update: 2019-01-06
</pre>

History
========
- 2018-12-21 initial content (@vmunich)
- 2019-01-06 renamed `send-transaction` action to `transfer` (@vmunich)
- 2019-01-06 ABNF grammar (@ItsANameToo)

Abstract
========

This AIP proposes a replacement for AIP 13 (URI Scheme), intending to support AIP11 (Upgrade of Transaction Protocol) dynamic fees, multiple networks, and to provide means of direct communication with specific sections of wallets, relays and services.

Motivation
==========

The current URI Scheme is limited to Type 0 (Transfer) transactions. The convenience of representing transfer requests by standard URIs has been a major part in building apps in the Ark Ecosystem. Bringing a similarly convenient mechanism to all transaction types would empower developers and services to precisely direct end-users to specific sections of the Ark Wallet, and offer a more pleasant user experience.

Rationale
=========
### Single Purpose URIs

An URI should have a single purpose, meaning that an URI should not contain instructions to perform two or more actions at the same time.

### Network Endpoint

In order to support many different sidechains, clients need a mechanism to easily communicate with other networks. By using the proposal endpoints and parameters, clients can support adding and communicating with different networks, by  following instructions in the URI.

### Relay Parameter

When the optional `relay` parameter is populated, a transaction can be sent through a specific relay node, bypassing the currently connected peer. This is particularly useful in ownership verification, IoT and hardware based applications, such as:

- Services that require a transaction to be sent for verification purposes, no longer have to wait for transactions to confirm. Clients can be instructed to post transactions directly to their endpoint. As a result, wallet verification becomes instantaneous.

- On Ark based PoS terminals, clients can be instructed to post the transaction directly to the terminal's endpoint. This allows the PoS terminal to verify that the transaction before it is actually forged in a block, enabling queues to move faster.

- Different NFC tags can instruct clients to post transactions to different relay nodes, based on their own requirements

- Local relay nodes could be used as a caching layer for offline transactions.

Specification
=========

### General Format

Ark URIs follow the general format for URIs as set forth in [RFC 3986](https://www.ietf.org/rfc/rfc3986.txt). The path component consists of an Ark address, and the query component provides additional instructions.

Elements of the query component may contain characters outside the valid range. These must first be encoded according to UTF-8, and then each octet of the corresponding UTF-8 sequence must be percent-encoded as described in RFC 3986.

### ABNF grammar

```
arkuri = "ark:" body
body = recipient / actions
recipient = *base58
amount = *digit [ "." *digit ]
actions = networkAction / transferAction / voteAction / delegateAction / signAction / otherAction
networkAction = "add-network?" networkParams
networkParams = "name=" *qchar "&seedServer=" *qchar ["&description=" *qchar]
transferAction = "transfer?" transferParams *[ "&" defaultParam ]
transferParams = "recipient=" recipient "&amount=" amount *[ "&" transferParam ]
transferParam = [ ("vendorField=" *qchar) / ("label=" *qchar) / otherParams ]
voteAction = "vote?" voteParams *[ "&" defaultParam ]
voteParams = "username=" *qchar
delegateAction = "register-delegate?" delegateParams *[ "&" defaultParam ]
delegateParams = "username=" *qchar
signAction = "sign-message?" signParams
signParams = "message=" *qchar
otherAction = qchar "?" *[ "&" otherParams ]
otherParams = qchar *qchar [ "=" *qchar ]
defaultParam = [ ("fee=" amount) / ("relay=" *qchar) / ("nethash=" *qchar) ]
```
Here, "qchar" corresponds to valid characters of an [RFC 3986](https://tools.ietf.org/html/rfc3986) URI query component, excluding the "=" and "&" characters, which this AIP takes as separators.

The scheme component ("ark:") is case-insensitive, and implementations must accept any combination of uppercase and lowercase letters. The rest of the URI is case-sensitive, including the query parameter keys.

## Queries

### Add Network

Instructs the wallet to add a new network.

#### Endpoint

`ark:add-network`

#### URI Parameters


| Parameter   | Description                     | Type          | Required           |
|-------------|---------------------------------|---------------|--------------------|
| name        | The name of this network        | UTF-8 Encoded | :white_check_mark: |
| seedServer  | The seed server of this network | UTF-8 Encoded | :white_check_mark: |
| description | The description of this network | UTF-8 Encoded | :x:                |
| otherparam  | Reserved for future extensions  | UTF-8 Encoded | :x:                |

#### Example

`ark:add-network?name=Mainnet&seedServer=https%3A%2F%2Fexplorer.ark.io%3A8443`

#### Specification

This query should be used to add new networks to clients. Clients should query the `/api/v2/node/configuration` API endpoint of the seed server that is provided, which will return the following data:
```
{
  "data": {
    "nethash": "6e84d08bd299ed97c212c886c98a57e36545c8f5d645ca7eeae63a8bd62d8988",
    "token": "ARK",
    "symbol": "Ѧ",
    "explorer": "https://explorer.ark.io",
    "version": 23,
    "ports": {},
    "constants": {
      "height": 6600000,
      "reward": 200000000,
      "activeDelegates": 51,
      "blocktime": 8,
      "block": {
        "version": 0,
        "maxTransactions": 150,
        "maxPayload": 6300000
      },
      "epoch": "2017-03-21T13:00:00.000Z",
      "fees": {}
    },
    "feeStatistics": []
  }
}
```
The required fields should be pre-filled using the data requested, and the end-user should be prompt to confirm the addition of the new network.


### Send Transaction

Instructs the wallet to send a transaction according to the given parameters.

#### Endpoint

`ark:transfer`

#### URI Parameters

| Parameter   | Description                                            | Type          | Required           |
|-------------|--------------------------------------------------------|---------------|--------------------|
| recipient   | The recipient of this transaction                      | Base58        | :white_check_mark: |
| amount      | The amount to be sent. It must be specified in decimal | Numeric       | :white_check_mark: |
| fee         | The fee amount to be used to send this transaction     | Numeric       | :x:                |
| vendorField | The smartbridge field                                  | UTF-8 Encoded | :x:              |
| relay       | The relay node to be used to send this transaction     | UTF-8 Encoded | :x:                |
| nethash     | The network to be used for this transaction            | String        | :x:                |
| label       | The contact name for this recipient on the client      | UTF-8 Encoded | :x:                |
| otherparam  | Reserved for future extensions                         | UTF-8 Encoded | :x:                |

#### Example

`ark:transfer?recipient=AUDud8tvyVZa67p3QY7XPRUTjRGnWQQ9Xv&amount=10&vendorField=Message%20for%20Ark`

#### Specification

- The `relay` parameter is used to instruct the client to send a `POST` request directly to the given relay, bypassing the peer that it was previously connected to.
- The `nethash` parameter is used to instruct the wallet to switch to a certain network. When this parameter is populated, clients should match the given nethash to an existing network with the same nethash. If no matching networks are found, clients should return an error stating that no matching networks were found.

### Vote

Prompts the wallet to vote for a given delegate

#### Endpoint

`ark:vote`

##### URI Parameters

| Parameter  | Description                                        | Type          | Required           |
|------------|----------------------------------------------------|---------------|--------------------|
| username   | The delegate name to vote for                      | String        | :white_check_mark: |
| fee        | The fee amount to be used to send this transaction | Numeric       | :x:                |
| relay      | The relay node to be used to send this transaction | UTF-8 Encoded | :x:                |
| nethash    | The network to be used for this transaction        | String        | :x:                |
| otherparam | Reserved for future extensions                     | UTF-8 Encoded | :x:                |

#### Example

`ark:vote?delegate=genesis_10&fee=0.10&relay=https%3A%2F%2Fexplorer.ark.io%3A8443`

#### Specification

- The `relay` parameter is used to instruct the client to send a `POST` request directly to the given relay, bypassing the peer that it was previously connected to.
- The `nethash` parameter is used to instruct the wallet to switch to a certain network. When this parameter is populated, clients should match the given nethash to an existing network with the same nethash. If no matching networks are found, clients should return an error stating that no matching networks were found.

### Register Delegate

Prompts the wallet to register a delegate

#### Endpoint

`ark:register-delegate`

#### URI Parameters

| Parameter  | Description                                        | Type          | Required           |
|------------|----------------------------------------------------|---------------|--------------------|
| username   | The delegate name to be registered                 | String        | :white_check_mark: |
| fee        | The fee amount to be used to send this transaction | Numeric       | :x:                |
| relay      | The relay node to be used to send this transaction | UTF-8 Encoded | :x:                |
| nethash    | The network to be used for this transaction        | String        | :x:                |
| otherparam | Reserved for future extensions                     | UTF-8 Encoded | :x:                |

#### Example

`ark:register-delegate?username=mydelegatename&fee=12&relay=https%3A%2F%2Fexplorer.ark.io%3A8443`


#### Specification

- The `relay` parameter is used to instruct the client to send a `POST` request directly to the given relay, bypassing the peer that it was previously connected to.
- The `nethash` parameter is used to instruct the wallet to switch to a certain network. When this parameter is populated, clients should match the given nethash to an existing network with the same nethash. If no matching networks are found, clients should return an error stating that no matching networks were found.

### Sign Message

Prompts the wallet to sign a given message

##### Endpoint

`ark:sign-message`

#### URI Parameters

| Parameter | Description              | Type          | Required           |
|-----------|--------------------------|---------------|--------------------|
| message   | The message to be signed | UTF-8 Encoded | :white_check_mark: |


#### Example

`ark:sign-message?message=This%20is%20my%20message`

### Backwards Compatibility

#### Sign Message

To ensure backwards compatibility with 1.x networks and clients, the AIP13 standard must be followed.

#### Endpoint

`ark:<arkaddress>`

#### URI Parameters

| Parameter   | Description                                            | Type          | Required           |
|-------------|--------------------------------------------------------|---------------|--------------------|
| label       | The contact name for this recipient on the client      | Base58        | :x:                |
| amount      | The amount to be sent. It must be specified in decimal | Numeric       | :white_check_mark: |
| vendorField | The smartbridge field                                  | Numeric       | :x:                |
| otherparam  | The network to be used for this transaction            | UTF-8 Encoded | :x:                |

#### Example

`ark:AePNZAAtWhLsGFLXtztGLAPnKm98VVC8tJ?label=John-Doe&amount=20.3`
