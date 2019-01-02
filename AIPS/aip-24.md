```
AIP: 24
Title: Double Forging improvements
Authors: Kristjan Kosic <kristjan@ark.io>, Joshua Noack <joshua@ark.io>, François-Xavier Thoorens <fx.thoorens@ark.io>
Status: Draft
Type: Protocol Track
Created: 2018-12-17
Last Update: 2018-12-18
```

## Abstract
The purpose of this AIP is to introduce rules and apply penalty for delegates that performed double forging. A protocol change is proposed to notify all nodes when double forged blocks are detected, thus reaching consensus on bad forger.

## Motivation
Handle double forging when it occurs, notify all peers and introduce a network wide system ban. Banning a rogue delegate would enable the network to move forward, without the need to handle fork logic and additional rebuilds, or ending up in splitting the chain up to N chains.

## Rationale
Proposed solution will detect the double forged block and notify all (also blocked) peers about the possible double forgery. Related nodes/peers will act upon received information and decide accordingly to their own logic.

## Specifications
### API: Introduce a new p2p endpoint to inform about possible double forging
A new public API endpoint will be introduced - `POST /blocks/doubleforgery`. This endpoint will receive the double forged detected block. The endpoint call will:
1. Detect and validate double forged block
2. Confirm or Deny double forgery
    Most likely all double forged blocks from the doubleforgery will be received, so it will be easier to accept the lowest id block and ignore others
3. Ban block generator for next 1500 blocks from the height detected by double forging (if confirmed doubleforgery)
4. Handle revert block logic accordingly

Endpoint must be carefully throttled. 

### Double forged block validation
1. We detect the double forged block
2. Detected double forged block is transmitted to all known peers (also blocked and banned) peers by calling the endpoint introduced above
3. Handle revert block logic accordingly

### Revert logic of double forged blocks
1. Double forgery is confirmed
2. We select the double forged block with the lowest id (need to replace if block with higher id is onchain already). All double forged blocks are received at the API endpoint `blocks/doubleforgery`. We pick the one with lowest id, and ignore others
3. Possible lost transactions (transactions from ignored blocks) are to be added back to the pool (if accepted by the transaction guard).

