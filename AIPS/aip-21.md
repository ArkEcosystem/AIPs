<pre>
  AIP: 21
  Title: Incentivize Relays (aka. ARK Masternode)
  Authors: galperins4 <galperins4@gmail.com>
  Status: Draft
  Type: Standards Track
  Category: Core/Protocol
  Created: *2018-09-29*
  Last Update: *2018-09-30*
</pre>

Abstract
========
This AIP proposes the creation of ARK Masternodes


Motivation
==========

With the current version of the ARK protocol the only nodes that are incentivized are Forging nodes. As such, there is no significant support outside of delegates to maintain a strong relay network to further decentralize and create and more resilient ARK network. This AIP proposes the following:
- Establishment of a new incentivized relay type (e.g., Ark Masternode)
- Change to block rewards to support ARK Masternodes
- Mechanism to check ARK Masternodes for performance
- Mechanism to reward ARK Masternodes

Specifications
==============

### 1. Create a new transaction type for masternodes:

**Type 9 (masternode registration, 1-256 bytes)**

| Description       | Size (bytes)  | Example                                                              |
| -------------     | ------------- | :-------                                                             |
| length            | 1             | 0x80 (minimum 0x03)                                                  |
| username (utf8)   | 3-255         | 0x6669786372797074   

- Masternodes will have a similar registration process as a delegate. The main difference is that there is a collateral requirement to run a masternode. To have a successful registration it is required that the account registering maintain the minimum collateral requirement.
- If the minimum collateral is not in the account at time of registration, the transaction will fail.
- The initial collateral requirements is proposed to be 2000 ark (which can be staked).
- A Type 9 transaction will cost 1 ARK.

### 2. Block Rewards Adjustment:
- Block rewards need to be adjusted to provide funding to masternode owners.
- Two possible suggestions: Increase block rewards to 2.2 (an increase of 10%) or reduce delegate share of block rewards to 1.8 (to maintain 2 ARK blocks).
- Masternode rewards per block initially set at 10& of current rewards (0.2) and a new database field (mn_reward) will be needed.
- Masternode rewards will be sent to the ARK masternode address after each block is created.

### 3. Updated Core Commander /  Ark Wallet:
- New transaction type will need to be handled by the Ark Wallet.
- New "configure" relay option will need to be created to link the relay node to the masternode account.
- Same options used for configuring forger node and protection of passphrase would apply.
- Masternode setup would also automatically set up arkstats on relay node.

### 3. Updated Core / Protocol
- Similar to forging rounds, a masternode round will need to be created to identify slots of randomized masternode accounts. 
- The round will not be limited to x masternodes. It will vary depending on the amount of qualified/registered masternodes.
- The round size will be determined by the # of masternodes where collateral in account >= requirement (2000ark).
- Similar to forging, nodes will need to cycle through each round of masternodes concurrently with forgers. 
- The same algorithm that decides if a forger can forge can be leveraged to determine if the masternode in the slot is eligible and can be awarded the masternode reward.

### 4. Masternode Performance / Rewards Distribution:
- 10,800 blocks (assuming perfect 8 second blocks in 24 hours) are created each day.
- Maximum rewards for masternodes will be 0.2 X 10,800 = 2,160 ark per day.
- Rewards will be distributed each block to the eligible masternode.
- For example, if  there are 500 masternodes eligible for payment, they will get an equal share of 0.2 per round. At a constant round of 500 that would equal approximately 4.32 ARK a day.
- At current prices, this would reward each relay owner with approximately 130 ark per month (~90$ USD) which should be profitable to maintain a reliable relay node at most VPS providers.
- Masternode distribution would need to be on a n+1 block basis. This is to avoid awarding a masternode award to a relay in the same round that a forger fails to forge (e.g., reward never exists). Mechanically it would look like this:
  - 

