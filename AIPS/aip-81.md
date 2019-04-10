---
  AIP: *81*
  Title: *CORE-VM module specifications*
  Authors: *Kristjan Kosic <kristjan@ark.io>*
  Status: *Draft, Rejected or Active*
  Discussions-To: https://github.com/arkecosystem/AIPS/issues
  Address: *Ark address used to collect votes for the specific AIP*
  Type: *Standards*
  Category *only required for Standards Track: <Core>
  Created: *2019-04-09*
  Last Update: *2019-04-09*
  Requires (*optional): <AIP-29>
  Replaces (*optional):  
--- 
 
## Abstract
The purpose of this document is to define specifications and expectations related to building ARK VM in terms of ARK’s technology stack, namely running as a core-plugin and enabling virtual machine execution inside a core-module.

## Copyright
MIT License

## Check items list/Questions/To Address
- [x] AIP: define transaction outside of core-mode, e.g. inside our new module (store contract transaction)
- [ ] Size, memory, execution stack limitations
- [ ] Size of script
- [ ] Number and size of storage options
- [ ] Private smart-contracts (e.g. whitelisting addresses)
- [ ]

## Motivation
The goal of this project is to launch ARK VM inside the `core` technology landscape and run it as a module, if enabled. Looking further at the virtual machine life-cycle and core execution lifecycle we have the following communication points with our core.

### Deployment stage
The goal of this project is to launch ARK VM inside the `core` technology landscape and run it as a module, if enabled. Looking further at the virtual machine life-cycle and core execution lifecycle we have the following communication points with our core.

### Forging stage/Confirmation stage
A smart-contract enters from the pool and is forged inside a block. This means that smart contract is now available for execution of its methods and parameters.

### Execution stage
Execution of the smart contract via one of the selected sandboxed environments. Secure and sandboxed design via javascript vm execution plugin. Current options are:
1. https://github.com/laverdet/isolated-vm
2. https://github.com/patriksimek/vm2
3. https://nodejs.org/api/vm.html. 

VM execution engine must be selected based on the security and memory management and overall isolation execution. State should be passed into the `vm` and used as the current source of trust/truth.

### Storage 
Virtual Machine will introduce a new storage option for smart contracts to store state in as secure and distributed way. A Light key-value database can be used, as state can be reproduced via rebuild - from transactions (blockchain replay). 
### General constraints
#### Interfaces to other modules
- core-blockchain
- wallet-manager

#### Hardware limitations
- size limitations related to  overall script size
- memory usage limitations 
- isolated running environments per smart contract

#### Audit function
- strict logging of outcomes
- a separate execution log of the VM engine

#### Safety and security
- sandboxed environment
- timeout 
- limited operations and execution scope
- private contracts (limited by white-listed senders)

#### Error handling and recovery
General timeout for all execution points. Has to be “forged” quickly. Confirmation and compilation stage will be done in the deployment phase - while entering pool.


# Technical Specification
The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations.


# References
1. Ethereum notes for outgoing transaction https://ethereum.stackexchange.com/questions/24031/how-ethereum-contracts-transfer-ether-without-a-blockchain-confirmation
2. https://www.mobilefish.com/developer/blockchain/blockchain_quickguide_ethereum_related_tutorials.html
3. https://github.com/takenobu-hs/ethereum-evm-illustrated
4. https://ethereum.stackexchange.com/questions/20781/at-which-point-the-smart-contracts-get-executed
5. https://ethereum.stackexchange.com/questions/765/what-is-the-difference-between-a-transaction-and-a-call
