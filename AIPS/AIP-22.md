<pre>
  AIP: 22
  Title: Token price discovery and creating high liquidity decentralized exchange in the Ark ecosystem using instant crosschain atomic swaps 
  Authors: Moazzam Abdullah Khan
  Status: Draft
  Type: Standards Track
  Created: 2018-07-31
  Last Update: 2018-08-05
</pre>

Abstract
========

This proposal introduces a mechanism to do on chain price discovery using 0-conf crosschain atomic swap transactions. We also discuss how to provide liquidity for this exchange using specialized addresses that we refer to as "liquidity gates" and how to manage their security.

Motivation
==========

New and existing bridgechains need to be listed on an exchange in order for the market to discover their fair price. These exchanges often extract exorbitant fees in order to list the tokens. This impacts adoption of the blockchain projects negatively and stunts their growth. Worse still it creates a bad risk/reward structure which rewards a ponzi-esque pump to get listed on bigger and bigger exchanges and punishes teams that work to create a useful product without spending all their money on marketing or exchange listings. We introduce a mechanism to solve all these issues by taking advantage of the Ark ecosystem's smartbridge functionality and multi-chain scaling ideology.

Rationale
=========

* Ability to register an address as a liquidity gate.
* Registering a liquidity gate reserves a "network name" for the address which makes routing cross chain transactions easier
* Liquidity gate registered on the bridgechain is called a local liquidity gate
* Liquidity gate registered on the Ark main chain is called a remote liquidity gate
* Once registered as a liquidity gate, an address can only be controlled by multiparty concensus via a distributed key generation mechanism (e.g Secure Distributed Key Generation for Discrete-Log Based Cryptosystems by Gennaro et al.). We specify that the delegates on a bridgechain create an 80% threshold DKG group public key in order to manage the liquidity gates connecting Ark to the bridge chain.
* Generated group public key is transmitted to the remote liquidity gate (discussed in more detail in upcoming AIP23)
* A pair of remote and local liquidity gates is used to perform cross chain atomic swaps between the two chains
* Liquidity gates use the vendorfield to track and update exchange price between the local and remote chain tokens
* A local liquidity gate only connects to a single remote liquidity gate due to how price is calculated(might change in the future)


Specifications
==============

* Prerequisites
	- 0-conf transactions
### Explanation of 0-conf:
- In order to accept a transaction without requiring any confirmations the following criteria must be met:
	- Blocks must never be full (i.e transaction must be included in next block)
	- Delegates must include first seen transaction first in a block
	- Delegates must relay all recieved transactions to other forging delegates to ensure that they aren't being given a transaction in isolation by bad actors trying to double spend 

### Liquidity gate registration

---------------------------------------------------
| Description | Example                          |
| --------------- | ------------------------------- |
| vendorfield | Name:ARKC,Price:80000 |


### Cross chain transaction format
example of a transaction being sent from the bridgechain to the Ark main chain

---------------------------------------------------
| Description | Example                          |
| --------------- | ------------------------------- |
| vendorfield | ARKC:(ARK-ADDRESS):(VALUE) |

where ARKC is the network name registered when creating the local liquidity gate

### Liquidity gate authentication update
After a change of forging delegates the new delegates create a new DKG public group key and publish it on the remote and local liquidity gates

### Price tracking and update
Initial price for a liquidity gate is specified upon creation by the creator. For instance if being used to do an ICO the price should be set to the lowest estimate of market cap as it will increase when more people buy the bridgechain tokens. If a group of people are creating a liquidity gate for an existing bridgechain it can be initialized with a current price taken from an exchange.

Price update for a bitcoin-like exponential increase asset is performed using the following formula:
$$p' = p * exp(-A * order)$$

where p' is the new price, p is the old price, numLiquidTokens is the number of tokens in the current local liquidity gate, order is the number of tokens being exchanged (-ve for bridgechain tokens being bought and +ve for bridgechain tokens being sold) and A is a dampening coefficient which can have any positive fractional value.

An alternate price update formula that can be used for createing a stable coin is as follows:
$$p' = p + \frac{MaxMCap}{numTotalTokens}*log(\frac{numLiquidTokens}{numLiquidTokens+order})$$

If all of the rules specified above are followed realtime price discovery can occur by sending a crosschain transaction with the desired price value. If a transaction has remained unfilled for more than 10 minutes it will be returned to the sending address. A liquidity gate token swap transactaion is only accepted if it doesn't change the price more than 1% in one block time unit.
