```
AIP: 17
title: Transaction Pool Wallet Manager
author: Kristjan KOSIC <chris@ark.io>, François-Xavier THOORENS <fx@ark.io>
type: Standards Track
category: Core/Protocol
status: Implemented
created: 2018-05-01
updated: 2018-08-21
```

History
========
- 2018-05-29 inital content (@fix)
- 2018-08-21 added more detailed explanation and related information (@kristjank)

Abstract
========

This is the description of transaction pool guarding mechanism, looking from the pool wallet manager implementation state, that will prevent double spending already on the transaction-pool level, thus reducing the options for spamming and other transaction related attacks.

Motivation
==========
In order to manage properly the transaction pool and prevent spam, the rules are described in this document and implemented in ark-core v2.

Specification
=========
Transaction pool wallet acceptance rules:

### 1. New unverified transaction is received:
  - verify the transaction (using AIP-11 SER/DER), signature
  - check if the txid (*the one computed*) is already in db -> if yes: reject
  - find the wallets of the recipient and of the sender inside the pool
  - if no wallet, COPY wallet from the blockchain wallet-manager to the pool-wallet manager
  - test wallet.canApply
  - if successful, use wallet.applyTransaction (accepting transaction if ok and adding to the pool)

### 2. New block added on blockchain
  - find the wallet of delegate in txpool. if found apply the block to the wallet, otherwise do nothing
  - for each transaction
    - if txid in pool:  remove tx from pool. 
      - if sender wallet is empty and no other tx with sender as recipient in txpool, remove it from txpool.
    - else find wallets from sender/recipient
      - if found recipient wallet, apply tx
      - if found sender wallet check `canApply`. 
        - if ok use `applyTransaction` on this wallet
        - else (ouch: double spending): drop all tx from the sender in the txpool, apply this tx (canApply should be ok otherwise the txpool should be resetted)


### 3. Dropping of transactions
  - drop transaction (that was already in txpool as per rule 1)
  - revert tx on wallets of sender and recipient. if recipient wallet is empty, remove from txpool

### 4. Cleaning
  - every now and then clean unchanged wallet for over 24 hours or so
  - apply some limit transactions per sender (configurable, around 100)


Specifications
==============
### Implementation
Transaction pool exists of the core module in the ark-core, called `core-transaction-pool`. It is the only module designed to interact with the `core-blockchain` and it can have any sub implementation for storage, that can be done in a very straighforward way, by following the `TransactionPoolInterface` class. 

And example of this can be seen in `core-transaction-pool-redis` where Redis storage implementation is done. A new `sqllite`, `memory` or any other implementation, can be done just by following the specifications of the interface class, leaving room for modular approach and integration with different caching mechanisms and platforms. 

### Pool configuration module.
```json
  maxTransactionsPerSender: process.env.ARK_TRANSACTION_POOL_MAX_PER_SENDER || 100,
  allowedSenders: ['senderPublicKey1', 'senderPublicKey2']
```
