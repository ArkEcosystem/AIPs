<pre>
  AIP: *16*
  Title: *Increasing network impact*
  Authors: *Jeremi Gendron (Blockchain Dealer) <gendronjeremi@gmail.com>*
  Status: *Draft*
  Type: *Reference*
  Created: *2018-07-16*
  Last Update: *2018-07-16*
</pre>

Abstract
========

Ark and most decentralized computing platforms offer core functionality that too often depends on external resources to be maintained, accessed or used.

This specification is short and currently provides a use case and defintion with a suitable example written in pseudocode.


Motivation
==========

Mainly, offer a basic conceptual description of version control, networking and consensus models that are suitable for a unified decentralized computing structure through a proposal system. 

Secondly, receiving feedback and improving this proposal.

Lastly, understand the demand for a differently tailored approach to expanding technology from an impactful ecosystem.

Rationale
=========

A lot of what is shared below will make you feel right at home, I was influenced by many factors and do not claim ownership of what is written.

The specification below was written to be easily communicated with others, perhaps a single read will greatly ameliorate most's capacity at even using the system with simple human interfaces.

I will offer my own personal proposals using the outlined framework after the use case and example has been given, with a testable logic.

This was written 2018-07-15 and evolution can be tracked at (https://github.com/jeremigendron/ark-default)[this repo for the moment].

Specifications
==============
____________

## Use case demonstration
A decentralized community wishes to maintain an active interface for its members to use. Opinions and decisions are an important part of every member's life, therefore a system to manage consensus is required. This system should promote sharing and contribution to ensure it remains healthy throughout its evolution.

Technology is an important part of the modern interface that is made to aid this community grow and organize itself. An issue is present in currently used methods to manage how the appropriate protocols, functions and standards should be used. For example, one can't just alter the Bitcoin Core code and have others find this idea without using an external bit of software.

This is solved readily by a simple and modular version control system, which is also the skeleton used for many other applications to keep compatibility across vital components.

The process of instantiating the version control system, or network, requires a genesis state which is agreed upon through averaging decisions posted by initial contributors. This is called the genesis proposal, first proposal, genesis block, etc.

Once a start to the network is defined, shared and agreed upon by a cluster of members, it can be used as a base to further developments of the community. A disagreeing cluster or member is encouraged to provide a different (or subsequent) proposal and see adoption from similar-minded individuals or groups. The main challenge is making the genesis state so unanimously powerful and irreplaceable that it is simply always a better choice to use it as the root. Otherwise, parallel networks which may not be completely compatible are discovered outside the scope of the community (multiple communities in the end); we can then safely prove that there is an underlying network to the one the community has attempted to build. CTRL + S

### Example
The network was bootstrapped with a Decentralized Autonomous Organization at its core for governance. The initial consensus model is Proof of Stake, meaning people pledged a unit of value that is outside the scope of the network in order to receive a representative amount of the network's native unit of value. The genesis concept was to include a message along with the pledged currency that contains a vote for how much of the total pool of native currency should be attributed and controllable through the DAO with help of the Proof of Stake consensus mechanism. The genesis rule was to weigh each vote based on the external currency pledge amount. The genesis proposal doesn't require economic parameters, as the unit of value operates inversly: the maximum amount of units is 1, or the whole, and 5 represents 20% of the total amount of units. A pledge transaction might look like:
	donation: 0.1 BTC
	transaction: 28ASD7F8923SDF878796823F -> data{vote: 10} (the governer gets 10%)
	signature: 88423098SD09F8S0D8F023KLJKJDSF78923

Very simply: Bitcoin account associated with transaction has donated 0.1 BTC and wishes the network to use 10% for the DAO. A (genesis) proposal might look like:
	concept: vote
	consensus: Weighed Average -> Proof of Stake (this could be proof of Interested)
	vote: 1

Very simply: vote with your pledge which is averaged and governor receives 100% of the calculated value. This means if someone doesn't appreciate that the genesis proposal yielded 10% network value to the DAO, they can easily make another proposal to use a different genesis proposal configuration and garner traction that way (other users who pledged might find both useful).

Also important: at any moment a user holding the native currency may decide to exchange it back to a proportional amount of the external currency, minus the calculated DAO stake.

* Advisable to not actually have anyone run the governor account on the Bitcoin blockchain, but rather to issue Bitcoin-tokens on the actual network in exchange for the native currency when demanded.

Is Bitcoin a security? Never.

Once the genesis occurs, users are encouraged to submit their proposals to include a base to use as the technological interface, this can be anything including an empty file, and anyone is also encouraged to freely copy for (or make private through encryption) their proposal data.
