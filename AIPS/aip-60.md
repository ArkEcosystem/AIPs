---
  AIP: 60
  Title: Make votes from inactive wallets obsolete
  Authors: roks0n
  Status: Draft
  Discussions-To: https://github.com/ArkEcosystem/AIPs/issues/48
  Address: **TODO**
  Type: Core
  Category: Core
  Created: 2019-02-02
  Last Update: 2019-02-02
  Requires (*optional): None
  Replaces (*optional): None
---

## Abstract
This AIP proposes a change which makes votes from inactive wallets obsolete.

## Copyright
MIT

## Motivation

Ark has a lot of large holders that participate in voting which currently have the power to put one or several delegates into the top 51 forging position. There is also a significant amount of inactive participants in the network who voted have either lost interest or forgot their poses Ark.

Example of current 5 biggest wallets that are voting for a delegate and haven't made any outgoing transaction since 1.1.2018:
```
arkane (1,154,647)
therock (781,689)
therock (609,892)
arknet (600,090)
doc (329,673)
```

Example of vote report from 2.2.2019 showing how many wallets are still voting for delegates that have either publicly resigned or left the network:
```
|  53  | dafty                     |  0.44  |    561,610 |   466  |
|  54  | dr10                      |  0.35  |    437,440 |  1010  |
|  55  | chris                     |  0.21  |    264,245 |   255  |
|  56  | sharkpool                 |  0.15  |    191,990 |   808  |
|  57  | arkade_delegate           |  0.11  |    142,295 |    34  |
|  58  | bbclubark                 |  0.09  |    109,869 |    56  |
|  59  | alphabit_fund             |  0.04  |     51,825 |     7  |
|  60  | wes                       |  0.03  |     42,028 |   125  |
|  61  | jamiec79                  |  0.02  |     29,498 |   208  |
|  62  | elegato                   |  0.02  |     25,002 |     7  |
```

As you can see a significant number of wallets, some of which also hold a considerable amount of Ark, are inactive on the network but still influence who is responsible for achieving consensus.

## Specification
A wallet should be considered as inactive if it hasn't made any transaction outgoing transaction in a given period. Based on the discussion on the AIP's issue I'd suggest a period of 365 days. We would mark a wallet as active or inactive directly on the `wallet` table which we can then use to filter out inactive wallets when querying who the forging delegates are.

A task that would run a check on inactive wallets and mark them as inactive would run at the time when the new round starts. If we think this is too frequent we could make it so that it would run every N-th round.

We would mark a wallet as active every time we apply a transaction to the sender, meaning only outgoing transaction would be able to change the state.

## Rationale
Marking wallets as inactive at the start of each round is better over having a separate cronjob like task that's ran periodically. It makes code better structured if such actions occurs at the time of a specific event as well as start of a new round seems like a reasonable place to do this given it effects who will be put into the next round.

Marking wallets as active every time an transaction to the sender is applied also makes most sense as it's the least disruptive given we make a write/update to the table already to update the wallet's balance.

## Backwards Compatibility
This change would require a hard-fork given that it could effect which delegates are forging.

## Reference Implementation
**TODO**

The reference implementation must be completed before any AIP is given status "Final", but it need not be completed before the AIP is accepted. It is better to finish the specification and rationale first and reach consensus on it before writing code.

The final implementation must include test code and documentation appropriate for the Ark protocol.
