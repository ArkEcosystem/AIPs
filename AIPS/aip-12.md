
<pre>
  AIP: 12
  Title: Smart Contract
  Authors: Adrian Cearnau, François-Xavier THOORENS <fx.thoorens@ark.io>
  Status: Draft
  Type: Standards Track
  Created: 2017-10-23
  Last Update: 2017-10-23
</pre>

Abstract
========
Integrate a modified version of Ethereum Virtual Machine (EVM) and define the transactions to register and call smart contracts.
Possible specific use to ark, after integration of smart contracts:
- delegate pools can improve the delegate sharing distribution making this automatic and public
- delegate votes weight can be automatically adjusted/optimisised according to their productivity/ranking etc...
- more powerful multisignature mechanics
- ...

This is early specifications subject to many changes as it is developed and tested.

Specifications
==============

1. Transactions 

*Type 9 (smart contract registration)*

| Description         | Size (bytes)  | Example                                                              |
| -------------       | ------------- | :-------                                                             |
| code                | ...           | serialised code in bytes                                             |

- The address of the contract is calculated by applying sha256 + ripemd160 to the transaction serialisation.
Then a byte 0x19 is added before the base58check performed.
Contract addresses will start with "C" instead of "A" for normal addresses

*Type 10 (smart contract call)*

| Description         | Size (bytes)  | Example                                                              |
| -------------       | ------------- | :-------                                                             |
| contract address    | 21            | 0x191dfc69b54c7fe901e91d5a9ab78388645e2427ea                         |
| amount              | 8             | 0x8096980000000000                                                   |
| params              | ...           | 0x171dfc69b54c7fe901e91d5a9ab78388645e2427ea                         |


2. Storage
New tables are implemented
- Contract to store transaction contract with the following fields:
  1. owner public key (indexed)
  2. code
  3. address of the contract (indexed) linking to mem_accounts
  
- Contract state
  1. contract address (indexed)
  2. state (BLOB)
  3. block_id (indexed)

- Contract logs
  1. contract address (indexed)
  2. event id (indexed)
  2. event
  3. sender (indexed)
  4. block_id (indexed)

3. Contract transactions
Output transactions from contract can be any existing supported transactions but they are accepted by other nodes if and only if they are created, signed by the block forger and are valid transactions. 

4. ARK VM
ArkVM will support most operations of EVM and adapted to property of ark (addresses, signatures, keys, block-id etc...)
