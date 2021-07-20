```
  AIP: 37
  Title: Switch Delegate With Single Vote Transaction
  Authors: Dmitry Tyshchenko <dmitry@ark.io>
  Status: Draft
  Discussions-To: https://github.com/arkecosystem/AIPS/issues
  Type: Standards Track
  Category: Core
  Created: 2021-07-20
  Last Update: 2021-07-20
```

## Abstract

Support both unvote and vote actions within a single vote transaction.

## Motivation

Currently to change wallet's delegate two separate transactions are necessary:

1. Remove current delegate (e.g. vote `-027716e659220085e41389efc7cf6a05f7f7c659cf3db9126caabce6cda9156582`).
2. Set new delegate (e.g vote `+03d3c6889608074b44155ad2e6577c3368e27e6e129c457418eb3e5ed029544e8d`).

Combining both actions into a single transaction would slightly reduce core resource consumption and significantly reduce fees that are charged.

## Specification

Fortunately existing vote transaction asset format already supports combining unvote and vote together:

```json
{
  "asset": {
    "votes": [
      "-027716e659220085e41389efc7cf6a05f7f7c659cf3db9126caabce6cda9156582",
      "+03d3c6889608074b44155ad2e6577c3368e27e6e129c457418eb3e5ed029544e8d"
    ]
  }
}
```

No changes to various platform-specific SDK are required.

## Backwards Compatibility

Handling vote transaction with asset that contains only a single action (vote or unvote) remains unchanged.

## Reference Implementation

It's already implemented, merged, published, and [forged on devnet] since height 5,640,858.

[forged at devnet]: https://dexplorer.ark.io/transactions/004ecc270fbb900c89df36212f85e8f99fa736ad51e2ab06d8307e9af7b89f54