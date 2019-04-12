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
  Requires (*optional): AIP-11, AIP-18, AIP-29
  Replaces (*optional):
--- 

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [-](#-)
- [Check items list/Questions/To Address [/] [%]](#check-items-listquestionsto-address--)
- [Motivation](#motivation)
    - [Deployment stage](#deployment-stage)
    - [Forging stage/Confirmation stage](#forging-stageconfirmation-stage)
    - [Execution stage](#execution-stage)
    - [Storage](#storage)
    - [General constraints](#general-constraints)
        - [Interfaces to other modules](#interfaces-to-other-modules)
        - [Rebuilding capability](#rebuilding-capability)
            - [Hardware limitations](#hardware-limitations)
            - [Audit function](#audit-function)
            - [Safety and security](#safety-and-security)
            - [Error handling and recovery](#error-handling-and-recovery)
- [Technical Specification(initial/will be further updated with reference implementation)](#technical-specificationinitialwill-be-further-updated-with-reference-implementation)
    - [Transaction Types](#transaction-types)
        - [TX: Store smart contracts/dApp](#tx-store-smart-contractsdapp)
            - [Field descriptions](#field-descriptions)
        - [TX: Execute smart contract/dApp calls](#tx-execute-smart-contractdapp-calls)
            - [Field descriptions](#field-descriptions-1)
    - [Core-vm module mechanics](#core-vm-module-mechanics)
        - [AbstractSmartContract class](#abstractsmartcontract-class)
        - [Compilation of Smart Contract](#compilation-of-smart-contract)
        - [Execution of Smart Contract](#execution-of-smart-contract)
            - [Storage definition](#storage-definition)
            - [Inter-transaction execution](#inter-transaction-execution)
        - [Storage capabilities](#storage-capabilities)
- [Reference implementation](#reference-implementation)
    - [Copyright](#copyright)
- [References](#references)

<!-- markdown-toc end -->


## Abstract
The purpose of this document is to define specifications and expectations related to building ARK VM in terms of ARK’s technology stack, namely running as a core-plugin and enabling virtual machine execution inside a core-module.


## Check items list/Questions/To Address [/] [%]
- [ ] AIP: define transaction outside of core-mode, e.g. inside our new module (store contract transaction)
- [ ] Size, memory, execution stack limitations
- [ ] Size of script
- [ ] Number and size of storage options
- [ ] Private smart-contracts (e.g. whitelisting addresses)
- [ ] Rebuild from zero - saving data on the blockchain
- [ ] Storage

## Motivation
The goal of this project is to launch ARK VM inside the `core` technology landscape and run it as a module, if enabled. Looking further at the virtual machine life-cycle and core execution lifecycle we have the following communication points with our core.

### Deployment stage
The goal of this project is to launch ARK VM inside the `core` technology landscape and run it as a module, if enabled. Looking further at the virtual machine life-cycle and core execution lifecycle we have the following communication points with our core.

### Forging stage/Confirmation stage
A smart-contract enters from the pool and is forged inside a block. This means that smart contract is now available for execution of its methods and parameters.

### Execution stage
Execution of the smart contract via one of the selected sandboxed environments. Secure and sandboxed design via javascript vm execution plugin. Current options are:
1. Isolated-VM - https://github.com/laverdet/isolated-vm
2. VM2 - https://github.com/patriksimek/vm2 
3. NodeJs VM - https://nodejs.org/api/vm.html. 

VM execution engine must be selected based on the security and memory management and overall isolation execution. State should be passed into the `vm` and used as the current source of trust/truth.

### Storage 
Virtual Machine will introduce a new storage option for smart contracts to store state in as secure and distributed way. A Light key-value database can be used, as state can be reproduced via rebuild - from transactions (blockchain replay). 
### General constraints
#### Interfaces to other modules
- core-blockchain
- wallet-manager

### Rebuilding capability
Smart contract data history (any calls with all the details) must be saved on the blockchain. We could use our asset field where we store this. 

#### Hardware limitations
- size limitations related to  overall script size
- memory usage limitations 
- isolated running environments per smart contract
- memory limitations
- CPU time limitations (`while (true) {}` limit)

#### Audit function
- strict logging of outcomes
- rebuilding relevant data must be added on the blockchain
- a separate execution log of the VM engine

#### Safety and security
- sandboxed environment
- timeout 
- limited operations and execution scope
- private contracts (limited by white-listed senders)
- general class implementation / as guideline and access to blockchain (wallet-manager)

#### Error handling and recovery
General timeout for all execution points. Has to be “forged” quickly. Confirmation and compilation stage will be done in the deployment phase - while entering pool.


# Technical Specification(initial/will be further updated with reference implementation)
The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations.

*THIS IS WIP, AND IT WILL CHANGE ALONG THE LINES AS REFERENCE IMPLEMENTATION CHANGES*

## Transaction Types
New transaction types need to be implemented in order to support interaction with the blockchain in terms of sending/loading smart contract to the blockchain and giving possibilities to call the smart contract methods.

### TX: Store smart contracts/dApp
Transaction type store smart contract will send smart contract payload to the chain `post/transactions` endpoint. 
dApp developer implements smart contract following the predefined interface provided from `core-vm`. SmartContract interface defines the structure and the methods an ARK smart contract must implement, as well as access to super class (TBD/Link).
A return value is status of smart contract deployment, and return address if successfully deployed. 

Size of the source code and duration of compilation(computing power needed) will be taken into account. Dynamic fee will be strongly affected by size.


| Description   | Size | Sample |
| ----------    | ---- | ------ |
| type          |      |        |
| fee           |      |        |
| interaction   |      |        |
| source-code   |      |        |

#### Field descriptions
1. interaction / similar to eth abi -
2. source code
3. compiled source

When smart contract enters the pool, we test it against internal compiled method from the virtual machine. If script is successfully compiled we accept it into the pool and it will be forged when its turn. 
If not we will return error code to the user sending the script.

### TX: Execute smart contract/dApp calls
When transaction is successfully deployed, it holds all the required information for anyone to execute the dApp (if allowed by the contract code). 

| Description      | Size | Sample |
| -----------      | ---- | -----  |
| type             |      |        |
| fee              |      |        |
| amount           |      |        |
| contract-address |      |        |
| method-name      |      |        |
| arguments        |      |        |

#### Field descriptions
1. contract-address
2. method-name (name of one of the methods specified via abi)
3. arguments (k/v pairs of arguments accepted by smart contract call). Must be aligned with dApp abi specifications


## Core-vm module mechanics

- [ ] Make basic code run
- [ ] Sync Variables
- [ ] Initialize a class
- [ ] Run methods from class
- [ ] Save data to common storage/outside of secure execution - just return
- [ ] Prepare internal transaction calls - based on smart contract output


### AbstractSmartContract class
SmartContrac

### Compilation of Smart Contract
Smart contract will be deployed as a normal javascript(transpiled typescript) source-code to the core. Both options can be supported. This will give use the power to leverage the huge community of js/ts developers.

- [ ] Determine to be run locally or on the node/ currently node is preferred / possible errors can be returned?

### Execution of Smart Contract
Output results of smart contract execution are storage updates or transaction execution - transfer from contract to another contract or normal address.

#### Storage definition

#### Inter-transaction execution

### Storage capabilities


# Reference implementation
Based on research and some demo stuff, currently `isolated-vm` looks like most promising.
- https://github.com/kristjank/virtual-machine/

## Copyright
MIT License

# References
1. Ethereum notes for outgoing transaction https://ethereum.stackexchange.com/questions/24031/how-ethereum-contracts-transfer-ether-without-a-blockchain-confirmation 
2. https://www.mobilefish.com/developer/blockchain/blockchain_quickguide_ethereum_related_tutorials.html
3. https://github.com/takenobu-hs/ethereum-evm-illustrated
4. https://ethereum.stackexchange.com/questions/20781/at-which-point-the-smart-contracts-get-executed
5. https://ethereum.stackexchange.com/questions/765/what-is-the-difference-between-a-transaction-and-a-call 
6. https://github.com/laverdet/isolated-vm
7. https://github.com/gianluca-venturini/isolated-vm-actors/blob/master/src/index.ts
