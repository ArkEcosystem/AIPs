---
  AIP: 38
  Title: BFT Consensus
  Authors: Brian Faust
  Status: *Draft*
  Discussions-To: https://github.com/arkecosystem/AIPS/issues
  Type: *Standards*
  Category: Consensus
  Created: *2021-09-02*
  Last Update: *2021-09-02*
--- 

## Abstract

This AIP proposes a new consensus implementation that will replace the current implementation. This new consensus model requires the signing of proposals and limits participation to active validators. 

## Motivation

The current consensus model has several weaknesses but the most critical one is the way quorum is derived. In order to be able to forge a block you need 66% of nodes on the network to agree on the state of the blockchain. Because all nodes count towards this 66% it is possible to cause the quorum to fail by spinning up a large number of relays that disagree with the state of the blockchain by broadcasting fake information which will cause forgers to believe something is wrong.

## Specification

The main goal of the new consensus is to limit it to validators instead of 66% of all network participants. This makes it less prone to attacks because a large number of validators would have to act maliciously rather than a large number of network participants, which can be relays or validators.

### Proposals

A proposal is at the beginning of a block's lifecycle. It consist of a block number, generator public key, payload, payload hash, signature and timestamp. All of these properties can be used by any node to calculate their own hash of the proposed block to ensure it matches the payload hash that they received.

If all of the data matches the proposal the node will continue by applying a variety of checks. If any of those checks fail the proposal will be rejected in order to avoid invalid data being accepted and persisted.

#### Criteria & Checks

1. If the proposed block doesn't have a height of `lastBlockHeight + 1` the proposal will be rejected.
2. If the proposed block doesn't match the expected generator public key the proposal will be rejected.
3. If the block hash, signature and generator public key don't match the proposal will be rejected.
4. If the block hash that was proposed and the block hash that was computed for verification don't match the proposal will be rejected.
5. If the proposal already exists and the generator public key is the same the proposal will be rejected.
6. If the proposal already exists and the generator public key is different the proposal will be replaced. **This would be the case if a slot has been missed and a new proposal for the same block number is required in order to move forward.**
7. If none of the above criteria match the proposal will be accepted and broadcasted to all peers.
8. Additionally to the previous step the proposal will also be signed with all private keys that are configured on a validator node.

### Signatures

Signatures serve as the vote of validators to indicate that they validated the proposal and approved it. Once a majority of signatures has been collected the proposal will be verified. If the proposal received enough signatures that verify it'll be approved. If it didn't receive enough signatures it'll be rejected.

At least of 66% of validators need to sign a proposal or the validation will not be attempted. This 66% serves as the 2/3 majority to ensure that only validators agree on the state of any proposals made by other validators.

#### Criteria & Checks

1. If the signature length doesn't match 192 characters the signature will be rejected. **The length is variable due to Core using ECDSA for block signing.**
2. If the signature already exists the signature will be rejected.
3. If the signature is not for the block that matches `lastBlockHeight + 1`  the signature will be rejected.
4. If the public key that created the signature is not a participant of the current round the signature will be rejected.
5. If the signature and block hash can't be verified the signature will be rejected.
6. If none of the above criteria match the signature will be accepted and broadcasted to all peers.

### Rationale

The rationale of this AIP is fairly straightforward. Core is in need of a more secure consensus that is limited to validators. All technical and implementation decisions are based on existing code in Core. This means there will be no big cryptographic changes or additions to get this working.

There will be slight performance penalties compared by using ECDSA instead of BIP340 or other algorithms but they won't be noticeable enough at the scale of ARK to warrant a complete overhaul of those aspects and are out of scope of this AIP.

Limiting the consensus to validators in the way that is described in previous sections will make it more difficult to attack the network just by spinning up more relays and also make it more expensive because a large number of delegate spots would be required. Malicious delegates that are not self-voting could be unvoted by voters and thus the threat they posed would be removed.

## Backwards Compatibility

This AIP is not backwards compatible because nodes that run the old consensus won't be able to accept blocks from the new consensus. A hardfork will be caused by this AIP going into production.

## Reference Implementation

A reference implementation can be seen at [faustbrian/odysseia](https://github.com/faustbrian/odysseia). This is a fork of 3.0 with various modifications like the new Consensus. It is implemented with a different cryptography algorithm for signing but the overall logic still applies.
