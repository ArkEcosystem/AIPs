```
AIP: 19
title: Incremental snapshot system
author: François-Xavier THOORENS <fx@ark.io>, Kristjan KOSIC <chris@ark.io>
type: Standards Track
category: core
status: draft
created: 2018-04-10
updated: 2018-09-03
```

History
========
- 2018-04-10 inital content (@fix)
- 2018-09-03 added more requirements, moved from issuse https://github.com/ArkEcosystem/core/issues/238

Abstract
========
The purpose of this AIP is to provide specification and developer guidlines for implementation of this AIP.
We want to provide the base tooling to make quick proof of blockhain without launching a full node, make use of local exports (snapshots) and execute them regularly. 

Motivation
==========
To provide better ways of snapshotting and restoring the blockchain. Giving the node runners the tools, to be dependant on themselves and not on central point of failures.

Requirements
==============
### Exporting of data
- Regular export block and transactions table. Unfinished work is here: https://github.com/fix/ark-core/blob/master/packages/core/lib/snapshot.js
- Cross-distribute snaps for faster download
- Cross database migration structure: migrating from sqlite to postgresql for instance without resyncing using
  - snapshot db, read snapshot from another node using another db and syncing back
- Small compact export (using aip11 serializeBlockFull)
- Each time a block is validated it is written on db AND on snapshot
- Every 12 hours (or instance), the snapshot is duplicated. One duplication is archived, the other is used or incremental build.
- Enable rollback of the blockchain easily via the snapshot (blocks: remove last heights, transactions: index based on height). 


### Importing of data
- Verify blocks.id (previous.id)
- Verify Hashes and signatures… -> if is in the block
- Verify the transactions (proof of the blockchain)
