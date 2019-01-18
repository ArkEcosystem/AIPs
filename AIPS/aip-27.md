<pre>
  AIP: 0027
  Title: Addressing Long Range Attacks
  Authors: Moazzam Abdullah Khan, Asif Mehmood
  Status: Draft
  Type: Standards Track
  Discussion: https://github.com/ArkEcosystem/AIPs/issues/45
  Created: 2019-01-11
  Last Update: 2019-01-11
</pre>


Abstract
========

Proof of Stake systems and Delegated Proof of stake systems are unable to follow the longest chain valid rule because of the risk of a long range attack. In this article we try to discuss some arpproaches that could be used to mitigate that risk and harden a (D)PoS network against long range attacks.

Motivation
==========

A significant issue that stops (D)PoS networks from following the longest chain rule is known as a Long Range Attack. In this attack adverserial actors that used to hold stake in the network in the past can collude if they had a majority anytime in the past and create a new fork from that point. This group of nodes can then selectively replay transactions in the newly created Long Range Chain (LRC) and keep adding blocks until it overtakes the original Honest Chain (HC). Since there's no proof of work involved the LRC doesn't have any limitations in how many blocks can be added per second therefore a (D)PoS network following the longest chain will have an increased chance of getting long range attacked as time passes and old stake holders or delegates sell their stake. Since these old stake holders have already sold their stake they have no disincentive that prevents them from considering to attack the honest chain. In this AIP we discuss how it might be possible to protect against such an attack.

Rationale
=========

The basic idea of protecting against long range attacks is to only allow fork resolution to happen near the head of the chain. This means that as more and more blocks are added to the chain the chance of a previous stake holder who can attack the network increases therefore a fork that has a common block far in the past should be trusted less than a fork happening in the very recent blocks.

Specifications
==============

In this section we discuss two different approaches that may be used in defending against long range attacks.

### Approach 1
One method that might be viable to protect against getting long range attacked is to set a hard depth limit beyond which a fork will not be considered for selecting the longest chain. This means that the delegates will calculate which block was the last common block between the two forked chains and only switch if the block is not older than the last 40 blocks for example. The depth limit of 40 is chosen here somewhat arbitrarily since it allows enough time for fork resolution to happen near the head of the chain but isn't far enough in the past where some exchanges would be affected. However this number can be changed as required. While this approach may protect exchanges, merchants using Ark as their payment gateway might not want to wait for 40 block times before handing out a product. One solution to ensure they can still trust the network and don't become victims of a double spend attack is to implement zero confirmation transactions.

Transactions can be trusted with zero confirmations (0 conf) if the network always has capacity to include a broadcasted transaction in the next block and if the network is forced to prioritize transactions based on the time they were received by the network rather than by the transaction fees. Additionally merchant solutions will often have full nodes or relays installed that will detect if a double spend was attempted and delay the order if one is detected until the chain settles. Generally the time needed to wait and confirm that a double spend wasn't attempted is 3 seconds since that's the time required for enough nodes in the network to propagate transactions so that the competing transaction can't overtake the network. As mentioned before 0 conf transactions require that the transaction can be included in the next block which means that the max block size has to be increased past what's required to accommodate the expected peak number of transactions. Finally the forging nodes must communicate, with other nodes in the network, each transaction they receive and signal if a double spend was detected so that every node in the network knows of the competing transaction.

### Approach 2
If for some reason it is decided that the above mentioned approach is not suitable for adoption in Ark then another approach can be considered in which instead of having an abrupt fork depth limit we can spread out the limit based on probabilistic acceptance as a function of depth of common block. This approach takes inspiration from how frequently forks of different depths occur on the Bitcoin network and the fact that they seem to follow an exponential decay pattern i.e allegedly a 1 block depth fork happens on the Bitcoin network every day, 2 block depth fork happens every week and a 6 block fork is expected to happen once every 100 years. The only difference in our approach would be that a certain small percentage of the network would be expected to be on a minority fork for a given block depth. We can specify how the distribution of this network split looks like by declaring as part of the protocol how the acceptance probability of a competing chain changes with depth. It is suggested that the probability function should take the form 

`p_accept = 1/(A^(fork_depth+1))`

where p_accept describes the weight with which the competing chain will be considered for acceptance in the previously mentioned kelly criterion formula to decide which fork the delegate should bid on. In the above formula A is a constant specified by the delegates and fork_depth describes the depth from current height where the last common block was located in the honest chain. When the kelly outputs for both competing forks have been calculated the node would choose one randomly with their relative weights. An additional constraint is that the reevaluation to accept a competing fork happens at regular intervals according to the node's internal clock instead of being triggered by outside events such as receiving a new block etc. This ensures that delegates can easily resolve forks occurring at the head of the chain in a probabilistic manner similar to PoW networks but the chance that a long range attack can be successful in taking over the whole network is so small that it can be expected not to happen within the lifetime of the universe. Some additional points that have to be followed when using this approach are that exchanges and wallets need to poll multiple different nodes in order to be satisfied that they got the correct state from the nodes. The probability that they get a bad state follows the same exponential decay formula and if 1% of the network's nodes agree on a certain state then it's safe to trust it as long as the required transaction happened before last `fork_depth` blocks corresponding to `p_accept=1%`. To make sure that a node can recover from choosing a bad fork they can offset the acceptance probability based on the number of blocks they have forged on the competing chain (which would be the honest chain here) in the past with decaying weight. For instance if a node randomly accepts a competing fork with common block at depth 30, however unlikely that may be, then it will be more likely to switch back because it will have more blocks that were forged by the delegate in question on the old honest chain than in the new long range attack chain. This would however necessitate that the blocks received by new delegates while syncing are on the honest chain and not on the long range attack chain.

Finally it must be stated that both of the approaches presented above assume that the network isn't under sybil attack. Defending against sybil attacks is a topic that we will try to address at another time.
