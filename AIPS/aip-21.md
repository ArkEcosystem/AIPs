<pre>
  AIP: 21
  Title: Incentivize Relays (aka. ARK Masternode)
  Authors: galperins4 <galperins4@gmail.com>
  Status: Draft
  Type: Standards Track
  Category: Core/Protocol
  Created: *2018-09-29*
  Last Update: *2018-09-29*
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

### 1. Block Rewards Adjustment:
- Creation of a masternode rewards address - managed by the ARK team
- This address will be utilized to accumulate masternode rewards and for daily distributions

### 2. Block Rewards Adjustment:
- Block rewards need to be adjusted to provide funding to masternode owners
- Two possible suggestions: Increase block rewards to 2.2 (an increase of 10%) or reduce delegate share of block rewards to 1.8 (to maintain 2 ARK blocks
- Masternode rewards per block initialy set at 10& of current rewards (0.2) and a new database field (mn_reward) will be needed
- Masternode rewards will be sent to the ARK masternode address after each block is created

### 3. Updated Core Commander:
- Similar to other masternodes there will be a collateral requirement to run a relay node
- 1 masternode will be allowed per ark address
- The initial collateral requirements is proposed to be 2000 ark (which can be staked)
- A new option in core-commander to register a relay node will be required. This registration should do 2 things

### 4. Masternode Performance Monitor:

### 5. Masternode rewards distribution:

