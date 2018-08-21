```
AIP: 16
title: Dynamic Fees
author: François-Xavier THOORENS <fx@ark.io>, Kristjan KOSIC <chris@ark.io>, Alex BARNSLEY <alex@ark.io>
type: Standards Track
category: Core
status: Active
created: 2018-05-01
updated: 2018-08-21
```

### Summary

Abstract
========
The purpose of dynamic fees is to enable a healthy market dynamics with options for delegates and users (transaction senders) to choose from. Acceptance fee threshold is decided by the delegate(s) - if fees are below threshold - they wont accept the payload into the transaction pool. The transaction fee is defined by the transaction sender.

Motivation
==========
The transaction size defines market dynamics - you pay more for (storage, computation, additional information). To optimize fees, a delegate puts minimum limits in the delegate config.  To reduce spamming forger rules need to be setup and implemented in the configs (starting with fixed ones).

Dynamic Fee calculation (minimum and max transaction ) represents the rule for including the transactions in the delegate pool and eventually forging them.

Actors
===============
## Users/Customers:
A user sets his exact fee he is willing to pay when sending a transaction (setting a custom fee in the transaction payload). An insanely low fee would result in transaction never being forged (fee will not make it into delegate pools).

## Delegates:
Define their own C (constant) for fee calculation according to the formula on API11, that is related to:
- Type of the transaction
- Size of the serialised transaction

## Node:
A client API can retrieve history values of fees, so market monitoring can be done based on the node config endpoint and new services can be provided to users to monitor the market behaviour. By calling [node configuration endpoint](https://docs.ark.io/developers/api/public/v2/node/retrieve-the-configuration.html#endpoint) a `feeStatistics` parameter is returned, where minimum, maximum and average fee for the last 30 days is retured, calcualted by transaction type. 
```
    "feeStatistics": [
      {
        "type": 0,
        "fees": {
          "minFee": 268421,
          "maxFee": 597781,
          "avgFee": 404591
        }
      }
    ],       
```

Returned values will be used in wallets and other GUI client application to help users choose optmial fees, when sending transations. 

## Formula calculation:
Dynamic fee calculation is related to the:
- Type of the transaction
- Size of the serialised transaction
- Offset value defined by the network

The calculation formula: `Fee = (T+S) * C`
- T: offset value depending on transaction type, defined by the network. T is here to account for extra processing power to process special transaction whose transfer value is null, and thus reducing economic interest to spam the network.

- S: size of the serialised transaction. For instance, for transfer we could have T = 0 byte, C = 1000 Arktoshi/byte. For a classic transfer transaction with empty VendorField S = 253 bytes, hence the fee is 0.00253000 Ark.

- C: fee multiplier constant (Arktoshi/byte) defined by the delegates for including the transaction in his forged block/transaction pool


Specifications
==============
## Client SDK
All client libraries will enable users to set a custom fee for the transaction, before the transaction is signed and sent. Some optional limits or security checks are recommended on client GUI level, to inform the users if the fee is above average or above current static fees defined in node configuration.

## Core/Node level configuration
### Delegate settings
A delegate can define his formula parameters for C and limit incomming transactions with `minAcceptableFee` value. All settings are in ARKTOSHI. Delegate settings can be found in [delegates.json](https://github.com/ArkEcosystem/core/blob/develop/packages/core/lib/config/testnet/delegates.json#L2-L4)

Example:
```
  "dynamicFees": {
    "feeMultiplier": 1000,
    "minAcceptableFee": 30000
  }
```

### Formula calculation network parameters
- `feeMultiplier` is C in the formula above. 
- `minAcceptableFee` is the limit for the inclusion of transaction by the delegate. If the incomming transaction has lower fee than delegates `minAcceptableFee` then the transaction is not included in the delegates pool, but it is only broadcasted to other nodes, where other delegates can pick it up, according to the defined rules.

`T - acts as offset` and is defined by the [network.json](https://github.com/ArkEcosystem/core/blob/c7a3bc75ffed5e5b9453d0de38937540fe48bce5/packages/crypto/lib/networks/ark/testnet.json#L39-L48) for each of the supported transaction types. Offsets are defined as following:

```
   "dynamicOffsets": {
      "transfer": 100,
      "secondSignature": 250,
      "delegateRegistration": 500,
      "vote": 100,
      "multiSignature": 500,
      "ipfs": 250,
      "timelockTransfer": 500,
      "multiPayment": 500,
      "delegateResignation": 500
   }
```

By setting the value `dynamic` to `true` and defining block height from which the settings should go into effect - we can enable or disable the dynamic fees on the node level. Settings are in [network.json](https://github.com/ArkEcosystem/core/blob/c7a3bc75ffed5e5b9453d0de38937540fe48bce5/packages/crypto/lib/networks/ark/testnet.json#L52-L55)

For example the settings below will enable dynamic fee acceptance from block height 10 onward.
```
    "height": 10,
    "fees":{
      "dynamic" : true
    }
```