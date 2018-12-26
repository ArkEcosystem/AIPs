<pre>
  AIP: 23
  Title: Creating a delegate market for bridgechains by restructuring Ark main chain's voting mechanism
  Authors: Moazzam Abdullah Khan
  Status: Draft
  Type: Standards Track
  Created: 2018-07-31
  Last Update: 2018-08-05
</pre>

Abstract
========

This proposal introduces a mechanism to create a delegate market for opted-in bridgechains by moving voting and delegate selection to the Ark main chain. As a result businesses and 3rd parties deploying their own bridgechains don't have to worry about finding and convincing delegates to power their chain. This mechanism creates an economic incentive to hold the Ark token therefore 51% attacks are made more expensive to pull off on the main chain. Since delegate selection happens on main chain each opted-in bridgechain is made immune to a permanent 51% takeover in which malicious delegates censor votes after taking 51% majority in the chain. As a result the entire ecosystem of opted-in chains is made more secure.


Motivation
==========

Currently any 3rd party deploying a bridgechain needs to convince people to become delegates and forge on their chain in order to keep it decentralized and secure. Which means a lot of back and forth communication and effort is involved which is only necessary because the bridgechain doesn't have a monetary price associated to it's token and therefore the delegate has to speculate on it's potential value in the future and make their decision accordingly. However it is important to note that delegates are service providers who want to earn monetary valuable rewards in return for the service they provide and therefore if the dollar (or ark) value of the block rewards on the bridgechain is well established as per AIP18 then all of this effort is superfluous and it can be replaced by an on-chain autonomous delegate market that matches each bridgechain up with the delegates it requires based on the monetary value of block-rewards and delegate rank. We discuss in detail how the delegate selection mechanism and voting mechanism could work in the later sections.

The Ark ecosystem is designed to manage and transmit monetary value for the entire world therefore it is important to keep in mind the security and scaling implications of each change. In DPoS the difficulty of executing a 51% attack does not depend on having large amount of computation capability but instead it depends on having a large amount of money such that an attacker can use it to buy up majority of the voting ark/bridgetoken. Therefore it is in our best interest that the chain on which voting is occurring should have a very large marketcap since the marketcap is directly correlated with the difficulty of buying up majority of voting tokens. However currently the Ark ecosystem doesn't have a strong game-theoretic mechanic that creates incentive to hold the Ark token long term instead of bridgechain tokens. Which implies that at any point in time the total economic value held in the Ark main blockchain is approximately 1/N of the total Ark ecosystem's economic value where N is the number of well established chains in the ecosystem. This means that as new chains are created and speculated on, a significant amount of monetary value is moved from the main chain to each of the bridgechains and as a result the difficulty of executing a 51% attack is approximately 1/N times the difficulty that the main chain would have had if it had retained all that economic value. This is a clear conflict between the desired behavior and the behavior that the economic incentives reward. The delegate market and voting rework suggested in this AIP would solve this conflict by creating incentive to hold the Ark token because of it's significance in the voting process for bridgechains. There is no effect adverse on scaling since each delegate on the bridgechain is running at most 2 different nodes at a single time, 1 on the Ark chain to keep track of votes and 1 on the delegate chain to forge, however the Ark relay node can potentially be replaced with an SPV client.

An additional threat that is currently faced by new bridgechains in the ecosystem is that some malicious actor who has the money to carry out a 51% attack can take control over the network permanently by censoring vote transactions that are not favourable to the attacker. The only option left in such a situation is to fork the network and attempt to convince others to join the new network which will likely have disasterous consequences on adoption of the bridgechain. However if the voting and delegate selection is happening on the main chain then the bridgechain's delegates are powerless to stop the vote transactions and the community can come together and punish the malicious delegates by unvoting them. It may also possible to come up with a way to blacklist malicious delegates however this AIP isn't concerned with the exact mechanism employed for punishing bad actors.

Rationale
=========

* Businesses and 3rd parties launching a bridgechain are able to get the service of delegates in a simple and fast manner.
* Delegates are ranked by the vote-weight they have received.
* Bridgecoins are ranked by the Ark-value of their block rewards.
* Top ranked bridgecoins are assigned top voted delegates.
* One address one vote principle still holds so the top 51 delegates would forge on Ark while the delegates ranked 52 and onwards might forge on a bridgechain due to ranking based on ark-value of blockrewards.
* Ark commander tracks Ark main chain and is able to forge on any bridgechain based on registered network details and can switch the bridgechains as it's assigned new bridgechains.

Specifications
==============

* Prerequisites
	* On chain price discovery mechanism as suggested in AIP22

- New bridgechains have an Ark address registered as a liquidity gate as per AIP22. This address is tracked and is used to establish the exchange price between ark and the bridgecoin. Additionally the forging rewards and block time are announced in a special transaction which is used to register the address as a bridgechain's liquidity gate.

- Block rewards per unit time at the current exchange rate are used to create a ranked list of all bridgechains going from highest reward to lowest reward.

- Top 51 delegates based on vote weight are assigned to forge the chain with the highest value block reward (most likely Ark) while the delegates ranked 52 and onwards are assigned the bridgechains to forge on.

- Delegates selected for each bridgechain come up with a 90%-threshold cryptographic group public key (DKG) which is broadcasted to the Ark main chain and signed by each relevant delegate. Delegates forging on the main chain will consider a transaction originating from the liquidity gate as valid if at least 90% of the delegates authorized to forge on the bridgechain have signed it however once a group key is registered for the new delegates only transactions signed using the group public key will be considered as valid (this is done in order to reduce the number of transactions on the Ark main chain).

- All forged blockrewards on the bridgechains are sent in a cross chain transaction (as defined in AIP22) to the delegate address on the main chain through the liquidity gate.

- Delegates forging on bridgechain detect outgoing cross chain transactions on liquidity gates. They then create and sign an appropriate transaction originating from the liquidity gate.

- When a change of delegates occurs for a bridgechain a new group key is generated by the new delegates and registered on the Ark main chain thereby giving the new delegates control over the liquidity gate.
