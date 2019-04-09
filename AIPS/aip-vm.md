---
  AIP: *XX*
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

## Motivation
The goal of this project is to launch ARK VM inside the `core` technology landscape and run it as a module, if enabled. Looking further at the virtual machine life-cycle and core execution lifecycle we have the following communication points with our core.

### Deployment stage
The goal of this project is to launch ARK VM inside the `core` technology landscape and run it as a module, if enabled. Looking further at the virtual machine life-cycle and core execution lifecycle we have the following communication points with our core.

### Forging stage/Confirmation stage
A smart-contract enters from the pool and is forged inside a block. This means that smart contract is now available for execution of its methods and parameters.

### Execution stage
Execution of the smart contract via one of the selected sandboxed environments. Secure and sandboxed design via javascript vm execution plugin. Current options are:
- https://github.com/patriksimek/vm2  and 
- https://nodejs.org/api/vm.html. 

Execution is tightly coupled with blockchain state - available  via wallet manager. 

### Storage 
VirtualMachine will introduce a new storage option for smart contracts to store state in as secure and distributed way. A Light key-value database can be used, as state can be reproduced via rebuild - from transactions (blockchain replay).

### General constraints
#### Interfaces to other modules
- core-blockchain
- wallet-manager
- transaction-pool

#### Hardware limitations
- size limitations related to number of variables, memory and storage options

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
